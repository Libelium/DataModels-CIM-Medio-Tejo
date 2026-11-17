# Eficiência energética em edifícios públicos municipais e iluminação pública (V4)

## Descrição

Os modelos de dados utilizados para representar a **eficiência energética dos edifícios públicos municipais** são:

- [`ACMeasurement`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Energia/Consumo%20de%20electricidade) do Catálogo Nacional de Modelos de Dados (CNMD), para as medições dos medidores de energia;
- [`Anomaly`](https://github.com/smart-data-models/dataModel.Alert/tree/master/Anomaly) dos Smart Data Models, para os alertas de consumo anómalo;
- [`KeyPerformanceIndicator`](https://github.com/smart-data-models/dataModel.KeyPerformanceIndicator/tree/master/KeyPerformanceIndicator) dos Smart Data Models, para os indicadores de consumo de cada edifício.

O CNMD não tem modelos para anomalias nem para indicadores de desempenho, pelo que nesses dois casos foi adotado o modelo dos Smart Data Models. Todos os modelos utilizam o formato NGSI-LD, sendo compatíveis com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

Nesta fase, a vertical não inclui entidades específicas de iluminação pública (por exemplo, os modelos de luminárias do CNMD).

## Origem dos dados

Os dados provêm de 19 medidores de energia com certificação MID, instalados em edifícios públicos municipais. Cada leitura de um medidor é publicada como uma entidade `ACMeasurement`, através do IoT Agent da plataforma iris360; o fornecedor dos medidores está por definir. Os campos de origem correspondem à tabela de campos da vertical:

| Campo de origem | Modelo | Atributos |
|-----------------|--------|-----------|
| `id_medidor` | `ACMeasurement`, `Anomaly` | `refDevice`, `detectedBy` e o identificador da entidade |
| `edificio_associado` | todos | `areaServed`, `name`, `refBuilding`, `location`, `address` |
| `consumo_kwh` | `ACMeasurement` | `totalActiveEnergyImport`, `activeEnergyImport`, `totalActivePower` |
| `fases` | `ACMeasurement` | `phaseType` |
| `tensao` | `ACMeasurement` | `phaseVoltage`, `phaseToPhaseVoltage` |
| `corrente` | `ACMeasurement` | `current` |
| `alerta_consumo_anomalo` | `Anomaly` | `anomalousProperty`, `thresholdBreach`, `description` |
| `relatorio_analise` | `KeyPerformanceIndicator` | `name`, `kpiValue`, `calculationPeriod`, `calculationFrequency`, `aggregatedData` |

**Os exemplos deste diretório são ilustrativos:** foram gerados com os dados de teste da PGU, com o mesmo formato com que as medições serão publicadas, e não correspondem a medições reais.

## Modelo `ACMeasurement`

Cada medidor tem uma entidade, com o identificador `urn:ngsi-ld:ACMeasurement:<id_medidor>`, estável por medidor; o histórico das leituras obtém-se pela API temporal NGSI-LD.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:ACMeasurement:<id_medidor>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `ACMeasurement`. |
| name | String | Nome do medidor ou quadro elétrico | Ex.: `Paços do Concelho de Tomar — Quadro geral`. |
| areaServed | String | Edifício público onde o medidor está instalado | Ex.: `Paços do Concelho de Tomar`. |
| refDevice | Relationship | Medidor de energia que efetuou a medição | Referência à entidade `Device` do medidor, onde fica registada a certificação MID. |
| refBuilding | Relationship | Edifício público onde o medidor está instalado | Referência à entidade `Building` do edifício, para agregar consumos por edifício. Atributo adicional (ver [Diferenças](#diferenças-em-relação-aos-modelos-de-referência)). |
| location | GeoJSON | Localização do edifício | `Point` com as coordenadas do edifício, atribuído por medidor. |
| address | Object | Morada do edifício | Modelo: [https://schema.org/address](https://schema.org/address). A localidade coincide com o município e a região corresponde à NUTS 2 conforme a nomenclatura do INE. |
| address.addressCountry | String | O país | Valor `Portugal`. |
| address.addressLocality | String | A localidade, coincidente com o município | Ex.: `Tomar`. |
| address.addressRegion | String | A região NUTS 2 | Valor `Oeste e Vale do Tejo`. |
| address.district | String | O distrito | Valor `Santarém`. |
| address.streetAddress | String | Endereço da rua | Ex.: `Praça da República`. |
| dataProvider | String | Fornecedor da entidade de dados | Valor `cimt` (CIM Médio Tejo). |
| source | String | Origem dos dados da entidade | Fornecedor dos medidores de energia. |
| phaseType | String | Tipo de fase | `threePhase` nas instalações trifásicas; `singlePhase` se algum medidor for monofásico. |
| totalActiveEnergyImport | Number | Energia ativa total consumida | Índice acumulado do contador desde `dateEnergyMeteringStarted`, soma das três fases. O consumo de um intervalo obtém-se pela diferença entre duas leituras. `unitCode` `KWH` (kWh). |
| activeEnergyImport | Object | Energia ativa consumida por fase | Objeto `{ L1, L2, L3 }`, acumulado. `unitCode` `KWH` (kWh). |
| totalActivePower | Number | Potência ativa total instantânea | Para acompanhamento em tempo real. `unitCode` `WTT` (W). |
| phaseVoltage | Object | Tensão entre cada fase e o neutro | Objeto `{ L1, L2, L3 }` (≈ 230 V). `unitCode` `VLT` (V). |
| phaseToPhaseVoltage | Object | Tensão entre fases | Objeto `{ L12, L23, L31 }` (≈ 400 V). `unitCode` `VLT` (V). |
| current | Object | Corrente por fase e no neutro | Objeto `{ L1, L2, L3, N }`. `unitCode` `AMP` (A). |
| dateObserved | DateTime | Data e hora da leitura | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. Hora de leitura reportada pelo medidor. |
| dateEnergyMeteringStarted | DateTime | Data de início da contagem da energia | ISO 8601 em UTC. Referência dos valores acumulados. |

As restantes propriedades do modelo do CNMD (energias exportadas, potências aparente e reativa, frequência, fator de potência e distorção harmónica, entre outras) não são publicadas, porque não constam dos campos de origem.

### Propriedades obrigatórias

Os atributos obrigatórios do modelo do CNMD e a sua situação na PGU são:

| Propriedade | Publicada |
|-------------|-----------|
| `id` | Sim |
| `type` | Sim |
| `address` | Sim |
| `dataProvider` | Sim |
| `dateObserved` (*) | Sim |
| `location` | Sim |
| `phaseType` | Sim |

(*) O CNMD exige `dateObserved` ou, em alternativa, o par `dateObservedFrom` e `dateObservedTo`. A PGU publica `dateObserved`.

## Modelo `Anomaly`

Cada alerta de consumo anómalo gera uma entidade, com o identificador `urn:ngsi-ld:Anomaly:<id_medidor>-<dateDetected>` (data e hora em formato compacto, por exemplo `20261116T100000Z`). Os limiares de potência são configuráveis por medidor.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Anomaly:<id_medidor>-<dateDetected>`. |
| type | String | Tipo de entidade | Valor constante igual a `Anomaly`. |
| name | String | Nome da anomalia | Ex.: `Anomalia de consumo — Paços do Concelho de Tomar — 2026-11-16T10:00:00Z`. |
| detectedBy | Relationship | Medidor em que a anomalia foi detetada | Referência à entidade `Device` do medidor. |
| anomalousProperty | String | Atributo em que foi detetada a anomalia | Ex.: `totalActivePower` ou `totalActiveEnergyImport`. |
| thresholdBreach | Object | Ultrapassagem do limiar que originou a anomalia | Objeto com `observedAt` (data e hora da observação), `value` (valor medido), `threshold` (limiar ultrapassado), `thresholdType` (`upper` ou `lower`) e `unitCode`. |
| dateDetected | DateTime | Data e hora em que a anomalia foi detetada pela primeira vez | ISO 8601 em UTC. |
| description | String | Descrição textual da anomalia | Ex.: `Consumo acima do limiar configurado (25 kW) durante 45 min; máximo 26.3 kW.` |
| areaServed | String | Edifício afetado | Igual ao da medição. |
| location | GeoJSON | Localização do edifício | Igual à da medição. |
| address | Object | Morada do edifício | Igual à da medição. |
| dataProvider | String | Fornecedor da entidade de dados | Valor `cimt`. |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo dos Smart Data Models são `id`, `type`, `anomalousProperty` e `dateDetected`, todos publicados. O [JSON Schema](schema/Anomaly.schema.json) deste repositório considera também obrigatórios os atributos que a PGU publica sempre: `detectedBy`, `thresholdBreach`, `location`, `address` e `dataProvider`.

## Modelo `KeyPerformanceIndicator`

Existe uma entidade por edifício e indicador, com o identificador `urn:ngsi-ld:KeyPerformanceIndicator:<edifício>-<indicador>`. O valor é calculado diariamente e o histórico obtém-se pela API temporal NGSI-LD. Os indicadores publicados são:

| Indicador | `kpiValue` | `currentStanding` |
|-----------|------------|-------------------|
| Consumo diário | kWh (`unitCode` `KWH`) | Sim |
| Consumo diário por área útil | kWh/m² | Sim |
| Potência máxima diária | kW (`unitCode` `KWT`) | Não |

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:KeyPerformanceIndicator:<edifício>-<indicador>`. |
| type | String | Tipo de entidade | Valor constante igual a `KeyPerformanceIndicator`. |
| name | String | Nome do indicador | Ex.: `Consumo diário (kWh) — Paços do Concelho de Tomar`. |
| description | String | Descrição do cálculo | Ex.: `Energia ativa consumida no dia, calculada a partir de totalActiveEnergyImport.` |
| kpiValue | Number | Valor do indicador no período | Com `unitCode` quando aplicável. |
| calculationPeriod | Object | Período a que o valor se refere | Objeto `{ from, to }` em ISO 8601 e UTC. |
| calculationFrequency | String | Periodicidade de cálculo | Valor `daily`. |
| aggregatedData | Array | Entidades e atributos agregados | Ex.: `[{ "entityType": "ACMeasurement", "attrs": ["totalActiveEnergyImport"] }]`. |
| category | Array | Categoria do indicador | Valor `["quantitative"]`. |
| currentStanding | String | Situação do indicador face ao objetivo | `veryGood`, `good`, `fair`, `bad` ou `veryBad`. Opcional. |
| organization | Object | Organização avaliada | Objeto `{ name }` com o município, ex.: `{ "name": "Município de Tomar" }`. |
| process | String | Processo avaliado | Valor `Eficiência energética dos edifícios públicos`. |
| areaServed | String | Edifício a que o indicador se refere | Igual ao da medição. |
| location | GeoJSON | Localização do edifício | Igual à da medição. |
| address | Object | Morada do edifício | Igual à da medição. |
| dataProvider | String | Fornecedor da entidade de dados | Valor `cimt`. |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo dos Smart Data Models são `id` e `type`, ambos publicados. O [JSON Schema](schema/KeyPerformanceIndicator.schema.json) deste repositório considera também obrigatórios os atributos que a PGU publica sempre: `name`, `kpiValue`, `calculationPeriod`, `calculationFrequency`, `aggregatedData`, `category`, `organization`, `process`, `areaServed`, `location`, `address` e `dataProvider`.

## Diferenças em relação aos modelos de referência

1. **`ACMeasurement.refDevice` é uma relação com um único medidor.** No CNMD é uma lista de referências. Cada leitura provém de um único medidor; o JSON Schema deste repositório aceita as duas formas.
2. **`ACMeasurement.refBuilding` é um atributo adicional**, que liga a medição à entidade `Building` do edifício. Conforme indicado no CNMD, pode ser ignorado na integração de dados provenientes de várias entidades.
3. **`Anomaly.thresholdBreach` é um objeto e não uma lista.** Cada anomalia regista a ultrapassagem que a originou. A correspondência com os campos dos Smart Data Models é: `observedAt` → `dateObserved`, `value` → `measuredValue`, `threshold` → `thresholdValue` e `thresholdType` `upper`/`lower` → `UPPER`/`LOWER`; o objeto inclui ainda o `unitCode` do valor e do limiar.
4. **`KeyPerformanceIndicator.organization` é um objeto `{ name }`** e não uma sequência de caracteres.
5. **`KeyPerformanceIndicator.calculationPeriod` usa data e hora** (ISO 8601 em UTC) e não apenas a data, para indicar com precisão os limites de cada período diário.
6. **`location` não inclui a metainformação `coordinateSystem`** indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
7. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD.

## Notas

Os valores numéricos são publicados como número, com a unidade indicada em `unitCode` (códigos UN/CEFACT). O instante de observação (`observedAt`) de cada atributo é igual a `dateObserved` nas medições, a `dateDetected` nas anomalias e ao fim do período (`calculationPeriod.to`) nos indicadores.

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `ACMeasurement` | [exemplos/ACMeasurement.json](exemplos/ACMeasurement.json) | [schema/ACMeasurement.schema.json](schema/ACMeasurement.schema.json) |
| `Anomaly` | [exemplos/Anomaly.json](exemplos/Anomaly.json) | [schema/Anomaly.schema.json](schema/Anomaly.schema.json) |
| `KeyPerformanceIndicator` | [exemplos/KeyPerformanceIndicator.json](exemplos/KeyPerformanceIndicator.json) | [schema/KeyPerformanceIndicator.schema.json](schema/KeyPerformanceIndicator.schema.json) |

Os exemplos estão em formato NGSI-LD normalizado. Os schemas derivam do schema do modelo de referência (CNMD ou Smart Data Models), adaptado às diferenças acima, e validam a representação simplificada (*key-values*) das entidades.
