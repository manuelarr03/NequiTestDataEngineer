# NequiTestDataEngineer
## Paso 1: Captura de datos

**Base de datos 1:** *bank_transactions.csv* tomado de https://www.kaggle.com/datasets/shivamb/bank-customer-segmentation 

Este archivo contiene las transacciones realizadas por al menos 800 mil clientes de un banco en la India. Los datos contienen información detallada de cada transacción como el identificador, la fecha de la transacción, el saldo y algunos datos adicionales del cliente como fecha de nacimiento, género y ubicación.

Esta base de datos puede ser usada para segmentar los clientes según su perfil y comportamiento, identificar posibles transacciones fraudulentas y/o predecir cantidad de transacciones en un determinado período de tiempo.

**Base de datos 2:** *Music_info.csv* y *User_Listening_History.csv* tomado de https://www.kaggle.com/datasets/undefinenull/million-song-dataset-spotify-lastfm?select=Music+Info.csv 

la primera base *Music_info* contiene información detallada de algunas canciones de Spotify como el nombre, el artista, duración de la canción, género y otras características. En la segunda BDD *User_Listening_History* encontramos el identificador de la canción, el usuario y un conteo de reproducciones.
Con esta información podemos identificar cuál es el TOP de canciones más escuchadas en general y por usuario, también podemos segmentar por género y artista. 

## Paso 2: Explorar y evaluar los datos, el EDA.

En Proyecto.ipynb se encontrará el paso a paso de la exploración de datos.

## Paso 3: Definir el modelo de datos

Para este paso se decidió crear dos modelos de datos para cada base de datos ya que no tienen relación entre si.

![Diagrama de bases de datos .png](https://github.com/manuelarr03/NequiTestDataEngineer/blob/261dbb8dcb146cff66c6b14c6dc393eb7a61e46b/Diagrama%20de%20bases%20de%20datos%20.png)

Para el primer modelo tenemos la base de datos *bank_transactions*. Para el segundo modelo, tenemos las bases *Music_info* y *User_Listening_History* que se relacionan entre sí por la variable *track_id* en una relación varios a uno. 

Para el diseño de la arquitectura se usará AWS con los servicios de *Amazon s3*, para almacenamiento de los datos crudos y procesados, *AWS Glue*, para la preparación de los datos y llevar control de la ETL y *Athena*, para hacer los respectivos análisis interactivos. Finalmente, cuando los datos estén procesados, se hará uso de herramientas como Quicksight o Power BI con conexiones directas a S3 o por ODBC con Athena.

![Arquitectura](https://github.com/manuelarr03/NequiTestDataEngineer/blob/1298a08c9ac9c8d8f29492ec6a6dbec5fbf28c74/Arquitectura.png)

Se decide usar estas herramientas ya que AWS es altamente escalable, el mismo se autogestiona para tener un mejor rendimiento, se paga por los recursos utilizados, tiene un porcentaje muy alto de disponibilidad, entre otros.

La frecuencia de actualización depende del uso de la data. Para la información bancaria es posible que se necesite identificar en tiempo real las transacciones ya que esto permitiría monitorear transacciones sospechosas de fraude. 
En el caso de la data de música, la frecuencia de actualización podría ser cada semana para sacar resúmenes, top 10 y segmentaciones periódicamente.

## Paso 4: Ejecutar la ETL

Primero se guardan los datos en buckets de s3, uno para cada fuente de información.

![Buckets](https://github.com/manuelarr03/NequiTestDataEngineer/blob/5f30f8db96c0be797c5041e09e172a9c21c6c8a8/Buckets.png)

En Glue - Databases se crean las bases de datos: *db-banks* y *db-songs*.


![Databases](https://github.com/manuelarr03/NequiTestDataEngineer/blob/c65aff933f9184c269fdc4120f0e8cd02df1b352/Databases.png)

Se crea el crawler, que apunta los bucket en donde se almacenaron los datos. En la base de datos *db-banks* se almacena la estructura de la tabla *kg_data_bank* y en *db-songs* se almacenan las estructuras de las tablas *music_info* y *user_listening*.


![Crawlers](https://github.com/manuelarr03/NequiTestDataEngineer/blob/c65aff933f9184c269fdc4120f0e8cd02df1b352/Crawlers.png)

El paso siguiente es crear la ETL con Glue job, en el que se harán las transformaciones necesarias para tener la data limpia y lista.

![ETL Bank](https://github.com/manuelarr03/NequiTestDataEngineer/blob/877741b0344f03b37e7f915512b296186290e331/ELT%20job%20bank.png)

En este job se procesa la tabla *kg_data_bank* haciendo cambio de formatos en fechas y creando una nueva variable *Edad* calculada en un nodo de SQL Query a partir de la variable *CustomerDOB*.

![Edad](https://github.com/manuelarr03/NequiTestDataEngineer/blob/256586d0374b402d42a930b0827cf6d35c81ebe9/Edad.png)

Se crea una ETL job para los datos de *db-songs*, para este caso, se cargan las dos tablas y se procesan al mismo tiempo, se cambian formatos y se hace un inner join entre las dos tablas para obtener una única tabla de salida.

![ETL songs](https://github.com/manuelarr03/NequiTestDataEngineer/blob/b687503a532bb7392d59596c5ab71d85084131cc/ELT%20job%20songs.png)

El siguiente paso es el almacenamiento de la data, para esto, se crea un nuevo bucket *kg-data-processed* y dentro de el, una carpeta por base de datos. Se almacenan los nuevos datos en formato parquet y comprimido por snappy. Se define este tipo de almacenamiento ya que se ahorran costos por almacenamiento.

![BuketProcessed](https://github.com/manuelarr03/NequiTestDataEngineer/blob/b687503a532bb7392d59596c5ab71d85084131cc/BucketProcessed.png)

![Data procesada](https://github.com/manuelarr03/NequiTestDataEngineer/blob/256586d0374b402d42a930b0827cf6d35c81ebe9/Data%20procesada.png)

## Paso 5: Completar la redacción del proyecto

El objetivo del proyecto es la construcción de un modelo y una arquitectura eficiente para que los datos puedan ser consumidos.  

* Si los datos se incrementaran en 100x. sería buena opción particionar la data ya que las consultas a bases particionadas son mucho más rápidas y efectivas, se tiene mayor escalabilidad en el almacenamiento y reduce costos por optimización de almacenamiento. O utilizar el servicio de AWS Redshift que permite la creación de un data warehouse y analizar grandes cantidades de datos. 

* Si se requiere hacer analítica en tiempo real, ¿cúales componentes cambiaría a su
arquitectura propuesta? Cambiaría el almacenamiendo de datos s3 por Redshift ya que este servicio permite el análisis de datos en tiempo real.
