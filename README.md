# 🧬 Resolución del TSP mediante Algoritmos Genéticos
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![Jupyter Notebook](https://img.shields.io/badge/jupyter-%23FA0F00.svg?style=for-the-badge&logo=jupyter&logoColor=white)
![NumPy](https://img.shields.io/badge/numpy-%23013243.svg?style=for-the-badge&logo=numpy&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/matplotlib-6A5ACD?style=for-the-badge)

> **Asignatura:** Inteligencia Artificial para la Ciencia de Datos        
> **Autor:** [Javier Ruano Hernández](https://github.com/javierruanohdez)

---

## 📋 Introducción
Este proyecto aborda el **Problema del Viajante de Comercio (TSP)** aplicando técnicas de **Computación Evolutiva**. El objetivo es encontrar una ruta eficiente que conecte 48 ciudades, minimizando la distancia total recorrida mediante la simulación de algoritmos genéticos, recreando operaciones biológicas como: selección, cruce y mutación.

### 🎯 Objetivo del Estudio
Analizar el comportamiento paramétrico del algoritmo frente a un dataset de 48 ciudades y sus coordenadas, buscando alcanzar una **solución óptima (distancia ≤ 35.000,00)**.

---

## 🛠️ Configuración del Experimento
La ejecución se ha estructurado en bloques funcionales dentro de un entorno **Jupyter Notebook**, facilitando el ajuste directo de los siguientes hiperparámetros:

| Parámetro | Descripción |
| :--- | :--- |
| `population_size` | Número de individuos (rutas) en cada generación. |
| `generations` | Límite máximo de iteraciones del proceso evolutivo. |
| `mutation_rate` | Probabilidad de alteración genética para mantener la diversidad. |
| **Criterio de Parada** | Distancia total $\le 35.000,00$ o límite de generaciones. |

---

## 📊 Estudio Paramétrico
En este apartado analizaremos la convergencia del algoritmo genético original variando el **tamaño de la población** `[100, 250, 500, 750, 1000]` y la **tasa de mutación** `[0.2, 0.4, 0.6]`.

El algoritmo original consta de una función cruce (crossover) y una función mutación que intentaremos mejorar posteriormente, pero veamos primero el funcionamiento de las ya definidas:
* **Crossover de Orden 1 (OX1):** Funciona como un "copia y pega". Elegimos al azar un segmento de la ruta del primer padre (parent1), lo copiamos en el hijo, y el resto lo rellenamos con el genoma del segundo padre (parent2) siguiendo su orden. En este caso, se sigue una **lógica toroidal**, es decir, tratamos el genoma como un anillo infinito, puesto que al fin y al cabo representa una ruta (esto nos ayuda a que no se pierda la secuencia de las ciudades).

* **Mutación Swap:** Conocida también como mutación por intercambio, y por ser una de las formas más simples de recombinación en el ámbito de la computación evolutiva. Pero, ¿cómo funciona? Elegimos dos alelos al azar del genoma del hijo, o lo que es lo mismo, dos ciudades al azar de la ruta, y las intercambiamos. 

### 📈 Análisis de Convergencia
![Análisis de Convergencia Algoritmo Original](results/original_algorithm_tsp_convergence.png)

**Observaciones clave:**

1. **Naturaleza Estocástica:** La gráfica revela un comportamiento **no lineal** y **altamente volátil**, donde el algoritmo original (OX1 + Swap) muestra una dependencia crítica del azar. No se observa una progresión suave, sino más bien cruces abruptos entre las diferentes tasas de mutación. Lo que sugiere que el algoritmo cae fácilmente en **óptimos locales**, de los cuales no siempre logra salir antes del límite de generaciones (2000).

2. **El "Caos Beneficioso" de la Mutación Alta:**
   Sorprendentemente, la **Tasa 0.6 (Púrpura)** es la que logra converger en más escenarios (3 de 5). 
   * **Interpretación:** En un espacio de búsqueda tan amplio (48 ciudades), una mutación baja (0.2) puede hacer que el algoritmo sea demasiado "conservador", quedando atrapado en rutas poco óptimas. En cambio, una mutación del 60% introduce suficiente ruido para escapar de esas pésimas soluciones, aunque el proceso sea errático. Es en esencia, una búsqueda aleatoria que compensa las carencias del operador OX1.

3. **Sensibilidad al Tamaño de Población:**
   * **Poblaciones de 750:** Presentan los mejores picos de velocidad. Esto indica que existe un punto de equilibrio, donde hay suficiente diversidad sin llegar al ruido excesivo de una población de 1000, donde la presión selectiva se diluye.
   * **El muro de las 2000 generaciones:** El hecho de que varias configuraciones (incluyendo poblaciones grandes) choquen contra este límite confirma que la recombinación por intercambio aleatorio (Swap) es insuficiente para optimizar rutas de 48 ciudades, a diferencia de cuando eran sólo 5 de ellas. Un buen símil sería que, intercambiar solo dos ciudades es como intentar resolver un cubo de Rubik moviendo las piezas al azar: funciona por pura probabilidad.
---

## 🚀 Propuesta de Mejora

Tras analizar la inestabilidad del algoritmo original, proponemos una transición de un enfoque basado en la **ordenación** a uno basado en la **adyacencia**. En el TSP, la calidad de una ruta no depende de si una ciudad es la tercera o la cuarta en dicha ruta, sino que lo que verdaderamente importa es quiénes son sus vecinos (o bordes).

### 1. Crossover de borde (Edge Recombination Crossover)
El operador **ERX** se centra exclusivamente en preservar las uniones (aristas) presentes en los padres, siendo el estándar para problemas donde la adyacencia es crítica.

* **Mecánica del Algoritmo:**
    1.  **Tabla de Bordes (Edge Table):** Se construye una lista de adyacencia $N(v)$ para cada ciudad $v$, recopilando todos los vecinos que tiene en ambos padres.
    2.  **Selección Inteligente:** Se elige una ciudad inicial. Para las siguientes, el algoritmo no elige al azar, sino que busca en $N(c)$ la ciudad $u$ con el **menor número de bordes restantes** ($|N(u)|$ mínimo).
    3.  **Gestión de Restricciones:** Al priorizar nodos con menos opciones (nodos más restringidos), el ERX minimiza la necesidad de introducir bordes aleatorios ("dead-ends"), logrando que el hijo herede entre el **95% y 99%** de las aristas de sus padres.

### 2. Mutación por inversión (Inversion Mutation)
Sustituimos el *Swap* por la **Mutación por inversión**, un operador mucho más sutil y eficiente para problemas de rutas.

* **Funcionamiento:** Se seleccionan dos puntos de corte aleatorios y se invierte el subsegmento comprendido entre ellos.
* **Ventaja Competitiva:** Mientras que el *Swap* rompe hasta 4 uniones de adyacencia, la **Inversión solo rompe 2**. Esto permite explorar nuevas rutas sin destruir masivamente la estructura de la solución que el crossover ha tardado generaciones en construir.

---

### ⚖️ Justificación de la Mejora
Consideramos que esta combinación superará significativamente al algoritmo original por tres razones fundamentales:

1.  **Preservación de la Herencia:** El algoritmo original (OX1) es excelente para preservar el orden relativo, pero el TSP es un problema de **distancias entre nodos adyacentes**. ERX está diseñado matemáticamente para no perder esas conexiones valiosas.
2.  **Reducción del Ruido Estocástico:** La inestabilidad observada en el análisis anterior se debía a que la mutación *Swap* era demasiado destructiva. La **Inversión** actúa como una "búsqueda local" más inteligente, permitiendo optimizar tramos de la ruta sin desordenar el resto.
3.  **Evasión de Óptimos Locales:** Al utilizar la **Tabla de Bordes**, el algoritmo tiene una "hoja de ruta" mucho más clara de qué conexiones son prometedoras, lo que reduce drásticamente la dependencia del azar y evita que el sistema se estanque en las soluciones subóptimas vistas anteriormente.

### 📈 Análisis de Convergencia
*(Imagen de convergencia)*

**Observaciones clave:**
1.  **Relación Población/Generaciones:** 
2.  **Impacto de la Mutación:**


---

## 🧪 Conclusiones


---

## 📦 Instrucciones de Ejecución
```bash
# Clonar el repositorio
git clone [https://github.com/pablo-fernandez-lopez/IACD_2025_26_EP2_javier_ruano_hernandez.git](https://github.com/pablo-fernandez-lopez/IACD_2025_26_EP2_javier_ruano_hernandez.git)

# Crear un entorno virtual (Recomendado)
python -m venv venv

# Activar el entorno
## En Windows:
venv\Scripts\activate o .\venv\Scripts\Activate.ps1

## En Linux y macOS:
source venv/bin/activate

# Instalar dependencias
pip install -r requirements.txt

# Ejecutar el estudio
jupyter notebook TSP.ipynb
```
---
