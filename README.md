# Mesago - API SMS

Bem vindo a API SMS da Mesago. Com ela você pode enviar SMS para todas as operadoras 
e receber status de entrega e respostas dos usuários de forma automática.

Para a versão em Inglês desse documento, por favor, visite [Mesago SMS API](http://core.mesago.co/docs)!

Nossa API suporta nativamente dois formatos:
- REST
    - O sistema deve realizar uma chamada para a URL https://core.mesago.co/api?version=1.0, 
    enviando os parâmetros por meio de GET ou POST. O retorno para cada chamada será um 
    JSON cujo formato é detalhado ao longo desta documentação.
- SOAP/XML
    - A requisição é feita por uma chamada SOAP/XML ao endereço https://core.mesago.co/ws/sms-service, 
    obtendo como retorno um XML. O descritor WSDL pode ser obtido [neste link](https://core.mesago.co/ws/sms-service?wsdl).



# Autenticação

Todas as requisições a API SMS da Mesago devem utilizar cabeçalhos **Basic Authorization**.
Neste campo deve ser informado sua conta e senha de integração que enviamos por e-mail. 
Caso haja erro na autenticação, o respectivo código de erro será retornado.

    Authorization: Basic Y29udGE6c2VuaGE=
    
O valor após a palavra `Basic` é uma chave Base64 da sua conta e senha. 
Para obter o valor, utilize o o comando base64 do linux:

    $ echo -n conta:senha | base64 
    Y29udGE6c2VuaGE=
 
O site [base64Encode](https://www.base64encode.org) também faz essa codificação gratuitamente.

# Parâmetro Id

Ao enviar um SMS, recomendamos que você utilize o parâmetro `correlationId`. 
Ele serve como um identificador único de sua mensagem em nossa plataforma e 
pode ser utilizado para consulta de status ou mesmo para proteção contra 
envios duplicados. 

# Tabela de Status

As chamadas à API irão retornar dois tipos de código. Um `statusCode` e `detailCode`
A seguir, veja a lista de status suportados pela nossa API:

### statusCode

```
    00 - Ok
    01 - Scheduled
    02 - Sent
    03 - Delivered
    04 - Not Received
    05 - Blocked - No Coverage
    06 - Blocked - Black listed
    07 - Blocked - Invalid Number
    08 - Blocked - Content not allowed
    08 - Blocked - Message Expired
    09 - Blocked
    10 - Error
```

### detailCode
```
    000 - Message Sent
    002 - Message successfully canceled
    010 - Empty message content
    011 - Message body invalid
    012 - Message content overflow
    013 - Incorrect or incomplete ‘destination’ mobile number
    014 - Empty ‘destination’ mobile number
    015 - Scheduling date invalid or incorrect
    016 - ID overflow
    017 - Parameter ‘url’ is invalid or incorrect
    021 - ‘id’ fieldismandatory
    080 - Message with same ID already sent
    100 - Message Queued
    110 - Message sent to operator
    111 - Message confirmation unavailable
    120 - Message received by mobile
    130 - Message blocked
    131 - Message blocked by predictive cleansing
    132 - Message already canceled
    133 - Message content in analysis
    134 - Message blocked by forbidden content
    135 - CorrelationId is Invalid or Inactive
    136 - Message expired
    140 - Mobile number not covered
    141 - International sending not allowed
    145 - Inactive mobile number
    150 - Message expired in operator
    160 - Operator network error
    161 - Message rejected by operator
    162 - Message cancelled or blocked by operator
    170 - Bad message
    171 - Bad number
    172 - Missing parameter
    173 - Gateway don't defined to user
    180 - Message ID notfound
    190 - Unknown error
    200 - Messages Sent
    210 - Messages scheduled but Account Limit Reached
    240 - File empty or not sent
    241 - File too large
    242 - File readerror
    300 - Received messages found
    301 - No received messages found
    400 - Entity saved
    900 - Authentication error
    901 - Account type not support this operation.
    990 - Account Limit Reached – Please contact support
    998 - Wrong operation requested
    999 - Unknown Error
```



# Group Serviços da API

## Envio de um único SMS [/api/send-sms]

### Testar envio de um único SMS [POST]

Este serviço envia um SMS para o celular do destinatário. Com ele é possível enviar mensagens de texto curtas e longas.

### Mensagens curtas (até 160 caracteres)

Veja abaixo os exemplos da chamada de mensagens curtas:

**Exemplo #1**:
```json
{
    "sendSmsRequest": {
        "destination":"5519900001111",
        "messageText":"Hello, I am a simple message"
        }
}
```
**Exemplo #2**:
```json
{
    "sendSmsRequest": {
        "destination":"5519900001111",
        "messageText":"Hello, I am a message",
        "correlationId":"myId",
        "extraInfo":"{\"name\": \"Bob\"}",
        "timeWindow":[
            11, 12, 18, 19, 20, 21
        ]
        }
}
```
**Exemplo #3**:
```json
{
    "sendSmsRequest": {
          "destination":"5519900001111",
          "messageText":"Hello, I am a message",
          "correlationId":"myId",
          "extraInfo":"{\"name\": \"Bob\"}",
          "timeWindow":[
            11, 12, 18, 19, 20, 21
          ],
          "expiresDate":"2016-06-10T21:00:00",
          "scheduledDate":"2016-06-08T10:00:00",
          "timeZone":"America/Sao_Paulo"
        }
}
```
**Exemplo #4**:
```json
{
    "sendSmsRequest": {
          "destination":"5519900001111",
          "messageText":"Hello, I am a message",
          "correlationId":"myId",
          "flashSMS":"true"
        }
}
```

Em resposta à chamada, a API da Mesago retornará um status de controle:

```json
    {
        "sendSmsResponse" : {
            "statusCode" : "00",
            "statusDescription" : "Ok",
            "detailCode" : "000",
            "detailDescription" : "Message Sent"
        }
    }
```

Outro exemplo de retorno síncrono da chamada:

```json
    {
        "sendSmsResponse" : {
            "statusCode" : "05",
            "statusDescription" : "Blocked",
            "detailCode" : "140",
            "detailDescription" : "Mobile number not covered"
        }
    }
```

### Mensagens longas (mais de 160 caracteres)

Para envio de mensagens longas é necessário entrar em contato com nossa equipe de atendimento e solicitar que sua conta seja habilitada para esse tipo de envio.

Cada parte da mensagem contém até 152 caracteres, pois há uma reserva de 8 caracteres para que seja feita a identificação das partes que compõem a mensagem. Essa identificação é necessária para que os aparelhos interepretem como uma única mensagem. Além disso, como o particionamento das mensagens será realizada no último caractere de espaço, ou seja, palavras não serão truncadas, a mensagem pode ser particionada com menos de 152 caracteres.

Para esse tipo de envio a requisição será idêntica a requisição de envio de mensagens curtas. Basta enviar o texto da mensagem longa no atributo ‘msg’ com mais de 160 caracteres. Nossa plataforma tratará o texto do atributo `msg` como uma mensagem longa. 

Há limitação de 1520 caracteres somando-se os caracteres dos atributos `from` e `msg`.

Veja abaixo o exemplo da chamada de mensagens longas:

```json
    {
        "sendSmsRequest": {
            "destination":"5519900001111",
            "messageText":"Lorem ipsum dolor sit amet, consectetur adipiscing elit. Ut blandit neque consectetur, faucibus tortor varius, posuere enim. Cras tincidunt lectus eget pulvinar consectetur. Suspendisse in nibh elit. Ut non pharetra risus, nec aliquet lectus. Sed convallis mauris vitae lectus consequat, non mollis nibh hendrerit. Ut tristique commodo ligula eu bibendum. Quisque vestibulum quis nisi id ultrices.",
            "correlationId":"myId",
            "callbackOption": "NONE",
            "extraInfo":"{\"name\": \"Bob\"}",
            "timeWindow":[
                11, 12, 18, 19, 20, 21
            ]
        }
    }
```

Em resposta à chamada, a API da Mesago retornará um status de controle. O objeto `parts` foi introduzido para representar as partes da mensagem longa e possui os atributos `partId` e `order` que representam o identificador e a ordem de cada parte, respectivamente:

```json
    {
        "sendSmsResponse" : {
            "statusCode": "00",
            "statusDescription": "Ok",
            "detailCode": "000",
            "detailDescription": "Message Sent",
            "parts": [
                {
                    "partId": "idteste_001",
                    "order": 1
                },
                {
                    "partId": "idteste_002",
                    "order": 2
                },
                {
                    "partId": "idteste_003",
                    "order": 3
                }
            ]
        }
    }
```


*Para receber retornos assíncronos, consulte o menu Callbacks da API.*

+ Attributes
    + destination: 555199887766 (number, required) - Número do celular do destinatário incluíndo o DDI (55).
    + messageText: Sua consulta está confirmada! (string, required) - Texto da mensagem a ser enviada.
    + scheduledDate: `2014-07-18T02:01:23` (string) - Data e hora em que a mensagem deve ser enviada no formato ISO 8691
    + correlationId: SEU_ID_001 (string) - Identificador da mensagem no sistema do cliente.
    + callbackOption: ALL, FINAL, NONE (enum) - Tipo de status de entrega. ALL: Envia status intermediários e final da mensagem. FINAL: Envia o status final de entrega da mensagem (recomendado). NONE: Não será feito callback do status de entrega.
    + extraInfo: 1231 (number) - Se sua conta utiliza o recurso de agregador de mensagens (centro de custo, campanha), este parametro deve informar o código do agregador desejado.

+ request (application/json)
    + Header
    
            Authorization: Basic YWRtaW46YWRtaW4xMjM=
            Accept: application/json
    
    + Body
    
            {
                "sendSmsRequest": {
                    "destination":"5519900001111",
                    "messageText":"Hello, I am a message",
                    "correlationId":"myId",
                    "extraInfo":"{\"name\": \"Bob\"}",
                    "timeWindow":[
                        11, 12, 18, 19, 20, 21
                    ],
                    "expiresDate":"2016-06-10T21:00:00",
                    "scheduledDate":"2016-06-08T10:00:00",
                    "timeZone":"America/Sao_Paulo"
                }
            }

+ Response 201 (application/json)

        {
            "sendSmsResponse" : {
                "statusCode" : "00",
                "statusDescription" : "Ok",
                "detailCode" : "000",
                "detailDescription" : "Message Sent"
            }
        }

+ Response 500 (application/json)

        {
            "sendSmsResponse" : {
                "statusCode" : "99",
                "statusDescription" : "Nok",
                "detailCode" : "999",
                "detailDescription" : "Invalid User"
            }
        }

## Envio de vários SMSs simultaneamente [/api/send-sms-multiple]

### Testar Envio de vários SMSs simultaneamente [POST]

### Mensagens curtas (até 160 caracteres)

Este método recebe uma lista de objetos `sendSmsRequest`, é possível configurar uma mensagem Default para todos os destinatários no campo `defaultValues`.


**Exemplo** #1:
``` json
{
        "sendSmsMultiRequest":{
              "messages":[
                {
                  "destination":"5519900001111",
                  "messageText":"First message"
                },
                {
                  "destination":"5519900002222"
                },
                {
                  "destination":"5519900003333"
                }
              ],
              "defaultValues":{
                "messageText":"Default message"
              }
        }
    }
```
**Exemplo** #2:
```json
    {
        "sendSmsMultiRequest":{
              "messages":[
                {
                  "destination":"5519900001111",
                  "messageText":"First message"
                },
                {
                  "destination":"5519900002222"
                }
              ],
              "timeZone":"America/Sao_Paulo",
              "scheduleDate": "2017-01-28T02:30:43",
              "timeWindow": [12, 15, 20],
              "defaultValues":{
                "messageText":"Default message"
              }
        }
    }
```
**Exemplo** #3:
```json
    {
        "sendSmsMultiRequest":{

              "messages":[
                {
                  "destination":"5519900001111"
                },
                {
                  "destination":"5519900002222"
                }
              ],
              "defaultValues":{
                "messageText":"Default message",
                "flashSMS":"true"
              }
        }
    }
```

### Mensagens longas (mais de 160 caracteres)

Para envio de mensagens longas é necessário entrar em contato com nossa equipe de atendimento e solicitar que sua conta seja habilitada para esse tipo de envio.

Cada parte da mensagem contém até 152 caracteres, pois há uma reserva de 8 caracteres para que seja feita a identificação das partes que compõem a mensagem. Essa identificação é necessária para que os aparelhos interepretem como uma única mensagem. Além disso, como o particionamento das mensagens será realizada no último caractere de espaço, ou seja, palavras não serão truncadas, a mensagem pode ser particionada com menos de 152 caracteres.

Para esse tipo de envio a requisição será idêntica a requisição de envio de mensagens curtas múltiplas.

```json
    {
        "sendSmsMultiRequest":{

              "messages":[
                {
                  "destination":"5519900001111",
                },
                {
                  "destination":"5519900002222"
                }
              ],
              "defaultValues":{
                "messageText":"Lorem ipsum dolor sit amet, consectetur adipiscing elit. Ut blandit neque consectetur, faucibus tortor varius, posuere enim. Cras tincidunt lectus eget pulvinar consectetur. Suspendisse in nibh elit. Ut non pharetra risus, nec aliquet lectus. Sed convallis mauris vitae lectus consequat, non mollis nibh hendrerit. Ut tristique commodo ligula eu bibendum. Quisque vestibulum quis nisi id ultrices.",
                "flashSMS":"true"
              }
        }
    }
```

Em resposta à chamada, a API da Mesago retornará um status de controle. O objeto `parts` foi introduzido para representar as partes da mensagem longa e possui os atributos `partId` e `order` que representam o identificador e a ordem de cada parte, respectivamente:

```json
    {
        "sendSmsMultiResponse": {
            "sendSmsResponseList": [
                {
                    "statusCode": "00",
                    "statusDescription": "Ok",
                    "detailCode": "000",
                    "detailDescription": "Message Sent",
                    "parts": [
                        {
                            "partId": "idteste1_001",
                            "order": 1
                        },
                        {
                            "partId": "idteste1_002",
                            "order": 2
                        },
                        {
                            "partId": "idteste1_003",
                            "order": 3
                        }
                    ]
                },
                {
                    "statusCode": "00",
                    "statusDescription": "Ok",
                    "detailCode": "000",
                    "detailDescription": "Message Sent",
                    "parts": [
                        {
                            "partId": "idteste2",
                            "order": 1
                        }
                    ]
                }
            ]
        }
    }
```

+ Attributes
    + destination: 555199887766 (number, required) - Número do celular do destinatário incluíndo o DDI (55).
    + messageText: Sua consulta está confirmada! (string, required) - Texto da mensagem a ser enviada.
    + scheduledDate: `2014-07-18T02:01:23` (string) - Data e hora em que a mensagem deve ser enviada no formato ISO 8691
    + correlationId: SEU_ID_001 (string) - Identificador da mensagem no sistema do cliente.
    + callbackOption: ALL, FINAL, NONE (enum) - Tipo de status de entrega. ALL: Envia status intermediários e final da mensagem. FINAL: Envia o status final de entrega da mensagem (recomendado). NONE: Não será feito callback do status de entrega.
    + extraInfo: 1231 (number) - Se sua conta utiliza o recurso de agregador de mensagens (centro de custo, campanha), este parametro deve informar o código do agregador desejado.

+ request (application/json)
    + Header
    
            Authorization: Basic YWRtaW46YWRtaW4=
            Accept: application/json
    
    + Body
    
            {
                "sendSmsMultiRequest":{
        
                      "messages":[
                        {
                          "destination":"5519900001111",
                          "correlationId":"0001"
                        },
                        {
                          "destination":"5519900002222",
                          "correlationId":"0002"
                        }
                      ],
                      "defaultValues":{
                        "messageText":"Default message",
                        "flashSMS":"true"
                      }
                }
            }
        

+ Response 201 (application/json)

            {
              "sendSmsMultiResponse": {
                "sendSmsResponseList": [
                  {
                    "statusCode": "00",
                    "statusDescription": "Ok",
                    "detailCode": "000",
                    "detailDescription": "Message Sent"
                  },
                  {
                    "statusCode": "00",
                    "statusDescription": "Ok",
                    "detailCode": "000",
                    "detailDescription": "Message Sent"
                  }
                ]
              }
            }

## Consulta Status de um SMS [/api/get-sms-status/id/{id}]

### Testar Consulta de Status de um SMS [GET]

Consulta o status de entrega de uma mensagem previamente enviada usando seu identificador `id` como referência.

**Importante:** a consulta a um SMS fica disponível por até 24 horas após seu envio.

Exemplo de retorno:

```json
    {
      "getSmsStatusResp": {
        "id": "006",
        "received": "2014-08-23T02:01:23",
        "shortcode": 69788,
        "mobileOperatorName": "claro",
        "statusCode": "03",
        "statusDescription": "Delivered",
        "detailCode": "120",
        "detailDescription": "Message received by mobile"
      }
    }
```

+ parameters
    + id: 004 (string, required) - Identificador da mensagem informado no momento do envio


+ request
    + Header
    
            Authorization: Basic YWRtaW46YWRtaW4=

+ response 200 (application/json)

        {
          "getSmsStatusResp": {
            "id": "006",
            "received": "2014-08-23T02:01:23",
            "shortcode": 69788,
            "mobileOperatorName": "claro",
            "statusCode": "03",
            "statusDescription": "Delivered",
            "detailCode": "120",
            "detailDescription": "Message received by mobile"
          }
        }


            
### Listar Novos SMS recebidos [/api/received/list]

## Testar Listar Novos SMS recebidos [POST]

Retorna a lista de novos SMSs recebidos. Uma vez cosultado, o SMS 
não irá mais ser retornado na chamada deste serviço.

Exemplo de retorno:

```json
    {
      "receivedResponse": {
        "statusCode": "00",
        "statusDescription": "Ok",
        "detailCode": "300",
        "detailDescription": "Received messages found",
        "receivedMessages": [
          {
            "id": 23190501,
            "dateReceived": "2014-08-22T14:49:36",
            "mobile": "5511991070316",
            "shortcode": "30133",
            "mobileOperatorName": "Claro",
            "correlationId": "hs863223748"
          }
        ]
      }
    }
```

+ request (application/json)
    + Header
    
            Authorization: Basic YWRtaW46YWRtaW4=

+ response 200 (application/json)

        {
          "receivedResponse": {
            "statusCode": "00",
            "statusDescription": "Ok",
            "detailCode": "300",
            "detailDescription": "Received messages found",
            "receivedMessages": [
              {
                "id": 23190501,
                "dateReceived": "2014-08-22T14:49:36",
                "mobile": "5511991070316",
                "shortcode": "30133",
                "mobileOperatorName": "Claro",
                "correlationId": "hs863223748"
              }
            ]
          }
        }

## Consultar SMS recebidos por Período [/api/received/search/{startDate}/{endDate}]

### Testar Consultar SMS recebidos por Período [GET]

Retorna a lista de SMSs recebidos em um período definido.

```json
    {
      "receivedResponse": {
        "statusCode": "00",
        "statusDescription": "Ok",
        "detailCode": "300",
        "detailDescription": "Received messages found",
        "receivedMessages": [
          {
            "id": 23190501,
            "dateReceived": "2014-08-22T14:49:36",
            "mobile": "5511991070316",
            "shortcode": "30133",
            "mobileOperatorName": "Claro",
            "correlationId": "hs863223748"
          }
        ]
      }
    }
```

**Atenção**: Para receber o parâmetro `mobileOperatorName` você deverá solicitar ao nosso atendimento que seja habilitado em sua conta.

+ parameters
    + startDate: `2014-08-22T00:00:00` (string, required) - Data de início da busca
    + endDate: `2014-08-22T23:59:59` (string, required) - Data de fim da busca

+ request (application/json)
    + Header
    
            Authorization: Basic YWRtaW46YWRtaW4=

+ response 200 (application/json)

        {
          "receivedResponse": {
            "statusCode": "00",
            "statusDescription": "Ok",
            "detailCode": "300",
            "detailDescription": "Received messages found",
            "receivedMessages": [
              {
                "id": 23190501,
                "dateReceived": "2014-08-22T14:49:36",
                "mobile": "5511991070316",
                "shortcode": "30133",
                "mobileOperatorName": "Claro",
                "correlationId": "hs863223748"
              }
            ]
          }
        }

## Cancelamento de SMS agendado [/api/cancel-sms/id/{id}]

### Testar Cancelamento de SMS agendado [POST]

Cancela o envio de um SMS previamente agendado.
Para realizar o cancelamento, é necessário que tenha sido utilizado o identificador `id` no momento do envio.

**Importante:** o cancelamento de um SMS só pode ser feito até sua data de agendamento. Após, o SMS terá sido enviado à operadora e não poderá ser cancelado.

Exemplo de Retorno:

```json
    {
        "cancelSmsResp" : {
            "statusCode" : "09",
            "statusDescription" : "Blocked",
            "detailCode” : "002",
            "detailDescription" : "Message successfully canceled"
        }
    }
```

+ parameters
    + id: 004 (string, required) - Identificador da mensagem informado no momento do envio

+ request
    + Header
    
            Authorization: Basic YWRtaW46YWRtaW4=

+ response 200 (application/json)

        {
            "cancelSmsResp" : {
                "statusCode" : "09",
                "statusDescription" : "Blocked",
                "detailCode” : "002",
                "detailDescription" : "Message successfully canceled"
            }
        }

# Group Callbacks da API

A API de SMS da Mesago pode realizar callbacks diretamente para o seu sistema, 
enviando status de entrega das mensagens enviadas ou informações sobre SMS 
recebidos de seus clientes/contatos.

## Callback de Status de Entrega

A plataforma Mesago envia ao sistema do cliente o status dos SMS enviados. 
É possível enviar status intermediários ou finais.

Para receber o status, é necessário configurar uma URL de notificação na 
plataforma Mesago.

Exemplo do que você receberá em seu sistema:

```json
    {
      "callbackMtRequest": {
        "status": "03",
        "statusMessage": "Delivered",
        "statusDetail": "120",
        "statusDetailMessage": "Message received by mobile",
        "id": "hs765939216",
        "received": "2014-08-26T12:55:48.593-03:00",
        "mobileOperatorName": "Claro"
      }
    }
```
    
## Callback de SMS Recebido

A plataforma Mesago envia ao sistema do cliente um SMS recebido de um celular 
(esta funcionalidade é utilizada, por exemplo, para enquetes, SAC, pesquisas, 
confirmações de visita etc).

Para receber um SMS em seu sistema, é necessário configurar uma URL de notificação na 
plataforma Mesago.

Exemplo do que você receberá em seu sistema:

```json
    {
      "callbackMoRequest": {
        "id": "20690090",
        "mobile": "555191951711",
        "shortCode": "40001",
        "account": "Mesago.envio ",
        "received": "2014-08-26T12:27:08.488-03:00",
        "campaignId": "hs765939061"
      }
    }
```
    
# Group Bibliotecas

A fim de facilitar a integração com nossa plataforma, disponibilizamos, a seguir, bibliotecas nas seguintes linguagens:

[JAVA](http://core.mesago.co/desenvolvedores/bibliotecas/java-rest-api.zip)

[PHP](http://core.mesago.co/desenvolvedores/bibliotecas/php-rest-api.zip)

[JAVASCRIPT](http://core.mesago.co/desenvolvedores/bibliotecas/html-js-rest-api.rar)


    
# Group Mesago.co

Somos líderes no mercado de serviços móveis no Brasil.
Há mais de 12 dias, entregamos soluções para ampliar resultados e 
gerar cada vez mais interatividade entre empresas e milhões de 
usuários no país.

Visite [nosso site](http://core.mesago.co)!

# Group Versão em Inglês

Para a versão em Inglês desse documento, por favor, visite [Mesago SMS API](http://docs.mesago.apiary.io)!
