# Análise de Especificação — ERP de Importações (IRP)

## 1. Visão Geral

Sistema ERP de pequeno porte focado em gestão de importações da China. 5 módulos: Importações (core), CRM, Estoque, Financeiro, Orçamentos. **Zero IA** — todas as funcionalidades são manuais/CRUD.

---

## 2. Análise por Módulo

| Módulo | Complexidade | Viabilidade | Riscos |
|--------|:-----------:|:-----------:|--------|
| **M1 - Importações** | 🟡 MÉDIA | ✅ Viável | Fase 3 (tracking) resolvido com estratégia gratuita |
| **M2 - CRM** | 🟢 BAIXA | ✅ Viável | CRUD clientes + funil manual |
| **M3 - Estoque** | 🟢 BAIXA | ✅ Viável | CRUD padrão, baixo risco |
| **M4 - Financeiro** | 🟡 MÉDIA | ✅ Viável | Complexidade em conciliação |
| **M5 - Orçamentos** | 🟡 MÉDIA | ✅ Viável | Geração de PDF e envio WhatsApp exigem integrações |

---

## 3. Pontos Fortes

| # | Ponto | Impacto |
|---|-------|---------|
| 1 | **Fluxo em fases bem definido** — 5 fases claras com campos objetivos | Facilita implementação e UX |
| 2 | **Escopo autocontido** — ERP pequeno, sem megalomania | Chance real de entrega |
| 3 | **Modelo de dados implícito consistente** — entidades relacionadas naturalmente | Menos retrabalho de arquitetura |
| 4 | **Priorização correta** — M1 (core) é o coração do negócio | MVP pode focar no que gera valor |
| 5 | **Documentação como parte do fluxo** — anexos em todas as fases | Rastreabilidade e auditoria |
| 6 | **Cliente tem visão clara do negócio** — fluxo de importação bem mapeado | Menos idas e vindas |

---

## 4. Pontos Fracos & Riscos

| # | Ponto | Risco | Mitigação |
|---|-------|-------|-----------|
| 1 | **Tracking de containers (Fase 3)** | ✅ Resolvido | Estratégia gratuita: manual + link shipping line + widget AIS |
| 2 | **Envio WhatsApp (M5)** | Exige WhatsApp Business API (aprovação Meta, custo) | MVP: link wa.me/; API Fase 2 |
| 3 | **Sem definição de multitenancy** | Se for SaaS, exige isolamento de dados | Esclarecer com cliente |
| 4 | **Sem requisitos de backup/segurança** | Dados financeiros e documentos fiscais | Adicionar no escopo |
| 5 | **NCM sem validação** | Código NCM sem tabela de referência | Integrar tabela NCM oficial |
| 6 | **Câmbio manual (Fases 2 e 4)** | Erro humano | Integrar API BCB na Fase 2 |

---

## 5. Pontos Mais Difíceis

| # | Ponto | Por quê | Esforço |
|---|-------|---------|:------:|
| 1 | **Geração de PDF estilizado** | Layout com tabelas, imagens, totais | 1-2 sem |
| 2 | **Conciliação financeira** | Contas a pagar da importação vs contas gerais | 2 sem |
| 3 | **Fluxo de caixa real-time** | Saldo projetado com base em vencimentos | 2-3 sem |

---

## 6. Estratégia de MVP

### FASE MVP

```
✅ M1 — Gestão de Importações (5 fases completas)
│   ├── Cadastro de Itens/Produtos
│   ├── Fase 1: Cadastro Inicial (PI, valores, FOB/CIF, fornecedor)
│   ├── Fase 2: Pagamento de Entrada (comprovantes, câmbio)
│   ├── Fase 3: Embarque (BL, IMO, tracking híbrido gratuito)
│   ├── Fase 4: Pagamento do Saldo
│   ├── Fase 5: Fechamento de Custos (conferência manual)
│   └── Dashboard de Containers

✅ M2 — CRM (cadastro clientes + funil manual 6 etapas)
✅ M3 — Controle de Estoque (entradas/saídas/ajustes, custo médio)
✅ M4 — Financeiro (contas a pagar/receber)
✅ M5 — Orçamentos (criação + PDF, sem WhatsApp)
```

### O que fica pra Fase 2

| Funcionalidade | Motivo |
|----------------|--------|
| API cotação Dólar (BCB) | Integração simples, baixa prioridade |
| Envio de email (orçamentos) | Precisa configurar SMTP/Resend |
| WhatsApp Business API | Depende de aprovação Meta |
| PostgreSQL | SQLite suficiente para MVP |

---

## 7. Stack Técnica

| Camada | Tecnologia | Justificativa |
|--------|-----------|---------------|
| Backend | FastAPI (Python 3) | Rápido, async, type-safe |
| ORM | SQLAlchemy + SQLite | MVP simples; migrar para PostgreSQL depois |
| Frontend | Jinja2 + HTMX + Bootstrap 5 | SSR sem build step |
| File Storage | Local /uploads | MVP; migrar para S3 depois |
| Auth | Session-based | Simples |
| Tracking | Widget iframe MarineTraffic + links shipping lines | Grátis, zero API key |
| PDF | WeasyPrint | HTML → PDF |

---

## 8. Recomendações

| # | Recomendação |
|---|-------------|
| 1 | **Tracking**: estratégia gratuita definida — link shipping line + widget AIS embed |
| 2 | **NCM**: integrar tabela oficial para validação |
| 3 | **Cotação**: integrar API BCB (grátis) na Fase 2 para câmbio automático |
| 4 | **Backup**: definir política antes de produção |
| 5 | **Multitenancy**: decidir antes de codar (single-tenant MVP) |
