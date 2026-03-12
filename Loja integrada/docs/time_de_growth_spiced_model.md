# SPICED Model - Loja Integrada

## Context

A Loja Integrada e uma plataforma SaaS de e-commerce brasileira (VTEX) com modelo freemium e ~25k lojas ativas. A receita vem de **assinatura** (5 planos) e **consumo** (taxas transacionais em pagamento e logistica). Este documento aplica o framework SPICED (Winning by Design) para estruturar discurso comercial, metricas de pipeline e modelo matematico de receita.

**Escopo:** Produtos internos apenas (Plataforma, Pagali, Enviali). Parcerias e servicos ficam para fase posterior. Movimentos de venda (playbooks) nao estao no escopo.

---

## 1. Produtos e Modelos de Cobranca

| Produto | Modelo | Como cobra | Faixa |
|---------|--------|-----------|-------|
| **Plataforma (Planos)** | Recorrencia | Assinatura mensal/anual | R$0 a R$429/mes |
| **Taxa de Plataforma** | Consumo | % sobre cada venda na loja | 0,70% a 2,00% |
| **Pagali** | Consumo | % + fixo por transacao processada | Card: 3,59-4,99% + R$0,50; Pix: 1%; Boleto: R$1,99-2,39 |
| **Enviali** | Consumo | Margem por etiqueta comprada | Spread sobre tarifa negociada com transportadoras |

> Komea e Hub de Canais sao funcionalidades embutidas que impulsionam GMV e engajamento, gerando receita indireta via consumo.

### Tiers de Plano

| Plano | Preco/mes | Produtos | Visitas/mes | Taxa plataforma |
|-------|-----------|----------|-------------|-----------------|
| Gratuito | R$0 | 50 | 5.000 | 2,00% |
| Crescimento | R$59 | 250 | 25.000 | 0,99% |
| Aceleracao | R$99 | 500 | 50.000 | 0,90% |
| Expansao | R$399 | Ilimitado | 300.000 | 0,70% |
| Elite | R$429 | Ilimitado | Ilimitado | Custom |

---

## 2. SPICED - Discurso Comercial

### 2.1 Aquisicao (Novo Lojista)

#### S - Situation (Situacao atual do prospect)

> "Voce esta comecando a vender online / tem uma loja fisica e quer expandir para o digital / ja vende em marketplace e quer ter loja propria."

**Perguntas de diagnostico:**
- Como voce vende hoje? (Fisico, WhatsApp, marketplace, nenhum)
- Quantos produtos pretende vender?
- Qual seu faturamento mensal atual?
- Tem equipe tecnica ou faz tudo sozinho?

**Contexto de mercado:** 90% das lojas online no Brasil fecham em ate 120 dias. A maioria dos micro-empreendedores opera sozinha e pelo celular.

#### P - Pain (Dor)

> "Montar uma operacao de e-commerce exige integrar pagamento, frete, catalogo e design -- cada um com fornecedor e custo diferente. E complexo, caro e demorado."

**Dores tipicas por persona:**

| Persona | Dor principal | Dor secundaria |
|---------|--------------|----------------|
| Iniciante (sem loja) | Nao sabe por onde comecar | Medo de investir e nao vender |
| Lojista fisico | Nao tem tempo/conhecimento tecnico | Custo de montar loja online |
| Vendedor de marketplace | Dependencia de um canal, taxas altas | Sem marca propria, sem dados do cliente |
| Lojista com plataforma concorrente | Custo mensal alto para volume baixo | Ferramentas fragmentadas |

#### I - Impact (Impacto quantificado)

> "Com a Loja Integrada voce esta vendendo em menos de 5 minutos, sem custo inicial. A Komea configura sua loja por conversa."

**Impacto racional (quantitativo):**

| Metrica de impacto | Calculo sugerido |
|--------------------|-----------------|
| Economia em taxas | (Taxa_concorrente - Taxa_LI) x GMV_mensal |
| Economia em frete | Desconto Enviali (ate 70% Correios) x Volume_etiquetas |
| Receita incremental | GMV via marketplaces adicionais (Hub de Canais) |
| Tempo economizado | Horas/semana eliminadas com gestao manual vs. painel unico |

**Impacto emocional (qualitativo):**
- Seguranca de ter tudo em um lugar so
- Autonomia para operar sem depender de tecnicos
- Confianca de uma plataforma com milhoes de lojas criadas

#### CE - Critical Event (Evento critico)

> "Quando voce precisa estar vendendo?"

**Eventos criticos tipicos:**

| Evento | Urgencia |
|--------|----------|
| Black Friday / Natal | Sazonalidade -- deadline fixo |
| Lancamento de produto | Timing de mercado |
| Fim de contrato com plataforma atual | Janela de migracao |
| Atingiu limite do plano gratis | Crescimento organico forcando upgrade |
| Primeira venda em marketplace | Expansao natural para loja propria |
| Fluxo de caixa apertando | Precisa vender mais com menos custo |

#### D - Decision (Criterios de decisao)

> "Voce pode comecar gratis agora e so pagar quando crescer. Sem contrato, sem fidelidade."

**Criterios mapeados:**

| Criterio | Resposta LI |
|----------|-------------|
| Custo inicial | R$0 (plano gratuito) |
| Facilidade de uso | Setup em 5 min, IA copiloto (Komea) |
| Pagamento integrado | Pagali nativo (Pix, cartao, boleto) |
| Frete integrado | Enviali nativo (Correios -70%, Jadlog, Loggi) |
| Marketplace | Hub de Canais (ML, Magalu, etc.) |
| Escalabilidade | 5 tiers ate ilimitado |
| Suporte | Incluido, gerente dedicado no Elite |

---

### 2.2 Expansao (Lojista ativo - upsell/cross-sell)

#### S - Situation

> "Voce esta no plano [X], faturando R$[Y]/mes, com [Z] produtos ativos e [W] visitas/mes."

**Dados de contexto (extraidos do CRM):**
- Plano atual e limites (produtos, visitas)
- GMV ultimos 30/60/90 dias
- Gateway de pagamento ativo (Pagali ou externo)
- Metodo de envio configurado (Enviali ou externo)
- Taxa de conversao da loja
- Uso da Komea (acessos, conversas, execucoes)

#### P - Pain

| Situacao detectada | Dor |
|-------------------|-----|
| Plano gratis + vendendo | Taxa de plataforma de 2% comendo margem |
| Usando Mercado Pago | Taxa mais alta que Pagali, liquidacao mais lenta |
| Frete manual / externo | Sem desconto, processo manual, sem rastreio automatico |
| Muitos produtos, poucas visitas | Catalogo travado no limite do plano |
| Vende so na loja propria | Perdendo acesso a 300M+ compradores de marketplaces |
| Nao usa Komea | Operando manualmente, perdendo oportunidades de otimizacao |

#### I - Impact

| Acao de expansao | Impacto estimado |
|-----------------|-----------------|
| Upgrade Gratuito→Crescimento | Taxa cai de 2,00% para 0,99% (-50,5%). Em R$10k GMV = R$101/mes economizados vs. custo de R$59 |
| Migrar para Pagali | Pix a 1% (vs. taxas externas maiores) + liquidacao imediata + checkout transparente (maior conversao) |
| Ativar Enviali | Ate 70% desconto Correios + rastreio automatico + gestao centralizada |
| Ativar Hub de Canais | Canal adicional de receita com inventario sincronizado |
| Upgrade para Expansao | Produtos e visitas ilimitados + taxa de 0,70% |

#### CE - Critical Event

- Visitas/mes se aproximando do limite do plano
- Numero de produtos se aproximando do limite
- Volume de vendas tornando upgrade rentavel (break-even)
- Reclamacoes de clientes sobre frete/pagamento
- Concorrente aparecendo nos mesmos marketplaces

#### D - Decision

- ROI claro: economia em taxas > custo do upgrade
- Sem lock-in: pode fazer downgrade a qualquer momento
- Migracao transparente: ativa Pagali/Enviali sem pausar operacao

---

## 3. Bow-Tie: Estagios do Pipeline

```
FUNIL ESQUERDO (Aquisicao)                    FUNIL DIREITO (Retencao & Expansao)
                                 |
Awareness → Signup → Activation → 1st Product → 1st Sale ←KNOT→ Onboarding → Adoption → Expansion → Renewal
                                 |
                            Paid Conversion
```

### 3.1 Funil Esquerdo - Estagios

| # | Estagio | Definicao | Marco |
|---|---------|-----------|-------|
| L1 | Awareness | Visitante conhece a LI | Sessao no site |
| L2 | Signup | Cria conta gratis | Store ID gerado |
| L3 | Activation | Completa wizard inicial | Wizard concluido |
| L4 | First Product | Cria primeiro produto | Produto publicado (target: 7 dias) |
| L5 | First Sale | Recebe primeiro pedido | Pedido confirmado |
| L6 | Paid Conversion | Assina plano pago | Subscription Started |

### 3.2 Funil Direito - Estagios

| # | Estagio | Definicao | Marco |
|---|---------|-----------|-------|
| R1 | Onboarding Completo | Configura pagamento + envio | Pagali OU gateway ativo + Enviali OU envio ativo |
| R2 | First Impact | Vendas recorrentes (>1 venda) | 2a venda confirmada |
| R3 | Product Adoption | Adota produtos nativos | Pagali ativo + Enviali ativo |
| R4 | Expansion | Upgrade de plano OU crescimento GMV | Tier upgrade OU aumento de consumo |
| R5 | Recurring Impact | Vendas consistentes mes a mes | GMV estavel/crescente por 3+ meses |
| R6 | Renewal | Renovacao do plano | Subscription Renewed |
| R7 | Maximum Impact | Hub de Canais ativo + Komea engajado | Multichannel + IA copiloto |

### 3.3 Como Identificar o Estagio de uma Loja

Para classificar qualquer loja no pipeline, percorra as perguntas abaixo de baixo para cima. O estagio da loja e o **ultimo marco atingido**.

```
Checklist de classificacao (de baixo para cima):

[R7] Hub de Canais com anuncios ativos E Komea com 3+ acessos/mes?
[R6] Renovacao de plano pago confirmada?
[R5] GMV estavel ou crescente por 3+ meses consecutivos?
[R4] Fez upgrade de tier OU GMV crescendo mes a mes?
[R3] Pagali ativo E Enviali ativo (ambos)?
[R2] Tem 2 ou mais vendas confirmadas?
[R1] Pagamento configurado (qualquer) E envio configurado (qualquer)?
[L6] Tem assinatura paga ativa?
[L5] Recebeu pelo menos 1 pedido confirmado?
[L4] Tem pelo menos 1 produto publicado?
[L3] Completou o wizard inicial?
[L2] Tem Store ID (conta criada)?
[L1] Visitou o site mas nao criou conta
```

> **Regra:** Uma loja pode estar em apenas um estagio. O estagio e sempre o mais avancado cujo marco foi atingido. Uma loja em R3 ja passou por L2→L3→L4→L5→L6→R1→R2→R3.

### 3.4 Exemplos Praticos

#### Exemplo 1: Loja "Dona Maria Bijuterias"

| Dado | Valor |
|------|-------|
| Signup | 15 dias atras |
| Wizard | Concluido via Komea |
| Produtos | 8 publicados |
| Pedidos | 0 |
| Plano | Gratuito |
| Pagamento | Mercado Pago (externo) |
| Envio | Nao configurado |

**Estagio: L4 (First Product)**

Dona Maria criou a loja, completou o wizard, cadastrou 8 produtos, mas ainda nao recebeu nenhum pedido. Ela precisa configurar envio (BC1) e atrair trafego para alcancar L5.

**Proxima acao recomendada:** Campanhas de ativacao de frete (BC1) + regua educacional de primeira venda (BC5).

---

#### Exemplo 2: Loja "TechStore SP"

| Dado | Valor |
|------|-------|
| Signup | 4 meses atras |
| Produtos | 120 publicados |
| Pedidos | 47 confirmados (ultimos 30 dias) |
| Plano | Crescimento (R$59/mes) |
| Pagamento | Pagali (Pix + cartao) |
| Envio | Enviali (Correios + Jadlog) |
| GMV ultimos 3 meses | R$18k → R$22k → R$25k |
| Hub de Canais | Nao ativo |
| Komea | 1 acesso no mes |

**Estagio: R4 (Expansion)**

TechStore ja tem Pagali + Enviali ativos (R3 atingido), e o GMV esta crescendo mes a mes. Isso a coloca em R4. O proximo passo natural e upgrade para Aceleracao (taxa cai de 0,99% para 0,90%) e ativacao do Hub de Canais.

**Proxima acao recomendada:** Apresentar ROI do upgrade (economia de R$22,50/mes em taxa com GMV de R$25k) + ativacao Hub de Canais (BC11).

---

#### Exemplo 3: Loja "Artesanato da Ju"

| Dado | Valor |
|------|-------|
| Signup | 2 meses atras |
| Produtos | 3 publicados |
| Pedidos | 1 confirmado |
| Plano | Gratuito |
| Pagamento | Pagali (Pix) |
| Envio | Frete manual (sem Enviali) |

**Estagio: L5 (First Sale)**

Ju tem sua primeira venda, mas ainda esta no plano Gratuito. Ela configurou pagamento mas nao envio via Enviali, entao R1 nao esta completo. Ela precisa ser convertida para plano pago (L6).

**Proxima acao recomendada:** Ainda cedo para upgrade — foco em segunda venda e ativacao Enviali (BC1). Quando o volume justificar, apresentar economia de taxa (2,00% → 0,99%).

---

#### Exemplo 4: Loja "ModaFit Oficial"

| Dado | Valor |
|------|-------|
| Signup | 14 meses atras |
| Produtos | 450 publicados |
| Pedidos | 380/mes (media ultimos 6 meses) |
| Plano | Expansao (R$399/mes), renovado 1x |
| Pagamento | Pagali (Pix + cartao + boleto) |
| Envio | Enviali (Correios + Loggi) |
| GMV | R$180k/mes estavel ha 6 meses |
| Hub de Canais | Mercado Livre ativo (52 anuncios) |
| Komea | 8 acessos/mes, 12 conversas |

**Estagio: R7 (Maximum Impact)**

ModaFit atingiu o estagio maximo: plano renovado (R6), GMV estavel ha 6+ meses (R5), Hub de Canais ativo com anuncios E Komea engajada (R7). Esta loja e referencia para case de sucesso.

**Proxima acao recomendada:** Manter e nutrir. Explorar Elite se precisar de visitas ilimitadas. Monitorar share of wallet do Pagali.

---

#### Exemplo 5: Loja "PetShop Amigo"

| Dado | Valor |
|------|-------|
| Signup | 6 meses atras |
| Produtos | 85 publicados |
| Pedidos | 25/mes |
| Plano | Crescimento (R$59/mes) |
| Pagamento | Mercado Pago (externo) |
| Envio | Enviali (Correios) |
| GMV | R$8k/mes |

**Estagio: R2 (First Impact)**

PetShop tem vendas recorrentes (R2 atingido), mas **nao** esta em R3 porque usa Mercado Pago em vez de Pagali. R3 exige Pagali **e** Enviali ativos. Oportunidade clara de cross-sell.

**Proxima acao recomendada:** Migrar para Pagali (BC6) — Pix a 1% vs. taxa externa maior, liquidacao imediata, checkout transparente com maior conversao.

---

#### Exemplo 6: Loja "Mega Imports" (Churn Risk)

| Dado | Valor |
|------|-------|
| Signup | 10 meses atras |
| Produtos | 200 publicados |
| Pedidos ultimos 3 meses | 45 → 28 → 12 |
| Plano | Aceleracao (R$99/mes) |
| Pagamento | Pagali (cartao) |
| Envio | Enviali (Correios) |
| GMV ultimos 3 meses | R$35k → R$18k → R$7k |
| Visitas/mes | Em queda |

**Estagio: R5 (Recurring Impact) — mas em REGRESSAO**

Mega Imports ja atingiu R5 no passado, porem o GMV esta em queda livre. O estagio nao "volta", mas a loja esta em risco de churn. Se o GMV continuar caindo, o downgrade ou cancelamento e provavel.

**Proxima acao recomendada:** Acionar CS proativamente. Diagnosticar causa da queda (sazonalidade? concorrencia? problema operacional?). Propor Hub de Canais (BC11) para diversificar receita. Considerar oferta de retencao antes que solicite downgrade.

---

#### Resumo Visual dos Exemplos

```
L1  L2  L3  L4  L5  L6  R1  R2  R3  R4  R5  R6  R7
                 ●                                       Dona Maria (L4) — sem vendas
                     ●                                   Artesanato da Ju (L5) — 1 venda, gratis
                                 ●                       PetShop Amigo (R2) — sem Pagali
                                         ●               TechStore SP (R4) — GMV crescendo
                                             ⚠           Mega Imports (R5) — em regressao
                                                     ●   ModaFit (R7) — maximum impact
```

---

## 4. Metricas por Estagio

### 4.1 Volume (VM) - "Quantos / Quanto"

| Metrica | Estagio | O que mede |
|---------|---------|-----------|
| VM1 | L1 - Awareness | Sessoes no site lojaintegrada.com.br |
| VM2 | L2 - Signup | Novas contas criadas (lojas) |
| VM3 | L3 - Activation | Lojas que completaram wizard |
| VM4 | L4 - First Product | Lojas com pelo menos 1 produto |
| VM5 | L5 - First Sale | Lojas com pelo menos 1 pedido |
| VM6 | L6 - Paid Conversion | Novas assinaturas pagas |
| VM7 | L6 | Novo MRR adicionado (R$) |
| VM8 | R1 - Onboarding | Lojas fully-configured (pagamento + envio) |
| VM9 | R3 - Adoption | Lojas com Pagali ativo |
| VM10 | R3 - Adoption | Lojas com Enviali ativo |
| VM11 | R4 - Expansion | Upgrades de plano realizados |
| VM12 | R4 - Expansion | Expansion MRR (R$) |
| VM13 | R6 - Renewal | Renovacoes realizadas |
| VM14 | R6 | Churned MRR (R$) |
| VM15 | R6 | Downgrade MRR (R$) |

**Volume de consumo:**

| Metrica | Produto | O que mede |
|---------|---------|-----------|
| VM-C1 | Pagali | GMV processado via Pagali (R$) |
| VM-C2 | Pagali | Transacoes processadas (#) |
| VM-C3 | Enviali | Etiquetas compradas (#) |
| VM-C4 | Enviali | Saldo adicionado (R$) |
| VM-C5 | Plataforma | GMV total da plataforma (R$) |
| VM-C6 | Hub de Canais | Anuncios publicados (#) |

### 4.2 Conversao (CR) - "Eficiencia entre estagios"

**Funil Esquerdo:**

| Metrica | Calculo | Benchmark tipico SaaS |
|---------|---------|----------------------|
| CR1: Visit → Signup | VM2 / VM1 | 2-5% (PLG freemium) |
| CR2: Signup → Activated | VM3 / VM2 | 40-60% |
| CR3: Activated → 1st Product | VM4 / VM3 | 30-50% |
| CR4: 1st Product → 1st Sale | VM5 / VM4 | 10-25% |
| CR5: 1st Sale → Paid | VM6 / VM5 | 5-15% (freemium) |
| CR-end: Visit → Paid (end-to-end) | VM6 / VM1 | 0,01-0,1% |

**Funil Direito:**

| Metrica | Calculo | Meta sugerida |
|---------|---------|--------------|
| CR6: Paid → Fully Configured | VM8 / VM6 | >80% |
| CR7: Configured → Recurring Sales | Lojas com 2+ vendas / VM8 | >50% |
| CR8: Active → Pagali Adoption | VM9 / Lojas ativas pagas | >40% |
| CR9: Active → Enviali Adoption | VM10 / Lojas ativas pagas | >30% |
| CR10: GRR (Gross Revenue Retention) | (MRR_retained) / MRR_inicio | >85% |
| CR11: NRR (Net Revenue Retention) | (MRR_retained + Expansion) / MRR_inicio | >100% |
| CR12: Logo Retention | Lojas ativas fim / Lojas ativas inicio | >75% (mensal) |

### 4.3 Tempo (delta-t) - "Velocidade entre estagios"

**Funil Esquerdo:**

| Metrica | Mede | Target sugerido |
|---------|------|----------------|
| dt1: Visit → Signup | Tempo de decisao de criar loja | Sessao unica (minutos) |
| dt2: Signup → Activated | Completar wizard | <1 dia |
| dt3: Activated → 1st Product | Configurar primeiro produto | <7 dias (meta BC5) |
| dt4: 1st Product → 1st Sale | Time to first revenue | <30 dias |
| dt5: 1st Sale → Paid | Time to monetize (para LI) | <60 dias |
| dt-total: Visit → Paid | Ciclo completo de aquisicao | <90 dias |

**Funil Direito:**

| Metrica | Mede | Target sugerido |
|---------|------|----------------|
| dt6: Paid → Fully Configured | Onboarding completo | <14 dias |
| dt7: Configured → 2nd Sale | Time to First Impact | <30 dias |
| dt8: Paid → Pagali Active | Adocao pagamento nativo | <30 dias |
| dt9: Paid → Enviali Active | Adocao logistica nativa | <30 dias |
| dt10: Paid → First Upgrade | Time to expansion | <180 dias |
| dt11: Lifetime | Duracao media do cliente | Maximizar |

---

## 5. Math & Data Model

### 5.1 Decomposicao de Receita

```
Receita Total = Receita de Recorrencia + Receita de Consumo

Receita de Recorrencia (MRR):
  MRR = SUM(lojas_por_tier x preco_tier)
  ARR = MRR x 12

Receita de Consumo:
  = Taxa de Plataforma + Pagali + Enviali

  Taxa de Plataforma = SUM(GMV_loja x taxa_plataforma_do_tier)
  Pagali = SUM(GMV_pagali_cartao x take_rate_cartao) + (txns_boleto x fee_boleto) + (GMV_pix x take_rate_pix)
  Enviali = SUM(etiquetas x margem_por_etiqueta)
```

---

### 5.2 Funil Esquerdo (Aquisicao) - Modelo Detalhado

**Objetivo:** Calcular quantas visitas sao necessarias para gerar o New MRR target.

#### Formula de pipeline reverso (de meta para topo)

```
New MRR Target         = R$ [META]
Novas lojas pagas      = New MRR Target / ARPU_novo
                         (ARPU_novo = ticket medio do primeiro plano, ~R$59-99)

Lojas com 1a venda     = Novas lojas pagas / CR5
Lojas com produto      = Lojas com 1a venda / CR4
Lojas ativadas         = Lojas com produto / CR3
Signups necessarios    = Lojas ativadas / CR2
Visitas necessarias    = Signups necessarios / CR1
```

#### Exemplo numerico

```
Meta: R$50.000 de New MRR

ARPU novo medio = R$79 (mix de Crescimento e Aceleracao)
→ Novas lojas pagas = 50.000 / 79 = 633

CR5 (1st Sale → Paid) = 10%
→ Lojas com 1a venda = 633 / 0,10 = 6.330

CR4 (Product → Sale) = 15%
→ Lojas com produto = 6.330 / 0,15 = 42.200

CR3 (Activated → Product) = 40%
→ Lojas ativadas = 42.200 / 0,40 = 105.500

CR2 (Signup → Activated) = 50%
→ Signups = 105.500 / 0,50 = 211.000

CR1 (Visit → Signup) = 3%
→ Visitas = 211.000 / 0,03 = 7.033.333
```

#### Custo de Aquisicao

```
CAC = Investimento total S&M / Novas lojas pagas

CAC Payback (meses) = CAC / (ARPU_mensal x Margem Bruta)

LTV = ARPU_total_mensal x Margem Bruta / Churn Rate mensal

LTV:CAC ratio → Target >= 3:1
```

> **ARPU_total** inclui assinatura + receita de consumo media por loja. E o indicador mais importante porque captura o valor real de cada loja para a LI.

#### Calculo do ARPU Total (Left Funnel)

```
ARPU_total = ARPU_subscription + ARPU_consumption

ARPU_subscription = MRR / Lojas pagas ativas

ARPU_consumption = (Taxa Plataforma + Pagali + Enviali) / Lojas ativas
                 = (GMV_medio x taxa_media) + (GMV_pagali_medio x take_rate) + (etiquetas_media x margem)
```

---

### 5.3 Funil Direito (Retencao & Expansao) - Modelo Detalhado

**Objetivo:** Maximizar NRR mantendo e expandindo a receita da base existente.

#### Decomposicao do MRR

```
MRR Final = MRR Inicio
            + New MRR              (vem do funil esquerdo)
            + Expansion MRR        (upgrades + crescimento consumo)
            - Contraction MRR      (downgrades)
            - Churned MRR          (cancelamentos)
```

#### Expansion MRR: Duas alavancas

**Alavanca 1 - Upgrade de tier (recorrencia):**
```
Upgrade MRR = SUM(lojas_upgradadas x (preco_novo_tier - preco_antigo_tier))

Exemplo:
  100 lojas Gratuito → Crescimento = 100 x (R$59 - R$0) = R$5.900
  50 lojas Crescimento → Aceleracao = 50 x (R$99 - R$59) = R$2.000
  Total Upgrade MRR = R$7.900
```

**Alavanca 2 - Crescimento de consumo (GMV-driven):**
```
Delta Consumption = Consumption_mes_atual - Consumption_mes_anterior

Onde Consumption = Taxa Plataforma + Pagali + Enviali
```

> Importante: na LI, uma loja pode crescer sua contribuicao de receita SEM fazer upgrade de plano, simplesmente vendendo mais (mais GMV = mais taxa de plataforma + mais taxas Pagali + mais etiquetas Enviali). Isso e a alavanca de consumo.

#### Retencao: GRR e NRR

```
GRR = (MRR_inicio - Contraction - Churn) / MRR_inicio
     Maximo: 100%. Nunca inclui expansion.
     Target SaaS: 85-95%

NRR = (MRR_inicio + Expansion - Contraction - Churn) / MRR_inicio
     Pode ser >100% (crescimento da base existente sem novas vendas).
     Target SaaS: >100%, best-in-class: 120-140%
```

#### Churn Analysis

```
Logo Churn Rate = Lojas canceladas no periodo / Lojas ativas no inicio

Revenue Churn Rate = MRR cancelado no periodo / MRR no inicio

Churn por tier:
  Churn_tier_i = Lojas canceladas tier_i / Lojas ativas tier_i no inicio
```

> Expectativa: Churn do tier Gratuito sera muito maior que dos tiers pagos. Separar as cohorts e critico.

#### Metricas de Adocao de Produto (Cross-sell)

```
Pagali Penetration = Lojas com Pagali ativo / Lojas pagas ativas
Pagali Share of Wallet = GMV via Pagali / GMV total

Enviali Penetration = Lojas com Enviali ativo / Lojas pagas ativas
Labels per Store = Etiquetas mes / Lojas com Enviali ativo

Hub Penetration = Lojas com marketplace ativo / Lojas pagas ativas

Komea Engagement = Lojas com 3+ acessos Komea/mes / Lojas ativas
```

#### Receita por Produto de Consumo

```
Receita Pagali:
  = (GMV_cartao x take_rate_cartao)
    + (Txns_boleto x R$2,39)
    + (GMV_pix x 1%)

  ARPU_Pagali = Receita Pagali / Lojas com Pagali

Receita Enviali:
  = Etiquetas x Margem media por etiqueta

  ARPU_Enviali = Receita Enviali / Lojas com Enviali

Receita Taxa Plataforma:
  = SUM por tier(GMV_tier x taxa_tier)
```

---

### 5.4 Indicadores Consolidados

| Indicador | Formula | Frequencia | Owner |
|-----------|---------|-----------|-------|
| **MRR** | SUM(lojas x preco_tier) | Mensal | Finance/RevOps |
| **ARR** | MRR x 12 | Mensal | Finance |
| **New MRR** | MRR de novas assinaturas no mes | Mensal | Sales/Growth |
| **Expansion MRR** | Upgrades + delta consumo | Mensal | CS/Product |
| **Churned MRR** | MRR de lojas canceladas | Mensal | CS |
| **Contraction MRR** | MRR perdido por downgrade | Mensal | CS |
| **NRR** | (MRR_i + Exp - Contr - Churn) / MRR_i | Mensal | RevOps |
| **GRR** | (MRR_i - Contr - Churn) / MRR_i | Mensal | RevOps |
| **ARPU Total** | (MRR + Consumo) / Lojas ativas | Mensal | RevOps |
| **CAC** | Spend S&M / New paid stores | Mensal | Marketing |
| **LTV** | ARPU_total x GM / Churn_rate | Trimestral | RevOps |
| **LTV:CAC** | LTV / CAC (target >= 3:1) | Trimestral | RevOps |
| **CAC Payback** | CAC / (ARPU x GM) em meses | Trimestral | RevOps |
| **Logo Churn** | Lojas canceladas / Lojas inicio | Mensal | CS |
| **Pagali Penetration** | Lojas Pagali / Lojas pagas | Mensal | Product |
| **Enviali Penetration** | Lojas Enviali / Lojas pagas | Mensal | Product |
| **GMV Total** | Volume total transacionado | Mensal | Finance |
| **Consumption Rev Share** | Receita consumo / Receita total | Mensal | Finance |

---

### 5.5 Waterfall de MRR (Visao Executiva)

```
MRR Inicio de Mes
  + New MRR                    ← Funil Esquerdo (novas assinaturas)
  + Reactivation MRR           ← Reconquista de churned
  + Upgrade MRR                ← Funil Direito (tier up)
  - Downgrade MRR              ← Funil Direito (tier down)
  - Churn MRR                  ← Funil Direito (cancelamento)
  = MRR Final de Mes

Receita Total do Mes = MRR + Consumo do Mes
```

---

## 6. Business Cases Vinculados ao Pipeline

| BC | Produto | Modelo | Estagio Pipeline | Impacto em receita |
|----|---------|--------|-----------------|-------------------|
| BC4 | Plataforma | Recorrencia | L6 → R6 (Conversao → Renewal) | New MRR + Renewal MRR |
| BC5 | Plataforma | - | L4 (First Product) | Leading indicator de conversao |
| BC6 | Pagali | Consumo | R1/R3 (Onboarding/Adoption) | Receita Pagali por transacao |
| BC1 | Enviali | Consumo | R1/R3 (Onboarding/Adoption) | Habilita BC2/BC3 |
| BC2 | Enviali | Consumo | R3/R4 (Adoption/Expansion) | Receita por etiqueta |
| BC3 | Enviali (Loggi) | Consumo | R3 (Adoption) | Receita por etiqueta Loggi |
| BC7 | Komea | - | L3/R1 (Activation/Onboarding) | Reduz dt3 (time to product) |
| BC8 | Komea | - | L3 (Activation) | Reduz dt2 (time to publish) |
| BC9 | Komea | - | R5 (Recurring Impact) | Melhora retencao e engagement |
| BC10 | NF-e | - | R3 (Adoption) | Feature de retencao |
| BC11 | Hub de Canais | Consumo indireto | R4/R7 (Expansion/Max Impact) | GMV incremental → mais consumo |

---

## 7. Como Implementar

### Passo 1: Instrumentar metricas de volume
- Configurar tracking de cada VM no analytics (CleverTap + Data Lake)
- Garantir que cada estagio do pipeline tem um evento discreto

### Passo 2: Calcular conversoes historicas
- Extrair CR1-CR12 dos ultimos 6-12 meses
- Segmentar por cohort (mes de signup) e por tier

### Passo 3: Medir tempos
- Calcular mediana e P75 de cada delta-t
- Identificar gargalos (estagios com maior drop-off ou maior tempo)

### Passo 4: Montar waterfall de MRR
- Implementar classificacao mensal: New, Expansion, Contraction, Churn, Reactivation
- Automatizar via Airflow/dbt no Data Lake

### Passo 5: Dashboards
- **Executivo:** Waterfall MRR + NRR + GRR + ARPU Total
- **Growth/Marketing:** Funil esquerdo (VM1→VM7, CR1→CR5, dt1→dt5)
- **CS/Product:** Funil direito (VM8→VM15, CR6→CR12, dt6→dt11)
- **Produto:** Penetracao Pagali/Enviali/Hub + ARPU por produto
