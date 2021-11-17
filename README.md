# PRACTICA_BDFI_FINAL1

### IMPORTANTE
Los archivos resultantes de la ejecución de este proyecto se pueden encontrar en la rama **master** de este mismo repositorio.

#### Autores
DANIEL MIGUEL FERNÁNDEZ

CARLOS GARCÍA BORREL

CURSO 2021/2022

ASIGNATURA: BDFI

MÁSTER EN INGENIERÍA DE TELECOMUNICACIÓN (MUIT)

El código base y el guión que se ha seguido para la realización de esta práctica se encuentra en la siguiente url:
- https://github.com/ging/practica_big_data_2019.git.

En este repositorio se hará un resumen de los procedimientos y de los detalles acerca de cómo se han realizado las diferentes partes.

## Recomendaciones importantes 

- Se recomienda realizar la ejecución de este proyecto en sistemas basados en el kernel de Linux como es el propio Linux o los dispositivos Mac.
- Para el caso de Windows se recomienda ejecutar Linux en una máquina virtual con Ubuntu 20.04 con VirtualBox. Algunos comandos podrían funcionar con la ventana de comandos de git bash pero nos dio problemas y tuvimos que buscar otra solución.

  ### Instalación máquina Virtual Linux
    - [Explicación](https://br.atsit.in/es/?p=31178)
    - [Imagen Ubuntu](https://ubuntu.com/download/desktop) (versión 20.04 LTS(.iso))
    - [VirtualBox](https://www.virtualbox.org/)

Se ha realizado el proyecto sobre la máquina virtual de Ubuntu 20.04 LTS donde ya venia instalado python 3.8

## Instalación

Instalar los siguientes componentes:

 - [Intellij](https://www.jetbrains.com/help/idea/installation-guide.html) (jdk_1.8)
 - [Python3](https://realpython.com/installing-python/) (Suggested version 3.7)
   - Para nuestro caso (tener ya instalado python 3.8) hemos realizado 3 comandos:
   
     ````
     sudo add-apt-repository ppa:deadsnakes/ppa
     sudo apt-get update
     sudo apt-get install python3.7
     ````
   - Como ahora disponemos de dos versiones de python cada vez que invoquemos un comando deberemos poner python3.7 ... 
 - [PIP](https://pip.pypa.io/en/stable/installing/)
    - Instalar con python3.7, así se nos asocia a él.
    - Comprobar versión y comprobar la version de python asociada con:
      ````
      pip -V
      ````
      o
      ````
      pip --version
      ````
 - [SBT](https://www.scala-sbt.org/release/docs/Setup.html) 
 - [MongoDB](https://docs.mongodb.com/manual/installation/)
   - Ejecucuón del .dev mediante el instalador de paquetes de Ubuntu. 
 - [Spark](https://spark.apache.org/docs/latest/) (Mandatory version 3.1.2)
    - No hace falta instalar nada solo descomprimir el paquete.   
 - [Scala](https://www.scala-lang.org) (Suggested version 2.12)
   - Ejecucuón del .dev mediante el instalador de paquetes de Ubuntu. 
 - [Zookeeper](https://zookeeper.apache.org/releases.html)
    - No hace falta instalar nada solo descomprimir el paquete. 
 - [Kafka](https://kafka.apache.org/quickstart) (Mandatory version kafka_2.12-3.0.0)
    - No hace falta instalar nada solo descomprimir el paquete.  
 
 ### Instalación de los requirements de python
 
 Tras la instalación:
 
 ```
 pip install -r requirements.txt
 ```

## Código

Debe clonarse mediante el comando:

```
git clone https://github.com/ging/practica_big_data_2019.git
```
Tras esto nos movemos al repositorio clonado en local mediante el comando:

```
cd practica_big_data_2019
```
y ejecutamos el siguiete comando para descargar los recursos que posteriormente utilizaremos para la predicción:

```
resources/download_data.sh
```

 
 ### Iniciar Zookeeper
 
 Abrimos una consola en la carpeta de Kafka descomprimida y ejecutamos:
 
 ```
 bin/zookeeper-server-start.sh config/zookeeper.properties
 ```
  ### Iniciar Kafka
  
  Abrimos una consola en la carpeta de Kafka descomprimida y ejecutamos:
  
  ```
  bin/kafka-server-start.sh config/server.properties
  ```
   Crear el sigueinte topic con:
   
  ```
  bin/kafka-topics.sh \
    --create \
    --bootstrap-server localhost:9092 \
    --replication-factor 1 \
    --partitions 1 \
    --topic flight_delay_classification_request
  ```
   Debería devolver:
   
  ```
  Created topic "flight_delay_classification_request".
  ```
  Podemos ver los topics que existen con:
  
  ```
  bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
  ```
  Nos debería salir:
  
  ```
  flight_delay_classification_request
  ```

  ## Importar distancias a la BD de MongodB
  
  Comprobamos si mongo está funcionando con:
  
  ```
  service mongod status
  ```
  Salida:
  
  ```
  mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor prese>
     Active: active (running) since Tue 2021-11-16 17:20:59 CET; 5s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 1814 (mongod)
     Memory: 197.8M
     CGroup: /system.slice/mongod.service
             └─1814 /usr/bin/mongod --config /etc/mongod.conf

nov 16 17:20:59 carlosgborrel-VirtualBox systemd[1]: Started MongoDB Database S>
  ```
  En el caso de que no esté corriendo, ejecutamos los siguientes comandos para iniciarlo:
  
  ````
  service mongod start
  ````
  ````
  service mongod status
  ````
  Ejecutamos el script import_distances.sh:
  
  ````
  ./resources/import_distances.sh
  ```` 
  Salida:
  
  ```
2021-11-16T17:23:32.868+0100	connected to: mongodb://localhost/
2021-11-16T17:23:33.192+0100	4696 document(s) imported successfully. 0 document(s) failed to import.
MongoDB shell version v4.4.8
connecting to: mongodb://127.0.0.1:27017/agile_data_science?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("9457ef1d-c660-486d-aa1b-50586ee5cb4c") }
MongoDB server version: 4.4.8
{
	"numIndexesBefore" : 2,
	"numIndexesAfter" : 2,
	"note" : "all indexes already exist",
	"ok" : 1
}
  ```
  
  ## Entrenar y guardar el modelo con PySpark mllib
  
  Nos movemos al repositorio clonado en local mediante el comando:
  
  ```
  cd practica_big_data_2019
  ```
  Establecemos la variable de entorno "JAVA_HOME" con la ruta del directorio donde tenemos instalado java:
  
  ```
  export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
  ```
  Asímismo, establecemos la variable de entorno "SPARK_HOME" con la ruta de la carpeta donde hemos descargado Spark:
  
  ```
  export SPARK_HOME=/home/daniel/Documentos/spark-3.1.2-bin-hadoop3.2
  ```
  Ejecutamos el script `train_spark_mllib_model.py`:
  
  ```
  python3.7 resources/train_spark_mllib_model.py .
  ```
  Como resultado podemos comprobar como han aparecido una serie de carpetas en directorio "models":
  
  ```
  ls models
  
  ```  
  
  ## Ejecutar el predictor de vuelos
  
  En la clase de scala "MakePrediction" cambiamos la siguiente línea por medio de un editor de código (IntellJ) añadiendo la ruta donde hemos clonado el repositorio:
  
  ```
  val base_path= "/home/daniel/Documentos/practica_big_data_2019"
  ``` 
  Ejecutamos el código usando spark-submit con los siguientes comandos desde la carpeta de spark: 
  
  ```
  //Compilar y construir un archivo JAR usando sbt (se guarda en practica_big_data_2019/flight_prediction/target)
  sbt clean
  sbt package
  
  //Ejecutar comando spark-submit
  bin/spark-submit --class es.upm.dit.ging.predictor.MakePrediction --packages org.mongodb.spark:mongo-spark-connector_2.12:3.0.0,org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.0 /home/daniel/Documentos/practica_big_data_2019/flight_prediction/target/scala-2.12/flight_prediction_2.12-0.1.jar
  ``` 
  - bin/spark-submit  -> Comando de spark para ejecutar Make prediction
  - --class <main-class>  -> Clase .scala que queremos ejecutar
  - --packages <packages> -> Paquetes y versiones que vamos a utilizar de los diferentes componentes 
	- Mongo 5.0
	- Spark 3.1.2
	- Kafka_2.12-3.0.0
  - <path.jar> -> Ejecutable de la aplicación de predicción de vuelos
  
  ## Realizar la petición de predicción desde la aplicación Web
  
  Establecemos la variable de entorno "PROJECT_HOME" con la ruta donde hemos clonado el repositorio:
  
  ```
  export PROJECT_HOME=/home/daniel/Documentos/practica_big_data_2019
  ```
  Accedemos al directorio "web" dentro del directorio "resources" y ejecutamos el archivo "predict_flask.py":
  ```
  cd practica_big_data_2019/resources/web
  python3.7 predict_flask.py
  
  ```
Visitamos el siguiente enlace: http://localhost:5000/flights/delays/predict_kafka y podemos comprobar cómo podemos realizar las peticiones de predicción dando un valor a los distintos parámetros y cómo el sistema nos devuelve la predicción efectuada.
  
  
  ## Comprobar las predicciones insertadas en MongoDB
  
  ```
   $ mongo
   > use use agile_data_science;
   > db.flight_delay_classification_response.find();
  ```
  Como resultado obtenemos algo como lo siguiente:
  
  ```
  { "_id" : ObjectId("5d8dcb105e8b5622696d6f2e"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 290, "Timestamp" : ISODate("2019-09-27T08:40:48.175Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "8e90da7e-63f5-45f9-8f3d-7d948120e5a2", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 3 }
  { "_id" : ObjectId("5d8dcba85e8b562d1d0f9cb8"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 291, "Timestamp" : ISODate("2019-09-27T08:43:20.222Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "d3e44ea5-d42c-4874-b5f7-e8a62b006176", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 3 }
  { "_id" : ObjectId("5d8dcbe05e8b562d1d0f9cba"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2019-09-27T08:44:16.432Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "a153dfb1-172d-4232-819c-8f3687af8600", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 1 }


```

### Train the model with Apache Airflow (optional)

- The version of Apache Airflow used is the 2.1.4 and it is installed with pip. For development it uses SQLite as database but it is not recommended for production. For the laboratory SQLite is sufficient.

- Install python libraries for Apache Airflow (suggested Python 3.7)

```shell
cd resources/airflow
pip install -r requirements.txt -c constraints.txt
```
- Set the `PROJECT_HOME` env variable with the path of you cloned repository, for example:
```
export PROJECT_HOME=/home/user/Desktop/practica_big_data_2019
```
- Configure airflow environment

```shell
export AIRFLOW_HOME=~/airflow
mkdir $AIRFLOW_HOME/dags
mkdir $AIRFLOW_HOME/logs
mkdir $AIRFLOW_HOME/plugins

airflow users create \
    --username admin \
    --firstname Jack \
    --lastname  Sparrow\
    --role Admin \
    --email example@mail.org
```
- Start airflow scheduler and webserver
```shell
airflow webserver --port 8080
airflow sheduler
```
Vistit http://localhost:8080/home for the web version of Apache Airflow.

- The DAG is defined in `resources/airflow/setup.py`.
- **TODO**: add the DAG and execute it to train the model (see the official documentation of Apache Airflow to learn how to exectue and add a DAG with the airflow command).
- **TODO**: explain the architecture of apache airflow (see the official documentation of Apache Airflow).
- **TODO**: analyzing the setup.py: what happens if the task fails?, what is the peridocity of the task?

![Apache Airflow DAG success](images/airflow.jpeg)





