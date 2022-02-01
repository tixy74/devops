FROM postgres:11.6-alpine
ENV POSTGRES_DB=db \
POSTGRES_USER=admin
COPY scrpits/01-CreateScheme.sql /docker-entrypoint-initdb.d
COPY scrpits/02-InsertData.sql /docker-entrypoint-initdb.d