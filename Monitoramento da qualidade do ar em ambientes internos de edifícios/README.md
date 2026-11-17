# Monitoramento da qualidade do ar em ambientes internos de edifícios públicos (V12)

## Descrição

A **monitorização da qualidade do ar interior** dos edifícios públicos (escolas, bibliotecas, centros de saúde e serviços municipais) é representada na PGU com três modelos do Catálogo Nacional de Modelos de Dados (CNMD):

| Modelo | Representa | Modelo do CNMD |
|--------|------------|----------------|
| `AirQualityObserved` | As medições de cada sensor de qualidade do ar interior (IAQ) | [Observações de qualidade do ar](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Ar/Observa%C3%A7%C3%B5es%20de%20qualidade%20do%20ar) |
| `Alert` | Os alertas de ventilação insuficiente (CO₂ acima do limiar) | [Alertas e avisos da proteção civil](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Justi%C3%A7a%2C%20sistema%20judici%C3%A1rio%20e%20seguran%C3%A7a%20p%C3%BAblica/Prote%C3%A7%C3%A3o%20e%20socorro/Alertas%20e%20avisos%20da%20prote%C3%A7%C3%A3o%20civil) |
| `AirQualityMonitoring` | Os relatórios periódicos de qualidade do ar de cada sensor | [Qualidade do ar (zonas e aglomerações)](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Ar/Qualidade%20do%20ar%20%28zonas%20e%20aglomera%C3%A7%C3%B5es%29) |

Os modelos utilizam o formato NGSI-LD, sendo compatíveis com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

## Origem dos dados

Os dados provêm de 13 sensores de qualidade do ar interior (IAQ) instalados em salas de edifícios públicos. As medições de cada sensor são publicadas como uma entidade `AirQualityObserved`, através do IoT Agent da plataforma iris360; os alertas e os relatórios são calculados pela PGU a partir dessas medições. O fornecedor dos sensores está por definir. Os campos de origem correspondem à tabela de campos da vertical:

| Campo de origem | Modelo | Atributos |
|-----------------|--------|-----------|
| `id_sensor_iaq` | todos | `refDevice` (`alertSource` no `Alert`) e o identificador da entidade |
| `edificio_sala` | todos | `name`, `refPointOfInterest`, `location`, `address` |
| `co2_ppm` | `AirQualityObserved` | `co2` |
| `cov` | `AirQualityObserved` | `volatileOrganicCompoundsTotal` |
| `particulas` | `AirQualityObserved` | `pm1`, `pm25`, `pm4`, `pm10` (e `pm05`, se o sensor o disponibilizar) |
| `temperatura` | `AirQualityObserved` | `temperature` |
| `humidade` | `AirQualityObserved` | `relativeHumidity` |
| `alerta_ventilacao` | `Alert` | `category`, `subCategory`, `severity`, `description`, `data`, `validFrom`, `validTo` |
| `relatorio_qualidade_ar` | `AirQualityMonitoring` | Agregados de cada parâmetro no período (`co2TSA`, `pm10TSA`, …) e `ventilationAlertTime` |

**Os exemplos deste diretório são ilustrativos:** foram gerados com os dados de teste da PGU, com o mesmo formato com que as medições serão publicadas, e não correspondem a medições reais. Os três exemplos descrevem o mesmo episódio de ventilação insuficiente numa sala de aula: a medição no pico de CO₂, o alerta e o relatório diário desse dia.

## Modelo `AirQualityObserved`

Cada sensor tem uma entidade, com o identificador `urn:ngsi-ld:AirQualityObserved:iaq-<n>`, em que `<n>` é o número do sensor; cada medição atualiza os seus atributos. O histórico obtém-se pela API temporal NGSI-LD.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:AirQualityObserved:iaq-<n>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `AirQualityObserved`. |
| name | String | Designação do espaço monitorizado | Edifício e sala, ex.: `Escola Básica — Sala 15`. |
| refDevice | Relationship | Sensor que efetuou a medição | Entidade `Device` do sensor, ex.: `urn:ngsi-ld:Device:iaq-08`. |
| refPointOfInterest | Relationship | Espaço monitorizado | Entidade `PointOfInterest` do espaço, ex.: `urn:ngsi-ld:PointOfInterest:espaco-iaq-08`. |
| typeofLocation | String | Tipo de local | Valor constante igual a `indoor`, que distingue estas medições das das estações exteriores. |
| location | GeoJSON | Localização do edifício | `Point`, atribuído a cada sensor. |
| address | Object | Morada do edifício | `streetAddress`, `addressLocality` (município), `addressRegion` (`Oeste e Vale do Tejo`), `district` (`Santarém`) e `addressCountry`. |
| dateObserved | DateTime | Data e hora da medição | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. |
| co2 | Number | Concentração de CO₂ | Em partes por milhão: `unitCode` `59`. Gama do sensor: 0–5000 ppm. |
| volatileOrganicCompoundsTotal | Number | Compostos orgânicos voláteis totais | `unitCode` `GQ` (µg/m³) ou `61` (ppb), conforme a saída do sensor. |
| pm1 | Number | Partículas ≤ 1 µm | `unitCode` `GQ` (µg/m³). |
| pm25 | Number | Partículas ≤ 2,5 µm | `unitCode` `GQ` (µg/m³). |
| pm4 | Number | Partículas ≤ 4 µm | `unitCode` `GQ` (µg/m³). Só nos sensores que disponibilizam este canal. |
| pm10 | Number | Partículas ≤ 10 µm | `unitCode` `GQ` (µg/m³). |
| pm05 | Number | Partículas ≤ 0,5 µm | Só se o sensor disponibilizar este canal. Atributo adicional. |
| temperature | Number | Temperatura do espaço interior | `unitCode` `CEL` (°C). |
| relativeHumidity | Number | Humidade relativa | Fração de 0 a 1 (54 % → 0,54), porque o modelo não aceita percentagens. |
| dataProvider | String | Fornecedor dos dados | `cimt` (CIM Médio Tejo). |
| source | String | Origem dos dados da entidade | O fornecedor dos sensores (por definir). |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `AirQualityObserved` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Sim | |
| `dataProvider` | Sim | |
| `dateObserved` | Sim | |
| `location` | Sim | |
| `name` | Sim | |
| `refPointOfInterest` | Sim | Refere o espaço monitorizado, e não uma estação de qualidade do ar. |
| `pm10` | Sim | |
| `pm25` | Sim | |
| `o3` | Não | Não é medido pelos sensores de qualidade do ar interior. |
| `no2` | Não | Não é medido pelos sensores de qualidade do ar interior. |
| `so2` | Não | Não é medido pelos sensores de qualidade do ar interior. |
| `c6h6` | Não | Não é medido pelos sensores de qualidade do ar interior. |
| `co` | Não | Não é medido pelos sensores de qualidade do ar interior. |

O [JSON Schema](schema/AirQualityObserved.schema.json) deste repositório considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `name`, `dateObserved`, `location`, `address`, `refDevice`, `refPointOfInterest`, `typeofLocation`, `co2`, `volatileOrganicCompoundsTotal`, `pm1`, `pm25`, `pm10`, `temperature`, `relativeHumidity`, `dataProvider` e `source`.

## Modelo `Alert`

Cada episódio de ventilação insuficiente tem uma entidade, com o identificador `urn:ngsi-ld:Alert:iaq-<n>-<data de emissão>` (ex.: `iaq-08-20261109T093000Z`). O alerta é criado quando o CO₂ de uma sala ultrapassa o limiar configurado, por exemplo 1250 ppm (limiar de proteção da Portaria n.º 138-G/2021), e é fechado com `validTo` quando o valor volta a ficar abaixo do limiar. Nos dados de teste, um episódio exige pelo menos duas medições seguidas acima do limiar.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Alert:iaq-<n>-<data de emissão>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `Alert`. |
| name | String | Nome do alerta | Ex.: `Alerta de ventilação — Escola Básica — Sala 15`. |
| description | String | Texto do alerta | Ex.: `Ventilação insuficiente: CO₂ acima de 1250 ppm durante 195 min.` |
| category | String | Categoria do alerta | Valor constante igual a `environment`. |
| subCategory | String | Subcategoria do alerta | Valor constante igual a `airPollution`. |
| severity | String | Gravidade do alerta | Enumerado: `informational`, `low`, `medium`, `high` ou `critical`. A regra de atribuição por patamares de CO₂ está por definir. |
| data | Object | Valores que dispararam o alerta | Valor máximo de CO₂ do episódio, limiar e unidade, ex.: `{"co2": 2178.4, "threshold": 1250, "unitCode": "59"}`. Atributo adicional. |
| dateIssued | DateTime | Data e hora de emissão do alerta | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. |
| validFrom | DateTime | Início do episódio | Igual a `dateIssued`. |
| validTo | DateTime | Fim do episódio | Omitido enquanto o alerta está ativo. |
| alertSource | Relationship | Sensor que originou o alerta | Entidade `Device` do sensor, ex.: `urn:ngsi-ld:Device:iaq-08`. |
| location | GeoJSON | Localização do edifício | Igual à da medição. |
| address | Object | Morada do edifício | Igual à da medição. |
| dataProvider | String | Fornecedor dos dados | `cimt` (CIM Médio Tejo). |
| source | String | Origem dos dados da entidade | O fornecedor dos sensores (por definir). |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `Alert` do CNMD são todos publicados: `id`, `type`, `address`, `alertSource`, `category` e `dateIssued`.

O [JSON Schema](schema/Alert.schema.json) deste repositório considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `name`, `description`, `category`, `subCategory`, `severity`, `data`, `dateIssued`, `validFrom`, `alertSource`, `location`, `address`, `dataProvider` e `source`.

## Modelo `AirQualityMonitoring`

Cada sensor tem um relatório por período de agregação, com o identificador `urn:ngsi-ld:AirQualityMonitoring:iaq-<n>-<período>` (ex.: `iaq-08-2026-11-09`, o relatório diário). O valor de cada parâmetro é um objeto com os agregados do período: último valor (`instValue`), mínimo (`minOverTime`), máximo (`maxOverTime`) e média (`averageValue`), conforme a estrutura `TimeSeriesAggregation` do modelo.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:AirQualityMonitoring:iaq-<n>-<período>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `AirQualityMonitoring`. |
| name | String | Nome do relatório | Ex.: `Relatório IAQ diário — Escola Básica — Sala 15 — 2026-11-09`. |
| refDevice | Relationship | Sensor cujas medições são agregadas | Entidade `Device` do sensor. Atributo adicional. |
| location | GeoJSON | Localização do edifício | Igual à da medição. |
| address | Object | Morada do edifício | Igual à da medição. |
| observationDateTime | DateTime | Data e hora da última medição do período | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. |
| aggregationPeriod | String | Granularidade do relatório | Enumerado: `day`, `week` ou `month`. Nos dados de teste, `day`. Atributo adicional. |
| periodStart | DateTime | Início do período agregado | Atributo adicional. |
| periodEnd | DateTime | Fim do período agregado (exclusivo) | Atributo adicional. |
| co2TSA | Object | Agregados de CO₂ | `unitCode` `59` (ppm). |
| pm10TSA | Object | Agregados de partículas ≤ 10 µm | `unitCode` `GQ` (µg/m³). |
| pm25TSA | Object | Agregados de partículas ≤ 2,5 µm | `unitCode` `GQ` (µg/m³). |
| pm1TSA | Object | Agregados de partículas ≤ 1 µm | `unitCode` `GQ` (µg/m³). Atributo adicional. |
| volatileOrganicCompoundsTotalTSA | Object | Agregados de compostos orgânicos voláteis totais | Com o `unitCode` das medições. Atributo adicional. |
| airTemperatureTSA | Object | Agregados da temperatura | `unitCode` `CEL` (°C). |
| relativeHumidityTSA | Object | Agregados da humidade relativa | Frações de 0 a 1: `unitCode` `C62`. |
| ventilationAlertTime | Number | Tempo com alerta de ventilação ativo | Percentagem do período (0–100) com um `Alert` de ventilação ativo, ex.: `31.25`. Atributo adicional. |
| dataProvider | String | Fornecedor dos dados | `cimt` (CIM Médio Tejo). |
| source | String | Origem dos dados da entidade | O fornecedor dos sensores (por definir). |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `AirQualityMonitoring` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Sim | |
| `location` | Sim | |
| `name` | Sim | |
| `observationDateTime` | Sim | |
| `airQualityIndex` | Não | O relatório agrega os valores medidos; não é calculado um índice. |
| `airQualityLevel` | Não | Não é calculado um nível qualitativo. |

O [JSON Schema](schema/AirQualityMonitoring.schema.json) deste repositório considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `name`, `observationDateTime`, `location`, `address`, `refDevice`, `aggregationPeriod`, `periodStart`, `periodEnd`, `co2TSA`, `pm10TSA`, `pm25TSA`, `pm1TSA`, `volatileOrganicCompoundsTotalTSA`, `airTemperatureTSA`, `relativeHumidityTSA`, `ventilationAlertTime`, `dataProvider` e `source`.

## Diferenças em relação ao CNMD

1. **Poluentes do ar exterior.** O modelo `AirQualityObserved` do CNMD exige `o3`, `no2`, `so2`, `c6h6` e `co`, que os sensores de qualidade do ar interior não medem. Estes atributos não são publicados: conforme as FAQ do CNMD, não se publicam valores artificiais.
2. **`refPointOfInterest`** refere o espaço monitorizado (edifício e sala), e não uma estação de qualidade do ar.
3. **Alertas de ventilação com o modelo `Alert` da proteção civil.** O CNMD não tem um modelo de alertas de qualidade do ar interior; é usado o modelo `Alert`, com `category` `environment` e `subCategory` `airPollution`, porque a lista de subcategorias não tem um valor próprio para o CO₂ ou a ventilação (o motivo é indicado em `description`). Além disso:
   - `alertSource` é uma relação com a entidade `Device` do sensor que originou o alerta, e não um URL;
   - `severity` tem os valores do modelo `Alert` dos Smart Data Models (`informational`, `low`, `medium`, `high`, `critical`); no CNMD é texto livre;
   - `validTo` é omitido enquanto o alerta está ativo.
4. **Índice e nível de qualidade do ar.** `airQualityIndex` e `airQualityLevel`, obrigatórios no modelo `AirQualityMonitoring` do CNMD, não são publicados: os relatórios agregam os valores medidos e não calculam um índice.
5. **Período agregado.** O período de cada relatório é publicado nos atributos adicionais `aggregationPeriod`, `periodStart` e `periodEnd`, e o sensor em `refDevice`.
6. **Atributos adicionais:** `pm05` (previsto, só se o sensor o disponibilizar) no `AirQualityObserved`; `data` no `Alert`; `refDevice`, `aggregationPeriod`, `periodStart`, `periodEnd`, `pm1TSA`, `volatileOrganicCompoundsTotalTSA` e `ventilationAlertTime` no `AirQualityMonitoring`. Conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
7. **`location` não inclui a metainformação `coordinateSystem`** indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
8. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD.

## Notas

Os valores numéricos são publicados como número, com a unidade indicada em `unitCode` (códigos UN/CEFACT): `59` (ppm) para o CO₂, `GQ` (µg/m³) para as partículas, `CEL` (°C) para a temperatura. A humidade relativa é uma fração de 0 a 1. O instante de observação (`observedAt`) dos atributos é igual a `dateObserved` nas medições, a `dateIssued` nos alertas e a `observationDateTime` nos relatórios.

As relações `refDevice`, `alertSource` e `refPointOfInterest` referem as entidades `Device` dos sensores e `PointOfInterest` dos espaços monitorizados, que não são publicadas nesta vertical.

Os patamares de gravidade dos alertas ainda não estão definidos; os dos dados de teste são ilustrativos.

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `AirQualityObserved` | [exemplos/AirQualityObserved.json](exemplos/AirQualityObserved.json) | [schema/AirQualityObserved.schema.json](schema/AirQualityObserved.schema.json) |
| `Alert` | [exemplos/Alert.json](exemplos/Alert.json) | [schema/Alert.schema.json](schema/Alert.schema.json) |
| `AirQualityMonitoring` | [exemplos/AirQualityMonitoring.json](exemplos/AirQualityMonitoring.json) | [schema/AirQualityMonitoring.schema.json](schema/AirQualityMonitoring.schema.json) |

Os exemplos estão em formato NGSI-LD normalizado. Os schemas derivam dos schemas do CNMD, adaptados às diferenças acima, e validam a representação simplificada (*key-values*) das entidades.
