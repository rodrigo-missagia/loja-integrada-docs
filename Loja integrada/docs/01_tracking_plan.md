# Tracking Plan - Loja Integrada

**Versão:** 2.0
**Data:** 12 de Março de 2026
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

> **Convenções de nomenclatura:**
> - Nomes de eventos: Português, formato `Objeto Acao` (sem acentos)
> - Nomes de propriedades: Português, formato `snake_case` (sem acentos)

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

| Evento                         | Descrição                     | Trigger            | Propriedades                                  | BC    |
| ------------------------------ | ----------------------------- | ------------------ | --------------------------------------------- | ----- |
| `Loja Criada`                  | Loja criada na plataforma     | Cadastro concluído | `id_loja`, `tipo_plano`, `origem_cadastro`    | -     |
| `Etapa Onboarding Concluida`   | Etapa do onboarding concluída | Conclusão de etapa | `nome_etapa`, `numero_etapa`                  | Todos |

### 2. Envio e Logística (BC1, BC2, BC3)

| Evento                            | Descrição                     | Trigger                      | Propriedades                                                                                                                                                 | BC      |
| --------------------------------- | ----------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- |
| `Plataforma Envio Ativada`        | Plataforma de envio ativada   | Ativar plataforma            | `plataforma_envio`, `origem_ativacao`, `campos_preenchidos`                                                                                                  | BC1     |
| `Metodo Envio Ativado`            | Método de envio ativado       | Ativação de transportadora   | `plataforma_envio`, `nome_transportadora`, `tipo_transportadora`, `eh_correios`, `servicos_ativados`                                                         | BC1/BC3 |
| `Fluxo Etiqueta Iniciado`         | Iniciou fluxo de emissão      | Acesso ao fluxo de etiquetas | `plataforma_envio`, `origem_fluxo`, `id_pedido`                                                                                                              | BC2     |
| `Etiqueta Comprada` ⭐            | Etiqueta comprada             | Compra concluída             | `plataforma_envio`, `id_pedido`, `nome_transportadora`, `tipo_transportadora`, `valor`, `meio_pagamento`, `prazo_entrega`, `eh_primeira_compra`, `id_etiqueta` | BC2/BC3 |
| `Etiqueta Emitida`                | Etiqueta emitida              | Emissão concluída            | `plataforma_envio`, `id_pedido`, `nome_transportadora`, `codigo_rastreio`                                                                                    | BC2     |
| `Saldo Envio Adicionado` ⭐       | Saldo adicionado à plataforma | Conclusão do pagamento       | `plataforma_envio`, `valor`, `meio_pagamento`, `novo_saldo`                                                                                                  | BC2     |
| `Pedido Enviado`                  | Pedido enviado                | Postagem confirmada          | `plataforma_envio`, `id_pedido`, `nome_transportadora`, `codigo_rastreio`                                                                                    | BC2     |

> **Nota:** Todos os eventos de logística incluem a propriedade `plataforma_envio` que identifica a plataforma de envio utilizada. Valores possíveis: `"enviali"`, `"fretnet"`, `"melhor_envio"`, `"mandabem"`, `"frete_barato"`, `"go_fretes"`, `"freep"`. Para filtrar por transportadora específica (ex: Loggi, Correios), use a propriedade `nome_transportadora` na segmentação.

### 3. Pagamentos e Assinatura (BC4, BC6)

| Evento                            | Descrição                   | Trigger                       | Propriedades                                                                                                                                    | BC  |
| --------------------------------- | --------------------------- | ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| `Pagina Planos Visualizada`       | Visualizou página de planos | Acesso à página               | `plano_atual`, `origem`                                                                                                                         | BC4 |
| `Plano Selecionado`               | Selecionou um plano         | Clique em selecionar plano    | `nome_plano`, `ciclo_cobranca`, `preco`                                                                                                         | BC4 |
| `Checkout Iniciado`               | Iniciou checkout            | Entrada no checkout           | `nome_plano`, `ciclo_cobranca`, `codigo_cupom`                                                                                                  | BC4 |
| `Assinatura Concluida` ⭐         | Assinatura concluída        | Pagamento confirmado          | `nome_plano`, `ciclo_cobranca`, `valor`, `meio_pagamento`, `cupom_utilizado`, `codigo_cupom`, `percentual_desconto`, `estado`, `cidade`         | BC4 |
| `Cadastro Gateway Iniciado`       | Iniciou cadastro no gateway   | Acesso ao cadastro            | `gateway_pagamento`, `origem_entrada`                                                                                                           | BC6 |
| `Cadastro Gateway Concluido`      | Finalizou cadastro no gateway | Envio para análise            | `gateway_pagamento`, `tipo_conta`, `documentos_enviados`, `etapas_concluidas`                                                                   | BC6 |
| `Conta Gateway Aprovada`          | Conta do gateway aprovada     | Aprovação da análise          | `gateway_pagamento`, `meios_pagamento_ativados`, `data_aprovacao`                                                                               | BC6 |
| `Conta Gateway Rejeitada`         | Conta do gateway rejeitada    | Rejeição da análise           | `gateway_pagamento`, `motivo_rejeicao`                                                                                                          | BC6 |
| `Meio Pagamento Ativado`          | Meio de pagamento ativado     | Ativação de Pix/Cartão/Boleto | `gateway_pagamento`, `tipo_meio_pagamento`                                                                                                      | BC6 |

> **Nota:** Todos os eventos de gateway de pagamento incluem a propriedade `gateway_pagamento` que identifica o gateway utilizado. Valores possíveis: `"pagali"`, `"mercado_pago"`, `"app_max"`, `"pagseguro"`, `"pagar_me"`, `"paypal"`. O evento `Mercado Pago Configured` foi absorvido pelos eventos genéricos — use `gateway_pagamento = "mercado_pago"` para filtrar.

### 4. Produtos e Catálogo (BC5)

| Evento                        | Descrição                  | Trigger                 | Propriedades                                                                     | BC  |
| ----------------------------- | -------------------------- | ----------------------- | -------------------------------------------------------------------------------- | --- |
| `Pagina Produto Acessada`     | Acessou página de produtos | Navegação ao menu       | `origem_entrada`                                                                 | BC5 |
| `Criacao Produto Iniciada`    | Iniciou criação de produto | Clique em criar produto | `metodo_criacao`, `origem_entrada`                                               | BC5 |
| `Produto Criado`              | Produto criado com sucesso | Salvamento do produto   | `id_produto`, `metodo_criacao`, `categoria`, `tem_imagens`, `tem_variacoes`      | BC5 |

### 5. Komea - Copiloto IA (BC7, BC8, BC9)

| Evento                              | Descrição                     | Trigger                  | Propriedades                                      | BC          |
| ----------------------------------- | ----------------------------- | ------------------------ | ------------------------------------------------- | ----------- |
| `Komea Acessada`                    | Acessou a Komea               | Entrada na interface     | `origem_entrada`, `numero_sessao`                 | BC7/BC8/BC9 |
| `Komea Fluxo Logo Selecionado`      | Selecionou caminho do logo    | Escolha subir ou gerar   | `tipo_caminho`                                    | BC7         |
| `Komea Logo Selecionado`            | Logo selecionado              | Escolha do logo final    | `gerado_por_ia`, `opcao_selecionada`              | BC7         |
| `Komea Cor Selecionada`             | Cor selecionada               | Escolha da cor           | `valor_cor`, `nome_cor`                           | BC7         |
| `Komea Personalizacao Concluida`    | Personalização concluída      | Finalização da vitrine   | `origem_logo`, `cor_selecionada`                  | BC7         |
| `Komea Publicacao Site Iniciada`    | Iniciou publicação do site    | Clique em publicar       | `tem_meio_pagamento`, `tem_produto`               | BC8         |
| `Komea Dados Site Preenchidos`      | Preencheu dados do site       | CPF/CNPJ e endereço      | `tipo_conta`, `estado`, `cidade`                  | BC8         |
| `Komea Site Publicado`              | Site publicado                | Saída do modo manutenção | `tem_meio_pagamento`, `tem_produto`               | BC8         |
| `Komea Oportunidades Visualizadas`  | Visualizou oportunidades      | Acesso ao painel         | `qtd_oportunidades`, `tipos_oportunidades`        | BC9         |
| `Komea Oportunidade Clicada`        | Clicou em oportunidade        | Clique para ver detalhes | `tipo_oportunidade`, `id_oportunidade`            | BC9         |
| `Komea Oportunidade Executada`      | Executou ação de oportunidade | Conclusão da ação        | `tipo_oportunidade`, `id_oportunidade`, `resultado` | BC9       |
| `Komea Assistente Acessado`         | Acessou assistente            | Entrada no assistente    | `tipo_assistente`                                 | BC9         |
| `Komea Pergunta Feita`              | Fez pergunta ao assistente    | Envio de pergunta        | `tipo_assistente`, `categoria_pergunta`            | BC9         |
| `Komea Acao Executada`              | Executou ação sugerida        | Conclusão de ação        | `tipo_acao`, `tipo_assistente`, `resultado`        | BC9         |
| `Komea Saiu Para Painel`            | Saiu da Komea para o painel   | Navegação ao painel      | `etapa_atual`                                     | BC7/BC8/BC9 |

### 6. Canais de Venda - Marketplaces (BC11)

| Evento                                     | Descrição                       | Trigger                    | Propriedades                                                                       | BC   |
| ------------------------------------------ | ------------------------------- | -------------------------- | ---------------------------------------------------------------------------------- | ---- |
| `Hub Canais Acessado`                      | Acessou Hub de Canais           | Navegação ao menu          | `plano_atual`                                                                      | BC11 |
| `Marketplace Selecionado`                  | Selecionou marketplace          | Clique no marketplace      | `marketplace`, `nome_marketplace`                                                  | BC11 |
| `Marketplace Conexao Iniciada`             | Iniciou conexão                 | Clique em configurar       | `marketplace`, `tem_conta_existente`                                               | BC11 |
| `Marketplace Conectado`                    | Conectou ao marketplace         | Login/cadastro concluído   | `marketplace`, `tipo_conta`, `data_conexao`                                        | BC11 |
| `Marketplace Conexao Falhou`               | Falha na conexão                | Erro na autenticação       | `marketplace`, `tipo_erro`, `mensagem_erro`                                        | BC11 |
| `Marketplace Config Inicial Iniciada`      | Iniciou config. inicial         | Entrada na configuração    | `marketplace`                                                                      | BC11 |
| `Marketplace Config Inicial Concluida`     | Completou config. inicial       | Salvamento das configs     | `marketplace`, `estoque_minimo`, `percentual_classico`, `percentual_premium`       | BC11 |
| `Marketplace Produtos Selecionados`        | Selecionou produtos p/ enviar   | Seleção de produtos        | `marketplace`, `qtd_produtos`, `categorias`                                        | BC11 |
| `Marketplace Tipo Anuncio Selecionado`     | Selecionou tipo de anúncio      | Escolha Classic/Premium    | `marketplace`, `tipo_anuncio`, `qtd_produtos`                                      | BC11 |
| `Marketplace Anuncio Publicado` ⭐         | Anúncio publicado com sucesso   | Confirmação do marketplace | `marketplace`, `id_produto`, `tipo_anuncio`, `id_anuncio`, `valor_total`, `categoria` | BC11 |
| `Marketplace Anuncio Falhou`               | Falha na publicação             | Erro do marketplace        | `marketplace`, `id_produto`, `tipo_erro`, `mensagem_erro`                          | BC11 |
| `Marketplace Venda Concluida` ⭐           | Venda realizada via marketplace | Pedido confirmado          | `marketplace`, `id_pedido`, `tipo_anuncio`, `valor`, `id_produto`                  | BC11 |

> **Nota:** Todos os eventos de marketplace incluem a propriedade `marketplace` para identificar o marketplace utilizado. Valores: `"mercado_livre"`, `"magalu"`, `"allever"`, `"compre_sua_peca"`. Propriedades como `tipo_anuncio`, `percentual_classico`, `percentual_premium` são específicas do Mercado Livre e opcionais para outros marketplaces.

### 7. Fiscal NFe (BC10)

| Evento                      | Descrição                | Trigger                | Propriedades                                            | BC   |
| --------------------------- | ------------------------ | ---------------------- | ------------------------------------------------------- | ---- |
| `NFe Pagina Acessada`       | Acessou página de NF     | Navegação ao menu      | `regime_tributario`                                     | BC10 |
| `NFe Configuracoes Salvas`  | Configurou dados fiscais | Salvamento das configs | `regime_tributario`, `certificado_enviado`               | BC10 |
| `NFe Emissao Iniciada`      | Iniciou emissão de NF    | Clique em emitir       | `id_pedido`, `regime_tributario`                        | BC10 |
| `NFe Emitida`               | NF emitida com sucesso   | Emissão concluída      | `id_pedido`, `numero_nfe`, `valor`, `eh_primeira_nfe`   | BC10 |
| `NFe Emissao Falhou`        | Falha na emissão         | Erro na emissão        | `id_pedido`, `tipo_erro`, `mensagem_erro`               | BC10 |

---

## Funnels Definidos

### Funil 1: Ativação de Plataforma de Envio (BC1)

**Objetivo:** Medir taxa de ativação completa da plataforma de envio

| Step | Evento                       | Taxa Esperada  |
| ---- | ---------------------------- | -------------- |
| 1    | `Plataforma Envio Ativada`   | 100% (entrada) |
| 2    | `Metodo Envio Ativado`       | ~50%           |

### Funil 2: Compra de Etiquetas (BC2)

**Objetivo:** Medir conversão de compra de etiquetas

| Step | Evento                    | Taxa Esperada  |
| ---- | ------------------------- | -------------- |
| 1    | `Fluxo Etiqueta Iniciado` | 100% (entrada) |
| 2    | `Etiqueta Comprada`       | ~55%           |
| 3    | `Etiqueta Emitida`        | ~50%           |
| 4    | `Pedido Enviado`          | ~45%           |

### Funil 3: Contratação de Plano (BC4)

**Objetivo:** Medir conversão de assinatura

| Step | Evento                        | Taxa Esperada  |
| ---- | ----------------------------- | -------------- |
| 1    | `Pagina Planos Visualizada`   | 100% (entrada) |
| 2    | `Plano Selecionado`           | ~60%           |
| 3    | `Checkout Iniciado`           | ~45%           |
| 4    | `Assinatura Concluida`        | ~25%           |

### Funil 4: Configuração do Gateway de Pagamento (BC6)

**Objetivo:** Medir taxa de aprovação do gateway de pagamento

| Step | Evento                        | Taxa Esperada  |
| ---- | ----------------------------- | -------------- |
| 1    | `Cadastro Gateway Iniciado`   | 100% (entrada) |
| 2    | `Cadastro Gateway Concluido`  | ~60%           |
| 3    | `Conta Gateway Aprovada`      | ~50%           |

### Funil 5: Primeiro Produto (BC5)

**Objetivo:** Medir taxa de criação do primeiro produto

| Step | Evento                      | Taxa Esperada  |
| ---- | --------------------------- | -------------- |
| 1    | `Pagina Produto Acessada`   | 100% (entrada) |
| 2    | `Criacao Produto Iniciada`  | ~75%           |
| 3    | `Produto Criado`            | ~50%           |

### Funil 6: Publicação do Site (BC8)

**Objetivo:** Medir taxa de publicação via Komea

| Step | Evento                            | Taxa Esperada  |
| ---- | --------------------------------- | -------------- |
| 1    | `Komea Publicacao Site Iniciada`  | 100% (entrada) |
| 2    | `Komea Dados Site Preenchidos`    | ~80%           |
| 3    | `Komea Site Publicado`            | ~70%           |

### Funil 7: Ativação de Marketplace (BC11)

**Objetivo:** Medir taxa de ativação de canais marketplace

| Step | Evento                                  | Taxa Esperada  |
| ---- | --------------------------------------- | -------------- |
| 1    | `Hub Canais Acessado`                   | 100% (entrada) |
| 2    | `Marketplace Conexao Iniciada`          | ~70%           |
| 3    | `Marketplace Conectado`                 | ~55%           |
| 4    | `Marketplace Config Inicial Concluida`  | ~45%           |
| 5    | `Marketplace Produtos Selecionados`     | ~35%           |
| 6    | `Marketplace Anuncio Publicado`         | ~30%           |

---

## Matriz Evento x Campanha

| Evento                             | Campanhas Relacionadas                                           |
| ---------------------------------- | ---------------------------------------------------------------- |
| `Plataforma Envio Ativada`         | Winback config. incompleta (Inaction), Incentivo transportadoras |
| `Metodo Envio Ativado`             | Confirmação config., Próximo passo, Incentivo primeira etiqueta  |
| `Fluxo Etiqueta Iniciado`          | Winback emissão (Inaction: Iniciado sem Comprado)                |
| `Etiqueta Comprada`                | Confirmação compra, Estímulo recorrência                         |
| `Pagina Planos Visualizada`        | Recuperação abandono (Inaction: Visualizada sem Concluida)       |
| `Checkout Iniciado`                | Recuperação abandono (Inaction: Iniciado sem Concluida)          |
| `Assinatura Concluida`             | Confirmação contratação, Meta assinatura                         |
| `Cadastro Gateway Iniciado`        | Winback cadastro (Inaction: Iniciado sem Concluido)              |
| `Conta Gateway Rejeitada`          | Orientação gateway alternativo                                   |
| `Criacao Produto Iniciada`         | Régua abandono (Inaction: Iniciada sem Criado)                   |
| `Produto Criado`                   | Régua educacional qualidade                                      |
| `Komea Personalizacao Concluida`   | Confirmação personalização                                       |
| `Komea Publicacao Site Iniciada`   | Régua engajamento publicação (Inaction)                          |
| `Marketplace Conexao Iniciada`     | Jornada ativação marketplace                                     |
| `Marketplace Produtos Selecionados`| Expansão catálogo                                                |
| `Marketplace Anuncio Publicado`    | Incentivo anúncios premium                                       |

---

## Eventos de Monetização (⭐)

Eventos marcados com ⭐ representam ações de monetização direta:

| Evento                        | Tipo de Receita           | Relevância |
| ----------------------------- | ------------------------- | ---------- |
| `Assinatura Concluida`        | Assinatura                | Alta       |
| `Etiqueta Comprada`           | Etiquetas                 | Alta       |
| `Saldo Envio Adicionado`      | Saldo plataforma de envio | Média      |
| `Marketplace Anuncio Publicado`  | Comissão indireta         | Média      |
| `Marketplace Venda Concluida`   | Comissão indireta         | Alta       |

---

## Tabela de Correspondência EN → PT

Para referência durante a migração, segue a correspondência entre os nomes antigos (inglês) e os novos (português):

### Eventos

| Antigo (EN)                           | Novo (PT)                                |
| ------------------------------------- | ---------------------------------------- |
| `Store Created`                       | `Loja Criada`                            |
| `Onboarding Step Completed`           | `Etapa Onboarding Concluida`             |
| `Shipping Platform Activated`         | `Plataforma Envio Ativada`               |
| `Shipping Method Enabled`             | `Metodo Envio Ativado`                   |
| `Label Flow Started`                  | `Fluxo Etiqueta Iniciado`                |
| `Label Purchased`                     | `Etiqueta Comprada`                      |
| `Label Issued`                        | `Etiqueta Emitida`                       |
| `Shipping Balance Added`              | `Saldo Envio Adicionado`                 |
| `Order Shipped`                       | `Pedido Enviado`                         |
| `Plans Page Viewed`                   | `Pagina Planos Visualizada`              |
| `Plan Selected`                       | `Plano Selecionado`                      |
| `Checkout Started`                    | `Checkout Iniciado`                      |
| `Subscription Completed`              | `Assinatura Concluida`                   |
| `Gateway Registration Started`        | `Cadastro Gateway Iniciado`              |
| `Gateway Registration Completed`      | `Cadastro Gateway Concluido`             |
| `Gateway Account Approved`            | `Conta Gateway Aprovada`                 |
| `Gateway Account Rejected`            | `Conta Gateway Rejeitada`                |
| `Payment Method Enabled`              | `Meio Pagamento Ativado`                 |
| `Product Page Accessed`               | `Pagina Produto Acessada`                |
| `Product Creation Started`            | `Criacao Produto Iniciada`               |
| `Product Created`                     | `Produto Criado`                         |
| `Komea Accessed`                      | `Komea Acessada`                         |
| `Komea Logo Path Selected`            | `Komea Fluxo Logo Selecionado`           |
| `Komea Logo Selected`                 | `Komea Logo Selecionado`                 |
| `Komea Color Selected`                | `Komea Cor Selecionada`                  |
| `Komea Customization Completed`       | `Komea Personalizacao Concluida`         |
| `Komea Site Publication Started`      | `Komea Publicacao Site Iniciada`         |
| `Komea Site Data Filled`              | `Komea Dados Site Preenchidos`           |
| `Komea Site Published`                | `Komea Site Publicado`                   |
| `Komea Opportunities Viewed`          | `Komea Oportunidades Visualizadas`       |
| `Komea Opportunity Clicked`           | `Komea Oportunidade Clicada`             |
| `Komea Opportunity Executed`          | `Komea Oportunidade Executada`           |
| `Komea Assistant Accessed`            | `Komea Assistente Acessado`              |
| `Komea Question Asked`                | `Komea Pergunta Feita`                   |
| `Komea Action Executed`               | `Komea Acao Executada`                   |
| `Komea Left For Panel`                | `Komea Saiu Para Painel`                 |
| `Hub Channels Accessed`               | `Hub Canais Acessado`                    |
| `Marketplace Selected`                | `Marketplace Selecionado`                |
| `Marketplace Connection Started`      | `Marketplace Conexao Iniciada`           |
| `Marketplace Connected`               | `Marketplace Conectado`                  |
| `Marketplace Connection Failed`       | `Marketplace Conexao Falhou`             |
| `Marketplace Initial Setup Started`   | `Marketplace Config Inicial Iniciada`    |
| `Marketplace Initial Setup Completed` | `Marketplace Config Inicial Concluida`   |
| `Marketplace Products Selected`       | `Marketplace Produtos Selecionados`      |
| `Marketplace Ad Type Selected`        | `Marketplace Tipo Anuncio Selecionado`   |
| `Marketplace Ad Published`            | `Marketplace Anuncio Publicado`          |
| `Marketplace Ad Failed`               | `Marketplace Anuncio Falhou`             |
| `Marketplace Sale Completed`          | `Marketplace Venda Concluida`            |
| `NFe Page Accessed`                   | `NFe Pagina Acessada`                    |
| `NFe Settings Configured`             | `NFe Configuracoes Salvas`               |
| `NFe Emission Started`                | `NFe Emissao Iniciada`                   |
| `NFe Emitted`                         | `NFe Emitida`                            |
| `NFe Emission Failed`                 | `NFe Emissao Falhou`                     |

### Propriedades

| Antigo (EN)                | Novo (PT)                  |
| -------------------------- | -------------------------- |
| `store_id`                 | `id_loja`                  |
| `plan_type`                | `tipo_plano`               |
| `signup_source`            | `origem_cadastro`          |
| `step_name`                | `nome_etapa`               |
| `step_number`              | `numero_etapa`             |
| `shipping_platform`        | `plataforma_envio`         |
| `activation_source`        | `origem_ativacao`          |
| `fields_completed`         | `campos_preenchidos`       |
| `carrier_name`             | `nome_transportadora`      |
| `carrier_type`             | `tipo_transportadora`      |
| `is_correios`              | `eh_correios`              |
| `services_enabled`         | `servicos_ativados`        |
| `flow_source`              | `origem_fluxo`             |
| `order_id`                 | `id_pedido`                |
| `amount`                   | `valor`                    |
| `payment_method`           | `meio_pagamento`           |
| `delivery_time`            | `prazo_entrega`            |
| `is_first_purchase`        | `eh_primeira_compra`       |
| `label_id`                 | `id_etiqueta`              |
| `tracking_code`            | `codigo_rastreio`          |
| `new_balance`              | `novo_saldo`               |
| `current_plan`             | `plano_atual`              |
| `referrer`                 | `origem`                   |
| `plan_name`                | `nome_plano`               |
| `billing_cycle`            | `ciclo_cobranca`           |
| `price`                    | `preco`                    |
| `coupon_code`              | `codigo_cupom`             |
| `coupon_used`              | `cupom_utilizado`          |
| `discount_percentage`      | `percentual_desconto`      |
| `state`                    | `estado`                   |
| `city`                     | `cidade`                   |
| `payment_gateway`          | `gateway_pagamento`        |
| `entry_source`             | `origem_entrada`           |
| `account_type`             | `tipo_conta`               |
| `documents_submitted`      | `documentos_enviados`      |
| `steps_completed`          | `etapas_concluidas`        |
| `payment_methods_enabled`  | `meios_pagamento_ativados` |
| `approval_date`            | `data_aprovacao`           |
| `rejection_reason`         | `motivo_rejeicao`          |
| `payment_method_type`      | `tipo_meio_pagamento`      |
| `creation_method`          | `metodo_criacao`           |
| `product_id`               | `id_produto`               |
| `category`                 | `categoria`                |
| `has_images`               | `tem_imagens`              |
| `has_variations`           | `tem_variacoes`            |
| `session_number`           | `numero_sessao`            |
| `path_type`                | `tipo_caminho`             |
| `is_ai_generated`          | `gerado_por_ia`            |
| `option_selected`          | `opcao_selecionada`        |
| `color_value`              | `valor_cor`                |
| `color_name`               | `nome_cor`                 |
| `logo_source`              | `origem_logo`              |
| `color_selected`           | `cor_selecionada`          |
| `has_payment_method`       | `tem_meio_pagamento`       |
| `has_product`              | `tem_produto`              |
| `opportunities_count`      | `qtd_oportunidades`        |
| `opportunities_types`      | `tipos_oportunidades`      |
| `opportunity_type`         | `tipo_oportunidade`        |
| `opportunity_id`           | `id_oportunidade`          |
| `result`                   | `resultado`                |
| `assistant_type`           | `tipo_assistente`          |
| `question_category`        | `categoria_pergunta`       |
| `action_type`              | `tipo_acao`                |
| `current_step`             | `etapa_atual`              |
| `marketplace`              | `marketplace`              |
| `marketplace_name`         | `nome_marketplace`         |
| `has_existing_account`     | `tem_conta_existente`      |
| `connection_date`          | `data_conexao`             |
| `error_type`               | `tipo_erro`                |
| `error_message`            | `mensagem_erro`            |
| `minimum_stock`            | `estoque_minimo`           |
| `classic_percentage`       | `percentual_classico`      |
| `premium_percentage`       | `percentual_premium`       |
| `products_count`           | `qtd_produtos`             |
| `categories`               | `categorias`               |
| `ad_type`                  | `tipo_anuncio`             |
| `ad_id`                    | `id_anuncio`               |
| `total_value`              | `valor_total`              |
| `tax_regime`               | `regime_tributario`        |
| `certificate_uploaded`     | `certificado_enviado`      |
| `NFe_number`               | `numero_nfe`               |
| `is_first_NFe`             | `eh_primeira_nfe`          |

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
| 12/03/2026 | 2.0    | Tradução completa: eventos e propriedades de EN para PT (sem acentos) | RMH   |
