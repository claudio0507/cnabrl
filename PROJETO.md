# PROJETO IRP — ERP de Importações da China

> **Documento de Projeto — Versão 1.0**  
> **Status:** Planejamento / Brainstorming  
> **Cliente:** [A definir]  
> **Data:** Julho/2026

---

## Sumário

1. [Visão Geral do Negócio](#1-visão-geral-do-negócio)
2. [Análise da Demanda](#2-análise-da-demanda)
3. [Pontos Fortes e Fracos](#3-pontos-fortes-e-fracos)
4. [Matriz de Complexidade](#4-matriz-de-complexidade)
5. [Arquitetura do Sistema](#5-arquitetura-do-sistema)
6. [Modelo de Dados Conceitual](#6-modelo-de-dados-conceitual)
7. [Fluxos de Trabalho](#7-fluxos-de-trabalho)
8. [MVP — Escopo e Definição](#8-mvp--escopo-e-definição)
9. [Roadmap de Implementação](#9-roadmap-de-implementação)
10. [Riscos e Decisões Pendentes](#10-riscos-e-decisões-pendentes)
11. [Stack Tecnológica Recomendada](#11-stack-tecnológica-recomendada)
12. [Estimativas de Esforço](#12-estimativas-de-esforço)

---

## 1. Visão Geral do Negócio

### O Problema
O cliente opera no ramo de **importação de materiais da China** e precisa de um sistema para gerenciar todo o ciclo de vida de uma importação — desde a negociação com o fornecedor até o fechamento dos custos aduaneiros. Atualmente, o controle provavelmente é feito em planilhas, e-mails e WhatsApp, sem rastreabilidade centralizada.

### O Que o Sistema Resolve

| DOR ATUAL | SOLUÇÃO IRP |
|-----------|-------------|
| Acompanhamento de containers descentralizado | Dashboard unificado com status por fase |
| Documentos fiscais (PI, BL, comprovantes) perdidos em e-mails | Repositório central por importação |
| Falta de visibilidade financeira (quanto já pagou, quanto falta) | Totais automáticos de valores pagos vs negociados |
| Erros em conferência de custos aduaneiros | Validação com alertas e registro estruturado |
| Leads e clientes sem follow-up organizado | CRM com funil de vendas |
| Orçamentos manuais sem controle de versão | Geração de proposta com histórico de aprovação |

### Stakeholders do Sistema
- **Importador (usuário principal):** cadastra importações, sobe documentos, acompanha status
- **Financeiro:** registra pagamentos, controla contas a pagar/receber
- **Comercial/CRM:** cadastra clientes, faz follow-up, envia orçamentos
- **Operações/Logística:** atualiza localização de containers, gerencia estoque

---

## 2. Análise da Demanda

### Estrutura dos Módulos

```
ERP IRP
├── M1 — GESTÃO DE IMPORTAÇÕES (CORE)
│   ├── Cadastro de Itens/Produtos
│   ├── Fase 1: Cadastro Inicial (PI, FOB/CIF, fornecedor, itens)
│   ├── Fase 2: Pagamento de Entrada (comprovantes, câmbio)
│   ├── Fase 3: Embarque Internacional (BL, IMO, tracking)
│   ├── Fase 4: Pagamento do Saldo (quitação)
│   ├── Fase 5: Fechamento de Custos (taxas, impostos, IA)
│   └── Dashboard de Containers
│
├── M2 — CRM SIMPLIFICADO
│   ├── Cadastro de Clientes (CNPJ, contatos, histórico)
│   └── Funil de Vendas (6 etapas)
│
├── M3 — CONTROLE DE ESTOQUE
│   ├── Entradas / Saídas / Ajustes
│   ├── Custo médio
│   └── Histórico de movimentações
│
├── M4 — FINANCEIRO
│   ├── Contas a Pagar
│   ├── Contas a Receber
│   └── Fluxo de Caixa
│
└── M5 — ORÇAMENTOS
    ├── Elaboração de propostas
    ├── Geração de PDF
    └── Envio por Email e WhatsApp
```

### O Que Está Claro na Especificação

| Item | Clareza | Observação |
|------|:------:|------------|
| Fluxo das 5 fases da importação | ✅ Alta | Campos bem definidos, sequência lógica |
| Itens com NCM e unidade de medida | ✅ Alta | Padrão de comércio exterior |
| CRM com funil de 6 etapas | ✅ Alta | Etapas bem nomeadas, movimentação manual |
| Controle de estoque | ✅ Alta | CRUD padrão com custo médio |
| Financeiro (Pagar/Receber) | ✅ Alta | Funcionalidades clássicas de ERP |
| Geração de PDF de orçamento | ✅ Média | OK; envio WhatsApp precisa de definição |

### O Que Está Vago ou Subespecificado

| Ponto | Problema | Impacto |
|-------|----------|---------|
| **Tracking em tempo real (Fase 3)** | "Localização atual da embarcação" — como obter? | ✅ Resolvido: tracking híbrido — manual + link direto para shipping line + AIS gratuito |
| **Agente de IA (M2)** | ❌ Removido | Escopo removido — CRM é manual |
| **Validação por IA (Fase 5)** | ❌ Removido | Conferência manual de custos |
| **Envio WhatsApp (M5)** | Exige WhatsApp Business API com aprovação da Meta | Custo e prazo de aprovação |
| **Multitenancy** | Não especificado se é SaaS multi-empresa ou single-tenant | Impacta arquitetura desde o dia 1 |
| **Perfis de usuário** | "Controle de usuários e permissões" sem detalhamento | Precisa definir RBAC: admin, operador, financeiro, etc. |
| **Câmbio** | Campo manual — sem fonte de cotação | Sujeito a erro; ideal integrar API (BCB) |

---

## 3. Pontos Fortes e Fracos

### Pontos Fortes

| # | Ponto | Por Quê |
|---|-------|---------|
| 1 | **Fluxo em fases bem definido** | As 5 fases da importação são claras, sequenciais, com campos objetivos. Isso facilita a UX e o desenvolvimento |
| 2 | **Escopo autocontido e realista** | É um "ERP de pequeno porte" — o cliente sabe que não é um SAP. Chance real de entrega |
| 3 | **Modelo de dados implícito consistente** | As entidades (Importação → Pagamento → Container → Itens) se relacionam naturalmente |
| 4 | **Priorização correta do core** | M1 (Importações) é o coração do negócio. Os demais módulos orbitam em torno dele |
| 5 | **Documentos como parte do fluxo** | Anexos em todas as fases garante rastreabilidade e auditoria |
| 6 | **Baixa complexidade em M3 e M4** | Estoque e Financeiro são CRUDs maduros, com padrões conhecidos |
| 7 | **Cliente tem visão clara do negócio** | Fluxo de importação bem mapeado, cada fase tem propósito definido |

### Pontos Fracos

| # | Ponto | Risco | Sugestão |
|---|-------|-------|----------|
| 1 | **Tracking real-time sem definição de fonte** | Resolvido: estratégia gratuita definida | MVP: tracking manual + link shipping line + AIS gratuito (MarineTraffic tier grátis) |
| 2 | **Agente de IA vago** | ✅ Removido | CRM é 100% manual, sem IA |
| 3 | **Validação IA de documentos aduaneiros** | ✅ Removido | Fase 5 é conferência manual de custos |
| 4 | **WhatsApp sem Business API** | Precisa aprovação Meta, número comercial, custo por mensagem | MVP: gerar link wa.me/ com texto predefinido |
| 5 | **Sem menção a backup/segurança** | Dados fiscais e financeiros críticos | Adicionar requisito de backup diário e criptografia |
| 6 | **NCM sem tabela de referência** | Erros de digitação em NCM | Integrar API da Receita Federal ou tabela estática |
| 7 | **Câmbio 100% manual** | Erro humano em cálculo de custos | Integrar API do Banco Central (grátis, dólar PTAX) |
| 8 | **Sem definição de multitenancy** | Se o sistema crescer para SaaS, retrabalho arquitetural | Decidir agora: single-tenant com possibilidade de multi depois |

---

## 4. Matriz de Complexidade

| Módulo / Funcionalidade | Desenvolvimento | Integração | IA/Dados | TOTAL |
|-------------------------|:--------------:|:----------:|:--------:|:-----:|
| M1 — Fases 1, 2 e 4 (CRUD + upload) | 🟢 Baixa | — | — | 🟢 |
| M1 — Fase 3 (tracking container) | 🟡 Média | 🟢 Baixa (gratuito) | — | 🟡 |
| M1 — Fase 5 (custos) | 🟡 Média | — | — | 🟡 |
| M1 — Dashboard de containers | 🟡 Média | — | — | 🟡 |
| M2 — Cadastro de clientes + funil | 🟢 Baixa | — | — | 🟢 |
| M3 — Controle de estoque | 🟢 Baixa | — | — | 🟢 |
| M4 — Financeiro (Pagar/Receber) | 🟡 Média | — | — | 🟡 |
| M5 — Orçamentos + PDF | 🟡 Média | — | — | 🟡 |
| M5 — Envio Email/WhatsApp | 🟢 Baixa | 🟡 Média | — | 🟡 |
| Infra (auth, upload, deploy) | 🟡 Média | — | — | 🟡 |

### O Que é Difícil (e Por Quê)

| # | Ponto Difícil | Motivo | Como Mitigar |
|---|---------------|--------|--------------|
| 1 | **Tracking real-time de containers** | Resolvido com abordagem híbrida gratuita | MVP: tracking manual + link direto pra shipping line + embed AIS gratuito (MarineTraffic widget) |
| 2 | **Sincronização financeira** | Conciliação entre contas a pagar da importação e contas a pagar gerais (evitar duplicidade) | Vincular `ContaPagar` à `Importacao` via FK opcional |
| 3 | **Geração de PDF estilizado** | Layout de proposta comercial com tabelas, imagens, termos | Usar WeasyPrint ou Playwright para PDF preciso |

## 5. Arquitetura do Sistema

### Diagrama de Alto Nível

```
┌─────────────────────────────────────────────────────────┐
│                     CLIENTE (Browser)                    │
│                  HTML + Bootstrap + HTMX                 │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTPS
┌──────────────────────▼──────────────────────────────────┐
│                   FASTAPI (Python 3.11+)                 │
│                                                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐ │
│  │ M1:      │ │ M2: CRM  │ │ M3:      │ │ M4:        │ │
│  │ Import.  │ │          │ │ Estoque  │ │ Financeiro │ │
│       │            │            │              │         │
│  ┌────┴────────────┴────────────┴──────────────┴──────┐ │
│  │              SQLAlchemy ORM                        │ │
│  └────────────────────────┬───────────────────────────┘ │
│                           │                             │
│  ┌────────────────────────┴───────────────────────────┐ │
│  │              PostgreSQL / SQLite                    │ │
│  └────────────────────────────────────────────────────┘ │
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │         File Storage: /uploads (local ou S3)       │ │
│  └────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘

        ┌─────────────── INTEGRAÇÕES EXTERNAS ───────────────┐
        │                                                    │
        │  🛰 MarineTraffic widget (AIS grátis) — mapa do navio     │
        │  🔗 Shipping lines (MSC, Maersk, COSCO) — link tracking │
        │  💰 API BCB — cotação Dólar PTAX                   │
        │  📧 SMTP/Resend — envio de emails                  │
        │  📱 WhatsApp Business API — envio mensagens        │
        │  📄 WeasyPrint — geração de PDF                    │
        │                                                    │
        └────────────────────────────────────────────────────┘
```

### Decisões Arquiteturais

| Decisão | Escolha | Justificativa |
|---------|---------|---------------|
| **Backend** | FastAPI (Python) | Async, type-safe, documentação automática (Swagger), ecossistema Python maduro |
| **ORM** | SQLAlchemy 2.0 | Migração fácil SQLite → PostgreSQL, migrations com Alembic |
| **Frontend** | Server-Side Rendering (Jinja2 + HTMX) | Sem build step, sem SPA complexa, SEO não é requisito |
| **Estilo** | Bootstrap 5 | Rápido, responsivo, componentes prontos |
| **Banco MVP** | SQLite | Zero configuração, single-file, ideal para MVP local |
| **Banco Produção** | PostgreSQL | Robustez, backups, multi-usuário |
| **Arquivos** | Disco local → S3 | MVP usa filesystem; produção migra para S3 com URL assinada |
| **Auth** | Session (FastAPI) + bcrypt | Simples para MVP; JWT pode vir depois se necessário API mobile |
| **PDF** | WeasyPrint (HTML → PDF) | Templates HTML fáceis de manter, saída precisa |

---

## 6. Modelo de Dados Conceitual

### Entidades Principais

```
┌─────────────┐       ┌──────────────────┐       ┌─────────────┐
│  Fornecedor │       │   IMPORTACAO      │       │    Item     │
├─────────────┤       ├──────────────────┤       ├─────────────┤
│ id          │◄──────│ fornecedor_id     │       │ id          │
│ nome        │  1:N  │ numero (IMP-...)  │       │ codigo      │
│ pais        │       │ modalidade(FOB/CIF)│      │ descricao   │
│ contato     │       │ valor_negociado   │       │ ncm         │
│ email       │       │ fase_atual (1..5) │       │ un_medida   │
└─────────────┘       │ status_container  │       └──────┬──────┘
                      │ pi_filepath       │              │
                      │ bl_filepath       │              │
                      │ container_numero  │              │
                      │ shipping_line     │              │
                      │ imo_number        │       ┌──────▼──────┐
                      │ embarcacao_nome   │       │ImportacaoItem│
                      │ localizacao_atual │       ├─────────────┤
                      │ previsao_chegada  │       │ importacao_id│
                      │ taxas_portuarias  │       │ item_id      │
                      │ impostos          │       │ quantidade   │
                      │ despesas_aduaneiras│      │ vlr_unitario │
                      │ frete_terrestre   │       └─────────────┘
                      │ outras_despesas   │
                      │ custo_total       │
                      └──┬───────┬───────┘
                         │       │
              ┌──────────▼─┐ ┌──▼────────────────┐
              │ Pagamento  │ │MovimentacaoContainer│
              │Importacao  │ ├───────────────────┤
              ├────────────┤ │ importacao_id     │
              │ importacao │ │ data              │
              │ tipo       │ │ status            │
              │ valor_pago │ │ localizacao       │
              │ cotacao    │ │ descricao         │
              │ comprovante│ └───────────────────┘
              └────────────┘
```

### Entidades de Suporte

```
┌──────────┐    ┌───────────────┐    ┌──────────────┐
│  Cliente │    │ ContaPagar    │    │ ContaReceber │
├──────────┤    ├───────────────┤    ├──────────────┤
│ id       │    │ id            │    │ id           │
│ razao    │    │ descricao     │    │ descricao    │
│ cnpj     │    │ valor         │    │ valor        │
│ email    │    │ vencimento    │    │ cliente_id ──┤──► Cliente
│ funil    │    │ status        │    │ vencimento   │
│ historico│    │ comprovante   │    │ status       │
└────┬─────┘    └───────────────┘    └──────────────┘
     │
     │          ┌──────────────┐    ┌──────────────┐
     └──────────│  Orcamento   │    │   Estoque    │
                ├──────────────┤    ├──────────────┤
                │ id           │    │ item_id ─────┤──► Item
                │ cliente_id ──┤    │ qtd_atual    │
                │ numero       │    │ custo_medio  │
                │ valor_total  │    └──────┬───────┘
                │ status       │           │
                └──────┬───────┘    ┌──────▼───────────┐
                       │            │MovimentacaoEstoque│
                ┌──────▼───────┐    ├───────────────────┤
                │ OrcamentoItem│    │ item_id           │
                ├──────────────┤    │ tipo (E/S/Ajuste) │
                │ orcamento_id │    │ quantidade        │
                │ descricao    │    │ motivo            │
                │ qtd          │    │ data              │
                │ vlr_unitario │    └───────────────────┘
                └──────────────┘
```

### Relacionamentos Chave

| Relação | Tipo | Descrição |
|---------|:----:|-----------|
| Fornecedor → Importação | 1:N | Um fornecedor pode ter várias importações |
| Importação → ImportacaoItem | 1:N | Uma importação tem vários itens |
| Item → ImportacaoItem | 1:N | Um item pode estar em várias importações |
| Importação → PagamentoImportacao | 1:N | Múltiplos comprovantes por fase |
| Importação → MovimentacaoContainer | 1:N | Histórico de localizações |
| Cliente → Orcamento | 1:N | Histórico de propostas |
| Cliente → ContaReceber | 1:N | Vários recebíveis por cliente |
| Item → Estoque | 1:1 | Um registro de estoque por item |
| Item → MovimentacaoEstoque | 1:N | Histórico de movimentações |

---

## 7. Fluxos de Trabalho

### 7.1 Fluxo Principal — Importação (5 Fases)

```
INÍCIO
  │
  ▼
┌─────────────────────────────────────────────┐
│ FASE 1: CADASTRO INICIAL                    │
│ • Upload PI (Proforma Invoice)              │
│ • Valor total negociado (USD)               │
│ • Modalidade: FOB ou CIF                    │
│ • Fornecedor + Itens + Quantidades          │
│ → Status: fase1_cadastro                    │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ FASE 2: PAGAMENTO DE ENTRADA               │
│ • Upload comprovante(s)                     │
│ • Valor pago                                │
│ • Tipo de câmbio + Cotação do dia           │
│ • Cálculo automático: BRL e USD             │
│ → Status: fase2_pagamento_entrada           │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ FASE 3: EMBARQUE INTERNACIONAL             │
│ • Upload BL (Bill of Lading)               │
│ • IMO Number / Nome da Embarcação           │
│ • Localização atual (manual/API)            │
│ • Previsão de chegada                       │
│ • Dashboard de tracking                     │
│ → Status: fase3_embarque                    │
│ → Container: em_transito                    │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ FASE 4: PAGAMENTO DO SALDO                 │
│ • Upload comprovante(s)                     │
│ • Valor pago (saldo remanescente)           │
│ • Tipo de câmbio + Cotação                  │
│ • Cálculo automático: BRL e USD             │
│ → Status: fase4_pagamento_saldo             │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ FASE 5: FECHAMENTO DOS CUSTOS              │
│ • Upload doc da assessoria aduaneira        │
│ • Taxas portuárias + Impostos               │
│ • Despesas aduaneiras + Frete terrestre     │
│ • Outras despesas                           │
│ • Custo total calculado                     │
│ • Conferência manual dos valores            │
│ → Status: concluida                         │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
                  FIM
```

#### 📦 Estratégia de Tracking de Containers (GRATUITA)

**Abordagem híbrida em 3 camadas — zero custo:**

| Camada | Fonte | O que entrega | Custo |
|--------|-------|---------------|:-----:|
| **1. Link direto shipping line** | Site da transportadora (MSC, Maersk, COSCO, CMA CGM, Hapag-Lloyd) | Tracking oficial por nº container ou BL | Grátis |
| **2. Widget AIS embed** | MarineTraffic/VesselFinder tier grátis (iframe) | Posição do navio no mapa em tempo real | Grátis |
| **3. Atualização manual** | Usuário informa localização/marco (ex: "Passou Canal do Panamá") | Contexto de negócio + histórico | Grátis |

**Como funciona no sistema:**

```
Fase 3 — Embarque:
  ├── Usuário cadastra: nº container, BL, shipping line, IMO
  ├── Sistema gera link direto: https://www.msc.com/track/?container=XXX
  ├── Sistema embebe widget MarineTraffic com o IMO number
  └── Usuário pode adicionar atualizações manuais (marcos da rota)

Dashboard:
  ├── Mostra última posição conhecida (manual)
  ├── Link "Rastrear no site da transportadora" (1 clique)
  └── Mapa AIS embed (iframe gratuito)
```

**Fontes gratuitas de AIS (para embed/mapa):**

| Fonte | URL | Limitação gratuita |
|-------|-----|-------------------|
| MarineTraffic | `marinetraffic.com` | Widget embed gratuito, 100 req/dia |
| VesselFinder | `vesselfinder.com` | Widget embed gratuito |
| AISHub | `aishub.net` | Dados AIS brutos, API gratuita com rate limit |
| ShipXplorer | `shipxplorer.com` | Widget embed gratuito |

**Por que isso funciona para o caso real (China → Brasil):**

- Rota tem ~30-45 dias de trânsito — não precisa de polling em tempo real
- Shipping lines confiáveis (MSC, Maersk, COSCO) têm tracking público
- Widget AIS gratuito cobre bem as rotas oceânicas principais
- Atualização manual a cada 3-5 dias é suficiente para o negócio

### 7.2 Fluxo — CRM / Funil de Vendas

```
┌──────────────┐    ┌──────────────────┐    ┌─────────────────────┐
│  NOVO LEAD   │───►│ CONTATO REALIZADO │───►│ INTERESSE DEMONSTRADO│
│ (cadastro)   │    │ (1º contato feito)│    │ (respondeu, quer ver)│
└──────────────┘    └──────────────────┘    └──────────┬──────────┘
                                                       │
                                                       ▼
                                            ┌──────────────────┐
                                            │ ORÇAMENTO ENVIADO │
                                            │ (proposta enviada)│
                                            └────────┬─────────┘
                                                     │
                                                     ▼
                                            ┌──────────────┐    ┌────────────────┐
                                            │  NEGOCIAÇÃO   │───►│ CLIENTE FECHADO │
                                            │ (ajustes)     │    │ (fechou negócio)│
                                            └──────┬───────┘    └────────────────┘
                                                   │
                                                   ▼
                                            ┌──────────────┐
                                            │   PERDIDO     │
                                            │ (desistência) │
                                            └──────────────┘
```

### 7.3 Fluxo — Orçamento → Proposta

```
CRIAR ORÇAMENTO
  │
  ├── Selecionar cliente
  ├── Adicionar itens/serviços
  ├── Quantidades e valores
  ├── Cálculo automático do total
  │
  ▼
┌────────────────┐
│   RASCUNHO     │── Editar / Ajustar
└───────┬────────┘
        │ Gerar PDF + Enviar
        ▼
┌────────────────┐
│    ENVIADO     │── Email / WhatsApp
└───────┬────────┘
        │
   ┌────┴────┐
   ▼         ▼
┌──────┐ ┌──────────┐
│APROV.│ │ REJEITADO │
└──────┘ └──────────┘
```

---

## 8. MVP — Escopo e Definição

### O Que ENTRA no MVP

| Módulo | Funcionalidade | Status |
|--------|---------------|:------:|
| **M1** | Cadastro de Itens (código, NCM, descrição, un., fornecedor) | ✅ |
| **M1** | Fase 1: Cadastro Inicial da Importação (PI, FOB/CIF, valor, itens) | ✅ |
| **M1** | Fase 2: Pagamento de Entrada (comprovantes, câmbio manual, BRL/USD) | ✅ |
| **M1** | Fase 3: Embarque (BL, IMO, tracking **híbrido gratuito**: manual + link shipping line + widget AIS) | ✅ |
| **M1** | Fase 4: Pagamento do Saldo (comprovantes, quitação) | ✅ |
| **M1** | Fase 5: Fechamento de Custos (taxas, impostos, conferência manual) | ✅ |
| **M1** | Dashboard de Containers (lista, status, valores, filtros) | ✅ |
| **M2** | Cadastro de Clientes (razão social, CNPJ, contatos) | ✅ |
| **M2** | Funil de Vendas (6 etapas, movimentação manual) | ✅ |
| **M3** | Controle de Estoque (entradas, saídas, ajustes, custo médio) | ✅ |
| **M4** | Contas a Pagar (cadastro, vencimento, baixa, comprovantes) | ✅ |
| **M4** | Contas a Receber (cadastro, vencimento, baixa) | ✅ |
| **M5** | Orçamentos (criação, itens, cálculo, status: rascunho/enviado/aprovado) | ✅ |
| **M5** | Geração de PDF simples (HTML → PDF com WeasyPrint) | ✅ |
| **Infra** | Auth básica (login/senha, sessão) | ✅ |
| **Infra** | Upload de documentos (PI, BL, comprovantes) | ✅ |

### O Que FICA PARA FASE 2

| Funcionalidade | Motivo |
|----------------|--------|
| Tracking automático de containers (API AIS paga) | Substituído por estratégia gratuita: link shipping line + widget AIS embed |
| Validação IA de documentos aduaneiros | ❌ Removido do escopo |
| Agente de IA para CRM (contato automático) | ❌ Removido do escopo |
| Envio WhatsApp (Business API) | Depende de aprovação Meta + número comercial |
| Envio de Email (SMTP/Resend) | Integração simples, mas prioridade menor |
| Fluxo de Caixa projetado | Requer dados históricos para ser útil |
| Multitenancy | Só necessário se confirmado SaaS |
| RBAC avançado (perfis customizados) | MVP usa admin/operador básico |
| API REST externa | MVP é server-side rendering |
| Notificações (email, push) | Fase 2 |
| Logs e auditoria | Fase 2 |

### Por Que Esse MVP?

1. **Entrega valor real**: cobre 100% do fluxo core de importação
2. **Reduz risco**: adia integrações externas caras/incertas (APIs, IA)
3. **Validação rápida**: cliente usa, valida, e priorizamos fase 2 juntos
4. **Baixo custo**: ~4-6 semanas de desenvolvimento para o MVP completo

---

## 9. Roadmap de Implementação

### FASE 1 — MVP (4-6 semanas)

| Semana | Entregável | Descrição |
|:------:|------------|-----------|
| **1** | Setup + Modelo de Dados | Projeto base, SQLAlchemy models, migrations, seeders |
| **1** | Auth + Layout | Login, sessão, template base, sidebar, navegação |
| **2** | M1 — Cadastro de Itens + Fase 1 e 2 | CRUD itens, formulário de nova importação, upload PI, pagamento entrada |
| **2** | M1 — Fase 3 e 4 | Upload BL, tracking manual, pagamento saldo |
| **3** | M1 — Fase 5 + Dashboard | Fechamento de custos, cálculo de totais, dashboard de containers |
| **3** | M3 — Estoque | Entradas, saídas, ajustes, custo médio, histórico |
| **4** | M2 — CRM | Cadastro clientes, funil de vendas, histórico |
| **4** | M4 — Financeiro | Contas a pagar e receber, baixas |
| **5** | M5 — Orçamentos | Criação de proposta, cálculo, PDF |
| **5-6** | Testes + Ajustes | Testes manuais, correções, deploy em staging |

### FASE 2 — Integrações (3-4 semanas)

| Semana | Entregável |
|:------:|------------|
| **7** | Integração API cotação Dólar (BCB) — automático nas fases 2 e 4 |
| **8** | Envio de email (SMTP/Resend) para orçamentos |
| **9** | WhatsApp Business API — envio de orçamentos |
| **10** | Melhorias UX com base no feedback do MVP |

### FASE 3 — Robustez e Escala (3-4 semanas)

| Semana | Entregável |
|:------:|------------|
| **11-12** | Migração SQLite → PostgreSQL |
| **13** | Multitenancy (se confirmado) |
| **14** | RBAC avançado, logs de auditoria, backup automático |
| **15** | Testes automatizados, documentação |

---

## 10. Riscos e Decisões Pendentes

### Matriz de Riscos

| Risco | Prob. | Impacto | Mitigação |
|-------|:-----:|:-------:|-----------|
| API de tracking muito cara ou indisponível | ✅ Resolvido | Baixo | Estratégia gratuita cobre 100%: link shipping line + widget AIS embed |
| WhatsApp Business API rejeitada pela Meta | Média | Médio | Fallback: envio por link wa.me/ + email |
| Cliente expandir escopo durante MVP | Alta | Alto | Contrato claro com milestone-based delivery |
| Falta de tabela NCM oficial | Baixa | Baixo | Usar tabela estática (disponível online); campo texto livre no MVP |
| Performance com muitos uploads | Baixa | Médio | MVP usa filesystem; migrar para S3 quando > 1000 documentos |
| Câmbio manual gerar erro financeiro | Média | Alto | Integrar API BCB na Fase 2 (prioridade alta) |

### Decisões Pendentes (Precisa Alinhar com Cliente)

| # | Decisão | Opções | Recomendação |
|---|---------|--------|--------------|
| 1 | **Multitenancy?** | (A) Single-tenant (1 empresa) / (B) Multi-tenant (SaaS) | **(A)** para MVP; arquitetura permite migrar para (B) |
| 2 | **Tracking container?** | ✅ Definido: híbrido gratuito (manual + link shipping line + widget AIS) | Sem custo adicional |
| 3 | **Agente IA do CRM?** | ❌ Removido — CRM manual | Simplifica escopo |
| 4 | **Envio orçamento?** | (A) PDF + link / (B) Email direto / (C) WhatsApp | **(A)** MVP; **(B)** e **(C)** Fase 2 |
| 5 | **Hospedagem?** | (A) VPS própria / (B) Cloud (AWS/Railway) / (C) On-premise | **(A)** ou **(B)** — depende da infra do cliente |
| 6 | **Domínio e identidade?** | Nome do sistema, logo, paleta de cores | Pendente — cliente precisa definir |

---

## 11. Stack Tecnológica Recomendada

| Camada | Tecnologia | Por Quê |
|--------|-----------|---------|
| **Linguagem** | Python 3.11+ | Ecossistema maduro, curva baixa, bibliotecas para tudo |
| **Framework Web** | FastAPI | Async, validação Pydantic, Swagger auto, performance |
| **ORM** | SQLAlchemy 2.0 + Alembic | Migrations, suporte a SQLite e PostgreSQL |
| **Banco MVP** | SQLite | Zero config, arquivo único |
| **Banco Prod** | PostgreSQL 16 | Robustez, backups, concorrência |
| **Frontend** | Jinja2 + HTMX + Bootstrap 5 | Sem build step, SSR, responsivo, rápido |
| **PDF** | WeasyPrint | HTML → PDF de alta qualidade |
| **Auth** | python-jose (JWT) + bcrypt | Stateless, escalável |
| **Upload** | python-multipart + aiofiles | Upload async |
| **Tracking** | Widget iframe MarineTraffic + links shipping lines | Grátis, sem API key |
| **Email (Fase 2)** | Resend API | Moderno, entrega boa, preço justo |
| **Deploy** | Docker + Nginx | Containerização, proxy reverso, SSL com Let's Encrypt |

---

## 12. Estimativas de Esforço

### MVP (Fase 1)

| Atividade | Dias | Perfil |
|-----------|:----:|--------|
| Setup do projeto (FastAPI, SQLAlchemy, templates) | 2 | Fullstack |
| Modelagem do banco + Alembic | 2 | Backend |
| Auth (login, sessão, middleware) | 2 | Backend |
| Layout base + navegação | 2 | Frontend |
| M1 — Itens (CRUD) | 2 | Fullstack |
| M1 — Importações Fase 1 + 2 | 3 | Fullstack |
| M1 — Importações Fase 3 + 4 | 3 | Fullstack |
| M1 — Importações Fase 5 + Dashboard containers | 4 | Fullstack |
| M3 — Estoque | 3 | Fullstack |
| M2 — CRM (clientes + funil) | 3 | Fullstack |
| M4 — Financeiro (pagar/receber) | 3 | Fullstack |
| M5 — Orçamentos + PDF | 3 | Fullstack |
| Testes manuais + ajustes | 5 | QA/Fullstack |
| Deploy staging | 1 | DevOps |
| **TOTAL** | **~38 dias** | **~7.5 semanas** |

### Time Necessário

| Perfil | Alocação |
|--------|----------|
| 1 Desenvolvedor Fullstack Senior | 100% (7-8 semanas) |
| 1 QA / Tester | 30% (últimas 2 semanas) |

### Custo Estimado (Referência Brasil — Jul/2026)

| Item | Valor |
|------|-------|
| Desenvolvimento MVP (160h) | R$ 28.000 – R$ 48.000 |
| Infra (VPS 6 meses) | R$ 600 – R$ 1.200 |
| Domínio + SSL | R$ 80 – R$ 200 |
| APIs (Fase 2) | R$ 200 – R$ 800/mês |
| **TOTAL MVP** | **~R$ 30.000 – R$ 50.000** |

---

## Apêndice A — Glossário do Domínio

| Sigla/Termo | Significado |
|-------------|-------------|
| **PI** | Proforma Invoice — fatura proforma, documento inicial da negociação |
| **BL** | Bill of Lading — conhecimento de embarque marítimo |
| **FOB** | Free On Board — vendedor entrega no porto de origem; comprador paga frete e seguro |
| **CIF** | Cost, Insurance and Freight — vendedor paga custo, seguro e frete até o destino |
| **IMO Number** | International Maritime Organization — número único de identificação da embarcação |
| **NCM** | Nomenclatura Comum do Mercosul — código fiscal do produto (8 dígitos) |
| **AIS** | Automatic Identification System — sistema de rastreamento de navios |
| **PTAX** | Taxa de câmbio divulgada pelo Banco Central do Brasil |
| **Drawback** | Regime aduaneiro especial de suspensão/imunidade de tributos (não mencionado, mas relevante) |

---

## Apêndice B — Telas Conceituais (Wireframes)

### Tela 1: Dashboard Principal

```
┌──────────────────────────────────────────────────────────────┐
│  IRP — ERP Importações                          [Admin ▼] [Sair] │
├──────────┬───────────────────────────────────────────────────────┤
│ ⚡ Dash   │                                                       │
│ 🚚 Import  │   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────┐  │
│ 📦 Itens   │   │ 12 Imp. │ │ 45 Itens│ │ 30 Cli. │ │R$ 15k Pag│  │
│ 👥 CRM     │   └─────────┘ └─────────┘ └─────────┘ └──────────┘  │
│ 📊 Estoque │                                                       │
│ 💰 Financ. │   ┌─── IMPORTAÇÕES POR FASE ───┐ ┌── ÚLTIMAS ──────┐ │
│ 📄 Orçam.  │   │ Fase 1: ████████ 8        │ │ IMP-2026-0012   │ │
│            │   │ Fase 2: ████ 4           │ │  FOB · Em trânsito│ │
│            │   │ Fase 3: ██ 2             │ │ IMP-2026-0011   │ │
│            │   │ Fase 4: ██ 2             │ │  CIF · Chegou    │ │
│            │   │ Fase 5: ██ 2             │ │ ...              │ │
│            │   │ Concluída: ██████ 6       │ │                  │ │
│            │   └────────────────────────────┘ └─────────────────┘ │
└──────────┴───────────────────────────────────────────────────────┘
```

### Tela 2: Detalhe da Importação (Visão por Fase)

```
┌──────────────────────────────────────────────────────────────┐
│  ← Voltar    IMP-2026-0012    FOB    $ 45,000.00              │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  ● FASE 1: Cadastro Inicial ✅ (concluída)                     │
│  │  PI: invoice_mar2026.pdf                                   │
│  │  Itens: 3 itens · 1500 unidades                            │
│  │                                                             │
│  ● FASE 2: Pagamento de Entrada ✅ (concluída)                  │
│  │  Valor: $ 20,000 · R$ 112,400 · Dólar R$ 5.62              │
│  │  Comprovante: pgto_entrada_01.pdf                           │
│  │                                                             │
│  ◉ FASE 3: Embarque 🔵 (atual — em andamento)                   │
│  │  ┌─────────────────────────────────────────┐                │
│  │  │ BL: bl_xyz.pdf                          │                │
│  │  │ Embarcação: MSC ARIES · IMO: 9785439    │                │
│  │  │ Container: MSCU1234567                  │                │
│  │  │ Shipping Line: MSC                      │                │
│  │  │                                         │                │
│  │  │ 📍 RASTREAMENTO:                        │                │
│  │  │  ┌──────────────────────────────────┐   │                │
│  │  │  │     [MAPA AIS EMBED GRÁTIS]      │   │                │
│  │  │  │   (MarineTraffic widget iframe)  │   │                │
│  │  │  └──────────────────────────────────┘   │                │
│  │  │  🔗 Rastrear no site da MSC →           │                │
│  │  │                                         │                │
│  │  │  Histórico (manual):                    │                │
│  │  │   12/07 ─ Embarque em Xiamen            │                │
│  │  │   14/07 ─ Em trânsito — Estreito Malaca │                │
│  │  │   [Atualizar Localização]               │                │
│  │  └─────────────────────────────────────────┘                │
│  │                                                             │
│  ○ FASE 4: Pagamento do Saldo (bloqueada)                      │
│  ○ FASE 5: Fechamento de Custos (bloqueada)                    │
│                                                                │
├──────────────────────────────────────────────────────────────┤
│  RESUMO FINANCEIRO:                                           │
│  Valor Negociado: $ 45,000  |  Pago: $ 20,000  |  Saldo: $ 25,000 │
└──────────────────────────────────────────────────────────────┘
```

---

> **Próximo Passo:** Validar este documento com o cliente, resolver as [decisões pendentes](#10-riscos-e-decisões-pendentes), e iniciar o desenvolvimento do MVP conforme [roadmap](#9-roadmap-de-implementação).
