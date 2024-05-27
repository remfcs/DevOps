docker run -d --name mypostgresqlguigui --network app-network -v /volume_DM:/var/lib/postgresql/data my_postgres:v1

docker build -t my_postgres:v1 .

docker run -p 8090:8080 --net=app-network --name=adminer -d adminer

dockerfile:
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

COPY CreateScheme.sql /docker-entrypoint-initdb.d/
COPY InsertData.sql /docker-entrypoint-initdb.d/

