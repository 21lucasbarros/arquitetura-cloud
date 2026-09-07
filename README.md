# PZaaS API - Cadastro e Identidade

Documentação oficial do microsserviço de Cadastro/Identidade responsável pelo gerenciamento de usuários, validação de tokens e integração com banco de dados na arquitetura distribuída da pizzaria.

---

## 1. Visão Geral

O **Serviço 08** tem como principal objetivo fornecer a identidade e os perfis dos usuários do ecossistema PZaaS. Desenvolvido utilizando o **n8n** como motor de integração e microsserviços, ele se comunica com um banco de dados relacional na nuvem para autenticar tokens de acesso e retornar dados cadastrais essenciais de forma segura e padronizada.

## 2. Tecnologias Utilizadas

- **n8n**: Orquestração e execução dos endpoints e lógica de negócio.
- **PostgreSQL (Neon Tech)**: Banco de dados relacional para armazenamento dos perfis de usuários.

- **Postman**: Ferramenta utilizada para testes manuais das rotas.

---

## 3. Arquitetura e Endpoints

O microsserviço possui duas rotas principais rodando de forma isolada e paralela:

### `GET /cadastro/health` (Healthcheck)

- **Objetivo**: Verificar se o microsserviço está online e operando corretamente.

- **Segurança**: Requer os headers globais de controle.

- **Resposta de Sucesso**: `HTTP 200 OK` informando que está tudo certo.

### `POST /cadastro/perfil` (Consulta de Perfil)

- **Objetivo**: Receber um token de acesso, consultá-lo na base de dados e retornar as informações detalhadas do usuário.

- **Contrato de Entrada (Body JSON)**:

```json
{
  "token": "token123"
}
```

- **Resposta de Sucesso (`HTTP 200 OK`)**:

```json
{
  "nome": "Cliente Teste",
  "perfil": "cliente",
  "email": "teste@pizzaria.com"
}
```

---

## 4. Segurança e Contrato Global

Para garantir a padronização entre todos os microsserviços da arquitetura da pizzaria, este serviço valida rigorosamente os seguintes headers em todas as requisições:

- `x-api-key`: Chave de autorização global (`turma2026`).

- `x-pedido-id`: Identificador de rastreio obrigatório para o fluxo da requisição.

> **Tratamento de Falhas**: Caso a requisição chegue sem o cabeçalho obrigatório `x-pedido-id`, o fluxo intercepta o erro e retorna imediatamente um `HTTP 400 Bad Request` padronizado.

---

## 5. Observabilidade e Logs

Em conformidade com os padrões da arquitetura distribuída, o serviço implementa **observabilidade assíncrona**:

- Cada operação bem-sucedida ou falha de validação dispara automaticamente um log estruturado em segundo plano (`POST /logs`) para o microsserviço de Logger da turma.

- O serviço é identificado no ecossistema global pelo **ID 8** (Cadastro/Identidade).

---

## 6. Como Executar os Testes (Exemplos no Postman)

Para testar o microsserviço localmente, utilize os seguintes parâmetros:

1. **Testando a Saúde da API (`GET /cadastro/health`)**:

- **URL**: `http://localhost:5678/webhook/cadastro/health`
- **Headers**: `x-api-key: turma2026` e `x-pedido-id: 2332`

2. **Consultando o Perfil (`POST /cadastro/perfil`)**:

- **URL**: `http://localhost:5678/webhook/cadastro/perfil`
- **Headers**: `Content-Type: application/json`, `x-api-key: turma2026`, `x-pedido-id: 2332`

- **Body**: `{"token": "token123"}`
