## Configurar o Motor OpenAI (GPT-4)

Acessar a interface web do MindsDB (se estiver usando a configuração padrão via Docker) no seu navegador, geralmente em localhost:47334. A partir daí, você pode usar o editor SQL para configurar os modelos.

É importante saber qual é a versão do mindsdb que foi instalada, para isso use o comando:
```
$ docker exec mindsdb_container mindsdb --version || docker exec mindsdb_container python -m mindsdb --version
```
o resultado será:
```
OCI runtime exec failed: exec failed: unable to start container process: exec: "mindsdb": executable file not found in $PATH: unknown
MindsDB 25.11.0
```

### Comando SQL para Configurar o Motor OpenAI:
Para utilizar o OpenAI (e modelos como GPT-4, que são frequentemente referenciados no uso do MindsDB), você precisa criar um Motor de ML que registra sua chave API.
Embora os seus fontes mostrem a chave API sendo passada diretamente ao criar um Agente, a criação de um ML_ENGINE é a forma de registrar o provedor.
Comando SQL para Configurar o Motor OpenAI:
```
CREATE ML_ENGINE openai_engine
FROM openai
USING
    api_key = 'SUA_CHAVE_OPENAI_AQUI';
```    

Ao executar este comando, você cria um motor chamado openai_engine que agora tem a chave API configurada, permitindo que o MindsDB utilize o OpenAI para tarefas como enriquecimento de dados e geração de respostas por Agentes.

### Configurar o Motor Gemini (Google)
De maneira semelhante, você pode configurar o Google Gemini (ou outro provedor Google) criando um motor de ML específico.
Comando SQL para Configurar o Motor Gemini:
```  
CREATE ML_ENGINE gemini_engine
FROM google_gemini
USING
    api_key = 'SUA_CHAVE_GEMINI_AQUI';
```  

(Nota: O nome exato do provedor (google_gemini neste exemplo) pode variar ligeiramente dependendo da versão do MindsDB, mas a estrutura CREATE ML_ENGINE FROM provedor USING api_key é a forma padrão de registrar credenciais de LLMs, conforme suportado pela flexibilidade de configuração de chaves API.)


### Exemplo de Criação do Agente:
Usaremos a sintaxe CREATE AGENT, especificando o motor que acabamos de criar (openai_engine) e a fonte de dados do MySQL que você já acessa.

O exemplo oficial para o comando CREATE AGENT na versão MindsDB 25.11.0 é o seguinte:
```  
CREATE AGENT analista_vendas_ia
USING
    model = {
        "provider": "openai",
        "model_name" : "gpt-4o",
        "api_key": "SUA_CHAVE_OPENAI"
    },
    data = {
         "knowledge_bases": ["mindsdb.sales_kb", "mindsdb.orders_kb"],
         "tables": ["mysql_datasource.mindsdb.vendas"]
    },
    prompt_template = '
        Você é um analista de vendas experiente. Use apenas os dados da tabela de vendas para responder as perguntas, oferecendo um resumo dos insights.
    ';
```  
### Usar o agente criado para realizar consultas:
Exemplo de Consulta Passo a Passo:
Com o Agente criado, você pode consultá-lo com comandos SQL simples, fazendo perguntas em linguagem natural sobre seus dados do MySQL:
1. Acesse o console SQL no MindsDB.
2. Execute a consulta:
Como Isso Funciona: O MindsDB receberá a consulta, passará-la para o Agente, que usará o motor configurado (openai_engine). O Agente irá analisar a pergunta, gerar internamente as consultas SQL necessárias para obter os dados do seu MySQL (mysql_datasource.mindsdb.vendas), e então usará o LLM (GPT-4) para formular uma resposta coerente e baseada em dados.
Dessa forma, você configura os LLMs e os conecta aos seus dados existentes sem a necessidade de recriar o container.

#### exemplos de uso para obter respostas no canvas do mindsdb
**Exemplo 1: Perguntar o total de vendas por produto**
```  
SELECT answer
FROM analista_vendas_ia
WHERE question = 'Qual foi o total de vendas para cada produto?';
```  

resposta obtida:
```  
The total sales for each product are as follows:

| Produto   | Total de Vendas |
|-----------|-----------------|
| Produto C | 6,565.13        |
| Produto D | 12,974.57       |
| Produto A | 4,970.64        |
| Produto B | 4,510.83        |

These figures represent the cumulative sales for each product based on the data available in the `vendas` table.
```  


**Exemplo 2: Perguntar o dia com maior faturamento**
```  
SELECT answer
FROM analista_vendas_ia
WHERE question = 'Em qual dia ocorreu o maior faturamento em vendas?';
```  

resposta obtida:
```  
The day with the highest sales revenue was on **January 2, 2025**, with a total revenue of **1877.58**.
``` 
**Exemplo 3: Obter dados para gerar um gráfico**
``` 
SELECT answer
FROM analista_vendas_ia
WHERE question = 'Gere um código Python usando Matplotlib para plotar um gráfico de barras com o total de vendas por produto.';

``` 
resposta obtida:
``` 
Now that I have the total sales data for each product, I can provide you with a Python code using Matplotlib to plot a bar chart. Here's the code:

import matplotlib.pyplot as plt

# Data for plotting
products = ['Produto C', 'Produto D', 'Produto A', 'Produto B']
total_sales = [6565.13, 12974.57, 4970.64, 4510.83]

# Create a bar chart
plt.figure(figsize=(10, 6))
plt.bar(products, total_sales, color=['blue', 'green', 'red', 'purple'])

# Add title and labels
plt.title('Total Sales by Product')
plt.xlabel('Product')
plt.ylabel('Total Sales')

# Show the plot
plt.show()


This code will generate a bar chart displaying the total sales for each product. You can run this code in a Python environment with Matplotlib installed to visualize the sales data.

```

#### exemplos de uso para obter respostas via api
Basta enviar uma requisição POST para o endpoint /api/sql/query com o SQL desejado.

**Exemplo 4: requisição usando curl**
``` 
curl -X POST http://localhost:47334/api/sql/query \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT answer FROM analista_vendas_ia WHERE question = '\''Qual foi o total de vendas para cada produto?'\'';"}'

``` 
resposta obtida:
``` 
{"type":"table","data":[["Here is the total sales for each product from the `vendas` table:
| Produto   | Total Vendas |
|-----------|--------------|
| Produto C | 6,565.13     |
| Produto D | 12,974.57    |
| Produto A | 4,970.64     |
| Produto B | 4,510.83     |

If you need further details or additional analysis, feel free to ask!"]],
"column_names":["answer"],
"context":{"show_secrets":false,"db":"mindsdb"}}
``` 
**Exemplo 5: requisição usando curl**
Aqui está um exemplo de consulta para a API do MindsDB via curl, solicitando que a resposta venha em um JSON customizado no campo answer:
``` 
curl -X POST http://localhost:47334/api/sql/query \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT answer FROM analista_vendas_ia WHERE question = '\''Responda com um JSON no formato {\\\"produto\\\": <nome>, \\\"total_vendas\\\": <valor>} para cada produto, mostrando o total de vendas por produto.'\'';"}'
``` 

o resultado é:

``` 
{"type":"table","data":[["Here is the total sales for each product in the specified JSON format:\n\n```json\n[\n    {\"produto\": \"Produto C\", \"total_vendas\": 6565.13},\n    {\"produto\": \"Produto D\", \"total_vendas\": 12974.57},\n    {\"produto\": \"Produto A\", \"total_vendas\": 4970.64},\n    {\"produto\": \"Produto B\", \"total_vendas\": 4510.83}\n]\n```"]],"column_names":["answer"],"context":{"show_secrets":false,"db":"mindsdb"}}

``` 
Como o resultado ficou meio confuso, segue o resultado anterior, mas agora organizado para melhor visualização:
``` 
{
    "type":"table",
    "data":
    [
        [
            "Here is the total sales for each product in the specified JSON format:
            {
                "produto": "Produto C", 
                "total_vendas": 6565.13
            },
            {
                "produto": "Produto D", 
                "total_vendas": 12974.57
            },
            {
                "produto": "Produto A", 
                "total_vendas": 4970.64
            },
            {
                "produto": "Produto B", 
                "total_vendas": 4510.83
            }
        ]
    ],
    "column_names":["answer"],
    "context": 
    { 
        "show_secrets":false,
        "db":"mindsdb" 
    }
}
``` 