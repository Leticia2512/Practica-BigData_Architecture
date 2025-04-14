# Práctica Módulo Big Data Arquitectura
### Bootcamp Big Data, Machine Learning & IA_Keepcoding
---

Este repositorio contiene el desarrollo de un ejercicio práctico en el que conectamos **Hadoop** con **ElasticSearch**, estableciendo una integración entre Hive y un índice de **ElasticSearch**, desarrollando así competencias clave en arquitectura Big Data y análisis de datos en tiempo real. Para todo esto se ha utilizado **Google Cloud Platform** como entorno principal.

---

## 🎯 Objetivos de la Práctica

El objetivo de esta práctica es entender y aplicar los fundamentos de la integración entre sistemas de Big Data y motores de búsqueda, específicamente mediante la conexión de Hadoop con **ElasticSearch**. A través de este ejercicio se configura un entorno completo, desde la creación de clusters y servidores hasta la realización de consultas y la visualización de datos. 

---

## 🧩 Desarrollo de la Práctica 

**PARTE 1: CONFIGURACIÓN ES-HADOOP**

En esta sección, configuramos la integración entre **ElasticSearch**  y **Hadoop**, utilizando las siguientes herramientas:

* Cluster standalone o estándar en Dataproc.

* ELK Stack: ElasticSearch, Logstash y Kibana.

Pasos principales:

1. Crear un cluster **Hadoop**.

2. Descargar y subir los JARs necesarios al cluster:

Los jars de configuración de elastichsearch-hadoop y commons-httpclient aquí 👉🏻 : [elasticsearch-hadoop](https://artifacts.elastic.co/downloads/elasticsearch-hadoop/elasticsearch-hadoop-8.14.1.zip), [commons-httpclient]( https://repo1.maven.org/maven2/commons-httpclient/commons-httpclient/3.1/commons-httpclient-3.1.jar).

Creamos un nuevo bucket para nuestro ES-Hadoop y subimos ambos jar a nuestro bucket en google storage. 

3. Descarga de los jar desde el bucket al filesystem del cluster: desde la consola SSH del nodo maestro del cluster los subimos con los siguientes comandos:

```
gsutil cp gs://bucket-para-elastic/jars/elastic/elasticsearch-hadoop-8.14.1.jar .

gsutil cp gs://bucket-para-elastic/jars/elastic/commons-httpclient-3.1.jar .
```
  

**PARTE 2: CONFIGURACIÓN DEL SERVIDOR ELASTICSEARCH**

Se implementa y configura un servidor **ElasticSearch** con los siguientes pasos:

1. Crear una nueva máquina virtual para **ElasticSearch**.
   
2. Descargar e instalar **ElasticSearch** y **Kibana**:

Primero, instalamos wget, que nos permitirá descargar cualquier instalable (lanzaremos los comandos como administrador):

```
sudo apt install wget
```

Descargamos las ultimas versiones de elastic y kibana:

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.14.1-amd64.deb
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.14.1-amd64.deb
```

Procedemos a instalar y configurar ambos:

```
sudo dpkg -i elasticsearch-8.14.1-amd64.deb
sudo dpkg -i kibana-8.14.1-amd64.deb
```

Para versiones anteriores a *Elasticsearch 8* añadimos lo siguiente para que no acepte solo conexiones locales, sino desde cualquier IP:

```
sudo sed -i -e '$ahttp.host: 0.0.0.0' /etc/elasticsearch/elasticsearch.yml
sudo cat /etc/elasticsearch/elasticsearch.yml
```

```
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Indicamos "false" en las siguientes variables:

*xpack.security.enabled: false*

*xpack.security.http.ssl:enabled: false*


Para configurar **Kibana**, independientemente de la versión:

```
sudo sed -i -e '$aserver.host: 0.0.0.0' /etc/kibana/kibana.yml
sudo cat /etc/kibana/kibana.yml
```

Finalmente, arrancamos (reiniciamos) tanto **ElasticSearch** como **Kibana**:

```
sudo service elasticsearch restart
sudo service kibana restart
```

3. Configurar las reglas de firewall para abrir los puertos 9200 y 5601.

4. Comprobar la conexión entre el cluster **Hadoop** y **ElasticSearch**:

```
curl -X GET "http://IP-MAQUINA-ESLASTIC:9200"
```


**PARTE 3: CONEXIÓN DEL CLUSTER HADOOP CON ELASTICSEARCH**

Configuramos Hive para integrarse con ElasticSearch:

1. Modificar el archivo hive-site.xml con las propiedades necesarias y cargar los JARs en **Hive**:

```
sudo sed -i '$d' /etc/hive/conf.dist/hive-site.xml

sudo sed -i '$a \  <property>\n    <name>es.nodes</name>\n    <value>IP-ELASTIC</value>\n  </property>\n' /etc/hive/conf.dist/hive-site.xml

sudo sed -i '$a \  <property>\n    <name>es.port</name>\n    <value>9200</value>\n  </property>\n' /etc/hive/conf.dist/hive-site.xml

sudo sed -i '$a \  <property>\n    <name>es.nodes.wan.only</name>\n    <value>true</value>\n  </property>\n' /etc/hive/conf.dist/hive-site.xml

sudo sed -i '$a \  <property>\n    <name>hive.aux.jars.path</name>\n   <value>/usr/lib/hive/lib/elasticsearch-hadoop-8.14.1.jar,/usr/lib/hive/lib/commons-httpclient-3.1.jar</value>\n  </property>\n</configuration>' /etc/hive/conf.dist/hive-site.xml

sudo cp elasticsearch-hadoop-8.14.1.jar /usr/lib/hive/lib/
sudo cp commons-httpclient-3.1.jar /usr/lib/hive/lib/

```

2. Reiniciar **Hive** desde SSH del nodo maestro del cluster de Hadoop para que se apliquen los cambios.


**PARTE 4: CONECTAR DATOS**

Desde el server ElasticSearch:

1. Crear un índice llamado *alumnos*:

```
curl -X POST "localhost:9200/alumnos/_doc/6" -H 'Content-Type: application/json' -d'
{
  "title": "New Document",
  "content": "This is a new document for the master class",
  "tag": ["general", "testing"]
}
'
```

2. Insertar documentos en el índex *alumnos* desde **Hadoop** usando comandos CURL:

```
curl -X POST "IP-SERVER-ELASTICSEARCH:9200/_bulk" -H 'Content-Type: application/json' -d'
{ "index": { "_index": "alumnos", "_id": "3" } }
{ "id": 3, "name": "Carlos", "last_name": "González" }
{ "index": { "_index": "alumnos", "_id": "4" } }
{ "id": 4, "name": "María", "last_name": "López" }
{ "index": { "_index": "alumnos", "_id": "5" } }
{ "id": 5, "name": "Luis", "last_name": "Martínez" }
{ "index": { "_index": "alumnos", "_id": "7" } }
{ "id": 7, "name": "Sofía", "last_name": "Ramírez" }
{ "index": { "_index": "alumnos", "_id": "8" } }
{ "id": 8, "name": "Pedro", "last_name": "Hernández" }
'
```

3. Realizar una consulta para ver los datos insertados:

```
curl -X GET "http://IP-SERVER-ELASTIC:9200/alumnos/_search?pretty"
```


**PARTE 5: Visualización con **Kibana****

Creamos un sencillo dashboard con **Kibana** para visualizar datos del index *alumons*.

Para ello desde nuestro navegador, entramos en: **http://IP_ELASTIC_SERVER:5601**, y en **Analitycs** hacemos click en **Dashboards**. A continuación, hacemos click en **Create Dataview**, dende selecionamos el index *alumnos*. 

Por último, en **Create Dashboard** y **Create Visualization** encontramos una variedad de gráficos que podemos elegir para la visulización que mejor se ajuste a las características de los datos. En este caso he elegido un gráfico de anillo.


El documento PDF con las capturas de las diferentes partes de la práctica se pueden ver en este enlace 👉🏻: [Práctica ElacticSearch_Hadoop.pdf](https://github.com/Leticia2512/Practica-BigData_Architecture/blob/main/Pra%CC%81ctica%20ElacticSearch_Hadoop.pdf).

___


## 🛠️ Lenguajes y Herramientas Utilizadas

- **Hadoop**: Framework empleado para el procesamiento y almacenamiento distribuido de grandes volúmenes de datos.

- **Hive**: Herramienta utilizada para la consulta y análisis de datos estructurados en el entorno de Hadoop.

- **ElasticSearch**: Motor de búsqueda y análisis utilizado para crear índices y realizar búsquedas en grandes cantidades de datos.

- **Kibana**: Plataforma de visualización empleada para analizar los datos almacenados en ElasticSearch.

- **Google Cloud Platform (GCP)**: Entorno en la nube para la implementación y gestión de las herramientas del proyecto.

- **Python**: Lenguaje de programación empleado para la integración y scripts complementarios.

___


## 🔗 Recursos útiles 

- [Documentación ElasticSearch-Hadoop](https://www.elastic.co/guide/en/elasticsearch/hadoop/index.html)

- [Logstash](https://www.elastic.co/logstash)

- [Kibana](https://www.elastic.co/kibana)

