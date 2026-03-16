# Sistema de Búsqueda de Estudiantes — Listas, ABB, B y B+

Este código implementa y compara tres estructuras de datos de árboles de búsqueda aplicadas a un sistema de búsqueda de estudiantes. Permite medir y comparar los tiempos de ejecución reales de cada estructura frente a una búsqueda lineal en lista, eliminando el ruido externo del sistema operativo para obtener mediciones limpias y confiables.

> Este archivo forma parte de la carpeta `arboles` del repositorio.

---

##  Tecnología

- **Lenguaje:** Python 3
- **Entorno:** Google Colab (Jupyter Notebook)
- **Librerías utilizadas:** `random`, `time`, `statistics`, `sys` — todas nativas de Python, no requieren instalación adicional.

---

## ¿Qué hace este código?

### 1. Generación de datos
Genera automáticamente **10,000 estudiantes únicos** con datos aleatorios:
- **ID:** número entre 1,000 y 99,999 (garantizado único)
- **Nombre:** seleccionado aleatoriamente de una lista predefinida
- **Promedio:** valor entre 6.0 y 10.0 con un decimal

Los estudiantes se insertan en **orden aleatorio** para garantizar un comportamiento realista y balanceado en los árboles.

### 2. Estructuras implementadas
El código construye las siguientes estructuras con los 10,000 estudiantes y mide el tiempo de **100 búsquedas aleatorias** en cada una:

| Estructura | Descripción | Complejidad de búsqueda |
|------------|-------------|------------------------|
| **Lista lineal** | Recorre uno por uno hasta encontrar el ID | O(n) |
| **ABB con AVL** | Árbol binario de búsqueda con autobalanceo | O(log n) |
| **Árbol B** | Árbol balanceado multiway, datos en todos los nodos | O(log n) |
| **Árbol B+** | Variante del B, datos solo en hojas enlazadas | O(log n) |

### 3. Medición de tiempos limpia
Para obtener tiempos de ejecución confiables y sin interferencia externa, se implementó una función de medición robusta que:
- Usa `time.perf_counter()` — reloj de alta resolución que mide solo tiempo de CPU
- Repite cada experimento **15 veces**
- **Descarta el 20% inferior y el 20% superior** de los tiempos para eliminar picos causados por el sistema operativo, el recolector de basura, accesos a disco o el scheduler
- Retorna la **mediana** de los tiempos restantes

### 4. Resultados
Al finalizar, el código imprime una tabla comparativa mostrando cuánto tardó cada estructura en las 100 búsquedas y cuántas veces más rápida fue respecto a la búsqueda lineal. También lista los primeros 5 estudiantes en orden de ID para verificar que los tres árboles están correctamente construidos.

---

## ¿Cómo ejecutarlo?

1. Abrir el archivo en **Google Colab**
2. Ejecutar la celda — genera los estudiantes, construye los tres árboles y muestra la comparación de tiempos

---

##  Estructura del código
```
├── Generación de 10,000 estudiantes aleatorios
├── Función medir_tiempo_limpio()
├── Búsqueda lineal en lista
├── ABB con balanceo AVL
│   ├── clase Nodo
│   ├── rotaciones (rotar_der, rotar_izq)
│   ├── balancear()
│   ├── insertar_abb()
│   ├── buscar_abb()
│   └── listar_abb()
├── Árbol B
│   ├── clase NodoB
│   └── clase ArbolB
│       ├── buscar()
│       ├── insertar()
│       ├── _insertar_no_lleno()
│       ├── _dividir_hijo()
│       └── listar()
├── Árbol B+
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

##  Ejemplo de salida esperada
```
10000 estudiantes generados.
Primeros 3: [(44599, 'Genesis'), (69081, 'Esteban'), (7109, 'Melany')]

 Lista     : 0.043500 segundos (100 búsquedas, mediana limpia)
 ABB (AVL) : 0.000312 segundos (100 búsquedas, mediana limpia)
 B         : 0.000289 segundos (100 búsquedas, mediana limpia)
 B+        : 0.000301 segundos (100 búsquedas, mediana limpia)

=======================================================
   Resultados para 10000 estudiantes:
   (tiempos mediana limpia — sin ruido del SO)
=======================================================
  Lista  : 0.043500 segundos
  ABB    : 0.000312 segundos  (¡139x más rápido que lista!)
  B      : 0.000289 segundos  (¡150x más rápido que lista!)
  B+     : 0.000301 segundos  (¡144x más rápido que lista!)
=======================================================
   El más rápido fue: B
=======================================================
```

---

## Cocreado con Inteligencia Artificial

Este código fue **cocreado con el apoyo de Inteligencia Artificial**, utilizando IA como herramienta de asistencia en el diseño, desarrollo, optimización y documentación de las implementaciones.

> El uso de IA no reemplaza el criterio académico ni la comprensión del desarrollador, sino que actúa como apoyo en el proceso de creación.

---

## Autores

Maria Segura Ortiz
