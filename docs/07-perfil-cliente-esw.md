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
> distrato, quitação)** e **cadastro dos contratos existentes** (migração da
> carteira), deixando CRM, mapa e portais para depois.

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
| Método de reajuste | **Reajuste anual = IGP-M (12m) + 1% fixo** | Geral |
| Índice de correção | IGP-M | Geral |
| Carência de correção | 12 meses | Geral |
| Periodicidade do reajuste | Anual (aniversário) | Geral |
| Juros mensais | **Não** (parcela fixa entre reajustes) | Geral |
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

## 4. Divergência importante a confirmar — modelo de juros/correção

O que foi dito antes (turno anterior): *“juros ao mês + correção IGP-M depois de
um ano”*. O que o formulário da ESW descreve é **diferente**:

- **Não há juros mensais compostos.** A parcela fica **fixa no valor do mês**.
- Uma vez por ano (a partir do 13º mês) o saldo/parcela é **reajustado** por
  **IGP-M acumulado dos últimos 12 meses + 1% (contratual)**.

Ou seja, é um **reajuste anual de aniversário = índice + acréscimo fixo**, e não a
Tabela Price com 1% a.m. Isso **refina o motor de cálculo** ([`05`](05-motor-calculo-financeiro.md)):
o reajuste anual precisa combinar **índice + um percentual fixo** (a “taxa
contratual”), com opção **sem juros mensais**.

> **A confirmar com o cliente do projeto:** (1) o acréscimo de 1% é **somado** ao
> IGP-M (aditivo) ou **composto** (×1,01)? (2) Existe algum caso com juros mensais
> (Price/SAC), ou o padrão é sempre “sem juros + reajuste anual”? (3) A premissa
> anterior (“1% ao mês”) vale para outro cliente/cenário, ou deve ser descartada?

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
- **Escopo/priorização ESW:** priorizar **carteira/cobrança + migração dos ~215
  contratos**; **mapa, CRM e portais** ficam para fases posteriores.

## 6. Próximos passos sugeridos

1. Confirmar o **modelo de juros/correção** (§4) — é o item que mais afeta o motor.
2. Definir **base da comissão** como parâmetro (venda × entrada).
3. Planejar a **migração da carteira** existente (importar ~215 contratos com seus
   saldos, planos e histórico) como parte do MVP para a ESW.
