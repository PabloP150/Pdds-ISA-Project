# Spring PetClinic — Source Code

Este directorio contiene el codigo fuente de la aplicacion Spring PetClinic.

## Quick Start (Docker)

```sh
docker-compose up --build -d
```

La app estara disponible en **http://localhost:8080** (~20s de inicio).

## Quick Start (Local)

```sh
./gradlew bootRun
# o con Maven:
./mvnw spring-boot:run
```

Requiere Java 17+.

## Estructura

```
src/
├── main/java/.../petclinic/
│   ├── owner/          # Owners, Pets, Visits
│   ├── vet/            # Veterinarios y Especialidades
│   ├── model/          # Entidades base
│   └── system/         # Configuracion (cache, web, crash)
└── main/resources/
    ├── templates/       # Vistas Thymeleaf
    ├── db/              # Scripts SQL (H2, MySQL, Postgres)
    └── application.properties
```

## Base de Datos

Por defecto usa **H2 in-memory**. Para MySQL (Docker Compose):

```sh
docker-compose up
```

Esto levanta la app con perfil `mysql` conectada a MySQL 9.5.

## License

[Apache License 2.0](LICENSE.txt)
