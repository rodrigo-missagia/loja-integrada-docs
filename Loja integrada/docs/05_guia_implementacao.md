# Guia de Implementação CleverTap - Loja Integrada

**Versão:** 1.1
**Data:** 26 de Janeiro de 2026
**Plataforma:** Web

---

## 1. Configuração Inicial

### 1.1 Credenciais

| Ambiente | Account ID  | Token       | Region      |
| -------- | ----------- | ----------- | ----------- |
| Produção | [A_DEFINIR] | [A_DEFINIR] | [A_DEFINIR] |
| Staging  | [A_DEFINIR] | [A_DEFINIR] | [A_DEFINIR] |

> **Nota:** Preencher com as credenciais fornecidas pelo time CleverTap após criação da conta.

### 1.2 Versão de SDK

| Plataforma | Versão Mínima | Versão Recomendada |
| ---------- | ------------- | ------------------ |
| Web        | 1.6.0         | 2.3.3+             |

---

## 2. Instalação por Plataforma

### 2.1 Web

#### Opção 1: Script Tag (Recomendado)

Adicionar antes do fechamento da tag `</head>`:

```html
<script type="text/javascript">
  var clevertap = {
    event: [],
    profile: [],
    account: [],
    onUserLogin: [],
    notifications: [],
    privacy: [],
  };
  clevertap.account.push({ id: "[ACCOUNT_ID]" });
  clevertap.privacy.push({ optOut: false });
  clevertap.privacy.push({ useIP: false }); // LGPD compliance
  (function () {
    var wzrk = document.createElement("script");
    wzrk.type = "text/javascript";
    wzrk.async = true;
    wzrk.src =
      ("https:" == document.location.protocol
        ? "https://d2r1yp2w7bber2.cloudfront.net"
        : "http://static.clevertap.com") + "/js/clevertap.min.js";
    var s = document.getElementsByTagName("script")[0];
    s.parentNode.insertBefore(wzrk, s);
  })();
</script>
```

#### Opção 2: NPM

```bash
npm install clevertap-web-sdk
```

```javascript
import clevertap from "clevertap-web-sdk";

// Inicialização
clevertap.init("[ACCOUNT_ID]", "[REGION]");
clevertap.privacy.push({ optOut: false });
clevertap.privacy.push({ useIP: false });
```

---

## 3. Identity Management

### 3.1 Estratégia de Identity Composto (Loja + Usuário)

#### Contexto

Na Loja Integrada, um mesmo usuário pode pertencer a múltiplas lojas com estados diferentes em cada uma. Para suportar este cenário, o `Identity` no CleverTap é uma **concatenação do ID da loja com o ID do usuário**:

```text
Identity = {store_id}_{user_id}
```

**Exemplos:**

- `store_12345_user_789` → Usuário 789 no contexto da Loja 12345
- `store_67890_user_789` → Mesmo usuário 789, mas no contexto da Loja 67890

#### Benefícios

- **Contexto único**: Cada par loja-usuário tem seu próprio perfil
- **Sincronização em massa**: Propriedades da loja são atualizadas para todos os usuários via batch
- **Campanhas segmentadas**: Permite segmentar por loja OU por usuário

> **Documentação completa**: Ver [02_user_profile_schema.md](./02_user_profile_schema.md#user-identity-strategy) e [06_user_identity_sync_flow.md](./06_user_identity_sync_flow.md)

### 3.2 onUserLogin (Identificação)

Chamar **sempre** quando o lojista faz login ou cria conta:

```javascript
// Gerar Identity composto
const compositeIdentity = `store_${storeId}_user_${userId}`;

clevertap.onUserLogin.push({
  Site: {
    // Required - Identity COMPOSTO
    Identity: compositeIdentity, // Ex: "store_12345_user_789"

    // Recommended
    Name: ownerName, // Nome do lojista
    Email: email, // Email
    Phone: phoneWithCountryCode, // Ex: +5511999999999

    // IDs separados para lookup
    store_id: storeId, // ID da loja (para queries batch)
    user_id: userId, // ID do usuário (para queries individuais)

    // Custom attributes
    store_name: storeName,
    account_type: accountType, // "pf" ou "pj"
    current_plan: currentPlan, // "free", "starter", etc.
    account_created_date: new Date(),
  },
});
```

### 3.3 Logout

Ao fazer logout, não é necessário chamar método específico. O próximo `onUserLogin` criará uma nova sessão.

### 3.4 Classificação de Propriedades: Loja vs Usuário

As propriedades são classificadas em dois tipos com comportamentos de atualização distintos:

#### Propriedades de LOJA (Store-Level)

Atualizadas em **TODOS os usuários** da loja via batch sync (Airflow).

```javascript
// Exemplo: Loja publicou o site - atualizar TODOS os usuários da loja
// Isso é feito via batch, não no frontend

// Propriedades de loja incluem:
// - current_plan, plan_start_date, billing_cycle
// - enviali_active, shipping_methods_active
// - pagali_account_status, payment_methods_configured
// - site_published, site_publish_date
// - products_count, has_products
// - mercado_livre_connected, ml_sales_count
// - gmv_30d, visitas_30d (métricas agregadas)
```

#### Propriedades de USUÁRIO (User-Level)

Atualizadas **APENAS para o usuário específico** que executou a ação.

```javascript
// Exemplo: Usuário abandonou checkout - atualizar APENAS este usuário
clevertap.profile.push({
  Site: {
    checkout_abandoned: true,
    checkout_abandoned_step: "payment",
    checkout_abandoned_date: new Date(),
  },
});

// Propriedades de usuário incluem:
// - Name, Email, Phone
// - MSG-email, MSG-push (preferências de comunicação)
// - checkout_abandoned, pagali_abandoned_step (jornada individual)
// - komea_access_count, komea_last_access_date (uso individual)
// - ultimo_login_painel (atividade individual)
```

### 3.5 Lookup Table: Loja → Usuários

Para atualizar propriedades de loja em todos os usuários, o backend mantém uma tabela de relacionamento:

| store_id | user_id | composite_identity   | is_active | role   |
| -------- | ------- | -------------------- | --------- | ------ |
| 12345    | 789     | store_12345_user_789 | true      | owner  |
| 12345    | 456     | store_12345_user_456 | true      | admin  |
| 12345    | 123     | store_12345_user_123 | false     | viewer |
| 67890    | 789     | store_67890_user_789 | true      | owner  |

**Uso no batch sync:**

```sql
-- Buscar todos os usuários ativos de uma loja para atualização
SELECT composite_identity
FROM store_users
WHERE store_id = 12345
  AND is_active = true;
```

---

## 4. Implementação de Eventos

### 4.1 Padrão de Eventos

Todos os eventos seguem o padrão:

```javascript
// Estrutura básica
clevertap.event.push("Event Name", {
  property_name: value,
  another_property: value2,
});
```

### 4.2 Eventos de Logística (BC1, BC2, BC3)

#### Enviali Activated

```javascript
function trackEnvialiActivated(source) {
  clevertap.event.push("Enviali Activated", {
    activation_source: source, // "menu_lateral" | "komea"
  });

  // Atualizar perfil
  clevertap.profile.push({
    Site: {
      enviali_active: true,
      enviali_activation_date: new Date(),
    },
  });
}
```

#### Label Purchased

```javascript
function trackLabelPurchased(labelData) {
  clevertap.event.push("Label Purchased", {
    order_id: labelData.orderId,
    carrier_name: labelData.carrierName,
    amount: labelData.amount,
    payment_method: labelData.paymentMethod,
    is_first_purchase: labelData.isFirstPurchase,
    label_id: labelData.labelId,
  });

  // Atualizar perfil
  const profileUpdates = {
    enviali_label_purchased: true,
    enviali_labels_count: { $incr: 1 },
    enviali_last_label_date: new Date(),
    enviali_total_spent: { $incr: labelData.amount },
    enviali_payment_method: labelData.paymentMethod,
  };

  if (labelData.isFirstPurchase) {
    profileUpdates["enviali_first_label_date"] = new Date();
  }

  clevertap.profile.push({ Site: profileUpdates });
}
```

#### Ativação da Loggi

> **Nota:** A ativação da Loggi utiliza o evento genérico `Shipping Method Enabled` com `carrier_name = "loggi"`, conforme decisão de simplificação do tracking plan.

```javascript
function trackLoggiActivated() {
  // Usar Shipping Method Enabled com carrier_name para Loggi
  clevertap.event.push("Shipping Method Enabled", {
    carrier_name: "loggi",
    carrier_type: "private",
    is_correios: false,
    activation_date: new Date().toISOString(),
  });

  clevertap.profile.push({
    Site: {
      loggi_active: true,
      loggi_activation_date: new Date(),
      loggi_configured_not_used: true,
      shipping_methods_active: { $add: ["loggi"] },
    },
  });
}
```

### 4.3 Eventos de Assinatura (BC4)

#### Subscription Completed

```javascript
function trackSubscriptionCompleted(subscriptionData) {
  clevertap.event.push("Subscription Completed", {
    plan_name: subscriptionData.planName,
    billing_cycle: subscriptionData.billingCycle,
    amount: subscriptionData.amount,
    payment_method: subscriptionData.paymentMethod,
    coupon_used: subscriptionData.couponUsed,
    coupon_code: subscriptionData.couponCode || null,
    state: subscriptionData.state,
    city: subscriptionData.city,
    is_upgrade: subscriptionData.isUpgrade,
    previous_plan: subscriptionData.previousPlan || null,
  });

  // Atualizar perfil
  const profileUpdates = {
    current_plan: subscriptionData.planName,
    plan_start_date: new Date(),
    billing_cycle: subscriptionData.billingCycle,
    plan_price: subscriptionData.amount,
    is_paying_customer: true,
    total_subscriptions: { $incr: 1 },
    subscription_value_total: { $incr: subscriptionData.amount },
    last_plan_change_date: new Date(),
    previous_plan: subscriptionData.previousPlan,
    state: subscriptionData.state,
    city: subscriptionData.city,
    checkout_abandoned: false,
  };

  if (subscriptionData.couponUsed) {
    profileUpdates["coupon_used"] = true;
    profileUpdates["last_coupon_code"] = subscriptionData.couponCode;
    profileUpdates["coupons_used_count"] = { $incr: 1 };
  }

  clevertap.profile.push({ Site: profileUpdates });
}
```

### 4.4 Eventos do Pagali (BC6)

#### Pagali Registration Step Completed

```javascript
function trackPagaliStepCompleted(stepName, stepNumber) {
  clevertap.event.push("Pagali Registration Step Completed", {
    step_name: stepName,
    step_number: stepNumber,
  });

  clevertap.profile.push({
    Site: {
      pagali_registration_step: stepName,
    },
  });
}
```

#### Pagali Account Approved

```javascript
function trackPagaliApproved(paymentMethodsEnabled) {
  clevertap.event.push("Pagali Account Approved", {
    payment_methods_enabled: paymentMethodsEnabled,
    approval_date: new Date().toISOString(),
  });

  const profileUpdates = {
    pagali_account_status: "approved",
    pagali_verification_status: "verified",
    pagali_approval_date: new Date(),
  };

  // Marcar cada método habilitado
  if (paymentMethodsEnabled.includes("pix")) {
    profileUpdates["pagali_pix_enabled"] = true;
  }
  if (paymentMethodsEnabled.includes("credit_card")) {
    profileUpdates["pagali_credit_card_enabled"] = true;
  }
  if (paymentMethodsEnabled.includes("boleto")) {
    profileUpdates["pagali_boleto_enabled"] = true;
  }

  profileUpdates["payment_methods_configured"] = {
    $add: paymentMethodsEnabled,
  };

  clevertap.profile.push({ Site: profileUpdates });
}
```

### 4.5 Eventos de Produtos (BC5)

#### Product Created

```javascript
function trackProductCreated(productData) {
  clevertap.event.push("Product Created", {
    product_id: productData.productId,
    creation_method: productData.creationMethod, // "manual" | "ai_komea"
    category: productData.category,
    has_images: productData.hasImages,
    has_variations: productData.hasVariations,
    is_first_product: productData.isFirstProduct,
  });

  const profileUpdates = {
    has_products: true,
    products_count: { $incr: 1 },
    last_product_date: new Date(),
    product_creation_method: productData.creationMethod,
  };

  if (productData.isFirstProduct) {
    profileUpdates["first_product_date"] = new Date();
  }

  if (productData.creationMethod === "ai_komea") {
    profileUpdates["products_created_via_ai"] = { $incr: 1 };
  }

  clevertap.profile.push({ Site: profileUpdates });
}
```

### 4.6 Eventos da Komea (BC7, BC8, BC9)

#### Komea Site Published

```javascript
function trackKomeaSitePublished(publishData) {
  clevertap.event.push("Komea Site Published", {
    has_payment_method: publishData.hasPaymentMethod,
    has_product: publishData.hasProduct,
    time_to_publish: publishData.timeToPublish, // em horas
  });

  clevertap.profile.push({
    Site: {
      site_published: true,
      site_publish_date: new Date(),
      site_publish_source: "komea",
      had_payment_before_publish: publishData.hasPaymentMethod,
    },
  });
}
```

#### Komea Action Executed

```javascript
function trackKomeaActionExecuted(actionData) {
  clevertap.event.push("Komea Action Executed", {
    action_type: actionData.actionType,
    assistant_type: actionData.assistantType,
    result: actionData.result, // "success" | "failed"
  });

  clevertap.profile.push({
    Site: {
      komea_actions_executed: { $incr: 1 },
    },
  });
}
```

### 4.7 Eventos do Mercado Livre (BC11)

#### ML Ad Published

```javascript
function trackMlAdPublished(adData) {
  clevertap.event.push("ML Ad Published", {
    product_id: adData.productId,
    ad_type: adData.adType, // "classic" | "premium"
    ad_id: adData.adId,
    is_first_ad: adData.isFirstAd,
  });

  const profileUpdates = {
    ml_total_ads_count: { $incr: 1 },
    ml_last_ad_date: new Date(),
    ml_first_ad_sent: true,
  };

  if (adData.adType === "premium") {
    profileUpdates["ml_premium_ads_count"] = { $incr: 1 };
  } else {
    profileUpdates["ml_classic_ads_count"] = { $incr: 1 };
  }

  if (adData.isFirstAd) {
    profileUpdates["ml_first_ad_date"] = new Date();
  }

  clevertap.profile.push({ Site: profileUpdates });
}
```

---

## 5. Atualização de Perfil

### 5.1 Operações Disponíveis

#### Set (Substituir)

```javascript
clevertap.profile.push({
  Site: {
    current_plan: "pro",
    last_activity_date: new Date(),
  },
});
```

#### Increment (Somar)

```javascript
clevertap.profile.push({
  Site: {
    products_count: { $incr: 1 },
    total_spent: { $incr: 99.9 },
  },
});
```

#### Append (Adicionar à lista)

```javascript
clevertap.profile.push({
  Site: {
    shipping_methods_active: { $add: ["loggi", "correios_pac"] },
    komea_assistants_used: { $add: ["data_assistant"] },
  },
});
```

#### Remove (Remover da lista)

```javascript
clevertap.profile.push({
  Site: {
    shipping_methods_active: { $remove: ["jadlog"] },
  },
});
```

### 5.2 Boas Práticas

1. **Agrupar updates**: Quando possível, agrupar múltiplas atualizações em uma única chamada
2. **Usar increment para contadores**: Nunca sobrescrever contadores, sempre incrementar
3. **Validar tipos**: Garantir que tipos estão corretos (string, number, boolean, date)
4. **Limites de arrays**: Arrays suportam máximo de 100 itens

---

## 6. Web Push Notifications

### 6.1 Service Worker

Criar arquivo `clevertap_sw.js` na raiz do site:

```javascript
importScripts("https://d2r1yp2w7bber2.cloudfront.net/js/sw_webpush.js");
```

### 6.2 Solicitar Permissão

```javascript
// Solicitar permissão com dialog nativo
clevertap.notifications.push({
  skipDialog: true,
});

// Ou com dialog customizado
clevertap.notifications.push({
  titleText: "Quer receber notificações?",
  bodyText: "Fique por dentro das novidades da sua loja",
  okButtonText: "Sim",
  rejectButtonText: "Não",
});
```

---

## 7. Deep Links

| Ação                 | Deep Link                        |
| -------------------- | -------------------------------- |
| Ativar Enviali       | `loja://enviali/activate`        |
| Setup Enviali        | `loja://enviali/setup`           |
| Transportadoras      | `loja://enviali/carriers`        |
| Gerenciar etiquetas  | `loja://enviali/labels`          |
| Etiqueta específica  | `loja://orders/{order_id}/label` |
| Página de planos     | `loja://plans`                   |
| Checkout de plano    | `loja://checkout/{plan_id}`      |
| Cadastro Pagali      | `loja://pagali/register`         |
| Criar produto        | `loja://products/new`            |
| Komea                | `loja://komea`                   |
| Personalização Komea | `loja://komea/customize`         |
| Oportunidades Komea  | `loja://komea/opportunities`     |
| Hub de Canais        | `loja://hub`                     |
| Mercado Livre        | `loja://hub/mercadolivre`        |
| Anúncios ML          | `loja://hub/mercadolivre/ads`    |

---

## 8. Validação e Debug

### 8.1 Modo Debug

Os eventos serão logados automaticamente no console do navegador.

### 8.2 User Lookup

1. Acessar **CleverTap Dashboard > Segments > Find People**
2. Buscar por Identity ou Email
3. Verificar:
   - Eventos recentes
   - Propriedades de perfil
   - Tokens de push

### 8.3 Event Viewer

1. Acessar **CleverTap Dashboard > Events**
2. Buscar evento específico
3. Verificar:
   - Quantidade de disparos
   - Propriedades enviadas
   - Erros ou anomalias

### 8.4 Checklist de Validação

| Item                     | Validação                    | Status |
| ------------------------ | ---------------------------- | ------ |
| SDK inicializa sem erros | Logs no console              | ☐      |
| onUserLogin funciona     | User Lookup mostra Identity  | ☐      |
| Eventos core registram   | Events aparecem no dashboard | ☐      |
| Propriedades corretas    | Tipos e valores válidos      | ☐      |
| Web Push funciona        | Recebe teste do dashboard    | ☐      |
| Perfil atualiza          | Atributos mudam após eventos | ☐      |

---

## 9. Troubleshooting

### Eventos não aparecem no dashboard

1. **Verificar credenciais**: Account ID e Token estão corretos?
2. **Verificar região**: Região (eu1, in1, sg1, us1) está correta?
3. **Aguardar**: Dashboard pode levar até 5 minutos para refletir eventos
4. **Verificar logs**: Habilitar debug mode e verificar erros
5. **Verificar conectividade**: Dispositivo está online?

### Web Push não funciona

1. Verificar se o Service Worker está registrado corretamente
2. Verificar se o usuário concedeu permissão de notificações
3. Verificar configuração HTTPS (obrigatório para Web Push)
4. Verificar se token está registrado (User Lookup)

### Perfil não atualiza

1. `onUserLogin` deve ser chamado **uma vez** por sessão
2. Para updates subsequentes, usar `profile.push`
3. Verificar tipos de dados (especialmente datas)
4. Arrays têm limite de 100 itens
5. Strings têm limite de 512 caracteres

### Duplicação de usuários

1. Sempre usar mesmo Identity para o mesmo usuário
2. Identity deve ser único e imutável (store_id)
3. Não chamar onUserLogin múltiplas vezes na mesma sessão

---

## 10. LGPD e Privacidade

### 10.1 Configuração de Privacidade

```javascript
// Desabilitar uso de IP para geolocalização
clevertap.privacy.push({ useIP: false });

// Opt-out completo (se usuário solicitar)
clevertap.privacy.push({ optOut: true });
```

### 10.2 Opt-out de Canais

```javascript
// Opt-out de email
clevertap.profile.push({
  Site: {
    "MSG-email": false,
  },
});

// Opt-out de push
clevertap.profile.push({
  Site: {
    "MSG-push": false,
  },
});
```

### 10.3 Exclusão de Dados

Para exclusão de dados conforme LGPD, entrar em contato com o suporte CleverTap através do dashboard.

---

## 11. Contatos e Suporte

| Recurso                | Link/Contato                    |
| ---------------------- | ------------------------------- |
| Documentação CleverTap | https://developer.clevertap.com |
| Dashboard CleverTap    | https://dashboard.clevertap.com |
| Suporte CleverTap      | Via dashboard > Help            |
| Time Loja Integrada    | [A_DEFINIR]                     |

---

## Changelog

| Data       | Versão | Alteração                            | Autor |
| ---------- | ------ | ------------------------------------ | ----- |
| 05/01/2026 | 1.0    | Versão inicial com 11 Business Cases | RMH   |
