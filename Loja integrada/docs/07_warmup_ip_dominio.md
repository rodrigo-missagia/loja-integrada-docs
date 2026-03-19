# Guia de Warmup de IP e Domínio — Loja Integrada

**Versão:** 2.0
**Data:** 18 de Março de 2026
**Plataforma:** CleverTap
**Status:** Aprovado

---

## Sumário

1. [Introdução](#1-introdução)
2. [O Que É e Por Que Fazer Warmup](#2-o-que-é-e-por-que-fazer-warmup)
3. [Alocação de IP e Agenda de Warmup](#3-alocação-de-ip-e-agenda-de-warmup)
4. [Riscos e Mitigações](#4-riscos-e-mitigações)
5. [Dicas e Boas Práticas](#5-dicas-e-boas-práticas)
6. [Planejamento de Sunset: HubSpot → CleverTap](#6-planejamento-de-sunset-hubspot--clevertap)

---

## 1. Introdução

A Loja Integrada é uma plataforma SaaS de e-commerce brasileira com aproximadamente **700k usuários** na base total. Desses, **370k estão ativos nos últimos 6 meses** e constituem a base operacional para o CleverTap. Aproximadamente **110k são engajados** (abrem/clicam emails regularmente). Os ~330k restantes não serão ativados.

Como parte da estratégia de CRM e engajamento, estamos migrando a operação de email marketing do **HubSpot** para o **CleverTap**, utilizando um novo subdomínio dedicado: `comunicacao.lojaintegrada.com.br`.

| Métrica                               | Valor  |
| ------------------------------------- | ------ |
| Base total                            | 700k   |
| Base ativa (últimos 6 meses)          | 370k   |
| Base engajada (opens/clicks recentes) | 110k   |
| Base inativa (não será ativada)       | ~330k  |

### 1.1. Cenário Atual

Hoje, **todos os fluxos de email** saem pelo domínio principal `@lojaintegrada.com.br`, sem separação por tipo:

| Fluxo                                       | Plataforma       | Domínio                 |
| ------------------------------------------- | ---------------- | ----------------------- |
| Marketing e automações                      | HubSpot          | `@lojaintegrada.com.br` |
| Sales (emails comerciais)                   | HubSpot          | `@lojaintegrada.com.br` |
| Transacional (confirmações, reset de senha) | HubSpot          | `@lojaintegrada.com.br` |
| Corporativo (equipe interna)                | GSuite Workspace | `@lojaintegrada.com.br` |

Essa mistura de fluxos em um único domínio cria riscos significativos de entregabilidade: campanhas de marketing com baixo engajamento podem contaminar a reputação do domínio, afetando até emails transacionais e corporativos.

### 1.2. Estado Futuro

Com a migração para o CleverTap, o email marketing passará a usar o subdomínio `comunicacao.lojaintegrada.com.br`, isolando a reputação de marketing do restante dos fluxos. Este documento define o processo de warmup necessário para construir a reputação desse novo subdomínio e IP dedicado.

### 1.3. Objetivo

Garantir que as **campanhas definidas** possam ser disparadas via CleverTap com máxima entregabilidade, sem impactar negativamente o domínio principal da Loja Integrada. O warmup deve estar concluído até **20 de abril de 2026**, quando haverá um disparo para a base completa (370k).

---

## 2. O Que É e Por Que Fazer Warmup

### 2.1. O Que É

Warmup (ou "aquecimento") é o processo de **aumentar gradualmente o volume de emails enviados** a partir de um novo IP ou domínio. O objetivo é demonstrar aos provedores de email (ISPs) — como Gmail, Yahoo e Outlook — que o remetente é legítimo, envia conteúdo relevante e tem boas práticas de envio.

> **Analogia:** Um IP novo é como um CPF recém-criado — sem histórico de crédito. Ninguém empresta dinheiro (entrega emails) para quem não tem histórico. O warmup constrói esse "histórico de crédito" gradualmente.

### 2.2. Por Que É Necessário

- **IPs novos têm reputação zero.** São considerados "frios" pelos ISPs. Enviar em volume sem histórico resulta em bloqueio ou redirecionamento para a pasta de spam.
- **O subdomínio `comunicacao.lojaintegrada.com.br` herda parcialmente a reputação do domínio pai** (`lojaintegrada.com.br`), mas precisa construir a sua própria reputação de envio.
- **ISPs avaliam continuamente:** volume de envio, consistência, taxas de engajamento (opens, clicks), bounces e reclamações de spam.
- **Reputação é avaliada em janelas de 30 dias** (rolling basis). Se parar de enviar por mais de 30 dias, o warmup precisa ser refeito.

### 2.3. IP Dedicado vs. Compartilhado

| Característica    | IP Dedicado         | IP Compartilhado                |
| ----------------- | ------------------- | ------------------------------- |
| Reputação         | 100% própria        | Influenciada por outros senders |
| Warmup necessário | Sim                 | Não                             |
| Recomendado para  | > 50.000 emails/mês | < 50.000 emails/mês             |
| Controle          | Total               | Limitado                        |

**Recomendação para a Loja Integrada:** IP dedicado no CleverTap. Com 370k de base ativa e campanhas recorrentes, o volume justifica amplamente um IP exclusivo.

### 2.4. Domínio vs. Subdomínio vs. IP

| Componente                                          | Papel no Warmup                                                                                     |
| --------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Domínio** (`lojaintegrada.com.br`)                | Reputação de domínio compartilhada entre todos os subdomínios. Já possui histórico (bom e ruim)     |
| **Subdomínio** (`comunicacao.lojaintegrada.com.br`) | Herda algo do pai, mas constrói reputação própria. **Isola marketing de transacional/corporativo**  |
| **IP dedicado**                                     | Reputação independente do domínio. Precisa de warmup separado mesmo se o domínio já tiver histórico |

> **Por que usar subdomínio?** Se campanhas de marketing gerarem spam complaints, apenas o subdomínio `comunicacao` é afetado — emails transacionais e corporativos no domínio principal permanecem intactos.

### 2.5. Requisitos Técnicos (Pré-requisitos)

Antes de iniciar o warmup, todos os itens abaixo devem estar configurados:

| Requisito                   | Detalhe                                                                             | Status       |
| --------------------------- | ----------------------------------------------------------------------------------- | ------------ |
| **SPF**                     | Registro DNS para `comunicacao.lojaintegrada.com.br` incluindo servidores CleverTap | ✅ Concluído |
| **DKIM**                    | Chave de assinatura dedicada para o subdomínio                                      | ✅ Concluído |
| **DMARC**                   | `p=none` durante warmup → migrar para `p=quarantine` após 30 dias de estabilidade   | ✅ Concluído |
| **Provider CleverTap**      | Configurar email provider no CleverTap (sender, domínio, IP dedicado)               | Pendente     |
| **Google Postmaster Tools** | Configurar para monitorar reputação do domínio/subdomínio                           | Pendente     |
| **MXToolbox**               | Bookmark para verificação diária de blocklists                                      | Pendente     |

> **Atenção:** Desde fevereiro de 2024, Gmail e Yahoo exigem DMARC configurado para qualquer remetente com mais de 5.000 emails/dia. Sem DMARC, os emails sequer serão aceitos.

---

## 3. Alocação de IP e Agenda de Warmup

### 3.1. Arquitetura de Domínios e IPs (Estado Futuro)

| Fluxo                         | Domínio/Subdomínio                 | Plataforma                   | IP                 | Warmup                 |
| ----------------------------- | ---------------------------------- | ---------------------------- | ------------------ | ---------------------- |
| Marketing/CRM                 | `comunicacao.lojaintegrada.com.br` | CleverTap                    | IP dedicado (novo) | **Necessário**         |
| Transacional                  | `@lojaintegrada.com.br`            | Infraestrutura atual         | IP existente       | Já aquecido            |
| Corporativo                   | `@lojaintegrada.com.br`            | GSuite Workspace             | IPs Google         | Gerenciado pelo Google |
| Sales e CS (email individual) | `@lojaintegrada.com.br`            | HubSpot → migrar futuramente | IP HubSpot         | N/A                    |

### 3.2. Segmentação de Audiência por Fase

A segmentação é baseada nos dados de engajamento do HubSpot (opens e clicks históricos). A **base engajada do HubSpot (110k)** é o ponto de partida — não haverá fase interna.

| Fase                | Dias  | Audiência                               | Critério de Seleção                                       | Volume Estimado |
| ------------------- | ----- | --------------------------------------- | --------------------------------------------------------- | --------------- |
| 1 — Engajados       | 1-10  | Lojistas com engajamento recente        | Base engajada do HubSpot (opens/clicks recentes)          | ~110.000        |
| 2 — Ativos          | 11-15 | Lojistas ativos sem engajamento de email | Ativos nos últimos 6 meses, fora da base engajada         | ~260.000        |
| 3 — Escalar         | 16-23 | Base completa (recorrente)              | Todos os 370k, escalando capacidade de envio diário       | Até 370k/dia    |
| 4 — Estabilizar     | 24-28 | Base completa                           | Manter capacidade de 370k/dia para disparo de 20/abr      | 370k/dia        |

> **Nota:** Não haverá fase interna. Os primeiros envios usam a base engajada do HubSpot, que já possui histórico de abertura comprovado. A lista de 370k do HubSpot é considerada higienizada.

> **Fonte dos dados:** Exportar do HubSpot antes de iniciar: email, data do último open, data do último click, total de opens nos últimos 90 dias. Importar como user properties no CleverTap via CSV.

### 3.3. Tabela de Agendamento (Base de 370.000)

Agenda moderada-conservadora com duração de **28 dias**. Os primeiros 15 dias cobrem a totalidade da base (370k contatos únicos). Os dias 16-28 escalam a capacidade de envio diário para suportar o disparo completo de 20/abril.

**Premissas:**

- Janela de envio: **12 horas** (ex: 8h às 20h)
- Limite/Hora ≈ Total do Dia ÷ 12 (arredondado — é um teto aproximado, não valor exato)
- Nunca exceder 2x o volume do dia anterior
- **Início:** 23 de março de 2026 (segunda-feira)
- **Deadline:** 20 de abril de 2026 (disparo base completa)

| Dia   | Data       | Limite/Hora | Total/Dia | Acumulado | % Base  | Fase              |
| ----- | ---------- | ----------- | --------- | --------- | ------- | ----------------- |
| 1     | 23/3 Seg   | 42          | 500       | 500       | 0,14%   | Engajados         |
| 2     | 24/3 Ter   | 85          | 1.000     | 1.500     | 0,41%   | Engajados         |
| 3     | 25/3 Qua   | 170         | 2.000     | 3.500     | 0,95%   | Engajados         |
| 4     | 26/3 Qui   | 335         | 4.000     | 7.500     | 2,03%   | Engajados         |
| 5     | 27/3 Sex   | 585         | 7.000     | 14.500    | 3,92%   | Engajados         |
| 6     | 28/3 Sáb   | 835         | 10.000    | 24.500    | 6,62%   | Engajados         |
| 7     | 29/3 Dom   | 1.250       | 15.000    | 39.500    | 10,68%  | Engajados         |
| 8     | 30/3 Seg   | 1.670       | 20.000    | 59.500    | 16,08%  | Engajados         |
| 9     | 31/3 Ter   | 2.085       | 25.000    | 84.500    | 22,84%  | Engajados         |
| 10    | 1/4 Qua    | 2.125       | 25.500    | 110.000   | 29,73%  | Engajados (100%)  |
| 11    | 2/4 Qui    | 2.500       | 30.000    | 140.000   | 37,84%  | Ativos            |
| 12    | 3/4 Sex    | 3.335       | 40.000    | 180.000   | 48,65%  | Ativos            |
| 13    | 4/4 Sáb    | 4.170       | 50.000    | 230.000   | 62,16%  | Ativos            |
| 14    | 5/4 Dom    | 5.000       | 60.000    | 290.000   | 78,38%  | Ativos            |
| 15    | 6/4 Seg    | 6.670       | 80.000    | 370.000   | 100,00% | Base completa     |
| 16    | 7/4 Ter    | 8.335       | 100.000   | —         | —       | Escalar           |
| 17    | 8/4 Qua    | 10.420      | 125.000   | —         | —       | Escalar           |
| 18    | 9/4 Qui    | 12.500      | 150.000   | —         | —       | Escalar           |
| 19    | 10/4 Sex   | 16.670      | 200.000   | —         | —       | Escalar           |
| 20    | 11/4 Sáb   | 20.835      | 250.000   | —         | —       | Escalar           |
| 21    | 12/4 Dom   | 25.000      | 300.000   | —         | —       | Escalar           |
| 22    | 13/4 Seg   | 29.170      | 350.000   | —         | —       | Escalar           |
| 23    | 14/4 Ter   | 30.835      | 370.000   | —         | —       | Capacidade total  |
| 24-28 | 15–19/4    | 30.835      | 370.000   | —         | —       | Estabilizar       |
| **29** | **20/4 Seg** | **30.835** | **370.000** | —       | —       | **Disparo geral** |

> **Marco dia 10:** Toda a base engajada (110k) foi alcançada. A partir do dia 11, começam os envios para lojistas ativos mas sem engajamento de email — monitorar métricas com atenção redobrada.

> **Marco dia 15:** Toda a base ativa (370k) foi alcançada. A partir do dia 16, o foco muda de "alcançar novos contatos" para "escalar a capacidade diária" até atingir 370k/dia.

> **Volume diário pós-warmup:** Para operação regular, o volume diário será de **50-100k** (campanhas segmentadas). A capacidade de **370k/dia** fica reservada para disparos especiais (como o de 20/abril).

### 3.4. Regras de Segurança (Circuit Breakers)

Estas regras são **obrigatórias** durante todo o período de warmup. Se qualquer uma for acionada, o volume deve ser reduzido ou pausado.

| Métrica                 | Threshold de Alerta              | Ação                                                                                                          |
| ----------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Bounce rate**         | > 5% em qualquer dia             | Avaliar motivo antes do novo envio, reduzir volume para 50% do dia anterior caso motivo não seja identificado |
| **Open rate**           | < 15% na fase de engajados       | Manter volume atual por 2 dias extras antes de escalar                                                        |
| **Spam complaint rate** | > 0,1%                           | Pausar imediatamente, revisar conteúdo e segmentação                                                          |
| **Blocklist**           | IP aparece em qualquer blocklist | Pausar, solicitar remoção, reiniciar do último volume estável                                                 |
| **Crescimento diário**  | > 2x o volume do dia anterior    | Não permitido (os dias 1-3 atingem exatamente 2x, aceitável dado o volume baixo e base altamente engajada)    |

---

## 4. Riscos e Mitigações

### 4.1. Domínio principal contaminado pelo uso misto no HubSpot

**Situação atual:** O HubSpot envia transacional, sales e marketing pelo `@lojaintegrada.com.br` sem segmentação de subdomínios. Isso significa que a reputação do domínio é resultado da média de todos os fluxos — incluindo marketing com taxas de engajamento potencialmente baixas.

**Risco:** A reputação do domínio pai pode estar degradada. Se o subdomínio `comunicacao.lojaintegrada.com.br` herdar uma reputação negativa, o warmup será mais difícil.

**Mitigação:**

- Auditar a reputação atual do domínio via **Google Postmaster Tools** e **MXToolbox** antes de iniciar o warmup
- Se a reputação estiver degradada (classificação "Bad" ou "Low" no Postmaster Tools), considerar um período maior de warmup (35-40 dias)
- O uso do subdomínio dedicado isola o marketing/CRM do domínio pai — a longo prazo, isso melhora a situação
- Reduzir gradualmente os envios de marketing pelo HubSpot conforme o CleverTap assume (ver Seção 6)

### 4.2. Duplicação de campanhas entre HubSpot e CleverTap

**Regra absoluta:** Uma jornada **nunca** roda nas duas plataformas ao mesmo tempo. Migrou para o CleverTap = desliga no HubSpot imediatamente.

**Risco:** Sem disciplina na migração, lojista recebe email duplicado, gerando spam complaints que prejudicam ambos os domínios.

**Mitigação:**

- **Corte seco por jornada:** Ao ativar uma jornada no CleverTap, desativar a equivalente no HubSpot no mesmo dia — sem período de sobreposição
- **Priorizar jornadas do HubSpot:** A ordem de migração segue a prioridade das jornadas que já estão ativas no HubSpot (ver Seção 6.2)
- **Frequency capping:** Máximo 1 email de marketing/dia por lojista (considerando que ambas plataformas coexistem durante o sunset, mas com jornadas diferentes)

### 4.3. Impacto no GSuite Workspace

**Situação:** Emails corporativos (`@lojaintegrada.com.br`) compartilham o domínio principal com os envios de marketing do HubSpot.

**Risco:** Se a reputação do domínio cair por conta de spam complaints do marketing, emails corporativos para parceiros, fornecedores e clientes podem ir para a pasta de spam.

**Mitigação:**

- A migração do marketing para `comunicacao.lojaintegrada.com.br` resolve esse problema a médio prazo
- **Priorizar** a migração das jornadas de marketing (maior volume, maior risco) no cronograma de sunset
- Considerar subdomínio dedicado para Sales no futuro (ex: `vendas.lojaintegrada.com.br`) para isolar completamente o corporativo

### 4.4. Perda de dados de engajamento na migração

**Situação:** Todo o histórico de opens, clicks e interações dos lojistas está no HubSpot. O CleverTap começa do zero.

**Risco:** Sem esses dados, não é possível segmentar corretamente as fases do warmup (engajados → ativos).

**Mitigação:**

- **Exportar dados de engajamento do HubSpot antes de iniciar** o warmup
- Campos a exportar: `email`, `ultimo_open_date`, `ultimo_click_date`, `total_opens_90d`, `total_clicks_90d`, `opt_out_status`
- Importar como **user properties no CleverTap** via upload CSV
- Criar segmentos no CleverTap baseados nessas properties para uso durante o warmup
- A base de 370k do HubSpot é considerada **higienizada** — não é necessária etapa adicional de limpeza de lista

### 4.5. Queda de entregabilidade durante warmup

**Risco:** Nos primeiros dias do warmup, é normal haver alguma flutuação na entregabilidade. Se não monitorado, pode escalar para bloqueio do IP.

**Mitigação:**

- Manter **todas as campanhas críticas** (transacional, confirmações de pedido, reset de senha) na infraestrutura atual durante todo o warmup
- Apenas campanhas de marketing migram para o CleverTap
- Monitorar métricas diariamente e aplicar os circuit breakers da Seção 3.4

---

## 5. Dicas e Boas Práticas

### 5.1. Qual Conteúdo Enviar em Cada Fase

A escolha do conteúdo durante o warmup é crítica. Emails com alto potencial de engajamento (opens e clicks) devem ser priorizados nas fases iniciais para construir reputação positiva.

#### Fase 1: Engajados (Dias 1-10)

Usar campanhas com maior potencial de abertura e interação. Como a base engajada do HubSpot já tem histórico de opens, priorizar conteúdo de valor:

- **CAMP-013** (Confirmação de Assinatura) — email transacional-like, open rate naturalmente alto
- **CAMP-014** (Meta de Assinatura / Celebração) — tom positivo, engajamento forte
- **CAMP-019** (Régua Educacional de Qualidade) — conteúdo de valor, lojistas já ativos

> **Dica:** Nos primeiros 3 dias, testar manualmente a entregabilidade em Gmail, Yahoo e Outlook. Se cair em spam, marcar como "não é spam" para treinar os filtros dos ISPs.

#### Fase 2: Ativos (Dias 11-15)

Expandir para campanhas de ativação que demandam ação do lojista:

- **CAMP-001** (Ativação do Enviali) — lojistas que precisam configurar envio
- **CAMP-009** (Incentivo à Ativação da Loggi) — cross-sell de transportadora
- **CAMP-024** (Jornada de Ativação de Marketplace) — expansão de canais

> **Atenção:** A partir do dia 11, os destinatários são lojistas ativos na plataforma mas sem engajamento de email. Esperar open rates menores (~15-20%). Monitorar circuit breakers com rigor.

#### Fase 3-4: Escalar + Estabilizar (Dias 16-28)

Todas as campanhas podem rodar. Incluir campanhas de recuperação e reengajamento:

- **CAMP-011** (Recuperação de Abandono de Checkout) — lojistas que iniciaram assinatura mas não concluíram
- **CAMP-018** (Régua de Abandono de Cadastro de Produto) — lojistas que pararam no meio do onboarding
- **CAMP-022** (Descoberta do Potencial da Komea) — apresentar funcionalidade para base ampla

### 5.2. Higienização de Lista

A base de **370k do HubSpot é considerada higienizada** — não será necessário processo adicional de limpeza antes do warmup. Os ~330k inativos (que não acessam a plataforma há mais de 6 meses) **não serão importados** para o CleverTap.

#### Recomendações de manutenção contínua

Mesmo com a base limpa, manter boas práticas após o warmup:

- Remover hard bounces imediatamente após cada envio
- Respeitar todos os unsubscribes e opt-outs (LGPD)
- Monitorar emails com engajamento zero nos últimos 90 dias e avaliar remoção trimestral
- Se necessário validar subconjuntos no futuro, usar NeverBounce, ZeroBounce ou Clearout

### 5.3. Autenticação DNS (Pré-requisito Obrigatório)

Todos os registros DNS devem ser configurados **antes** do primeiro envio pelo CleverTap.

| Registro        | Configuração                              | Detalhes                                                                 |
| --------------- | ----------------------------------------- | ------------------------------------------------------------------------ |
| **SPF**         | `comunicacao.lojaintegrada.com.br`        | Incluir IPs/servidores CleverTap no registro. Manter < 10 lookups        |
| **DKIM**        | `comunicacao.lojaintegrada.com.br`        | Chave RSA 2048-bit mínimo, configurada no CleverTap                      |
| **DMARC**       | `_dmarc.comunicacao.lojaintegrada.com.br` | Fase 1: `p=none` (monitoramento) → Fase 2 (após 30 dias): `p=quarantine` |
| **Return-Path** | Customizado                               | Configurar para o subdomínio, garantir alinhamento com SPF               |

> **Validação:** Após configurar, enviar email de teste e verificar os headers com ferramentas como mail-tester.com ou Google Admin Toolbox.

### 5.4. Monitoramento Diário Durante Warmup

O monitoramento deve ser feito **todos os dias** durante os 28 dias de warmup. Designar um responsável no time de Growth.

#### Métricas e Metas

| Métrica             | Meta              | Alerta            | Ferramenta              |
| ------------------- | ----------------- | ----------------- | ----------------------- |
| Delivery rate       | > 98%             | < 95%             | CleverTap Dashboard     |
| Open rate           | > 20% (engajados) | < 15%             | CleverTap Dashboard     |
| Click rate          | > 2%              | < 1%              | CleverTap Dashboard     |
| Bounce rate         | < 2%              | > 5%              | CleverTap Dashboard     |
| Spam complaint rate | < 0,05%           | > 0,1%            | Google Postmaster Tools |
| Blocklist status    | Limpo             | Qualquer listagem | MXToolbox               |
| Domain reputation   | Medium/High       | Low/Bad           | Google Postmaster Tools |

#### Comparação com HubSpot

Manter um baseline das métricas históricas do HubSpot para comparação. Se as métricas do CleverTap estiverem significativamente piores que o HubSpot nas mesmas campanhas, investigar antes de escalar volume.

### 5.5. Manter Consistência de Envio (Regra dos 30 Dias)

- ISPs mantêm dados de reputação por aproximadamente **30 dias** em base rolling
- Se parar de enviar por mais de 30 dias, será necessário **refazer o warmup do zero**
- Após completar o warmup, manter volume mínimo de **~1.000 emails/dia** por ISP principal (Gmail, Yahoo, Outlook) para preservar a reputação
- Se não houver campanha para enviar, considerar digest semanal ou newsletter de conteúdo

### 5.6. Timing de Início

O warmup inicia em **23 de março de 2026** (segunda-feira), período excelente (pós-festas, ISPs menos congestionados). O disparo completo de 20/abril ocorre 28 dias depois, com 5 dias de estabilização (dias 24-28) antes do envio geral.

> **Regra geral:** Não iniciar warmup em véspera de grandes campanhas sazonais. Concluir o warmup com pelo menos 15 dias de antecedência de grandes eventos.

### 5.7. Engajamento Contínuo Pós-Warmup

Após completar o warmup (dia 28+) e o disparo geral de 20/abril:

- Manter volume diário regular de **50-100k** (campanhas segmentadas) para preservar reputação
- Para lojistas da base ativa (370k) com baixo engajamento, enviar em **lotes menores** (5-10k/dia) com conteúdo de alto valor
- Campanhas de reengajamento com incentivo: desconto no plano, trial de funcionalidade premium (Komea, Enviali)
- Se o engajamento for muito baixo (open rate < 5% após 3 tentativas), **remover da lista ativa**
- Os ~330k inativos **não serão reativados** — foco na qualidade da base de 370k

---

## 6. Planejamento de Sunset: HubSpot → CleverTap

### 6.1. Princípios da Transição

- **Contrato HubSpot:** encerra em **maio de 2026**. Todas as jornadas de marketing devem estar 100% no CleverTap até essa data.
- **Início do warmup:** Segunda-feira, **23 de março de 2026**.
- **Disparo base completa:** **20 de abril de 2026** (370k) — warmup deve estar concluído.
- **Janela total:** ~6 semanas (23/mar → 3/mai). Warmup (4 semanas) + migração de jornadas em paralelo + buffer (1 semana).
- **Jornadas no HubSpot:** ~7 jornadas ativas a serem migradas.
- **Corte seco:** Migrou para o CleverTap = desliga no HubSpot no mesmo dia. Nenhuma jornada roda nas duas plataformas simultaneamente.
- **Configuração em paralelo:** As 7 jornadas são configuradas no CleverTap durante as semanas 1-2 do warmup, e ativadas gradualmente a partir da semana 3.

### 6.2. Cronograma de Sunset (6 semanas — 23/mar a 3/mai/2026)

> **Regra de migração:** Para cada jornada: (1) configurar no CleverTap, (2) testar com amostra de 5%, (3) se métricas OK → ativar no CleverTap e desligar no HubSpot **no mesmo dia**. Sem período de coexistência.

| Fase | Semana | Data          | Ação                                                   | HubSpot                          | CleverTap                                |
| ---- | ------ | ------------- | ------------------------------------------------------ | -------------------------------- | ---------------------------------------- |
| 1    | S1     | 23/3–29/3     | Warmup dias 1-7                                        | Nada muda                        | Warmup com base engajada                 |
| —    | S1-S2  | 23/3–5/4      | **Paralelo:** configurar 7 jornadas no CT              | —                                | Templates + audiências + triggers        |
| 2    | S2     | 30/3–5/4      | Warmup dias 8-14                                       | Nada muda                        | Warmup com base engajada                 |
| 3    | S3     | 6/4–12/4      | Warmup dias 15-21 + primeiras migrações                | Desligar 2-3 jornadas            | Ativar 2-3 jornadas                      |
| 4    | S4     | 13/4–19/4     | Estabilização + mais migrações                         | Desligar 2-3 jornadas            | Ativar 2-3 jornadas                      |
| —    | —      | **20/4 (Seg)** | **Disparo base completa — 370k**                       | —                                | **Envio geral**                          |
| 5    | S5     | 20/4–26/4     | Migrar últimas jornadas                                | Desligar 1-2 últimas jornadas    | Ativar 1-2 últimas jornadas              |
| 6    | S6     | 27/4–3/5      | Buffer + desligamento total                            | **HubSpot OFF para marketing**   | **Operação 100% no CleverTap**           |

### 6.2.1. Preparação Paralela (Crítico para cumprir o prazo)

Enquanto o warmup acontece (S1-S2), o time deve trabalhar em paralelo na preparação das jornadas:

- [ ] **S1:** Levantar inventário completo das 7 jornadas ativas no HubSpot (triggers, audiências, conteúdo, frequência)
- [ ] **S1-S2:** Configurar todas as 7 jornadas no CleverTap (templates, audiências, triggers) — sem ativar
- [ ] **S1:** Exportar dados de engajamento do HubSpot e importar como user properties no CleverTap
- [ ] **S2:** Testar primeiras jornadas configuradas com amostra de 5% antes do corte
- [ ] **S3:** Iniciar corte seco — ativar no CT e desligar no HS no mesmo dia
- [ ] **S3-S4:** Ir ligando jornadas gradualmente, 2-3 por semana, monitorando métricas

> **Ritmo de migração:** Com 7 jornadas e 3 semanas de migração ativa (S3-S5), o ritmo é de ~2-3 jornadas por semana. Priorizar as jornadas de maior volume primeiro.

### 6.3. Checklist por Jornada Migrada

Para cada jornada individual, executar este checklist. O corte deve ser feito no mesmo dia — sem período de sobreposição.

- [ ] Jornada recriada no CleverTap com mesmo conteúdo e segmentação
- [ ] Audiência configurada com as mesmas regras de inclusão/exclusão
- [ ] Teste com amostra de 5% da audiência — comparar métricas vs. histórico do HubSpot
- [ ] Se métricas OK: ativar 100% no CleverTap **E desativar no HubSpot no mesmo dia**
- [ ] Confirmar que a jornada no HubSpot está desligada (verificar status)
- [ ] Documentar resultado: data do corte, métricas comparadas, observações

### 6.4. O Que Permanece em Cada Plataforma (Pós-Sunset — Maio/2026+)

| Plataforma               | Fluxos                                                              | Domínio                            |
| ------------------------- | ------------------------------------------------------------------- | ---------------------------------- |
| **CleverTap**             | 100% das jornadas de CRM e marketing                                | `comunicacao.lojaintegrada.com.br` |
| **HubSpot**               | **Contrato encerrado.** Renovação apenas para Sales (se necessário) | —                                  |
| **Infraestrutura atual**  | Transacional (confirmações, reset de senha, notificações de pedido) | `@lojaintegrada.com.br`            |
| **GSuite Workspace**      | Emails corporativos (equipe interna)                                | `@lojaintegrada.com.br`            |

> **Atenção:** Se a equipe comercial depende do HubSpot CRM para emails de Sales, será necessário avaliar: (a) renovar contrato do HubSpot apenas para Sales, (b) migrar Sales para outra ferramenta, ou (c) usar subdomínio dedicado (`vendas.lojaintegrada.com.br`) no CleverTap. Essa decisão deve ser tomada até a **S4 (13/abr)** para não bloquear o desligamento.

---

## Referências

- [Twilio SendGrid — IP Warm Up Guide](https://www.twilio.com/en-us/resource-center/email-guide-ip-warm-up)
- [CleverTap — IP Warmup](https://docs.clevertap.com/docs/ip-warmup)
- [CleverTap — Email Sender Reputation](https://docs.clevertap.com/docs/email-sender-reputation)
