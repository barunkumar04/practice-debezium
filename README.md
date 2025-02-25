# practice-debezium

## What, How & Why
  - What: An open source CDC tool, used for streaming database chnges. Built on top of kafka, which enables it to let applications consume changed data, correctly and ocmpletly.
  - How: Reads transaction log and uses Kafka to publish it to consumer(s).
  - Why: To solve business problems like data sycn, data refresh etc.

## Tutorial
  - https://debezium.io/documentation/reference/3.0/tutorial.html

## Steps

### Docker
  - Docker is used for containrize services required, like Kafka, connector etc.
  - Refer here: https://docs.docker.com/engine/install/
  - Docker Desk top is used for this tutorial.

### Zookeeper
  - Image pull: docker pull quay.io/debezium/zookeeper:latest
  - Start: docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 quay.io/debezium/zookeeper:latest

### Kafka
  - Image pull: docker pull quay.io/debezium/kafka:latest
  - Start: docker run -it --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper quay.io/debezium/kafka:latest

### MySQL database
  - Image pull: docker pull quay.io/debezium/example-mysql:latest
  - Start: docker run -it --rm --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw quay.io/debezium/example-mysql:latest

### MySQL CLI client 
  - Start: docker run -it --rm --name mysqlterm --link mysql mysql:latest sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
  - In case of error, try running from Docker Desktop. Set this as env variable: MYSQL_ROOT_PASSWORD:debezium
#### Steps to connect to mysql>
      - In Docker Desktop -> Container -> Terminal -> mysql -uroot -pdebezium
#### Prep dataset for demo
      - create databse practice;
      - use database practice;
      - create table
          CREATE TABLE Employee (
          EmployeeId int,
          LastName varchar(255),
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255),
          Designation varchar(30),
          YOE int 
          );
     - insert data
      INSERT INTO Employee (EmployeeId, LastName, FirstName, Address, City, Designation, YOE) VALUES (1, ’Wood’, ‘John’, ’somewhere’ , ’somewhere’ ,’Associate’, 10);
      INSERT INTO Employee (EmployeeId, LastName, FirstName, Address, City, Designation, YOE) VALUES (2, ’Mehta’, ‘Govind’, ’somewhere’ , ’somewhere’ ,’Associate’, 10);
      INSERT INTO Employee (EmployeeId, LastName, FirstName, Address, City, Designation, YOE) VALUES (3, ’Wilmer’, ‘Rob’,’somewhere’ , ’somewhere’ ,’Manager’, 20);

### Kafka connect
  - Image pull: docker pull quay.io/debezium/connect:latest
  - Start: $ docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link kafka:kafka --link mysql:mysql quay.io/debezium/connect:latest
#### Check if any connector is registered
  - curl -H "Accept:application/json" localhost:8083/connectors/

### Deploying Debezium MySQL connector
  - $ curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "topic.prefix": "dbserver1", "database.include.list": "inventory", "schema.history.internal.kafka.bootstrap.servers": "kafka:9092", "schema.history.internal.kafka.topic": "schemahistory.inventory" } }'

  - Note: Above connector uses
     -   user: debezium : CREATE USER 'debezium'@'%' IDENTIFIED BY 'dbz';
     -   grant: GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'debezium'@'%' WITH GRANT O PTION;
     -   Create tables and data accordingly
 -   Now check "Check if any connector is registered" and you should see a connector listed.

### Vewing Create event
  - Add watcher: $ docker run -it --rm --name watcher --link zookeeper:zookeeper --link kafka:kafka quay.io/debezium/kafka:latest watch-topic -a -k dbserver1.inventory.Employee



