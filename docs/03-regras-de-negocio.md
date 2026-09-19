# 03 — Regras de Negócio

Regras de negócio consolidadas (RN-xxx). Elas complementam os requisitos
funcionais de [`01-requisitos.md`](01-requisitos.md) e devem ser **validadas
com o cliente** — vários parâmetros (prazos, percentuais, índices) são
configuráveis e os valores abaixo são exemplos.

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

## Financeiro e recebíveis

| ID | Regra |
|----|-------|
| RN-030 | O **plano de pagamento** é composto por entrada + parcelas mensais + intermediárias (balões) + parcela final, conforme a proposta aprovada. |
| RN-031 | O **saldo devedor** é corrigido pelo **índice contratado** (ex.: INCC, IGP-M, IPCA) na periodicidade pactuada; o índice e a data-base ficam registrados no contrato. |
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
