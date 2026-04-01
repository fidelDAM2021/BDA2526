---
# Informació general del document
title: Big Data Aplicat 
subtitle: Annex - Grafana
authors: 
    - Departament d'informàtica
lang: ca
page-background: img/bg.png

# Portada
titlepage: true
titlepage-rule-height: 0
titlepage-background: img/portada.png

# Taula de continguts
toc: true
toc-own-page: true
toc-title: Continguts

# Capçaleres i peus
header-left: Big Data Aplicat. Annex - Grafana
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

# Grafana

**Grafana** és una plataforma de visualització de dades i monitorització de codi obert que permet crear dashboards interactius a partir de múltiples fonts de dades. És àmpliament utilitzada en entorns empresarials per monitoritzar sistemes, aplicacions i infraestructures, així com per analitzar dades en temps real.

## Característiques principals

- **Multi-font de dades**: Connecta amb Prometheus, Elasticsearch, PostgreSQL, MySQL, InfluxDB, MongoDB, i moltes altres
- **Visualitzacions interactives**: Gràfiques de línies, barres, gauges, mapes de calor, taules, etc.
- **Alertes intel·ligents**: Configuració d'alertes amb múltiples canals (email, Slack, webhook, PagerDuty)
- **Plantilles i variables**: Dashboards dinàmics amb variables reutilitzables
- **Plugins**: Sistema de plugins per estendre funcionalitats
- **Col·laboració**: Compartició de dashboards, anotacions i comentaris

## Grafana OSS vs Enterprise

**Grafana OSS (Open Source Software)** és completament gratuïta i de codi obert. La versió **Enterprise** ofereix característiques avançades com:

- Control d'accessos basat en rols (RBAC)
- Suport prioritari
- Capacitats d'aprovisionament avançades
- Integració amb SAML i LDAP

Per a entorns educatius i la majoria d'usos, la versió OSS és més que suficient.

## Comparativa: Grafana vs Kibana

Com que heu vist un poc de dashboards amb Kibana, ací teniu una breu comparativa de les dues eines.

| Característica | Grafana | Kibana |
|----------------|---------|--------|
| **Enfocament principal** | Mètriques, sèries temporals | Logs, cerca, anàlisi |
| **Fonts de dades** | Moltes (Prometheus, SQL, NoSQL, Cloud...) | Principalment Elasticsearch |
| **Alertes** | Integrades, multi-canal | Via Watcher o alerting |
| **Learning curve** | Moderada | Moderada |
| **Dashboards** | Molt flexible | Flexible |

Grafana és ideal quan necessites visualitzar dades de múltiples fonts en un mateix lloc. Kibana és millor quan treballes exclusivament amb Elasticsearch i logs.

---

## Instal·lació amb Docker

### Stack bàsic: Grafana + Prometheus + node-exporter

El següent `docker-compose.yml` arranca Grafana juntament amb Prometheus (per a mètriques) i node-exporter (per a obtindre dades del sistema).

```yaml
# docker-compose.yml
services:

  # ── 1. PROMETHEUS ──────────────────────────────────────────────
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    networks:
      - monitoring

  # ── 2. NODE EXPORTER ───────────────────────────────────────────
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    ports:
      - "9100:9100"
    networks:
      - monitoring

  # ── 3. GRAFANA ─────────────────────────────────────────────────
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana_data:/var/lib/grafana
    depends_on:
      - prometheus
    networks:
      - monitoring

# ── VOLÚMENES I XARXA ────────────────────────────────────────────────
volumes:
  prometheus_data: {}
  grafana_data: {}

networks:
  monitoring:
    driver: bridge
```

Necessitem el fitxer `prometheus.yml`:

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['prometheus:9090']

  - job_name: node-exporter
    static_configs:
      - targets: ['node-exporter:9100']
```

Per arrancar:

```bash
docker compose up -d
```

### Credencials i URLs d'accés

| Servei | URL | Credencials |
|--------|-----|-------------|
| Grafana | http://localhost:3000 | admin / admin123 |
| Prometheus | http://localhost:9090 | - |
| node-exporter | http://localhost:9100 | - |

### Connexió amb ELK (xarxa externa)

Per connectar Grafana amb un stack ELK ja existent, afegim la xarxa `elk_elk`:

```yaml
# Afegir a networks:
networks:
  monitoring:
    driver: bridge
  elk_elk:
    external: true

# Afegir al servei grafana:
  grafana:
    ...
    networks:
      - monitoring
      - elk_elk
```

> **Nota**: El nom `elk_elk` pot variar. Comprova amb `docker network ls` quin és el nom exacte de la xarxa del teu stack ELK.

Haurieu d'arrancar primer el contenidor amb Elastic, i després el de Grafana.

---

## Prometheus: Conceptes Bàsics

### Què és Prometheus?

**Prometheus** és un sistema de monitorització i alertes de codi obert que:

- Recopila mètriques com a sèries temporals
- Utilitza un model de dades basat en etiquetes (labels)
- Permet consultes potents amb PromQL
- S'integra perfectament amb Grafana

### Explorant Prometheus

Accedeix a http://localhost:9090 per veure la interfície de Prometheus.

![Prometheus UI](images/prometheus01.jpg)

### Mètriques útils de node-exporter

| Mètrica | Descripció |
|---------|------------|
| `node_cpu_seconds_total` | Segons de CPU consumits (per mode, cpu) |
| `node_memory_MemAvailable_bytes` | Memòria disponible en bytes |
| `node_memory_MemTotal_bytes` | Memòria total |
| `node_filesystem_avail_bytes` | Espai disponible en sistemes de fitxers |
| `node_network_receive_bytes_total` | Bytes rebuts per interfície |
| `node_network_transmit_bytes_total` | Bytes transmesos |
| `node_disk_read_bytes_total` | Bytes llegits del disc |
| `node_disk_written_bytes_total` | Bytes escrits al disc |
| `up` | Estat del target (1=up, 0=down) |

### Explorar totes les mètriques

Fes clic en **"Explore metrics"** (icona de brúixola) per veure totes les mètriques disponibles.

![Prometheus Explore](images/prometheus02.jpg)

---

### PromQL: Llenguatge de Consultes

**Estructura d'una mètrica**

```
nom_mètrica{etiqueta1="valor1", etiqueta2="valor2"}
```

Exemple:
```
node_cpu_seconds_total{mode="idle", cpu="0", instance="localhost:9100"}
```

**Tipus de dades**

| Tipus | Descripció | Exemple |
|-------|------------|---------|
| **Gauge** | Valor que pot pujar i baixar | `node_memory_MemAvailable_bytes` |
| **Counter** | Valor que només puja | `node_cpu_seconds_total` |
| **Histogram** | Distribució de valors | `http_request_duration_seconds` |
| **Summary** | Percentils calculats | `http_request_duration_seconds` |

**Funcions de temps**

```promql
# Valor actual d'una mètrica
node_memory_MemAvailable_bytes

# Valor fa 5 minuts
node_memory_MemAvailable_bytes offset 5m

# Taxa de canvi (per segon)
rate(node_cpu_seconds_total[5m])

# Increment en un interval
increase(node_cpu_seconds_total[1h])
```

**Funcions d'agregació**

```promql
# Suma de tots els valors
sum(node_cpu_seconds_total)

# Mitjana
avg(node_cpu_seconds_total) by (mode)

# Màxim
max(node_memory_MemAvailable_bytes)

# Mínim
min(node_disk_read_bytes_total)

# Comptador
count(node_cpu_seconds_total)
```

**Funcions útils**

```promql
# Predicció lineal (serà 2h en el futur)
predict_linear(node_cpu_seconds_total[5m], 2*60*60)

# Absent (detecta quan una mètrica desapareix)
absent(up{job="prometheus"})

# Canvi de taxa (útil per a counters)
rate(node_network_receive_bytes_total[5m])

# Percentil
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

**Operadors**

```promql
# Suma
node_memory_MemAvailable_bytes / 1024 / 1024 / 1024

# Multiplicació
rate(node_cpu_seconds_total[5m]) * 100

# Comparació
node_memory_MemAvailable_bytes > 1024 * 1024 * 1024
```

**Operadors lògics**

```promql
# AND (interseció)
up{job="prometheus"} and up{instance="localhost:9100"}

# OR (unió)
up{job="prometheus"} or up{job="node-exporter"}

# Unless (exclusió)
up and absent(up{job="prometheus"})
```

### Exemples pràctics

**Utilització de CPU**

```promql
# CPU utilitzada (no idle)
100 - (rate(node_cpu_seconds_total{mode="idle"}[5m]) * 100)

# Per cpu individual
100 - (rate(node_cpu_seconds_total{mode="idle",cpu="0"}[5m]) * 100)
```

**Memòria en GB**

```promql
node_memory_MemAvailable_bytes / 1073741824
```

**Espai en disc lliure (percentatge)**

```promql
(node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100
```

**Rendiment de la xarxa**

```promql
# MB/s rebuts
rate(node_network_receive_bytes_total{device="eth0"}[1m]) / 1024 / 1024

# MB/s transmesos
rate(node_network_transmit_bytes_total{device="eth0"}[1m]) / 1024 / 1024
```

**Requests per segon (servidor web)**

```promql
rate(http_requests_total[5m])
```

### Regles de gravació (Recording Rules)

Les regles de gravació precalculen consultes freqüents per millorar el rendiment.

```yaml
# prometheus.yml
rule_files:
  - "recording_rules.yml"
  - "alert_rules.yml"
```

```yaml
# recording_rules.yml
groups:
  - name: cpu_rules
    rules:
      - record: job:node_cpu_utilisation:rate5m
        expr: 100 - (rate(node_cpu_seconds_total{mode="idle"}[5m]) * 100)

  - name: memory_rules
    rules:
      - record: job:node_memory_free_percent
        expr: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```

Ara haurieu de crear volums en el docker-compose per mapejar les regles. Per exemple, en el nostre cas que l'arxiu se diu `recording_rules.yml`:

```yaml
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml
    - ./recording_rules.yml:/etc/prometheus/recording_rules.yml        # afegiriem esta línia
```

En l'arxiu `prometheus.yml` també hem d'afegir l'arxiu de regles:

```yml
rule_files:
  - "recording_rules.yml"    
```

Si ho tornem a reiniciar tot, ara des de Prometheus podem demanar eixes mètriques cridant a la regla corresponent.

```
job:node_memory_free_percent
```

Igualment podem cridar-les des de Grafana quan ens demana quina mètrica volem.

---

## Connexió de Grafana amb Prometheus

### Configurar Prometheus com a Datasource

1. Accedeix a Grafana: http://localhost:3000
2. Ves a **Connections → Data Sources**
3. Clica **Add data source**
4. Busca **Prometheus** (Time series databases)
5. En **Connection**, posa: `http://prometheus:9090`
6. Clica **Save & Test**

![Data Source Prometheus](images/grafana03.jpg)

### Crear un Dashboard bàsic

1. Ves a **Dashboards → New → New Dashboard**
2. Clica **Add visualization**
3. Selecciona **Prometheus** com a font de dades
4. Escriu una consulta PromQL:

```promql
100 - (rate(node_cpu_seconds_total{mode="idle"}[5m]) * 100)
```

5. Clica **Run queries** per veure el resultat

![Query Prometheus](images/grafana04.jpg)

## Tipus de visualitzacions

Grafana ofereix molts tipus de panells:

| Tipus | Ús |
|-------|-----|
| **Time series** | Sèries temporals (gràfica de línies) |
| **Stat** | Valor individual amb tendència |
| **Gauge** | Valor amb rang visual |
| **Bar chart** | Comparacions entre categories |
| **Table** | Dades tabulars |
| **Pie chart** | Proporcions |
| **Heatmap** | Densitat de dades |
| **Logs** | Visualització de logs |
| **Text** | Text estàtic o Markdown |

---

## Elasticsearch com a Datasource

### Configuració de la xarxa

Assegura't que Grafana i Elasticsearch estan en la mateixa xarxa Docker si no estan en el mateix arxiu docker-compose.yml:

```yaml
# Afegir a docker-compose.yml on tenim Grafana
networks:
  elk_elk:
    external: true

# En el servei grafana del mateix docker-compose.yml:
networks:
  - monitoring
  - elk_elk
```

>Teniu en compte que cada fitxer docker-compose crea les seues xarxes, afegint al nom de la xarxa el nom del servei que la crea. Per això la xarxa elk del docker-compose.yml d'Elastic ací s'anomena elk_elk. A més hem de posar `external:true` en lloc de `driver:bridge`.

### Crear el Datasource

1. Ves a **Connections → Data Sources → Add data source**
2. Busca **Elasticsearch**
3. Configura:

| Camp | Valor |
|------|-------|
| URL | `http://elasticsearch:9200` |
| Auth | **Basic auth** |
| User | `elastic` |
| Password | `la_teua_contrasenya` |

4. En **Elasticsearch Details**:
   - **Index name**: `kibana_sample_data_ecommerce` (o el teu índex)
   - **Time field**: `@timestamp`

5. Clica **Save & Test**

![Elasticsearch Config](images/grafana10.jpg)

### Consultes en Elasticsearch

Les consultes al datasource se poden fer amb el llenguatge `Lucene`, o des del formulari que ens ofereix el datasource.

**Utilitzant Lucene**

Si aneu al datasource, feu clic en `Explore` i seleccioneu l'opció `Logs`, podeu fer consultes directament des del quadre de text `Lucene Query`. Per exemple, podeu provar:

```lucene
# Tots els documents
*

# Amb filtre
category:"Shoes"

geoip.continent_name:"Africa"

# Rang de dates
order_date: [2026-03-20 TO 2026-03-30]

# Múltiples condicions
category:"Electronics" AND manufacturer:"Apple"
```

**Metrics i Group By**

Grafana permet agregar dades directament:

| Setting | Descripció |
|---------|------------|
| **Count** | Nombre de documents |
| **Average** | Mitjana del camp |
| **Sum** | Suma del camp |
| **Min/Max** | Valors extrems |
| **Unique count** | Valors únics |

**Group By**

| Tipus | Descripció |
|-------|------------|
| **Terms** | Agrupar per valors d'un camp |
| **Date Histogram** | Agrupar per intervals de temps |
| **Histogram** | Rang de valors numèrics |

Per exemple, des de el datasource podem fer clic en `Explore` i crear una consulta tal com se veu a la figura:

![Consulta de prova des de Explore](images/grafana13.jpg)

Si feu scroll cap a baix, en la part inferior podeu veure el gràfic en la secció `Graph`, i les dades en la secció `Table`. Podeu accedir directament des del menú esquerre canviant de `Queries` a `Graph` o `Table`.

![Resultat de la consulta](images/grafana14.jpg)

Podeu provar diferents consultes sobre el datasource. Quan ja vos funciona i obteniu el resultat que voleu, siga una consulta en Lucene o des del formulari, podeu afegir-lo directament a un Dashboard existent, o crear-ne un de nou. 

![Afegir la consulta a un Dashboard](images/grafana15.jpg)

Si tenim les dades en Elastic és més natural fer els dashboards en Kibana, però ara ja sabeu fer-los també utilitzant Grafana. 

Anem a veure altres tipus de fonts de dades.

---

## Bases de dades SQL

Com ja sabeu, les bases de dades SQL (Structured Query Language) són bases de dades **relacionals** que emmagatzemen les dades en taules amb files i columnes. Són ideals quan les dades tenen una estructura clarament definida i estable, o si necessitem establir relaciones complexes entre les dades. Hem de tindre en compte que si la quantitat d'informació és molt elevada, segurament funcionaran més lentes que una no SQL.

### Quan utilitzar SQL vs NoSQL amb Grafana

| Situació | Recomanació |
|----------|------------|
| Dades transaccionals (comandes, inventari) | **SQL** (PostgreSQL/MySQL) |
| Mètriques amb alta granularitat (IoT) | **InfluxDB** |
| Dades de monitorització clàssica | **Prometheus** |
| Logs i cerca de text | **Elasticsearch** o **Loki** |
| Documents amb estructura variable | **MongoDB** |

---

### Exemple amb PostgreSQL

**PostgreSQL** és un sistema de gestió de bases de dades objecte-relacional (ORDBMS) de codi obert, àmpliament considerat com una de les bases de dades més avançades i potents disponibles. Suporta l'estàndard SQL, tipus de dades avançats, programació, etc.

Anem a fer un exemple amb PostgresSQL. Primer alcem el servei. Ho haurem de fer en la mateixa xarxa de Grafana.

**docker-compose per a PostgreSQL**

```yaml
# docker-compose.postgresql.yml
services:
  postgres:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: monitoring
      POSTGRES_USER: grafana
      POSTGRES_PASSWORD: grafana123
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - monitoring

volumes:
  postgres_data: {}

networks:
  monitoring:
    driver: bridge
```

Si copieu l'arxiu `init.sql` en la mateixa carpeta on teniu el yaml per alçar el contenidor de PostgreSQL, se suposa que s'executarà i ja tindreu dades per fer proves.

**Script d'inicialització**

```sql
-- Init SQL per a PostgreSQL
-- Executat automàticament quan es crea el contenidor

SET NAMES utf8mb4;
SET CHARACTER SET utf8mb4;

-- Taula de sensors IoT
CREATE TABLE IF NOT EXISTS sensors (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    location VARCHAR(100),
    temperature DECIMAL(5,2),
    humidity DECIMAL(5,2),
    pressure DECIMAL(7,2),
    reading_time TIMESTAMP DEFAULT NOW()
);

-- Taula de mètriques de servidor
CREATE TABLE IF NOT EXISTS server_metrics (
    id SERIAL PRIMARY KEY,
    server VARCHAR(50) NOT NULL,
    cpu_usage DECIMAL(5,2),
    memory_usage DECIMAL(5,2),
    disk_usage DECIMAL(5,2),
    timestamp TIMESTAMP DEFAULT NOW()
);

-- Taula de comandes
CREATE TABLE IF NOT EXISTS orders (
    id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    product VARCHAR(100),
    quantity INT,
    price DECIMAL(10,2),
    order_date TIMESTAMP DEFAULT NOW()
);

-- Inserir dades de'exemple
INSERT INTO sensors (name, location, temperature, humidity, pressure) VALUES
('Sensor-001', 'Warehouse-A', 22.5, 45.2, 1013.25),
('Sensor-002', 'Warehouse-B', 23.1, 46.8, 1013.10),
('Sensor-003', 'Office', 21.8, 42.5, 1013.30),
('Sensor-004', 'Server-Room', 18.5, 35.0, 1013.50);

INSERT INTO server_metrics (server, cpu_usage, memory_usage, disk_usage) VALUES
('server-01', 45.5, 72.3, 65.0),
('server-02', 32.1, 58.9, 45.2),
('server-03', 78.3, 85.1, 82.5),
('server-01', 48.2, 73.1, 65.1),
('server-02', 35.8, 59.2, 45.3);

INSERT INTO orders (customer_name, product, quantity, price) VALUES
('Anna García', 'Portàtil HP ProBook', 1, 799.99),
('Joan Martí', 'Ratolí Wireless', 2, 29.99),
('Maria López', 'Teclat Mecànic', 1, 149.99),
('Pere Sánchez', 'Monitor 27"', 2, 249.00),
('Laura Torres', 'Auriculars USB', 3, 49.99);
```

#### Configurar PostgreSQL a Grafana

1. **Connections → Data Sources → Add data source**
2. Busca **PostgreSQL**
3. Configura:

| Camp | Valor |
|------|-------|
| Host | `postgres:5432` |
| Database | `monitoring` |
| User | `grafana` |
| Password | `grafana123` |
| SSL mode | `disable` |

4. Clica **Save & Test**

>Si teniu problemes de connexió entre Grafana i PostgreSQL, comproveu si els dos serveis estan en la mateixa xarxa (feu `docker network ls` per comprovar els noms de les xarxes), i també si la versió de Grafana i la de PostgreSQL són compatibles. 

#### Consultes PostgreSQL

Les consultes se poden fer des del mateix datasource amb l'opció `explore`. Tenim l'opció d'utilitzar el formulari que ens apareix, o canviar a l'opció `Code` en el selector de l'esquerra. Amb `Builder` tornem al formulari.

![Consultes des del formulari](images/grafana16.jpg)

![Consultes des de Code](images/grafana17.jpg)

Consultes en SQL per provar-les des de l'opció `Code`:

**Utilització de CPU per servidor**

>Comproveu els valors de temps en les taules per si heu d'ajustar el seu valor en la consulta. Per exemple, si les dades són de fa 1 hora si poseu `5m` no vos eixirà res.

```sql
SELECT
    $__timeGroup(timestamp, '5m'),
    server,
    avg(cpu_usage) as "CPU %"
FROM server_metrics
WHERE $__timeFilter(timestamp)
GROUP BY 1, server
ORDER BY 1
```

**Temperatura mitjana per sensor**

```sql
SELECT
    $__timeGroup(reading_time, '1h'),
    name as "Sensor",
    avg(temperature) as "Temp (°C)"
FROM sensors
WHERE $__timeFilter(reading_time)
GROUP BY 1, name
ORDER BY 1
```

**Últim valor de cada sensor**

```sql
SELECT
    name,
    location,
    temperature,
    humidity,
    reading_time
FROM sensors s
WHERE reading_time = (
    SELECT MAX(reading_time) 
    FROM sensors 
    WHERE name = s.name
)
```

**Tots els servidors i tots els sensors**

```sql
-- Tots els servidors
SELECT DISTINCT server FROM server_metrics ORDER BY server

-- Tots els sensors
SELECT DISTINCT name FROM sensors ORDER BY name
```

---

### MySQL

Per a treballar amb MySQL només heu de tindre un servei dockeritzat com hem vist abans, i connectar-lo amb Grafana.

Copieu també l'arxiu `init_mysql.sql` a la carpeta on estiga el `docker-compose.yml` i així quan alceu el contenidor ja tindreu dades en la base de dades.

**docker-compose per a MySQL**

```yaml
# docker-compose.mysql.yml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: monitoring
      MYSQL_USER: grafana
      MYSQL_PASSWORD: grafana123
    ports:
      - "3307:3306"  # pose 3307 perquè el 3306 el tinc ocupat en el meu ordinador
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init_mysql.sql:/docker-entrypoint-initdb.d/init_mysql.sql
    networks:
      - grafana_monitoring
    command: --default-authentication-plugin=mysql_native_password

volumes:
  mysql_data: {}

networks:
  grafana_monitoring:
    external: true
```

**Script d'inicialització**

```sql
-- init_mysql.sql
-- Init SQL per a MySQL
-- Executat automàticament quan es crea el contenidor

SET NAMES utf8mb4;
SET CHARACTER SET utf8mb4;

-- Taula d'estadístiques de lloc web
CREATE TABLE IF NOT EXISTS website_stats (
    id INT AUTO_INCREMENT PRIMARY KEY,
    url VARCHAR(255),
    status_code INT,
    response_time_ms INT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Taula de comandes
CREATE TABLE IF NOT EXISTS orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100),
    product VARCHAR(100),
    category VARCHAR(50),
    quantity INT,
    price DECIMAL(10,2),
    order_date DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Taula d'usuaris actius
CREATE TABLE IF NOT EXISTS user_activity (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100),
    action VARCHAR(100),
    page VARCHAR(255),
    duration_seconds INT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Inserir dades de exemple
INSERT INTO website_stats (url, status_code, response_time_ms) VALUES
('https://api.example.com/users', 200, 45),
('https://api.example.com/products', 200, 32),
('https://api.example.com/orders', 500, 0),
('https://api.example.com/users', 200, 52),
('https://api.example.com/products', 200, 28),
('https://api.example.com/cart', 200, 78);

INSERT INTO orders (customer_name, product, category, quantity, price) VALUES
('Anna García', 'Portàtil HP', 'Electrònica', 1, 799.99),
('Joan Martí', 'Ratolí Wireless', 'Perifèrics', 2, 29.99),
('Maria López', 'Teclat Mecànic', 'Perifèrics', 1, 149.99),
('Pere Sánchez', 'Monitor 27"', 'Electrònica', 2, 249.00),
('Laura Torres', 'Auriculars USB', 'Perifèrics', 3, 49.99),
('David Ruiz', 'Webcam HD', 'Perifèrics', 1, 79.99);

INSERT INTO user_activity (username, action, page, duration_seconds) VALUES
('user1', 'view', '/products', 120),
('user1', 'click', '/cart', 30),
('user2', 'view', '/home', 45),
('user2', 'view', '/products', 180),
('user3', 'purchase', '/checkout', 60);

```

#### Configurar MySQL a Grafana

1. **Connections → Data Sources → Add data source**
2. Busca **MySQL**
3. Configura:

| Camp | Valor |
|------|-------|
| Host | `mysql:3306` |
| Database | `monitoring` |
| User | `grafana` |
| Password | `grafana123` |

>Teniu en compte que el port que poseu és el intern del servei en Docker, que és 3306 encara que cap a fora l'exposem com a 3307

4. Clica **Save & Test**

#### Consultes MySQL

Estes consultes en SQL són per posar-les en la pestanya `Code` de `Explore`, però també podeu intentar fer consultes amb el formulari igual que en PostgreSQL.



**Temps de resposta mitjà per URL**

```sql
SELECT
    $__timeGroup(timestamp, '5m'),
    url,
    AVG(response_time_ms) as "Response Time (ms)"
FROM website_stats
WHERE $__timeFilter(timestamp)
  AND status_code = 200
GROUP BY 1, url
ORDER BY 1
```

**Comandes per client**

```sql
SELECT
    customer_name,
    COUNT(*) as "Total Orders",
    SUM(quantity * price) as "Total Spent"
FROM orders
WHERE $__timeFilter(order_date)
GROUP BY customer_name
ORDER BY "Total Spent" DESC
```

**Distribució de codis de resposta**

```sql
SELECT
    status_code,
    COUNT(*) as count
FROM website_stats
WHERE $__timeFilter(timestamp)
GROUP BY status_code
```

En tots els exemples que hem vist, sempre teniu l'opció de, quan ja teniu la consulta que necessiteu, afegir-la a un Dahsboard (existent o nou) amb el botó `Add`-> `Add to dashboard`.

Per exemple, afegim a un dashboard nou la consulta de comandes totals per client.

```sql
SELECT
    customer_name,
    COUNT(*) as "Total Orders",
    SUM(quantity * price) as "Total Spent"
FROM orders
WHERE $__timeFilter(order_date)
GROUP BY customer_name
ORDER BY "Total Spent" DESC
```

En el dashboard veurem el resultat de la consulta en format taula.

![Resultat de la consulta en format taula](images/grafana18.jpg)

Però si aneu a la part dreta i canvieu el tipus de visualització a `Bar chart`, per exemple, veureu com també canvia la visualització:

![Resultat de la consulta en format bar chart](images/grafana19.jpg)

---

## InfluxDB

**InfluxDB** és una base de dades de codi obert especialment dissenyada per emmagatzemar i consultar **sèries temporals** (time series). Una sèrie temporal és una seqüència de dades mesurades al llarg del temps, com ara temperatures d'un sensor cada segon, el preu d'una acció cada minut, o l'ús de CPU cada 5 segons.

### Conceptes clau d'InfluxDB

InfluxDB utilitza un model de dades específic que cal entendre:

| Concepte | Descripció | Equivalent SQL |
|---------|------------|----------------|
| **Measurement** | Categoria de dades (com una taula) | Table |
| **Tag** | Metadades indexades (per a cerques ràpides) | Indexada column |
| **Field** | Les dades reals (no indexades) | Non-indexed column |
| **Timestamp** | Moment de la mesura | Primary key (timestamp) |
| **Point** | Una única mesura amb tags i fields | Row |

**Exemple pràctic:** Imaginem un sensor de temperatura:

```
Measurement: temperature
Tags:        location=warehouse-A, sensor_id=TEMP-001
Fields:      value=22.5
Timestamp:   2024-01-15 10:30:00 UTC
```

Això es traduiria en InfluxDB com:
```
temperature,location=warehouse-A,sensor_id=TEMP-001 value=22.5 1705315800000000000
```

### Diferències entre InfluxDB i Prometheus

| Característica | InfluxDB | Prometheus |
|----------------|----------|------------|
| **Model de dades** | Pull/Push (flexible) | Pull only |
| **Escriptura** | Alta velocitat d'ingesta | Limitada |
| **Consultes** | Flux o InfluxQL | PromQL |
| **Retenció** | Configurable (dies, mesos) | Limitada |
| **Escalabilitat** | Molt alta | Alta |
| **Ús recomanat** | IoT, finances, DevOps | Monitorització app |

### Quan utilitzar InfluxDB?

- **IoT i sensors**: Milers de sensors enviant dades cada segon
- **Aplicacions financeres**: Tick data, preus, volatilitat
- **Monitorització industrial**: Temperatura, humitat, pressió
- **Anàlisi de rendiment**: Mètriques d'aplicacions amb alta granularitat
- **Dades científiques**: Experiments amb moltes lectures temporals

### Avantatges sobre Prometheus

- **Millor per a grans volums**: Pot ingestar milions de punts per segon
- **Consultes més potents**: Flux és molt expressiu
- **Continuous queries**: Precalcula dades automàticament
- **Downsampling**: Redueix resolució de dades antigues automàticament
- **Multiple buckets**: Diferents polítiques de retenció per a diferents dades

**docker-compose per a InfluxDB**

El següent `docker-compose.yml` arranca InfluxDB amb Telegraf (per a recopilar mètriques del sistema) i Grafana.

### Configuració amb variables d'entorn

```yaml
# docker-compose.influxdb.yml
services:
  influxdb:
    image: influxdb:2.7
    container_name: influxdb
    restart: unless-stopped
    ports:
      - "8086:8086"
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      - DOCKER_INFLUXDB_INIT_PASSWORD=admin123
      - DOCKER_INFLUXDB_INIT_ORG=myorg
      - DOCKER_INFLUXDB_INIT_BUCKET=sensors
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=my-super-secret-token
    volumes:
      - influxdb_data:/var/lib/influxdb2
    networks:
      - monitoring

  telegraf:
    image: telegraf:1.29
    container_name: telegraf
    restart: unless-stopped
    volumes:
      - ./telegraf.conf:/etc/telegraf/telegraf.conf:ro
    depends_on:
      - influxdb
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - monitoring
    depends_on:
      - influxdb

volumes:
  influxdb_data: {}
  grafana_data: {}

networks:
  monitoring:
    driver: bridge
```

### Explicació de les variables d'entorn

Quan InfluxDB s'inicialitza per primera vegada amb `DOCKER_INFLUXDB_INIT_MODE=setup`, es creen automàticament els recursos inicials:

| Variable | Descripció | Valor en l'exemple |
|----------|------------|-------------------|
| `DOCKER_INFLUXDB_INIT_MODE` | Mode d'inicialització (`setup` o `upgrade`) | `setup` |
| `DOCKER_INFLUXDB_INIT_USERNAME` | Nom d'usuari de l'administrador | `admin` |
| `DOCKER_INFLUXDB_INIT_PASSWORD` | Contrasenya de l'administrador | `admin123` |
| `DOCKER_INFLUXDB_INIT_ORG` | Organització (agrupació lògica) | `myorg` |
| `DOCKER_INFLUXDB_INIT_BUCKET` | Bucket inicial (base de dades) | `sensors` |
| `DOCKER_INFLUXDB_INIT_ADMIN_TOKEN` | **Token d'accés** (necessari per a totes les connexions) | `my-super-secret-token` |

> **IMPORTANT - El token**: El token és una clau d'autenticació necessària perquè qualsevol servei (Telegraf, Grafana, etc.) puga connectar a InfluxDB. En l'exemple hem posat `my-super-secret-token`, però en producció hauries de generar un token únic i complex.

### Com generar un token segur

Per generar un token segur, pots utilitzar:

```bash
# Generar un token aleatori de 32 caràcters (Linux/Mac)
openssl rand -base64 32

# O amb Python
python3 -c "import secrets; print(secrets.token_urlsafe(32))"

# O utilitzar un generador online com https://randomkeygen.com/
```

El token generat ha de reemplaçar `my-super-secret-token` a:
1. La variable `DOCKER_INFLUXDB_INIT_ADMIN_TOKEN` del docker-compose
2. El fitxer `telegraf.conf`
3. La configuració del datasource a Grafana

### Alternativa: InfluxDB sense inicialització automàtica

Si prefereixes configurar InfluxDB manualment des de la interfície web:

1. **Elimina** les variables d'inicialització del docker-compose:

```yaml
environment:
  - DOCKER_INFLUXDB_INIT_MODE=setup  # Eliminar
  # - DOCKER_INFLUXDB_INIT_USERNAME=admin  # Eliminar
  # ...
```

2. **Arranca** el contenidor: `docker compose -f docker-compose/influxdb.yml up -d`

3. **Accedeix** a http://localhost:8086 i segueix l'assistent de configuració

4. **Crea** una organització, bucket i token manualment

5. **Utilitza** el token generat a Grafana

### Configuració de Telegraf

**Telegraf** és l'agent que recull mètriques i les envia a InfluxDB. És altament configurable i suporta centenars de inputs i outputs.

```toml
# telegraf.conf
[agent]
  interval = "10s"          # Quan de temps entre lectures
  round_interval = true     # Arrodoneix l'interval
  metric_batch_size = 1000  # Punts per lot
  metric_buffer_limit = 10000

# ── INPUTS: Dades que recollim ────────────────────────────────
# CPU: Ús de processador
[[inputs.cpu]]
  percpu = true             # Per CPU individual
  totalcpu = true           # Total de totes les CPUs
  collect_cpu_time = false

# MEM: Ús de memòria
[[inputs.mem]]

# DISK: Espai en disc
[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660"]

# FILESYSTEM: Sistemes de fitxers
[[inputs.filesystem]]

# NET: Tràfic de xarxa
[[inputs.net]]
  interfaces = ["eth*", "en*"]

# PROCESSES: Processos en execució
[[inputs.processes]]

# SYSTEM: Informació general del sistema
[[inputs.system]]

# ── OUTPUT: On enviem les dades ────────────────────────────────
[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"]  # URL del contenidor
  token = "my-super-secret-token"  # Token d'autenticació
  organization = "myorg"            # Organització
  bucket = "sensors"                # Bucket destí
  content_encoding = "gzip"         # Compressió
```

### Verificar que Telegraf funciona

Després d'arrancar el stack, pots verificar que Telegraf està enviant dades:

```bash
# Veure logs de Telegraf
docker compose logs telegraf

# Hauries de veure missatges com:
# 2024-01-15T10:30:00Z I! Starting Telegraf 1.29.0
# 2024-01-15T10:30:00Z I! Loaded inputs: cpu disk mem net
# 2024-01-15T10:30:00Z I! Loaded outputs: influxdb_v2
```

També pots accedir a la interfície web d'InfluxDB (http://localhost:8086) i verificar que apareixen dades al bucket `sensors`.

### Configurar InfluxDB a Grafana

1. **Accedeix a Grafana**: http://localhost:3000
2. **Navega a connexions**: Menú lateral → **Connections** → **Data Sources**
3. **Afegeix un nou datasource**: Clica el botó blau **Add data source**
4. **Busca InfluxDB**: Escriu "InfluxDB" al cercador o busca'l a la categoria **Time series databases**
5. **Configura el datasource**:

| Camp | Valor | Explicació |
|------|-------|------------|
| Name | `InfluxDB Local` | Nom identificatiu |
| URL | `http://influxdb:8086` | URL del contenidor Docker |
| Auth | **Browser** | Mètode d'autenticació |
| Organization | `myorg` | Organització creada a InfluxDB |
| Token | `my-super-secret-token` | **Token generat anteriorment** |
| Default Bucket | `sensors` | Bucket on Telegraf envia les dades |

> **Important**: La URL ha de ser `http://influxdb:8086` (nom del servei Docker), no `localhost:8086`. Això és perquè Grafana accedix al servei des de dins de la xarxa Docker.

6. **Verifica la connexió**: Clica **Save & test**

Si tot funciona, veuràs un missatge verd: **"Data source is working"**

### Obtenir el token des de la interfície d'InfluxDB

Si no has utilitzat la inicialització automàtica i has configurat InfluxDB manualment:

1. Accedeix a http://localhost:8086
2. Inicia sessió amb les teues credencials
3. Ves a **Load Data** → **Tokens**
4. Clica el token que vulgues utilitzar o crea'n un de nou amb **Generate Token** → **Read/Write Token**
5. Copia el token i utilitza'l a Grafana

### Resolució de problemes

| Error | Solució |
|-------|---------|
| "Connection refused" | Verifica que InfluxDB està arrancat: `docker compose ps` |
| "Token is invalid" | Comprova que el token coincidisca exactament |
| "Bucket not found" | Verifica que el bucket `sensors` existix a InfluxDB |
| "Permission denied" | Crea un token amb permisos de lectura i escriptura |

### Flux: Llenguatge de consultes

InfluxDB 2.x utilitza **Flux** com a llenguatge de consultes.

**Consultes bàsiques**

```flux
// Totes les dades d'un mesurament
from(bucket: "sensors")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu")

// Amb condicions
from(bucket: "sensors")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> filter(fn: (r) => r.cpu == "cpu0")

// Múltiples measurements
from(bucket: "sensors")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu" or r._measurement == "mem")

// Agregacions
from(bucket: "sensors")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> mean()
```

**Funcions d'agregació**

```flux
// Mitjana
|> mean()

// Màxim
|> max()

// Mínim
|> min()

// Suma
|> sum()

// Comptador
|> count()

// Percentils
|> quantile(q: 0.95)
```

**Grouping i windowing**

```flux
from(bucket: "sensors")
  |> range(start: -24h)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> aggregateWindow(every: 5m, fn: mean)
  |> group(columns: ["cpu"])
```

### Queries a Grafana

A Grafana, les consultes Flux es defineixen en el panell:

```flux
from(bucket: "sensors")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> filter(fn: (r) => r._field == "usage_idle")
  |> filter(fn: (r) => r.cpu == "${cpu}")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> map(fn: (r) => ({r with _value: 100.0 - r._value}))
```

---

## MongoDB

**MongoDB** és una base de dades NoSQL de codi obert **orientada a documents**. A diferència de les bases de dades relacionals (SQL), MongoDB emmagatzema les dades en documents similars a **JSON** anomenats **BSON** (Binary JSON). Conceptes claus en MongoDB:

| Concepte MongoDB | Equivalent SQL | Descripció |
|-----------------|---------------|------------|
| **Database** | Database | Contenidor de col·leccions |
| **Collection** | Table | Grup de documents |
| **Document** | Row | Una entrada amb camps i valors |
| **Field** | Column | Parell clau-valor dins d'un document |
| **ObjectId** | Primary Key | Identificador únic del document |
| **Embedded document** | - | Document dins d'un altre document |

**Exemple de document:**

```json
{
  "_id": ObjectId("..."),
  "username": "admin",
  "email": "admin@example.com",
  "role": "admin",
  "profile": {
    "name": "Administrador",
    "department": "IT"
  },
  "created_at": ISODate("2024-01-15"),
  "tags": ["admin", "security", "manager"]
}
```

### Quan utilitzar MongoDB amb Grafana?

MongoDB és ideal quan:

- **Estructura flexible**: Les dades no tenen un esquema fix (sensors IoT amb diferents tipus de mesures)
- **Documents embebuts**: Pots guardar totes les dades relacionades en un sol document
- **Esquema variable**: L'estructura de les dades canvia amb el temps
- **Volum gran**: Milions de documents amb cerca ràpida per índex
- **Prototipat ràpid**: No cal definir l'esquema abans d'inserir dades

### Comparativa: MongoDB vs Bases de dades SQL

| Característica | MongoDB | PostgreSQL/MySQL |
|----------------|---------|-----------------|
| **Model** | Documents (BSON) | Taules (files/columnes) |
| **Esquema** | Flexible (sense esquema fix) | Rigid (cal definir-lo) |
| **Relacions** | Documents embebuts o referències | JOINs entre taules |
| **Transaccions** | Sessions, transactions | ACID complet |
| **Índexs** | Camps simples i compostos | Molt optimitzats |
| **Millor per** | IoT, logs, catàlegs | Dades transaccionals |

### Avantatges de MongoDB

- **Sense esquema fix**: Pots afegir nous camps sense migracions
- **Documents embebuts**: Redueix consultes, millora rendiment
- **Horitzontalment escalable**: Sharding per a dades massives
- **Índexs potents**: Suport per a índexs geoespacials, text, i compostos
- **BSON**: Suport natiu per a dates, ObjectId, i altres tipus especials

**docker-compose per a MongoDB**

```yaml
# docker-compose.mongodb.yml
services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb
    restart: unless-stopped
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin123
      MONGO_INITDB_DATABASE: monitoring
    volumes:
      - mongodb_data:/data/db
      - ./init.js:/docker-entrypoint-initdb.d/init.js
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - monitoring
    depends_on:
      - mongodb

volumes:
  mongodb_data: {}
  grafana_data: {}

networks:
  monitoring:
    driver: bridge
```

### Script d'inicialització

```javascript
// init.js
use('monitoring');

// Col·lecció de sensors IoT
db.sensors.insertMany([
    {
        sensor_id: "TEMP-001",
        type: "temperature",
        location: { building: "A", floor: 1, room: "101" },
        readings: [
            { timestamp: new Date(), value: 22.5, unit: "°C" },
            { timestamp: new Date(Date.now() - 3600000), value: 21.8, unit: "°C" }
        ]
    },
    {
        sensor_id: "HUM-001",
        type: "humidity",
        location: { building: "A", floor: 1, room: "101" },
        readings: [
            { timestamp: new Date(), value: 45.2, unit: "%" }
        ]
    }
]);

// Col·lecció de logs d'aplicació
db.app_logs.insertMany([
    { level: "INFO", message: "Application started", timestamp: new Date(), service: "api" },
    { level: "WARN", message: "High memory usage detected", timestamp: new Date(), service: "worker" },
    { level: "ERROR", message: "Connection timeout", timestamp: new Date(), service: "api" }
]);

print("MongoDB initialized with sample data");
```

### Plugin de MongoDB per a Grafana

Grafana no té suport natiu per a MongoDB. Necessitem instal·lar el plugin **"mongodb-datasource"**.

#### Instal·lació del plugin

```bash
docker exec -it grafana grafana-cli plugins install volkovlabs-mongodb-datasource
docker restart grafana
```

O afegir al docker-compose:

```yaml
grafana:
  ...
  environment:
    GF_INSTALL_PLUGINS: volkovlabs-mongodb-datasource
```

### Configurar MongoDB a Grafana

1. **Connections → Data Sources → Add data source**
2. Cerca **MongoDB** (ha d'aparèixer després d'instal·lar el plugin)
3. Configura:

| Camp | Valor |
|------|-------|
| Server | `mongodb://admin:admin123@mongodb:27017` |
| Database | `monitoring` |

4. Clica **Save & Test**

### Consultes MongoDB amb l'editor visual

L'editor de MongoDB a Grafana permet:

- **Collection**: Seleccionar la col·lecció
- **Find**: Filtrar documents
- **Project**: Seleccionar camps
- **Sort**: Ordenar resultats
- **Limit**: Limitar resultats

**Exemple: Temperatures del darrer dia**

```
Collection: sensors
Find: { type: "temperature" }
Project: { sensor_id: 1, readings: 1 }
```

### Pipeline d'agregació

Per a consultes complexes, utilitza l'aggregation pipeline:

```javascript
[
    { $match: { type: "temperature" } },
    { $unwind: "$readings" },
    { $match: { "readings.timestamp": { $gte: ISODate("2024-01-01") } } },
    { $group: {
        _id: "$sensor_id",
        avg_temp: { $avg: "$readings.value" },
        max_temp: { $max: "$readings.value" },
        min_temp: { $min: "$readings.value" }
    }},
    { $sort: { avg_temp: -1 } }
]
```

---

## Variables i Plantilles en Grafana

### Concepte de Variables

Les variables permeten crear dashboards dinàmics on pots canviar valors (servidors, sensors, time ranges) sense duplicar dashboards.

### Avantatges

- Un sol dashboard per a múltiples servidors
- Consultes més llegibles
- Filtratge dinàmic

### Crear Variables

1. Ves a **Dashboard Settings → Variables**
2. Clica **Add variable**

**Variable d'un datasource**

```
Name: server
Label: Servidor
Type: Query
Data source: Prometheus
Query: label_values(up, instance)
```

**Variable per a intervals de temps**

```
Name: interval
Label: Interval
Type: Interval
Values: 30s, 1m, 5m, 10m, 30m, 1h
```

**Variable amb valors personalitzats**

```
Name: environment
Label: Entorn
Type: Custom
Values: production, staging, development
```

### Utilitzar Variables en Consultes

```promql
# Substituir amb $variable
rate(node_cpu_seconds_total{instance="$server"}[5m])

# Amb expressió regular
rate(node_cpu_seconds_total{instance=~"$server.*"}[5m])

# Variable en el nom de mètrica
${metric_name}{instance="$server"}

# Variable com a interval
rate(node_cpu_seconds_total{instance="$server"}[$interval])
```

### Multi-select

```promql
# Seleccionar múltiples valors
{instance=~"${server:regex}"}

# Amb variable multi-select
rate(node_cpu_seconds_total{instance=~"${server:pipe}"}[5m])
```

### Variables amb PostgreSQL

```sql
-- Variable: Tots els servidors
SELECT DISTINCT server FROM cpu_metrics ORDER BY server

-- Variable: Rang de dates
SELECT 
    $__timeGroup(timestamp, '$__interval') as time,
    avg(cpu_usage) 
FROM cpu_metrics
WHERE $__timeFilter(timestamp)
  AND server = '$server'
GROUP BY 1
```

## Templates i Links

### Enllaços entre dashboards

1. Ves a **Dashboard Settings → Variables**
2. Clica **Add variable → Constant**
3. Configura:
   - Name: `target_dashboard`
   - Hide: Hidden
   - Options: ID del dashboard destí

4. En el panell, crea un link:
   - Type: **Link**
   - URL: `/d/$target_dashboard?var-server=${server}`

### Variables globals

```yaml
# grafana.ini
[users]
default_theme = light

[dashboards]
default_home_dashboard_path = /etc/grafana/provisioning/dashboards/overview.json
```

---

## Panells i Visualitzacions

Hi ha diferents tipus de visualitzacions que podem incorporar a un dashboard en Grafana. Alguns dels meś utilitzats són:

### Time Series (Sèrie Temporal)

El panell per defecte per a dades temporals.

**Configuració comuna:**
- **Graph styles**: Línia, àrea, barres
- **Line interpolation**: Suau, pas a pas
- **Point size**: Mida dels punts
- **Gradient mode**: Opacitat de l'àrea sota la corba

### Stat

Mostra un valor individual amb opcions de tendència.

```
Configuració:
- Show: Value only / Label and Value / All
- Color mode: Background solid / Background gradient / None
- Graph mode: None / Area
```

### Gauge

Indicador visual amb rangs de colors.

```
Configuració:
- Min/Max: Valors límit
- Thresholds: Verd (0-50), Groc (50-80), Vermell (80-100)
- Unit: percent, bytes, etc.
```

### Bar Gauge

Barres horitzontals o verticals amb gradient de colors.

### Table

Dades tabulars amb opcions de cerca i ordenació.

**Opcions:**
- **Cell display mode**: Color background, Color text, JSON view
- **Filter by name**: Cercar columnes
- **Sort by**: Ordenar per columna

### Pie Chart

Gràfic circular per mostrar proporcions.

### Histogram

Distribució de valors en intervals.

### Heatmap

Densitat de dades amb colors (similar a Kibana).

### Logs

Visualització de logs amb cerca i filtratge.

### Text

Panell estàtic amb suport Markdown i HTML.

## Transformacions

Les transformacions permeten modificar les dades abans de visualitzar-les. Alguns exemples:

**Ordenar per camp**

```
Input: Taula desordenada
Output: Taula ordenada per columna específica
```

**Agrupar per temps**

```
Combina múltiples series en intervals de temps comuns
```

**Fusionar camps**

```
Combina camps amb el mateix nom de múltiples consultes
```

**Reduir**

```
De sèrie temporal a valor únic (mitjana, suma, màxim, etc.)
```

**Outer Join**

```
Uneix consultes basant-se en una etiqueta comuna
```

**Configurar des de camp**

```
Crea columnes dinàmiques des de camps JSON
```

Podeu anar fent proves canviant la forma de mostrar la mateixa consulta. 

## Field Overrides

Els overrides permeten personalitzar camps específics sense afectar altres.

### Exemple: Color diferent per cada servei

1. Clica **Overrides → Add override**
2. Selecciona camps: `Series: /.*/`
3. Afegir regla:
   - **Fields with name**: instance
   - **Static text**: Color: #FF5733

### Overrides comuns

| Override | Ús |
|----------|-----|
| **Hide from** | Ocultar en legend, tooltip, etc. |
| **Display name** | Canviar el nom de la serie |
| **Colors** | Assignar colors condicionals |
| **Thresholds** | Canviar llindars |
| **Data links** | Afegir enllaços als valors |

---

## Sistema d'Alertes

El sistema d'alertes de Grafana (`Grafana Alerting`) té dos modes:

1. **Legacy Alerts**: Sistema clàssic (encara funcional)
2. **Grafana Alerting (unified alerting)**: Nou sistema centralitzat

Utilitzarem el **unified alerting** (recomanat).

### Crear una Alerta

1. Edita un panell existent
2. Clica en **Alert** a la pestanya lateral

#### Pas 1: Definir la consulta

```promql
# CPU superior al 80%
100 - (rate(node_cpu_seconds_total{mode="idle"}[5m]) * 100) > 80
```

#### Pas 2: Configurar condicions

```
WHEN: avg() OF query(A)
IS ABOVE: 80
FOR: 5m
```

#### Pas 3: Crear notificacions

1. Ves a **Alerting → Contact points**
2. Clica **Add contact point**

**Email**

```
Name: admin-email
Email: admin@example.com
```

**Slack**

```
Name: slack-alerts
Integration: Slack
Webhook URL: https://hooks.slack.com/services/XXX/YYY/ZZZ
```

**Webhook**

```
Name: webhook-alerts
Webhook URL: http://myservice:8080/alerts
```

#### Pas 4: Crear política d'alertes

1. Ves a **Alerting → Notification policies**
2. Clica **Add policy**
3. Configura:
   - **Match by**: Severity = critical
   - **Contact point**: slack-alerts
   - **Timing**: Send every 5m

### Templates d'Alertes

Personalitza el contingut dels missatges:

```go
{{ define "my_alert" }}
[{{ .Status }}] {{ .GroupLabels.alertname }}

{{ range .Alerts }}
  {{ if .Labels.instance }}Instance: {{ .Labels.instance }}{{ end }}
  {{ if .Labels.job }}Job: {{ .Labels.job }}{{ end }}
  Value: {{ .Values }}
  {{ .Annotations.summary }}
{{ end }}
{{ end }}
```

### docker-compose amb Alertmanager (opcional)

```yaml
# docker-compose.alerts.yml
services:
  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    restart: unless-stopped
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
      - alertmanager_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - monitoring

volumes:
  alertmanager_data: {}
  grafana_data: {}

networks:
  monitoring:
    driver: bridge
```

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#alerts'
        send_resolved: true
        api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
        title: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
```

---

## Importar i Exportar Dashboards

### Exportar un Dashboard

1. Ves al dashboard
2. Clica **Share → Export**
3. Selecciona **Export for sharing externally** (opcional)
4. Desa com a JSON

### Via API

```bash
curl -s -H "Authorization: Bearer $GF_API_KEY" \
  http://localhost:3000/api/dashboards/uid/$DASHBOARD_UID | jq .
```

### Importar un Dashboard

1. **Dashboards → Import**
2. Puja el fitxer JSON o enganxa el contingut
3. Selecciona el datasource
4. Clica **Import**

### Importar Dashboards predefinits des de Grafana.com

1. Ves a https://grafana.com/grafana/dashboards/
2. Cerca el dashboard (ex: "Node Exporter Full")
3. Clica **Copy ID to Clipboard**
4. A Grafana: **Dashboards → Import → Paste the ID**

## Aprovisionament (Provisioning)

Per crear dashboards automàticament amb Docker:

```yaml
# docker-compose.provisioned.yml
services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./provisioning/dashboards:/etc/grafana/provisioning/dashboards
      - ./provisioning/datasources:/etc/grafana/provisioning/datasources
      - ./dashboards:/var/lib/grafana/dashboards
    networks:
      - monitoring
```

```yaml
# provisioning/datasources/prometheus.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
```

```yaml
# provisioning/dashboards/dashboards.yml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    options:
      path: /var/lib/grafana/dashboards
```

---

## Troubleshooting

### "Connection refused" o "Connection timeout"

**Causes possibles:**

1. **El servei no està arrancat**
   ```bash
   docker compose ps
   ```

2. **El port està ocupat**
   ```bash
   # Linux
   lsof -i :3000
   
   # Windows (PowerShell)
   netstat -ano | findstr :3000
   ```

3. **Xarxes Docker diferents**
   ```bash
   # Verificar que Grafana i el datasource estan en la mateixa xarxa
   docker network inspect monitoring
   ```

**Solució:** Assegura't que Grafana pot accedir al datasource utilitzant el nom del servei Docker (no `localhost`).

### "Permission denied" a la base de dades

**PostgreSQL:**
```sql
-- Crear usuari amb permisos
GRANT ALL PRIVILEGES ON DATABASE monitoring TO grafana;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO grafana;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO grafana;
```

**MySQL:**
```sql
GRANT ALL PRIVILEGES ON monitoring.* TO 'grafana'@'%';
FLUSH PRIVILEGES;
```

### Problemes amb Prometheus

#### "No data" en els panells

1. **Verificar que Prometheus scrapeja les dades:**
   ```bash
   curl http://localhost:9090/api/v1/targets
   ```

2. **Comprovar expressions al navegador de Prometheus:**
   - Ves a http://localhost:9090/graph
   - Executa la consulta directament

3. **Verificar el fitxer prometheus.yml:**
   - Els targets han d'utilitzar el nom del servei Docker
   - Els ports han de ser correctes

#### "Timeout" en les consultes

Afegir `timeout` al fitxer prometheus.yml:

```yaml
global:
  scrape_timeout: 10s
  evaluation_interval: 15s
```

### Problemes amb Grafana

#### Pàgina en blanc o errors de CORS

Verificar la configuració de CORS al grafana.ini:

```ini
[server]
domain = localhost

[cors]
enabled = true
allow_credentials = true
allow_origins = *
```

#### Dashboards molt lents

1. **Reduir el rang de dates** a la consulta
2. **Utilitzar regles de gravació** per a consultes complexes
3. **Canviar l'interval de scrutini:**
   ```promql
   rate(node_cpu_seconds_total[5m])  # No 1m o 10s
   ```

#### Dades incorrectes amb dates

Assegurar-se que el timezone és correcte:

```yaml
environment:
  - GF_TIMEZONE=Europe/Madrid
```

### Comprovacions Ràpides

```bash
# 1. Status de tots els contenidors
docker compose ps

# 2. Logs d'un servei
docker compose logs grafana --tail=50

# 3. Logs en temps real
docker compose logs -f prometheus

# 4. Accedir a un contenidor
docker exec -it grafana /bin/sh

# 5. Verificar que el port està escoltant
curl -I http://localhost:3000

# 6. Testar connexió des de Grafana a Prometheus
docker exec -it grafana wget -qO- http://prometheus:9090/-/healthy

# 7. Inspectar xarxes
docker network ls
docker network inspect monitoring
```

### Reset Complet

Si tens problemes persistents, prova un reset complet:

```bash
# Aturar i eliminar contenidors i volums
docker compose down -v

# Eliminar imatges antigues (opcional)
docker image prune -f

# Recompilar
docker compose up -d

# Verificar
docker compose ps
```

---

## Referència Ràpida

### Instruccions Docker

```bash
# Arrancar
docker compose up -d

# Aturar
docker compose down

# Aturar i eliminar volums
docker compose down -v

# Reiniciar un servei
docker compose restart grafana

# Veure logs
docker compose logs -f grafana

# Entrar al contenidor
docker exec -it grafana /bin/sh
```

### Credencials per defecte

| Servei | Usuari | Contrasenya |
|--------|--------|-------------|
| Grafana | admin | admin123 |
| Prometheus | - | - |
| PostgreSQL | grafana | grafana123 |
| MySQL | grafana | grafana123 |
| InfluxDB | admin | admin123 |
| MongoDB | admin | admin123 |

### Ports

| Port | Servei |
|------|--------|
| 3000 | Grafana |
| 9090 | Prometheus |
| 9100 | node-exporter |
| 5432 | PostgreSQL |
| 3306 | MySQL |
| 8086 | InfluxDB |
| 27017 | MongoDB |
| 9093 | Alertmanager |
| 3100 | Loki |

## Recursos

- Documentació oficial: https://grafana.com/docs/grafana/latest/
- PromQL reference: https://prometheus.io/docs/prometheus/latest/querying/basics/
- Dashboards compartits: https://grafana.com/grafana/dashboards/
- Plugins: https://grafana.com/grafana/plugins/
