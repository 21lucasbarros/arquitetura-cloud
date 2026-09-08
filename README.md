# PZaaS API - Cadastro e Identidade

Documentação oficial do microsserviço de Cadastro/Identidade (Serviço 08) responsável pelo gerenciamento de usuários, autenticação, validação de tokens e integração com banco de dados na arquitetura distribuída da pizzaria.

---

## 1. Visão Geral

O **Serviço 08** fornece a identidade e os perfis dos usuários do ecossistema PZaaS. Desenvolvido no **n8n** (servidor compartilhado da turma) e integrado ao **PostgreSQL (Neon Tech)**, ele autentica credenciais, cria novas contas e retorna dados cadastrais utilizando tokens seguros.

## 2. Tecnologias Utilizadas

* **n8n**: Orquestração e execução dos endpoints e lógica de negócio.
* **PostgreSQL (Neon Tech)**: Banco de dados relacional para armazenamento dos perfis (tabela `usuarios_pzaas`).
* **Postman**: Ferramenta utilizada para testes das rotas.

---

## 3. Arquitetura e Endpoints

O microsserviço possui quatro rotas principais rodando de forma isolada (`cadastro_b`) no servidor de produção.

### `GET /cadastro_b/health` (Healthcheck)

* **Objetivo**: Verificar se o microsserviço está online.


* **Resposta de Sucesso (`HTTP 200 OK`)**: Retorna `{"status": "tudo certo"}`.



### `POST /cadastro_b/cadastro` (Criação de Usuário)

* **Objetivo**: Registrar um novo cliente e gerar um token de acesso.


* **Body (JSON)**:
```json
{
  "nome": "Lucas Barros",
  "email": "lucas@vpo.tech",
  "senha": "123"
}

```


* **Resposta de Sucesso (`HTTP 201 Created`)**: Retorna a mensagem de sucesso e o token gerado pelo banco de dados.



### `POST /cadastro_b/login` (Autenticação)

* **Objetivo**: Validar credenciais e devolver o token de acesso do usuário.


* **Body (JSON)**:
```json
{
  "email": "lucas@vpo.tech",
  "senha": "123"
}

```


* **Resposta de Sucesso (`HTTP 200 OK`)**: Devolve o token de acesso configurado no formato `{"token": "uuid-do-banco"}`.


* **Resposta de Erro (`HTTP 401 Unauthorized`)**: Retorna `{"erro": "Credenciais invalidas"}` se a busca no banco falhar.



### `POST /cadastro_b/perfil` (Consulta de Perfil)

* **Objetivo**: Receber um token de acesso e retornar os dados detalhados do usuário.


* **Body (JSON)**:
```json
{
  "token": "uuid-do-banco"
}

```


* **Resposta de Sucesso (`HTTP 200 OK`)**: Retorna os campos `nome`, `perfil` e `email` associados ao token.



---

## 4. Segurança e Contrato Global

Para garantir a padronização entre todos os microsserviços da arquitetura da pizzaria, este serviço valida rigorosamente os seguintes headers em **todas** as requisições:

* `x-api-key`: Chave de autorização global (`turma2026`).


* `x-pedido-id`: Identificador de rastreio obrigatório para o fluxo da requisição.



> **Tratamento de Falhas (Header)**: Caso a requisição chegue sem o cabeçalho obrigatório `x-pedido-id`, a API intercepta a chamada e retorna imediatamente um `HTTP 400 Bad Request` com o detalhe de que o header é obrigatório.
> 
> 

---

## 5. Observabilidade e Logs

Em conformidade com os padrões da arquitetura distribuída, o serviço implementa **observabilidade assíncrona**:

* O serviço é identificado no ecossistema global pelo **ID 8** (Cadastro/Identidade).


* As ações de sucesso como `CRIAR_USUARIO`, `LOGIN` e `CONSULTAR_PERFIL` disparam logs estruturados em segundo plano (`POST /logs`) para o microsserviço de Logger da turma.


* Tentativas de acesso sem o cabeçalho obrigatório disparam automaticamente um log de alerta no nível `WARN` sob a ação `VALIDAR_CABECALHO`.



---

## 6. Como Executar os Testes (Exemplos no Postman)

Utilize a URL de produção do servidor da turma para realizar os testes práticos. Configure a aba **Headers** em todas as requisições:

* `Content-Type`: `application/json` (para as requisições POST)
* `x-api-key`: `turma2026`
* `x-pedido-id`: `12345` (ou qualquer valor numérico de rastreio)

**URLs de Acesso:**

1. **Criar Usuário**: `POST [https://pzaas.online/webhook/cadastro_b/cadastro](https://pzaas.online/webhook/cadastro_b/cadastro)`
2. **Fazer Login**: `POST [https://pzaas.online/webhook/cadastro_b/login](https://pzaas.online/webhook/cadastro_b/login)`
3. **Consultar Perfil**: `POST [https://pzaas.online/webhook/cadastro_b/perfil](https://pzaas.online/webhook/cadastro_b/perfil)`
4. **Verificar Saúde**: `GET [https://pzaas.online/webhook/cadastro_b/health](https://pzaas.online/webhook/cadastro_b/health)`
