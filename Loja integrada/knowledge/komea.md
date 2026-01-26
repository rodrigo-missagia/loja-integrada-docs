# Komea - Copiloto de IA da Loja Integrada

Documentação detalhada sobre a Komea para suporte ao tracking plan.

---

## Visão Geral

A **Komea** é o copiloto de IA da Loja Integrada, projetado para auxiliar lojistas em diversas operações da loja virtual. Disponível em beta aberto para todos os planos.

**Business Cases relacionados:**

- BC5: Criar Primeiro Produto (via Komea)
- BC6: Configurar Meio de Pagamento (via Komea)
- BC7: Personalizar Vitrine na Komea
- BC8: Publicar Site na Komea
- BC9: Usar Komea como Copiloto

---

## Funcionalidades Principais

### 1. Criação de Produtos com IA

**Descrição:** Automatiza o preenchimento de informações de produtos usando inteligência artificial, economizando tempo no cadastro.

**Jornada do usuário:**

1. Acessar a Komea
2. Selecionar "Criar produto"
3. Fornecer informações básicas (nome, categoria)
4. IA preenche automaticamente descrição, atributos
5. Revisar e confirmar
6. Produto criado

**Atributos relevantes:**

- Produto criado via IA (sim/não)
- Tempo de criação
- Categoria do produto
- Campos preenchidos automaticamente

---

### 2. Analista de Produtos

**Descrição:** Monitora visitas, vendas e métricas de conversão por produto, com alertas inteligentes.

**Status:** Beta

**Jornada do usuário:**

1. Acessar a Komea
2. Navegar até "Analista de Produtos"
3. Visualizar métricas organizadas
4. Receber alertas inteligentes
5. Executar ações sugeridas

---

### 3. Criador de Promoções

**Descrição:** Configura descontos, brindes e campanhas promocionais usando comandos em linguagem natural.

**Status:** Beta

**Jornada do usuário:**

1. Acessar a Komea
2. Descrever promoção desejada em linguagem natural
3. IA configura a promoção
4. Revisar e ativar

---

### 4. Operador de Carrinho (Recuperação de Vendas)

**Descrição:** Automatiza a recuperação de carrinhos abandonados, gerenciando produtos abandonados e pedidos cancelados.

**Status:** Beta

**Jornada do usuário:**

1. Ativar Operador de Carrinho
2. Configurar regras de recuperação
3. Sistema automatiza contato com clientes
4. Vendas recuperadas

---

### 5. Assistente de Dados

**Descrição:** Permite consultar dados de vendas, produtos e clientes usando linguagem natural.

**Status:** Beta

**Jornada do usuário:**

1. Acessar Assistente de Dados
2. Fazer pergunta em linguagem natural
3. Receber resposta com dados
4. Executar ação baseada nos dados (opcional)

---

### 6. Painel de Oportunidades

**Descrição:** Identifica oportunidades de crescimento e sugere ações personalizadas para a loja.

**Jornada do usuário:**

1. Acessar Komea
2. Visualizar Painel de Oportunidades
3. Ver sugestões personalizadas
4. Executar ação sugerida

**Atributos relevantes:**

- Tipo de oportunidade
- Ação executada
- Resultado da ação

---

### 7. Segmentação RFV (Recência, Frequência, Valor)

**Descrição:** Segmenta clientes por Recência, Frequência e Valor para reativação e aumento de vendas.

**Jornada do usuário:**

1. Acessar análise RFV na Komea
2. Visualizar segmentos de clientes
3. Identificar clientes para reativação
4. Criar campanha direcionada

---

### 8. Gestão de Conversas

**Descrição:** Permite renomear e deletar conversas nos Agentes de IA para manter histórico organizado.

---

## Fluxos Específicos dos Business Cases

### BC5: Criar Primeiro Produto via Komea

**Jornada completa:**

1. Usuário acessa Komea
2. Inicia cadastro de produto
3. IA sugere/preenche informações
4. Usuário confirma dados
5. Produto criado com sucesso

**Pontos de abandono:**

- Saiu antes de iniciar cadastro
- Abandonou durante preenchimento
- Não confirmou produto final

---

### BC6: Configurar Meio de Pagamento via Komea

**Jornada completa:**

1. Usuário acessa Komea
2. Komea direciona para configurar Pagali
3. Usuário segue fluxo de configuração do Pagali
4. Retorna à Komea após configuração

**Pontos de abandono:**

- Saiu da Komea antes de ir ao Pagali
- Abandonou configuração do Pagali

---

### BC7: Personalizar Vitrine na Komea

**Jornada completa:**

1. Acessar Komea
2. Selecionar caminho: subir logo ou gerar por IA
3. Se gerar: IA apresenta 3 opções de logo
4. Selecionar logo preferido
5. Selecionar 1 cor (4 opções disponíveis)
6. Finalizar personalização

**Pontos de abandono:**

- Subiu logo mas não selecionou cor
- Gerou logo mas não finalizou
- Abandonou seleção de cor

**Atributos relevantes:**

- Logo gerado por IA (sim/não)
- Logo escolhido por IA (sim/não)
- Cor selecionada
- Modificação de cor pós-publicação

---

### BC8: Publicar Site na Komea

**Jornada completa:**

1. Acessar Komea
2. Selecionar "Publicar site"
3. Preencher CPF/CNPJ
4. Preencher endereço
5. Confirmar publicação
6. Site sai do status de manutenção

**Fluxo alternativo (com pagamento configurado):**

1. Acessar Komea
2. Selecionar "Publicar site"
3. Publicar diretamente (dados já preenchidos via Pagali)

**Pontos de abandono:**

- Abandonou preenchimento de CPF/CNPJ
- Abandonou preenchimento de endereço
- Não confirmou publicação

**Atributos relevantes:**

- Configurou meio de pagamento antes
- Acessou painel (saiu do fluxo Komea)

---

### BC9: Usar Komea como Copiloto (Base Existente)

**Jornada Opção 1 - Oportunidades:**

1. Acessar Komea
2. Navegar pelas oportunidades
3. Executar uma ação sugerida

**Jornada Opção 2 - Assistentes:**

1. Acessar Komea
2. Entrar nos assistentes
3. Fazer uma pergunta
4. Executar uma ação

**Atributos relevantes:**

- Quantidade de acessos
- Data do último acesso
- Quantidade de conversas
- Quantidade de execuções

---

## Status das Funcionalidades

| Funcionalidade             | Status     |
| -------------------------- | ---------- |
| Criação de Produtos com IA | Disponível |
| Analista de Produtos       | Beta       |
| Criador de Promoções       | Beta       |
| Operador de Carrinho       | Beta       |
| Assistente de Dados        | Beta       |
| Painel de Oportunidades    | Disponível |
| Segmentação RFV            | Disponível |
| Gestão de Conversas        | Disponível |
