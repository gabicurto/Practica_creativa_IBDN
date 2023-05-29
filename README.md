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


# Pasos seguidos para el funcionamiento de la práctica
1.	Clonamos el repositorio:
```
git clone https://github.com/Big-Data-ETSIT/practica_creativa.git
```

2. Descargamos los datos:
```
resources/download_data.sh
```
3. Instalamos todos los componentes incluidos en la arquitectura de la práctica

4. Creamos y usamos el entorno de Python:
```
python3 -m venv env
source env/bin/activate
```
5. Instalamos librerías necesarias
```
pip install -r requirements.txt
```
6. Abrimos la consola y vamos al directorio de descarga de Kafka y ejecutamos el siguiente comando para iniciar Zookeeper:
```
bin/zookeeper-server-start.sh config/zookeeper.properties
```
 7. Después, en otra terminal, volvemos a acceder al directorio de descarga y ejecutamos el siguiente comando para iniciar Kafka:
```
bin/kafka-server-start.sh config/server.properties
```
8. En este mismo directorio, creamos un nuevo topic mediante:
```
bin/kafka-topics.sh \
      --create \
      --bootstrap-server localhost:9092 \
      --replication-factor 1 \
      --partitions 1 \
      --topic flight_delay_classification_request
 ```    
 Y obtenemos el resultado esperado:
 ![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/22ed2616-e276-4380-a163-c103a7ed7abc)
 
 También comprobamos que se ha creado correctamente el topic viendo la lista de topics disponible:
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/47419f93-ec7a-4938-93ca-d80db2a7c3e8)

Abrimos una nueva consola con un consumidor para ver los mensajes mandados a ese topic
```
bin/kafka-console-consumer.sh \
    --bootstrap-server localhost:9092 \
    --topic flight_delay_classification_request \
    --from-beginning
```


 ## Import the distance records to MongoDB
 1. Primero comprobamos que tenemos Mongo y que se está ejecutando
 ![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/244c1cb3-4256-4b7c-9949-e1cffc1b2203)
 
 2. Ejecutamos el script import_distances realizando la siguiente modificación para su correcto funcionamiento
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/d9131407-7243-4ff0-af62-d91d8bde82aa)
- Obtenemos un resultado diferente al proporcionado en github debido a la versión instalada de mongo, que en nuestro caso ha sido mongosh.
- Podemos comprobar que se importaron 4696 documentos correctamente y se crearon los índices esperados en la colección

![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/c9af6ab8-796a-4965-ad8d-ff7460ae232b)


## Train and Save de the model with PySpark mllib
1.Establecemos la variable de entorno JAVA_HOME con la ruta del directorio de instalación de Java, y establecemos la variable de entorno SPARK_HOME con la ruta con la ruta del directorio de instalación de Spark.
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/f1884a99-27e7-480b-b804-54f41df11240)
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/286cf26b-1486-48bd-9393-b9ac5091baf1)

2. Por último, ejecutamos el script

![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/40b3e0cb-b405-4a24-988f-70e4b56872ca)

![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/29b6af82-2370-4072-b3c2-dc7eda2f8db9)

![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/518709df-6386-4601-a5c1-68f8a59aab13)

Y comprobamos los ficheros que se han guardado en la carpeta models:
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/2170bc47-75f3-43e7-97f2-5dacd90abd67)


## Run Flight Predictor con spark-submit
1.Primero debemos cambiar el valor ‘base_path’ en la clase scala MakePrediction, a la ruta donde se encuentra nuestro repositorio de clones:

Modificamos primero en la clase MakePrediction de scala:
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/09265e47-a308-4683-a4ba-9adc9ada04ff)

Ejecutamos el código utilizando spark-submit.
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/8d17aaf9-e681-4f8b-a652-a9a23733bd50)

![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/832e8fea-3818-4d31-add9-9db542325527)

Adicionalmente visualizamos la consola de Kafka, iniciada previamente en apartados anteriores:
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/39d810f6-7871-4971-90d9-ff1c8f6b9ec4)

## Start the prediction request Web Application
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/7c6477e8-1bfe-40ce-a1ad-8528f295661f)
1. Establecemos la variable de entorno PROJECT_HOME con la ruta de nuestro repositorio clonado y vamos al directorio ‘web’ en resources y ejecutamos el archivo de aplicación web flask predict_flask.py. 
2. Visualizamos los resultados que se muestran en el navegador, para ver si se hace la predicción, y efectivamente podemos verlo
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/1cd048d4-772d-4efd-8476-c6078a77951a)
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/187eae20-ef59-45a8-bf9e-3f6a8f012778)

Observamos la salida de depuración en la consola JavaScript mientras el cliente realiza repetidamente consultas o solicitudes a un punto final de respuesta para obtener información actualizada.

![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/d8c267bc-67a9-4ecb-b6cd-cf03b543b06a)

Como información adicional, podemos visualizar abriendo una nueva consola con un consumidor para ver los mensajes enviado a a ese topic
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/4de19ac0-af76-46cc-bfcb-c9af0bde505b)

## Check the predictions records inserted in MongoDB
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/9bc8110c-57c1-43af-9bd5-c44eea79beb6)
Nuestra salida se ve de la siguiente manera:
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/d80e1953-717b-4f8d-823c-88c4e7828802)


# Dockerizar cada uno de los servicios que componen la arquitectura completa y desplegar el escenario completo usando docker-compose
Pasamos al segundo escenario de la práctica, en el que dockerizamos los servicios para su posterior despliegue en Google Cloud. Cre

# Entrenar el modelo con Apache Airflow
1. Instalamos las depencias de Apache Airflow
```
cd resources/airflow
pip install -r requirements.txt -c constraints.txt
```
![image](https://github.com/gabicurto/Practica_creativa_IBDN/assets/127130231/1ef2c3e8-1666-4d91-8fb9-413701628cb4)







