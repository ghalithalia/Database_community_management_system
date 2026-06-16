# Database_community_management_system
The **Resilience Project** is a decentralized collaborative platform designed to connect individuals and communities within a network of mutual assistance. It enables users to self-organize, manage skills, exchange services and vote on community governance.
## Community Management System — The Resilience Project

##  Key Features

* **Unified Entity Architecture:** Uses Class-Table Inheritance where both individuals and communities share a base `Entity` type, allowing seamless interaction across all features.
* **Democratic Governance:** Communities can vote to exclude members. If more than 50% of active members vote against a person, they are automatically flagged for removal.
* **Skill & Service Exchange:** Entities can list skills (levels 1 to 5) and offer services under three modalities: Free, Barter (exchange), or Commercial (priced in **Ğ1 cryptocurrency**).
* **Unidirectional Graphed Links:** Supports customized social links between individuals (e.g., "neighbor") and structural collaborations between communities.
* **Asymmetric Messaging:** Full ownership of sent messages with native reply-thread support.
* **Spatial Proximity:** Built-in geospatial tracking enabling a 1 km radius proximity view using location coordinates.

## 📊 Conceptual Data Model (UML)

Below is the conceptual class diagram representing the system's architecture. 

![UML Class Diagram](docs/conceptual_schema.png)

## 💾 Logical Data Model (LDM)

The relational schema is optimized for **PostgreSQL** and enforces strict data integrity via domain constraints and composite keys.

### 1. Core Entities & Inheritance
*   **`Entity`** (`id` INT [PK], `displayName` VARCHAR, `createdAt` TIMESTAMP)
*   **`Person`** (`id` INT [PK, FK → Entity.id], `email` VARCHAR [UNIQUE], `latitude` FLOAT, `longitude` FLOAT)
*   **`Community`** (`id` INT [PK, FK → Entity.id], `name` VARCHAR [UNIQUE], `description` TEXT)

### 2. Memberships & Governance
*   **`Membership`** (`personId` INT [FK → Person.id, ON DELETE RESTRICT], `communityId` INT [FK → Community.id, ON DELETE CASCADE], `joinedAt` TIMESTAMP, `isExcluded` BOOLEAN [DEFAULT FALSE])
    * *Primary Key:* `(personId, communityId)`
*   **`ExclusionVote`** (`id` INT [PK], `communityId` INT [FK → Community.id], `targetPersonId` INT [FK → Person.id], `voterPersonId` INT [FK → Person.id], `votedAt` TIMESTAMP, `vote` BOOLEAN)

### 3. Networks & Links (Self-Associations)
*   **`IndividualLink`** (`fromPersonId` INT [FK → Person.id], `toPersonId` INT [FK → Person.id], `description` TEXT)
    * *Primary Key:* `(fromPersonId, toPersonId)`
*   **`CommunityLink`** (`fromCommunityId` INT [FK → Community.id], `toCommunityId` INT [FK → Community.id], `description` TEXT)
    * *Primary Key:* `(fromCommunityId, toCommunityId)`

### 4. Skills & Services Marketplace
*   **`Skill`** (`id` INT [PK], `label` VARCHAR [UNIQUE], `description` TEXT)
*   **`EntitySkill`** (`entityId` INT [FK → Entity.id], `skillId` INT [FK → Skill.id], `level` INT, `declaredAt` TIMESTAMP)
    * *Primary Key:* `(entityId, skillId)`
    * *Constraint:* `CHECK (level BETWEEN 1 AND 5)`
*   **`Service`** (`id` INT [PK], `providerId` INT [FK → Entity.id], `title` VARCHAR, `description` TEXT, `type` VARCHAR, `priceG1` FLOAT, `wantedServiceId` INT [FK → Service.id], `createdAt` TIMESTAMP)
    * *Constraint:* `CHECK (type IN ('FREE', 'EXCHANGE', 'COMMERCIAL_G1'))`

### 5. Wallets & Messaging
*   **`G1Account`** (`id` INT [PK], `entityId` INT [FK → Entity.id], `publicKey` VARCHAR [UNIQUE], `label` VARCHAR, `createdAt` TIMESTAMP)
*   **`Message`** (`id` INT [PK], `senderId` INT [FK → Entity.id], `recipientId` INT [FK → Entity.id], `sentAt` TIMESTAMP, `subject` VARCHAR, `body` TEXT, `inReplyToId` INT [FK → Message.id], `ownerId` INT [FK → Entity.id])

### 6. Geospatial Views (Derived)
*   **`ProximityView`** (`entityA` INT, `entityB` INT, `distanceMeters` FLOAT)
    * *Calculated dynamically using spatial distance formulas.*
