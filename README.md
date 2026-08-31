# Reporte de Tomas Clandestinas (2023)
## Reporte de Tomas Clandestinas 2019-2022 | Oficina de enlace de la subgerencia de gestión de derechos de uso y ocupación superficial (SGDUOS) Salamanca.

### Link al Dasboard: https://app.powerbi.com/groups/me/reports/e6334dd7-fa4c-4836-b8ef-bbd6625b9142/d057179ca41268483d49?experience=power-bi

## Resumen del Proyecto

En 2023 La oficina de enlace de la subgerencia de gestión de derechos de uso y ocupación superficial (SGDUOS) Salamanca, requería conocer el estado de las tomas clandestinas del sector bajío, con el propósito de evaluar las políticas implementadas desde el nivel federal a finales de 2018. 
Si bien, distintas áreas cuentan con archivos que contienen información sobre las incidencias, esta información no esta consolidada ni unificada, peor aun, ya que la información proviene directamente de reportes de campo, la información suele venir incompleta, no estandarizada o de plano incorrecta (coordenadas geográficas que no están en el país por ejemplo).
Es por esto que se me solicito desarrollar una solución que atendiera esta problemática. La solución fue desarrollar un pipeline para los reportes anuales. Que fuese replicable y aplicable para los siguientes años, con este pipeline se puede extraer
la información de los reportes de campo o informes anuales (usualmente en formato .xlsx conocidos popularmente como "Excel"), limpiarla y normalizarla para posteriormente subirla a una base de datos y disponer de ella. 
Esta Información sirvió para elaboración de un reporte ejecutivo que la oficina de enlace SGDUOS Salamanca, presentó a la subgerencia.

## Objetivo

Construir un pipeline reproducible de ETL/ELT que permita:

* Ingestar datos provenientes de archivos Excel.
* Conservar los datos originales mediante una capa Bronze.
* Detectar y corregir problemas de calidad.
* Estandarizar nombres y tipos de datos.
* Normalizar información geográfica y operacional.
* Transformar los datos en un modelo dimensional.
* Almacenar los datos procesados en PostgreSQL.
* Exponer la información para análisis y visualización en Power BI.

## Arquitectura 

<img width="962" height="192" alt="arquitectura_tomas_c_19_22" src="https://github.com/user-attachments/assets/dd5f4d18-0fe4-4001-bf60-5ba6a3c0385c" />

### Bronze

La capa Bronze representa la primera etapa de procesamiento.

El objetivo es conservar una representación estructurada de los datos originales sin realizar transformaciones analíticas agresivas.

* Lectura de archivos Excel mediante pandas.
* Eliminación de columnas completamente vacías.
* Renombrado de columnas utilizando snake_case.
* Homologación inicial de tipos de datos.
* Identificación de valores problemáticos.
* Manejo de columnas numéricas con valores no numéricos.

### Silver

Contiene los datos limpios, tipificados y normalizados.

Esta etapa concentra la mayor parte de las transformaciones de calidad de datos

* Procesos de limpieza.
* Estandarización de nombres.
* Se corrigieron variaciones como:

"GUANJUATO" = GUANAJUATO   

"MICHOACAN", "MICOACAN" = MICHOACÁN

Los reportes de campo contenían múltiples formas de referirse al mismo ducto.

Por ejemplo:

"Poliducto de 16"Ø Salamanca- Guadalajara", "Poliducto de 16"Ø Salamanca-Guadalajara", "poliducto de 16" Salamanca-Guadalajara"

**Normalización de poblados y municipios**

En lugar de intentar construir manualmente un catálogo exhaustivo de miles de localidades, se utilizaron tablas maestras con información oficial del INEGI para una normalización orientada a:

* estandarizar formato.
* eliminar inconsistencias evidentes.
* conservar la información disponible.
* evitar correcciones arbitrarias para casos individuales.

Coordenadas geográficas

Las coordenadas originales se encontraban principalmente en formato DMS:

20° 44' 59.50" N
-101° 10' 15.851" W

además de presentar diferentes variantes de símbolos, espacios y formatos.

Se transformaron a grados decimales:

20.749861
-101.171070

Esto permite utilizarlas directamente en:

* PostgreSQL.
* Power BI.
* análisis geoespacial.
* visualizaciones cartográficas.

**Data Quality**

Se implementó un reporte de calidad para comparar el estado de los datos antes y después de las transformaciones.

El reporte contempla:
* cantidad de registros.
* cantidad de columnas.
* duplicados.
* valores nulos.
* valores únicos.
* tipos de datos.
* cambios realizados por columna.
* reducción de valores inconsistentes.
* validación de coordenadas.




### Gold

La capa Gold transforma los datos Silver en un modelo dimensional orientado al análisis. Se utiliza un Star Schema.

Power BI

Power BI consume los datos almacenados en PostgreSQL.

El modelo permite construir visualizaciones como:

* número de tomas por estado.
* número de tomas por municipio.
* tomas por ducto. 
* evolución temporal. 
* concentración geográfica.
* mapas de ubicaciones.
* rankings de municipios y ductos.


# El Reporte Ejecutivo

La Oficina de Enlace de SGDUOS Salamanca, Guanajuato requería conocer la situación de las tomas clandestinas en el sector que supervisa. En este caso el corredor mencionado comprende el estado de Guanajuato y se vincula directamente
con el estado de Querétaro, Michoacán, San Luis Potosí y Jalisco. Otros estados involucrados son Hidalgo, Estado de México y Zacatecas que aunque no son colindantes, la comunicación de los ductos, hace necesario conocer y dar seguimiento a los incidentes registrados
en la red de ductos.

### Primera Parte

La primera del reporte contiene una vista general de los incidentes, los municipios, los estados y los ductos con mayor cantidad de incidencias. Como gráfica central tenemos el total de tomas clandestinas por año y la tendencia que se desarrollo en los  años subsecuentes.

<img width="1366" height="782" alt="captura_dashboard_tomas_1" src="https://github.com/user-attachments/assets/5e1756d9-0852-4b9d-bb0e-a566359e3966" />

La ventaja con la que contó la oficina de enlace al tener un reporte interactivo, hizo que una aparente tendencia a la baja, revelara información que de otro modo pudo pasar desapercibida:

<img width="1337" height="766" alt="captura_dashboard_tomas_2" src="https://github.com/user-attachments/assets/daa0fc88-86a4-494d-85ad-ea1780c4a340" />

Como se puede observar, a pesar de que la tendencia principal iba a la baja, en algunos municipios como Cuitzeo, experimentaron un incremento de incidencias en los años siguientes. Esta información sirvió en su momento para tomar las medidas pertinentes.

### Segunda Parte

La siguiente página del reporte se centra en la temporalidad de los eventos, la evolución de los incidentes en cada trimestre por año, los trimestres mas activos y como se distribuyeron a lo largo del año. También se creó un "Heat Map" o mapa de calor
con el objetivo de poder tener de manera gráfica un informe de los meses con mayor concentración de incidencias. En esta pagina se colocó el tipo de falla, ya que para la oficina de enlace tiene mas sentido visualizar 
el tipo de fallas relacionadas a las tomas clandestinas (Fallas Herméticas) mes con mes a diferencia de relacionarlas directamente con un ducto, municipio o estado.

<img width="1272" height="701" alt="captura_dashboard_tomas_3" src="https://github.com/user-attachments/assets/e20663b1-5b1e-4e3f-b0b7-4e10669aa269" />

Nuevamente estas gráficas arrojaron datos bastante interesantes, demostrando el acierto de haberlas implementado, en particular el "Heat Map".

<img width="571" height="474" alt="captura_dashboard_tomas_4" src="https://github.com/user-attachments/assets/7be2f7c0-a36c-41fb-95f6-1b529385a5ee" />

Por ejemplo, se observó que durante el mes de junio de 2022, no se registró ningún incidente, algo que según la oficina de enlace, era bastante inusual y extraño según fui informado.

### Tercera Parte

La tercera y ultima parte del reporte no solo contiene información detallada de cada toma, si no que incluso puede funcionar como un localizador, ya que contiene filtros para buscar por estado, municipio, población o directamente con el numero de folio o permiso.


<img width="1323" height="737" alt="captura_dashboard_tomas_5" src="https://github.com/user-attachments/assets/c9c0b4dd-1905-4a67-98e3-99fdbc38d43e" />

Todo esto fue posible debido a la normalización de coordenadas geográficas, que al final nos dan una vista mas que clara de el área supervisada.

<img width="1372" height="679" alt="Captura de pantalla 2026-08-30 233227" src="https://github.com/user-attachments/assets/082bbafa-e302-4bf0-94f3-c74fe9cde25e" />

Con ello se puede identificar el punto exacto.

## Tecnologías
| Tecnología	| Uso |
| --- | --- |
| Python      |	Desarrollo del pipeline |
| Pandas      |	Manipulación y transformación |
| PyArrow     |	Formato Parquet |
| Parquet     |	Almacenamiento intermedio |
| PostgreSQL  |	Base de datos analítica |
| SQLAlchemy	| Conexión Python–PostgreSQL |
| Power BI    | Visualización |


## Autor

Fausto G. Osorio Cruz









