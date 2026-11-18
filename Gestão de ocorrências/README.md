# Gestão de ocorrências (V11)

## Descrição

A **gestão de ocorrências** municipais de não emergência é representada na PGU com dois modelos dos [Smart Data Models](https://github.com/smart-data-models/dataModel.IssueTracking), que transpõem para NGSI o standard [Open311 GeoReport v2](http://wiki.open311.org/GeoReport_v2/), usado por vários municípios para gerir ocorrências:

| Modelo | Representa | Modelo de referência |
|--------|------------|----------------------|
| `Open311_ServiceRequest` | Uma ocorrência (pedido de serviço) | [`service_requests`](https://github.com/smart-data-models/dataModel.IssueTracking/tree/master/service_requests) (dataModel.IssueTracking, versão 0.1.0) |
| `services` | Uma tipologia de ocorrência | [`services`](https://github.com/smart-data-models/dataModel.IssueTracking/tree/master/services) (dataModel.IssueTracking, versão 0.1.0) |

O Catálogo Nacional de Modelos de Dados (CNMD) tem o modelo [`Incident`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Popula%C3%A7%C3%A3o%20e%20sociedade/Socio-econ%C3%B3micos/Ocorr%C3%AAncias/Ocorr%C3%AAncias%20de%20n%C3%A3o%20emerg%C3%AAncia) para as ocorrências de não emergência. O modelo de dados da PGU adota o Open311 porque a lista de tipologias é própria de cada município e é gerida como entidades `services`, que cada município pode criar e alterar sem mudar o modelo das ocorrências; no `Incident` do CNMD, a categoria é uma lista fechada de 17 valores. A correspondência entre os dois modelos é a seguinte:

| `Incident` (CNMD) | Na PGU | Nota |
|-------------------|--------|------|
| `id` | `id` | |
| `type` | `type` | `Open311_ServiceRequest`. |
| `category` | `service_code`, `service_name` | A tipologia do município, descrita na entidade `services` correspondente (com o seu grupo em `group`). |
| `description` | `description` | Só em Ourém. |
| `status` | `status`, `incidentStatus` | Ver [Diferenças](#diferenças-em-relação-ao-cnmd-e-aos-modelos-de-referência). |
| `dateCreated`, `startDate` | `requested_datetime` | As origens só indicam a data de registo da ocorrência. |
| `dateModified` | `dateModified` | |
| `endDate` | `updated_datetime` | Nas ocorrências com `status` `closed`, `updated_datetime` é a data do fecho. |
| `notes` | `status_notes` | Só em Ourém. |
| `address` | `address` | Com outra semântica dos campos; ver [Diferenças](#diferenças-em-relação-ao-cnmd-e-aos-modelos-de-referência). |
| `location` | `location` | |
| `source` | `source` | |
| `expectedResponseTime` | `expected_datetime` | Não publicado. |
| `assignedTo` | `agency_responsible` | Não publicado. |
| `images` | `media_url` | Não publicado. |
| `reportedById` | `account_id` | Não publicado (dados pessoais). |
| `name`, `areaServed`, `severity`, `urgencyLevel` | — | Sem equivalente no Open311; não publicados. |

Os modelos utilizam o formato NGSI-LD, sendo compatíveis com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

## Origem dos dados

Os dados provêm dos sistemas de gestão de ocorrências de dois municípios:

| Município | Origem | Ocorrências | Tipologias | Registadas entre | Leitura |
|-----------|--------|-------------|------------|------------------|---------|
| Ourém | API ENTI do município (`/ocorrencias`) | 1550 | 54, em 13 grupos | 01/01/2019 e 20/10/2026 | 22/10/2026 |
| Abrantes | API EntiServices do município (`/analitica/ocorrencias`), servidor de desenvolvimento | 112 | 6 | 25/03/2026 e 13/09/2026 | 26/10/2026 |

A ingestão na PGU é horária, através do IoT Agent JSON da plataforma iris360: em cada execução só são enviadas as ocorrências que mudaram, precedidas das tipologias que usam. Em Ourém não são publicados os registos anteriores a 01/01/2019.

As tipologias obtêm-se das próprias ocorrências:

- **Ourém:** cada par tipo/subtipo da origem é uma tipologia. O tipo é o grupo (`group`), por exemplo `Jardins e espaços verdes`, e o subtipo é a designação (`service_name`), por exemplo `Queda de ramos ou árvores`.
- **Abrantes:** a origem indica a categoria umas vezes como código (`cleanliness`) e outras como designação (`Limpeza`). As duas formas são agrupadas numa só tipologia, com o código como `service_code` e a designação como `service_name` e `group`: `Limpeza`, `Infraestrutura`, `Serviços Públicos`, `Outros`, `Segurança` e `Ambiente`. As ocorrências sem categoria ficam em `Outros`.

O estado da origem é publicado tal como vem em `sourceStatus` e convertido para o estado Open311 (`status`) e para o estado detalhado do fluxo municipal (`incidentStatus`):

| Município | Estado na origem | `status` | `incidentStatus` | Ocorrências |
|-----------|------------------|----------|------------------|-------------|
| Ourém | `Analise`, `Validacao`, `Sugestao` | `open` | `reportada` | 185 |
| Ourém | `Encaminhado`, `Resolucao` | `open` | `em_curso` | 403 |
| Ourém | `Concluido` | `closed` | `resolvida` | 740 |
| Ourém | `Arquivado`, `Invalidas` | `closed` | (omitido) | 222 |
| Abrantes | `open` | `open` | `reportada` | 32 |
| Abrantes | `inProgress` | `open` | `em_curso` | 60 |
| Abrantes | `resolved` | `closed` | `resolvida` | 15 |
| Abrantes | `closed` | `closed` | (omitido) | 5 |

## Modelo `Open311_ServiceRequest`

Cada ocorrência tem uma entidade, com o identificador `urn:ngsi-ld:Open311_ServiceRequest:<município>-<identificador>`, em que `<identificador>` é o identificador da ocorrência na origem (ex.: `ourem-1920`, `abrantes-2026-X22`). O histórico dos estados obtém-se pela API temporal NGSI-LD: cada atualização fica registada com o seu instante de observação (`observedAt`), que é a data e hora da última alteração (`dateModified`).

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Open311_ServiceRequest:<município>-<identificador>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `Open311_ServiceRequest`. |
| service_request_id | String | Identificador da ocorrência | Ourém: o identificador numérico da origem, ex.: `1920`. Abrantes: o identificador da origem com o prefixo do município, ex.: `abrantes-2026-X22`. |
| incidentCode | String | Código da ocorrência no sistema municipal | Ex.: `30_2026` (Ourém), `2026-X22` (Abrantes). Atributo adicional. |
| service_code | String | Código da tipologia | Referencia a entidade `services` da tipologia. Ourém: grupo e subtipo, ex.: `jardins-e-espacos-verdes--queda-de-ramos-ou-arvores`. Abrantes: o código da categoria, ex.: `cleanliness`. |
| service_name | String | Designação da tipologia | Ex.: `Queda de ramos ou árvores`, `Limpeza`. |
| jurisdiction_id | String | Entidade responsável | O município: `ourem` ou `abrantes`. |
| description | String | Descrição da ocorrência | Só em Ourém (todas as ocorrências). |
| location | GeoJSON | Localização da ocorrência | `Point`, em todas as ocorrências. |
| address | Object | Morada da ocorrência | Ver a tabela seguinte. |
| address_string | String | Descrição livre do local | Só em Ourém, em 2 ocorrências. |
| status | String | Estado Open311 | Enumerado: `open` ou `closed`. |
| incidentStatus | String | Estado detalhado do fluxo municipal | Enumerado: `reportada`, `em_curso` ou `resolvida`. Omitido nas ocorrências fechadas sem resolução. Atributo adicional. |
| sourceStatus | String | Estado na origem | Ex.: `Concluido`, `inProgress`. Atributo adicional. |
| sourceCategory | String | Categoria na origem | Só em Abrantes, em 103 das 112 ocorrências. Atributo adicional. |
| status_notes | String | Justificação ou detalhe do estado | Só em Ourém: a descrição da resolução, em 521 ocorrências. |
| requested_datetime | DateTime | Data e hora do registo da ocorrência | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. |
| updated_datetime | DateTime | Data e hora da última alteração | Nas ocorrências com `status` `closed`, é a data e hora do fecho (semântica Open311). Ver [Notas](#notas). |
| dateModified | DateTime | Data e hora da última alteração na origem | Igual a `updated_datetime`, exceto em 77 ocorrências fechadas de Ourém, cuja data de fecho é anterior. |
| documentRecord | String | Número de registo no sistema de gestão documental | Só em Ourém, em 256 ocorrências, ex.: `MGD Nª. 32002/2026`. Atributo adicional. |
| source | String | Origem dos dados da entidade | `enti:<servidor da API>` (Ourém) ou `entiservices:<servidor da API>` (Abrantes). |

A morada (`address`) é publicada com os seguintes campos:

| Campo | Ourém | Abrantes | No CNMD |
|-------|-------|----------|---------|
| `streetAddress` | Morada da ocorrência | Rua; por vezes com o código postal e a localidade | Endereço da rua |
| `postalCode` | — | Em 40 das 112 ocorrências | Código postal |
| `addressLocality` | Lugar, ex.: `Pinhel` | Município (`Abrantes`) | Município |
| `district` | Freguesia, ex.: `Atouguia` (13 freguesias) | — | Distrito administrativo, ex.: `Santarém` |
| `addressRegion` | Município (`Ourém`) | — | Região NUTS 2, ex.: `Oeste e Vale do Tejo` |
| `addressCountry` | `PT` | `PT` | País, ex.: `Portugal` |

### Propriedades obrigatórias

O modelo `service_requests` dos Smart Data Models só exige `id` e `type`. Os atributos obrigatórios do modelo `Incident` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Não | O tipo é `Open311_ServiceRequest`. |
| `address` | Em parte | Com outra semântica dos campos e sem os campos obrigatórios do CNMD (`addressRegion` e `district` com a região NUTS 2 e o distrito). |
| `dateCreated` | Não | Publicada como `requested_datetime`. |
| `startDate` | Não | Publicada como `requested_datetime`. |
| `location` | Sim | |
| `status` | Em parte | Com os valores do Open311 (`open`, `closed`) e o detalhe em `incidentStatus`. |
| `category` | Não | Publicada como tipologia do município (`service_code`, `service_name`), sem correspondência com a lista do CNMD. |

O [JSON Schema](schema/Open311_ServiceRequest.schema.json) deste repositório considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `service_request_id`, `incidentCode`, `service_code`, `service_name`, `jurisdiction_id`, `location`, `address`, `status`, `sourceStatus`, `requested_datetime`, `updated_datetime`, `dateModified` e `source`.

## Modelo `services`

Cada tipologia tem uma entidade, com o identificador `urn:ngsi-ld:services:<município>-<service_code>` (ex.: `ourem-jardins-e-espacos-verdes--queda-de-ramos-ou-arvores`, `abrantes-cleanliness`). O instante de observação dos seus atributos é a data do registo mais antigo das ocorrências da origem, e não muda entre envios.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:services:<município>-<service_code>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `services`. |
| service_code | String | Código da tipologia | O mesmo `service_code` das ocorrências. |
| service_name | String | Designação da tipologia | Ex.: `Queda de ramos ou árvores`. |
| group | String | Grupo da tipologia | Ourém: o tipo da origem, ex.: `Jardins e espaços verdes`. Abrantes: igual a `service_name`. |
| description | String | Descrição da tipologia | Ourém: grupo e designação, ex.: `Jardins e espaços verdes: Queda de ramos ou árvores`. Abrantes: igual a `service_name`. |
| source | String | Origem dos dados da entidade | O mesmo `source` das ocorrências. |

### Propriedades obrigatórias

O modelo `services` dos Smart Data Models só exige `id` e `type`, ambos publicados. O [JSON Schema](schema/services.schema.json) deste repositório considera obrigatórios os atributos que a PGU publica sempre: `id`, `type`, `service_code`, `service_name`, `group`, `description` e `source`.

## Diferenças em relação ao CNMD e aos modelos de referência

1. **Modelo Open311 em vez do `Incident` do CNMD.** Pelo motivo indicado na [Descrição](#descrição), a PGU segue os modelos Open311 dos Smart Data Models. A correspondência com o `Incident` é a da tabela da Descrição; os obrigatórios do CNMD em falta são os da tabela de [Propriedades obrigatórias](#propriedades-obrigatórias).
2. **Tipo de entidade.** As ocorrências são publicadas com o tipo `Open311_ServiceRequest`, e não `service_requests`, porque é o tipo que o painel de gestão de ocorrências da PGU lê; os atributos são os do modelo `service_requests`. As tipologias mantêm o tipo `services`.
3. **Estado.** `status` tem os valores do Open311 (`open`, `closed`), que não fazem parte da lista do CNMD (`dispatch`, `in progress`, `under resolution`, `monitoring`, `closed`). O estado detalhado do fluxo municipal vai na extensão `incidentStatus` (`reportada`, `em_curso`, `resolvida`), prevista no modelo de dados da PGU, e o estado da origem em `sourceStatus`. As ocorrências fechadas sem resolução (arquivadas ou inválidas) não têm `incidentStatus`.
4. **Categoria.** Em vez da lista de 17 categorias do CNMD, cada ocorrência indica a tipologia do seu município (`service_code` e `service_name`), descrita numa entidade `services`. As tipologias não estão associadas às categorias do CNMD.
5. **Datas.** A data de registo é publicada em `requested_datetime` (no CNMD, `dateCreated` e `startDate`) e a data do fecho em `updated_datetime` das ocorrências fechadas (no CNMD, `endDate`).
6. **Morada.** Os campos de `address` não têm a semântica do CNMD (ver a tabela de [Propriedades](#propriedades)): em Ourém, `addressLocality` é o lugar, `district` a freguesia e `addressRegion` o município; em Abrantes, `addressLocality` é o município, como no CNMD, e não são publicados `district` nem `addressRegion`, que o painel da PGU leria como freguesia e município. O país é publicado com o código ISO `PT`; os exemplos do CNMD usam `Portugal`.
7. **Atributos não publicados, porque as origens não os disponibilizam:** `media_url`, `agency_responsible`, `service_notice` e `expected_datetime` nas ocorrências; `keywords`, `provider` e o prazo de resolução `slaHours` nas tipologias. Também não são publicadas as extensões previstas no modelo de dados da PGU para a ordem de serviço (`workOrderId`), as fotografias (`mediaUrls`) e os duplicados sugeridos (`refSuggestedDuplicates`), nem `dataProvider`. Conforme as FAQ do CNMD, não se publicam valores artificiais.
8. **Dados pessoais.** Os dados de contacto de quem reporta, admitidos pelo modelo `service_requests` (`email`, `first_name`, `last_name`, `phone` e `account_id`), não são publicados, por exigência do RGPD.
9. **Indicadores.** O modelo de dados da PGU prevê indicadores de desempenho (`KeyPerformanceIndicator`): tempo médio de resolução, ocorrências abertas, ocorrências criadas e percentagem resolvida dentro do prazo. Nesta fase não são publicados como entidades; o painel da PGU calcula-os a partir das ocorrências.
10. **Atributos adicionais:** `incidentCode`, `incidentStatus`, `sourceStatus`, `sourceCategory` e `documentRecord` não existem no modelo `service_requests`, e `slaHours` (previsto, não publicado) não existe no modelo `services`. Conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
11. **`location` não inclui a metainformação `coordinateSystem`** indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
12. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD. O identificador inclui o município (`ourem-`, `abrantes-`).

## Notas

A data do fecho (`updated_datetime` das ocorrências fechadas) obtém-se de forma diferente em cada município:

- **Ourém:** é a data da última mudança de estado indicada pela origem e, quando esta não existe, a data da última atualização da ocorrência.
- **Abrantes:** a origem não indica nenhuma data além da de registo. A PGU regista o instante em que deteta cada alteração de uma ocorrência (com a precisão da ingestão, uma hora) e usa-o em `updated_datetime` e `dateModified`. As ocorrências que já estavam fechadas na primeira leitura têm o instante dessa leitura, pelo que os seus tempos de resolução são mais longos do que os reais.

## Qualidade dos dados de origem

A análise dos dados recebidos identificou os seguintes problemas, tratados na ingestão e a rever com os municípios:

| Município | Campo | Problema | Tratamento |
|-----------|-------|----------|------------|
| Ourém | Data da última atualização | 976 das 1550 ocorrências têm a última atualização em 14/12/2022, incluindo 728 das 962 fechadas (522 das 740 concluídas). | Publicada como vem; os tempos de resolução dessas ocorrências não são fiáveis. |
| Ourém | Data da mudança de estado | Vem quase sempre vazia. | Usa-se a data da última atualização (ver [Notas](#notas)). |
| Ourém | Prazos e equipas | A origem não indica o prazo de resolução de cada tipologia nem a equipa responsável. | `expected_datetime`, `slaHours` e `agency_responsible` não são publicados. |
| Ourém | Horas | As datas vêm em hora local, sem fuso horário. | Convertidas de Europe/Lisbon para UTC. |
| Ourém | Freguesias | Os nomes das uniões de freguesias vêm com `Uniăo`, com um carácter errado. | Corrigido para `União`. |
| Abrantes | Servidor | Os dados provêm do servidor de desenvolvimento da API e os códigos das ocorrências (ex.: `2026-X22`) parecem de teste. | Publicados como vêm, até haver dados de produção. |
| Abrantes | Datas | A origem não indica a data da última atualização nem a do fecho. | Registadas pela PGU (ver [Notas](#notas)). |
| Abrantes | Descrição | As ocorrências não têm descrição. | `description` não é publicada. |
| Abrantes | Categoria | A categoria vem umas vezes como código e outras como designação (ex.: `cleanliness` e `Limpeza`); 9 ocorrências não têm categoria. | Agrupadas em 6 tipologias; as que não têm categoria ficam em `Outros`. A categoria original vai em `sourceCategory`. |
| Abrantes | Morada | O código postal só vem em 40 das 112 ocorrências; a rua inclui por vezes o código postal e a localidade. | Publicada como vem. |

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `Open311_ServiceRequest` | [exemplos/Open311_ServiceRequest-ourem.json](exemplos/Open311_ServiceRequest-ourem.json), [exemplos/Open311_ServiceRequest-abrantes.json](exemplos/Open311_ServiceRequest-abrantes.json) | [schema/Open311_ServiceRequest.schema.json](schema/Open311_ServiceRequest.schema.json) |
| `services` | [exemplos/services.json](exemplos/services.json) | [schema/services.schema.json](schema/services.schema.json) |

Os exemplos são entidades reais, tal como são publicadas na PGU, em formato NGSI-LD normalizado: uma ocorrência resolvida de cada município e a tipologia da ocorrência de Ourém. Não contêm dados pessoais. Os schemas derivam dos schemas `service_requests` e `services` dos Smart Data Models (versão 0.1.0), adaptados às diferenças acima, e validam a representação simplificada (*key-values*) das entidades.
