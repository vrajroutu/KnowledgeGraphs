
# Property Graph Construction with Predefined Schemas

This repository demonstrates how to build a property graph using Neo4j, Ollama, and Huggingface. The notebook uses `SchemaLLMPathExtractor` to specify an exact schema containing possible entity types, relation types, and defining how they can be connected together. This approach is useful when you have a specific graph in mind and want to limit what the LLM is predicting.

## Getting Started

### Prerequisites

Ensure you have the following installed:

- Docker
- Python 3.7 or later
- Jupyter Notebook or Jupyter Lab

### Installation

Clone the repository:

```bash
git clone https://github.com/vrajroutu/KnowledgeGraphs.git
cd KnowledgeGraphs
```

Install the required Python packages:

```bash
pip install llama-index
pip install llama-index-llms-ollama
pip install llama-index-embeddings-huggingface
pip install llama-index-graph-stores-neo4j
```

### Data Setup

Download the sample data used in the notebook:

```bash
mkdir -p 'data/paul_graham/'
wget 'https://raw.githubusercontent.com/yourusername/your-repo-name/main/data/paul_graham/paul_graham_essay.txt' -O 'data/paul_graham/paul_graham_essay.txt'
```

### Running the Notebook

To run the notebook, use Jupyter Notebook or Jupyter Lab:

```bash
jupyter notebook knowledgegraphs.ipynb
```

### Setting Up Neo4j

Ensure you have Docker installed and run the following command to set up Neo4j:

```bash
docker run \
    -p 7474:7474 -p 7687:7687 \
    -v $PWD/data:/data -v $PWD/plugins:/plugins \
    --name neo4j-apoc \
    -e NEO4J_apoc_export_file_enabled=true \
    -e NEO4J_apoc_import_file_enabled=true \
    -e NEO4J_apoc_import_file_use__neo4j__config=true \
    -e NEO4JLABS_PLUGINS=[\"apoc\"] \
    neo4j:latest
```

Access Neo4j at [http://localhost:7474/](http://localhost:7474/) and log in with the default username/password `neo4j`/`neo4j`. Change the password upon first login.

### Usage

Follow the steps in the notebook to:

1. Load data.
2. Construct the property graph.
3. Query the graph.

## Screenshots

![local graph](./local_kg.png)

## Documentation

For detailed documentation, refer to the module guides:

- [SchemaLLMPathExtractor](https://link-to-schema-llm-path-extractor-docs)
- [LLMSynonymRetriever](https://link-to-llm-synonym-retriever-docs)
- [VectorContextRetriever](https://link-to-vector-context-retriever-docs)

## Contributing

Contributions are welcome! Please open an issue or submit a pull request for any changes.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgements

- [Llama Index](https://github.com/run-llama/llama_index)
- [Neo4j](https://neo4j.com/)
- [Huggingface](https://huggingface.co/)

## Contact

For any questions or issues, please open an issue in the repository.

