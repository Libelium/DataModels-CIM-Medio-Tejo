# Gestão de ativos (V7)

## Descrição

Os modelos de dados utilizados para representar a **gestão de ativos municipais** são:

- [`WasteContainer`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Ambiente/Res%C3%ADduos/N%C3%ADveis%20de%20enchimento%20dos%20contentores) do Catálogo Nacional de Modelos de Dados (CNMD), para as papeleiras;
- [`WasteContainerModel`](https://github.com/smart-data-models/dataModel.WasteManagement/tree/master/WasteContainerModel) dos Smart Data Models, para os modelos de papeleira, porque o CNMD não tem este modelo;
- [`PointOfInterest`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Comuns/POI) do CNMD, para os multibancos, bancos, bebedouros, paragens de autocarro e pontos de interesse municipais.

Todos os modelos utilizam o formato NGSI-LD, sendo compatíveis com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

## Origem dos dados

| Município | Origem | Camada | Modelo | Registos na origem | Entidades publicadas |
|-----------|--------|--------|--------|--------------------|----------------------|
| Ourém | API de ativos municipais (ENTI) | Papeleiras | `WasteContainer` e `WasteContainerModel` | 473 | 464 papeleiras e 8 modelos |
| Ourém | API de ativos municipais (ENTI) | Multibancos | `PointOfInterest` | 48 | 48 |
| Ourém | API de ativos municipais (ENTI) | Bancos e bebedouros | `PointOfInterest` | 0 | 0 |
| Abrantes | SIG municipal (EntiServices, `/analitica/sig/pontos-interesse`) | Paragens de autocarro e pontos de interesse | `PointOfInterest` | 282 | 282 |

Os números correspondem às leituras de 22/10/2026 (Ourém) e 26/10/2026 (Abrantes). A ingestão na PGU é diária, através do IoT Agent JSON da plataforma iris360, e só são reenviados os ativos novos ou alterados. As papeleiras sem coordenadas (9) não são publicadas; os pontos do SIG de Abrantes sem coordenadas são publicados sem `location`.

A API de Ourém ainda não tem bancos nem bebedouros. **O exemplo de banco deste diretório é ilustrativo:** foi gerado com a mesma regra de conversão que a PGU aplica a essas camadas e não corresponde a um registo real.

## Modelo `WasteContainer`

Cada papeleira tem uma entidade, com o identificador `urn:ngsi-ld:WasteContainer:ourem-papeleira-<id>`, em que `<id>` é o identificador da papeleira na origem.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:WasteContainer:ourem-papeleira-<id>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `WasteContainer`. |
| name | String | Nome da papeleira | `Papeleira <id>`. |
| location | GeoJSON | Localização da papeleira | `Point` em WGS84, copiado da origem. |
| address | Object | Morada | `addressRegion` com o município (`Ourém`) e `addressCountry` (`PT`). Ver [Diferenças](#diferenças-em-relação-aos-modelos-de-referência). |
| category | String | Categoria do contentor | Valor `fixed` (papeleira fixa na via pública). |
| storedWasteOrigin | String | Origem dos resíduos | Valor `municipal`. |
| binCapacity | Number | Capacidade da papeleira | Em litros, lida do texto livre da origem («60 litros», «50 L», «100L»). Publicada em 108 papeleiras. |
| refWasteContainerModel | Relationship | Modelo da papeleira | Referência à entidade `WasteContainerModel`, quando o texto de modelo da origem é reconhecido (100 papeleiras). |
| serialNumber | String | Número de série | Código numérico que precede o modelo na origem (ex.: «133 - SHEET da Larus» dá `133`); o significado está por confirmar com o município. Publicado em 66 papeleiras. |
| source | String | Origem dos dados da entidade | `enti:<servidor da API>`. |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `location` | Sim | |
| `address` | Sim | Com o município em `addressRegion`, sem `addressLocality` nem `district`. |
| `binCapacity` | Em parte | Só nas 108 papeleiras com capacidade na origem. |
| `binColor` | Não | A origem não a disponibiliza. |
| `binFullnessThreshold` | Não | A origem não o disponibiliza; as papeleiras não têm sensor de enchimento. |
| `status` | Não | A origem não o disponibiliza. |
| `storedWasteKind` | Não | A origem não o disponibiliza. |
| `dataProvider` | Não | Não é publicado nas entidades de Ourém. |

O [JSON Schema](schema/WasteContainer.schema.json) deste repositório considera obrigatórios os atributos `id`, `type`, `name`, `location`, `address`, `category`, `storedWasteOrigin` e `source`.

## Modelo `WasteContainerModel`

Cada família de modelo reconhecida no texto livre da origem (73 variantes) tem uma entidade, com o identificador `urn:ngsi-ld:WasteContainerModel:ourem-model-<família>`. As dimensões são as mais frequentes entre as papeleiras desse modelo, tal como a origem as indica.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:WasteContainerModel:ourem-model-<família>`. |
| type | String | Tipo de entidade | Valor constante igual a `WasteContainerModel`. |
| name | String | Nome do modelo | Marca e modelo, ex.: `Larus Sheet`. |
| brandName | String | Marca | Quando o texto da origem a indica. |
| modelName | String | Modelo dado pelo fabricante | Quando o texto da origem o indica. |
| dimensions | String | Dimensões, tal como indicadas na origem | Ex.: `525*370*840mm`, `A840*D350mm`. Atributo adicional (ver [Diferenças](#diferenças-em-relação-aos-modelos-de-referência)). |
| category | String | Categoria | Valor `fixed`, a mesma das papeleiras. |
| source | String | Origem dos dados da entidade | `enti:<servidor da API>`. |

As famílias reconhecidas são:

| Família | `brandName` | `modelName` | `dimensions` | Papeleiras |
|---------|-------------|-------------|--------------|------------|
| `larus-sheet` | Larus | Sheet | `525*370*840mm` | 35 |
| `citi-xxi-dae-nova` | CITI XXI | DAE Nova Simples | — | 28 |
| `atlantica-p1` | IETA design | Atlântica P1 (IDMU00040) | `350*930*500mm` | 11 |
| `almoverde-prima-linea` | Almoverde | Prima Linea | `A840*D350mm` | 10 |
| `larus-serralves` | Larus | Serralves | `550*950mm` | 8 |
| `veco-pa-15a` | Veco Urban Design | PA 15A | `D380*884mm` | 5 |
| `desli` | — | Desli | — | 2 |
| `fabregas` | Fábregas | — | — | 1 |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo dos Smart Data Models são `id`, `type` e `name`, todos publicados. O [JSON Schema](schema/WasteContainerModel.schema.json) deste repositório considera também obrigatórios `category` e `source`.

## Modelo `PointOfInterest`

Cada ponto tem uma entidade. A camada de origem distingue-se pelo identificador e pela categoria:

| Camada | Identificador | `category` | Subcategoria do CNMD |
|--------|---------------|------------|----------------------|
| Multibancos (Ourém) | `urn:ngsi-ld:PointOfInterest:ourem-atm-<id>` | `159` (ATM na taxonomia Factual) | Não existe |
| Bancos (Ourém) | `urn:ngsi-ld:PointOfInterest:ourem-banco-<id>` | `bench` | Não existe |
| Bebedouros (Ourém) | `urn:ngsi-ld:PointOfInterest:ourem-bebedouro-<id>` | `drinkingWater` | `Fontanário/ponto de água` |
| Paragens de autocarro (Abrantes) | `urn:ngsi-ld:PointOfInterest:abrantes-sig-<camada>-<n>` | `Paragem de autocarro` | `Paragem de autocarro` |
| Pontos de interesse (Abrantes) | `urn:ngsi-ld:PointOfInterest:abrantes-sig-<camada>-<n>` | A categoria da origem (ex.: `Farmácia`, `Igreja`); `Pontos de Interesse` quando a origem não indica nenhuma (77 pontos) | A mesma, exceto `Pontos de Interesse` |

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | Ver a tabela acima. |
| type | String | Tipo de entidade | Valor constante igual a `PointOfInterest`. |
| name | String | Nome do ponto | Multibancos: designação, rua, número e lugar (ex.: `Multibanco Avenida D. Nuno Alvares Pereira 201, Ourém`). Bancos e bebedouros: `Banco <id>` / `Bebedouro <id>`. Abrantes: o nome da origem. Os nomes repetidos recebem a linha de autocarro ou o identificador, para serem únicos. |
| category | Array | Categorias do ponto | Ver a tabela acima. |
| location | GeoJSON | Localização do ponto | `Point` em WGS84. Em Abrantes, só 49 pontos (as paragens da Rota Amarela) têm coordenadas na origem. |
| address | Object | Morada | `addressRegion` com o município e `addressCountry` (`PT`). Nos multibancos inclui também `streetAddress`, `streetNr` (10 de 48), `postalCode` e `addressLocality` (o lugar). Ver [Diferenças](#diferenças-em-relação-aos-modelos-de-referência). |
| areaServed | String | Freguesia | Só nos multibancos, com a freguesia da origem. |
| description | String | Camada de origem | Só em Abrantes (ex.: `Paragens Rota Amarela`, `Pontos de Interesse`). |
| busLine | String | Linha de autocarro | Só nas paragens de Abrantes, obtida da camada (`Rota Amarela`, `Rota Verde`, `Rota Vermelha`, `Rota Azul`). Atributo adicional. |
| sourceCategory | Array | Categorias originais | Só em Abrantes, quando a origem as indica. Atributo adicional. |
| modelName | String | Modelo do banco ou do bebedouro | Só em bancos e bebedouros, quando a origem o indica. Atributo adicional. |
| serialNumber | String | Código que precede o modelo | Só em bancos e bebedouros, como nas papeleiras. Atributo adicional. |
| dataProvider | String | Fornecedor da entidade de dados | Só em Abrantes: `Câmara Municipal de Abrantes`, como na origem. |
| sourceId | String | Identificador do ponto na origem | Só em Abrantes. Atributo adicional. |
| source | String | Origem dos dados da entidade | `enti:<servidor da API>` (Ourém) ou `entiservices:<servidor da API>` (Abrantes). |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `name` | Sim | |
| `address` | Sim | Com o município em `addressRegion`, sem `district`. |
| `category` | Sim | Com valores fora da tabela de subcategorias do CNMD (ver a tabela acima). |
| `location` | Em parte | Em todos os pontos de Ourém; em 49 dos 282 pontos de Abrantes. |
| `dataProvider` | Em parte | Só em Abrantes. |
| `source` | Sim | |

O [JSON Schema](schema/PointOfInterest.schema.json) deste repositório considera obrigatórios os atributos `id`, `type`, `name`, `category`, `address` e `source`.

## Diferenças em relação aos modelos de referência

1. **Morada.** Em todos os ativos, `address.addressRegion` contém o município (`Ourém`, `Abrantes`) e `addressCountry` o código `PT`; não é publicado `district`, e `addressLocality` só existe nos multibancos, com o lugar. O CNMD exige em `addressRegion` a região NUTS 2 (`Oeste e Vale do Tejo`), o distrito (`Santarém`) e `addressLocality` igual ao município; os exemplos do CNMD indicam o país por extenso (`Portugal`). Esta forma é a que a PGU utiliza hoje para agrupar os ativos por município; a correção para a morada do CNMD está prevista.
2. **Atributos obrigatórios do `WasteContainer` sem valor na origem.** `binColor`, `binFullnessThreshold`, `status` e `storedWasteKind` não são publicados, e `binCapacity` só existe em 108 das 464 papeleiras. Conforme as FAQ do CNMD, não se publicam valores artificiais; estes dados devem ser completados na origem.
3. **`dataProvider`.** Não é publicado nas entidades de Ourém; nas de Abrantes contém o valor da origem.
4. **`category` de `WasteContainer` e `WasteContainerModel` é texto** (`fixed`) e não uma lista. No `WasteContainerModel`, o valor `fixed` também não pertence à enumeração dos Smart Data Models (`dumpster`, `trashCan`, `wheelieBin`, `other`): a PGU usa a categoria das papeleiras.
5. **Dimensões do `WasteContainerModel`.** São publicadas como texto em `dimensions`, tal como a origem as indica, e não em `width`, `depth` e `height` (metros), porque os formatos da origem são heterogéneos e por vezes ambíguos. O peso (`weight`) não é publicado: nenhuma papeleira com modelo reconhecido tem peso na origem.
6. **Categorias do `PointOfInterest`.** No CNMD, `category` tem de usar as subcategorias da sua tabela. Os multibancos (`159`) e os bancos (`bench`) não têm subcategoria correspondente; os bebedouros (`drinkingWater`) correspondem a `Fontanário/ponto de água`; os 77 pontos de Abrantes com `Pontos de Interesse` não têm categoria na origem. As restantes categorias de Abrantes, incluindo `Paragem de autocarro`, pertencem à tabela do CNMD.
7. **`location` em falta em 233 pontos de Abrantes**, porque a origem não tem as suas coordenadas.
8. **Atributos adicionais.** `busLine`, `sourceCategory`, `sourceId`, `modelName` e `serialNumber` no `PointOfInterest`, e `dimensions` no `WasteContainerModel`, não existem nos modelos de referência. Conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
9. **`location` não inclui a metainformação `coordinateSystem`** indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
10. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD.

## Notas

Os ativos são um inventário: o instante de observação (`observedAt`) de cada atributo é o da execução da ingestão em que o ativo foi criado ou alterado. A capacidade (`binCapacity`) é publicada em litros, sem `unitCode`.

## Qualidade dos dados de origem

A análise dos dados recebidos identificou os seguintes problemas, tratados na ingestão e a rever com os municípios:

| Camada / campo | Problema | Tratamento |
|----------------|----------|------------|
| Papeleiras: coordenadas | 9 registos sem coordenadas. | Não publicados (`location` é obrigatória). |
| Papeleiras: capacidade, dimensões, peso, modelo | Preenchidos só em 23 %, 17 %, 3 % e 21 % dos registos. | Publicados quando existem; completar na origem. |
| Papeleiras: modelo | Texto livre com 73 variantes, código numérico no início e um valor que é uma morada. | Agrupado em 8 famílias de modelo; o valor inválido é ignorado. |
| Papeleiras: dimensões | Formatos diferentes e um valor implausível (`D38*884mm`). | Publicadas como texto, sem interpretação. |
| Multibancos: código postal | 21 de 48 só com o CP4 (`2435-`). | Publicado o CP4; completar na origem. |
| Multibancos: topónimo | Em maiúsculas e com espaços a mais. | Normalizado. |
| Multibancos: lugar | 1 registo com lugar vazio. | Omitido em `addressLocality` e no nome. |
| Bancos e bebedouros | A API ainda não tem registos. | Nada a publicar. |
| Abrantes: coordenadas | 233 dos 282 pontos sem coordenadas. | Publicados sem `location`; completar na origem. |
| Abrantes: categoria | 77 pontos sem categoria. | Publicados com a designação da camada (`Pontos de Interesse`). |

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `WasteContainer` | [exemplos/WasteContainer.json](exemplos/WasteContainer.json) | [schema/WasteContainer.schema.json](schema/WasteContainer.schema.json) |
| `WasteContainerModel` | [exemplos/WasteContainerModel.json](exemplos/WasteContainerModel.json) | [schema/WasteContainerModel.schema.json](schema/WasteContainerModel.schema.json) |
| `PointOfInterest` (multibanco) | [exemplos/PointOfInterest-multibanco.json](exemplos/PointOfInterest-multibanco.json) | [schema/PointOfInterest.schema.json](schema/PointOfInterest.schema.json) |
| `PointOfInterest` (paragem de autocarro) | [exemplos/PointOfInterest-paragem.json](exemplos/PointOfInterest-paragem.json) | [schema/PointOfInterest.schema.json](schema/PointOfInterest.schema.json) |
| `PointOfInterest` (banco, ilustrativo) | [exemplos/PointOfInterest-banco.json](exemplos/PointOfInterest-banco.json) | [schema/PointOfInterest.schema.json](schema/PointOfInterest.schema.json) |

Os exemplos de papeleira, modelo, multibanco e paragem são entidades reais tal como são publicadas na PGU, em formato NGSI-LD normalizado. Os schemas derivam do schema do modelo de referência (CNMD ou Smart Data Models), adaptado às diferenças acima, e validam a representação simplificada (*key-values*) das entidades.
