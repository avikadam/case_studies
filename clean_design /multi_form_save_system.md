# Multi-Form Save System

This project implements a scalable and efficient multi-form save architecture where each form type has its own static table. It supports draft saves, multi-step progression, and clean validation, while avoiding common pitfalls like schema bloat and weak data integrity.

---

## 🧩 Architecture Overview

### Core Design

- ✅ A central `forms` table for all form instances (metadata).
- ✅ Dedicated static tables per form type (e.g. `claim_form`, `registration_form`) with strictly defined fields.
- ✅ Support for multi-step saving using a `step_completed` column.
- ✅ Form status tracking: DRAFT, IN_PROGRESS, SUBMITTED.

---

## 📁 Database Schema

### `forms`
| Column         | Type       | Description                  |
|----------------|------------|------------------------------|
| form_id        | UUID (PK)  | Unique form identifier       |
| user_id        | UUID       | Linked user                  |
| form_type      | VARCHAR    | Form type (e.g., claim)      |
| status         | VARCHAR    | DRAFT, SUBMITTED, etc.       |
| created_at     | TIMESTAMP  | When form was created        |
| updated_at     | TIMESTAMP  | Last update timestamp        |

---

### Example: `claim_form`
| Column             | Type     | Description               |
|--------------------|----------|---------------------------|
| form_id            | UUID     | FK to `forms`             |
| claim_amount       | DECIMAL  | Amount claimed            |
| claim_reason       | TEXT     | Reason for the claim      |
| claim_date         | DATE     | Incident date             |
| attached_documents | TEXT     | References to documents   |
| step_completed     | INT      | Multi-step form progress  |

---

### Example: `registration_form`
| Column         | Type     | Description             |
|----------------|----------|-------------------------|
| form_id        | UUID     | FK to `forms`           |
| full_name      | VARCHAR  | Name of registrant      |
| dob            | DATE     | Date of birth           |
| email          | VARCHAR  | Email address           |
| phone          | VARCHAR  | Phone number            |
| address        | TEXT     | Postal address          |
| step_completed | INT      | Multi-step form progress|

---

## 🚀 Features

- Multi-step form save and resume support
- Separation of form logic by type for safety and clarity
- Strong validation possible at DB schema level
- Optimized for query performance and reporting
- Easy extensibility for new forms

---

## 🚫 Why Not Use a Single Table for All Forms?

Although tempting, storing all form fields in one large table introduces major drawbacks. 
Putting all static fields from multiple forms into one wide table (often called a **God table** or **monolithic form table**) introduces several key issues, especially when forms have very different fields:

| Issue                        | Problem Description |
|-----------------------------|---------------------|
| **Sparse Columns & NULLs**  | Each form uses only some fields → many NULLs per row |
| **Poor Validation**         | Hard to enforce constraints only for relevant fields |
| **Complex Queries**         | Queries become verbose, error-prone with irrelevant fields |
| **Indexing Inefficiency**   | Indexes for some fields are useless for other form types |
| **Tight Coupling**          | Adding/modifying a field for one form risks breaking others |
| **Difficult to Maintain**   | Schema is cluttered, logic hard to reason about |
| **No Type Safety**          | Can't use strict types for each form structure |

✅ **Solution**: Dedicated tables per form ensure schema clarity, safety, and better performance.

---

## 🌐 API Endpoints (Example)

- POST /forms -> Create new form entry
- PUT /forms/{id} -> Update status
- POST /claim-form -> Save or update a claim form
- POST /registration-form -> Save or update registration form
- GET /forms/{id} -> Retrieve full form metadata

---

## 🧪 Testing

- Unit tests for form creation, step progression, and validation
- Integration tests covering full multi-form submission workflows

---

## 📌 Conventions

- Use `form_id` as the shared foreign key
- Keep each form table limited to fields relevant only to that form
- Define and enforce constraints in form-specific tables
- Use `step_completed` to enable form resumption

---

## 🛠️ Tech Stack

- PostgreSQL / MySQL
- Java Spring Boot or Node.js for backend APIs
- JSON APIs for frontend integration
- Optional React/Vue UI for form rendering and progress saving

---

## 📞 Contact

For implementation support or to extend the system to new form types, contact the project maintainer.

---
