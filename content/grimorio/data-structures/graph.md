---
title: Graph
tags:
  - data-structures
alias:
  - grafo
  - grafos
  - graph
  - red
---
## 1. Qué es y cómo funciona

### Intuición

Muchos problemas no son lineales ni jerárquicos: son relaciones arbitrarias entre entidades. Pensá en una red social (personas que se siguen entre sí), un mapa de rutas (ciudades conectadas por caminos) o las materias de una carrera (algunas son correlativas de otras). Una [[linked list]] solo modela "uno sigue a otro" y un árbol solo modela "un padre, varios hijos", pero ninguna de las dos alcanza para representar relaciones de muchos a muchos.

Un Grafo resuelve esto modelando el problema como un conjunto de **entidades** (nodos o vértices) y un conjunto de **relaciones** entre pares de esas entidades (aristas o edges). Es la estructura más general para representar conexiones: tanto la lista como el árbol son, en el fondo, casos particulares de grafo con restricciones adicionales.

### Definición / propiedades

Un Grafo se define formalmente como un par $G = (V, E)$, donde:

- $V$ es el conjunto de **vértices** (nodos).
- $E$ es el conjunto de **aristas** (pares de vértices), con $E \subseteq V \times V$.

**Propiedades clave:**

- **Dirigido vs. no dirigido:** en un grafo dirigido, la arista $(u, v)$ tiene sentido de $u$ a $v$ y no implica $(v, u)$. En uno no dirigido, la conexión es simétrica.
- **Ponderado vs. no ponderado:** las aristas pueden tener un peso o costo asociado (ej. distancia, tiempo) o simplemente indicar presencia/ausencia de relación.
- **Grado de un vértice:** cantidad de aristas incidentes a él. En dirigidos se distingue grado de entrada (in-degree) y de salida (out-degree).
- **Camino:** secuencia de vértices conectados por aristas. Un **ciclo** es un camino que vuelve al vértice de origen.
- **Conectividad:** un grafo es conexo si existe un camino entre cualquier par de vértices. Un DAG (*Directed Acyclic Graph*) es un grafo dirigido sin ciclos.
- **Densidad:** un grafo es *disperso* (sparse) si $|E|$ es cercano a $|V|$, y *denso* si se acerca a $|V|^2$.

### Representación

Hay dos formas principales de representar un grafo en memoria:

**Lista de adyacencia:** cada vértice guarda una lista con sus vecinos.

![](/attachments/grimorio/data-structures/lista-adyacencia.svg)

**Matriz de adyacencia:** una matriz $|V| \times |V|$ donde la celda $(i, j)$ indica si existe (o el peso de) la arista entre $i$ y $j$.

![](/attachments/grimorio/data-structures/matriz-adyacencia.svg)

La lista de adyacencia es más eficiente en espacio para grafos dispersos (el caso más común en la práctica) y es la representación por defecto. La matriz de adyacencia conviene cuando el grafo es denso o se necesita consultar si existe una arista específica en $O(1)$.

---

## 2. Operaciones y complejidad

### Operaciones principales

- **`agregar_vertice(v)`:** incorpora un nuevo vértice al grafo.
- **`agregar_arista(u, v, peso=None)`:** conecta dos vértices, opcionalmente con un peso.
- **`eliminar_arista(u, v)`:** desconecta dos vértices.
- **`eliminar_vertice(v)`:** quita un vértice y todas sus aristas asociadas.
- **`vecinos(v)`:** devuelve los vértices adyacentes a `v`.
- **`existe_arista(u, v)`:** indica si hay conexión directa entre `u` y `v`.
- **`recorrer()`:** visita todos los vértices alcanzables, típicamente con BFS o DFS.

### Complejidad

Sean $|V|$ la cantidad de vértices y $|E|$ la cantidad de aristas.

| Operación | Lista de adyacencia | Matriz de adyacencia |
| :--- | :--- | :--- |
| `agregar_vertice` | $O(1)$ | $O(\|V\|^2)$<sup>1</sup> |
| `agregar_arista` | $O(1)$ | $O(1)$ |
| `eliminar_arista` | $O(\text{grado}(u))$ | $O(1)$ |
| `eliminar_vertice` | $O(\|V\| + \|E\|)$ | $O(\|V\|^2)$ |
| `existe_arista(u, v)` | $O(\text{grado}(u))$ | $O(1)$ |
| `vecinos(v)` | $O(\text{grado}(v))$ | $O(\|V\|)$ |
| Espacio | $O(\|V\| + \|E\|)$ | $O(\|V\|^2)$ |

<sup>1</sup>Requiere redimensionar la matriz completa para agregar una fila y columna nuevas.

### Detalles operativos

- **Grafos dispersos vs. densos:** con lista de adyacencia, un grafo disperso ($|E| \approx |V|$) usa memoria proporcional a $O(|V|)$; la matriz siempre usa $O(|V|^2)$ sin importar cuántas aristas existan realmente.
- **Aristas paralelas y multigrafos:** salvo que se implemente explícitamente, la mayoría de las representaciones no soportan más de una arista entre el mismo par de vértices.
- **Self-loops:** una arista de un vértice a sí mismo es válida en ambas representaciones, pero debe manejarse con cuidado en algoritmos de recorrido para no generar ciclos infinitos.
- **Grafos dirigidos:** en lista de adyacencia, eliminar un vértice requiere recorrer todas las listas para quitar las referencias entrantes, no solo su propia lista de salida.

---

## 3. Implementación

### Idea de implementación

La forma más común es usar una [[hash table]] (ej. `dict` en Python) donde cada clave es un vértice y el valor es la colección de sus vecinos. Esto combina la flexibilidad de identificar vértices por cualquier tipo hasheable con el acceso eficiente típico de una tabla hash.

### Invariantes

- Toda arista `(u, v)` requiere que tanto `u` como `v` existan como vértices del grafo.
- En un grafo no dirigido, agregar la arista `(u, v)` implica agregar también `(v, u)` en la lista de adyacencia de `v`.
- Eliminar un vértice debe eliminar también todas las aristas que lo referencian, para no dejar referencias colgantes.

### Ejemplo de código

```python
class Grafo:
    def __init__(self, dirigido=False):
        self.dirigido = dirigido
        self.adyacencia = {}

    def agregar_vertice(self, v):
        self.adyacencia.setdefault(v, {})

    def agregar_arista(self, u, v, peso=1):
        self.agregar_vertice(u)
        self.agregar_vertice(v)
        self.adyacencia[u][v] = peso
        if not self.dirigido:
            self.adyacencia[v][u] = peso

    def vecinos(self, v):
        return self.adyacencia.get(v, {})

    def existe_arista(self, u, v):
        return v in self.adyacencia.get(u, {})

    def bfs(self, inicio):
        visitados = {inicio}
        cola = [inicio]
        orden = []
        while cola:
            actual = cola.pop(0)
            orden.append(actual)
            for vecino in self.vecinos(actual):
                if vecino not in visitados:
                    visitados.add(vecino)
                    cola.append(vecino)
        return orden
```

#### Ejemplo de uso típico

```python
g = Grafo()
g.agregar_arista("A", "B")
g.agregar_arista("A", "C")
g.agregar_arista("C", "D")

print(g.bfs("A"))  # ['A', 'B', 'C', 'D']
```

---

## 4. Uso y criterio

### Casos de uso

- **Redes sociales:** modelar seguidores, amistades o conexiones entre usuarios.
- **Sistemas de mapas y rutas:** ciudades como vértices, caminos como aristas ponderadas por distancia o tiempo.
- **Dependencias:** correlatividades de materias, dependencias entre paquetes o tareas de un build (ordenamiento topológico).
- **Motores de recomendación:** relaciones entre usuarios y productos para inferir afinidades.
- **Redes de comunicación y transporte de datos:** enrutamiento de paquetes entre nodos de una red.

### Cuándo NO usarlo

- Cuando la relación entre los datos es estrictamente jerárquica (un padre, varios hijos): un árbol es más simple y eficiente.
- Cuando la relación es lineal (cada elemento se conecta solo con el siguiente): alcanza con una [[linked list]].
- Cuando el grafo sería trivialmente denso y pequeño y solo interesa la presencia de una relación puntual: una [[map]] o [[set]] de pares puede ser suficiente sin la sobrecarga conceptual de un grafo.

### Comparaciones

- **vs Árbol:** un árbol es un grafo conexo, acíclico y con una raíz definida; no permite múltiples caminos entre dos nodos. Un grafo general no tiene esas restricciones y puede tener ciclos y múltiples caminos entre el mismo par de vértices.
- **vs Lista de adyacencia vs Matriz de adyacencia:** la lista es preferible en grafos dispersos (la mayoría de los casos reales) por su menor uso de memoria; la matriz conviene en grafos densos o cuando se necesita consultar la existencia de una arista específica en $O(1)$ de forma constante.
- **vs Hash Table:** una hash table asocia una clave a un único valor; un grafo asocia un vértice a un conjunto de relaciones con otros vértices, permitiendo modelar conexiones de muchos a muchos.

### Ventajas / desventajas

**Ventajas:**

- Máxima flexibilidad para modelar relaciones arbitrarias entre entidades.
- Existe una amplia base de algoritmos bien estudiados (BFS, DFS, Dijkstra, Kruskal, entre otros) para resolver problemas sobre grafos.
- Se adapta tanto a relaciones simples (no ponderadas) como a escenarios más ricos (dirigidos, ponderados, con múltiples atributos por arista).

**Desventajas:**

- Mayor complejidad conceptual y de implementación que estructuras lineales.
- Muchos algoritmos sobre grafos son costosos en grafos grandes y densos.
- Elegir mal la representación (lista vs. matriz) puede degradar significativamente el rendimiento según el caso de uso.

### Señales de reconocimiento

- "Modelar conexiones entre entidades" o "quién está conectado con quién".
- "Encontrar el camino más corto/más barato entre dos puntos".
- "Detectar si existe un ciclo" o "determinar un orden válido de ejecución" (ordenamiento topológico).
- Problemas descriptos en términos de nodos y relaciones, redes o dependencias.

---

## 5. Relaciones y extensiones

### Variantes

- **Grafo dirigido (Digraph):** las aristas tienen sentido; usado para modelar dependencias o flujos.
- **Grafo ponderado:** las aristas tienen un costo asociado, base de algoritmos como Dijkstra o Kruskal.
- **DAG (Directed Acyclic Graph):** grafo dirigido sin ciclos; permite ordenamiento topológico y es la base de sistemas de build y de resolución de dependencias.
- **Multigrafo:** permite más de una arista entre el mismo par de vértices.
- **Árbol:** caso particular de grafo conexo y acíclico con una raíz definida.

### Relación con otras estructuras

- Cada vértice suele modelarse como un **[[struct]]** que agrupa su valor y sus referencias a los vecinos.
- La representación por lista de adyacencia se implementa habitualmente sobre una **[[hash table]]** (vértice → colección de vecinos) o sobre un **[[array]]** cuando los vértices son identificables por índices enteros consecutivos.
- El recorrido en anchura (BFS) usa una cola (ver **[[deque]]**), mientras que el recorrido en profundidad (DFS) usa una pila (ver **[[stack]]**) o la pila de llamadas mediante recursión.
- Un **[[set]]** se usa típicamente para llevar el registro de los vértices ya visitados durante un recorrido.
- Una **[[linked list]]** y un árbol son, conceptualmente, grafos con restricciones adicionales sobre su cantidad de conexiones.

### Notas avanzadas

#### Grafos a gran escala

Cuando $|V|$ y $|E|$ son muy grandes (ej. redes sociales con millones de usuarios), ni la lista ni la matriz de adyacencia en memoria son viables por sí solas. Se recurre a representaciones comprimidas, particionado del grafo entre múltiples máquinas, o bases de datos especializadas (bases de datos de grafos como Neo4j), que optimizan el almacenamiento y las consultas de vecindad a gran escala.

#### Algoritmos fundamentales

El grafo como estructura es la base de una familia extensa de algoritmos: BFS y DFS para recorridos, Dijkstra y Bellman-Ford para caminos mínimos, Kruskal y Prim para árboles de expansión mínima, y algoritmos de flujo máximo. La elección de representación (lista vs. matriz) impacta directamente en la complejidad final de estos algoritmos.

---

## 6. Referencias y recursos

- [[COR2011]] - Capítulo 22: "Elementary Graph Algorithms".
- [[KLE2005]] - Capítulo 3: "Graphs".
