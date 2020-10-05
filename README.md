# Elastic Search Logstash & Kibana

https://www.docker.elastic.co/

## Inicio
```bash
mkdir ELK-example
cd ELK-example
docker network create -d bridge mynetwork   
```

### 1.- Crear la red 

```bash
docker network create -d bridge mynetwork   
```
## 2.- Levantar el servicio de ElasticSearch

```bash
docker run \
-p 9200:9200 -p 9300:9300 \
--net=mynetwork \
--name elasticsearch \
-e "discovery.type=single-node" \
docker.elastic.co/elasticsearch/elasticsearch:7.6.2
```

### 3.- Levantar el servicio de Logstash 
### 3.1 Hacer el archivo de configuración logstash.conf
```bash
mkdir pipeline
touch pipeline/logstash.conf
vim pipeline/logstash.conf
```

```bash
input {
 stdin {}
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
  }
}
```
### 3.2 Levantar el servicio de Logstrash para terminal

```bash
docker run -it \
--rm \
--net=mynetwork \
--name logstash \
-v "$PWD/pipeline":/app \
--entrypoint bash \
docker.elastic.co/logstash/logstash:7.6.2 
```
### 3.3 Cargar el archivo de configuración

```bash
logstash -f /app/logstash.conf
```
### 3.4.-  Revisar nuestros datos en

http://localhost:9200/_search?pretty&size=1000
 ó
```bash
 wget http://localhost:9200/_search?pretty&size=1000
```

### 4.- Levantar el servicio de Kibana 

```bash
docker run --rm \
--name kibana \
--net=mynetwork \
-p 5601:5601 \
kibana:7.6.2
```

### 5.- Revisar nuestros servicios de ELK en la red mynetwork
```bash
docker inspect mynetwork
```

### 6.- Revisión transmision de datos

```bash
curl http://localhost:9200/_cat/indices\?v
```



### 7.- Cargar a un servicio de Logstrash informacíon de un CSV 

#### 7.1 Obtener el .csv y guardarlo en el directorio csv
```bash
mkdir csv
```
https://www.kaggle.com/ptoscano230382/air-bnb-ny-2019

#### 7.2 hacer el archivo de configuracion para Logstash

```bash
mkdir csv
vim csv/logstash.conf
```

```bash
input {
 	file{
		path => "/app/*.csv"
		start_position => "beginning"
		sincedb_path => "NULL"
	}
}

filter{
	csv{
		separator => ","
		columns => ["id","name","host_id","host_name","neighbourhood_group","neighbourhood","latitude","longitude","room_type","price","minimum_nights","number_of_reviews","last_review","reviews_per_month","calculated_host_listings_count","availability_365"]
	}
}

output {
    elasticsearch {
			    hosts => ["elasticsearch:9200"]
			    index => "ab_nyc_2019"
			  }
    stdout{}
}

```
#### 7.3 Levantar el servicio de Logstash
```bash
docker run -it \
--rm \
--net=mynetwork \
--name logstash_CSV \
-v "$PWD/csv":/app \
--entrypoint bash \
docker.elastic.co/logstash/logstash:7.6.2 
```
#### 7.2 Correr el archivo de configuración

```bash
logstash -f /app/logstash.conf
```
#### 7.2 Ver nuestra información en kibana

http://localhost:5601/




License
----

MIT


**Free Software, Hell Yeah!**


