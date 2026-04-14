# Árbol KD para búsqueda espacial de puntos de entrega

## Descripción

Este proyecto implementa desde cero un **árbol KD (KD-Tree)** para resolver un problema de búsqueda espacial en un contexto de entregas. El objetivo es identificar cuáles de **10.000 puntos geográficos** se encuentran dentro de un radio de **500 metros** respecto a un punto de consulta. Para evaluar su eficiencia, se compara el rendimiento del árbol KD con un método de **fuerza bruta**.

El problema principal se trabaja en **2 dimensiones**, usando coordenadas de **latitud y longitud**. Además, se incluye una prueba adicional en **50 dimensiones** para mostrar cómo cambia el comportamiento del algoritmo cuando aumenta la dimensionalidad.

---

## Problema que resuelve

La tarea principal es una **búsqueda por rango**: dado un punto geográfico y un radio fijo, encontrar todos los puntos que se encuentren dentro de esa distancia.

> **¿Qué puntos de entrega están a menos de 500 metros de una ubicación dada?**

Resolver esto con fuerza bruta implica revisar los **10.000 puntos** y calcular la distancia desde el punto de consulta hacia cada uno. En cambio, el árbol KD organiza los datos espacialmente para descartar regiones completas que no pueden contener resultados, reduciendo el número de comparaciones necesarias.

---

## Fundamento del árbol KD

Un **KD-Tree** es una estructura de datos para organizar puntos en un espacio de `k` dimensiones. Su idea principal es dividir el espacio recursivamente usando uno de los ejes en cada nivel del árbol.

El eje de partición se define con:

$$\text{axis} = \text{depth} \bmod k$$

En este proyecto, como el caso principal está en **2D**, el árbol alterna entre:

- eje `0`: latitud  
- eje `1`: longitud

Cada nodo almacena:

- el punto
- el eje usado para dividir
- el subárbol izquierdo
- el subárbol derecho

```python
class KDNode:
    def __init__(self, point, axis):
        self.point = point
        self.axis = axis
        self.left = None
        self.right = None
```

---

##  Cómo se construyó el árbol

La implementación principal se apoya en dos clases:

### `KDNode`
Representa cada nodo del árbol y guarda el punto, el eje y las referencias a sus hijos.

### `KDTree`
Contiene toda la lógica del proyecto:
- Construcción del árbol
- Inserción y búsqueda
- Cálculo de distancias
- Búsqueda por radio
- Comparación con fuerza bruta
- Visualización y análisis de tiempos

El radio de la Tierra usado es:

$$R = 6371 \times 10^3 \text{ metros}$$

---

## Construcción balanceada y cálculo de la mediana

El árbol se construye mediante una estrategia **balanceada por mediana**, lo que permite mantener una estructura más equilibrada y mejorar la eficiencia de las búsquedas.

**Procedimiento:**
1. Elegir el eje según la profundidad
2. Ordenar los puntos respecto a ese eje
3. Tomar el punto central como raíz del subárbol
4. Construir recursivamente el subárbol izquierdo con la mitad izquierda
5. Construir recursivamente el subárbol derecho con la mitad derecha

La mediana se calcula como:

$$\text{median\\_index} = \left\lfloor \frac{n}{2} \right\rfloor$$

```python
def build_kdtree(points, depth=0):
    if not points:
        return None

    k = len(points[0])
    current_axis = depth % k

    points.sort(key=lambda p: p[current_axis])
    median_index = len(points) // 2

    node = KDNode(points[median_index], current_axis)
    node.left = build_kdtree(points[:median_index], depth + 1)
    node.right = build_kdtree(points[median_index + 1:], depth + 1)

    return node
```

---

## Datos utilizados

Se generan **10.000 puntos aleatorios** en una zona geográfica de **Buenos Aires, Argentina**, dentro de los rangos:

| Coordenada | Rango |
|------------|-------|
| Latitud    | `-34.7` a `-34.5` |
| Longitud   | `-58.6` a `-58.3` |

Radio de búsqueda fijo: **500 metros**

```python
import random

points = [
    (random.uniform(-34.7, -34.5), random.uniform(-58.6, -58.3))
    for _ in range(10000)
]

query_point = (
    random.uniform(-34.7, -34.5),
    random.uniform(-58.6, -58.3)
)

radius_meters = 500
```

---

##Distancia utilizada: Haversine

Como el problema es geográfico, se implementa la **fórmula de Haversine**, que calcula la distancia entre dos puntos sobre la superficie terrestre.

$$\Delta \phi = \phi_2 - \phi_1$$

$$\Delta \lambda = \lambda_2 - \lambda_1$$

$$a = \sin^2\left(\frac{\Delta \phi}{2}\right) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\Delta \lambda}{2}\right)$$

$$c = 2 \arctan2(\sqrt{a}, \sqrt{1-a})$$

$$d = R \cdot c$$

```python
import math

def haversine_distance(point1, point2):
    lat1, lon1 = map(math.radians, point1)
    lat2, lon2 = map(math.radians, point2)

    dlat = lat2 - lat1
    dlon = lon2 - lon1

    a = math.sin(dlat / 2) ** 2 + math.cos(lat1) * math.cos(lat2) * math.sin(dlon / 2) ** 2
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))

    R = 6371 * 10**3
    return R * c
```

---

## Búsqueda por radio en el árbol KD

La búsqueda principal se realiza con `find_points_in_radius(query_point, radius_meters)` y tiene dos partes:

### 1. Verificación exacta
En cada nodo visitado se calcula la distancia Haversine entre el punto almacenado y el punto de consulta. Si la distancia es menor o igual al radio, el punto se agrega al resultado.

### 2. Poda espacial
Para evitar recorrer ramas innecesarias, el radio en metros se convierte a una aproximación angular en grados:

angular_radius_lat = degrees(r / R)

angular_radius_lon = degrees(r / (R · cos(φ)))

```python
def find_points_in_radius(node, query_point, radius_meters, results=None):
    if node is None:
        return results

    if results is None:
        results = []

    distance = haversine_distance(node.point, query_point)
    if distance <= radius_meters:
        results.append(node.point)

    axis = node.axis
    lat = math.radians(query_point[0])
    R = 6371 * 10**3

    angular_radius_lat = math.degrees(radius_meters / R)
    angular_radius_lon = math.degrees(radius_meters / (R * math.cos(lat)))

    delta = angular_radius_lat if axis == 0 else angular_radius_lon

    if query_point[axis] - delta <= node.point[axis]:
        find_points_in_radius(node.left, query_point, radius_meters, results)

    if query_point[axis] + delta >= node.point[axis]:
        find_points_in_radius(node.right, query_point, radius_meters, results)

    return results
```

---

## Fuerza bruta

La estrategia de fuerza bruta recorre **todos** los puntos sin aprovechar ninguna estructura espacial:

```python
def brute_force_search(points, query_point, radius_meters):
    results = []
    for point in points:
        if haversine_distance(point, query_point) <= radius_meters:
            results.append(point)
    return results
```

---

## Comparación de rendimiento

Ambos métodos encuentran los **mismos puntos**, confirmando la correctitud de la implementación. La diferencia está en el tiempo de ejecución:

| Método | Tiempo aproximado |
|--------|-------------------|
| KD-Tree | `0.000338 s` |
| Fuerza bruta | `0.016816 s` |

```python
import time

start_kd = time.time()
kd_results = tree.find_points_in_radius(query_point, radius_meters)
end_kd = time.time()

start_brute = time.time()
brute_results = brute_force_search(points, query_point, radius_meters)
end_brute = time.time()

kd_time = end_kd - start_kd
brute_time = end_brute - start_brute
```

---

## KD-Tree vs Fuerza bruta en más dimensiones

El notebook incluye una prueba adicional en **50 dimensiones** con 10 puntos y distancia euclidiana. En este caso, la fuerza bruta resulta **ligeramente más rápida** porque:

- Hay muy pocos datos
- Construir y recorrer el árbol introduce sobrecarga
- En alta dimensionalidad la poda pierde efectividad
- lo que confirm que el arbol kd es mucho mas eficiente para dimensiones bajas y muchos datos

> El KD-Tree **no siempre es la mejor opción**: funciona muy bien en baja dimensión, pero puede perder ventaja en espacios de muchas dimensiones.

---

## Visualizaciones

El notebook incluye visualizaciones para validar y explicar los resultados.

### Vista general
- Punto de consulta
- Puntos dentro del radio
- Puntos fuera del radio
- Circulo punteado representando la zona de búsqueda

### Vista ampliada
Zoom alrededor del punto consultado para observar los vecinos encontrados dentro del radio de 500 metros.

```python
import matplotlib.pyplot as plt
from matplotlib.patches import Ellipse

plt.figure(figsize=(10, 8))
plt.scatter(x_out, y_out, s=10, alpha=0.4, label='Fuera del radio')
plt.scatter(x_in, y_in, s=20, label='Dentro del radio')
plt.scatter(query_point[1], query_point[0], s=100, marker='x', label='Punto de consulta')

ellipse = Ellipse(
    xy=(query_point[1], query_point[0]),
    width=2 * angular_radius_lon,
    height=2 * angular_radius_lat,
    fill=False,
    linestyle='--'
)

plt.gca().add_patch(ellipse)
plt.legend()
plt.xlabel('Longitud')
plt.ylabel('Latitud')
plt.title('Búsqueda por radio con KD-Tree')
plt.show()
```

---

## Tecnologías utilizadas

| Librería | Uso |
|----------|-----|
| `math` | Cálculos trigonométricos |
| `random` | Generación de datos |
| `time` | Medición de tiempos |
| `numpy` | Operaciones numéricas |
| `matplotlib` | Visualizaciones |
| `seaborn` | Estilo de gráficos |
| `matplotlib.patches.Ellipse` | Zona de búsqueda visual |

Desarrollado en **Python** dentro de un entorno **Google Colab / Jupyter Notebook**.

---

## Conclusiones

- El **KD-Tree es muy eficiente** para búsquedas espaciales en baja dimensión con muchos datos.
- En el problema principal (10.000 puntos, 2D), el KD-Tree superó claramente a la fuerza bruta gracias a la **poda espacial**.
- En una prueba con **50 dimensiones y pocos puntos**, la fuerza bruta fue más rápida, evidenciando el problema con la dimensionalidad

### Resumen de decisiones

| Aspecto | Decisión |
|---------|----------|
| Dimensiones principales | 2D (latitud y longitud) |
| Construcción del árbol | Balanceada por mediana |
| Métrica de distancia | Haversine |
| Datos de prueba | 10.000 puntos en Buenos Aires |
| Radio de búsqueda | 500 metros |

> **La elección entre KD-Tree y fuerza bruta depende de la dimensionalidad y del tamaño del conjunto de datos.**

