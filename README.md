# Knowledge Graphs with Azure OpenAI

This repository provides a Jupyter Notebook that demonstrates how to use Neo4j and Azure OpenAI to build a property graph. By leveraging the `SchemaLLMPathExtractor`, we can construct a graph with predefined schemas, enabling precise control over entity types, relation types, and their connections.

## Table of Contents

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Installation](#installation)
- [Data Loading](#data-loading)
- [Graph Construction](#graph-construction)
- [Neo4j Setup](#neo4j-setup)
- [Usage](#usage)
- [Contact](#contact)

## Introduction

In this notebook, we walk through using Neo4j and Azure OpenAI to build a property graph. Specifically, we use the `SchemaLLMPathExtractor` to define and extract entities and relations based on a specific schema, ensuring that the LLM predictions are constrained to our defined graph structure.

## Requirements

To run this notebook, you will need the following:

- Python 3.8 or higher
- Jupyter Notebook
- Docker
- Libraries:
  - `llama-index`
  - `llama-index-llms-ollama`
  - `llama-index-embeddings-huggingface`
  - `llama-index-graph-stores-neo4j`
  - `openai`
  - `llama-index-embeddings-azure-openai`
  - `llama-index-llms-azure-openai`

## Installation

Install the required libraries using pip:

```bash
pip install llama-index
pip install llama-index-llms-ollama
pip install llama-index-embeddings-huggingface
pip install llama-index-graph-stores-neo4j
pip install openai
pip install llama-index-embeddings-azure-openai
pip install llama-index-llms-azure-openai
```

## Data Loading

To load data, create the necessary directory and download the sample data:

```bash
mkdir -p './azdev/paulgraham'
curl -o 'data/paul_graham/paul_graham_essay.txt' -O 'https://raw.githubusercontent.com/run-llama/llama_index/main/docs/docs/examples/data/paul_graham/paul_graham_essay.txt'
```

Then, use the following code snippet to read the data:

```python
from llama_index.core import SimpleDirectoryReader

documents = SimpleDirectoryReader("/azdev/paulgraham/").load_data()
```

## Graph Construction

To construct the graph, use the `SchemaLLMPathExtractor` with a predefined schema:

```python
import nest_asyncio
from typing import Literal
from llama_index.llms.azure_openai import AzureOpenAI
from llama_index.core.indices.property_graph import SchemaLLMPathExtractor

nest_asyncio.apply()

entities = Literal["PERSON", "PLACE", "ORGANIZATION"]
relations = Literal["HAS", "PART_OF", "WORKED_ON", "WORKED_WITH", "WORKED_AT"]

validation_schema = {
    "PERSON": ["HAS", "PART_OF", "WORKED_ON", "WORKED_WITH", "WORKED_AT"],
    "PLACE": ["HAS", "PART_OF", "WORKED_AT"],
    "ORGANIZATION": ["HAS", "PART_OF", "WORKED_WITH"],
}

kg_extractor = SchemaLLMPathExtractor(
    llm=AzureOpenAI(
        model="gpt-4",
        deployment_name="",
        api_key='',
        azure_endpoint="",
        api_version="2024-02-01",
    ),
    possible_entities=entities,
    possible_relations=relations,
    kg_validation_schema=validation_schema,
    strict=True
)
```

## Neo4j Setup

To launch Neo4j locally, ensure Docker is installed and run the following command:

```bash
docker run \
    -p 7474:7474 -p 7687:7687 \
    -v $PWD/data:/data -v $PWD/plugins:/plugins \
    --name neo4j-apoc \
    -e NEO4J_apoc_export_file_enabled=true \
    -e NEO4J_apoc_import_file_enabled=true \
    -e NEO4J_apoc_import_file_use__neo4j__config=true \
    -e NEO4J_AUTH=neo4j/database4591 \
    -e NEO4JLABS_PLUGINS='["apoc"]' \
    neo4j:latest
```

Access the database at [http://localhost:7474/](http://localhost:7474/). The default username/password is `neo4j/neo4j`.

## Usage

Initialize the Neo4j graph store:

```python
from llama_index.graph_stores.neo4j import Neo4jPGStore

graph_store = Neo4jPGStore(
    username="neo4j",
    password="",
    url="bolt://localhost:7687",
)
```

Set environment variables and initialize the Azure OpenAI client:

```python
import os
import asyncio
from openai import AzureOpenAI
from llama_index.core import PropertyGraphIndex

os.environ["AZURE_OPENAI_API_KEY"] = ""
os.environ["AZURE_OPENAI_ENDPOINT"] = ""

client = AzureOpenAI(
    api_key=os.getenv("AZURE_OPENAI_API_KEY"),
    api_version="2024-02-01",
    azure_endpoint=os.getenv("AZURE_OPENAI_ENDPOINT")
)

deployment_name = 'embedding'

def azure_openai_embedding(text):
    try:
        response = client.embeddings.create(
            model=deployment_name,
            input=text
        )
        return response['data'][0]['embedding']
    except Exception as e:
        print(f"Error getting embedding: {e}")
        return None

class AzureOpenAIEmbedding:
    def __init__(self, client):
        self.client = client

    def embed(self, text):
        return azure_openai_embedding(text)

    async def aget_text_embedding_batch(self, texts, **kwargs):
        try:
            loop = asyncio.get_event_loop()
            tasks = [loop.run_in_executor(None, azure_openai_embedding, text) for text in texts]
            responses = await asyncio.gather(*tasks)
            return [response for response in responses if response is not None]
        except Exception as e:
            print(f"Error getting batch embeddings: {e}")
            return None

embed_model = AzureOpenAIEmbedding(client)

try:
    index = PropertyGraphIndex.from_documents(
        documents,
        kg_extractors=[kg_extractor],
        embed_model=embed_model,
        property_graph_store=graph_store,
    )
except Exception as e:
    print(f"Error creating PropertyGraphIndex: {e}")
```

## Contact

For any questions or feedback, please reach out via email: vrajroutu@gmail.com.

---

Feel free to contribute to this repository by submitting issues or pull requests. Your contributions are highly appreciated!
