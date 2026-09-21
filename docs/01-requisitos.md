# 01 — Requisitos

Sistema de Gestão de Empreendimentos Imobiliários Rurais.

- [1. Escopo](#1-escopo)
- [2. Atores e perfis](#2-atores-e-perfis)
- [3. Visão de módulos](#3-visão-de-módulos)
- [4. Requisitos funcionais](#4-requisitos-funcionais)
- [5. Requisitos não-funcionais](#5-requisitos-não-funcionais)
- [6. Requisitos de conformidade legal e regulatória](#6-requisitos-de-conformidade-legal-e-regulatória)
- [7. Integrações externas](#7-integrações-externas)
- [8. Restrições e decisões de arquitetura em aberto](#8-restrições-e-decisões-de-arquitetura-em-aberto)
- [9. Premissas e questões em aberto](#9-premissas-e-questões-em-aberto)

---

## 1. Escopo

### 1.1 Dentro do escopo

- Gestão de **empreendimentos rurais** (loteamentos, condomínios de chácaras,
  áreas de recreio) e seus **lotes** georreferenciados, suportando **os dois
  modelos jurídicos**: loteamento/desmembramento com **matrícula individual por
  lote** e **condomínio por fração ideal**.
- **Estoque e disponibilidade** de lotes com mapa/espelho de vendas
  (coordenadas e **dimensões planas**, sem complexidade topográfica).
- **CRM comercial**: leads, funil, reservas, propostas.
- **Contratos** e documentação da venda.
- **Vendas nas três formas de pagamento**: **à vista**, **financiamento
  bancário** e **financiamento próprio (carteira)**.
- **Financeiro de recebíveis** do financiamento próprio: carnês, **juros
  mensais**, **correção pelo IGP-M a partir do 13º mês**, cobrança,
  inadimplência, distrato e repasses. Métodos e índices **configuráveis**.
- **Comissionamento** de corretores e parceiros (à vista e conforme recebimento).
- **Portais web** de autoatendimento (cliente e corretor).
- **Relatórios e indicadores** gerenciais.

### 1.2 Fora do escopo (nesta versão)

- **Interface/ferramentas de topografia** e tratamento de **complexidade
  topográfica** (relevo, curvas de nível, modelagem 3D, edição CAD). O sistema
  apenas **armazena e exibe** coordenadas e dimensões planas já definidas.
- **Aplicativo mobile** — o acesso nesta etapa é **somente via navegador (web)**.
- Sistema de **execução de obras/engenharia** do loteamento (cronograma físico
  detalhado, diário de obra) — apenas o acompanhamento macro de fases é previsto.
- **Contabilidade fiscal completa** (SPED, apuração de tributos) — prevê-se
  **integração** com ERP/contábil, não a substituição dele.
- **Cartório eletrônico** — prevê-se controle do processo de registro, não a
  execução do registro em si.
- Marketplace/portal público de anúncios — prevê-se **integração** com portais,
  não a construção de um portal público próprio.

## 2. Atores e perfis

| Ator | Descrição | Acesso típico |
|------|-----------|---------------|
| **Administrador** | Configura o sistema, parâmetros, usuários e empresas. | Total |
| **Gestor comercial / Gerente de vendas** | Acompanha funil, aprova propostas/descontos, define metas. | Comercial + relatórios |
| **Corretor / Vendedor** | Atende leads, reserva lotes, cria propostas. | CRM, estoque, próprias comissões |
| **Imobiliária parceira** | Empresa parceira que também vende os lotes. | Estoque e reservas (escopo limitado) |
| **Financeiro / Cobrança** | Gere recebíveis, boletos, baixas, inadimplência, distratos. | Financeiro |
| **Jurídico** | Cuida de contratos, distratos, documentação e registro. | Contratos e documentos |
| **Backoffice / Cadastro** | Cadastra empreendimentos, lotes e importa dados geográficos. | Cadastros |
| **Diretoria / Sócios** | Consome indicadores e relatórios estratégicos. | Dashboards (leitura) |
| **Loteador / Proprietário da gleba** | Dono da terra; recebe repasses e acompanha as vendas. | Portal de repasses (leitura) |
| **Cliente / Comprador** | Adquire o lote e acompanha seu contrato e pagamentos. | Portal do cliente |
| **Sistema (integrações)** | Bancos, assinatura eletrônica, mapas, mensageria, ERP. | APIs |

## 3. Visão de módulos

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            ADMINISTRAÇÃO E SEGURANÇA                        │
│              (usuários, perfis/RBAC, multiempresa, auditoria)              │
└──────────────────────────────────────────────────────────────────────────┘
┌───────────────┐  ┌──────────────────┐  ┌───────────────────────────────┐
│ EMPREENDIMENTOS│→ │ LOTES + GEORREF.  │→ │  MAPA / ESPELHO DE VENDAS      │
└───────────────┘  └──────────────────┘  └───────────────────────────────┘
        │                    │                          │
        ▼                    ▼                          ▼
┌──────────────────────────────────────────────────────────────────────────┐
│  CRM (leads) → RESERVAS → PROPOSTAS → VENDAS → CONTRATOS                   │
└──────────────────────────────────────────────────────────────────────────┘
        │                                              │
        ▼                                              ▼
┌───────────────────────────────┐          ┌───────────────────────────────┐
│ FINANCEIRO (recebíveis,        │          │ COMISSÕES                      │
│ boletos, cobrança, distrato)   │          │ (cálculo, split, pagamento)    │
└───────────────────────────────┘          └───────────────────────────────┘
        │                                              │
        ▼                                              ▼
┌───────────────────────────────┐          ┌───────────────────────────────┐
│ CONTAS A PAGAR/RECEBER,        │          │ RELATÓRIOS E BI                │
│ REPASSES, CONCILIAÇÃO          │          │ (VGV, recebíveis, inadimpl.)   │
└───────────────────────────────┘          └───────────────────────────────┘
┌───────────────┐  ┌──────────────────┐  ┌───────────────────────────────┐
│ GED/DOCUMENTOS │  │ PORTAL DO CLIENTE │  │ PORTAL DO CORRETOR            │
└───────────────┘  └──────────────────┘  └───────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────────┐
│  INTEGRAÇÕES (bancos/CNAB/PIX, assinatura eletrônica, mapas, Receita/CEP,  │
│              mensageria WhatsApp/e-mail/SMS, ERP/contábil, portais)        │
└──────────────────────────────────────────────────────────────────────────┘
```

### 3.1 Princípio transversal — Parametrização hierárquica com herança

Para o sistema **não ser engessado** e se adaptar à realidade de cada
empreendimento e ao formato de negociação de cada empresa usuária, as **regras e
parâmetros** de negócio são resolvidos por uma **cadeia hierárquica com herança
e sobrescrita**:

```
Geral (sistema)  →  Empreendimento  →  Setor/Quadra  →  Lote  →  Contrato
   (menos específico)  ───────────────────────────────►   (mais específico)
```

- Cada parâmetro pode ser **definido em qualquer nível aplicável**.
- O **valor efetivo** é o do **nível mais específico** que define o parâmetro;
  na ausência, herda-se do nível imediatamente acima, até o nível **Geral** e,
  por fim, o **default** do sistema.
- Um nível mais específico **sobrescreve** (override) o valor herdado; remover o
  override restaura a herança.
- Cada parâmetro declara **em quais níveis** pode ser definido (ex.: "prazo de
  reserva" faz sentido em Geral/Empreendimento; "taxa de juros ao mês" pode
  chegar até o Contrato).

Resolução (do mais específico para o mais geral):

```
valorEfetivo(P, alvo) =
    contrato.P  ??  lote.P  ??  setor.P  ??  empreendimento.P  ??  geral.P  ??  default(P)
```

**Congelamento no contrato (snapshot):** ao **efetivar a venda / gerar o
contrato**, os parâmetros efetivos relevantes são **resolvidos e persistidos no
contrato**. Assim, mudanças posteriores nos níveis superiores **não alteram
contratos vigentes** (coerente com RN-021 e RN-073). É uma decisão de projeto
recomendada — ver [`03-regras-de-negocio.md`](03-regras-de-negocio.md#parametrização-hierárquica).

**Exemplos de parâmetros cascateáveis:** índice de correção, carência de
correção, taxa de juros ao mês, método de amortização, multa e juros de mora,
entrada mínima, prazo/nº de parcelas, alçadas de desconto, prazo de reserva,
régua de cobrança, tabela/percentual e regra de comissão, percentual de retenção
no distrato, valor por m² e fatores de valorização.

**Rastreabilidade:** para qualquer valor efetivo, o sistema indica **de qual
nível ele veio** (definido/herdado/sobrescrito) e registra as alterações em
auditoria. Os requisitos correspondentes estão no módulo **`PAR`** (seção 4.16).

## 4. Requisitos funcionais

> Prioridade segundo **MoSCoW**: **M** = Must (essencial ao MVP), **S** = Should,
> **C** = Could, **W** = Won't (por ora). O backlog consolidado por fase está em
> [`04-backlog-e-roadmap.md`](04-backlog-e-roadmap.md).

### 4.1 Módulo Empreendimentos (`EMP`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-EMP-001 | Cadastrar empreendimento com dados gerais: nome, tipo (loteamento, condomínio, chácaras de recreio), município/UF, endereço/acesso, área total e situação (em aprovação, em lançamento, em vendas, esgotado, entregue). | M |
| RF-EMP-002 | Registrar dados jurídicos: matrícula-mãe, cartório de registro de imóveis, nº de registro do parcelamento/loteamento, proprietário(es)/loteador(es). | M |
| RF-EMP-003 | Registrar dados fiscais/rurais: CCIR (INCRA), NIRF/ITR, código do imóvel rural. | S |
| RF-EMP-004 | Registrar dados ambientais: nº do CAR, licenças ambientais, Reserva Legal e Áreas de Preservação Permanente (APP). | S |
| RF-EMP-005 | Definir o **perímetro georreferenciado** do empreendimento (polígono) e o sistema de coordenadas/datum utilizado (ex.: SIRGAS 2000, UTM). | M |
| RF-EMP-006 | Organizar o empreendimento em **fases/etapas de lançamento**, **quadras** e/ou **setores**. | M |
| RF-EMP-007 | Acompanhar **fases de infraestrutura/obra** em nível macro (terraplenagem, vias, energia, água, portaria, paisagismo) com percentual de conclusão. | S |
| RF-EMP-008 | Anexar documentos do empreendimento (planta aprovada, memorial descritivo, licenças, ART/RRT, contrato com o loteador). | M |
| RF-EMP-009 | Definir a **tabela de preços** vigente do empreendimento e seu histórico de reajustes. | M |
| RF-EMP-010 | Configurar **condições comerciais padrão** por empreendimento (entrada mínima, prazos, índice de correção, juros, carência de correção, alçadas de desconto). | M |
| RF-EMP-011 | Definir o **modelo jurídico** do empreendimento — **loteamento/desmembramento** (matrícula individual por lote) ou **condomínio** (fração ideal) — refletindo-o na titularidade do lote, no contrato e no fluxo de registro. | M |

### 4.2 Módulo Lotes e Georreferenciamento (`LOT`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-LOT-001 | Cadastrar lote com identificação (nº do lote, quadra/setor, fase) e vinculação ao empreendimento. | M |
| RF-LOT-002 | Registrar as **coordenadas dos vértices** do lote (polígono), com o sistema de coordenadas/datum. | M |
| RF-LOT-003 | Registrar **dimensões planas**: medidas de frente, fundos e laterais, além de **área** e **perímetro**. | M |
| RF-LOT-004 | **Calcular automaticamente** área e perímetro planos a partir do polígono de vértices, permitindo comparar com os valores do memorial descritivo. | M |
| RF-LOT-005 | Registrar **confrontações/limites** (confrontantes em cada face) e a matrícula individual do lote (quando houver desmembramento). | S |
| RF-LOT-006 | Classificar o lote por **tipo/uso** (residencial, comercial, misto, área institucional, área verde, sistema viário, área não comercializável). | M |
| RF-LOT-007 | Registrar **características físicas** como rótulos simples (ex.: plano/aclive/declive; esquina; frente para lago/rua/área verde) usadas como **fatores de valorização** — sem modelagem de relevo/topografia. | S |
| RF-LOT-008 | Definir **preço do lote**: valor base, valor por m², e aplicação de fatores de valorização/deságio. | M |
| RF-LOT-009 | Controlar o **status do lote**: disponível, reservado, em proposta, vendido, quitado, bloqueado, permutado, caução/garantia, indisponível. | M |
| RF-LOT-010 | **Importar lotes em massa** a partir de planilha (CSV/XLSX) e de arquivos geográficos simples (**KML/KMZ, GeoJSON**), com pré-visualização e validação. Shapefile é opcional/futuro; **formatos CAD (DXF/DWG) estão fora do escopo**. | M |
| RF-LOT-011 | **Exportar** os lotes e o perímetro em formatos geográficos (KML, GeoJSON) e planilha. | S |
| RF-LOT-012 | Manter **histórico** de cada lote (mudanças de preço, status, reservas, vendas, distratos). | M |
| RF-LOT-013 | Detectar **inconsistências geométricas** (sobreposição entre lotes, vértices duplicados, polígono não fechado) na importação/edição. | C |
| RF-LOT-014 | Suportar **remembramento/desmembramento** de lotes (unir ou dividir), preservando rastreabilidade. | C |

### 4.3 Módulo Mapa Interativo / Espelho de Vendas (`MAP`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-MAP-001 | Exibir o empreendimento em **mapa interativo** com camada de satélite e/ou planta do loteamento (imagem georreferenciada). | M |
| RF-MAP-002 | Sobrepor os **polígonos dos lotes** ao mapa, **coloridos por status** (ex.: verde = disponível, amarelo = reservado, vermelho = vendido). | M |
| RF-MAP-003 | Selecionar um lote no mapa e ver seus **detalhes** (dimensões, preço, status, condições) em painel lateral. | M |
| RF-MAP-004 | Executar ações a partir do mapa conforme permissão: **reservar**, **iniciar proposta**, **bloquear**. | M |
| RF-MAP-005 | **Filtrar** o mapa por status, faixa de preço, faixa de área, quadra/fase e características. | M |
| RF-MAP-006 | Oferecer **espelho de vendas** em formato de grade/lista (visão tabular do estoque) sincronizado com o mapa. | M |
| RF-MAP-007 | Alternar/sobrepor **camadas** informativas (APP, Reserva Legal, infraestrutura, áreas comuns). | C |
| RF-MAP-008 | Ferramentas de **medição** de distância e área diretamente no mapa. | C |
| RF-MAP-009 | Exibir mapa em **modo público/apresentação** (sem preços sensíveis) para uso comercial com o cliente. | S |

### 4.4 Módulo CRM — Leads e Clientes (`CRM`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-CRM-001 | Cadastrar **leads/prospects** com origem (site, landing page, portal, indicação, redes sociais, stand). | M |
| RF-CRM-002 | **Capturar leads automaticamente** via integração (formulários web, portais, WhatsApp) e por importação. | S |
| RF-CRM-003 | Gerenciar o **funil de vendas** com etapas configuráveis (novo, em atendimento, visita, proposta, negociação, ganho, perdido) e motivos de perda. | M |
| RF-CRM-004 | **Distribuir leads** a corretores por regra (rodízio, por empreendimento, por origem) e permitir redistribuição. | S |
| RF-CRM-005 | Registrar **interações/atendimentos** (ligações, mensagens, e-mails, anotações) na linha do tempo do lead. | M |
| RF-CRM-006 | **Agendar visitas** ao empreendimento e gerar lembretes/notificações. | S |
| RF-CRM-007 | Converter lead em **cliente** com cadastro completo: PF (CPF, RG, estado civil, cônjuge, profissão, renda) ou PJ (CNPJ, representantes), endereço, contatos e dados bancários. | M |
| RF-CRM-008 | Validar documentos e dados (CPF/CNPJ, CEP) e **evitar duplicidade** de cadastro. | S |
| RF-CRM-009 | Suportar **múltiplos compradores** por venda (coproprietários) e representação (procurador). | S |
| RF-CRM-010 | Registrar **consentimentos LGPD** e preferências de contato. | M |

### 4.5 Módulo Reservas (`RES`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-RES-001 | Reservar um lote **disponível**, vinculando lead/cliente e corretor, com data/hora e **prazo de expiração** configurável. | M |
| RF-RES-002 | **Bloquear automaticamente** o lote enquanto a reserva estiver ativa, impedindo dupla reserva. | M |
| RF-RES-003 | **Expirar automaticamente** reservas vencidas, liberando o lote e notificando o corretor. | M |
| RF-RES-004 | Exigir **aprovação** da reserva pelo gestor quando fora dos parâmetros (ex.: prazo estendido). | C |
| RF-RES-005 | Manter **fila de interesse / lista de espera** por lote reservado. | C |
| RF-RES-006 | **Prorrogar** ou **cancelar** reserva com registro de motivo e responsável. | S |
| RF-RES-007 | **Converter** reserva em proposta/venda preservando o vínculo (lote, cliente, corretor). | M |

### 4.6 Módulo Propostas e Vendas (`VEN`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-VEN-001 | Registrar **proposta comercial** de um lote com o cliente, corretor e condições pretendidas. | M |
| RF-VEN-002 | **Simular condições de pagamento**: valor à vista, entrada/sinal, nº e valor de parcelas, parcelas intermediárias (balões), parcela final, índice de correção e juros. | M |
| RF-VEN-003 | Aplicar **descontos** respeitando alçadas por perfil; acima do limite, exigir **aprovação** do gestor. | M |
| RF-VEN-004 | Registrar **contraproposta/negociação** com histórico de versões da proposta. | S |
| RF-VEN-005 | Suportar as **três formas de pagamento**: **à vista**, **financiamento bancário** e **financiamento próprio (carteira)**, cada uma com seu fluxo, documentação e reflexo no plano financeiro. | M |
| RF-VEN-006 | **Efetivar a venda** (proposta aprovada → venda), atualizando o status do lote para "vendido" e disparando a geração de contrato e do plano financeiro. | M |
| RF-VEN-007 | Gerar **número/identificador único** da venda e do contrato. | M |
| RF-VEN-008 | Suportar **cancelamento/distrato** da venda com as regras financeiras associadas (ver módulo Financeiro). | M |
| RF-VEN-009 | Registrar **checklist de documentos** exigidos para a venda e controlar pendências (ver GED). | S |
| RF-VEN-010 | Suportar **permuta** e **dação em pagamento** como composição do negócio (entrada/parte do pagamento). | S |

### 4.7 Módulo Contratos (`CTR`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-CTR-001 | Gerar **contrato** (ex.: promessa de compra e venda) a partir de **templates** com preenchimento automático dos dados da venda, cliente e lote. | M |
| RF-CTR-002 | Manter **modelos de contrato** versionados e com campos dinâmicos (merge fields). | M |
| RF-CTR-003 | Enviar o contrato para **assinatura eletrônica** e acompanhar o status (enviado, visualizado, assinado). | S |
| RF-CTR-004 | Gerar **aditivos** (alteração de condições, cessão de direitos, mudança de titularidade). | S |
| RF-CTR-005 | Registrar e controlar **distratos/rescisões** com o respectivo documento. | M |
| RF-CTR-006 | Acompanhar o **processo de registro em cartório** e a emissão da matrícula individual (controle de etapas e prazos). | C |
| RF-CTR-007 | Armazenar os contratos assinados no **GED** vinculados à venda. | M |

### 4.8 Módulo Financeiro — Recebíveis (`FIN`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-FIN-001 | Gerar o **plano de pagamento** (cronograma de parcelas) da venda: entrada, parcelas mensais, intermediárias (balões) e parcela final. | M |
| RF-FIN-002 | Aplicar **juros** (métodos configuráveis: Price, SAC, juros simples) e **correção monetária** por índice contratado (IGP-M, INCC, IPCA, etc.), com **carência de correção configurável**. Regra atual do cliente: **juros ao mês** + **correção pelo IGP-M somente a partir do 13º mês** de contrato. | M |
| RF-FIN-003 | **Emitir boletos/carnê** (integração bancária) e disponibilizar a **2ª via**; suportar **PIX** (QR/copia-e-cola). | M |
| RF-FIN-004 | Registrar **recebimentos**: baixa manual, baixa automática por **retorno bancário (CNAB 240/400)** e conciliação por PIX. | M |
| RF-FIN-005 | Calcular **juros, multa e correção por atraso** na quitação de parcelas vencidas. | M |
| RF-FIN-006 | Operar a **régua de cobrança**: lembretes de vencimento e avisos de atraso por e-mail/SMS/WhatsApp, escalonados por dias de atraso. | S |
| RF-FIN-007 | Gerir **inadimplência**: painel de parcelas vencidas, notificação/interpelação e marcação de contratos inadimplentes. | M |
| RF-FIN-008 | **Renegociar dívida**: reparcelamento, acordo, alteração de vencimento/índice, com histórico e novo plano. | S |
| RF-FIN-009 | Permitir **antecipação de parcelas** com regra de desconto (deságio) configurável. | S |
| RF-FIN-010 | Processar **distrato**: cálculo do valor a devolver ao cliente (retenções, taxas administrativas), cronograma de devolução e liberação do lote. | M |
| RF-FIN-011 | Emitir **recibos** e demonstrativos, e gerar o **informe de rendimentos / demonstrativo anual** para o comprador. | S |
| RF-FIN-012 | Manter **extrato financeiro** por contrato (pagas, a vencer, vencidas, saldo devedor atualizado). | M |
| RF-FIN-013 | Projetar **recebíveis futuros** (fluxo de caixa previsto) por empreendimento e consolidado. | S |
| RF-FIN-014 | Suportar **múltiplas contas/carteiras bancárias** e centros de custo por empreendimento. | S |

### 4.9 Módulo Comissões (`COM`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-COM-001 | Cadastrar **corretores** (com CRECI) e **imobiliárias parceiras**. | M |
| RF-COM-002 | Configurar **tabelas de comissão** por empreendimento/campanha: percentual, faixas, valores fixos. | M |
| RF-COM-003 | **Calcular a comissão** de cada venda e distribuir em **split** entre os envolvidos (corretor, captador, gerente, imobiliária). | M |
| RF-COM-004 | Definir a **regra de pagamento** da comissão: à vista, parcelada ou **conforme o recebimento** das parcelas do cliente. | S |
| RF-COM-005 | Gerar **extrato de comissões** por corretor/parceiro e controlar o status (a pagar, pago). | S |
| RF-COM-006 | **Estornar/ajustar** comissão em caso de distrato ou inadimplência conforme política. | S |
| RF-COM-007 | Definir **metas** e acompanhar o **desempenho** de vendas por corretor/equipe. | C |

### 4.10 Módulo Contas a Pagar/Receber, Repasses e Conciliação (`FCX`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-FCX-001 | Registrar **contas a receber** (parcelas dos clientes) e **contas a pagar** (comissões, repasses, despesas do empreendimento). | S |
| RF-FCX-002 | Calcular e controlar o **repasse ao loteador/proprietário** conforme o contrato (percentual/valor por venda ou por recebimento). | S |
| RF-FCX-003 | Consolidar o **fluxo de caixa** (realizado e projetado) por empreendimento e por empresa. | S |
| RF-FCX-004 | Realizar **conciliação bancária** (extrato x lançamentos) com apoio do retorno CNAB/PIX. | C |
| RF-FCX-005 | **Exportar** lançamentos para o ERP/contabilidade (integração ou arquivo). | S |

### 4.11 Módulo Documentos / GED (`GED`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-GED-001 | Armazenar documentos vinculados a empreendimento, lote, cliente, venda e contrato. | M |
| RF-GED-002 | Definir **checklists de documentos** por tipo de operação e controlar **pendências**. | S |
| RF-GED-003 | Versionar documentos e registrar autor/data de cada upload. | S |
| RF-GED-004 | Controlar **validade** de documentos/licenças e alertar sobre vencimentos. | C |

### 4.12 Módulo Relatórios e BI (`REL`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-REL-001 | **Dashboard comercial**: VGV (Valor Geral de Vendas), unidades vendidas x estoque, ticket médio, vendas por período/empreendimento/corretor. | M |
| RF-REL-002 | **Espelho de vendas** exportável (PDF/Excel) com o status de cada lote. | M |
| RF-REL-003 | **Relatório de inadimplência** (aging de parcelas vencidas) e de **recebíveis futuros**. | M |
| RF-REL-004 | **Relatório de comissões** por corretor/parceiro e período. | S |
| RF-REL-005 | **Desempenho de corretores** (conversão do funil, vendas, metas). | C |
| RF-REL-006 | **Relatórios customizáveis** e exportação (PDF, Excel, CSV). | C |
| RF-REL-007 | **Repasses ao loteador** e demonstrativos por empreendimento. | S |

### 4.13 Módulo Administração e Segurança (`ADM`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-ADM-001 | Gerir **usuários** (criação, bloqueio, reset de senha) e **autenticação** (senha forte, 2FA opcional). | M |
| RF-ADM-002 | Definir **perfis e permissões (RBAC)** granulares por módulo/ação e por empreendimento. | M |
| RF-ADM-003 | Suportar **multiempresa/multifilial** e **multiempreendimento** com segregação de dados. | S |
| RF-ADM-004 | Registrar **trilha de auditoria** (quem fez o quê e quando) nas operações sensíveis (preço, status de lote, baixa financeira, distrato). | M |
| RF-ADM-005 | Parametrizar o sistema: índices de correção, taxas, alçadas, templates, régua de cobrança, tabelas de comissão — seguindo o modelo **hierárquico com herança** (ver módulo `PAR` e seção 3.1). | M |
| RF-ADM-006 | Gerir **consentimentos e solicitações LGPD** (acesso, correção, exclusão/anonimização de dados). | S |

### 4.14 Portal do Cliente (`PCL`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-PCL-001 | Acesso autenticado do comprador ao seu(s) contrato(s). | S |
| RF-PCL-002 | Emitir **2ª via de boleto** e **PIX** das parcelas. | S |
| RF-PCL-003 | Consultar **extrato**: parcelas pagas, a vencer e vencidas, com saldo devedor atualizado. | S |
| RF-PCL-004 | Baixar **contratos e documentos** e o **informe anual** para IR. | S |
| RF-PCL-005 | Atualizar **dados cadastrais** e abrir **solicitações/atendimento**. | C |
| RF-PCL-006 | Acompanhar o **andamento da obra/infraestrutura** do empreendimento. | C |

### 4.15 Portal do Corretor (`PCO`)

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-PCO-001 | Consultar o **estoque disponível** e o mapa/espelho de vendas. | S |
| RF-PCO-002 | **Reservar** lotes e registrar **propostas** pelo portal. | S |
| RF-PCO-003 | Acompanhar **comissões** e extratos. | S |
| RF-PCO-004 | Acessar **materiais de venda** (plantas, tabela, apresentações). | C |
| RF-PCO-005 | **App mobile** com uso em campo (modo offline) — **fora do escopo inicial**; nesta etapa o acesso é somente via navegador (web responsivo). | W |

### 4.16 Módulo Parametrização Hierárquica (`PAR`) — transversal/fundacional

> Mecanismo de configuração que permeia todos os módulos (ver seção 3.1).
> Fundacional: entra na **Fase 0**.

| ID | Requisito | Prior. |
|----|-----------|:------:|
| RF-PAR-001 | Manter um **catálogo de parâmetros** de negócio, cada um com chave, tipo de dado, **níveis aplicáveis**, valor **default** e descrição. | M |
| RF-PAR-002 | Definir/editar valores de parâmetros em cada nível da cadeia (**Geral, Empreendimento, Setor/Quadra, Lote, Contrato**), respeitando os níveis aplicáveis de cada parâmetro. | M |
| RF-PAR-003 | **Resolver o valor efetivo** de um parâmetro para um alvo (lote/contrato), aplicando **herança e sobrescrita** (o mais específico vence; default como último recurso). | M |
| RF-PAR-004 | Exibir, para cada valor, a **origem** (definido neste nível / herdado de X / sobrescrito) e permitir **pré-visualizar o valor efetivo** de um lote/contrato. | S |
| RF-PAR-005 | **Congelar (snapshot)** os parâmetros efetivos no contrato ao efetivar a venda, de modo que alterações posteriores em níveis superiores **não afetem contratos vigentes**. | M |
| RF-PAR-006 | Controlar **quem pode definir/sobrescrever** cada parâmetro em cada nível (RBAC + alçadas), especialmente os sensíveis (juros, índice, retenção, alçada). | S |
| RF-PAR-007 | **Auditar** alterações de parâmetros (parâmetro, nível, valor anterior/novo, autor, data/hora). | M |
| RF-PAR-008 | Suportar **extensibilidade**: incluir novos parâmetros (e, idealmente, novos níveis) sem alteração estrutural do sistema. | C |
| RF-PAR-009 | Ao alterar um parâmetro em um nível, **sinalizar o impacto** (quantos itens subordinados sem override serão afetados). | C |

## 5. Requisitos não-funcionais

| ID | Categoria | Requisito |
|----|-----------|-----------|
| RNF-001 | **Usabilidade** | Interface responsiva (desktop/tablet/mobile) e em **português (pt-BR)**; fluxos comerciais otimizados para poucos cliques. |
| RNF-002 | **Localização** | Moeda em Real (R$), formatos de data/número pt-BR, fuso horário configurável. |
| RNF-003 | **Desempenho** | O mapa deve renderizar com fluidez empreendimentos com **milhares de lotes** (ex.: até ~5.000 polígonos) com recursos de simplificação/clusterização. |
| RNF-004 | **Desempenho** | Operações comuns (abrir estoque, gerar simulação, emitir boleto) devem responder em tempo interativo (meta: < 2 s em cenário típico). |
| RNF-005 | **Disponibilidade** | Alvo de disponibilidade ≥ 99,5%; janelas de manutenção comunicadas. |
| RNF-006 | **Escalabilidade** | Arquitetura que suporte crescimento de empreendimentos, contratos ativos e portais externos sem reengenharia. |
| RNF-007 | **Segurança** | Criptografia em trânsito (TLS) e em repouso para dados sensíveis; senhas com hash forte; 2FA opcional; princípio do menor privilégio. |
| RNF-008 | **Privacidade / LGPD** | Base legal e consentimento para tratamento de dados pessoais; anonimização/exclusão sob solicitação; registro de operações sobre dados pessoais. |
| RNF-009 | **Auditabilidade** | Log imutável das operações sensíveis (financeiras, de preço e de status), com retenção definida. |
| RNF-010 | **Backup e continuidade** | Rotina de backup automático e plano de recuperação (RPO/RTO a definir). |
| RNF-011 | **Integridade financeira** | Cálculos monetários com precisão adequada (evitar erros de arredondamento); operações financeiras transacionais e idempotentes (ex.: baixa de boleto). |
| RNF-012 | **Confiabilidade geográfica** | Precisão e consistência dos cálculos de área/perímetro; suporte explícito ao datum/sistema de coordenadas; preservação da precisão dos vértices. |
| RNF-013 | **Integrações** | Comunicação resiliente com serviços externos (retentativas, filas, tratamento de indisponibilidade) sem travar a operação. |
| RNF-014 | **Manutenibilidade** | Parametrização sem necessidade de deploy para índices, taxas, alçadas e templates. |
| RNF-015 | **Acessibilidade** | Boas práticas de acessibilidade (contraste, navegação por teclado) nos portais públicos. |
| RNF-016 | **Observabilidade** | Monitoramento, métricas e alertas (erros, filas de integração, jobs de cobrança). |
| RNF-017 | **Compatibilidade** | Suporte aos navegadores modernos mais usados; portais leves para conexões móveis. |

## 6. Requisitos de conformidade legal e regulatória

> ⚠️ **Itens a validar com as áreas jurídica e contábil.** Servem de checklist de
> conformidade e podem impactar o modelo de dados e os fluxos. Não constituem
> orientação jurídica.

| ID | Tema | Observação |
|----|------|-----------|
| RF-LEG-001 | **Parcelamento do solo** | Registrar aprovação/registro do parcelamento no órgão competente e no cartório; refletir a modelagem escolhida (loteamento/desmembramento com matrícula individual x condomínio por fração ideal). |
| RF-LEG-002 | **INCRA / Imóvel rural** | Guardar CCIR, código do imóvel rural e informações de georreferenciamento/certificação, quando aplicável. |
| RF-LEG-003 | **Ambiental** | CAR, Reserva Legal e APP como camadas/atributos e como restrições de comercialização de áreas não edificáveis. |
| RF-LEG-004 | **Tributação** | Suporte a emissão/registro de tributos aplicáveis (ex.: ITR do imóvel; ISS sobre comissões/serviços; ITBI no momento oportuno), preferencialmente via integração contábil. |
| RF-LEG-005 | **Contratos e consumidor** | Cláusulas obrigatórias, regras de distrato e retenção conforme legislação vigente. |
| RF-LEG-006 | **LGPD** | Tratamento, consentimento, retenção e direitos do titular (ver RNF-008 e RF-ADM-006). |

## 7. Integrações externas

| ID | Integração | Uso |
|----|-----------|-----|
| RF-INT-001 | **Banco / meios de pagamento** | Emissão e registro de boletos, retorno CNAB 240/400 para baixa automática, PIX (cobrança e conciliação). |
| RF-INT-002 | **Assinatura eletrônica** | Envio e acompanhamento de contratos/aditivos/distratos (ex.: provedores de e-signature). |
| RF-INT-003 | **Mapas e cartografia** | Base de mapas/satélite e renderização de camadas (ex.: provedores de tiles/mapas); leitura de formatos geográficos simples (KML/KMZ, GeoJSON) e planilha. Sem suporte a formatos CAD (DXF/DWG). |
| RF-INT-004 | **Receita Federal / cadastros** | Validação de CPF/CNPJ e situação cadastral. |
| RF-INT-005 | **CEP / endereço** | Autopreenchimento de endereço por CEP. |
| RF-INT-006 | **Mensageria** | Envio de e-mail, SMS e **WhatsApp** para cobrança, avisos e marketing. |
| RF-INT-007 | **ERP / Contabilidade** | Exportação de lançamentos financeiros e fiscais. |
| RF-INT-008 | **Portais imobiliários / captação** | Recepção de leads de portais, site e landing pages. |
| RF-INT-009 | **BI externo** | Exposição de dados para ferramentas de BI (opcional). |

## 8. Restrições e decisões de arquitetura em aberto

- **Plataforma**: web responsivo como base; app mobile do corretor a decidir
  (nativo, híbrido ou PWA) — ver RF-PCO-005.
- **Multitenancy**: definir se haverá uma instância por imobiliária ou base
  compartilhada com segregação lógica (impacta RF-ADM-003).
- **Provedor de mapas** e **provedor bancário/assinatura**: a selecionar
  (impacta integrações da seção 7).
- **Motor de cálculo financeiro**: decidir entre construir internamente ou
  integrar a um sistema de gestão de recebíveis existente.

## 9. Premissas e decisões

### 9.1 Decisões confirmadas com o cliente

1. **Formas de pagamento**: o sistema suporta **as três** — **à vista**,
   **financiamento bancário** e **financiamento próprio (carteira)**. Todas no
   escopo do MVP. *(RF-VEN-005)*
2. **Financiamento direto (prática atual)**: **juros ao mês** + **correção pelo
   IGP-M somente a partir do 13º mês** de contrato (carência de 12 meses sem
   correção). Os **métodos de amortização e índices são configuráveis** e todos
   devem ser implementados. *(RF-FIN-002, RN-031, RN-031a)*
3. **Modelagem jurídica**: suportar **os dois modelos** — loteamento/
   desmembramento com **matrícula individual por lote** e **condomínio por
   fração ideal**. *(RF-EMP-011)*
4. **Comissão**: suportar **os dois formatos** de pagamento — **à vista** e
   **conforme o recebimento** das parcelas. *(RF-COM-004, RN-052)*
5. **Acesso**: nesta etapa, **somente via navegador (web responsivo)**; **sem
   app mobile**. *(RF-PCO-005 → fora do escopo inicial)*
6. **Georreferenciamento simplificado**: o sistema **armazena e exibe**
   coordenadas (vértices) e **dimensões planas** dos lotes, **sem** interface de
   topógrafo e **sem** tratamento de complexidade topográfica (relevo, curvas de
   nível, edição CAD/DWG). *(seção 1.2; RF-LOT-007/010)*

### 9.2 Premissas adotadas

1. Cada empreendimento é subdividido em **lotes**, opcionalmente agrupados em
   **quadras/setores** e **fases**.
2. "Dimensões planas" = **área/medidas projetadas no plano horizontal**,
   independentes do relevo.
3. A imobiliária pode administrar **vários empreendimentos** de **um ou mais
   loteadores/proprietários**, exigindo controle de **repasses**.
4. Corretores próprios e **imobiliárias parceiras** vendem os lotes, exigindo
   **comissionamento com split**.
5. Na venda por **financiamento bancário**, o repasse do agente financeiro
   quita (à vista, para a loteadora) o valor financiado; o sistema controla o
   processo, não a operação de crédito do banco.

### 9.3 Questões ainda em aberto

1. **Correção do financiamento direto**: a taxa de **juros ao mês** é única ou
   varia por empreendimento/campanha? Após o 13º mês, a correção pelo IGP-M é
   **mensal** ou **anual** (no aniversário)? Há reajuste da parcela ou apenas do
   saldo devedor?
2. **Parcelas intermediárias (balões)**: são praticadas? Em qual periodicidade?
3. **Distrato**: qual a **política de retenção** e o modelo de devolução
   (percentuais, cláusula penal, parcelamento da devolução)?
4. **Split de comissão**: como é a divisão entre captador, corretor, gerente e
   imobiliária?
5. **Volume**: quantos empreendimentos, lotes por empreendimento e contratos
   ativos (para dimensionar RNF de desempenho/escala)?
6. **Portal do cliente**: entra no MVP ou em fase posterior?
7. **Integrações prioritárias**: qual banco (boleto/CNAB/PIX), qual provedor de
   assinatura eletrônica e qual ERP/contábil?
8. **Origem dos dados geográficos**: em qual formato os lotes chegarão
   (KML/KMZ, GeoJSON, planilha) e em qual sistema de coordenadas/datum
   (ex.: SIRGAS 2000, UTM)?
