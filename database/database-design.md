
# Database Design
## Government Form & Scheme Eligibility Assistant

## 1. Database Overview

**Database:** PostgreSQL

The database stores user accounts, user profiles, government schemes, eligibility rules, uploaded documents, eligibility assessment results, and subscription/payment information.

The database is designed to support Spring Boot and Spring Data JPA.

## 2. Table Definitions

### 2.1 app_user

**Purpose:** Stores registered user account information.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| full_name | VARCHAR(150) | NOT NULL |
| email | VARCHAR(255) | UNIQUE, NOT NULL |
| password_hash | VARCHAR(255) | NOT NULL |
| account_status | VARCHAR(20) | NOT NULL |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL |
| updated_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

### 2.2 user_profile

**Purpose:** Stores personal information required for scheme eligibility checks.

| Column | Data Type | Constraint |
|---|---|---|
| user_id | BIGINT | Primary Key, Foreign Key |
| date_of_birth | DATE | NULL |
| state | VARCHAR(100) | NULL |
| district | VARCHAR(100) | NULL |
| occupation | VARCHAR(100) | NULL |
| annual_income | NUMERIC(12,2) | NULL |
| education_level | VARCHAR(100) | NULL |
| family_size | INTEGER | NULL |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL |
| updated_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

Relationship: One user has one profile.

### 2.3 scheme

**Purpose:** Stores information about government schemes.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| scheme_name | VARCHAR(255) | NOT NULL |
| description | TEXT | NULL |
| department | VARCHAR(200) | NULL |
| official_url | TEXT | NULL |
| status | VARCHAR(20) | NOT NULL |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

### 2.4 eligibility_rule

**Purpose:** Stores eligibility conditions associated with government schemes.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| scheme_id | BIGINT | Foreign Key |
| rule_name | VARCHAR(150) | NOT NULL |
| rule_type | VARCHAR(50) | NOT NULL |
| rule_definition | JSONB | NOT NULL |
| is_mandatory | BOOLEAN | NOT NULL |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

Relationship: One scheme can have multiple eligibility rules.

### 2.5 scheme_document

**Purpose:** Stores metadata for official government scheme PDFs.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| scheme_id | BIGINT | Foreign Key |
| original_filename | VARCHAR(255) | NOT NULL |
| storage_key | TEXT | NOT NULL |
| mime_type | VARCHAR(100) | NOT NULL |
| file_size_bytes | BIGINT | NOT NULL |
| uploaded_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

### 2.6 user_document

**Purpose:** Stores metadata for PDFs uploaded by users.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| user_id | BIGINT | Foreign Key |
| document_type | VARCHAR(100) | NULL |
| original_filename | VARCHAR(255) | NOT NULL |
| storage_key | TEXT | NOT NULL |
| mime_type | VARCHAR(100) | NOT NULL |
| file_size_bytes | BIGINT | NOT NULL |
| processing_status | VARCHAR(30) | NOT NULL |
| uploaded_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

### 2.7 eligibility_assessment

**Purpose:** Records each eligibility assessment requested by a user.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| user_id | BIGINT | Foreign Key |
| status | VARCHAR(30) | NOT NULL |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL |
| completed_at | TIMESTAMP WITH TIME ZONE | NULL |

### 2.8 assessment_result

**Purpose:** Stores the eligibility outcome for each scheme evaluated in an assessment.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| assessment_id | BIGINT | Foreign Key |
| scheme_id | BIGINT | Foreign Key |
| eligibility_status | VARCHAR(30) | NOT NULL |
| explanation | TEXT | NULL |
| missing_information | JSONB | NULL |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

Possible eligibility statuses:
- SATISFIED
- NOT_SATISFIED
- CANNOT_DETERMINE

One assessment can produce results for multiple schemes.

### 2.9 subscription_plan

**Purpose:** Stores the available subscription plans.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| plan_name | VARCHAR(100) | UNIQUE, NOT NULL |
| price | NUMERIC(10,2) | NOT NULL |
| currency | VARCHAR(3) | NOT NULL |
| billing_interval | VARCHAR(20) | NOT NULL |
| included_uploads | INTEGER | NOT NULL |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

### 2.10 user_subscription

**Purpose:** Stores subscription details associated with users.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| user_id | BIGINT | Foreign Key |
| plan_id | BIGINT | Foreign Key |
| gateway_subscription_id | VARCHAR(255) | UNIQUE |
| status | VARCHAR(30) | NOT NULL |
| starts_at | TIMESTAMP WITH TIME ZONE | NULL |
| ends_at | TIMESTAMP WITH TIME ZONE | NULL |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

### 2.11 payment

**Purpose:** Stores payment transaction information.

| Column | Data Type | Constraint |
|---|---|---|
| id | BIGINT | Primary Key |
| user_id | BIGINT | Foreign Key |
| subscription_id | BIGINT | Foreign Key, NULL |
| gateway_payment_id | VARCHAR(255) | UNIQUE |
| amount | NUMERIC(10,2) | NOT NULL |
| currency | VARCHAR(3) | NOT NULL |
| payment_status | VARCHAR(30) | NOT NULL |
| paid_at | TIMESTAMP WITH TIME ZONE | NULL |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL |

### 2.12 assessment_document

**Purpose:** Connects uploaded user documents with eligibility assessments.

| Column | Data Type | Constraint |
|---|---|---|
| assessment_id | BIGINT | Primary Key, Foreign Key |
| document_id | BIGINT | Primary Key, Foreign Key |

Relationship: Many-to-Many between eligibility_assessment and user_document.

## 3. Table Relationships

1. app_user → user_profile: One-to-One
2. app_user → user_document: One-to-Many
3. app_user → eligibility_assessment: One-to-Many
4. scheme → eligibility_rule: One-to-Many
5. scheme → scheme_document: One-to-Many
6. eligibility_assessment → assessment_result: One-to-Many
7. scheme → assessment_result: One-to-Many
8. app_user → user_subscription: One-to-Many
9. subscription_plan → user_subscription: One-to-Many
10. app_user → payment: One-to-Many
11. user_subscription → payment: One-to-Many
12. eligibility_assessment → assessment_document: One-to-Many
13. user_document → assessment_document: One-to-Many

## 4. Important Database Design Decisions

- PostgreSQL is the selected relational database.
- BIGINT identity columns are used for primary keys.
- Foreign keys maintain referential integrity.
- JSONB stores flexible eligibility-rule definitions.
- PDF files are stored in file storage; PostgreSQL stores their metadata and storage keys.
- Timestamps use TIMESTAMP WITH TIME ZONE.
- Eligibility results use predefined statuses.
- Payment and subscription records support the planned subscription feature.
- The first three free PDF uploads must be enforced by the backend.

## 5. Security Considerations

- Never store plain-text passwords.
- Store only securely hashed passwords.
- Never store database passwords or personal connection details in this repository.
- Payment credentials and API secrets must be stored in environment variables.
- Payment verification must happen on the backend.

## 6. Backend Integration

The backend developer must review this design before implementing Java entities.

Spring Data JPA will map the Java entity classes to the corresponding PostgreSQL tables.

After integration, Hibernate should use schema validation rather than automatically creating or modifying production tables.

## 7. Future Extensions

- Document chunks for page-aware PDF retrieval.
- pgvector embeddings for semantic search.
- Citation records for source-backed AI responses.
- More detailed document-processing history.
