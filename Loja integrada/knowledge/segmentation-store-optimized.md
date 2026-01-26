# Relatório de Otimização - Segmentation Store

Este documento apresenta sugestões de otimização para redução de campos no CRM da Loja Integrada, consolidando propriedades redundantes e simplificando a estrutura de dados.

---

## Sumário Executivo

| Métrica | Valor |
|---------|-------|
| **Total de propriedades originais** | ~193 |
| **Propriedades após otimização** | ~89 |
| **Redução estimada** | ~54% |

---

## Tipo 1: Flags Booleanas Substituíveis por Datas

### Princípio
Quando existe uma flag booleana acompanhada de um campo de data relacionado, a presença da data pode servir como verificação equivalente ao booleano `TRUE`. A ausência de data (null) equivale a `FALSE`.

### Otimizações Identificadas

#### 1.1 Meio de Pagamento Configurado
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `pagamento_ativo` (Flag) | `data_cadastro_pagamento` (Data) |
| `data_cadastro_pagamento` (Data) | |

**Justificativa:** A presença de `data_cadastro_pagamento` indica que o pagamento foi configurado. Se a data existe, o pagamento está ativo. Elimina 1 campo.

#### 1.2 Meio de Envio Configurado
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `envio_ativo` (Flag) | `data_cadastro_envio` (Data) |
| `data_cadastro_envio` (Data) | |

**Justificativa:** A presença de `data_cadastro_envio` indica que o envio foi configurado. Elimina 1 campo.

#### 1.3 Cupom Ativo
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `cupom_ativo` (Flag) | `data_que_ativou_o_primeiro_cupom` (Data) |
| `data_que_ativou_o_primeiro_cupom` (Data) | |

**Justificativa:** A presença de data indica que cupom foi ativado. Elimina 1 campo.

#### 1.4 Compre Junto
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `compre_junto_ativo` (Flag) | `data_que_ativou_compre_junto` (Data) |
| `data_que_ativou_compre_junto` (Data) | |

**Justificativa:** A presença de data indica ativação. Elimina 1 campo.

#### 1.5 Compre Junto com Desconto
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `compre_junto_com_desconto_ativo` (Flag) | `data_que_ativou_compre_junto_desconto` (Data) |

**Justificativa:** Pode ser combinado com compre_junto usando o campo de data. Elimina 1 campo.

#### 1.6 Carrinho Abandonado
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `carrinho_abandonado_auto` (Flag) | `data_que_ativou_carrinho_abandonado` (Data) |
| `data_que_ativou_carrinho_abandonado` (Data) | |

**Justificativa:** A presença de data indica ativação. Elimina 1 campo.

#### 1.7 Avaliação LI
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `ferramenta_avaliacao` (Flag) | `data_ativacao_avaliacao` (Data) |
| `data_ativacao_avaliacao` (Data) | |

**Justificativa:** A presença de data indica ativação. Elimina 1 campo.

#### 1.8 Abandono de Produto
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `abandono_de_produto_ativo` (Flag) | `data_abandono_produto` (Data) |
| `data_abandono_produto` (Data) | |

**Justificativa:** A presença de data indica ativação. Elimina 1 campo.

#### 1.9 Promoção Ativa
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `funcionalidade_promocao_ativa` (Flag) | `data_da_primeira_promocao` (Data) |
| `data_da_primeira_promocao` (Data) | |

**Justificativa:** A presença de data indica ativação. Elimina 1 campo.

#### 1.10 Configuração Enviali
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `finalizou_configuracao_enviali` (Flag) | `data_configuracao_do_enviali` (Data) |
| `data_configuracao_do_enviali` (Data) | |

**Justificativa:** A presença de data indica configuração finalizada. Elimina 1 campo.

---

## Tipo 2: Múltiplas Flags Consolidáveis em Arrays

### Princípio
Quando existem múltiplas flags booleanas que representam opções de uma mesma categoria (ex: meios de pagamento, meios de envio), podem ser consolidadas em um único campo array.

### Otimizações Identificadas

#### 2.1 Meios de Pagamento Ativos
| Campos Originais (20 campos) | Campo Otimizado |
|------------------------------|-----------------|
| `appmax_cartao` (Flag) | `meios_pagamento_ativos` (Array) |
| `appmax_boleto` (Flag) | |
| `cielo_cartao` (Flag) | Exemplo de valor: |
| `cielo_boleto` (Flag) | `["pagali_pix", "pagali_cartao", "mercado_pago_cartao"]` |
| `mercado_pago_redirect` (Flag) | |
| `mercado_pago_boleto` (Flag) | |
| `mercado_pago_cartao` (Flag) | |
| `outros_deposito` (Flag) | |
| `outros_pagamento_externo` (Flag) | |
| `pagali_pix` (Flag) | |
| `pagali_cartao` (Flag) | |
| `pagali_boleto` (Flag) | |
| `pagarme_boleto` (Flag) | |
| `pagarme_cartao` (Flag) | |
| `paghiper_boleto` (Flag) | |
| `pagseguro_cartao` (Flag) | |
| `pagseguro_boleto` (Flag) | |
| `pagseguro_redirect` (Flag) | |
| `paypal_cartao` (Flag) | |
| `paypal_redirect` (Flag) | |

**Justificativa:** 20 flags booleanas podem ser consolidadas em 1 array. Permite segmentação por "contém" e facilita adição de novos meios de pagamento sem criar novos campos. **Elimina 19 campos.**

#### 2.2 Meios de Envio Ativos
| Campos Originais (12 campos) | Campo Otimizado |
|------------------------------|-----------------|
| `pac_correios` (Flag) | `meios_envio_ativos` (Array) |
| `sedex` (Flag) | |
| `enviali` (Flag) | Exemplo de valor: |
| `melhor_envio` (Flag) | `["pac_correios", "sedex", "enviali", "loggi"]` |
| `frenet` (Flag) | |
| `manda_bem` (Flag) | |
| `kangu` (Flag) | |
| `frete_barato` (Flag) | |
| `forma_de_envio_retirada_em_maos` (Flag) | |
| `forma_de_envio_transportadora` (Flag) | |
| `forma_de_envio_motoboy` (Flag) | |
| `forma_de_envio_personalizada` (Flag) | |

**Justificativa:** 12 flags consolidadas em 1 array. **Elimina 11 campos.**

#### 2.3 Integrações/Apps Ativos
| Campos Originais (30+ campos) | Campo Otimizado |
|-------------------------------|-----------------|
| `allintegra_ativo` (Flag) | `integracoes_ativas` (Array) |
| `dropi_ativo` (Flag) | |
| `emanda_ativo` (Flag) | Exemplo de valor: |
| `enviou_ativo` (Flag) | `["dropi", "bling", "google_tag_manager", "ga4"]` |
| `fidelizar_ativo` (Flag) | |
| `google_tag_manager_ativo` (Flag) | |
| `google_search_console_ativo` (Flag) | |
| `mailbiz_ativo` (Flag) | |
| `ga4_ativo` (Flag) | |
| `rd_station_ativo` (Flag) | |
| `bling_ativo` (Flag) | |
| `atendente_ai__ativo` (Flag) | |
| `anymarket_ativo` (Flag) | |
| `facebook_anuncios_dinamicos_ativo` (Flag) | |
| `google_merchant_ativo` (Flag) | |
| `e_vendas_automacao_de_whatsapp_ativo` (Flag) | |
| `login_social_via_google_ativo` (Flag) | |
| `indexa_ai_ativo` (Flag) | |
| `google_adwords_ativo` (Flag) | |
| `api_de_conversoes_pixel_meta_ativo` (Flag) | |
| `recaptcha_ativo` (Flag) | |
| `sizebay_ativo` (Flag) | |
| `base_ativo` (Flag) | |
| `replay_ativo` (Flag) | |
| `magis5_ativo` (Flag) | |
| `olist_tiny_ativo` (Flag) | |
| `pluggto_ativo` (Flag) | |
| `go_fretes` (Flag) | |

**Justificativa:** ~28 flags consolidadas em 1 array. Facilita manutenção e extensibilidade. **Elimina ~27 campos.**

#### 2.4 Configurações de Abandono de Carrinho
| Campos Originais (3 campos) | Campo Otimizado |
|-----------------------------|-----------------|
| `abandono_carrinho_1h` (Flag) | `abandono_carrinho_intervalos` (Array) |
| `abandono_carrinho_6h` (Flag) | |
| `abandono_carrinho_24h` (Flag) | Exemplo: `["1h", "6h", "24h"]` |

**Justificativa:** 3 flags consolidadas em 1 array. **Elimina 2 campos.**

#### 2.5 Hubs Integradores
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `magis5_ativo` (Flag) | `hubs_integradores` (Array) |
| `olist_tiny_ativo` (Flag) | |
| `pluggto_ativo` (Flag) | Exemplo: `["magis5", "olist_tiny", "pluggto"]` |
| `anymarket_ativo` (Flag) | |

**Justificativa:** Hubs integradores são uma categoria específica. **Elimina 3 campos.**

---

## Tipo 3: Campos Redundantes ou Duplicados

### Otimizações Identificadas

#### 3.1 Tema da Loja
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `tema_gratis` (Flag) | `tipo_tema` (Texto) |
| `tema_agencia` (Flag) | Valores: `"gratis"`, `"agencia"`, `"pago"` |

**Justificativa:** São mutuamente exclusivos. **Elimina 1 campo.**

#### 3.2 Visitas Duplicadas
| Campos Originais | Observação |
|------------------|------------|
| `visitas_60d` (Flag - linha 169) | Campo marcado como Flag mas deveria ser Número |
| `visitas_60d` (Número - linha 183 como `qtde_visitas_60d`) | Duplicado |
| `visitas_90d` (linha 182 como `qtde_visitas_60d`) | Nome trocado |

**Justificativa:** Há confusão e duplicação. Manter apenas `visitas_30d`, `visitas_60d`, `visitas_90d` como Número. **Elimina 2 campos.**

#### 3.3 Plano Atual
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `plan_id_cloned_` (Texto) | `plano_atual` (Objeto ou campos separados) |
| `id_plano_atual` (Número) | |

**Justificativa:** `id_plano_atual` pode ser suficiente se houver lookup. Se precisar do nome, usar apenas `plano_atual` (Texto). **Elimina 1 campo.**

#### 3.4 Economia Enviali Duplicada
| Campos Originais | Observação |
|------------------|------------|
| `economia_enviali` (linha 158) | Duplicado |
| `economia_enviali` (linha 186) | Duplicado |

**Justificativa:** Campo duplicado. **Elimina 1 campo.**

#### 3.5 Data Última Etiqueta Duplicada
| Campos Originais | Observação |
|------------------|------------|
| `data_da_ultima_emissao_de_etiqueta_enviali` (linha 49) | Duplicado |
| `data_da_ultima_emissao_de_etiqueta_enviali` (linha 187) | Duplicado |

**Justificativa:** Campo duplicado. **Elimina 1 campo.**

#### 3.6 Pagali Cartão Duplicado
| Campos Originais | Observação |
|------------------|------------|
| `pagali_cartao` (linha 62) | Duplicado |
| `pagali_cartao` (linha 189) | Duplicado |

**Justificativa:** Campo duplicado. **Elimina 1 campo.**

---

## Tipo 4: Flags do Magalu para Array

### Otimização
| Campos Originais (7 campos) | Campo Otimizado |
|-----------------------------|-----------------|
| `data_evento_conectar_magalu` (Data) | `magalu_jornada` (Objeto) |
| `data_evento_tenhoconta_magalu` (Data) | |
| `data_evento_criarconta_magalu` (Data) | Estrutura: |
| `data_configuracao_magalu` (Data) | `{ conectou: Date, tem_conta: Date, criou_conta: Date, configurou: Date, vendeu: Date }` |
| `data_vendeu_magalu` (Data) | |
| `flag_token_magalu` (Flag) | Ou manter datas separadas + consolidar flags: |
| `flag_selecao_produtos_magalu` (Flag) | `magalu_etapas_concluidas` (Array) |
| `flag_publicacao_oferta_magalu` (Flag) | `["token", "selecao_produtos", "publicacao_oferta"]` |

**Justificativa:** Relacionados ao mesmo fluxo. **Elimina 2-3 campos.**

---

## Tipo 5: Vendas Enviali - Simplificação de Milestones

### Otimização
| Campos Originais (8 campos) | Campo Otimizado |
|-----------------------------|-----------------|
| `primeira_venda_enviali_data` (Data) | `vendas_enviali_milestones` (Objeto/JSON) |
| `primeira_venda_enviali_status` (Texto) | |
| `segunda_venda_enviali_data` (Data) | Estrutura: |
| `segunda_venda_enviali_status` (Texto) | ```json |
| `quinta_venda_enviali_data` (Data) | { |
| `quinta_venda_enviali_status` (Texto) |   "1": {"data": "2024-01-01", "status": "aprovado"}, |
| `decima_venda_enviali_data` (Data) |   "2": {"data": "2024-01-02", "status": "aprovado"}, |
| `decima_venda_enviali_status` (Texto) |   "5": {"data": "2024-01-05", "status": "aprovado"}, |
|  |   "10": {"data": "2024-01-10", "status": "aprovado"} |
|  | } |
|  | ``` |

**Alternativa mais simples:** Manter apenas `qtd_vendas_enviali` (Número) + `ultima_venda_enviali_status` (Texto) + `data_primeira_venda_enviali` (Data)

**Justificativa:** 8 campos podem ser reduzidos para 1-3. **Elimina 5-7 campos.**

---

## Tipo 6: Campos que podem usar presença de data

### 6.1 Newsletter e Avise-me
| Campos Originais | Observação |
|------------------|------------|
| `newsletter_ativa` (Data) | Já está como Data - OK |
| `aviseme_auto_ativo` (Data) | Já está como Data - OK |
| `frete_gratis_ativo` (Data) | Já está como Data - OK |

**Observação:** Estes já estão corretos, usando Data ao invés de Flag.

### 6.2 Google Shopping
| Campos Originais | Campo Otimizado |
|------------------|-----------------|
| `google_shopping_ativo` (Flag) | `data_criacao_google_shopping` (Data) |
| `data_criacao_google_shopping` (Data) | |
| `google_shopping_tem_campanha` (Data) | |

**Justificativa:** Se tem data de criação, está ativo. **Elimina 1 campo.**

---

## Resumo das Otimizações

| Categoria | Campos Removidos |
|-----------|------------------|
| Flags substituídas por Datas | 10 |
| Meios de Pagamento → Array | 19 |
| Meios de Envio → Array | 11 |
| Integrações → Array | 27 |
| Abandono Carrinho → Array | 2 |
| Hubs Integradores | 3 |
| Tipo Tema | 1 |
| Duplicados/Redundantes | 7 |
| Magalu Flags → Array | 3 |
| Vendas Enviali | 5 |
| Google Shopping | 1 |
| **Total** | **~89** |

---

## Lista Final de Propriedades Otimizadas

### Identificação e Configuração da Loja (14 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `id_loja` | Número | ID único da loja |
| `website` | Texto | URL do site (domínio ou subdomínio) |
| `subdomain` | Texto | Subdomínio .lojaintegrada.com.br |
| `status_da_loja` | Texto | Status atual da loja |
| `loja_em_manutencao` | Flag | Se a loja está em manutenção |
| `descricao_tier` | Texto | Descrição do tier |
| `cluster` | Texto | Clusterização da loja |
| `segmento_da_loja` | Texto | Segmento de atuação |
| `City` | Texto | Cidade da loja |
| `state` | Texto | Estado da loja |
| `tipo_de_pessoa` | Texto | PF ou PJ |
| `modelo_de_loja` | Texto | Como a loja será usada |
| `data_criacao_dl` | Data | Data de criação da loja |
| `ranking` | Número | Ranking da Loja Integrada |

### Plano e Assinatura (9 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `plano_atual` | Texto | Nome do plano atual |
| `id_plano_atual` | Número | ID do plano atual |
| `plano_anterior` | Texto | Plano anterior |
| `tipo_plano` | Texto | Ciclo (anual/mensal) |
| `inicio_de_ciclo` | Data | Início do ciclo atual |
| `fim_de_ciclo` | Data | Fim do ciclo atual |
| `movimento_plano` | Texto | upgrade/downgrade/churn/novo |
| `data_primeira_assinatura` | Data | Data da primeira assinatura |
| `forma_pagamento_assinatura_atual` | Texto | Forma de pagamento da assinatura |

### Atividade e Engajamento (6 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_ultimo_acesso` | Data | Último acesso ao painel |
| `data_primeira_venda` | Data | Data da primeira venda aprovada |
| `finalizou_wizard` | Flag | Se completou o wizard |
| `loja_reativada` | Flag | Se a loja foi reativada |
| `dominio_proprio` | Flag | Se configurou domínio próprio |
| `dominio_inativo` | Flag | Se o domínio está inativo |

### Pagamento (3 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_cadastro_pagamento` | Data | Data que configurou pagamento (presença = ativo) |
| `meios_pagamento_ativos` | Array | Lista de meios de pagamento ativos |
| `status_pagali` | Texto | Status do Pagali |

### Envio (3 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_cadastro_envio` | Data | Data que configurou envio (presença = ativo) |
| `meios_envio_ativos` | Array | Lista de meios de envio ativos |
| `usa_jadlog_enviali` | Flag | Usa Jadlog no Enviali |

### Enviali (10 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_configuracao_do_enviali` | Data | Data de ativação (presença = configurado) |
| `saldo_enviali` | Número | Saldo disponível |
| `data_da_primeira_etiqueta_enviali` | Data | Primeira emissão de etiqueta |
| `data_da_ultima_emissao_de_etiqueta_enviali` | Data | Última emissão de etiqueta |
| `etiquetas_disponiveis_enviali` | Número | Etiquetas disponíveis |
| `economia_enviali` | Número | Economia no último mês |
| `data_ultimo_pedido_enviali` | Data | Data do último pedido |
| `vendas_enviali_milestones` | JSON | Milestones de vendas (1ª, 2ª, 5ª, 10ª) |
| `flag_pac_contrato` | Flag | Se tem contrato PAC |

### Produtos (2 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_cadastro_produto` | Data | Data do primeiro produto |
| `produtos_ativos` | Número | Quantidade de produtos ativos |

### GMV e Métricas Financeiras (6 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `gmv_mes_atual` | Número | GMV do mês atual |
| `gmv_30d` | Número | GMV últimos 30 dias |
| `gmv_60_dias` | Número | GMV últimos 60 dias |
| `gmv_90_dias` | Número | GMV últimos 90 dias |
| `gmv_cartao_30d` | Número | GMV em cartão (30 dias) |
| `taxa_de_aprovacao_em_cartao_ultimo_mes_fechado` | Número | Taxa de aprovação em cartão |

### Métricas de Conversão e Visitas (7 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `taxa_de_conversao_ultimo_mes_fechado` | Número | Taxa de conversão |
| `taxa_de_conversao_do_segmento_da_loja_ultimo_mes_fechado` | Número | Taxa de conversão do segmento |
| `visitas_30d` | Número | Visitas últimos 30 dias |
| `visitas_60d` | Número | Visitas últimos 60 dias |
| `visitas_90d` | Número | Visitas últimos 90 dias |
| `qtde_pedido_30d` | Número | Pedidos últimos 30 dias |
| `qtde_pedido_60d` | Número | Pedidos últimos 60 dias |
| `qtde_pedido_90d` | Número | Pedidos últimos 90 dias |

### Funcionalidades da Loja (11 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `checkout_sem_senha` | Flag | Checkout sem senha ativo |
| `data_que_ativou_carrinho_abandonado` | Data | Data ativação (presença = ativo) |
| `abandono_carrinho_intervalos` | Array | Intervalos configurados (1h, 6h, 24h) |
| `newsletter_ativa` | Data | Data de ativação |
| `aviseme_auto_ativo` | Data | Data de ativação |
| `frete_gratis_ativo` | Data | Data de ativação |
| `data_que_ativou_o_primeiro_cupom` | Data | Data ativação cupom (presença = ativo) |
| `data_que_ativou_compre_junto` | Data | Data ativação (presença = ativo) |
| `data_que_ativou_brinde` | Data | Data de ativação brinde |
| `data_da_primeira_promocao` | Data | Data primeira promoção (presença = ativo) |
| `desconto_no_pix_ativo` | Flag | Desconto no Pix ativo |

### Personalização (4 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `tipo_tema` | Texto | gratis/agencia/pago |
| `data_instalacao_tema` | Data | Data de instalação do tema |
| `logo_da_loja` | Flag | Se tem logo |
| `banner` | Flag | Se tem banner |

### Integrações e Apps (2 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `integracoes_ativas` | Array | Lista de integrações ativas |
| `hubs_integradores` | Array | Lista de hubs (magis5, olist_tiny, etc) |

### Magalu (6 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_evento_conectar_magalu` | Data | Data do evento conectar |
| `data_evento_tenhoconta_magalu` | Data | Data tenho conta |
| `data_evento_criarconta_magalu` | Data | Data criar conta |
| `data_configuracao_magalu` | Data | Data de configuração |
| `data_vendeu_magalu` | Data | Data da primeira venda |
| `magalu_etapas_concluidas` | Array | [token, selecao_produtos, publicacao_oferta] |

### Google Shopping (2 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_criacao_google_shopping` | Data | Data de criação (presença = ativo) |
| `google_shopping_tem_campanha` | Data | Data da primeira campanha |

### Avaliação (1 campo)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_ativacao_avaliacao` | Data | Data de ativação (presença = ativo) |

### Abandono de Produto (1 campo)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_abandono_produto` | Data | Data de ativação (presença = ativo) |

### WhatsApp (3 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `data_que_cadastrou_botao_do_whatsapp` | Data | Data do cadastro |
| `recuperacao_whatsapp` | Flag | Usa recuperação por WhatsApp |
| `hs_whatsapp_phone_number` | Número | Número do WhatsApp |

### Dados do Contato (7 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `firstname` | Texto | Nome do contato |
| `email` | Texto | E-mail do contato |
| `phone` | Número | Telefone |
| `telefone_de_cobranca` | Número | Telefone de cobrança |
| `contato_responsavel_pela_conta` | Flag | Se é o responsável |
| `ultimo_login_painel` | Data | Último login do usuário |

### Origem e Aquisição (3 campos)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `origem_do_lead___aquisicao` | Texto | Origem do lead |
| `campanha_do_lead_aquisicao` | Texto | Campanha de aquisição |
| `midia_do_lead` | Texto | Mídia do lead |

### Outros (1 campo)
| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `politica_privacidade` | Flag | Política de privacidade configurada |

---

## Total de Campos Otimizados: ~89 campos

### Comparativo
- **Antes:** ~193 campos
- **Depois:** ~89 campos
- **Redução:** ~54%

---

## Considerações de Implementação

### Para CleverTap
1. **Arrays:** O CleverTap suporta arrays nativamente. Use operadores como `contains` para segmentação.
2. **Datas como Flags:** Use `exists` ou `is not null` para verificar presença de data.
3. **Migração:** Será necessário script de migração para converter dados existentes.

### Vantagens da Otimização
1. **Manutenibilidade:** Adicionar novo meio de pagamento é apenas incluir no array, sem criar novo campo.
2. **Consultas mais simples:** Segmentar por "tem pagali" é `meios_pagamento_ativos contains "pagali_pix"`.
3. **Menor custo:** Menos campos = menor custo de armazenamento e processamento.
4. **Consistência:** Elimina redundâncias e duplicações.

### Riscos
1. **Migração de dados:** Necessário planejar migração cuidadosa.
2. **Integrações existentes:** Verificar se há integrações que dependem dos campos antigos.
3. **Relatórios:** Atualizar relatórios que usam os campos removidos.

---

## Tipo 7: Propriedades Substituíveis por Eventos

### Princípio

No CleverTap, **propriedades de usuário** representam o **estado atual** do cliente (ex: "tem Pagali ativo"), enquanto **eventos** registram **ações que ocorreram** (ex: "ativou Pagali em 01/01/2024").

Para segmentação, muitas vezes uma propriedade de "data de primeira ocorrência" pode ser **derivada do primeiro evento** correspondente, usando segmentação por `Did Event > First Time`. Isso elimina a necessidade de armazenar a data como propriedade.

**Critérios para substituição:**
1. A propriedade representa uma data de "primeira vez" ou "ativação"
2. Existe um evento correspondente no tracking plan
3. A segmentação por "Did Event" atende às necessidades de campanha

**Quando NÃO substituir:**
- Propriedades que representam **estado atual** (ex: `saldo_enviali`, `produtos_ativos`)
- Propriedades de métricas agregadas (ex: `gmv_30d`, `visitas_90d`)
- Propriedades de configuração que podem mudar (ex: `plano_atual`, `meios_pagamento_ativos`)
- Dados históricos anteriores à implementação do tracking

### Otimizações Identificadas

#### 7.1 Data de Ativação do Enviali

| Propriedade | Evento Substituto | Segmentação CleverTap |
|-------------|-------------------|----------------------|
| `data_configuracao_do_enviali` | `Enviali Activated` | `Did Event "Enviali Activated" > First Time` |

**Racional:** O evento `Enviali Activated` é disparado quando o lojista ativa o Enviali. A data do primeiro evento equivale à `data_configuracao_do_enviali`.

**Segmentação equivalente:**
- Antes: `data_configuracao_do_enviali exists`
- Depois: `Did Event "Enviali Activated" at least 1 time`

**Elimina:** 1 propriedade

---

#### 7.2 Data da Primeira Etiqueta

| Propriedade | Evento Substituto | Segmentação CleverTap |
|-------------|-------------------|----------------------|
| `data_da_primeira_etiqueta_enviali` | `Label Issued` | `Did Event "Label Issued" > First Time` |

**Racional:** O evento `Label Issued` é disparado quando uma etiqueta é emitida. O primeiro evento com `is_first_purchase = true` ou simplesmente o primeiro `Label Issued` fornece a data.

**Segmentação equivalente:**
- Antes: `data_da_primeira_etiqueta_enviali exists`
- Depois: `Did Event "Label Issued" at least 1 time`

**Elimina:** 1 propriedade

---

#### 7.3 Data da Última Etiqueta

| Propriedade | Evento Substituto | Segmentação CleverTap |
|-------------|-------------------|----------------------|
| `data_da_ultima_emissao_de_etiqueta_enviali` | `Label Issued` | `Did Event "Label Issued" > Last Time` |

**Racional:** A data do último evento `Label Issued` equivale a `data_da_ultima_emissao_de_etiqueta_enviali`.

**Segmentação equivalente:**
- Antes: `data_da_ultima_emissao_de_etiqueta_enviali < 30 days ago`
- Depois: `Did Event "Label Issued" > Last Time < 30 days ago`

**Elimina:** 1 propriedade

---

#### 7.4 Data do Último Pedido Enviali

| Propriedade | Evento Substituto | Segmentação CleverTap |
|-------------|-------------------|----------------------|
| `data_ultimo_pedido_enviali` | `Order Shipped` | `Did Event "Order Shipped" > Last Time` |

**Racional:** O evento `Order Shipped` registra quando um pedido é enviado via Enviali. A data do último evento equivale à propriedade.

**Elimina:** 1 propriedade

---

#### 7.5 Data Primeira Assinatura

| Propriedade | Evento Substituto | Segmentação CleverTap |
|-------------|-------------------|----------------------|
| `data_primeira_assinatura` | `Subscription Completed` | `Did Event "Subscription Completed" > First Time` |

**Racional:** O evento `Subscription Completed` é disparado quando uma assinatura é concluída. O primeiro evento fornece a data da primeira assinatura.

**Segmentação equivalente:**
- Antes: `data_primeira_assinatura exists`
- Depois: `Did Event "Subscription Completed" at least 1 time`

**Elimina:** 1 propriedade

---

#### 7.6 Data Cadastro Primeiro Produto

| Propriedade | Evento Substituto | Segmentação CleverTap |
|-------------|-------------------|----------------------|
| `data_cadastro_produto` | `Product Created` | `Did Event "Product Created" > First Time` |

**Racional:** O evento `Product Created` é disparado quando um produto é criado. O primeiro evento fornece a data do primeiro produto.

**Segmentação equivalente:**
- Antes: `data_cadastro_produto exists`
- Depois: `Did Event "Product Created" at least 1 time`

**Elimina:** 1 propriedade

---

#### 7.7 Data Primeira Venda

| Propriedade | Evento Substituto | Segmentação CleverTap |
|-------------|-------------------|----------------------|
| `data_primeira_venda` | Evento de venda (a definir) | `Did Event "Sale Completed" > First Time` |

**Racional:** Se existir um evento de venda no tracking plan, a data da primeira venda pode ser derivada do primeiro evento.

**Nota:** Verificar se há evento de venda no tracking plan. Se não houver, manter a propriedade ou criar o evento.

**Elimina:** 1 propriedade (condicional)

---

#### 7.8 Datas da Jornada Magalu

| Propriedades | Eventos Substitutos |
|--------------|---------------------|
| `data_evento_conectar_magalu` | `ML Connection Started` |
| `data_configuracao_magalu` | `ML Initial Setup Completed` |
| `data_vendeu_magalu` | `ML Sale Completed` |

**Racional:** Todos esses eventos estão mapeados no tracking plan do BC11 (Mercado Livre). As datas podem ser derivadas dos eventos.

**Segmentação equivalente:**
- Antes: `data_evento_conectar_magalu exists`
- Depois: `Did Event "ML Connection Started" at least 1 time`

**Elimina:** 3 propriedades

---

#### 7.9 Data Publicação do Site (Komea)

| Propriedade | Evento Substituto | Segmentação CleverTap |
|-------------|-------------------|----------------------|
| `loja_em_manutencao` (inverso) | `Komea Site Published` | `Did Event "Komea Site Published" at least 1 time` |

**Racional:** O evento `Komea Site Published` indica que o site saiu do modo manutenção. A presença do evento indica que a loja foi publicada.

**Nota:** A propriedade `loja_em_manutencao` ainda pode ser útil para verificar o estado atual, mas para campanhas de "publicou o site", o evento é suficiente.

**Elimina:** 0 propriedades (manter para estado atual)

---

### Resumo: Propriedades Substituíveis por Eventos

| Propriedade | Evento | Redução |
|-------------|--------|---------|
| `data_configuracao_do_enviali` | `Enviali Activated` | 1 |
| `data_da_primeira_etiqueta_enviali` | `Label Issued` (first) | 1 |
| `data_da_ultima_emissao_de_etiqueta_enviali` | `Label Issued` (last) | 1 |
| `data_ultimo_pedido_enviali` | `Order Shipped` (last) | 1 |
| `data_primeira_assinatura` | `Subscription Completed` (first) | 1 |
| `data_cadastro_produto` | `Product Created` (first) | 1 |
| `data_evento_conectar_magalu` | `ML Connection Started` | 1 |
| `data_configuracao_magalu` | `ML Initial Setup Completed` | 1 |
| `data_vendeu_magalu` | `ML Sale Completed` (first) | 1 |
| **Total** | | **9 propriedades** |

---

### Propriedades que NÃO Devem Ser Substituídas

As seguintes propriedades **devem ser mantidas** mesmo existindo eventos relacionados:

| Propriedade | Motivo para Manter |
|-------------|-------------------|
| `saldo_enviali` | Estado atual, não derivável de eventos |
| `etiquetas_disponiveis_enviali` | Estado atual, não derivável de eventos |
| `economia_enviali` | Métrica agregada calculada |
| `produtos_ativos` | Estado atual (produtos podem ser desativados) |
| `plano_atual` | Estado atual que muda com o tempo |
| `meios_pagamento_ativos` | Estado atual (podem ser desativados) |
| `meios_envio_ativos` | Estado atual (podem ser desativados) |
| `gmv_30d`, `gmv_60_dias`, `gmv_90_dias` | Métricas agregadas de janela móvel |
| `visitas_30d`, `visitas_60d`, `visitas_90d` | Métricas agregadas de janela móvel |
| `qtde_pedido_30d`, `qtde_pedido_60d`, `qtde_pedido_90d` | Métricas agregadas de janela móvel |
| `status_pagali` | Estado atual do processo de aprovação |
| `loja_em_manutencao` | Estado atual que pode mudar |

**Princípio:** Propriedades de **estado atual** e **métricas agregadas** não podem ser substituídas por eventos, pois eventos registram ações pontuais e não refletem o estado corrente.

---

### Considerações para Implementação

#### Quando usar Evento vs Propriedade

| Necessidade | Usar |
|-------------|------|
| "Cliente já fez X alguma vez?" | Evento (`Did Event`) |
| "Quando o cliente fez X pela primeira vez?" | Evento (`First Time`) |
| "Quando o cliente fez X pela última vez?" | Evento (`Last Time`) |
| "Quantas vezes o cliente fez X?" | Evento (`Count`) |
| "O cliente TEM X ativo agora?" | Propriedade |
| "Quanto o cliente tem de saldo/GMV?" | Propriedade |
| "Qual o plano atual do cliente?" | Propriedade |

#### Limitações da Substituição por Eventos

1. **Dados históricos:** Eventos só existem após implementação. Dados anteriores precisam de propriedades.
2. **Retenção de eventos:** CleverTap tem limite de retenção de eventos. Propriedades são persistentes.
3. **Performance:** Segmentação por propriedade é mais rápida que por eventos em bases muito grandes.
4. **Complexidade:** Algumas segmentações ficam mais complexas com eventos (múltiplos filtros).

#### Recomendação

Para a migração, recomenda-se:

1. **Fase 1:** Implementar todos os eventos do tracking plan
2. **Fase 2:** Após 90 dias de coleta, começar a deprecar propriedades substituíveis
3. **Fase 3:** Manter propriedades apenas para dados históricos e estados atuais

---

## Sumário Executivo Atualizado

| Métrica | Valor |
|---------|-------|
| **Total de propriedades originais** | ~193 |
| **Redução por consolidação (Tipos 1-6)** | ~89 campos |
| **Redução adicional por eventos (Tipo 7)** | ~9 campos |
| **Propriedades finais estimadas** | ~80 |
| **Redução total estimada** | ~59% |
