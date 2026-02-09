# User Profile Schema - Loja Integrada

**Versão:** 3.0
**Data:** 09 de Fevereiro de 2026
**Plataforma:** CleverTap

---

## Filosofia de Simplificação

Este documento adota o princípio **"Events over Properties"**:

- **Segmentação por eventos** é preferível para: datas (first/last time), contadores, histórico
- **Propriedades de perfil** são necessárias apenas para: status atual, configurações ativas, dados que não vêm de eventos

> **Benefício:** Redução de ~95 para ~35 propriedades customizadas. Menor complexidade de sincronização e manutenção.

### Estratégia de Atualização

- **Backend (Data Lake → Airflow DAG):** Maioria das propriedades. Sync diário para todos os usuários da loja.
- **Frontend (CleverTap SDK):** Apenas para propriedades user-level (Komea) e Subscription (dual-write).
- **Dual-Write (Frontend + Backend):** Subscription props atualizadas imediatamente para o usuário ativo via SDK, e para todos os usuários via DAG diário.

> Motivo: Múltiplos usuários por loja. Frontend só atualiza o usuário que executou a ação.
> Para coerência entre usuários, propriedades store-level devem vir exclusivamente do backend.
> Detalhes no documento 06_user_identity_sync_flow.

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

| Atributo       | Tipo   | Descrição        | Evento Origem   | Fonte   |
| -------------- | ------ | ---------------- | --------------- | ------- |
| `store_id`     | String | ID único da loja | `Store Created` | Backend |
| `store_name`   | String | Nome da loja     | `Store Created` | Backend |
| `account_type` | String | Tipo: pf ou pj   | `Store Created` | Backend |
| `state`        | String | Estado (UF)      | `Store Created` | Backend |
| `city`         | String | Cidade           | `Store Created` | Backend |

> **Nota:** Não há vínculo direto com Business Cases. São dados cadastrais básicos.

---

## Atributos de Plano e Assinatura

**Business Case:** BC4 - Contratação de Planos Pagos

| Atributo             | Tipo    | Descrição                                                       | Evento Origem            | Operação | Fonte              |
| -------------------- | ------- | --------------------------------------------------------------- | ------------------------ | -------- | ------------------ |
| `current_plan`       | String  | Plano atual: gratuito, crescimento, aceleração, expansão, elite | `Subscription Completed` | set      | Frontend + Backend |
| `billing_cycle`      | String  | Ciclo: monthly, annual                                          | `Subscription Completed` | set      | Frontend + Backend |
| `is_paying_customer` | Boolean | Cliente pagante                                                 | `Subscription Completed` | set      | Frontend + Backend |
| `plan_start_date`    | Date    | Data de início do plano atual                                   | `Subscription Completed` | set      | Backend            |

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

| Atributo                   | Tipo          | Descrição                    | Evento Origem                                                           | Operação | Fonte   |
| -------------------------- | ------------- | ---------------------------- | ----------------------------------------------------------------------- | -------- | ------- |
| `enviali_active`           | Boolean       | Enviali ativado na loja      | `Shipping Platform Activated` (filtro: `shipping_platform = "enviali"`) | set      | Backend |
| `shipping_methods_active`  | Array[String] | Lista de métodos ativos      | `Shipping Method Enabled`                                               | append   | Backend |
| `correios_direct_contract` | Boolean       | Tem contrato direto Correios | `Shipping Method Enabled` (filtro: `carrier_name = "correios"`)         | set      | Backend |
| `enviali_balance_amount`   | Number        | Valor do saldo atual         | `Shipping Balance Added` (filtro: `shipping_platform = "enviali"`)      | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade             | Segmentação CleverTap                                                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------ |
| Data de ativação        | Evento `Shipping Platform Activated` com filtro `shipping_platform = "enviali"` e filtro de data |
| Comprou etiqueta        | Evento `Label Purchased` "Did"                                                                   |
| Primeira etiqueta       | Evento `Label Purchased` "Did for the first time"                                                |
| Quantidade de etiquetas | Count de eventos `Label Purchased`                                                               |

---

## Atributos da Loggi

**Business Case:** BC3 - Ativação e Monetização Loggi

| Atributo       | Tipo    | Descrição     | Evento Origem                                                    | Operação | Fonte   |
| -------------- | ------- | ------------- | ---------------------------------------------------------------- | -------- | ------- |
| `loggi_active` | Boolean | Loggi ativada | `Shipping Method Enabled` (filtro: `carrier_name = "loggi"`) | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade             | Segmentação CleverTap                                                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Data de ativação        | Evento `Shipping Method Enabled` com filtro `carrier_name = "loggi"` e filtro de data                                              |
| Comprou etiqueta Loggi  | Evento `Label Purchased` com filtro `carrier_name = "loggi"` "Did"                                                                 |
| Ativou mas nunca usou   | Evento `Shipping Method Enabled` com `carrier_name = "loggi"` "Did" AND `Label Purchased` com `carrier_name = "loggi"` "Did not" |
| Quantidade de etiquetas | Count de eventos `Label Purchased` com filtro `carrier_name = "loggi"`                                                             |

---

## Atributos do Pagali

**Business Case:** BC6 - Configurar Meio de Pagamento (Pagali)

| Atributo                     | Tipo          | Descrição                                        | Evento Origem                                                                                  | Operação | Fonte   |
| ---------------------------- | ------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------------- | -------- | ------- |
| `pagali_account_status`      | String        | Status: approved, rejected, pending, not_started | `Gateway Account Approved` / `Gateway Account Rejected` (filtro: `payment_gateway = "pagali"`) | set      | Backend |
| `payment_methods_configured` | Array[String] | Lista de meios configurados                      | `Payment Method Enabled` (filtro: `payment_gateway = "pagali"`)                                | append   | Backend |
| `mercado_pago_configured`    | Boolean       | Mercado Pago configurado (fallback)              | `Gateway Registration Completed` (filtro: `payment_gateway = "mercado_pago"`)                  | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade            | Segmentação CleverTap                                                                                                                                      |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Iniciou cadastro       | Evento `Gateway Registration Started` com filtro `payment_gateway = "pagali"` "Did"                                                                        |
| Data de aprovação      | Evento `Gateway Account Approved` com filtro `payment_gateway = "pagali"` e filtro de data                                                                 |
| Abandonou cadastro     | Inaction: `Gateway Registration Started` "Did" AND `Gateway Registration Completed` "Did not" (filtro: `payment_gateway = "pagali"`) nos últimos X dias |
| Rejeitado por motivo X | Evento `Gateway Account Rejected` com filtro `payment_gateway = "pagali"` e `rejection_reason`                                                             |

---

## Atributos de Produtos

**Business Case:** BC5 - Criar Primeiro Produto

| Atributo       | Tipo    | Descrição                | Evento Origem     | Operação | Fonte   |
| -------------- | ------- | ------------------------ | ----------------- | -------- | ------- |
| `has_products` | Boolean | Tem produtos cadastrados | `Product Created` | set      | Backend |

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

| Atributo                 | Tipo    | Descrição                           | Evento Origem          | Operação  | Fonte    |
| ------------------------ | ------- | ----------------------------------- | ---------------------- | --------- | -------- |
| `site_published`         | Boolean | Site publicado (fora de manutenção) | `Komea Site Published` | set       | Backend  |
| `komea_access_count`     | Number  | Contador de acessos à Komea         | `Komea Accessed`       | increment | Frontend |
| `komea_last_access_date` | Date    | Data do último acesso à Komea       | `Komea Accessed`       | set       | Frontend |

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

**Business Case:** BC11 - Ativação Canal Marketplace

| Atributo                  | Tipo    | Descrição               | Evento Origem                                                     | Operação | Fonte   |
| ------------------------- | ------- | ----------------------- | ----------------------------------------------------------------- | -------- | ------- |
| `mercado_livre_connected` | Boolean | Mercado Livre conectado | `Marketplace Connected` (filtro: `marketplace = "mercado_livre"`) | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade                | Segmentação CleverTap                                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------------------------ |
| Data de conexão            | Evento `Marketplace Connected` com filtro `marketplace = "mercado_livre"` e filtro de data             |
| Enviou anúncio             | Evento `Marketplace Products Selected` com filtro `marketplace = "mercado_livre"` "Did"                |
| Primeiro anúncio publicado | Evento `Marketplace Ad Published` com filtro `marketplace = "mercado_livre"` "Did for the first time" |
| Quantidade de anúncios     | Count de eventos `Marketplace Ad Published` com filtro `marketplace = "mercado_livre"`                 |
| Realizou venda             | Evento `Marketplace Sale Completed` com filtro `marketplace = "mercado_livre"` "Did"                   |

---

## Atributos Fiscais

**Business Case:** BC10 - Emissão de Nota Fiscal _(especificação pendente)_

| Atributo         | Tipo    | Descrição                                      | Evento Origem             | Operação | Fonte   |
| ---------------- | ------- | ---------------------------------------------- | ------------------------- | -------- | ------- |
| `nfe_configured` | Boolean | Configuração fiscal concluída                  | `NFe Settings Configured` | set      | Backend |
| `tax_regime`     | String  | Regime: simples_nacional, lucro_presumido, mei | `NFe Settings Configured` | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade         | Segmentação CleverTap                         |
| ------------------- | --------------------------------------------- |
| Primeira NF emitida | Evento `NFe Emitted` "Did for the first time" |
| Quantidade de NFs   | Count de eventos `NFe Emitted`                |
| Falha em emissão    | Evento `NFe Emission Failed` "Did"            |

---

## Métricas de Negócio

Propriedades agregadas calculadas diariamente pelo Data Lake.

| Atributo          | Tipo   | Descrição                | Operação | Fonte   |
| ----------------- | ------ | ------------------------ | -------- | ------- |
| `gmv_30d`         | Number | GMV dos últimos 30 dias  | set      | Backend |
| `visitas_30d`     | Number | Visitas últimos 30 dias  | set      | Backend |
| `qtde_pedido_30d` | Number | Pedidos últimos 30 dias  | set      | Backend |

> **Nota:** Estas métricas já são calculadas no Data Lake e sincronizadas via DAG diário. Não possuem evento de origem específico.

---

## Resumo: Propriedades Essenciais

Total: **27 propriedades customizadas** (vs. 95+ anteriormente)

### Por Business Case

| BC          | Propriedades                                                                                     | Fonte              | Justificativa                                |
| ----------- | ------------------------------------------------------------------------------------------------ | ------------------ | -------------------------------------------- |
| —           | `store_id`, `store_name`, `account_type`, `state`, `city`                                        | Backend            | Dados cadastrais (sem BC específico)         |
| BC4         | `current_plan`, `billing_cycle`, `is_paying_customer`, `plan_start_date`                         | Frontend + Backend | Status atual do plano precisa ser atualizado |
| BC1/BC2     | `enviali_active`, `shipping_methods_active`, `correios_direct_contract`, `enviali_balance_amount` | Backend            | Estado de configuração e saldo               |
| BC3         | `loggi_active`                                                                                   | Backend            | Estado de configuração                       |
| BC6         | `pagali_account_status`, `payment_methods_configured`, `mercado_pago_configured`                 | Backend            | Status e meios ativos                        |
| BC5         | `has_products`                                                                                   | Backend            | Flag de ativação básica                      |
| BC7/BC8/BC9 | `site_published`, `komea_access_count`, `komea_last_access_date`                                 | Backend / Frontend | Estado do site + métricas user-level         |
| BC11        | `mercado_livre_connected`                                                                        | Backend            | Estado de conexão                            |
| BC10        | `nfe_configured`, `tax_regime`                                                                   | Backend            | Estado de configuração fiscal                |
| —           | `gmv_30d`, `visitas_30d`, `qtde_pedido_30d`                                                      | Backend            | Métricas de negócio agregadas                |

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
    current_plan: "crescimento",
    is_paying_customer: true,
  },
});
```

### increment

Incrementa valor numérico.

```javascript
clevertap.profile.push({
  Site: {
    komea_access_count: { $incr: 1 },
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
- Event: "Shipping Platform Activated" → Did
  - Where: shipping_platform = "enviali"
- AND Event: "Label Purchased" → Did not
```

### Exemplo 2: Lojistas com primeira venda no marketplace nos últimos 7 dias

```
Segment Criteria:
- Event: "Marketplace Sale Completed" → Did for the first time → in the last 7 days
  - Where: marketplace = "mercado_livre"
```

### Exemplo 3: Lojistas que abandonaram cadastro de gateway de pagamento

```
Segment Criteria:
- Event: "Gateway Registration Started" → Did
  - Where: payment_gateway = "pagali"
- AND Event: "Gateway Registration Completed" → Did not
  - Where: payment_gateway = "pagali"
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

| Data       | Versão | Alteração                                                                                               | Autor |
| ---------- | ------ | ------------------------------------------------------------------------------------------------------- | ----- |
| 09/02/2026 | 3.0    | Revisão: nomes de eventos genericizados, coluna Fonte, props Komea/métricas, estratégia de sync backend | RMH   |
| 05/01/2026 | 1.0    | Versão inicial com 11 Business Cases                                                                    | RMH   |
