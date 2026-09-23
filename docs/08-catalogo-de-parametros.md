# 08 — Catálogo de Parâmetros

Catálogo inicial dos **parâmetros de negócio** do sistema (RF-PAR-001). Cada
parâmetro é resolvido pela cascata **Geral → Empreendimento → Setor → Lote →
Contrato** ([§3.1](01-requisitos.md#31-princípio-transversal--parametrização-hierárquica-com-herança),
RN-070..079) e alimenta o motor de cálculo ([`05`](05-motor-calculo-financeiro.md)).
A coluna **ESW** traz a configuração do caso real ([`07`](07-perfil-cliente-esw.md)).

**Objetivo de desenho:** atender o caso real **e** os modelos de mercado
facilmente previsíveis **só por configuração** — ver os
[presets](#4-presets-modelos-de-configuração).

- [1. Como ler este catálogo](#1-como-ler-este-catálogo)
- [2. Parâmetros por domínio](#2-parâmetros-por-domínio)
- [3. Validações entre parâmetros](#3-validações-entre-parâmetros)
- [4. Presets (modelos de configuração)](#4-presets-modelos-de-configuração)
- [5. Decisões de projeto e pontos de atenção](#5-decisões-de-projeto-e-pontos-de-atenção)
- [6. Pendências a validar (ESW)](#6-pendências-a-validar-esw)

---

## 1. Como ler este catálogo

**Chaves:** `domínio.subdomínio.nome` em minúsculas (ex.: `reajuste.acrescimo_fixo`).
São estáveis — servem de identificador no banco, na API e na auditoria.

| Coluna | Significado |
|--------|-------------|
| **Valores** | Tipo e domínio: `enum` (uma opção), `lista` (várias), `%`, `% a.m.`, `R$`, `int` (com unidade), `bool`, `tabela` (linhas estruturadas), `ref` (aponta para um cadastro) |
| **Default** | Valor da **plataforma** quando nenhum nível define o parâmetro. “— (obrigatório)” = sem default; precisa ser configurado (VP-11) |
| **ESW** | Valor do caso real. “a validar” = não respondido ou ambíguo no questionário (ver §6) |
| **Níveis** | Onde pode ser definido: **G** Geral (da empresa assinante) · **E** Empreendimento · **S** Setor/Quadra · **L** Lote · **C** Contrato |
| **Contrato** | **C** = congela na venda (snapshot temporal; só muda por revisão governada, RF-PAR-010) · **D** = dinâmico (sempre o valor vigente na cascata) · **P** = regra de proposta (valida/simula; o valor negociado vai ao contrato) · **—** = não se aplica |
| **Q** | Item do [questionário](06-questionario-configuracao.md) que informa o valor |
| 🔒 | Parâmetro **sensível**: exige alçada/permissão e gera auditoria (RN-075) |

## 2. Parâmetros por domínio

### 2.1 Venda e formas de pagamento

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `venda.formas_habilitadas` | Formas de pagamento aceitas | lista: à vista, próprio, bancário, permuta, dação | à vista, próprio | à vista, próprio, permuta/dação | G E | — | 2.1 |
| `venda.desconto.alcada_corretor` 🔒 | Desconto máximo sem aprovação | % | 0% | a validar | G E | P | 10.3 |
| `venda.desconto.alcada_gerente` 🔒 | Desconto máximo com aprovação do gerente | % | 5% | a validar | G E | P | 10.3 |
| `venda.proposta.validade_dias` | Validade da proposta | int (dias) | 7 | a validar | G E | P | — |

### 2.2 Entrada

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `entrada.percentual_min` | Entrada mínima sobre o valor da venda | % | 10% | 6% | G E S L | P | 3.1 |
| `entrada.percentual_padrao` | Entrada sugerida na simulação | % | 10% | 10% | G E S L | P | 3.1 |
| `entrada.parcelavel` | A entrada pode ser parcelada | bool | não | sim | G E | P | 3.2 |
| `entrada.parcelas_max` | Nº máximo de parcelas da entrada | int | 1 | 4 | G E | P | 3.2 |

### 2.3 Plano de pagamento

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `plano.prazo_min_parcelas` | Prazo mínimo | int (parcelas) | 1 | a validar | G E S L | P | 3.3 |
| `plano.prazo_max_parcelas` | Prazo máximo | int (parcelas) | 120 | 120 | G E S L | P | 3.3 |
| `plano.prazo_padrao_parcelas` | Prazo sugerido na simulação | int (parcelas) | 60 | 120 | G E S L | P | 3.3 |
| `plano.intermediarias.habilitadas` | Permite parcelas intermediárias (reforços) | bool | não | não | G E | P | 3.4 |
| `plano.intermediarias.periodicidade_meses` | Intervalo entre intermediárias | int (meses) | 12 | — | G E | P | 3.4 |
| `plano.parcela_final.habilitada` | Permite parcela final maior (balão) | bool | não | não | G E | P | 3.5 |
| `plano.dia_vencimento` | Dia do mês do vencimento | int (1–28) | 10 | a validar | G E C | C | — |
| `plano.primeiro_vencimento_dias` | Dias entre a venda e a 1ª parcela | int (dias) | 30 | a validar | G E | P | — |
| `plano.arredondamento` | Parcela que absorve a diferença de centavos | enum: primeira, última | última | última | G | C | — |

### 2.4 Juros de financiamento (eixo ① do motor)

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `juros.metodo` 🔒 | Como o capital é remunerado | enum: nenhum, price, sac, simples | nenhum | nenhum | G E S L C | C | 4.1, 4.7 |
| `juros.taxa_mensal` 🔒 | Taxa de juros | % a.m. | 0% | — | G E S L C | C | 4.1 |
| `juros.simples.convencao` | Convenção de juros simples | enum: linear (outras a definir) | linear | — | G E | C | — |

### 2.5 Reajuste periódico (eixo ② do motor)

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `reajuste.periodicidade` 🔒 | Frequência do reajuste | enum: nenhum, mensal, anual | nenhum | anual | G E S L C | C | 4.6 |
| `reajuste.indice` 🔒 | Índice de atualização | enum: nenhum, IGP-M, INCC, IPCA | nenhum | IGP-M | G E S L C | C | 4.3, 4.4 |
| `reajuste.acrescimo_fixo` 🔒 | Percentual somado/composto ao índice | % | 0% | 1% | G E S L C | C | 4.1, 4.4 |
| `reajuste.acrescimo_modo` | Como o acréscimo combina com o índice | enum: aditivo, composto | aditivo | aditivo | G E C | C | — |
| `reajuste.carencia_meses` 🔒 | Meses iniciais sem reajuste | int (meses) | 12 | 12 | G E S L C | C | 4.5 |
| `reajuste.data_base` | Referência do ciclo de reajuste | enum: aniversário do contrato, 1ª parcela, data fixa | aniversário do contrato | aniversário do contrato | G E C | C | 4.5 |
| `reajuste.base_aplicacao` | Onde o reajuste incide | enum: saldo devedor, parcela | saldo devedor | a validar (assumido saldo) | G E C | C | 4.7 |
| `reajuste.defasagem_meses` | Defasagem do índice (ex.: índice do mês anterior) | int (meses) | 1 | a validar | G E C | C | — |
| `reajuste.indice_negativo` 🔒 | Tratamento de índice acumulado negativo | enum: piso no índice, piso no total, aplicar | piso no total | piso no índice (conta como 0) | G E C | C | — |

### 2.6 Encargos por atraso

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `mora.multa` 🔒 | Multa por atraso | % | 2% | 2% | G E C | C | 5.1 |
| `mora.juros_mensal` 🔒 | Juros de mora | % a.m. | 1% | 1% | G E C | C | 5.2 |
| `mora.calculo` | Contagem dos juros de mora | enum: pro rata die, mês cheio | pro rata die | pro rata die (a validar) | G E C | C | 5.3 |
| `mora.tolerancia_dias` | Dias de tolerância antes dos encargos | int (dias) | 0 | 0 | G E C | C | 5.3 |
| `mora.base_multa` | Sobre o que a multa incide | enum: parcela atualizada, parcela nominal | parcela atualizada | a validar | G E C | C | — |
| `mora.corrige_atraso` | Atualiza monetariamente o período em atraso | bool | não | a validar | G E C | C | — |

### 2.7 Cobrança (operacional)

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `cobranca.meios` | Meios de pagamento oferecidos | lista: boleto, PIX, carnê, transferência | boleto, PIX | boleto, PIX, carnê | G E | D | 5.5 |
| `cobranca.canais_envio` | Canais de envio de boletos e avisos | lista: e-mail, WhatsApp, SMS, impresso | e-mail | e-mail, WhatsApp | G E | D | 5.4, 5.5 |
| `cobranca.aviso_pre_vencimento` | Aviso antes do vencimento | int (dias) ou desligado | desligado | desligado | G E | D | 5.4 |
| `cobranca.regua` | Régua de cobrança após o vencimento | tabela: dias após, canal, mensagem | desligada | WhatsApp/e-mail (passos a definir) | G E | D | 5.4 |
| `cobranca.dias_inadimplencia` | Dias de atraso para marcar inadimplente | int (dias) | 30 | 1 | G E | D | 5.3 |
| `cobranca.conta_bancaria` | Conta/carteira de cobrança | ref (cadastro bancário) | — | Bradesco | G E | D | 5.6 |

### 2.8 Antecipação e quitação

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `antecipacao.permitida` | O cliente pode antecipar parcelas | bool | sim | sim | G E | D | 6.1 |
| `antecipacao.desconto_modo` | Desconto ao antecipar parcelas | enum: nenhum, percentual fixo, valor presente | nenhum | nenhum | G E | D | 6.2 |
| `antecipacao.desconto_percentual` | Percentual, se “percentual fixo” | % | 0% | — | G E | D | 6.2 |
| `quitacao.desconto_modo` | Desconto na quitação total | enum: nenhum, percentual fixo, valor presente | nenhum | percentual fixo | G E | D | 6.3 |
| `quitacao.desconto_percentual` 🔒 | Desconto padrão de quitação | % | 0% | 5% | G E | D | 6.3 |
| `quitacao.desconto_max_sem_aprovacao` 🔒 | Acima deste valor exige alçada | % | 0% | 5% (a validar) | G E | D | 6.3 |

> “Valor presente” = traz as parcelas futuras a valor presente pela taxa do
> contrato (só tem efeito quando há juros de financiamento).

### 2.9 Distrato

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `distrato.retencao` 🔒 | Percentual retido do valor pago | % | — (obrigatório) | 30% (devolve 70%) | G E C | C | 7.2 |
| `distrato.retencao_max_legal` 🔒 | Teto de retenção (guarda legal) | % | — (definir com o jurídico) | — | G | — | — |
| `distrato.base` | Base da devolução | enum: valor pago, valor pago atualizado | valor pago | a validar | G E C | C | 7.2 |
| `distrato.devolucao_forma` | Forma de devolução | enum: à vista, parcelada | à vista | parcelada | G E C | C | 7.3 |
| `distrato.devolucao_parcelas` | Regra do nº de parcelas da devolução | enum: igual às parcelas pagas, número fixo, tabela por faixas | igual às pagas | conforme parcelas pagas (a validar) | G E C | C | 7.3 |
| `distrato.estorno_comissao` | Tratamento da comissão já paga | enum: estorna integral, proporcional, não estorna | proporcional | a validar | G E | C | — |

### 2.10 Comissão

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `comissao.base` 🔒 | Sobre o que a comissão incide | enum: valor da venda, valor da entrada, valor recebido | valor da venda | valor da entrada | G E C | C | 8.2 |
| `comissao.percentual` 🔒 | Percentual de comissão | % | — (obrigatório) | 50% | G E C | C | 8.2, 8.3 |
| `comissao.pagamento` | Quando a comissão é paga | enum: na venda, conforme recebimento, parcelas fixas | na venda | conforme recebimento | G E C | C | 8.5 |
| `comissao.split` 🔒 | Divisão entre participantes | tabela: papel, % (soma 100%) | corretor 100% | empresa / corretor (% a validar) | G E C | C | 8.4 |

### 2.11 Repasse ao dono da terra

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `repasse.habilitado` | Há repasse ao loteador/parceiro | bool | não | não (sim em parcerias) | E | C | 9.1, 9.2 |
| `repasse.base` | Base do repasse | enum: valor da venda, valor recebido | valor recebido | por parceria | E | C | 9.2 |
| `repasse.percentual` 🔒 | Percentual repassado | % | — | por parceria (variável) | E | C | 9.2 |

### 2.12 Reserva

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `reserva.prazo_horas` | Validade da reserva | int (horas) | 48 | a validar | G E | — | — |
| `reserva.max_por_cliente` | Reservas simultâneas por cliente | int | 2 | a validar | G | — | — |
| `reserva.prorrogacao_aprovacao` | Prorrogação exige aprovação | bool | sim | a validar | G E | — | — |

### 2.13 Preço

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `preco.modelo` | Como o preço é definido | enum: tabela, valor por m², valor fechado | tabela | tabela | G E | — | 11.1 |
| `preco.valor_m2` | Valor por m² (se modelo = m²) | R$/m² | — | — | E S | — | 11.1 |
| `preco.fatores_valorizacao` | Ajustes por característica do lote | tabela: critério, ajuste % | nenhum | tamanho (ajuste a definir) | E S L | — | 11.2 |
| `preco.reajuste_tabela` | Como a tabela é reajustada | enum: manual, periódico | manual | manual (conforme valorização) | G E | — | 11.3 |

### 2.14 Contrato

| Chave | O que define | Valores | Default | ESW | Níveis | Contrato | Q |
|-------|--------------|---------|---------|-----|--------|:--------:|---|
| `contrato.modelo_juridico` | Loteamento (matrícula) ou condomínio (fração ideal) | enum: loteamento, condomínio | — (obrigatório por empreendimento) | condomínio (fração ideal) | E | C | 12.3 |
| `contrato.modelo_documento` | Modelo (template) do contrato | ref (cadastro de modelos) | padrão do sistema | 1 modelo | G E | — | 12.1 |
| `contrato.assinatura` | Forma de assinatura | enum: papel, eletrônica, ambas | ambas | ambas | G E | — | 12.2 |

**Total: 70 parâmetros** em 14 domínios.

## 3. Validações entre parâmetros

Verificadas ao salvar (RF-PAR-012). **Bloqueiam** configurações inválidas e
**alertam** as que ficariam sem efeito.

| ID | Regra |
|----|-------|
| VP-01 | `juros.taxa_mensal` > 0 é **obrigatória** se `juros.metodo` ≠ nenhum; é ignorada se `nenhum`. |
| VP-02 | `juros.simples.convencao` é obrigatória se `juros.metodo` = simples. |
| VP-03 | Se `reajuste.periodicidade` ≠ nenhum, então `reajuste.indice` ≠ nenhum **ou** `reajuste.acrescimo_fixo` > 0 — senão o reajuste não tem efeito (alerta). *Permite “reajuste fixo anual” sem índice.* |
| VP-04 | `entrada.percentual_min` ≤ `entrada.percentual_padrao` ≤ 100%. |
| VP-05 | `plano.prazo_min_parcelas` ≤ `plano.prazo_padrao_parcelas` ≤ `plano.prazo_max_parcelas`. |
| VP-06 | `entrada.parcelas_max` > 1 só se `entrada.parcelavel` = sim. |
| VP-07 | `comissao.split` soma exatamente 100%. |
| VP-08 | `distrato.retencao` entre 0% e 100% e ≤ `distrato.retencao_max_legal`, quando definido. |
| VP-09 | `mora.multa` ≤ teto legal configurado (ex.: limite de multa em relações de consumo — **validar com o jurídico**). |
| VP-10 | `quitacao.desconto_percentual` acima de `quitacao.desconto_max_sem_aprovacao` exige aprovação (alçada). |
| VP-11 | Parâmetros **sem default** (`distrato.retencao`, `comissao.percentual`, `contrato.modelo_juridico`) precisam estar definidos antes da **primeira venda** do empreendimento. |
| VP-12 | Um parâmetro só pode ser definido nos seus **níveis aplicáveis** (RN-072). |

## 4. Presets (modelos de configuração)

Pontos de partida prontos. Aplicar um preset **copia** os valores para o nível
escolhido (Geral ou Empreendimento) — não cria vínculo; depois, qualquer valor
pode ser sobrescrito (RN-078, RF-PAR-011). A imobiliária também pode **salvar a
própria configuração como preset**.

> Os valores dos presets de mercado são **exemplos ilustrativos** (“ex.:”), não
> recomendações — cada imobiliária ajusta os seus.

| Parâmetro | ESW (caso real) | Price + IGP-M anual | SAC + IPCA mensal | Sem juros, sem reajuste |
|-----------|-----------------|---------------------|-------------------|-------------------------|
| `juros.metodo` | nenhum | price | sac | nenhum |
| `juros.taxa_mensal` | — | ex.: 1% | ex.: 0,8% | — |
| `reajuste.periodicidade` | anual | anual | mensal | nenhum |
| `reajuste.indice` | IGP-M | IGP-M | IPCA | nenhum |
| `reajuste.acrescimo_fixo` | 1% (aditivo) | 0% | 0% | — |
| `reajuste.carencia_meses` | 12 | 12 | 0 | — |
| `reajuste.indice_negativo` | piso no índice | ex.: piso no total | ex.: piso no total | — |
| `entrada.percentual_min` | 6% | ex.: 10% | ex.: 20% | ex.: 20% |
| `entrada.parcelas_max` | 4 | 1 | 1 | ex.: 3 |
| `plano.prazo_max_parcelas` | 120 | ex.: 180 | ex.: 240 | ex.: 24 |
| `mora.multa` / `mora.juros_mensal` | 2% / 1% a.m. | 2% / 1% a.m. | 2% / 1% a.m. | 2% / 1% a.m. |
| `antecipacao.desconto_modo` | nenhum | valor presente | valor presente | nenhum |
| `quitacao.desconto_modo` | 5% fixo | valor presente | valor presente | nenhum |
| `comissao.base` / `percentual` | entrada / 50% | ex.: venda / 5% | ex.: venda / 5% | ex.: venda / 5% |
| `comissao.pagamento` | conforme recebimento | na venda | na venda | na venda |
| `distrato.retencao` | 30% | a definir (jurídico) | a definir (jurídico) | a definir (jurídico) |

## 5. Decisões de projeto e pontos de atenção

1. **Defaults conservadores.** Sem configuração, o sistema **não** cobra juros,
   **não** reajusta (método e índice = `nenhum`) e **não** envia mensagens (régua
   desligada): nada acontece “por padrão” sem que a imobiliária tenha escolhido.
   Decisões comerciais/jurídicas críticas **não têm default** (VP-11). Os presets
   ligam o que cada modelo precisa (RN-079).
2. **Índice negativo (deflação)** — já ocorreu com o IGP-M e interage com o
   acréscimo fixo. Exemplo ESW com IGP-M acumulado de −3% e +1% aditivo:

   | Política (`reajuste.indice_negativo`) | Reajuste resultante |
   |---------------------------------------|---------------------|
   | piso no índice (índice conta como 0) — **escolha da ESW** | +1% |
   | piso no total (nunca reduz) — *default* | 0% |
   | aplicar | −2% (saldo diminui) |

3. **Dimensão do participante.** A cascata G→E→S→L→C cobre o **produto**. A
   comissão também pode variar por **quem vende** (corretor/parceria). Proposta a
   confirmar: tratar como override por participante, com precedência
   **Contrato > Participante > Lote/Setor/Empreendimento > Geral**.
4. **Guardas legais como tetos.** Multa e retenção de distrato têm limites legais
   que dependem do tipo de operação. O produto **não fixa esses números**: o
   jurídico configura os tetos (`distrato.retencao_max_legal`, teto de multa) e o
   sistema valida (VP-08, VP-09).
5. **Congela × dinâmico.** Termos financeiros do contrato (juros, reajuste, mora,
   distrato, comissão) **congelam** na venda. Políticas operacionais (cobrança,
   régua, canais, desconto de quitação) são **dinâmicas** — mudam para todos
   quando a imobiliária muda a política, sem revisão contratual.

## 6. Pendências a validar (ESW)

Itens marcados “a validar” acima. Os itens 2 e 3 afetam cálculo:

1. ~~`reajuste.indice_negativo`~~ — **definido**: índice negativo **conta como
   zero** (piso no índice). Em ano de IGP-M negativo, o reajuste fica só no **+1%**.
2. **`reajuste.base_aplicacao`** — o reajuste recalcula pelo **saldo devedor** ou
   atualiza a **parcela**?
3. **`reajuste.defasagem_meses`** — o “IGP-M dos últimos 12 meses” termina em
   qual mês em relação ao aniversário?
4. **Mora**: juros pro rata die? multa sobre a parcela atualizada? há correção no
   período de atraso?
5. **Distrato**: devolução sobre valor pago ou atualizado; regra exata do nº de
   parcelas; o que acontece com a comissão já paga.
6. **Comissão**: percentuais do split **empresa × corretor**.
7. **Quitação**: quem aprova desconto acima de 5%.
8. Menos críticos para a ESW (operação de carteira): alçadas de desconto, dia de
   vencimento, 1º vencimento e reserva.
