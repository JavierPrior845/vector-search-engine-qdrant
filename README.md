# vector-search-engine-qdrant
Este repositorio contiene mi implementación y notas técnicas sobre el uso de Qdrant para la construcción de sistemas de búsqueda semántica y arquitecturas RAG.

## 🎯 Objetivos de Aprendizaje
- Gestión de Colecciones y Vectores.
- Implementación de búsqueda por similitud (Cosine Similarity).
- Filtrado de metadatos (Payload filtering) para búsquedas precisas.
- Integración con modelos de Embeddings de Hugging Face.

## 🛠️ Stack Tecnológico
- **Vector DB:** Qdrant (vía Docker).
- **Language:** Python.
- **Tools:** `qdrant-client`, `sentence-transformers`.

## 🚀 Cómo ejecutarlo localmente
```bash
# Levantar Qdrant en Docker
docker run -p 6333:6333 qdrant/qdrant