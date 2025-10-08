---
# Informació general del document
title: Repàs Docker Compose
subtitle: 
authors: 
    - Departament d'informàtica
lang: ca
page-background: img/bg.png

# Portada
titlepage: true
titlepage-rule-height: 0
# titlepage-rule-color: AA0000
# titlepage-text-color: AA0000
titlepage-background: img/portada.png
# logo: img/logotext.png

# Taula de continguts
toc: true
toc-own-page: true
toc-title: Continguts

# Capçaleres i peus
header-left: Unitat 03.2 - Repàs Docker Compose
header-right: Curs 2025-2026
footer-left: IES Jaume II El Just
footer-right: \thepage/\pageref{LastPage}

# Imatges
float-placement-figure: H
caption-justification: centering

# Llistats de codi
listings-no-page-break: false
listings-disable-line-numbers: false

header-includes:
     - \usepackage{lastpage}
---

# Docker Compose

**Docker Compose** és una eina que permet definir i executar aplicacions
amb varis contenidors. Docker Compose ens permet definir, crear i alçar 
un conjunt de contenidors a parti d'un únic arxiu de configuració i amb
una sola instrucció.

El primer pas és definir un **arxiu YAML** per configurar els serveis de 
l'aplicació. A continuació, amb una instrucció ***docker compose***, es crea i posa 
en marxa tot l'entorn des de la configuració. Així, podem treballar amb diferents
serveis i volums, amb l'opció de definir una xarxa comuna per a tots ells.

Amb Docker Compose podem:

* Definir, arrancar i parar serveis
* Veure l'estat dels serveis
* Veure els logs dels serveis en marxa
* Executar una instrucció en un serveis

En general Docker Compose és similar a Kubernetes pel que fa a muntar un entorn de treball amb diferents contenidors connectats. Docker Compose és més senzill i està pensat per a entorns de desenvolupament i per a fer proves.

Per a utilitzar Docker Compose cal tenir instal·lat Docker al nostre sistema. Per comprovar que tenim instal·lat Docker Compose podem executar la següent instrucció:

```bash
docker-compose --version
```

Si no tenim instal·lat `docker-compose`, podem fer-ho amb la següent instrucció:

```bash
sudo apt install docker-compose
```

De totes formes, en versions recents de Docker poder fer `docker compose` en lloc de `docker-compose`.

## El fitxer docker-compose.yml 

Com hem comentat, el primer pas per a utilitzar Docker Compose és crear un arxiu de configuració anomenat ***docker-compose.yml***. Aquest arxiu és un arxiu de configuració YAML que conté la definició dels serveis, xarxes i volums que formen l'aplicació que volem alçar.

L'estructura bàsica d'un arxiu ***docker-compose.yml*** és la següent:

```yaml
version: '3.8'
services:
  servei1:
    image: imatge1
    ports:
      - "port_host:port_container"
    expose:
      - "port_container"
    volumes:
      - "volum_host:volum_container"
  servei2:
    image: imatge2
    ports:
      - "port_host:port_container"
    volumes:
      - "volum_host:volum_container"
networks:
  xarxa1:
    driver: bridge
  xarxa2:
    driver: bridge
```

* **version**: Indica la versió de la sintaxi de Docker Compose que estem utilitzant.
* **services**: Conté la definició dels serveis que formen l'aplicació.
* **volumes**: Volums que formen l'aplicació i mapeig entre el host i el contenidor.
* **image**: Les imatges que s'utilitzaran per a crear els contenidors.
* **ports**: Mapeig de ports entre el host i el contenidor.
* **expose**: Exposa un port del contenidor sense fer-lo accessible des de l'exterior.
* **networks**: Definició i configuració de les xarxes.
* **driver**: Driver de la xarxa. La xarxa per defecte de Docker és ***bridge***.

Podem utilitzar **depends_on** per a indicar que un servei depèn d'un altre, de forma que el servei depenent no arrancarà fins que arranque el servei del qual depèn.

```yaml
version: '3.8'
services:
  servei1:
    image: imatge1
    ports:
      - "port_host:port_container"
    volumes:
      - "volum_host:volum_container"
  servei2:
    image: imatge2
    ports:
      - "port_host:port_container"
    volumes:
      - "volum_host:volum_container"
    depends_on:
      - servei1
```

Com a alternativa a **image**, amb l'etiqueta **build** podem indicar la ruta on es troba el Dockerfile per a construir la imatge.

Per exemple, si en la mateixa carpeta on tenim l'arxiu ***docker-compose.yml*** tenim un Dockerfile, podem fer:

```yaml
version: '3.8'
services:
  servei1:
    build: .
    ports:
      - "port_host:port_container"
    volumes:
      - "volum_host:volum_container"
```

També podem crear variables d'entorn en els contenidors amb l'etiqueta **environment**.

```yaml
version: '3.8'
services:
  servei1:
    image: imatge1
    ports:
      - "port_host:port_container"
    volumes:
      - "volum_host:volum_container"
    environment:
      - VAR1=valor1
      - VAR2=valor2
```

Amb l'etiqueta **restart** podem indicar com volem que es comporte el contenidor en cas de reinici. 

* ***restart: no***: No es reiniciarà mai. Això implica que si el contenidor es para, no es tornarà a alçar a no ser que ho fem manualment.
* ***restart: always***: Es reiniciarà sempre que es pare.
* ***restart: on-failure***: Es reiniciarà només si el contenidor falla. 
* ***restart: unless-stopped***: Es reiniciarà sempre que es pare, a no ser que el parem manualment.

## Comandes bàsiques de Docker Compose

Un cop tenim l'arxiu de configuració ***docker-compose.yml***, podem utilitzar

* **docker-compose up**: Crea i alça els contenidors especificats a l'arxiu de configuració.
* **docker-compose down**: Para i elimina els contenidors especificats a l'arxiu de configuració.
* **docker-compose ps**: Mostra l'estat dels contenidors especificats a l'arxiu de configuració.
* **docker-compose logs**: Mostra els logs dels contenidors especificats a l'arxiu de configuració.
* **docker-compose exec**: Executa una instrucció en un contenidor.
* **docker-compose start**: Arranca els contenidors (o el contenidor especificat)
* **docker-compose stop**: Para els contenidors (o el contenidor especificat) 
* **docker-compose restart**: Reinicia els contenidors (o el contenidor especificat)   
* **docker-compose build**: Construeix les imatges especificades a l'arxiu de configuració.  

## Exemple d'ús de Docker Compose

Suposem que volem alçar un entorn amb un servidor web i una base de dades MySQL/MariaDB.

Creem un arxiu ***docker-compose.yml*** amb el següent contingut:

```yaml
version: '3.8'
services:
  webserver:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./web:/usr/share/nginx/html
  db:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: test
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - ./db:/var/lib/mysql
```

Si ara, des de la mateixa carpeta on tenim l'arxiu, fem:

```bash
docker-compose up
```

Podrem veure com se creen i s'alcen els contenidors. El servidor web estarà disponible a l'adreça ***http://localhost:8080***.

## Escalar serveis amb Docker Compose

Amb Docker Compose podem escalar serveis de forma senzilla. Per exemple, si volem tenir 3 instàncies del servei webserver, podem fer:

```bash
docker-compose up --scale webserver=3
```

Amb aquesta instrucció, es crearan 3 instàncies del servei webserver.

## Exemple: dockeritzar serveis amb Python

En **Aules** teniu un arxiu anomenat **api_con_dockerfile.zip** que conté un exemple senzill de com dockeritzar un servei amb Python. Baixeu l'arxiu i descomprimiu-lo. Tindreu un fitxer ***docker.compose.yml*** amb el següent contingut:

```yaml
version: "3.7"
services:

  # API 
  api:
    restart: always
    container_name: api
    build: ./folder/
    ports:
      - "8010:8010"
    expose:
      - "8010"
    volumes:
      - ./folder/data:/folder/data
```

El que fa el fitxer és alçar un contenidor amb un servei API que escolta pel port 8010. El servei està definit en un Dockerfile que es troba a la carpeta ***folder***. El contingut del fitxer eś el següent:

```Dockerfile
FROM python:3.9

WORKDIR /folder

ADD . /folder

RUN pip install -r requirements.txt

EXPOSE 8010

CMD ["python", "-u", "/folder/script.py"]

LABEL MAINTAINER jclemente@prodevelop.es
```

El Dockerfile crea un contenidor amb Python 3.9, copia el contingut de la carpeta ***folder*** al contenidor, instal·la les dependències del projecte, exposa el port 8010 i executa el script ***script.py*** que es troba a la carpeta ***folder***. L'arxiu ***requirements.txt*** conté les dependències del projecte. L'script ***script.py*** crea una API simple que escolta el port 8010 i que ofereix dos serveis: un GET en /getList i un POST en /postExample que, si li passem un paràmetre `name`, el mostrarà per pantalla. 

Alceu el contenidor i comproveu que els serveis funcionen correctament. El servei GET el podeu provar directament des del navegador. Per a provar el servei POST, podeu fer servir l'eina ***curl***. Per exemple:

```bash
curl -X POST --header 'Content-Type:application/json' -d '{"name":"Fidel"}' http://localhost:8010/postExample 
```

o bé utilitzar una eina com ***Postman*** o ***Hoppscotch***.

> En Windows és possible que no funcione amb ***localhost***. En aquest cas, podeu provar amb la IP de la màquina o bé amb
> host.docker.internal en lloc de localhost.

## Exemple: dockeritzar Node-Red

Un altre exemple senzill és el de dockeritzar **Node-Red**. Node-Red és una eina de programació visual per a IoT. Per a dockeritzar Node-Red, podem fer servir la imatge oficial de Node-Red.

Creeu un arxiu ***docker-compose.yml*** amb el següent contingut:

```yaml
services:
  node-red:
    image: nodered/node-red:latest
    container_name: nodeRED
    restart: unless-stopped
    networks:
      - net
    ports:
      - "1881:1880"
    volumes:
      - "./node-red:/data"
  
networks:
  net:
    driver: bridge
```

Amb aquest arxiu, alçarem un contenidor amb Node-Red que escoltarà pel port 1880. La carpeta ***data*** es mapejarà amb la carpeta ***data*** del contenidor, de forma que les dades de Node-Red es guardaran en la carpeta ***data*** de la màquina host. El port 1880 del contenidor se mapeja amb el port 1881 del sistema amfitrió. 

Podeu provar a accedir a Node-Red a través del navegador a l'adreça ***http://localhost:1881***.

En la següent unitat veurem a fons com utilitzar Node-Red.
