# Домашнее задание к занятию "`Репликация и масштабирование. Часть 1`" - `Маркин Алексей`

### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

---

### Решение 1

Режим master-slave нужен когда пишется на один сервер, а читается со второго.

Режим master-master позвоялет записывать и читать данные с обоих серверов.

---

### Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

---

### Решение2

docker-compose.yml
```yml
services:
  mysql-master:
    image: 'mysql:8.0'
    container_name: mysql-master
    env_file: .env
    volumes:
      - ./master/my.cnf:/etc/my.cnf
      - ./master/master.sql:/docker-entrypoint-initdb.d/start.sql
    environment:
      - MYSQL_ROOT_PASSWORD:${MYSQL_ROOT_PASSWORD}
    ports:
      - 3306:3306

  mysql-slave:
    image: 'mysql:8.0'
    container_name: mysql-slave
    env_file: .env
    volumes:
      - ./slave/my.cnf:/etc/my.cnf
      - ./slave/slave.sql:/docker-entrypoint-initdb.d/start.sql
    environment:
      - MYSQL_ROOT_PASSWORD:${MYSQL_ROOT_PASSWORD}
    ports:
      - 3307:3306
    depends_on:
      - mysql-master
```

my.cnf Master
```
[mysqld]
server-id=1
binlog_format=ROW
log-bin
```

my.cnf Slave
```
[mysqld]
server-id=2
read_only = 1
```

start.sql Master

```sql
CREATE USER repl@'%' IDENTIFIED WITH mysql_native_password BY 'slavepass';
GRANT REPLICATION SLAVE ON *.* TO repl@'%';
```

start.sql Slave
```sql
CHANGE MASTER TO 
MASTER_HOST='mysql-master',
MASTER_USER='repl',
MASTER_PASSWORD='slavepass';
```


![Задание 2](https://github.com/Markin-AI/12-6/blob/main/img/2-1.png)

![Задание 2](https://github.com/Markin-AI/12-6/blob/main/img/2-2.png)

![Задание 2](https://github.com/Markin-AI/12-6/blob/main/img/2-3.png)

![Задание 2](https://github.com/Markin-AI/12-6/blob/main/img/2-4.png)


---