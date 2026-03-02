---
title: Unitat 07.03. Logstash.
titlepage: true
titlepage-rule-height: 0
titlepage-rule-color: 757575
titlepage-text-color: 757575
titlepage-background: ../template/portada.png
subtitle:
author: Fidel Oltra
toc: true
toc-own-page: true
toc-title: Continguts
linkcolor: magenta
listings-no-page-break: true
header-includes: \usepackage{lastpage}
header-left: Unitat 07.03. Logstash.
header-right: Curs 2025-2026
footer-left: IES Jaume II el Just - CE IA i Big Data
footer-right: \thepage/\pageref{LastPage}
---

# LOGSTASH

**Logstash** és una eina de processament de dades en temps real, oberta i gratuïta, que forma part de la **suite ELK** (Elasticsearch, Logstash, Kibana). 

**Logstash** permet la ingesta, transformació i enviament de dades des de diverses fonts cap a una destinació, com ara **Elasticsearch**. Té un format de ***pipeline*** que se pot configurar d'una manera bastant senzilla i flexible per a poder ingestar dades de diferents fonts, aplicar filtres i enviar el resultat a la destinació requerida.

Els arxius de **Logstash** estan disponibles per a descarregar en http://www.elastic.co/downloads. Nosaltres, de totes formes, anem a seguir treballant en Docker i alçarem **Logstash** com un servei més dins del nostre fitxer **Docker-compose.yml**.

## EL SERVEI

En el mateix **Docker-compose** on tenim alçats els serveis d'**Elasticsearch**, **Kibana** i **Node-RED** podem afegir el servei:

```yaml
  # Service for deploying logstash
  logstash:
    restart: unless-stopped
    image: docker.elastic.co/logstash/logstash:8.19.11
    container_name: logstash
    links:
      - elasticsearch
    volumes:
      - ./logstash:/config-dir # exemple de input Kafka --> Elastic
      #- ./logstash_file:/config-dir # exemple de input file --> Elastic
      #- ./logstash_submarino:/config-dir
      #- ./logstash_submarino2:/config-dir
    command: logstash -f /config-dir/logstash.conf
    depends_on:
      - elasticsearch
    environment:
      #- xpack.security.enabled=true
      #- xpack.monitoring.enabled=true
      #- ELASTICSEARCH_USERNAME=elastic
      #- ELASTICSEARCH_PASSWORD=tavernes
      - ELASTICSEARCH_URL=http://elasticsearch:9200
      - ELASTICSEARCH_HOST=elasticsearch
      - ELASTICSEARCH_PORT=9200
      - SERVER_HOST=0.0.0.0
    networks:
      - elk
```

Com podem veure, el servei de **Logstash** depèn d'**Elastic**. A més utilitzem unes variables d'entorn per a comunicar amb **Elastic**. Eixes variables s'haurien de modificar en funció del nom i el port que estem utilitzant. 

Fixeu-vos també que necessitem un arxiu **logstash.conf**. Després veurem quin format ha de tindre. Eixe arxiu és el que defineix què se fa en cada fase (ingesta, processament/filtres, emmagatzematge) del procés. Eixe arxiu ha d'estar en el volum que creem (la subcarpeta **logstash** dins de la carpeta on tenim el **Docker-compose**).

Recordeu que en Linux canviarem l'adreça **host.docker.internal** per **localhost** o per la IP 172.17.0.1.

> Vosaltres ja teniu un **docker-compose** preparat en Aules, dins de la carpeta **Material Logstash**. A més també té **Kafka** i **Zookeeper** per a posteriors exemples i pràctiques.

## LA CONFIGURACIÓ

En Aules teniu també un fitxer **logstash_template.txt** on podeu veure l'estructura que hem de seguir en l'arxiu **logstash.conf**. 

- secció **input**: obligatòria, és on definim la font de dades
- secció **filter**: opcional, definim els filtres o processaments que volem realitzar sobre les dades d'entrada
- secció **output**: obligatòria, és on definim on se van a guardar les dades d'eixida

**Estructura arxiu logstash.conf**:

```conf
input { 
    # Required - An input plugin to pass some data to Elasticsearch - file or API 
} 
filter { 
    # Can be empty 
} 
output { 
    # All of the following connection details 
elasticsearch { 
    # Your node IP addresses from the Instaclustr Console 
        hosts => ["https://<NodeIP1>:9200","https://<NodeIP2>:9200","https://<NodeIP3>:9200"] 
        
        # SSL enabled 
        ssl => true 
        ssl_certificate_verification => true 
        
        # Path to your Cluster Certificate .pem downloaded earlier 
        cacert => "<Path to: cluster-ca-certificate.pem>" 
    
        # The Logstash Username and Password created Earlier 
        user => "<LogstashUserName>" 
        password => "<LogstashPassword>" 

        # The name of the Index 
        index => "<Intended Name of Your Index>" 
        ilm_enabled => false 
}
}
```

> **Logstash** intentarà llegir les dades de la secció **input** des del mateix moment que arranquem els contenidors. Si no existeixen les dades, quedarà a l'espera fins que estiguen. Una vegada llegides, si hi ha algun filtre en la secció **filter**, Logstash aplicarà eixes transformacions a les dades abans d'enviar-les a la secció **output**. Mentre estiga en funcionament, **Logstash** estarà a l'espera de noves dades d'entrada per a processar-les i enviar-les a la secció **output**.

### Exemple d'arxiu logstash.conf fent ingesta d'un arxiu log i enviant-lo a Elastic sense aplicar cap filtre

```conf
input {
  file {
    path => "/config-dir/logstash.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
   // el deixariem en blanc si no volem fer cap canvi
}
output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs"
    user => "elastic"
    password => "tavernes"
  }
}
```

### Exemple d'un arxiu logstash.conf amb filtre
  
```conf
# este logstash sirve para exportar una sola vez un archivo .json (logs.json) a elastic
input {
 file {
   codec => "json"
   path => "/config-dir/logs.json"
   start_position => "beginning"
   sincedb_path => "/dev/null"  # Útil para evitar el seguimiento del archivo
 }
}
filter {
 json {
   source => "message"  
 }
}
output {
   #stdout { codec => rubydebug { metadata => true } }
   elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs_json"
    user => "elastic"
    password => "tavernes"
  } 
}
```

> Si existeis l'arxiu json, farà la ingesta de forma automàtica. Si no existeix, quedarà a l'espera fins que aparega. Recordeu que el servei de **Logstash** s'ha d'arrancar després de tenir l'arxiu **logstash.conf** preparat i en la carpeta correcta.

### Exemple d'un arxiu logstash.conf fent ingesta utilitzant Kafka

En este cas anem a fer que Kafka genere una subscripció a uns topics concrets i que **Logstash** ingeste eixes dades i les envie a **Elasticsearch**. L'índex creat en Elastic tindrà el mateix nom que el tòpic.

```conf
input {
  kafka{
    codec => json
    bootstrap_servers => "localhost:29092"
    topics => ["prueba","pruebas"]
    decorate_events => true
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][kafka][topic]}"
    #user => "elastic"
    #password => "tavernes"
  } 
}
```

> Recordeu sempre en Windows, si teniu problemes posant `localhost`, canviar-ho per `host.docker.internal` o per la IP del host.

### Exemple d'un arxiu logstash.conf amb una secció filter complexa

```conf
input {
  file {
    path => "/data/access_logs.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

filter {

  # 1. Parsear el CSV asignando nombre a cada columna
  csv {
    separator => ","
    skip_header => true
    columns => ["timestamp", "ip", "method", "url", "status_code", "bytes", "response_time"]
  }

  # 2. Convertir tipos: sin esto, todos los campos llegan como string
  mutate {
    convert => {
      "status_code"   => "integer"
      "bytes"         => "integer"
      "response_time" => "integer"
    }
  }

  # 3. Parsear el timestamp del CSV como fecha real de Elasticsearch
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
    target => "@timestamp"
    timezone => "Europe/Madrid"
  }

  # 4. Añadir un campo calculado: clasificar la petición según el código HTTP
  if [status_code] >= 500 {
    mutate { add_field => { "nivel_alerta" => "crític" } }
  } else if [status_code] >= 400 {
    mutate { add_field => { "nivel_alerta" => "advertència" } }
  } else {
    mutate { add_field => { "nivel_alerta" => "normal" } }
  }

  # 5. Detectar posible escaneo: URLs sospechosas
  if [url] =~ /wp-admin|\.env|\.git|admin|phpmyadmin/ {
    mutate { add_field => { "sospitós" => true } }
  } else {
    mutate { add_field => { "sospitós" => false } }
  }

  # 6. Eliminar campos que no queremos en Elasticsearch
  mutate {
    remove_field => ["message", "host", "log", "event"]
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-access"
    user => "elastic"
    password => "tavernes"
  }
}
```

## TREBALLAR AMB MÉS D'UN ARXIU LOGSTASH

L'arxiu sempre s'ha de dir **logstash.conf**, i sempre ha d'estar en la carpeta **config-dir** del servei dockeritzat. Per tant, no podem tindre noms diferents.

El que si podem fer és mapejar en el volum diferents carpetes amb diferents arxius **logstash.conf**.

Així, en el nostre cas, si volem utilitzar el mateix servei **Elastic** i **Kibana**, i tindre arxius **logstash** diferents per provar vàries opcions (per exemple, llegir un csv, un arxiu log i un Kafka), podem mantindre el mateix **docker-compose** i comentar/descomentar les línies corresponents del servei **logstash**. Per exemple:

```yaml
volumes:
      #- ./logstash:/config-dir # exemple de input Kafka --> Elastic
      #- ./logstash_file:/config-dir # exemple de input file --> Elastic
      #- ./logstash_filter:/config-dir # exemple d'input file amb filtres --> Elastic
      - ./logstash_submarino:/config-dir # exemple de input Kafka --> Elastic
    command: logstash -f /config-dir/logstash.conf
```

En la meua carpeta on tinc el **docker-compose** tinc una carpeta **logstash**, una carpeta **logstash_file** i una carpeta **logstash_submarino**. En cada carpeta tinc un arxiu **logstash.conf** diferent. En funció de quin vull treballar, comente o descomente la línia corresponent del **docker-compose**.

El servei s'ha de reiniciar cada vegada que canviem d'arxiu **logstash.conf**. Podem intentar baixar i tornar a alçar només **logstash** però si vos dóna problemes el més segur és fer un **docker-compose down** i un **docker-compose up -d**.
