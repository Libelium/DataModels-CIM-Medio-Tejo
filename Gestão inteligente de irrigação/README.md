# Gestão inteligente de irrigação (V8)

## Descrição

Os modelos de dados utilizados para representar a **gestão inteligente da rega dos espaços verdes municipais** são:

- [`GreenspaceRecord`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/%C3%81gua/Rega) do Catálogo Nacional de Modelos de Dados (CNMD), para as leituras das sondas de humidade do solo;
- [`WaterConsumptionObserved`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/%C3%81gua/Consumo%20de%20agua) do CNMD, para as leituras dos contadores de água;
- [`WeatherObserved`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Meteorologia/Observa%C3%A7%C3%B5es%20metereol%C3%B3gicas) do CNMD, para as observações das estações meteorológicas;
- [`Device`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Comuns/Sensores) do CNMD, para os controladores de rega, que recebem também os comandos de rega;
- [`Anomaly`](https://github.com/smart-data-models/dataModel.Alert/tree/master/Anomaly) dos Smart Data Models, para os alertas de consumo de água anómalo e de humidade do solo fora dos parâmetros.

O CNMD não tem um modelo para anomalias, pelo que nesse caso foi adotado o modelo dos Smart Data Models, tal como na vertical V4. Todos os modelos utilizam o formato NGSI-LD, sendo compatíveis com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

## Origem dos dados

Os dados provêm de 25 sondas de humidade do solo, 28 contadores de água, 11 estações meteorológicas e 110 controladores de rega, um por setor de rega. Os fornecedores dos equipamentos estão por definir. As leituras são publicadas através do IoT Agent da plataforma iris360, e os campos de origem correspondem à tabela de campos da vertical:

| Campo de origem | Modelo | Atributos |
|-----------------|--------|-----------|
| `id_sonda` | `GreenspaceRecord` | `refDevice` e o identificador da entidade |
| `humidade_solo` | `GreenspaceRecord` | `soilMoistureVwc` |
| `temperatura_solo` | `GreenspaceRecord` | `soilTemperature` |
| `consumo_agua` | `WaterConsumptionObserved` | `waterConsumption` |
| `vento` | `WeatherObserved` | `windSpeed`, `windDirection` |
| `precipitacao` | `WeatherObserved` | `precipitation` |
| `humidade_relativa` | `WeatherObserved` | `relativeHumidity` |
| `estado_controlador` | `Device` | `deviceState` |
| `water_budget` | `Device` | `waterBudget` |
| `horario_programado` | `Device` | `irrigationSchedule` |
| `comando_rega` | `Device` | `irrigationCommand` (escrita) |
| `alerta_consumo_anomalo` | `Anomaly` | `detectedBy`, `anomalousProperty`, `thresholdBreach`, `description` |

As sondas, o contador e o controlador de um mesmo setor partilham o valor de `areaServed` (ex.: `Setor 01 — Parque do Mouchão`), que é a chave com que a PGU os agrupa por setor de rega.

**Os exemplos deste diretório são ilustrativos:** foram gerados com os dados de teste da PGU, com o mesmo formato com que as leituras serão publicadas, e não correspondem a medições reais.

## Modelo `GreenspaceRecord`

Cada sonda de humidade tem uma entidade, com o identificador `urn:ngsi-ld:GreenspaceRecord:<id_sonda>`, estável por sonda; o histórico das leituras obtém-se pela API temporal NGSI-LD.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:GreenspaceRecord:<id_sonda>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `GreenspaceRecord`. |
| name | String | Nome da sonda | Ex.: `Sonda de humidade 01 — Setor 01 — Parque do Mouchão`. |
| areaServed | String | Setor de rega onde está a sonda | Ex.: `Setor 01 — Parque do Mouchão`. |
| refDevice | Relationship | Sonda que efetuou a leitura | Referência à entidade `Device` da sonda. |
| refGreenspace | Relationship | Jardim ou canteiro onde está a sonda | Ex.: `urn:ngsi-ld:Greenspace:jardim-01`. Ver [Diferenças](#diferenças-em-relação-aos-modelos-de-referência). |
| location | GeoJSON | Localização da sonda | `Point`. |
| soilMoistureVwc | Number | Humidade volumétrica do solo (VWC) | Fração de 0 a 1 (23 % → `0.23`). `unitCode` `C62`. |
| soilTemperature | Number | Temperatura do solo | `unitCode` `CEL` (°C). |
| dateObserved | DateTime | Data e hora da leitura | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. |
| dataProvider | String | Fornecedor da entidade de dados | Valor `cimt` (CIM Médio Tejo). |
| source | String | Origem dos dados da entidade | Fornecedor das sondas de humidade. |

As restantes propriedades do modelo do CNMD (`soilMoistureEc`, `soilMoisturePressure`, `refWeatherObserved` e `litterCoverage`) não são publicadas, porque não constam dos campos de origem.

### Propriedades obrigatórias

Os atributos obrigatórios do modelo do CNMD e a sua situação na PGU são:

| Propriedade | Publicada |
|-------------|-----------|
| `id` | Sim |
| `type` | Sim |
| `address` | Não |
| `dateObserved` | Sim |
| `location` | Sim |
| `refGreenspace` | Sim |
| `soilTemperature` (*) | Sim |

(*) O CNMD exige pelo menos uma das propriedades `soilMoistureEc`, `soilMoisturePressure` ou `soilTemperature`. A PGU publica `soilTemperature`.

O [JSON Schema](schema/GreenspaceRecord.schema.json) deste repositório não exige `address` e considera também obrigatórios os atributos que a PGU publica sempre: `name`, `refDevice`, `soilMoistureVwc`, `areaServed`, `dataProvider` e `source`.

## Modelo `WaterConsumptionObserved`

Cada contador de água tem uma entidade, com o identificador `urn:ngsi-ld:WaterConsumptionObserved:<id_contador>`; o histórico das leituras obtém-se pela API temporal NGSI-LD.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:WaterConsumptionObserved:<id_contador>`. |
| type | String | Tipo de entidade | Valor constante igual a `WaterConsumptionObserved`. |
| name | String | Nome do contador | Ex.: `Contador de água — Setor 01 — Parque do Mouchão`. |
| areaServed | String | Setor de rega abastecido pelo contador | Igual ao das sondas e do controlador do setor. |
| refDevice | Relationship | Contador que efetuou a leitura | Referência à entidade `Device` do contador. |
| location | GeoJSON | Localização do contador | `Point`. |
| waterConsumption | Number | Volume de água consumido no intervalo de leitura | Consumo da última hora, e não a leitura acumulada do contador (ver [Diferenças](#diferenças-em-relação-aos-modelos-de-referência)). `unitCode` `MTQ` (m³). |
| dateObserved | DateTime | Data e hora da leitura | ISO 8601 em UTC. |
| dataProvider | String | Fornecedor da entidade de dados | Valor `cimt`. |
| source | String | Origem dos dados da entidade | Fornecedor dos contadores de água. |

As propriedades de alarme e de caudal do modelo do CNMD não são publicadas, porque não constam dos campos de origem; o consumo anómalo é publicado como entidade `Anomaly`.

### Propriedades obrigatórias

Os atributos obrigatórios do modelo do CNMD e a sua situação na PGU são:

| Propriedade | Publicada |
|-------------|-----------|
| `id` | Sim |
| `type` | Sim |
| `address` | Não |
| `dataProvider` | Sim |
| `dateObserved` | Sim |
| `location` | Sim |
| `waterConsumption` | Sim, com outro significado (consumo da hora) |

O [JSON Schema](schema/WaterConsumptionObserved.schema.json) deste repositório não exige `address` e considera também obrigatórios `name`, `refDevice`, `areaServed` e `source`.

## Modelo `WeatherObserved`

Cada estação meteorológica tem uma entidade, com o identificador `urn:ngsi-ld:WeatherObserved:<id_estacao>`; o histórico das observações obtém-se pela API temporal NGSI-LD.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:WeatherObserved:<id_estacao>`. |
| type | String | Tipo de entidade | Valor constante igual a `WeatherObserved`. |
| name | String | Nome da estação | Ex.: `Estação meteorológica de Tomar`. |
| refDevice | Relationship | Estação que efetuou a observação | Referência à entidade `Device` da estação. |
| location | GeoJSON | Localização da estação | `Point`. |
| address | Object | Morada da estação | Modelo: [https://schema.org/address](https://schema.org/address). A localidade coincide com o município e a região corresponde à NUTS 2 conforme a nomenclatura do INE. |
| address.addressCountry | String | O país | Valor `Portugal`. |
| address.addressLocality | String | A localidade, coincidente com o município | Ex.: `Tomar`. |
| address.addressRegion | String | A região NUTS 2 | Valor `Oeste e Vale do Tejo`. |
| address.district | String | O distrito | Valor `Santarém`. |
| address.streetAddress | String | Endereço da rua | Ex.: `Parque do Mouchão`. |
| windSpeed | Number | Velocidade do vento | `unitCode` `MTS` (m/s). |
| windDirection | Number | Direção de onde sopra o vento | Em graus (0–360). `unitCode` `DD`. |
| precipitation | Number | Precipitação acumulada no intervalo de medição | Em l/m², equivalente a mm. `unitCode` `MMT`. |
| relativeHumidity | Number | Humidade relativa do ar | Fração de 0 a 1 (65 % → `0.65`). `unitCode` `C62`. |
| dateObserved | DateTime | Data e hora da observação | ISO 8601 em UTC. |
| dataProvider | String | Fornecedor da entidade de dados | Valor `cimt`. |
| source | String | Origem dos dados da entidade | Fornecedor das estações meteorológicas. |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo do CNMD e a sua situação na PGU são:

| Propriedade | Publicada |
|-------------|-----------|
| `id` | Sim |
| `type` | Sim |
| `address` | Sim |
| `dataProvider` | Sim |
| `dateObserved` | Sim |
| `location` | Sim |
| `precipitation` | Sim |
| `relativeHumidity` | Sim |
| `windSpeed` | Sim |
| `windDirection` | Sim |
| `temperature` | Não |
| `refPointOfInterest` | Não |

O [JSON Schema](schema/WeatherObserved.schema.json) deste repositório não exige `temperature` nem `refPointOfInterest` e considera também obrigatórios `name`, `refDevice` e `source`.

## Modelo `Device`

Cada controlador de rega tem uma entidade, com o identificador `urn:ngsi-ld:Device:ctrl-rega-<setor>`. É também a entidade sobre a qual se enviam os comandos de rega.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Device:ctrl-rega-<setor>`. |
| type | String | Tipo de entidade | Valor constante igual a `Device`. |
| name | String | Nome do controlador | Ex.: `Controlador de rega — Setor 01 — Parque do Mouchão`. |
| category | Array | Categoria do dispositivo | Valor `["actuator"]`. Ver [Diferenças](#diferenças-em-relação-aos-modelos-de-referência). |
| areaServed | String | Setor de rega que o controlador controla | Igual ao das sondas e do contador do setor. |
| location | GeoJSON | Localização do controlador | `Point`. |
| deviceState | String | Estado operacional do controlador | `regando`, `parado`, `falha` ou `sem comunicação` (lista a acordar com o fornecedor). |
| dateLastValueReported | DateTime | Data e hora da última comunicação do controlador | ISO 8601 em UTC. Deixa de avançar enquanto o controlador está `sem comunicação`. |
| waterBudget | Number | Ajuste do orçamento hídrico do setor | Percentagem sobre a programação base (100 = programação base). `unitCode` `P1` (%). Atributo adicional. |
| irrigationSchedule | Object | Programação de rega do setor | Objeto `{ days, startTimes, durationMin }`, ex.: `{ "days": ["mon", "wed", "fri"], "startTimes": ["06:00"], "durationMin": 25 }`. As horas são locais. Atributo adicional. |
| irrigationCommand | String | Comando de rega (escrita) | `start` (iniciar), `stop` (parar) ou `changeBudget` (alterar o orçamento; o novo valor é enviado em `waterBudget` na mesma atualização). Atributo adicional. |
| irrigationCommand_status | String | Estado de execução do comando | Devolvido pelo IoT Agent, ex.: `PENDING`, `OK`, `ERROR`. Atributo adicional. |
| dataProvider | String | Fornecedor da entidade de dados | Valor `cimt`. |
| source | String | Origem dos dados da entidade | Fornecedor dos controladores de rega. |

O comando é o único campo de escrita da vertical: a aplicação atualiza `irrigationCommand` e o IoT Agent encaminha a ordem para o controlador. As ordens em massa enviam-se com a operação em lote do NGSI-LD (`POST /ngsi-ld/v1/entityOperations/update`) sobre a lista de controladores afetados, e o acesso a esta operação deve estar restrito a perfis autorizados.

### Propriedades obrigatórias

Os atributos obrigatórios do modelo do CNMD e a sua situação na PGU são:

| Propriedade | Publicada |
|-------------|-----------|
| `id` | Sim |
| `type` | Sim |
| `address` | Não |
| `controlledProperty` | Não |
| `dataProvider` | Sim |
| `deviceCategory` | Não (publicado `category`) |
| `refDeviceModel` | Não |
| `value` | Não (publicado `irrigationCommand`) |

O [JSON Schema](schema/Device.schema.json) deste repositório não exige estes cinco atributos e considera obrigatórios os que a PGU publica sempre: `name`, `category`, `deviceState`, `dateLastValueReported`, `waterBudget`, `irrigationSchedule`, `location`, `areaServed`, `dataProvider` e `source`.

## Modelo `Anomaly`

Cada alerta gera uma entidade, com o identificador `urn:ngsi-ld:Anomaly:<dispositivo>-<dateDetected>` (data e hora em formato compacto, por exemplo `20261108T010000Z`). Cobre os dois casos da tabela de origem:

| Caso | `anomalousProperty` | `detectedBy` | Critério nos dados de teste |
|------|---------------------|--------------|-----------------------------|
| Consumo de água anómalo | `waterConsumption` | Contador de água | Consumo fora do horário de rega acima de 0,1 m³ numa hora (possível fuga). |
| Humidade do solo fora dos parâmetros | `soilMoistureVwc` | Sonda de humidade | Humidade abaixo de 0,15 (15 %). |

Os limiares são configuráveis por contador e por sonda.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Anomaly:<dispositivo>-<dateDetected>`. |
| type | String | Tipo de entidade | Valor constante igual a `Anomaly`. |
| name | String | Nome da anomalia | Ex.: `Anomalia de consumo de água — Setor 03 — Jardim do Castelo de Abrantes — 2026-11-08T01:00:00Z`. |
| detectedBy | Relationship | Dispositivo em que a anomalia foi detetada | Referência à entidade `Device` do contador ou da sonda. |
| anomalousProperty | String | Atributo em que foi detetada a anomalia | `waterConsumption` ou `soilMoistureVwc`. |
| thresholdBreach | Object | Ultrapassagem do limiar que originou a anomalia | Objeto com `observedAt` (data e hora da observação), `value` (valor medido), `threshold` (limiar ultrapassado), `thresholdType` (`upper` ou `lower`) e `unitCode`. |
| dateDetected | DateTime | Data e hora em que a anomalia foi detetada pela primeira vez | ISO 8601 em UTC. |
| description | String | Descrição textual da anomalia | Ex.: `Consumo de água fora do horário de rega: 1.88 m³ em 4 h (possível fuga).` |
| areaServed | String | Setor de rega afetado | Igual ao do contador ou da sonda. |
| location | GeoJSON | Localização do contador ou da sonda | Igual à da leitura. |
| dataProvider | String | Fornecedor da entidade de dados | Valor `cimt`. |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo dos Smart Data Models são `id`, `type`, `anomalousProperty` e `dateDetected`, todos publicados. O [JSON Schema](schema/Anomaly.schema.json) deste repositório é o mesmo da vertical V4, sem `address`, que estas anomalias não publicam, e considera também obrigatórios `detectedBy`, `thresholdBreach`, `location`, `areaServed` e `dataProvider`.

## Diferenças em relação aos modelos de referência

1. **`address` não é publicada nas sondas (`GreenspaceRecord`), nos contadores (`WaterConsumptionObserved`) nem nos controladores (`Device`).** O CNMD exige-a nos três modelos. O modelo de dados da PGU só prevê a morada nas estações meteorológicas; nos restantes equipamentos, a localização é dada por `location` e o setor por `areaServed`.
2. **`WaterConsumptionObserved.waterConsumption` é o consumo do intervalo de leitura** (uma hora) e não a leitura acumulada do contador, como define o CNMD. O consumo por hora é o que permite comparar o consumo com o horário de rega e detetar fugas.
3. **`WeatherObserved` não publica `temperature` nem `refPointOfInterest`**, obrigatórios no CNMD. A temperatura do ar não consta dos campos de origem, e as estações meteorológicas não são modeladas como pontos de interesse.
4. **`Device` não publica `controlledProperty`, `refDeviceModel`, `value` nem `deviceCategory`**, obrigatórios no CNMD. A categoria é publicada em `category` com o valor `actuator`, que o CNMD prevê em `deviceCategory` (a lista de `category` do CNMD é `Sensor`, `Contador`, `AVAC`, `Rede` e `Multimédia`). Os comandos são enviados em `irrigationCommand`, e não em `value`, porque incluem a alteração do orçamento hídrico. O modelo do controlador e as propriedades controladas dependem do fornecedor, que está por definir.
5. **Atributos adicionais do `Device`:** `waterBudget`, `irrigationSchedule`, `irrigationCommand` e `irrigationCommand_status` não existem no modelo do CNMD. Conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
6. **`GreenspaceRecord.refDevice` é uma relação com uma única sonda.** No CNMD é uma lista de referências; cada leitura provém de uma única sonda.
7. **`GreenspaceRecord.refGreenspace` refere uma entidade que a PGU ainda não publica**, com o tipo `Greenspace`, que não corresponde a um modelo do CNMD nem dos Smart Data Models; o CNMD prevê o modelo `Garden` na mesma área temática. O tipo da entidade do jardim está por confirmar.
8. **`Anomaly.thresholdBreach` é um objeto e não uma lista**, como na vertical V4. A correspondência com os campos dos Smart Data Models é: `observedAt` → `dateObserved`, `value` → `measuredValue`, `threshold` → `thresholdValue` e `thresholdType` `upper`/`lower` → `UPPER`/`LOWER`; o objeto inclui ainda o `unitCode` do valor e do limiar.
9. **`location` não inclui a metainformação `coordinateSystem`** indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
10. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD.

## Notas

Os valores numéricos são publicados como número, com a unidade indicada em `unitCode` (códigos UN/CEFACT). As humidades (`soilMoistureVwc`, `relativeHumidity`) são frações de 0 a 1, porque os modelos não aceitam percentagens. O instante de observação (`observedAt`) de cada atributo é igual a `dateObserved` nas leituras, ao instante do estado nos controladores e a `dateDetected` nas anomalias.

As relações `refDevice` e `detectedBy` referem as entidades `Device` das sondas, dos contadores e das estações; nesta vertical, só os controladores de rega são publicados como `Device`.

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `GreenspaceRecord` | [exemplos/GreenspaceRecord.json](exemplos/GreenspaceRecord.json) | [schema/GreenspaceRecord.schema.json](schema/GreenspaceRecord.schema.json) |
| `WaterConsumptionObserved` | [exemplos/WaterConsumptionObserved.json](exemplos/WaterConsumptionObserved.json) | [schema/WaterConsumptionObserved.schema.json](schema/WaterConsumptionObserved.schema.json) |
| `WeatherObserved` | [exemplos/WeatherObserved.json](exemplos/WeatherObserved.json) | [schema/WeatherObserved.schema.json](schema/WeatherObserved.schema.json) |
| `Device` (controlador, após um comando `changeBudget`) | [exemplos/Device.json](exemplos/Device.json) | [schema/Device.schema.json](schema/Device.schema.json) |
| `Anomaly` (consumo de água) | [exemplos/Anomaly-consumo.json](exemplos/Anomaly-consumo.json) | [schema/Anomaly.schema.json](schema/Anomaly.schema.json) |
| `Anomaly` (humidade do solo) | [exemplos/Anomaly-humidade.json](exemplos/Anomaly-humidade.json) | [schema/Anomaly.schema.json](schema/Anomaly.schema.json) |

Os exemplos estão em formato NGSI-LD normalizado. Os schemas derivam do schema do modelo de referência (CNMD ou Smart Data Models), adaptado às diferenças acima, e validam a representação simplificada (*key-values*) das entidades.
