# Sistema de Logistica de Entregas 
## Quadtree vs Fuerza Bruta

**Laboratorio — Curso de Estructura de Datos**

---

## Descripcion General

Este laboratorio pertenece al curso de Estructura de Datos y tiene como objetivo implementar desde cero un arbol Quadtree en Python para resolver un problema real de logistica urbana: dado un conjunto de 10.000 puntos de entrega en una ciudad, responder de forma eficiente las siguientes consultas:

- Cuales puntos de entrega se encuentran dentro de un radio de 500 metros desde una ubicacion dada (Range Search).
- Cual es el punto de entrega mas cercano a una ubicacion dada (Nearest Neighbor).

Adicionalmente se compara el rendimiento del Quadtree contra una solucion de fuerza bruta basada en listas, con el fin de identificar a partir de que volumen de datos el arbol comienza a ser mas eficiente.

---

## Herramientas Utilizadas

**Lenguaje y entorno**

- Python 3.10 o superior
- Google Colab como entorno de ejecucion (compatible con cualquier entorno Python estandar)

**Bibliotecas**

| Biblioteca | Uso |
|---|---|
| `matplotlib` | Generacion de graficas y visualizaciones |
| `numpy` | Generacion de datos y soporte para graficas |
| `math` | Calculo de distancias euclidianas |
| `time` | Medicion de tiempos de ejecucion para el benchmark |
| `random` | Generacion de puntos de entrega y consultas aleatorias |
| `collections` | Definicion del namedtuple Point |

No se utilizaron librerias de arboles espaciales como `scipy.spatial.KDTree`, `sklearn.neighbors` ni ninguna estructura de arbol pre-construida. Toda la logica del Quadtree fue implementada desde cero.

---

## Ecuaciones y Metodos

### Distancia Euclidiana

Toda medicion de distancia entre dos puntos en el plano cartesiano se calcula con:

```
d = sqrt( (x2 - x1)^2 + (y2 - y1)^2 )
```

Las coordenadas estan expresadas en metros, por lo que la distancia resultante tambien esta en metros. Un radio de 500 metros equivale directamente a r = 500 en las unidades del sistema de coordenadas.

### Bounding Box e Interseccion con Circulo

Cada nodo del Quadtree delimita su region mediante un rectangulo definido por su centro `(cx, cy)` y sus semiancho y semialto `(hw, hh)`. Para determinar si una region debe explorarse durante un Range Search, se calcula la distancia desde el centro del circulo de consulta al punto mas cercano del rectangulo:

```
nearest_x = clamp(cx_query, cx - hw, cx + hw)
nearest_y = clamp(cy_query, cy - hh, cy + hh)
dist^2    = (cx_query - nearest_x)^2 + (cy_query - nearest_y)^2
```

Si `dist^2 <= r^2`, el bounding box intersecta el circulo y la rama debe explorarse. En caso contrario se poda la rama completa sin examinar ninguno de sus puntos.

### Criterio de Subdivision

Un nodo hoja se subdivide en cuatro cuadrantes (NE, NW, SE, SW) cuando acumula mas de `MAX_CAPACITY` puntos (configurado en 8) y no ha alcanzado la profundidad maxima `MAX_DEPTH` (configurado en 20). Al subdividirse, todos los puntos existentes se redistribuyen entre los cuatro hijos segun su posicion geografica.

### Range Search

Algoritmo recursivo que recorre el arbol descartando ramas mediante la interseccion bounding box - circulo. Solo cuando se llega a un nodo hoja se evalua la distancia euclidiana de cada punto al centro de consulta.

```
Complejidad promedio: O(log N + k)
k = numero de puntos encontrados dentro del radio
```

### Nearest Neighbor

Algoritmo de busqueda del vecino mas cercano con poda de ramas. Mantiene una variable `best = [punto, distancia]` actualizada durante el recorrido. En cada nodo, si la distancia minima posible al bounding box es mayor o igual que la mejor distancia conocida, la rama se descarta. Los hijos se visitan en orden ascendente de distancia al centro de la consulta para maximizar la efectividad de la poda.

### Fuerza Bruta

Recorre la lista completa de N puntos y evalua la distancia euclidiana para cada uno.

```
Complejidad: O(N) por consulta, sin posibilidad de poda.
```

---

## Secciones del Codigo

### Estructuras base

- `Point`: namedtuple con campos `x`, `y` e `id`. Representa un punto de entrega.
- `BoundingBox`: define el rectangulo delimitador de un nodo. Contiene los metodos `contains()`, `intersects_circle()` e `intersects_box()`.

### Clase QuadtreeNode

Nodo individual del arbol. Contiene:

- `subdivide()`: divide el nodo en cuatro hijos y redistribuye los puntos existentes.
- `insert()`: inserta un punto en el nodo o en el hijo correspondiente, subdividiendo si es necesario.
- `range_search()`: busqueda recursiva por radio con poda de bounding box.
- `nearest_neighbor()`: busqueda recursiva del vecino mas cercano con poda por distancia minima.
- `collect_nodes()`: recolecta todos los nodos del arbol para su visualizacion.

### Clase Quadtree

Wrapper de alto nivel que encapsula el nodo raiz y expone la interfaz publica: `insert()`, `range_search()`, `nearest_neighbor()` y `stats()`.

### Funciones de Fuerza Bruta

- `brute_range_search()`: filtra la lista completa de puntos por distancia al centro de consulta.
- `brute_nearest_neighbor()`: recorre la lista completa y retorna el punto con menor distancia euclidiana.

### Generacion de Datos

`generate_points()` genera N puntos con distribucion mixta que simula una ciudad real: 70% en 8 clusters gaussianos (zonas urbanas densas) y 30% distribuidos aleatoriamente en toda el area.

### Verificacion de Correctitud

`verify_correctness()` ejecuta consultas aleatorias y compara los resultados del Quadtree contra la fuerza bruta, verificando que los conjuntos de puntos encontrados sean identicos y que las distancias al vecino mas cercano coincidan con una precision de 1e-6 metros.

### Benchmark

`benchmark()` mide el tiempo promedio de respuesta en milisegundos para ambos metodos sobre distintos tamanos de datos. Ejecuta 30 consultas aleatorias por tamano y promedia los tiempos.

### Visualizaciones

- `plot_range_search()`: genera dos vistas de la busqueda por radio: el arbol completo con la region de busqueda y un zoom sobre los puntos encontrados.
- `plot_nearest_neighbor()`: muestra multiples consultas con lineas que conectan cada punto de consulta con su vecino mas proximo y la distancia en metros.
- `plot_benchmark()`: grafica comparativa en escala logaritmica del tiempo de respuesta de ambos metodos en funcion del numero de puntos.

---

## Resultados del Analisis Comparativo

### Correctitud

Las 8 consultas de verificacion ejecutadas pasaron correctamente para ambas operaciones, confirmando que el Quadtree produce resultados identicos a la fuerza bruta.

### Tabla de Tiempos

| N puntos | FB Range (ms) | QT Range (ms) | Speedup Range | FB NN (ms) | QT NN (ms) | Speedup NN |
|---:|---:|---:|:---:|---:|---:|:---:|
| 100 | 0.021 | 0.012 | 1.7x | 0.019 | 0.018 | 1.1x |
| 500 | 0.106 | 0.022 | 4.9x | 0.098 | 0.022 | 4.4x |
| 1.000 | 0.198 | 0.026 | 7.6x | 0.188 | 0.022 | 8.5x |
| 2.000 | 0.530 | 0.048 | 11.1x | 0.376 | 0.028 | 13.6x |
| 5.000 | 1.061 | 0.075 | 14.1x | 1.041 | 0.495 | 2.1x |
| 10.000 | 2.481 | 0.135 | 18.4x | 2.274 | 0.046 | 49.0x |
| 25.000 | 5.949 | 0.292 | 20.4x | 5.325 | 0.050 | 107x |
| 50.000 | 12.327 | 0.410 | 30.0x | 11.333 | 0.049 | 232x |

### Desde que tamano el Quadtree supera a la Fuerza Bruta

El Quadtree comienza a superar a la fuerza bruta desde aproximadamente 100 puntos, tanto en Range Search como en Nearest Neighbor. Esto ocurre porque incluso con ese volumen de datos el arbol ya tiene suficiente estructura para evitar comparaciones innecesarias, y ese ahorro supera el pequeno costo de navegar los nodos.

La fuerza bruta usa una lista y recorre todos los N puntos en cada consulta con complejidad O(N). El Quadtree poda ramas enteras y opera en complejidad promedio O(log N), lo que lo hace mucho mas escalable a medida que N crece.

### Observaciones Clave

- Range Search: la ventaja crece de forma sostenida. A 50.000 puntos el Quadtree es 30 veces mas rapido.
- Nearest Neighbor: la poda es aun mas efectiva. A 50.000 puntos el Quadtree es 232 veces mas rapido, ya que en cuanto encuentra un buen candidato descarta ramas enteras.
- El costo de construccion del arbol (61 ms para 10.000 puntos) se amortiza rapidamente cuando se realizan multiples consultas sobre el mismo conjunto de datos, que es el caso tipico en logistica real.
- La fuerza bruta tiene complejidad lineal O(N): duplicar los puntos duplica exactamente el tiempo de respuesta.
- El Quadtree tiene complejidad logaritmica O(log N): pasar de 10.000 a 50.000 puntos solo incrementa su tiempo de forma marginal.

### Conclusion

Para un sistema de logistica urbana con 10.000 puntos de entrega, el Quadtree responde consultas de radio en 0.13 ms frente a los 2.5 ms de la fuerza bruta. Si el sistema atiende 1.000 consultas por segundo, la diferencia es entre usar el 13% del procesador o saturarlo al 250%. A mayor escala de la ciudad y mayor numero de puntos, la ventaja del Quadtree se vuelve indispensable para mantener la eficiencia del sistema.
