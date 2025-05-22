# Actividad 3
## Las siguientes líneas son utilizadas para obtener el total de ingresos para cada una de las estaciones de la Línea 1 del metro para la fecha 2021-01-01 a partir del archivo afluenciastc_desglosado_02_2025.csv:
grep 2021-01-01 afluenciastc_desglosado_02_2025.csv |
grep -w "Línea 1" |
cut -d, -f 5 |
uniq > estaciones
grep 2021-01-01 afluenciastc_desglosado_02_2025.csv |
grep -w "Línea 1" > ingresos
cat estaciones | while read EST; do
echo -n "$EST: "
grep "$EST" ingresos | cut -d, -f7 |
awk '{ VAR += $1} ; END {print VAR}'
done

## a) ¿Qué resultado se obtiene al eliminar `grep -w "Línea 1"` de la secuencia de comandos?
Al eliminar `grep -w "Línea 1"`:
- Se procesarán todas las líneas del metro (no solo Línea 1)
- Los resultados incluirán estaciones de todas las líneas para la fecha 2021-01-01
- Puede haber múltiples entradas para la misma estación si pertenece a varias líneas


## b) A partir del razonamiento anterior, ¿qué modificación debe realizarse para obtener el total de ingresos de todas las estaciones (todas las líneas del metro) para el mes de enero de 2021?

# Filtrar todos los registros de enero 2021
grep ",Enero,2021," afluenciastc_desglosado_02_2025.csv > ingresos_enero_2021

# Obtener lista única de estaciones (considerando que pueden estar en múltiples líneas)
cut -d, -f5 ingresos_enero_2021 | sort | uniq > estaciones_enero_2021

# Calcular el total por estación
echo "Total de ingresos por estación en enero 2021:"
cat estaciones_enero_2021 | while read EST; do
  echo -n "$EST: "
  grep "$EST" ingresos_enero_2021 | cut -d, -f7 |
  awk '{sum += $1} END {print sum}'
done

# Calcular el GRAN TOTAL para todas las estaciones en enero 2021
echo -n "GRAN TOTAL ENERO 2021: "
cut -d, -f7 ingresos_enero_2021 | awk '{sum += $1} END {print sum}'

## Explicación:
1. Filtrado por mes y año: Usamos `grep ",Enero,2021,"` para capturar todos los registros de enero 2021, independientemente de la línea.
2. Lista de estaciones únicas: `cut -d, -f5 | sort | uniq` obtiene los nombres de todas las estaciones que tuvieron actividad en enero 2021.
3. Cálculo por estación: Para cada estación, sumamos todos los valores de afluencia (columna 7) donde aparece su nombre.
4. Gran total: Sumamos directamente todos los valores de la columna 7 del archivo filtrado para obtener el total general.


## c) A partir del resultado anterior, ¿qué modificaciones deben realizarse para obtener el total de ingresos de todas las estaciones (todas la líneas del metro) para el año 2021?
Para calcular los ingresos totales de todas las estaciones (todas las líneas) para todo el año 2021, adecuándolo a las modificaciones del inciso b):

# Filtrar todos los registros del año 2021
grep ",2021," afluenciastc_desglosado_02_2025.csv > ingresos_2021

# Obtener lista única de estaciones (considerando múltiples líneas)
cut -d, -f5 ingresos_2021 | sort | uniq > estaciones_2021

# Calcular el total por estación durante 2021
echo "Total de ingresos por estación en 2021:"
cat estaciones_2021 | while read EST; do
  echo -n "$EST: "
  grep "$EST" ingresos_2021 | cut -d, -f7 |
  awk '{sum += $1} END {print sum}'
done

# Calcular el GRAN TOTAL para todas las estaciones en 2021
echo -n "GRAN TOTAL 2021: "
cut -d, -f7 ingresos_2021 | awk '{sum += $1} END {print sum}'

## Explicación:

1. Filtrado por año: Cambiamos `grep ",Enero,2021,"` por `grep ",2021,"` para capturar todos los meses del año.

2. Archivos temporales: Renombramos los archivos temporales a `ingresos_2021` y `estaciones_2021` para reflejar el nuevo alcance.

3. Mismo principio de cálculo: Mantenemos la misma lógica de suma pero aplicada a todo el año.

## Versión con awk:

echo "Total de ingresos por estación en 2021:"
awk -F, '$3 == "2021" {a[$5] += $7} END {for (i in a) print i ": " a[i]}' afluenciastc_desglosado_02_2025.csv

# GRAN TOTAL 2021
awk -F, '$3 == "2021" {sum += $7} END {print "GRAN TOTAL 2021: " sum}' afluenciastc_desglosado_02_2025.csv


## d) Reportar la estación con más número de ingresos para los años 2021, 2022, 2023, 2024
Para identificar la estación con mayor número de ingresos cada año, se podría usar:
#!/bin/bash

# Encabezado del reporte
echo "ESTACIÓN CON MÁS INGRESOS POR AÑO"
echo "----------------------------------"

# Procesar cada año desde 2021 hasta 2024
for YEAR in {2021..2024}; do
    echo "Año $YEAR:"
    
    # Filtrar datos del año y calcular total por estación
    awk -F, -v year="$YEAR" '$3 == year {a[$5] += $7} 
    END {
        max = 0
        for (estacion in a) {
            if (a[estacion] > max) {
                max = a[estacion]
                estacion_max = estacion
            }
        }
        print "  Estación: " estacion_max
        print "  Ingresos totales: " max "\n"
    }' afluenciastc_desglosado_02_2025.csv
done

## e) A partir de los resultados anteriores, ¿ qué modificaciones deben realizarse para obtener el total de ingresos de todas las estaciones para todos los registros en el archivo, en otras palabras para todos los años registrados 2021 .. 2025?
Para obtener el total acumulado de ingresos de todas las estaciones considerando todos los años registrados (2021-2025):

# 1. Calcular totales por estación para todo el periodo
echo "PROCESANDO DATOS COMPLETOS (2021-2025)..."
echo "========================================"
echo "Total de ingresos por estación (2021-2025):"
echo ""

# Usamos awk para procesamiento eficiente de todo el archivo
awk -F, '
{
    # Sumar ingresos (columna 7) por estación (columna 5)
    ingresos_por_estacion[$5] += $7
    # Calcular gran total 
    gran_total += $7
} 
END {
    # Ordenar resultados por ingresos (descendente)
    n = asorti(ingresos_por_estacion, estaciones_ordenadas, "@val_num_desc")
    
    # @ - Indica que es un tipo de ordenamiento especial
    # val - Ordena por valores (no por índices)
    # num - Considera los valores como numéricos
    # desc - Orden descendente (de mayor a menor)
    
    
    # Imprimir resultados ordenados
    for (i = 1; i <= n; i++) {
        estacion = estaciones_ordenadas[i]
        printf "%-30s %12'd\n", estacion, ingresos_por_estacion[estacion]
    }
    
    # Imprimir gran total
    printf "\nGRAN TOTAL (2021-2025): %'d\n", gran_total
}
' afluenciastc_desglosado_02_2025.csv > reporte_final.txt

echo "Reporte generado en: reporte_final.txt"

# Calcular directamente el gran total
echo "CALCULANDO TOTAL GLOBAL (2021-2025)..."

TOTAL=$(awk -F, '{sum += $7} END {print sum}' afluenciastc_desglosado_02_2025.csv)

echo "El total de ingresos acumulados para todas las estaciones (2021-2025) es:"
printf "%'d\n" $TOTAL

El script maneja automáticamente casos como la estación "Chabacano" que aparece en múltiples líneas, sumando todos sus ingresos independientemente de la línea a la que corresponda cada registro.
