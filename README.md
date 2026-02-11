# Reasoning Behind the Choice of Project

### 1. Real-World Architectural Challenges
While other projects are intentionally broken or vulnerable, **Spring PetClinic** represents the most frequent challenge faced by developers: the **monolithic design** it currently has. Selecting this allows us to gain experience decoupling legacy dependencies without disrupting existing business value.

### 2. Implementation of Domain-Driven Design
PetClinic provides a clear business domain (Vets, Owners, Visits) that is currently coupled at the database and service layers. This gives us a good opportunity to implement a **Domain-Driven Design**, which will allow us to align software structure with business logic.

### 3. Ecosystem
Since this project uses a **Java/Spring ecosystem**, this gives us a great environment to integrate various observability tools like SonarQube for analysis.