# Analisis Completo: Spring PetClinic (pdds-IS-spring-petclinic-project)

> Generado el 23 de febrero de 2026. Autor: Pablo Pineda — Curso PDDS-ISA.
> Archivo de referencia para Claude. Contiene TODO el codigo fuente, configuraciones, tests y recursos.

---

## INDICE

1. [Identificacion del proyecto](#1-identificacion)
2. [Build tools y versiones](#2-build-tools)
3. [Dependencias completas](#3-dependencias)
4. [Plugins de build](#4-plugins)
5. [CI/CD — build.yml completo](#5-cicd)
6. [Arquitectura y estructura de paquetes](#6-arquitectura)
7. [Codigo fuente completo — Main](#7-codigo-fuente-main)
8. [Codigo fuente completo — Tests](#8-codigo-fuente-tests)
9. [Base de datos — Schemas SQL](#9-base-de-datos)
10. [Configuracion — Properties](#10-configuracion-properties)
11. [Templates Thymeleaf](#11-templates)
12. [Internacionalizacion](#12-i18n)
13. [Infraestructura DevOps](#13-devops)
14. [Deuda tecnica — Hotspots](#14-deuda-tecnica)
15. [Historial Git](#15-historial-git)
16. [Archivos raiz](#16-archivos-raiz)

---

## 1. IDENTIFICACION

| Campo | Valor |
|-------|-------|
| GroupId | org.springframework.samples |
| ArtifactId | spring-petclinic |
| Version | 4.0.0-SNAPSHOT |
| Spring Boot Parent | 4.0.1 |
| Java requerido | 17 |
| Rama actual | delivery2 |
| Rama principal | main |
| Repo origin | https://github.com/PabloP150/pdds-IS-spring-petclinic-project.git |
| Repo destino | https://github.com/PabloP150/Pdds-ISA-Project.git |
| SonarCloud org | pablop150 |
| SonarCloud project key | PabloP150_Pdds-ISA-Project |
| SonarCloud host | https://sonarcloud.io |

---

## 2. BUILD TOOLS

### Maven (principal) — version 3.9.12
- Wrapper: `.mvn/` — Distribution: apache-maven-3.9.12-bin.zip
- `./mvnw spring-boot:run` — levantar app
- `./mvnw package` — empaquetar
- `./mvnw test` — correr tests
- `./mvnw -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar` — CI completo con Sonar
- `./mvnw package -P css` — compilar SCSS a CSS

### Gradle (alternativo) — version 9.2.1
- Wrapper: `gradle/wrapper/gradle-wrapper.properties`
- `./gradlew bootRun` — levantar app
- `./gradlew build` — build completo

---

## 3. DEPENDENCIAS

### Spring Boot Starters (compile)
- spring-boot-starter-actuator — endpoints /actuator (health, metrics, info)
- spring-boot-starter-cache — soporte @Cacheable
- spring-boot-starter-data-jpa — ORM con Hibernate
- spring-boot-starter-thymeleaf — templates HTML server-side
- spring-boot-starter-validation — Jakarta Validation (@NotBlank, @Pattern)
- spring-boot-starter-webmvc — Spring MVC

### Core
- javax.cache:cache-api — JCache API (JSR-107)
- jakarta.xml.bind:jakarta.xml.bind-api — JAXB para XML/JSON marshalling

### Runtime
- com.h2database:h2 — BD en memoria (default dev)
- com.github.ben-manes.caffeine:caffeine — provider de cache Caffeine
- com.mysql:mysql-connector-j — driver MySQL
- org.postgresql:postgresql — driver PostgreSQL
- org.webjars:webjars-locator-lite:1.1.2
- org.webjars.npm:bootstrap:5.3.8
- org.webjars.npm:font-awesome:4.7.0

### Development
- spring-boot-devtools (optional=true) — hot reload

### Test
- spring-boot-starter-data-jpa-test
- spring-boot-starter-restclient-test
- spring-boot-starter-webmvc-test
- spring-boot-testcontainers
- spring-boot-docker-compose
- org.testcontainers:testcontainers-junit-jupiter
- org.testcontainers:testcontainers-mysql

---

## 4. PLUGINS

| Plugin | Version | Proposito |
|--------|---------|-----------|
| maven-enforcer-plugin | managed | Fuerza Java >= 17 |
| spring-javaformat-maven-plugin | 0.0.47 | Formato Spring style (fase validate) |
| maven-checkstyle-plugin | 3.6.0 | sun_checks.xml, failsOnError=false, failOnViolation=false |
| native-maven-plugin | managed | GraalVM native image |
| spring-boot-maven-plugin | managed | Fat jar + build-info para actuator |
| jacoco-maven-plugin | 0.8.14 | prepare-agent + report en fase test |
| git-commit-id-maven-plugin | managed | git.properties para actuator (failOnNoGitDirectory=false) |
| cyclonedx-maven-plugin | managed | SBOM CycloneDX |
| libsass-maven-plugin | 0.3.4 | SCSS→CSS solo en perfil 'css' |

### Propiedades en pom.xml
```
java.version=17
sonar.organization=pablop150
sonar.host.url=https://sonarcloud.io
project.build.outputTimestamp=2024-11-28T14:37:52Z
webjars-locator.version=1.1.2
webjars-bootstrap.version=5.3.8
webjars-font-awesome.version=4.7.0
checkstyle.version=12.1.2
jacoco.version=0.8.14
libsass.version=0.3.4
lifecycle-mapping=1.0.0
maven-checkstyle.version=3.6.0
nohttp-checkstyle.version=0.0.11
spring-format.version=0.0.47
```

### Perfiles Maven
- `css`: desempaqueta Bootstrap webjar → compila SCSS de `src/main/scss/` → `src/main/resources/static/resources/css/`
- `m2e`: integracion Eclipse (ignora checkstyle, build-info, javaformat en IDE)

---

## 5. CI/CD

### .github/workflows/build.yml (completo)
```yaml
name: SonarQube
on:
  push:
    branches:
      - main
      - delivery2
  pull_request:
    branches:
      - main
      - delivery2
    types: [opened, synchronize, reopened]
jobs:
  build:
    name: Build and analyze
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: 17
          distribution: 'zulu'
      - name: Cache SonarQube packages
        uses: actions/cache@v4
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar
      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2
      - name: Build and analyze
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=PabloP150_Pdds-ISA-Project -Dsonar.organization=pablop150 -Dsonar.branch.name=main
```

### .github/dco.yml
```yaml
require:
  members: false
```

---

## 6. ARQUITECTURA

### Estructura de paquetes
```
src/main/java/org/springframework/samples/petclinic/
├── PetClinicApplication.java
├── PetClinicRuntimeHints.java
├── model/
│   ├── BaseEntity.java        (abstract @MappedSuperclass)
│   ├── NamedEntity.java       (@MappedSuperclass, extiende BaseEntity)
│   ├── Person.java            (@MappedSuperclass, extiende BaseEntity)
│   └── package-info.java
├── owner/                     (paquete principal del dominio)
│   ├── Owner.java             (@Entity tabla owners)  ⚠ CC=22
│   ├── OwnerController.java   (@Controller)           ⚠ CC=21
│   ├── OwnerRepository.java   (JpaRepository<Owner,Integer>)
│   ├── Pet.java               (@Entity tabla pets)
│   ├── PetController.java     (@Controller)           ⚠ CC=27
│   ├── PetType.java           (@Entity tabla types)
│   ├── PetTypeRepository.java (JpaRepository<PetType,Integer>)
│   ├── PetValidator.java      (implements Validator)
│   ├── PetTypeFormatter.java  (@Component Formatter<PetType>)
│   ├── Visit.java             (@Entity tabla visits)
│   ├── VisitController.java   (@Controller)
│   └── package-info.java
├── vet/
│   ├── Vet.java               (@Entity tabla vets)
│   ├── VetController.java     (@Controller)
│   ├── VetRepository.java     (Repository<Vet,Integer> con @Cacheable)
│   ├── Specialty.java         (@Entity tabla specialties)
│   ├── Vets.java              (POJO wrapper @XmlRootElement)
│   └── package-info.java
└── system/
    ├── WelcomeController.java
    ├── CrashController.java
    ├── CacheConfiguration.java
    ├── WebConfiguration.java
    └── package-info.java
```

### Jerarquia JPA
```
BaseEntity  implements Serializable
  id: Integer  @Id @GeneratedValue(IDENTITY)
  isNew(): boolean
  ├── NamedEntity
  │     name: String  @Column @NotBlank
  │     ├── Pet       @Entity @Table(pets)
  │     ├── PetType   @Entity @Table(types)
  │     └── Specialty @Entity @Table(specialties)
  └── Person
        firstName: String  @Column @NotBlank
        lastName:  String  @Column @NotBlank
        ├── Owner  @Entity @Table(owners)
        │          address:   @NotBlank
        │          city:      @NotBlank
        │          telephone: @Pattern(\\d{10})
        │          pets: List<Pet>  @OneToMany CascadeALL Eager @OrderBy("name")
        └── Vet    @Entity @Table(vets)
                   specialties: Set<Specialty>  @ManyToMany Eager @JoinTable(vet_specialties)
```

### Arquitectura sin Service Layer
El proyecto SOLO tiene 2 capas: Controller → Repository.
Los Controllers inyectan Repositories directamente. No existe capa de servicios.
Esto es la principal deuda tecnica identificada en Delivery 2.

---

## 7. CODIGO FUENTE MAIN

### PetClinicApplication.java
```java
package org.springframework.samples.petclinic;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ImportRuntimeHints;

@SpringBootApplication
@ImportRuntimeHints(PetClinicRuntimeHints.class)
public class PetClinicApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetClinicApplication.class, args);
    }
}
```

### PetClinicRuntimeHints.java
```java
package org.springframework.samples.petclinic;

import org.springframework.aot.hint.RuntimeHints;
import org.springframework.aot.hint.RuntimeHintsRegistrar;
import org.springframework.samples.petclinic.model.BaseEntity;
import org.springframework.samples.petclinic.model.Person;
import org.springframework.samples.petclinic.vet.Vet;

public class PetClinicRuntimeHints implements RuntimeHintsRegistrar {
    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        hints.resources().registerPattern("db/*");
        hints.resources().registerPattern("messages/*");
        hints.resources().registerPattern("mysql-default-conf");
        hints.serialization().registerType(BaseEntity.class);
        hints.serialization().registerType(Person.class);
        hints.serialization().registerType(Vet.class);
    }
}
```

### model/BaseEntity.java
```java
package org.springframework.samples.petclinic.model;

import java.io.Serializable;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.MappedSuperclass;

@MappedSuperclass
public class BaseEntity implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    public Integer getId() { return id; }
    public void setId(Integer id) { this.id = id; }
    public boolean isNew() { return this.id == null; }
}
```

### model/NamedEntity.java
```java
package org.springframework.samples.petclinic.model;

import jakarta.persistence.Column;
import jakarta.persistence.MappedSuperclass;
import jakarta.validation.constraints.NotBlank;

@MappedSuperclass
public class NamedEntity extends BaseEntity {
    @Column
    @NotBlank
    private String name;

    public String getName() { return this.name; }
    public void setName(String name) { this.name = name; }

    @Override
    public String toString() {
        String name = this.getName();
        return (name != null) ? name : "<null>";
    }
}
```

### model/Person.java
```java
package org.springframework.samples.petclinic.model;

import jakarta.persistence.Column;
import jakarta.persistence.MappedSuperclass;
import jakarta.validation.constraints.NotBlank;

@MappedSuperclass
public class Person extends BaseEntity {
    @Column @NotBlank
    private String firstName;

    @Column @NotBlank
    private String lastName;

    public String getFirstName() { return this.firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    public String getLastName() { return this.lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
}
```

### owner/Owner.java  [HOTSPOT CC=22]
```java
package org.springframework.samples.petclinic.owner;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
import org.springframework.core.style.ToStringCreator;
import org.springframework.samples.petclinic.model.Person;
import org.springframework.util.Assert;
import jakarta.persistence.*;
import jakarta.validation.constraints.*;

@Entity
@Table(name = "owners")
public class Owner extends Person {
    @Column @NotBlank
    private String address;

    @Column @NotBlank
    private String city;

    @Column @NotBlank
    @Pattern(regexp = "\\d{10}", message = "{telephone.invalid}")
    private String telephone;

    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    @JoinColumn(name = "owner_id")
    @OrderBy("name")
    private final List<Pet> pets = new ArrayList<>();

    // getters/setters: address, city, telephone, pets

    public void addPet(Pet pet) {
        if (pet.isNew()) { getPets().add(pet); }
    }

    public Pet getPet(String name) { return getPet(name, false); }

    public Pet getPet(Integer id) {
        for (Pet pet : getPets()) {
            if (!pet.isNew() && Objects.equals(pet.getId(), id)) return pet;
        }
        return null;
    }

    public Pet getPet(String name, boolean ignoreNew) {
        for (Pet pet : getPets()) {
            String compName = pet.getName();
            if (compName != null && compName.equalsIgnoreCase(name)) {
                if (!ignoreNew || !pet.isNew()) return pet;
            }
        }
        return null;
    }

    public void addVisit(Integer petId, Visit visit) {
        Assert.notNull(petId, "Pet identifier must not be null!");
        Assert.notNull(visit, "Visit must not be null!");
        Pet pet = getPet(petId);
        Assert.notNull(pet, "Invalid Pet identifier!");
        pet.addVisit(visit);
    }
}
```

### owner/OwnerController.java  [HOTSPOT CC=21]
```java
package org.springframework.samples.petclinic.owner;

import org.springframework.data.domain.*;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.*;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;
import jakarta.validation.Valid;
import java.util.*;

@Controller
class OwnerController {
    private static final String VIEWS_OWNER_CREATE_OR_UPDATE_FORM = "owners/createOrUpdateOwnerForm";
    private final OwnerRepository owners;

    public OwnerController(OwnerRepository owners) { this.owners = owners; }

    @InitBinder
    public void setAllowedFields(WebDataBinder dataBinder) {
        dataBinder.setDisallowedFields("id");
    }

    @ModelAttribute("owner")
    public Owner findOwner(@PathVariable(name = "ownerId", required = false) Integer ownerId) {
        return ownerId == null ? new Owner()
            : this.owners.findById(ownerId)
                .orElseThrow(() -> new IllegalArgumentException("Owner not found with id: " + ownerId));
    }

    @GetMapping("/owners/new")
    public String initCreationForm() { return VIEWS_OWNER_CREATE_OR_UPDATE_FORM; }

    @PostMapping("/owners/new")
    public String processCreationForm(@Valid Owner owner, BindingResult result, RedirectAttributes redirectAttributes) {
        if (result.hasErrors()) {
            redirectAttributes.addFlashAttribute("error", "There was an error in creating the owner.");
            return VIEWS_OWNER_CREATE_OR_UPDATE_FORM;
        }
        this.owners.save(owner);
        redirectAttributes.addFlashAttribute("message", "New Owner Created");
        return "redirect:/owners/" + owner.getId();
    }

    @GetMapping("/owners/find")
    public String initFindForm() { return "owners/findOwners"; }

    @GetMapping("/owners")
    public String processFindForm(@RequestParam(defaultValue = "1") int page, Owner owner,
                                  BindingResult result, Model model) {
        String lastName = owner.getLastName();
        if (lastName == null) lastName = "";
        Page<Owner> ownersResults = findPaginatedForOwnersLastName(page, lastName);
        if (ownersResults.isEmpty()) {
            result.rejectValue("lastName", "notFound", "not found");
            return "owners/findOwners";
        }
        if (ownersResults.getTotalElements() == 1) {
            owner = ownersResults.iterator().next();
            return "redirect:/owners/" + owner.getId();
        }
        return addPaginationModel(page, model, ownersResults);
    }

    private String addPaginationModel(int page, Model model, Page<Owner> paginated) {
        model.addAttribute("currentPage", page);
        model.addAttribute("totalPages", paginated.getTotalPages());
        model.addAttribute("totalItems", paginated.getTotalElements());
        model.addAttribute("listOwners", paginated.getContent());
        return "owners/ownersList";
    }

    private Page<Owner> findPaginatedForOwnersLastName(int page, String lastname) {
        Pageable pageable = PageRequest.of(page - 1, 5);
        return owners.findByLastNameStartingWith(lastname, pageable);
    }

    @GetMapping("/owners/{ownerId}/edit")
    public String initUpdateOwnerForm() { return VIEWS_OWNER_CREATE_OR_UPDATE_FORM; }

    @PostMapping("/owners/{ownerId}/edit")
    public String processUpdateOwnerForm(@Valid Owner owner, BindingResult result,
                                         @PathVariable("ownerId") int ownerId,
                                         RedirectAttributes redirectAttributes) {
        if (result.hasErrors()) {
            redirectAttributes.addFlashAttribute("error", "There was an error in updating the owner.");
            return VIEWS_OWNER_CREATE_OR_UPDATE_FORM;
        }
        if (!Objects.equals(owner.getId(), ownerId)) {
            result.rejectValue("id", "mismatch", "The owner ID in the form does not match the URL.");
            redirectAttributes.addFlashAttribute("error", "Owner ID mismatch. Please try again.");
            return "redirect:/owners/{ownerId}/edit";
        }
        owner.setId(ownerId);
        this.owners.save(owner);
        redirectAttributes.addFlashAttribute("message", "Owner Values Updated");
        return "redirect:/owners/{ownerId}";
    }

    @GetMapping("/owners/{ownerId}")
    public ModelAndView showOwner(@PathVariable("ownerId") int ownerId) {
        ModelAndView mav = new ModelAndView("owners/ownerDetails");
        Owner owner = this.owners.findById(ownerId)
            .orElseThrow(() -> new IllegalArgumentException("Owner not found with id: " + ownerId));
        mav.addObject(owner);
        return mav;
    }
}
```

### owner/OwnerRepository.java
```java
package org.springframework.samples.petclinic.owner;

import java.util.Optional;
import org.springframework.data.domain.*;
import org.springframework.data.jpa.repository.JpaRepository;

public interface OwnerRepository extends JpaRepository<Owner, Integer> {
    Page<Owner> findByLastNameStartingWith(String lastName, Pageable pageable);
    Optional<Owner> findById(Integer id);
}
```

### owner/Pet.java
```java
package org.springframework.samples.petclinic.owner;

import java.time.LocalDate;
import java.util.*;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.samples.petclinic.model.NamedEntity;
import jakarta.persistence.*;

@Entity
@Table(name = "pets")
public class Pet extends NamedEntity {
    @Column
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private LocalDate birthDate;

    @ManyToOne
    @JoinColumn(name = "type_id")
    private PetType type;

    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    @JoinColumn(name = "pet_id")
    @OrderBy("date ASC")
    private final Set<Visit> visits = new LinkedHashSet<>();

    public void setBirthDate(LocalDate birthDate) { this.birthDate = birthDate; }
    public LocalDate getBirthDate() { return this.birthDate; }
    public PetType getType() { return this.type; }
    public void setType(PetType type) { this.type = type; }
    public Collection<Visit> getVisits() { return this.visits; }
    public void addVisit(Visit visit) { getVisits().add(visit); }
}
```

### owner/PetController.java  [HOTSPOT CC=27]
```java
package org.springframework.samples.petclinic.owner;

import java.time.LocalDate;
import java.util.*;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.util.*;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.*;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;
import jakarta.validation.Valid;

@Controller
@RequestMapping("/owners/{ownerId}")
class PetController {
    private static final String VIEWS_PETS_CREATE_OR_UPDATE_FORM = "pets/createOrUpdatePetForm";
    private final OwnerRepository owners;
    private final PetTypeRepository types;

    public PetController(OwnerRepository owners, PetTypeRepository types) {
        this.owners = owners; this.types = types;
    }

    @ModelAttribute("types")
    public Collection<PetType> populatePetTypes() { return this.types.findPetTypes(); }

    @ModelAttribute("owner")
    public Owner findOwner(@PathVariable("ownerId") int ownerId) {
        return this.owners.findById(ownerId)
            .orElseThrow(() -> new IllegalArgumentException("Owner not found with id: " + ownerId));
    }

    @ModelAttribute("pet")
    public Pet findPet(@PathVariable("ownerId") int ownerId,
                       @PathVariable(name = "petId", required = false) Integer petId) {
        if (petId == null) return new Pet();
        Owner owner = this.owners.findById(ownerId)
            .orElseThrow(() -> new IllegalArgumentException("Owner not found with id: " + ownerId));
        return owner.getPet(petId);
    }

    @InitBinder("owner")
    public void initOwnerBinder(WebDataBinder dataBinder) { dataBinder.setDisallowedFields("id"); }

    @InitBinder("pet")
    public void initPetBinder(WebDataBinder dataBinder) { dataBinder.setValidator(new PetValidator()); }

    @GetMapping("/pets/new")
    public String initCreationForm(Owner owner, ModelMap model) {
        owner.addPet(new Pet());
        return VIEWS_PETS_CREATE_OR_UPDATE_FORM;
    }

    @PostMapping("/pets/new")
    public String processCreationForm(Owner owner, @Valid Pet pet, BindingResult result,
                                      RedirectAttributes redirectAttributes) {
        if (StringUtils.hasText(pet.getName()) && pet.isNew() && owner.getPet(pet.getName(), true) != null)
            result.rejectValue("name", "duplicate", "already exists");
        if (pet.getBirthDate() != null && pet.getBirthDate().isAfter(LocalDate.now()))
            result.rejectValue("birthDate", "typeMismatch.birthDate");
        if (result.hasErrors()) return VIEWS_PETS_CREATE_OR_UPDATE_FORM;
        owner.addPet(pet);
        this.owners.save(owner);
        redirectAttributes.addFlashAttribute("message", "New Pet has been Added");
        return "redirect:/owners/{ownerId}";
    }

    @GetMapping("/pets/{petId}/edit")
    public String initUpdateForm() { return VIEWS_PETS_CREATE_OR_UPDATE_FORM; }

    @PostMapping("/pets/{petId}/edit")
    public String processUpdateForm(Owner owner, @Valid Pet pet, BindingResult result,
                                    RedirectAttributes redirectAttributes) {
        String petName = pet.getName();
        if (StringUtils.hasText(petName)) {
            Pet existingPet = owner.getPet(petName, false);
            if (existingPet != null && !Objects.equals(existingPet.getId(), pet.getId()))
                result.rejectValue("name", "duplicate", "already exists");
        }
        if (pet.getBirthDate() != null && pet.getBirthDate().isAfter(LocalDate.now()))
            result.rejectValue("birthDate", "typeMismatch.birthDate");
        if (result.hasErrors()) return VIEWS_PETS_CREATE_OR_UPDATE_FORM;
        updatePetDetails(owner, pet);
        redirectAttributes.addFlashAttribute("message", "Pet details has been edited");
        return "redirect:/owners/{ownerId}";
    }

    private void updatePetDetails(Owner owner, Pet pet) {
        Integer id = pet.getId();
        Assert.state(id != null, "'pet.getId()' must not be null");
        Pet existingPet = owner.getPet(id);
        if (existingPet != null) {
            existingPet.setName(pet.getName());
            existingPet.setBirthDate(pet.getBirthDate());
            existingPet.setType(pet.getType());
        } else { owner.addPet(pet); }
        this.owners.save(owner);
    }
}
```

### owner/PetType.java
```java
package org.springframework.samples.petclinic.owner;

import org.springframework.samples.petclinic.model.NamedEntity;
import jakarta.persistence.*;

@Entity
@Table(name = "types")
public class PetType extends NamedEntity {}
```

### owner/PetTypeRepository.java
```java
package org.springframework.samples.petclinic.owner;

import java.util.List;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

public interface PetTypeRepository extends JpaRepository<PetType, Integer> {
    @Query("SELECT ptype FROM PetType ptype ORDER BY ptype.name")
    List<PetType> findPetTypes();
}
```

### owner/PetValidator.java
```java
package org.springframework.samples.petclinic.owner;

import org.springframework.util.StringUtils;
import org.springframework.validation.*;

public class PetValidator implements Validator {
    private static final String REQUIRED = "required";

    @Override
    public void validate(Object obj, Errors errors) {
        Pet pet = (Pet) obj;
        // name validation
        if (!StringUtils.hasText(pet.getName()))
            errors.rejectValue("name", REQUIRED, REQUIRED);
        // type validation (only for new pets)
        if (pet.isNew() && pet.getType() == null)
            errors.rejectValue("type", REQUIRED, REQUIRED);
        // birth date validation
        if (pet.getBirthDate() == null)
            errors.rejectValue("birthDate", REQUIRED, REQUIRED);
    }

    @Override
    public boolean supports(Class<?> clazz) { return Pet.class.isAssignableFrom(clazz); }
}
```

### owner/PetTypeFormatter.java
```java
package org.springframework.samples.petclinic.owner;

import org.springframework.format.Formatter;
import org.springframework.stereotype.Component;
import java.text.ParseException;
import java.util.*;

@Component
public class PetTypeFormatter implements Formatter<PetType> {
    private final PetTypeRepository types;

    public PetTypeFormatter(PetTypeRepository types) { this.types = types; }

    @Override
    public String print(PetType petType, Locale locale) {
        String name = petType.getName();
        return (name != null) ? name : "<null>";
    }

    @Override
    public PetType parse(String text, Locale locale) throws ParseException {
        for (PetType type : this.types.findPetTypes()) {
            if (Objects.equals(type.getName(), text)) return type;
        }
        throw new ParseException("type not found: " + text, 0);
    }
}
```

### owner/Visit.java
```java
package org.springframework.samples.petclinic.owner;

import java.time.LocalDate;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.samples.petclinic.model.BaseEntity;
import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;

@Entity
@Table(name = "visits")
public class Visit extends BaseEntity {
    @Column(name = "visit_date")
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private LocalDate date;

    @NotBlank
    private String description;

    public Visit() { this.date = LocalDate.now(); }  // default: today

    public LocalDate getDate() { return this.date; }
    public void setDate(LocalDate date) { this.date = date; }
    public String getDescription() { return this.description; }
    public void setDescription(String description) { this.description = description; }
}
```

### owner/VisitController.java
```java
package org.springframework.samples.petclinic.owner;

import java.util.*;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.*;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;
import jakarta.validation.Valid;

@Controller
class VisitController {
    private final OwnerRepository owners;

    public VisitController(OwnerRepository owners) { this.owners = owners; }

    @InitBinder
    public void setAllowedFields(WebDataBinder dataBinder) { dataBinder.setDisallowedFields("id"); }

    @ModelAttribute("visit")
    public Visit loadPetWithVisit(@PathVariable("ownerId") int ownerId,
                                  @PathVariable("petId") int petId,
                                  Map<String, Object> model) {
        Owner owner = owners.findById(ownerId)
            .orElseThrow(() -> new IllegalArgumentException("Owner not found with id: " + ownerId));
        Pet pet = owner.getPet(petId);
        if (pet == null)
            throw new IllegalArgumentException("Pet with id " + petId + " not found for owner " + ownerId);
        model.put("pet", pet);
        model.put("owner", owner);
        Visit visit = new Visit();
        pet.addVisit(visit);
        return visit;
    }

    @GetMapping("/owners/{ownerId}/pets/{petId}/visits/new")
    public String initNewVisitForm() { return "pets/createOrUpdateVisitForm"; }

    @PostMapping("/owners/{ownerId}/pets/{petId}/visits/new")
    public String processNewVisitForm(@ModelAttribute Owner owner, @PathVariable int petId,
                                      @Valid Visit visit, BindingResult result,
                                      RedirectAttributes redirectAttributes) {
        if (result.hasErrors()) return "pets/createOrUpdateVisitForm";
        owner.addVisit(petId, visit);
        this.owners.save(owner);
        redirectAttributes.addFlashAttribute("message", "Your visit has been booked");
        return "redirect:/owners/{ownerId}";
    }
}
```

### vet/Vet.java
```java
package org.springframework.samples.petclinic.vet;

import java.util.*;
import java.util.stream.Collectors;
import org.springframework.samples.petclinic.model.NamedEntity;
import org.springframework.samples.petclinic.model.Person;
import jakarta.persistence.*;
import jakarta.xml.bind.annotation.XmlElement;

@Entity
@Table(name = "vets")
public class Vet extends Person {
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "vet_specialties",
        joinColumns = @JoinColumn(name = "vet_id"),
        inverseJoinColumns = @JoinColumn(name = "specialty_id"))
    private Set<Specialty> specialties;

    protected Set<Specialty> getSpecialtiesInternal() {
        if (this.specialties == null) this.specialties = new HashSet<>();
        return this.specialties;
    }

    @XmlElement
    public List<Specialty> getSpecialties() {
        return getSpecialtiesInternal().stream()
            .sorted(Comparator.comparing(NamedEntity::getName))
            .collect(Collectors.toList());
    }

    public int getNrOfSpecialties() { return getSpecialtiesInternal().size(); }
    public void addSpecialty(Specialty specialty) { getSpecialtiesInternal().add(specialty); }
}
```

### vet/VetController.java
```java
package org.springframework.samples.petclinic.vet;

import java.util.List;
import org.springframework.data.domain.*;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
class VetController {
    private final VetRepository vetRepository;

    public VetController(VetRepository vetRepository) { this.vetRepository = vetRepository; }

    @GetMapping("/vets.html")
    public String showVetList(@RequestParam(defaultValue = "1") int page, Model model) {
        Vets vets = new Vets();
        Page<Vet> paginated = findPaginated(page);
        vets.getVetList().addAll(paginated.toList());
        return addPaginationModel(page, paginated, model);
    }

    private String addPaginationModel(int page, Page<Vet> paginated, Model model) {
        model.addAttribute("currentPage", page);
        model.addAttribute("totalPages", paginated.getTotalPages());
        model.addAttribute("totalItems", paginated.getTotalElements());
        model.addAttribute("listVets", paginated.getContent());
        return "vets/vetList";
    }

    private Page<Vet> findPaginated(int page) {
        return vetRepository.findAll(PageRequest.of(page - 1, 5));
    }

    @GetMapping({"/vets"})
    public @ResponseBody Vets showResourcesVetList() {
        Vets vets = new Vets();
        vets.getVetList().addAll(this.vetRepository.findAll());
        return vets;
    }
}
```

### vet/VetRepository.java
```java
package org.springframework.samples.petclinic.vet;

import org.springframework.cache.annotation.Cacheable;
import org.springframework.dao.DataAccessException;
import org.springframework.data.domain.*;
import org.springframework.data.repository.Repository;
import org.springframework.transaction.annotation.Transactional;
import java.util.Collection;

public interface VetRepository extends Repository<Vet, Integer> {
    @Transactional(readOnly = true)
    @Cacheable("vets")
    Collection<Vet> findAll() throws DataAccessException;

    @Transactional(readOnly = true)
    @Cacheable("vets")
    Page<Vet> findAll(Pageable pageable) throws DataAccessException;
}
```

### vet/Specialty.java
```java
package org.springframework.samples.petclinic.vet;

import org.springframework.samples.petclinic.model.NamedEntity;
import jakarta.persistence.*;

@Entity
@Table(name = "specialties")
public class Specialty extends NamedEntity {}
```

### vet/Vets.java
```java
package org.springframework.samples.petclinic.vet;

import java.util.*;
import jakarta.xml.bind.annotation.*;

@XmlRootElement
public class Vets {
    private List<Vet> vets;

    @XmlElement
    public List<Vet> getVetList() {
        if (vets == null) vets = new ArrayList<>();
        return vets;
    }
}
```

### system/WelcomeController.java
```java
package org.springframework.samples.petclinic.system;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
class WelcomeController {
    @GetMapping("/")
    public String welcome() { return "welcome"; }
}
```

### system/CrashController.java
```java
package org.springframework.samples.petclinic.system;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
class CrashController {
    @GetMapping("/oups")
    public String triggerException() {
        throw new RuntimeException(
            "Expected: controller used to showcase what happens when an exception is thrown");
    }
}
```

### system/CacheConfiguration.java
```java
package org.springframework.samples.petclinic.system;

import org.springframework.boot.cache.autoconfigure.JCacheManagerCustomizer;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.*;
import javax.cache.configuration.MutableConfiguration;

@Configuration(proxyBeanMethods = false)
@EnableCaching
class CacheConfiguration {
    @Bean
    public JCacheManagerCustomizer petclinicCacheConfigurationCustomizer() {
        return cm -> cm.createCache("vets", cacheConfiguration());
    }

    private javax.cache.configuration.Configuration<Object, Object> cacheConfiguration() {
        return new MutableConfiguration<>().setStatisticsEnabled(true);
    }
}
```

### system/WebConfiguration.java
```java
package org.springframework.samples.petclinic.system;

import org.springframework.context.annotation.*;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.*;
import org.springframework.web.servlet.i18n.*;
import java.util.Locale;

@Configuration
@SuppressWarnings("unused")
public class WebConfiguration implements WebMvcConfigurer {
    @Bean
    public LocaleResolver localeResolver() {
        SessionLocaleResolver resolver = new SessionLocaleResolver();
        resolver.setDefaultLocale(Locale.ENGLISH);
        return resolver;
    }

    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor interceptor = new LocaleChangeInterceptor();
        interceptor.setParamName("lang");
        return interceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }
}
```

---

## 8. CODIGO FUENTE TESTS

### Estructura de tests
```
src/test/java/org/springframework/samples/petclinic/
├── MySqlIntegrationTests.java          @SpringBootTest mysql + Testcontainers
├── MysqlTestApplication.java           Config para arranque manual con MySQL
├── PetClinicIntegrationTests.java      @SpringBootTest basico (H2)
├── PostgresIntegrationTests.java       @SpringBootTest postgres + Docker Compose
├── model/
│   └── ValidatorTests.java             Unit test validacion Jakarta
├── owner/
│   ├── OwnerControllerTests.java       @WebMvcTest 13 tests
│   ├── PetControllerTests.java         @WebMvcTest 9 tests + @Nested
│   ├── PetTypeFormatterTests.java      Unit 3 tests
│   ├── PetValidatorTests.java          Unit 4 tests + @Nested
│   └── VisitControllerTests.java       @WebMvcTest 3 tests
├── service/
│   ├── ClinicServiceTests.java         @DataJpaTest 10 tests
│   └── EntityUtils.java               Utilidad para buscar entidades por ID
├── system/
│   ├── CrashControllerIntegrationTests.java  @SpringBootTest 2 tests
│   ├── CrashControllerTests.java             Unit 1 test
│   └── I18nPropertiesSyncTest.java           2 tests i18n sync
└── vet/
    ├── VetControllerTests.java         @WebMvcTest 2 tests
    └── VetTests.java                   Unit 1 test serializacion
```

### MySqlIntegrationTests.java
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@ActiveProfiles("mysql")
@Testcontainers(disabledWithoutDocker = true)
@DisabledInNativeImage
@DisabledInAotMode
class MySqlIntegrationTests {
    @ServiceConnection @Container
    static MySQLContainer container = new MySQLContainer(DockerImageName.parse("mysql:9.5"));
    @LocalServerPort int port;
    @Autowired VetRepository vets;
    @Autowired RestTemplateBuilder builder;

    @Test void testFindAll() { vets.findAll(); vets.findAll(); /* served from cache */ }
    @Test void testOwnerDetails() {
        RestTemplate template = builder.rootUri("http://localhost:" + port).build();
        ResponseEntity<String> result = template.exchange(RequestEntity.get("/owners/1").build(), String.class);
        assertThat(result.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
}
```

### PetClinicIntegrationTests.java
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class PetClinicIntegrationTests {
    @LocalServerPort int port;
    @Autowired VetRepository vets;
    @Autowired RestTemplateBuilder builder;

    @Test void testFindAll() { vets.findAll(); vets.findAll(); }
    @Test void testOwnerDetails() { /* HTTP GET /owners/1 → 200 OK */ }

    public static void main(String[] args) {
        SpringApplication.run(PetClinicApplication.class, "--spring.docker.compose.lifecycle-management=NONE");
    }
}
```

### PostgresIntegrationTests.java (resumen)
```java
@SpringBootTest(properties = {
    "spring.docker.compose.skip.in-tests=false",
    "spring.docker.compose.start.arguments=--force-recreate,--renew-anon-volumes,postgres"})
@ActiveProfiles("postgres")
@DisabledInNativeImage
public class PostgresIntegrationTests {
    @BeforeAll static void available() { assumeTrue(DockerClientFactory.instance().isDockerAvailable()); }
    // Tests: testFindAll(), testOwnerDetails()
    // Inner class PropertiesLogger: ApplicationListener que loguea properties del environment
}
```

### ClinicServiceTests.java — tests de repository (resumen)
```
@DataJpaTest  @AutoConfigureTestDatabase(replace=Replace.NONE)
Tests con datos reales de H2:
- shouldFindOwnersByLastName()        → findByLastNameStartingWith("Davis") → 2 resultados
- shouldFindSingleOwnerWithPet()      → findById(1) → Franklin, 1 pet, tipo "cat"
- shouldInsertOwner()                 → save nuevo owner, verifica count +1
- shouldUpdateOwner()                 → update lastName, verifica en BD
- shouldFindAllPetTypes()             → tipos[1]=cat, tipos[4]=snake
- shouldInsertPetIntoDatabaseAndGenerateId() → addPet("bowser"), verifica ID generado
- shouldUpdatePetName()               → update pet name, verifica en BD
- shouldFindVets()                    → vet[3]=Douglas, 2 specialties: dentistry+surgery
- shouldAddNewVisitForPet()           → addVisit, verifica count +1 y ID generado
- shouldFindVisitsByPetId()           → pet7 tiene 2 visitas con fecha no nula
```

### OwnerControllerTests.java — 13 tests @WebMvcTest
```
testInitCreationForm()                GET /owners/new → 200 + view createOrUpdateOwnerForm
testProcessCreationFormSuccess()      POST /owners/new valid → 3xx redirect
testProcessCreationFormHasErrors()    POST /owners/new sin address/phone → 200 + errores en address,telephone
testInitFindForm()                    GET /owners/find → 200 + view findOwners
testProcessFindFormSuccess()          GET /owners?page=1 → 200 ownersList (multiple results)
testProcessFindFormByLastName()       GET /owners?lastName=Franklin → redirect /owners/1
testProcessFindFormNoOwnersFound()    GET /owners?lastName=Unknown → 200 + error notFound en lastName
testInitUpdateOwnerForm()             GET /owners/1/edit → 200 + propiedades del owner
testProcessUpdateOwnerFormSuccess()   POST /owners/1/edit valid → redirect /owners/{ownerId}
testProcessUpdateOwnerFormUnchangedSuccess() POST /owners/1/edit sin params → redirect
testProcessUpdateOwnerFormHasErrors() POST /owners/1/edit sin address/phone → errores
testShowOwner()                       GET /owners/1 → 200 + datos Franklin + pets con visitas
testProcessUpdateOwnerFormWithIdMismatch() POST con ID mismatch → redirect + flash error
```

### PetControllerTests.java — @WebMvcTest con @Nested
```
testInitCreationForm()                GET /owners/1/pets/new → 200
testProcessCreationFormSuccess()      POST /owners/1/pets/new {Betty,hamster,2015-02-12} → redirect
Nested: ProcessCreationFormHasErrors:
  testProcessCreationFormWithBlankName()    → error name=required
  testProcessCreationFormWithDuplicateName() → error name=duplicate
  testProcessCreationFormWithMissingPetType() → error type=required
  testProcessCreationFormWithInvalidBirthDate() → error birthDate=typeMismatch.birthDate
  testInitUpdateForm()               GET /owners/1/pets/1/edit → 200
testProcessUpdateFormSuccess()       POST /owners/1/pets/1/edit valid → redirect
Nested: ProcessUpdateFormHasErrors:
  testProcessUpdateFormWithInvalidBirthDate() → error birthDate=typeMismatch
  testProcessUpdateFormWithBlankName() → error name=required
```

### I18nPropertiesSyncTest.java — tests criticos de i18n
```
checkNonInternationalizedStrings()  Verifica que no haya strings hardcodeados en HTML/Java
                                    (sin th:text y sin #{key})
checkI18nPropertyFilesAreInSync()   Verifica que todos los archivos messages_XX.properties
                                    tengan las mismas keys que messages.properties
                                    (excepto messages_en.properties que usa fallback)
```

---

## 9. BASE DE DATOS

### H2 — schema.sql (completo)
```sql
CREATE TABLE IF NOT EXISTS vets (
  id         INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  first_name VARCHAR(30),
  last_name  VARCHAR(30),
  INDEX(last_name)
);

CREATE TABLE IF NOT EXISTS specialties (
  id   INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name VARCHAR(80),
  INDEX(name)
);

CREATE TABLE IF NOT EXISTS vet_specialties (
  vet_id       INTEGER NOT NULL REFERENCES vets(id),
  specialty_id INTEGER NOT NULL REFERENCES specialties(id),
  UNIQUE (vet_id, specialty_id)  -- implicitamente via primary key dual
);

CREATE TABLE IF NOT EXISTS types (
  id   INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name VARCHAR(80),
  INDEX(name)
);

CREATE TABLE IF NOT EXISTS owners (
  id         INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  first_name VARCHAR(30),
  last_name  VARCHAR_IGNORECASE(30),
  address    VARCHAR(255),
  city       VARCHAR(80),
  telephone  VARCHAR(20),
  INDEX(last_name)
);

CREATE TABLE IF NOT EXISTS pets (
  id         INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name       VARCHAR(30),
  birth_date DATE,
  type_id    INTEGER NOT NULL REFERENCES types(id),
  owner_id   INTEGER NOT NULL REFERENCES owners(id),
  INDEX(name)
);

CREATE TABLE IF NOT EXISTS visits (
  id          INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  pet_id      INTEGER NOT NULL REFERENCES pets(id),
  visit_date  DATE,
  description VARCHAR(255),
  INDEX(pet_id)
);
```

### H2 — data.sql (datos de ejemplo)
```
Vets (6): James Carter, Helen Leary, Linda Douglas, Rafael Ortega, Henry Stevens, Sharon Jenkins
Specialties (3): radiology(1), surgery(2), dentistry(3)
Vet-Specialties: Helen→radiology, Linda→surgery+dentistry, Rafael→surgery, Henry→radiology
Types (6): cat(1), dog(2), lizard(3), snake(4), bird(5), hamster(6)
Owners (10): George Franklin, Betty Davis, Eduardo Rodriguez, Harold Davis, Peter McTavish,
             Jean Coleman, Jeff Black, Maria Escobito, David Schroeder, Carlos Estaban
Pets (13): Leo(cat,Franklin), Basil(hamster,Davis), Rosy(dog,Rodriguez), Jewel(dog,Rodriguez),
           Iggy(lizard,Davis2), George(snake,McTavish), Samantha(cat,Coleman),
           Max(cat,Coleman), Lucky(bird,Black), Mulligan(dog,Escobito),
           Freddy(bird,Schroeder), Lucky(dog,Estaban), Sly(cat,Estaban)
Visits (4): Leo(2013-01-01,"rabies shot"), Basil(2013-01-02,"rabies shot"),
            Samantha(2013-01-02,"neutered"), Max(2013-01-03,"spayed")
```

### MySQL — schema.sql (diferencias vs H2)
- Usa AUTO_INCREMENT en lugar de GENERATED BY DEFAULT AS IDENTITY
- IDs: `INT(4) UNSIGNED NOT NULL AUTO_INCREMENT`
- Engine: `InnoDB DEFAULT CHARSET=utf8`
- Constraint UNIQUE en vet_specialties: `UNIQUE KEY (vet_id, specialty_id)`

### MySQL — user.sql
```sql
CREATE DATABASE IF NOT EXISTS petclinic CHARACTER SET utf8 COLLATE utf8_unicode_ci;
CREATE USER IF NOT EXISTS 'petclinic'@'%' IDENTIFIED BY 'petclinic';
GRANT ALL PRIVILEGES ON petclinic.* TO 'petclinic'@'%';
FLUSH PRIVILEGES;
```

### PostgreSQL — schema.sql (diferencias)
- Usa `TEXT` en lugar de `VARCHAR` para la mayoria de campos
- Usa `GENERATED BY DEFAULT AS IDENTITY` (igual que H2)
- Crea indices con `CREATE INDEX IF NOT EXISTS`

### PostgreSQL — data.sql
- Usa `INSERT INTO ... WHERE NOT EXISTS (SELECT ...)` para ser idempotente
- Usa `ON CONFLICT (vet_id, specialty_id) DO NOTHING` para vet_specialties
- Mismos datos que MySQL pero con fechas de mascotas entre 1995-2002

---

## 10. CONFIGURACION PROPERTIES

### application.properties (default — H2)
```properties
database=h2
spring.sql.init.schema-locations=classpath*:db/${database}/schema.sql
spring.sql.init.data-locations=classpath*:db/${database}/data.sql
spring.thymeleaf.mode=HTML
spring.jpa.hibernate.ddl-auto=none
spring.jpa.open-in-view=false
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategySnakeCaseImpl
spring.messages.basename=messages/messages
management.endpoints.web.exposure.include=*
logging.level.org.springframework=INFO
spring.web.resources.cache.cachecontrol.max-age=12h
```

### application-mysql.properties
```properties
database=mysql
spring.datasource.url=${MYSQL_URL:jdbc:mysql://localhost/petclinic}
spring.datasource.username=${MYSQL_USER:petclinic}
spring.datasource.password=${MYSQL_PASS:petclinic}
spring.sql.init.mode=always
```

### application-postgres.properties
```properties
database=postgres
spring.datasource.url=${POSTGRES_URL:jdbc:postgresql://localhost/petclinic}
spring.datasource.username=${POSTGRES_USER:petclinic}
spring.datasource.password=${POSTGRES_PASS:petclinic}
spring.sql.init.mode=always
```

---

## 11. TEMPLATES THYMELEAF

Ubicacion: `src/main/resources/templates/`

### Estructura
```
fragments/
├── layout.html     Master layout: navbar Bootstrap, menu (Home/FindOwners/Vets/Error), CDN links
├── inputField.html Fragment: input text/date + validacion con Font Awesome icons
└── selectField.html Fragment: select dropdown + validacion
welcome.html        Pagina inicio: imagen pets.png
error.html          Manejo de errores: 404, 500 y generico — mensaje de excepcion visible
owners/
├── findOwners.html        Formulario busqueda por apellido + link "Add Owner"
├── ownersList.html        Tabla paginada (firstName, lastName, address, city, telephone, pets)
│                          Paginacion: first/prev/next/last buttons
├── ownerDetails.html      Detalle: datos owner, pets y visitas por mascota
│                          Flash messages: success (verde) y error (rojo), auto-hide 3s JS
│                          Botones: Edit Owner, Add New Pet
└── createOrUpdateOwnerForm.html  Formulario: firstName, lastName, address, city, telephone
pets/
├── createOrUpdatePetForm.html    Formulario: nombre owner (readonly), name, birthDate, type(select)
└── createOrUpdateVisitForm.html  Formulario: tabla de visitas previas, date, description
vets/
└── vetList.html    Tabla paginada: Name, Specialties ("none" si no tiene)
```

### Nota importante sobre ownerDetails.html
- Flash message de exito: `th:if="${message}"` con auto-dismiss via JS
- Flash message de error: `th:if="${error}"` con auto-dismiss via JS
- Visitas mostradas por mascota en tabla anidada

---

## 12. I18N

### Claves de messages.properties (51 claves)
```
welcome, owner, ownerFirstName, ownerLastName, ownerAddress, ownerCity, ownerTelephone
pet, petName, petBirthDate, petType
date, description
addOwner, findOwner, findOwners, updateOwner
addPet, editPet, updatePet
addVisit, editVisit
vets, vet, vetSpecialties
homes, error, pages
required, notFound, duplicate, nonNumeric
duplicateFormSubmission, telephone.invalid
error.404, error.500, error.general
```

### Idiomas soportados
- messages.properties (default/fallback)
- messages_en.properties (VACIO — usa fallback)
- messages_es.properties (espanol completo)
- messages_de.properties (aleman completo)
- messages_pt.properties (portugues)
- messages_ru.properties (ruso)
- messages_ko.properties (coreano)
- messages_fa.properties (farsi)
- messages_tr.properties (turco)

### Mecanismo i18n (WebConfiguration.java)
- `SessionLocaleResolver` — default: Locale.ENGLISH, almacena en sesion HTTP
- `LocaleChangeInterceptor` — parametro URL: `?lang=es`, `?lang=de`, etc.

---

## 13. DEVOPS

### docker-compose.yml (completo)
```yaml
services:
  mysql:
    image: mysql:9.5
    ports: ["3306:3306"]
    environment:
      MYSQL_ROOT_PASSWORD: ""
      MYSQL_ALLOW_EMPTY_PASSWORD: "true"
      MYSQL_USER: petclinic
      MYSQL_PASSWORD: petclinic
      MYSQL_DATABASE: petclinic
    volumes: ["./conf.d:/etc/mysql/conf.d:ro"]

  postgres:
    image: postgres:18.1
    ports: ["5432:5432"]
    environment:
      POSTGRES_PASSWORD: petclinic
      POSTGRES_USER: petclinic
      POSTGRES_DB: petclinic
```

### k8s/petclinic.yml
```yaml
# Service: NodePort 80 → containerPort 8080, selector app=petclinic
# Deployment: imagen dsyer/petclinic, 1 replica
#   env: SPRING_PROFILES_ACTIVE=postgres, SERVICE_BINDING_ROOT=/bindings
#   livenessProbe:  GET /livez  puerto 8080
#   readinessProbe: GET /readyz puerto 8080
#   volumeMount: /bindings/secret (readonly) desde secret demo-db
```

### k8s/db.yml
```yaml
# Secret: servicebinding.io/postgresql
#   type: postgresql, provider: postgresql
#   host: demo-db, port: 5432, database: petclinic, username: user, password: pass
# Service: ClusterIP puerto 5432, selector app=demo-db
# Deployment: postgres:18.1, env de secret, TCP probes en 5432
```

### .devcontainer/devcontainer.json
```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "java": {"version": "21", "installMaven": false, "installGradle": false, "distribution": "Oracle"},
    "azure-cli": {},
    "docker-in-docker": {},
    "github-cli": {}
  },
  "customizations": {
    "vscode": {
      "extensions": ["redhat.vscode-xml", "visualstudioexptteam.vscodeintellicode", "vscjava.vscode-java-pack"]
    }
  },
  "remoteUser": "vscode"
}
```
**Nota:** devcontainer usa Java 21 Oracle, pero el proyecto requiere Java 17. Java 21 es retrocompatible.

### .devcontainer/Dockerfile
```
Base: mcr.microsoft.com/vscode/devcontainers/java:0-17-bullseye
Java: 17.0.7-ms via SDKMAN
Optional: Node.js via NVM
Volumes: /home/vscode/.m2, /home/vscode/.gradle
```

### .gitpod.yml
```yaml
image:
  file: ./.devcontainer/Dockerfile
tasks:
  - before: sudo usermod -a -G sdkman gitpod && sudo usermod -a -G nvm gitpod && sudo chown -R gitpod /usr/local/sdkman /usr/local/share/nvm
  - init: ./mvnw install
vscode:
  extensions:
  - vscjava.vscode-java-pack
  - redhat.vscode-xml
```

---

## 14. DEUDA TECNICA

### Hotspots por Complejidad Ciclomatica (SonarCloud)

| Clase | CC | Problema principal |
|-------|----|--------------------|
| PetController.java | 27 CRITICO | Fat Controller |
| Owner.java | 22 ALTO | Entidad con logica de negocio |
| OwnerController.java | 21 ALTO | Fat Controller |

### PetController (CC=27) — causas
- 3 metodos @ModelAttribute con busqueda de Owner duplicada
- Validacion de nombre duplicado: `owner.getPet(name, true)`
- Validacion de fecha futura: `pet.getBirthDate().isAfter(LocalDate.now())`
- Metodo privado `updatePetDetails` con logica de busqueda + actualizacion
- Dos @InitBinder diferentes (owner e pet)
- Todos los flujos de creacion Y edicion en un solo controller

### Owner (CC=22) — causas
- `getPet(String name)` — busqueda lineal por nombre
- `getPet(String name, boolean ignoreNew)` — variante con flag
- `getPet(Integer id)` — busqueda lineal por ID
- `addVisit(Integer petId, Visit visit)` — logica de negocio en entidad
- Cascada CascadeType.ALL en relacion con pets

### OwnerController (CC=21) — causas
- `processFindForm` maneja 3 casos: vacio, 1 resultado, multiples
- `processCreationForm` con validacion y redirect
- `processUpdateOwnerForm` con validacion de ID mismatch
- `@ModelAttribute findOwner` con logica condicional
- Acceso directo a OwnerRepository sin service layer

### Plan Strangler Fig Pattern (3 fases)
- **Fase 1 — Interception:** Crear `OwnerService`, `PetService`
- **Fase 2 — Strangling:** Migrar metodos gradualmente, CI + JaCoCo validan
- **Fase 3 — Elimination:** Eliminar logica duplicada, SonarCloud confirma mejora

---

## 15. HISTORIAL GIT

### Commits Delivery 2 (todos el 16-feb-2026, rama delivery2)

| Hash | Mensaje | Archivos clave |
|------|---------|----------------|
| eccd8ab | docs: add comprehensive delivery 2 report | Documentation_Delivery2.pdf (1.9MB) |
| a11e16a | fix: route delivery2 analysis to main dashboard | build.yml (sonar.branch.name=main) |
| 28cb8da | feat: final sonar command with organization key and clean workflows | Elimina 4 workflows, pom.xml optimizado |
| aac4c0a | fix: force disable all checkstyle failures to allow sonar sync | pom.xml: failsOnError=false |
| 88230e8 | change to permite failsOnError | pom.xml ajuste |
| 368fdec | feat: complete SonarQube integration with pom.xml and build workflow | Crea build.yml, pom.xml con sonar |
| 7c76352 | feat: Add Governance Workflow | governance config |
| 967abf5 | feat: Delivery 2 Governance Pipeline and Quality Gates | pom.xml base (+43/-5) |

### Ancestro comun con main: a4fcf04 (2026-02-07, "Fix main method for docker compose")
### Remotes:
- origin → https://github.com/PabloP150/pdds-IS-spring-petclinic-project.git
- destino → https://github.com/PabloP150/Pdds-ISA-Project.git

---

## 16. ARCHIVOS RAIZ

| Archivo | Proposito |
|---------|-----------|
| pom.xml | Build Maven principal (fuente de verdad) |
| build.gradle | Build Gradle alternativo |
| settings.gradle | `rootProject.name = 'spring-petclinic'` |
| docker-compose.yml | MySQL 9.5 + PostgreSQL 18.1 para dev local |
| README.md | Documentacion oficial Spring PetClinic |
| Documentation_Delivery2.pdf | Reporte Delivery 2 (Governance + Tech Debt) |
| Analisis_Completo_SpringPetClinic.pdf | Analisis generado por Claude |
| PROYECTO_ANALISIS.md | Este archivo |
| LICENSE.txt | Apache License 2.0 |
| .editorconfig | utf-8, LF, tabs para java/xml, spaces para html/sql/gradle |
| .gitignore | target/, build/, .idea/, *.css (excepto petclinic.css) |
| .gitattributes | LF para .java, CRLF para .cmd/.bat |
| .gitpod.yml | init: ./mvnw install |
| mvnw / mvnw.cmd | Maven Wrapper 3.9.12 |
| gradlew / gradlew.bat | Gradle Wrapper 9.2.1 |
| .github/workflows/build.yml | CI/CD SonarCloud unificado |
| .github/dco.yml | DCO: require.members=false |
| .devcontainer/ | VSCode Dev Container (Java 21 Oracle) |
| k8s/ | petclinic.yml + db.yml Kubernetes |
| src/checkstyle/ | nohttp-checkstyle.xml + suppressions.xml |
| src/main/scss/ | Fuentes SCSS → compilar con `./mvnw package -P css` |

### build.gradle (completo — config Gradle)
```gradle
plugins {
  id 'java'
  id 'checkstyle'
  id 'org.springframework.boot' version '4.0.1'
  id 'io.spring.dependency-management' version '1.1.7'
  id 'org.graalvm.buildtools.native' version '0.11.3'
  id 'org.cyclonedx.bom' version '3.0.2'
  id 'io.spring.javaformat' version '0.0.47'
  id "io.spring.nohttp" version "0.0.11"
}

gradle.startParameter.excludedTaskNames += ["checkFormatAot", "checkFormatAotTest"]
group = 'org.springframework.samples'
version = '4.0.0-SNAPSHOT'
java { toolchain { languageVersion = JavaLanguageVersion.of(17) } }

ext {
  checkstyleVersion = "12.1.2"
  springJavaformatCheckstyleVersion = "0.0.47"
  webjarsLocatorLiteVersion = "1.1.2"
  webjarsFontawesomeVersion = "4.7.0"
  webjarsBootstrapVersion = "5.3.8"
}

checkstyle {
  configDirectory = project.file('src/checkstyle')
  configFile = file('src/checkstyle/nohttp-checkstyle.xml')
}

// formatMain y formatTest dependen de checkstyle
// checkstyleAot, checkstyleAotTest, formatAot, formatAotTest → disabled
```

---

## RESUMEN DE CONTEOS

| Metrica | Valor |
|---------|-------|
| Clases Java main | 29 |
| Clases Java test | 17 |
| Tests totales | ~50 metodos de test |
| Entidades JPA | 6 (Owner, Pet, PetType, Visit, Vet, Specialty) |
| Tablas BD | 7 |
| Controllers | 5 (Owner, Pet, Visit, Vet, Welcome/Crash) |
| Repositories | 3 (Owner, PetType, Vet) |
| Endpoints HTTP | 17 |
| Templates Thymeleaf | 11 HTML |
| Idiomas i18n | 9 |
| Perfiles Spring | 3 (default/H2, mysql, postgres) |
| Plugins Maven | 9 |
| Workflows GitHub Actions | 1 (build.yml) |
