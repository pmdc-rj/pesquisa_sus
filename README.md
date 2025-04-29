# API da PMDC - Documentação

<p align="center">
  <img src="https://duquedecaxias.rj.gov.br/images/dc-logo.png" width="400" alt="Prefeitura Municipal de Duque de Caxias">
</p>

## Índice

1. [Introdução](#introdução)
2. [Autenticação](#autenticação)
   - [Login](#login)
   - [Logout](#logout)
3. [API do SUS (CADSUS)](#api-do-sus-cadsus)
   - [Consultar CPF](#consultar-cpf)
4. [Observações Importantes](#observações-importantes)
5. [Solução de Problemas](#solução-de-problemas)

## Introdução

Esta documentação descreve os endpoints disponíveis na API da Prefeitura Municipal de Duque de Caxias (PMDC) para integração com o CADSUS (Cadastro Nacional de Usuários do SUS).

A API utiliza o Laravel como framework e implementa autenticação via tokens usando Laravel Sanctum para manter a segurança das operações.

## Autenticação

A API utiliza Laravel Sanctum para autenticação baseada em tokens. Você precisará obter um token antes de acessar rotas protegidas.

### Login

**Endpoint**: `POST /api/login`

**Descrição**: Autentica um usuário e retorna um token de acesso.

**Corpo da requisição**:
```json
{
  "email": "seu.email@exemplo.com",
  "password": "sua_senha"
}
```

**Resposta de sucesso (200 OK)**:
```json
{
  "status": "success",
  "message": "Login realizado com sucesso",
  "user": {
    "id": 1,
    "name": "Nome do Usuário",
    "email": "seu.email@exemplo.com",
    "created_at": "2025-04-29T16:45:22.000000Z",
    "updated_at": "2025-04-29T16:45:22.000000Z"
  },
  "token": "1|a1b2c3d4e5f6g7h8i9j0..."
}
```

**Resposta de erro (401 Unauthorized)**:
```json
{
  "message": "As credenciais fornecidas estão incorretas."
}
```

### Logout

**Endpoint**: `POST /api/logout`

**Descrição**: Revoga o token atual do usuário.

**Autenticação**: Bearer Token (obrigatório)

**Headers**:
```
Authorization: Bearer seu_token_aqui
Accept: application/json
```

**Resposta de sucesso (200 OK)**:
```json
{
  "status": "success",
  "message": "Logout realizado com sucesso"
}
```

**Resposta de erro (401 Unauthorized)**:
```json
{
  "message": "Unauthenticated."
}
```

## API do SUS (CADSUS)

### Consultar CPF

**Endpoint**: `POST /api/sus/consulta`

**Descrição**: Consulta informações de um cidadão no CADSUS através do CPF.

**Autenticação**: Bearer Token (obrigatório)

**Headers**:
```
Authorization: Bearer seu_token_aqui
Accept: application/json
Content-Type: application/json
```

**Corpo da requisição**:
```json
{
  "cpf": "12345678900"
}
```

**Resposta de sucesso (200 OK)**:
```json
{
  "status": "success",
  "dados": {
    "cpf": "12345678900",
    "cns": "123456789012345",
    "nome": "NOME COMPLETO DO CIDADÃO",
    "nascimento": {
      "data": "1990-01-01",
      "data_formatada": "01/01/1990"
    },
    "sexo": {
      "codigo": "M",
      "descricao": "Masculino"
    },
    "filiacao": {
      "mae": "NOME DA MÃE DO CIDADÃO",
      "pai": "NOME DO PAI DO CIDADÃO"
    },
    "contato": {
      "telefones": [
        {
          "tipo": "Celular",
          "ddi": "55",
          "ddd": "21",
          "numero": "999999999",
          "completo": "+55 (21) 999999999"
        }
      ],
      "emails": [
        {
          "email": "email@exemplo.com",
          "tipo": {
            "codigo": "P",
            "descricao": "Pessoal"
          }
        }
      ]
    },
    "localizacao": {
      "uf": "RJ",
      "municipio": {
        "codigo": "330170",
        "nome": "DUQUE DE CAXIAS"
      },
      "pais": {
        "codigo": "10",
        "nome": "BRASIL"
      }
    },
    "situacao": "REGULAR",
    "percentual_qualidade": "87.50"
  }
}
```

**Respostas de erro**:

- **400 Bad Request** (CPF não fornecido):
```json
{
  "status": "error",
  "message": "Por favor, informe um CPF para consultar."
}
```

- **404 Not Found** (CPF não encontrado):
```json
{
  "status": "error",
  "message": "CPF não encontrado ou não possui cadastro no CADSUS."
}
```

- **500 Internal Server Error** (Erro no serviço CADSUS):
```json
{
  "status": "error",
  "message": "Erro ao consultar o serviço CADSUS. O serviço pode estar temporariamente indisponível."
}
```

## Observações Importantes

1. **Registro de consultas**: 
   Cada consulta ao CADSUS é registrada no sistema com informações sobre:
   - Usuário que realizou a consulta
   - Data e hora da consulta
   - IP de origem
   - CPF pesquisado
   - Dados retornados
   - Status da consulta

2. **Autenticação**:
   - O token de acesso deve ser incluído em todas as requisições à API do SUS como um Bearer Token.
   - Tokens não têm um prazo de expiração definido por padrão, mas podem ser revogados pelo usuário ou pelo administrador.
   - Em caso de problemas de autenticação, solicite um novo token utilizando a rota de login.

3. **Limites de uso**:
   - Por razões de segurança e desempenho, podem ser aplicados limites ao número de requisições por usuário.
   - Consulte com o administrador da API para obter informações sobre limites específicos.

4. **Segurança**:
   - Mantenha seu token seguro.
   - Não compartilhe tokens entre aplicações.
   - Realize logout quando não estiver utilizando a API.

## Solução de Problemas

### Erros comuns e soluções

1. **"Unauthenticated"**:
   - Verifique se o token foi enviado corretamente como Bearer Token.
   - O token pode ter expirado ou sido revogado. Faça login novamente para obter um novo token.

2. **"Token not provided"**:
   - Certifique-se de que o token está sendo enviado no header Authorization.

3. **Erro 500 ao consultar o CADSUS**:
   - O serviço CADSUS pode estar temporariamente indisponível.
   - Verifique a conectividade com a internet.
   - Tente novamente mais tarde.

4. **"CPF não encontrado"**:
   - Confirme se o CPF foi digitado corretamente.
   - É possível que o cidadão não esteja cadastrado no CADSUS.

### Contato para suporte

Para obter suporte técnico ou relatar problemas com a API, entre em contato com:

- Email: rhylton.segov@duquedecaxias.rj.gov.br
- Telefone: (21) 98311-0504
