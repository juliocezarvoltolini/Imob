# 03 — Regras de Negócio

Regras de negócio consolidadas (RN-xxx). Elas complementam os requisitos
funcionais de [`01-requisitos.md`](01-requisitos.md) e devem ser **validadas
com o cliente** — vários parâmetros (prazos, percentuais, índices) são
configuráveis e os valores abaixo são exemplos.

- [Parametrização hierárquica](#parametrização-hierárquica)
- [Estoque e disponibilidade de lotes](#estoque-e-disponibilidade-de-lotes)
- [Reservas](#reservas)
- [Precificação](#precificação)
- [Propostas e vendas](#propostas-e-vendas)
- [Financeiro e recebíveis](#financeiro-e-recebíveis)
- [Distrato](#distrato)
- [Comissões](#comissões)
- [Repasses ao loteador](#repasses-ao-loteador)
- [Geográficas](#geográficas)
- [Segurança e auditoria](#segurança-e-auditoria)

---

## Parametrização hierárquica

Aplica-se a todos os parâmetros de negócio, tornando o sistema flexível e
adaptável a cada empreendimento e a cada empresa usuária (ver seção 3.1 de
[`01-requisitos.md`](01-requisitos.md#31-princípio-transversal--parametrização-hierárquica-com-herança)).

| ID | Regra |
|----|-------|
| RN-070 | O **valor efetivo** de um parâmetro é o definido no **nível mais específico** da cadeia (**Contrato → Lote → Setor → Empreendimento → Geral**); se nenhum nível o define, aplica-se o **default** do sistema. |
| RN-071 | Um nível mais específico **sobrescreve** o valor herdado. **Remover** o override em um nível **restaura** a herança do nível acima. |
| RN-072 | Cada parâmetro declara os **níveis em que pode ser definido**; defini-lo em nível não aplicável é rejeitado. |
| RN-073 | Ao **efetivar a venda**, os parâmetros efetivos são **resolvidos e persistidos no contrato** como **versão inicial (snapshot)**. Editar um parâmetro em nível superior **não altera** contratos vigentes — isso protege contra propagação **acidental**, mas **não** torna o contrato imutável (ver RN-076). Generaliza RN-021. |
| RN-073a | Os **parâmetros do contrato são temporais (versionados por vigência)**: cada valor tem período de validade. O motor financeiro **recalcula** parcelas/saldo respeitando a **linha do tempo** dos parâmetros (ex.: juros de 5% até uma data e 1% a partir dela). |
| RN-074 | Alterar um parâmetro em um nível passa a valer, dali em diante, para todos os itens subordinados **sem override próprio**; itens com override permanecem inalterados. |
| RN-075 | A definição/sobrescrita de **parâmetros sensíveis** (juros, índice, carência, retenção de distrato, alçadas) é restrita por **perfil/alçada** (RBAC) e registrada em **auditoria** (RN-060). |
| RN-076 | Alterações **deliberadas** de parâmetros de **contratos vigentes** são permitidas — **individualmente ou em massa** — mediante: seleção de **escopo** de contratos (contrato, empreendimento, filtro, todos), **data de vigência**, **abrangência do recálculo** (ver RN-077), **justificativa**, **alçada/aprovação** e **auditoria**; quando aplicável, geram **aditivo contratual**. A operação é **reversível/estornável**. *(ex.: reduzir juros de contratos firmados de 5% para 1%)* |
| RN-077 | Na alteração, o usuário seleciona a **abrangência do recálculo** (uma ou mais opções): **(a) parcelas em aberto** — já no plano e não pagas (vencidas e a vencer); **(b) parcelas a gerar** — futuras ainda não emitidas; **(c) parcelas já pagas** — recalcula retroativamente e apura o que o cliente **pagou a mais**. A opção **(c)** gera **crédito ao cliente** (abatimento em parcelas futuras) ou **devolução/estorno**, conforme política. **Default: (a) + (b)** — prospectivo, sem tocar em parcelas pagas (comportamento adotado no caso real 5% → 1%). |

## Estoque e disponibilidade de lotes

| ID | Regra |
|----|-------|
| RN-001 | Um lote só pode ser **reservado, proposto ou vendido** se estiver com status **Disponível**. |
| RN-002 | Um lote **Reservado** ou **Em proposta** fica indisponível para outras negociações (impede dupla reserva/venda). |
| RN-003 | Lotes marcados como **área institucional, área verde, APP, sistema viário** ou **não comercializável** não podem ser vendidos. |
| RN-004 | Toda mudança de **status** e de **preço** do lote deve ser registrada em histórico/auditoria (RN-060). |
| RN-005 | Um lote **Vendido** só volta a **Disponível** por **distrato** aprovado (RN-030). |

## Reservas

| ID | Regra |
|----|-------|
| RN-010 | A reserva tem **prazo de expiração** configurável por empreendimento (ex.: 24–72 h). Ao expirar, o lote é liberado automaticamente. |
| RN-011 | A **prorrogação** de reserva além do padrão exige aprovação de gestor. |
| RN-012 | Um mesmo lead/cliente não pode manter mais de N reservas simultâneas (N configurável) para evitar "trava" de estoque. |
| RN-013 | Ao converter a reserva em proposta/venda, mantêm-se o **corretor** e o **cliente** originais (para fins de comissão). |

## Precificação

| ID | Regra |
|----|-------|
| RN-020 | O preço do lote deriva de **valor base** ou **valor/m² × área**, ajustado por **fatores de valorização** (esquina, frente para lago/área verde, topografia). |
| RN-021 | Reajustes na tabela de preços **não alteram** vendas já efetivadas (o contrato "congela" as condições pactuadas). |
| RN-022 | Descontos são limitados por **alçadas** conforme o perfil; acima do limite exigem aprovação (RN-025). |

## Propostas e vendas

| ID | Regra |
|----|-------|
| RN-025 | Proposta com **desconto acima da alçada** do corretor fica **pendente de aprovação** do gestor antes de virar venda. |
| RN-026 | A **entrada/sinal mínima** e o **prazo máximo** de parcelamento são definidos por empreendimento; propostas fora do padrão exigem aprovação. |
| RN-027 | A **efetivação da venda** dispara: atualização do lote para **Vendido**, geração do **contrato** e do **plano de pagamento**, e apuração das **comissões**. |
| RN-028 | Uma venda pode ter **múltiplos compradores** (coproprietários), com responsabilidade solidária pelas parcelas. |
| RN-029 | A venda adota **uma das três formas de pagamento**: **à vista** (quitação integral), **financiamento bancário** (dependente de aprovação de crédito do agente financeiro, que repassa o valor à loteadora) ou **financiamento próprio** (parcelamento em carteira). A forma escolhida define o **plano de pagamento** e a documentação exigida. |
| RN-029a | O **modelo jurídico** do empreendimento determina a titularidade e o instrumento contratual: **loteamento/desmembramento** transfere a **matrícula individual** do lote; **condomínio** transfere **fração ideal**. O fluxo de registro segue o modelo aplicável. |

## Financeiro e recebíveis

| ID | Regra |
|----|-------|
| RN-030 | O **plano de pagamento** é composto por entrada + parcelas mensais + intermediárias (balões) + parcela final, conforme a proposta aprovada e a **forma de pagamento** (à vista, financiamento bancário ou financiamento próprio). |
| RN-031 | No **financiamento próprio**, a dívida evolui por **dois mecanismos configuráveis e combináveis**: **(a) juros de financiamento** (método `nenhum`/Price/SAC/juros simples + taxa ao mês) e **(b) reajuste periódico** por **índice** (IGP-M/INCC/IPCA) somado de um **acréscimo fixo** (aditivo ou composto), com **periodicidade** (mensal/anual) e **carência**. Todos parametrizáveis e registrados no contrato. Qualquer mecanismo pode ser `nenhum`. |
| RN-031a | **Caso real (ESW), confirmado**: **sem juros mensais**; **reajuste anual = IGP-M acumulado (12m) + 1% fixo (aditivo)**, a partir do 13º mês (carência de 12 meses). A premissa anterior de “1% ao mês” foi **descartada**. Os parâmetros permitem também padrões de mercado (Price/SAC com juros mensais, correção mensal). *(ver [`07`](07-perfil-cliente-esw.md) e o motor [`05`](05-motor-calculo-financeiro.md))* |
| RN-032 | Parcela paga **após o vencimento** sofre **multa + juros de mora + correção**, conforme parâmetros do contrato. |
| RN-033 | A **baixa** de uma parcela pode ser manual, por **retorno CNAB** ou por **PIX**; a baixa é **idempotente** (um mesmo pagamento não baixa em duplicidade). |
| RN-034 | A **antecipação** de parcelas pode receber **desconto (deságio)** sobre juros/correção futuros, conforme política configurável. |
| RN-035 | Um contrato é marcado **inadimplente** após X dias de atraso (configurável) e entra na **régua de cobrança**. |
| RN-036 | A **régua de cobrança** dispara avisos escalonados (pré-vencimento, D+1, D+7, D+15, D+30...) por e-mail/SMS/WhatsApp. |
| RN-037 | A **renegociação** gera um **novo plano de pagamento** que substitui o anterior, preservando o histórico. |

## Distrato

| ID | Regra |
|----|-------|
| RN-040 | O **distrato** calcula o valor a **devolver** ao comprador = valores pagos − retenções (taxa administrativa, comissão, cláusula penal), conforme contrato e legislação. |
| RN-041 | O distrato pode gerar **cronograma de devolução** (parcelado) ao comprador. |
| RN-042 | Ao concluir o distrato, o lote retorna a **Disponível** e as comissões associadas podem ser **estornadas/ajustadas** (RN-052). |
| RN-043 | O distrato exige **aprovação** (jurídico/gestor) e registro documental. |

## Comissões

| ID | Regra |
|----|-------|
| RN-050 | A comissão é calculada sobre a **base definida** (valor da venda ou valor recebido) pelo **percentual/tabela** vigente do empreendimento/campanha. |
| RN-051 | A comissão pode ser dividida em **split** (captador, corretor, gerente, imobiliária) segundo regra configurável, somando 100% do valor de comissão. |
| RN-052 | Quando a política é "pagamento **conforme recebimento**", a comissão é liberada proporcionalmente às parcelas efetivamente recebidas; distrato/inadimplência ajustam o saldo. |

## Repasses ao loteador

| ID | Regra |
|----|-------|
| RN-055 | O **repasse** ao loteador/proprietário é calculado por venda ou por recebimento, conforme o contrato de parceria (percentual ou valor). |
| RN-056 | Distrato reverte/ajusta os repasses correspondentes. |

## Geográficas

| ID | Regra |
|----|-------|
| RN-057 | A **área e o perímetro** calculados a partir dos vértices devem ser consistentes com o **sistema de coordenadas/datum** do empreendimento. |
| RN-058 | Divergências relevantes entre a **área calculada** e a **área do memorial descritivo** devem ser sinalizadas para conferência. |
| RN-059 | Lotes **não podem se sobrepor** geometricamente dentro do mesmo empreendimento (validação na importação/edição). |

## Segurança e auditoria

| ID | Regra |
|----|-------|
| RN-060 | Operações sensíveis (alteração de preço, mudança de status de lote, baixa/estorno financeiro, distrato, aprovação de desconto) geram **registro de auditoria** imutável com autor, data/hora e valores antes/depois. |
| RN-061 | O acesso a dados e ações é restrito pelo **perfil (RBAC)** e pelos **empreendimentos** autorizados ao usuário. |
| RN-062 | Dados pessoais seguem a **LGPD**: consentimento registrado, e atendimento a pedidos de acesso/correção/exclusão-anonimização. |
