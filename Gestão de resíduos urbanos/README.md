# Gestão de resíduos urbanos (V9)

## Descrição

O modelo de dados utilizado para representar a **gestão de resíduos urbanos** é o `WasteContainerExtended`, o tipo com que a PGU publica cada contentor de resíduos com o seu nível de enchimento e as recolhas mais recentes. Deriva do modelo [`WasteContainer`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Res%C3%ADduos/N%C3%ADveis%20de%20enchimento%20dos%20contentores) do Catálogo Nacional de Modelos de Dados (CNMD) e reúne numa só entidade informação que o CNMD distribui por três modelos da mesma área temática:

| Informação | Modelo do CNMD | Na PGU |
|------------|----------------|--------|
| Contentor e nível de enchimento | [`WasteContainer`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Res%C3%ADduos/N%C3%ADveis%20de%20enchimento%20dos%20contentores) | Atributos do `WasteContainerExtended` |
| Ilha de contentores | [`WasteContainerIsle`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Res%C3%ADduos/Ilhas%20de%20contentores) | `refWasteContainerIsle` e `isleCode` do contentor |
| Recolha de resíduos | [`WasteObserved`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Res%C3%ADduos/Recolha%20de%20res%C3%ADduos) | Atributos da última recolha do contentor |

O modelo utiliza o formato NGSI-LD, sendo compatível com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

## Origem dos dados

Os dados provêm da API de análise de resíduos do Município de Abrantes (EntiServices), que expõe três coleções:

| Coleção da origem | Tipo na origem | Registos | Uso na PGU |
|-------------------|----------------|----------|------------|
| Contentores (`/analitica/residuos/contentores`) | `WasteContainer` | 2895 | Uma entidade `WasteContainerExtended` por contentor |
| Ilhas (`/analitica/residuos/ilhas`) | `WasteContainerIsle` | 1657 | Só o código da ilha, no nome e em `isleCode` |
| Recolhas (`/analitica/residuos/observacoes`) | `WasteObserved` | 15046 | Atributos da recolha no contentor correspondente |

Os números correspondem à leitura de 26/10/2026; as recolhas abrangem o período de 16/08/2026 a 15/09/2026. A ingestão na PGU é horária, através do IoT Agent JSON da plataforma iris360: cada contentor é reenviado só quando muda, e cada recolha é enviada uma vez.

As recolhas da origem não indicam o contentor esvaziado, apenas a posição. Como todos os contentores de uma ilha partilham as coordenadas, cada recolha é associada à ilha pela posição (100 % das recolhas têm ilha) e as recolhas de uma mesma visita são distribuídas pelos contentores da ilha, numa rotação fixa. Os quilogramas e os esvaziamentos por ilha coincidem com a origem; por contentor, são uma estimativa. Das 2895 entidades, 2464 têm pelo menos uma recolha associada.

## Modelo `WasteContainerExtended`

Cada contentor tem uma entidade, com o identificador `urn:ngsi-ld:WasteContainerExtended:abrantes-waste-container-<n>`, em que `<n>` é o número do contentor na origem. O histórico do nível de enchimento e das recolhas obtém-se pela API temporal NGSI-LD.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:WasteContainerExtended:abrantes-waste-container-<n>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `WasteContainerExtended`. |
| name | String | Nome do contentor | Código da ilha e número do contentor, ex.: `1401ABR9239 2148`. |
| location | GeoJSON | Localização do contentor | `Point`, igual para todos os contentores da ilha. |
| address | String | Morada do contentor, como texto | Rua e localidade, ex.: `rua da cabeça alta, Pego`. Publicada em 1866 contentores. |
| parish | String | Freguesia (localidade) | Ex.: `Pego`. 47 valores distintos. Atributo adicional. |
| municipality | String | Município | Valor `Abrantes`. Atributo adicional. |
| fillingLevel | Number | Nível de enchimento | Fração de 0 a 1, tal como a origem a indica. |
| fillingPercent | Number | Nível de enchimento em percentagem | `fillingLevel` × 100. Atributo adicional. |
| capacity | Number | Capacidade do contentor | Em litros: 1100 (2067 contentores) ou 800 (828). Atributo adicional, com o valor de `binCapacity` da origem. |
| status | String | Estado do contentor do ponto de vista da segurança | Valor `ok` em todos os contentores. |
| storedWasteKind | String | Tipo de resíduos | Valor `other`: a origem não indica o tipo de resíduos por contentor. |
| refWasteContainerIsle | String | Ilha onde está o contentor | Identificador da ilha na origem, ex.: `urn:smabrantes:waste-container-isle:242`. |
| isleCode | String | Código da ilha | Ex.: `1401ABR9239`. Atributo adicional. |
| sourceId | String | Identificador do contentor na origem | Ex.: `urn:smabrantes:waste-container:2148`. Atributo adicional. |
| dateLastEmptying | DateTime | Data e hora da última recolha | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. |
| lastEmptyingDate | DateTime | Data e hora da última recolha | Igual a `dateLastEmptying`. Atributo adicional. |
| weightCollectedKg | Number | Peso recolhido na última recolha | Soma dos pesos das recolhas do contentor nesse instante, tal como a origem os indica. Atributo adicional. |
| emptyingsInPeriod | Number | Esvaziamentos no instante da última recolha | Normalmente 1. Atributo adicional. |
| vehicleId | String | Veículo de recolha | Número do veículo na origem (6 veículos). Atributo adicional. |
| source | String | Origem dos dados da entidade | `entiservices:<servidor da API>`. |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `WasteContainer` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Não | O tipo é `WasteContainerExtended`. |
| `location` | Sim | |
| `address` | Em parte | Como texto, em 1866 dos 2895 contentores. |
| `binCapacity` | Não | Publicada como `capacity`. |
| `binColor` | Não | A origem não a disponibiliza. |
| `binFullnessThreshold` | Não | A origem não o disponibiliza. |
| `status` | Sim | |
| `storedWasteKind` | Sim | Sempre `other`. |
| `dataProvider` | Não | |

O [JSON Schema](schema/WasteContainerExtended.schema.json) deste repositório considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `name`, `location`, `fillingLevel`, `fillingPercent`, `capacity`, `status`, `storedWasteKind`, `municipality`, `refWasteContainerIsle`, `isleCode`, `sourceId` e `source`.

## Diferenças em relação ao CNMD

1. **Tipo de entidade.** A PGU publica `WasteContainerExtended`, e não `WasteContainer`, porque é o tipo que o painel de resíduos urbanos da PGU lê, com os atributos adicionais das recolhas. A correspondência com o CNMD é a da tabela da [Descrição](#descrição).
2. **`address` é texto** (rua e localidade) e não o objeto com a morada estruturada do CNMD; a freguesia e o município são publicados em `parish` e `municipality`. Não é publicada nos 1029 contentores cuja origem não tem rua nem localidade.
3. **A capacidade é publicada em `capacity`**, e não em `binCapacity`, sem `unitCode` (litros). O nível de enchimento é publicado em `fillingLevel` (fração) e repetido em `fillingPercent` (percentagem).
4. **`binColor`, `binFullnessThreshold` e `dataProvider` não são publicados.** A origem não indica a cor nem o limiar de enchimento; conforme as FAQ do CNMD, não se publicam valores artificiais.
5. **As recolhas não são publicadas como entidades `WasteObserved`**, mas como atributos da última recolha do contentor (`dateLastEmptying`, `weightCollectedKg`, `emptyingsInPeriod`, `vehicleId`), com o histórico na API temporal. A atribuição de cada recolha a um contentor é uma estimativa (ver [Origem dos dados](#origem-dos-dados)).
6. **As ilhas não são publicadas como entidades `WasteContainerIsle`.** `refWasteContainerIsle` é uma propriedade com o identificador da ilha na origem, e não uma relação com uma entidade publicada; o código da ilha vai em `isleCode`.
7. **Alertas e indicadores.** O modelo de dados da PGU prevê alertas de nível crítico (`Anomaly`) e indicadores do histórico de enchimento (`KeyPerformanceIndicator`), como nas verticais V4 e V8. Nesta fase não são publicados como entidades. Com a origem atual, o nível de enchimento é 0 em quase todos os contentores (ver [Qualidade dos dados de origem](#qualidade-dos-dados-de-origem)).
8. **Atributos adicionais:** `fillingPercent`, `capacity`, `parish`, `municipality`, `isleCode`, `sourceId`, `lastEmptyingDate`, `weightCollectedKg`, `emptyingsInPeriod` e `vehicleId` não existem no modelo do CNMD. Conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
9. **`location` não inclui a metainformação `coordinateSystem`** indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
10. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD.

## Notas

O instante de observação (`observedAt`) dos atributos do contentor é a hora do último registo do nível na origem (`binLoggedTime`); o dos atributos da recolha é a data e hora da recolha (`dateObserved` da origem). Por isso, numa mesma entidade, os dois grupos de atributos podem ter instantes diferentes.

## Qualidade dos dados de origem

A análise dos dados recebidos identificou os seguintes problemas, tratados na ingestão e a rever com o município:

| Campo | Problema | Tratamento |
|-------|----------|------------|
| Nível de enchimento | `fillingLevel` é 0 em 2890 dos 2895 contentores. | Publicado como vem; o painel fica sem variação até a origem o reportar. |
| Estado | `status` é `ok` em todos os contentores. | Publicado como vem. |
| Hora do nível | Em 160 contentores, o último registo do nível é anterior a 2026 (o mais antigo de 2016). | Publicado como vem. |
| Coordenadas | Os pontos vêm na ordem [latitude, longitude]. | Publicados como [longitude, latitude], a ordem do GeoJSON. |
| Recolhas | Não indicam o contentor (`idUriContainer`), apenas a posição. | Associadas por posição à ilha e distribuídas pelos seus contentores. |
| Peso das recolhas | 13 539 das 15 046 recolhas pesam menos de 1 (mediana 0,01); a unidade está por confirmar. | Publicado como vem, em `weightCollectedKg`. |
| Tipo de resíduos | Todas as recolhas indicam `other` e os contentores não têm tipo. | Publicado `other`. |
| Morada | 1029 contentores sem rua nem localidade; 1044 ruas e 575 localidades em minúsculas. A região indicada é `Centro`, quando Abrantes pertence a `Oeste e Vale do Tejo`. | Publicada como texto quando existe; as localidades em minúsculas são capitalizadas; a região não é usada. |

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `WasteContainerExtended` | [exemplos/WasteContainerExtended.json](exemplos/WasteContainerExtended.json) | [schema/WasteContainerExtended.schema.json](schema/WasteContainerExtended.schema.json) |

O exemplo é uma entidade real, tal como é publicada na PGU, em formato NGSI-LD normalizado: um dos cinco contentores com nível de enchimento diferente de 0, com a sua última recolha. O schema deriva do schema `WasteContainer` do CNMD, adaptado às diferenças acima, e valida a representação simplificada (*key-values*) das entidades.
