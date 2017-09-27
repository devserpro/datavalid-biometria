# Datavalid - Pacote Biometria <span id="trialSpan"></span>

Solução de validação de informações que garante autenticidade e confiabilidade aos dados em tempo real.

O DataValid é disponibilizado pela plataforma APIGOV (Plataforma que contempla todas as API's disponibilizadas e comercializadas pelo SERPRO) e utiliza o protocolo Oauth2 - Client Credential Grant ([https://tools.ietf.org/html/rfc6749#section-4.4](https://tools.ietf.org/html/rfc6749#section-4.4)) para realizar a autenticação e autorização de acesso para consumo das API's contratadas, conforme figura abaixo:

<img title="Processo de autenticação e autorização APIS" src="https://raw.githubusercontent.com/devserpro/consulta-cpf/master/img/oauth.png" style="width=50%;" 

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

De posse do Token de Acesso, faça uma requisição via POST ao gateway informando os parâmetros do Datavalid. Exemplo:

```curlBearer
curl -X POST --header "Accept: application/json" --header "Authorization: Bearer c66a7de41c96f7008a0c397dc588b6d7" -d "{
  \"key\": \"string\",
  \"answer\": {
    \"nome\": \"string\",
    \"sexo\": \"F\",
    \"data_nascimento\": \"string\",
    \"filiacao\": {
      \"nome_mae\": \"string\",
      \"nome_pai\": \"string\"
    },
    \"nacionalidade\": 1,
    \"documento\": {
      \"tipo\": 1,
      \"numero\": \"string\",
      \"orgao_expedidor\": \"string\",
      \"uf_expedidor\": \"string\"
    },
    \"cnh\": {
      \"numero_registro\": \"string\",
      \"categoria\": \"string\",
      \"data_primeira_habilitacao\": \"string\",
      \"data_validade\": \"string\"
    }
  }
}" "https://apigateway.serpro.gov.br/datavalid/vbeta1/"
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
Nos links abaixo disponibilizamos exemplos de requisição aos metodos do Datavalid, com todos as informaçes válidos:
