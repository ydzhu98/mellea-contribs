<img src="https://github.com/generative-computing/mellea-contribs/raw/main/mellea-contribs.jpg" height=100>

# Mellea Contribs

The `mellea-contribs` repository is an incubation point for contributions to the Mellea ecosystem. It provides a library for incubating generative programming tools and utilities that integrate with the Mellea LLM agent framework.

## Features

### Knowledge Graph-Enhanced RAG (KG-RAG)

A powerful Knowledge Graph Retrieval-Augmented Generation system built with the Mellea framework. It combines Neo4j graph databases with large language models to solve complex reasoning tasks.

**Key Capabilities:**
- **Multi-hop Reasoning**: Navigate complex relationships across knowledge graphs
- **Multi-Route Problem Solving**: Break down questions into multiple solving routes
- **Consensus-Based Answers**: Validate answers across different reasoning paths
- **Entity/Relation Extraction**: Extract and align entities and relations from documents
- **Temporal Awareness**: Handle time-sensitive information in graphs
- **Explainable Results**: Get reasoning paths and justifications for answers

**Example Use Cases:**
- Movie industry queries: "Who won Best Picture in 2024 and what was their previous work?"
- Knowledge base Q&A: Multi-hop queries with supporting evidence
- Graph updating: Extract entities and relations from documents and merge with existing KG
- Domain-specific reasoning: Movies, finance, or any domain with structured data

### Validators & Requirements

Domain-specific validators extending Mellea's validation framework:
- Legal case citation validation
- Python import error repair
- Court classification
- Statute data handling

### Selection & Ranking Tools

Reusable LLM-based algorithms:
- **Top-K Selection**: Generic selection engine with rejection sampling
- **Double Round Robin**: Pairwise comparison scoring

## Quick Start

### Knowledge Graph RAG

```python
import asyncio
from mellea import start_session
from mellea_contribs.kg import (
    Neo4jBackend,
    orchestrate_qa_retrieval,
    break_down_question,
)

async def main():
    # Initialize Mellea session
    session = start_session(backend_name="litellm", model_id="gpt-4o-mini")

    # Connect to Neo4j or use MockGraphBackend for testing
    backend = Neo4jBackend(
        uri="bolt://localhost:7687",
        auth=("neo4j", "password"),
    )

    # Multi-route QA with consensus validation
    query = "Who won the Best Picture Oscar in 2024?"
    answer = await orchestrate_qa_retrieval(
        session=session,
        backend=backend,
        query=query,
        domain="movies",
        num_routes=3,  # Explore 3 different solving routes
    )
    print(f"Answer: {answer}")

    # Or use individual generative functions
    routes = await break_down_question(
        query=query,
        query_time="2024-03-10",
        domain="movies",
        route=3,
        hints="Consider temporal aspects of awards",
    )
    print(f"Solving routes suggested: {len(routes.routes)}")

    await backend.close()

asyncio.run(main())
```

### Document-Based KG Updates

```python
from mellea_contribs.kg import orchestrate_kg_update

result = await orchestrate_kg_update(
    session=session,
    backend=backend,
    doc_text="Leonardo DiCaprio starred in Inception (2010) and Titanic (1997)...",
    domain="movies",
    entity_types="Person,Movie",
    relation_types="ACTED_IN,DIRECTED",
)

print(f"Extracted {len(result['extracted_entities'])} entities")
print(f"Extracted {len(result['extracted_relations'])} relations")
```

## Installation

```bash
# Basic installation
pip install mellea-contribs

# With Knowledge Graph support (Neo4j)
pip install mellea-contribs[kg]

# Development installation
pip install -e ".[dev]"
```

## KG-RAG Architecture

The system follows a 4-layer architecture:

```
Layer 1: Application Orchestration
├─ orchestrate_qa_retrieval() - Multi-route QA
└─ orchestrate_kg_update() - Document-based KG updates

Layer 2: Components
├─ GraphQuery, CypherQuery, SparqlQuery
├─ GraphResult with format_for_llm()
└─ explain_query_result(), natural_language_to_cypher()

Layer 3: LLM-Guided Logic (@generative functions)
├─ QA Functions (8):
│  ├─ break_down_question
│  ├─ extract_topic_entities
│  ├─ align_topic_entities
│  ├─ prune_relations
│  ├─ prune_triplets
│  ├─ evaluate_knowledge_sufficiency
│  ├─ validate_consensus
│  └─ generate_direct_answer
├─ Update Functions (5):
│  ├─ extract_entities_and_relations
│  ├─ align_entity_with_kg
│  ├─ decide_entity_merge
│  ├─ align_relation_with_kg
│  └─ decide_relation_merge

Layer 4: Backends
├─ GraphBackend (abstract interface)
├─ Neo4jBackend (production Neo4j)
└─ MockGraphBackend (in-memory testing)
```

## QA Pipeline: How It Works

1. **Question Breakdown**: Convert complex questions into multiple solving routes
2. **Entity Extraction**: Extract topic entities from the query
3. **Entity Alignment**: Match entities with KG and score relevance (0-1)
4. **Relation Pruning**: Filter relevant relations from discovered entities
5. **Knowledge Evaluation**: Determine if retrieved knowledge suffices to answer
6. **Consensus Validation**: Validate answer across multiple routes
7. **Final Answer**: Return answer with reasoning path

## Update Pipeline: How It Works

1. **Entity/Relation Extraction**: Extract entities and relations from documents
2. **Entity Alignment**: Find matching entities in existing KG
3. **Entity Merge**: Decide how to merge extracted vs. KG entities
4. **Relation Alignment**: Find matching relations in KG
5. **Relation Merge**: Decide how to merge extracted vs. KG relations
6. **KG Update**: Write merged data back to graph database

## Testing

```bash
# Run tests without Neo4j dependency
pytest test/kg/test_base.py test/kg/test_mock_backend.py -v

# Run all tests (requires Neo4j)
export NEO4J_URI=bolt://localhost:7687
export NEO4J_USER=neo4j
export NEO4J_PASSWORD=password
pytest test/kg/ -v

# Start Neo4j for testing
docker run -d --name neo4j-test -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/testpassword \
  neo4j:5.0
```

## Documentation

- [KG Library Documentation](./mellea_contribs/kg/README.md) - Detailed backend/query guide
- [CLAUDE.md](./CLAUDE.md) - Development guide and architecture overview
