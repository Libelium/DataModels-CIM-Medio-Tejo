# Monitoramento do Meio Ambiente e Proteção Civil (V3)

## Descrição

A **monitorização do meio ambiente e da proteção civil** é representada na PGU com quatro modelos do Catálogo Nacional de Modelos de Dados (CNMD), que correspondem a modelos dos [Smart Data Models](https://github.com/smart-data-models) da FIWARE Foundation:

| Modelo | Representa | Modelo do CNMD |
|--------|------------|----------------|
| `AirQualityObserved` | As observações de qualidade do ar e de condições ambientais de cada estação | [Observações de qualidade do ar](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Ar/Observa%C3%A7%C3%B5es%20de%20qualidade%20do%20ar) |
| `NoiseLevelObserved` | As observações de nível de ruído de cada estação | [Ruído](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Ru%C3%ADdo) |
| `Alert` | Os alertas de risco emitidos pela plataforma de gestão de riscos | [Alertas e avisos da proteção civil](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Justi%C3%A7a%2C%20sistema%20judici%C3%A1rio%20e%20seguran%C3%A7a%20p%C3%BAblica/Prote%C3%A7%C3%A3o%20e%20socorro/Alertas%20e%20avisos%20da%20prote%C3%A7%C3%A3o%20civil) |
| `Device` | As estações de monitorização | [Sensores](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Comuns/Sensores) |

Os modelos utilizam o formato NGSI-LD, sendo compatíveis com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

## Origem dos dados

Os dados provêm das estações de monitorização do ambiente da CIMT, com uma estação por município (duas em Tomar e duas em Torres Novas). O fornecedor das estações está por definir. Cada estação publica, no mesmo instante, uma observação de qualidade do ar (`AirQualityObserved`) e uma de ruído (`NoiseLevelObserved`), e é referida nelas como entidade `Device`, que publica também o estado de comunicação do equipamento. Os alertas (`Alert`) são emitidos pela plataforma de gestão de riscos, e a localização de um alerta sem zona própria é a da estação que o originou.

Os campos de origem correspondem aos atributos seguintes:

| Campo de origem | Modelo | Atributos |
|-----------------|--------|-----------|
| `id_estacao` | todos | O identificador da entidade e `refDevice` (`AirQualityObserved` e `NoiseLevelObserved`) |
| `timestamp` | `AirQualityObserved`, `NoiseLevelObserved` | `dateObserved` |
| `localizacao` | `AirQualityObserved`, `NoiseLevelObserved` | `location` |
| `temperatura` | `AirQualityObserved` | `temperature` |
| `humidade` | `AirQualityObserved` | `relativeHumidity` |
| `pressao` | `AirQualityObserved` | `atmosphericPressure` |
| `pm1`, `pm2_5`, `pm10` | `AirQualityObserved` | `pm1`, `pm25`, `pm10` |
| `nox` | `AirQualityObserved` | `nox` (e `no` e `no2`, se a origem os enviar em separado) |
| `co` | `AirQualityObserved` | `co` |
| `cov` | `AirQualityObserved` | `volatileOrganicCompoundsTotal` |
| `nivel_ruido` | `NoiseLevelObserved` | `LAmax` (picos) e `LAeq` (média) |
| `estado_comunicacao` | `Device` | `deviceState`, `dateLastValueReported`, `simState` e `batteryLevel` |
| `alerta_risco` | `Alert` | `id`, `category`, `subCategory`, `severity`, `description`, `dateIssued`, `validFrom`, `validTo` e `location` |

**Os exemplos deste diretório são ilustrativos:** têm o mesmo formato com que as entidades serão publicadas, mas as coordenadas, o identificador da estação e os valores não correspondem a dados reais. Os exemplos referem a estação `tomar-01`.

## Modelo `AirQualityObserved`

Cada estação tem uma entidade, com o identificador `urn:ngsi-ld:AirQualityObserved:cimt:<estação>`, em que `<estação>` é o identificador da estação, normalizado para ASCII; cada observação atualiza os seus atributos. O identificador é estável, pelo que a ingestão é idempotente, e o histórico obtém-se pela API temporal NGSI-LD.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:AirQualityObserved:cimt:<estação>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `AirQualityObserved`. |
| refDevice | Relationship | Estação que efetuou a observação | Entidade `Device` da estação, ex.: `urn:ngsi-ld:Device:cimt:tomar-01`. |
| dateObserved | DateTime | Data e hora da observação | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. Instante ou intervalo ISO 8601. |
| location | GeoJSON | Localização da estação | `Point`. |
| address | Object | Morada da estação | `addressLocality` (município), `addressRegion`, `district` e `addressCountry`, atribuídos por estação. |
| temperature | Number | Temperatura do ar | `unitCode` `CEL` (°C). |
| relativeHumidity | Number | Humidade relativa | Fração de 0 a 1 (68 % → 0,68), porque o modelo não aceita percentagens: a origem envia percentagens, que a PGU divide por 100. `unitCode` `C62`. |
| atmosphericPressure | Number | Pressão atmosférica | `unitCode` `A97` (hPa). Atributo adicional. |
| pm1 | Number | Partículas ≤ 1 µm | `unitCode` `GQ` (µg/m³). |
| pm25 | Number | Partículas ≤ 2,5 µm | `unitCode` `GQ` (µg/m³). |
| pm10 | Number | Partículas ≤ 10 µm | `unitCode` `GQ` (µg/m³). |
| nox | Number | Óxidos de azoto | `unitCode` `61` (ppb). Se a origem enviar NO e NO₂ em separado, são também publicados `no` e `no2`. |
| co | Number | Monóxido de carbono | `unitCode` `59` (ppm). |
| volatileOrganicCompoundsTotal | Number | Compostos orgânicos voláteis totais | `unitCode` `61` (ppb). |
| dataProvider | String | Fornecedor dos dados | `cimt` (CIM Médio Tejo). |
| source | String | Origem dos dados da entidade | O fornecedor das estações (por definir). |

O modelo do CNMD tem ainda outros poluentes e atributos que a PGU não publica, como `o3`, `no2`, `so2`, `c6h6`, `name`, `refPointOfInterest`, `airQualityIndex` e `airQualityLevel`. Estão previstos no [JSON Schema](schema/AirQualityObserved.schema.json), como opcionais.

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
| `name` | Não | A estação é identificada por `refDevice`. |
| `refPointOfInterest` | Não | A estação é uma entidade `Device`, e não um ponto de interesse. |
| `pm10` | Sim | |
| `pm25` | Sim | |
| `o3` | Não | Ver [Diferenças](#diferenças-em-relação-ao-cnmd). |
| `no2` | Não | Só se a origem enviar NO e NO₂ em separado. |
| `so2` | Não | Ver [Diferenças](#diferenças-em-relação-ao-cnmd). |
| `c6h6` | Não | Ver [Diferenças](#diferenças-em-relação-ao-cnmd). |
| `co` | Sim | |

O [JSON Schema](schema/AirQualityObserved.schema.json) deste repositório não exige `name`, `refPointOfInterest` nem os poluentes `o3`, `no2`, `so2` e `c6h6`, e considera obrigatórios `id`, `type`, `dateObserved`, `location`, `address`, `refDevice`, `dataProvider` e `source`. Os valores medidos são opcionais, porque dependem dos sensores de cada estação.

## Modelo `NoiseLevelObserved`

Cada estação tem uma entidade, com o identificador `urn:ngsi-ld:NoiseLevelObserved:cimt:<estação>`; cada observação atualiza os seus atributos, e o histórico obtém-se pela API temporal NGSI-LD.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:NoiseLevelObserved:cimt:<estação>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `NoiseLevelObserved`. |
| refDevice | Relationship | Estação que efetuou a observação | Entidade `Device` da estação. |
| dateObserved | DateTime | Data e hora da observação | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. Instante ou intervalo ISO 8601. |
| location | GeoJSON | Localização da estação | Igual à do `AirQualityObserved`. |
| address | Object | Morada da estação | Igual à do `AirQualityObserved`. |
| LAeq | Number | Nível sonoro contínuo equivalente ponderado A no período (média) | `unitCode` `2N` (dB). |
| LAmax | Number | Nível sonoro máximo no período (picos) | `unitCode` `2N` (dB). |
| dataProvider | String | Fornecedor dos dados | `cimt` (CIM Médio Tejo). |
| source | String | Origem dos dados da entidade | O fornecedor das estações (por definir). |

O modelo do CNMD tem ainda `dateObservedFrom`, `dateObservedTo`, `sonometerClass`, `LAS`, `LAeq_d`, `refPointOfInterest`, `refWeatherObserved`, `obstacles`, `heightAverage` e `distanceAverage`, que a PGU não publica. Estão previstos no [JSON Schema](schema/NoiseLevelObserved.schema.json), como opcionais.

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `NoiseLevelObserved` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `dateObservedFrom` | Não | Publicada como `dateObserved`. |
| `dateObservedTo` | Não | Publicada como `dateObserved`. |
| `location` | Sim | |
| `address` | Sim | |
| `dataProvider` | Sim | |
| `LAeq` | Sim | |
| `refDevice` | Sim | |
| `source` | Sim | |

O [JSON Schema](schema/NoiseLevelObserved.schema.json) deste repositório não exige `dateObservedFrom` nem `dateObservedTo` e considera também obrigatórios `dateObserved` e `LAmax`.

## Modelo `Alert`

Cada alerta tem uma entidade, com o identificador `urn:ngsi-ld:Alert:cimt:<alerta>`, em que `<alerta>` é o identificador do alerta na origem, normalizado para ASCII. Se a origem não o fornecer, constrói-se com o identificador da estação e a data de emissão: `<estação>-<data de emissão>` (ex.: `tomar-01-20261117T142000Z`).

### Propriedades

Os atributos seguintes são garantidos mesmo com um alerta mínimo:

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Alert:cimt:<alerta>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `Alert`. |
| category | String | Categoria do alerta | O tipo de risco da origem, traduzido para a lista do modelo (`traffic`, `naturalDisaster`, `weather`, `environment`, `health`, `security` ou `agriculture`). Se a origem não o indicar, `environment`. |
| dateIssued | DateTime | Data e hora de emissão do alerta | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. É o único campo imprescindível da origem. |
| alertSource | String | Origem do alerta | A plataforma de gestão de riscos, em texto. |

Os atributos seguintes só são publicados se a origem os fornecer; caso contrário, são omitidos:

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| severity | String | Gravidade do alerta | O nível da origem, traduzido para `informational`, `low`, `medium`, `high` ou `critical`. |
| description | String | Texto do alerta | A mensagem, tal como vem da origem. |
| subCategory | String | Subcategoria do alerta | O subtipo de risco, se a origem o distinguir. Valor da lista do modelo (ex.: `airPollution`, `heatWave`, `forestFire`). |
| validFrom, validTo | DateTime | Período de validade do alerta | Em UTC. |
| location | GeoJSON | Zona afetada | Se não vier, herda a localização da estação que originou o alerta. |
| dataProvider | String | Fornecedor dos dados | `cimt` (CIM Médio Tejo). |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `Alert` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Não | A zona afetada é dada por `location`. |
| `alertSource` | Sim | Em texto, e não como URL. |
| `category` | Sim | |
| `dateIssued` | Sim | |

O [JSON Schema](schema/Alert.schema.json) deste repositório não exige `address` e considera obrigatórios `id`, `type`, `category`, `dateIssued` e `alertSource`.

## Modelo `Device`

Cada estação de monitorização tem uma entidade, com o identificador `urn:ngsi-ld:Device:cimt:<estação>`. É a entidade referida por `refDevice` no `AirQualityObserved` e no `NoiseLevelObserved`, e publica o estado de comunicação do equipamento: o estado operacional, a última comunicação, o estado do cartão SIM e o nível da bateria.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Device:cimt:<estação>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `Device`. |
| location | GeoJSON | Localização da estação | `Point`. |
| address | Object | Morada da estação | Atribuída por estação. |
| deviceState | String | Estado operacional da estação | Enumerado: `active` (ativa), `communication` (problema de comunicação) ou `fault` (avaria). |
| dateLastValueReported | DateTime | Data e hora da última comunicação da estação à plataforma | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. |
| simState | String | Estado do cartão SIM da estação | Enumerado: `active` (ativo) ou `inactive` (inativo). Atributo adicional. |
| batteryLevel | Number | Nível da bateria da estação | Fração de 0 a 1 (91 % → 0,91): 1,0 com a bateria cheia e 0,0 com a bateria vazia. |
| dataProvider | String | Fornecedor dos dados | `cimt` (CIM Médio Tejo). |
| source | String | Origem dos dados da entidade | O fornecedor das estações (por definir). |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `Device` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Em parte | Atribuída por estação. |
| `controlledProperty` | Não | |
| `refDeviceModel` | Não | |
| `value` | Não | |
| `deviceCategory` | Não | |
| `dataProvider` | Sim | |

O [JSON Schema](schema/Device.schema.json) deste repositório não exige `address`, `controlledProperty`, `refDeviceModel`, `value` nem `deviceCategory` e considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `deviceState`, `dateLastValueReported`, `simState`, `batteryLevel`, `dataProvider` e `source`.

## Diferenças em relação ao CNMD

1. **Poluentes do ar que as estações não medem.** O modelo `AirQualityObserved` do CNMD exige `o3`, `no2`, `so2` e `c6h6`, que as estações não medem (medem `nox`, `co`, partículas e compostos orgânicos voláteis; `no` e `no2` só são publicados se o sensor os enviar em separado). Estes atributos não são publicados: conforme as FAQ do CNMD, não se publicam valores artificiais.
2. **Unidades.** O `co` é publicado em partes por milhão (`unitCode` `59`), e o `nox` e os compostos orgânicos voláteis em partes por bilião (`unitCode` `61`), que são as unidades dos sensores; o CNMD descreve concentrações em µg/m³ ou mg/m³.
3. **`refDevice` em vez de `name` e `refPointOfInterest`.** A estação é uma entidade `Device`, referida em `refDevice`; o `name` e o `refPointOfInterest`, obrigatórios no `AirQualityObserved` do CNMD, não são publicados.
4. **`atmosphericPressure`.** A pressão atmosférica é publicada neste atributo, do modelo `WeatherObserved`, e não em `pressure`, o atributo do modelo `AirQualityObserved`.
5. **`NoiseLevelObserved` com `dateObserved`.** A observação é publicada com um único instante, e não com o período (`dateObservedFrom` e `dateObservedTo`) que o CNMD exige.
6. **`Alert` sem `address`.** O CNMD exige `address`; os alertas da PGU indicam a zona afetada em `location`, que herda a da estação quando a origem não indica zona.
7. **`alertSource` em texto.** No CNMD é um URL; na PGU é o nome da plataforma de gestão de riscos.
8. **`severity` com os valores do modelo `Alert` dos Smart Data Models** (`informational`, `low`, `medium`, `high`, `critical`); no CNMD é texto livre.
9. **Atributos adicionais:** `atmosphericPressure` no `AirQualityObserved` (do modelo `WeatherObserved`) e `simState` no `Device` não existem nos modelos do CNMD e, conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
10. **`Device` sem `controlledProperty`, `refDeviceModel`, `value`, `deviceCategory` nem `address` obrigatórios.** A estação publica o seu estado de comunicação (`deviceState`, `dateLastValueReported`, `simState` e `batteryLevel`), e não os atributos de configuração do dispositivo exigidos pelo CNMD.
11. **`location` não inclui a metainformação `coordinateSystem`** (`EPSG:4258`) indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
12. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD.

## Notas

O valor da humidade relativa é uma fração de 0 a 1. O instante de observação (`observedAt`) dos atributos é igual a `dateObserved` nas observações e a `dateIssued` nos alertas.

A relação `refDevice` refere a entidade `Device` da estação, publicada nesta vertical. O fornecedor das estações, os identificadores das estações e o contexto JSON-LD próprio da PGU estão por definir.

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `AirQualityObserved` | [exemplos/AirQualityObserved.json](exemplos/AirQualityObserved.json) | [schema/AirQualityObserved.schema.json](schema/AirQualityObserved.schema.json) |
| `NoiseLevelObserved` | [exemplos/NoiseLevelObserved.json](exemplos/NoiseLevelObserved.json) | [schema/NoiseLevelObserved.schema.json](schema/NoiseLevelObserved.schema.json) |
| `Alert` | [exemplos/Alert.json](exemplos/Alert.json) | [schema/Alert.schema.json](schema/Alert.schema.json) |
| `Device` | [exemplos/Device.json](exemplos/Device.json) | [schema/Device.schema.json](schema/Device.schema.json) |

Os exemplos estão em formato NGSI-LD normalizado. Os schemas derivam dos schemas do CNMD, adaptados às diferenças acima, e validam a representação simplificada (*key-values*) das entidades.
