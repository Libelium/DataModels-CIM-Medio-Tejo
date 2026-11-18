# Fluxo de pessoas e turistas (V2)

## Descrição

O **fluxo de pessoas e turistas** é representado na PGU com o modelo [`CrowdFlowObserved`](https://github.com/smart-data-models/dataModel.Transportation/tree/master/CrowdFlowObserved) dos Smart Data Models (dataModel.Transportation, versão 0.0.3) da FIWARE Foundation, que descreve a observação do fluxo de pessoas numa via pedonal, num determinado local e momento:

| Modelo | Representa | Modelo de referência |
|--------|------------|----------------------|
| `CrowdFlowObserved` | As contagens de pessoas de cada ponto de contagem, em ruas, praças e largos | [`CrowdFlowObserved`](https://github.com/smart-data-models/dataModel.Transportation/tree/master/CrowdFlowObserved) (dataModel.Transportation, versão 0.0.3) |

O Catálogo Nacional de Modelos de Dados (CNMD) não tem um modelo equivalente: a única área relacionada, [Turismo](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Governo%20e%20setor%20p%C3%BAblico/Turismo), reúne conjuntos de dados do Turismo de Portugal (alojamento, equipamentos e ordenamento turístico). Por isso, e conforme previsto no Caderno de Encargos, foi adotado o modelo dos Smart Data Models.

O modelo utiliza o formato NGSI-LD, sendo compatível com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

## Origem dos dados

Os dados provêm da API EntiServices do município de Abrantes (servidor de desenvolvimento), do ponto de acesso `/analitica/fluxos-pessoas`, que devolve um registo por evento de contagem das câmaras: pessoas e veículos, passagens e permanências. A PGU publica como `CrowdFlowObserved` só os eventos de pessoas (`tipoObjeto` igual a `pessoa`); os de veículos pertencem à vertical de transporte e estacionamento (V1).

A ingestão é feita de 15 em 15 minutos, através do IoT Agent JSON da plataforma iris360. Cada evento é publicado tal como chega, com o seu valor e o seu instante, sem ser somado: as somas por dia ou por hora são feitas na consulta.

Os campos de origem correspondem aos atributos seguintes:

| Campo de origem | Atributos |
|-----------------|-----------|
| `idRua` | O identificador da entidade (ponto de contagem) |
| `address` | `name` |
| `intensity` | `peopleCount` |
| `tipoContagem` | `countType` (`passagem` → `passage`, `permanencia` → `dwell`) |
| `dateObserved` | `dateObserved` |
| `location` | `location` |
| `tipoObjeto` | Só se publicam os eventos com o valor `pessoa`; não é publicado |
| `id`, `idUri` | Não são publicados: identificam o registo individual da origem, e não o ponto de contagem |

**Os exemplos deste diretório são ilustrativos:** têm o mesmo formato com que as entidades são publicadas, mas as coordenadas, os pontos de contagem e os valores não correspondem a dados reais. Os dois exemplos mostram os dois tipos de contagem: uma passagem numa rua e uma permanência numa praça.

## Modelo `CrowdFlowObserved`

Cada ponto de contagem tem uma entidade por tipo de contagem, com o identificador `urn:ngsi-ld:CrowdFlowObserved:abrantes-people-<ponto>-<tipo de contagem>-unspecified`, em que `<ponto>` é o identificador do ponto de contagem na origem (`idRua`; se for numérico, com 8 dígitos, ex.: `00000007`) e `<tipo de contagem>` é `passage` ou `dwell`. O identificador é estável, pelo que a ingestão é idempotente; cada evento atualiza os atributos da entidade, e o histórico obtém-se pela API temporal NGSI-LD. As passagens e as permanências de um ponto são entidades diferentes, porque as contagens dos dois tipos não são somáveis entre si.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:CrowdFlowObserved:abrantes-people-<ponto>-<tipo de contagem>-unspecified`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `CrowdFlowObserved`. |
| name | String | Nome do ponto de contagem | A rua do ponto. |
| peopleCount | Number | Número de pessoas detetadas no evento | Inteiro maior ou igual a 0. |
| countType | String | Tipo de contagem do ponto | `passage` (passagem) ou `dwell` (permanência, em praças e largos); qualquer outro valor da origem é publicado como o seu slug. Atributo adicional. |
| direction | String | Sentido de deslocação na via pedonal | Enumerado: `inbound`, `outbound` ou `unspecified`. A origem não indica o sentido, pelo que a PGU publica sempre `unspecified`. |
| dateObserved | DateTime | Data e hora da observação | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. Instante ou intervalo ISO 8601. |
| location | GeoJSON | Localização do ponto de contagem | `Point`. O modelo admite também `LineString`, `Polygon`, `MultiPoint`, `MultiLineString` e `MultiPolygon`. |
| address | Object | Morada do ponto de contagem | `addressLocality` (o município) e `addressCountry` (`PT`), atribuídos por ponto. Só é publicada se o município estiver configurado. |
| aggregationPeriod | String | Período com que a plataforma agrega os eventos | Duração ISO 8601, ex.: `PT1H`. A origem não o fornece: é definido na integração. Atributo adicional, previsto e ainda não publicado. |
| crowdPattern | Object | Tendências sazonais e padrões de comportamento do fluxo de pessoas | Calculado na camada de relatórios a partir do histórico. Atributo adicional, previsto e ainda não publicado. |
| dataProvider | String | Fornecedor dos dados | `abrantes`. Está previsto passar a `cimt` (CIM Médio Tejo) quando houver dados de outros municípios. |
| source | String | Origem dos dados da entidade | `entiservices:<servidor da API>`. |

O modelo dos Smart Data Models tem ainda propriedades que a PGU não publica, como `peopleCountTowards`, `peopleCountAway`, `occupancy`, `averageCrowdSpeed`, `congested`, `averageHeadwayTime`, `refRoadSegment`, `dateObservedFrom` e `dateObservedTo`. Estão previstas no [JSON Schema](schema/CrowdFlowObserved.schema.json), como opcionais.

### Propriedades obrigatórias

O modelo `CrowdFlowObserved` dos Smart Data Models só exige `id`, `type` e `dateObserved`, que a PGU publica sempre. O [JSON Schema](schema/CrowdFlowObserved.schema.json) deste repositório considera obrigatórios, além destes, os atributos que a PGU publica sempre: `peopleCount`, `direction`, `countType`, `location`, `dataProvider` e `source`. Não exige `address`, que só é publicada se o município estiver configurado.

## Diferenças em relação ao modelo de referência

1. **Sem correspondência no CNMD.** Pelo motivo indicado na [Descrição](#descrição), a PGU segue o modelo dos Smart Data Models. Em consequência, `address` não é obrigatória, ao contrário do que o CNMD exige nos modelos que cobre: é publicada só com o município (`addressLocality`) e o país (`addressCountry`), sem `addressRegion` nem `district`.
2. **`direction` admite `unspecified`.** O modelo só prevê `inbound` e `outbound`; a origem não indica o sentido de deslocação, pelo que a PGU publica `unspecified`. O sentido de cada ponto, quando for conhecido, deve ser documentado por ponto de contagem.
3. **Atributos adicionais:** `countType`, `aggregationPeriod` e `crowdPattern` não existem no modelo dos Smart Data Models e, conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
4. **`dataProvider`** é publicado como `abrantes`, o município de origem dos dados; está previsto aumentá-lo para `cimt` (CIM Médio Tejo) quando houver dados de outros municípios.
5. **`location` não inclui a metainformação `coordinateSystem`** indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
6. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD. O identificador inclui o tipo de contagem e o sentido (`unspecified`), além do ponto de contagem.

## Notas

Os eventos de pessoas e de veículos partilham o mesmo ponto de acesso da origem; a PGU publica como `CrowdFlowObserved` os de pessoas, e os de veículos como `TrafficFlowObserved` (V1).

Os atributos descritos como «previstos» (`aggregationPeriod` e `crowdPattern`) fazem parte do modelo de dados da vertical, mas a ingestão atual não os publica: são opcionais no schema. O fornecedor das câmaras e o contexto JSON-LD próprio da PGU, que define os atributos adicionais, estão por definir.

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `CrowdFlowObserved` (passagem) | [exemplos/CrowdFlowObserved-passagem.json](exemplos/CrowdFlowObserved-passagem.json) | [schema/CrowdFlowObserved.schema.json](schema/CrowdFlowObserved.schema.json) |
| `CrowdFlowObserved` (permanência) | [exemplos/CrowdFlowObserved-permanencia.json](exemplos/CrowdFlowObserved-permanencia.json) | [schema/CrowdFlowObserved.schema.json](schema/CrowdFlowObserved.schema.json) |

Os exemplos estão em formato NGSI-LD normalizado. O schema deriva do schema dos Smart Data Models, adaptado às diferenças acima, e valida a representação simplificada (*key-values*) das entidades.
