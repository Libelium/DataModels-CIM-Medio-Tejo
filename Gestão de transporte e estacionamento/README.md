# Gestão de transporte e estacionamento (V1)

## Descrição

A **gestão de transporte e estacionamento** é representada na PGU com quatro modelos do Catálogo Nacional de Modelos de Dados (CNMD), que correspondem a modelos dos [Smart Data Models](https://github.com/smart-data-models) da FIWARE Foundation:

| Modelo | Representa | Modelo do CNMD |
|--------|------------|----------------|
| `TrafficFlowObserved` | As contagens de veículos das câmaras de tráfego | [Contadores de viaturas](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Transportes%20e%20infraestruturas/Tr%C3%A1fego/Contadores%20de%20viaturas) |
| `ParkingSpot` | O estado de ocupação de cada lugar de estacionamento com sensor | [Estacionamento — Lugar](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Transportes%20e%20infraestruturas/Mobilidade/Estacionamento%20-%20Lugar) |
| `ParkingGroup` | As zonas de estacionamento, com a ocupação agregada dos seus lugares | [Estacionamento — Parque](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Transportes%20e%20infraestruturas/Mobilidade/Estacionamento%20-%20Parque) |
| `Device` | Os sensores dos lugares de estacionamento e as câmaras de contagem de tráfego | [Sensores](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Comuns/Sensores) |

Os modelos utilizam o formato NGSI-LD, sendo compatíveis com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

## Origem dos dados

Os dados provêm da API EntiServices do município de Abrantes (servidor de desenvolvimento), com a seguinte correspondência entre os pontos de acesso e os modelos:

| Ponto de acesso | Conteúdo | Modelos | Ingestão |
|-----------------|----------|---------|----------|
| `/analitica/estacionamentos` | Estado atual de cada lugar com sensor | `ParkingSpot`, `Device` | A cada 5 minutos |
| `/analitica/estacionamentos/historico` | Cada mudança de estado de cada lugar | `ParkingSpot` | A cada 5 minutos |
| `/analitica/fluxos-pessoas` | Eventos de contagem; só os de veículos pertencem a esta vertical | `TrafficFlowObserved` | A cada 15 minutos |

Nos identificadores, `<fonte>` é o município de origem dos dados (hoje, `abrantes`). A ingestão é feita através do IoT Agent JSON da plataforma iris360. Os grupos de estacionamento (`ParkingGroup`) não vêm da API: são criados na PGU, e a plataforma calcula os seus valores agregados a partir dos lugares que os compõem. A API de Abrantes não indica o estado dos sensores nem das câmaras.

Os campos de origem correspondem aos atributos seguintes:

| Campo de origem | Modelo | Atributos |
|-----------------|--------|-----------|
| `idUri` | `ParkingSpot` | `id` e `sourceId` |
| `status` | `ParkingSpot` | `status` e `r_prk_slot` |
| `dateModified` | `ParkingSpot`, `Device` | `TimeInstant` |
| `category` | `ParkingSpot` | `category` |
| `refParkingGroup` | `ParkingSpot` | `refParkingGroup` |
| `refDevice` | `ParkingSpot`, `Device` | `refDevice` (no `ParkingSpot`), `serialNumber` e o identificador (no `Device`) |
| `location` | `ParkingSpot`, `Device`, `TrafficFlowObserved` | `location` |
| `idRua` | `TrafficFlowObserved` | O identificador da entidade (ponto de contagem) |
| `address` | `TrafficFlowObserved` | `name` |
| `intensity` | `TrafficFlowObserved` | `intensity` |
| `tipoObjeto` | `TrafficFlowObserved` | `vehicleType` |
| `tipoContagem` | `TrafficFlowObserved` | `countType` |
| `dateObserved` | `TrafficFlowObserved` | `dateObserved` |

**Os exemplos deste diretório são ilustrativos:** têm o mesmo formato com que as entidades são publicadas, mas as coordenadas, os identificadores e os valores não correspondem a dados reais. O `Device` da câmara (`Device-camara.json`) corresponde a atributos ainda não publicados (ver [Notas](#notas)).

## Modelo `TrafficFlowObserved`

Cada ponto de contagem tem uma entidade por tipo de contagem, com o identificador `urn:ngsi-ld:TrafficFlowObserved:<fonte>-traffic-<ponto>_TFO`, em que `<ponto>` é o identificador do ponto de contagem na origem (`idRua`; se for numérico, com 8 dígitos, ex.: `00000012`). Quando a contagem não é uma passagem, o tipo de contagem fica antes do sufixo: `<fonte>-traffic-<ponto>-dwell_TFO`. O identificador é estável, pelo que a ingestão é idempotente; cada evento de contagem atualiza os atributos da entidade, e o histórico obtém-se pela API temporal NGSI-LD. Cada evento é publicado tal como chega, sem ser somado: as agregações são feitas na consulta.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:TrafficFlowObserved:<fonte>-traffic-<ponto>_TFO`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `TrafficFlowObserved`. |
| name | String | Nome do ponto de contagem | A rua da câmara. |
| intensity | Number | Número de veículos detetados no evento | Inteiro maior ou igual a 0. |
| vehicleType | String | Tipo de veículo | Enumerado do modelo. A origem distingue `car` (`carro`), `lorry` (`camiao`), `bus` (`autocarro`), `bicycle` (`bicicleta`) e `motorcycle` (`motociclo`); qualquer outro valor é publicado como o seu slug em minúsculas ASCII. Os eventos de pessoas (`pessoa`) não pertencem a esta vertical. |
| countType | String | Tipo de contagem do evento | `passage` (passagem) ou `dwell` (permanência); qualquer outro valor da origem é publicado como o seu slug. Atributo adicional. |
| dateObserved | DateTime | Data e hora da observação | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. Instante ou intervalo ISO 8601. |
| location | GeoJSON | Localização do ponto de contagem | `Point`. O modelo admite também `LineString`, `Polygon`, `MultiPoint`, `MultiLineString` e `MultiPolygon`. |
| address | Object | Morada do ponto de contagem | Município, região, distrito e país, atribuídos por ponto (a definir). |
| laneDirection | String | Sentido habitual de circulação na faixa | Enumerado: `forward` ou `backward`. Atributo previsto, ainda não publicado. |
| aggregationPeriod | String | Período de agregação dos dados de contagem | Enumerado: `hour` (hora) ou `day` (dia). Atributo adicional, previsto e ainda não publicado. |
| trafficPattern | Object | Indicadores agregados de padrões horários e sazonais do fluxo de tráfego | Atributo adicional, previsto e ainda não publicado. |
| dataProvider | String | Fornecedor dos dados | Município de origem dos dados (hoje, `abrantes`). Está previsto passar a `cimt` (CIM Médio Tejo) quando houver dados de vários municípios. |
| source | String | Origem dos dados da entidade | `entiservices:<servidor da API>`. |

O modelo do CNMD tem ainda propriedades que a PGU não publica, como `occupancy`, `averageVehicleSpeed`, `averageVehicleLength`, `averageGapDistance`, `averageHeadwayTime`, `congested`, `reversedLane`, `laneId`, `refRoadSegment` e `vehicleSubType`. Estão previstas no [JSON Schema](schema/TrafficFlowObserved.schema.json), como opcionais.

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `TrafficFlowObserved` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Não | Ver [Diferenças](#diferenças-em-relação-ao-cnmd). |
| `dataProvider` | Sim | |
| `dateObserved` | Sim | |
| `intensity` | Sim | |
| `location` | Sim | |

O [JSON Schema](schema/TrafficFlowObserved.schema.json) deste repositório não exige `address` e considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `intensity`, `vehicleType`, `countType`, `location`, `dateObserved`, `dataProvider` e `source`.

## Modelo `ParkingSpot`

Cada lugar de estacionamento com sensor tem uma entidade, com o identificador `urn:ngsi-ld:ParkingSpot:<fonte>-parking-spot-<n>`, em que `<n>` é o número do lugar na origem. A cada mudança de estado são atualizados `status`, `r_prk_slot` e `TimeInstant`; os restantes atributos só voltam a ser enviados se mudarem. O histórico obtém-se pela API temporal NGSI-LD.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:ParkingSpot:<fonte>-parking-spot-<n>`, normalizado para ASCII. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `ParkingSpot`. |
| status | String | Estado de ocupação do lugar | Enumerado: `free`, `occupied`, `closed` ou `unknown`. |
| r_prk_slot | Number | Ocupação do lugar: 0 (livre) ou 1 (ocupado) | Atributo adicional, usado pela plataforma para calcular a ocupação dos grupos. Só é publicado com `free` ou `occupied`; ver [Notas](#notas). |
| TimeInstant | DateTime | Data e hora da última mudança de estado | Em UTC. Atributo adicional, da plataforma. |
| category | Array | Tipo de estacionamento | Enumerado: `onStreet` ou `offStreet`. |
| location | GeoJSON | Localização do lugar | `Point`. |
| refParkingGroup | Relationship | Zona a que o lugar pertence | Entidade `ParkingGroup`, ex.: `urn:ngsi-ld:ParkingGroup:abrantes:111`. |
| refDevice | Relationship | Sensor do lugar | Entidade `Device`, ex.: `urn:ngsi-ld:Device:abrantes-parking-sensor-SNS-0260`. No modelo é uma lista; a PGU publica um único valor. |
| sourceId | String | Identificador do lugar na origem | Ex.: `urn:ngsi-ld:ParkingSpot:260`. Atributo adicional, conservado para rastreabilidade. |
| source | String | Origem dos dados da entidade | `entiservices:<servidor da API>`. |
| address | Object | Morada do lugar | Atributo previsto, ainda não publicado. |

O modelo do CNMD tem ainda `length`, `width`, `refParkingSite`, `name` e `dataProvider`, que a PGU não publica; estão previstos no [JSON Schema](schema/ParkingSpot.schema.json), como opcionais.

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `ParkingSpot` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Não | Ver [Diferenças](#diferenças-em-relação-ao-cnmd). |
| `location` | Sim | |
| `category` | Sim | |
| `refParkingSite` | Não | O lugar refere a zona (`refParkingGroup`), e não um parque. |
| `status` | Sim | |
| `dataProvider` | Não | |

O [JSON Schema](schema/ParkingSpot.schema.json) deste repositório não exige `address`, `refParkingSite` nem `dataProvider` e considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `category`, `location`, `status`, `refParkingGroup`, `TimeInstant` e `source`.

## Modelo `ParkingGroup`

Cada zona de estacionamento tem uma entidade, com o identificador `urn:ngsi-ld:ParkingGroup:<fonte>:<zona>` (zonas 111 a 119). A entidade é criada na PGU, com o nome da zona fornecido pelo município; os valores agregados são calculados periodicamente pela plataforma a partir do `r_prk_slot` dos lugares da zona.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:ParkingGroup:<fonte>:<zona>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `ParkingGroup`. |
| name | String | Nome da zona | Fornecido pelo município. |
| totalSpotNumber | Number | Número de lugares da zona | Calculado pela plataforma. |
| occupiedSpotNumber | Number | Lugares ocupados | Lugares com `r_prk_slot` igual a 1. Atributo adicional. |
| availableSpotNumber | Number | Lugares livres | Total menos ocupados. |
| location | GeoJSON | Localização da zona | `Point`: o centróide dos lugares da zona. |

O modelo do CNMD tem ainda propriedades que a PGU não publica, como `address`, `category`, `refParkingSite`, `refParkingSpot`, `chargeType`, `allowedVehicleType`, `parkingMode`, `maximumParkingDuration` e `requiredPermit`. Estão previstas no [JSON Schema](schema/ParkingGroup.schema.json), como opcionais.

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `ParkingGroup` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Não | Ver [Diferenças](#diferenças-em-relação-ao-cnmd). |
| `refParkingSite` | Não | |
| `category` | Não | |
| `totalSpotNumber` | Sim | |
| `dataProvider` | Não | |

O [JSON Schema](schema/ParkingGroup.schema.json) deste repositório não exige `address`, `refParkingSite`, `category` nem `dataProvider` e considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `name`, `location`, `totalSpotNumber`, `occupiedSpotNumber` e `availableSpotNumber`.

## Modelo `Device`

A vertical tem dois tipos de dispositivo, ambos com o tipo `Device`:

- **Sensores dos lugares de estacionamento**, com o identificador `urn:ngsi-ld:Device:<fonte>-parking-sensor-<sensor>`, em que `<sensor>` é o identificador do sensor na origem. Só têm a identidade, a localização e o lugar que monitorizam.
- **Câmaras de contagem de tráfego**, cujos atributos estão previstos mas ainda não são publicados (ver [Notas](#notas)).

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Device:<fonte>-parking-sensor-<sensor>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `Device`. |
| serialNumber | String | Identificador do sensor na origem | Ex.: `SNS-0260`. |
| controlledAsset | Array | Lugar que o sensor monitoriza | Lista com a entidade `ParkingSpot`, ex.: `["urn:ngsi-ld:ParkingSpot:abrantes-parking-spot-260"]`. |
| location | GeoJSON | Localização do dispositivo | `Point`. |
| deviceState | String | Estado operacional | Nas câmaras: `active` (ativa), `fault` (avaria) ou `communication` (problema de comunicação). Atributo previsto, ainda não publicado. |
| dataProvider | String | Fornecedor dos dados | Município de origem dos dados (hoje, `abrantes`). Atributo previsto, ainda não publicado. |
| source | String | Origem dos dados da entidade | `entiservices:<servidor da API>` nos sensores de estacionamento; o fornecedor das câmaras (por definir) nas câmaras. |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `Device` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Não | |
| `controlledProperty` | Não | |
| `refDeviceModel` | Não | |
| `value` | Não | |
| `deviceCategory` | Não | |
| `dataProvider` | Não | Só nas câmaras (previsto). |

O [JSON Schema](schema/Device.schema.json) deste repositório não exige estes seis últimos atributos e considera obrigatórios `id`, `type` e `source`.

## Diferenças em relação ao CNMD

1. **`address` não é publicada** nos lugares (`ParkingSpot`), nas zonas (`ParkingGroup`) nem nos dispositivos (`Device`), e está previsto atribuí-la por ponto de contagem nas contagens (`TrafficFlowObserved`). O CNMD exige-a em todos os modelos; nesta vertical, a localização é dada por `location`.
2. **`ParkingSpot` refere a zona e não o parque.** `refParkingSite`, obrigatório no CNMD, não é publicado: cada lugar refere o seu grupo (`refParkingGroup`), e o grupo é a zona do município.
3. **`ParkingSpot.refDevice` é uma relação com um único sensor.** No CNMD é uma lista de referências; cada lugar tem um único sensor.
4. **`ParkingGroup` não publica `category`, `refParkingSite` nem `dataProvider`**, obrigatórios no CNMD, porque o município não fornece esta informação.
5. **`Device` não publica `controlledProperty`, `refDeviceModel`, `value`, `deviceCategory` nem `address`**, obrigatórios no CNMD, porque a API de Abrantes só indica o identificador do sensor, a sua localização e o lugar que monitoriza.
6. **`dataProvider` não é publicado** nos lugares, nas zonas nem nos sensores de estacionamento.
7. **Atributos adicionais:** `r_prk_slot`, `TimeInstant` e `sourceId` no `ParkingSpot`; `occupiedSpotNumber` no `ParkingGroup`; `countType`, `aggregationPeriod` e `trafficPattern` no `TrafficFlowObserved`. Não existem nos modelos do CNMD nem dos Smart Data Models e, conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
8. **`location` não inclui a metainformação `coordinateSystem`** (`EPSG:4258`) indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
9. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD.

## Notas

Os eventos de veículos e de pessoas partilham o mesmo ponto de acesso da origem; a PGU publica como `TrafficFlowObserved` os de veículos, e os de pessoas pertencem à vertical do fluxo de pessoas e turistas.

O `r_prk_slot` só é publicado com os estados `free` (0) e `occupied` (1). Com `closed`, `unknown` ou qualquer estado novo, só é publicado `status`, e o lugar mantém o último `r_prk_slot`; a plataforma pode ser configurada para publicar estes estados como ocupados, para que nunca contem como livres.

Os atributos descritos como «previstos» (`laneDirection`, `aggregationPeriod`, `trafficPattern`, `deviceState` nas câmaras e `dataProvider` nos sensores) fazem parte do modelo de dados da vertical, mas a ingestão atual não os publica: são opcionais nos schemas. O fornecedor das câmaras e o contexto JSON-LD próprio da PGU, que define os atributos adicionais, estão por definir.

As relações `refParkingGroup`, `refDevice` e `controlledAsset` referem entidades desta vertical: `ParkingGroup` e `Device`.

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `TrafficFlowObserved` | [exemplos/TrafficFlowObserved.json](exemplos/TrafficFlowObserved.json) | [schema/TrafficFlowObserved.schema.json](schema/TrafficFlowObserved.schema.json) |
| `ParkingSpot` | [exemplos/ParkingSpot.json](exemplos/ParkingSpot.json) | [schema/ParkingSpot.schema.json](schema/ParkingSpot.schema.json) |
| `ParkingGroup` | [exemplos/ParkingGroup.json](exemplos/ParkingGroup.json) | [schema/ParkingGroup.schema.json](schema/ParkingGroup.schema.json) |
| `Device` (sensor de estacionamento) | [exemplos/Device-sensor-estacionamento.json](exemplos/Device-sensor-estacionamento.json) | [schema/Device.schema.json](schema/Device.schema.json) |
| `Device` (câmara de tráfego) | [exemplos/Device-camara.json](exemplos/Device-camara.json) | [schema/Device.schema.json](schema/Device.schema.json) |

Os exemplos estão em formato NGSI-LD normalizado. Os schemas derivam dos schemas do CNMD, adaptados às diferenças acima, e validam a representação simplificada (*key-values*) das entidades.
