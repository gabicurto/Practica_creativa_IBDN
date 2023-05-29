# Practica_creativa_IBDN_ Lucía_Martínez__Gabriela_Curto
A continuación mostramos las indicaciones seguidas para la realización de la práctica. La práctica se dividirá en tres escenarios principales, el primero, donde nos limitamos a seguir los pasos propuestos en el GitHub proporcionado, la ejecución del Job de predicción con Spark-Submit, y el apartado opcional de entrenar el modelo propuesto con Apache Airflow.

En el segundo escenario pasaremos a Dockerizar cada uno de los servicios, Mongo, Kafka, Zookeper, Spark y el web server. Para ello, diseñamos un dichero docker-compose para crear las imagenes correspondientes a cada contenedor. Adicionalmente, los contenedores de Mongo y Webserver los creamos mediante un fichero Dockerfile al que luego referenciaremos en el fichero docker compose. Una vez tenemos el escenario completamente dockerizado, y comprobado que se hace correctamente, pasamos a desplegar este mismo escenario completo en la plataforma Google Cloud.

El tercer escenario es el correspondiente a Kubernetes, se nos pide ahora desplegar de nuevo el escenario completo pero en este caso usando Kubernetes.


# Prediccion de vuelos
El objetivo de este proyecto es implementar un sistema que permite realizar predicciones de retraso de vuelos. Dicho sistema de predicción está formado por una serie de módulos los cuales permiten realizar predicciones analíticas y en tiempo real a partir de una serie de trazas y así poder mostrar el retraso del correspondiente vuelo. 
Básicamente el sistema funciona de la siguiente manera:

- Se descarga el dataset de los datos relacionados con los vuelos con información  para  entrenar el modelo y predecir los retrasos.
- Se entrena el modelo de Machine Learning a partir del dataset.
- Se despliega el job de predicción de retrasos de los vuelos Spark, que  las predicciones mediante el modelo creado
- Se introducen los datos del vuelo a predecir en el frontal web y se envían al servidor web de Flask por medio de la cola de mensajería Kafka especificando el tópic.
- Se entrena el modelo predictivo empleando el algoritmo RandomForest con los datos obtenidos.
- El job de Spark en el servidor realiza la predicción de los retrasos de los vuelos por medio de los datos del tópic al que se encuentra suscrito de Kafka.
- La ejecución del job se realiza por medio del fichero jar para Scala generado por medio de spark-submit.
- Se guardan las predicciones en la base de datos de Mongo.
- Se realiza la consulta de los resultados de la predicción a través del uso de polling que flask realiza sobre Mongo y se se muestran en el servidor web.


# PASOS SEGUIDOS PARA EL FUNCIONAMIENTO DE LA PRÁCTICA:

