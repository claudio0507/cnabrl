# Plano de Implementação Revisado — Projeto IRP (ERP de Importações)

> **Substitui o roadmap do PROJETO.md §9**, incorporando as correções da [ANALISE-CRITICA.md](ANALISE-CRITICA.md).
> **Stack:** FastAPI + SQLAlchemy 2.0 + Alembic + Jinja2/HTMX/Bootstrap 5 + SQLite (WAL) → PostgreSQL
> **Prazo:** 8 semanas de desenvolvimento + 2 de buffer = **10 semanas**
> **Time:** 1 dev fullstack sênior (100%) + QA 30% nas 2 últimas semanas

---

## Princípios que corrigem a spec original

1. **Auth por sessão + bcrypt** (não JWT) — app é SSR; JWT só se surgir API externa/mobile
2. **pytest desde o dia 1** — cobertura obrigatória nos cálculos financeiros (câmbio, rateio, custo médio, totais)
3. **Backup diário no MVP** — cópia do arquivo SQLite + rotação de 30 dias, configurado na semana 1
4. **PTAX automática no MVP** — API BCB/Olinda gratuita, com override manual
5. **Landed cost integra M1 → M3** — rateio de custos por item na Fase 5 alimenta o estoque
6. **Widget AIS é enhancement, não compromisso** — contratualmente o tracking é manual + link shipping line
7. **Single-tenant** — arquitetura permite multi depois; decisão registrada, não postergada

---

## Fase 0 — Fundação (Semana 1)

### Decisões a fechar com o cliente ANTES de codar
- [ ] Confirmar single-tenant (1 empresa)
- [ ] Hospedagem: VPS própria vs cloud (Railway/AWS)
- [ ] Nível de acabamento do PDF de orçamento (simples = 2–3 dias; rico = +1 semana)
- [ ] Identidade visual (nome, logo, cores) — pode ser placeholder no MVP

### Modelo de dados corrigido

Mudanças em relação ao diagrama do PROJETO.md §6:

```
Importacao (enxugada)
 ├── fornecedor_id, numero, modalidade (FOB/CIF), moeda, valor_negociado
 ├── fase_atual (1..5), status
 └── custo_total_brl (calculado na Fase 5)

Container (NOVO — N:1 com Importacao)          ◄── corrige god-entity
 ├── importacao_id, numero, shipping_line
 ├── imo_number, embarcacao_nome
 └── previsao_chegada

MovimentacaoContainer → aponta para Container (não para Importacao)

Documento (NOVO — substitui pi_filepath/bl_filepath)
 ├── importacao_id, fase (1..5), tipo (PI|BL|COMPROVANTE|ADUANA|OUTRO)
 └── filepath, nome_original, tamanho, uploaded_at, uploaded_by

PagamentoImportacao
 ├── importacao_id, fase (2|4), valor_moeda, moeda
 ├── cotacao (PTAX do dia, editável), valor_brl (calculado)
 └── data_pagamento

CustoImportacaoItem (NOVO — resultado do rateio da Fase 5)
 ├── importacao_id, item_id
 ├── custo_unitario_brl (landed cost)
 └── criterio_rateio (valor_fob)

ContaPagar
 └── + importacao_id NULLABLE                   ◄── conciliação modelada

Orcamento
 └── aprovação gera ContaReceber (gatilho de integração)
```

### Setup técnico
- [ ] Projeto FastAPI: estrutura por módulos (`app/m1_importacoes/`, `app/m2_crm/`, ...)
- [ ] SQLAlchemy 2.0 + Alembic (migration inicial com o modelo corrigido)
- [ ] SQLite com `PRAGMA journal_mode=WAL` na inicialização
- [ ] Jinja2 + HTMX + Bootstrap 5: template base, sidebar, navegação
- [ ] Auth: login/senha, sessão server-side, bcrypt, middleware de proteção
- [ ] Upload: whitelist (pdf/jpg/png), limite 10 MB, nomes sanitizados, `/uploads` fora do webroot
- [ ] pytest + fixtures de banco em memória; CI simples (GitHub Actions: lint + testes)
- [ ] Script de backup diário (cron: cópia SQLite + rotação 30 dias)

**Critério de saída:** login funciona, migration roda, 1 teste passa no CI, backup agendado.

---

## Fase 1 — Core M1: Importações (Semanas 2–4)

| Entrega | Detalhe | Dias |
|---------|---------|:----:|
| CRUD Itens | código, descrição, NCM com **validação soft contra tabela estática** (importada de fonte oficial), unidade de medida | 2 |
| Fase 1 — Cadastro | PI (via `Documento`), FOB/CIF, moeda, valor, fornecedor, itens+quantidades | 2 |
| Fase 2 — Pagamento entrada | comprovantes múltiplos, **cotação PTAX automática (API BCB) + override manual**, cálculo BRL/USD | 2 |
| Fase 3 — Embarque | BL, entidade `Container` (N por importação), tracking manual + link gerado para shipping line; widget AIS como enhancement | 3 |
| Fase 4 — Pagamento saldo | idem Fase 2; validação de quitação (Σ pagamentos ≥ valor negociado) | 1 |
| Fase 5 — Fechamento | taxas, impostos, despesas; **rateio por item proporcional ao valor FOB → `CustoImportacaoItem`**; custo total BRL | 3 |
| Máquina de estados | transições validadas (não pula fase; reabertura só por admin) | 1 |
| Dashboard containers | lista, filtros por fase/status, totais pagos vs negociados | 2 |

**Testes obrigatórios:** conversão de câmbio, rateio de custos (soma dos rateios = custo total), transições de fase, quitação.

**Critério de saída:** uma importação completa percorre as 5 fases com documentos e o custo unitário real de cada item é exibido.

---

## Fase 2 — Módulos Satélites (Semanas 5–6)

| Entrega | Detalhe | Dias |
|---------|---------|:----:|
| M3 — Estoque | entradas/saídas/ajustes, custo médio ponderado, histórico; **ao concluir Fase 5, oferecer entrada automática com landed cost** | 3 |
| M4 — Financeiro | contas a pagar (com `importacao_id` opcional — evita duplicidade na conciliação) e a receber; baixa com comprovante | 3 |
| M2 — CRM | cadastro clientes (razão, CNPJ com validação de dígito, contatos), funil 6 etapas com movimentação manual, histórico | 3 |

**Testes obrigatórios:** custo médio após entrada com landed cost; baixa de conta; unicidade de CNPJ.

**Critério de saída:** fechar uma importação alimenta o estoque com custo real; conta a pagar vinculada não duplica.

---

## Fase 3 — Orçamentos e Fechamento (Semanas 7–8)

| Entrega | Detalhe | Dias |
|---------|---------|:----:|
| M5 — Orçamentos | criação, itens, cálculo automático, status (rascunho/enviado/aprovado/rejeitado) | 2 |
| PDF | WeasyPrint, template HTML do nível acordado na Fase 0; botão "gerar link wa.me/ com texto" (WhatsApp sem API) | 2 |
| Integração M5→M4 | aprovação de orçamento gera `ContaReceber` | 1 |
| Testes E2E | fluxo completo: importação 5 fases → estoque → orçamento → aprovação → conta a receber | 2 |
| Deploy staging | Docker + Nginx + Let's Encrypt; backup validado em produção | 1 |
| UAT com cliente | roteiro de aceite por módulo; correções | 2 |

**Critério de saída:** cliente executa o roteiro de UAT sem bloqueios; staging com backup funcionando.

---

## Semanas 9–10 — Buffer

Reservadas para: correções de UAT, ajustes de UX, imprevistos. **Se não usadas, antecipam o Pós-MVP.** Não vender essas semanas como prazo de features.

---

## Pós-MVP (Produto Fase 2 — priorizar com o cliente após uso real)

| Item | Gatilho para priorizar |
|------|------------------------|
| Envio de email (Resend/SMTP) para orçamentos | Cliente enviando > N orçamentos/semana |
| WhatsApp Business API | Aprovação Meta + volume justificar custo por mensagem |
| Migração SQLite → PostgreSQL | 2ª empresa (multi-tenant) OU > ~5 usuários simultâneos |
| RBAC avançado (perfis além de admin/operador) | Equipe do cliente crescer |
| Fluxo de caixa projetado | ≥ 3 meses de dados históricos no sistema |
| Logs de auditoria | Requisito do contador/fiscal do cliente |
| Multitenancy | Decisão de virar SaaS |

---

## Resumo de Esforço

| Fase | Semanas | Dias úteis |
|------|:-------:|:----------:|
| Fase 0 — Fundação | 1 | 5 |
| Fase 1 — Core M1 | 2–4 | 16 |
| Fase 2 — Satélites | 5–6 | 9 |
| Fase 3 — Fechamento | 7–8 | 10 |
| Buffer | 9–10 | (reserva) |
| **Total** | **10** | **~40 dias ≈ 320 h** |

Custo estimado @ R$ 100–150/h: **R$ 32.000 – R$ 48.000** (dev) + R$ 700–1.400 (infra 6 meses).

---

> **Próximo passo:** validar este plano e a [ANALISE-CRITICA.md](ANALISE-CRITICA.md) com o cliente, fechar as decisões da Fase 0 e iniciar o desenvolvimento.
