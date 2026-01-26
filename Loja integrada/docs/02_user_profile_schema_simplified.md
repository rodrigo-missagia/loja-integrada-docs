# User Profile Schema - Loja Integrada

**Versão:** 2.0
**Data:** 26 de Janeiro de 2026
**Plataforma:** CleverTap

---

## Filosofia de Simplificação

Este documento adota o princípio **"Events over Properties"**:

- **Segmentação por eventos** é preferível para: datas (first/last time), contadores, histórico
- **Propriedades de perfil** são necessárias apenas para: status atual, configurações ativas, dados que não vêm de eventos

> **Benefício:** Redução de ~95 para ~35 propriedades customizadas. Menor complexidade de sincronização e manutenção.

---

## Reserved Attributes (CleverTap)

Atributos padrão obrigatórios.

| Atributo       | Tipo    | Descrição                      | Obrigatório |
| -------------- | ------- | ------------------------------ | ----------- |
| `Identity`     | String  | ID único do lojista (store_id) | **Sim**     |
| `Name`         | String  | Nome completo do lojista       | Sim         |
| `Email`        | String  | Email do lojista               | Sim         |
| `Phone`        | String  | Telefone com código país       | Recomendado |
| `MSG-email`    | Boolean | Opt-in para email              | Não         |
| `MSG-push`     | Boolean | Opt-in para push               | Não         |
| `MSG-sms`      | Boolean | Opt-in para SMS                | Não         |
| `MSG-whatsapp` | Boolean | Opt-in para WhatsApp           | Não         |

---

## Atributos de Conta (Cadastro Inicial)

Propriedades definidas uma única vez na criação da conta.

| Atributo       | Tipo   | Descrição        | Evento Origem   |
| -------------- | ------ | ---------------- | --------------- |
| `store_id`     | String | ID único da loja | `Store Created` |
| `store_name`   | String | Nome da loja     | `Store Created` |
| `account_type` | String | Tipo: pf ou pj   | `Store Created` |
| `state`        | String | Estado (UF)      | `Store Created` |
| `city`         | String | Cidade           | `Store Created` |

> **Nota:** Não há vínculo direto com Business Cases. São dados cadastrais básicos.

---

## Atributos de Plano e Assinatura

**Business Case:** BC4 - Contratação de Planos Pagos

| Atributo             | Tipo    | Descrição                                         | Evento Origem            | Operação |
| -------------------- | ------- | ------------------------------------------------- | ------------------------ | -------- |
| `current_plan`       | String  | Plano atual: free, starter, pro, plus, enterprise | `Subscription Completed` | set      |
| `billing_cycle`      | String  | Ciclo: monthly, annual                            | `Subscription Completed` | set      |
| `is_paying_customer` | Boolean | Cliente pagante                                   | `Subscription Completed` | set      |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade                 | Segmentação CleverTap                                               |
| --------------------------- | ------------------------------------------------------------------- |
| Primeira assinatura paga    | Evento `Subscription Completed` com filtro "Did for the first time" |
| Mudou de plano recentemente | Evento `Subscription Completed` nos últimos X dias                  |
| Abandonou checkout          | Evento `Subscription Abandoned` nos últimos X dias                  |
| Usou cupom                  | Evento `Subscription Completed` com `coupon_code` presente          |
| Total de assinaturas        | Count de eventos `Subscription Completed`                           |

---

## Atributos do Enviali

**Business Cases:** BC1 - Configuração de Envio via Enviali, BC2 - Compra de Etiquetas via Enviali

| Atributo                   | Tipo          | Descrição                    | Evento Origem             | Operação |
| -------------------------- | ------------- | ---------------------------- | ------------------------- | -------- |
| `enviali_active`           | Boolean       | Enviali ativado na loja      | `Enviali Activated`       | set      |
| `shipping_methods_active`  | Array[String] | Lista de métodos ativos      | `Shipping Method Enabled` | append   |
| `correios_direct_contract` | Boolean       | Tem contrato direto Correios | `Correios Activated`      | set      |
| `enviali_balance_amount`   | Number        | Valor do saldo atual         | `Enviali Balance Added`   | set      |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade             | Segmentação CleverTap                                                   |
| ----------------------- | ----------------------------------------------------------------------- |
| Data de ativação        | Evento `Enviali Activated` com filtro de data                           |
| Comprou etiqueta        | Evento `Label Purchased` "Did"                                          |
| Primeira etiqueta       | Evento `Label Purchased` "Did for the first time"                       |
| Quantidade de etiquetas | Count de eventos `Label Purchased`                                      |
| Teve cotação sem compra | Evento `Shipping Quote Requested` "Did" AND `Label Purchased` "Did not" |

---

## Atributos da Loggi

**Business Case:** BC3 - Ativação e Monetização Loggi

| Atributo       | Tipo    | Descrição     | Evento Origem     | Operação |
| -------------- | ------- | ------------- | ----------------- | -------- |
| `loggi_active` | Boolean | Loggi ativada | `Loggi Activated` | set      |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade             | Segmentação CleverTap                                                |
| ----------------------- | -------------------------------------------------------------------- |
| Data de ativação        | Evento `Loggi Activated` com filtro de data                          |
| Comprou etiqueta Loggi  | Evento `Loggi Label Purchased` "Did"                                 |
| Ativou mas nunca usou   | Evento `Loggi Activated` "Did" AND `Loggi Label Purchased` "Did not" |
| Quantidade de etiquetas | Count de eventos `Loggi Label Purchased`                             |

---

## Atributos do Pagali

**Business Case:** BC6 - Configurar Meio de Pagamento (Pagali)

| Atributo                     | Tipo          | Descrição                                        | Evento Origem                      | Operação |
| ---------------------------- | ------------- | ------------------------------------------------ | ---------------------------------- | -------- |
| `pagali_account_status`      | String        | Status: approved, rejected, pending, not_started | `Pagali Account Approved/Rejected` | set      |
| `payment_methods_configured` | Array[String] | Lista de meios configurados                      | `Pagali Payment Method Enabled`    | append   |
| `mercado_pago_configured`    | Boolean       | Mercado Pago configurado (fallback)              | `Mercado Pago Configured`          | set      |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade            | Segmentação CleverTap                                              |
| ---------------------- | ------------------------------------------------------------------ |
| Iniciou cadastro       | Evento `Pagali Registration Started` "Did"                         |
| Data de aprovação      | Evento `Pagali Account Approved` com filtro de data                |
| Abandonou em etapa X   | Evento `Pagali Registration Abandoned` com filtro `abandoned_step` |
| Rejeitado por motivo X | Evento `Pagali Account Rejected` com filtro `rejection_reason`     |

---

## Atributos de Produtos

**Business Case:** BC5 - Criar Primeiro Produto

| Atributo       | Tipo    | Descrição                | Evento Origem     | Operação |
| -------------- | ------- | ------------------------ | ----------------- | -------- |
| `has_products` | Boolean | Tem produtos cadastrados | `Product Created` | set      |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade            | Segmentação CleverTap                                     |
| ---------------------- | --------------------------------------------------------- |
| Primeiro produto       | Evento `Product Created` "Did for the first time"         |
| Criou via IA           | Evento `Product Created` com `creation_method = ai_komea` |
| Quantidade de produtos | Count de eventos `Product Created`                        |
| Abandonou criação      | Evento `Product Creation Abandoned` "Did"                 |

---

## Atributos da Komea

**Business Cases:** BC7 - Personalizar Vitrine (em dev), BC8 - Publicar Site (em dev), BC9 - Copiloto (em dev)

| Atributo         | Tipo    | Descrição                           | Evento Origem          | Operação |
| ---------------- | ------- | ----------------------------------- | ---------------------- | -------- |
| `site_published` | Boolean | Site publicado (fora de manutenção) | `Komea Site Published` | set      |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade              | Segmentação CleverTap                        |
| ------------------------ | -------------------------------------------- |
| Acessou Komea            | Evento `Komea Accessed` "Did"                |
| Quantidade de acessos    | Count de eventos `Komea Accessed`            |
| Usou assistente          | Evento `Komea Assistant Accessed` "Did"      |
| Executou ação            | Evento `Komea Action Executed` "Did"         |
| Personalizou logo/cor    | Evento `Komea Customization Completed` "Did" |
| Abandonou personalização | Evento `Komea Customization Abandoned` "Did" |

> **Nota:** BC7, BC8 e BC9 estão em desenvolvimento. A propriedade `pagali_left_komea_flow` foi removida pois pode ser segmentada via evento `Komea Left For Panel`.

---

## Atributos do Mercado Livre

**Business Case:** BC11 - Ativação Canal Mercado Livre

| Atributo                  | Tipo    | Descrição               | Evento Origem             | Operação |
| ------------------------- | ------- | ----------------------- | ------------------------- | -------- |
| `mercado_livre_connected` | Boolean | Mercado Livre conectado | `Mercado Livre Connected` | set      |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade                | Segmentação CleverTap                               |
| -------------------------- | --------------------------------------------------- |
| Data de conexão            | Evento `Mercado Livre Connected` com filtro de data |
| Enviou anúncio             | Evento `ML Ad Sent` "Did"                           |
| Primeiro anúncio publicado | Evento `ML Ad Published` "Did for the first time"   |
| Quantidade de anúncios     | Count de eventos `ML Ad Published`                  |
| Realizou venda             | Evento `ML Sale Completed` "Did"                    |

---

## Atributos Fiscais

**Business Case:** BC10 - Emissão de Nota Fiscal _(especificação pendente)_

| Atributo         | Tipo    | Descrição                                      | Evento Origem             | Operação |
| ---------------- | ------- | ---------------------------------------------- | ------------------------- | -------- |
| `nfe_configured` | Boolean | Configuração fiscal concluída                  | `NFe Settings Configured` | set      |
| `tax_regime`     | String  | Regime: simples_nacional, lucro_presumido, mei | `NFe Settings Configured` | set      |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade         | Segmentação CleverTap                         |
| ------------------- | --------------------------------------------- |
| Primeira NF emitida | Evento `NFe Emitted` "Did for the first time" |
| Quantidade de NFs   | Count de eventos `NFe Emitted`                |
| Falha em emissão    | Evento `NFe Emission Failed` "Did"            |

---

## Resumo: Propriedades Essenciais

Total: **20 propriedades customizadas** (vs. 95+ anteriormente)

### Por Business Case

| BC          | Propriedades                                                                                      | Justificativa                                |
| ----------- | ------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| —           | `store_id`, `store_name`, `account_type`, `state`, `city`                                         | Dados cadastrais (sem BC específico)         |
| BC4         | `current_plan`, `billing_cycle`, `is_paying_customer`                                             | Status atual do plano precisa ser atualizado |
| BC1/BC2     | `enviali_active`, `shipping_methods_active`, `correios_direct_contract`, `enviali_balance_amount` | Estado de configuração e saldo               |
| BC3         | `loggi_active`                                                                                    | Estado de configuração                       |
| BC6         | `pagali_account_status`, `payment_methods_configured`, `mercado_pago_configured`                  | Status e meios ativos                        |
| BC5         | `has_products`                                                                                    | Flag de ativação básica                      |
| BC7/BC8/BC9 | `site_published`                                                                                  | Estado do site                               |
| BC11        | `mercado_livre_connected`                                                                         | Estado de conexão                            |
| BC10        | `invoice_configured`, `tax_regime`                                                                | Estado de configuração fiscal                |

### Propriedades Removidas (segmentáveis via eventos)

As seguintes propriedades do schema anterior foram removidas por serem deriváveis via segmentação de eventos:

- Todas as propriedades `*_date` (first/last) → "Did for the first time" ou filtro de data
- Todas as propriedades `*_count` → Count de eventos
- Propriedades de abandono → Evento de abandono correspondente
- `pagali_left_komea_flow` → Evento `Komea Left For Panel`
- Etapas de jornada → Propriedades do evento correspondente

---

## Operações CleverTap

### set

Sobrescreve o valor anterior.

```javascript
clevertap.profile.push({
  Site: {
    current_plan: "pro",
    is_paying_customer: true,
  },
});
```

### append

Adiciona item à lista (máx 100 items).

```javascript
clevertap.profile.push({
  Site: {
    shipping_methods_active: { $add: ["loggi"] },
    payment_methods_configured: { $add: ["pix"] },
  },
});
```

### remove

Remove item da lista.

```javascript
clevertap.profile.push({
  Site: {
    shipping_methods_active: { $remove: ["jadlog"] },
  },
});
```

---

## Exemplos de Segmentação por Eventos

### Exemplo 1: Lojistas que ativaram Enviali mas nunca compraram etiqueta

```
Segment Criteria:
- Event: "Enviali Activated" → Did
- AND Event: "Label Purchased" → Did not
```

### Exemplo 2: Lojistas com primeira venda no ML nos últimos 7 dias

```
Segment Criteria:
- Event: "ML Sale Completed" → Did for the first time → in the last 7 days
```

### Exemplo 3: Lojistas que abandonaram cadastro Pagali na etapa de documentos

```
Segment Criteria:
- Event: "Pagali Registration Abandoned" → Did
  - Where: abandoned_step = "documents"
```

### Exemplo 4: Lojistas pagantes que nunca criaram produto

```
Segment Criteria:
- Property: is_paying_customer = true
- AND Event: "Product Created" → Did not
```

---

## Limites Técnicos CleverTap

| Item                             | Limite         |
| -------------------------------- | -------------- |
| Custom attribute keys por perfil | 256            |
| Nome do atributo                 | 120 caracteres |
| Valor string                     | 512 caracteres |
| Array                            | 100 items      |

---

## Changelog

| Data       | Versão | Alteração                            | Autor |
| ---------- | ------ | ------------------------------------ | ----- | --- |
| 05/01/2026 | 1.0    | Versão inicial com 11 Business Cases | RMH   |     |
