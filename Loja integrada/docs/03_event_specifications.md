# Event Specifications - Loja Integrada

**Versão:** 1.1
**Data:** 26 de Janeiro de 2026
**Plataforma:** CleverTap

---

## Registro de IDs Removidos

Os seguintes IDs de eventos foram removidos/consolidados e não devem ser reutilizados:

| ID Removido | Evento Original                    | Motivo           | Substituído Por                                      |
| ----------- | ---------------------------------- | ---------------- | ---------------------------------------------------- |
| EVT-0XX     | `Loggi Activated`                  | Consolidação     | `Shipping Method Enabled` com `carrier_name="loggi"` |
| EVT-0XX     | `Loggi Label Purchased`            | Consolidação     | `Label Purchased` com `carrier_name="loggi"`         |
| EVT-0XX     | `Mercado Livre Connection Started` | Padronização ML  | `ML Connection Started`                              |
| EVT-0XX     | `Mercado Livre Connected`          | Padronização ML  | `ML Connected`                                       |
| EVT-0XX     | Eventos `*Abandoned`               | Uso de Inaction  | Segmentação Inaction no CleverTap                    |
| EVT-0XX     | `Invoice Emitted`                  | Padronização NFe | `NFe Emitted`                                        |

> **Nota:** IDs marcados como EVT-0XX serão preenchidos quando identificados os IDs originais. Novos eventos devem usar IDs sequenciais a partir do último ID válido.

---

## Sumário

1. [Eventos de Envio e Logística](#1-eventos-de-envio-e-logística-bc1-bc2-bc3)
2. [Eventos de Pagamentos e Assinatura](#2-eventos-de-pagamentos-e-assinatura-bc4-bc6)
3. [Eventos de Produtos](#3-eventos-de-produtos-bc5)
4. [Eventos da Komea](#4-eventos-da-komea-bc7-bc8-bc9)
5. [Eventos de Canais de Venda](#5-eventos-de-canais-de-venda-bc11)
6. [Eventos Fiscais](#6-eventos-fiscais-bc10)

---

## 1. Eventos de Envio e Logística (BC1, BC2, BC3)

### Enviali Activated

**ID:** EVT-001
**Categoria:** Logística
**Criticidade:** Alta
**Business Case:** BC1

#### Descrição

Disparado quando o lojista ativa o Enviali em sua loja, clicando em "Ativar na minha loja".

#### Trigger

Clique no botão "Ativar na minha loja" na página do Enviali.

#### Propriedades

| Propriedade         | Tipo          | Obrigatório | Descrição                      | Exemplo                                |
| ------------------- | ------------- | ----------- | ------------------------------ | -------------------------------------- |
| `activation_source` | String        | Sim         | Origem da ativação             | `"menu_lateral"`, `"komea"`            |
| `fields_completed`  | Array[String] | Sim         | Campos preenchidos na ativação | `["address", "contact", "store_data"]` |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Enviali Activated", {
  activation_source: "menu_lateral",
  fields_completed: ["address", "contact", "store_data"],
});

// Atualizar perfil
clevertap.profile.push({
  Site: {
    enviali_active: true,
    enviali_activation_date: new Date(),
    enviali_initial_data_filled: true,
  },
});
```

#### Campanhas Relacionadas

- Winback de configuração incompleta
- Incentivo à ativação de transportadoras

---

### Shipping Method Enabled

**ID:** EVT-003
**Categoria:** Logística
**Criticidade:** Alta
**Business Case:** BC1

#### Descrição

Disparado quando o lojista ativa um método de envio (Correios ou transportadora) no Enviali.

#### Trigger

Ativação de qualquer transportadora ou serviço dos Correios.

#### Propriedades

| Propriedade        | Tipo          | Obrigatório | Descrição                       | Exemplo                             |
| ------------------ | ------------- | ----------- | ------------------------------- | ----------------------------------- |
| `carrier_name`     | String        | Sim         | Nome da transportadora          | `"correios"`, `"loggi"`, `"jadlog"` |
| `carrier_type`     | String        | Sim         | Tipo da transportadora          | `"postal"`, `"private"`             |
| `is_correios`      | Boolean       | Sim         | É dos Correios                  | `true`                              |
| `services_enabled` | Array[String] | Não         | Serviços habilitados (Correios) | `["pac", "sedex"]`                  |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Shipping Method Enabled", {
  carrier_name: "correios",
  carrier_type: "postal",
  is_correios: true,
  services_enabled: ["pac", "sedex"],
});

clevertap.profile.push({
  Site: {
    shipping_methods_active: { $add: ["correios_pac", "correios_sedex"] },
    shipping_methods_count: { $incr: 2 },
  },
});
```

---

### Label Flow Started

**ID:** EVT-004
**Categoria:** Logística
**Criticidade:** Média
**Business Case:** BC2

#### Descrição

Disparado quando o lojista inicia o fluxo de emissão de etiquetas.

#### Trigger

Acesso à listagem de pedidos ou gerenciador de etiquetas com intenção de emissão.

#### Propriedades

| Propriedade   | Tipo   | Obrigatório | Descrição                    | Exemplo                           |
| ------------- | ------ | ----------- | ---------------------------- | --------------------------------- |
| `flow_source` | String | Sim         | Origem do fluxo              | `"order_list"`, `"label_manager"` |
| `order_id`    | String | Não         | ID do pedido (se específico) | `"order_123456"`                  |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Label Flow Started", {
  flow_source: "order_list",
  order_id: "order_123456",
});

clevertap.profile.push({
  Site: {
    enviali_label_flow_accessed: true,
    enviali_label_flow_source: "order_list",
  },
});
```

---

### Label Purchased 💰

**ID:** EVT-005
**Categoria:** Logística / Monetização
**Criticidade:** Alta
**Business Case:** BC2

#### Descrição

Disparado quando a compra da etiqueta é concluída com sucesso. Este é um evento de monetização. Consolida todas as transportadoras (Correios, Loggi, Jadlog, etc.) usando a propriedade `carrier_name`.

#### Trigger

Confirmação do sistema de que a etiqueta foi comprada.

#### Propriedades

| Propriedade         | Tipo    | Obrigatório | Descrição                      | Exemplo                                 |
| ------------------- | ------- | ----------- | ------------------------------ | --------------------------------------- |
| `order_id`          | String  | Sim         | ID do pedido                   | `"order_123456"`                        |
| `carrier_name`      | String  | Sim         | Transportadora                 | `"correios_pac"`, `"loggi"`, `"jadlog"` |
| `carrier_type`      | String  | Sim         | Tipo da transportadora         | `"postal"`, `"private"`                 |
| `amount`            | Number  | Sim         | Valor pago                     | `18.50`                                 |
| `payment_method`    | String  | Sim         | Método de pagamento            | `"balance"`, `"card"`, `"pix"`          |
| `delivery_time`     | Number  | Sim         | Prazo de entrega em dias úteis | `5`                                     |
| `is_first_purchase` | Boolean | Sim         | Primeira compra de etiqueta    | `true`                                  |
| `label_id`          | String  | Sim         | ID da etiqueta                 | `"label_789"`                           |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Label Purchased", {
  order_id: "order_123456",
  carrier_name: "correios_pac",
  carrier_type: "postal",
  amount: 18.5,
  payment_method: "balance",
  delivery_time: 5,
  is_first_purchase: true,
  label_id: "label_789",
});

// Evento Charged (monetização)
clevertap.event.push("Charged", {
  Amount: 18.5,
  Currency: "BRL",
  "Payment Mode": "balance",
  "Charged ID": "label_789",
  Items: [
    {
      Name: "Etiqueta Correios PAC",
      Category: "shipping_label",
      Carrier: "correios_pac",
      "Order ID": "order_123456",
    },
  ],
});

// Atualizar perfil
clevertap.profile.push({
  Site: {
    enviali_label_purchased: true,
    enviali_labels_count: { $incr: 1 },
    enviali_last_label_date: new Date(),
    enviali_total_spent: { $incr: 18.5 },
    enviali_payment_method: "balance",
  },
});

// Se for primeira compra
if (isFirstPurchase) {
  clevertap.profile.push({
    Site: {
      enviali_first_label_date: new Date(),
    },
  });
}

// Se for Loggi, atualizar atributos específicos
if (carrierName === "loggi") {
  clevertap.profile.push({
    Site: {
      loggi_labels_count: { $incr: 1 },
    },
  });
}
```

#### Campanhas Relacionadas

- Confirmação de compra realizada
- Estímulo à recorrência de envios

> **Nota:** Este evento consolida `Loggi Label Purchased` anterior. Para filtrar etiquetas Loggi, use `carrier_name = "loggi"` na segmentação.

---

### Enviali Balance Added 💰

**ID:** EVT-010
**Categoria:** Logística / Monetização
**Criticidade:** Alta
**Business Case:** BC2

#### Descrição

Disparado quando o lojista adiciona saldo ao Enviali.

#### Trigger

Conclusão do pagamento para adicionar saldo.

#### Propriedades

| Propriedade      | Tipo   | Obrigatório | Descrição           | Exemplo           |
| ---------------- | ------ | ----------- | ------------------- | ----------------- |
| `amount`         | Number | Sim         | Valor adicionado    | `100.00`          |
| `payment_method` | String | Sim         | Método de pagamento | `"card"`, `"pix"` |
| `new_balance`    | Number | Sim         | Novo saldo total    | `150.00`          |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Enviali Balance Added", {
  amount: 100.0,
  payment_method: "pix",
  new_balance: 150.0,
});

// Evento Charged (monetização)
clevertap.event.push("Charged", {
  Amount: 100.0,
  Currency: "BRL",
  "Payment Mode": "pix",
  "Charged ID": "balance_" + Date.now(),
  Items: [
    {
      Name: "Saldo Enviali",
      Category: "balance_topup",
      "New Balance": 150.0,
    },
  ],
});

clevertap.profile.push({
  Site: {
    enviali_has_balance: true,
    enviali_balance_amount: 150.0,
    enviali_balance_added: true,
  },
});
```

---

### Shipping Quote Requested

**ID:** EVT-013
**Categoria:** Logística
**Criticidade:** Média
**Business Case:** BC2, BC3

#### Descrição

Disparado quando há uma cotação de frete via Enviali no checkout do cliente final.

#### Trigger

Solicitação de cotação de frete no checkout da loja.

#### Propriedades

| Propriedade       | Tipo          | Obrigatório | Descrição               | Exemplo                     |
| ----------------- | ------------- | ----------- | ----------------------- | --------------------------- |
| `order_id`        | String        | Sim         | ID do pedido/carrinho   | `"cart_789"`                |
| `origin_zip`      | String        | Sim         | CEP de origem           | `"01310100"`                |
| `destination_zip` | String        | Sim         | CEP de destino          | `"04538132"`                |
| `carriers_quoted` | Array[String] | Sim         | Transportadoras cotadas | `["correios_pac", "loggi"]` |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Shipping Quote Requested", {
  order_id: "cart_789",
  origin_zip: "01310100",
  destination_zip: "04538132",
  carriers_quoted: ["correios_pac", "correios_sedex", "loggi"],
});

clevertap.profile.push({
  Site: {
    enviali_quote_received: true,
  },
});
```

---

### Shipping Quote Carrier Selected

**ID:** EVT-014
**Categoria:** Logística
**Criticidade:** Média
**Business Case:** BC2, BC3

#### Descrição

Disparado quando o cliente final seleciona uma transportadora no checkout.

#### Trigger

Seleção de opção de frete pelo cliente no checkout.

#### Propriedades

| Propriedade     | Tipo   | Obrigatório | Descrição                  | Exemplo          |
| --------------- | ------ | ----------- | -------------------------- | ---------------- |
| `order_id`      | String | Sim         | ID do pedido               | `"order_123456"` |
| `carrier_name`  | String | Sim         | Transportadora selecionada | `"loggi"`        |
| `price`         | Number | Sim         | Preço do frete             | `12.90`          |
| `delivery_time` | Number | Sim         | Prazo em dias              | `3`              |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Shipping Quote Carrier Selected", {
  order_id: "order_123456",
  carrier_name: "loggi",
  price: 12.9,
  delivery_time: 3,
});

clevertap.profile.push({
  Site: {
    checkout_carrier_selected: "loggi",
  },
});

// Se for Loggi
if (carrierName === "loggi") {
  clevertap.profile.push({
    Site: {
      loggi_checkout_quotes: { $incr: 1 },
    },
  });
}
```

#### Campanhas Relacionadas

- Conversão de cotação em envio

---

### Order Shipped

**ID:** EVT-015
**Categoria:** Logística
**Criticidade:** Alta
**Business Case:** BC2

#### Descrição

Disparado quando o pedido é marcado como enviado/postado.

#### Trigger

Confirmação de postagem do pedido.

#### Propriedades

| Propriedade     | Tipo   | Obrigatório | Descrição                | Exemplo           |
| --------------- | ------ | ----------- | ------------------------ | ----------------- |
| `order_id`      | String | Sim         | ID do pedido             | `"order_123456"`  |
| `carrier_name`  | String | Sim         | Transportadora utilizada | `"correios_pac"`  |
| `tracking_code` | String | Sim         | Código de rastreio       | `"BR123456789BR"` |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Order Shipped", {
  order_id: "order_123456",
  carrier_name: "correios_pac",
  tracking_code: "BR123456789BR",
});

clevertap.profile.push({
  Site: {
    enviali_shipments_count: { $incr: 1 },
  },
});

// Atualizar milestone
updateShipmentMilestone(shipmentsCount);
```

---

## 2. Eventos de Pagamentos e Assinatura (BC4, BC6)

### Plans Page Viewed

**ID:** EVT-016
**Categoria:** Monetização
**Criticidade:** Alta
**Business Case:** BC4

#### Descrição

Disparado quando o lojista visualiza a página de planos.

#### Trigger

Acesso à página de planos/preços.

#### Propriedades

| Propriedade    | Tipo   | Obrigatório | Descrição        | Exemplo                           |
| -------------- | ------ | ----------- | ---------------- | --------------------------------- |
| `current_plan` | String | Sim         | Plano atual      | `"free"`                          |
| `referrer`     | String | Não         | Origem do acesso | `"dashboard"`, `"upgrade_banner"` |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Plans Page Viewed", {
  current_plan: "free",
  referrer: "upgrade_banner",
});
```

---

### Plan Selected

**ID:** EVT-017
**Categoria:** Monetização
**Criticidade:** Alta
**Business Case:** BC4

#### Descrição

Disparado quando o lojista seleciona um plano para contratar.

#### Trigger

Clique em "Selecionar" ou "Assinar" em um plano.

#### Propriedades

| Propriedade     | Tipo   | Obrigatório | Descrição         | Exemplo                 |
| --------------- | ------ | ----------- | ----------------- | ----------------------- |
| `plan_name`     | String | Sim         | Nome do plano     | `"pro"`                 |
| `billing_cycle` | String | Sim         | Ciclo de cobrança | `"monthly"`, `"annual"` |
| `price`         | Number | Sim         | Preço do plano    | `79.90`                 |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Plan Selected", {
  plan_name: "pro",
  billing_cycle: "monthly",
  price: 79.9,
});
```

---

### Checkout Started

**ID:** EVT-018
**Categoria:** Monetização
**Criticidade:** Alta
**Business Case:** BC4

#### Descrição

Disparado quando o lojista inicia o checkout de assinatura.

#### Trigger

Entrada na página de checkout/pagamento.

#### Propriedades

| Propriedade      | Tipo   | Obrigatório | Descrição                  | Exemplo     |
| ---------------- | ------ | ----------- | -------------------------- | ----------- |
| `plan_name`      | String | Sim         | Nome do plano              | `"pro"`     |
| `billing_cycle`  | String | Sim         | Ciclo de cobrança          | `"annual"`  |
| `coupon_code`    | String | Não         | Código de cupom aplicado   | `"PROMO20"` |
| `original_price` | Number | Sim         | Preço original             | `79.90`     |
| `final_price`    | Number | Sim         | Preço final (com desconto) | `63.92`     |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Checkout Started", {
  plan_name: "pro",
  billing_cycle: "annual",
  coupon_code: "PROMO20",
  original_price: 79.9,
  final_price: 63.92,
});
```

#### Campanhas Relacionadas

- Recuperação de abandono de contratação

---

### Subscription Completed 💰

**ID:** EVT-019
**Categoria:** Monetização
**Criticidade:** Alta
**Business Case:** BC4

#### Descrição

Disparado quando a assinatura é concluída com sucesso. Este é o principal evento de monetização.

#### Trigger

Confirmação do pagamento da assinatura.

#### Propriedades

| Propriedade      | Tipo    | Obrigatório | Descrição                   | Exemplo                              |
| ---------------- | ------- | ----------- | --------------------------- | ------------------------------------ |
| `plan_name`      | String  | Sim         | Nome do plano               | `"pro"`                              |
| `billing_cycle`  | String  | Sim         | Ciclo de cobrança           | `"annual"`                           |
| `amount`         | Number  | Sim         | Valor pago                  | `63.92`                              |
| `payment_method` | String  | Sim         | Método de pagamento         | `"credit_card"`, `"boleto"`, `"pix"` |
| `coupon_used`    | Boolean | Sim         | Usou cupom                  | `true`                               |
| `coupon_code`    | String  | Não         | Código do cupom             | `"PROMO20"`                          |
| `state`          | String  | Sim         | Estado do lojista           | `"SP"`                               |
| `city`           | String  | Sim         | Cidade do lojista           | `"São Paulo"`                        |
| `is_upgrade`     | Boolean | Sim         | É upgrade de plano          | `true`                               |
| `previous_plan`  | String  | Não         | Plano anterior (se upgrade) | `"free"`                             |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Subscription Completed", {
  plan_name: "pro",
  billing_cycle: "annual",
  amount: 63.92,
  payment_method: "credit_card",
  coupon_used: true,
  coupon_code: "PROMO20",
  state: "SP",
  city: "São Paulo",
  is_upgrade: true,
  previous_plan: "free",
});

// Evento Charged (monetização)
clevertap.event.push("Charged", {
  Amount: 63.92,
  Currency: "BRL",
  "Payment Mode": "credit_card",
  "Charged ID": "sub_" + Date.now(),
  Items: [
    {
      Name: "Plano Pro",
      Category: "subscription",
      "Billing Cycle": "annual",
      "Coupon Code": "PROMO20",
    },
  ],
});

// Atualizar perfil completo
clevertap.profile.push({
  Site: {
    current_plan: "pro",
    plan_start_date: new Date(),
    billing_cycle: "annual",
    plan_price: 63.92,
    is_paying_customer: true,
    total_subscriptions: { $incr: 1 },
    subscription_value_total: { $incr: 63.92 },
    last_plan_change_date: new Date(),
    previous_plan: "free",
    coupon_used: true,
    last_coupon_code: "PROMO20",
    coupons_used_count: { $incr: 1 },
    state: "SP",
    city: "São Paulo",
  },
});

// Se for primeira assinatura paga
if (isFirstPaidSubscription) {
  clevertap.profile.push({
    Site: {
      first_paid_plan_date: new Date(),
    },
  });
}
```

#### Campanhas Relacionadas

- Confirmação de contratação
- Meta de assinatura (celebração)

---

### Pagali Registration Started

**ID:** EVT-021
**Categoria:** Pagamentos
**Criticidade:** Alta
**Business Case:** BC6

#### Descrição

Disparado quando o lojista inicia o cadastro no Pagali.

#### Trigger

Acesso à página de cadastro do Pagali.

#### Propriedades

| Propriedade    | Tipo   | Obrigatório | Descrição        | Exemplo                          |
| -------------- | ------ | ----------- | ---------------- | -------------------------------- |
| `entry_source` | String | Sim         | Origem do acesso | `"komea"`, `"panel"`, `"direct"` |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Pagali Registration Started", {
  entry_source: "komea",
});

clevertap.profile.push({
  Site: {
    pagali_registration_date: new Date(),
    pagali_entry_source: "komea",
    pagali_account_status: "pending",
  },
});
```

---

### Pagali Account Approved

**ID:** EVT-023
**Categoria:** Pagamentos
**Criticidade:** Alta
**Business Case:** BC6

#### Descrição

Disparado quando a conta do Pagali é aprovada.

#### Trigger

Notificação de aprovação da análise.

#### Propriedades

| Propriedade               | Tipo          | Obrigatório | Descrição         | Exemplo                            |
| ------------------------- | ------------- | ----------- | ----------------- | ---------------------------------- |
| `payment_methods_enabled` | Array[String] | Sim         | Meios liberados   | `["pix", "credit_card", "boleto"]` |
| `approval_date`           | Date          | Sim         | Data da aprovação | `"2026-01-26T14:00:00Z"`           |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Pagali Account Approved", {
  payment_methods_enabled: ["pix", "credit_card", "boleto"],
  approval_date: new Date().toISOString(),
});

clevertap.profile.push({
  Site: {
    pagali_account_status: "approved",
    pagali_verification_status: "verified",
    pagali_approval_date: new Date(),
    pagali_pix_enabled: true,
    pagali_credit_card_enabled: true,
    pagali_boleto_enabled: true,
    payment_methods_configured: { $add: ["pix", "credit_card", "boleto"] },
  },
});
```

---

### Pagali Account Rejected

**ID:** EVT-024
**Categoria:** Pagamentos
**Criticidade:** Alta
**Business Case:** BC6

#### Descrição

Disparado quando a conta do Pagali é rejeitada.

#### Trigger

Notificação de rejeição da análise.

#### Propriedades

| Propriedade        | Tipo   | Obrigatório | Descrição                    | Exemplo                                          |
| ------------------ | ------ | ----------- | ---------------------------- | ------------------------------------------------ |
| `rejection_reason` | String | Sim         | Motivo da rejeição (interno) | `"incomplete_documents"`, `"data_inconsistency"` |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Pagali Account Rejected", {
  rejection_reason: "incomplete_documents",
});

clevertap.profile.push({
  Site: {
    pagali_account_status: "rejected",
    pagali_rejection_reason: "incomplete_documents",
  },
});
```

#### Campanhas Relacionadas

- Orientação para configurar Mercado Pago

---

## 3. Eventos de Produtos (BC5)

### Product Creation Started

**ID:** EVT-025
**Categoria:** Catálogo
**Criticidade:** Alta
**Business Case:** BC5

#### Descrição

Disparado quando o lojista inicia a criação de um produto.

#### Trigger

Clique em "Criar produto" ou acesso ao formulário de cadastro.

#### Propriedades

| Propriedade       | Tipo   | Obrigatório | Descrição         | Exemplo                                     |
| ----------------- | ------ | ----------- | ----------------- | ------------------------------------------- |
| `creation_method` | String | Sim         | Método de criação | `"manual"`, `"ai_komea"`                    |
| `entry_source`    | String | Sim         | Origem do acesso  | `"products_menu"`, `"komea"`, `"dashboard"` |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Product Creation Started", {
  creation_method: "manual",
  entry_source: "products_menu",
});
```

---

### Product Created

**ID:** EVT-026
**Categoria:** Catálogo
**Criticidade:** Alta
**Business Case:** BC5

#### Descrição

Disparado quando um produto é criado com sucesso.

#### Trigger

Salvamento bem-sucedido do produto.

#### Propriedades

| Propriedade        | Tipo    | Obrigatório | Descrição            | Exemplo                  |
| ------------------ | ------- | ----------- | -------------------- | ------------------------ |
| `product_id`       | String  | Sim         | ID do produto        | `"prod_123456"`          |
| `creation_method`  | String  | Sim         | Método de criação    | `"manual"`, `"ai_komea"` |
| `category`         | String  | Não         | Categoria do produto | `"roupas"`               |
| `has_images`       | Boolean | Sim         | Tem imagens          | `true`                   |
| `has_variations`   | Boolean | Sim         | Tem variações        | `false`                  |
| `is_first_product` | Boolean | Sim         | É o primeiro produto | `true`                   |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Product Created", {
  product_id: "prod_123456",
  creation_method: "manual",
  category: "roupas",
  has_images: true,
  has_variations: false,
  is_first_product: true,
});

clevertap.profile.push({
  Site: {
    has_products: true,
    products_count: { $incr: 1 },
    last_product_date: new Date(),
    product_creation_method: "manual",
  },
});

// Se for primeiro produto
if (isFirstProduct) {
  clevertap.profile.push({
    Site: {
      first_product_date: new Date(),
    },
  });
}

// Se criado via IA
if (creationMethod === "ai_komea") {
  clevertap.profile.push({
    Site: {
      products_created_via_ai: { $incr: 1 },
    },
  });
}
```

#### Campanhas Relacionadas

- Régua educacional de qualidade baseada no primeiro cadastro

---

## 4. Eventos da Komea (BC7, BC8, BC9)

### Komea Accessed

**ID:** EVT-028
**Categoria:** Engajamento
**Criticidade:** Média
**Business Case:** BC7, BC8, BC9

#### Descrição

Disparado quando o lojista acessa a Komea.

#### Trigger

Entrada na interface da Komea.

#### Propriedades

| Propriedade      | Tipo   | Obrigatório | Descrição        | Exemplo                                   |
| ---------------- | ------ | ----------- | ---------------- | ----------------------------------------- |
| `entry_source`   | String | Sim         | Origem do acesso | `"menu"`, `"dashboard"`, `"notification"` |
| `session_number` | Number | Sim         | Número da sessão | `5`                                       |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Komea Accessed", {
  entry_source: "menu",
  session_number: 5,
});

clevertap.profile.push({
  Site: {
    komea_access_count: { $incr: 1 },
    komea_last_access_date: new Date(),
  },
});

// Se for primeiro acesso
if (sessionNumber === 1) {
  clevertap.profile.push({
    Site: {
      komea_first_access_date: new Date(),
    },
  });
}
```

---

### Komea Logo Generated

**ID:** EVT-029
**Categoria:** Customização
**Criticidade:** Média
**Business Case:** BC7

#### Descrição

Disparado quando a IA gera opções de logo.

#### Trigger

Conclusão da geração de logos pela IA.

#### Propriedades

| Propriedade         | Tipo   | Obrigatório | Descrição            | Exemplo |
| ------------------- | ------ | ----------- | -------------------- | ------- |
| `options_generated` | Number | Sim         | Quantidade de opções | `3`     |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Komea Logo Generated", {
  options_generated: 3,
});

clevertap.profile.push({
  Site: {
    komea_logo_ai_generated: true,
  },
});
```

---

### Komea Customization Completed

**ID:** EVT-030
**Categoria:** Customização
**Criticidade:** Alta
**Business Case:** BC7

#### Descrição

Disparado quando o lojista conclui a personalização da vitrine.

#### Trigger

Finalização do fluxo de personalização (logo + cor).

#### Propriedades

| Propriedade      | Tipo   | Obrigatório | Descrição              | Exemplo                        |
| ---------------- | ------ | ----------- | ---------------------- | ------------------------------ |
| `logo_source`    | String | Sim         | Origem do logo         | `"uploaded"`, `"ai_generated"` |
| `color_selected` | String | Sim         | Cor selecionada        | `"#FF5733"`                    |
| `time_spent`     | Number | Não         | Tempo total (segundos) | `300`                          |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Komea Customization Completed", {
  logo_source: "ai_generated",
  color_selected: "#FF5733",
  time_spent: 300,
});

clevertap.profile.push({
  Site: {
    komea_customization_completed: true,
    komea_logo_ai_selected: true,
    komea_color_selected: "#FF5733",
  },
});
```

---

### Komea Site Published

**ID:** EVT-031
**Categoria:** Onboarding
**Criticidade:** Alta
**Business Case:** BC8

#### Descrição

Disparado quando o site é publicado (sai do modo manutenção).

#### Trigger

Confirmação de publicação do site.

#### Propriedades

| Propriedade          | Tipo    | Obrigatório | Descrição                            | Exemplo |
| -------------------- | ------- | ----------- | ------------------------------------ | ------- |
| `has_payment_method` | Boolean | Sim         | Tem meio de pagamento                | `true`  |
| `has_product`        | Boolean | Sim         | Tem produto cadastrado               | `true`  |
| `time_to_publish`    | Number  | Não         | Tempo desde criação da conta (horas) | `48`    |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Komea Site Published", {
  has_payment_method: true,
  has_product: true,
  time_to_publish: 48,
});

clevertap.profile.push({
  Site: {
    site_published: true,
    site_publish_date: new Date(),
    site_publish_source: "komea",
    had_payment_before_publish: true,
  },
});
```

---

### Komea Opportunity Executed

**ID:** EVT-032
**Categoria:** Engajamento
**Criticidade:** Média
**Business Case:** BC9

#### Descrição

Disparado quando o lojista executa uma ação sugerida pela Komea.

#### Trigger

Conclusão de uma ação recomendada.

#### Propriedades

| Propriedade        | Tipo   | Obrigatório | Descrição            | Exemplo                                                      |
| ------------------ | ------ | ----------- | -------------------- | ------------------------------------------------------------ |
| `opportunity_type` | String | Sim         | Tipo de oportunidade | `"optimize_images"`, `"add_product"`, `"configure_shipping"` |
| `opportunity_id`   | String | Sim         | ID da oportunidade   | `"opp_123"`                                                  |
| `result`           | String | Sim         | Resultado            | `"completed"`, `"skipped"`                                   |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Komea Opportunity Executed", {
  opportunity_type: "optimize_images",
  opportunity_id: "opp_123",
  result: "completed",
});

clevertap.profile.push({
  Site: {
    komea_opportunities_executed: { $incr: 1 },
  },
});
```

---

### Komea Action Executed

**ID:** EVT-033
**Categoria:** Engajamento
**Criticidade:** Média
**Business Case:** BC9

#### Descrição

Disparado quando o lojista executa uma ação sugerida pelo assistente.

#### Trigger

Conclusão de uma ação após conversa com assistente.

#### Propriedades

| Propriedade      | Tipo   | Obrigatório | Descrição        | Exemplo                                                |
| ---------------- | ------ | ----------- | ---------------- | ------------------------------------------------------ |
| `action_type`    | String | Sim         | Tipo de ação     | `"create_promotion"`, `"update_price"`, `"send_email"` |
| `assistant_type` | String | Sim         | Assistente usado | `"data_assistant"`, `"promotion_creator"`              |
| `result`         | String | Sim         | Resultado        | `"success"`, `"failed"`                                |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("Komea Action Executed", {
  action_type: "create_promotion",
  assistant_type: "promotion_creator",
  result: "success",
});

clevertap.profile.push({
  Site: {
    komea_actions_executed: { $incr: 1 },
  },
});
```

---

## 5. Eventos de Canais de Venda (BC11)

### ML Connected

**ID:** EVT-034
**Categoria:** Canais
**Criticidade:** Alta
**Business Case:** BC11

#### Descrição

Disparado quando o lojista conecta sua conta do Mercado Livre.

#### Trigger

Login/cadastro bem-sucedido no Mercado Livre via Hub de Canais.

#### Propriedades

| Propriedade       | Tipo   | Obrigatório | Descrição       | Exemplo                  |
| ----------------- | ------ | ----------- | --------------- | ------------------------ |
| `account_type`    | String | Sim         | Tipo de conta   | `"existing"`, `"new"`    |
| `connection_date` | Date   | Sim         | Data da conexão | `"2026-01-26T10:00:00Z"` |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("ML Connected", {
  account_type: "existing",
  connection_date: new Date().toISOString(),
});

clevertap.profile.push({
  Site: {
    hub_channels_active: true,
    ml_connected: true,
    ml_connection_date: new Date(),
    ml_account_type: "existing",
  },
});
```

---

### ML Ad Published 💰

**ID:** EVT-035
**Categoria:** Canais / Monetização
**Criticidade:** Alta
**Business Case:** BC11

#### Descrição

Disparado quando um anúncio é publicado com sucesso no Mercado Livre.

#### Trigger

Confirmação do Mercado Livre de publicação.

#### Propriedades

| Propriedade   | Tipo    | Obrigatório | Descrição           | Exemplo                  |
| ------------- | ------- | ----------- | ------------------- | ------------------------ |
| `product_id`  | String  | Sim         | ID do produto       | `"prod_123"`             |
| `ad_type`     | String  | Sim         | Tipo de anúncio     | `"classic"`, `"premium"` |
| `ad_id`       | String  | Sim         | ID do anúncio no ML | `"MLB123456789"`         |
| `is_first_ad` | Boolean | Sim         | Primeiro anúncio    | `true`                   |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("ML Ad Published", {
  product_id: "prod_123",
  ad_type: "premium",
  ad_id: "MLB123456789",
  is_first_ad: true,
});

// Evento Charged (monetização) - taxa do anúncio premium
clevertap.event.push("Charged", {
  Amount: adFee, // Taxa cobrada pelo ML (premium tem custo maior)
  Currency: "BRL",
  "Payment Mode": "ml_fee",
  "Charged ID": "ad_" + "MLB123456789",
  Items: [
    {
      Name: "Anúncio Mercado Livre Premium",
      Category: "ml_ad",
      "Ad Type": "premium",
      "Product ID": "prod_123",
    },
  ],
});

clevertap.profile.push({
  Site: {
    ml_total_ads_count: { $incr: 1 },
    ml_last_ad_date: new Date(),
    ml_first_ad_sent: true,
  },
});

// Incrementar contador específico do tipo
if (adType === "premium") {
  clevertap.profile.push({
    Site: {
      ml_premium_ads_count: { $incr: 1 },
    },
  });
} else {
  clevertap.profile.push({
    Site: {
      ml_classic_ads_count: { $incr: 1 },
    },
  });
}

// Se for primeiro anúncio
if (isFirstAd) {
  clevertap.profile.push({
    Site: {
      ml_first_ad_date: new Date(),
    },
  });
}
```

#### Campanhas Relacionadas

- Incentivo a anúncios premium
- Expansão de catálogo

---

### ML Sale Completed 💰

**ID:** EVT-036
**Categoria:** Canais / Monetização
**Criticidade:** Alta
**Business Case:** BC11

#### Descrição

Disparado quando uma venda é realizada no Mercado Livre.

#### Trigger

Pedido confirmado vindo do Mercado Livre.

#### Propriedades

| Propriedade        | Tipo    | Obrigatório | Descrição                  | Exemplo          |
| ------------------ | ------- | ----------- | -------------------------- | ---------------- |
| `order_id`         | String  | Sim         | ID do pedido               | `"ml_order_789"` |
| `ad_type`          | String  | Sim         | Tipo do anúncio que vendeu | `"premium"`      |
| `amount`           | Number  | Sim         | Valor da venda             | `199.90`         |
| `product_id`       | String  | Sim         | ID do produto              | `"prod_123"`     |
| `is_first_ml_sale` | Boolean | Sim         | Primeira venda no ML       | `true`           |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("ML Sale Completed", {
  order_id: "ml_order_789",
  ad_type: "premium",
  amount: 199.9,
  product_id: "prod_123",
  is_first_ml_sale: true,
});

// Evento Charged (monetização) - receita da venda no ML
clevertap.event.push("Charged", {
  Amount: 199.9,
  Currency: "BRL",
  "Payment Mode": "mercado_livre",
  "Charged ID": "ml_order_789",
  Items: [
    {
      Name: "Venda Mercado Livre",
      Category: "ml_sale",
      "Ad Type": "premium",
      "Product ID": "prod_123",
    },
  ],
});

clevertap.profile.push({
  Site: {
    ml_sales_count: { $incr: 1 },
    ml_total_revenue: { $incr: 199.9 },
    ml_last_sale_date: new Date(),
  },
});

// Se for primeira venda
if (isFirstMlSale) {
  clevertap.profile.push({
    Site: {
      ml_first_sale_date: new Date(),
    },
  });
}
```

---

## 6. Eventos Fiscais (BC10)

### NFe Emitted

**ID:** EVT-037
**Categoria:** Fiscal
**Criticidade:** Alta
**Business Case:** BC10

#### Descrição

Disparado quando uma nota fiscal é emitida com sucesso.

#### Trigger

Confirmação de emissão da NF pelo sistema.

#### Propriedades

| Propriedade    | Tipo    | Obrigatório | Descrição         | Exemplo              |
| -------------- | ------- | ----------- | ----------------- | -------------------- |
| `order_id`     | String  | Sim         | ID do pedido      | `"order_123"`        |
| `nfe_number`   | String  | Sim         | Número da NF      | `"NF-000123"`        |
| `amount`       | Number  | Sim         | Valor da NF       | `299.90`             |
| `is_first_nfe` | Boolean | Sim         | Primeira NF       | `true`               |
| `tax_regime`   | String  | Sim         | Regime tributário | `"simples_nacional"` |

#### Código de Implementação

**Web (JavaScript):**

```javascript
clevertap.event.push("NFe Emitted", {
  order_id: "order_123",
  nfe_number: "NF-000123",
  amount: 299.9,
  is_first_nfe: true,
  tax_regime: "simples_nacional",
});

clevertap.profile.push({
  Site: {
    nfes_emitted: { $incr: 1 },
    last_nfe_date: new Date(),
  },
});

// Se for primeira NF
if (isFirstNfe) {
  clevertap.profile.push({
    Site: {
      first_nfe_date: new Date(),
    },
  });
}
```

---

## Tipos de Propriedade Suportados

| Tipo            | Descrição                  | Exemplo                  |
| --------------- | -------------------------- | ------------------------ |
| `String`        | Texto até 512 chars        | `"produto_abc"`          |
| `Number`        | Integer ou Float           | `99.90`                  |
| `Boolean`       | true/false                 | `true`                   |
| `Date`          | ISO 8601 ou epoch          | `"2026-01-26T10:30:00Z"` |
| `Array[String]` | Lista de strings (max 100) | `["tag1", "tag2"]`       |

---

## Checklist de Validação por Evento

Para cada evento implementado, verificar:

- [ ] Evento aparece no dashboard em < 5 minutos
- [ ] Todas as propriedades obrigatórias estão presentes
- [ ] Tipos de dados estão corretos
- [ ] Não há duplicação de eventos
- [ ] Atributos de perfil são atualizados corretamente
- [ ] Nomes de eventos seguem padrão `Object Action`
- [ ] Nomes de propriedades estão em snake_case

---

## Changelog

| Data       | Versão | Alteração                            | Autor |
| ---------- | ------ | ------------------------------------ | ----- |
| 05/01/2026 | 1.0    | Versão inicial com 11 Business Cases | RMH   |
