# Datos de ejemplo — Delitos CABA

Esta carpeta contiene instrucciones para descargar los datos públicos de delitos
de la **Ciudad Autónoma de Buenos Aires (CABA)**, utilizados como dataset de
demostración para los plugins de este repositorio.

Los datos **no se incluyen directamente** en el repositorio para mantenerlo
liviano. Se descargan desde el portal oficial del Gobierno de la Ciudad.

---

## Fuente oficial

**Portal:** Buenos Aires Data — Gobierno de la Ciudad Autónoma de Buenos Aires
**Dataset:** Delitos
**URL:** https://data.buenosaires.gob.ar/dataset/delitos
**Licencia:** [Creative Commons Attribution (CC BY)](https://creativecommons.org/licenses/by/4.0/)
**Responsable:** Dirección General Estadística Criminal y Mapa del Delito,
Subsecretaría de Investigación y Estadística Criminal, Ministerio de Seguridad GCBA
**Actualización:** Anual (última actualización: marzo 2026)

**Contenido:** Homicidios, hurtos (sin violencia), lesiones y robos (con violencia)
ocurridos en el ámbito de la Ciudad de Buenos Aires, con fecha, hora y ubicación geográfica.

---

## Descarga directa

### Formato CSV (recomendado para los plugins)

| Año | Enlace de descarga |
|-----|-------------------|
| 2024 | https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource/49f58c2e-21d7-4766-84e0-4bb753d28478/download |
| 2023 | https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource/dbec0c29-1ada-40df-b13c-75cf3013ca42/download |
| 2022 | https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource/3fbc3808-14c7-4559-8ba5-f68e919fee40/download |
| 2021 | https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource/3a691e3e-6df9-412b-a300-6c611733c2c2/download |
| 2020 | https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource/3f5fe778-bc8c-48ef-96fe-99aa62943152/download |
| 2019 | https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource/51ba3181-fabe-4b5f-8cd4-6dfcb24b0d67/download |
| 2018 | https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource/d4c82cef-3783-4da1-9e47-02df3ebba9e2/download |
| 2017 | https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource/217cf8ce-d4bc-477e-ab5a-173e9393478d/download |
| 2016 | https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource/97ff15b0-fbd5-4064-8856-48798f8347e9/download |

---

## Estructura del CSV

Los archivos tienen la siguiente estructura:

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| `id` | Identificador único del hecho | 12345 |
| `tipo_delito` | Tipo de delito | Robo (con violencia) |
| `subtipo_delito` | Subtipo del delito | Arrebato |
| `fecha` | Fecha del hecho (YYYY-MM-DD) | 2023-03-15 |
| `franja_horaria` | Franja horaria del hecho | Noche |
| `comuna` | Comuna de CABA (1–15) | 1 |
| `barrio` | Barrio de CABA | San Telmo |
| `lat` | Latitud WGS84 | -34.6201 |
| `long` | Longitud WGS84 | -58.3731 |
| `cantidad` | Cantidad de hechos | 1 |

> **Nota:** La estructura exacta puede variar entre años. Los archivos más
> recientes (2021–2024) incluyen coordenadas `lat`/`long`. Los años anteriores
> pueden requerir geocodificación a partir del barrio o comuna.

---

## Compatibilidad con los plugins

Los datos de CABA son compatibles con los tres plugins del repositorio.
A continuación se indica la equivalencia de columnas:

### Crime Analyst CBA

| Campo CABA | Campo esperado | Notas |
|------------|----------------|-------|
| `lat` | `latitud` | Detectado automáticamente |
| `long` | `longitud` | Detectado automáticamente |
| `fecha` | `fecha_hora` | Detectado automáticamente |
| `tipo_delito` | `tipo_delito` | Detectado automáticamente |
| `comuna` | `distrito` | Equivalente funcional |

El plugin detecta las columnas automáticamente. No se requiere renombrar nada.

### Near Repeat CBA

Misma compatibilidad que Crime Analyst CBA. Para el Knox test se recomienda
filtrar por un solo tipo de delito y una ventana temporal acotada (máximo
2–3 años) para mantener tiempos de cómputo razonables.

### Rossmo CBA

Para el perfilamiento geográfico de Rossmo se requiere una serie de hechos
**vinculados a un mismo ofensor serial**. Los datos de CABA son útiles para
demostraciones académicas y validación del modelo, pero el plugin acepta
cualquier capa de puntos — no necesariamente todos los delitos de la ciudad.

---

## Descarga masiva (todos los años)

Para descargar todos los años disponibles de una vez, ejecutá el siguiente
script desde la terminal:

```bash
# Crear carpeta de datos
mkdir -p data/caba

# Descargar CSV 2016-2024
declare -A urls=(
  ["2016"]="97ff15b0-fbd5-4064-8856-48798f8347e9"
  ["2017"]="217cf8ce-d4bc-477e-ab5a-173e9393478d"
  ["2018"]="d4c82cef-3783-4da1-9e47-02df3ebba9e2"
  ["2019"]="51ba3181-fabe-4b5f-8cd4-6dfcb24b0d67"
  ["2020"]="3f5fe778-bc8c-48ef-96fe-99aa62943152"
  ["2021"]="3a691e3e-6df9-412b-a300-6c611733c2c2"
  ["2022"]="3fbc3808-14c7-4559-8ba5-f68e919fee40"
  ["2023"]="dbec0c29-1ada-40df-b13c-75cf3013ca42"
  ["2024"]="49f58c2e-21d7-4766-84e0-4bb753d28478"
)

BASE="https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource"

for year in "${!urls[@]}"; do
  echo "Descargando $year..."
  curl -L "$BASE/${urls[$year]}/download" -o "data/caba/delitos_caba_${year}.csv"
done

echo "Descarga completada."
```

O desde Python:

```python
import urllib.request
import os

os.makedirs("data/caba", exist_ok=True)

archivos = {
    "2016": "97ff15b0-fbd5-4064-8856-48798f8347e9",
    "2017": "217cf8ce-d4bc-477e-ab5a-173e9393478d",
    "2018": "d4c82cef-3783-4da1-9e47-02df3ebba9e2",
    "2019": "51ba3181-fabe-4b5f-8cd4-6dfcb24b0d67",
    "2020": "3f5fe778-bc8c-48ef-96fe-99aa62943152",
    "2021": "3a691e3e-6df9-412b-a300-6c611733c2c2",
    "2022": "3fbc3808-14c7-4559-8ba5-f68e919fee40",
    "2023": "dbec0c29-1ada-40df-b13c-75cf3013ca42",
    "2024": "49f58c2e-21d7-4766-84e0-4bb753d28478",
}

base = "https://data.buenosaires.gob.ar/es_AR/dataset/delitos/resource"

for anio, uid in archivos.items():
    url = f"{base}/{uid}/download"
    destino = f"data/caba/delitos_caba_{anio}.csv"
    print(f"Descargando {anio}...")
    urllib.request.urlretrieve(url, destino)

print("Descarga completada.")
```

---

## Atribución requerida (CC BY)

Si usás estos datos en publicaciones, presentaciones o clases, la atribución
correcta es:

> Dirección General Estadística Criminal y Mapa del Delito, Subsecretaría de
> Investigación y Estadística Criminal, Ministerio de Seguridad, Gobierno de la
> Ciudad Autónoma de Buenos Aires. *Dataset Delitos* (2016–2024).
> https://data.buenosaires.gob.ar/dataset/delitos
> Licencia Creative Commons Attribution 4.0 Internacional (CC BY 4.0).

---

## Otros datasets públicos de delitos en Argentina

| Fuente | Cobertura | URL |
|--------|-----------|-----|
| Buenos Aires Data — Delitos | CABA, 2016–2024 | https://data.buenosaires.gob.ar/dataset/delitos |
| SNIC — Min. de Seguridad Nación | Nacional, agregado | https://www.argentina.gob.ar/seguridad/estadisticascriminales |
| Portal de Datos Justicia Argentina | Nacional | https://datos.jus.gob.ar |
| Datos Abiertos PBA | Prov. Buenos Aires | https://datos.estadistica.ec.gba.gov.ar |
