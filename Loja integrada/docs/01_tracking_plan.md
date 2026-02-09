# Tracking Plan - Loja Integrada

**Versão:** 1.2
**Data:** 09 de Fevereiro de 2026
**Plataforma de Analytics:** CleverTap
**Status:** Em Revisão

---

## Sumário Executivo

Este documento define o plano de rastreamento (tracking plan) para a Loja Integrada, uma plataforma SaaS de e-commerce brasileira. O plano cobre 11 Business Cases (casos de uso de negócio) organizados em 6 categorias principais de eventos:

1. **Envio e Logística** (Plataformas, Etiquetas, Transportadoras)
2. **Pagamentos e Assinatura** (Planos, Gateway de Pagamento)
3. **Configuração de Loja** (Produtos, Vitrine, Publicação)
4. **Komea - Copiloto IA** (Assistentes, Oportunidades)
5. **Canais de Venda** (Marketplaces)
6. **Fiscal** (Nota Fiscal)

**Objetivo Principal:** Instrumentar todos os pontos críticos da jornada do lojista para habilitar campanhas de engajamento, retenção e monetização.

---

## Business Cases Mapeados

| ID   | Business Case                     | Categoria   | Objetivo                             |
| ---- | --------------------------------- | ----------- | ------------------------------------ |
| BC1  | Configuração de Envio             | Logística   | Ativar plataformas e transportadoras |
| BC2  | Compra de Etiquetas               | Logística   | Monetização via etiquetas            |
| BC3  | Ativação e Monetização Loggi      | Logística   | Uso recorrente da Loggi              |
| BC4  | Contratação de Planos Pagos       | Monetização | Converter gratuito para plano pago   |
| BC5  | Criar Primeiro Produto            | Onboarding  | Produto configurado em 7 dias        |
| BC6  | Configurar Meio de Pagamento      | Onboarding  | Ativar gateway de pagamento com validação |
| BC7  | Personalizar Vitrine na Komea     | Onboarding  | Cor e logo configurados              |
| BC8  | Publicar Site na Komea            | Onboarding  | Site fora de manutenção              |
| BC9  | Usar Komea como Copiloto          | Engajamento | Uso recorrente da Komea              |
| BC10 | Emissão de Nota Fiscal            | Fiscal      | Primeira NF emitida                  |
| BC11 | Ativação de Canais Marketplace    | Expansão    | Anúncios em marketplaces             |

---

## Eventos por Categoria

### 1. Onboarding e Ativação

| Evento                      | Descrição                     | Trigger            | Propriedades                             | BC    |
| --------------------------- | ----------------------------- | ------------------ | ---------------------------------------- | ----- |
| `Store Created`             | Loja criada na plataforma     | Cadastro concluído | `store_id`, `plan_type`, `signup_source` | -     |
| `Onboarding Step Completed` | Etapa do onboarding concluída | Conclusão de etapa | `step_name`, `step_number`               | Todos |

### 2. Envio e Logística (BC1, BC2, BC3)

| Evento                        | Descrição                     | Trigger                      | Propriedades                                                                                                                          | BC      |
| ----------------------------- | ----------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| `Shipping Platform Activated` | Plataforma de envio ativada   | Ativar plataforma            | `shipping_platform`, `activation_source`, `fields_completed`                                                                          | BC1     |
| `Shipping Method Enabled`     | Método de envio ativado       | Ativação de transportadora   | `shipping_platform`, `carrier_name`, `carrier_type`, `is_correios`, `services_enabled`                                                | BC1/BC3 |
| `Label Flow Started`          | Iniciou fluxo de emissão      | Acesso ao fluxo de etiquetas | `shipping_platform`, `flow_source`, `order_id`                                                                                        | BC2     |
| `Label Purchased` ⭐          | Etiqueta comprada             | Compra concluída             | `shipping_platform`, `order_id`, `carrier_name`, `carrier_type`, `amount`, `payment_method`, `delivery_time`, `is_first_purchase`, `label_id` | BC2/BC3 |
| `Label Issued`                | Etiqueta emitida              | Emissão concluída            | `shipping_platform`, `order_id`, `carrier_name`, `tracking_code`                                                                      | BC2     |
| `Shipping Balance Added` ⭐   | Saldo adicionado à plataforma | Conclusão do pagamento       | `shipping_platform`, `amount`, `payment_method`, `new_balance`                                                                        | BC2     |
| `Order Shipped`               | Pedido enviado                | Postagem confirmada          | `shipping_platform`, `order_id`, `carrier_name`, `tracking_code`                                                                      | BC2     |

> **Nota:** Todos os eventos de logística incluem a propriedade `shipping_platform` que identifica a plataforma de envio utilizada. Valores possíveis: `"enviali"`, `"fretnet"`, `"melhor_envio"`, `"mandabem"`, `"frete_barato"`, `"go_fretes"`, `"freep"`. Para filtrar por transportadora específica (ex: Loggi, Correios), use a propriedade `carrier_name` na segmentação.

### 3. Pagamentos e Assinatura (BC4, BC6)

| Evento                          | Descrição                   | Trigger                       | Propriedades                                                                                                                   | BC  |
| ------------------------------- | --------------------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | --- |
| `Plans Page Viewed`             | Visualizou página de planos | Acesso à página               | `current_plan`, `referrer`                                                                                                     | BC4 |
| `Plan Selected`                 | Selecionou um plano         | Clique em selecionar plano    | `plan_name`, `billing_cycle`, `price`                                                                                          | BC4 |
| `Checkout Started`              | Iniciou checkout            | Entrada no checkout           | `plan_name`, `billing_cycle`, `coupon_code`                                                                                    | BC4 |
| `Subscription Completed` ⭐     | Assinatura concluída        | Pagamento confirmado          | `plan_name`, `billing_cycle`, `amount`, `payment_method`, `coupon_used`, `coupon_code`, `discount_percentage`, `state`, `city` | BC4 |
| `Gateway Registration Started`   | Iniciou cadastro no gateway   | Acesso ao cadastro            | `payment_gateway`, `entry_source`                                                                                              | BC6 |
| `Gateway Registration Completed` | Finalizou cadastro no gateway | Envio para análise            | `payment_gateway`, `account_type`, `documents_submitted`, `steps_completed`                                                    | BC6 |
| `Gateway Account Approved`       | Conta do gateway aprovada     | Aprovação da análise          | `payment_gateway`, `payment_methods_enabled`, `approval_date`                                                                  | BC6 |
| `Gateway Account Rejected`       | Conta do gateway rejeitada    | Rejeição da análise           | `payment_gateway`, `rejection_reason`                                                                                          | BC6 |
| `Payment Method Enabled`         | Meio de pagamento ativado     | Ativação de Pix/Cartão/Boleto | `payment_gateway`, `payment_method_type`                                                                                       | BC6 |

> **Nota:** Todos os eventos de gateway de pagamento incluem a propriedade `payment_gateway` que identifica o gateway utilizado. Valores possíveis: `"pagali"`, `"mercado_pago"`, `"app_max"`, `"pagseguro"`, `"pagar_me"`, `"paypal"`. O evento `Mercado Pago Configured` foi absorvido pelos eventos genéricos — use `payment_gateway = "mercado_pago"` para filtrar.

### 4. Produtos e Catálogo (BC5)

| Evento                     | Descrição                  | Trigger                 | Propriedades                                                                | BC  |
| -------------------------- | -------------------------- | ----------------------- | --------------------------------------------------------------------------- | --- |
| `Product Page Accessed`    | Acessou página de produtos | Navegação ao menu       | `entry_source`                                                              | BC5 |
| `Product Creation Started` | Iniciou criação de produto | Clique em criar produto | `creation_method`, `entry_source`                                           | BC5 |
| `Product Created`          | Produto criado com sucesso | Salvamento do produto   | `product_id`, `creation_method`, `category`, `has_images`, `has_variations` | BC5 |

### 5. Komea - Copiloto IA (BC7, BC8, BC9)

| Evento                           | Descrição                     | Trigger                  | Propriedades                                   | BC          |
| -------------------------------- | ----------------------------- | ------------------------ | ---------------------------------------------- | ----------- |
| `Komea Accessed`                 | Acessou a Komea               | Entrada na interface     | `entry_source`, `session_number`               | BC7/BC8/BC9 |
| `Komea Logo Path Selected`       | Selecionou caminho do logo    | Escolha subir ou gerar   | `path_type`                                    | BC7         |
| `Komea Logo Selected`            | Logo selecionado              | Escolha do logo final    | `is_ai_generated`, `option_selected`           | BC7         |
| `Komea Color Selected`           | Cor selecionada               | Escolha da cor           | `color_value`, `color_name`                    | BC7         |
| `Komea Customization Completed`  | Personalização concluída      | Finalização da vitrine   | `logo_source`, `color_selected`                | BC7         |
| `Komea Site Publication Started` | Iniciou publicação do site    | Clique em publicar       | `has_payment_method`, `has_product`            | BC8         |
| `Komea Site Data Filled`         | Preencheu dados do site       | CPF/CNPJ e endereço      | `account_type`, `state`, `city`                | BC8         |
| `Komea Site Published`           | Site publicado                | Saída do modo manutenção | `has_payment_method`, `has_product`            | BC8         |
| `Komea Opportunities Viewed`     | Visualizou oportunidades      | Acesso ao painel         | `opportunities_count`, `opportunities_types`   | BC9         |
| `Komea Opportunity Clicked`      | Clicou em oportunidade        | Clique para ver detalhes | `opportunity_type`, `opportunity_id`           | BC9         |
| `Komea Opportunity Executed`     | Executou ação de oportunidade | Conclusão da ação        | `opportunity_type`, `opportunity_id`, `result` | BC9         |
| `Komea Assistant Accessed`       | Acessou assistente            | Entrada no assistente    | `assistant_type`                               | BC9         |
| `Komea Question Asked`           | Fez pergunta ao assistente    | Envio de pergunta        | `assistant_type`, `question_category`          | BC9         |
| `Komea Action Executed`          | Executou ação sugerida        | Conclusão de ação        | `action_type`, `assistant_type`, `result`      | BC9         |
| `Komea Left For Panel`           | Saiu da Komea para o painel   | Navegação ao painel      | `current_step`                                 | BC7/BC8/BC9 |

### 6. Canais de Venda - Marketplaces (BC11)

| Evento                                  | Descrição                     | Trigger                  | Propriedades                                                                    | BC   |
| --------------------------------------- | ----------------------------- | ------------------------ | ------------------------------------------------------------------------------- | ---- |
| `Hub Channels Accessed`                 | Acessou Hub de Canais         | Navegação ao menu        | `current_plan`                                                                  | BC11 |
| `Marketplace Selected`                  | Selecionou marketplace        | Clique no marketplace    | `marketplace`, `marketplace_name`                                               | BC11 |
| `Marketplace Connection Started`        | Iniciou conexão               | Clique em configurar     | `marketplace`, `has_existing_account`                                           | BC11 |
| `Marketplace Connected`                 | Conectou ao marketplace       | Login/cadastro concluído | `marketplace`, `account_type`, `connection_date`                                | BC11 |
| `Marketplace Connection Failed`         | Falha na conexão              | Erro na autenticação     | `marketplace`, `error_type`, `error_message`                                    | BC11 |
| `Marketplace Initial Setup Started`     | Iniciou config. inicial       | Entrada na configuração  | `marketplace`                                                                   | BC11 |
| `Marketplace Initial Setup Completed`   | Completou config. inicial     | Salvamento das configs   | `marketplace`, `minimum_stock`, `classic_percentage`, `premium_percentage`       | BC11 |
| `Marketplace Products Selected`         | Selecionou produtos p/ enviar | Seleção de produtos      | `marketplace`, `products_count`, `categories`                                   | BC11 |
| `Marketplace Ad Type Selected`          | Selecionou tipo de anúncio    | Escolha Classic/Premium  | `marketplace`, `ad_type`, `products_count`                                      | BC11 |
| `Marketplace Ad Published` ⭐           | Anúncio publicado com sucesso | Confirmação do marketplace | `marketplace`, `product_id`, `ad_type`, `ad_id`, `total_value`, `category`    | BC11 |
| `Marketplace Ad Failed`                 | Falha na publicação           | Erro do marketplace      | `marketplace`, `product_id`, `error_type`, `error_message`                      | BC11 |
| `Marketplace Sale Completed` ⭐         | Venda realizada via marketplace | Pedido confirmado      | `marketplace`, `order_id`, `ad_type`, `amount`, `product_id`                    | BC11 |

> **Nota:** Todos os eventos de marketplace incluem a propriedade `marketplace` para identificar o marketplace utilizado. Valores: `"mercado_livre"`, `"magalu"`, `"allever"`, `"compre_sua_peca"`. Propriedades como `ad_type`, `classic_percentage`, `premium_percentage` são específicas do Mercado Livre e opcionais para outros marketplaces.

### 7. Fiscal NFe (BC10)

| Evento                    | Descrição                | Trigger                | Propriedades                                       | BC   |
| ------------------------- | ------------------------ | ---------------------- | -------------------------------------------------- | ---- |
| `NFe Page Accessed`       | Acessou página de NF     | Navegação ao menu      | `tax_regime`                                       | BC10 |
| `NFe Settings Configured` | Configurou dados fiscais | Salvamento das configs | `tax_regime`, `certificate_uploaded`               | BC10 |
| `NFe Emission Started`    | Iniciou emissão de NF    | Clique em emitir       | `order_id`, `tax_regime`                           | BC10 |
| `NFe Emitted`             | NF emitida com sucesso   | Emissão concluída      | `order_id`, `NFe_number`, `amount`, `is_first_NFe` | BC10 |
| `NFe Emission Failed`     | Falha na emissão         | Erro na emissão        | `order_id`, `error_type`, `error_message`          | BC10 |

---

## Funnels Definidos

### Funil 1: Ativação de Plataforma de Envio (BC1)

**Objetivo:** Medir taxa de ativação completa da plataforma de envio

| Step | Evento                        | Taxa Esperada  |
| ---- | ----------------------------- | -------------- |
| 1    | `Shipping Platform Activated` | 100% (entrada) |
| 2    | `Shipping Method Enabled`     | ~50%           |

### Funil 2: Compra de Etiquetas (BC2)

**Objetivo:** Medir conversão de compra de etiquetas

| Step | Evento               | Taxa Esperada  |
| ---- | -------------------- | -------------- |
| 1    | `Label Flow Started` | 100% (entrada) |
| 2    | `Label Purchased`    | ~55%           |
| 3    | `Label Issued`       | ~50%           |
| 4    | `Order Shipped`      | ~45%           |

### Funil 3: Contratação de Plano (BC4)

**Objetivo:** Medir conversão de assinatura

| Step | Evento                   | Taxa Esperada  |
| ---- | ------------------------ | -------------- |
| 1    | `Plans Page Viewed`      | 100% (entrada) |
| 2    | `Plan Selected`          | ~60%           |
| 3    | `Checkout Started`       | ~45%           |
| 4    | `Subscription Completed` | ~25%           |

### Funil 4: Configuração do Gateway de Pagamento (BC6)

**Objetivo:** Medir taxa de aprovação do gateway de pagamento

| Step | Evento                           | Taxa Esperada  |
| ---- | -------------------------------- | -------------- |
| 1    | `Gateway Registration Started`   | 100% (entrada) |
| 2    | `Gateway Registration Completed` | ~60%           |
| 3    | `Gateway Account Approved`       | ~50%           |

### Funil 5: Primeiro Produto (BC5)

**Objetivo:** Medir taxa de criação do primeiro produto

| Step | Evento                     | Taxa Esperada  |
| ---- | -------------------------- | -------------- |
| 1    | `Product Page Accessed`    | 100% (entrada) |
| 2    | `Product Creation Started` | ~75%           |
| 3    | `Product Created`          | ~50%           |

### Funil 6: Publicação do Site (BC8)

**Objetivo:** Medir taxa de publicação via Komea

| Step | Evento                           | Taxa Esperada  |
| ---- | -------------------------------- | -------------- |
| 1    | `Komea Site Publication Started` | 100% (entrada) |
| 2    | `Komea Site Data Filled`         | ~80%           |
| 3    | `Komea Site Published`           | ~70%           |

### Funil 7: Ativação de Marketplace (BC11)

**Objetivo:** Medir taxa de ativação de canais marketplace

| Step | Evento                                | Taxa Esperada  |
| ---- | ------------------------------------- | -------------- |
| 1    | `Hub Channels Accessed`               | 100% (entrada) |
| 2    | `Marketplace Connection Started`      | ~70%           |
| 3    | `Marketplace Connected`               | ~55%           |
| 4    | `Marketplace Initial Setup Completed` | ~45%           |
| 5    | `Marketplace Products Selected`       | ~35%           |
| 6    | `Marketplace Ad Published`            | ~30%           |

---

## Matriz Evento x Campanha

| Evento                           | Campanhas Relacionadas                                           |
| -------------------------------- | ---------------------------------------------------------------- |
| `Shipping Platform Activated`    | Winback config. incompleta (Inaction), Incentivo transportadoras |
| `Shipping Method Enabled`        | Confirmação config., Próximo passo, Incentivo primeira etiqueta  |
| `Label Flow Started`             | Winback emissão (Inaction: Started sem Purchased)                |
| `Label Purchased`                | Confirmação compra, Estímulo recorrência                         |
| `Plans Page Viewed`              | Recuperação abandono (Inaction: Viewed sem Completed)            |
| `Checkout Started`               | Recuperação abandono (Inaction: Started sem Completed)           |
| `Subscription Completed`         | Confirmação contratação, Meta assinatura                         |
| `Gateway Registration Started`   | Winback cadastro (Inaction: Started sem Completed)               |
| `Gateway Account Rejected`       | Orientação gateway alternativo                                   |
| `Product Creation Started`       | Régua abandono (Inaction: Started sem Created)                   |
| `Product Created`                | Régua educacional qualidade                                      |
| `Komea Customization Completed`  | Confirmação personalização                                       |
| `Komea Site Publication Started` | Régua engajamento publicação (Inaction)                          |
| `Marketplace Connection Started`   | Jornada ativação marketplace                                     |
| `Marketplace Products Selected`    | Expansão catálogo                                                |
| `Marketplace Ad Published`         | Incentivo anúncios premium                                       |

---

## Eventos de Monetização (⭐)

Eventos marcados com ⭐ representam ações de monetização direta:

| Evento                   | Tipo de Receita           | Relevância |
| ------------------------ | ------------------------- | ---------- |
| `Subscription Completed` | Assinatura                | Alta       |
| `Label Purchased`        | Etiquetas                 | Alta       |
| `Shipping Balance Added` | Saldo plataforma de envio | Média      |
| `Marketplace Ad Published`    | Comissão indireta         | Média      |
| `Marketplace Sale Completed`  | Comissão indireta         | Alta       |

---

## Requisitos Técnicos

### SDK Versions

| Plataforma | Versão Mínima | Versão Recomendada |
| ---------- | ------------- | ------------------ |
| Web        | v1.6.0        | v2.3.3+            |

### Integrações Necessárias

- [ ] CleverTap Web SDK
- [ ] Email Provider integration
- [ ] WhatsApp BSP (integração Blip)
- [ ] Web Native Channels

---

## Changelog

| Data       | Versão | Alteração                            | Autor |
| ---------- | ------ | ------------------------------------ | ----- |
| 05/01/2026 | 1.0    | Versão inicial com 11 Business Cases | RMH   |
| 09/02/2026 | 1.1    | Eventos BC6 gateway-agnósticos + nomes de planos atualizados | RMH   |
| 09/02/2026 | 1.2    | Eventos BC11 marketplace-agnósticos (ML * → Marketplace *) | RMH   |
