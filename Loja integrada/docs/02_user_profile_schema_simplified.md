# User Profile Schema - Loja Integrada

**Versão:** 4.0
**Data:** 12 de Março de 2026
**Plataforma:** CleverTap

---

## Filosofia de Simplificação

Este documento adota o princípio **"Events over Properties"**:

- **Segmentação por eventos** é preferível para: datas (first/last time), contadores, histórico
- **Propriedades de perfil** são necessárias apenas para: status atual, configurações ativas, dados que não vêm de eventos

> **Benefício:** Redução de ~95 para ~35 propriedades customizadas. Menor complexidade de sincronização e manutenção.

### Estratégia de Atualização

- **Backend (Data Lake → Airflow DAG):** Maioria das propriedades. Sync diário para todos os usuários da loja.
- **Frontend (CleverTap SDK):** Apenas para propriedades user-level (Komea) e Subscription (dual-write).
- **Dual-Write (Frontend + Backend):** Subscription props atualizadas imediatamente para o usuário ativo via SDK, e para todos os usuários via DAG diário.

> Motivo: Múltiplos usuários por loja. Frontend só atualiza o usuário que executou a ação.
> Para coerência entre usuários, propriedades store-level devem vir exclusivamente do backend.
> Detalhes no documento 06_user_identity_sync_flow.

---

## Reserved Attributes (CleverTap)

Atributos padrão obrigatórios.

| Atributo       | Tipo    | Descrição                      | Obrigatório |
| -------------- | ------- | ------------------------------ | ----------- |
| `Identity`     | String  | ID único do lojista (id_loja) | **Sim**     |
| `Name`         | String  | Nome completo do lojista       | Sim         |
| `Email`        | String  | Email do lojista               | Sim         |
| `Phone`        | String  | Telefone com código país       | Recomendado |
| `MSG-email`    | Boolean | Opt-in para email              | Não         |
| `MSG-push`     | Boolean | Opt-in para push               | Não         |
| `MSG-sms`      | Boolean | Opt-in para SMS                | Não         |
| `MSG-whatsapp` | Boolean | Opt-in para WhatsApp           | Não         |

---

## Atributos de Conta (Cadastro Inicial)

Propriedades definidas uma única vez na criação da conta.

| Atributo       | Tipo   | Descrição        | Evento Origem   | Fonte   |
| -------------- | ------ | ---------------- | --------------- | ------- |
| `id_loja`     | String | ID único da loja | `Loja Criada` | Backend |
| `nome_loja`   | String | Nome da loja     | `Loja Criada` | Backend |
| `tipo_conta` | String | Tipo: pf ou pj   | `Loja Criada` | Backend |
| `estado`        | String | Estado (UF)      | `Loja Criada` | Backend |
| `cidade`         | String | Cidade           | `Loja Criada` | Backend |

> **Nota:** Não há vínculo direto com Business Cases. São dados cadastrais básicos.

---

## Atributos de Plano e Assinatura

**Business Case:** BC4 - Contratação de Planos Pagos

| Atributo             | Tipo    | Descrição                                                       | Evento Origem            | Operação | Fonte              |
| -------------------- | ------- | --------------------------------------------------------------- | ------------------------ | -------- | ------------------ |
| `plano_atual`       | String  | Plano atual: gratuito, crescimento, aceleração, expansão, elite | `Assinatura Concluida` | set      | Frontend + Backend |
| `ciclo_cobranca`      | String  | Ciclo: monthly, annual                                          | `Assinatura Concluida` | set      | Frontend + Backend |
| `cliente_pagante` | Boolean | Cliente pagante                                                 | `Assinatura Concluida` | set      | Frontend + Backend |
| `data_inicio_plano`    | Date    | Data de início do plano atual                                   | `Assinatura Concluida` | set      | Backend            |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade                 | Segmentação CleverTap                                               |
| --------------------------- | ------------------------------------------------------------------- |
| Primeira assinatura paga    | Evento `Assinatura Concluida` com filtro "Did for the first time" |
| Mudou de plano recentemente | Evento `Assinatura Concluida` nos últimos X dias                  |
| Abandonou checkout          | Evento `Assinatura Abandonada` nos últimos X dias                  |
| Usou cupom                  | Evento `Assinatura Concluida` com `codigo_cupom` presente          |
| Total de assinaturas        | Count de eventos `Assinatura Concluida`                           |

---

## Atributos do Enviali

**Business Cases:** BC1 - Configuração de Envio via Enviali, BC2 - Compra de Etiquetas via Enviali

| Atributo                   | Tipo          | Descrição                    | Evento Origem                                                           | Operação | Fonte   |
| -------------------------- | ------------- | ---------------------------- | ----------------------------------------------------------------------- | -------- | ------- |
| `enviali_ativo`           | Boolean       | Enviali ativado na loja      | `Plataforma Envio Ativada` (filtro: `plataforma_envio = "enviali"`) | set      | Backend |
| `metodos_envio_ativos`  | Array[String] | Lista de métodos ativos      | `Metodo Envio Ativado`                                               | append   | Backend |
| `contrato_direto_correios` | Boolean       | Tem contrato direto Correios | `Metodo Envio Ativado` (filtro: `nome_transportadora = "correios"`)         | set      | Backend |
| `saldo_enviali`   | Number        | Valor do saldo atual         | `Saldo Envio Adicionado` (filtro: `plataforma_envio = "enviali"`)      | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade             | Segmentação CleverTap                                                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------ |
| Data de ativação        | Evento `Plataforma Envio Ativada` com filtro `plataforma_envio = "enviali"` e filtro de data |
| Comprou etiqueta        | Evento `Etiqueta Comprada` "Did"                                                                   |
| Primeira etiqueta       | Evento `Etiqueta Comprada` "Did for the first time"                                                |
| Quantidade de etiquetas | Count de eventos `Etiqueta Comprada`                                                               |

---

## Atributos da Loggi

**Business Case:** BC3 - Ativação e Monetização Loggi

| Atributo       | Tipo    | Descrição     | Evento Origem                                                    | Operação | Fonte   |
| -------------- | ------- | ------------- | ---------------------------------------------------------------- | -------- | ------- |
| `loggi_ativa` | Boolean | Loggi ativada | `Metodo Envio Ativado` (filtro: `nome_transportadora = "loggi"`) | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade             | Segmentação CleverTap                                                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Data de ativação        | Evento `Metodo Envio Ativado` com filtro `nome_transportadora = "loggi"` e filtro de data                                              |
| Comprou etiqueta Loggi  | Evento `Etiqueta Comprada` com filtro `nome_transportadora = "loggi"` "Did"                                                                 |
| Ativou mas nunca usou   | Evento `Metodo Envio Ativado` com `nome_transportadora = "loggi"` "Did" AND `Etiqueta Comprada` com `nome_transportadora = "loggi"` "Did not" |
| Quantidade de etiquetas | Count de eventos `Etiqueta Comprada` com filtro `nome_transportadora = "loggi"`                                                             |

---

## Atributos do Pagali

**Business Case:** BC6 - Configurar Meio de Pagamento (Pagali)

| Atributo                     | Tipo          | Descrição                                        | Evento Origem                                                                                  | Operação | Fonte   |
| ---------------------------- | ------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------------- | -------- | ------- |
| `status_conta_pagali`      | String        | Status: approved, rejected, pending, not_started | `Conta Gateway Aprovada` / `Conta Gateway Rejeitada` (filtro: `gateway_pagamento = "pagali"`) | set      | Backend |
| `meios_pagamento_configurados` | Array[String] | Lista de meios configurados                      | `Meio Pagamento Ativado` (filtro: `gateway_pagamento = "pagali"`)                                | append   | Backend |
| `mercado_pago_configurado`    | Boolean       | Mercado Pago configurado (fallback)              | `Cadastro Gateway Concluido` (filtro: `gateway_pagamento = "mercado_pago"`)                  | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade            | Segmentação CleverTap                                                                                                                                      |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Iniciou cadastro       | Evento `Cadastro Gateway Iniciado` com filtro `gateway_pagamento = "pagali"` "Did"                                                                        |
| Data de aprovação      | Evento `Conta Gateway Aprovada` com filtro `gateway_pagamento = "pagali"` e filtro de data                                                                 |
| Abandonou cadastro     | Inaction: `Cadastro Gateway Iniciado` "Did" AND `Cadastro Gateway Concluido` "Did not" (filtro: `gateway_pagamento = "pagali"`) nos últimos X dias |
| Rejeitado por motivo X | Evento `Conta Gateway Rejeitada` com filtro `gateway_pagamento = "pagali"` e `motivo_rejeicao`                                                             |

---

## Atributos de Produtos

**Business Case:** BC5 - Criar Primeiro Produto

| Atributo       | Tipo    | Descrição                | Evento Origem     | Operação | Fonte   |
| -------------- | ------- | ------------------------ | ----------------- | -------- | ------- |
| `tem_produtos` | Boolean | Tem produtos cadastrados | `Produto Criado` | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade            | Segmentação CleverTap                                     |
| ---------------------- | --------------------------------------------------------- |
| Primeiro produto       | Evento `Produto Criado` "Did for the first time"         |
| Criou via IA           | Evento `Produto Criado` com `metodo_criacao = ai_komea` |
| Quantidade de produtos | Count de eventos `Produto Criado`                        |
| Abandonou criação      | Evento `Criacao Produto Abandonada` "Did"                 |

---

## Atributos da Komea

**Business Cases:** BC7 - Personalizar Vitrine (em dev), BC8 - Publicar Site (em dev), BC9 - Copiloto (em dev)

| Atributo                 | Tipo    | Descrição                           | Evento Origem          | Operação  | Fonte    |
| ------------------------ | ------- | ----------------------------------- | ---------------------- | --------- | -------- |
| `site_publicado`         | Boolean | Site publicado (fora de manutenção) | `Komea Site Publicado` | set       | Backend  |
| `komea_qtd_acessos`     | Number  | Contador de acessos à Komea         | `Komea Acessada`       | increment | Frontend |
| `komea_data_ultimo_acesso` | Date    | Data do último acesso à Komea       | `Komea Acessada`       | set       | Frontend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade              | Segmentação CleverTap                        |
| ------------------------ | -------------------------------------------- |
| Acessou Komea            | Evento `Komea Acessada` "Did"                |
| Quantidade de acessos    | Count de eventos `Komea Acessada`            |
| Usou assistente          | Evento `Komea Assistente Acessado` "Did"      |
| Executou ação            | Evento `Komea Acao Executada` "Did"         |
| Personalizou logo/cor    | Evento `Komea Personalizacao Concluida` "Did" |
| Abandonou personalização | Evento `Komea Personalizacao Abandonada` "Did" |

> **Nota:** BC7, BC8 e BC9 estão em desenvolvimento. A propriedade `pagali_left_komea_flow` foi removida pois pode ser segmentada via evento `Komea Saiu Para Painel`.

---

## Atributos do Mercado Livre

**Business Case:** BC11 - Ativação Canal Marketplace

| Atributo                  | Tipo    | Descrição               | Evento Origem                                                     | Operação | Fonte   |
| ------------------------- | ------- | ----------------------- | ----------------------------------------------------------------- | -------- | ------- |
| `mercado_livre_conectado` | Boolean | Mercado Livre conectado | `Marketplace Conectado` (filtro: `marketplace = "mercado_livre"`) | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade                | Segmentação CleverTap                                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------------------------ |
| Data de conexão            | Evento `Marketplace Conectado` com filtro `marketplace = "mercado_livre"` e filtro de data             |
| Enviou anúncio             | Evento `Marketplace Produtos Selecionados` com filtro `marketplace = "mercado_livre"` "Did"                |
| Primeiro anúncio publicado | Evento `Marketplace Anuncio Publicado` com filtro `marketplace = "mercado_livre"` "Did for the first time" |
| Quantidade de anúncios     | Count de eventos `Marketplace Anuncio Publicado` com filtro `marketplace = "mercado_livre"`                 |
| Realizou venda             | Evento `Marketplace Venda Concluida` com filtro `marketplace = "mercado_livre"` "Did"                   |

---

## Atributos Fiscais

**Business Case:** BC10 - Emissão de Nota Fiscal _(especificação pendente)_

| Atributo         | Tipo    | Descrição                                      | Evento Origem             | Operação | Fonte   |
| ---------------- | ------- | ---------------------------------------------- | ------------------------- | -------- | ------- |
| `nfe_configurada` | Boolean | Configuração fiscal concluída                  | `NFe Configuracoes Salvas` | set      | Backend |
| `regime_tributario`     | String  | Regime: simples_nacional, lucro_presumido, mei | `NFe Configuracoes Salvas` | set      | Backend |

### Segmentação via Eventos (não precisam de propriedades)

| Necessidade         | Segmentação CleverTap                         |
| ------------------- | --------------------------------------------- |
| Primeira NF emitida | Evento `NFe Emitida` "Did for the first time" |
| Quantidade de NFs   | Count de eventos `NFe Emitida`                |
| Falha em emissão    | Evento `NFe Emissao Falhou` "Did"            |

---

## Métricas de Negócio

Propriedades agregadas calculadas diariamente pelo Data Lake.

| Atributo          | Tipo   | Descrição                | Operação | Fonte   |
| ----------------- | ------ | ------------------------ | -------- | ------- |
| `gmv_30d`         | Number | GMV dos últimos 30 dias  | set      | Backend |
| `visitas_30d`     | Number | Visitas últimos 30 dias  | set      | Backend |
| `qtde_pedido_30d` | Number | Pedidos últimos 30 dias  | set      | Backend |

> **Nota:** Estas métricas já são calculadas no Data Lake e sincronizadas via DAG diário. Não possuem evento de origem específico.

---

## Resumo: Propriedades Essenciais

Total: **27 propriedades customizadas** (vs. 95+ anteriormente)

### Por Business Case

| BC          | Propriedades                                                                                     | Fonte              | Justificativa                                |
| ----------- | ------------------------------------------------------------------------------------------------ | ------------------ | -------------------------------------------- |
| —           | `id_loja`, `nome_loja`, `tipo_conta`, `estado`, `cidade`                                        | Backend            | Dados cadastrais (sem BC específico)         |
| BC4         | `plano_atual`, `ciclo_cobranca`, `cliente_pagante`, `data_inicio_plano`                         | Frontend + Backend | Status atual do plano precisa ser atualizado |
| BC1/BC2     | `enviali_ativo`, `metodos_envio_ativos`, `contrato_direto_correios`, `saldo_enviali` | Backend            | Estado de configuração e saldo               |
| BC3         | `loggi_ativa`                                                                                   | Backend            | Estado de configuração                       |
| BC6         | `status_conta_pagali`, `meios_pagamento_configurados`, `mercado_pago_configurado`                 | Backend            | Status e meios ativos                        |
| BC5         | `tem_produtos`                                                                                   | Backend            | Flag de ativação básica                      |
| BC7/BC8/BC9 | `site_publicado`, `komea_qtd_acessos`, `komea_data_ultimo_acesso`                                 | Backend / Frontend | Estado do site + métricas user-level         |
| BC11        | `mercado_livre_conectado`                                                                        | Backend            | Estado de conexão                            |
| BC10        | `nfe_configurada`, `regime_tributario`                                                                   | Backend            | Estado de configuração fiscal                |
| —           | `gmv_30d`, `visitas_30d`, `qtde_pedido_30d`                                                      | Backend            | Métricas de negócio agregadas                |

### Propriedades Removidas (segmentáveis via eventos)

As seguintes propriedades do schema anterior foram removidas por serem deriváveis via segmentação de eventos:

- Todas as propriedades `*_date` (first/last) → "Did for the first time" ou filtro de data
- Todas as propriedades `*_count` → Count de eventos
- Propriedades de abandono → Evento de abandono correspondente
- `pagali_left_komea_flow` → Evento `Komea Saiu Para Painel`
- Etapas de jornada → Propriedades do evento correspondente

---

## Operações CleverTap

### set

Sobrescreve o valor anterior.

```javascript
clevertap.profile.push({
  Site: {
    plano_atual: "crescimento",
    cliente_pagante: true,
  },
});
```

### increment

Incrementa valor numérico.

```javascript
clevertap.profile.push({
  Site: {
    komea_qtd_acessos: { $incr: 1 },
  },
});
```

### append

Adiciona item à lista (máx 100 items).

```javascript
clevertap.profile.push({
  Site: {
    metodos_envio_ativos: { $add: ["loggi"] },
    meios_pagamento_configurados: { $add: ["pix"] },
  },
});
```

### remove

Remove item da lista.

```javascript
clevertap.profile.push({
  Site: {
    metodos_envio_ativos: { $remove: ["jadlog"] },
  },
});
```

---

## Exemplos de Segmentação por Eventos

### Exemplo 1: Lojistas que ativaram Enviali mas nunca compraram etiqueta

```
Segment Criteria:
- Event: "Plataforma Envio Ativada" → Did
  - Where: plataforma_envio = "enviali"
- AND Event: "Etiqueta Comprada" → Did not
```

### Exemplo 2: Lojistas com primeira venda no marketplace nos últimos 7 dias

```
Segment Criteria:
- Event: "Marketplace Venda Concluida" → Did for the first time → in the last 7 days
  - Where: marketplace = "mercado_livre"
```

### Exemplo 3: Lojistas que abandonaram cadastro de gateway de pagamento

```
Segment Criteria:
- Event: "Cadastro Gateway Iniciado" → Did
  - Where: gateway_pagamento = "pagali"
- AND Event: "Cadastro Gateway Concluido" → Did not
  - Where: gateway_pagamento = "pagali"
```

### Exemplo 4: Lojistas pagantes que nunca criaram produto

```
Segment Criteria:
- Property: cliente_pagante = true
- AND Event: "Produto Criado" → Did not
```

---

## Limites Técnicos CleverTap

| Item                             | Limite         |
| -------------------------------- | -------------- |
| Custom attribute keys por perfil | 256            |
| Nome do atributo                 | 120 caracteres |
| Valor string                     | 512 caracteres |
| Array                            | 100 items      |

---

## Changelog

| Data       | Versão | Alteração                                                                                               | Autor |
| ---------- | ------ | ------------------------------------------------------------------------------------------------------- | ----- |
| 12/03/2026 | 4.0    | Tradução completa: eventos e propriedades de EN para PT | RMH   |
| 09/02/2026 | 3.0    | Revisão: nomes de eventos genericizados, coluna Fonte, props Komea/métricas, estratégia de sync backend | RMH   |
| 05/01/2026 | 1.0    | Versão inicial com 11 Business Cases                                                                    | RMH   |
