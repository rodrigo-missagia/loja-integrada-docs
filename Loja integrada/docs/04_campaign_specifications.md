# Campaign Specifications - Loja Integrada

**Versão:** 2.0
**Data:** 12 de Março de 2026
**Última Atualização:** Tradução completa: eventos e propriedades de EN para PT
**Plataforma:** CleverTap

---

## Sumário

1. [Campanhas de Logística (BC1, BC2, BC3)](#1-campanhas-de-logística-bc1-bc2-bc3)
2. [Campanhas de Assinatura (BC4)](#2-campanhas-de-assinatura-bc4)
3. [Campanhas de Gateway de Pagamento (BC6)](#3-campanhas-de-gateway-de-pagamento-bc6)
4. [Campanhas de Produtos (BC5)](#4-campanhas-de-produtos-bc5)
5. [Campanhas da Komea (BC7, BC8, BC9)](#5-campanhas-da-komea-bc7-bc8-bc9)
6. [Campanhas de Canais - Marketplaces (BC11)](#6-campanhas-de-canais---marketplaces-bc11)

---

## 1. Campanhas de Logística (BC1, BC2, BC3)

### CAMP-001: Ativação do Enviali

**ID:** CAMP-001
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC1

#### Objetivo

**Goal:** Guiar lojistas que ainda não ativaram o Enviali para realizar a ativação inicial.
**KPI Primário:** Taxa de ativação do Enviali
**Meta:** >30% de conversão

#### Audiência

**Segmento:** Lojistas sem Enviali ativo
**Tamanho Estimado:** Variável (lojistas com `enviali_ativo = false` ou null)

**Critérios de Inclusão:**

- `enviali_ativo` != true
- Conta criada há mais de 24 horas
- Tem pelo menos 1 produto cadastrado OU está no plano pago

**Critérios de Exclusão:**

- Recebeu esta campanha < 7 dias
- Opt-out do canal
- Conta suspensa

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Event-Based + Scheduled

- Trigger inicial: 48h após `Store Created` se `enviali_ativo` = false
- Reminder: 7 dias após primeiro envio se ainda não ativou

#### Conteúdo

**Email:**

```
Subject: Ative o Enviali e economize até 70% no frete 📦
Preview: Configure sua loja para enviar pedidos com desconto

---

Olá {{profile.Name}},

Você sabia que pode economizar até 70% nas etiquetas dos Correios?

Com o Enviali, você:
✓ Emite etiquetas com desconto
✓ Tem acesso a múltiplas transportadoras
✓ Rastreia todos os envios em um só lugar

[Ativar Enviali agora]

---
```

**Push Notification:**

```
Título: Economize até 70% no frete! 📦
Corpo: Ative o Enviali e envie pedidos com desconto. Leva menos de 5 minutos.
CTA: Ativar agora
Deep Link: loja://enviali/activate
```

#### Conversão

**Evento de Conversão:** `Plataforma Envio Ativada`
**Janela de Atribuição:** 7 dias

#### Frequency Capping

- Máx 2 vezes por mês
- Cooling off: 7 dias após receber

---

### CAMP-002: Winback de Configuração Incompleta do Enviali

**ID:** CAMP-002
**Categoria:** Winback
**Prioridade:** Alta
**Business Case:** BC1

#### Objetivo

**Goal:** Recuperar lojistas que ativaram o Enviali mas não ativaram nenhuma transportadora.
**KPI Primário:** Taxa de ativação de transportadora
**Meta:** >25% de recuperação

#### Audiência

**Segmento:** Enviali ativado mas sem método de envio configurado
**Tipo de Segmentação:** Inaction (Plataforma Envio Ativada sem Metodo Envio Ativado)

**Critérios de Inclusão:**

- `enviali_ativo` = true
- `shipping_methods_count` = 0 ou null
- Inaction: Evento `Plataforma Envio Ativada` sem `Metodo Envio Ativado` em 24h

**Critérios de Exclusão:**

- Recebeu esta campanha < 5 dias
- Completou configuração (tem transportadora ativa)

#### Canal e Timing

**Canal Primário:** Push
**Canal Fallback:** Email

**Tipo:** Inaction-Based

- Trigger: 24h após `Plataforma Envio Ativada` sem `Metodo Envio Ativado`

#### Conteúdo

**Push Notification:**

```
Título: Falta pouco para usar o Enviali! ✨
Corpo: Ative uma transportadora e comece a economizar no frete.
CTA: Completar agora
Deep Link: loja://enviali/setup
```

**Email:**

```
Subject: Você está a um passo de economizar no frete
Preview: Complete a configuração do Enviali em 2 minutos

---

Olá {{profile.Name}},

Notamos que você ativou o Enviali, mas ainda não ativou uma transportadora.

Para começar a emitir etiquetas com desconto, só falta:
→ Ativar os Correios ou outra transportadora

Leva menos de 2 minutos!

[Ativar transportadora]

---
```

#### Conversão

**Evento de Conversão:** `Metodo Envio Ativado`
**Janela de Atribuição:** 48 horas

---

### CAMP-003: Incentivo à Ativação de Transportadoras

**ID:** CAMP-003
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC1

#### Objetivo

**Goal:** Incentivar lojistas com Enviali ativo mas sem transportadoras configuradas.
**KPI Primário:** Taxa de ativação de transportadora
**Meta:** >40% de conversão

#### Audiência

**Segmento:** Enviali ativo, sem transportadora
**Tipo de Segmentação:** Inaction (Plataforma Envio Ativada sem Metodo Envio Ativado)

**Critérios de Inclusão:**

- `enviali_ativo` = true
- `shipping_methods_count` = 0 ou null

**Critérios de Exclusão:**

- Recebeu esta campanha < 7 dias
- Já ativou transportadora

#### Canal e Timing

**Canal Primário:** In-App
**Canal Fallback:** Push

**Tipo:** Inaction-Based

- Trigger: 48h após `Plataforma Envio Ativada` sem `Metodo Envio Ativado`

#### Conteúdo

**In-App (Interstitial):**

```
Headline: Ative uma transportadora para começar a vender!
Body: Seus clientes precisam de opções de frete para finalizar compras.
CTA Primário: Ativar Correios
CTA Secundário: Ver todas as opções
```

**Push Notification:**

```
Título: Sua loja ainda não tem frete ativo 🚚
Corpo: Ative os Correios ou outra transportadora para seus clientes escolherem o envio.
CTA: Ativar transportadora
Deep Link: loja://enviali/carriers
```

#### Conversão

**Evento de Conversão:** `Metodo Envio Ativado`
**Janela de Atribuição:** 7 dias

---

### CAMP-004: Incentivo à Primeira Compra de Etiqueta

**ID:** CAMP-004
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC2

#### Objetivo

**Goal:** Converter lojistas com Enviali configurado que nunca compraram etiqueta.
**KPI Primário:** Taxa de primeira compra
**Meta:** >20% de conversão

#### Audiência

**Segmento:** Enviali configurado, sem etiquetas compradas

**Critérios de Inclusão:**

- `enviali_ativo` = true
- `shipping_methods_count` > 0
- `enviali_label_purchased` = false ou null
- Tem pelo menos 1 pedido nos últimos 30 dias

**Critérios de Exclusão:**

- Recebeu esta campanha < 7 dias
- Já comprou etiqueta

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Scheduled

- Frequência: Semanal para elegíveis
- Best Time: Sim

#### Conteúdo

**Email:**

```
Subject: Você tem pedidos para enviar? Economize no frete! 📦
Preview: Emita sua primeira etiqueta com desconto pelo Enviali

---

Olá {{profile.Name}},

Vimos que você tem o Enviali configurado, mas ainda não emitiu nenhuma etiqueta.

Que tal experimentar?

💰 Economize até 70% nas etiquetas dos Correios
🚀 Emissão rápida e prática
📍 Código de rastreio automático

[Emitir primeira etiqueta]

---
```

#### Conversão

**Evento de Conversão:** `Etiqueta Comprada`
**Janela de Atribuição:** 7 dias

---

### CAMP-006: Winback de Emissão de Etiqueta Abandonada

**ID:** CAMP-006
**Categoria:** Winback
**Prioridade:** Alta
**Business Case:** BC2

#### Objetivo

**Goal:** Recuperar lojistas que iniciaram o fluxo de emissão mas não compraram etiqueta.
**KPI Primário:** Taxa de recuperação
**Meta:** >20% de recuperação

#### Audiência

**Segmento:** Iniciou fluxo de emissão mas não comprou
**Tipo de Segmentação:** Inaction (Fluxo Etiqueta Iniciado sem Etiqueta Comprada)

**Critérios de Inclusão:**

- Inaction: Evento `Fluxo Etiqueta Iniciado` sem `Etiqueta Comprada` em 2h

**Critérios de Exclusão:**

- Já comprou a etiqueta
- Recebeu esta campanha < 3 dias

#### Canal e Timing

**Canal Primário:** Push
**Canal Fallback:** Email

**Tipo:** Inaction-Based

- Trigger: 2h após `Fluxo Etiqueta Iniciado` sem `Etiqueta Comprada`

#### Conteúdo

**Push Notification:**

```
Título: Você não terminou de emitir a etiqueta 📦
Corpo: Continue de onde parou e envie seu pedido com desconto.
CTA: Continuar
Deep Link: loja://enviali/labels
```

#### Conversão

**Evento de Conversão:** `Etiqueta Comprada`
**Janela de Atribuição:** 48 horas

---

### CAMP-007: Confirmação de Compra de Etiqueta

**ID:** CAMP-007
**Categoria:** Transactional
**Prioridade:** Alta
**Business Case:** BC2

#### Objetivo

**Goal:** Confirmar compra e orientar próximos passos.
**KPI Primário:** Taxa de abertura
**Meta:** >60% open rate

#### Audiência

**Segmento:** Comprou etiqueta

**Critérios de Inclusão:**

- Evento `Etiqueta Comprada` disparado

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Event-Based

- Trigger: Imediato após `Etiqueta Comprada`

#### Conteúdo

**Email:**

```
Subject: Etiqueta emitida com sucesso! 🎉
Preview: Próximos passos para enviar seu pedido

---

Olá {{profile.Name}},

Sua etiqueta foi emitida com sucesso!

📦 Pedido: #{{event.order_id}}
🚚 Transportadora: {{event.nome_transportadora}}
💰 Valor: R$ {{event.valor}}
📍 Rastreio: {{event.tracking_code}}

Próximos passos:
1. Imprima a etiqueta
2. Cole na embalagem
3. Poste na agência ou agende coleta

[Imprimir etiqueta] [Ver detalhes do pedido]

---
```

#### Conversão

**Evento de Conversão:** `Pedido Enviado`
**Janela de Atribuição:** 7 dias

---

### CAMP-008: Estímulo à Recorrência de Envios

**ID:** CAMP-008
**Categoria:** Retention
**Prioridade:** Média
**Business Case:** BC2

#### Objetivo

**Goal:** Incentivar lojistas a continuarem usando o Enviali.
**KPI Primário:** Taxa de recompra de etiquetas
**Meta:** >40% de recorrência

#### Audiência

**Segmento:** Comprou etiqueta há mais de 14 dias sem nova compra

**Critérios de Inclusão:**

- `enviali_label_purchased` = true
- `enviali_last_label_date` < 14 dias atrás
- Tem pedidos sem etiqueta emitida

**Critérios de Exclusão:**

- Comprou etiqueta nos últimos 14 dias
- Recebeu esta campanha < 14 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Scheduled

- Frequência: Quinzenal para elegíveis

#### Conteúdo

**Email:**

```
Subject: Sentimos sua falta no Enviali! 📦
Preview: Continue economizando no frete dos seus pedidos

---

Olá {{profile.Name}},

Já faz um tempo que você não usa o Enviali para enviar pedidos.

Lembre-se: você pode economizar até 70% nas etiquetas dos Correios!

Você já emitiu {{profile.enviali_labels_count}} etiquetas e economizou bastante.

[Emitir nova etiqueta]

---
```

#### Conversão

**Evento de Conversão:** `Etiqueta Comprada`
**Janela de Atribuição:** 14 dias

---

### CAMP-009: Incentivo à Ativação da Loggi

**ID:** CAMP-009
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC3

#### Objetivo

**Goal:** Incentivar lojistas com Enviali a ativarem a Loggi.
**KPI Primário:** Taxa de ativação Loggi
**Meta:** >25% de conversão

#### Audiência

**Segmento:** Enviali ativo, Loggi não ativada

**Critérios de Inclusão:**

- `enviali_ativo` = true
- Sem evento `Metodo Envio Ativado` com `nome_transportadora` = "Loggi"
- Localizado em região atendida pela Loggi

**Critérios de Exclusão:**

- Recebeu esta campanha < 14 dias
- Já ativou Loggi (`Metodo Envio Ativado` com `nome_transportadora` = "Loggi")

#### Canal e Timing

**Canal Primário:** In-App
**Canal Fallback:** Email

**Tipo:** Scheduled

- Frequência: Mensal para elegíveis

#### Conteúdo

**In-App (Banner):**

```
Headline: Nova opção de envio: Loggi! 🚀
Body: Entregas rápidas e econômicas via LoggiPonto.
CTA: Ativar Loggi
```

**Email:**

```
Subject: Conheça a Loggi: entregas urbanas rápidas e econômicas
Preview: Nova transportadora disponível no seu Enviali

---

Olá {{profile.Name}},

Temos uma novidade para você: a Loggi agora está disponível no Enviali!

Com a Loggi você tem:
🚀 Entregas rápidas em áreas urbanas
💰 Preços competitivos
📍 Pontos de coleta convenientes

[Ativar Loggi agora]

---
```

#### Conversão

**Evento de Conversão:** `Metodo Envio Ativado` com `nome_transportadora` = "Loggi"
**Janela de Atribuição:** 14 dias

---

### CAMP-010: Incentivo à Primeira Etiqueta Loggi

**ID:** CAMP-010
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC3

#### Objetivo

**Goal:** Converter lojistas com Loggi ativada para primeira etiqueta.
**KPI Primário:** Taxa de primeira compra Loggi
**Meta:** >30% de conversão

#### Audiência

**Segmento:** Loggi ativada, sem etiqueta Loggi
**Tipo de Segmentação:** Inaction (Metodo Envio Ativado com nome_transportadora=Loggi sem Etiqueta Comprada com nome_transportadora=Loggi)

**Critérios de Inclusão:**

- `Metodo Envio Ativado` com `nome_transportadora` = "Loggi" disparado
- Sem `Etiqueta Comprada` com `nome_transportadora` = "Loggi"

**Critérios de Exclusão:**

- Recebeu esta campanha < 7 dias
- Já comprou etiqueta Loggi

#### Canal e Timing

**Canal Primário:** Push
**Canal Fallback:** Email

**Tipo:** Inaction-Based

- Trigger: 72h após `Metodo Envio Ativado` (nome_transportadora=Loggi) sem `Etiqueta Comprada` (nome_transportadora=Loggi)

#### Conteúdo

**Push Notification:**

```
Título: Que tal testar a Loggi? 🚀
Corpo: Você ativou a Loggi mas ainda não enviou. Experimente em seu próximo pedido!
CTA: Enviar via Loggi
Deep Link: loja://enviali/labels
```

#### Conversão

**Evento de Conversão:** `Etiqueta Comprada` com `nome_transportadora` = "Loggi"
**Janela de Atribuição:** 7 dias

---

## 2. Campanhas de Assinatura (BC4)

### CAMP-011: Recuperação de Abandono de Checkout

**ID:** CAMP-011
**Categoria:** Winback
**Prioridade:** Alta
**Business Case:** BC4

#### Objetivo

**Goal:** Recuperar lojistas que iniciaram checkout de assinatura mas não completaram.
**KPI Primário:** Taxa de recuperação
**Meta:** >15% de conversão

#### Audiência

**Segmento:** Iniciou checkout mas não completou assinatura
**Tipo de Segmentação:** Inaction (Checkout Iniciado sem Assinatura Concluida)

**Critérios de Inclusão:**

- Inaction: Evento `Checkout Iniciado` sem `Assinatura Concluida` em 1h

**Critérios de Exclusão:**

- Já completou assinatura
- Recebeu esta campanha < 3 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Inaction-Based

- Trigger: 1h após `Checkout Iniciado` sem `Assinatura Concluida`
- Reminder: 24h se não converteu

#### Conteúdo

**Variante A - Email (Urgência):**

```
Subject: Você estava quase lá! Complete sua assinatura 🎯
Preview: Seu carrinho com o plano {{event.nome_plano}} está esperando

---

Olá {{profile.Name}},

Notamos que você começou a assinar o plano {{event.nome_plano}} mas não finalizou.

Não perca os benefícios:
✓ [Lista de benefícios do plano]

{{if event.codigo_cupom}}
E seu cupom {{event.codigo_cupom}} ainda está ativo!
{{/if}}

[Continuar assinatura]

---
```

**Variante B - Email (Benefício):**

```
Subject: O plano {{event.nome_plano}} está te esperando ✨
Preview: Desbloqueie todas as funcionalidades da Loja Integrada

---

Olá {{profile.Name}},

Você selecionou o plano {{event.nome_plano}} - uma ótima escolha!

Com ele você terá acesso a:
✓ [Benefícios específicos do plano]

Complete sua assinatura e comece a crescer.

[Finalizar assinatura]

---
```

#### A/B Test

**Divisão:** 50/50
**Métrica de Sucesso:** Conversion Rate
**Hipótese:** Copy focado em benefícios converte mais que urgência

#### Conversão

**Evento de Conversão:** `Assinatura Concluida`
**Janela de Atribuição:** 7 dias

---

### CAMP-012: Migração de Meio de Pagamento (Boleto → Cartão)

**ID:** CAMP-012
**Categoria:** Engagement
**Prioridade:** Média
**Business Case:** BC4

#### Objetivo

**Goal:** Incentivar lojistas que pagam por boleto a migrarem para cartão.
**KPI Primário:** Taxa de migração
**Meta:** >10% de conversão

#### Audiência

**Segmento:** Assinantes que pagam por boleto

**Critérios de Inclusão:**

- `cliente_pagante` = true
- Último `meio_pagamento` em `Assinatura Concluida` = "boleto"
- Assinatura ativa

**Critérios de Exclusão:**

- Recebeu esta campanha < 30 dias
- Já migrou para cartão

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** In-App

**Tipo:** Scheduled

- Frequência: Mensal

#### Conteúdo

**Email:**

```
Subject: Simplifique sua assinatura: pague com cartão 💳
Preview: Ativação imediata e sem preocupação com vencimento

---

Olá {{profile.Name}},

Que tal simplificar o pagamento da sua assinatura?

Vantagens do pagamento por cartão:
✓ Ativação imediata (sem esperar compensação)
✓ Renovação automática (nunca perca acesso)
✓ Mais segurança e praticidade

[Atualizar forma de pagamento]

---
```

#### Conversão

**Evento de Conversão:** `Assinatura Concluida` com `meio_pagamento` = "credit_card"
**Janela de Atribuição:** 30 dias

---

### CAMP-013: Confirmação de Assinatura

**ID:** CAMP-013
**Categoria:** Transactional
**Prioridade:** Alta
**Business Case:** BC4

#### Objetivo

**Goal:** Confirmar assinatura e reforçar valor.
**KPI Primário:** Taxa de abertura
**Meta:** >70% open rate

#### Audiência

**Segmento:** Completou assinatura

**Critérios de Inclusão:**

- Evento `Assinatura Concluida` disparado

#### Canal e Timing

**Canal Primário:** Email

**Tipo:** Event-Based

- Trigger: Imediato após `Assinatura Concluida`

#### Conteúdo

**Email:**

```
Subject: Bem-vindo ao plano {{event.nome_plano}}! 🎉
Preview: Sua assinatura foi confirmada

---

Olá {{profile.Name}},

Parabéns! Sua assinatura do plano {{event.nome_plano}} foi confirmada.

📋 Detalhes da assinatura:
• Plano: {{event.nome_plano}}
• Ciclo: {{event.ciclo_cobranca}}
• Valor: R$ {{event.valor}}

Agora você tem acesso a todas as funcionalidades:
✓ [Lista de benefícios]

Próximos passos recomendados:
1. Configure seu meio de pagamento
2. Cadastre seus produtos
3. Ative o Enviali para economizar no frete

[Acessar painel]

---
```

#### Conversão

**Evento de Conversão:** `Cadastro Gateway Iniciado` ou `Produto Criado`
**Janela de Atribuição:** 7 dias

---

### CAMP-014: Meta de Assinatura (Celebração)

**ID:** CAMP-014
**Categoria:** Retention
**Prioridade:** Baixa
**Business Case:** BC4

#### Objetivo

**Goal:** Celebrar marcos de assinatura e reforçar relacionamento.
**KPI Primário:** Engagement rate
**Meta:** >40% open rate

#### Audiência

**Segmento:** Aniversário de assinatura (1 ano)

**Critérios de Inclusão:**

- `first_paid_plan_date` = exatamente 1 ano atrás
- `cliente_pagante` = true

#### Canal e Timing

**Canal Primário:** Email

**Tipo:** Scheduled

- Frequência: Anual (data específica)

#### Conteúdo

**Email:**

```
Subject: 1 ano juntos! Obrigado por confiar na Loja Integrada 🎂
Preview: Celebrando sua jornada de sucesso

---

Olá {{profile.Name}},

Hoje faz 1 ano que você é assinante da Loja Integrada!

Nesse tempo, sua loja cresceu muito:
• {{profile.qtd_produtos}} produtos cadastrados
• {{profile.enviali_labels_count}} etiquetas emitidas
• E muito mais...

Obrigado por fazer parte da nossa comunidade!

Como presente, aqui vai um cupom especial: [ANIVERSARIO20]
20% de desconto na próxima renovação.

[Continuar crescendo]

---
```

---

## 3. Campanhas de Gateway de Pagamento (BC6)

### CAMP-015: Winback de Cadastro de Gateway Abandonado

**ID:** CAMP-015
**Categoria:** Winback
**Prioridade:** Alta
**Business Case:** BC6

#### Objetivo

**Goal:** Recuperar lojistas que iniciaram cadastro do gateway de pagamento mas não completaram.
**KPI Primário:** Taxa de conclusão
**Meta:** >25% de recuperação

#### Audiência

**Segmento:** Iniciou cadastro do gateway mas não completou
**Tipo de Segmentação:** Inaction (Cadastro Gateway Iniciado sem Cadastro Gateway Concluido)

**Critérios de Inclusão:**

- Inaction: Evento `Cadastro Gateway Iniciado` sem `Cadastro Gateway Concluido` em 24h

**Critérios de Exclusão:**

- Completou cadastro (`Cadastro Gateway Concluido` disparado)
- Recebeu esta campanha < 5 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Inaction-Based

- Trigger: 24h após `Cadastro Gateway Iniciado` sem `Cadastro Gateway Concluido`

#### Conteúdo

**Email:**

```
Subject: Complete seu cadastro de pagamento e comece a vender
Preview: Faltam apenas alguns passos para receber pagamentos

---

Olá {{profile.Name}},

Você começou a configurar seu meio de pagamento mas não finalizou.

Sem um meio de pagamento, seus clientes não conseguem comprar na sua loja.

O cadastro é simples e você pode receber via:
✓ Pix (pagamento instantâneo)
✓ Cartão de crédito
✓ Boleto bancário

{{if profile.pagali_abandoned_step == "documents"}}
Dica: Tenha em mãos seus documentos para agilizar o processo.
{{/if}}

[Continuar cadastro]

---
```

#### Conversão

**Evento de Conversão:** `Cadastro Gateway Concluido`
**Janela de Atribuição:** 7 dias

---

### CAMP-016: Explicação da Importância dos Dados

**ID:** CAMP-016
**Categoria:** Engagement
**Prioridade:** Média
**Business Case:** BC6

#### Objetivo

**Goal:** Educar lojistas sobre a importância de completar dados do gateway de pagamento.
**KPI Primário:** Taxa de conclusão de etapa
**Meta:** >30% de avanço

#### Audiência

**Segmento:** Parou em etapa específica do cadastro

**Critérios de Inclusão:**

- `pagali_registration_step` = etapa específica
- Tempo na etapa > 48h

**Critérios de Exclusão:**

- Avançou para próxima etapa
- Recebeu esta campanha < 7 dias

#### Canal e Timing

**Canal Primário:** Email

**Tipo:** Event-Based

- Trigger: 48h após parar em etapa

#### Conteúdo

**Email (para etapa "income_info"):**

```
Subject: Por que pedimos informações de renda?
Preview: Transparência sobre o processo de verificação do gateway de pagamento

---

Olá {{profile.Name}},

Notamos que você parou no preenchimento de informações financeiras do gateway de pagamento.

Por que precisamos desses dados?
→ Segurança antifraude para proteger você e seus clientes
→ Conformidade com regulamentações financeiras
→ Liberação de limites adequados ao seu negócio

Seus dados são protegidos e usados apenas para verificação.

[Continuar cadastro]

---
```

#### Conversão

**Tipo de Conversão:** Segmentação Inaction
**Critério:** Evento `Cadastro Gateway Iniciado` seguido por `Cadastro Gateway Concluido` (próxima etapa concluída)
**Janela de Atribuição:** 7 dias

> **Nota:** O evento `Pagali Registration Step Completed` foi removido. A conversão é medida via segmentação Inaction, verificando se o usuário avançou no cadastro (evento `Cadastro Gateway Concluido` ou saída do status "pending").

---

### CAMP-017: Orientação para Gateway Alternativo (Fallback)

**ID:** CAMP-017
**Categoria:** Engagement
**Prioridade:** Alta
**Business Case:** BC6

#### Objetivo

**Goal:** Guiar lojistas com gateway rejeitado para alternativa.
**KPI Primário:** Taxa de configuração gateway alternativo
**Meta:** >40% de conversão

#### Audiência

**Segmento:** Rejeitados no gateway de pagamento

**Critérios de Inclusão:**

- `pagali_account_status` = "rejected"
- `mercado_pago_configured` = false

**Critérios de Exclusão:**

- Já configurou gateway alternativo
- Recebeu esta campanha < 7 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Event-Based

- Trigger: 2h após `Conta Gateway Rejeitada`

#### Conteúdo

**Email:**

```
Subject: Alternativa de pagamento disponível
Preview: Configure um meio de pagamento alternativo e comece a vender hoje

---

Olá {{profile.Name}},

Infelizmente não foi possível aprovar sua conta no gateway de pagamento neste momento.

Mas não se preocupe! Você pode configurar um meio de pagamento alternativo.

Com outros gateways você também recebe via:
✓ Pix
✓ Cartão de crédito
✓ Boleto

A configuração é simples e rápida.

[Configurar meio de pagamento alternativo]

---

Quer tentar novamente? Entre em contato com nosso suporte.

---
```

#### Conversão

**Evento de Conversão:** `Cadastro Gateway Concluido` com `gateway_pagamento` diferente do gateway rejeitado
**Janela de Atribuição:** 7 dias

---

## 4. Campanhas de Produtos (BC5)

### CAMP-018: Régua de Abandono de Cadastro de Produto

**ID:** CAMP-018
**Categoria:** Winback
**Prioridade:** Alta
**Business Case:** BC5

#### Objetivo

**Goal:** Recuperar lojistas que iniciaram criação de produto mas não completaram.
**KPI Primário:** Taxa de conclusão
**Meta:** >20% de recuperação

#### Audiência

**Segmento:** Iniciou criação de produto mas não completou
**Tipo de Segmentação:** Inaction (Criacao Produto Iniciada sem Produto Criado)

**Critérios de Inclusão:**

- Inaction: Evento `Criacao Produto Iniciada` sem `Produto Criado` em 2h

**Critérios de Exclusão:**

- Criou produto (`Produto Criado` disparado)
- Recebeu esta campanha < 3 dias

#### Canal e Timing

**Canal Primário:** Push
**Canal Fallback:** Email

**Tipo:** Inaction-Based

- Trigger: 2h após `Criacao Produto Iniciada` sem `Produto Criado`

#### Conteúdo

**Push Notification:**

```
Título: Não desista! Seu produto está quase pronto 📦
Corpo: Continue de onde parou e cadastre seu primeiro produto.
CTA: Continuar
Deep Link: loja://products/new
```

**Email:**

```
Subject: Seu produto está esperando para ser cadastrado
Preview: Continue de onde parou - leva só alguns minutos

---

Olá {{profile.Name}},

Notamos que você começou a cadastrar um produto mas não finalizou.

Dicas para um cadastro de sucesso:
✓ Título claro e descritivo
✓ Fotos de qualidade
✓ Preço competitivo
✓ Descrição completa

{{if profile.product_abandoned_step == "images"}}
Dica: Fotos de boa qualidade aumentam muito as vendas!
{{/if}}

[Continuar cadastro]

---
```

#### Conversão

**Evento de Conversão:** `Produto Criado`
**Janela de Atribuição:** 7 dias

---

### CAMP-019: Régua Educacional de Qualidade

**ID:** CAMP-019
**Categoria:** Engagement
**Prioridade:** Média
**Business Case:** BC5

#### Objetivo

**Goal:** Educar lojistas sobre cadastro de produtos de qualidade.
**KPI Primário:** Melhoria na qualidade dos produtos
**Meta:** >30% de engajamento

#### Audiência

**Segmento:** Criou primeiro produto

**Critérios de Inclusão:**

- Evento `Produto Criado` com `is_first_product` = true
- `qtd_produtos` = 1

**Critérios de Exclusão:**

- Recebeu esta campanha

#### Canal e Timing

**Canal Primário:** Email

**Tipo:** Event-Based (Série de 3 emails)

- Email 1: 24h após primeiro produto
- Email 2: 3 dias após
- Email 3: 7 dias após

#### Conteúdo

**Email 1 - Fotos:**

```
Subject: Dica #1: Fotos que vendem 📸
Preview: Como tirar fotos que aumentam suas vendas

---

Olá {{profile.Name}},

Parabéns pelo seu primeiro produto!

Agora vamos melhorar suas vendas com fotos incríveis:

📸 Dicas de fotografia:
• Use luz natural sempre que possível
• Fundo branco ou neutro
• Múltiplos ângulos do produto
• Mostre o produto em uso

[Ver exemplos de boas fotos]

---
```

**Email 2 - Descrição:**

```
Subject: Dica #2: Descrições que convertem ✍️
Preview: Escreva descrições que seus clientes querem ler

---

Olá {{profile.Name}},

Hoje vamos falar sobre descrições de produto.

Uma boa descrição deve:
• Destacar benefícios, não só características
• Responder dúvidas comuns
• Usar palavras-chave relevantes
• Ser fácil de escanear (use bullets)

[Ver exemplos de boas descrições]

---
```

#### Conversão

**Evento de Conversão:** `Produto Criado` (segundo produto ou edição)
**Janela de Atribuição:** 14 dias

---

## 5. Campanhas da Komea (BC7, BC8, BC9)

### CAMP-020: Régua de Engajamento para Personalização

**ID:** CAMP-020
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC7

#### Objetivo

**Goal:** Incentivar conclusão da personalização da vitrine.
**KPI Primário:** Taxa de conclusão
**Meta:** >35% de conversão

#### Audiência

**Segmento:** Acessou Komea mas não completou personalização
**Tipo de Segmentação:** Inaction (Komea Acessada sem Komea Personalizacao Concluida)

**Critérios de Inclusão:**

- Inaction: Evento `Komea Acessada` sem `Komea Personalizacao Concluida` em 24h

**Critérios de Exclusão:**

- Completou personalização (`Komea Personalizacao Concluida` disparado)
- Recebeu esta campanha < 5 dias

#### Canal e Timing

**Canal Primário:** Push
**Canal Fallback:** Email

**Tipo:** Inaction-Based

- Trigger: 24h após `Komea Acessada` sem `Komea Personalizacao Concluida`

#### Conteúdo

**Push Notification:**

```
Título: Sua loja precisa de uma cara! ✨
Corpo: Complete a personalização do logo e cores da sua vitrine.
CTA: Personalizar agora
Deep Link: loja://komea/customize
```

#### Conversão

**Evento de Conversão:** `Komea Personalizacao Concluida`
**Janela de Atribuição:** 7 dias

---

### CAMP-021: Régua de Engajamento para Publicação

**ID:** CAMP-021
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC8

#### Objetivo

**Goal:** Incentivar publicação do site (saída do modo manutenção).
**KPI Primário:** Taxa de publicação
**Meta:** >40% de conversão

#### Audiência

**Segmento:** Site em manutenção

**Critérios de Inclusão:**

- `site_publicado` = false
- `tem_produtos` = true (pelo menos 1 produto)
- Conta criada há mais de 48h

**Critérios de Exclusão:**

- Site já publicado
- Recebeu esta campanha < 7 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** In-App

**Tipo:** Scheduled

- Frequência: Semanal para elegíveis

#### Conteúdo

**Email:**

```
Subject: Sua loja está pronta para sair do ar? 🚀
Preview: Publique seu site e comece a vender

---

Olá {{profile.Name}},

Sua loja tem {{profile.qtd_produtos}} produto(s) cadastrado(s), mas ainda está em modo de manutenção.

Isso significa que ninguém consegue acessar sua loja!

Para publicar, você precisa:
{{if profile.pagali_account_status != "approved"}}
☐ Configurar um meio de pagamento
{{else}}
☑ Meio de pagamento configurado
{{/if}}
☐ Preencher dados básicos (CPF/CNPJ e endereço)

[Publicar minha loja]

---
```

**In-App (Interstitial):**

```
Headline: Sua loja está em manutenção!
Body: Publique agora e comece a receber clientes.
CTA Primário: Publicar site
CTA Secundário: Depois
```

#### Conversão

**Evento de Conversão:** `Komea Site Publicado`
**Janela de Atribuição:** 7 dias

---

### CAMP-022: Descoberta do Potencial da Komea

**ID:** CAMP-022
**Categoria:** Engagement
**Prioridade:** Média
**Business Case:** BC9

#### Objetivo

**Goal:** Engajar lojistas da base a descobrirem a Komea.
**KPI Primário:** Taxa de primeiro acesso
**Meta:** >20% de engajamento

#### Audiência

**Segmento:** Lojistas ativos que nunca usaram Komea

**Critérios de Inclusão:**

- `komea_access_count` = 0 ou null
- Conta criada há mais de 30 dias
- `cliente_pagante` = true OU teve vendas nos últimos 30 dias

**Critérios de Exclusão:**

- Já acessou Komea
- Recebeu esta campanha < 30 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Scheduled

- Frequência: Mensal para elegíveis

#### Conteúdo

**Email:**

```
Subject: Conheça a Komea: sua assistente de IA 🤖
Preview: Automatize tarefas e encontre oportunidades para sua loja

---

Olá {{profile.Name}},

Você conhece a Komea?

É a assistente de IA da Loja Integrada que te ajuda a:
🎯 Encontrar oportunidades de vendas
📊 Analisar dados da sua loja
✨ Criar promoções automaticamente
🛒 Recuperar carrinhos abandonados

Tudo isso com comandos simples, como se estivesse conversando.

[Conhecer a Komea]

---
```

#### Conversão

**Evento de Conversão:** `Komea Acessada`
**Janela de Atribuição:** 14 dias

---

### CAMP-023: Engajamento para Uso Recorrente da Komea

**ID:** CAMP-023
**Categoria:** Retention
**Prioridade:** Média
**Business Case:** BC9

#### Objetivo

**Goal:** Manter lojistas engajados com uso da Komea.
**KPI Primário:** Taxa de retenção semanal
**Meta:** >30% de uso recorrente

#### Audiência

**Segmento:** Usou Komea mas não voltou em 14 dias

**Critérios de Inclusão:**

- `komea_access_count` > 0
- `komea_last_access_date` < 14 dias atrás

**Critérios de Exclusão:**

- Acessou Komea nos últimos 14 dias
- Recebeu esta campanha < 14 dias

#### Canal e Timing

**Canal Primário:** Push
**Canal Fallback:** Email

**Tipo:** Scheduled

- Frequência: Quinzenal para elegíveis

#### Conteúdo

**Push Notification:**

```
Título: A Komea tem novidades para você! ✨
Corpo: Veja as oportunidades identificadas para sua loja.
CTA: Ver oportunidades
Deep Link: loja://komea/opportunities
```

#### Conversão

**Evento de Conversão:** `Komea Acessada`
**Janela de Atribuição:** 14 dias

---

## 6. Campanhas de Canais - Marketplaces (BC11)

### CAMP-024: Jornada de Ativação de Marketplace

**ID:** CAMP-024
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC11

#### Objetivo

**Goal:** Guiar conclusão de todas as etapas de configuração do marketplace.
**KPI Primário:** Taxa de conclusão
**Meta:** >35% de conversão

#### Audiência

**Segmento:** Conectou marketplace mas não completou configuração inicial
**Tipo de Segmentação:** Inaction (Marketplace Conectado sem Marketplace Config Inicial Concluida) com filtro `marketplace = "mercado_livre"`

**Critérios de Inclusão:**

- Inaction: Evento `Marketplace Conectado` com `marketplace = "mercado_livre"` sem `Marketplace Config Inicial Concluida` com `marketplace = "mercado_livre"` em 48h

**Critérios de Exclusão:**

- Completou configuração (`Marketplace Config Inicial Concluida` com `marketplace = "mercado_livre"` disparado)
- Recebeu esta campanha < 5 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Inaction-Based

- Trigger: 48h após `Marketplace Conectado` (marketplace = "mercado_livre") sem `Marketplace Config Inicial Concluida`

#### Conteúdo

**Email:**

```
Subject: Falta pouco para vender no Mercado Livre! 🛒
Preview: Complete a configuração e envie seus primeiros anúncios

---

Olá {{profile.Name}},

Você conectou sua conta do Mercado Livre, mas ainda não completou a configuração.

Próximos passos:
1. Definir estoque mínimo
2. Configurar percentuais por tipo de anúncio
3. Enviar seus primeiros produtos

O Mercado Livre é o maior marketplace da América Latina. Seus produtos podem alcançar milhões de compradores!

[Completar configuração]

---
```

#### Conversão

**Evento de Conversão:** `Marketplace Config Inicial Concluida` com `marketplace = "mercado_livre"`
**Janela de Atribuição:** 7 dias

---

### CAMP-025: Incentivo à Adesão do Canal Marketplace

**ID:** CAMP-025
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC11

#### Objetivo

**Goal:** Incentivar lojistas pagos a conectarem o Mercado Livre.
**KPI Primário:** Taxa de conexão
**Meta:** >15% de conversão

#### Audiência

**Segmento:** Lojistas pagos sem marketplace conectado
**Filtro:** `marketplace = "mercado_livre"`

**Critérios de Inclusão:**

- `cliente_pagante` = true
- Sem evento `Marketplace Conectado` com `marketplace = "mercado_livre"` no histórico
- `tem_produtos` = true
- `qtd_produtos` >= 5

**Critérios de Exclusão:**

- Já conectou ML (`Marketplace Conectado` com `marketplace = "mercado_livre"` disparado)
- Recebeu esta campanha < 30 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** In-App

**Tipo:** Scheduled

- Frequência: Mensal para elegíveis

#### Conteúdo

**Email:**

```
Subject: Venda no maior marketplace da América Latina 🌎
Preview: Conecte o Mercado Livre e alcance milhões de compradores

---

Olá {{profile.Name}},

Você tem {{profile.qtd_produtos}} produtos na sua loja. Que tal vendê-los também no Mercado Livre?

Por que vender no Mercado Livre?
🌎 Maior marketplace da América Latina
👥 Milhões de compradores ativos
💰 Aumente suas vendas sem esforço extra
📦 Gestão centralizada pelo Hub de Canais

A integração é simples e seus produtos são sincronizados automaticamente.

[Conectar Mercado Livre]

---
```

#### Conversão

**Evento de Conversão:** `Marketplace Conectado` com `marketplace = "mercado_livre"`
**Janela de Atribuição:** 14 dias

---

### CAMP-026: Incentivo ao Primeiro Anúncio Marketplace

**ID:** CAMP-026
**Categoria:** Activation
**Prioridade:** Alta
**Business Case:** BC11

#### Objetivo

**Goal:** Incentivar envio do primeiro anúncio ao marketplace.
**KPI Primário:** Taxa de primeiro anúncio
**Meta:** >40% de conversão

#### Audiência

**Segmento:** Marketplace configurado, sem anúncios publicados
**Tipo de Segmentação:** Inaction (Marketplace Config Inicial Concluida sem Marketplace Anuncio Publicado) com filtro `marketplace = "mercado_livre"`

**Critérios de Inclusão:**

- Inaction: Evento `Marketplace Config Inicial Concluida` com `marketplace = "mercado_livre"` sem `Marketplace Anuncio Publicado` com `marketplace = "mercado_livre"` em 72h

**Critérios de Exclusão:**

- Publicou anúncios (`Marketplace Anuncio Publicado` com `marketplace = "mercado_livre"` disparado)
- Recebeu esta campanha < 7 dias

#### Canal e Timing

**Canal Primário:** Push
**Canal Fallback:** Email

**Tipo:** Inaction-Based

- Trigger: 72h após `Marketplace Config Inicial Concluida` (marketplace = "mercado_livre") sem `Marketplace Anuncio Publicado`

#### Conteúdo

**Push Notification:**

```
Título: Seu Mercado Livre está esperando anúncios! 📦
Corpo: Envie seu primeiro produto e comece a vender para milhões.
CTA: Enviar anúncio
Deep Link: loja://hub/mercadolivre/ads
```

#### Conversão

**Evento de Conversão:** `Marketplace Anuncio Publicado` com `marketplace = "mercado_livre"`
**Janela de Atribuição:** 7 dias

---

### CAMP-027: Expansão de Catálogo Marketplace

**ID:** CAMP-027
**Categoria:** Engagement
**Prioridade:** Média
**Business Case:** BC11

#### Objetivo

**Goal:** Incentivar envio de mais produtos ao marketplace.
**KPI Primário:** Aumento de anúncios
**Meta:** +50% de anúncios em 30 dias

#### Audiência

**Segmento:** Poucos produtos anunciados no ML
**Filtro:** `marketplace = "mercado_livre"`

**Critérios de Inclusão:**

- `ml_total_ads_count` > 0 e < 10
- `qtd_produtos` > `ml_products_sent` + 5

**Critérios de Exclusão:**

- Recebeu esta campanha < 14 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Scheduled

- Frequência: Quinzenal para elegíveis

#### Conteúdo

**Email:**

```
Subject: Você tem mais produtos para vender no Mercado Livre!
Preview: Aumente suas vendas enviando mais anúncios

---

Olá {{profile.Name}},

Você tem {{profile.ml_total_ads_count}} produtos no Mercado Livre, mas {{profile.qtd_produtos}} na sua loja.

Por que não enviar mais produtos?

Mais anúncios = mais visibilidade = mais vendas!

Dica: Comece pelos seus produtos mais vendidos na loja.

[Enviar mais produtos]

---
```

#### Conversão

**Evento de Conversão:** `Marketplace Anuncio Publicado` com `marketplace = "mercado_livre"`
**Janela de Atribuição:** 14 dias

---

### CAMP-028: Conversão para Anúncios Premium

**ID:** CAMP-028
**Categoria:** Engagement
**Prioridade:** Média
**Business Case:** BC11

#### Objetivo

**Goal:** Incentivar uso de anúncios premium no ML.
**KPI Primário:** Taxa de conversão para premium
**Meta:** >20% de conversão

#### Audiência

**Segmento:** Apenas anúncios clássicos

**Critérios de Inclusão:**

- `ml_classic_ads_count` > 0
- `ml_premium_ads_count` = 0
- `ml_sales_count` > 0 (já vendeu)

**Critérios de Exclusão:**

- Tem anúncios premium
- Recebeu esta campanha < 30 dias

#### Canal e Timing

**Canal Primário:** Email
**Canal Fallback:** Push

**Tipo:** Scheduled

- Frequência: Mensal para elegíveis

#### Conteúdo

**Email:**

```
Subject: Aumente suas vendas com anúncios Premium 🌟
Preview: Mais visibilidade para seus produtos no Mercado Livre

---

Olá {{profile.Name}},

Seus {{profile.ml_classic_ads_count}} anúncios clássicos já geraram vendas. Que tal aumentar ainda mais?

Anúncios Premium oferecem:
⭐ Maior destaque nas buscas
📈 Até 3x mais visualizações
🏆 Melhor posicionamento na listagem

Experimente converter alguns dos seus produtos mais populares para Premium.

[Criar anúncio Premium]

---
```

#### Conversão

**Evento de Conversão:** `Marketplace Anuncio Publicado` com `marketplace = "mercado_livre"` e `tipo_anuncio` = "premium"
**Janela de Atribuição:** 14 dias

---

## Checklist de Lançamento de Campanha

Para cada campanha, verificar antes do lançamento:

- [ ] Segmento configurado e validado (tamanho do público)
- [ ] Conteúdo aprovado (copy + design)
- [ ] Deep links testados em todos os dispositivos
- [ ] Personalização testada (preview com dados reais)
- [ ] A/B test configurado (se aplicável)
- [ ] Evento de conversão configurado
- [ ] Frequency cap aplicado
- [ ] Critérios de exclusão implementados
- [ ] Teste enviado para stakeholders
- [ ] Aprovação final recebida

---

## Métricas de Acompanhamento

| Métrica          | Fórmula              | Benchmark                 |
| ---------------- | -------------------- | ------------------------- |
| Delivery Rate    | Delivered / Sent     | >95%                      |
| Open Rate        | Opens / Delivered    | >20% (push), >25% (email) |
| CTR              | Clicks / Delivered   | >5%                       |
| Conversion Rate  | Conversions / Clicks | Varia por campanha        |
| Unsubscribe Rate | Unsubs / Delivered   | <0.5%                     |

---

## Changelog

| Data       | Versão | Alteração                            | Autor |
| ---------- | ------ | ------------------------------------ | ----- |
| 05/01/2026 | 1.0    | Versão inicial com 11 Business Cases | RMH   |
| 09/02/2026 | 1.1    | Campanhas BC6 gateway-agnósticas | RMH   |
| 09/02/2026 | 1.2    | Campanhas BC11 marketplace-agnósticas (ML * → Marketplace *) | RMH   |
| 12/03/2026 | 2.0    | Tradução completa: eventos e propriedades de EN para PT | RMH   |
