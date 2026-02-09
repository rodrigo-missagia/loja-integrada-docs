# Guia de Implementação CleverTap - Loja Integrada

**Versão:** 1.3
**Data:** 09 de Fevereiro de 2026
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

> **Documentação completa**: Ver [02_user_profile_schema_simplified.md](./02_user_profile_schema_simplified.md) e [06_user_identity_sync_flow.md](./06_user_identity_sync_flow.md)

### 3.2 onUserLogin (Identificação)

Chamar **sempre** quando o lojista faz login ou cria conta:

```javascript
// Gerar Identity composto
const compositeIdentity = `store_${storeId}_user_${userId}`;

clevertap.onUserLogin.push({
  Site: {
    // Required - Identity COMPOSTO
    Identity: compositeIdentity, // Ex: "store_12345_user_789"

    // Recommended - Dados de contato (User-Level)
    Name: ownerName, // Nome do lojista
    Email: email, // Email
    Phone: phoneWithCountryCode, // Ex: +5511999999999

    // IDs separados para lookup
    store_id: storeId, // ID da loja (para queries batch)
    user_id: userId, // ID do usuário (para queries individuais)
  },
});

// Nota: store_name, account_type, current_plan e demais propriedades
// store-level são sincronizadas via DAG Airflow diário (Backend).
// Ver 06_user_identity_sync_flow.md para detalhes.
```

### 3.3 Logout

Ao fazer logout, não é necessário chamar método específico. O próximo `onUserLogin` criará uma nova sessão.

### 3.4 Classificação de Propriedades: Store-Level vs User-Level vs Dual-Write

As propriedades são classificadas em três categorias com mecanismos de atualização distintos:

#### Propriedades STORE-LEVEL (Backend — DAG Airflow diário)

Atualizadas em **TODOS os usuários** da loja via batch sync (Airflow). **Não devem ser atualizadas via frontend.**

```text
Propriedades Store-Level (20):
  Conta:       store_id, store_name, account_type, state, city
  Plano:       plan_start_date
  Enviali:     enviali_active, shipping_methods_active, correios_direct_contract, enviali_balance_amount
  Loggi:       loggi_active
  Pagali:      pagali_account_status, payment_methods_configured, mercado_pago_configured
  Produtos:    has_products
  Komea/Site:  site_published
  Marketplace: mercado_livre_connected
  Fiscal:      nfe_configured, tax_regime
  Métricas:    gmv_30d, visitas_30d, qtde_pedido_30d
```

> **Nota:** O frontend **não** faz `profile.push()` para estas propriedades. Ver [06_user_identity_sync_flow.md](./06_user_identity_sync_flow.md) para detalhes da DAG.

#### Propriedades DUAL-WRITE (Frontend imediato + DAG diário)

Propriedades de assinatura atualizadas **imediatamente** para o usuário ativo via SDK, e sincronizadas para **todos** os usuários da loja via DAG diário.

```text
Propriedades Dual-Write (3):
  current_plan, billing_cycle, is_paying_customer
```

> **Motivo:** Quando o lojista contrata/muda de plano, o perfil dele precisa refletir imediatamente (para campanhas de confirmação). Mas outros usuários da mesma loja também precisam do plano atualizado — isso é feito pela DAG.

#### Propriedades USER-LEVEL (Frontend SDK)

Atualizadas **APENAS para o usuário específico** que executou a ação.

```javascript
// Exemplo: Usuário acessou a Komea
clevertap.profile.push({
  Site: {
    komea_access_count: { $incr: 1 },
    komea_last_access_date: new Date(),
  },
});

// Propriedades User-Level:
// - Name, Email, Phone (dados de contato)
// - MSG-email, MSG-push (preferências de comunicação)
// - komea_access_count, komea_last_access_date (uso individual)
```

> **Nota:** Fluxos de abandono (checkout, cadastro Pagali) devem usar **segmentação Inaction no CleverTap** (evento de início "Did" AND evento de conclusão "Did not") em vez de propriedades de perfil.

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

#### Shipping Platform Activated

```javascript
function trackShippingPlatformActivated(platform, source, fieldsCompleted) {
  clevertap.event.push("Shipping Platform Activated", {
    shipping_platform: platform, // "enviali", "fretnet", "melhor_envio", etc.
    activation_source: source, // "menu_lateral" | "komea"
    fields_completed: fieldsCompleted, // ["address", "contact", "store_data"]
  });

  // Nota: enviali_active e demais propriedades store-level são
  // atualizadas via DAG Airflow diário (Backend).
}
```

#### Label Purchased

```javascript
function trackLabelPurchased(labelData) {
  clevertap.event.push("Label Purchased", {
    shipping_platform: labelData.shippingPlatform,
    order_id: labelData.orderId,
    carrier_name: labelData.carrierName,
    carrier_type: labelData.carrierType, // "postal" | "private"
    amount: labelData.amount,
    payment_method: labelData.paymentMethod,
    delivery_time: labelData.deliveryTime, // prazo em dias úteis
    is_first_purchase: labelData.isFirstPurchase,
    label_id: labelData.labelId,
  });

  // Nota: Propriedades de perfil (contadores, datas) foram removidas.
  // Segmentação por contagem/data de etiquetas deve usar o próprio evento.
  // Ver 02_user_profile_schema_simplified.md > "Segmentação via Eventos".

  // Nota: Este evento de monetização deve ser acompanhado de um evento
  // "Charged" para o dashboard de receita do CleverTap. Ver seção 4.8.
}
```

#### Ativação da Loggi

> **Nota:** A ativação da Loggi utiliza o evento genérico `Shipping Method Enabled` com `carrier_name = "loggi"`, conforme decisão de simplificação do tracking plan.

```javascript
function trackLoggiActivated() {
  // Usar Shipping Method Enabled com carrier_name para Loggi
  clevertap.event.push("Shipping Method Enabled", {
    shipping_platform: "enviali",
    carrier_name: "loggi",
    carrier_type: "private",
    is_correios: false,
  });

  // Nota: loggi_active, shipping_methods_active e demais propriedades
  // store-level são atualizadas via DAG Airflow diário (Backend).
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
    discount_percentage: subscriptionData.discountPercentage || null,
    state: subscriptionData.state,
    city: subscriptionData.city,
    is_upgrade: subscriptionData.isUpgrade,
    previous_plan: subscriptionData.previousPlan || null,
  });

  // DUAL-WRITE: Atualizar perfil APENAS com as 3 props de assinatura.
  // Estas props são atualizadas imediatamente para o usuário ativo via SDK,
  // e sincronizadas para TODOS os usuários da loja via DAG Airflow diário.
  // Ver 06_user_identity_sync_flow.md > seção "DUAL-WRITE".
  clevertap.profile.push({
    Site: {
      current_plan: subscriptionData.planName,
      billing_cycle: subscriptionData.billingCycle,
      is_paying_customer: true,
    },
  });

  // Nota: Este evento de monetização deve ser acompanhado de um evento
  // "Charged" para o dashboard de receita do CleverTap. Ver seção 4.8.
}
```

### 4.4 Eventos de Gateway de Pagamento (BC6)

#### Gateway Registration Started

```javascript
function trackGatewayRegistrationStarted(gateway, entrySource) {
  clevertap.event.push("Gateway Registration Started", {
    payment_gateway: gateway, // "pagali", "mercado_pago", "app_max", etc.
    entry_source: entrySource, // "komea" | "panel" | "direct"
  });

  // Nota: pagali_account_status e demais propriedades store-level são
  // atualizadas via DAG Airflow diário (Backend).
}
```

#### Gateway Registration Completed

```javascript
function trackGatewayRegistrationCompleted(gateway, accountType, documentsSubmitted, stepsCompleted) {
  clevertap.event.push("Gateway Registration Completed", {
    payment_gateway: gateway,
    account_type: accountType,
    documents_submitted: documentsSubmitted,
    steps_completed: stepsCompleted,
  });

  // Nota: pagali_account_status e demais propriedades store-level são
  // atualizadas via DAG Airflow diário (Backend).
}
```

#### Gateway Account Approved

```javascript
function trackGatewayAccountApproved(gateway, paymentMethodsEnabled) {
  clevertap.event.push("Gateway Account Approved", {
    payment_gateway: gateway,
    payment_methods_enabled: paymentMethodsEnabled,
    approval_date: new Date().toISOString(),
  });

  // Nota: pagali_account_status, payment_methods_configured e demais
  // propriedades store-level são atualizadas via DAG Airflow diário (Backend).
}
```

#### Gateway Account Rejected

```javascript
function trackGatewayAccountRejected(gateway, rejectionReason) {
  clevertap.event.push("Gateway Account Rejected", {
    payment_gateway: gateway,
    rejection_reason: rejectionReason,
  });

  // Nota: pagali_account_status é atualizado via DAG Airflow diário (Backend).
}
```

#### Payment Method Enabled

```javascript
function trackPaymentMethodEnabled(gateway, paymentMethodType) {
  clevertap.event.push("Payment Method Enabled", {
    payment_gateway: gateway,
    payment_method_type: paymentMethodType, // "pix" | "credit_card" | "boleto"
  });

  // Nota: payment_methods_configured é atualizado via DAG Airflow diário (Backend).
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

  // Nota: has_products é atualizado via DAG Airflow diário (Backend).
  // Contadores e datas de produto são segmentáveis via evento.
}
```

### 4.6 Eventos da Komea (BC7, BC8, BC9)

#### Komea Accessed

```javascript
function trackKomeaAccessed(entrySource, sessionNumber) {
  clevertap.event.push("Komea Accessed", {
    entry_source: entrySource, // "menu" | "dashboard" | "notification"
    session_number: sessionNumber,
  });

  // USER-LEVEL: Atualizar perfil com props individuais do usuário.
  // Estas propriedades são legítimas no frontend pois representam
  // atividade individual (não da loja).
  clevertap.profile.push({
    Site: {
      komea_access_count: { $incr: 1 },
      komea_last_access_date: new Date(),
    },
  });
}
```

#### Komea Site Published

```javascript
function trackKomeaSitePublished(publishData) {
  clevertap.event.push("Komea Site Published", {
    has_payment_method: publishData.hasPaymentMethod,
    has_product: publishData.hasProduct,
    time_to_publish: publishData.timeToPublish, // em horas
  });

  // Nota: site_published é atualizado via DAG Airflow diário (Backend).
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

  // Nota: Contagem de ações é segmentável via count do evento.
}
```

### 4.7 Eventos de Marketplace (BC11)

#### Marketplace Ad Published

```javascript
function trackMarketplaceAdPublished(marketplace, adData) {
  clevertap.event.push("Marketplace Ad Published", {
    marketplace: marketplace, // "mercado_livre", "magalu", "allever", "compre_sua_peca"
    product_id: adData.productId,
    ad_type: adData.adType, // "classic" | "premium" (específico do marketplace)
    ad_id: adData.adId,
    is_first_ad: adData.isFirstAd,
  });

  // Nota: Propriedades de perfil ml_* foram removidas do schema simplificado.
  // Contadores e datas de anúncios são segmentáveis via evento.
  // mercado_livre_connected é atualizado via DAG Airflow diário (Backend).

  // Nota: Este evento de monetização deve ser acompanhado de um evento
  // "Charged" para o dashboard de receita do CleverTap. Ver seção 4.8.
}
```

### 4.8 Evento Charged — Padrão de Monetização

O CleverTap usa o evento reservado `Charged` para rastreamento de receita. Todo evento de monetização (marcado com ⭐ no tracking plan) deve disparar um `Charged` correspondente.

> **Documentação completa:** Ver [03_event_specifications.md](./03_event_specifications.md) para implementação detalhada de cada evento Charged.

#### Exemplo: Subscription Completed → Charged

```javascript
// Após disparar "Subscription Completed", disparar o Charged:
clevertap.event.push("Charged", {
  Amount: subscriptionData.amount,
  Currency: "BRL",
  "Payment Mode": subscriptionData.paymentMethod,
  "Charged ID": "sub_" + Date.now(),
  Items: [
    {
      Name: `Plano ${subscriptionData.planName}`,
      Category: "subscription",
      "Billing Cycle": subscriptionData.billingCycle,
      "Coupon Code": subscriptionData.couponCode || null,
    },
  ],
});
```

#### Eventos que geram Charged

| Evento Original              | Categoria de Receita   |
| ---------------------------- | ---------------------- |
| `Subscription Completed`     | Assinatura de plano    |
| `Label Purchased`            | Etiqueta de envio      |
| `Shipping Balance Added`     | Saldo plataforma envio |
| `Marketplace Ad Published`   | Comissão marketplace   |
| `Marketplace Sale Completed` | Comissão marketplace   |

---

## 5. Atualização de Perfil

### 5.1 Operações Disponíveis

#### Set (Substituir)

```javascript
clevertap.profile.push({
  Site: {
    current_plan: "aceleração",
    billing_cycle: "annual",
    is_paying_customer: true,
  },
});
```

#### Increment (Somar)

```javascript
clevertap.profile.push({
  Site: {
    komea_access_count: { $incr: 1 },
  },
});
```

#### Append (Adicionar à lista) — Apenas Backend/DAG

```javascript
// Nota: operações append em shipping_methods_active e payment_methods_configured
// são executadas exclusivamente via DAG Airflow (Backend).
// Exemplo do payload que a DAG envia via Upload API:
// { "shipping_methods_active": { "$add": ["loggi", "correios_pac"] } }
// { "payment_methods_configured": { "$add": ["pix", "credit_card"] } }
```

#### Remove (Remover da lista) — Apenas Backend/DAG

```javascript
// Nota: operações remove também são executadas via DAG Airflow (Backend).
// Exemplo: { "shipping_methods_active": { "$remove": ["jadlog"] } }
```

### 5.2 Boas Práticas

1. **Frontend apenas para User-Level e Dual-Write**: O SDK só deve fazer `profile.push()` para propriedades user-level (Komea) e dual-write (Subscription). Todas as demais são Backend.
2. **Usar increment para contadores**: Nunca sobrescrever contadores, sempre incrementar (ex: `komea_access_count`)
3. **Validar tipos**: Garantir que tipos estão corretos (string, number, boolean, date)
4. **Limites de arrays**: Arrays suportam máximo de 100 itens
5. **Segmentação por eventos**: Preferir segmentação via eventos para datas (first/last), contadores e histórico — ver [02_user_profile_schema_simplified.md](./02_user_profile_schema_simplified.md)

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
| Magalu               | `loja://hub/magalu`              |
| Allever              | `loja://hub/allever`             |
| Compre Sua Peça      | `loja://hub/compre-sua-peca`     |

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

| Data       | Versão | Alteração                                                                                                   | Autor |
| ---------- | ------ | ----------------------------------------------------------------------------------------------------------- | ----- |
| 05/01/2026 | 1.0    | Versão inicial com 11 Business Cases                                                                        | RMH   |
| 09/02/2026 | 1.1    | Eventos BC6 gateway-agnósticos + nomes de planos atualizados                                                | RMH   |
| 09/02/2026 | 1.2    | Eventos BC11 marketplace-agnósticos (ML * → Marketplace *)                                                  | RMH   |
| 09/02/2026 | 1.3    | Revisão integral: schema simplificado, ~40 profile.push removidos, Backend-first/DUAL-WRITE, Komea Accessed | RMH   |
