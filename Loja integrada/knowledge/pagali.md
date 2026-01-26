# Pagali - Solução de Pagamento da Loja Integrada

Documentação detalhada sobre o Pagali para suporte ao tracking plan.

---

## Visão Geral

O **Pagali** é a solução de pagamento nativa da Loja Integrada, oferecendo Pix, boleto, cartão de crédito e link de pagamento. Disponível para todos os planos de assinatura.

**Business Cases relacionados:**

- BC6: Configurar Meio de Pagamento (Pagali)

---

## Meios de Pagamento Disponíveis

| Meio de Pagamento | Disponibilidade | Descrição |
|-------------------|-----------------|-----------|
| Pix | Todos os planos | Pagamento instantâneo |
| Boleto Bancário | Todos os planos | Pagamento via boleto |
| Cartão de Crédito | Todos os planos | Pagamento com cartão |
| Link de Pagamento | Todos os planos | Links para pagamento externo |

---

## Funcionalidades Principais

### 1. Checkout Transparente

**Descrição:** Permite que o cliente finalize a compra sem ser redirecionado para páginas externas, aumentando a conversão.

**Benefícios:**

- Maior taxa de conversão
- Experiência de compra fluida
- Credibilidade da marca mantida

---

### 2. Sistema Antifraude

**Descrição:** Proteção automática para vendas com cartão de crédito, com explicações sobre recusas de transações.

**Funcionalidades:**

- Análise automática de transações
- Proteção contra fraudes
- Explicação de recusas

---

### 3. Gestão de Recebimentos

**Descrição:** Controle de saldo, agendamento de pagamentos e gerenciamento de saques.

**Funcionalidades:**

- Visualização de saldo
- Agendamento de recebimentos
- Solicitação de saques
- Limites e regras de saque

---

### 4. Chargeback

**Descrição:** Processo de disputa de chargebacks com documentação e procedimentos definidos.

---

### 5. Estornos

**Descrição:** Cancelamento e reembolso de vendas para cartão de crédito e Pix.

---

## Processo de Cadastro e Verificação

### Etapas do Cadastro

1. **Criação de conta**
   - Acesso ao Pagali
   - Preenchimento de dados básicos

2. **Verificação de conta**
   - Envio de documentação
   - Validação de dados
   - Ativação de funcionalidades completas
   - Aumento de limite de saque

3. **Configuração de meios de pagamento**
   - Ativação de Pix
   - Ativação de cartão de crédito
   - Ativação de boleto
   - Configuração de link de pagamento

### Dados Necessários por Tipo de Cadastro

**Pessoa Física (CPF):**

- CPF
- Nome completo
- Data de nascimento
- Endereço
- Dados bancários

**Pessoa Jurídica (CNPJ):**

- CNPJ
- Razão social
- Dados do representante legal
- Endereço comercial
- Dados bancários

---

## Fluxo do BC6: Configurar Meio de Pagamento

### Jornada Principal (via Komea)

1. Acessar Komea
2. Selecionar configuração de meio de pagamento
3. Ser direcionado ao Pagali
4. Preencher informações necessárias (varia CPF/CNPJ)
5. Aguardar análise
6. **Resultado: Aprovado** → Meios de pagamento liberados
7. **Resultado: Não aprovado** → Fallback para Mercado Pago

### Jornada Alternativa (Direto pelo Painel)

1. Acessar menu lateral
2. Clicar em Pagali
3. Iniciar cadastro
4. Preencher informações
5. Aguardar aprovação

### Pontos de Abandono Críticos

| Etapa | Descrição |
|-------|-----------|
| Acesso inicial | Não iniciou cadastro |
| Dados pessoais | Abandonou preenchimento de CPF/CNPJ |
| Renda extra | Não preencheu informações de renda |
| Documentação | Não enviou documentos necessários |
| Dados bancários | Não completou dados bancários |

---

## Resultados de Aprovação

### Aprovado

**Meios liberados podem incluir:**

- Pix
- Cartão de crédito
- Boleto

**Nota:** Nem todos os meios são liberados automaticamente. Depende da análise.

### Não Aprovado

**Motivos comuns (internos):**

- Documentação incompleta
- Dados inconsistentes
- Restrição cadastral
- Atividade de risco

**Ação do usuário:**

- Corrigir pendências
- Configurar Mercado Pago como alternativa

---

## Atributos para Tracking

### Atributos de Status

| Atributo | Tipo | Descrição |
|----------|------|-----------|
| `pagali_account_status` | string | approved, rejected, pending, not_started |
| `pagali_verification_status` | string | verified, unverified |
| `pagali_registration_date` | date | Data do cadastro |
| `pagali_approval_date` | date | Data da aprovação |

### Atributos de Configuração

| Atributo | Tipo | Descrição |
|----------|------|-----------|
| `pagali_pix_enabled` | boolean | Pix ativado |
| `pagali_credit_card_enabled` | boolean | Cartão ativado |
| `pagali_boleto_enabled` | boolean | Boleto ativado |
| `pagali_payment_link_enabled` | boolean | Link de pagamento ativado |

### Atributos de Abandono

| Atributo | Tipo | Descrição |
|----------|------|-----------|
| `pagali_registration_step` | string | Última etapa completada |
| `pagali_abandoned_step` | string | Etapa onde abandonou |
| `pagali_rejection_reason` | string | Motivo da não aprovação (interno) |

### Atributos de Jornada

| Atributo | Tipo | Descrição |
|----------|------|-----------|
| `pagali_entry_source` | string | komea, panel, direct |
| `pagali_left_komea_flow` | boolean | Saiu do fluxo Komea para o painel |

---

## Eventos Potenciais para Tracking

### Cadastro

| Evento | Descrição |
|--------|-----------|
| `Pagali Registration Started` | Iniciou cadastro |
| `Pagali Registration Step Completed` | Completou etapa do cadastro |
| `Pagali Registration Abandoned` | Abandonou cadastro |
| `Pagali Registration Completed` | Finalizou cadastro |

### Verificação

| Evento | Descrição |
|--------|-----------|
| `Pagali Verification Started` | Iniciou verificação |
| `Pagali Verification Completed` | Verificação concluída |
| `Pagali Account Approved` | Conta aprovada |
| `Pagali Account Rejected` | Conta rejeitada |

### Configuração de Meios

| Evento | Descrição |
|--------|-----------|
| `Pagali Pix Enabled` | Pix ativado |
| `Pagali Credit Card Enabled` | Cartão ativado |
| `Pagali Boleto Enabled` | Boleto ativado |
| `Pagali Payment Link Enabled` | Link de pagamento ativado |

### Transações

| Evento | Descrição |
|--------|-----------|
| `Pagali Transaction Completed` | Transação concluída |
| `Pagali Transaction Failed` | Transação falhou |
| `Pagali Chargeback Received` | Chargeback recebido |
| `Pagali Refund Processed` | Estorno processado |

### Saques

| Evento | Descrição |
|--------|-----------|
| `Pagali Withdrawal Requested` | Saque solicitado |
| `Pagali Withdrawal Completed` | Saque concluído |

---

## Documentação Fiscal

### Nota Fiscal do Serviço

- Nota fiscal do serviço financeiro disponível
- Emitida pela Loja Integrada

### DIRF (Declaração de Imposto de Renda)

- Documentação disponível para declaração de IR
- Lojista pode obter notas fiscais para fins de imposto

---

## Produtos Restritos

Existem categorias de produtos que não podem ser vendidos usando o Pagali. O lojista deve consultar a lista de produtos restritos para evitar suspensão de conta.

---

## Suporte

- Canal de chamados específico para questões do Pagali
- Suporte para chargebacks
- Orientação para consumidores (mercadoria não recebida)
