```markdown
# Knowledge Graphs with Azure OpenAI

This repository contains a Jupyter Notebook that demonstrates how to use Neo4j and Azure OpenAI to build a property graph. Specifically, it showcases the use of the `SchemaLLMPathExtractor` to construct a graph with a predefined schema, limiting the predictions of the Large Language Model (LLM) to the specified schema.

## Table of Contents
- [Installation](#installation)
- [Usage](#usage)
- [Graph Construction](#graph-construction)
- [Launching Neo4j](#launching-neo4j)
- [Querying the Graph](#querying-the-graph)
- [Contact](#contact)

## Installation

To run this notebook, you will need to install the following Python packages:

```bash
pip install llama-index
pip install llama-index-llms-ollama
pip install llama-index-embeddings-huggingface
pip install llama-index-graph-stores-neo4j
pip install openai
pip install llama-index-embeddings-azure-openai
pip install llama-index-llms-azure-openai
```

## Usage

### Load Data

The notebook starts by loading data from a sample file. Make sure you have the necessary directories and data files:

```bash
mkdir -p './azdev/paulgraham'
curl -o 'https://raw.githubusercontent.com/run-llama/llama_index/main/docs/docs/examples/data/paul_graham/paul_graham_essay.txt' -O 'data/paul_graham/paul_graham_essay.txt'
```

### Graph Construction

The notebook demonstrates how to construct a graph using the `SchemaLLMPathExtractor`. It defines a schema with possible entity types and relation types, ensuring the LLM predictions conform to this schema.

```python
from typing import Literal
from llama_index.llms.azure_openai import AzureOpenAI
from llama_index.core.indices.property_graph import SchemaLLMPathExtractor

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

### Launching Neo4j

To run Neo4j locally, ensure you have Docker installed. Launch the database using the following command:

```bash
docker run \
    -p 7474:7474 -p 7687:7687 \
    -v $PWD/data:/data -v $PWD/plugins:/plugins \
    --name neo4j-apoc \
    -e NEO4J_apoc_export_file_enabled=true \
    -e NEO4J_apoc_import_file_enabled=true \
    -e NEO4J_apoc_import_file_use__neo4j__config=true \
    -e NEO4J_AUTH=neo4j/database4591 \
    -e NEO4JLABS_PLUGINS=[\"apoc\"] \
    neo4j:latest
```

Access Neo4j at [http://localhost:7474/](http://localhost:7474/) with the default credentials (`neo4j`/`neo4j`). Change the password upon first login.

### Querying the Graph

After constructing the graph, the notebook demonstrates how to query it using a lower-level API and custom retrievers.

```python
from llama_index.core.indices.property_graph import (
    LLMSynonymRetriever,
    VectorContextRetriever,
)
from llama_index.llms.azure_openai import AzureOpenAI
from llama_index.embeddings.azure_openai import AzureOpenAIEmbedding

llm = AzureOpenAI(
    model="gpt-4",
    deployment_name="chat",
    api_key="",
    azure_endpoint="",
    api_version="2024-02-01",
)

embed_model = AzureOpenAIEmbedding(
    model="text-embedding-ada-002",
    deployment_name="my-custom-embedding",
    api_key="",
    azure_endpoint="",
    api_version="2024-02-01",
)

llm_synonym = LLMSynonymRetriever(
    index.property_graph_store,
    # Add more code as needed for querying
)
```

## Contact

For any questions or inquiries, please contact:

Email: vrajroutu@gmail.com
```
