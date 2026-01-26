# User Profile Schema - Loja Integrada

**Versão:** 1.1
**Data:** 26 de Janeiro de 2026
**Plataforma:** CleverTap
**Status:** ✅ Versão Oficial

> **Nota:** Este é o documento oficial do User Profile Schema. O arquivo `02_user_profile_schema_simplified.md` é mantido como referência da filosofia "Events over Properties" para casos onde simplificação é desejada, mas este documento é a especificação completa a ser seguida na implementação.

---

## Visão Geral

Este documento define o schema de atributos de perfil de usuário para a Loja Integrada. Os atributos são organizados em categorias que correspondem aos Business Cases e funcionalidades da plataforma.

**Total de atributos:** 95+ atributos customizados

> **Nota:** A coluna `Campo Segmentation Store` indica o campo correspondente no CRM otimizado conforme [segmentation-store.md](../../knowledge/segmentation-store.md). Campos marcados com "—" não possuem equivalente direto identificado.

---

## User Identity Strategy

### Contexto do Problema

Na Loja Integrada, a relação **usuário ↔ loja** é única e independente:

1. **Um usuário pode pertencer a múltiplas lojas** com estados diferentes em cada uma
2. **Uma loja pode ter múltiplos usuários** (proprietário, funcionários, parceiros)
3. **Propriedades da loja** devem ser sincronizadas para todos os usuários daquela loja
4. **Propriedades do usuário** são individuais e específicas de cada usuário

**Exemplo prático:**

- Usuário "João" é proprietário da Loja A (plano Pro) e funcionário da Loja B (plano Free)
- Quando a Loja A publica o site, João (contexto Loja A) deve ter `site_published = true`
- Mas João (contexto Loja B) permanece com `site_published = false`

### Solução: Identity Composto

O `Identity` no CleverTap será uma **concatenação do ID da loja com o ID do usuário**:

```
Identity = {store_id}_{user_id}
```

**Exemplos:**

- `store_12345_user_789` → Usuário 789 no contexto da Loja 12345
- `store_12345_user_456` → Usuário 456 no contexto da Loja 12345
- `store_67890_user_789` → Mesmo usuário 789, mas no contexto da Loja 67890

### Benefícios

| Benefício                  | Descrição                                                                    |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Contexto único**         | Cada par loja-usuário tem seu próprio perfil no CleverTap                    |
| **Sincronização em massa** | Propriedades da loja podem ser atualizadas para todos os usuários via lookup |
| **Histórico individual**   | Eventos do usuário são rastreados por contexto de loja                       |
| **Campanhas segmentadas**  | Permite segmentar por características da loja OU do usuário                  |

### Matriz de Propriedades: Loja vs Usuário

As propriedades do perfil são classificadas em duas categorias com comportamentos de atualização distintos:

#### Propriedades de LOJA (Store-Level)

Atualizadas em **TODOS os usuários** da loja quando a propriedade muda.

| Categoria              | Propriedades                                                                                                                                                                                                          |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Conta/Cadastro**     | `store_id`, `store_name`, `account_created_date`, `account_type`, `document_number`, `state`, `city`, `zip_code`, `region`                                                                                            |
| **Plano/Assinatura**   | `current_plan`, `plan_start_date`, `billing_cycle`, `plan_price`, `is_paying_customer`, `first_paid_plan_date`, `previous_plan`, `last_plan_change_date`                                                              |
| **Enviali/Logística**  | `enviali_active`, `enviali_activation_date`, `shipping_methods_active`, `shipping_methods_count`, `correios_active`, `correios_direct_contract`, `correios_services_enabled`, `loggi_active`, `loggi_activation_date` |
| **Etiquetas**          | `enviali_label_purchased`, `enviali_labels_count`, `enviali_first_label_date`, `enviali_last_label_date`, `enviali_balance_amount`, `enviali_total_spent`, `loggi_labels_count`                                       |
| **Pagali**             | `pagali_account_status`, `pagali_verification_status`, `pagali_approval_date`, `pagali_pix_enabled`, `pagali_credit_card_enabled`, `pagali_boleto_enabled`, `payment_methods_configured`                              |
| **Produtos**           | `has_products`, `products_count`, `first_product_date`, `last_product_date`                                                                                                                                           |
| **Site/Publicação**    | `site_published`, `site_publish_date`                                                                                                                                                                                 |
| **Mercado Livre**      | `mercado_livre_connected`, `mercado_livre_connection_date`, `ml_setup_completed`, `ml_products_sent`, `ml_total_ads_count`, `ml_sales_count`                                                                          |
| **Fiscal**             | `tax_regime`, `NFe_configured`, `NFes_emitted`                                                                                                                                                                        |
| **Métricas Agregadas** | `gmv_30d`, `visitas_30d`, `qtde_pedido_30d`, etc. (via batch sync)                                                                                                                                                    |

#### Propriedades de USUÁRIO (User-Level)

Atualizadas **APENAS para o usuário específico** que executou a ação.

| Categoria                       | Propriedades                                                                                                                                                 |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Identidade**                  | `Name`, `Email`, `Phone`, `user_id`                                                                                                                          |
| **Preferências de Comunicação** | `MSG-email`, `MSG-push`, `MSG-sms`, `MSG-whatsapp`                                                                                                           |
| **Ações Individuais**           | `product_creation_method` (se criou via AI ou manual), `komea_access_count`, `komea_last_access_date`, `komea_conversations_count`, `komea_actions_executed` |
| **Jornada/Abandono**            | `checkout_abandoned`, `checkout_abandoned_step`, `checkout_abandoned_date`, `pagali_abandoned_step`, `product_abandoned_step`                                |
| **Origem do Acesso**            | `pagali_entry_source`, `enviali_label_flow_source`, `site_publish_source`                                                                                    |
| **Sessão/Atividade**            | `ultimo_login_painel`, `data_ultimo_acesso`                                                                                                                  |

### Fluxo de Sincronização

Consulte o diagrama detalhado em [06_user_identity_sync_flow.md](./06_user_identity_sync_flow.md).

**Resumo do fluxo:**

1. **Evento na aplicação** → Identifica se é propriedade de LOJA ou USUÁRIO
2. **Se LOJA**: DAG no Airflow busca todos os usuários ativos da loja e atualiza propriedade em todos
3. **Se USUÁRIO**: Atualização direta apenas no perfil do usuário que executou a ação

---

## Reserved Attributes (CleverTap)

Atributos padrão reconhecidos pela CleverTap. Usar exatamente estes nomes.

| Atributo       | Tipo    | Descrição                           | Obrigatório | Exemplo                   | Campo Segmentation Store |
| -------------- | ------- | ----------------------------------- | ----------- | ------------------------- | ------------------------ |
| `Identity`     | String  | ID composto: `{store_id}_{user_id}` | **Sim**     | `"store_123456_user_789"` | `id_loja` + `user_id`    |
| `Name`         | String  | Nome completo do lojista            | Sim         | `"João Silva"`            | `firstname`              |
| `Email`        | String  | Email do lojista                    | Sim         | `"joao@loja.com.br"`      | `email`                  |
| `Phone`        | String  | Telefone com código país            | Recomendado | `"+5511999999999"`        | `phone`                  |
| `MSG-email`    | Boolean | Opt-in para email                   | Não         | `true`                    | —                        |
| `MSG-push`     | Boolean | Opt-in para push                    | Não         | `true`                    | —                        |
| `MSG-sms`      | Boolean | Opt-in para SMS                     | Não         | `false`                   | —                        |
| `MSG-whatsapp` | Boolean | Opt-in para WhatsApp                | Não         | `true`                    | —                        |

---

## Atributos de Conta e Cadastro

### Informações Básicas

| Atributo               | Tipo   | Descrição                   | Evento Origem   | Operação | Campo Segmentation Store |
| ---------------------- | ------ | --------------------------- | --------------- | -------- | ------------------------ |
| `store_id`             | String | ID único da loja            | `Store Created` | set      | `id_loja`                |
| `store_name`           | String | Nome da loja                | `Store Created` | set      | —                        |
| `account_created_date` | Date   | Data de criação da conta    | `Store Created` | set      | `data_criacao_dl`        |
| `account_type`         | String | Tipo: pf (CPF) ou pj (CNPJ) | `Store Created` | set      | `tipo_de_pessoa`         |
| `document_number`      | String | CPF ou CNPJ (mascarado)     | `Store Created` | set      | —                        |

### Localização

| Atributo   | Tipo   | Descrição                   | Evento Origem                              | Operação | Campo Segmentation Store |
| ---------- | ------ | --------------------------- | ------------------------------------------ | -------- | ------------------------ |
| `state`    | String | Estado (UF)                 | `Store Created` / `Subscription Completed` | set      | `state`                  |
| `city`     | String | Cidade                      | `Store Created` / `Subscription Completed` | set      | `City`                   |
| `zip_code` | String | CEP                         | `Store Created`                            | set      | —                        |
| `region`   | String | Região (Sul, Sudeste, etc.) | Calculado                                  | set      | —                        |

---

## Atributos de Plano e Assinatura (BC4)

### Status do Plano

| Atributo             | Tipo    | Descrição                                         | Evento Origem            | Operação | Campo Segmentation Store |
| -------------------- | ------- | ------------------------------------------------- | ------------------------ | -------- | ------------------------ |
| `current_plan`       | String  | Plano atual: free, starter, pro, plus, enterprise | `Subscription Completed` | set      | `plano_atual`            |
| `plan_start_date`    | Date    | Data de início do plano atual                     | `Subscription Completed` | set      | `inicio_de_ciclo`        |
| `billing_cycle`      | String  | Ciclo: monthly, annual                            | `Subscription Completed` | set      | `tipo_plano`             |
| `plan_price`         | Number  | Valor do plano                                    | `Subscription Completed` | set      | —                        |
| `is_paying_customer` | Boolean | Cliente pagante (true/false)                      | `Subscription Completed` | set      | —                        |

### Histórico de Assinatura

| Atributo                   | Tipo   | Descrição                        | Evento Origem            | Operação   | Campo Segmentation Store     |
| -------------------------- | ------ | -------------------------------- | ------------------------ | ---------- | ---------------------------- |
| `first_paid_plan_date`     | Date   | Data da primeira assinatura paga | `Subscription Completed` | set (once) | `data_primeira_assinatura` ¹ |
| `total_subscriptions`      | Number | Total de assinaturas realizadas  | `Subscription Completed` | increment  | —                            |
| `subscription_value_total` | Number | Valor total pago em assinaturas  | `Subscription Completed` | increment  | —                            |
| `last_plan_change_date`    | Date   | Data da última mudança de plano  | `Subscription Completed` | set        | —                            |
| `previous_plan`            | String | Plano anterior                   | `Subscription Completed` | set        | `plano_anterior`             |

### Cupons

| Atributo             | Tipo    | Descrição                    | Evento Origem            | Operação  | Campo Segmentation Store |
| -------------------- | ------- | ---------------------------- | ------------------------ | --------- | ------------------------ |
| `coupon_used`        | Boolean | Já usou cupom                | `Subscription Completed` | set       | —                        |
| `last_coupon_code`   | String  | Último código de cupom usado | `Subscription Completed` | set       | —                        |
| `coupons_used_count` | Number  | Total de cupons usados       | `Subscription Completed` | increment | —                        |

### Checkout e Abandono

> **Nota sobre Inaction:** Conforme decisão de simplificação do tracking plan, eventos "Abandoned" foram removidos. A identificação de abandonos deve ser feita via **segmentação Inaction no CleverTap**. Para checkout abandonado: Evento `Checkout Started` "Did" AND `Subscription Completed` "Did not" nos últimos X dias. Para cadastro abandonado: Evento de início sem evento de conclusão no período definido.
>
> As propriedades `checkout_abandoned`, `checkout_abandoned_step`, `checkout_abandoned_date` e `checkout_abandoned_plan` foram **removidas** e não devem ser implementadas.

> ¹ Campo substituível por evento `Subscription Completed` (first time) conforme otimização Tipo 7.

---

## Atributos do Enviali (BC1, BC2)

### Ativação

| Atributo                      | Tipo    | Descrição                  | Evento Origem                 | Operação | Campo Segmentation Store         |
| ----------------------------- | ------- | -------------------------- | ----------------------------- | -------- | -------------------------------- |
| `enviali_active`              | Boolean | Enviali ativado na loja    | `Enviali Activated`           | set      | — ²                              |
| `enviali_activation_date`     | Date    | Data de ativação           | `Enviali Activated`           | set      | `data_configuracao_do_enviali` ¹ |
| `enviali_initial_data_filled` | Boolean | Dados iniciais preenchidos | `Enviali Initial Data Filled` | set      | —                                |

> ¹ Campo substituível por evento `Enviali Activated` (first time) conforme otimização Tipo 7.
> ² O Segmentation Store otimizado usa a presença de `data_configuracao_do_enviali` como indicador de ativação.

### Transportadoras Ativas

| Atributo                      | Tipo          | Descrição                        | Evento Origem             | Operação  | Campo Segmentation Store |
| ----------------------------- | ------------- | -------------------------------- | ------------------------- | --------- | ------------------------ |
| `shipping_methods_active`     | Array[String] | Lista de métodos ativos          | `Shipping Method Enabled` | append    | `meios_envio_ativos`     |
| `shipping_methods_count`      | Number        | Quantidade de métodos ativos     | `Shipping Method Enabled` | increment | —                        |
| `shipping_source`             | String        | Origem do método: enviali, other | `Shipping Method Enabled` | set       | —                        |
| `other_shipping_intermediary` | String        | Nome do outro intermediador      | Manual                    | set       | —                        |

### Correios

| Atributo                    | Tipo          | Descrição                            | Evento Origem        | Operação | Campo Segmentation Store |
| --------------------------- | ------------- | ------------------------------------ | -------------------- | -------- | ------------------------ |
| `correios_active`           | Boolean       | Correios ativado                     | `Correios Activated` | set      | —                        |
| `correios_direct_contract`  | Boolean       | Tem contrato direto                  | `Correios Activated` | set      | `flag_pac_contrato`      |
| `correios_conflict`         | Boolean       | Conflito: Enviali + contrato próprio | Calculado            | set      | —                        |
| `correios_services_enabled` | Array[String] | Serviços ativos (PAC, SEDEX, etc.)   | `Correios Activated` | set      | —                        |

### Etiquetas

| Atributo                      | Tipo    | Descrição                         | Evento Origem        | Operação   | Campo Segmentation Store                       |
| ----------------------------- | ------- | --------------------------------- | -------------------- | ---------- | ---------------------------------------------- |
| `enviali_label_purchased`     | Boolean | Já comprou etiqueta               | `Label Purchased`    | set        | —                                              |
| `enviali_labels_count`        | Number  | Total de etiquetas emitidas       | `Label Purchased`    | increment  | —                                              |
| `enviali_first_label_date`    | Date    | Data da primeira etiqueta         | `Label Purchased`    | set (once) | `data_da_primeira_etiqueta_enviali` ¹          |
| `enviali_last_label_date`     | Date    | Data da última etiqueta           | `Label Purchased`    | set        | `data_da_ultima_emissao_de_etiqueta_enviali` ¹ |
| `enviali_label_flow_accessed` | Boolean | Já acessou fluxo de emissão       | `Label Flow Started` | set        | —                                              |
| `enviali_label_flow_source`   | String  | Origem: order_list, label_manager | `Label Flow Started` | set        | —                                              |

> ¹ Campos substituíveis por eventos `Label Issued` (first/last time) conforme otimização Tipo 7.

### Intenção (Pré-monetização)

| Atributo                      | Tipo    | Descrição                                     | Evento Origem                     | Operação | Campo Segmentation Store |
| ----------------------------- | ------- | --------------------------------------------- | --------------------------------- | -------- | ------------------------ |
| `enviali_quote_received`      | Boolean | Teve cotação via Enviali                      | `Shipping Quote Requested`        | set      | —                        |
| `checkout_carrier_selected`   | String  | Última transportadora selecionada no checkout | `Shipping Quote Carrier Selected` | set      | —                        |
| `orders_quoted_without_label` | Number  | Pedidos com cotação sem etiqueta              | Calculado                         | set      | —                        |

### Saldo e Financeiro

| Atributo                 | Tipo    | Descrição                           | Evento Origem           | Operação  | Campo Segmentation Store |
| ------------------------ | ------- | ----------------------------------- | ----------------------- | --------- | ------------------------ |
| `enviali_has_balance`    | Boolean | Possui saldo                        | `Enviali Balance Added` | set       | —                        |
| `enviali_balance_amount` | Number  | Valor do saldo atual                | `Enviali Balance Added` | set       | `saldo_enviali`          |
| `enviali_balance_added`  | Boolean | Já adicionou saldo                  | `Enviali Balance Added` | set       | —                        |
| `enviali_total_spent`    | Number  | Total gasto em etiquetas            | `Label Purchased`       | increment | —                        |
| `enviali_payment_method` | String  | Último meio de pagamento: card, pix | `Label Purchased`       | set       | —                        |

### Milestones de Envio

| Atributo                     | Tipo   | Descrição                              | Evento Origem   | Operação  | Campo Segmentation Store    |
| ---------------------------- | ------ | -------------------------------------- | --------------- | --------- | --------------------------- |
| `enviali_shipment_milestone` | String | Milestone: first, second, fifth, tenth | `Order Shipped` | set       | `vendas_enviali_milestones` |
| `enviali_shipments_count`    | Number | Total de envios                        | `Order Shipped` | increment | —                           |

---

## Atributos da Loggi (BC3)

| Atributo                       | Tipo    | Descrição                              | Evento Origem                     | Operação   | Campo Segmentation Store |
| ------------------------------ | ------- | -------------------------------------- | --------------------------------- | ---------- | ------------------------ |
| `loggi_active`                 | Boolean | Loggi ativada                          | `Loggi Activated`                 | set        | —                        |
| `loggi_activation_date`        | Date    | Data de ativação                       | `Loggi Activated`                 | set        | —                        |
| `loggi_configured_not_used`    | Boolean | Ativa mas sem etiqueta                 | Calculado                         | set        | —                        |
| `loggi_label_purchased`        | Boolean | Já comprou etiqueta Loggi              | `Loggi Label Purchased`           | set        | —                        |
| `loggi_labels_count`           | Number  | Total de etiquetas Loggi               | `Loggi Label Purchased`           | increment  | —                        |
| `loggi_first_label_date`       | Date    | Data da primeira etiqueta Loggi        | `Loggi Label Purchased`           | set (once) | —                        |
| `loggi_checkout_quotes`        | Number  | Pedidos com Loggi no checkout          | `Shipping Quote Carrier Selected` | increment  | —                        |
| `loggi_orders_quoted_no_label` | Number  | Cotações Loggi sem etiqueta            | Calculado                         | set        | —                        |
| `loggi_label_milestone`        | String  | Milestone: first, second, fifth, tenth | `Loggi Label Purchased`           | set        | —                        |

> **Nota:** Os campos específicos de Loggi não possuem equivalentes diretos no Segmentation Store. A Loggi é tratada como item do array `meios_envio_ativos`.

---

## Atributos do Pagali (BC6)

### Status da Conta

| Atributo                     | Tipo   | Descrição                                        | Evento Origem                      | Operação | Campo Segmentation Store  |
| ---------------------------- | ------ | ------------------------------------------------ | ---------------------------------- | -------- | ------------------------- |
| `pagali_account_status`      | String | Status: approved, rejected, pending, not_started | `Pagali Account Approved/Rejected` | set      | `status_pagali`           |
| `pagali_verification_status` | String | Status: verified, unverified                     | `Pagali Account Approved`          | set      | —                         |
| `pagali_registration_date`   | Date   | Data do cadastro                                 | `Pagali Registration Started`      | set      | `data_cadastro_pagamento` |
| `pagali_approval_date`       | Date   | Data da aprovação                                | `Pagali Account Approved`          | set      | —                         |

### Meios de Pagamento

| Atributo                      | Tipo          | Descrição                   | Evento Origem                   | Operação | Campo Segmentation Store |
| ----------------------------- | ------------- | --------------------------- | ------------------------------- | -------- | ------------------------ |
| `pagali_pix_enabled`          | Boolean       | Pix ativado                 | `Pagali Payment Method Enabled` | set      | — ³                      |
| `pagali_credit_card_enabled`  | Boolean       | Cartão ativado              | `Pagali Payment Method Enabled` | set      | — ³                      |
| `pagali_boleto_enabled`       | Boolean       | Boleto ativado              | `Pagali Payment Method Enabled` | set      | — ³                      |
| `pagali_payment_link_enabled` | Boolean       | Link de pagamento ativado   | `Pagali Payment Method Enabled` | set      | —                        |
| `payment_methods_configured`  | Array[String] | Lista de meios configurados | `Pagali Payment Method Enabled` | append   | `meios_pagamento_ativos` |

> ³ No Segmentation Store otimizado, os meios de pagamento individuais (pagali_pix, pagali_cartao, pagali_boleto) são consolidados no array `meios_pagamento_ativos`.

### Abandono e Jornada

| Atributo                   | Tipo    | Descrição                         | Evento Origem                        | Operação | Campo Segmentation Store |
| -------------------------- | ------- | --------------------------------- | ------------------------------------ | -------- | ------------------------ |
| `pagali_registration_step` | String  | Última etapa completada           | `Pagali Registration Step Completed` | set      | —                        |
| `pagali_abandoned_step`    | String  | Etapa onde abandonou              | `Pagali Registration Abandoned`      | set      | —                        |
| `pagali_rejection_reason`  | String  | Motivo da não aprovação (interno) | `Pagali Account Rejected`            | set      | —                        |
| `pagali_entry_source`      | String  | Origem: komea, panel, direct      | `Pagali Registration Started`        | set      | —                        |
| `pagali_left_komea_flow`   | Boolean | Saiu do fluxo Komea para painel   | `Komea Left For Panel`               | set      | —                        |

### Fallback Mercado Pago

| Atributo                   | Tipo    | Descrição                           | Evento Origem             | Operação | Campo Segmentation Store |
| -------------------------- | ------- | ----------------------------------- | ------------------------- | -------- | ------------------------ |
| `mercado_pago_configured`  | Boolean | Mercado Pago configurado            | `Mercado Pago Configured` | set      | — ⁴                      |
| `mercado_pago_as_fallback` | Boolean | MP configurado após rejeição Pagali | `Mercado Pago Configured` | set      | —                        |

> ⁴ A presença de meios Mercado Pago é indicada via `meios_pagamento_ativos` (ex: `mercado_pago_cartao`, `mercado_pago_boleto`).

---

## Atributos de Produtos (BC5)

| Atributo                     | Tipo    | Descrição                      | Evento Origem                | Operação   | Campo Segmentation Store  |
| ---------------------------- | ------- | ------------------------------ | ---------------------------- | ---------- | ------------------------- |
| `has_products`               | Boolean | Tem produtos cadastrados       | `Product Created`            | set        | —                         |
| `products_count`             | Number  | Total de produtos              | `Product Created`            | increment  | `produtos_ativos`         |
| `first_product_date`         | Date    | Data do primeiro produto       | `Product Created`            | set (once) | `data_cadastro_produto` ¹ |
| `last_product_date`          | Date    | Data do último produto criado  | `Product Created`            | set        | —                         |
| `product_creation_method`    | String  | Método: manual, ai_komea       | `Product Created`            | set        | —                         |
| `products_created_via_ai`    | Number  | Produtos criados via IA        | `Product Created`            | increment  | —                         |
| `product_creation_abandoned` | Boolean | Abandonou criação recentemente | `Product Creation Abandoned` | set        | —                         |
| `product_abandoned_step`     | String  | Etapa do abandono              | `Product Creation Abandoned` | set        | —                         |

> ¹ Campo substituível por evento `Product Created` (first time) conforme otimização Tipo 7.

---

## Atributos da Komea (BC7, BC8, BC9)

### Uso Geral

| Atributo                    | Tipo   | Descrição                 | Evento Origem           | Operação   | Campo Segmentation Store |
| --------------------------- | ------ | ------------------------- | ----------------------- | ---------- | ------------------------ |
| `komea_access_count`        | Number | Total de acessos à Komea  | `Komea Accessed`        | increment  | —                        |
| `komea_last_access_date`    | Date   | Data do último acesso     | `Komea Accessed`        | set        | —                        |
| `komea_conversations_count` | Number | Total de conversas        | `Komea Question Asked`  | increment  | —                        |
| `komea_actions_executed`    | Number | Total de ações executadas | `Komea Action Executed` | increment  | —                        |
| `komea_first_access_date`   | Date   | Data do primeiro acesso   | `Komea Accessed`        | set (once) | —                        |

> **Nota:** Os atributos de Komea (BC7-BC9) não possuem equivalentes no Segmentation Store atual, pois representam funcionalidades em desenvolvimento.

### Personalização (BC7)

| Atributo                             | Tipo    | Descrição                | Evento Origem                   | Operação | Campo Segmentation Store |
| ------------------------------------ | ------- | ------------------------ | ------------------------------- | -------- | ------------------------ |
| `komea_logo_ai_generated`            | Boolean | Logo gerado por IA       | `Komea Logo Generated`          | set      | —                        |
| `komea_logo_ai_selected`             | Boolean | Logo de IA selecionado   | `Komea Logo Selected`           | set      | —                        |
| `komea_color_selected`               | String  | Cor selecionada          | `Komea Color Selected`          | set      | —                        |
| `komea_color_changed_after_publish`  | Boolean | Mudou cor após publicar  | `Komea Color Selected`          | set      | —                        |
| `komea_customization_completed`      | Boolean | Personalização concluída | `Komea Customization Completed` | set      | —                        |
| `komea_customization_abandoned`      | Boolean | Abandonou personalização | `Komea Customization Abandoned` | set      | —                        |
| `komea_customization_abandoned_step` | String  | Etapa do abandono        | `Komea Customization Abandoned` | set      | —                        |

### Publicação (BC8)

| Atributo                     | Tipo    | Descrição                           | Evento Origem                      | Operação | Campo Segmentation Store |
| ---------------------------- | ------- | ----------------------------------- | ---------------------------------- | -------- | ------------------------ |
| `site_published`             | Boolean | Site publicado (fora de manutenção) | `Komea Site Published`             | set      | — ⁵                      |
| `site_publish_date`          | Date    | Data da publicação                  | `Komea Site Published`             | set      | —                        |
| `site_publish_source`        | String  | Origem: komea, panel                | `Komea Site Published`             | set      | —                        |
| `site_publication_abandoned` | Boolean | Abandonou publicação                | `Komea Site Publication Abandoned` | set      | —                        |
| `had_payment_before_publish` | Boolean | Tinha pagamento configurado         | `Komea Site Published`             | set      | —                        |

> ⁵ O campo `loja_em_manutencao` no Segmentation Store indica o estado inverso (em manutenção = não publicado).

### Copiloto (BC9)

| Atributo                       | Tipo          | Descrição                   | Evento Origem                | Operação  | Campo Segmentation Store |
| ------------------------------ | ------------- | --------------------------- | ---------------------------- | --------- | ------------------------ |
| `komea_opportunities_viewed`   | Boolean       | Visualizou oportunidades    | `Komea Opportunities Viewed` | set       | —                        |
| `komea_opportunities_executed` | Number        | Oportunidades executadas    | `Komea Opportunity Executed` | increment | —                        |
| `komea_assistant_used`         | Boolean       | Usou assistente             | `Komea Assistant Accessed`   | set       | —                        |
| `komea_assistants_used`        | Array[String] | Lista de assistentes usados | `Komea Assistant Accessed`   | append    | —                        |

---

## Atributos do Hub de Canais / Mercado Livre (BC11)

### Conexão

| Atributo                        | Tipo    | Descrição               | Evento Origem             | Operação | Campo Segmentation Store        |
| ------------------------------- | ------- | ----------------------- | ------------------------- | -------- | ------------------------------- |
| `hub_channels_active`           | Boolean | Hub de Canais ativado   | `Hub Channels Accessed`   | set      | —                               |
| `mercado_livre_connected`       | Boolean | Mercado Livre conectado | `Mercado Livre Connected` | set      | —                               |
| `mercado_livre_connection_date` | Date    | Data de conexão         | `Mercado Livre Connected` | set      | `data_evento_conectar_magalu` ¹ |
| `mercado_livre_account_type`    | String  | Tipo de conta ML        | `Mercado Livre Connected` | set      | —                               |

> ¹ Campo substituível por evento `ML Connection Started` (first time) conforme otimização Tipo 7. Nota: O Segmentation Store usa "magalu" mas refere-se ao Mercado Livre.

### Configuração

| Atributo                | Tipo    | Descrição                      | Evento Origem                | Operação | Campo Segmentation Store |
| ----------------------- | ------- | ------------------------------ | ---------------------------- | -------- | ------------------------ |
| `ml_setup_completed`    | Boolean | Configuração inicial concluída | `ML Initial Setup Completed` | set      | —                        |
| `ml_minimum_stock`      | Number  | Estoque mínimo configurado     | `ML Initial Setup Completed` | set      | —                        |
| `ml_classic_percentage` | Number  | % para anúncios clássicos      | `ML Initial Setup Completed` | set      | —                        |
| `ml_premium_percentage` | Number  | % para anúncios premium        | `ML Initial Setup Completed` | set      | —                        |

### Anúncios

| Atributo               | Tipo   | Descrição                 | Evento Origem     | Operação   | Campo Segmentation Store |
| ---------------------- | ------ | ------------------------- | ----------------- | ---------- | ------------------------ |
| `ml_products_sent`     | Number | Produtos enviados ao ML   | `ML Ad Sent`      | increment  | —                        |
| `ml_classic_ads_count` | Number | Anúncios clássicos ativos | `ML Ad Published` | increment  | —                        |
| `ml_premium_ads_count` | Number | Anúncios premium ativos   | `ML Ad Published` | increment  | —                        |
| `ml_total_ads_count`   | Number | Total de anúncios         | `ML Ad Published` | increment  | —                        |
| `ml_first_ad_date`     | Date   | Data do primeiro anúncio  | `ML Ad Published` | set (once) | —                        |
| `ml_last_ad_date`      | Date   | Data do último anúncio    | `ML Ad Published` | set        | —                        |

### Vendas

| Atributo             | Tipo   | Descrição              | Evento Origem       | Operação   | Campo Segmentation Store |
| -------------------- | ------ | ---------------------- | ------------------- | ---------- | ------------------------ |
| `ml_sales_count`     | Number | Vendas no ML           | `ML Sale Completed` | increment  | —                        |
| `ml_total_revenue`   | Number | Receita total no ML    | `ML Sale Completed` | increment  | —                        |
| `ml_first_sale_date` | Date   | Data da primeira venda | `ML Sale Completed` | set (once) | `data_vendeu_magalu` ¹   |
| `ml_last_sale_date`  | Date   | Data da última venda   | `ML Sale Completed` | set        | —                        |

> ¹ Campo substituível por evento `ML Sale Completed` (first time) conforme otimização Tipo 7.

### Jornada

| Atributo           | Tipo    | Descrição                   | Evento Origem              | Operação | Campo Segmentation Store |
| ------------------ | ------- | --------------------------- | -------------------------- | -------- | ------------------------ |
| `ml_setup_step`    | String  | Etapa atual da configuração | `ML Initial Setup Started` | set      | —                        |
| `ml_first_ad_sent` | Boolean | Primeiro anúncio enviado    | `ML Ad Sent`               | set      | —                        |

> **Nota:** Os campos do Segmentation Store relacionados ao Magalu/ML incluem também: `data_evento_tenhoconta_magalu`, `data_evento_criarconta_magalu`, `data_configuracao_magalu`, `magalu_etapas_concluidas`.

---

## Atributos Fiscais (BC10)

| Atributo               | Tipo    | Descrição                                                 | Evento Origem             | Operação   | Campo Segmentation Store |
| ---------------------- | ------- | --------------------------------------------------------- | ------------------------- | ---------- | ------------------------ |
| `tax_regime`           | String  | Regime tributário: simples_nacional, lucro_presumido, mei | `NFe Settings Configured` | set        | —                        |
| `NFe_configured`       | Boolean | Configuração fiscal concluída                             | `NFe Settings Configured` | set        | —                        |
| `certificate_uploaded` | Boolean | Certificado digital enviado                               | `NFe Settings Configured` | set        | —                        |
| `NFes_emitted`         | Number  | Total de NFs emitidas                                     | `NFe Emitted`             | increment  | —                        |
| `first_NFe_date`       | Date    | Data da primeira NF                                       | `NFe Emitted`             | set (once) | —                        |
| `last_NFe_date`        | Date    | Data da última NF                                         | `NFe Emitted`             | set        | —                        |
| `NFe_emission_failed`  | Boolean | Teve falha em emissão recente                             | `NFe Emission Failed`     | set        | —                        |

> **Nota:** Os atributos fiscais (BC10) não possuem equivalentes no Segmentation Store atual. A especificação do BC10 está pendente.

---

## Operações CleverTap

### set

Sobrescreve o valor anterior. Usar para dados que mudam completamente.

```javascript
clevertap.profile.push({
  Site: {
    current_plan: "pro",
    plan_start_date: new Date(),
  },
});
```

### set (once)

Define apenas se não existir. Usar para atributos "first time".

```javascript
// Implementar lógica no backend
if (!user.first_product_date) {
  clevertap.profile.push({
    Site: {
      first_product_date: new Date(),
    },
  });
}
```

### increment

Adiciona ao valor atual. Usar para contadores.

```javascript
clevertap.profile.push({
  Site: {
    products_count: { $incr: 1 },
    enviali_total_spent: { $incr: 45.9 },
  },
});
```

### append

Adiciona item à lista (máx 100 items).

```javascript
clevertap.profile.push({
  Site: {
    shipping_methods_active: { $add: ["loggi"] },
    komea_assistants_used: { $add: ["data_assistant"] },
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

## Limites Técnicos CleverTap

| Item                             | Limite         |
| -------------------------------- | -------------- |
| Custom attribute keys por perfil | 256            |
| Nome do atributo                 | 120 caracteres |
| Valor string                     | 512 caracteres |
| Array                            | 100 items      |
| Caracteres especiais no nome     | Não permitido  |

---

## Checklist de Validação

- [ ] `Identity` está definido para todos os lojistas
- [ ] Atributos reserved usam nomes exatos da CleverTap
- [ ] Tipos de dado estão corretos (String, Number, Boolean, Date, Array)
- [ ] Regras de atualização fazem sentido (set vs increment vs append)
- [ ] Não excede 256 custom attributes
- [ ] Evento de origem está definido para cada atributo
- [ ] Atributos de "first time" usam set (once) no backend

---

## Campos do Segmentation Store Não Mapeados

Os seguintes campos do [segmentation-store.md](../../knowledge/segmentation-store.md) (versão otimizada) não possuem equivalentes diretos no User Profile Schema atual. Estes campos podem ser relevantes para futuras implementações ou campanhas de segmentação.

### Identificação e Configuração da Loja

| Campo Segmentation Store | Tipo   | Descrição                           |
| ------------------------ | ------ | ----------------------------------- |
| `website`                | Texto  | URL do site (domínio ou subdomínio) |
| `subdomain`              | Texto  | Subdomínio .lojaintegrada.com.br    |
| `status_da_loja`         | Texto  | Status atual da loja                |
| `loja_em_manutencao`     | Flag   | Se a loja está em manutenção        |
| `descricao_tier`         | Texto  | Descrição do tier                   |
| `cluster`                | Texto  | Clusterização da loja               |
| `segmento_da_loja`       | Texto  | Segmento de atuação                 |
| `modelo_de_loja`         | Texto  | Como a loja será usada              |
| `ranking`                | Número | Ranking da Loja Integrada           |

### Plano e Assinatura

| Campo Segmentation Store           | Tipo   | Descrição                        |
| ---------------------------------- | ------ | -------------------------------- |
| `id_plano_atual`                   | Número | ID do plano atual                |
| `fim_de_ciclo`                     | Data   | Fim do ciclo atual               |
| `movimento_plano`                  | Texto  | upgrade/downgrade/churn/novo     |
| `forma_pagamento_assinatura_atual` | Texto  | Forma de pagamento da assinatura |

### Atividade e Engajamento

| Campo Segmentation Store | Tipo | Descrição                       |
| ------------------------ | ---- | ------------------------------- |
| `data_ultimo_acesso`     | Data | Último acesso ao painel         |
| `data_primeira_venda`    | Data | Data da primeira venda aprovada |
| `finalizou_wizard`       | Flag | Se completou o wizard           |
| `loja_reativada`         | Flag | Se a loja foi reativada         |
| `dominio_proprio`        | Flag | Se configurou domínio próprio   |
| `dominio_inativo`        | Flag | Se o domínio está inativo       |

### GMV e Métricas Financeiras

| Campo Segmentation Store                         | Tipo   | Descrição                   |
| ------------------------------------------------ | ------ | --------------------------- |
| `gmv_mes_atual`                                  | Número | GMV do mês atual            |
| `gmv_30d`                                        | Número | GMV últimos 30 dias         |
| `gmv_60_dias`                                    | Número | GMV últimos 60 dias         |
| `gmv_90_dias`                                    | Número | GMV últimos 90 dias         |
| `gmv_cartao_30d`                                 | Número | GMV em cartão (30 dias)     |
| `taxa_de_aprovacao_em_cartao_ultimo_mes_fechado` | Número | Taxa de aprovação em cartão |

### Métricas de Conversão e Visitas

| Campo Segmentation Store                                   | Tipo   | Descrição                     |
| ---------------------------------------------------------- | ------ | ----------------------------- |
| `taxa_de_conversao_ultimo_mes_fechado`                     | Número | Taxa de conversão             |
| `taxa_de_conversao_do_segmento_da_loja_ultimo_mes_fechado` | Número | Taxa de conversão do segmento |
| `visitas_30d`                                              | Número | Visitas últimos 30 dias       |
| `visitas_60d`                                              | Número | Visitas últimos 60 dias       |
| `visitas_90d`                                              | Número | Visitas últimos 90 dias       |
| `qtde_pedido_30d`                                          | Número | Pedidos últimos 30 dias       |
| `qtde_pedido_60d`                                          | Número | Pedidos últimos 60 dias       |
| `qtde_pedido_90d`                                          | Número | Pedidos últimos 90 dias       |

### Funcionalidades da Loja

| Campo Segmentation Store           | Tipo  | Descrição                                 |
| ---------------------------------- | ----- | ----------------------------------------- |
| `checkout_sem_senha`               | Flag  | Checkout sem senha ativo                  |
| `abandono_carrinho_intervalos`     | Array | Intervalos configurados (1h, 6h, 24h)     |
| `newsletter_ativa`                 | Data  | Data de ativação                          |
| `aviseme_auto_ativo`               | Data  | Data de ativação                          |
| `frete_gratis_ativo`               | Data  | Data de ativação                          |
| `data_que_ativou_brinde`           | Data  | Data de ativação brinde                   |
| `desconto_no_pix_ativo`            | Flag  | Desconto no Pix ativo                     |
| `data_que_ativou_compre_junto`     | Data  | Data ativação (presença = ativo)          |
| `data_da_primeira_promocao`        | Data  | Data primeira promoção (presença = ativo) |
| `data_que_ativou_o_primeiro_cupom` | Data  | Data ativação cupom (presença = ativo)    |

### Personalização

| Campo Segmentation Store | Tipo  | Descrição                  |
| ------------------------ | ----- | -------------------------- |
| `tipo_tema`              | Texto | gratis/agencia/pago        |
| `data_instalacao_tema`   | Data  | Data de instalação do tema |
| `logo_da_loja`           | Flag  | Se tem logo                |
| `banner`                 | Flag  | Se tem banner              |

### Integrações e Apps

| Campo Segmentation Store | Tipo  | Descrição                               |
| ------------------------ | ----- | --------------------------------------- |
| `integracoes_ativas`     | Array | Lista de integrações ativas             |
| `hubs_integradores`      | Array | Lista de hubs (magis5, olist_tiny, etc) |

### Enviali (campos adicionais)

| Campo Segmentation Store        | Tipo   | Descrição              |
| ------------------------------- | ------ | ---------------------- |
| `etiquetas_disponiveis_enviali` | Número | Etiquetas disponíveis  |
| `economia_enviali`              | Número | Economia no último mês |
| `usa_jadlog_enviali`            | Flag   | Usa Jadlog no Enviali  |

### Google Shopping

| Campo Segmentation Store       | Tipo | Descrição                          |
| ------------------------------ | ---- | ---------------------------------- |
| `data_criacao_google_shopping` | Data | Data de criação (presença = ativo) |
| `google_shopping_tem_campanha` | Data | Data da primeira campanha          |

### Avaliação e Abandono

| Campo Segmentation Store  | Tipo | Descrição                           |
| ------------------------- | ---- | ----------------------------------- |
| `data_ativacao_avaliacao` | Data | Data de ativação (presença = ativo) |
| `data_abandono_produto`   | Data | Data de ativação (presença = ativo) |

### WhatsApp

| Campo Segmentation Store               | Tipo   | Descrição                    |
| -------------------------------------- | ------ | ---------------------------- |
| `data_que_cadastrou_botao_do_whatsapp` | Data   | Data do cadastro             |
| `recuperacao_whatsapp`                 | Flag   | Usa recuperação por WhatsApp |
| `hs_whatsapp_phone_number`             | Número | Número do WhatsApp           |

### Dados do Contato (adicionais)

| Campo Segmentation Store         | Tipo   | Descrição               |
| -------------------------------- | ------ | ----------------------- |
| `telefone_de_cobranca`           | Número | Telefone de cobrança    |
| `contato_responsavel_pela_conta` | Flag   | Se é o responsável      |
| `ultimo_login_painel`            | Data   | Último login do usuário |

### Origem e Aquisição

| Campo Segmentation Store     | Tipo  | Descrição             |
| ---------------------------- | ----- | --------------------- |
| `origem_do_lead___aquisicao` | Texto | Origem do lead        |
| `campanha_do_lead_aquisicao` | Texto | Campanha de aquisição |
| `midia_do_lead`              | Texto | Mídia do lead         |

### Outros

| Campo Segmentation Store | Tipo | Descrição                           |
| ------------------------ | ---- | ----------------------------------- |
| `politica_privacidade`   | Flag | Política de privacidade configurada |

---

## Decisões para o Time de Dados

Esta seção registra oportunidades de otimização identificadas durante o mapeamento entre o User Profile Schema (CleverTap) e o Segmentation Store. Estas decisões devem ser avaliadas pelo time de dados para garantir consistência entre sistemas.

### Decisão 1: Unificação de Nomenclatura Magalu/Mercado Livre

**Contexto:** O Segmentation Store usa nomenclatura "magalu" (`data_evento_conectar_magalu`, `data_vendeu_magalu`, etc.) enquanto o User Profile Schema usa "mercado_livre" (`mercado_livre_connected`, `ml_first_sale_date`, etc.).

**Recomendação:** Definir nomenclatura única para evitar confusão. Sugestão: usar `ml_` como prefixo padrão em ambos os sistemas.

**Impacto:** ~6 campos do Segmentation Store

---

### Decisão 2: Campos de GMV e Métricas de Janela Móvel

**Contexto:** O Segmentation Store possui métricas agregadas de janela móvel (`gmv_30d`, `gmv_60_dias`, `gmv_90_dias`, `visitas_30d`, `qtde_pedido_30d`, etc.) que não existem no User Profile Schema atual.

**Recomendação:** Avaliar se estas métricas devem ser:

1. Sincronizadas como propriedades de usuário no CleverTap (atualização periódica via batch)
2. Calculadas via eventos no CleverTap (usando segmentação por período)
3. Mantidas apenas no Segmentation Store para campanhas via integração direta

**Impacto:** ~15 campos de métricas

---

### Decisão 3: Consolidação de Flags Booleanas em Arrays

**Contexto:** O User Profile Schema mantém flags booleanas individuais para meios de pagamento Pagali (`pagali_pix_enabled`, `pagali_credit_card_enabled`, `pagali_boleto_enabled`) enquanto o Segmentation Store otimizado consolidou em array `meios_pagamento_ativos`.

**Recomendação:** Avaliar migração do User Profile Schema para o padrão de arrays, reduzindo de 4 campos para 1. Benefício adicional: mesma segmentação funciona para Pagali e outros meios.

**Impacto:** 3-4 campos no User Profile Schema

---

### Decisão 4: Campos de Status vs. Datas de Ativação

**Contexto:** O User Profile Schema usa flags booleanas (`enviali_active`, `loggi_active`) enquanto o Segmentation Store otimizado usa a presença de datas como indicador (`data_configuracao_do_enviali` presente = ativo).

**Recomendação:** Manter ambos os padrões ou escolher um:

- **Manter flags:** Mais explícito, melhor para queries diretas
- **Usar datas:** Menos campos, informação adicional (quando ativou)

**Trade-off:** Flags permitem desativação (ativo = false), enquanto datas só indicam "foi ativado alguma vez".

---

### Decisão 5: Campos de Correios Específicos

**Contexto:** O User Profile Schema possui campos específicos de Correios (`correios_active`, `correios_direct_contract`, `correios_services_enabled`) que não têm equivalentes diretos no Segmentation Store otimizado.

**Recomendação:** Verificar se estes campos são necessários para campanhas no CleverTap ou se podem ser derivados do array `meios_envio_ativos`.

**Impacto:** 3-4 campos

---

### Decisão 6: Integração de Dados de Visitas e Conversão

**Contexto:** Métricas como `visitas_30d`, `taxa_de_conversao_ultimo_mes_fechado` existem no Segmentation Store mas não no User Profile Schema.

**Recomendação:** Estas métricas são fundamentais para segmentação de campanhas de reativação e growth. Avaliar:

1. Sincronização batch diária/semanal do Segmentation Store → CleverTap
2. Criação de eventos de sessão para cálculo nativo no CleverTap

**Impacto:** Alta prioridade para campanhas de engajamento

---

### Decisão 7: Campos de Origem/Aquisição de Lead

**Contexto:** O Segmentation Store possui campos de atribuição (`origem_do_lead___aquisicao`, `campanha_do_lead_aquisicao`, `midia_do_lead`) que não existem no User Profile Schema.

**Recomendação:** Estes dados são importantes para análise de ROI de campanhas e devem ser incluídos no User Profile Schema se campanhas de lifecycle/retenção precisarem segmentar por origem.

**Impacto:** 3 campos

---

### Decisão 8: Campos de Funcionalidades da Loja (Cupom, Promoção, etc.)

**Contexto:** O Segmentation Store possui campos de funcionalidades ativas (`data_que_ativou_o_primeiro_cupom`, `data_que_ativou_compre_junto`, `data_da_primeira_promocao`, `frete_gratis_ativo`, etc.) que podem ser úteis para campanhas de educação/adoção de features.

**Recomendação:** Avaliar quais funcionalidades são alvo de campanhas de adoção e incluir os campos correspondentes no User Profile Schema.

**Impacto:** ~8-10 campos potenciais

---

## Changelog

| Data       | Versão | Alteração                            | Autor |
| ---------- | ------ | ------------------------------------ | ----- |
| 05/01/2026 | 1.0    | Versão inicial com 11 Business Cases | RMH   |
