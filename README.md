# ratticdb-docker-mysql
ratticdb on docker with uwsgi, nginx and mysql

### Mount directory must have SELinux privileges

```
chcon -Rt svirt_sandbox_file_t /root/docker/mysql/ratticdb

```

### Run mysql server

```
docker run -d \
	--name ratticdb \
  	--restart=always \
  	-v /root/docker_vol/mysql/ratticdb:/var/lib/mysql \
  	-e MYSQL_ROOT_PASSWORD=r00tme \
  	-e MYSQL_DATABASE=rattic \
  	-e MYSQL_USER=rattic \
  	-e MYSQL_PASSWORD=***rattic-db-password*** \
  	mysql/mysql-server:5.6
```

add grant to rattic user on db open docker to login


```
docker exec -it ratticdb bash

mysql -u root -pr00tme
    create database rattic;
    create user 'rattic'@'%' identified by '***rattic-db-password***';
    grant all on rattic.* to rattic;
exit
```

### Run uwsgi server

Move to *docker-ratticdb-uwsgi/* and build docker

```
docker build -t kilerkarol/ratticdb-uwsgi .
```

then run it

```
docker run -d \
	--name 'ratticdb-uwsgi' \
  	--link 'ratticdb:mysql' \
		-e 'TIMEZONE=UTC' \
	  -e 'VIRTUAL_HOST=somedomain.example.com' \
	  -e 'SECRETKEY=someverysecretkeyforsessions' \
	  -e 'EMAIL_HOST=smtp.example.com' \
	  -e 'EMAIL_PORT=587' \
	  -e 'EMAIL_USER=example@example.com' \
	  -e 'EMAIL_PASSWORD=someemailpassword' \
	  -e 'EMAIL_FROM=emailed-from@example.com' \
  	kilerkarol/ratticdb-uwsgi
```

### Run nginx server
Move to *docker-ratticdb-nginx/* and build docker

```
docker build -t kilerkarol/ratticdb-nginx .
```

then run it

```
docker run \
		--name 'ratticdb-nginx' \
		-p 80:80 \
		-e 'PROXY_MODE=on' \
		-e 'VIRTUAL_HOST=somedomain.example.com' \
		-e 'CERT_NAME=default' \
		--link 'ratticdb-uwsgi:uwsgi' \
		--volumes-from 'ratticdb-uwsgi' \
  	kilerkarol/ratticdb-nginx
```
