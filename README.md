# Sistema de Gestão de Empreendimentos Imobiliários Rurais

> Levantamento de requisitos e catálogo de funcionalidades para um sistema de
> gestão de **loteamentos e condomínios rurais**, cobrindo desde o cadastro
> georreferenciado dos lotes até a venda, os contratos, o financiamento próprio
> e o pós-venda.

---

## 1. Contexto

A imobiliária administra **empreendimentos imobiliários rurais** (loteamentos,
condomínios de chácaras, áreas de recreio, glebas parceladas) e realiza a
**venda dos lotes** que os compõem.

- Cada **empreendimento** é subdividido em vários **lotes** (podendo haver
  agrupamento por **quadras** ou **setores**).
- Cada **lote** possui **coordenadas geográficas** (polígono de vértices) e
  **dimensões planas** (frente, fundos, laterais, área e perímetro).
- A venda tipicamente ocorre em **parcelamento de longo prazo com financiamento
  direto** (carteira própria da imobiliária/loteadora), o que torna a **gestão
  financeira de recebíveis** tão central quanto a venda em si.

O sistema precisa unir três mundos que normalmente vivem em planilhas
separadas: o **geográfico/cartográfico** (mapa, lotes, áreas), o **comercial/CRM**
(leads, reservas, propostas, contratos) e o **financeiro** (carnês, cobrança,
inadimplência, comissões, repasses).

## 2. Objetivos do sistema

1. Centralizar o cadastro dos empreendimentos e o **estoque de lotes** com
   informação georreferenciada.
2. Oferecer um **mapa/espelho de vendas interativo** com o status de cada lote
   em tempo real (disponível, reservado, vendido, bloqueado, quitado).
3. Gerenciar o **funil comercial** — da captação do lead ao fechamento — com
   reservas, propostas e contratos.
4. Controlar todo o **ciclo financeiro** do financiamento próprio: planos de
   pagamento, correção monetária, boletos, recebimentos, inadimplência,
   distratos e repasses.
5. Calcular e controlar **comissões** de corretores e parceiros.
6. Dar **transparência ao comprador** (2ª via de boleto, extrato, contratos,
   informe para IR) por meio de um portal do cliente.
7. Prover **indicadores gerenciais** (VGV, unidades vendidas, recebíveis futuros,
   inadimplência) para a tomada de decisão.

## 3. Como esta documentação está organizada

| Documento | Conteúdo |
|-----------|----------|
| [`docs/01-requisitos.md`](docs/01-requisitos.md) | Atores, requisitos funcionais por módulo (com IDs rastreáveis) e requisitos não-funcionais. |
| [`docs/02-modelo-de-dominio.md`](docs/02-modelo-de-dominio.md) | Entidades principais, diagrama entidade-relacionamento (ER) e glossário do setor. |
| [`docs/03-regras-de-negocio.md`](docs/03-regras-de-negocio.md) | Regras de negócio consolidadas (RN-xxx). |
| [`docs/04-backlog-e-roadmap.md`](docs/04-backlog-e-roadmap.md) | Backlog de funcionalidades priorizado (MoSCoW) e roadmap sugerido em fases. |
| [`docs/05-motor-calculo-financeiro.md`](docs/05-motor-calculo-financeiro.md) | Motor de cálculo financeiro: métodos (Price/SAC/simples), correção com carência, mora, antecipação, recálculo temporal, requisitos (RF-CALC) e exemplos numéricos. |

Convenção de identificadores usada em todo o material:

- **RF-XXX-000** — Requisito Funcional (o `XXX` indica o módulo, ex.: `RF-LOT-010`).
- **RF-CALC-000** — Requisito do motor de cálculo financeiro.
- **RNF-000** — Requisito Não-Funcional.
- **RN-000** — Regra de Negócio; **RC-00** — Regra de cálculo (motor financeiro).

## 4. Resumo executivo das funcionalidades (módulos)

| # | Módulo | Para quê serve |
|---|--------|----------------|
| 1 | **Empreendimentos** | Cadastro do loteamento: dados jurídicos, ambientais, perímetro, fases de obra e infraestrutura. |
| 2 | **Lotes e Georreferenciamento** | Cadastro dos lotes com vértices, dimensões planas e área/perímetro calculados, com importação simples (KML/KMZ, GeoJSON, planilha) e entrada manual. Sem ferramentas de topografia/CAD. |
| 3 | **Mapa Interativo / Espelho de Vendas** | Visualização dos lotes sobre imagem de satélite/planta, coloridos por status, com filtros e ações (reservar/vender). |
| 4 | **CRM / Leads e Clientes** | Captação e qualificação de leads, funil de vendas, distribuição para corretores, agenda de visitas e cadastro completo do cliente. |
| 5 | **Reservas** | Reserva temporária de lote com expiração, aprovação e fila de espera. |
| 6 | **Propostas e Vendas** | Simulação de condições, negociação, alçadas de desconto e fechamento. |
| 7 | **Contratos** | Geração por template, assinatura eletrônica, aditivos e distratos. |
| 8 | **Financeiro (Recebíveis)** | Planos de pagamento, correção/juros, boletos, baixa (CNAB/PIX), régua de cobrança, renegociação e distrato. |
| 9 | **Comissões** | Tabelas de comissão, cálculo por venda, splits e pagamento conforme recebimento. |
| 10 | **Contas a Pagar/Receber e Repasses** | Fluxo de caixa, repasse ao loteador/proprietário e conciliação bancária. |
| 11 | **Documentos (GED)** | Repositório de documentos e checklist de pendências por venda. |
| 12 | **Relatórios e BI** | Dashboards de vendas, recebíveis, inadimplência e desempenho. |
| 13 | **Administração e Segurança** | Usuários, perfis/permissões (RBAC), multiempresa, auditoria e parâmetros. |
| 14 | **Portal do Cliente** | Autoatendimento do comprador (boletos, extrato, contratos, informe de IR). |
| 15 | **Portal do Corretor** | Estoque, reservas, propostas, comissões e materiais de venda. |
| 16 | **Integrações** | Bancos, assinatura eletrônica, mapas, Receita/CEP, mensageria e contabilidade. |

> **Mecanismo transversal — Parametrização hierárquica.** Para o sistema não ser
> engessado, um mecanismo de parâmetros com **herança e sobrescrita**
> (`Geral → Empreendimento → Setor → Lote → Contrato`, o nível mais específico
> prevalece) permeia todos os módulos. Ver seção 3.1 de
> [`docs/01-requisitos.md`](docs/01-requisitos.md#31-princípio-transversal--parametrização-hierárquica-com-herança).

## 5. Definições do cliente e questões em aberto

As decisões abaixo foram **confirmadas com o cliente** e orientam todo o
material (detalhes na seção 9 de [`docs/01-requisitos.md`](docs/01-requisitos.md#9-premissas-e-decisões)):

- **Formas de pagamento**: o sistema suporta **financiamento próprio (carteira)**,
  **à vista** e **financiamento bancário** — os três são escopo do MVP.
- **Financiamento direto (prática atual)**: **juros ao mês** + **correção pelo
  IGP-M a partir do 13º mês** (sem correção no primeiro ano de contrato). Os
  métodos de amortização e índices são **configuráveis** (todos implementados).
- **Modelagem jurídica**: o sistema suporta **os dois modelos** —
  loteamento/desmembramento com **matrícula individual por lote** e **condomínio
  por fração ideal**.
- **Comissão**: suporta **os dois formatos** de pagamento — **à vista** e
  **conforme o recebimento** das parcelas.
- **Acesso**: inicialmente **somente via navegador (web)**; **sem app mobile**
  nesta etapa.
- **Georreferenciamento**: escopo **simplificado** — armazenar e exibir
  coordenadas e **dimensões planas** dos lotes. **Não** haverá interface de
  topógrafo nem tratamento de **complexidade topográfica** (relevo, curvas de
  nível, edição CAD).
- **Parametrização flexível (arquitetura)**: parâmetros de negócio com
  **herança e sobrescrita** na cadeia `Geral → Empreendimento → Setor → Lote →
  Contrato` (o nível mais específico prevalece), com **congelamento no contrato**
  no momento da venda. Deve ser implementada **desde o início** (Fase 0).

Ainda a dimensionar: **volume** (nº de empreendimentos, lotes por empreendimento
e contratos ativos) e a definição dos **provedores de integração** (banco,
assinatura eletrônica, mapas).

> ⚠️ **Aviso**: os itens de conformidade legal/regulatória (INCRA, CAR, ITR,
> registro em cartório, parcelamento do solo, LGPD) estão listados como
> **requisitos a validar com as áreas jurídica e contábil**. Não substituem
> orientação profissional especializada.
