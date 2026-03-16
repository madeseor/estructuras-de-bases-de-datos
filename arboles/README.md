# Sistema de Búsqueda de Estudiantes — Listas, ABB, B y B+

Este código implementa y compara tres estructuras de datos de árboles de búsqueda aplicadas a un sistema de búsqueda de estudiantes. Permite medir y comparar los tiempos de ejecución reales de cada estructura frente a una búsqueda en lista, eliminando el ruido externo del sistema operativo para obtener mediciones limpias y confiables.

> Este archivo forma parte de la carpeta `arboles` del repositorio.

---

##  Tecnología

- **Lenguaje:** Python 3
- **Entorno:** Google Colab (Jupyter Notebook)
- **Librerías utilizadas:** `random`, `time`, `statistics`, `sys`, `bisect` — todas nativas de Python, no requieren instalación adicional.

---

##  ¿Qué hace este código?

### 1. Generación de datos
Genera automáticamente **n estudiantes únicos** con datos aleatorios:
- **ID:** número entre 1,000 y 999,999 — el rango amplio garantiza que el `while` siempre encuentre IDs disponibles para completar la cantidad solicitada
- **Nombre:** seleccionado aleatoriamente de una lista predefinida
- **Promedio:** valor entre 6.0 y 10.0 con un decimal

Se usa un diccionario como estructura de generación porque no permite claves duplicadas, garantizando unicidad sin necesidad de limpieza posterior. El orden de inserción es configurable: aleatorio u ordenado por ID.

### 2. Estructuras implementadas
El código construye las siguientes estructuras con los n estudiantes y mide el tiempo de **m búsquedas aleatorias** en cada una:

| Estructura | Descripción | Complejidad de búsqueda |
|------------|-------------|------------------------|
| **Lista** | Búsqueda binaria (ordenada) o lineal (desordenada) | O(log n) / O(n) |
| **ABB con AVL** | Árbol binario de búsqueda con autobalanceo | O(log n) |
| **Árbol B** | Árbol balanceado multiway, datos en todos los nodos | O(log n) |
| **Árbol B+** | Variante del B, datos solo en hojas enlazadas | O(log n) |

### 3. Comportamiento de la lista según el orden de los datos
La lista no usa siempre el mismo algoritmo de búsqueda — se adapta según el orden de inserción:
- **Datos ordenados** → búsqueda **binaria** con `bisect` → O(log n), puede competir con los árboles
- **Datos desordenados** → búsqueda **lineal** → O(n), significativamente más lenta

Esto refleja el comportamiento real: ordenar los datos previamente tiene un impacto directo y medible en el rendimiento de búsqueda.

### 4. Medición de tiempos limpia — solo búsquedas en RAM
Para obtener tiempos de ejecución confiables, se tomaron las siguientes decisiones:

**Separación construcción / medición:** cada estructura se construye completamente antes de que empiece cualquier medición. Los tiempos reflejan únicamente el costo de la búsqueda, no el de construir el árbol.

**Carga explícita en RAM:** todas las estructuras se mantienen en memoria durante las mediciones. No hay accesos a disco durante las búsquedas.

**Función `medir_tiempo_limpio`:**
- Usa `time.perf_counter()` — reloj de alta resolución que mide solo tiempo de CPU, sin verse afectado por cambios del reloj del sistema operativo
- Repite cada experimento **15 veces**
- Descarta el **20% inferior y 20% superior** de los tiempos, eliminando picos causados por el sistema operativo, el recolector de basura, accesos a disco o el scheduler
- Retorna la **mediana** de los tiempos restantes

### 5. Optimizaciones aplicadas al árbol B
El árbol B fue diseñado originalmente para disco duro, donde leer un bloque grande es más barato que muchos accesos pequeños. En RAM esto genera overhead. Se aplicaron dos optimizaciones:

**Optimización 1 — búsqueda binaria dentro de cada nodo:** la implementación original recorría las claves de cada nodo una por una O(T). Con T=50 cada nodo tiene hasta 99 claves. Se reemplazó por `bisect`, reduciendo la búsqueda interna de O(T) lineal a O(log T) binaria.

**Optimización 2 — lista de IDs precalculada (`self.ids`):** la optimización anterior aún creaba una lista temporal `[c[0] for c in nodo.claves]` en cada nodo visitado durante la búsqueda. Se agregó `self.ids` al nodo para guardar solo los IDs sincronizados con `self.claves`, eliminando completamente la creación de listas temporales en tiempo de búsqueda.

### 6. Optimización aplicada al árbol B+
Los nodos internos del B+ también tenían búsqueda lineal O(T) para decidir en qué hijo bajar. Se reemplazó por `bisect`, reduciendo esto a O(log T). Las hojas no necesitan esta optimización porque el B+ ya guarda sus claves como IDs directos (no tuplas), por lo que `bisect` opera sobre `self.claves` sin overhead adicional.

### 7. Resultados
Al finalizar, el código imprime una tabla comparativa con los tiempos de cada estructura y el ganador. También lista los primeros 5 estudiantes en orden de ID en los tres árboles para verificar que están correctamente construidos.

---

## ¿Cómo ejecutarlo?

1. Abrir el archivo en **Google Colab**
2. Para probar con datos **ordenados**: dejar activo `estudiantes.sort(...)` y `ORDEN_INSERCION = "ORDENADO"`
3. Para probar con datos **aleatorios**: comentar el sort y activar `random.shuffle(estudiantes)` y `ORDEN_INSERCION = "ALEATORIO"`
4. Ejecutar la celda

---

## Estructura del código
```
├── Generación de 100,000 estudiantes aleatorios
├── Función medir_tiempo_limpio()
├── Lista con búsqueda binaria (ordenada) o lineal (desordenada)
├── ABB con balanceo AVL
│   ├── clase Nodo
│   ├── rotaciones (rotar_der, rotar_izq)
│   ├── balancear()
│   ├── insertar_abb()
│   ├── buscar_abb()
│   └── listar_abb()
├── Árbol B (con self.ids precalculado + bisect interno)
│   ├── clase NodoB
│   └── clase ArbolB
│       ├── buscar()
│       ├── insertar()
│       ├── _insertar_no_lleno()
│       ├── _dividir_hijo()
│       └── listar()
├── Árbol B+ (con bisect en nodos internos)
│   ├── clase NodoBP
│   └── clase ArbolBPlus
│       ├── buscar()
│       ├── insertar()
│       ├── _ins()
│       ├── _div_hoja()
│       ├── _div_int()
│       └── listar()
└── Resultados finales y listado de primeros 5 por árbol
```

---

## Ejemplo de salida esperada
```
 100000 estudiantes generados.
Primeros 3: [(1015, 'Melany'), (1022, 'Santiago'), (1033, 'Ester')]
Orden de inserción: ORDENADO

 Lista     : 0.000557 segundos (400 búsquedas)
 ABB (AVL) : 0.001451 segundos (400 búsquedas)
 B         : 0.007038 segundos (400 búsquedas)
 B+        : 0.000528 segundos (400 búsquedas)

=======================================================
   Resultados para 100000 estudiantes:
   Orden de inserción: ORDENADO
   (tiempos mediana limpia — sin ruido del SO)
=======================================================
  Lista  : 0.000557 segundos
  ABB    : 0.001451 segundos
  B      : 0.007038 segundos
  B+     : 0.000528 segundos
=======================================================
   El más rápido fue: B+
=======================================================
```

---

## Cocreado con Inteligencia Artificial

Este código fue **cocreado con el apoyo de Inteligencia Artificial**, utilizando IA como herramienta de asistencia en el diseño, desarrollo, optimización y documentación de las implementaciones.

> El uso de IA no reemplaza el criterio académico ni la comprensión del desarrollador, sino que actúa como apoyo en el proceso de creación.

---

## Autores

Maria Segura Ortiz
