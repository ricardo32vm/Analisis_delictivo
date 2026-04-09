# QGIS Crime Analysis Plugins — UNVM

[![QGIS](https://img.shields.io/badge/QGIS-3.22+-green)](https://qgis.org)
[![Python](https://img.shields.io/badge/Python-3.9+-blue)](https://python.org)
[![License](https://img.shields.io/badge/License-GPL%20v2-orange)](LICENSE)
[![Institución](https://img.shields.io/badge/UNVM-Licenciatura%20en%20Seguridad-darkblue)](https://www.unvm.edu.ar)

Ecosistema de plugins para QGIS orientado al análisis espaciotemporal del delito, desarrollado en la **cátedra Nuevas Tecnologías Aplicadas a la Gestión de la Seguridad** de la **Licenciatura en Seguridad**, Universidad Nacional de Villa María (Argentina).

Los plugins fueron diseñados para trabajar con datos institucionales policiales de forma **completamente local**, sin transmitir información a servicios externos.

---

## Plugins disponibles

| Plugin | Descripción | Técnicas |
|--------|-------------|----------|
| [Crime Analyst CBA](#crime-analyst-cba) | Análisis y predicción de delitos | KDE, análisis temporal, predicción estacional, narrativa LLM |
| [Near Repeat CBA](#near-repeat-cba) | Victimización cercana y repetida | Knox test, Monte Carlo, KDE sobre red |
| [Rossmo CBA](#rossmo-cba) | Perfilamiento geográfico criminal | CGT (Criminal Geographic Targeting) |

---

## Crime Analyst CBA

Herramienta principal de análisis espaciotemporal. Integra seis módulos accesibles desde pestañas independientes.

**Funcionalidades:**

- **KDE histórico** — Kernel Density Estimation euclidiano con rampa de calor, filtrable por tipo de delito, distrito y período
- **Análisis temporal** — tendencia anual, índice estacional mensual, distribución por día de la semana, heatmap hora × día y comparativa por tipo de delito
- **Predicción de zonas de riesgo** — KDE ponderado por factor estacional para próxima semana o mes
- **Narrativa táctica con LLM** — informe automático en lenguaje policial argentino mediante Llama 3.2 corriendo localmente con Ollama
- **KDE sobre red vial** — Kernel Density Estimation sobre lixels de la red vial (kernel de Epanechnikov), requiere SHP de red
- **Getis-Ord Gi\*** — identificación estadística de hotspots y coldspots sobre la red vial, con niveles de confianza 90%, 95% y 99%

**Datos compatibles:**

Cualquier CSV, Excel o SHP con coordenadas geográficas. Detecta automáticamente la estructura del sistema DPAD de la Policía de la Provincia de Córdoba:

```
id, prevenible, fecha_hech, hora_hecho, calle, barrio,
distrito, comisaria, zona, cuadrantes, latitud, longitud
```

**Requisitos adicionales:**

- [Ollama](https://ollama.com) corriendo localmente
- Modelo Llama 3.2: `ollama pull llama3.2`
- SHP de red vial para las pestañas 5 y 6

---

## Near Repeat CBA

Implementación del Knox test para análisis de victimización cercana y repetida (near repeat victimization).

**Funcionalidades:**

- **Knox test con Monte Carlo** — detección de patrones near repeat con estimación del p-valor por permutación aleatoria de tiempos (999 simulaciones por defecto)
- **Análisis multibanda** — matriz de p-valores y Knox ratios equivalente al Near Repeat Calculator de Ratcliffe (Temple University)
- **4 capas de visualización** — puntos coloreados por nivel de riesgo NR, buffers de zona de influencia, líneas entre pares near repeat, heatmap de concentración
- **KDE near repeat sobre red vial** — proyección de pares near repeat sobre lixels, ponderado por nivel de riesgo
- **Gi\* NR** — Getis-Ord sobre densidad near repeat en red, identifica tramos estructuralmente críticos

**Parámetros configurables:**

- Umbral espacial (distancia máxima entre eventos, en metros)
- Umbral temporal (diferencia máxima entre fechas, en días)
- Número de simulaciones Monte Carlo
- Bandas espaciales y temporales para análisis multibanda

**Interpretación del Knox ratio:**

| Knox ratio | Interpretación |
|------------|----------------|
| < 1.0 | Sin patrón near repeat |
| 1.0–1.5 | Patrón débil |
| 1.5–3.0 | Patrón moderado a fuerte |
| > 3.0 | Patrón muy fuerte |

---

## Rossmo CBA

Implementación del modelo **Criminal Geographic Targeting (CGT)** de Kim Rossmo (1996, 2000) para perfilamiento geográfico en casos de crímenes seriales.

**Funcionalidades:**

- **Superficie de probabilidad (Jeopardy Surface)** — raster con la probabilidad estimada de que cada punto del área sea la base de operaciones del ofensor
- **Anchor point estimado** — punto de máxima probabilidad, representado como estrella en el mapa
- **CGT Hit Score** — porcentaje del área de caza que debe buscarse para localizar al ofensor, métrica estándar de evaluación del modelo
- **Evaluación retrospectiva** — si se conoce el anchor real, calcula la distancia al punto predicho y el Hit Score oficial
- **Perfil de probabilidad** — gráfico del corte horizontal en el anchor estimado

**Fórmula implementada:**

```
P(i,j) = k · Σ [φ/(|Xi-xn|+|Yj-yn|)^f + (1-φ)·B^(g-f)/(2B-|Xi-xn|-|Yj-yn|)^g]

φ = 1  si  |Xi-xn| + |Yj-yn| > B  (fuera del buffer)
φ = 0  si  |Xi-xn| + |Yj-yn| ≤ B  (dentro del buffer)
```

**Parámetros recomendados:**

| Contexto | Buffer B | f | g |
|----------|----------|---|---|
| Ciudad grande (> 1M hab.) | 1000–2000 m | 1.0 | 1.0–2.0 |
| Ciudad mediana | 500–1000 m | 1.0 | 1.0–1.5 |
| Caso Mataviejitas (CDMX) | 2000 m | 1.0 | 2.0 |
| CrimeStat (por defecto) | 0 | 1.2 | 1.2 |

**Referencia de validación:** Suárez-Meaney, T., Palomares López, A.J. & Chías Becerril, L. (2017). Predictibilidad locacional y perfilamiento geográfico en el homicidio serial con gvSIG. Caso Barraza. *Revista Mapping*, 26(182), 52–63.

---

## Instalación

### 1. Requisitos del sistema

- QGIS 3.22 o superior (recomendado: 3.44)
- Python 3.9+ (incluido con QGIS)
- Windows 10/11, Linux o macOS

### 2. Instalar dependencias Python

Desde la **Consola Python de QGIS** (Complementos → Consola Python de QGIS):

```python
import subprocess, sys

subprocess.run([
    sys.executable, "-m", "pip", "install",
    "geopandas", "scipy", "pyproj", "rasterio",
    "matplotlib", "shapely", "--user"
])
```

### 3. Instalar cada plugin

Para cada plugin:

1. Descargar el ZIP del repositorio
2. QGIS → **Complementos → Administrar e instalar complementos**
3. Pestaña **Instalar desde ZIP**
4. Seleccionar el archivo ZIP
5. Clic en **Instalar complemento**

### 4. Solo para Crime Analyst CBA — instalar Ollama

```bash
# Descargar desde https://ollama.com/download
# Luego en terminal:
ollama pull llama3.2
```

Ollama se instala como servicio en Windows y arranca automáticamente. No requiere configuración adicional.

---

## Formato de datos

Los plugins son compatibles con cualquier capa de puntos QGIS. Para carga directa desde CSV, las columnas mínimas requeridas son:

| Campo | Nombres aceptados |
|-------|-------------------|
| Latitud | `latitud`, `lat`, `y`, `coord_y` |
| Longitud | `longitud`, `lon`, `x`, `coord_x` |
| Fecha/hora | `fecha_hora`, `fecha_hech`, `fecha`, `datetime` |
| Tipo de delito | `prevenible`, `tipo_delito`, `delito`, `tipo` |
| Distrito | `distrito`, `distrito_policial`, `seccional` |

**Sistema de coordenadas:** WGS84 (EPSG:4326) para CSV/Excel. Los SHP en POSGAR 98 son reproyectados automáticamente.

---

## Contexto institucional

Estos plugins fueron desarrollados para la **Licenciatura en Seguridad** de la **Universidad Nacional de Villa María** (UNVM), en el marco de la cátedra **Nuevas Tecnologías Aplicadas a la Gestión de la Seguridad**.

El estudiantado está compuesto por personal en actividad de la **Policía de la Provincia de Córdoba**, lo que impone el requisito de que todo el procesamiento de datos ocurra en infraestructura local, sin transmisión a servicios externos. Los plugins fueron validados con datos del sistema **DPAD** (Departamento de Planificación y Análisis del Delito) de la Policía de la Provincia de Córdoba, período 2019–2023 (318.931 registros).

---

## Publicaciones relacionadas

- Castro, R.L. (2025). *Crime Analyst CBA: diseño e implementación de un plugin QGIS para el análisis espaciotemporal y la predicción de delitos mediante inteligencia artificial local.* UNVM.
- Castro, R.L. (2025). *Near Repeat CBA: diseño e implementación de un plugin QGIS para el análisis de victimización cercana y repetida mediante el Knox test.* UNVM.

---

## Autor

**Ricardo Luis Castro**
Docente — Cátedra Nuevas Tecnologías Aplicadas a la Gestión de la Seguridad
Licenciatura en Seguridad — Universidad Nacional de Villa María

---

## Licencia

GPL v2 — los plugins son software libre. Podés redistribuirlos y modificarlos bajo los términos de la Licencia Pública General GNU versión 2.

---

## Agradecimientos

- [QGIS Development Team](https://qgis.org) por la plataforma SIG de código abierto
- [Meta AI](https://llama.meta.com) por el modelo Llama 3.2
- [Ollama](https://ollama.com) por la infraestructura de inferencia local
- Kim Rossmo por el modelo CGT de perfilamiento geográfico criminal
- Suárez-Meaney, Palomares López & Chías Becerril por la implementación de referencia del algoritmo de Rossmo en gvSIG
