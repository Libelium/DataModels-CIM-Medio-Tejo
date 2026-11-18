# Saúde e bem-estar inteligente (V10)

## Descrição

A **saúde e o bem-estar inteligente** (acompanhamento de pessoas idosas isoladas com relógios inteligentes) é representada na PGU com seis modelos: dois do Catálogo Nacional de Modelos de Dados (CNMD) e quatro dos [Smart Data Models](https://github.com/smart-data-models) da FIWARE Foundation, que não têm modelo equivalente no CNMD:

| Modelo | Representa | Modelo de referência |
|--------|------------|----------------------|
| `Device` | Cada relógio: o seu estado, a sua posição e as suas relações com a conta e o paciente | CNMD ([Sensores](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Comuns/Sensores)) |
| `Alert` | Cada queda detetada por um relógio | CNMD ([Alertas e avisos da proteção civil](https://github.com/amagovpt/CATALOGO_MODELOS_DADOS_TI/tree/main/Justi%C3%A7a%2C%20sistema%20judici%C3%A1rio%20e%20seguran%C3%A7a%20p%C3%BAblica/Prote%C3%A7%C3%A3o%20e%20socorro/Alertas%20e%20avisos%20da%20prote%C3%A7%C3%A3o%20civil)) |
| `Patient` | Cada paciente que usa um relógio | Smart Data Models ([Patient](https://github.com/smart-data-models/dataModel.HL7/tree/master/Patient), dataModel.HL7, versão 0.0.2) |
| `Person` | O titular da conta a que estão associados os pacientes e os relógios | Smart Data Models ([Person](https://github.com/smart-data-models/dataModel.Organization/tree/master/Person), dataModel.Organization, versão 0.0.1) |
| `DeviceMeasurement` | Os sinais vitais e os dados de movimento de cada relógio | Smart Data Models ([DeviceMeasurement](https://github.com/smart-data-models/dataModel.Device/tree/master/DeviceMeasurement), dataModel.Device, versão 0.1.0) |
| `DeviceOperation` | Os registos de funcionamento de cada relógio (opcional) | Smart Data Models ([DeviceOperation](https://github.com/smart-data-models/dataModel.Device/tree/master/DeviceOperation), dataModel.Device, versão 0.0.1) |

Os modelos utilizam o formato NGSI-LD, sendo compatíveis com [ETSI GS CIM 009 V1.5.1, Context Information Management (CIM) - NGSI-LD API](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.05.01_60/gs_cim009v010501p.pdf), da [ETSI](https://www.etsi.org/).

> **Nota.** Este modelo segue os Smart Data Models, com extensões, e a especificação da API do fornecedor. As unidades, as escalas e os tipos de evento marcados «por confirmar» dependem do fornecedor e serão ajustados quando forem confirmados.

## Origem dos dados

Os dados provêm da API de parceiros da Idicare, o fornecedor dos relógios. A API é só de leitura, e cada município tem a sua conta: o token de uma conta só vê os dados dessa conta, e a PGU publica os dados de cada município no tenant do próprio município. A ingestão é horária, através do IoT Agent JSON da plataforma iris360.

| Ponto de acesso | Conteúdo | Modelo |
|-----------------|----------|--------|
| `/api/accounts` | O titular da conta | `Person` |
| `/api/patients` | Os pacientes da conta | `Patient` |
| `/api/devices` | Os relógios da conta: `imei`, `accountId`, `watchType`, `active` e `patientId` | `Device` |
| `/api/readings/{imei}` | As leituras de cada relógio | `DeviceMeasurement` e `Device` |
| `/api/falls/{imei}` | As quedas detetadas por cada relógio | `Alert` |
| `/api/logs/{imei}` | Os registos de funcionamento (início de sessão, SOS, relógio não colocado…) | `DeviceOperation` |

A conta, os pacientes e os relógios só são publicados quando mudam; as leituras, as quedas e os registos são publicados uma única vez, por ordem. Cada leitura chega com todos os valores em texto e a maior parte pode ser nula: um valor nulo, vazio ou não numérico é omitido, e um valor zero é um valor. Uma leitura que só traz bateria, sinal ou posição atualiza só o `Device`, e só uma leitura com pelo menos um sinal vital atualiza o `DeviceMeasurement`. Os identificadores têm o formato `urn:ngsi-ld:<Tipo>:idicare:<identificador>`, sem o município: o tenant isola os dados de cada um.

**Os exemplos deste diretório são ilustrativos:** têm o mesmo formato com que as entidades são publicadas, mas o IMEI, os identificadores, a posição e os valores não correspondem a dados reais, e os valores cifrados são aleatórios e não escondem nenhum texto.

## Dados pessoais

Os dados de saúde, o nome, o email, a data de nascimento e a posição de pessoas idosas são dados pessoais, e a sua publicação está **sujeita à validação do encarregado de proteção de dados (DPO), nos termos do RGPD**. Nenhum município deve ser ativado sem essa validação. Em consequência:

- A API do fornecedor devolve em claro o **nome e o email do titular** (`Person`) e o **nome e a data de nascimento do paciente** (`Patient`). A PGU **cifra-os antes de os guardar** (AES-256-GCM, em base64 de IV de 12 bytes, etiqueta de 16 bytes e texto cifrado), pelo que a plataforma só tem os valores cifrados. A chave é própria da PGU e não é conhecida pela plataforma: quem a conhece introduz-a no painel, e é o navegador que decifra os valores, que ficam em claro apenas na memória desse separador. Por serem cifrados, o `name` e o `birthDate` do `Patient` são texto, e não a lista `HumanName` nem a data do FHIR.
- O nome, o email e a data de nascimento só se veem em claro com a chave (palavra-passe), que se introduz no painel; sem ela, só se veem os valores cifrados. A posição do relógio e os sinais vitais não são cifrados.
- A **posição** (`location` do `Device`) é um dado pessoal, e só é gerada quando a latitude e a longitude vêm preenchidas e não são ambas zero (que se toma como «sem posição»).
- Os registos da ingestão nunca mostram nomes, emails, datas de nascimento nem coordenadas.

## Modelo `Device`

Cada relógio tem uma entidade, com o identificador `urn:ngsi-ld:Device:idicare:<imei>`, em que `<imei>` é o IMEI do relógio. Recebe atributos de dois pedidos da origem, com atualização parcial: `/api/devices` (identidade, relações e estado) e as leituras (posição, bateria e sinal).

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Device:idicare:<imei>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `Device`. |
| refAccount | Relationship | Conta a que o relógio está associado | Entidade `Person`, ex.: `urn:ngsi-ld:Person:idicare:102`. Omitida se a origem não a indicar. Atributo adicional. |
| refDeviceModel | Relationship | Modelo do relógio | URN `urn:ngsi-ld:DeviceModel:idicare:<modelo>` (ex.: `L19PRO`). A entidade `DeviceModel` não é criada, porque a origem não dá dados do modelo. Omitida se a origem não o indicar. |
| deviceState | String | Estado do relógio | Enumerado: `active` ou `inactive`, derivado do indicador de atividade da origem. |
| refPatient | Relationship | Paciente que usa o relógio | Entidade `Patient`, ex.: `urn:ngsi-ld:Patient:idicare:7`. Omitida enquanto o relógio não tiver paciente. Atributo adicional. |
| batteryLevel | Number | Nível da bateria | Fração de 0 a 1 (83 % → 0,83): a origem envia uma percentagem, que a PGU divide por 100 (por confirmar com o fornecedor). |
| rssi | Number | Intensidade do sinal móvel | A escala (dBm, percentagem ou barras) está por confirmar, pelo que é publicada sem `unitCode`. |
| location | GeoJSON | Posição do relógio | `Point`, em WGS84 (assumido, por confirmar). Dado pessoal. |
| locationSource | String | Método de localização | Ex.: `GPS`. Os valores possíveis estão por confirmar (previsivelmente `GPS`, `LBS` ou `WiFi`). Atributo adicional. |

O modelo do CNMD tem ainda propriedades que a PGU não publica, como `address`, `controlledProperty`, `value` e `deviceCategory`. Estão previstas no [JSON Schema](schema/Device.schema.json), como opcionais.

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `Device` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Não | A posição é dada por `location`. |
| `controlledProperty` | Não | Ver [Diferenças](#diferenças-em-relação-ao-cnmd-e-aos-modelos-de-referência). |
| `refDeviceModel` | Sim | Omitida se a origem não indicar o modelo. |
| `value` | Não | |
| `deviceCategory` | Não | |
| `dataProvider` | Não | |

O [JSON Schema](schema/Device.schema.json) deste repositório só exige `id` e `type`, porque o relógio recebe os atributos de dois pedidos e com atualização parcial.

## Modelo `Patient`

Cada paciente tem uma entidade, com o identificador `urn:ngsi-ld:Patient:idicare:<paciente>`, em que `<paciente>` é o inteiro da origem (ex.: `7`). O modelo dos Smart Data Models deriva do HL7 FHIR R4B; só `id` e `type` são obrigatórios.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Patient:idicare:<paciente>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `Patient`. |
| refAccount | Relationship | Conta responsável pelo paciente | Entidade `Person`, ex.: `urn:ngsi-ld:Person:idicare:102`. O modelo não tem atributo de conta. Atributo adicional. |
| name | String | Nome do paciente | Dado pessoal: cifrado pela PGU. É um texto, e não a lista `HumanName` do FHIR. |
| birthDate | String | Data de nascimento do paciente | Dado pessoal: cifrada pela PGU (a data de origem é `AAAA-MM-DD`). É um texto, e não uma data. |

O [JSON Schema](schema/Patient.schema.json) deste repositório inclui só estes atributos: os restantes atributos do FHIR (identificadores, género, contactos, morada…) não são publicados.

## Modelo `Person`

Cada conta tem uma entidade, com o identificador `urn:ngsi-ld:Person:idicare:<conta>`, em que `<conta>` é o inteiro da origem (ex.: `102`). A conta é de um particular, que pode ter vários pacientes e relógios associados.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Person:idicare:<conta>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `Person`. |
| name | String | Nome do titular da conta | Dado pessoal: cifrado pela PGU. |
| email | String | Email do titular da conta | Dado pessoal: cifrado pela PGU. Atributo adicional, não existente nos Smart Data Models. |

## Modelo `DeviceMeasurement`

Cada relógio tem uma única entidade de medidas, com o identificador `urn:ngsi-ld:DeviceMeasurement:idicare:<imei>`, que é atualizada com cada leitura; o histórico obtém-se pela API temporal NGSI-LD. A entidade só é criada se a leitura trouxer pelo menos um sinal vital. Os Smart Data Models não têm atributos para sinais vitais, pelo que cada medida é um atributo adicional, com o código LOINC equivalente.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:DeviceMeasurement:idicare:<imei>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `DeviceMeasurement`. |
| refDevice | Relationship | Relógio que efetuou a leitura | Entidade `Device` do relógio. |
| dateObserved | DateTime | Data e hora da leitura | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. A origem envia segundos Unix. |
| heartRate | Number | Frequência cardíaca | Em batimentos por minuto (bpm). LOINC 8867-4. Atributo adicional. |
| bloodPressureSystolic | Number | Pressão arterial sistólica | Em mmHg (unidade assumida, por confirmar). LOINC 8480-6. Atributo adicional. |
| bloodPressureDiastolic | Number | Pressão arterial diastólica | Em mmHg (unidade assumida, por confirmar). LOINC 8462-4. Atributo adicional. |
| bodyTemperature | Number | Temperatura corporal | `unitCode` `CEL` (°C). LOINC 8310-5. Atributo adicional. |
| spo2 | Number | Saturação de oxigénio no sangue | Em percentagem. LOINC 59408-5. Atributo adicional. |
| steps | Number | Número de passos | LOINC 55423-8. Por confirmar se é o acumulado diário ou o de cada leitura. Atributo adicional. |
| altitude | Number | Altitude | Unidade por confirmar (previsivelmente metros). Atributo adicional. |
| locationAccuracy | Number | Precisão da localização | Unidade por confirmar (previsivelmente metros). Atributo adicional. |
| speed | Number | Velocidade de deslocação | Unidade por confirmar. Atributo adicional. |

O modelo dos Smart Data Models tem ainda `numValue`, `textValue`, `controlledProperty`, `deviceType`, `measurementType`, `outlier` e `unit`, que a PGU não publica. Estão previstos no [JSON Schema](schema/DeviceMeasurement.schema.json), como opcionais.

## Modelo `Alert`

Cada queda tem uma entidade, com o identificador `urn:ngsi-ld:Alert:idicare:fall-<queda>`, em que `<queda>` é o inteiro da origem. Só as quedas geram alertas: o SOS e o relógio não colocado são publicados como `DeviceOperation`, sem alerta. O paciente obtém-se através do relógio (`alertSource`, e depois `Device.refPatient`).

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:Alert:idicare:fall-<queda>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `Alert`. |
| category | String | Categoria do alerta | Valor constante igual a `health`. |
| subCategory | String | Subcategoria do alerta | Valor constante igual a `fallenPatient`. |
| severity | String | Gravidade do alerta | Valor constante igual a `high`: uma queda é sempre prioritária. |
| dateIssued | DateTime | Data e hora da queda | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. A origem envia segundos Unix (por confirmar). |
| alertSource | Relationship | Relógio que detetou a queda | Entidade `Device`, ex.: `urn:ngsi-ld:Device:idicare:860000000000012`. |

### Propriedades obrigatórias

Os atributos obrigatórios do modelo `Alert` do CNMD e a sua situação na PGU são:

| Propriedade | Publicada | Nota |
|-------------|-----------|------|
| `id` | Sim | |
| `type` | Sim | |
| `address` | Não | A posição obtém-se através do relógio. |
| `alertSource` | Sim | Relação com o relógio, e não um URL. |
| `category` | Sim | |
| `dateIssued` | Sim | |

O [JSON Schema](schema/Alert.schema.json) deste repositório não exige `address` e considera obrigatórios `id`, `type`, `category`, `dateIssued` e `alertSource`.

## Modelo `DeviceOperation`

Cada registo de funcionamento tem uma entidade, com o identificador `urn:ngsi-ld:DeviceOperation:idicare:<registo>`, em que `<registo>` é o inteiro da origem. É uma entidade opcional: só se integra se os registos forem necessários na PGU.

### Propriedades

| Propriedade | Tipo | Descrição | Nota |
|-------------|------|-----------|------|
| id | URI | Identificador único da entidade | `urn:ngsi-ld:DeviceOperation:idicare:<registo>`. Segue a convenção de identificadores deste repositório. |
| type | String | Tipo de entidade | Valor constante igual a `DeviceOperation`. |
| refDevice | Relationship | Relógio a que se refere o registo | Entidade `Device`. Atributo adicional, que substitui o `device` do modelo. |
| startedAt | DateTime | Data e hora do registo | De acordo com a norma [ISO 8601-1:2019](https://www.iso.org/standard/70907.html), em UTC. A origem envia segundos Unix (por confirmar). |
| operationType | String | Tipo do registo | Texto livre tal como vem da origem, sem lista de valores. |

O modelo dos Smart Data Models tem ainda `device`, `result`, `status`, `operator`, `plannedStartAt`, `plannedEndAt`, `endedAt`, `reportedAt` e `addressedAt`, que a PGU não publica. Estão previstos no [JSON Schema](schema/DeviceOperation.schema.json), como opcionais.

## Diferenças em relação ao CNMD e aos modelos de referência

1. **Modelos dos Smart Data Models.** O CNMD só tem modelos equivalentes para o `Device` e o `Alert`; o `Patient`, a `Person`, o `DeviceMeasurement` e o `DeviceOperation` seguem os Smart Data Models, como previsto no Caderno de Encargos.
2. **`name` e `birthDate` do `Patient` cifrados.** Por serem dados pessoais cifrados pela PGU, o `name` deixa de ser a lista `HumanName` do FHIR e a `birthDate` deixa de ser uma data: ambos são texto. O mesmo se aplica ao `name` da `Person`.
3. **`DeviceMeasurement` com um atributo por sinal vital.** O modelo prevê uma medida genérica por entidade (`controlledProperty`, `numValue` e `unit`); a PGU publica uma entidade por relógio, com um atributo adicional por sinal vital e por dado de movimento, porque os Smart Data Models não têm atributos para sinais vitais. `refDevice` e `dateObserved` são os do modelo.
4. **`DeviceOperation` com `refDevice` e `operationType` em texto livre.** O modelo usa `device`, e `operationType` é uma lista fechada (`event`, `maintenance`, `fault`, `installation`, `upgrade`, `other`); a origem não tem lista de valores, pelo que os tipos de registo são publicados como vêm.
5. **`Device` sem `controlledProperty`, `value`, `deviceCategory` nem `address`**, obrigatórios no CNMD, porque a origem só dá a identidade, o estado, a bateria, o sinal e a posição do relógio. A `controlledProperty` está prevista no modelo de dados, mas a ingestão atual não a publica.
6. **`rssi` sem `unitCode`.** O modelo pede dBm ou mW; enquanto a escala da origem não estiver confirmada, é publicado sem unidade.
7. **O `DeviceModel` não é criado.** `refDeviceModel` refere um URN de uma entidade que não existe, porque a origem não dá dados do modelo.
8. **`dataProvider` e `source` não são publicados.** Todas as entidades vêm da mesma origem, e o município é dado pelo tenant.
9. **Atributos adicionais:** `refAccount`, `refPatient` e `locationSource` no `Device`; `refAccount` no `Patient`; `email` na `Person`; `heartRate`, `bloodPressureSystolic`, `bloodPressureDiastolic`, `bodyTemperature`, `spo2`, `steps`, `altitude`, `locationAccuracy` e `speed` no `DeviceMeasurement`; `refDevice` no `DeviceOperation`. Não existem nos modelos de referência e, conforme indicado no CNMD, podem ser ignorados na integração de dados provenientes de várias entidades.
10. **`location` não inclui a metainformação `coordinateSystem`** indicada nas notas do CNMD. As coordenadas são publicadas em GeoJSON (WGS84), de acordo com as convenções deste repositório.
11. **Identificadores.** Seguem a convenção deste repositório, `urn:ngsi-ld:<Tipo>:<identificador>`, em vez da estrutura `urn:<organização>:<tipo-do-recurso>:<identificador-local>` recomendada nas FAQ do CNMD. Não incluem o município.
12. **Alertas só para quedas.** O SOS e o relógio não colocado chegam como `DeviceOperation`, e não como `Alert`.

## Notas

Os identificadores das quedas e dos registos não incluem o IMEI: se o fornecedor os numerar por relógio e não globalmente, dois relógios podem partilhar o mesmo identificador (pergunta pendente ao fornecedor).

O contexto JSON-LD próprio da PGU, que define os atributos adicionais, está por definir.

## Exemplos e schemas

| Modelo | Exemplo | Schema |
|--------|---------|--------|
| `Device` | [exemplos/Device.json](exemplos/Device.json) | [schema/Device.schema.json](schema/Device.schema.json) |
| `Patient` | [exemplos/Patient.json](exemplos/Patient.json) | [schema/Patient.schema.json](schema/Patient.schema.json) |
| `Person` | [exemplos/Person.json](exemplos/Person.json) | [schema/Person.schema.json](schema/Person.schema.json) |
| `DeviceMeasurement` | [exemplos/DeviceMeasurement.json](exemplos/DeviceMeasurement.json) | [schema/DeviceMeasurement.schema.json](schema/DeviceMeasurement.schema.json) |
| `Alert` | [exemplos/Alert.json](exemplos/Alert.json) | [schema/Alert.schema.json](schema/Alert.schema.json) |
| `DeviceOperation` | [exemplos/DeviceOperation.json](exemplos/DeviceOperation.json) | [schema/DeviceOperation.schema.json](schema/DeviceOperation.schema.json) |

Os exemplos estão em formato NGSI-LD normalizado. Os schemas derivam dos schemas do CNMD e dos Smart Data Models, adaptados às diferenças acima, e validam a representação simplificada (*key-values*) das entidades.
