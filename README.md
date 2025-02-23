# azure-openai-proxy

[![License](https://img.shields.io/github/license/koordinator-sh/koordinator.svg?color=4EB1BA&style=flat-square)](https://opensource.org/licenses/Apache-2.0)
[![GitHub release](https://img.shields.io/github/v/release/stulzq/azure-openai-proxy.svg?style=flat-square)](https://github.com/stulzq/azure-openai-proxy/releases/latest)
[![Go Report Card](https://goreportcard.com/badge/github.com/stulzq/azure-openai-proxy)](https://goreportcard.com/badge/github.com/stulzq/azure-openai-proxy)
[![CI](https://img.shields.io/github/actions/workflow/status/stulzq/azure-openai-proxy/ci.yml?label=CI&logo=github&style=flat-square&branch=master)](https://github.com/stulzq/azure-openai-proxy/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/actions/workflow/status/stulzq/azure-openai-proxy/release.yml?label=Release&logo=github&style=flat-square&branch=master)](https://github.com/stulzq/azure-openai-proxy/actions/workflows/release.yml)
[![PRs Welcome](https://badgen.net/badge/PRs/welcome/green?icon=https://api.iconify.design/octicon:git-pull-request.svg?color=white&style=flat-square)](CONTRIBUTING.md)

English|[中文](https://www.cnblogs.com/stulzq/p/17271937.html)

Azure OpenAI Service Proxy, convert OpenAI official API request to Azure OpenAI API request, support all models, support GPT-4.

![aoai-proxy.jpg](assets/images/aoai-proxy.jpg)

## Get Start

### Retrieve key and endpoint

To successfully make a call against Azure OpenAI, you'll need the following:

| Name                  | Desc                                                         | Default                                                  |
| --------------------- | ------------------------------------------------------------ | ----------------------------- |
| AZURE_OPENAI_ENDPOINT | This value can be found in the **Keys & Endpoint** section when examining your resource from the Azure portal. Alternatively, you can find the value in **Azure OpenAI Studio** > **Playground** > **Code View**. An example endpoint is: `https://docs-test-001.openai.azure.com/`. | N |
| AZURE_OPENAI_API_VER  | [See here](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/quickstart?tabs=command-line&pivots=rest-api) or Azure OpenAI Studio | 2023-03-15-preview |
| AZURE_OPENAI_MODEL_MAPPER   | This value will correspond to the custom name you chose for your deployment when you deployed a model. This value can be found under **Resource Management** > **Deployments** in the Azure portal or alternatively under **Management** > **Deployments** in Azure OpenAI Studio. | gpt-3.5-turbo=gpt-35-turbo |

`AZURE_OPENAI_MODEL_MAPPER` is a mapping from Azure OpenAI deployed model names to official OpenAI model names. You can use commas to separate multiple mappings.

**Format：**

`AZURE_OPENAI_MODEL_MAPPER`: \<OpenAI Model Name\>= \<Azure OpenAI deployment model name\>

OpenAI Model Names: https://platform.openai.com/docs/models

Azure Deployment Names: **Resource Management** > **Deployments**

**Example:**

````shell
AZURE_OPENAI_MODEL_MAPPER: gpt-3.5-turbo=azure-gpt-35
````

![Screenshot of the overview UI for an OpenAI Resource in the Azure portal with the endpoint & access keys location circled in red.](assets/images/endpoint.png)

API Key: This value can be found in the **Keys & Endpoint** section when examining your resource from the Azure portal. You can use either `KEY1` or `KEY2`. 


### Use Docker

````shell
docker run -d -p 8080:8080 --name=azure-openai-proxy \
  --env AZURE_OPENAI_ENDPOINT=your_azure_endpoint \
  --env AZURE_OPENAI_API_VER=your_azure_api_ver \
  --env AZURE_OPENAI_MODEL_MAPPER=your_azure_deploy_mapper \
  stulzq/azure-openai-proxy:latest
````

Call API:

````shell
curl --location --request POST 'localhost:8080/v1/chat/completions' \
-H 'Authorization: Bearer <Azure OpenAI Key>' \
-H 'Content-Type: application/json' \
-d '{
    "max_tokens": 1000,
    "model": "gpt-3.5-turbo",
    "temperature": 0.8,
    "top_p": 1,
    "presence_penalty": 1,
    "messages": [
        {
            "role": "user",
            "content": "Hello"
        }
    ],
    "stream": true
}'
````

### Use ChatGPT-Web

ChatGPT Web: https://github.com/Chanzhaoyu/chatgpt-web

![chatgpt-web](assets/images/chatgpt-web.png)

Envs:

- `OPENAI_API_KEY` Auzre OpenAI API Key
- `AZURE_OPENAI_ENDPOINT` Auzre OpenAI API Endpoint
- `AZURE_OPENAI_MODEL_MAPPER` Auzre OpenAI API Deployment Name Mappings

docker-compose.yml:

````yaml
version: '3'

services:
  chatgpt-web:
    image: chenzhaoyu94/chatgpt-web
    ports:
      - 3002:3002
    environment:
      OPENAI_API_KEY: <Auzre OpenAI API Key>
      OPENAI_API_BASE_URL: http://azure-openai:8080
      AUTH_SECRET_KEY: ""
      MAX_REQUEST_PER_HOUR: 1000
      TIMEOUT_MS: 60000
    depends_on:
      - azure-openai
    links:
      - azure-openai
    networks:
      - chatgpt-ns

  azure-openai:
    image: stulzq/azure-openai-proxy
    ports:
      - 8080:8080
    environment:
      AZURE_OPENAI_ENDPOINT: <Auzre OpenAI API Endpoint>
      AZURE_OPENAI_MODEL_MAPPER: <Auzre OpenAI API Deployment Mapper>
      AZURE_OPENAI_API_VER: 2023-03-15-preview
    networks:
      - chatgpt-ns

networks:
  chatgpt-ns:
    driver: bridge
````

Run:

````shell
docker compose up -d
````

