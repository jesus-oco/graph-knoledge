# graph-knoledge

 AWS Well-Architected Neo4j QA

Proyecto de referencia para:

- Ingerir el PDF de la **AWS Well-Architected Framework**.
- Extraer entidades y relaciones relevantes con un **LLM**.
- Persistir el conocimiento en **Neo4j** usando **Cypher**.
- Permitir que un usuario haga preguntas en lenguaje natural y que el sistema:
  1. Convierta la pregunta a Cypher con el LLM.
  2. Ejecute la consulta en Neo4j.
  3. Devuelva una respuesta amigable, también generada por el LLM.

---

## 1. Arquitectura lógica

Componentes principales:

1. **Ingestor de PDF**  
   - Lee el PDF de la AWS Well-Architected Framework.
   - Hace *chunking* (por sección, por número de páginas, etc.).
   - Normaliza el texto (limpieza, eliminación de encabezados/pies de página, etc.).

2. **Extractor de entidades y relaciones (LLM)**  
   - Recibe texto de los *chunks*.
   - Usa un prompt de extracción para detectar:
     - Entidades: `Pillar`, `BestPractice`, `Question`, `Risk`, `Control`, etc.
     - Relaciones: `(:Pillar)-[:CONTAINS]->(:BestPractice)`, `(:BestPractice)-[:MITIGATES]->(:Risk)`, etc.
   - Devuelve JSON estructurado con nodos y relaciones.

3. **Capa de persistencia en Neo4j**  
   - Traduce el JSON a comandos **Cypher**.
   - Ejecuta los `CREATE` / `MERGE` sobre Neo4j.
   - Aplica un esquema de grafos definido (labels, propiedades, tipos de relaciones).

4. **Servicio de Q&A (API)**  
   - Expuesto típicamente vía **FastAPI** o similar.
   - Endpoints:
     - `POST /ingest` → dispara el pipeline de ingesta del PDF.
     - `POST /query` → recibe preguntas del usuario.
   - Flujo de `POST /query`:
     1. LLM convierte la pregunta en **consulta Cypher** basada en el esquema del grafo.
     2. Se ejecuta la consulta en Neo4j.
     3. El LLM resume los resultados en lenguaje natural, citando pilares, preguntas, riesgos, etc.

5. **LLM Orchestrator**  
   - Módulo para coordinar las llamadas al modelo:
     - `extract_entities(texto)`  
     - `question_to_cypher(pregunta_usuario, schema_grafo)`  
     - `answer_from_results(pregunta_usuario, resultados_cypher)`  

---

## 2. Tecnologías sugeridas

- **Lenguaje**: Python 3.11+
- **BD de grafos**: Neo4j 5.x (o AuraDB)
- **Driver Neo4j**: `neo4j` (official Python driver)
- **Framework API**: FastAPI
- **Cliente HTTP**: `httpx` o `requests`
- **Procesamiento de PDF**: `pypdf` o `pdfplumber`
- **LLM**:
  - Azure OpenAI, OpenAI u otro proveedor compatible.
- **Gestión de entorno**: `poetry` o `pip + venv`
- **Configuración**: variables de entorno (`.env` + `python-dotenv`)
