# Gestão de auxílios sociais (V5)

## Descrição

O modelo de dados utilizado para representar a **gestão de apoios sociais** é o [`SocialSupportObservation`](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Governo%20e%20setor%20p%C3%BAblico/Gest%C3%A3o%20de%20apoio%20social) do Catálogo Nacional de Modelos de Dados (CNMD). Este modelo utiliza o formato NGSI-LD, sendo compatível com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

Cada entidade representa o valor de um indicador de apoio social num período de referência. Na PGU, o modelo é utilizado para os dados de apoios sociais do Município de Abrantes.

## Origem dos dados

Os dados provêm da API EntiServices do Município de Abrantes (endpoint `/analitica/apoios-sociais`), que já descreve cada registo como uma entidade do tipo `SocialSupportObservation`. A ingestão na PGU é diária, através do IoT Agent JSON da plataforma iris360: é criada uma entidade por registo da origem e só são reenviados os registos novos ou alterados.

Em 27/10/2026 a origem disponibilizava 258 registos, todos do indicador `atendimento` ao nível de agregação `individual`, isto é, um registo por atendimento dos serviços de ação social, entre 19/03/2026 e 15/09/2026. Os registos não contêm dados pessoais.

## Propriedades

Na tabela abaixo são apresentadas as propriedades publicadas na PGU.

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:SocialSupportObservation:abrantes-social-<id>`, em que `<id>` é o último segmento do identificador da origem sem o município (ex.: `urn:ngsi-ld:SocialAction:abrantes-2026-1000245641` dá `abrantes-social-2026-1000245641`). Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `SocialSupportObservation`. |
| address | Object | Morada associada à observação | Morada do município, sem campos vazios. Modelo: [https://schema.org/address](https://schema.org/address). A localidade coincide com o município e a região corresponde à NUTS 2 conforme a nomenclatura do INE. |
| address.addressCountry | String | O país | Valor `Portugal`. Modelo: [https://schema.org/addressCountry](https://schema.org/addressCountry) |
| address.addressLocality | String | A localidade, coincidente com o município | Ex.: `Abrantes`. Modelo: [https://schema.org/addressLocality](https://schema.org/addressLocality) |
| address.addressRegion | String | A região em que se situa a localidade | Valor `Oeste e Vale do Tejo` (NUTS 2). |
| address.district | String | O distrito em que se situa a localidade | Valor `Santarém`. |
| aggregationLevel | String | Nível de agregação dos dados | Atualmente `individual` (um registo por atendimento). Outros níveis enviados pela origem (ex.: `facility`, `parish`, `municipality`) são publicados sem alterações. |
| indicator | String | Indicador observado | Atualmente `atendimento`. Novos indicadores enviados pela origem são publicados sem alterações. |
| indicatorValue | Number | Valor numérico do indicador | Corresponde à propriedade `value` do CNMD (ver [Diferenças em relação ao CNMD](#diferenças-em-relação-ao-cnmd)). É uma contagem; a unidade (`C62`, código UN/CEFACT) não é enviada. |
| location | GeoJSON | Localização da observação | Só é publicada quando a origem envia um ponto válido diferente de `[0, 0]`. Atualmente nenhum registo tem localização. |
| source | String | Origem dos dados da entidade | No formato `entiservices:<servidor da API>`. |
| sourceId | String | Identificador do registo na origem | Ex.: `urn:ngsi-ld:SocialAction:abrantes-2026-1000245641`. |
| validFrom | DateTime | Início do período de referência dos dados | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC e ao segundo. É também o instante de observação (`observedAt`) de todos os atributos da entidade. |
| validTo | DateTime | Fim do período de referência dos dados | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC e ao segundo. Quando a origem indica um fim anterior ao início, é publicado igual a `validFrom`. |

As restantes propriedades do modelo do CNMD (`areaServed`, `description`, `facilityRef`, `facilityType`, `financialPurpose`, `observedBy`, `publicService` e `supportType`) não são publicadas, porque a origem não as disponibiliza.

## Propriedades obrigatórias

Os atributos obrigatórios do modelo do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Sim | |
| `location` | Não | A origem envia `[0, 0]` em todos os registos. |
| `validFrom` | Sim | |
| `validTo` | Sim | |
| `aggregationLevel` | Sim | |
| `indicator` | Sim | |
| `value` | Sim | Publicada como `indicatorValue`. |
| `observedBy` | Não | A origem não a disponibiliza. |
| `publicService` | Não | A origem não a disponibiliza. |

O [JSON Schema](schema/SocialSupportObservation.schema.json) deste repositório considera obrigatórios os atributos `id`, `type`, `address`, `aggregationLevel`, `indicator`, `indicatorValue`, `validFrom` e `validTo`.

## Diferenças em relação ao CNMD

1. **`value` é publicada como `indicatorValue`.** No core context NGSI-LD, o termo `value` está associado a `ngsi-ld:hasValue`, o membro que contém o valor de cada propriedade, pelo que não pode ser usado como nome de atributo. O conteúdo é o mesmo: o valor numérico do indicador.
2. **`location` não é publicada.** Todos os registos da origem têm a localização `[0, 0]`, que não corresponde a nenhuma posição. Conforme as FAQ do CNMD, não se publicam valores artificiais; a localização do serviço ou do atendimento será pedida ao Município de Abrantes. Quando existir, será publicada em GeoJSON, de acordo com as convenções deste repositório.
3. **`observedBy` e `publicService` não são publicadas.** A origem não indica a organização que reporta os dados nem o serviço ou programa de apoio social. Os valores a utilizar serão acordados com o Município de Abrantes.
4. **Atributos adicionais.** `sourceId` (identificador do registo na origem) e `source` (servidor de origem) não existem no modelo do CNMD. Conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
5. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD.

## Notas

O instante de observação (`observedAt`) de cada atributo é igual a `validFrom`, de forma que a série temporal fica na data do atendimento e não na data da ingestão. Os indicadores agregados (por exemplo, o número de atendimentos por mês) calculam-se a partir destas entidades.

Os registos não contêm dados pessoais: nem beneficiário, nem agregado familiar, nem tipo ou montante do apoio. O nível `individual` identifica apenas um atendimento. Se a origem passar a enviar dados que identifiquem pessoas, estes não são publicados: a PGU publica apenas os atributos indicados na tabela de propriedades.

## Qualidade dos dados de origem

A análise dos 258 registos recebidos identificou os seguintes problemas, tratados na ingestão e a rever com o Município de Abrantes:

| Campo | Problema | Tratamento |
|-------|----------|------------|
| `location` | Os 258 registos têm `[0, 0]`. | Não publicada. Pedir a localização do serviço ou do atendimento, se fizer sentido. |
| `value` | Vale 0 em 252 registos e 1 em 6, sem significado documentado. | Publicado como `indicatorValue`. Confirmar com o Município o que mede. |
| `validFrom` / `validTo` | Iguais ao segundo em quase todos os registos; 8 com o fim anterior ao início; 1 período de 56 dias. | O fim anterior ao início passa a ser igual ao início; os restantes publicam-se tal como chegam. |
| `indicator` | Um único indicador (`atendimento`). | Publicado. Novos indicadores entram sem alterações. |
| (em falta) | Sem freguesia, serviço, tipo de apoio nem montante atribuído. | Nada a publicar. Acordar com o Município se forem necessários. |
| `id` | Tipo `SocialAction` no identificador e `SocialSupportObservation` no tipo. | O identificador publicado usa o tipo da entidade; o original fica em `sourceId`. |
| Servidor | Só está disponível o servidor de desenvolvimento da API. | A ingestão passará para o servidor de produção quando este existir. |

## Exemplos e schema

- [exemplos/SocialSupportObservation.json](exemplos/SocialSupportObservation.json): entidade real tal como é publicada na PGU, em formato NGSI-LD normalizado.
- [schema/SocialSupportObservation.schema.json](schema/SocialSupportObservation.schema.json): JSON Schema derivado do schema do CNMD (versão 0.0.2), adaptado às diferenças acima. Valida a representação simplificada (*key-values*) das entidades.
