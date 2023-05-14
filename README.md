# NequiTestDataEngineer
## Paso 1: Captura de datos

**Base de datos 1:** *bank_transactions.csv* tomado de https://www.kaggle.com/datasets/shivamb/bank-customer-segmentation 
Este archivo contiene las transacciones realizadas por al menos 800 mil clientes de un banco en la India. Los datos contienen información detallada de cada transacción como el identificador, la fecha de la transacción, el saldo y algunos datos adicionales del cliente como fecha de nacimiento, genero y ubicación.

Esta base de datos puede ser usada para segmentar los clientes según su perfil y comportamiento, identifficar posibles transacciones fraudulentas y/o predecir cantidad de transacciones en un determinado período de tiempo.

**Base de datos 2:** *Music_info.csv* y *User_Listening_History.csv* tomado de https://www.kaggle.com/datasets/undefinenull/million-song-dataset-spotify-lastfm?select=Music+Info.csv 

la primera BDD *Music_info* continete información detallada de algunas canciones de spotify como el nombre, el artista, duración de la canción, genero y otras características. En la segunda BDD *User_Listening_History* encontramos el identificador de la canción, el usuario y un conteo de reproducciones.
Con esta información podemos identificar cuál es el TOP de canciones más escuchadas en general y por usuario, también podemos segmentar por genero y artista. 

## Paso 2: Explorar y evaluar los datos, el EDA.

En Proyecto.ipynb se encontrará el paso a paso de la exploración de datos.

## Paso 3: Definir el modelo de datos

Para este paso se decidió crear dos modelos de datos para cada base de datos ya que no tienen relación entre si.

![Diagrama de bases de datos .png](https://github.com/manuelarr03/NequiTestDataEngineer/blob/261dbb8dcb146cff66c6b14c6dc393eb7a61e46b/Diagrama%20de%20bases%20de%20datos%20.png)

Para el primer modelo tenemos la base de datos *bank_transactions*. Para el segundo modelo, tenemos las bases *Music_info* y *User_Listening_History* que se relacionan entre sí por la variable *track_id* en una relación varios a uno. 

Para el diseño de la arquitectura se usará AWS con los servicios de *Amazon s3*, para almacenamiendo de los datos crudos y procesados, *AWS Glue*, para la preparación de los datos y llevar control de la ETL y *Athena*, para hacer los respectivos analisis interactivos. Finalmente, cuando los datos estén procesados, se hará uso de herramientas como Quicksight o Power BI con conexiones directas a S3 o por ODBC con Athena.

![Arquitectura](https://github.com/manuelarr03/NequiTestDataEngineer/blob/1298a08c9ac9c8d8f29492ec6a6dbec5fbf28c74/Arquitectura.png)

disponiilidad de la información, costos por usabilidad, escalabilidad el mismo se autogestiona para tener una mejor 

La frecuencia de actualización depende del uso de la data. Para la información bancaria es posible que se necesite identificar en tiempo real las transacciones ya que esto permitiría monitorear transacciones sospechosas de fraude. 
En le caso de la data de música, la frecuencia de actualización podría ser cada semana para sacar resumenes, top 10 y segmentaciones periodicamente.

## Paso 4: Ejecutar la ETL

Primero se guardan los datos en buckets de s3, uno para cada fuente:

![Buckets](https://github.com/manuelarr03/NequiTestDataEngineer/blob/5f30f8db96c0be797c5041e09e172a9c21c6c8a8/Buckets.png)

En Glue - Databases se crean las bases de datos: *db-banks* y *db-songs*, una para cada fuente de información.


![Databases](https://github.com/manuelarr03/NequiTestDataEngineer/blob/c65aff933f9184c269fdc4120f0e8cd02df1b352/Databases.png)

se crea el crawler, que apunta los bucket en donde se almacenaron los datos. En la base de datos *db-banks* se almacena la estructura de la tabla *kg_data_bank* y en *db-songs* se almacenan las estructuras de las tablas *music_info* y *user_listening*.


![Crawlers](https://github.com/manuelarr03/NequiTestDataEngineer/blob/c65aff933f9184c269fdc4120f0e8cd02df1b352/Crawlers.png)

El paso siguiente es crear la ETL jobs, en el que se harán las transformaciones necesarias para tener la data limpia y lista.

![ETL Bank](https://github.com/manuelarr03/NequiTestDataEngineer/blob/877741b0344f03b37e7f915512b296186290e331/ELT%20job%20bank.png)

En este job se procesa la tabla *kg_data_bank* haciendo cambio de formatos en fechas y creando una nueva variable *Edad* calculada en un nodo de SQL Query a partir de la variable *CustomerDOB*.

![Edad](https://github.com/manuelarr03/NequiTestDataEngineer/blob/256586d0374b402d42a930b0827cf6d35c81ebe9/Edad.png)

Se crea una ETL job para los datos de *db-songs*, para este caso, se cargan las dos tablas y se procesan al mismo tiempo, se cambian formatos y se hace un inner join entre las dos tablas para obtener una única tabla de salida.

![ETL songs](https://github.com/manuelarr03/NequiTestDataEngineer/blob/256586d0374b402d42a930b0827cf6d35c81ebe9/ELT%20job%20songs.png)

![Data procesada](https://github.com/manuelarr03/NequiTestDataEngineer/blob/256586d0374b402d42a930b0827cf6d35c81ebe9/Data%20procesada.png)

Se cambian los formatos de fecha y se eliminan duplicados
* se guarda la data proceesada en un bucket de s3 parquetisada, se define este formato ya que por ser comprimido se ahorran costos
## Paso 5: Completar la redacción del proyecto

El objetivo del proyecto es la construcción de un modelo y una arquitectura eficiente para que los datos puedan ser consumidos.  

* Si los datos se incrementaran en 100x. sería buena opcion particionar la data ya que las consultas a bases particionadas son mucho más rápidas y efectivas, se tiene mayor escalabilidad en el almacenamiento y reduce costos por optimización de almacenamiento. O utilizar el servicio de AWS Redshift que permite almacenar y analizar grandes cantidades de datos. 

* Si se requiere hacer analítica en tiempo real, ¿cuales componentes cambiaria a su
arquitectura propuesta? Cambiaría el almacenamiendo de datos s3 por Redshift ya que este servicio permite la creación de un data warehouse y el análisis de datos en tiempo real.
