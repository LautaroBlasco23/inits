# repository developed for personal purposes.

I want to store here some things about software that I personally found useful.

**For example:**

* Boilerplate code.
* Properties for spring boot projects.
* Basic docker scripts

# Script to install .deb packages in ubuntu (just to remember)

```
dpkg -i archivo.deb
```

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

You should create the RSAKeyRecord class:

```
@ConfigurationProperties(prefix = "jwt")
public record RSAKeyRecord (RSAPublicKey rsaPublicKey, RSAPrivateKey rsaPrivateKey){
}
```

And append it to your main class:

```
@EnableConfigurationProperties(RSAKeyRecord.class)
```

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
    id BIGINT PRIMARY KEY,
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

# Cosas interesantes de SQL

Script para crear una tabla que une a comentarios con usuarios en una relacion many to many. y tiene como primary key la combinacion de ambos ids, para evitar que se repitan.

```
CREATE TABLE comment_likes (
  comment_id BIGINT NOT NULL,
  user_id BIGINT NOT NULL,
  PRIMARY KEY (comment_id, user_id),
  FOREIGN KEY (comment_id) REFERENCES comments(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users_profiles(id) ON DELETE CASCADE
);
```

# Entidades en JPA con tablas de relaciones

```
    // ejemplo usando @JoinTable y @ManyToMany, para unir usuarios y las rutinas que están haciendo en ese momento.
    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "users_routines",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "routine_id")
    )
    private List<RoutineEntity> routines;

    // Y otro ejemplo de mi tabla de users:
    @ManyToMany
    @JoinTable(
        name = "users_follows",
        joinColumns = @JoinColumn(name = "followed"),
        inverseJoinColumns = @JoinColumn(name = "follower")
    )
    private Set<UserEntity> followers;

    @ManyToMany(mappedBy = "followers", fetch = FetchType.LAZY, cascade = {CascadeType.REMOVE})
    private Set<UserEntity> following;
```
