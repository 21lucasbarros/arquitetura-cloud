# PZaaS API - Cadastro e Identidade

Documentação oficial do microsserviço de Cadastro/Identidade (Serviço 08) responsável pelo gerenciamento de usuários, autenticação, validação de tokens e integração com banco de dados na arquitetura distribuída da pizzaria.

---

## 1. Visão Geral

O **Serviço 08** fornece a identidade e os perfis dos usuários do ecossistema PZaaS. Desenvolvido no **n8n** (servidor compartilhado da turma) e integrado ao **PostgreSQL (Neon Tech)**, ele autentica credenciais, cria novas contas e retorna dados cadastrais utilizando tokens UUID gerados pelo banco.

## 2. Tecnologias Utilizadas

- **n8n**: Orquestração e execução dos endpoints e lógica de negócio.
- **PostgreSQL (Neon Tech)**: Banco de dados relacional para armazenamento dos perfis (tabela `usuarios_pzaas`).
- **Postman**: Ferramenta utilizada para testes das rotas.

---

## 3. Arquitetura e Endpoints

O microsserviço possui quatro rotas principais rodando de forma isolada (`cadastro_b`) no servidor de produção.

### `GET /cadastro_b/health` (Healthcheck)

- **Objetivo**: Verificar se o microsserviço está online.
- **Resposta de Sucesso (`HTTP 200 OK`)**: Retorna `{"status": "tudo certo"}`.

### `POST /cadastro_b/cadastro` (Criação de Usuário)

- **Objetivo**: Registrar um novo cliente e gerar um token de acesso.
- **Body (JSON)**:

```json
{
  "nome": "Lucas Barros",
  "email": "lucas@vpo.tech",
  "senha": "123"
}
```

- **Resposta de Sucesso (`HTTP 201 Created`)**:

```json
{
  "mensagem": "Usuario criado com sucesso",
  "token": "uuid-gerado-pelo-banco"
}
```

### `POST /cadastro_b/login` (Autenticação)

- **Objetivo**: Validar credenciais e devolver o token de acesso do usuário.
- **Body (JSON)**:

```json
{
  "email": "lucas@vpo.tech",
  "senha": "123"
}
```

- **Resposta de Sucesso (`HTTP 200 OK`)**: Devolve o token de acesso configurado no formato `{"token": "uuid-do-banco"}`.

- **Resposta de Erro (`HTTP 401 Unauthorized`)**: Retorna `{"erro": "Credenciais invalidas"}` se a busca no banco falhar.

### `POST /cadastro_b/perfil` (Consulta de Perfil)

- **Objetivo**: Receber um token de acesso e retornar os dados detalhados do usuário.
- **Body (JSON)**:

```json
{
  "token": "uuid-do-banco"
}
```

- **Resposta de Sucesso (`HTTP 200 OK`)**:

```json
{
  "nome": "Lucas Barros",
  "perfil": "cliente",
  "email": "lucas@vpo.tech"
}
```

- **Resposta de Erro (`HTTP 401 Unauthorized`)**: Retorna `{"erro": "Token invalido ou nao encontrado"}` quando o token não está associado a um usuário.

---

## 4. Segurança e Contrato Global

As quatro rotas usam a autenticação `headerAuth` configurada no n8n. Na configuração de produção, as portas de entrada seguem estas regras de cabeçalho:

- `x-api-key`: Chave de autorização global (`turma2026`).
- `x-pedido-id`: Identificador de rastreio opcional.

As rotas de cadastro e login podem ser chamadas antes da existência de um pedido formal. Por isso, a ausência de `x-pedido-id` não impede o fluxo nem gera erro. Quando enviado, o identificador é repassado via Header para a API de logs.

---

## 5. Observabilidade e Logs

Em conformidade com os novos padrões da arquitetura distribuída, o serviço implementa **observabilidade assíncrona e resiliente** integrada à Logger API:

- O serviço é identificado no ecossistema global pelo número inteiro **8**.

- As ações de sucesso `CRIAR_USUARIO`, `LOGIN` e `CONSULTAR_PERFIL` disparam logs estruturados em segundo plano para o novo endpoint `POST /v1/log` do microsserviço da turma.

- O identificador de rastreio (`x-pedido-id`) é encaminhado para a Logger API de forma nativa através dos cabeçalhos da requisição HTTP (e não mais pelo body).

- **Tolerância a Falhas**: O envio dos logs nas três rotas utiliza a configuração `onError: continueRegularOutput`. Isso garante explicitamente que uma indisponibilidade do Logger (como erros 404 ou 503) seja ignorada pelo sistema, não interrompendo a jornada do cliente e concluindo a operação de identidade com sucesso.

---

## 6. Como Executar os Testes (Exemplos no Postman)

Utilize a URL de produção do servidor da turma para realizar os testes práticos. Configure a aba **Headers** em todas as requisições:

- `Content-Type`: `application/json` (para as requisições POST)
- `x-api-key`: `turma2026` (conforme a credencial `headerAuth` configurada no ambiente)
- `x-pedido-id`: `12345` (opcional; teste com e sem esse header)

**URLs de Acesso:**

1. **Verificar Saúde**: `GET` [Abrir endpoint de healthcheck](https://pzaas.online/webhook/cadastro_b/health)
2. **Criar Usuário**: `POST` [Abrir endpoint de cadastro](https://pzaas.online/webhook/cadastro_b/cadastro)
3. **Fazer Login**: `POST` [Abrir endpoint de login](https://pzaas.online/webhook/cadastro_b/login)
4. **Consultar Perfil**: `POST` [Abrir endpoint de perfil](https://pzaas.online/webhook/cadastro_b/perfil)

Os endpoints de cadastro, login e perfil consultam ou alteram a tabela `usuarios_pzaas`. O cadastro cria o perfil padrão `cliente` e gera o token com `gen_random_uuid()`. O login compara `email` e `senha` diretamente no banco; a proteção da senha não é realizada pelo workflow atual.
