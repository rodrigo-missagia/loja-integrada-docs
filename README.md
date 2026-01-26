# Loja Integrada - Analytics Documentation

Repositório de especificações de analytics para a **Loja Integrada**, plataforma SaaS de e-commerce brasileira.

**Plataforma de Analytics:** CleverTap (Web SDK v1.6.0+)

---

## Índice de Documentos

### Documentação Principal

| Documento | Descrição |
|-----------|-----------|
| [01 - Tracking Plan](Loja%20integrada/docs/01_tracking_plan.md) | Plano de rastreamento com todos os eventos por Business Case |
| [02 - User Profile Schema](Loja%20integrada/docs/02_user_profile_schema.md) | Esquema de atributos do perfil do usuário |
| [02 - User Profile (Simplificado)](Loja%20integrada/docs/02_user_profile_schema_simplified.md) | Versão simplificada do perfil do usuário |
| [03 - Event Specifications](Loja%20integrada/docs/03_event_specifications.md) | Especificações detalhadas de cada evento |
| [04 - Campaign Specifications](Loja%20integrada/docs/04_campaign_specifications.md) | Especificações de campanhas e réguas |
| [05 - Guia de Implementação](Loja%20integrada/docs/05_guia_implementacao.md) | Guia técnico para implementação |
| [06 - User Identity Sync Flow](Loja%20integrada/docs/06_user_identity_sync_flow.md) | Fluxo de sincronização de identidade |

### Knowledge Base

| Documento | Descrição |
|-----------|-----------|
| [Enviali](Loja%20integrada/knowledge/enviali.md) | Documentação do módulo de envio Enviali |
| [Pagali](Loja%20integrada/knowledge/pagali.md) | Documentação do gateway de pagamentos Pagali |
| [Komea](Loja%20integrada/knowledge/komea.md) | Documentação do copiloto de IA Komea |
| [Hub de Canais](Loja%20integrada/knowledge/hub-de-canais.md) | Documentação do Hub de Canais (Mercado Livre) |
| [Funcionalidades](Loja%20integrada/knowledge/funcionalidades.md) | Visão geral das funcionalidades da plataforma |
| [Segmentation Store](Loja%20integrada/knowledge/segmentation-store-optimized.md) | Mapeamento de segmentação otimizado |

### Templates e Referências

| Documento | Descrição |
|-----------|-----------|
| [Event Spec Template](.claude/clevertap-implementation/references/event_spec_template.md) | Template para especificação de eventos |
| [Implementation Template](.claude/clevertap-implementation/references/implementation_template.md) | Template de implementação |
| [SDK Snippets](.claude/clevertap-implementation/references/sdk_snippets.md) | Snippets de código do SDK |
| [User Profile Template](.claude/clevertap-implementation/references/user_profile_template.md) | Template de perfil de usuário |
| [Campaign Template](.claude/clevertap-implementation/references/campaign_template.md) | Template de campanhas |
| [Segmentation Template](.claude/clevertap-implementation/references/segmentation_template.md) | Template de segmentação |
| [Attribute Mapping](.claude/clevertap-implementation/references/attribute_mapping.md) | Mapeamento de atributos |
| [Event Normalization](.claude/clevertap-implementation/references/event_normalization.md) | Normalização de eventos |
| [Vertical Events](.claude/clevertap-implementation/references/vertical_events.md) | Eventos verticais |

---

## Business Cases

| ID | Business Case | Categoria | Objetivo |
|----|---------------|-----------|----------|
| BC1 | Configuração de Envio via Enviali | Logística | Ativar transportadoras e Correios |
| BC2 | Compra de Etiquetas via Enviali | Logística | Monetização via etiquetas |
| BC3 | Ativação e Monetização Loggi | Logística | Uso recorrente da Loggi |
| BC4 | Contratação de Planos Pagos | Monetização | Converter free para paid |
| BC5 | Criar Primeiro Produto | Onboarding | Produto configurado em 7 dias |
| BC6 | Configurar Meio de Pagamento | Onboarding | Ativar Pagali com validação |
| BC7 | Personalizar Vitrine na Komea | Onboarding | Cor e logo configurados |
| BC8 | Publicar Site na Komea | Onboarding | Site fora de manutenção |
| BC9 | Usar Komea como Copiloto | Engajamento | Uso recorrente da Komea |
| BC10 | Emissão de Nota Fiscal | Fiscal | Primeira NF emitida |
| BC11 | Ativação Canal Mercado Livre | Expansão | Anúncios no Mercado Livre |

---

## Convenções

### Nomenclatura de Eventos
- **Padrão:** `Object Action` (ex: "Subscription Started", "Label Purchased")
- **Idioma:** Inglês
- **Eventos de monetização:** Marcados com 💰

### Nomenclatura de Propriedades
- **Formato:** snake_case (ex: `plan_name`, `shipping_carrier`)

---

## Stack Técnica

- **Plataforma:** CleverTap
- **SDK:** Web SDK v1.6.0+
- **Escopo:** SaaS Web (sem mobile nativo)

---

## Arquivos do Projeto

```
loja-integrada/
├── Loja integrada/
│   ├── docs/                    # Documentação principal
│   └── knowledge/               # Base de conhecimento
├── .claude/
│   └── clevertap-implementation/
│       └── references/          # Templates e referências
├── archived/                    # Documentos arquivados
├── CLAUDE.md                    # Instruções para Claude Code
└── TODO.MD                      # Lista de tarefas pendentes
```

---

## Licença

Documentação interna - Loja Integrada
