# Enviali - Solução de Fretes da Loja Integrada

Documentação detalhada sobre o Enviali para suporte ao tracking plan.

---

## Visão Geral

O **Enviali** é a solução de fretes da Loja Integrada que ajuda na redução do custo do frete, oferecendo integração com múltiplas transportadoras e gestão centralizada de envios.

**Business Cases relacionados:**

- BC1: Configuração de Envio via Enviali
- BC2: Compra de Etiquetas via Enviali
- BC3: Ativação e Monetização Loggi

---

## Transportadoras Disponíveis

| Transportadora | Tipo    | Descrição                                  |
| -------------- | ------- | ------------------------------------------ |
| Correios       | Estatal | PAC, SEDEX, SEDEX 10, SEDEX 12, SEDEX Hoje |
| Jadlog         | Privada | Envios econômicos e expressos              |
| Loggi          | Privada | LoggiPonto - entregas urbanas              |

---

## Funcionalidades Principais

### 1. Gestão Centralizada de Fretes

**Descrição:** Painel único para gerenciar todas as opções de envio da loja.

**Funcionalidades:**

- Visualização de transportadoras ativas
- Configuração de métodos de envio
- Gestão de etiquetas
- Rastreamento de pedidos

---

### 2. Calculadora de Fretes

**Descrição:** Simulador integrado para cálculo de frete antes da venda.

**Uso:**

- Simular custos de envio
- Comparar transportadoras
- Definir preços de frete na loja

---

### 3. Emissão de Etiquetas

**Descrição:** Geração e impressão de etiquetas de envio diretamente pela plataforma.

**Funcionalidades:**

- Emissão individual ou em lote
- Impressão prática
- Cancelamento e reembolso
- Edição de dimensões
- Reimpressão de etiquetas perdidas
- Alteração de nome da loja na etiqueta

**Economia:** Até 70% de desconto em etiquetas dos Correios.

---

### 4. Rastreamento Automático

**Descrição:** Código de rastreio enviado automaticamente ao cliente.

**Funcionalidades:**

- Atualização automática de status
- Notificação ao cliente
- Histórico de movimentação

---

### 5. Gestão de Saldo

**Descrição:** Carteira para compra de etiquetas com valores pré-definidos.

**Funcionalidades:**

- Adição de créditos
- Valores mínimos e máximos
- Histórico de transações

---

## Processo de Ativação (BC1)

### Jornada Principal - Menu Lateral

1. Lojista acessa o painel administrativo
2. Clica em "Enviali" no menu lateral
3. Clica em "Ativar na minha loja"
4. Clica em "Acessar o painel do Enviali"
5. Pop-up de ativação exibido e fechado
6. Preenche dados iniciais pela faixa (CTA "Preencher")
7. Acessa "Correios e mais transportadoras"
8. Ativa método de envio (Correios ou transportadora)

### Dados Iniciais Necessários

- Endereço de origem (remetente)
- Dados da loja
- Informações de contato

### Pontos de Abandono

| Etapa           | Descrição                |
| --------------- | ------------------------ |
| Acesso inicial  | Não clicou em Enviali    |
| Ativação        | Não completou ativação   |
| Dados iniciais  | Não preencheu dados      |
| Transportadoras | Não ativou nenhum método |

---

## Compra de Etiquetas (BC2)

### Jornada Opção 1 - Listagem de Pedidos

1. Menu lateral > Vendas > Listar pedidos
2. Seleciona o pedido
3. Clica em "Pagar frete com desconto"
4. Escolhe o método de envio
5. Confirma o envio

### Jornada Opção 2 - Gerenciar Etiquetas

1. Menu lateral > Enviali > Gerenciar etiquetas
2. Visualiza etiquetas pendentes
3. Seleciona o pedido
4. Clica em "Pagar frete com desconto"
5. Pop-up "Continuar"
6. Escolhe opção de envio
7. Confirma pagamento
8. (Se necessário) Adicionar saldo:
   - Seleciona o valor
   - Continua para pagamento
   - Paga via cartão ou Pix
   - Finaliza pagamento

### Pontos de Abandono

| Etapa               | Descrição                                |
| ------------------- | ---------------------------------------- |
| Seleção de pedido   | Não selecionou pedido para envio         |
| Início do pagamento | Não clicou em "Pagar frete com desconto" |
| Escolha de envio    | Não selecionou transportadora            |
| Confirmação         | Não confirmou pagamento                  |
| Adição de saldo     | Abandonou ao precisar adicionar saldo    |

---

## Ativação da Loggi (BC3)

### Jornada de Ativação

1. Acessa menu lateral
2. Acessa página do Enviali
3. Confirma informações de envio (se necessário)
4. Clica em "Correios e mais transportadoras"
5. Seleciona "Loggi"
6. Clica em "Configurar"
7. Clica em "Ativar na minha loja"

### Jornada de Uso da Loggi

**Opção 1 - Listagem de Pedidos:**

1. Menu lateral > Vendas > Listar pedidos
2. Seleciona pedido
3. Clica em "Pagar frete com desconto"
4. Escolhe "Loggi"
5. Confirma envio

**Opção 2 - Gerenciar Etiquetas:**
(Mesmo fluxo do BC2, selecionando Loggi como transportadora)

### Requisitos da Loggi

- Documentação obrigatória por tipo de cadastro
- Limites de peso e dimensões específicos
- PUDOs (pontos de coleta) e horário de corte impactam prazo
- Lista de itens proibidos

---

## Detalhes por Transportadora

### Correios

**Serviços disponíveis:**

| Serviço    | Descrição       | Status     |
| ---------- | --------------- | ---------- |
| PAC        | Econômico       | Disponível |
| SEDEX      | Expresso        | Disponível |
| SEDEX 10   | Entrega até 10h | Disponível |
| SEDEX 12   | Entrega até 12h | Disponível |
| SEDEX Hoje | Same-day        | Beta       |

**Serviços adicionais:**

- Valor declarado
- Mão própria
- Aviso de Recebimento (AR)

**Requisitos:**

- Dimensões máximas e mínimas
- Regras de postagem
- Produtos frágeis (embalagem especial)

**Opções de envio:**

- Postagem em agência
- Coleta (quando disponível)

### Jadlog

**Funcionalidades:**

- Ativação da transportadora
- Homologação de embalagens
- Regras de envio específicas

**Requisitos:**

- Embalagens homologadas
- Dimensões e peso dentro dos limites
- Documentação fiscal correta

### Loggi

**Serviço principal:** LoggiPonto (entregas urbanas)

**Funcionalidades:**

- Ativação no sistema
- Integração com Enviali
- Geração de etiquetas
- Rastreamento

**Requisitos:**

- Documentação obrigatória
- Limites de peso e dimensões
- Embalagem correta
- Itens permitidos

---

## Cancelamentos, Sinistros e Indenizações

### Cancelamento de Etiquetas

- Processo de cancelamento disponível
- Reembolso conforme política

### Sinistros

**Tipos:**

- Extravio
- Avaria
- Atraso

**Processo:**

1. Identificar problema
2. Abrir chamado
3. Enviar documentação
4. Aguardar análise
5. Receber indenização (se aprovado)

### Política de Indenização

- Indenização por atraso diferenciada versus envios diretos
- Ressarcimento por extravio/avaria (Loggi, Jadlog)
- Avarias Correios (processo específico)

---

## Integração com Bling

**Funcionalidades:**

- Configuração da integração
- Vinculação manual de pedidos
- Geração de etiquetas no Bling
- Impressão integrada
- Rastreamento integrado
- Cancelamento de remessas

**Disponibilidade:** Funciona com plano gratuito

---
