# Spring PetClinic — PDDS ISA Project

[![Java](https://img.shields.io/badge/Java-17-orange)](https://openjdk.org/projects/jdk/17/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0.1-green)](https://spring.io/projects/spring-boot)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue)](https://docs.docker.com/compose/)
[![License](https://img.shields.io/badge/License-Apache%202.0-lightgrey)](PetClinic/LICENSE.txt)

Proyecto basado en [Spring PetClinic](https://github.com/spring-projects/spring-petclinic) para el curso de Ingenieria de Software (PDDS-ISA).

<img width="1042" alt="petclinic-screenshot" src="https://cloud.githubusercontent.com/assets/838318/19727082/2aee6d6c-9b8e-11e6-81fe-e889a5ddfded.png">

---

## Tabla de Contenido

- [Quick Start](#quick-start)
- [Tech Stack](#tech-stack)
- [Deliveries](#deliveries)
  - [Delivery 1 — Setup Inicial](#delivery-1--setup-inicial)
  - [Delivery 2 — CI/CD Pipeline & Testing](#delivery-2--cicd-pipeline--testing)
  - [Delivery 3 — Security Hardening](#delivery-3--security-hardening-devsecops)
  - [Delivery 4 — One-Command Setup & ADR](#delivery-4--one-command-setup--adr)
  - [Delivery 5 — FinOps Optimization](#delivery-5--finops-optimization--performance)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Equipo](#equipo)

---

## Quick Start

```sh
cd PetClinic
docker-compose up --build -d
```

La app estara disponible en **http://localhost:8080** (esperar ~20s a que Spring Boot inicie).

### Requisitos
- Docker & Docker Compose

---

## Tech Stack

| Categoria | Tecnologia |
|---|---|
| Lenguaje | Java 17 |
| Framework | Spring Boot 4.0.1 |
| ORM | Hibernate / Spring Data JPA |
| Base de Datos | MySQL 9.5 (Docker) / H2 (desarrollo) |
| Frontend | Thymeleaf + Bootstrap |
| Build | Gradle 9.4 / Maven |
| Contenedores | Docker + Docker Compose |
| CI/CD | GitHub Actions |
| Seguridad | Trivy (vulnerability scanner), CycloneDX (SBOM) |
| Cache | JCache (JSR-107) |
| Testing | JUnit 5, Spring Test, Testcontainers |

---

## Deliveries

### Delivery 1 — Setup Inicial
**Branch:** `main`

Setup inicial del proyecto Spring PetClinic. Configuracion base del repositorio, estructura del proyecto y primer despliegue local.

---

### Delivery 2 — CI/CD Pipeline & Testing
**Branch:** `delivery2` | [Documentacion](docs/delivery2/Documentation_Delivery2.pdf)

Configuracion del pipeline de integracion y entrega continua con GitHub Actions. Implementacion de tests automatizados para validar la calidad del codigo en cada push.

---

### Delivery 3 — Security Hardening (DevSecOps)
**Branch:** `delivery3` | [Documentacion](docs/delivery3/Documentation_Delivery3.md) · [PDF](docs/delivery3/Documentation_Delivery3.pdf)

| Tarea | Herramienta | Resultado |
|---|---|---|
| SBOM Generation | CycloneDX Maven Plugin | 108 componentes documentados |
| Vulnerability Scanning & Patching | Trivy 0.69.3 | 3 vulnerabilidades encontradas → 0 tras parche |
| Secret Protection | Pre-commit Hook (shell) | Bloquea API keys, tokens y passwords |

Reportes: [Before](docs/delivery3/trivy-report-before.txt) · [After](docs/delivery3/trivy-report-after.txt)

---

### Delivery 4 — One-Command Setup & ADR
**Branch:** `delivery4` | [ADR-001](docs/delivery4/ADR-001-migration-to-microservices.md) · [PDF](docs/delivery4/ADR_%20Migration%20to%20a%20Service-Based%20Architecture.pdf)

| Tarea | Descripcion |
|---|---|
| Docker Setup | `Dockerfile` multi-stage (Gradle build + JRE 17) + `docker-compose.yml` con MySQL 9.5 |
| ADR | Propuesta de migracion a arquitectura basada en servicios (Vets, Visits, Customer) |

Un solo comando levanta la app completa: `docker-compose up --build`

---

### Delivery 5 — FinOps Optimization & Performance
**Branch:** `delivery5` | [Documentacion](docs/delivery5/Documentation_Delivery5.md) · [PDF](docs/delivery5/Documentation_Delivery5.pdf)

**Problema identificado:** Las entidades JPA usaban `FetchType.EAGER` en cadena (Owner → Pets → Visits), generando queries N+1 innecesarias y alto consumo de memoria/DB.

**Optimizaciones aplicadas:**

| # | Cambio | Archivo | Impacto |
|---|---|---|---|
| 1 | EAGER → LAZY loading | `Owner.java`, `Pet.java`, `Vet.java` | Evita carga innecesaria de datos relacionados |
| 2 | JOIN FETCH queries | `OwnerRepository.java`, `VetRepository.java` | 1 query eficiente en vez de N+1 |
| 3 | @Cacheable petTypes | `PetTypeRepository.java`, `CacheConfiguration.java` | Elimina queries repetitivas |
| 4 | Optimizacion sort | `Vet.java` | Reduce overhead de CPU |

**Resultados del benchmark:**

| Metrica | Before | After | Mejora |
|---|---|---|---|
| `GET /owners?lastName=` | 14.8ms | 9.8ms | **33.8% mas rapido** |
| Queries DB (owner list) | 1 + N + N*M | 1 | **~85% menos queries** |
| Queries DB (petTypes) | 1/request | 1 (cached) | **~99% menos queries** |

Benchmarks: [Before](docs/delivery5/benchmark-before.txt) · [After](docs/delivery5/benchmark-after.txt)

---

## Estructura del Proyecto

```
/
├── PetClinic/                       # Codigo fuente
│   ├── Dockerfile                   # Imagen Docker multi-stage
│   ├── docker-compose.yml           # App + MySQL
│   ├── pom.xml / build.gradle       # Build config
│   ├── src/                         # Codigo Java + recursos
│   ├── sbom.xml / sbom.json         # Software Bill of Materials
│   └── .githooks/pre-commit         # Hook para bloquear secretos
│
├── docs/                            # Documentacion por delivery
│   ├── delivery2/                   # CI/CD Pipeline & Testing
│   ├── delivery3/                   # Security (Trivy reports, SBOM)
│   ├── delivery4/                   # ADR Microservices
│   └── delivery5/                   # Benchmarks & FinOps
│
└── README.md
```

---

## Equipo

| Nombre |
|---|
| Pablo Pineda |
| Christian Martinez |

Curso: **PDDS — Ingenieria de Software Aplicada (ISA)**