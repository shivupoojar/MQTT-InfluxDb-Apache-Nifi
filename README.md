
# Storing an IoT data into InlfuxDB database server using Apache NiFi

Th objective of the project is to collect the IoT sensed data from sensors using light weight MQTT protocol and insert into InfluxDB time series database using Apache NiFi processors.
 The following are the requirements to run the project:
 * Apache Nifi
 * MQTT server
 * InfluxDb database server
 
 
## Installing an Apache NiFi 
Apache NiFi can be installed in the system using docker container :

```
sudo docker run --name nifi \
  -p 8080:8080 \
  -p 8081:8081 \
  -d \
  -e NIFI_WEB_HTTP_PORT='8080' \
  apache/nifi:latest
  ```
  
  ## Installing MQTT server 
MQTT server can be installed on the same machine :
Mosquitto runs at port number:1883, access url could be tcp://INSTANCE_IP:1883
 ```
 sudo apt-get install mosquitto
 ``` 
 ```
 sudo apt-get install mosquitto-clients
 ``` 
## Installing the InfluxDB server :
Install the influxDB server in docker container :
```
docker run -p 8086:8086 \
      -v $PWD:/var/lib/influxdb \
      influxdb
 ```
Create the database with name "iot" using influxDB cli.
* To access the influxdb cli exec to influxdb docker container 
```
sudo docker exec -it 31c9e202139f /bin/bash
```
* Access the influx cli
```
influx
```
* Create a database with name "iot"
```
create database iot
use iot
```

* Insert a sample data using influxDB query
```
INSERT temperature,machine=unit42,type=assembly external=89,internal=197
```
The query consists of Measurement, tags and fields

Measurements = temperature

Tags = machine, type

Fields= external,internal

## Creating Data pipeline using Apache NiFi
Access the NiFi user interface using http://ip:8086/nifi. Here we are creating two processor groups 

* Generate a sample iot data and publish as a MQTT topic 
* Consume MQTT topic and insert in to influxDB
![Overview of two Processors ](/processors-nifi.png)

### Creating a processor group to generate a sample iot data and publish as a MQTT topic 
The overall processor group looks like:
![Overview of two Processors ](/processor1.png)


* The GenerateFlowFile processor can be used to generate a sample iot data. 
Modify the property:= Custom Text: external=${random():mod(50):plus(50)},internal=${random():mod(200):plus(150)}
![GenerateFlowFile ](/GenerateFlowFile.png)

* Publish flowfile through MQTT topic
Modify the properties:=
Broker URI:tcp://ip:1883,client ID: you can choose freely , Topic:Data
![GenerateFlowFile ](/PublishMQTT.png)

### Creating a processor group to consume MQTT topic and insert into Inlfuxdb 

* ConsumeMQTT processor can be used to consume the MQTT topic. Modify the following properties such as Broker URL:tcp://ip:1883,client ID: you can choose freely , Topic:Data

![ConsumeMQTT ](/ConsumeMQTT.png)

* Format the data according to the line protocol structure of influxDB.
We are using ReplaceText processor to append the measurement name, tag values to flowfile recieved in the pipeline.

![ReplaceText ](/ReplaceText.png)

* PutInfluxdb processor can be used to insert data in to influxdb and modify the properties such as Database name:iot, Influxdb connection URL: http://ip:8086

![Influxdb ](/influxdb.png)



