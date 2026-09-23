# 07 — Perfil operacional (caso real: ESW Incorporadora)

Consolidação do **questionário preenchido** por um cliente potencial, usado para
(a) validar/refinar os requisitos e (b) derivar a **configuração inicial de
parâmetros**. Referencia o questionário ([`06`](06-questionario-configuracao.md)),
o modelo de parâmetros ([`PAR`, §3.1](01-requisitos.md#31-princípio-transversal--parametrização-hierárquica-com-herança))
e o motor de cálculo ([`05`](05-motor-calculo-financeiro.md)).

- **Empresa:** ESW Incorporadora Ltda.
- **Respondente:** Santana Pereira de Carvalho (Coordenadora) — contato em arquivo.
- **Data:** 22/09/2026

---

## 1. Resumo executivo do perfil

- Operação **madura, focada em administração de carteira**: ~6 empreendimentos
  (quase todos já vendidos), **~215 contratos ativos**, **≤10 vendas/mês**.
- **2 usuários** (área administrativa); hoje sem ERP integrado.
- Venda predominante: **parcelado direto (carteira própria)** via **boletos/carnês**
  (Bradesco), enviados por e-mail; também **à vista** e **permuta/dação**. **Não**
  usa financiamento bancário.
- Modelo jurídico: **condomínio / fração ideal**.
- Regras **uniformes** entre empreendimentos e dentro deles (pouca variação).

> **Implicação de escopo para este cliente:** o valor está em **recebíveis /
> cobrança / gestão de carteira**, não no funil de vendas nem no mapa. Para a ESW,
> o MVP deve priorizar **Financeiro (cobrança, boletos, carnê, inadimplência,
> distrato, quitação)**, deixando CRM, mapa e portais para depois. A **migração
> dos contratos existentes** fica para uma das últimas etapas (Fase 7 —
> Implantação, no [roadmap](04-backlog-e-roadmap.md#3-roadmap-sugerido-em-fases)).

## 2. Respostas consolidadas por tema

| Tema | Resposta da ESW |
|------|-----------------|
| **Formas de pagamento** | Parcelado próprio (principal), à vista, permuta/dação. Sem banco. |
| **Entrada** | **6% a 10%** do valor da venda; parcelável em **até 4×**. |
| **Prazo** | Restante em **até 120×** (mais comum). Sem balões/reforços; sem parcela final. |
| **Juros / correção** | **Reajuste anual** (a cada 12 meses, a partir do 1º ano): **IGP-M acumulado (12m) + 1% contratual**. Parcela **fixa no valor do mês** entre reajustes. **Uma vez por ano.** *(ver §4 — diverge da premissa anterior)* |
| **Multa/atraso** | Multa **2%** + juros de mora **1% a.m.**, cobrados **a partir de 1 dia**. Sem aviso prévio; cobrança por WhatsApp/e-mail. |
| **Antecipação** | Antecipar parcelas: **sem desconto**. **Quitação total: desconto padrão 5%** (pode aumentar). |
| **Distrato** | Acontece “às vezes”; **devolvem 70%** (retêm 30%); **devolução parcelada** conforme nº de parcelas pagas. |
| **Comissão** | **50% do valor da entrada**; igual em todos os empreendimentos; dividida **empresa/corretor**; paga **conforme o cliente paga**. Vendem imobiliária própria + parceiros. |
| **Repasse ao dono da terra** | Terra **própria** (sem repasse). Em **parcerias**, há repasse **variável**. |
| **Flexibilidade** | Regras **iguais** entre e dentro dos empreendimentos. Raramente negociam fora da tabela. Já editaram contratos assinados **apenas para corrigir dados digitados**. |
| **Preço** | **Tabela pronta**; valoriza por **tamanho**; reajuste de tabela **sem periodicidade fixa** (conforme valorização). |
| **Contrato** | **1 modelo padrão**; assinatura **papel + digital**. |
| **Ferramentas / captação** | Sem ERP. Captação: **redes sociais, indicação, “corpo a corpo”**. |
| **Portais** | Cliente: “seria bom, não urgente”. Corretor: **não precisa**. |

## 3. Configuração inicial de parâmetros sugerida (ESW)

Valores a aplicar no nível **Geral** (a operação é uniforme; poucos overrides).

| Parâmetro | Valor (ESW) | Nível |
|-----------|-------------|-------|
| Formas de pagamento habilitadas | Próprio, À vista, Permuta/Dação (banco **off**) | Geral |
| Modelo jurídico | Condomínio (fração ideal) | Geral/Empreendimento |
| Entrada mínima | 6%–10% (faixa) | Geral |
| Entrada parcelável | Sim, até 4× | Geral |
| Prazo de parcelamento | até 120× (padrão) | Geral |
| Método de reajuste | **Reajuste anual = IGP-M (12m) + 1% aditivo** | Geral |
| Índice de correção | IGP-M | Geral |
| Carência de correção | 12 meses | Geral |
| Periodicidade do reajuste | Anual (aniversário) | Geral |
| Juros mensais | **Não** (parcela fixa entre reajustes) | Geral |
| Índice negativo (deflação) | **Conta como zero**; o +1% se mantém | Geral |
| Multa por atraso | 2% | Geral |
| Juros de mora | 1% a.m. | Geral |
| Início de encargos por atraso | 1 dia | Geral |
| Desconto de antecipação (parcela) | 0% | Geral |
| Desconto de quitação total | 5% (ajustável) | Geral |
| Retenção no distrato | 30% (devolve 70%) | Geral |
| Forma de devolução do distrato | Parcelada (conforme parcelas pagas) | Geral |
| Comissão | 50% da **entrada**; split empresa/corretor; paga conforme recebimento | Geral |
| Repasse ao loteador | Só em parcerias, valor **variável** (por acordo) | Empreendimento |
| Precificação | Tabela pronta; fator de valorização = tamanho | Empreendimento |

## 4. Modelo de juros/correção — confirmado

Confirmado com o cliente do projeto (substitui a premissa anterior de “1% ao mês”,
**descartada**):

- **Sem juros mensais.** A parcela fica **fixa no valor do mês**.
- **Reajuste anual** (a partir do 13º mês): **IGP-M acumulado (12m) + 1%**, com o
  **1% somado** ao índice (**aditivo**), não composto.
- É, no momento, o **único caso** real.
- **Índice negativo conta como zero**: em ano de IGP-M acumulado negativo, a
  parcela **não diminui** — o reajuste fica só no **+1%**.

```
Fator anual (aditivo) = 1 + ( IGP-M₁₂ₘ + 1% )   → aplicado ao saldo devedor
Parcelas remanescentes recalculadas (sem juros ⇒ saldo / nº restante)
```

**Decisão de produto:** o motor é **parametrizado** para atender este caso **e**
padrões de mercado (Price/SAC com juros mensais, correção mensal), sem engessar.
Os dois mecanismos — **juros de financiamento** e **reajuste (índice + acréscimo
fixo)** — são configuráveis e qualquer um pode ser `nenhum`. Ver
[`05`, §5–§6](05-motor-calculo-financeiro.md#5-métodos-de-amortização).

Residual a validar: o reajuste recalcula a partir do **saldo devedor** (assumido)
ou corrige a **parcela** na competência?

## 5. Outras implicações para os requisitos

- **Confirma** a necessidade dos **dois modelos jurídicos** — a ESW usa **fração
  ideal** (RF-EMP-011).
- **Confirma** comissão **conforme recebimento** e **base = entrada** (não o valor
  total) — o cálculo de comissão precisa aceitar **base configurável** (valor da
  venda **ou** valor da entrada). *(refina RF-COM-002/003)*
- **Confirma** antecipação com política distinta entre **antecipar parcela**
  (sem desconto) e **quitação total** (desconto) — RF-CALC-008/014.
- **Confirma** distrato com **retenção** e **devolução parcelada** — RF-FIN-010.
- **Reforça** que a alteração de contratos vigentes (RF-PAR-010) é usada na
  prática ao menos para **correção de dados** (com auditoria).
- **Valida** o modelo de parametrização: como a ESW é uniforme, quase tudo vive no
  nível **Geral** — a cascata não “atrapalha” operações simples (bom sinal de que
  o design não engessa nem complica).
- **Escopo/priorização ESW:** priorizar **carteira/cobrança**; **mapa, CRM e
  portais** ficam para fases posteriores; a **migração dos ~215 contratos** é uma
  das últimas etapas (Fase 7).

## 6. Próximos passos sugeridos

1. ~~Confirmar o modelo de juros/correção~~ — **confirmado** (§4).
2. ~~Base da comissão como parâmetro~~ — **feito**: `comissao.base` no
   [catálogo](08-catalogo-de-parametros.md#210-comissão).
3. ~~Tratamento de IGP-M negativo~~ — **definido**: conta como zero (§4).
4. Validar as demais **pendências de parâmetros** da ESW
   ([`08` §6](08-catalogo-de-parametros.md#6-pendências-a-validar-esw)) — em
   especial a base do reajuste (saldo × parcela) e a defasagem do índice.
5. **Migração da carteira** (~215 contratos com saldos, planos e histórico):
   **uma das últimas etapas** — Fase 7 (Implantação) do
   [roadmap](04-backlog-e-roadmap.md#3-roadmap-sugerido-em-fases).
