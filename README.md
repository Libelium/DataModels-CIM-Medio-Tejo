# Modelos de Dados — Plataforma de Gestão Urbana do Médio Tejo

<p align="center">
  <img src="https://img.shields.io/badge/estado-em%20revis%C3%A3o%20ARTE-orange" alt="Estado: em revisão ARTE">
  <img src="https://img.shields.io/badge/NGSI--LD-ETSI%20GS%20CIM%20009-blue" alt="NGSI-LD">
  <img src="https://img.shields.io/badge/formato-JSON%20Schema-000000?logo=json" alt="Formato: JSON Schema">
  <img src="https://img.shields.io/badge/idioma-PT--PT-green" alt="Idioma: PT-PT">
</p>

Este repositório reúne os **modelos de dados utilizados na Plataforma de Gestão Urbana (PGU) da Comunidade Intermunicipal do Médio Tejo (CIMT)**, desenvolvida no âmbito do concurso público **CP/03/2026** e implementada sobre a plataforma **iris360** da Libelium.

O objetivo do repositório é disponibilizar à **[ARTE — Territórios Inteligentes](https://territoriosinteligentes.gov.pt)** os esquemas e exemplos de cada modelo de dados, de forma a permitir a sua revisão e a verificação do alinhamento com o **[Catálogo Nacional de Modelos de Dados (CNMD)](https://territoriosinteligentes.gov.pt)** e, quando aplicável, com os **[Smart Data Models](https://smartdatamodels.org)** da FIWARE Foundation.

## Âmbito

A PGU integra, para a CIMT e para os seus 11 municípios (Abrantes, Alcanena, Constância, Entroncamento, Ferreira do Zêzere, Mação, Ourém, Sardoal, Tomar, Torres Novas e Vila Nova da Barquinha), os dados provenientes das seguintes verticais:

| Vertical | Área temática | Modelo(s) de dados | Referência |
|---|---|---|---|
| V1 | Gestão de transporte e estacionamento |  |  |
| V2 | Fluxo de pessoas e turistas |  |  |
| V3 | Monitoramento do Meio Ambiente e Proteção Civil |  |  |
| V4 | Eficiência energética em edifícios públicos municipais e iluminação pública | `ACMeasurement`, `Anomaly`, `KeyPerformanceIndicator` | [CNMD — Consumo de eletricidade](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Energia/Consumo%20de%20electricidade); Smart Data Models ([Anomaly](https://github.com/smart-data-models/dataModel.Alert/tree/master/Anomaly), [KeyPerformanceIndicator](https://github.com/smart-data-models/dataModel.KeyPerformanceIndicator/tree/master/KeyPerformanceIndicator)) |
| V5 | Gestão de auxílios sociais | `SocialSupportObservation` | [CNMD — Gestão de apoio social](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Governo%20e%20setor%20p%C3%BAblico/Gest%C3%A3o%20de%20apoio%20social) |
| V7 | Gestão de ativos | `WasteContainer`, `WasteContainerModel`, `PointOfInterest` | CNMD ([Níveis de enchimento dos contentores](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Res%C3%ADduos/N%C3%ADveis%20de%20enchimento%20dos%20contentores), [POI](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Comuns/POI)); Smart Data Models ([WasteContainerModel](https://github.com/smart-data-models/dataModel.WasteManagement/tree/master/WasteContainerModel)) |
| V8 | Gestão inteligente de irrigação | `GreenspaceRecord`, `WaterConsumptionObserved`, `WeatherObserved`, `Device`, `Anomaly` | CNMD ([Rega](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/%C3%81gua/Rega), [Consumo de água](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/%C3%81gua/Consumo%20de%20agua), [Observações meteorológicas](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Meteorologia/Observa%C3%A7%C3%B5es%20metereol%C3%B3gicas), [Sensores](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Comuns/Sensores)); Smart Data Models ([Anomaly](https://github.com/smart-data-models/dataModel.Alert/tree/master/Anomaly)) |
| V9 | Gestão de resíduos urbanos |  |  |
| V10 | Saúde e bem-estar inteligente |  |  |
| V11 | Gestão de incidentes | `Open311_ServiceRequest`, `services` | Smart Data Models ([service_requests](https://github.com/smart-data-models/dataModel.IssueTracking/tree/master/service_requests), [services](https://github.com/smart-data-models/dataModel.IssueTracking/tree/master/services)); correspondência com [CNMD — Ocorrências de não emergência](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Popula%C3%A7%C3%A3o%20e%20sociedade/Socio-econ%C3%B3micos/Ocorr%C3%AAncias/Ocorr%C3%AAncias%20de%20n%C3%A3o%20emerg%C3%AAncia) |
| V12 | Monitoramento da qualidade do ar em ambientes internos de edifícios públicos | `AirQualityObserved`, `Alert`, `AirQualityMonitoring` | CNMD ([Observações de qualidade do ar](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Ar/Observa%C3%A7%C3%B5es%20de%20qualidade%20do%20ar), [Alertas e avisos da proteção civil](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Justi%C3%A7a%2C%20sistema%20judici%C3%A1rio%20e%20seguran%C3%A7a%20p%C3%BAblica/Prote%C3%A7%C3%A3o%20e%20socorro/Alertas%20e%20avisos%20da%20prote%C3%A7%C3%A3o%20civil), [Qualidade do ar (zonas e aglomerações)](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Ar/Qualidade%20do%20ar%20%28zonas%20e%20aglomera%C3%A7%C3%B5es%29)) |

> Sempre que existe um modelo equivalente no CNMD, foi esse o modelo adotado. Nos restantes casos foi utilizado o modelo correspondente dos Smart Data Models, conforme previsto no Caderno de Encargos. Eventuais atributos adicionais ou adaptações estão identificados no `README.md` de cada área temática.

## Organização

O repositório está organizado por áreas temáticas, correspondentes às verticais da PGU. Para cada área temática existe:

- um ficheiro `README.md`, que descreve o modelo (ou modelos) usado na área temática, os seus atributos, a origem dos dados e as eventuais diferenças em relação ao modelo de referência (CNMD ou Smart Data Models);
- uma diretoria `exemplos`, que contém um ou mais exemplos de entidades tal como são publicadas na PGU (anonimizados quando necessário);
- uma diretoria `schema`, que contém os JSON Schemas que permitem validar o(s) modelo(s).

```
DataModels-CIM-Medio-Tejo/
├── README.md
├── Eficiência energética em edifícios públicos municipais e iluminação pública/
│   ├── README.md
│   ├── exemplos/
│   └── schema/
├── Fluxo de pessoas e turistas/
│   ├── README.md
│   ├── exemplos/
│   └── schema/
├── Gestão de ativos/
├── Gestão de auxílios sociais/
├── Gestão de incidentes/
├── Gestão de resíduos urbanos/
├── Gestão de transporte e estacionamento/
├── Gestão inteligente de irrigação/
├── Monitoramento da qualidade do ar em ambientes internos de edifícios públicos/
├── Monitoramento do Meio Ambiente e Proteção Civil/
└── Saúde e bem-estar inteligente/
```

## Convenções

- **Protocolo:** os modelos são publicados através da API NGSI-LD (ETSI GS CIM 009) do _context broker_ da plataforma iris360.
- **Identificadores:** as entidades são identificadas por URN, no formato `urn:ngsi-ld:<Tipo>:<identificador>`.
- **Contexto JSON-LD:** cada exemplo inclui o `@context` utilizado, com referência ao contexto do modelo de origem.
- **Datas e horas:** formato ISO 8601 em UTC.
- **Georreferenciação:** os atributos `location` são publicados em GeoJSON (WGS84), conforme a especificação NGSI-LD; a representação em PT-TM06/ETRS89 é feita na camada de visualização da PGU.
- **Unidades:** sempre que possível, indicadas através de `unitCode` (códigos UN/CEFACT).
