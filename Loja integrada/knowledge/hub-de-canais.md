# Hub de Canais - Integração com Marketplaces

Documentação detalhada sobre o Hub de Canais para suporte ao tracking plan.

---

## Visão Geral

O **Hub de Canais** é a solução da Loja Integrada para centralizar vendas e integrar com múltiplos marketplaces, permitindo gerenciamento unificado de catálogo e pedidos.

**Disponibilidade:** Planos pagos

**Business Cases relacionados:**

- BC11: Ativação Canal Mercado Livre

---

## Marketplaces Integrados

| Marketplace     | Região         | Descrição                   |
| --------------- | -------------- | --------------------------- |
| Mercado Livre   | América Latina | Maior marketplace da região |
| Magalu          | Brasil         | Magazine Luiza marketplace  |
| Allever         | Brasil         | Marketplace de moda         |
| Compre Sua Peça | Brasil         | Marketplace de autopeças    |

---

## Funcionalidades Principais

### 1. Gestão Centralizada

**Descrição:** Painel único para gerenciar vendas em múltiplos canais.

**Funcionalidades:**

- Catálogo unificado
- Gestão de estoque centralizada
- Pedidos consolidados
- Sincronização automática

---

### 2. Publicação de Anúncios

**Descrição:** Envio de produtos da loja para os marketplaces integrados.

**Funcionalidades:**

- Envio em lote
- Configuração de atributos por marketplace
- Definição de tipos de anúncio
- Preços diferenciados por canal

---

### 3. Gestão de Estoque

**Descrição:** Controle de estoque sincronizado entre loja e marketplaces.

**Configurações:**

- Estoque mínimo
- Percentuais por tipo de anúncio
- Reserva de estoque

---

### 4. Recebimentos

**Descrição:** Fluxo de pagamento transparente para vendas em marketplaces.

**Nota:** Pagamentos são processados pelo marketplace e repassados conforme regras de cada plataforma.

---

## Mercado Livre (BC11)

### Visão Geral

O **Mercado Livre** é o maior marketplace da América Latina e uma das principais integrações do Hub de Canais.

### Tipos de Anúncio

| Tipo     | Descrição          | Comissão       |
| -------- | ------------------ | -------------- |
| Clássico | Anúncio básico     | Menor comissão |
| Premium  | Maior visibilidade | Maior comissão |

### Jornada de Ativação (BC11)

1. **Acesso ao Hub de Canais**
   - Menu lateral > Hub de Canais

2. **Configurar Mercado Livre**
   - Selecionar Mercado Livre
   - Iniciar configuração

3. **Autenticação**
   - Login com conta existente do Mercado Livre
   - OU cadastro de nova conta

4. **Configurações Iniciais**
   - Estoque mínimo
   - Percentuais por tipo de anúncio
   - Outras configurações específicas

5. **Envio de Anúncios**
   - Selecionar produtos
   - Escolher tipo de anúncio (Clássico/Premium)
   - Configurar atributos específicos do ML
   - Definir preços (opcional: preços estáveis)

6. **Conclusão**
   - Confirmar envio
   - Anúncios publicados no Mercado Livre

### Pontos de Abandono

| Etapa               | Descrição                             |
| ------------------- | ------------------------------------- |
| Acesso              | Não acessou o Hub de Canais           |
| Seleção             | Não selecionou Mercado Livre          |
| Autenticação        | Abandonou login/cadastro              |
| Configuração        | Não completou configurações iniciais  |
| Seleção de produtos | Não selecionou produtos para enviar   |
| Tipo de anúncio     | Não definiu tipo de anúncio           |
| Atributos           | Não configurou atributos obrigatórios |
| Confirmação         | Não confirmou envio                   |

### Funcionalidades Específicas

**Publicação:**

- Envio de novos produtos ao catálogo
- Atualização de anúncios existentes
- Configuração de atributos do ML

**Gestão:**

- Reautenticação de loja (quando necessário)
- Renovação de autorização de integração
- Configuração de frete alternativo

---

## Magalu

### Funcionalidades

- Vender produtos usando Hub de Canais (planos pagos)
- Reautenticação de loja
- Configuração de frete alternativo
- Renovação de autorização
- Envio de novos produtos

---

## Estratégia Omnichannel

### Vender em Marketplace e Loja Própria

**Abordagem recomendada:**

1. Iniciar vendas em marketplaces (alcance imediato)
2. Construir loja própria em paralelo
3. Desenvolver presença de marca
4. Balancear canais conforme estratégia

**Benefícios:**

- Diversificação de receita
- Menor dependência de um único canal
- Construção de marca própria
- Dados próprios de clientes (na loja)

---

## Campanhas Sugeridas (BC11)

### 1. Jornada de Ativação

**Objetivo:** Guiar conclusão de todas as etapas de configuração

**Segmento:** Lojistas que iniciaram mas não completaram a configuração

### 2. Incentivo à Adesão

**Objetivo:** Incentivar lojistas a conectar o Mercado Livre

**Segmento:** Lojistas com planos pagos sem ML conectado

### 3. Primeiro Anúncio

**Objetivo:** Incentivar envio do primeiro anúncio

**Segmento:** Lojistas conectados sem anúncios enviados

### 4. Expansão de Catálogo

**Objetivo:** Incentivar envio de mais produtos

**Segmento:** Lojistas com poucos produtos anunciados

### 5. Conversão para Premium

**Objetivo:** Incentivar uso de anúncios premium

**Segmento:** Lojistas apenas com anúncios clássicos

---

## Requisitos e Limitações

### Planos

- Hub de Canais disponível apenas para planos pagos
- Funcionalidades podem variar por plano

### Marketplace

- Cada marketplace tem requisitos próprios
- Atributos obrigatórios variam por categoria
- Políticas de comissão específicas
- Regras de envio do marketplace

### Estoque

- Sincronização automática
- Atenção a vendas simultâneas em múltiplos canais
- Configuração de estoque de segurança recomendada
