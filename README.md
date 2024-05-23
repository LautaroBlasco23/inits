# repository developed for personal purposes.

I want to store here some things about software that I personally found useful.

**For example:**

* Boilerplate code.
* Properties for spring boot projects.
* Basic docker scripts


# RSA generate private and public key.

**create rsa key pair:**

```
openssl genrsa -out keypair.pem 2048
```

**extract public key:**

```
openssl rsa -in keypair.pem -pubout -out public.pem
```

**create private key in PKCS#8 format:**

```
openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in keypair.pem -out private.pem
```

**if you want to use this keys in spring boot app: (yaml properties)**


```
jwt:
  rsa-private-key: classpath:certs/privateKey.pem
  rsa-public-key: classpath:certs/publicKey.pem
```

# PostgreSQL properties for spring boot app (yaml properties).

```
spring:
  # PostgreSQL
  datasource:
    url: jdbc:postgresql://localhost:5432/ecommercedb
    username: lauti
    password: test123
    driver-class-name: org.postgresql.Driver
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    hibernate:
      ddl-auto: update
```

add to hibernate if you want debug mode:

```
properties:
  hibernate:
    show_sql: true
    format_sql: true
```

# PostgreSQL Init script to create a generic database with a basic users table in it.

**Init sql:**

```
CREATE DATABASE generic;

GRANT ALL PRIVILEGES ON DATABASE generic TO testing;

\c generic

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    password VARCHAR(255)
);
```

**Dockerfile:**

```
FROM postgres:latest

COPY init.sql /docker-entrypoint-initdb.d/

ENV POSTGRES_USER=testing
ENV POSTGRES_PASSWORD=testing
```

**Makefile:**

```
database-up:
	docker build -t my-postgres .
	docker run -d -p 5432:5432 my-postgres
```

