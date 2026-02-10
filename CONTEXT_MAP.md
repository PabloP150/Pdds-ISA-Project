# AI-Driven Discovery: Context Map & Architecture

**Methodology:**
We utilized a Large Language Model (LLM) to perform a static analysis of the source code located in `src/main/java/org/springframework/samples/petclinic`. The goal was to reverse-engineer the architecture and identify Bounded Contexts based on package cohesion, inheritance structures, and JPA annotations.

### 1. AI Codebase Analysis (Deep Dive)

The analysis of the `org.springframework.samples.petclinic` package revealed a **Monolithic Architecture** structured around three distinct Domain Areas.

#### A. Core Domain: The "Owner" Context
* **Definition:** This is the heart of the business logic. It handles the primary workflow of the clinic (registering clients and treating patients).
* **Evidence in Code:**
    * **Aggregate Root:** The `Owner` class acts as the root. It holds a strictly consistent collection of `Pet` entities (`@OneToMany` relationship).
    * **Bounded Data:** The `Visit` entity is located inside the `owner` package (not separate). This confirms that a Visit has no meaning without a Pet.
    * **Key Insight:** Code analysis shows `Owner.java` uses eager fetching for pets, enforcing that an Owner is always loaded with their context.

#### B. Supporting Domain: The "Vet" Context
* **Definition:** A supporting subdomain that manages internal staff data. It provides reference data to the Core Domain but does not depend on it.
* **Evidence in Code:**
    * **Independence:** The `Vet` entity defines a Many-to-Many relationship with `Specialty` (`@ManyToMany`), but has **zero** dependencies on Owners or Pets.
    * **Separation:** There are no foreign keys in the `Vet` class linking to specific visits. The link is logical (via the schedule), not structural in the database.

#### C. Shared Kernel: The "Model" Package
* **Definition:** A library of shared code used by multiple contexts to enforce consistency and reduce duplication (DRY Principle).
* **Evidence in Code:**
    * **Base Classes:** We identified `Person.java` and `BaseEntity.java` annotated with `@MappedSuperclass`.
    * **Impact:** Both `Owner` (Core) and `Vet` (Supporting) extend `Person`. This creates a permanent coupling: any change to `Person` (e.g., changing how names are stored) immediately affects both domains.

---

### 2. Architecture Diagram (MermaidJS)

This diagram visualizes the **Inheritance Strategies** and **Composition** discovered during the audit.

```mermaid
classDiagram
    %% Styles & Namespaces
    classDef core fill:#e3f2fd,stroke:#1565c0,stroke-width:2px;
    classDef support fill:#f3e5f5,stroke:#4a148c,stroke-width:2px;
    classDef shared fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,stroke-dasharray: 5 5;

    %% --- SHARED KERNEL ---
    namespace Shared_Kernel {
        class BaseEntity {
            <<MappedSuperclass>>
            +Integer id
            +isNew()
        }
        class NamedEntity {
            +String name
            +toString()
        }
        class Person {
            <<MappedSuperclass>>
            +String firstName
            +String lastName
        }
    }

    %% --- CORE DOMAIN ---
    namespace Owner_Context {
        class Owner {
            <<Aggregate Root>>
            +String address
            +String city
            +String telephone
            +getPets()
        }
        class Pet {
            <<Entity>>
            +LocalDate birthDate
            +getVisits()
        }
        class Visit {
            <<Entity>>
            +LocalDate date
            +String description
        }
        class PetType {
            <<Value Object>>
            +String name
        }
    }

    %% --- SUPPORTING DOMAIN ---
    namespace Vet_Context {
        class Vet {
            <<Entity>>
            +getSpecialties()
        }
        class Specialty {
            <<Entity>>
        }
    }

    %% Relationships - Inheritance
    BaseEntity <|-- NamedEntity : extends
    BaseEntity <|-- Person : extends
    Person <|-- Owner : extends
    Person <|-- Vet : extends
    NamedEntity <|-- Pet : extends
    NamedEntity <|-- PetType : extends
    NamedEntity <|-- Specialty : extends
    BaseEntity <|-- Visit : extends

    %% Relationships - Aggregation/Composition
    Owner "1" *-- "0..*" Pet : @OneToMany
    Pet "1" *-- "0..*" Visit : @OneToMany
    Pet --> PetType : has type
    Vet --> Specialty : @ManyToMany

    %% Styling Application
    class Owner core
    class Pet core
    class Visit core
    class PetType core
    class Vet support
    class Specialty support
    class BaseEntity shared
    class NamedEntity shared
    class Person shared```