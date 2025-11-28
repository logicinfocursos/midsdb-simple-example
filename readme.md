# MindsDB - Ambiente de Machine Learning com LLMs e MySQL

## Sobre o MindsDB
MindsDB é uma plataforma open source que permite integrar modelos de machine learning (incluindo LLMs como OpenAI e Gemini) diretamente a bancos de dados relacionais, facilitando a análise preditiva e geração de insights usando SQL ou linguagem natural. Com o MindsDB, você pode conectar múltiplas fontes de dados, criar agentes inteligentes e consultar modelos via API ou SQL.

---

## Como usar os arquivos de Docker Compose

### 1. `docker-compose.yml`
Este arquivo sobe um ambiente básico com dois serviços:
- **mysql**: Banco de dados MySQL para armazenar seus dados.
- **mindsdb**: Plataforma MindsDB, com APIs HTTP e MySQL habilitadas.

**Uso:**
```bash
docker-compose up -d
```
Esse comando irá criar os containers usando as configurações padrão, com as credenciais e chaves definidas diretamente no arquivo.

---

### 2. `docker-compose.env.llms.yml` + `.env`
Este arquivo é uma versão aprimorada, que utiliza um arquivo `.env` para armazenar dados sensíveis (senhas e chaves de API dos LLMs). Ele já está preparado para receber as chaves de API do OpenAI e Gemini, facilitando a integração com esses provedores de LLM.

**Passos para uso:**
1. Edite o arquivo `.env` e preencha com suas credenciais e chaves de API:
   ```env
   MYSQL_ROOT_PASSWORD=...
   MYSQL_DATABASE=...
   MYSQL_USER=...
   MYSQL_PASSWORD=...
   OPENAI_API_KEY=...
   GEMINI_API_KEY=...
   ```
2. Suba os containers usando:
   ```bash
   docker-compose -f docker-compose.env.llms.yml --env-file .env up -d
   ```

---

## Diferenças entre os dois scripts
- **docker-compose.yml**: 
  - Configuração básica, com variáveis sensíveis escritas diretamente no arquivo.
  - Não é recomendado para produção, pois expõe senhas e chaves.
  - Não está preparado para múltiplos LLMs via variáveis de ambiente.

- **docker-compose.env.llms.yml**:
  - Utiliza o arquivo `.env` para variáveis sensíveis, aumentando a segurança e flexibilidade.
  - Já define variáveis para integração com LLMs (OpenAI e Gemini).
  - Recomendado para ambientes onde é necessário gerenciar múltiplas chaves e manter as credenciais fora do versionamento.

---

## Mysql
Para criar dados de exemplo para uso no mindsdb, você pode usar o script "mindsdb.sql"

## Referências
- [Documentação oficial MindsDB](https://docs.mindsdb.com/)
- [Exemplo de uso com LLMs](https://docs.mindsdb.com/mindsdb_sql/agents/agent)
- [notebook lm](https://notebooklm.google.com/notebook/e70d06a1-3528-45b0-bcba-487579c69930)

---
