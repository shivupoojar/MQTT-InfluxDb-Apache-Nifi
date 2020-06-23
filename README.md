
# Storing a IoT data into InlfuxDb database using Apache Nifi

Th objective of the project is to collect the IoT sensed data from sensors using light weight MQTT protocol and insert into InfluxDb time series database using Apache Nifi processors.
 The following are the requirements to run the project:
 * Apache Nifi
 * MQTT server
 * InfluxDb database server
 
 
## Installing an Apache Nifi 
Apache Nifi can be installed in the system, probably using docker container :

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
Mosquitto runs at port number:1883, access urll could be tcp://INSTANCE_IP:1883
 ```
 sudo apt-get install mosquitto
 ``` 
 ```
 sudo apt-get install mosquitto-clients
 ``` 
## Installing the InfluxDb server :
Install influxdb server in docker container :
```
docker run -p 8086:8086 \
      -v $PWD:/var/lib/influxdb \
      influxdb
 ```
Create the database with name iot using influxdb cli.
* To access the influxdb cli exec to influxdb docker container 
```
sudo docker exec -it 31c9e202139f /bin/bash
```
* Access the influx cli
```
influx
```
* Create a database with name iot
```
create database iot
use iot
```

* Insert a sample data as
```
INSERT temperature,machine=unit42,type=assembly external=89,internal=197
```
The query consists of Measurement, tags, fields

Measurements = temperature
Tags = machine, type
Fields= external,internal

## Creating Data pipeline using Apache Nifi
Access the Nifi user interface using http://<ip>:8086/nifi. Here we are creating two processor groups 
* Genearte a data and publish as a MQTT topic 
* Consume MQTT topic and insert in to influxdb


### Creating a processor group- Genearte a data and publish as a MQTT topic 

