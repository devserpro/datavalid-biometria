# Datavalid - Pacote Biometria <span id="trialSpan"></span>

Solução de validação de informações que garante autenticidade e confiabilidade aos dados em tempo real.

O DataValid é disponibilizado pela plataforma APIGOV (Plataforma que contempla todas as API's disponibilizadas e comercializadas pelo SERPRO) e utiliza o protocolo Oauth2 - Client Credential Grant ([https://tools.ietf.org/html/rfc6749#section-4.4](https://tools.ietf.org/html/rfc6749#section-4.4)) para realizar a autenticação e autorização de acesso para consumo das API's contratadas, conforme figura abaixo:

<img title="Processo de autenticação e autorização APIS" src="https://raw.githubusercontent.com/devserpro/consulta-cpf/master/img/oauth.png" style="width=50%;" />

## Como fazer consultas ao Datavalid

Para consumir o Datavalid, você deverá utilizar os dois códigos (Consumer Key e Consumer Secret) disponibilizados na Área do Cliente. Esses códigos servem para identificar o contrato e deverão ser informados sempre que uma consulta for realizada.
Exemplos de códigos: 

**Consumer Key**: djaR21PGoYp1iyK2n2ACOH9REdUb

**Consumer Secret**: ObRsAJWOL4fv2Tp27D1vd8fB3Ote

### 1 – Como solicitar o Token de Acesso (Bearer)
Para consultar a Datavalid, é necessário obter um token de acesso temporário (Bearer). Esse token possui um tempo de validade e sempre que expirado, este passo de requisição de um novo token de acesso deve ser repetido. 

Para solicitar o token temporário é necessário realizar uma requisição HTTP POST para o endpoint Token https://apigateway.serpro.gov.br/token, informando as credenciais de acesso(consumerKey:consumerSecret) no HTTP Header Authorization, no formato base64, conforme exemplo abaixo. As credenciais de acesso devem ser obtidas a partir do portal do cliente Serpro - https://minhaconta.serpro.gov.br

```
[POST] grant_type=client_credentials
[HEAD] Authorization: Basic base64(Consumer Key:Consumer Secret)
```

Abaixo segue um exemplo de chamada via cUrl:

```curl
curl -k -d "grant_type=client_credentials" -H "Authorization: Basic ZGphUjIxUEdvWXAxaXlLMm4yQUNPSDlSRWRVYjpPYlJzQUpXT0w0ZnYyVHAyN0QxdmQ4ZkIzT3RlCg" https://apigateway.serpro.gov.br/token
```

A chave informada no exemplo acima "ZGphUjIxUEdvWXAxaXlLMm4yQUNPSD
lSRWRVYjpPYlJzQUpXT0w0ZnYyVHAyN0QxdmQ4ZkIzT3RlCg" é resultado do BASE64 dos códigos Consumer Key e Consumer Secret separados pelo caracter “:”, conforme exemplo a seguir:

```curl
echo "djaR21PGoYp1iyK2n2ACOH9REdUb:ObRsAJWOL4fv2Tp27D1vd8fB3Ote" | base64
```

**Receba o Token**

Como resultado, o endpoint informará o token de acesso a API, no campo access_token da mensagem json de retorno. Este token deve ser informado nos próximos passos.

```json
{"scope":"am_application_scope default","token_type":"Bearer","expires_in":3295,"access_token":"c66a7def1c96f7008a0c397dc588b6d7"}
```

**Renovação do Token de Acesso**

Atentar que sempre que o token de acesso temporário expirar, o gateway vai retornar um _HTTP CODE 401_ após realizar uma requisição para uma API. Neste caso, deve ser repetido o passo anterior (**Como solicitar o Token de Acesso (Bearer)**) para geração de um novo token de acesso temporário.


### 2 – Como realizar a consulta ao Datavalid

De posse do Token de Acesso, faça a requisição a um dos serviços do Datavalid. Exemplo:

```curlBearer
curl -X POST "https://apigateway.serpro.gov.br/datavalid/biometria/vbeta1/validate/pf-face" -H  "accept: application/json" -H  "Authorization: Bearer b42daf721648cb626ba3774bde927e48" -H  "content-type: application/json" -d "{  \"key\": {    \"cpf\": \"05137518743\"  },  \"answer\": {    \"nome\": \"João\",    \"sexo\": \"F\",    \"data_nascimento\": \"2000-10-10\",    \"situacao_cpf\": \"regular\",    \"filiacao\": {      \"nome_mae\": \"Mãe do João\",      \"nome_pai\": \"Pai do João\"    },    \"nacionalidade\": 1,    \"endereco\": {      \"logradouro\": \"Nome do Logradouro\",      \"numero\": \"0007\",      \"complemento\": \"APTO 2015\",      \"bairro\": \"Nome do Bairro\",      \"cep\": \"0000001\",      \"municipio\": \"Nome do Municipio\",      \"uf\": \"DF\"    },    \"documento\": {      \"tipo\": 1,      \"numero\": \"000001\",      \"orgao_expedidor\": \"SSP\",      \"uf_expedidor\": \"MG\"    },    \"cnh\": {      \"numero_registro\": \"0000001\",      \"categoria\": \"AB\",      \"data_primeira_habilitacao\": \"2000-10-10\",      \"data_validade\": \"2000-10-10\"    },    \"biometria_face\": \"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\"  }}"
```

No exemplo acima foram utilizados os seguintes parametros:

**[HEADER] Accept: application/json** - Informamos o tipo de dados que estamos requerendo, nesse caso JSON

**[HEADER] Authorization: Bearer <span class="bearer">c66a7de41c96f7008a0c397dc588b6d7</span>** - Informamos o token de acesso recebido

**[POST] https://apigateway.serpro.gov.br/datavalid/biometria/vbeta1/validate/pf-face**: chamamos a url do serviço de validação de PF com biometria de face do Datavalid passando como argumento -d, o corpo da requisiço REST."

Exemplo de resposta para validação de dados de PF (Pessoa Física) com biometria de face

```json
{
  "nome": true,
  "nome_similaridade": 1,
  "sexo": true,
  "data_nascimento": false,
  "informacoes_adicionais": {
    "cpf_regular": true
  }
}
```

Exemplo de resposta para validação de dados de PJ (Pessoa Jurídica):

```json
{
    "razao_social": false,
    "razao_social_similaridade": 0.96,
    "nome_fantasia": true,
    "nome_fantasia_similaridade": 1,
    "data_abertura": true,
    "cnae_principal": {
        "codigo": true,
        "descricao": true,
        "descricao_similaridade": 1
    },
    "natureza_juridica": {
        "codigo": true,
        "descricao": true,
        "descricao_similaridade": 1
    },
    "endereco": {
        "logradouro": false,
        "logradouro_similaridade": 0.9130434782608696,
        "numero": true,
        "numero_similaridade": 1,
        "complemento": false,
        "complemento_similaridade": 0,
        "bairro": true,
        "bairro_similaridade": 1,
        "cep": true,
        "municipio": true,
        "municipio_similaridade": 1,
        "uf": true,
        "uf_similaridade": 1
    },
    "situacao_cadastral": {
        "codigo": true,
        "data": true,
        "motivo": false,
        "motivo_similaridade": 0
    },
    "situacao_especial": true,
    "situacao_especial_similaridade": 1,
    "nome_orgao": true,
    "nome_orgao_similaridade": 1,
    "ente_federativo": true,
    "ente_federativo_similaridade": 1,
    "correio_eletronico": true,
    "correio_eletronico_similaridade": 1,
    "capital_social": true,
    "porte": true,
    "telefone": {
        "ddd": true,
        "numero": true
    }
}
```
Abaixo disponibilizamos body's de requisições que podem ser utilizados na demonstração dos metodos do Datavalid. Esses body's possuem dados fictícios e podem ser usados para simular cenários nos quais o Datavalid valida positivamente todos os dados enviados.


**Validação de PF com Biometria Facial (validate/pf-face) :**
```json

{  
   "key":{  
      "cpf":"81845995104"
   },
   "answer":{  
      "nome":"ADEMAR VEGA XIMENES",
      "sexo":"M",
      "data_nascimento":"1994-06-23",
      "situacao_cpf":"cAncelada por obito sem EspOlio",
      "filiacao":{  
         "nome_mae":"MARIA VEGA XIMENES",
         "nome_pai":"JOÃO VEGA XIMENES"
      },
      "endereco":{  
         "logradouro":"Travessa Serrano",
         "numero":"9754",
         "complemento":"",
         "cep":"12983406",
         "bairro":"CENTRO",
         "municipio":"Nova Iguaçu",
         "uf":"AC"
      },
      "nacionalidade":0,
      "documento":{  
         "tipo":1,
         "numero":"6694845",
         "orgao_expedidor":"DIC",
         "uf_expedidor":"MA"
      },
      "cnh":{  
         "numero_registro":"98668270420",
         "categoria":"A",
         "data_primeira_habilitacao":"1980-11-28",
         "data_validade":"2018-09-02"
      },
      "biometria_face":"/9j/4TyoRXhpZgAASUkqAAgAAAAOACWIBAABAAAALhcAABABAgAGAAAAtgAAAA4BAgAFAAAAvAAAABIBAwABAAAACAAAADIBAgAUAAAAwQAAAAABBAABAAAAAAoAABMCAwABAAAAAQAAACgBAwABAAAAAgAAAAEBBAABAAAAgAcAABsBBQABAAAA1QAAADEBAgAIAAAA3QAAAGmHBAABAAAA8gAAABoBBQABAAAA5QAAAA8BAgAFAAAA7QAAAHsXAABaMDBBRABKcGVnADIwMTc6MDY6MjggMTY6NDE6MDEASAAAAAEAAABBbmRyb2lkAEgAAAABAAAAQVNVUwAjAAGgAwABAAAAAQAAAASQAgAUAAAAnAIAAJ2CBQABAAAAsAIAAAqSBQABAAAAuAIAAAKSBQABAAAAwAIAAAiSAwABAAAAAAAAAAKkAwABAAAAAAAAAJKSAgAEAAAAMzA0AAOgBAABAAAAgAcAAAWSBQABAAAAyAIAAAakAwABAAAAAAAAAJGSAgAEAAAAMzA0AASkBQABAAAA0AIAAAmkAwABAAAAAAAAACKIAwABAAAAAgAAAAqkAwABAAAAAAAAAAOk"
   }
}

```

**Validação de PJ (validate/pj) :**
```json
{
   "key":{
      "cnpj":"34238864000168"
   },
   "answer":{
      "razao_social":"SERVICO DE E-COMERCE LTDA",
      "nome_fantasia":"E-COMERCE",
      "data_abertura":"1967-06-30",
      "cnae_principal":{
         "codigo":"6204000",
         "descricao":"Consultoria em e-comerce"
      },
      "natureza_juridica":{
         "codigo":"2011",
         "descricao":"Empresa Privada"
      },
      "endereco":{
         "logradouro":"ST DE GRANDE AREA NORTE",
         "numero":"Q.601",
         "complemento":"LOTE V",
         "cep":"70836900",
         "bairro":"ASA NORTE",
         "municipio":"BRASILIA",
         "uf":"DF"
      },
      "situacao_especial":"",
      "situacao_cadastral":{
         "codigo":2,
         "data":"2004-05-22",
         "motivo":""
      },
      "nome_orgao":"BRASILIA",
      "ente_federativo":"BR",
      "correio_eletronico":"",
      "capital_social":0,
      "porte":"05",
      "telefone":{
         "ddd":"61",
         "numero":"4338456"
      }
   }
}
```
