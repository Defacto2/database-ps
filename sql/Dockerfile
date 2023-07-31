# syntax=docker/dockerfile:1
FROM alpine:3.18

WORKDIR /opt/sql
COPY create-table.sql /opt/sql/

# WINDOWS USERS
# EOL-Sequence of this file MUST always be LF (Unix) or the RUN command will fail.
RUN <<EOT
  apk version zstd curl
  apk add zstd curl
  curl https://defacto2.net/sql/d2-sql-insert.sql --output d2-sql-insert.sql
  cat create-table.sql d2-sql-insert.sql > create-defacto2.sql
  mkdir -p /docker-entrypoint-initdb.d
  cp -v create-defacto2.sql /docker-entrypoint-initdb.d
  rm d2-sql-insert.sql create-table.sql
  zstd --rm /docker-entrypoint-initdb.d/create-defacto2.sql
  apk del --quiet zstd curl
EOT