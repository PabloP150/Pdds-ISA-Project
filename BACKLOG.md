# Backlog Recovery: User Stories

**Methodology:**
Reverse-engineering of requirements based on the existing Controller classes found in `src/main/java/org/springframework/samples/petclinic`.

## Epic: Owner Management (Core Domain)

### User Story 1: Register New Owner
**As a** Receptionist,
**I want to** register a new Pet Owner in the system,
**So that** we can start tracking their pets and medical history.

* **Acceptance Criteria:**
    * Must capture details like Address, City, and Telephone.
    * System checks for errors before saving.
* **Technical Mapping:**
    * **Controller:** `OwnerController.java`
    * **Method:** `processCreationForm` (POST `/owners/new`)

### User Story 2: Find Owner
**As a** Veterinarian or Receptionist,
**I want to** search for an owner by their last name,
**So that** I can quickly access their profile.

* **Acceptance Criteria:**
    * Supports partial matching (starts with).
    * Supports pagination (5 items per page).
    * If one result is found, redirect to details; if multiple, show list.
* **Technical Mapping:**
    * **Controller:** `OwnerController.java`
    * **Method:** `processFindForm` (GET `/owners`)

### User Story 3: Edit Owner Details
**As a** Receptionist,
**I want to** update an existing owner's contact information,
**So that** our records remain accurate.

* **Acceptance Criteria:**
    * Validates that the Owner ID in the form matches the URL.
* **Technical Mapping:**
    * **Controller:** `OwnerController.java`
    * **Method:** `processUpdateOwnerForm` (POST `/owners/{ownerId}/edit`)

---

## Epic: Pet Management

### User Story 4: Add Pet to Owner
**As a** Registered Owner (via Receptionist),
**I want to** add a new pet to my profile,
**So that** the clinic can schedule visits for it.

* **Acceptance Criteria:**
    * Must validate birth date (cannot be in the future).
    * Prevents duplicate pet names for the same owner.
* **Technical Mapping:**
    * **Controller:** `PetController.java`
    * **Method:** `processCreationForm` (POST `/owners/{ownerId}/pets/new`)

### User Story 5: Edit Pet Details
**As a** Receptionist,
**I want to** update a pet's details (e.g., correct a birth date),
**So that** the medical record is precise.

* **Acceptance Criteria:**
    * Updates existing record instead of creating a new one.
* **Technical Mapping:**
    * **Controller:** `PetController.java`
    * **Method:** `processUpdateForm` (POST `/pets/{petId}/edit`)

---

## Epic: Visit Management

### User Story 6: Schedule a Visit
**As a** Veterinarian,
**I want to** add a visit record (date and description) to a specific pet,
**So that** we maintain a medical history.

* **Acceptance Criteria:**
    * Must be linked to a valid Pet and Owner.
    * Ensures data freshness by reloading the pet before booking.
* **Technical Mapping:**
    * **Controller:** `VisitController.java`
    * **Method:** `processNewVisitForm` (POST `/owners/{ownerId}/pets/{petId}/visits/new`)

---

## Epic: Veterinary Staff

### User Story 7: View Veterinarians
**As a** User,
**I want to** see a list of all veterinarians and their specialties,
**So that** I know who works at the clinic.

* **Acceptance Criteria:**
    * List supports pagination.
    * Available as HTML view or JSON/XML resource.
* **Technical Mapping:**
    * **Controller:** `VetController.java`
    * **Method:** `showVetList` (GET `/vets.html`)