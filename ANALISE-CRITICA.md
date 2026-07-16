# Análise Crítica de Viabilidade — Projeto IRP (ERP de Importações)

> **Documento de revisão independente** — analisa `PROJETO.md` (v1.0) e `ANALISE.md`.
> **Data:** Julho/2026
> **Veredicto:** ✅ **VIÁVEL, com correções obrigatórias antes de codar** — ver [PLANO-IMPLEMENTACAO.md](PLANO-IMPLEMENTACAO.md)

---

## 1. Veredicto Geral

O projeto é **viável**: é um ERP CRUD de pequeno porte, com domínio bem mapeado e escopo autocontido. A remoção da IA (`IA-ESTRATEGIA.md`) foi a decisão de de-risking correta. Um desenvolvedor fullstack sênior entrega o MVP.

**Porém**, a documentação atual tem três categorias de problemas que, se não corrigidos antes do desenvolvimento, geram retrabalho e estouro de prazo:

| Categoria | Gravidade | Resumo |
|-----------|:---------:|--------|
| Inconsistências internas | 🟡 Média | Prazos, horas, custo e auth se contradizem entre seções |
| Lacunas no modelo de dados | 🔴 Alta | Landed cost não flui para o estoque; módulos são ilhas |
| Riscos subestimados | 🟡 Média | Backup fora do MVP, câmbio manual mantido, zero testes |

**Prazo honesto: 8–10 semanas** (não as 4–6 anunciadas). **Custo recalculado: ver §5.**

---

## 2. Pontos Fortes (confirmados)

| # | Ponto | Por quê importa |
|---|-------|-----------------|
| 1 | **Fluxo de 5 fases claro e sequencial** | Vira uma máquina de estados natural no código — baixo risco de arquitetura |
| 2 | **Escopo autocontido e realista** | O cliente sabe que não é SAP; chance real de entrega |
| 3 | **Estratégia de tracking gratuita** | Manual + link shipping line + widget AIS é pragmático: rota China→Brasil leva 30–45 dias, não precisa de polling em tempo real |
| 4 | **Priorização correta do core (M1)** | Os demais módulos orbitam as importações |
| 5 | **Remoção da IA** | Eliminou risco de alucinação em valores fiscais e ~2 semanas de prazo |
| 6 | **Glossário de domínio e wireframes** | Demonstram entendimento real do negócio; reduzem idas e vindas |

---

## 3. Inconsistências Internas (docs se contradizem)

Cada item cita a seção exata onde ocorre. **Todas precisam ser resolvidas em uma única versão do documento antes de assinar contrato/prazo.**

| # | Inconsistência | Onde | Correção sugerida |
|---|----------------|------|-------------------|
| 1 | **Prazo tem 4 valores diferentes**: "4–6 semanas" (PROJETO §8), roadmap de 6 semanas (§9), soma de ~38 dias ≈ 7,5 semanas (§12), "5–6 semanas" (ANALISE §6) | PROJETO §8, §9, §12; ANALISE.md | Adotar a soma real do §12 (38 dias úteis) + buffer → **8–10 semanas** |
| 2 | **Horas vs custo não fecham**: §12 cita "160h", mas 38 dias × 8h = **304h**. A faixa R$ 28–48k foi calculada sobre 160h — o custo real é ~2× o anunciado | PROJETO §12 | Recalcular com premissa única (ver §5 abaixo) |
| 3 | **Auth contraditória**: arquitetura diz "Session (FastAPI) + bcrypt" (§5); stack diz "python-jose (JWT) + bcrypt" (§11) | PROJETO §5 vs §11 | **Sessão + bcrypt.** App é SSR (Jinja2/HTMX) — JWT não tem função sem API externa/mobile |
| 4 | **Resíduo de IA não limpo**: estrutura de módulos ainda lista "Fase 5: Fechamento de Custos (taxas, impostos, **IA**)" | PROJETO §2 | Remover a menção; Fase 5 é conferência manual |
| 5 | **Esforço de PDF diverge 3–5×**: ANALISE estima "1–2 semanas" só para o PDF; PROJETO §12 dá 3 dias para o M5 inteiro (orçamentos + PDF) | ANALISE §5 vs PROJETO §12 | WeasyPrint com template simples: 2–3 dias é factível; layout rico: +1 semana. Definir o nível de acabamento com o cliente |
| 6 | **Fluxo de caixa ambíguo**: listado na estrutura do M4 (§2), mas o escopo do MVP (§8) o joga para Fase 2 | PROJETO §2 vs §8 | Explicitar: MVP entrega só Pagar/Receber; fluxo de caixa projetado é pós-MVP |

---

## 4. Lacunas do Modelo de Dados (o problema mais crítico)

### 4.1 🔴 Landed cost não flui para o estoque

A Fase 5 calcula `custo_total` da importação, mas **não existe rateio de custos por item** nem entrada automática no estoque com custo real. O `custo_medio` do M3 fica desconectado do custo real de importação — o usuário fecharia a importação no M1 e digitaria o custo de novo, à mão, no M3.

**Isso é *o* valor central de um ERP de importações**: saber quanto cada unidade custou de verdade (mercadoria + frete + impostos + taxas). Sem isso, o sistema é um arquivo de documentos com dashboard.

**Correção:** ao concluir a Fase 5, ratear os custos totais entre os itens (proporcional ao valor FOB de cada linha) e oferecer entrada automática no estoque com o custo unitário real ("landed cost").

### 4.2 Demais lacunas

| # | Lacuna | Consequência | Correção |
|---|--------|--------------|----------|
| 1 | `Importacao` é god-entity: campos de container, custos e filepaths inline | Modelo assume **1 container por importação**; importações reais têm N containers | Extrair entidade `Container` (N:1 com `Importacao`) |
| 2 | Anexos como colunas únicas (`pi_filepath`, `bl_filepath`) | Contradiz "múltiplos comprovantes por fase" (§6, relacionamentos) | Entidade `Documento` (importacao_id, fase, tipo, filepath, uploaded_at) |
| 3 | `ContaPagar` sem FK para `Importacao` no diagrama | A mitigação de conciliação citada no §4 ("Vincular ContaPagar à Importacao via FK opcional") não está modelada | Adicionar `importacao_id` nullable em `ContaPagar` |
| 4 | Módulos são ilhas: orçamento aprovado não gera nada | Aprovação no M5 deveria criar `ContaReceber` (e opcionalmente saída de estoque) | Definir os gatilhos de integração entre módulos |
| 5 | Moedas ambíguas | `valor_negociado` em USD, custos da Fase 5 em BRL, `custo_total` sem moeda definida | Campo `moeda` explícito + cotação registrada em cada pagamento; `custo_total` sempre em BRL |

---

## 5. Riscos Subestimados e Decisões Questionáveis

| # | Problema | Análise | Recomendação |
|---|----------|---------|--------------|
| 1 | **Backup fora do MVP** | O próprio doc lista "sem backup" como fraqueza (§3) e mesmo assim o deixa para a Fase 3 (§9) — com dados fiscais e comprovantes de pagamento | Com SQLite, backup é trivial (cópia do arquivo + rotação via cron). **Entra no MVP, semana 1** |
| 2 | **API PTAX adiada para Fase 2** | A matriz de riscos (§10) classifica "câmbio manual gerar erro financeiro" como prob. média / impacto **alto** — e a mitigação é adiada. A API do BCB (Olinda/PTAX) é gratuita, sem chave, ~2h de integração | **Entra no MVP**: cotação automática com override manual |
| 3 | **Widget MarineTraffic tratado como garantido** | Termos de embed gratuito mudam sem aviso; "100 req/dia" não é confirmável; iframe pode quebrar | Tratar como *progressive enhancement*. O compromisso contratual é: tracking manual + link shipping line (que já resolvem o caso de uso) |
| 4 | **Upload sem validação** | Nenhuma menção a validação de tipo/tamanho de arquivo, nem limite de storage | Whitelist de extensões (pdf, jpg, png), limite por arquivo (ex.: 10 MB), nome de arquivo sanitizado |
| 5 | **LGPD ausente** | Sistema guarda CNPJ, contatos e histórico de clientes | Mínimo: política de acesso, criptografia em repouso do banco em produção, cláusula no contrato |
| 6 | **Zero testes automatizados no MVP** | Roadmap só prevê "testes manuais" nas semanas 5–6 — em um sistema financeiro | pytest desde o dia 1; cobertura mínima nos cálculos (câmbio, rateio, custo médio, totais) |
| 7 | **SQLite multiusuário sem WAL** | 2+ usuários simultâneos com uploads podem colidir em write lock | Ativar WAL mode na inicialização; suficiente para equipe pequena |

---

## 6. Recálculo de Esforço e Custo (premissa única)

Premissa: soma do §12 do PROJETO.md está correta (**~38 dias úteis** de desenvolvimento), acrescida das correções desta análise (+4 dias: rateio de custos, PTAX, backup, testes).

| Item | Valor |
|------|-------|
| Esforço de desenvolvimento | ~42 dias úteis ≈ **336 h** |
| Prazo calendário (1 dev fullstack sênior, 100%) | **8 semanas + 2 de buffer = 10 semanas** |
| Custo dev @ R$ 100–150/h (mercado BR, pleno/sênior) | **R$ 33.600 – R$ 50.400** |
| Infra (VPS 6 meses + domínio + SSL) | R$ 700 – R$ 1.400 |
| **Total MVP** | **~R$ 34.000 – R$ 52.000** |

> A faixa original (R$ 28–48k) não estava errada em valor absoluto — estava errada na **base de horas** (160h declaradas vs ~304h somadas). Com a base corrigida, a faixa se sustenta apenas no teto.

---

## 7. Conclusão

| Pergunta | Resposta |
|----------|----------|
| O projeto é viável? | **Sim** — CRUD ERP clássico, domínio claro, stack madura |
| Pode codar direto da spec atual? | **Não** — corrigir modelo de dados e inconsistências primeiro (1 semana de Fase 0) |
| Maior risco técnico | Rateio de custos ↔ estoque mal projetado = retrabalho no coração do sistema |
| Maior risco de negócio | Prazo vendido como 4–6 semanas quando a própria estimativa soma 7,5+ |
| Próximo passo | Executar a **Fase 0** do [PLANO-IMPLEMENTACAO.md](PLANO-IMPLEMENTACAO.md) |
