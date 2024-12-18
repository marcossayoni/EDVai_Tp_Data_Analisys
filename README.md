# Análisis de delitos en la ciudad de Londres
Trabajo Final EDVai - Data Analyst BootCamp - Diciembre 2024.
Autor: Marcos E. Sayoni
https://www.linkedin.com/in/marcos-sayoni-67b938a/

## Contenido

- [INTRODUCCIÓN](#INTRODUCCIÓN)
- [FUENTES DE DATOS](#FUENTES-DE-DATOS)
- [HIPÓTESIS](#HIPÓTESIS)
- [PLAN DE MÉTRICAS](#PLAN-DE-MÉTRICAS)
- [MODELO DE DATOS](#MODELO-DE-DATOS)
- [TRANSFORMACIÓN Y CARGA DE DATOS](#TRANSFORMACIÓN-Y-CARGA-DE-DATOS)
- [FLUJO DE DATOS](#FLUJO-DE-DATOS)
- [CONSTRUCCIÓN DE MÉTRICAS EN POWER BI](#CONSTRUCCIÓN-DE-MÉTRICAS-EN-POWER-BI)
- [DESARROLLO DE DASHBOARDS](#DESARROLLO-DE-DASHBOARDS)
- [CONCLUSIÓN](#CONCLUSIÓN)
- [LINKS](#LINKS)

## INTRODUCCIÓN

<p>En este trabajo práctico, se utilizan tres datasets públicos como caso de estudio. 
El primero contiene información sobre los crímenes registrados en los distritos de 
la ciudad de Londres entre 2008 y 2016. El segundo dataset incluye datos sobre los
distritos de Londres, como su superficie en kilómetros cuadrados y la población por
distrito. El tercero proporciona información climática promedio de Londres durante 
el mismo periodo (2008-2016).</p>

Los datasets provienen de diferentes fuentes:

<ol>
  <li><strong>Crímenes en Londres:</strong> extraído de los datos públicos de Google Cloud en Big Query Public Data.</li>
  <li><strong>Distritos de Londres:</strong> obtenido de Wikipedia.</li>
  <li><strong>Clima promedio en Londres:</strong> basado en referencias climáticas 
del sitio<a href="https://www.metoffice.gov.uk/research/climate/maps-and-data"> 
Met Office</a>.</li>
</ol>

## FUENTES DE DATOS

### Dataset London_crime

<p>Este dataset contiene información detallada sobre crímenes en Londres a dos niveles
geográficos: LSOA (Áreas de Salida a Nivel Bajo, por sus siglas en inglés) y distritos.
Los datos están organizados por año y tipo de delito, y se dividen en las siguientes 
categorías principales y sus subcategorías:</p>

<ol>
  <li><strong>Robo (Burglary):</strong></li>
  <ul>
    <li>Robo en viviendas.</li>
    <li>Robo en otros edificios.</li>
    <li>Robo residencial.</li>
    <li>Robo comercial y comunitario.</li>
  </ul>


  <li><strong>Daño criminal (Criminal Damage):</strong></li>
  <ul>
    <li>Daño a viviendas.</li>
    <li>Daños a vehículos.</li>
    <li>Daños a otros edificios.</li>
    <li>Daños generales.</li>
  </ul>


  <li><strong>Drogas (Drugs):</strong></li>
  <ul>
    <li>Tráfico de drogas.</li>
    <li>Posesión de drogas.</li>
    <li>Otros delitos relacionados con drogas.</li>
  </ul>


  <li><strong>Fraude o falsificación (Fraud or Forgery):</strong></li>
  <ul>
    <li>Delitos contabilizados por víctima (*no disponibles a nivel LSOA).</li>
  </ul>


  <li><strong>Otros delitos notificables (Other Notifiable Offences):</strong></li>
  <ul>
    <li>Incluyen posesión de armas y otros delitos menores.</li>
  </ul>


  <li><strong>Robos (Robbery):</strong></li>
  <ul>
    <li>Robos a propiedades comerciales.</li>
    <li>Robos personales.</li>
  </ul>


  <li><strong>Delitos sexuales (Sexual Offences):</strong></li>
  <ul>
    <li>Violación.</li>
    <li>Otros delitos sexuales.</li>
  </ul>


  <li><strong>Robo y manejo de bienes robados (Theft and Handling):</strong></li>
  <ul>
    <li>Robo de vehículos.</li>
    <li>Robo de bicicletas.</li>
    <li>Otros tipos de robo.</li>
  </ul>


  <li><strong>Delitos violentos contra personas (Violence Against the Person):</strong></li>
  <ul>
    <li>Agresiones.</li>
    <li>Homicidios.</li>
    <li>Uso de armas ofensivas.</li>
    <li>Acoso.</li>
  </ul>
</ol>

#### Estructura del dataset

<p>(link github de la imagen del dataset crime_by_lsoa)</p>

<p><strong>lsoa_code:</strong> LSOA significa <strong>Lower Layer Super Output Area</strong>, 
que es una unidad geográfica utilizada en Inglaterra y Gales para analizar 
estadísticas locales. Fue diseñada por la Oficina Nacional de Estadísticas (ONS) 
como una subdivisión de las áreas administrativas más grandes, como los distritos
o municipios. Es una unidad inferior a los distritos y para nuestro caso de estudio 
no será utilizada, ya que se prioriza el estudio por distrito a nivel general.</p>


<p><strong>borough:</strong> distritos de la ciudad de Londres.</p>
<p><strong>major_category:</strong> rubro o categoría del crimen, es un campo 
jerárquico el cual agrupa los tipos de crímenes minor_category.</p>
<p><strong>minor_category:</strong> tipo de crimen cometido, según la segmentación del
dataset esta descripción está contenida dentro del major_category.</p>

<p><strong>value:</strong> cantidad de crímenes cometidos.</p>
<p><strong>year:</strong> año</p>
<p><strong>month:</strong> mes</p>

### Dataset distritos de Londres (dim_distritos)

<p>Este dataset incluye información sobre todos los distritos de Londres, detallando su superficie en kilómetros cuadrados (km²) y su población.</p>

<p>La población registrada originalmente correspondía al año 2001, pero para hacerla más representativa
del análisis hasta 2016, se realizó un ajuste considerando el crecimiento poblacional. Según los datos obtenidos de la página 
oficial de la embajada del Reino Unido, la población total de Londres en 2001 era de aproximadamente <strong>7.12 millones (MM)</strong>, mientras 
que en la actualidad (2024) es de <strong>8.8 MM</strong>, lo que representa un crecimiento del 23% en 23 años.</p>

#### <i>Ajuste de población al año 2016</i>

<p>Dado que el dataset de crímenes contiene información hasta 2016 y que se requiere comparar delitos por distrito con su respectiva población,
se asumió un crecimiento lineal de la población entre 2001 y 2024. Bajo esta hipótesis, se estimó un incremento anual promedio del 1% (23% dividido por 23 años).</p>

Para calcular la población estimada en 2016:
  <ul>
    <li>Se consideró un crecimiento acumulado del 16% entre 2001 y 2016.</li>
    <li>Se ajustó la población de cada distrito aplicando este porcentaje.</li>
  </ul>
Estas transformaciones fueron realizadas localmente antes de cargar el dataset a BigQuery.

#### Estructura del dataset

<p>(link github de la imagen del dataset dim_ditrito)</p>

<p><strong>id_distrito:</strong> clave principal.</p>
<p><strong>distrito:</strong> distrito de Londres.</p>
<p><strong>superficie_km2:</strong> km2 de la superficie del distrito.</p>
<p><strong>poblacion:</strong> cantidad de habitantes del distrito ponderada al 2016 (año completo).</p>
<p><strong>latitud:</strong> latitud del distrito.</p>
<p><strong>longitud:</strong> longitud del distrito.</p>

### Dataset clima de Londres (dim_clima)

<p>Este dataset contiene la temperatura promedio por mes, las estaciones del año, el promedio de mm de lluvia y tipo de clima del 2008-2016 en de la ciudad de Londres. </p>

<p>Estos datos se construyeron en base a la información declarada en el sitio Met Office. Este sitio posee la información correspondiente al servicio meteorológico nacional
del Reino Unido proporciona datos climáticos históricos detallados, incluyendo temperaturas promedio, precipitaciones y condiciones climáticas por región y período.</p>

#### Estructura del dataset

<p>(link github de la imagen del dataset dim_clima)</p>

<p><strong>id_clima:</strong> clave principal.</p>
<p><strong>mes:</strong> mes calendario.</p>
<p><strong>estacion:</strong> estación del año.</p>
<p><strong>temperatura_media:</strong> temperatura media en grados Celsius del 2008 al 2016 (año completo).</p>
<p><strong>precipitacion_media:</strong> precipitación media en mm del 2008 al 2016 (año completo).</p>
<p><strong>condicion_climatica:</strong> definida en base al promedio entre los 2008 al 2016 (año completo).</p>

## HIPÓTESIS

<p>Se plantea como premisa que los distritos con mayor densidad poblacional podrían ser más propensos a delitos violentos debido a una mayor interacción
social y al estrés propio de los entornos urbanos. Identificar estas áreas permitirá enfocar estrategias como el incremento del patrullaje, 
la implementación de campañas de concientización y la promoción de actividades comunitarias.</p>

<p>Además, se busca analizar si existe alguna correlación entre las estaciones del año y las condiciones climáticas con el aumento de la delincuencia.</p>

<p>El objetivo final del análisis es identificar patrones históricos que sirvan como base para diseñar políticas de seguridad más efectivas, 
con un enfoque particular en la reducción de la delincuencia y, especialmente, de los delitos violentos (Violence Against the Person).</p>

## PLAN DE MÉTRICAS
<p>Se realiza un plan de métricas amplio que permita ver en los reportes la incidencia de los delitos con más ocurrencia con especial énfasis en los delitos violentos, 
sus distribuciones temporales y espaciales, comportamientos frente a las distintas estaciones del año y por distrito.</p>

<p>Link al plan de métricas</p>

<p>Para poder desarrollar las métricas planteadas se creo una tabla de hechos con la cantidad de crímenes por distrito, fecha y categoría del crimen.

<p>Se generaron 4 dimensiones para su segmentación, la dimensión calendario para realizar cortes por segmentos de tiempo, la dimensión con la categoría 
del crimen y la descripción del crimen, la dimensión por distrito y la dimensión clima.</p>

## MODELO DE DATOS

<p>Link al modelo de datos</p>

## TRANSFORMACIÓN Y CARGA DE DATOS

#### <i>Contrucción de la capa Silver</i>

<p>SQL de la dimensión <strong>dim_crimen</strong></p>

```sql
create or replace TABLE london_crime_clean.dim_crimen AS
SELECT 
    ROW_NUMBER() OVER (ORDER BY tipo_crimen, categoria) AS id_dim_crimen,
    tipo_crimen as categoria,
    categoria as crimen
  FROM (
    SELECT DISTINCT tipo_crimen, categoria
    FROM `silverlayer-442418.london_crime_clean.crimenes_londres`
)
order by id_dim_crimen;
```
<p>Para la creación de las dimensiones <strong>dim_distritos</strong></p>

<p>Se descargo la información de Wikipedia a un Excel y se conformó el csv con la estructura mencionada y se importó a Bigquery.</p>

<p>Para la creación de las dimensiones <strong>dim_clima</strong></p>

<p>Se descargo la información del sitio <a href="https://www.metoffice.gov.uk/research/climate/maps-and-data"> 
Met Office</a> a un Excel y se conformó el csv con la estructura mencionada y se importó a Bigquery.</p>

<p>DAX de la dimensión <strong>dim_calendario</strong></p>

<p>Esta dimensión se generó directamente en Power BI, ya que solo se usará para este proyecto</p>

```DAX
dim_calendario = ADDCOLUMNS (
CALENDAR ( DATE( 2008,1,1), DATE( 2016, 12, 31 ) ),
"id_calendario", FORMAT ( [Date], "YYYYMMDD" ),
"#Año", YEAR ( [Date] ),
"#Trimestre", QUARTER ( [Date] ),
"#Mes", MONTH ( [Date] ),
"#Día", DAY ( [Date] ),
"Trimestre", "T" & FORMAT ( [Date], "Q" ),
"MesCorto", FORMAT ( [Date], "MMM" )
 )
```

<p>Generación de la tabla de hechos <strong>fact_crimenes_londres</strong>, esta tabla va a contener la cantidad de crimenes por distrito, categoria, fecha y delito.</p>

<p>1.- Primera transformación, se descartó el campo lsoa_code se y agrupo por distrito, año, mes, categoría del crimen y crimen.</p> 
```sql
create or replace TABLE london_crime_clean.crimenes_londres AS
(SELECT
  borough as distrito, 
  major_category as tipo_crimen,
  minor_category as categoria,
  sum(value) as cantidad_crimenes,
  year as anio,
  month as mes
FROM
  `bigquery-public-data.london_crime.crime_by_lsoa`
group by borough, major_category, minor_category, year, month
order by borough);
```
<p>Link a la imagen de la tabla crimen_london</p>

<p>2.- Luego con la segunda transformación, se mapearon las dimensiones dim_crimen, dim_calendario, dim_distrito, dim_clima y dim_calendario (se terminará mapenado en Power BI)</p>

```sql
create or replace table london_crime_clean.fact_crimenes_londres as (
SELECT 
ROW_NUMBER() OVER () AS id_crimenes, -- Genera un ID único para cada registro
CONCAT(CAST(f.anio AS STRING), 
LPAD(CAST(f.mes AS STRING), 2, '0'), '01') AS id_calendario, -- Concatenación de anio, mes y día "01"
d.id_distrito AS id_distrito,-- Relación con la dimensión de distrito
cl.id_clima as id_clima, --Relación con la dimensión clima
crim.id_crimen AS id_crimen, --Relación con la dimensión crimen
f.cantidad_crimenes AS cant_crimenes --Relación con la dimensión crimen
FROM  `silverlayer-442418.london_crime_clean.crimenes_londres` f
INNER JOIN `silverlayer-442418.london_crime_clean.dim_distrito` d
ON d.distrito = f.distrito
INNER JOIN `silverlayer-442418.london_crime_clean.dim_clima` cl
ON cl.mes =f.mes
INNER JOIN `silverlayer-442418.london_crime_clean.dim_crimen` crim
ON 
crim.categoria = f.tipo_crimen AND crim.crimen = f.categoria
);
```

<p>Link a la imagen de la tabla fact_crimenes_londres</p>

## FLUJO DE DATOS

<p>Link a la imagen del flujo de datos</p>

#### Carga de datos en Power BI

<p>Link a la imagen de la carga de datos en Power BI</p>

## CONSTRUCCIÓN DE MÉTRICAS EN POWER BI


<p><strong>1.- Tasa de delitos violentos cada 1000 habitantes.</strong></p>

```DAX
001 #tasa de delitos violentos = 
DIVIDE(
    CALCULATE(
        SUM(fact_crimenes_londres[cant_crimenes]),
        dim_crimen[categoria] = "Violence Against the Person"
    ),
    SUM(dim_distrito[poblacion])
) * 1000
```
<p><strong>Propósito:</strong> Identificar distritos con mayor incidencia de crímenes violentos en relación a su población. </p>


<p><strong>2.- Cantidad de delitos </strong> </p>

<p><strong>Propósito:</strong> Permite ver la cantidad de delitos totales a nivel general, estos luego podrán ser segmentados por distrito, tipo de delito, fecha o condición climática.</p>

```DAX
002 #cant. crimenes = SUM(fact_crimenes_londres[cant_crimenes]) 
```

<p><strong>3.- Índice de diversidad de delitos </strong> </p>

<p><strong>Propósito:</strong> Se aplica la formula de Shannon(-SUM(Proporcion*LN(Proporcion))). Este índice mide cuán diversificada o concentrada está la distribución de tipos de delitos en un área específica o período de tiempo.
Una baja diversidad (índice bajo) indica que un tipo de delito específico domina en la zona, lo que puede orientar los esfuerzos policiales o de prevención hacia ese problema concreto.
Una alta diversidad (índice alto) sugiere la necesidad de abordar múltiples tipos de delitos con estrategias más integrales.</p>

```DAX
003 #indice diversidad delitos = 
VAR TotalDelitos = 
    CALCULATE(SUM(fact_crimenes_londres[cant_crimenes]), ALL(dim_crimen))
RETURN
    -SUMX(
        dim_crimen,
        VAR ProporcionDelitos = 
            DIVIDE(
                CALCULATE(SUM(fact_crimenes_londres[cant_crimenes])),
                TotalDelitos
            )
        RETURN
            IF(ProporcionDelitos > 0, ProporcionDelitos * LN(ProporcionDelitos), 0)
    )
```

<p><strong>4.- Densidad poblacional </strong> </p>

<p><strong>Propósito:</strong> Esta métrica tiene como objetivo complementar las métricas de delitos y ver si existe correlación con la densidad de población y la cantidad de delitos.</p>

```DAX
004 #densidad poblacional = 
DIVIDE(
	SUM('dim_distrito'[poblacion]),
	SUM('dim_distrito'[superficie_km2])
)
```
<p><strong>5.- Tasa de delitos violentos cada 1000 habitantes.</strong></p>

<p><strong>Propósito:</strong> Identificar distritos con mayor incidencia de delitos no violentos en relación a su población y para compararlos contra los violentos. </p>

```DAX
005 #tasa de delitos no violentos = 
DIVIDE(
    CALCULATE(
        SUM(fact_crimenes_londres[cant_crimenes]),
        dim_crimen[categoria] <> "Violence Against the Person"
    ),
    SUM(dim_distrito[poblacion])
) * 1000
```
<p><strong>6.- Temperatura promedio.</strong></p>

<p><strong>Propósito:</strong> Este índice se complementa con la métricas de delitos, para ver si hay influencia en la temperatura al aumento o disminución delictiva. </p>

```DAX
006 #temperatura promedio mes = SUM(dim_clima[temperatura_media])
```
