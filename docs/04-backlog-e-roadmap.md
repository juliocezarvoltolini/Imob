# 04 — Backlog de Funcionalidades e Roadmap

Priorização das funcionalidades (MoSCoW) e proposta de roadmap em fases.
Referencia os requisitos de [`01-requisitos.md`](01-requisitos.md).

- [1. Épicos e histórias de usuário](#1-épicos-e-histórias-de-usuário)
- [2. Priorização MoSCoW](#2-priorização-moscow)
- [3. Roadmap sugerido em fases](#3-roadmap-sugerido-em-fases)
- [4. Critérios de aceite (exemplos)](#4-critérios-de-aceite-exemplos)

---

## 1. Épicos e histórias de usuário

Formato: **Como** \<ator\>, **quero** \<ação\>, **para** \<valor\>.

### Épico 0 — Fundação e Parametrização (transversal)
- Como **administrador**, quero definir parâmetros em vários níveis (geral,
  empreendimento, setor, lote, contrato) com **herança e sobrescrita**, **para**
  adaptar o sistema à realidade de cada empreendimento sem engessar as regras.
  *(RF-PAR-001..003)*
- Como **financeiro**, quero que os parâmetros sejam **persistidos no contrato**
  no momento da venda, **para** que reajustes futuros **não afetem por engano**
  contratos vigentes. *(RF-PAR-005, RN-073)*
- Como **gestor/financeiro**, quero **alterar em massa** um parâmetro de
  contratos já firmados (ex.: **reduzir juros de 5% para 1%**) com **data de
  vigência** e recálculo, **para** cumprir decisões comerciais/jurídicas sem
  refazer contrato a contrato. *(RF-PAR-010, RN-076, RN-073a)*

### Épico A — Cadastro e Estoque Georreferenciado
- Como **backoffice**, quero cadastrar um empreendimento e importar seus lotes a
  partir de um arquivo geográfico simples (KML/GeoJSON) ou planilha, **para** montar o
  estoque sem digitação manual. *(RF-EMP-001, RF-LOT-010)*
- Como **backoffice**, quero que a **área e o perímetro** de cada lote sejam
  calculados automaticamente a partir dos vértices, **para** conferir com o
  memorial descritivo. *(RF-LOT-004, RN-058)*

### Épico B — Mapa / Espelho de Vendas
- Como **corretor**, quero ver os lotes no mapa **coloridos por status**, **para**
  apresentar rapidamente o que está disponível. *(RF-MAP-002)*
- Como **corretor**, quero **reservar um lote pelo mapa**, **para** agilizar o
  atendimento. *(RF-MAP-004, RF-RES-001)*

### Épico C — Funil Comercial
- Como **corretor**, quero registrar leads e suas interações, **para** organizar
  meu atendimento. *(RF-CRM-001, RF-CRM-005)*
- Como **gestor**, quero aprovar propostas com desconto acima da alçada, **para**
  manter a política comercial. *(RF-VEN-003, RN-025)*

### Épico D — Venda e Contrato
- Como **corretor**, quero **simular condições de pagamento** e fechar a venda,
  **para** converter a negociação. *(RF-VEN-002, RF-VEN-006)*
- Como **jurídico**, quero gerar o contrato por template e enviá-lo para
  **assinatura eletrônica**, **para** formalizar com agilidade. *(RF-CTR-001,
  RF-CTR-003)*

### Épico E — Financeiro / Recebíveis (Carteira)
- Como **financeiro**, quero gerar o **carnê** com correção e emitir boletos,
  **para** cobrar as parcelas. *(RF-FIN-001, RF-FIN-003)*
- Como **financeiro**, quero **baixar pagamentos automaticamente** pelo retorno
  bancário, **para** reduzir trabalho manual. *(RF-FIN-004, RN-033)*
- Como **financeiro**, quero uma **régua de cobrança** e um painel de
  inadimplência, **para** reduzir a perda. *(RF-FIN-006, RF-FIN-007)*
- Como **financeiro**, quero processar **distratos** com o cálculo de devolução,
  **para** encerrar contratos corretamente. *(RF-FIN-010, RN-040)*

### Épico F — Comissões e Repasses
- Como **gestor**, quero que a **comissão** seja calculada e dividida no split
  automaticamente, **para** pagar corretamente os envolvidos. *(RF-COM-003,
  RN-051)*
- Como **financeiro**, quero calcular os **repasses ao loteador**, **para**
  honrar o contrato de parceria. *(RF-FCX-002, RN-055)*

### Épico G — Portais
- Como **cliente**, quero emitir a **2ª via do boleto** e ver meu extrato,
  **para** me autoatender. *(RF-PCL-002, RF-PCL-003)*
- Como **corretor**, quero acessar o **estoque e minhas comissões** pelo celular,
  **para** trabalhar em campo. *(RF-PCO-001, RF-PCO-005)*

### Épico H — Gestão e Indicadores
- Como **diretoria**, quero um **dashboard de vendas e recebíveis**, **para**
  decidir com dados. *(RF-REL-001, RF-REL-003)*
- Como **administrador**, quero gerir **perfis e permissões**, **para** controlar
  o acesso. *(RF-ADM-002)*

## 2. Priorização MoSCoW

### Must (MVP)
Cadastro de empreendimentos (com **os dois modelos jurídicos**: matrícula
individual e fração ideal) e lotes com **georreferenciamento simplificado**
(coordenadas + dimensões planas) e cálculo de área/perímetro; importação
geográfica (KML/GeoJSON/planilha); mapa/espelho de vendas por status; CRM básico
e reservas; propostas com simulação e alçadas nas **três formas de pagamento**
(à vista, bancário, próprio); efetivação de venda; geração de contrato por
template; **plano de pagamento do financiamento próprio com juros mensais e
correção IGP-M a partir do 13º mês, boletos e baixa (manual + CNAB)**;
inadimplência básica; distrato; comissão com split (à vista e conforme
recebimento); usuários/RBAC e auditoria; dashboard comercial e espelho
exportável.

*(RF-EMP-001/002/005/006/008/009/010/011, RF-LOT-001..004/006/008/009/010/012,
RF-MAP-001..006, RF-CRM-001/003/005/007/010, RF-RES-001..003/007,
RF-VEN-001..003/005..008, RF-CTR-001/002/005/007, RF-FIN-001..005/007/010/012,
RF-COM-001..004, RF-GED-001, RF-REL-001..003, RF-ADM-001/002/004/005,
RF-PAR-001..003/005/007,
RF-CALC-001..007/009/010/012/013/015/017, RNF-001..004/007..009/011/012)*

### Should
Captação automática de leads e distribuição; visitas; assinatura eletrônica;
régua de cobrança; renegociação e antecipação; informe de IR; extrato de
comissões e regra "conforme recebimento"; contas a pagar/receber, repasses e
exportação contábil; checklist documental; portal do cliente e do corretor
(web); multiempresa; relatórios de comissão e repasse; mensageria; validações
Receita/CEP; **alteração de parâmetros de contratos vigentes** (individual/em
massa) com vigência temporal e recálculo (RF-PAR-010, RF-FIN-015).

### Could
Camadas informativas do mapa (APP/Reserva Legal), medição e modo apresentação;
fila de espera em reservas; remembramento/desmembramento; detecção de
sobreposição geométrica; metas/performance; conciliação bancária; relatórios
customizáveis; controle de validade documental; BI externo; acompanhamento de
registro em cartório.

### Won't (por ora)
**App mobile** (nesta etapa o acesso é somente via navegador); **interface de
topógrafo** e tratamento de **complexidade topográfica** (relevo, curvas de
nível, edição CAD/DWG); execução detalhada de obra/engenharia; contabilidade
fiscal completa; cartório eletrônico; portal público de anúncios próprio.
*(ver "Fora do escopo" em
[`01-requisitos.md`](01-requisitos.md#12-fora-do-escopo-nesta-versão))*

## 3. Roadmap sugerido em fases

> Sequência lógica de entregas. Datas/estimativas dependem do time e das
> definições em aberto (ver seção 9 de `01-requisitos.md`).

| Fase | Tema | Entregas principais |
|------|------|---------------------|
| **Fase 0 — Fundação** | Base técnica | Autenticação, RBAC, multiempresa, **parametrização hierárquica** (herança Geral→Empreendimento→Setor→Lote→Contrato), auditoria, GED básico. |
| **Fase 1 — Estoque e Mapa** | "Ver e cadastrar" | Empreendimentos, lotes, importação geográfica, cálculo de área/perímetro, mapa/espelho de vendas por status. |
| **Fase 2 — Comercial** | "Vender" | CRM, reservas, propostas com simulação e alçadas nas **três formas de pagamento**, efetivação da venda, contrato por template (para os **dois modelos jurídicos**). |
| **Fase 3 — Financeiro** | "Receber" | **Motor de cálculo** (Price/SAC, correção com carência, mora, antecipação), plano de pagamento do financiamento próprio (**juros mensais + IGP-M a partir do 13º mês**), boletos/PIX, baixa CNAB, inadimplência, distrato, extrato por contrato, **revisão de parâmetros de contratos vigentes** (individual/em massa, com vigência temporal). |
| **Fase 4 — Comissões e Repasses** | "Distribuir" | Tabelas e split de comissão, extratos, repasses ao loteador, contas a pagar/receber. |
| **Fase 5 — Autoatendimento e Cobrança** | "Escalar" | Portal do cliente (boletos/extrato/IR), régua de cobrança, mensageria, assinatura eletrônica. |
| **Fase 6 — Inteligência e Mobilidade** | "Otimizar" | Dashboards avançados/BI, portal e app do corretor (offline), camadas avançadas do mapa, conciliação. |

```mermaid
graph LR
    F0[Fase 0<br/>Fundação] --> F1[Fase 1<br/>Estoque e Mapa]
    F1 --> F2[Fase 2<br/>Comercial]
    F2 --> F3[Fase 3<br/>Financeiro]
    F3 --> F4[Fase 4<br/>Comissões e Repasses]
    F4 --> F5[Fase 5<br/>Autoatendimento]
    F5 --> F6[Fase 6<br/>BI e Mobilidade]
```

## 4. Critérios de aceite (exemplos)

Exemplos no formato **Dado / Quando / Então** para orientar o refinamento.

**Resolução de parâmetro por herança (RF-PAR-003, RN-070/071)**
- **Dado** que o índice de correção é **IGP-M** no nível Geral, o **Empreendimento
  X** o sobrescreve para **INCC** e o **Lote 12** não define índice,
- **Quando** o sistema resolve o índice para um contrato do Lote 12,
- **Então** o valor efetivo é **INCC** (herdado do Empreendimento) e a origem é
  exibida como "herdado de Empreendimento".

**Proteção contra propagação acidental (RF-PAR-005, RN-073)**
- **Dado** um contrato assinado com juros de 1% a.m. (resolvidos na venda),
- **Quando** o Empreendimento altera depois o juros para 1,2% a.m. (edição de
  configuração, sem alteração deliberada de contratos vigentes),
- **Então** o contrato vigente **mantém** 1% a.m.; apenas novas vendas usam 1,2%.

**Alteração deliberada em massa de contratos vigentes (RF-PAR-010, RN-076/077)**
- **Dado** 300 contratos ativos com juros de **5% a.m.**,
- **Quando** o gestor aplica uma **alteração em massa** para **1% a.m.** com
  vigência a partir de 01/07/2026, **abrangência = parcelas em aberto**,
  justificativa e aprovação,
- **Então** as **parcelas em aberto** dos 300 contratos são recalculadas a
  **1%** e as **parcelas já pagas não são tocadas**, gerando aditivo/histórico e
  auditoria;
- **E**, se também marcasse **parcelas já pagas**, o sistema apuraria o valor
  pago a mais e geraria **crédito** ao cliente.

**Importação de lotes (RF-LOT-010)**
- **Dado** um arquivo KML válido com 120 polígonos,
- **Quando** o backoffice importa no empreendimento X,
- **Então** os 120 lotes são criados com vértices, área e perímetro calculados,
  e um relatório de validação aponta eventuais sobreposições ou polígonos não
  fechados.

**Reserva com expiração (RF-RES-001, RN-010)**
- **Dado** um lote Disponível e um prazo de reserva de 48 h,
- **Quando** o corretor reserva o lote,
- **Então** o lote fica Reservado, indisponível para outros, e é liberado
  automaticamente se não houver proposta em 48 h.

**Simulação e alçada (RF-VEN-002/003, RN-025)**
- **Dado** um corretor com alçada de 5% de desconto,
- **Quando** ele monta uma proposta com 8% de desconto,
- **Então** a proposta fica pendente de aprovação do gestor antes de virar venda.

**Baixa automática por CNAB (RF-FIN-004, RN-033)**
- **Dado** um arquivo de retorno bancário com pagamentos do dia,
- **Quando** o financeiro processa o retorno,
- **Então** as parcelas correspondentes são baixadas (sem duplicidade) e o
  extrato do contrato é atualizado.

**Distrato (RF-FIN-010, RN-040/042)**
- **Dado** um contrato com 10 parcelas pagas,
- **Quando** o distrato é aprovado com retenção de 20%,
- **Então** o sistema calcula o valor a devolver, gera o cronograma de
  devolução, libera o lote (volta a Disponível) e ajusta as comissões.
