## **Entidades Fundamentales (Día 0)**

* **Colecciones (Collections):** Son el equivalente a las tablas en una base de datos relacional, pero diseñadas para almacenar datos vectoriales. Al crear una colección, debes definir su nombre, el tamaño (dimensionalidad) de los vectores y la métrica de distancia a utilizar.
* **Puntos (Points):** Es la entidad de datos central en Qdrant. Cada punto está compuesto por tres elementos:
* **ID:** El identificador único del punto, que puede ser un número entero positivo de 64 bits o un UUID.
* **Vector:** El arreglo de valores numéricos que representa los datos en el espacio vectorial. Qdrant soporta múltiples vectores por punto mediante **Vectores Nombrados (Named Vectors)**.
* **Payload (Metadatos):** Información estructurada en formato clave-valor (ej. etiquetas, precios, fechas) que acompaña al vector y es crucial para filtrar búsquedas. Los tipos de datos pueden ser escalares, categóricos, de geolocalización o marcas de tiempo.



---

## **Vectores, Distancias y Chunking (Día 1)**

### **Tipos de Vectores**

* **Vectores Densos (Dense Vectors):** Creados por redes neuronales (embeddings). Capturan el significado y contexto profundo (relaciones semánticas).
* **Vectores Dispersos (Sparse Vectors):** Vectores donde la mayoría de los valores son cero. Ideal para búsqueda exacta de palabras clave (búsqueda léxica) usando modelos estadísticos como BM25 o neuronales como SPLADE.
* **Multivectores:** Matrices de vectores densos por punto. Usados en interacciones a nivel de token con modelos como ColBERT.

### **Métricas de Distancia**

Definen matemáticamente cómo de "similares" son dos vectores:

1. **Similitud Coseno (Cosine):** Mide el ángulo entre vectores, enfocándose en la dirección (significado) e ignorando su longitud (magnitud).
2. **Distancia Euclidiana:** Mide la distancia en línea recta. Útil cuando cada magnitud numérica tiene importancia estricta, pero es muy sensible a la escala.
3. **Producto Punto (Dot Product):** Premia a los vectores que apuntan en la misma dirección y además tienen gran magnitud.

### **Chunking (Fragmentación)**

Dividir documentos largos en fragmentos manejables antes de convertirlos en embeddings. Estrategias comunes incluyen:

* Fragmentación por tamaño fijo.
* Por oraciones o párrafos.
* **Semantic Chunking:** Detecta cambios de significado para dividir.

> **Nota:** El chunking previene que el significado de un documento se diluya en un solo vector.

---

## **Indexación y Filtrado Eficiente (Día 2)**

### **Búsqueda por Fuerza Bruta vs. HNSW**

La fuerza bruta compara la consulta contra todos los vectores, lo cual es ineficiente a gran escala. Qdrant usa el índice **HNSW (Hierarchical Navigable Small World)**, un grafo multicapa que permite navegar rápidamente desde categorías generales hasta similitudes exactas sin evaluar todo el dataset.

**Parámetros de HNSW:**

* **m:** Número de conexiones por nodo en el grafo.
* **ef_construct:** Cuántos vecinos se evalúan al indexar (mayor valor = grafo más preciso pero indexación más lenta).
* **ef:** Cuántos vecinos se evalúan durante la búsqueda (afecta velocidad vs. precisión).

### **Filterable HNSW y Payload Indexing**

Al filtrar por Payload, si se eliminan muchos puntos, la ruta de búsqueda en el grafo podría romperse. Qdrant soluciona esto construyendo **subgrafos** que mantienen la conectividad bajo condiciones de filtrado. Es vital indexar los campos del payload para que el filtro sea eficiente.

---

## **Estrategias de Búsqueda Avanzada (Día 3)**

* **Búsqueda Híbrida (Hybrid Search):** Combina las fortalezas de los vectores densos (semántica) y los dispersos (palabras clave) en una sola llamada usando la *Universal Query API*.
* **Algoritmos de Fusión:** Como los puntajes no son comparables directamente, se usan algoritmos como **RRF (Reciprocal Rank Fusion)** para combinar los resultados basándose en la posición del ranking de cada método.
* **Late Interaction (Interacción Tardía):** Modelos como ColBERT mantienen el documento a nivel de "token" (múltiples vectores).
* En consulta, se compara cada token de la búsqueda con cada token del documento (**MaxSim**).
* **Flujo común:** Recuperar candidatos rápido con vectores densos y luego re-ordenarlos (*re-rank*) con multivectores.



---

## **Escalamiento, Rendimiento y Cuantización (Días 4+)**

### **Cuantización (Quantization)**

Técnica para comprimir vectores en RAM:

* **Escalar (Scalar):** Pasa de *Float32* a *Int8* (ahorro del 75% de memoria).
* **Binaria (Binary):** Reduce dimensiones a 0s o 1s (ahorro de hasta 32x y aceleración de hasta 40x). Ideal para alta dimensionalidad (>1024).

### **Oversampling, Rescoring y Reranking**

Al usar cuantización, se pueden perder resultados óptimos. Para compensar:

1. **Oversampling:** Recuperar candidatos adicionales.
2. **Rescoring:** Re-evaluar rápidamente usando los vectores originales no comprimidos.
3. **Reranking:** Reordenar el top final para recuperar la precisión perdida.

### **Ingesta a Gran Escala**

Para millones de vectores, el `upsert` punto por punto es lento. Se recomiendan métodos paralelos:

* `upload_points`: Envíos en lotes desde memoria.
* `upload_collection`: Lectura progresiva directamente desde el disco.

> **Tip:** Ajustar el número y tamaño máximo de los segmentos de optimización para un desempeño óptimo.