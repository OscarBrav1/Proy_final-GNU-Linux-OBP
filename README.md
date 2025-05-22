# Proyecto final Herramientas GNU/LINUX - Oscar Bravo Pérez

## Automatizar el procesamiento de datos de temperatura y precipitación de las estaciones meteorológicas del Programa PEMBU (UNAM) mediante un script en BASH para generar un archivo CSV estructurado, y utilizar Python para visualizar gráficas analíticas.

Actividad 1: 70 puntos
## Script para generar el archivo .csv
#!/bin/bash

# Ruta base donde se encuentran los archivos CSV
BASE_PATH="/LUSTRE/tmp/temp/estaciones-ENP/"

# Archivo de salida
OUTPUT_FILE="datos_estaciones_mensuales.csv"

# Encabezado del archivo CSV
echo "Estacion,Longitud,Latitud,Tmax,Pmax,Date" > "$OUTPUT_FILE"

# Primero recolectamos todos los datos
declare -A datos_estaciones

# Procesar cada mes (1 a 12)
for mes in 1 2 3 4 5 6 7 8 9 10 11 12; do
    # Formatear mes con dos dígitos
    mes_formateado=$(printf "%02d" $mes)
    
    # Procesar cada estación (ENP1 a ENP9)
    for estacion in {1..9}; do
        archivo="${BASE_PATH}2022-${mes_formateado}-ENP${estacion}-L1.CSV"
        
        if [ -f "$archivo" ]; then
            # Extraer coordenadas solo si no las tenemos
            if [ -z "${datos_estaciones["ENP${estacion}_lat"]}" ]; then
                coords=$(grep "Lat" "$archivo" | head -n 1)
                latitud=$(echo "$coords" | awk -F'Lat ' '{print $2}' | awk '{print $1}' | tr -d ',N')
                longitud=$(echo "$coords" | awk -F'Lon ' '{print $2}' | awk '{print $1}' | tr -d ',W')
                longitud="-$longitud"
                datos_estaciones["ENP${estacion}_lat"]=$latitud
                datos_estaciones["ENP${estacion}_lon"]=$longitud
            fi
            
            # Procesar datos (omitir primeras 8 líneas)
            datos=$(tail -n +9 "$archivo")
            
            # Temperatura máxima (dejar vacío si no hay datos)
            tmax_mes=$(echo "$datos" | awk -F',' '{if ($2 ~ /^[0-9.]+$/) print $2}' | sort -nr | head -n 1)
            datos_estaciones["ENP${estacion}_2022-${mes_formateado}_tmax"]=$tmax_mes
            
            # Precipitación máxima (dejar vacío si no hay datos)
            pmax_mes=$(echo "$datos" | awk -F',' '{if ($9 ~ /^[0-9.]+$/) print $9}' | sort -nr | head -n 1)
            datos_estaciones["ENP${estacion}_2022-${mes_formateado}_pmax"]=$pmax_mes
        fi
    done
done

# Generar salida ordenada por mes
for mes in 01 02 03 04 05 06 07 08 09 10 11 12; do
    for estacion in {1..9}; do
        latitud=${datos_estaciones["ENP${estacion}_lat"]}
        longitud=${datos_estaciones["ENP${estacion}_lon"]}
        tmax=${datos_estaciones["ENP${estacion}_2022-${mes}_tmax"]}
        pmax=${datos_estaciones["ENP${estacion}_2022-${mes}_pmax"]}
        
        if [ -n "$latitud" ] && [ -n "$longitud" ]; then
            # Si no hay datos, dejar el campo vacío
            echo "ENP${estacion},$longitud,$latitud,${tmax:-},${pmax:-},2022-${mes}" >> "$OUTPUT_FILE"
        fi
    done
done

echo "Procesamiento completado. Resultados en $OUTPUT_FILE"

## Explicación Detallada del Script

### 1. Configuración Inicial
- `BASE_PATH`: Ruta base donde se encuentran los archivos CSV
- `OUTPUT_FILE`: Nombre del archivo de salida consolidado

### 2. Encabezado del CSV de Salida
echo "Estacion,Longitud,Latitud,Tmax,Pmax,Date" > "$OUTPUT_FILE"
Crea el archivo de salida con el encabezado especificado.

### 3. Array Asociativo para Datos
declare -A datos_estaciones
Crea un array asociativo para almacenar temporalmente todos los datos antes de generar la salida.

### 4. Procesamiento por Meses (1-12)
for mes in 1 2 3 4 5 6 7 8 9 10 11 12; do
    mes_formateado=$(printf "%02d" $mes)
- Itera sobre los 12 meses del año
- `printf "%02d"` formatea el mes con dos dígitos (01-12)

### 5. Procesamiento por Estaciones (1-9)
for estacion in {1..9}; do
    archivo="${BASE_PATH}2022-${mes_formateado}-ENP${estacion}-L1.CSV"
Construye el nombre del archivo para cada estación y mes.

### 6. Verificación de Archivo
if [ -f "$archivo" ]; then
Verifica si el archivo existe antes de procesarlo.

### 7. Extracción de Coordenadas
coords=$(grep "Lat" "$archivo" | head -n 1)
latitud=$(echo "$coords" | awk -F'Lat ' '{print $2}' | awk '{print $1}' | tr -d ',N')
longitud=$(echo "$coords" | awk -F'Lon ' '{print $2}' | awk '{print $1}' | tr -d ',W')
longitud="-$longitud"
- `grep "Lat"`: Busca la línea con coordenadas
- `awk` y `tr`: Extraen y limpian los valores de latitud y longitud
- La longitud se hace negativa (oeste)

### 8. Procesamiento de Datos
datos=$(tail -n +9 "$archivo")
Omite las primeras 8 líneas de metadatos del archivo CSV.

### 9. Extracción de Temperatura Máxima
tmax_mes=$(echo "$datos" | awk -F',' '{if ($2 ~ /^[0-9.]+$/) print $2}' | sort -nr | head -n 1)
- `awk`: Filtra solo valores numéricos en la columna 2 (temperatura)
- `sort -nr`: Ordena numéricamente en orden descendente
- `head -n 1`: Toma el valor máximo

### 10. Extracción de Precipitación Máxima
pmax_mes=$(echo "$datos" | awk -F',' '{if ($9 ~ /^[0-9.]+$/) print $9}' | sort -nr | head -n 1)
Misma lógica que para temperatura, pero para la columna 9 (precipitación)

### 11. Almacenamiento en Array
datos_estaciones["ENP${estacion}_lat"]=$latitud
datos_estaciones["ENP${estacion}_lon"]=$longitud
datos_estaciones["ENP${estacion}_2022-${mes_formateado}_tmax"]=$tmax_mes
datos_estaciones["ENP${estacion}_2022-${mes_formateado}_pmax"]=$pmax_mes
Guarda los datos en el array asociativo para su posterior procesamiento.

### 12. Generación del Archivo de Salida
for mes in 01 02 03 04 05 06 07 08 09 10 11 12; do
    for estacion in {1..9}; do
        echo "ENP${estacion},$longitud,$latitud,${tmax:-},${pmax:-},2022-${mes}" >> "$OUTPUT_FILE"
    done
done
- Itera primero por meses, luego por estaciones
- `${variable:-}` deja el campo vacío si no hay datos
- Los resultados se ordenan cronológicamente

## Ejecución

chmod +x procesar_estaciones.sh
./procesar_estaciones.sh

## Salida Esperada
Archivo CSV con estructura:
Estacion,Longitud,Latitud,Tmax,Pmax,Date
ENP1,-99.1220,19.2713,25.3,,2022-01
ENP2,-99.1234,19.2815,,0.5,2022-01
...
ENP9,-99.1322,19.3338,32.0,0.0,2022-12

