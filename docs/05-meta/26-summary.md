# Documentation on How To Build Apps With CauferoAppStarter FileMaker Application

# FileMaker Table Creation Rules (CauferoAppStarter)

These rules define how to create new tables so they match the structure, naming, and field patterns already used in this FileMaker system.

---

## 1. Schema Preflight Rule (Strict, Must Run First)

Any AI agent tasked to design or create this FileMaker application MUST run a schema preflight before proposing or creating any new tables or fields.

### 1.1 Goal
Prevent duplicate tables and duplicate fields by reusing what already exists.

### 1.2 What Counts As A Duplicate
A duplicate is any of the following:
- A table name that already exists in **Current Tables in CauferoAppStarter**
- A field name that already exists in the target table’s **Fields of the Different Tables**
- A "same meaning, different name" field where an equivalent field already exists (example: creating `Phone Number` when `Phone` already exists and is used system-wide)

### 1.3 Preflight Steps (Required)
Before outputting any new table or field definition, the AI MUST do this:

1. List the required entities implied by the application description (example: Staff, Patients, Vendors, Purchase Orders).
2. Check existing tables first:
   - If a table already exists and matches the entity, reuse it.
   - If a table partially matches, extend it only by adding missing fields (after field checks below).
3. For each table to be reused or created, check fields first:
   - Compare each required field against the existing field list for that table.
   - Reuse the existing field name if it exists and fits the meaning.
4. Normalize names when comparing (for duplication detection):
   - Trim spaces
   - Case-insensitive match
   - Treat `_` and multiple spaces as equivalent separators for comparison
   - Example: `Full Name`, `Fullname`, `Full_Name` should be treated as potential duplicates and resolved into the existing system preference for that table.
5. Resolve collisions using these rules:
   - If the exact name exists: do not create a new one
   - If the meaning exists under a different name: reuse the existing one and document the mapping
   - If two meanings truly differ, create a new field using a clearer name that does not clash (example: `Contact Phone` vs `Phone`)
6. Only after Steps 1 to 5:
   - Propose new tables
   - Propose new fields
   - Propose join tables and child tables

### 1.4 Reuse Priority Rules (When Choosing Between Similar Existing Fields)
When multiple existing fields are similar, the AI MUST choose using this priority order:
1. Field already used in the same module (example: Clinical tables)
2. Field name pattern already used across the system (example: `Mobile` in Patients, `Phone` in Staff, `Contact Phone` in Vendors)
3. Field that matches the system’s indexing and audit style for that table type

### 1.5 Strict Prohibitions
- The AI MUST NOT create a new table or field if an equivalent already exists.
- The AI MUST NOT rename existing schema objects as part of creation output.
- The AI MUST NOT create "helper duplicates" like `Status 2`, `Status Text`, `New Status`. Equivalent fields must be reused, or the proper field strategy must be applied.

---

## 2. Relationship and Layout Architecture Rule (SQL + WebViewer Only)

This system is SQL-driven and WebViewer-driven.

### 2.1 Do Not Create Relationships In The Relationship Graph
- The AI MUST NOT create relationships in FileMaker’s Relationship Graph.
- The AI MUST NOT add table occurrences solely for relationship purposes.
- The AI MUST NOT attempt to design layouts around related records.

### 2.2 Data Retrieval and Linking Method
- All relationships will be expressed inside **ExecuteSQL** queries.
- All layouts are **WebViewer-based**.
- All data displayed in UI will be fetched via **ExecuteSQL** queries (and passed into WebViewer as JSON or other structured formats).

### 2.3 Implication For Table Design
- Foreign keys must still be created normally (Text fields ending with ` ID`, indexed).
- Join tables must still be created normally.
- The absence of a relationship graph does not remove the need for clean keys, join tables, audit fields, indexing, and naming rules.

---

## 3. Table Naming Rules

### 3.1 Base Tables
- Use plural only when the real-world concept is naturally plural in your system style, otherwise keep it clean and simple.
- Examples:
  - `Staff`, `Patients`, `Businesses`, `Vendors`, `Materials`, `Equipment`, `Projects`, `Products`

### 3.2 Child Tables (Subtables)
- When a parent record can have multiple entries of a related type, create a child table.
- Child tables must start with the parent name.
- Pattern:
  - `<Parent> <Child>`
- Examples:
  - `Staff Education`
  - `Staff Documents`
  - `Patient Visit Vitals`
  - `Equipment Documents`

### 3.3 Join Tables (Many-to-Many)
- When two entities relate many-to-many, create a join table.
- Join table name must read naturally as a link between both tables.
- Patterns:
  - `<A> <B>`
  - `<A> <B> <C>` (when the join has an extra layer)
- Examples:
  - `Role Links`
  - `Lab Test Specimens`
  - `Lab Test Specimen Analytes`

### 3.4 Hierarchy Tables
- For nested structures, use a Parent ID.
- Pattern:
  - `Parent ID` or `<Entity> Parent ID`
- Examples:
  - `Links::Parent Link ID`
  - `Job Titles::Parent ID`

### 3.5 Section Tables for Long Documents
- When a table extends a main document table, suffix it with ` _ ` and a section name.
- Pattern:
  - `<Main Table> _ <Section>`
- Examples:
  - `Software Requirement Documents _ Functional Requirements`
  - `Software Requirement Documents _ Price Breakdown`

---

## 4. Relationship Field Rules

### 4.1 Foreign Keys
- Foreign keys are Text fields that end with ` ID`.
- Pattern:
  - `<Related Table> ID`
- Examples:
  - `Staff ID`, `Patient ID`, `Business ID`, `Vendor ID`, `Equipment ID`
- Options:
  - Indexed
- Rule:
  - Foreign keys appear near the top of the table, immediately before the audit fields.
- This includes role-based foreign keys (examples: `Doctor User ID`, `Approver User ID`, `Maker ID`, `Checker ID`, `Decider Staff ID`). These must still be Text, Indexed, and positioned with the other foreign keys.
- Rule:
  - Role-based keys are still foreign keys even when the prefix is a role name (example: Doctor, Maker, Checker) instead of a table name.

### 4.2 Join Table Structure
A join table must contain:
- `ID`
- Both foreign keys linking the two tables
- `Date Created`
- `Time Created`

Optional:
- Additional context fields only if the join itself has attributes.

---

## 5. Core Field Pack (Every Table Must Have This)

### 5.1 Primary Key
- Every table must have:
  - `ID` (Text)
- Options:
  - Indexed
  - Auto-enter Calculation
- Rule:
  - `ID` is always the first field in the table.

### 5.2 Created Audit Fields
- Every table must have:
  - `Date Created` (Date, Creation Date, Indexed)
  - `Time Created` (Time, Creation Time, Indexed)
- Rule:
  - These fields appear right after `ID` and foreign keys.

### 5.3 Modified Audit Fields (Conditional)
- Add these only for tables that store documents, uploaded files, or content that is expected to be updated after creation:
  - `Date Modified` (Date, Indexed)
  - `Time Modified` (Time, Indexed)

---

## 6. Indexing Rules

Index these by default:
- `ID`
- All foreign keys (`... ID`)
- All audit fields (`Date Created`, `Time Created`, etc)
- Primary label fields (fields used for searching or display), such as:
  - `Name`, `Title`, `Number`, `Type`, `Status`
- Operational filter fields:
  - Dates used in scheduling, tracking, or reporting
  - Times used in tracking or audit
- Sorting fields:
  - `Order` (Number)

Do not index:
- Long narrative blocks. Example: `Description`

---

## 7. Ordering and UI Control Rules

If records must be displayed in a controlled order, include:
- `Order` (Number, Indexed)

Common use cases:
- Menus and navigation lists
- Questionnaire sections and questions
- Feature lists
- Any list shown in a UI that must be stable and predictable

---

## 8. Status Rules

### 8.1 Standard Status Field
- Add a standard `Status` field only when the table represents an entity that has a single, primary lifecycle state.
- Field:
  - `Status` (Text)
- Indexing:
  - Indexed
- Auto-Enter:
  - Auto-enter Data (default status), where appropriate

Examples where `Status` makes sense:
- Master data and operational entities such as `Vendors`, `Products`, `Lab Tests`, `KPIs`, `Leave Types`, `Locations`

### 8.2 Process-Specific Status Fields (Preferred For Workflows)
- When a workflow has multiple checkpoints or parallel steps, do not overload a single `Status`.
- Instead, create explicit step statuses so each checkpoint is independently trackable.
- Pattern:
  - `<Step> Status`
- Indexing:
  - Each `<Step> Status` must be Text and Indexed
- Auto-Enter:
  - Auto-enter Data may be used to set safe defaults (example: `Pending`)

Examples:
- `Vitals Status`
- `Consultation Status`
- `Consultation Billed Status`
- `Consultation Payment Status`
- `Admission Status`
- `Received Status`

Rule:
- If a table needs both an overall lifecycle and step statuses, use `Status` for the overall lifecycle and keep the step statuses separate.

---

## 9. Snapshot Label Rules (ID + Name Together)

When a table represents a transaction or historical event, store both:
- The foreign key (`... ID`)
- The display label at the time of the event (snapshot)

Examples of snapshot fields:
- `Material Name`
- `Medication Name`
- `Service Name`

Rule:
- The ID remains the source of truth.
- The snapshot is used for display, reporting, and history preservation.

---

## 10. Media and Attachments Rules

### 10.1 Container Storage
Use Container fields with `External (Secure)` for:
- Photos
- Uploaded files
- Logos
- App icons

### 10.2 URL Storage
Use Text URL fields when:
- The file is hosted externally
- You want a lightweight reference instead of storing the file

Examples:
- `Photo URL`
- `Logo URL`
- `Image URL`
- `Product URL`

---

## 11. Settings and Globals Rules

### 11.1 Where Globals Go
Globals belong in the `Settings` table.
Do not place global fields inside operational tables.

### 11.2 Global Naming
Use clear names that describe their purpose.
Examples:
- `Search`
- `Date` (Date[2] global range)
- `Selected <Entity> ID`

---

## 12. Reporting Helper Field Rules

When reports or dashboards need grouping, add helper fields such as:
- `Month Created` (Text, Indexed)
- `Month Created Number` (Number, Indexed)
- `Year` (Number, Indexed)

Rule:
- Helpers exist to reduce calculation load in reporting and make grouping consistent.

---

## 13. Field Order Rules (In the Table Definition)

Use this field ordering pattern:

1. `ID`
2. Foreign keys (`... ID`)
3. Created audit fields:
   - `Date Created`
   - `Time Created`
4. Modified audit fields (only if applicable):
   - `Date Modified`
   - `Time Modified`
5. Primary label fields (`Name`, `Title`, `Number`, etc.)
6. Operational fields (dates, times, amounts, descriptions)
7. UI fields (`Order`, icons, display text)
8. Long text fields (notes, narrative content)
9. Status fields (`Status`, workflow statuses)

---

## 14. AI Agent Output Requirements

When the AI agent creates or extends tables, it must output the following.

### 14.1 Table Summary
- Table name
- Table type: Base, Child, Join, Hierarchy, Section
- Action: Reuse, Extend, Create

### 14.2 Field List (With Options)
For each field:
- Field name
- Type
- Key options (Indexed, Auto-enter, Creation Date, Global, External Secure, etc.)

### 14.3 Schema Preflight Report (Mandatory)
Before any table definitions, the AI must output a preflight report with:

#### 14.3.1 Tables Reused
- Table name
- Why it matches the requirement

#### 14.3.2 Fields Reused (Per Table)
For each reused table:
- Fields that already exist and will be reused
- Required fields that are missing and will be added

#### 14.3.3 New Tables Proposed (Only If Needed)
For each new table:
- Why no existing table fits
- Confirmation the table name does not already exist

#### 14.3.4 Field Name Mapping (When A Different Name Already Exists)
If the application description mentions a field name that differs from this schema:
- Show the mapping
  - Example: `"Phone Number" -> "Phone"` (Staff)
  - Example: `"Phone" -> "Mobile"` (Patients)

### 14.4 Compliance Check
The AI must confirm:
- `ID` exists and matches the standard format
- Audit fields exist
- Foreign keys are Text and indexed
- Status fields are included when appropriate
- `Order` exists when UI ordering is needed
- Modified fields exist when the table stores files or editable content

### 14.5 Duplicate Prevention Compliance Check (Mandatory)
The AI must explicitly confirm:
- Every proposed table name is not already in the current tables list
- Every proposed field name is not already in the target table field list
- Any equivalent fields were reused instead of recreated
- Any naming collisions were resolved using the Schema Preflight Rule (Section 1)

---

## 15. Error Control Protocol (Strict, Must Run Before Final Output)

Before the AI outputs any final table definition, it MUST run an error control pass and include the results in the output.

### 15.1 Purpose
Prevent contradictions, missing required fields, wrong field types, wrong indexing, and accidental violations of these rules.

### 15.2 Required Checks
The AI MUST validate all of the following:

1. **Schema Preflight Completed**
   - Confirms Section 1 was executed.
   - Confirms reuse decisions were made before creating anything.

2. **Architecture Compliance**
   - Confirms no relationship graph design is proposed.
   - Confirms no table occurrences are proposed for relationship purposes.
   - Confirms data retrieval is described using ExecuteSQL and WebViewer.

3. **Naming Compliance**
   - Table name matches the naming rules (Base, Child, Join, Hierarchy, Section).
   - Foreign keys end with ` ID`.
   - Join tables read naturally and contain both foreign keys.

4. **Field Pack Compliance**
   - `ID` exists and is the first field.
   - Created audit fields exist.
   - Modified audit fields exist only when appropriate.

5. **Field Type Compliance**
   - All foreign keys are **Text**.
   - All date and time fields use Date or Time, and Timestamp only when needed.
   - Containers use External (Secure) only when storing files.

6. **Indexing Compliance**
   - Confirms all fields that must be indexed are indexed.
   - Confirms fields that should not be indexed are not indexed (example: long narrative fields like `Description`).

7. **Calculation Field Rule Compliance**
   - Confirms there are zero Calculation field types.
   - Confirms only approved auto-enter calculations are used.

### 15.3 Required Output Format For This Protocol
The AI MUST include a short validation block like this before the final table definition:

- `Schema Preflight`: PASS or FAIL
- `Architecture Rule`: PASS or FAIL
- `Naming Rules`: PASS or FAIL
- `Core Field Pack`: PASS or FAIL
- `Field Types`: PASS or FAIL
- `Indexing Rules`: PASS or FAIL
- `Calculation Field Rule`: PASS or FAIL

If any item fails, the AI MUST stop and output what must be fixed first.

---

## 16. Quick Templates the AI Must Reuse

### 16.1 Base Table Template
- `ID` (Text, Indexed, Auto-enter Calculation)
- `Date Created` (Date, Indexed, Creation Date)
- `Time Created` (Time, Indexed, Creation Time)
- `<Primary Label>` (Text, Indexed)
- Optional:
  - `Description` (Text)
  - `Status` (Text, Indexed, Auto-enter Data)

### 16.2 Child Table Template
- `ID`
- `Parent ID` (Text, Indexed)
- `Date Created`
- `Time Created`
- Child fields (Indexed where needed)
- Optional:
  - `Status`

### 16.3 Join Table Template
- `ID`
- `A ID` (Text, Indexed)
- `B ID` (Text, Indexed)
- `Date Created`
- `Time Created`

### 16.4 Document/File Table Template
- `ID`
- `<Parent> ID` (Text, Indexed)
- `Date Created`
- `Time Created`
- `Date Modified`
- `Time Modified`
- `Uploaded By` (Text, Indexed)
- `Type` (Text, Indexed)
- `Name` (Text, Indexed)
- `File Extension` (Text, Indexed)

---

## 17. Calculation Field Rule (Strict)

### 17.1 Default Rule
- Apart from `ID` (and any other approved auto-enter calculations listed below), no field is allowed to be a Calculation field.
- All non-key fields must be stored fields (Text, Number, Date, Time, Timestamp, Container) and populated by:
  - Scripts
  - Auto-enter Data (where appropriate)
  - User entry
  - Imports

### 17.2 Allowed Auto-Enter Calculations
Only the following types of auto-enter calculations are allowed:
- `ID` (primary key generation)
- Device or environment capture fields (example: `Device Type`, `Machine ID`, `IP Address`)
- Optional reporting helpers where you intentionally store the result (example: `Month Created`, `Month Created Number`, `Year`), as stored values

Rule:
- These fields are still stored fields. They may use auto-enter calculation to set the value, but they must not be defined as Calculation field types.

### 17.3 Prohibited
- No unstored calculations.
- No Calculation field types for:
  - Labels (example: `Fullname`)
  - Status derivation
  - Totals
  - Rollups
  - Anything you plan to display in UI

If a derived value is needed, create a normal stored field and set it via script (or via auto-enter data when safe).

### 17.4 AI Agent Compliance Check
Before outputting a table definition, the AI must confirm:
- The only calculations used are in approved auto-enter settings.
- There are zero Calculation field types in the table.

### 17.5 Preflight Still Applies
Even when adding fields under the rules in this section:
- The AI must still run the Schema Preflight Rule (Section 1)
- If a required stored field already exists, reuse it
- Do not create an extra stored field simply because calculation fields are prohibited

<br/>

---

<br/>

## Current Tables in CauferoAppStarter

### Settings
- Settings
- Objects_as_JS
- ChartJS
- Apps

---

### Links
- Links

---

### Media
- Images

---

### Access Control
- Roles
- Role Links
- Users
- User Activities

---

### Registration
- Registration
- Logo Files
- Company Colours
- Departments
- Job Titles
- Job Title Notches
- Job Title Responsibilities
- Job Title Requirements
- Leave Types
- Scoring Systems
- Scoring System Scores
- Appraisal Templates
- Appraisal Template Question Categories
- Appraisal Template Questions
- Appraisal Periods
- KPIs

---

### Staff
- Staff
- Staff Emergency Contacts
- Staff Education
- Staff Documents
- Staff Job Titles
- Staff Job Title Notch History
- Staff Attendance
- Staff Leave Requests
- Staff Appraisal Templates
- Staff Appraisals
- Staff Appraisal Answers
- KPI Assignments

---

### Businesses
- Industries
- Business Types
- Businesses
- Activities Regarding Businesses
- Business Operational Challenges
- Business Solution Features
- Software Requirement Documents
- Software Requirement Documents _ Functional Requirements
- Software Requirement Documents _ Price Breakdown

---

### Lab
- Specimens
- Analytes
- Lab Tests
- Lab Test Specimens
- Lab Test Specimen Analytes

---

### Insurance
- Insurers
- Insurance Policies

---

### Clinical
- Medication Dosages
- Patients
- Patient Emergency Contacts
- Patient Primary Care Physicians
- Patient Payment Options
- Patient Appointments
- Patient Visit Vitals
- Patient Visit Lab Tests
- Patient Visit Lab Test Analyte Results
- Patient Visit Lab Results Documents
- Patient Visit Diagnoses
- Patient Visit Prescriptions
- Patient Visit Services
- Patient Visit Treatment Administrations
- Patient Visit Consultation Notes
- InPatient Notes
- InPatient Vitals
- Walk_Ins
- Patient Visit Bill Items

---

### Projects
- Projects

---

### Inventory
- Material Categories
- Materials
- Stock Adjustments
- Equipment Categories
- Equipment
- Operating Conditions
- Equipment Optimal Operating Conditions
- Equipment Related Parts
- Equipment Maintenance
- Equipment Decommissioning
- Equipment Documents
- Vendors
- Locations
- Products
- Product Features

---

### Maintenance
- Maintenance Schedule Sheets
- Maintenance Schedules
- Maintenance Activities
- Maintenance Materials
- Maintenance Labour

---

### Purchasing
- Purchase Orders
- Purchase Order Items

---

### Social Media
- Social Media Files
- Social Media Posts

---

### Payroll
- Payroll Items
- Payroll Items For Job Title Notches
- Payment Groups
- Payment Groups of Payroll Items
- Payroll Periods
- Remunerations
- Remuneration Details

---

### Billing
- Other Billing Items

---

### SMS
- SMS Account
- SMS Credits

---

### Alerts
- Alerts

---

<br>

## Fields of the Different Tables

### Table Name: Settings

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| App ID | Text | Indexed |
| Notes App ID | Text | Indexed |
| App Icon | Container | External (Secure) |
| App Logo _Lighter | Container | External (Secure) |
| App Logo _Darker | Container | External (Secure) |
| --- |  |  |
| Business ID | Text | Indexed |
| Business Logo | Container | External (Secure) |
| Menu Mode | Text | Indexed |
| --- |  |  |
| Template_ImportFile | Container | External (Secure) |
| Search | Text | Global |
| Date | Date [2] | Global |
| Location | Text | Indexed |
| General Notes | Text | Indexed |
| Displayed Notes | Text | Indexed |
| Last Bath | Timestamp | Indexed |
| Most Previous Receipt Number | Text | Indexed |
| Dev Threshold Minutes | Number | Indexed |
| Dev Fire Script Name | Text | Indexed |
| Email Credits | Number | Indexed |
| Google API Refresh Token | Text | Indexed |
| Google API Device Code | Text | Indexed |
| Google API User Code | Text | Indexed |
| Google API Poll Duration | Text | Indexed |
| --- |  |  |
| Shipping Address | Text | Indexed |
| Songs | Text | Indexed |
| --- |  |  |
| Selected Link ID | Text | Global |
| Selected User ID | Text | Global |
| Selected Role ID | Text | Global |
| --- |  |  |
| Selected Job Title ID | Text | Global |
| Selected Item ID | Text | Global |
| Selected Material ID | Text | Global |
| Selected Equipment ID | Text | Global |
| Selected PM Schedule Sheet ID | Text | Global |
| Selected PM Schedule ID | Text | Global |
| --- |  |  |
| Selected Purchase Order ID | Text | Global |
| Selected Purchase Order Item ID | Text | Global |

---

### Table Name: Objects_as_JS

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Object Name | Text |  |
| Object | Container |  |
| JS | Text |  |

---

### Table Name: ChartJS

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| JS | Text |  |

---

### Table Name: Apps

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Name | Text | Indexed |
| Slogan | Text | Indexed |
| Splash Background | Text |  |
| Splash Screen Loading Statement | Text | Indexed |
| Partners | Text | Indexed |
| App Launch Year | Number | Indexed |
| Available Themes | Text | Indexed |
| Notes | Text | Indexed |

---

### Table Name: Links

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed |
| Parent Link ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Link | Text | Indexed |
| Description | Text | Indexed |
| Order | Number | Indexed |
| Layout | Text | Indexed |
| SVG Icon | Text | Indexed |
| --- |  |  |
| Pending | Number | Indexed, Auto-enter Data |
| Note | Text | Indexed |
| --- |  |  |
| ChatGPT Query | Text | Indexed |

---

### Table Name: Roles

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| App ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Role | Text | Indexed |

---

### Table Name: Role Links

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Role ID | Text | Indexed |
| Link ID | Text | Indexed |
| App ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |

---

### Table Name: Users

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Role ID | Text | Indexed |
| Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Photo | Container | External (Secure) |
| First Name | Text | Indexed |
| Last Name | Text | Indexed |
| Username | Text | Indexed |
| --- |  |  |
| Preferred Colour Theme | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |
| Logged In | Number | Indexed, Auto-enter Data |

---

### Table Name: User Activities

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| User ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Device Type | Text | Auto-enter Calculation |
| Machine ID | Text | Auto-enter Calculation |
| IP Address | Text | Auto-enter Calculation |
| Activity Type | Text | Indexed |
| Activity | Text | Indexed |

---

### Table Name: Registration

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Number | Text | Indexed |
| TIN | Text | Indexed |
| Phones | Text | Indexed |
| Emails | Text | Indexed |
| Website | Text | Indexed |
| Address | Text |  |

---

### Table Name: Logo Files

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |

---

### Table Name: Company Colours

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Colour | Text | Indexed |
| Gradient1 | Text |  |
| Gradient2 | Text |  |

---

### Table Name: Departments

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| HOD Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Description | Text |  |

---

### Table Name: Job Titles

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Department ID | Text | Indexed |
| Parent ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Description | Text |  |

---

### Table Name: Job Title Notches

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Job Title ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Notch | Number | Indexed |
| Key | Text | Indexed |

---

### Table Name: Job Title Responsibilities

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Job Title ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Responsibility | Text | Indexed |

---

### Table Name: Job Title Requirements

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Job Title ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Requirement | Text | Indexed |

---

### Table Name: Leave Types

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Description | Text |  |
| Total Permissible Days | Number | Indexed |
| Status | Text | Indexed |

---

### Table Name: Scoring Systems

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |

---

### Table Name: Scoring System Scores

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Scoring System ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Score | Number | Indexed |
| Description | Text |  |

---

### Table Name: Appraisal Templates

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Scoring System ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Description | Text |  |

---

### Table Name: Appraisal Template Question Categories

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Appraisal Template ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Order | Number | Indexed |
| Category | Text | Indexed |

---

### Table Name: Appraisal Template Questions

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Appraisal Template ID | Text | Indexed |
| Appraisal Template Question Category ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Order | Number | Indexed |
| Title | Text | Indexed |
| Question | Text |  |
| --- |  |  |
| Status | Text | Indexed |

---

### Table Name: Appraisal Periods

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Start Date | Date | Indexed |
| End Date | Date | Indexed |
| Year | Number | Indexed |

---

### Table Name: KPIs

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Appraisal Period ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Title | Text | Indexed |
| Description | Text |  |
| Due Date | Date | Indexed |
| Priority | Number | Indexed |
| Score | Number | Indexed |
| --- |  |  |
| Status | Text | Indexed |

---

### Table Name: Staff

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Photo URL | Text | Indexed |
| Staff Number | Text | Indexed |
| Title | Text | Indexed |
| First Name | Text | Indexed |
| Last Name | Text | Indexed |
| Other Names | Text | Indexed |
| Maiden Name | Text | Indexed |
| --- |  |  |
| Gender | Text | Indexed |
| Date of Birth | Date | Indexed |
| Date of Birth _Day | Text | Indexed |
| Date of Birth _MonthName | Text | Indexed |
| Date of Birth _MonthNumber | Number | Indexed |
| Hometown | Text | Indexed |
| Hometown Region | Text | Indexed |
| Nationality | Text | Indexed |
| --- |  |  |
| Phone | Text | Indexed |
| Email | Text | Indexed |
| Residential Address | Text | Indexed |
| GPS | Text | Indexed |
| Postal Address | Text | Indexed |
| --- |  |  |
| Marital Status | Text | Indexed |
| Wedding Day | Text | Indexed |
| Number of Children | Text | Indexed |
| --- |  |  |
| Languages | Text | Indexed |
| --- |  |  |
| Bank Name | Text | Indexed |
| Bank Branch | Text | Indexed |
| Bank Account Number | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Staff Emergency Contacts

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Relationship | Text | Indexed |
| Phone | Text | Indexed |
| Email | Text | Indexed |
| Address | Text | Indexed |

---

### Table Name: Staff Education

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Institution | Text | Indexed |
| Programme | Text | Indexed |
| Result | Text | Indexed |
| Start Date | Date | Indexed |
| Finish Date | Date | Indexed |

---

### Table Name: Staff Documents

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| Date Modified | Date | Indexed |
| Time Modified | Time | Indexed |
| --- |  |  |
| Uploaded By | Text | Indexed |
| Type | Text | Indexed |
| Number | Text | Indexed |
| Name | Text | Indexed |
| File Extension | Text | Indexed |

---

### Table Name: Staff Job Titles

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Staff ID | Text | Indexed |
| Job Title ID | Text | Indexed |
| Current Notch ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Employment Type | Text | Indexed |
| Location Type | Text | Indexed |
| Start Date | Date | Indexed |
| End Date | Date | Indexed |

---

### Table Name: Staff Job Title Notch History

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Staff ID | Text | Indexed |
| Job Title ID | Text | Indexed |
| Notch ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Start Date | Date | Indexed |
| End Date | Date | Indexed |

---

### Table Name: Staff Attendance

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Date In | Date | Indexed |
| Time In | Time | Indexed |
| Date Out | Date | Indexed |
| Time Out | Time | Indexed |

---

### Table Name: Staff Leave Requests

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Staff ID | Text | Indexed |
| Leave Type ID | Text | Indexed |
| Decider Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| Month Name Created | Text |  |
| --- |  |  |
| Start Date | Date | Indexed |
| End Date | Date | Indexed |
| Return Date | Date | Indexed |
| Total Days | Text | Indexed |
| --- |  |  |
| Reason | Text |  |
| --- |  |  |
| Decision Date | Date | Indexed |
| Decision Time | Time | Indexed |
| --- |  |  |
| Status | Text | Indexed |

---

### Table Name: Staff Appraisal Templates

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Appraisal Template ID | Text | Indexed |
| Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |

---

### Table Name: Staff Appraisals

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Staff ID | Text | Indexed |
| Appraisal Template ID | Text | Indexed |
| Scoring System ID | Text | Indexed |
| Appraisal Period ID | Text | Indexed |
| Appraiser Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Template Highest Score | Number | Indexed |
| Total Score | Number | Indexed |
| Total Score% | Number | Indexed |
| Total KPI Score | Number | Indexed |
| Total KPI Score% | Number | Indexed |
| Overall Score% | Number | Indexed |
| --- |  |  |
| Date Submitted | Date | Indexed |
| Time Submitted | Time | Indexed |
| Date Appraised | Date | Indexed |
| Time Appraised | Time | Indexed |
| --- |  |  |
| Staff Remarks | Text |  |
| Supervisor Remarks | Text |  |
| --- |  |  |
| Status | Text | Indexed |

---

### Table Name: Staff Appraisal Answers

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Staff Appraisal ID | Text | Indexed |
| Appraisal Template Question ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Answer | Number | Indexed |

---

### Table Name: KPI Assignments

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| KPI ID | Text | Indexed |
| Assignee ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Assignee Type | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed |

---

### Table Name: Industries

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Description | Text | Indexed |
| Annual Revenue | Number | Indexed |

---

### Table Name: Business Types

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Industry ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Type | Text | Indexed |
| Contact Template Content | Text |  |
| Contact Template Subject | Text | Indexed |

---

### Table Name: Businesses

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Industry ID | Text | Indexed |
| Business Type ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Description | Text | Indexed |
| Logo URL | Text | Indexed |
| Logo Icon URL | Text | Indexed |
| Menu Mode | Text | Indexed |
| Registration Number | Text | Indexed |
| Registration Year | Text | Indexed |
| Location | Text | Indexed |
| City | Text | Indexed |
| Region | Text | Indexed |
| Phone | Text | Indexed |
| Email | Text | Indexed |
| Contacts | Text | Indexed |
| Website | Text | Indexed |
| Total Staff | Text | Indexed |
| Total Branches | Text | Indexed |
| Building Size | Text | Indexed |
| Values | Text | Indexed |
| Mission Statement | Text | Indexed |
| What They Should Be Looking Like | Text | Indexed |
| Brand Colours | Text | Indexed |
| Preferred Currency | Text | Indexed |
| Staff Code Prefix 1 | Text | Indexed |
| Staff Code Prefix 2 | Text | Indexed |
| Current Status | Text | Indexed |
| --- |  |  |
| Proposed Solution Name | Text | Indexed |
| Proposed Solution Description | Text | Indexed |
| Proposed Solution Technologies | Text | Indexed |
| Proposed Solution Hosting | Text | Indexed |
| Proposed Solution Integrations | Text | Indexed |
| Proposed Solution Cost | Text | Indexed |
| SRD Prepared For | Text | Indexed |
| --- |  |  |
| App Menu Mode | Text | Indexed |
| Notes | Text |  |
| --- |  |  |
| Status | Text | Indexed |

---

### Table Name: Activities Regarding Businesses

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Business ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Date | Date | Indexed |
| Time | Time | Indexed |
| Activity | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed |

---

### Table Name: Business Operational Challenges

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Business ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Challenge | Text | Indexed |
| Current Method | Text | Indexed |
| Recommended Solution | Text | Indexed |
| --- |  |  |
| Order | Number | Indexed |

---

### Table Name: Business Solution Features

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Business ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Description | Text | Indexed |
| Image URL | Text | Indexed |
| --- |  |  |
| Order | Number | Indexed |

---

### Table Name: Software Requirement Documents

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Business ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| App Description | Text | Indexed |
| Cover Image URL | Text | Indexed |
| Table of Contents 1 | Text |  |
| Table of Contents 2 | Text |  |
| Table of Contents 3 | Text |  |
| Table of Contents Bold Text | Text | Indexed |
| --- |  |  |
| Introduction_Image URL | Text | Indexed |
| Introduction_Statement | Text | Indexed |
| Introduction_Background | Text |  |
| Introduction_Purpose of System | Text |  |
| Introduction_About Caufero | Text |  |
| --- |  |  |
| Business Context_Image URL | Text | Indexed |
| Business Context_Statement | Text | Indexed |
| Business Context_Company Operations | Text |  |
| Business Context_Need for Digital Transformation | Text |  |
| --- |  |  |
| System Overview_Image URL | Text | Indexed |
| System Overview_Statement | Text | Indexed |
| System Overview_Core Structure_N_Components | Text |  |
| System Overview_Design_N_Operational Flow | Text |  |
| --- |  |  |
| Functional Requirements_Image URL | Text | Indexed |
| Functional Requirements_Statement | Text | Indexed |
| --- |  |  |
| Non_Functional Requirements_Image URL | Text | Indexed |
| Non_Functional Requirements_Statement | Text | Indexed |
| Non_Functional Requirements_Performance | Text |  |
| Non_Functional Requirements_Reliability | Text |  |
| Non_Functional Requirements_Security | Text |  |
| Non_Functional Requirements_Usability | Text |  |
| Non_Functional Requirements_Maintainability | Text |  |
| Non_Functional Requirements_Portability | Text |  |
| --- |  |  |
| UI Guidelines_Image URL | Text | Indexed |
| UI Guidelines_Statement | Text | Indexed |
| UI Guidelines_Layout | Text |  |
| UI Guidelines_Navigation | Text |  |
| UI Guidelines_Role Focus | Text |  |
| UI Guidelines_Responsiveness | Text |  |
| UI Guidelines_Feedback | Text |  |
| UI Guidelines_Visual Design | Text |  |
| --- |  |  |
| Development Process_Image URL | Text | Indexed |
| Development Process_Statement | Text | Indexed |
| Development Process | Text |  |
| --- |  |  |
| Suggested Improvements_Image URL | Text | Indexed |
| Suggested Improvements_Statement | Text | Indexed |
| Suggested Improvements_Operational Enhancements | Text |  |
| Suggested Improvements_Customization_N_Reporting | Text |  |
| --- |  |  |
| Phase Prioritization_Image URL | Text | Indexed |
| Phase Prioritization_Statement | Text | Indexed |
| Phase Prioritization_Phase1 | Text |  |
| Phase Prioritization_Phase2 | Text |  |
| --- |  |  |
| Future Enhancements_Image URL | Text | Indexed |
| Future Enhancements_Statement | Text | Indexed |
| Future Enhancements_Integration_N_Automation | Text |  |
| Future Enhancements_Scalability_N_Customization | Text |  |
| --- |  |  |
| Cost_N_Timelines_Image URL | Text | Indexed |
| Cost_N_Timelines_Statement | Text | Indexed |
| Cost_N_Timelines_Rate | Number | Indexed |
| Cost_N_Timelines_Hours | Number | Indexed |
| Cost_N_Timelines_Duration | Number | Indexed |
| Cost_N_Timelines_Cost | Number | Indexed |
| Cost_N_Timelines_Support | Number | Indexed |
| --- |  |  |
| Pricing_N_Support | Text |  |
| --- |  |  |
| Payment Terms_Image URL | Text | Indexed |
| Payment Terms_Statement | Text | Indexed |
| Payment Terms | Text |  |
| --- |  |  |
| Customization Options_Image URL | Text | Indexed |
| Customization Options_Statement | Text | Indexed |
| Customization Options | Text |  |
| --- |  |  |
| Next Steps_Image URL | Text | Indexed |
| Next Steps_Statement | Text | Indexed |
| Next Steps | Text |  |

---

### Table Name: Software Requirement Documents _ Functional Requirements

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Business ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Order | Number | Indexed |
| Requirement | Text | Indexed |
| Features | Text | Indexed |

---

### Table Name: Software Requirement Documents _ Price Breakdown

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Business ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Order | Number | Indexed |
| Item Type | Text | Indexed |
| Item | Text | Indexed |
| Duration | Number | Indexed |
| Hours | Number | Indexed |
| Hours Per Week | Number | Indexed |
| Rate | Number | Indexed |
| Cost | Number | Indexed |

---

### Table Name: Specimens

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |

---

### Table Name: Analytes

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Result Type | Text | Indexed |
| Unit | Text | Indexed |
| Value List Options | Text | Indexed |

---

### Table Name: Lab Tests

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Description | Text | Indexed |
| Cost | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed |

---

### Table Name: Lab Test Specimens

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Lab Test ID | Text | Indexed |
| Specimen ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |

---

### Table Name: Lab Test Specimen Analytes

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Lab Test Specimen ID | Text | Indexed |
| Analyte ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |

---

### Table Name: Insurers

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Type | Text | Indexed |
| Portal URL | Text | Indexed |
| Phone | Text | Indexed |
| Email | Text | Indexed |

---

### Table Name: Insurance Policies

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Insurer ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |

---

### Table Name: Medication Dosages

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Medication ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Dosage | Text | Indexed |

---

### Table Name: Patients

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| Month Created | Text | Indexed |
| Month Created Number | Number | Indexed |
| --- |  |  |
| Number | Text | Indexed |
| Title | Text | Indexed |
| First Name | Text | Indexed |
| Last Name | Text | Indexed |
| Other Names | Text | Indexed |
| Initials | Text | Indexed |
| Gender | Text | Indexed |
| Date of Birth | Date | Indexed |
| Age | Number | Indexed |
| Mobile | Text | Indexed |
| Email | Text | Indexed |
| Street | Text | Indexed |
| City | Text | Indexed |
| Country | Text | Indexed, Auto-enter Data |
| Marital Status | Text | Indexed |
| Maiden Name | Text | Indexed |
| Hometown | Text | Indexed |
| Hometown Region | Text | Indexed |
| Nationality | Text | Indexed, Auto-enter Data |
| ID Card Type | Text | Indexed |
| ID Card Number | Text | Indexed |
| Languages | Text | Indexed |
| Profession | Text | Indexed |
| Religion | Text | Indexed, Auto-enter Data |

---

### Table Name: Patient Emergency Contacts

| Field | Type | Options |
|---|---|---|
| ID | Text | Auto-enter Calculation |
| Patient ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Relationship | Text | Indexed |
| Full Name | Text | Indexed |
| Mobile | Text | Indexed |

---

### Table Name: Patient Primary Care Physicians

| Field | Type | Options |
|---|---|---|
| ID | Text | Auto-enter Calculation |
| Patient ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Full Name | Text | Indexed |
| Mobile | Text | Indexed |
| Residential Address | Text | Indexed |

---

### Table Name: Patient Payment Options

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Option | Text | Indexed |
| --- |  |  |
| MoMo Network | Text | Indexed |
| MoMo Number | Text | Indexed |
| --- |  |  |
| Insurer ID | Text | Indexed |
| Insurance Policy ID | Text | Indexed |
| Insurance Policy Number | Text | Indexed |

---

### Table Name: Patient Appointments

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Doctor User ID | Text | Indexed |
| Bill Payment Option ID | Text | Indexed |
| Parent ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Date Scheduled | Date | Indexed |
| Time Scheduled | Time | Indexed |
| Reason | Text | Indexed |
| --- |  |  |
| Doctors Notes | Text |  |
| --- |  |  |
| Vitals Status | Text | Indexed, Auto-enter Data |
| Consultation Status | Text | Indexed, Auto-enter Data |
| Consultation Billed Status | Text | Indexed, Auto-enter Data |
| Consultation Payment Status | Text | Indexed, Auto-enter Data |
| --- |  |  |
| Next Appointment Period Number | Text | Indexed |
| Next Appointment Period Name | Text | Indexed |
| --- |  |  |
| Admission Status | Text | Indexed |
| Admission Date | Date | Indexed |
| Admission Time | Time | Indexed |
| Discharge Date | Date | Indexed |
| Discharge Time | Time | Indexed |
| --- |  |  |
| Discount | Number | Indexed |

---

### Table Name: Patient Visit Vitals

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Height | Text | Indexed |
| Weight | Text | Indexed |
| BMI | Text | Indexed |
| Temperature | Text | Indexed |
| Blood Pressure Top | Text | Indexed |
| Blood Pressure Bottom | Text | Indexed |
| Pulse | Text | Indexed |
| Respiratory Rate | Text | Indexed |
| Oxygen Saturation | Text | Indexed |
| Glucose Level | Text | Indexed |

---

### Table Name: Patient Visit Lab Tests

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| Lab Test ID | Text | Indexed |
| Lab Test Specimen ID | Text | Indexed |
| Doctor User ID | Text | Indexed |
| Lab Scientist User ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Instructions | Text |  |
| --- |  |  |
| Date Submitted | Date | Indexed |
| Time Submitted | Time | Indexed |
| --- |  |  |
| Results | Text | Indexed |
| Billed Status | Text | Indexed |

---

### Table Name: Patient Visit Lab Test Analyte Results

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| Lab Test ID | Text | Indexed |
| Lab Test Specimen Analyte ID | Text | Indexed |
| Patient Visit Lab Test ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Result | Text |  |

---

### Table Name: Patient Visit Lab Results Documents

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| Lab Test ID | Text | Indexed |
| Patient Visit Lab Test ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| Date Modified | Date | Indexed |
| Time Modified | Time | Indexed |
| --- |  |  |
| Uploaded By | Text | Indexed |
| Type | Text | Indexed |
| Number | Text | Indexed |
| Name | Text | Indexed |
| File Extension | Text | Indexed |

---

### Table Name: Patient Visit Diagnoses

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| Medical Condition ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Remarks | Text |  |

---

### Table Name: Patient Visit Prescriptions

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| Medication ID | Text | Indexed |
| Medication Dispensed ID | Text | Indexed |
| Doctor User ID | Text | Indexed |
| Pharmacist User ID | Text | Indexed |
| Dosage ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Medication Name | Text | Indexed |
| Medication Dispensed Name | Text | Indexed |
| Quantity | Text | Indexed, Auto-enter Data |
| Quantity Number | Number | Indexed, Auto-enter Data |
| Dosage | Text | Indexed |
| Dosage For Dispensed Medication | Text | Indexed |
| --- |  |  |
| Dispensary Status | Text | Indexed, Auto-enter Data |
| Billed Status | Text | Indexed |

---

### Table Name: Patient Visit Services

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| Service Item ID | Text | Indexed |
| Doctor User ID | Text | Indexed |
| Nurse User ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Service Name | Text | Indexed |
| Quantity | Text | Indexed, Auto-enter Data |
| Quantity Number | Number | Indexed, Auto-enter Data |
| Notes | Text | Indexed |
| --- |  |  |
| Dispensary Status | Text | Indexed, Auto-enter Data |
| Billed Status | Text | Indexed |

---

### Table Name: Patient Visit Treatment Administrations

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| Patient Visit Prescription ID | Text | Indexed |
| Supervisor User ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Date Administered | Date | Indexed |
| Time Administered | Time | Indexed |
| Timestamp Administered | Timestamp | Indexed |
| --- |  |  |
| Quantity | Text | Indexed |
| Quantity Number | Number | Indexed |

---

### Table Name: Patient Visit Consultation Notes

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| Doctor User ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Notes | Text | Indexed |

---

### Table Name: InPatient Notes

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| User ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Type | Text | Indexed |
| Notes | Text |  |

---

### Table Name: InPatient Vitals

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| User ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Height | Text | Indexed |
| Weight | Text | Indexed |
| BMI | Text | Indexed |
| Temperature | Text | Indexed |
| Blood Pressure Top | Text | Indexed |
| Blood Pressure Bottom | Text | Indexed |
| Pulse | Text | Indexed |
| Respiratory Rate | Text | Indexed |
| Oxygen Saturation | Text | Indexed |
| Glucose Level | Text | Indexed |

---

### Table Name: Walk_Ins

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| User ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| Month Created | Text | Indexed |
| Month Created Number | Number | Indexed |
| --- |  |  |
| Patient ID | Text | Indexed |
| Fullname | Text | Indexed |
| Mobile | Text | Indexed |
| Email | Text | Indexed |
| --- |  |  |
| Items | Text | Indexed |
| Bill Amount | Number | Indexed |
| Paid Amount | Number | Indexed |

---

### Table Name: Patient Visit Bill Items

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Patient ID | Text | Indexed |
| Patient Appointment ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| Month Created | Text | Indexed |
| Month Created Number | Number | Indexed |
| --- |  |  |
| Item ID | Text | Indexed |
| Cost | Number | Indexed |
| Quantity | Number | Indexed |
| Unit Cost | Number | Indexed |
| Paid Status | Text | Indexed, Auto-enter Data |
| Discount Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Projects

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Business ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Title | Text | Indexed |
| Table of Contents 1 | Text | Indexed |
| Table of Contents 2 | Text | Indexed |
| Table of Contents 3 | Text | Indexed |
| --- |  |  |
| Introduction Statement | Text | Indexed |
| Background | Text | Indexed |
| Purpose | Text | Indexed |
| --- |  |  |
| Business Context Statement | Text | Indexed |
| Operations | Text | Indexed |
| Need for Digitalisation | Text | Indexed |
| System Overview | Text | Indexed |
| Structure | Text | Indexed |
| Workflow | Text | Indexed |
| --- |  |  |
| Functional Requirements Statement | Text | Indexed |
| Functional Requirement 1 Title | Text | Indexed |
| Functional Requirement 1 Details | Text | Indexed |
| Functional Requirement 2 Title | Text | Indexed |
| Functional Requirement 2 Details | Text | Indexed |
| Functional Requirement 3 Title | Text | Indexed |
| Functional Requirement 3 Details | Text | Indexed |
| Functional Requirement 4 Title | Text | Indexed |
| Functional Requirement 4 Details | Text | Indexed |
| --- |  |  |
| NonFunctional Requirements Statement | Text | Indexed |
| Performance | Text | Indexed |
| Reliability | Text | Indexed |
| Security | Text | Indexed |
| Usability | Text | Indexed |
| Maintainability | Text | Indexed |
| Portability | Text | Indexed |
| --- |  |  |
| UI Guidelines Statement | Text | Indexed |
| Layout | Text | Indexed |
| Navigation | Text | Indexed |
| Role Focus | Text | Indexed |
| Responsiveness | Text | Indexed |
| Feedback | Text | Indexed |
| Visual Design | Text | Indexed |
| --- |  |  |
| Improvements Statement | Text | Indexed |
| Operational Enhancements | Text | Indexed |
| Customisation N Reporting | Text | Indexed |
| --- |  |  |
| Phases Statement | Text | Indexed |
| Phase 1 | Text | Indexed |
| Phase 2 | Text | Indexed |
| --- |  |  |
| Future Enhancements Statement | Text | Indexed |
| Integration N Automation | Text | Indexed |
| Scalability N Customisation | Text | Indexed |
| --- |  |  |
| Cost N Timeline Statement | Text | Indexed |

---

### Table Name: Material Categories

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Category | Text | Indexed |
| App Name | Text | Indexed |

---

### Table Name: Materials

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Category ID | Text | Indexed |
| Vendor ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Translarity Number | Text | Indexed |
| Vendor Number | Text | Indexed |
| Image | Container | External (Secure) |
| Name | Text | Indexed |
| AKA | Text | Indexed |
| Manufacturer | Text | Indexed |
| Description | Text |  |
| Issue Unit | Text | Indexed |
| Purchase Unit | Text | Indexed |
| Barcode Contents | Text | Indexed |
| Reorder Level | Text | Indexed |
| Minimum Order Quantity | Text | Indexed |
| Quantity In Order Unit | Text | Indexed |
| Unit Cost | Text | Indexed |
| URL | Text | Indexed |
| Quantity Available | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Stock Adjustments

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Material ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Stock Date | Date | Indexed |
| Stock Time | Time | Indexed |
| --- |  |  |
| Quantity | Text | Indexed |
| Quantity Number | Number | Indexed |
| Unit | Text | Indexed |

---

### Table Name: Equipment Categories

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Category | Text | Indexed |

---

### Table Name: Equipment

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Category ID | Text | Indexed |
| Supplier Vendor ID | Text | Indexed |
| Servicing Vendor ID | Text | Indexed |
| Location ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Translarity Number | Text | Indexed |
| Vendor Number | Text | Indexed |
| Image URL | Text | Indexed |
| Name | Text | Indexed |
| AKA | Text | Indexed |
| Manufacturer | Text | Indexed |
| Description | Text |  |
| Barcode Contents | Text | Indexed |
| Purchased Date | Date | Indexed |
| Warranty _Number | Text | Indexed |
| Warranty _Period | Text | Indexed |
| Warranty Expiration | Date | Indexed |
| Length | Text | Indexed |
| Width | Text | Indexed |
| Height | Text | Indexed |
| Size Unit | Text | Indexed |
| Workspace Length | Text | Indexed |
| Workspace Width | Text | Indexed |
| Workspace Size Unit | Text | Indexed |
| Weight _Number | Text | Indexed |
| Weight _Unit | Text | Indexed |
| Cost | Number | Indexed |
| --- |  |  |
| Notes | Text |  |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Operating Conditions

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Condition | Text | Indexed |
| Unit | Text | Indexed |

---

### Table Name: Equipment Optimal Operating Conditions

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Equipment ID | Text | Indexed |
| Operating Condition ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Optimal Value | Text | Indexed |

---

### Table Name: Equipment Related Parts

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Equipment ID | Text | Indexed |
| Supplier Vendor ID | Text | Indexed |
| Servicing Vendor ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| AKA | Text | Indexed |
| Description | Text |  |
| Quantity | Text | Indexed |
| Cost | Text | Indexed |
| Translarity Number | Text | Indexed |
| Vendor Number | Text | Indexed |
| Manufacturer | Text | Indexed |
| Barcode Contents | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Equipment Maintenance

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Equipment ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Date Scheduled | Date | Indexed |
| Date Completed | Date | Indexed |
| Request | Text |  |
| Repair Summary | Text |  |
| Parts Used | Text |  |
| Hours | Text | Indexed |
| Reference Number | Text | Indexed |

---

### Table Name: Equipment Decommissioning

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Equipment ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Date Decommissioned | Date | Indexed |
| Decommissioning Type | Text | Indexed |
| Reason | Text |  |
| Current Value | Text | Indexed |
| Delta Value | Text | Indexed |

---

### Table Name: Equipment Documents

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Equipment ID | Text | Indexed |
| Uploaded By | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| Date Modified | Date | Indexed |
| Time Modified | Time | Indexed |
| --- |  |  |
| File Extension | Text | Indexed |
| Name | Text | Indexed |

---

### Table Name: Vendors

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Location | Text | Indexed |
| Description | Text |  |
| Contact Name | Text | Indexed |
| Contact Phone | Text | Indexed |
| Contact Email | Text | Indexed |
| Preferred Currency | Text | Indexed |
| Payment Term | Text | Indexed |
| Shipping Method | Text | Indexed |
| Lead Time | Number | Indexed |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Locations

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Products

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Code | Text | Indexed |
| Description | Text |  |
| Domain | Text | Indexed |
| Product URL | Text | Indexed |
| --- |  |  |
| Feature Post Image URL | Text | Indexed |
| Feature Post Title | Text | Indexed |
| Feature Post Content | Text |  |
| Feature Post About | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Product Features

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Product ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Image URL | Text | Indexed |
| Name | Text | Indexed |
| Order | Number | Indexed |
| Description | Text |  |

---

### Table Name: Maintenance Schedule Sheets

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Created By | Text | Indexed |
| Modified By | Text | Indexed |
| --- |  |  |
| Name | Text | Indexed |
| Status | Text | Indexed |

---

### Table Name: Maintenance Schedules

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Maintenance Schedule Sheet ID | Text | Indexed |
| Equipment ID | Text | Indexed |
| Assigned To Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Title | Text | Indexed |
| Start Date | Date | Indexed |
| Previous Date | Date | Indexed |
| Next Date | Date | Indexed |
| Frequency Number | Number | Indexed |
| Frequency Period | Text | Indexed |
| Weekly Repeat Days | Text | Indexed |
| Estimated Number of Hours | Text | Indexed |
| Labour Cost | Text | Indexed |

---

### Table Name: Maintenance Activities

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Maintenance Schedule ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Activity | Text | Indexed |

---

### Table Name: Maintenance Materials

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Maintenance Activity ID | Text | Indexed |
| Material ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |

---

### Table Name: Maintenance Labour

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Maintenance Activity ID | Text | Indexed |
| User ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |

---

### Table Name: Purchase Orders

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Maker ID | Text | Indexed |
| Checker ID | Text | Indexed |
| Vendor ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Date Decided On | Date | Indexed |
| Time Decided On | Time | Indexed |
| Month Decided On | Text | Indexed |
| Month Decided On Number | Number | Indexed |
| --- |  |  |
| Number | Text | Indexed |
| Project | Text | Indexed |
| Customer | Text | Indexed |
| Department | Text | Indexed |
| Description | Text |  |
| Notes | Text |  |
| Total Amount | Number | Indexed |
| Payment Term | Text | Indexed |
| Lead Time | Number | Indexed |
| Ship To Location | Text | Indexed |
| Referencing SO | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |
| Received Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Purchase Order Items

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Purchase Order ID | Text | Indexed |
| Material ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Material Name | Text | Indexed |
| Quantity | Number | Indexed |
| Unit Price | Number | Indexed |
| Total Price | Number | Indexed |
| --- |  |  |
| Date of Expected Delivery | Date | Indexed |
| Date of Actual Delivery | Date | Indexed |
| --- |  |  |
| URL | Text | Indexed |

---

### Table Name: Payroll Items

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Item | Text | Indexed |
| Type | Text | Indexed |
| Payment Model | Text | Indexed |
| Service | Text | Indexed |

---

### Table Name: Payroll Items For Job Title Notches

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Job Title ID | Text | Indexed |
| Notch ID | Text | Indexed |
| Payroll Item ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Amount | Number | Indexed |

---

### Table Name: Payment Groups

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |

---

### Table Name: Payment Groups of Payroll Items

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Payment Group ID | Text | Indexed |
| Payroll Item ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |

---

### Table Name: Payroll Periods

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Payment Group ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Start Date | Date | Indexed |
| End Date | Date | Indexed |
| Payment Date | Date | Indexed |
| Year | Number | Indexed |
| Total Staff Paid | Number | Indexed |
| Total Amount Paid | Number | Indexed |

---

### Table Name: Remunerations

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Payroll Period ID | Text | Indexed |
| Staff ID | Text | Indexed |
| Job Title ID | Text | Indexed |
| Notch ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Earnings | Number | Indexed |
| Deductions | Number | Indexed |
| Net Pay | Number | Indexed |
| --- |  |  |
| Pay Status | Text | Indexed, Auto-enter Data |

---

### Table Name: Remuneration Details

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| Payroll Period ID | Text | Indexed |
| Remuneration ID | Text | Indexed |
| Payroll Item ID | Text | Indexed |
| Staff ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Payroll Item Name | Text | Indexed |
| Payroll Item Type | Text | Indexed |
| Rate | Text | Indexed, Auto-enter Data |
| Quantity | Text | Indexed, Auto-enter Data |
| Amount | Number | Indexed |
| Amount _YTD | Number | Indexed |
| --- |  |  |
| Year | Number | Indexed |

---

### Table Name: Other Billing Items

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Name | Text | Indexed |
| Description | Text | Indexed |
| Category | Text | Indexed |
| Cost | Text | Indexed |
| --- |  |  |
| Status | Text | Indexed, Auto-enter Data |

---

### Table Name: SMS Account

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Sender Name | Text | Indexed |
| --- |  |  |
| Total Credits | Number | Indexed |
| Used Credits | Number | Indexed |
| Available Credits | Number | Indexed |

---

### Table Name: SMS Credits

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| SMS Account ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| Total | Number | Indexed |

---

### Table Name: Alerts

| Field | Type | Options |
|---|---|---|
| ID | Text | Indexed, Auto-enter Calculation |
| User ID | Text | Indexed |
| Reference ID | Text | Indexed |
| Link ID | Text | Indexed |
| --- |  |  |
| Date Created | Date | Indexed, Creation Date |
| Time Created | Time | Indexed, Creation Time |
| --- |  |  |
| App Section | Text | Indexed |
| Tab | Text | Indexed |
| Message | Text | Indexed |
| Date Scheduled | Date | Indexed |
| Time Scheduled | Time | Indexed |
| --- |  |  |
| Status | Text | Indexed |

---

# Custom Functions (CauferoAppStarter)

## Custom Functions Preamble

These custom functions are a shared utility layer for CauferoAppStarter. They exist to keep scripts short, consistent, and predictable by centralizing repeated logic (list handling, text cleanup, date/time formatting, ID generation, SQL helpers, HTML and JS helpers, validation, and encoding).

This document is also training material for an AI agent that builds FileMaker applications in the same style. The agent should:
- Use existing custom functions whenever they solve the problem cleanly.
- Suggest new custom functions when a pattern repeats, when a script becomes noisy, or when a rule needs to be enforced consistently across the file.
- Create new functions using the same conventions and standards in this library.

### Core Rules For Using Custom Functions
- Prefer calling a custom function over rewriting the same logic in multiple scripts.
- Treat each custom function as a stable API. Avoid changes that break existing scripts.
- If behavior must evolve, prefer creating a new function (or a clearly named v2 variant) instead of changing outputs silently.
- Keep functions deterministic and side-effect free. A custom function should return a value only, with no dependency on UI state.
- Handle empty inputs gracefully. When possible, return an empty result or a sensible default instead of raising errors.
- Keep outputs predictable, with a clearly stated return type (Text, Number, Date, Timestamp, JSON as Text).

### Conventions For Creating New Custom Functions
**Naming**
- Use PascalCase for function names.
- Use clear action names for transformers (Add, Remove, Convert, Escape, Format, Generate, Validate, Get).
- Use noun-based names only for pure getters or helpers that return a single concept (Example: `GetDevice`).

**Parameter Naming**
- Use descriptive names aligned with the existing library.
- Use a trailing underscore for generic or commonly reused parameters: `value_`, `string_`, `Text_`.
- Use a leading underscore when you want to visually separate a “raw input value” from processed values: `_Date`, `_Time`.
- Keep parameter order consistent with the existing patterns:
  - Primary input first (the thing being modified or evaluated)
  - Options next (format, precision, separators, limits)
  - Secondary inputs last (reference values, previous codes)

**Delimiters and Data Shapes**
- Default list shape is return-delimited text (`¶`) unless the function explicitly states otherwise.
- If a function encodes or decodes a delimiter (Example: `*****` or `|`), document it clearly and keep it consistent.
- For JSON-producing helpers, keep output keys stable and documented (Example: `data`, `records`, `error`, `message`).

**Dependencies**
- If a new function relies on other custom functions, list them under Notes as dependencies.
- Prefer using existing helpers (Example: `ValueExists`, `RemoveValues`, `EscapeDoubleQuotes`) instead of re-implementing similar logic.

### Documentation Standard For Each Function
Every custom function entry should include:
- **Purpose**: one clear sentence describing what it does.
- **Parameters**: each parameter described with expected type and delimiter shape if relevant.
- **Returns**: type plus a short description of the output.
- **Example**: input and output with realistic values.
- **Notes**: edge cases, assumptions, dependencies, and any delimiter constraints.

### When The AI Agent Should Suggest A New Function
Suggest a new custom function when:
- The same calculation or transformation appears in 2 or more scripts.
- A script needs repeated “cleanup” steps (text normalization, escaping, trimming, list manipulation).
- A consistent formatting rule is needed across layouts, reports, or WebViewer output.
- A script needs a reusable mapping table (statuses, labels, units, platform detection).
- A script builds JSON or parses SQL results in a repeated pattern.

When suggesting a new function, the agent should provide:
- Proposed **Name** and **Signature**
- Target **Category** (List, Text, DateTime, Formatting, SQL, HTML/JS, Validation, Utility)
- **Purpose**, **Returns**, **Example**
- Compatibility notes, especially if replacing an older pattern

<br/><br/>

---

## Current Custom Functions in CauferoAppStarter

- `AddValue`(ValueList; Value_)
- `RemoveValue`(ValueList; Value_)
- `RemoveValues`(ValueList; ValuesToRemove)
- `AddOrRemoveValue`(value_; content)
- `ValueExists`(ValueList; Value_)
- `RemoveEmptyValues`(Text)
- `MaxListValue`(~list)

---

- `EncodeListAsScriptParameters`(list_)
- `DecodeListAsScriptParameters`(list_)
- `GetScriptParameters`()
- `ConvertPipeDelimitedToReturnDelimited`(String)

---

- `DefaultIfEmpty`(string; replaceWith)
- `PadLeft`(value_; character; stringlength)
- `RemoveTrailingComma`(Text)
- `RemoveSpecialCharacters`(String)

---

- `EscapeDoubleQuotes`(String)
- `EscapeSingleQuoteInHTML`(String)
- `DecodeURIComponent`(EncodedText)

---

- `ConvertReturnsToBR`(string_)
- `ConvertLineBreakMarkersToReturns`(String)
- `JSNormalizeTextChain`()
- `HTMLToTextStripTags`(text)

---

- `FormatNumber`(Number; DecimalPrecision; ThousandSeparator)
- `AbbreviateNumber`(number; precision)
- `Ordinal`(number)

---

- `ShortenString`(string; totalCharacters)
- `Ellipsis`(Text_; MaxTextLength_)
- `Singular`(string_)
- `PluralizeText`(text; quantity)
- `PluralizeTerm`(term; quantity)
- `AOrAn`(text_)

---

- `FullNameTitleFirstMiddleLast`(title; first_name; middle_name; last_name)
- `FullNameTitleLastFirstMiddle`(title; first_name; middle_name; last_name)
- `FullNameLastFirstMiddleTitle`(title; first_name; middle_name; last_name)
- `FullNameFirstMiddleLast`(first_name; middle_name; last_name)
- `FullNameFirstLast`(first_name; last_name)
- `AbbreviateText`(text)
- `AbbreviateTextCustom`(text; spaces; hyphens; periods)

---

- `DateInWordsShortMonth`(_Date)
- `DateInWordsShortMonthNoYear`(_Date)
- `DateInWordsShortMonthNDay`(_Date)
- `DateInWordsShortMonthNDayNoYear`(_Date)
- `DateInWordsMonthNYear`(_Date)
- `DateInWordsLongMonth`(_Date)
- `RelativeDateInWords`(_Date)

---

- `MySQLDateToFMPDateText`(MySQL_Date)
- `MySQLTimestampToFMPTimestampText`(MySQL_Timestamp)
- `FMPDateTextToMySQLDate`(FMP_Date)
- `DateInShortWordsToFMPDate`(inputDate)

---

- `MinutesToTextTime`(Minutes)
- `Elapsed`(StartDate; EndDate)
- `ElapsedTime`(TStampBegin; TStampEnd; Format)
- `TimeNoSeconds`(_Time)
- `FormatTimeHHMM12`(theTime)
- `RelativeElapsedPeriodLong`(Timestamp1; Timestamp2)
- `RelativeElapsedPeriodShort`(Timestamp1; Timestamp2)
- `RelativePeriodShort`(Timestamp1; Timestamp2)
- `Age`(DOB)
- `WeekdayCount`(StartDate; EndDate)
- `MondayOfGivenWeek`(week_; year_)

---

- `AddDaysToDate`(Date_; DaysToAdd_)
- `AddWeeksToDate`(Date_; WeeksToAdd_)
- `AddMonthsToDate`(Date_; MonthsToAdd_)
- `AddYearsToDate`(Date_; YearsToAdd_)
- `AddPeriodToDate`(Date_; NumberToAdd_; PeriodTypeToAdd_)

---

- `RandomInteger`(start; stop)
- `GeneratePassword`(Len; Letters; Numbers; Symbols)
- `GenerateNextRecordCode`(Alphabets; NumericLength; LastSerial)
- `GenerateNextCode`(prefix; serialLength; previousCode)

---

- `GetDevice`()
- `GetExtension`(Some Filename)
- `GetFileSize`(Bytes; Precision; UOM; Format)
- `GenerateFileIconHTML`(Extension)
- `RecordStatus`(status)

---

- `Pronoun`(Gender; Type)

---

- `ExecuteSQLResultToJSON`(sqlResult; fieldSub; rowSub; fieldList; limit)
- `SQLFieldNames`(_sqlText)
- `IsValidEmailFormat`(_email)

---

- `Base64EncodeFile`(file)

<br/><br/>

---
## Details of Each Custom Function in CauferoAppStarter


## AddValue ( ValueList ; Value_ )

### Purpose
Appends a value to the end of a return-delimited list.

### Parameters
- **ValueList**  
  A return-delimited list of values.
- **Value_**  
  The value to append.

### Returns
- Text: the updated list (return-delimited).

### Example
**Input**  
ValueList = Red¶Blue  
Value_ = Green  

**Output**  
Red¶Blue¶Green

### Notes
- If `ValueList` is empty, the result is just `Value_` (no leading return).

---

## RemoveValue ( ValueList ; Value_ )

### Purpose
Removes all exact matches of a value from a return-delimited list.

### Parameters
- **ValueList**  
  A return-delimited list of values.
- **Value_**  
  The exact value to remove (exact match).

### Returns
- Text: the updated list (return-delimited), with the matching value removed.

### Example
**Input**  
ValueList = Red¶Blue¶Green¶Blue  
Value_ = Blue  

**Output**  
Red¶Green

### Notes
- Matching is exact (case and spacing sensitive).
- Implemented using recursion (processes the list from first value to last).

---

## RemoveValues ( ValueList ; ValuesToRemove )

### Purpose
Removes multiple values from a return-delimited list.

### Parameters
- **ValueList**  
  A return-delimited list of values.
- **ValuesToRemove**  
  A return-delimited list of values to remove.

### Returns
- Text: the updated list (return-delimited), with the specified values removed.

### Example
**Input**  
ValueList = Red¶Blue¶Green¶Blue Sky¶Blue  
ValuesToRemove = Blue¶Green  

**Output**  
Red¶Blue Sky

### Notes
- Removes whole list items only (does not remove partial text inside an item).
- Blank rows in `ValuesToRemove` are ignored.
- Implemented using recursion (processes ValuesToRemove one value at a time).

---

## AddOrRemoveValue ( value_ ; content )

### Purpose
Toggles a value in a return-delimited list.
- If the value exists, it is removed.
- If the value does not exist, it is added to the end.

### Parameters
- **value_**  
  The value to toggle.
- **content**  
  A return-delimited list of values.

### Returns
- Text: the updated list (return-delimited).

### Example
**Input**  
content = Red¶Blue¶Green  
value_ = Blue  

**Output**  
Red¶Green

### Notes
- Depends on these helper functions:
  - `ValueExists ( content ; value_ )`
  - `RemoveValues ( content ; value_ )`
  - `AddValue ( content ; value_ )`

---

## ValueExists ( ValueList ; Value_ )

### Purpose
Checks whether a value exists in a return-delimited list (exact match).

### Parameters
- **ValueList**  
  A return-delimited list of values.
- **Value_**  
  The value to check for (exact match).

### Returns
- Number: `1` if the value exists.
- Empty if the value does not exist (acts like false).

### Example
**Input**  
ValueList = Red¶Blue¶Green  
Value_ = Blue  

**Output**  
1

### Notes
- Matching is exact (case and spacing sensitive).
- Implemented using recursion (scans the list from first value to last).

---

## RemoveEmptyValues ( Text )

### Purpose
Removes empty lines from return-delimited text.
Also trims leading and trailing returns.

### Parameters
- **Text**  
  Return-delimited text that may contain blank rows.

### Returns
- Text: a cleaned return-delimited list with no empty rows.

### Example
**Input (Text)**  
A¶¶¶B¶C¶

**Output**  
A¶B¶C

### Notes
- Collapses repeated returns (¶¶) into a single return.
- Trims leading and trailing returns.
- Uses recursion until the text is fully cleaned.

---

## MaxListValue ( ~list )

### Purpose
Returns the maximum value from a return-delimited list.

### Parameters
- **~list**  
  A return-delimited list of values (typically numbers).

### Returns
- Number (or Text): the largest value in the list.

### Example
**Input (~list)**  
1¶2¶3

**Output**  
3

### Notes
- Uses a `While` loop to scan the entire list.
- Best used with clean numeric values for predictable results.

---

## EncodeListAsScriptParameters ( list_ )

### Purpose
Encodes a return-delimited list into a single string by replacing each return (¶) with a marker (`*****`).  
Useful for passing lists as script parameters.

### Parameters
- **list_**  
  A return-delimited list.

### Returns
- Text: the encoded string with `*****` used as the delimiter.

### Example
**Input (list_)**  
Red¶Blue¶Green

**Output**  
Red*****Blue*****Green

### Notes
- Intended to be paired with `DecodeListAsScriptParameters ( list_ )`.
- Assumes your list values do not contain the literal text `*****` (or it will be ambiguous).

---

## DecodeListAsScriptParameters ( list_ )

### Purpose
Decodes a string back into a return-delimited list by replacing the marker (`*****`) with returns (¶).  
Typically used to restore lists passed through script parameters.

### Parameters
- **list_**  
  A string that uses `*****` as the delimiter.

### Returns
- Text: a return-delimited list.

### Example
**Input (list_)**  
Red*****Blue*****Green

**Output**  
Red¶Blue¶Green

### Notes
- Intended to be paired with `EncodeListAsScriptParameters ( list_ )`.
- If a value legitimately contains `*****`, decoding will split it incorrectly.

---

## GetScriptParameters ( )

### Purpose
Converts the current script parameter from pipe-delimited (`|`) into a return-delimited list (`¶`).

### Parameters
- None (uses `Get ( ScriptParameter )` internally)

### Returns
- Text: a return-delimited list of parameter values.

### Example
**Input (ScriptParameter)**  
123|Edit|1

**Output**  
123¶Edit¶1

### Notes
- Useful together with `GetValue ( GetScriptParameters ( ) ; n )` to fetch the nth parameter.
- Assumes `|` is the delimiter used when passing script parameters.

---

## ConvertPipeDelimitedToReturnDelimited ( String )

### Purpose
Converts a pipe-delimited string (`|`) into a return-delimited list (`¶`).

### Parameters
- **String**  
  The input text containing `|` separators.

### Returns
- Text: the same values separated by returns (`¶`).

### Example
**Input (String)**  
ID123|Pending|High

**Output**  
ID123¶Pending¶High

### Notes
- Useful when you want to pass multiple values in one string and then use `GetValue ( list ; n )`.

---

## DefaultIfEmpty ( string ; replaceWith )

### Purpose
Returns a fallback value when the input string is empty.

### Parameters
- **string**  
  The value to check.
- **replaceWith**  
  The fallback value to return when `string` is empty.

### Returns
- Text: either `string` (if not empty) or `replaceWith` (if empty).

### Example
**Input**  
string = (empty)  
replaceWith = N/A  

**Output**  
N/A

### Notes
- Useful for display fields, labels, and formatted text where blanks are undesirable.

---

## PadLeft ( value_ ; character ; stringlength )

### Purpose
Pads the beginning of a value with a character until it reaches a target length.

### Parameters
- **value_**  
  The text to pad.
- **character**  
  The character to prepend repeatedly (for example `0`).
- **stringlength**  
  The target total length.

### Returns
- Text: the padded value.

### Example
**Input**  
value_ = 57  
character = 0  
stringlength = 5  

**Output**  
00057

### Notes
- Implemented using recursion.
- Common use case: generating fixed-length codes (like `00057`).

---

## RemoveTrailingComma ( Text )

### Purpose
Removes the last comma (final `,`) from a text string.

### Parameters
- **Text**  
  The input text containing comma separators.

### Returns
- Text: the same string with only the last comma removed.

### Example
**Input (Text)**  
A, B, C

**Output**  
A, B C

### Notes
- Removes only the last comma, not all commas.
- Does not insert spaces; it simply deletes the comma character.

---

## RemoveSpecialCharacters ( String )

### Purpose
Removes selected special characters from a string.

### Parameters
- **String**  
  The input text.

### Returns
- Text: the cleaned string with these characters removed: `"`, `/`, `'`.

### Example
**Input (String)**  
Cyril's "Folder"/Docs

**Output**  
Cyrils FolderDocs

### Notes
- Only removes the characters listed above.
- Useful for generating safer labels, filenames, or IDs from user-entered text.

---

## EscapeDoubleQuotes ( String )

### Purpose
Escapes double quotes in a string by converting `"` into `\"`.

### Parameters
- **String**  
  The input text that may contain double quotes.

### Returns
- Text: the escaped string.

### Example
**Input (String)**  
He said "Hello"

**Output**  
He said \"Hello\"

### Notes
- Useful when generating JSON or JavaScript strings to avoid broken syntax.

---

## EscapeSingleQuoteInHTML ( String )

### Purpose
Escapes single quotes (`'`) for HTML by converting them into the HTML entity `&#39;`.

### Parameters
- **String**  
  The input text that may contain single quotes.

### Returns
- Text: the escaped string.

### Example
**Input (String)**  
Cyril's laptop

**Output**  
Cyril&#39;s laptop

### Notes
- Useful when building HTML attributes or HTML content from user-entered text.
- Only escapes `'` in this implementation (it does not escape other characters like `&`, `<`, or `"`).

---

## DecodeURIComponent ( EncodedText )

### Purpose
Decodes selected URL-encoded (percent-encoded) sequences into readable characters for display. Also converts `%0A` into `<br>` for HTML line breaks.

### Parameters
- **EncodedText**  
  URL-encoded text containing `%xx` sequences.

### Returns
- Text: partially decoded output (only the encodings listed in the function are converted).

### Example
**Input (EncodedText)**  
Line%0A%E2%80%A2%20Item%201

**Output**  
Line<br>• Item 1

### Notes
- This is a partial decoder (it does not decode every possible `%xx` value).
- `%0A` is converted to `<br>`, which makes the output HTML-friendly.
- If an encoded sequence is not listed, it remains unchanged in the output.

---

## ConvertReturnsToBR ( string_ )

### Purpose
Converts return-delimited text into HTML by replacing each return (¶) with `<br>`.

### Parameters
- **string_**  
  Text that may contain returns (¶).

### Returns
- Text: HTML-friendly text with `<br>` tags inserted.

### Example
**Input (string_)**  
Line 1¶Line 2¶Line 3

**Output**  
Line 1<br>Line 2<br>Line 3

### Notes
- Useful when displaying FileMaker multi-line text in a WebViewer or HTML report.
- Only replaces returns; it does not escape HTML characters like `<` or `&`.

---

## ConvertLineBreakMarkersToReturns ( String )

### Purpose
Converts certain line break markers into FileMaker returns (`¶`).

### Parameters
- **String**  
  Text that may contain line break markers.

### Returns
- Text: the same text with markers converted to returns.

### Example
**Input (String)**  
Line 1yyyyLine 2<br>Line 3

**Output**  
Line 1¶Line 2¶Line 3

### Notes
- Replaces the literal string `yyyy` with a return (`¶`).
- Also converts `<br>` into a return (`¶`).
- Despite the name, it does not output HTML; it normalizes text into return-delimited format.

---

## JSNormalizeTextChain ( )

### Purpose
Returns a JavaScript replacement chain (as text) to normalize URL-encoded text for HTML display:
- converts encoded line breaks into `<br>`
- replaces pipe characters (`|`) with a placeholder string (`ppippe`)

### Parameters
- None

### Returns
- Text: a JavaScript `.replace(...)` chain that can be appended to a JS string expression.

### Example
**Input (conceptual JS input)**  
Hello%0AWorld|Now

**Output (after applying the chain)**  
Hello<br>WorldppippeNow

### Notes
- Intended to be pasted into JavaScript, not executed in FileMaker.
- Handles `%0D%0A`, `%0A`, and `%0D` line break encodings.
- Replaces `|` with `ppippe` as a placeholder (you can decode it later if needed).

---

## HTMLToTextStripTags ( text )

### Purpose
Converts basic HTML into readable plain text by:
- turning `<br>` and paragraphs into line breaks (`¶`)
- turning list items into bullet lines
- decoding common HTML entities
- removing any remaining HTML tags

### Parameters
- **text**  
  HTML or HTML-like text to convert.

### Returns
- Text: plain text formatted with FileMaker returns (`¶`).

### Example
**Input (text)**  
<p>Hello&nbsp;Cyril</p><ul><li>One</li><li>Two</li></ul>

**Output**  
Hello Cyril  
- One  
- Two

### Notes
- Converts `<li>` items into lines prefixed with `- `.
- Decodes: `&nbsp;`, `&amp;`, `&lt;`, `&gt;`.
- Recursively strips any other tags in `<...>`.

---

## FormatNumber ( Number ; DecimalPrecision ; ThousandSeparator )

### Purpose
Formats a number with thousand separators and a fixed number of decimal places.

### Parameters
- **Number**  
  The number to format (can be positive or negative).
- **DecimalPrecision**  
  How many decimal places to show (for example `0`, `2`, `3`).
- **ThousandSeparator**  
  The character used to group thousands in the integer part (for example `,`, `.`, or a space).

### Returns
- Text: the formatted number.

### Example
**Input**  
Number = 100000  
DecimalPrecision = 0  
ThousandSeparator = ,  

**Output**  
100,000

### Notes
- Rounds the value to the requested decimal precision.
- Supports negative numbers (adds `-` prefix).
- Intended for display formatting (reports, UI labels, etc.).

---

## AbbreviateNumber ( number ; precision )

### Purpose
Abbreviates a number using `K`, `M`, or `B` for thousands, millions, and billions.

### Parameters
- **number**  
  The number to abbreviate (supports negative values).
- **precision**  
  Number of decimal places to keep in the abbreviated result.

### Returns
- Text: abbreviated number like `1.2K`, `2.5M`, `3B`.

### Example
**Input**  
number = 1200  
precision = 2  

**Output**  
1.2K

### Notes
- Uses these suffixes:
  - `K` = thousands
  - `M` = millions
  - `B` = billions
- Values under 1,000 are returned as a rounded number without a suffix.

---

## Ordinal ( number )

### Purpose
Returns a number with its ordinal suffix (`st`, `nd`, `rd`, `th`).

### Parameters
- **number**  
  The input number (decimals are truncated to an integer).

### Returns
- Text: the integer plus its ordinal suffix (for example `1st`, `2nd`, `11th`).

### Example
**Input**  
number = 21  

**Output**  
21st

### Notes
- Uses `th` for 11, 12, and 13 (special cases).
- Uses only the integer part of the input.

---

## ShortenString ( string ; totalCharacters )

### Purpose
Returns the first N characters of a string, and appends an ellipsis (`…`) only when truncation happens.

### Parameters
- **string**  
  The text to shorten.
- **totalCharacters**  
  Maximum number of characters to keep.

### Returns
- Text: shortened text, optionally ending with `…`.

### Example
**Input**  
string = FileMaker is fast when you design it well  
totalCharacters = 12  

**Output**  
FileMaker is …

### Notes
- Uses a single ellipsis character (`…`) when the string is longer than the limit.

---

## Ellipsis ( Text_ ; MaxTextLength_ )

### Purpose
Truncates text to a maximum length and appends an ellipsis (`…`) when truncation occurs.

### Parameters
- **Text_**  
  The text to shorten.
- **MaxTextLength_**  
  Maximum number of characters to keep.

### Returns
- Text: shortened text with `…` when needed, otherwise the original text.

### Example
**Input**  
Text_ = Dokondo makes business stories easy to read  
MaxTextLength_ = 12  

**Output**  
Dokondo make…

### Notes
- Uses a single ellipsis character (`…`).
- If the text is already within the limit, nothing is appended.

---

## Singular ( string_ )

### Purpose
Attempts to convert a plural word to singular by removing common plural endings (`ies`, `es`, `s`).

### Parameters
- **string_**  
  The plural word.

### Returns
- Text: the word with the plural suffix removed.
- Empty if none of the endings match.

### Example
**Input (string_)**  
boxes

**Output**  
box

### Notes
- This is a simple suffix remover and does not handle irregular plurals.
- Words ending in `ies` are shortened by removing `ies` (no replacement to `y`).

---

## PluralizeText ( text ; quantity )

### Purpose
Returns the singular or plural form of the last word in a text based on a quantity.
Handles many common irregular nouns and several standard pluralization rules.

### Parameters
- **text**  
  The input text (only the last word is used).
- **quantity**  
  If `Abs(quantity) = 1`, the result is singular. Otherwise, plural rules are applied.

### Returns
- Text: the last word, singular or pluralized.

### Example
**Input**  
text = person  
quantity = 2  

**Output**  
people

### Notes
- Uses an internal JSON map of common irregular nouns (for example `person → people`, `child → children`).
- Applies common rules for `-s/-x/-z/-ch/-sh` (`+es`), `-y` (`-ies`), and `-f/-fe` (`-ves`) when not irregular.
- Returns only the last word (not the full phrase).

---

## PluralizeTerm ( term ; quantity )

### Purpose
Pluralizes only the last word in a term or phrase, based on quantity.

### Parameters
- **term**  
  A word or multi-word phrase (for example `Project Task`).
- **quantity**  
  If `quantity > 1`, the last word is pluralized. Otherwise, the term is returned unchanged.

### Returns
- Text: the original term with only the last word pluralized when needed.

### Example
**Input**  
term = Project Task  
quantity = 2  

**Output**  
Project Tasks

### Notes
- Uses `PluralizeText ( last_word ; quantity )` to pluralize the final word.
- Keeps all preceding words exactly as they were.

---

## AOrAn ( text_ )

### Purpose
Returns `a` or `an` based on the first character of a word.

### Parameters
- **text_**  
  The word to evaluate.

### Returns
- Text: `an` if the word starts with `a/e/i/o/u` (lowercase), otherwise `a`.

### Example
**Input (text_)**  
orange

**Output**  
an

### Notes
- Checks only the first character and only for lowercase vowels.
- Does not trim leading spaces.
- Uses spelling, not pronunciation.

---

## FullNameTitleFirstMiddleLast ( title ; first_name ; middle_name ; last_name )

### Purpose
Builds a full name in the order: Title, First Name, Middle Name, Last Name.  
Only non-empty parts are included.

### Parameters
- **title**  
  Optional title (for example `Mr.`, `Mrs.`, `Dr.`).
- **first_name**  
  First name.
- **middle_name**  
  Middle name (optional).
- **last_name**  
  Last name (optional).

### Returns
- Text: the formatted full name.

### Example
**Input**  
title = Dr.  
first_name = Cyril  
middle_name = Kofi  
last_name = Amegah  

**Output**  
Dr. Cyril Kofi Amegah

### Notes
- Skips any empty parts without leaving double spaces.

---

## FullNameTitleLastFirstMiddle ( title ; first_name ; middle_name ; last_name )

### Purpose
Builds a full name in the order: Title, Last Name, First Name, Middle Name.
Formats the last name in **bold** and places a comma after it.

### Parameters
- **title**  
  Optional title (for example `Mr.`, `Mrs.`, `Dr.`).
- **first_name**  
  First name.
- **middle_name**  
  Middle name (optional).
- **last_name**  
  Last name (optional).

### Returns
- Text: the formatted full name.

### Example
**Input**  
title = Dr.  
first_name = Cyril  
middle_name = Kofi  
last_name = Amegah  

**Output**  
Dr. AMEGAH, Cyril Kofi

### Notes
- The last name is styled using `TextStyleAdd ( last_name ; Bold )`.
- Empty parts are skipped without leaving extra spaces.

---

## FullNameLastFirstMiddleTitle ( title ; first_name ; middle_name ; last_name )

### Purpose
Builds a full name in the order: Last Name, First Name, Middle Name, then Title in parentheses.

### Parameters
- **title**  
  Optional title (displayed at the end in parentheses).
- **first_name**  
  First name.
- **middle_name**  
 Middle name (optional).
- **last_name**  
  Last name (optional).

### Returns
- Text: the formatted full name.

### Example
**Input**  
title = Dr.  
first_name = Cyril  
middle_name = Kofi  
last_name = Amegah  

**Output**  
Amegah, Cyril Kofi (Dr.)

### Notes
- If `title` is present, it is appended at the end as `(title)`.
- Empty parts are skipped without leaving extra commas or spaces.

---

## FullNameFirstMiddleLast ( first_name ; middle_name ; last_name )

### Purpose
Builds a full name in the order: First Name, Middle Name, Last Name.  
Only non-empty parts are included.

### Parameters
- **first_name**  
  First name (optional).
- **middle_name**  
  Middle name (optional).
- **last_name**  
  Last name (optional).

### Returns
- Text: the formatted full name.

### Example
**Input**  
first_name = Cyril  
middle_name = Kofi  
last_name = Amegah  

**Output**  
Cyril Kofi Amegah

### Notes
- Skips any empty parts without leaving double spaces.

---

## FullNameFirstLast ( first_name ; last_name )

### Purpose
Builds a name in the order: First Name, Last Name.  
Only non-empty parts are included.

### Parameters
- **first_name**  
  First name (optional).
- **last_name**  
  Last name (optional).

### Returns
- Text: the formatted name.

### Example
**Input**  
first_name = Cyril  
last_name = Amegah  

**Output**  
Cyril Amegah

### Notes
- Skips empty parts without leaving double spaces.

---

## AbbreviateText ( text )

### Purpose
Returns initials from a phrase by taking the first letter of each word (uppercase) and formatting with periods.  
Hyphenated words are preserved as hyphenated initials (for example `Jean-Paul` → `J.-P.`).

### Parameters
- **text**  
  The input phrase or name.

### Returns
- Text: initials formatted like `P. W.` or `J.-P. S.`

### Example
**Input (text)**  
Jean-Paul Sartre

**Output**  
J.-P. S.

### Notes
- Hyphens are handled so hyphenated words produce hyphenated initials.
- Intended to be used with `AbbreviateTextCustom ( text ; spaces ; hyphens ; periods )` for additional formatting control.

---

## AbbreviateTextCustom ( text ; spaces ; hyphens ; periods )

### Purpose
Creates initials from a name or phrase and lets you control whether spaces, hyphens, and periods are included.

### Parameters
- **text**  
  The input text to abbreviate (for example a person name).
- **spaces**  
  `0` = remove spaces  
  `1` = keep spaces
- **hyphens**  
  `0` = remove hyphens (or convert hyphen styling to non-hyphen output depending on the mode)  
  `1` = keep hyphens
- **periods**  
  `0` = remove all periods  
  `1` = remove all periods and add a trailing period at the end  
  `2` = keep periods as produced by `AbbreviateText`

### Returns
- Text: initials formatted according to the chosen options.

### Example
**Input**  
text = Mary Jane  
spaces = 0  
hyphens = 0  
periods = 2  

**Output**  
M.J.

### Notes
- Depends on `AbbreviateText ( text )` for the base initials.
- Best used for name abbreviations, short labels, and compact UI text.

---

## DateInWordsShortMonth ( _Date )

### Purpose
Formats a date as `Mon D, YYYY` using a 3-letter month abbreviation.

### Parameters
- **_Date**  
  The date to format.

### Returns
- Text: formatted date (example: `Dec 13, 2025`).

### Example
**Input (_Date)**  
2025-12-13

**Output**  
Dec 13, 2025

### Notes
- Month is derived from `MonthName ( _Date )` and shortened to 3 letters.

---

## DateInWordsShortMonthNoYear ( _Date )

### Purpose
Formats a date as `Mon D` using a 3-letter month abbreviation (no year).

### Parameters
- **_Date**  
  The date to format.

### Returns
- Text: formatted date (example: `Dec 13`).

### Example
**Input (_Date)**  
2025-12-13

**Output**  
Dec 13

### Notes
- Month is derived from `MonthName ( _Date )` and shortened to 3 letters.

---

## DateInWordsShortMonthNDay ( _Date )

### Purpose
Formats a date as `Day, Mon D, YYYY` using 3-letter weekday and month abbreviations.

### Parameters
- **_Date**  
  The date to format.

### Returns
- Text: formatted date (example: `Sat, Dec 13, 2025`).

### Example
**Input (_Date)**  
2025-12-13

**Output**  
Sat, Dec 13, 2025

### Notes
- Weekday is derived from `DayName ( _Date )` and shortened to 3 letters.
- Month is derived from `MonthName ( _Date )` and shortened to 3 letters.

---

## DateInWordsShortMonthNDayNoYear ( _Date )

### Purpose
Formats a date as `Day, Mon D` using 3-letter weekday and month abbreviations (no year).

### Parameters
- **_Date**  
  The date to format.

### Returns
- Text: formatted date (example: `Sat, Dec 13`).

### Example
**Input (_Date)**  
2025-12-13

**Output**  
Sat, Dec 13

### Notes
- Weekday is derived from `DayName ( _Date )` and shortened to 3 letters.
- Month is derived from `MonthName ( _Date )` and shortened to 3 letters.

---

## DateInWordsMonthNYear ( _Date )

### Purpose
Formats a date as `Month, YYYY` using the full month name.

### Parameters
- **_Date**  
  The date to format.

### Returns
- Text: formatted month and year (example: `December, 2025`).

### Example
**Input (_Date)**  
2025-12-13

**Output**  
December, 2025

### Notes
- Uses `MonthName ( _Date )` for the full month name.

---

## DateInWordsLongMonth ( _Date )

### Purpose
Formats a date as `Month D, YYYY` using the full month name.

### Parameters
- **_Date**  
  The date to format.

### Returns
- Text: formatted date (example: `December 13, 2025`).

### Example
**Input (_Date)**  
2025-12-13

**Output**  
December 13, 2025

### Notes
- Uses `MonthName ( _Date )` for the full month name.

---

## RelativeDateInWords ( _Date )

### Purpose
Returns a friendly relative label for dates near today (up to 3 days before/after).  
Dates outside that range are returned in short date format.

### Parameters
- **_Date**  
  The date to format.

### Returns
- Text: `Yesterday`, `Today`, `Tomorrow`, `2 days ago`, `In 2 days`, etc.  
  Otherwise: a formatted date like `Dec 13, 2025`.

### Example
Assuming current date is 2025-12-13:

**Input (_Date)**  
2025-12-12

**Output**  
Yesterday

### Notes
- Uses `Get ( CurrentDate )` as the reference point.
- Uses `DateInWordsShortMonth ( _Date )` as the fallback format.

---

## MySQLDateToFMPDateText ( MySQL_Date )

### Purpose
Converts a MySQL date string (`YYYY-MM-DD`) into `DD/MM/YYYY` format.

### Parameters
- **MySQL_Date**  
  A date string in MySQL format: `YYYY-MM-DD`.

### Returns
- Text: the reformatted date string `DD/MM/YYYY`.

### Example
**Input (MySQL_Date)**  
2025-12-13

**Output**  
13/12/2025

### Notes
- Returns a text date string.
- If you need a real FileMaker Date value, convert the output according to your file’s date settings.

---

## MySQLTimestampToFMPTimestampText ( MySQL_Timestamp )

### Purpose
Converts a MySQL timestamp string (`YYYY-MM-DD HH:MM:SS`) into `DD/MM/YYYY HH:MM:SS` format.

### Parameters
- **MySQL_Timestamp**  
  A timestamp string in MySQL format: `YYYY-MM-DD HH:MM:SS`.

### Returns
- Text: the reformatted timestamp string `DD/MM/YYYY HH:MM:SS`.

### Example
**Input (MySQL_Timestamp)**  
2025-12-13 14:05:09

**Output**  
13/12/2025 14:05:09

### Notes
- Returns a text timestamp string.
- If you need a real FileMaker Timestamp value, convert the output according to your file’s timestamp settings.

---

## FMPDateTextToMySQLDate ( FMP_Date )

### Purpose
Converts a date string from `DD/MM/YYYY` format into MySQL `YYYY-MM-DD` format.

### Parameters
- **FMP_Date**  
  A date string in `DD/MM/YYYY` format.

### Returns
- Text: the reformatted date string `YYYY-MM-DD`.

### Example
**Input (FMP_Date)**  
13/12/2025

**Output**  
2025-12-13

### Notes
- Expects a text date formatted as `DD/MM/YYYY`.
- If you have a FileMaker Date value, convert it to the expected format before using this function.

---

## DateInShortWordsToFMPDate ( inputDate )

### Purpose
Converts a short-word date string (3-letter month name) into a FileMaker Date value.

### Parameters
- **inputDate**  
  A date string containing a short month name like `Jan`, `Feb`, `Mar`, etc.  
  Expected shape: `DayName, Mon DD YYYY` (comma is optional).

### Returns
- Date: a FileMaker Date value created using `Date ( monthNumber ; dayNumber ; year_ )`.

### Example
**Input (inputDate)**  
Sat, Dec 13 2025

**Output**  
13/12/2025

### Notes
- Month names must be exactly: `Jan`, `Feb`, `Mar`, `Apr`, `May`, `Jun`, `Jul`, `Aug`, `Sep`, `Oct`, `Nov`, `Dec`.
- Output display depends on the file’s date formatting settings.

---

## MinutesToTextTime ( Minutes )

### Purpose
Converts a duration in minutes into a readable text string using hours and minutes.

### Parameters
- **Minutes**  
  Total number of minutes.

### Returns
- Text: formatted duration such as `45 min`, `1 hr 15 min`, or `2 hrs`.

### Example
**Input (Minutes)**  
75

**Output**  
1 hr 15 min

### Notes
- Uses hours for values of 60 minutes or more.
- Adds the minutes remainder only when it is greater than 0.
- Hours are rounded, so some edge cases may look unexpected for certain values.

---

## Elapsed ( StartDate ; EndDate )

### Purpose
Returns an approximate elapsed duration between two dates as a readable string: `X Years Y Months Z Days`.

### Parameters
- **StartDate**  
  The starting date.
- **EndDate**  
  The ending date.

### Returns
- Text: elapsed duration in years, months, and days.

### Example
**Input**  
StartDate = 2025-01-01  
EndDate = 2026-01-01  

**Output**  
1 Years 0 Months 0 Days

### Notes
- Uses average values (365.25 days per year and 30.4375 days per month), so results are approximate.
- Intended for friendly display, not strict calendar-accurate date math.

---

## ElapsedTime ( TStampBegin ; TStampEnd ; Format )

### Purpose
Calculates elapsed time between two timestamp fields and returns the result in a chosen format.

### Parameters
- **TStampBegin**  
  Start timestamp (must be a timestamp field/value).
- **TStampEnd**  
  End timestamp (must be a timestamp field/value).
- **Format**  
  One of: `Seconds`, `Minutes`, `Hours`, `Days`, `String`.

### Returns
- Number when `Format = Seconds`
- Text for `Minutes`, `Hours`, `Days`, and `String`

### Example
**Input**  
TStampBegin = 2025-12-13 10:00:00  
TStampEnd = 2025-12-13 12:15:30  
Format = String  

**Output**  
2 hrs, 15 mins

### Notes
- In `String` format, seconds are shown only when days, hours, and minutes are all zero.
- `Minutes`, `Hours`, and `Days` may include decimals because they are calculated from total seconds.

---

## TimeNoSeconds ( _Time )

### Purpose
Formats a time value as a 12-hour clock string without seconds, including an AM or PM suffix.

### Parameters
- **_Time**  
  A FileMaker Time value.

### Returns
- Text: formatted time like `2:15 PM`.

### Example
**Input (_Time)**  
14:15:30

**Output**  
2:15 PM

### Notes
- Uses `Mod ( Hour ( _Time ) ; 12 )`, so 12 o’clock can display as `0:xx`.
- Minutes are not zero-padded in this implementation.
- The AM or PM suffix is taken from the last two characters of the time’s text format.

---

## FormatTimeHHMM12 ( theTime )

### Purpose
Formats a time into 12-hour `h:mm AM/PM` format with zero-padded minutes.

### Parameters
- **theTime**  
  A value that can be interpreted as a FileMaker Time (time value or time-like text).

### Returns
- Text: formatted time like `2:05 PM`.
- Empty if the input cannot be converted to a valid time.

### Example
**Input (theTime)**  
14:05:59

**Output**  
2:05 PM

### Notes
- Uses `GetAsTime` to parse the input.
- Converts 0 hour to `12` for midnight.
- Minutes are always shown as two digits.

---

## RelativeElapsedPeriodLong ( Timestamp1 ; Timestamp2 )

### Purpose
Returns a human-friendly description of the elapsed time between two timestamps using the most appropriate unit (minutes, hours, days, weeks, months, or years) with pluralization.

### Parameters
- **Timestamp1**  
  First timestamp.
- **Timestamp2**  
  Second timestamp.

### Returns
- Text: elapsed period such as `5 mins`, `2 hours`, `1 day`, `3 weeks`, `6 months`, `2 years`.

### Example
**Input**  
Timestamp1 = 2025-12-13 10:00:00  
Timestamp2 = 2025-12-13 12:05:00  

**Output**  
2 hours

### Notes
- Uses absolute difference, so the order of timestamps does not matter.
- Uses approximate conversions:
  - month = 30 days
  - year = 365 days
- Values are rounded down to whole units.

---

## RelativeElapsedPeriodShort ( Timestamp1 ; Timestamp2 )

### Purpose
Returns a compact elapsed time label between two timestamps using:
- minutes (`m`)
- hours (`h`)
- days (`d`)
- weeks (`w`)

### Parameters
- **Timestamp1**  
  First timestamp.
- **Timestamp2**  
  Second timestamp.

### Returns
- Text: elapsed period like `15m`, `2h`, `6d`, `3w`.

### Example
**Input**  
Timestamp1 = 2025-12-13 10:00:00  
Timestamp2 = 2025-12-13 10:42:00  

**Output**  
42m

### Notes
- Uses absolute difference, so order does not matter.
- Values are rounded down to whole units.
- For differences under 60 seconds, the result can be `0m`.

---

## RelativePeriodShort ( Timestamp1 ; Timestamp2 )

### Purpose
Returns a compact relative time phrase showing how `Timestamp1` relates to `Timestamp2`.

### Parameters
- **Timestamp1**  
  The timestamp being described.
- **Timestamp2**  
  The reference timestamp.

### Returns
- Text:
  - `Xm ago`, `Xh ago`, `Xd ago`, `Xw ago` when Timestamp1 is earlier than Timestamp2
  - `in Xm`, `in Xh`, `in Xd`, `in Xw` when Timestamp1 is later than Timestamp2
  - `now` when both timestamps are equal

### Example
**Input**  
Timestamp1 = 2025-12-13 10:00:00  
Timestamp2 = 2025-12-13 10:42:00  

**Output**  
42m ago

### Notes
- Uses whole units (rounded down).
- Unit selection: minutes < 60, hours < 24, days < 7, otherwise weeks.
- For differences under 60 seconds, output can be `0m ago` / `in 0m` unless the timestamps are exactly equal.

---

## Age ( DOB )

### Purpose
Calculates age in years from a date of birth.

### Parameters
- **DOB**  
  Date of birth.

### Returns
- Number: age in years.
- Empty if `DOB` is empty.

### Example
Assuming current date is 2025-12-13:

**Input (DOB)**  
2000-12-20

**Output**  
24

### Notes
- Subtracts 1 if the birthday has not yet occurred in the current year.
- Uses `Get ( CurrentDate )` as the reference date.

---

## WeekdayCount ( StartDate ; EndDate )

### Purpose
Returns the number of weekdays (Monday to Friday) between two dates, including both StartDate and EndDate.

### Parameters
- **StartDate**  
  The first date in the range.
- **EndDate**  
  The last date in the range.

### Returns
- Number: count of weekdays in the date range.

### Example
**Input**  
StartDate = 2025-12-12  
EndDate = 2025-12-15  

**Output**  
2

### Notes
- Excludes Saturdays and Sundays.
- Includes both StartDate and EndDate in the calculation.
- Does not account for holidays.
- Depends on `DayName ( StartDate )` matching English day names (Monday, Tuesday, etc.).

---

## MondayOfGivenWeek ( week_ ; year_ )

### Purpose
Returns the date of the Monday for a given week number in a given year (ISO week logic).

### Parameters
- **week_**  
  Week number (for example `1` to `52/53` depending on the year).
- **year_**  
  The year (4-digit).

### Returns
- Date: the Monday date for the specified week and year.

### Example
**Input**  
week_ = 1  
year_ = 2025  

**Output**  
(Monday date of ISO week 1 in 2025)

### Notes
- Uses the rule that January 4th is always in ISO week 1.
- Useful for schedules and week-based reporting.

---

## AddDaysToDate ( Date_ ; DaysToAdd_ )

### Purpose
Adds a number of days to a date and returns the resulting date.

### Parameters
- **Date_**  
  The starting date.
- **DaysToAdd_**  
  Number of days to add (use a negative number to subtract days).

### Returns
- Date: the adjusted date.

### Example
**Input**  
Date_ = 2025-12-13  
DaysToAdd_ = 5  

**Output**  
2025-12-18

### Notes
- Negative values move backward in time.

---

## AddWeeksToDate ( Date_ ; WeeksToAdd_ )

### Purpose
Adds a number of weeks to a date and returns the resulting date.

### Parameters
- **Date_**  
  The starting date.
- **WeeksToAdd_**  
  Number of weeks to add (use a negative number to subtract weeks).

### Returns
- Date: the adjusted date.

### Example
**Input**  
Date_ = 2025-12-13  
WeeksToAdd_ = 2  

**Output**  
2025-12-27

### Notes
- Internally treats 1 week as 7 days.

---

## AddMonthsToDate ( Date_ ; MonthsToAdd_ )

### Purpose
Adds months to a date by adjusting the month component and rebuilding the date.

### Parameters
- **Date_**  
  The starting date.
- **MonthsToAdd_**  
  Number of months to add.

### Returns
- Date: the adjusted date.

### Example
**Input**  
Date_ = 2025-10-13  
MonthsToAdd_ = 3  

**Output**  
2026-01-13

### Notes
- Handles month rollover only when the resulting month is greater than 12 (adds 1 year).
- Does not handle negative month values or large month offsets beyond a single year.
- If the resulting day does not exist in the target month (for example 31st), FileMaker will normalize the date.

---

## AddYearsToDate ( Date_ ; YearsToAdd_ )

### Purpose
Adds years to a date by keeping the same month/day and incrementing the year.

### Parameters
- **Date_**  
  The starting date.
- **YearsToAdd_**  
  The number of years to add (use a negative number to subtract years).

### Returns
- Date: the adjusted date.

### Example
**Input**  
Date_ = 2025-12-13  
YearsToAdd_ = 1  

**Output**  
2026-12-13

### Notes
- Uses: `Date ( Month ( Date_ ) ; Day ( Date_ ) ; Year ( Date_ ) + YearsToAdd_ )`
- Dates like Feb 29 may be normalized when the target year is not a leap year.

---

## AddPeriodToDate ( Date_ ; NumberToAdd_ ; PeriodTypeToAdd_ )

### Purpose
Adds a specified period (days, weeks, months, years) to a date based on a period type string.

### Parameters
- **Date_**  
  The starting date.
- **NumberToAdd_**  
  How many units to add.
- **PeriodTypeToAdd_**  
  One of: `Hours`, `Days`, `Weeks`, `Months`, `Years`.

### Returns
- Date: the adjusted date (or the original date for `Hours`).

### Example
**Input**  
Date_ = 2025-12-13  
NumberToAdd_ = 5  
PeriodTypeToAdd_ = Days  

**Output**  
2025-12-18

### Notes
- `Hours` currently returns the original date (no time calculation).
- Depends on helper functions: `AddDaysToDate`, `AddWeeksToDate`, `AddMonthsToDate`, `AddYearsToDate`.
- If an unsupported period type is provided, the result is empty.

---

## RandomInteger ( start ; stop )

### Purpose
Returns a random integer between two numbers (inclusive).

### Parameters
- **start**  
  The minimum integer.
- **stop**  
  The maximum integer.

### Returns
- Number: a random whole number between `start` and `stop` (inclusive).

### Example
**Input**  
start = 1  
stop = 6  

**Output (example)**  
4

### Notes
- Output is random, so it will vary each time.
- Designed to behave correctly even when negative ranges are used.

---

## GeneratePassword ( Len ; Letters ; Numbers ; Symbols )

### Purpose
Generates a random password using selected character groups (letters, numbers, symbols).

### Parameters
- **Len**  
  Target password length.
- **Letters**  
  `0` = none  
  `1` = uppercase only  
  `2` = lowercase only  
  `3` = uppercase + lowercase
- **Numbers**  
  `0` = exclude, `1` = include digits `0-9`
- **Symbols**  
  `0` = exclude, `1` = include symbols from this set: `$%&@#~`

### Returns
- Text: a randomly generated password.

### Example
**Input**  
Len = 12  
Letters = 3  
Numbers = 1  
Symbols = 0  

**Output (example)**  
aQ7mT2pL9xWb

### Notes
- Output is random, so it will vary each time.
- Character groups are included in the pool, but it is not guaranteed that each selected group appears at least once.
- Uses recursion and temporary global variables internally, then clears them.

---

## GenerateNextRecordCode ( Alphabets ; NumericLength ; LastSerial )

### Purpose
Generates the next sequential record code using a text prefix and a zero-padded number.

### Parameters
- **Alphabets**  
  The prefix to place at the start of the code (for example `INV` or `EMP-`).
- **NumericLength**  
  The fixed length of the numeric portion (pads with leading zeros).
- **LastSerial**  
  The last used serial number (the function increments this by 1).

### Returns
- Text: the next code in the sequence.

### Example
**Input**  
Alphabets = INV  
NumericLength = 5  
LastSerial = 24  

**Output**  
INV00025

### Notes
- Numeric part is always padded to `NumericLength` using leading zeros.
- Assumes `LastSerial` is a number.

---

## GenerateNextCode ( prefix ; serialLength ; previousCode )

### Purpose
Generates the next sequential code by extracting the numeric serial portion from `previousCode`, incrementing it, zero-padding it to `serialLength`, and prefixing it with `prefix`.

### Parameters
- **prefix**  
  The text prefix (example: `INV-`, `CUS`).
- **serialLength**  
  The desired length of the numeric serial portion.
- **previousCode**  
  The previous code value (expected to start with `prefix`).

### Returns
- Text: the next code, formatted as `prefix + zero-padded serial`.

### Example
**Input**  
prefix = INV-  
serialLength = 5  
previousCode = INV-00042  

**Output**  
INV-00043

### Notes
- Assumes `previousCode` begins with `prefix`.
- Assumes the serial portion is numeric.
- Padding uses a fixed 10-zero template; ensure `serialLength` is within that range.

---

## GetDevice ( )

### Purpose
Returns a readable device/platform name based on FileMaker's `Get ( Device )`.

### Parameters
- None

### Returns
- Text: `Mac`, `Windows`, `iPad`, `iPhone`, `Android`, or `Unknown`.

### Example
If FileMaker returns `Get ( Device ) = 1`

**Output**  
Mac

### Notes
- Useful for platform-specific UI decisions, behaviors, or messaging.

---

## GetExtension ( Some Filename )

### Purpose
Returns the file extension (the text after the last `.`) from a filename.

### Parameters
- **Some Filename**  
  The filename or path as text.

### Returns
- Text: the extension (for example `pdf`, `png`, `gz`).

### Example
**Input (Some Filename)**  
archive.tar.gz

**Output**  
gz

### Notes
- Returns only the last extension segment.
- If there is no `.`, the result will be the entire input string.

---

## GetFileSize ( Bytes ; Precision ; UOM ; Format )

### Purpose
Converts a byte count into a human-readable file size string (KB, MB, GB, etc.) with configurable precision, unit system, and label style.

### Parameters
- **Bytes**  
  Raw size in bytes.
- **Precision**  
  Number of decimal places to round to (defaults to `0` when empty).
- **UOM**  
  Unit of measure base:
  - empty = binary (`1024`)
  - `metric` or `M` = metric (`1000`)
  - `binary` or `B` = binary (`1024`)
- **Format**  
  Label style:
  - empty = no label
  - `short` or `S` = `KB`, `MB`, `GB`, etc.
  - `long` or `L` = `Kilobyte(s)`, `Megabyte(s)`, `Gigabyte(s)`, etc.

### Returns
- Text: formatted size, optionally with a unit label.

### Example
**Input**  
Bytes = 430714880  
Precision = 0  
UOM = (empty)  
Format = short  

**Output**  
411 MB

### Notes
- Uses logarithms to pick the best magnitude up to Yottabytes.
- Pluralizes long labels when the numeric size is not 1 (for example `2 Megabytes`).

---

## GenerateFileIconHTML ( Extension )

### Purpose
Returns an HTML Font Awesome `<i>` icon tag based on a file extension, for use in WebViewer/HTML UIs.

### Parameters
- **Extension**  
  File extension (example: `pdf`, `jpg`, `xlsx`).

### Returns
- Text: an HTML `<i>` tag for the matching icon.
- Empty string when the extension is not recognized.

### Extension Mapping
- `pdf` → file-pdf icon (red)
- `png`, `jpg`, `jpeg`, `webp` → file-image icon (purple)
- `csv`, `xlsx` → file-excel icon (green)
- `doc`, `txt`, `tab`, `log` → file-word icon (blue)
- `zip`, `rar` → file-archive icon (orange)

### Example
**Input (Extension)**  
pdf

**Output**  
<i class='fas fa-file-pdf icon' style='color: red;'></i>

### Notes
- Assumes Font Awesome is available in the HTML page.
- Consider lowercasing the extension before calling for consistent matching.

---

## RecordStatus ( status )

### Purpose
Maps a record status text to a color label string for UI display or conditional formatting.

### Parameters
- **status**  
  The record status text.

### Returns
- Text: `gray`, `yellow`, `red`, or `green`.
- Empty if the status value is not recognized.

### Status Mapping
- **gray**
  - Draft
  - Not Submitted
  - Not Filled
  - Not Completed
  - Walk-In

- **yellow**
  - Pending
  - Pending Approval
  - Low Stock
  - Submitted
  - Paused
  - Will be Meeting
  - Pending Feedback

- **red**
  - Inactive
  - Out of Stock
  - Denied
  - Planning To Engage
  - Started To Engage
  - Made First Contact
  - Not Prospect

- **green**
  - Active
  - Completed
  - Complete
  - Done
  - In Stock
  - Approved
  - Appraised
  - Posted
  - Dispensed
  - Client
  - Prospect

### Example
**Input (status)**  
Pending Approval

**Output**  
yellow

### Notes
- Exact matching is used, so spelling and capitalization must match your list.

---

## Pronoun ( Gender ; Type )

### Purpose
Returns a pronoun based on gender and a requested pronoun type.

### Parameters
- **Gender**  
  Expected values: `Male`, `Female`.  
  Any other value returns a neutral pronoun option.
- **Type**  
  One of:
  - `His/Her`
  - `Him/Her`
  - `He/She`
  - `His/Hers`

### Returns
- Text: the selected pronoun.

### Example
**Input**  
Gender = Male  
Type = He/She  

**Output**  
He

### Notes
- Neutral outputs used when gender is not `Male` or `Female`:
  - `Their`, `Them`, `They`, `Theirs`
- Unsupported `Type` values return empty.

---

## ExecuteSQLResultToJSON ( sqlResult ; fieldSub ; rowSub ; fieldList ; limit )

### Purpose
Converts an `ExecuteSQL` text result into a JSON object with:
- `data`: an array of row objects
- `records`: the row count  
Returns an error JSON object when inputs do not match expectations.

### Parameters
- **sqlResult**  
  The raw text returned from `ExecuteSQL`.
- **fieldSub**  
  The column delimiter used inside each row (must be a unique string not present in data).
- **rowSub**  
  The row delimiter used between rows (must be a unique string not present in data).
- **fieldList**  
  Return-delimited list of JSON keys matching SQL columns.  
  You can cast types by appending:
  - `|JSONNumber`
  - `|JSONBoolean`  
  Default type is string.
- **limit**  
  Max number of rows to include. If `0`, all rows are included.

### Returns
**Success**
- JSON with keys:
  - `data` (array)
  - `records` (number)

**Failure**
- JSON with keys:
  - `error` = 1
  - `message` (text)

### Example
**Input (conceptual)**
- fieldSub = `|#!`
- rowSub = `[*>`
- fieldList:
  - name
  - age|JSONNumber

**Output (shape)**
{
  "data": [
    { "name": "John", "age": 12 },
    { "name": "Ama", "age": 10 }
  ],
  "records": 2
}

### Notes
- `fieldList` count must match the number of SQL columns.
- Delimiter strings must not appear in real data.
- Uses an internal placeholder to temporarily replace real returns inside field values.

---

## SQLFieldNames ( _sqlText )

### Purpose
Extracts the field/column list from an SQL `SELECT ... FROM ...` statement and returns it as a return-delimited list.

### Parameters
- **_sqlText**  
  The SQL statement text.

### Returns
- Text: selected column expressions, one per line.

### Example
**Input (_sqlText)**  
SELECT t1."First Name", t1."Last Name", t2."Age" FROM People t1

**Output**  
First Name  
Last Name  
Age

### Notes
- Designed for simple SQL shaped like `SELECT ... FROM ...`.
- Removes `"` and only strips hard-coded aliases `t1.` and `t2.`.
- Does not fully parse complex SQL (functions, nested selects, other alias patterns).

---

## IsValidEmailFormat ( _email )

### Purpose
Validates the basic *format* of an email address. Returns `1` when the email is correctly formatted, otherwise `0`.

### Parameters
- **_email**  
  Email address text to validate.

### Returns
- Number:
  - `1` = valid format
  - `0` = invalid format

### Example
**Input (_email)**  
cyril.amegah@example.com

**Output**  
1

### Notes
- This checks format only (it does not verify the domain exists).
- Empty values return `1` in the current implementation because of `IsEmpty ( _email ) or ...`.
- Rules enforced include:
  - exactly one `@`
  - allowed characters only
  - no `..`
  - at least one `.` after the `@`
  - first and last character are alphanumeric

---

## Base64EncodeFile ( file )

### Purpose
Encodes a container/file value into a clean Base64 payload string:
- removes line breaks and hidden separator characters
- converts URL-safe Base64 (`_` and `-`) into standard Base64 (`/` and `+`)
- strips any `data:...base64,` header and returns only the Base64 content

### Parameters
- **file**  
  A FileMaker container value or file data to encode.

### Returns
- Text: Base64 payload only (no header, no line breaks).

### Example
**Input (file)**  
(container holding `report.pdf`)

**Output**  
A Base64 string like `JVBERi0xLjQKJc...`

### Notes
- The output is the Base64 content after the first comma in the encoded string.
- Useful for APIs that expect a raw Base64 payload rather than a full data URL.

---

# Script: Get Value List

### Purpose
`Get Value List` is a single, centralized list generator that returns UI-ready list content for many different WebViewer patterns in CauferoAppStarter.

It is a list service that:
- selects the correct dataset based on `List Style` and `List Key`
- fetches rows either via `ExecuteSQL` or via static inline `$Data`
- formats the result into the exact output shape required by the target UI pattern
- returns the formatted output as the script result (Text Result)

The goal is to standardize list generation across the entire file, reduce duplicated list logic, and keep WebViewer UIs consistent.

---

## Inputs and Output

### Input (ScriptParameter)
Single text parameter with 3 parts:

`List Style|List Key|Selected Item`

- delimiter: `|`
- the script converts `|` to `¶` and reads:
  - `$Style`        = GetValue ( paramAsList ; 1 )
  - `$ListKey`      = GetValue ( paramAsList ; 2 )
  - `$SelectedItem` = GetValue ( paramAsList ; 3 )

### Output
- Script exits with: `Exit Script [ Text Result: $GeneratedList ]`
- `$GeneratedList` is already formatted for direct use by the caller:
  - HTML snippets
  - JavaScript datasets
  - JSON payloads
  - plain lists (IDs, receipt rows)

---

## Hard Rules (Enforced)

### 1) Always reset `$Data` at script start
`$Data` is the fallback carrier for static lists (Countries, Currencies, Regions, etc.). It must be cleared on every run to prevent carryover from previous calls.

Required initialization:
- `Set Variable [ $Data ; Value: "" ]`

### 2) Master-Details details description variable
In the Master-Details details loop, the script sets:
- `$Details Description`

Therefore, the generated details object must use:
- `$Details Description`

Any reference to `$Details Item Description` should be removed or renamed for consistency.

### 3) ExecuteSQL parameterization policy
Either approach is acceptable:
- parameterized `ExecuteSQL` using `?`
- string concatenation

Recommendation for the AI Agent:
- prefer parameterized SQL when inserting values that might contain quotes or user-entered text
- string concatenation is fine for stable IDs and trusted globals when convenient

### 4) JS placeholder encoding and decoding
For `Style = "JS"`, the script encodes unsafe characters into placeholder tokens before building a JS object-text output.

Assumption (confirmed): all calling scripts implement the same decoding logic.

Hard rule:
- any consumer of `Style = "JS"` must decode placeholders before display, searching, or comparisons

If the encode mapping changes in this script, the shared decode mapping in callers must be updated in the same change set.

---

## High-Level Flow

1. Parse parameter into `$Style`, `$ListKey`, `$SelectedItem`
2. Initialize variables (including clearing `$Data`)
3. Determine `$SQL` or `$Data` based on `$Style` + `$ListKey`
4. Build `$List`
   - from `ExecuteSQL` if `$SQL` is set
   - from `$Data` if `$SQL` is empty
5. Transform `$List` into `$GeneratedList` based on `$Style`
6. Exit with `$GeneratedList`

---

## Supported List Styles
Style values are case sensitive and must match exactly:

1. Dropdown
2. Searchable Dropdown
3. Checklist
4. JS
5. JS 2
6. Mini-List
7. Notes List
8. Master-Details
9. Grid
10. ID List
11. Hierarchical Form Setup
12. Hierarchical Form With Scores
13. Table In First Card
14. Table In Second Card
15. Table With Checkboxes
16. Table with Images
17. List for Receipt
18. Bill with Checkboxes
19. JSON
20. Remunerations

Note: Some styles are placeholders and are not fully implemented yet (see Placeholder Styles section).

---

## Data Source Contract

### `$SQL` mode
If `$SQL` is not empty:
- the script runs `ExecuteSQL ( $SQL ; fieldSeparator ; rowSeparator ; [optional parameters...] )`
- the result becomes `$List`

Default separators (most styles):
- fieldSeparator = `|`
- rowSeparator   = `¶`

### `$Data` mode
If `$SQL` is empty:
- `$List` becomes `$Data`

Expected `$Data` format:
- rows separated by `¶`
- columns separated by `|`

For simple static dropdowns that only need a single label per row:
- `$Data` is usually one value per row
- the Dropdown formatter will treat the single value as the label, and also use it as the ID

---

## Output Contracts by Style
This section defines the strict contract the AI Agent must follow when adding new list keys or consuming outputs.

### Style: Dropdown
Use case:
- HTML `<select>` options

Expected `$List` row shape:
- SQL mode: `ID|Label`
- Data mode: `Label`

Output shape:
- begins with a default option:
  - `<option value='' selected>Select One</option>` when `$SelectedItem` is empty
- then one `<option>` per record
- the option is marked `selected` when the record ID equals `$SelectedItem`

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Dropdown|Departments|" ]

---

### Style: Searchable Dropdown
Use case:
- JS dataset for a searchable dropdown widget

Expected `$List` row shape:
- `ID|Label`

Output shape:
- comma-separated JS objects:
  - `{ name: 'Label', uuid: 'ID' }`

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Searchable Dropdown|Patients|" ]

---

### Style: Checklist
Use case:
- HTML checklist with checkboxes and multi-line display

Expected `$List` row shape:
- `ID|Title|Subtitle|Description`

Output shape:
- `<div class='list-item' data-id='ID'>...</div>` for each row
- wrapped inside `<div class='list-container'>...</div>`
- includes footer with selected counter

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Checklist|Analytes|" ]

---

### Style: JS
Use case:
- JS object map keyed by record ID
- each record contains dynamic fields `c2..cN` derived from SQL columns 2..N

Expected `$List` row shape:
- column 1 = ID
- column 2..N = fields required by the consuming JavaScript

Output shape:
- text containing JS object entries (caller typically wraps in `{ ... }`)
- one entry per ID:
  `"ID": { c2: "value2", c3: "value3", ... }`

Important:
- output is placeholder-encoded (see next section)
- caller must decode placeholders before display or logic

Example call:
    Perform Script [ "Get Value List" ; Parameter: "JS|Job Titles|" ]

---

## JS Placeholder Encoding Contract

### Why encoding exists
The JS style returns JavaScript object text. Raw data can contain characters that break JS string literals and WebViewer parsing. Encoding prevents breakage and reduces injection risks.

### Encode mapping (script side)
In the JS branch, the script encodes `$List` with `Substitute` using these mappings:

- `¶`  -> `yyyyyyyyyy`
- `<`  -> `lessthan`
- `>`  -> `greaterthan`
- `/`  -> `forwardslash`
- `\`  -> `backslash`
- `#`  -> `hashhhh`
- `'`  -> `singlequote`
- `"`  -> `doublequotes`
- tab -> `sometab`
- `=`  -> `equalto`
- `::` -> `doublecolon`
- `;`  -> `semicolon`
- `&`  -> `ampersand`
- `|`  -> `ppippe`

Then it restores row breaks by converting the JS rowSeparator token back to `¶`.

### Decode mapping (caller side)
All callers must reverse the mappings above.

Hard rules:
- decode must happen before rendering to users
- decode must happen before searching or comparing values
- if the encode mapping changes in this script, the decode mapping must change in the same update

Testing rule:
- choose one record that contains multiple encoded characters (quotes, ampersand, slash)
- confirm the WebViewer UI shows the original text after decode
- confirm filtering or searching uses the decoded value

---

### Style: JS 2
Use case:
- specialized key:value mapping output, mainly for answers keyed by question ID

Expected `$List` row shape:
- produced by SQL designed to output key and value columns

Transformation:
- replaces `|` with `:`
- replaces `¶` with `,¶`
- applies a guard for empty output cases

Example call:
    Perform Script [ "Get Value List" ; Parameter: "JS 2|Staff Appraisal Answers|<StaffAppraisalID>" ]

---

### Style: Mini-List
Use case:
- compact clickable table rows inside a card area

Expected `$List` row shape:
- depends on list key
- commonly: `ID|Main|Sub1|Sub2|...`

Output shape:
- `<table class='mini-list'>...</table>`

Side effects:
- for remunerations mini-list, it accumulates `$$Year To Date`

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Mini-List|Selected Staff's Education|" ]

---

### Style: Notes List
Use case:
- notes history display with selected note highlighting

Expected `$List` row shape:
- `ID|Date|Time|Author|Notes`

Output shape:
- list container with clickable items
- selected state uses `$$Note ID`

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Notes List|Consultation Notes History|" ]

---

### Style: Master-Details
Use case:
- a master list dataset where each master item includes a nested details dataset

Master expected `$List` row shape:
- `MasterID|MasterName|ImageURL|Definition`

Details expected row shape (via `$Details Table SQL`):
- `DetailID|DetailName|DetailDescription`

Output shape:
- text containing JS object entries per master ID, each containing:
  - item_name
  - item_image_url
  - item_description
  - details array (or similar detail structure)

Hard rule (details description variable):
- use `$Details Description` in the generated details objects

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Master-Details|Specimen|<LabTestID>" ]

---

### Style: Grid
Use case:
- image + text card grid output as HTML

Expected `$List` row shape:
- `ID|Info1|Info2|Info3`

Behavior:
- fetches a base64 image by ID from `Images` table
- uses a placeholder base64 image when missing
- appends each ID to `$$Selected IDs`

Output shape:
- HTML blocks per grid item

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Grid|Selected Job Title's Staff|<JobTitleID>" ]

---

### Style: ID List
Use case:
- return an ID list and store it in `$$Selected IDs`

Expected `$List` row shape:
- one column: ID

Output shape:
- `ID¶ID¶ID¶`

Example call:
    Perform Script [ "Get Value List" ; Parameter: "ID List|Selected Specimen's Analytes|<LabTestSpecimenID>" ]

---

### Style: Hierarchical Form Setup
Use case:
- renders categories and nested question rows, used for template setup

Level 1 expected row shape:
- `CategoryID|Category|TotalQuestions`

Level 2 expected row shape:
- `QuestionID|Title|Question`

Output shape:
- HTML table rows with category rows and question rows

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Hierarchical Form Setup|Appraisal Template|<TemplateID>" ]

---

### Style: Hierarchical Form With Scores
Use case:
- same hierarchy as setup, with scoring-related UI markup

SQL shapes:
- same as Hierarchical Form Setup

Output shape:
- category rows plus question rows

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Hierarchical Form With Scores|Appraisal Template|<TemplateID>" ]

---

### Style: Table In First Card
Use case:
- dual-list UI where items are moved between two lists

Expected `$List` row shape:
- `ID|ItemLabel|Order|ParentLinkID`

Output shape:
- `<li ...>` items with move button, and optional indent for child items

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Table In First Card|Links|<RoleID>" ]

---

### Style: Table In Second Card
Same contract as Table In First Card, with the opposite move direction.

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Table In Second Card|Selected Role's Links|<RoleID>" ]

---

### Style: List for Receipt
Use case:
- build pipe-delimited rows for receipt printing or downstream processing

Expected `$List` row shape:
- `ID|Item Name|Cost`

Output shape:
- `ID|Name|Cost¶`

Example call:
    Perform Script [ "Get Value List" ; Parameter: "List for Receipt|Selected Appointment's Bill Items|<AppointmentID>" ]

---

### Style: Bill with Checkboxes
Use case:
- billing table rows with a checkbox per item, reflecting paid status

Expected `$List` row shape:
- `ID|Item Name|Cost|Paid Status`

Behavior:
- query logic changes based on `$$Service Type`

Output shape:
- HTML `<tr>` rows with checkbox state

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Bill with Checkboxes|Selected Appointment's Bill Items|<AppointmentID>" ]

---

### Style: JSON
Use case:
- returns JSON created from SQL result using `ExecuteSQLResultToJSON`

Output schema (confirmed):
~~~json
{
  "data": [
    { "Field1": "value", "Field2": 123, "Field3": true }
  ],
  "records": 1
}
~~~

Error schema:
~~~json
{
  "error": 1,
  "message": "Error Message"
}
~~~

Typing support:
- in the `fieldList`, append:
  - `|JSONNumber`
  - `|JSONBoolean`
- default is string

Example call:
    Perform Script [ "Get Value List" ; Parameter: "JSON|Specimen|" ]

---

### Style: Remunerations
Use case:
- builds a payslip-like HTML table and totals

Expected `$SelectedItem` values (confirmed):
- `Earnings`
- `Deductions`

Expected `$List` row shape:
- `Payroll Item Name|Rate|Quantity|Amount|Amount_YTD`

Output shape:
- full `<table class='payslip-table'>...</table>` with header and totals footer

Required totals initialization (to avoid carryover):
- `Set Variable [ $Total Amount ; Value: 0 ]`
- `Set Variable [ $Total YTD ; Value: 0 ]`

Example call:
    Perform Script [ "Get Value List" ; Parameter: "Remunerations|Staff Remunerations|Earnings" ]

---

## Placeholder Styles (Incomplete)
These styles exist but do not yet generate output rows:
- Table With Checkboxes
- Table with Images

AI Agent guidance:
- do not assume they work
- implement them fully before using them in new screens
- when implementing, define a strict row shape and a strict output shape similar to other styles

---

## Dependencies

### Custom Functions used
- ExecuteSQLResultToJSON
- GenerateFileIconHTML
- MySQLDateToFMPDateText
- DateInWordsShortMonth
- RelativeDateInWords
- ConvertLineBreakMarkersToReturns
- DefaultIfEmpty
- Singular

### Globals read
```
- $$App Name
- $$Material Category ID
- $$Job Title ID
- $$Patient ID
- $$Appointment ID
- $$App ID
- $$Equipment ID
- $$My Staff ID
- $$Remuneration ID
- $$Preferred Currency
- $$No Image URL
- $$Note ID
- $$Service Type
```

### Globals written
```
- $$Selected IDs
- $$Year To Date
```

AI Agent rule:
- any calling script that depends on a global must set it before calling `Get Value List`
- any caller that reads a global written here must not assume it is untouched by other calls

---

## Safe Extension Rules (For the AI Agent)

### Adding a new List Key
1. Choose the target `$Style` based on the required output shape.
2. Add a new `$ListKey` branch under that style in the SQL decision section.
3. Set `$SQL` or `$Data` using the expected row shape for that style.
4. Ensure the SQL column order exactly matches the style’s row contract.
5. Include `ORDER BY` for stable UI ordering.
6. Test:
   - empty result
   - one record
   - records containing special characters (especially for JS style)

### Changing the JS placeholder mapping
If you add or change a placeholder token:
- update this script mapping
- update the shared decode mapping in all callers
- add a quick regression test record that includes the affected character

### Recommendation for future improvements
- centralize the decode mapping in one shared WebViewer JS utility (single source of truth)
- gradually migrate JS outputs to strict JSON where practical, since JSON escaping is safer than manual placeholder encoding

---

# Layouts (CauferoAppStarter)

This document describes the **4 FileMaker layouts used for UI rendering** in `CauferoAppStarter.fmp12`.

## Overview

- **File name:** `CauferoAppStarter.fmp12`
- **Relationships:** None (no relationship graph)
- **Purpose:** A configurable starter file that can be set up to run as different solutions (hospital management, construction management, retail, inventory, and more).
- **UI strategy:** All UI is rendered using **Web Viewers**. The file uses **4 layouts** as UI containers:
  1. **Splash Screen**
  2. **Menu**
  3. **Page**
  4. **Modal**

---

# 1. Splash Screen Layout

**Layout name:** `Splash Screen`  
**Purpose:** Display the splash screen UI during application startup.

## 1.1 Web Viewer on Splash Screen Layout

The `Splash Screen` layout contains **one Web Viewer**.

#### 1) Splash Web Viewer

- **Purpose:** Renders the splash screen UI.
- **Render source:** `$$Splash`
  - `$$Splash` contains the complete HTML (and related CSS + JS) for the splash screen.

## 1.2 How the Splash Screen UI is Generated

#### Script that builds `$$Splash`

- **Script name:** `+++ Splash Page`
- **What it does (splash-specific):**
  - Pulls splash CSS by running `🖌️ Use Splash Screen CSS` and stores the returned CSS in `$styles`.
  - Builds the splash HTML fragment and stores it in `$HTML`.
  - Builds the splash JavaScript and stores it in `$Scripts`.
  - Assembles the final full-page HTML document and stores it in `$$Splash`.
  - Performs `WebDirect Pause`, then refreshes the window to ensure the Web Viewer renders the latest `$$Splash`.

## 1.3 How the Splash Screen Layout is Shown

- **Script name:** `Startup`
- **Splash rendering behavior (startup-specific):**
  - Goes to the `Splash Screen` layout at the start of the script.
  - If `$$App Name ≠ "The Bridge"`, it calls `+++ Splash Page` to populate `$$Splash`, then pauses briefly so the splash remains visible during startup.

## 1.4 Key Global Variables Used by the Splash Screen Layout

- `$$Splash`  
  Final HTML rendered by the Splash Web Viewer.

---

# 2. Menu Layout

**Layout name:** `Menu`  
**Role:** Displays the main menu UI for the application.

## 2.1 Web Viewer on Menu Layout

The `Menu` layout contains **one Web Viewer**.

#### 1) Menu Web Viewer

- **Purpose:** Renders the full menu UI.
- **Render source:** `$$Menu`
  - `$$Menu` contains the complete HTML (and related CSS + JS) for the menu.

### How the Menu UI is Generated

#### Script that builds `$$Menu`

- **Script name:** `+++ Menu Page`
- **What it does (menu-specific):**
  - Builds the menu links HTML and stores it in `$$My Links`.
  - Generates the final menu HTML document and stores it in `$$Menu`.
  - Pulls menu styling by running `🖌️ Use Menu CSS` and injecting the returned CSS into the menu HTML.
  - Builds support JS used by the menu, including functions that call FileMaker scripts from inside the Web Viewer.

#### Submenu generation used by the menu

- **Script name:** `Get Sub Menu Links 1`
  - Returns HTML for level-1 sub links for a given parent link.
  - If there are deeper sub links, it outputs an accordion-style structure.
- **Script name:** `Get Sub Menu Links 2`
  - Returns HTML for level-2 sub links for a given parent link.
  - Uses the same pattern as level-1 generation.

## 2.2 How the Menu Layout is Shown

- **Script name:** `Go To Menu`
- **Menu window behavior:**
  - Calls `+++ Menu Page` to rebuild `$$Menu`.
  - If `$$Menu Mode` is not `"Persistent Side Menu"`, a **new window** is opened using the `Menu` layout.
  - If `$$Menu Mode` is `"Persistent Side Menu"`, the menu is shown as part of the persistent UI flow (no separate menu window is created).

## 2.3 Selection State on the Menu

The menu HTML uses global selection variables to visually reflect the current navigation state:

- `$$Link ID`  
  Used to mark the currently selected link in the menu (HTML uses a `selected` class when the link’s ID matches `$$Link ID`).

- `$$Parent Link ID`  
  Used to keep the correct accordion section expanded when the selected item is inside a grouped (parent) section.

## 2.4 Key Global Variables Used by the Menu Layout

- `$$Menu`  
  Final HTML rendered by the Menu Web Viewer.

- `$$My Links`  
  HTML fragment containing the generated list of links (and accordion blocks) that is injected into `$$Menu`.

- `$$Link ID`  
  The selected link record ID, used by the menu to highlight the selected item.

- `$$Parent Link ID`  
  The selected parent link ID, used to control which accordion section is expanded.

- `$$Menu Mode`  
  Controls whether the menu appears in its own window or as a persistent side menu.


---

# 3. Page Layout

**Layout name:** `Page`  
**Role:** Hosts all **main application pages**. This is where the majority of the app UI renders.

## 3.1 Web Viewers on the Page Layout

The `Page` layout contains **two Web Viewers**:

### A) Banner Web Viewer

- **Position:** Lines the top of the layout.
- **Purpose:** Displays the page banner (top header area).
- **Render source:** Global variable `$$Banner`
  - `$$Banner` contains the code that renders the banner UI.
- **Where `$$Banner` is set:**
  - `$$Banner` is set in one script: `+++ Banner Page`

#### Banner title behavior

- The banner displays the **title of the current page**.
- The page title is stored in the global variable `$$Layout Name`.
- When a main menu link is clicked, the FileMaker script **`Select Link`** runs and:
  - sets `$$Layout Name` to the title of the selected page
  - calls `+++ Banner Page` so the banner always reflects the current page title
- While generating the banner code, the value inside `$$Layout Name` is inserted into the banner output (and then stored into `$$Banner`).

---

### B) Page Web Viewer

- **Position:** Covers the rest of the layout (main body area).
- **Purpose:** Displays the main body of the current page.
- **Render source:** Global variable `$$Page`
  - `$$Page` contains the code that renders the main page UI.
- **Where `$$Page` is set:**
  - `$$Page` is set by **different page-rendering scripts**.
  - Each page in the application typically has its own render script that sets `$$Page` to the correct HTML/CSS/JS for that page.

---

# 4. Modal Layout

**Layout name:** `Modal`  
**Purpose:** Display UI in a modal window (card-style window) for viewing or editing a selected record while staying in context of the current page.

## 4.1 Web Viewer on Modal Layout

The `Modal` layout contains **one Web Viewer**.

#### 1) Modal Web Viewer

- **Purpose:** Renders the modal UI.
- **Render source:** `$$Modal`
  - `$$Modal` contains the complete HTML (and related CSS + JS) for the modal window.

## 4.2 How Modal UI is Generated

`$$Modal` is not set by one central script. It is set by **many modal-specific scripts**, where each script is responsible for generating the UI for one specific modal use case.

Example pattern:
- A “selected record” ID is passed into a modal script (usually via `Get ( ScriptParameter )`).
- The script queries the database (usually via `ExecuteSQL`) to fetch the selected record’s details.
- The script builds the modal page HTML/CSS/JS, then stores the final HTML document in `$$Modal`.
- The script opens the modal by creating a **new card-style window** that uses the `Modal` layout.

## 4.3 Common Structure of a Modal Page Script

Most modal scripts follow this structure:

1. **Capture the selected record ID**
   - Store the selected record ID in a global variable (example: `$$KPI ID`, `$$Staff Education ID`, `$$Insurance Policy ID`, `$$Payroll Item ID`).

2. **Fetch record details**
   - Use `ExecuteSQL` to retrieve the record fields needed for the modal.

3. **(Optional) Fetch value lists**
   - If the modal needs dropdowns, call `Get Value List` and store results into local variables to inject into the HTML.

4. **Generate modal CSS**
   - Run `🖌️ Use Modal CSS` and store the returned CSS in `$styles`.
   - Some modals append extra CSS to `$styles` when needed (example: dropdown styling).

5. **Generate modal HTML**
   - Build `$HTML` containing the modal structure.
   - Standard modal layout pattern in HTML usually includes:
     - `.modal-header` for the title section
     - `.modal-body` for the main form/content

6. **Generate modal JavaScript**
   - Build JS functions that call FileMaker scripts using `FileMaker.PerformScript(...)`.
   - Common JS actions include:
     - **Cancel** (calls the close-window script)
     - **Save** (collects form values and calls a save script)
     - UI helpers (date pickers, segmented controls, toggles, etc.)

7. **Assemble the full modal page**
   - Combine `$styles`, `$HTML`, and `$Scripts` into a complete HTML document stored in `$$Modal`.

8. **Open the modal window**
   - Use:
     - `New Window [ Style: Card ; Using layout: “Modal” ... ]`
   - Modal size is set per modal script (height and width vary by use case).
   - Some modals call `WebDirect Pause` before opening the window (commonly for WebDirect stability).

## 4.4 Key Global Variables Used by the Modal Layout

- `$$Modal`  
  Final HTML rendered by the Modal Web Viewer.

- *(Per modal script)* `$$<Selected Record ID Variable>`  
  Each modal script typically sets its own global “selected record id” variable, specific to the entity being opened in the modal (for example `$$KPI ID` or `$$Staff Education ID`).


---

# Notes for App Builders

## One File, Many Apps

- New applications are developed **inside `CauferoAppStarter`**.
- Do **not** create a separate `.fmp12` file per new app.

## Planning UI Using Links

- Menu links guide what pages and modals you need to create.
- Example link: **“Staff Profiles”** may require:
  - Records List View Page (staff list)
  - Record Details View Page (selected staff)
  - Modal windows for expanding subtable records, for example:
    - Selected emergency contact person
    - Selected staff education history record
    - Selected staff remuneration payment
    - Selected staff attendance record
    - Selected staff KPI record
    - Selected staff document record

> Starter note: Script naming standards and the full “create a new application” workflow are documented elsewhere. This file focuses on layouts only.

---

# Splash Screen CSS Generator (CauferoAppStarter)

## Purpose
This document defines the canonical Splash Screen CSS skeleton and the rules an AI agent must follow when creating a new splash screen theme in the Splash Screen CSS generator script.

The AI agent must create a new theme by changing CSS values only, while keeping the exact same selectors and the exact same property names.

---

## Script Context
- Input: `$$Theme`
- Output: `$styles` (full CSS stylesheet string)
- The script ends with: `Exit Script [ Text Result: $styles ]`

Each theme branch does:
- `Set Variable [ $styles ; Value: "<FULL CSS STRING>" ]`

---

## Hard Rules (CSS Contract)
When creating a new Splash Screen theme:
1. Keep the exact same selectors.
2. Keep the exact same property names under each selector.
3. Change values only.

Never:
- add selectors
- remove selectors
- rename selectors
- add properties
- remove properties

If a requested visual change would require adding selectors or properties, do not implement it. Achieve the best possible result using value changes only.

---

## Canonical Splash Screen CSS Skeleton (Source Of Truth)
This is the real Splash Screen CSS skeleton that the AI must copy and edit for every new theme.

Notes:
- This CSS is designed for a FileMaker WebViewer.
- It uses layered background elements: `.background`, `.overlay`, `.colour`, and foreground content.
- The z-index ordering must remain intact for correct layering.

~~~css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Open Sans', sans-serif;
}

body, html {
  margin: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  overflow: hidden;
  background: linear-gradient(135deg, #2f2e41, #3b3a58);
}

.background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: hue-rotate(-10deg) saturate(1.08);
  z-index: 1;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(47, 46, 65, 0.52);
  z-index: 2;
}

.colour {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    to top,
    rgba(47, 46, 65, 1) 34%,
    rgba(47, 46, 65, 0.74) 68%,
    rgba(47, 46, 65, 0.32) 82%,
    rgba(0, 0, 0, 0) 92%
  );
  z-index: 3;
}

.container {
  text-align: center;
  max-width: 100%;
  padding: 20px;
  margin-bottom: 0px;
  position: relative;
  z-index: 4;
}

.title {
  font-size: 5em;
  font-weight: bold;
  text-transform: uppercase;
  font-family: 'Montserrat', sans-serif;
  letter-spacing: 1.2px;
  background: linear-gradient(90deg, #ff6a2a, #ffb020);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.22);
  margin-top: 10%;
  margin-bottom: 10px;
  filter: brightness(1.25);
}

.subtitle {
  font-size: 1.5em;
  font-family: 'Open Sans', sans-serif;
  color: #e5e7eb;
  letter-spacing: 0.6px;
  margin-bottom: 50px;
}

.partners {
  font-size: 0.95em;
  color: #ffffff;
  opacity: 0.88;
  margin-bottom: 20%;
}

.loading-bar {
  width: 50%;
  height: 10px;
  background-color: rgba(255, 255, 255, 0.28);
  border-radius: 5px;
  overflow: hidden;
  margin: 26px auto 10px;
  position: relative;
  z-index: 4;
}

.loading-bar-inner {
  width: 0;
  height: 100%;
  background-color: #66bb6a;
  border-radius: 5px;
  animation: loading 2s infinite;
}

@keyframes loading {
  0% { width: 0; }
  50% { width: 50%; }
  100% { width: 100%; }
}

.loading-message {
  font-size: 0.9em;
  color: #66bb6a;
  margin-bottom: 14px;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.32);
  position: relative;
  z-index: 4;
}

.footer {
  font-size: 0.8em;
  color: #d7dde6;
  text-align: center;
  margin-bottom: 0px;
  position: relative;
  z-index: 4;
}

.partners .developers {
  font-weight: bold;
  color: #66bb6a;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.32);
}
~~~

---

## Contract Manifest (Selectors and Property Names)
The AI agent must ensure every new theme keeps these exact selectors and property names.

- `*`
  - box-sizing, margin, padding, font-family

- `body, html`
  - margin, width, height, display, flex-direction, justify-content, align-items, position, overflow, background

- `.background`
  - position, top, left, width, height, object-fit, filter, z-index

- `.overlay`
  - position, top, left, width, height, background, z-index

- `.colour`
  - position, top, left, width, height, background, z-index

- `.container`
  - text-align, max-width, padding, margin-bottom, position, z-index

- `.title`
  - font-size, font-weight, text-transform, font-family, letter-spacing, background, -webkit-background-clip, -webkit-text-fill-color, text-shadow, margin-top, margin-bottom, filter

- `.subtitle`
  - font-size, font-family, color, letter-spacing, margin-bottom

- `.partners`
  - font-size, color, opacity, margin-bottom

- `.loading-bar`
  - width, height, background-color, border-radius, overflow, margin, position, z-index

- `.loading-bar-inner`
  - width, height, background-color, border-radius, animation

- `@keyframes loading`
  - 0% width, 50% width, 100% width

- `.loading-message`
  - font-size, color, margin-bottom, text-shadow, position, z-index

- `.footer`
  - font-size, color, text-align, margin-bottom, position, z-index

- `.partners .developers`
  - font-weight, color, text-shadow

---

## Theme Creation Procedure
When creating a new theme:
1. Create a new theme name (unique, Title Case).
2. Copy the canonical Splash Screen CSS skeleton exactly.
3. Change values only, mainly:
   - background gradient in `body, html`
   - image filter values in `.background`
   - overlay darkness and colour fade values in `.overlay` and `.colour`
   - title gradient colors in `.title`
   - title and layout spacing values (for example: `.title` margin-top, `.subtitle` margin-bottom, `.partners` margin-bottom)
   - accent color used in `.loading-bar-inner`, `.loading-message`, and `.partners .developers`
   - text colors and opacities for `.subtitle`, `.partners`, `.footer`
4. Ensure it looks professional, sharp, modern, cohesive, and remains readable on top of the background image.

---

## FileMaker Script Branch Template
Add a new branch to the Splash Screen CSS generator script:

~~~text
Else If [ $$Theme = "New Theme Name" ]
  Set Variable [ $styles ; Value: "PASTE FULL UPDATED CSS STRING HERE" ]
End If
~~~

---

## AI Request Template
Create a new Splash Screen theme using the canonical Splash Screen CSS in this document.

Theme Name: <New Theme Name>
Mode: <Light or Dark>
Vibe Keywords: <example: premium, modern, cohesive>

Rules:
- Copy the canonical Splash Screen CSS exactly.
- Change values only.
- Do not add, remove, or rename selectors.
- Do not add or remove properties.
- Do not make sizes bigger. Make sizes similar to the original values or smaller.

Deliver:
1. Full updated CSS string
2. Full FileMaker `Else If` branch for the new theme

---

# Banner Page CSS Generator (CauferoAppStarter)

## Purpose
This document defines the canonical Banner page CSS skeleton and the rules an AI agent must follow when creating a new theme in the Banner CSS generator script.

The AI agent must create a new theme by changing CSS values only, while keeping the exact same selectors and the exact same property names.

---

## Script Context
- Input: `$$Theme`
- Output: `$styles` (full CSS stylesheet string)
- The script ends with: `Exit Script [ Text Result: $styles ]`

Each theme branch does:
- `Set Variable [ $styles ; Value: "<FULL CSS STRING>" ]`

---

## Hard Rules (CSS Contract)
When creating a new Banner theme:
1. Keep the exact same selectors.
2. Keep the exact same property names under each selector.
3. Change values only.

Never:
- add selectors
- remove selectors
- rename selectors
- add properties
- remove properties

If a requested visual change would require adding selectors or properties, do not implement it. Achieve the best possible result using value changes only.

---

## Canonical Banner CSS Skeleton (Source Of Truth)
This is the real Banner CSS skeleton that the AI must copy and edit for every new theme.

~~~css
/* FileMaker WebViewer-safe base.
   html/body are forced to 100% so the layout references the WebViewer height, not the device viewport. */
html, body {
  height: 100%;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Open Sans', sans-serif;
}

/* Use 100% instead of 100vh for embedded WebViewer stability. */
body {
  margin: 0;
  height: 100%;
  overflow: hidden;
  background: linear-gradient(135deg, #121423, #1d2134);
}

/* Banner fills the WebViewer height.
   In FileMaker, control the banner thickness by resizing the WebViewer object. */
.banner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 18px;
  width: 100%;
  height: 100%;
  background: linear-gradient(145deg, #1a1d2f, #252a43);
  color: #f8fafc;
  position: relative;
  box-shadow: 0 10px 26px rgba(0, 0, 0, 0.35);
  overflow: hidden;
}

/* Keep the gloss subtle for a modern premium look.
   If it ever washes out contrast, reduce opacity slightly. */
.banner::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    75deg,
    rgba(255, 255, 255, 0.18) 18%,
    rgba(255, 255, 255, 0.08) 48%,
    rgba(255, 255, 255, 0) 78%
  );
  opacity: 0.35;
  pointer-events: none;
  transform: rotate(-10deg) translateY(-10%);
  mix-blend-mode: screen;
}

.banner-logo {
  display: flex;
  align-items: center;
  gap: 12px;
  position: relative;
}

.logo-container {
  position: relative;
  width: 40px;
  height: 40px;
  cursor: pointer;
}

.banner-logo img, .list-icon {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  position: absolute;
  top: 0;
  left: 0;
  transition: opacity 0.3s ease, transform 0.3s ease;
  box-shadow: 0 10px 18px rgba(0, 0, 0, 0.28);
}

.banner-logo img {
  background: #ffffff;
  padding: 3px;
  z-index: 2;
}

.list-icon {
  background: #22c55e;
  color: #ffffff;
  width: 36px;
  height: 36px;
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  opacity: 0;
  transform: scale(0.92);
  z-index: 1;
}

.logo-container:hover img {
  opacity: 0;
  transform: scale(0.92);
}

.logo-container:hover .list-icon {
  opacity: 1;
  transform: scale(1);
}

/* Kept vivid but controlled. This is already in a good modern range. */
.banner-logo .logo-text {
  font-size: 18px;
  font-family: 'Montserrat', sans-serif;
  font-weight: 650;
  background: linear-gradient(90deg, #ff6a2a, #ffb020);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-transform: uppercase;
  letter-spacing: 1.1px;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.22);
  filter: brightness(1.25);
}

.banner-logo .logo-subtitle {
  font-size: 12px;
  color: #cbd5e1;
  opacity: 0.85;
  letter-spacing: 0.6px;
  margin-top: 2px;
}

/* Clear accent, compact sizing. Works well for a thin banner. */
.banner-title {
  font-size: 15px;
  font-weight: 800;
  text-transform: uppercase;
  color: #22c55e;
  text-shadow: 0 2px 3px rgba(0, 0, 0, 0.32);
  letter-spacing: 0.4px;
  margin-right: 12px;
  display: flex;
  align-items: center;
  gap: 12px;
}

.notification-icon, .chat-icon {
  position: relative;
  width: 32px;
  height: 32px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.10);
  border-radius: 50%;
  box-shadow: 0 10px 18px rgba(0, 0, 0, 0.22);
  transition: transform 0.2s ease, background-color 0.2s ease;
}

.notification-icon:hover, .chat-icon:hover {
  background: rgba(255, 255, 255, 0.16);
  transform: scale(1.08);
}

.notification-icon svg, .chat-icon svg {
  width: 20px;
  height: 20px;
  fill: #ffffff;
  transition: fill 0.2s ease;
}

/* Cohesive: hover ties back to the same accent green used in the header. */
.notification-icon:hover svg, .chat-icon:hover svg {
  fill: #22c55e;
}

.notification-icon .alert-badge, .chat-icon .alert-badge {
  position: absolute;
  top: -5px;
  right: -5px;
  background: #ef4444;
  color: #ffffff;
  width: 16px;
  height: 16px;
  font-size: 12px;
  font-weight: bold;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.25);
}

~~~

---

## Contract Manifest (Selectors and Property Names)
The AI agent must ensure every new theme keeps these exact selectors and property names.

- `html, body`
  - height

- `*`
  - box-sizing, margin, padding, font-family

- `body`
  - margin, height, overflow, background

- `.banner`
  - display, align-items, justify-content, padding, width, height, background, color, position, box-shadow, overflow

- `.banner::before`
  - content, position, top, left, width, height, background, opacity, pointer-events, transform, mix-blend-mode

- `.banner-logo`
  - display, align-items, gap, position

- `.logo-container`
  - position, width, height, cursor

- `.banner-logo img, .list-icon`
  - width, height, border-radius, position, top, left, transition, box-shadow

- `.banner-logo img`
  - background, padding, z-index

- `.list-icon`
  - background, color, width, height, font-size, display, align-items, justify-content, cursor, opacity, transform, z-index

- `.logo-container:hover img`
  - opacity, transform

- `.logo-container:hover .list-icon`
  - opacity, transform

- `.banner-logo .logo-text`
  - font-size, font-family, font-weight, background, -webkit-background-clip, -webkit-text-fill-color, text-transform, letter-spacing, text-shadow, filter

- `.banner-logo .logo-subtitle`
  - font-size, color, opacity, letter-spacing, margin-top

- `.banner-title`
  - font-size, font-weight, text-transform, color, text-shadow, letter-spacing, margin-right, display, align-items, gap

- `.notification-icon, .chat-icon`
  - position, width, height, cursor, display, align-items, justify-content, background, border-radius, box-shadow, transition

- `.notification-icon:hover, .chat-icon:hover`
  - background, transform

- `.notification-icon svg, .chat-icon svg`
  - width, height, fill, transition

- `.notification-icon:hover svg, .chat-icon:hover svg`
  - fill

- `.notification-icon .alert-badge, .chat-icon .alert-badge`
  - position, top, right, background, color, width, height, font-size, font-weight, border-radius, display, align-items, justify-content, box-shadow

---

## Theme Creation Procedure
When creating a new theme:
1. Create a new theme name (unique, Title Case).
2. Copy the canonical CSS skeleton exactly.
3. Change values only, mainly:
   - gradient colors in `body` and `.banner`
   - overlay gradient colors and opacity in `.banner::before`
   - highlight colors in `.list-icon`, `.banner-title`, icon backgrounds, hover fills
   - shadows and opacities
   - font-family values (only where they already exist)
4. Ensure it looks professional, sharp, modern, cohesive.

---

## FileMaker Script Branch Template
Add a new branch to the Banner CSS generator script:

~~~text
Else If [ $$Theme = "New Theme Name" ]
  Set Variable [ $styles ; Value: "PASTE FULL UPDATED CSS STRING HERE" ]
End If
~~~

---

## AI Request Template
Create a new Banner theme using the canonical Banner CSS in this document.

Theme Name: <New Theme Name>
Mode: <Light or Dark>
Vibe Keywords: <example: premium, modern, cohesive>

Rules:
- Copy the canonical Banner CSS exactly.
- Change values only.
- Do not add, remove, or rename selectors.
- Do not add or remove properties.
- Do not make sizes bigger. Make sizes similar to the original values or smaller.

Deliver:
1. Full updated CSS string
2. Full FileMaker `Else If` branch for the new theme

---

# Dashboard CSS Generator (CauferoAppStarter)

## Purpose
This document defines the canonical Dashboard CSS skeleton and the rules an AI agent must follow when creating a new Dashboard theme in CauferoAppStarter.

The AI agent must create a new theme by changing CSS values only, while keeping the exact same selectors and the exact same property names.

This CSS is used for dashboard widgets like:
- KPI cards
- Chart cards (pie/doughnut charts, labels, legends)
- Counters
- Progress bars and progress circles
- Tables and gantt charts
- Timeline UI elements

---

## Script Context
- Input: `$$Theme` (or other theme selector used in your CSS script)
- Output: `$styles` (full CSS stylesheet string)
- The script ends with: `Exit Script [ Text Result: $styles ]`

Important dynamic blocks inside the CSS string:
- The `grid-template-columns` value in `.cards` is built using FileMaker logic:
  - `grid-template-columns: repeat(auto-fit, minmax(<calculated px>, 1fr));`
- The KPI card color definitions are injected using:
  - `" & $$KPI Card Colours & "`

The AI must preserve these dynamic FileMaker insertions exactly.

---

## Hard Rules (CSS Contract)
When creating a new Dashboard theme:
1. Keep the exact same selectors.
2. Keep the exact same property names under each selector.
3. Change values only.

Never:
- add selectors
- remove selectors
- rename selectors
- add properties
- remove properties
- remove or change FileMaker merge insertions such as `" & $$KPI Card Colours & "` or the `Case ( $$Menu Mode ... )` expression inside `.cards`

If a requested visual change would require adding selectors or properties, do not implement it. Achieve the best possible result using value changes only.

---

## Canonical Dashboard CSS Skeleton (Source Of Truth)
This is the canonical dashboard skeleton that the AI must copy and edit for every new dashboard theme.

Notes:
- Keep the `$$Menu Mode` calculation inside `.cards` exactly as-is.
- Keep `" & $$KPI Card Colours & "` exactly as-is.

~~~css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

body {
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(145deg, #D1D9E6, #A4B6D0);
  color: #333;
  padding: 20px;
  overflow: auto;
}

.chart-card {
  background: rgba(247, 249, 249, 1);
  backdrop-filter: blur(6px);
  border-radius: 12px;
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
  padding: 15px;
  text-align: center;
}

.chart-card h2 {
  color: #003A70;
  font-size: 1.2em;
  margin-bottom: 10px;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.chart-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
}

canvas {
  max-width: 100%;
  height: auto;
}

.chart-label, .chart-doughnut_label {
  position: absolute;
  text-align: center;
  font-size: 1em;
  font-weight: bold;
  z-index: 1;
}

.chart-label {
  color: white;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.6);
}

.chart-doughnut_label {
  color: #003A70;
}

.chart-label span {
  font-size: 0.7em;
  font-weight: normal;
  display: block;
  margin-top: 5px;
  color: white;
}

.legend-custom {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 5px;
  margin-top: 5px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #666;
  font-size: 0.8em;
}

.legend-color {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  display: inline-block;
  background-color: #aaa;
}

.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(" & Case ( $$Menu Mode = "Persistent Side Menu" ; 100 ; 180 ) & "px, 1fr));
}

.kpi-card {
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 12px;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
  padding: 15px;
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  overflow: hidden;
  transition: background-color 0.3s, transform 0.3s, box-shadow 0.3s;
  text-decoration: none;
  color: inherit;
}

" & $$KPI Card Colours & "

.kpi-card svg {
  width: 40px;
  height: 40px;
  margin-bottom: 10px;
  transition: transform 0.3s ease, fill 0.3s ease;
  fill: white;
}

.kpi-card:hover svg {
  transform: scale(1.1);
}

.kpi-card h3 {
  font-size: 1em;
  margin-bottom: 5px;
  z-index: 1;
  font-weight: 500;
  color: white;
}

.kpi-card .value {
  font-size: 1.8em;
  font-weight: bold;
  margin-bottom: 5px;
  z-index: 1;
  color: white;
}

.kpi-card p {
  font-size: 0.8em;
  z-index: 1;
  color: white;
}

a {
  text-decoration: none;
}

.gauge-labels {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: -40%;
  padding: 0 20px;
  font-size: 0.9em;
  color: #666;
}

.gauge_performance-details {
  margin-top: 8%;
  font-size: 0.85em;
  color: #666;
}

.counter {
  font-size: 3em;
  font-weight: bold;
  margin: 20px 0;
  color: #0057A3;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.counter_details {
  font-size: 0.9em;
  color: #888;
}

.progress_bar-container {
  position: relative;
  height: 20px;
  width: 100%;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 10px;
  overflow: hidden;
  margin: 10px 0 20px;
}

.progress-bar {
  height: 100%;
  width: 0;
  background: linear-gradient(90deg, #003A70 0%, #0057A3 100%);
  border-radius: 10px 0 0 10px;
  transition: width 0.2s ease-in-out;
}

.progress_bar-label {
  font-size: 0.9em;
  margin-top: 5px;
  color: #888;
}

.progress-circle {
  position: relative;
  width: 150px;
  height: 150px;
  margin: 0 auto;
}

.progress-circle canvas {
  position: absolute;
  top: 0;
  left: 0;
}

.progress-circle .progress_circle-value {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 2em;
  font-weight: bold;
  color: #28A745;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.progress_circle_details {
  font-size: 0.9em;
  color: #666;
  margin-top: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
  margin-top: 10px;
  font-size: 0.85em;
}

th, td {
  padding: 8px 10px;
}

thead {
  background: rgba(0, 58, 112, 0.1);
}

th {
  color: #003A70;
  font-weight: bold;
  border-bottom: 1px solid rgba(0, 58, 112, 0.2);
}

tbody tr {
  transition: background 0.2s;
}

tbody tr:nth-child(even) {
  background: rgba(0, 87, 163, 0.05);
}

tbody tr:hover {
  background: rgba(0, 87, 163, 0.1);
}

td {
  color: #0057A3;
}

@media screen and (max-width: 600px) {
  th, td {
    padding: 6px 8px;
    font-size: 0.75em;
  }
}

.gantt-chart {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
}

.gantt-chart th, .gantt-chart td {
  padding: 10px;
  font-size: 0.85em;
}

.gantt-chart thead {
  background: rgba(0, 58, 112, 0.1);
}

.gantt-chart th {
  color: #003A70;
  font-weight: bold;
  text-align: left;
  border-bottom: 1px solid rgba(0, 58, 112, 0.2);
}

.gantt-chart td {
  color: #0057A3;
  border-bottom: 1px solid rgba(0, 87, 163, 0.1);
  position: relative;
  vertical-align: middle;
}

.gantt_task-bar {
  position: absolute;
  top: 50%;
  left: 0;
  transform: translateY(-50%);
  height: 30px;
  border-radius: 4px;
  background: linear-gradient(90deg, #003A70, #0057A3);
  display: flex;
  align-items: center;
  justify-content: flex-end;
  padding-right: 10px;
  color: #FFFFFF;
  font-weight: bold;
  font-size: 0.8em;
  white-space: nowrap;
  overflow: hidden;
}

.gantt_task-bar.delay {
  background: linear-gradient(90deg, #DC3545, #FF6A6A);
}

.gantt_task-label {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  z-index: 2;
  left: 5px;
  font-size: 0.8em;
  font-weight: bold;
  color: #003A70;
}

.timeline-item {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100px;
}

.timeline-point {
  width: 30px;
  height: 30px;
  background: linear-gradient(90deg, #E9EBEF 0%, #F7F9F9 100%);
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 0.9em;
  font-weight: bold;
  color: grey;
  transition: background 0.3s ease;
}

.timeline-point.active {
  background: linear-gradient(90deg, #28A745 0%, #0057A3 100%);
  color: white;
}

.timeline-label {
  margin-top: 10px;
  font-size: 0.85em;
  color: #777777;
  text-align: center;
}

.timeline-line {
  position: absolute;
  top: 15px;
  left: 10px;
  right: 10px;
  height: 5px;
  background: linear-gradient(90deg, #E9EBEF 0%, #F7F9F9 100%);
}

.timeline-fill {
  height: 100%;
  width: 0;
  background: linear-gradient(90deg, #28A745 0%, #0057A3 100%);
  transition: width 0.3s ease;
}
~~~

---

## Contract Manifest (Selectors and Property Names)
The AI agent must ensure every new theme keeps these exact selectors and property names.

- `*`
  - box-sizing, margin, padding, font-family

- `body`
  - display, justify-content, align-items, background, color, padding, overflow

- `.chart-card`
  - background, backdrop-filter, border-radius, box-shadow, padding, text-align

- `.chart-card h2`
  - color, font-size, margin-bottom, text-shadow

- `.chart-container`
  - position, display, justify-content, align-items, margin-bottom

- `canvas`
  - max-width, height

- `.chart-label, .chart-doughnut_label`
  - position, text-align, font-size, font-weight, z-index

- `.chart-label`
  - color, text-shadow

- `.chart-doughnut_label`
  - color

- `.chart-label span`
  - font-size, font-weight, display, margin-top, color

- `.legend-custom`
  - display, justify-content, flex-wrap, gap, margin-top

- `.legend-item`
  - display, align-items, gap, color, font-size

- `.legend-color`
  - width, height, border-radius, display, background-color

- `.cards`
  - display, grid-template-columns

- `.kpi-card`
  - border, border-radius, box-shadow, padding, text-align, display, flex-direction, justify-content, align-items, position, overflow, transition, text-decoration, color

- `.kpi-card svg`
  - width, height, margin-bottom, transition, fill

- `.kpi-card:hover svg`
  - transform

- `.kpi-card h3`
  - font-size, margin-bottom, z-index, font-weight, color

- `.kpi-card .value`
  - font-size, font-weight, margin-bottom, z-index, color

- `.kpi-card p`
  - font-size, z-index, color

- `a`
  - text-decoration

- `.gauge-labels`
  - display, justify-content, align-items, margin-top, padding, font-size, color

- `.gauge_performance-details`
  - margin-top, font-size, color

- `.counter`
  - font-size, font-weight, margin, color, text-shadow

- `.counter_details`
  - font-size, color

- `.progress_bar-container`
  - position, height, width, background, border-radius, overflow, margin

- `.progress-bar`
  - height, width, background, border-radius, transition

- `.progress_bar-label`
  - font-size, margin-top, color

- `.progress-circle`
  - position, width, height, margin

- `.progress-circle canvas`
  - position, top, left

- `.progress-circle .progress_circle-value`
  - position, top, left, transform, font-size, font-weight, color, text-shadow

- `.progress_circle_details`
  - font-size, color, margin-top

- `table`
  - width, border-collapse, text-align, margin-top, font-size

- `th, td`
  - padding

- `thead`
  - background

- `th`
  - color, font-weight, border-bottom

- `tbody tr`
  - transition

- `tbody tr:nth-child(even)`
  - background

- `tbody tr:hover`
  - background

- `td`
  - color

- `@media screen and (max-width: 600px) th, td`
  - padding, font-size

- `.gantt-chart`
  - width, border-collapse, text-align

- `.gantt-chart th, .gantt-chart td`
  - padding, font-size

- `.gantt-chart thead`
  - background

- `.gantt-chart th`
  - color, font-weight, text-align, border-bottom

- `.gantt-chart td`
  - color, border-bottom, position, vertical-align

- `.gantt_task-bar`
  - position, top, left, transform, height, border-radius, background, display, align-items, justify-content, padding-right, color, font-weight, font-size, white-space, overflow

- `.gantt_task-bar.delay`
  - background

- `.gantt_task-label`
  - position, top, transform, z-index, left, font-size, font-weight, color

- `.timeline-item`
  - position, display, flex-direction, align-items, width

- `.timeline-point`
  - width, height, background, border-radius, display, justify-content, align-items, font-size, font-weight, color, transition

- `.timeline-point.active`
  - background, color

- `.timeline-label`
  - margin-top, font-size, color, text-align

- `.timeline-line`
  - position, top, left, right, height, background

- `.timeline-fill`
  - height, width, background, transition

---

## Theme Creation Procedure
When creating a new theme:
1. Create a new theme name (unique, Title Case).
2. Copy the canonical Dashboard CSS skeleton exactly.
3. Change values only, mainly:
   - page background gradient and base text color in `body`
   - card backgrounds, blur strength, border radii, shadows (`.chart-card`, `.kpi-card`)
   - heading colors and text shadows (`.chart-card h2`, `.counter`)
   - neutral text colors used across legends, labels, details (`.legend-item`, `.gauge-labels`, `.counter_details`, `.progress_bar-label`, `.progress_circle_details`)
   - progress gradients and accent colors (`.progress-bar`, `.progress-circle .progress_circle-value`)
   - table header and row background opacities (`thead`, `tbody tr:nth-child(even)`, `tbody tr:hover`)
   - gantt bar gradients and delay color (`.gantt_task-bar`, `.gantt_task-bar.delay`)
   - timeline inactive and active gradients (`.timeline-point`, `.timeline-point.active`, `.timeline-fill`)
4. Ensure it looks professional, sharp, compact, modern, cohesive, and remains readable.

---

## FileMaker Script Branch Template
Add a new branch to the Dashboard CSS generator script:

~~~text
Else If [ $$Theme = "New Theme Name" ]
  Set Variable [ $styles ; Value: "PASTE FULL UPDATED CSS STRING HERE (KEEP ALL FILEMAKER INSERTIONS INTACT)" ]
End If
~~~

---

## AI Request Template
Create a new Dashboard theme using the canonical Dashboard CSS in this document.

Theme Name: <New Theme Name>
Mode: <Light or Dark>
Vibe Keywords: <example: premium, compact, modern, cohesive>

Rules:
- Copy the canonical Dashboard CSS exactly.
- Change values only.
- Do not add, remove, or rename selectors.
- Do not add or remove properties.
- Do not remove or change FileMaker merge insertions (for example `" & $$KPI Card Colours & "` and the `.cards` grid-template-columns expression).

Deliver:
1. Full updated CSS string
2. Full FileMaker `Else If` branch for the new theme

---

# Menu CSS Generator (CauferoAppStarter)

## Purpose
This document defines the canonical Menu CSS skeleton and the rules an AI agent must follow when creating a new Menu theme in CauferoAppStarter.

The AI agent must create a new theme by changing CSS values only, while keeping the exact same selectors and the exact same property names.

This CSS styles the menu WebViewer UI, including:
- Sidebar container
- Top-right utility icons (reset password, close menu)
- User avatar block and role text
- Search input
- Accordion links, panels, and submenu behavior
- Selected menu item state
- Notification widget
- Bottom buttons (Close Application, Change Theme)

---

## Script Context
- Input: `$$Theme` (or other theme selector used in your Menu CSS generator script)
- Output: `$styles` (full CSS stylesheet string)
- The script ends with: `Exit Script [ Text Result: $styles ]`

---

## Hard Rules (CSS Contract)
When creating a new Menu theme:
1. Keep the exact same selectors.
2. Keep the exact same property names under each selector.
3. Change values only.

Never:
- add selectors
- remove selectors
- rename selectors
- add properties
- remove properties

If a requested visual change would require adding selectors or properties, do not implement it. Achieve the best possible result using value changes only.

---

## Canonical Menu CSS Skeleton (Source Of Truth)
This is the canonical menu skeleton that the AI must copy and edit for every new menu theme.

~~~css
/* General reset and body styling */
* {
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

body {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  padding: 20px;
  overflow: hidden;
}

.sidebar {
  width: auto;
  height: auto;
  position: fixed;
  inset: 0;
  display: flex;
  flex-direction: column;
  padding: 20px;
  overflow: hidden;
  border-right: 1px solid rgba(0,0,0,0.06);
  background: linear-gradient(
    -45deg,
    #f5fcfb 0%,
    #f5fcfb 50%,
    #ffffff 50%,
    #ffffff 100%
  );
}

.menu_links {
  overflow-y: auto;
}

.sidebar a:not(.selected), .sidebar .selected {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #34495e;
  text-decoration: none;
  font-size: 1em;
  transition: color 0.3s;
  padding: 10px 10px;
}

.reset-password-icon,
.close-menu {
  position: absolute;
  top: 20px;
  cursor: pointer;
  color: #2C3E50;
  transition: color 0.3s;
}

.reset-password-icon {
  right: 50px;
  font-size: 0.8em;
}

.close-menu {
  right: 20px;
  font-size: 1em;
}

.reset-password-icon:hover,
.close-menu:hover {
  color: #3AAEA5;
}

/* User Avatar section */
.user-avatar {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #34495e;
}

.user-avatar img {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  border: 2px solid #bdc3c7;
  object-fit: cover;
}

.user-name {
  font-size: 1.1em;
  font-weight: bold;
}

.user-role {
  font-size: 0.9em;
  color: rgba(52, 73, 94, 0.8);
}

.divider {
  border: none;
  height: 1px;
  background: linear-gradient(to right, #3AAEA5, #e9f5f8, #3AAEA5);
  margin: 15px 0;
  border-radius: 1px;
  box-shadow: 0px 1px 3px rgba(0, 0, 0, 0.1);
}

.menu-search-container {
  margin-top: 20px;
  margin-bottom: 5px;
}

.menu-search-input {
  width: 100%;
  padding: 12px 16px;
  font-size: 1em;
  border: 1px solid #b3e5ea;
  border-radius: 8px;
  box-sizing: border-box;
  outline: none;
  color: #34495e;
  background-color: #ffffff;
  transition: border-color 0.3s, box-shadow 0.3s;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.05);
}

.menu-search-input:focus {
  border-color: #3AAEA5;
  box-shadow: 0 0 0 3px rgba(58, 174, 165, 0.2);
}

/* Accordion styling */
.accordion {
  width: 100%;
  background: none;
  color: #34495e;
  font-size: 1em;
  text-align: left;
  outline: none;
  cursor: pointer;
  padding: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  transition: color 0.3s;
  border: none;
}

.accordion:hover {
  color: #2C3E50;
  background-color: #e9f5f8;
  border-radius: 8px;
}

.sidebar .selected {
  background-color: #cce5ff;
  color: #004a99;
  font-weight: 600;
  border-radius: 8px;
  box-shadow: inset 0 0 0 2px #3AAEA5,
              0 2px 6px rgba(0,0,0,0.08);
}

/* Panel without height restriction */
.panel {
  overflow: hidden;
  padding-left: 20px;
  border-radius: 5px;
  margin-bottom: 10px;
  display: none;
}

.panel.show {
  display: block;
}

/* Sub-menu links */
.panel a {
  padding: 8px 10px;
  padding-left: 20px;
  color: #34495e;
  text-decoration: none;
  transition: color 0.3s;
  gap: 10px;
}

.panel a:hover {
  color: #2C3E50;
}

.menu_links {
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: #3AAEA5 #e9f5f8;
}

.link-left {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 1em;
  color: #34495e;
  transition: color 0.3s;
}

.link-left:hover {
  color: #3AAEA5;
}

.arrow {
  width: 10px;
  height: 10px;
  transition: transform 0.3s ease, fill 0.3s;
  fill: #b3e5ea;
}

.expanded .arrow {
  transform: rotate(90deg);
  fill: #3AAEA5;
}

.submenu {
  display: none;
  padding-left: 20px;
  margin-top: 6px;
  background: rgba(233, 245, 248, 0.8);
  border-left: 3px solid #3AAEA5;
  border-radius: 6px;
}

.expanded .submenu {
  display: block;
  box-shadow: 0px 2px 8px rgba(0, 0, 0, 0.1);
}

/* SVG Icon Styling */
.icon {
  width: 18px;
  height: 18px;
  fill: #3AAEA5;
  transition: fill 0.3s;
}

.accordion:hover .icon,
.selected .icon {
  fill: #007BFF;
}

.notification-area {
  background: rgba(233, 245, 248, 0.9);
  border: 1px solid #b3e5ea;
  border-radius: 12px;
  padding: 16px;
  box-shadow: 0px 6px 15px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  transition: box-shadow 0.3s, transform 0.3s;
}

.notification-area:hover {
  box-shadow: 0px 8px 20px rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.notification-header {
  font-weight: bold;
  color: #3AAEA5;
  margin-bottom: 8px;
}

.notification-content {
  font-size: 0.9em;
  color: #34495e;
  display: flex;
  align-items: center;
  gap: 10px;
}

.notification-content i {
  font-size: 1.4em;
  color: #3AAEA5;
}

.notification-actions {
  display: flex;
  justify-content: space-between;
  margin-top: 12px;
  font-size: 0.8em;
  font-weight: bold;
}

.notification-actions a {
  color: #3AAEA5;
  text-decoration: none;
  cursor: pointer;
  transition: color 0.3s, text-shadow 0.3s;
}

.notification-actions a:hover {
  color: #2C3E50;
  text-shadow: 0px 1px 2px rgba(0, 0, 0, 0.1);
  text-decoration: underline;
}

.bottom_buttons {
  display: flex;
  flex-direction: column;
  gap: 2px;
  flex-wrap: wrap;
}

/* Close Application Button */
.close-application {
  margin-bottom: 20px;
  justify-content: center;
  background: #3AAEA5;
  color: #ffffff;
  border: none;
  padding: 10px;
  border-radius: 5px;
  width: 100%;
  cursor: pointer;
  font-size: 1em;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: background 0.3s, color 0.3s;
}

.close-application:hover {
  background: #2C3E50;
}

.close_application_icon {
  width: 20px;
  height: 20px;
  fill: white;
  transition: fill 0.3s, transform 0.2s;
}

.change_theme {
  background: #e9f5f8;
  color: #34495e;
  border: 1px solid #b3e5ea;
  padding: 12px 16px;
  border-radius: 8px;
  width: 100%;
  cursor: pointer;
  font-size: 1em;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  transition: background 0.3s, transform 0.2s, box-shadow 0.2s;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.05);
}

.change_theme:hover {
  background: #cfeff3;
  color: #2C3E50;
  border-color: #8ed1da;
  transform: translateY(-2px);
  box-shadow: 0px 6px 12px rgba(0, 0, 0, 0.1);
}

.change_theme_icon {
  width: 20px;
  height: 20px;
  fill: #3AAEA5;
  transition: fill 0.3s, transform 0.2s;
}

.change_theme:hover .change_theme_icon {
  fill: #2C3E50;
}
~~~

---

## Contract Manifest (Selectors and Property Names)
The AI agent must ensure every new theme keeps these exact selectors and property names.

- `*`
  - margin, padding, font-family

- `body`
  - display, justify-content, align-items, height, padding, overflow

- `.sidebar`
  - width, height, position, inset, display, flex-direction, padding, overflow, border-right, background

- `.menu_links`
  - overflow-y

- `.sidebar a:not(.selected), .sidebar .selected`
  - display, align-items, gap, color, text-decoration, font-size, transition, padding

- `.reset-password-icon, .close-menu`
  - position, top, cursor, color, transition

- `.reset-password-icon`
  - right, font-size

- `.close-menu`
  - right, font-size

- `.reset-password-icon:hover, .close-menu:hover`
  - color

- `.user-avatar`
  - display, align-items, gap, color

- `.user-avatar img`
  - width, height, border-radius, border, object-fit

- `.user-name`
  - font-size, font-weight

- `.user-role`
  - font-size, color

- `.divider`
  - border, height, background, margin, border-radius, box-shadow

- `.menu-search-container`
  - margin-top, margin-bottom

- `.menu-search-input`
  - width, padding, font-size, border, border-radius, box-sizing, outline, color, background-color, transition, box-shadow

- `.menu-search-input:focus`
  - border-color, box-shadow

- `.accordion`
  - width, background, color, font-size, text-align, outline, cursor, padding, display, justify-content, align-items, gap, transition, border

- `.accordion:hover`
  - color, background-color, border-radius

- `.sidebar .selected`
  - background-color, color, font-weight, border-radius, box-shadow

- `.panel`
  - overflow, padding-left, border-radius, margin-bottom, display

- `.panel.show`
  - display

- `.panel a`
  - padding, padding-left, color, text-decoration, transition, gap

- `.panel a:hover`
  - color

- `.menu_links`
  - overflow-y, scrollbar-width, scrollbar-color

- `.link-left`
  - display, align-items, gap, font-size, color, transition

- `.link-left:hover`
  - color

- `.arrow`
  - width, height, transition, fill

- `.expanded .arrow`
  - transform, fill

- `.submenu`
  - display, padding-left, margin-top, background, border-left, border-radius

- `.expanded .submenu`
  - display, box-shadow

- `.icon`
  - width, height, fill, transition

- `.accordion:hover .icon, .selected .icon`
  - fill

- `.notification-area`
  - background, border, border-radius, padding, box-shadow, backdrop-filter, transition

- `.notification-area:hover`
  - box-shadow, transform

- `.notification-header`
  - font-weight, color, margin-bottom

- `.notification-content`
  - font-size, color, display, align-items, gap

- `.notification-content i`
  - font-size, color

- `.notification-actions`
  - display, justify-content, margin-top, font-size, font-weight

- `.notification-actions a`
  - color, text-decoration, cursor, transition

- `.notification-actions a:hover`
  - color, text-shadow, text-decoration

- `.bottom_buttons`
  - display, flex-direction, gap, flex-wrap

- `.close-application`
  - margin-bottom, justify-content, background, color, border, padding, border-radius, width, cursor, font-size, display, align-items, gap, transition

- `.close-application:hover`
  - background

- `.close_application_icon`
  - width, height, fill, transition

- `.change_theme`
  - background, color, border, padding, border-radius, width, cursor, font-size, display, justify-content, align-items, gap, transition, box-shadow

- `.change_theme:hover`
  - background, color, border-color, transform, box-shadow

- `.change_theme_icon`
  - width, height, fill, transition

- `.change_theme:hover .change_theme_icon`
  - fill

---

## Theme Creation Procedure
When creating a new theme:
1. Create a new theme name (unique, Title Case).
2. Copy the canonical Menu CSS skeleton exactly.
3. Change values only, mainly:
   - background gradient colors in `.sidebar`
   - main text colors used across links, headings, and content (`.sidebar a...`, `.user-avatar`, `.user-role`, `.notification-content`)
   - selected state colors and outline accents (`.sidebar .selected`, `.accordion:hover .icon, .selected .icon`)
   - border and shadow strengths to keep it sharp and modern (hairlines, glows, hover shadows)
   - input border and focus glow (`.menu-search-input`, `.menu-search-input:focus`)
   - submenu and notification panel backgrounds and opacities (`.submenu`, `.notification-area`)
   - button colors, hover colors, and icon fills (`.close-application`, `.change_theme`, `.change_theme_icon`)
   - scrollbar thumb and track colors (`.menu_links`)
4. Ensure it looks professional, sharp, compact, modern, cohesive, and remains readable.

---

## FileMaker Script Branch Template
Add a new branch to the Menu CSS generator script:

~~~text
Else If [ $$Theme = "New Theme Name" ]
  Set Variable [ $styles ; Value: "PASTE FULL UPDATED CSS STRING HERE" ]
End If
~~~

---

## AI Request Template
Create a new Menu theme using the canonical Menu CSS in this document.

Theme Name: <New Theme Name>
Mode: <Light or Dark>
Vibe Keywords: <example: premium, compact, modern, cohesive>

Rules:
- Copy the canonical Menu CSS exactly.
- Change values only.
- Do not add, remove, or rename selectors.
- Do not add or remove properties.

Deliver:
1. Full updated CSS string
2. Full FileMaker `Else If` branch for the new theme

---

# List Pages CSS Generator (CauferoAppStarter)

## Purpose
This document defines the canonical CSS skeleton for List Pages (table view + grid view + empty state) in CauferoAppStarter.

The AI agent must create a new theme by changing CSS values only, while keeping the exact same selectors and the exact same property names.

This CSS covers:
- WebViewer list page container (full-viewport fixed content)
- Sticky header area (search, count, add button)
- Scrollable table with sticky header and selectable rows
- Optional record grid cards (with selected-card state)
- Category header rows inside the table
- Theming tokens via `:root` variables
- Row hover and selection effects using `#recordsTable` selectors
- “No Record Found” empty-state message card with CTA and animations

---

## Script Context
- Input: `$$Theme` (or whatever theme selector your List CSS generator uses)
- Output: `$styles` (full CSS stylesheet string)
- Script ends with: `Exit Script [ Text Result: $styles ]`

---

## Hard Rules (CSS Contract)
When creating a new List Page theme:
1. Keep the exact same selectors.
2. Keep the exact same property names under each selector.
3. Change values only.

Never:
- add selectors
- remove selectors
- rename selectors
- add properties
- remove properties

If a requested visual change would require adding selectors or properties, do not implement it. Achieve the best possible result using value changes only.

---

## Canonical List Page CSS Skeleton (Source Of Truth)
This is the canonical skeleton the AI must copy and edit for every new list-page theme.

~~~css
/* General reset and body styling */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

body {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f3f4f6;
  height: 100vh;
  padding: 20px;
  overflow: hidden;
}

/* Main container styling */
.main-content {
  width: auto;
  height: auto;
  position: fixed;
  inset: 0;
  margin: 20px !important;
  display: flex;
  flex-direction: column;
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

/* Sticky header styling */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: sticky;
  top: 0;
  background-color: #fff;
  padding: 20px;
  z-index: 10;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
}

.header .search-bar {
  width: 250px;
}

.search-wrapper {
  position: relative;
  width: 100%;
}

.search-wrapper i {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #999;
  font-size: 16px;
  pointer-events: none;
}

.search-wrapper input[type='search'] {
  padding: 10px 12px 10px 38px;
  border: 1px solid #dcdcdc;
  border-radius: 6px;
  background-color: #f9f9f9;
  color: #555;
  outline: none;
  width: 100%;
  font-size: 14px;
  transition: border-color 0.3s, box-shadow 0.3s;
}

.search-wrapper input[type='search']:focus {
  border-color: #007bff;
  box-shadow: 0px 4px 8px rgba(0, 123, 255, 0.2);
  background-color: #ffffff;
}

.search-wrapper input[type='search']::placeholder {
  color: #aaa;
  opacity: 0.8;
}

/* Record count display */
.record-count {
  display: flex;
  align-items: center;
  font-size: 18px;
  color: #333;
}

.record-count .count-number {
  font-size: 20px;
  font-weight: bold;
  color: #007bff;
  margin-left: 5px;
}

/* Button styling */
.add-button button {
  background-color: #28a745;
  color: #fff;
  padding: 12px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
}

.add-button button:hover {
  background-color: #006b7a;
}

/* Table styling */
.table-container {
  flex-grow: 1;
  overflow-y: auto;
  border-radius: 8px;
  padding: 0 20px;
}

/* Table Wrapper for Scrollable Body */
.table-wrapper {
  max-height: 400px;
  overflow-y: auto;
  border: 1px solid #dcdcdc;
  border-radius: 8px;
  background-color: #ffffff;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

/* Fixed Header */
thead th {
  position: sticky;
  top: 0;
  background: #f5f5f5;
  font-weight: bold;
  padding: 12px;
  border-bottom: 2px solid #dcdcdc;
  z-index: 2;
  color: #333;
}

th,
td {
  padding: 12px;
  text-align: left;
  font-size: 14px;
  color: #555;
}

th {
  background-color: #f5f5f5;
  font-weight: bold;
}

/* Alternating row colors */
tbody tr:nth-child(odd) {
  background-color: #f9fafb;
}

tbody tr:nth-child(even) {
  background-color: #ffffff;
}

tr:hover {
  background-color: #e9eef2;
}

/* Selected row styling */
.selected-row {
  background-color: #d1e7ff !important;
}

/* Status Chip Styling */
.status {
  display: inline-block;
  padding: 5px 10px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
}

.status.gray {
  background-color: #6c757d;
  color: #fff;
}

.status.yellow {
  background-color: #f0ad4e;
  color: #fff;
}

.status.green {
  background-color: #5cb85c;
  color: #fff;
}

.status.red {
  background-color: #dc3545;
  color: #fff;
}

.icon {
  font-size: 16px;
  color: #999;
  cursor: pointer;
}

.record-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(170px, 1fr));
  gap: 10px;
  padding: 20px;
  overflow-y: auto;
  height: auto;
  background-color: rgba(255, 255, 255, 1);
  height: 100vh;
}

.record-grid-card {
  background-color: #ffffff;
  border-radius: 6px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  text-align: center;
  padding: 8px;
  transition: transform 0.2s, box-shadow 0.2s;
  cursor: pointer;
  height: 170px;
}

.record-grid-card img {
  width: 100%;
  height: 100px;
  object-fit: cover;
  border-radius: 4px;
}

.record-grid-info h3 {
  font-size: 13px;
  margin-top: 6px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.record-grid-info p {
  font-size: 12px;
  color: #555;
  margin-top: 4px;
}

h2 {
  font-size: 1.2rem;
  font-weight: 700;
  color: #007bff;
  border-bottom: 1px solid #007bff;
  text-transform: uppercase;
  letter-spacing: 0.03em;
}

/* Category Header Styling - Aligned to Bottom */
.category-header {
  height: 60px;
  padding-bottom: 5px!important;
  font-weight: 700;
  text-transform: uppercase;
  color: #007bff;
  border-bottom: 1px solid #007bff;
  letter-spacing: 0.03em;
  text-shadow: 0px 1px 3px rgba(0, 123, 255, 0.15);
  font-size: 1.1rem;
}

/* Reduce space above category header */
.category-header:first-child {
  margin-top: 0;
  padding-top: 8px;
}

/* Reduce spacing between category and next row */
.category-header + tr {
  margin-top: 0;
  padding-top: 0;
}

.category-header td {
  vertical-align: bottom;
  padding-top: 2px;
  padding-bottom: 5px;
  font-size: 1.1rem;
  color: #007bff;
  font-weight: 600;
}

/* ===== Theming tokens (updated for theme) ===== */
:root {
  --accent: #007bff;
  --accent-light: #66b2ff;
  --accent-strong: #0056b3;
  --row-hover-bg: #e9eef2;
  --row-selected-bg: #d1e7ff;
}

/* Make the whole row feel clickable */
#recordsTable tbody tr.clickable-row {
  cursor: pointer;
}

/* Hover effect */
#recordsTable tbody tr.clickable-row:hover {
  background-color: var(--row-hover-bg) !important;
  box-shadow: inset 0 0 0 2px var(--accent-light);
  transition: none;
}

#recordsTable tbody tr.clickable-row:hover td:first-child {
  box-shadow: inset 8px 0 0 var(--accent-light);
  transition: none;
}

/* Selected effect */
#recordsTable tbody tr.selected-row {
  background-color: var(--row-selected-bg) !important;
  color: #333;
  box-shadow: inset 0 0 0 3px var(--accent-strong);
  transition: none;
}

#recordsTable tbody tr.selected-row td:first-child {
  box-shadow: inset 8px 0 0 var(--accent-strong);
  transition: none;
}

/* Neutralize default hover to avoid conflicts */
#recordsTable tbody tr:hover {
  background-color: inherit;
}

/* Selected card styling - matches grid theme */
.selected-card {
  outline: 2px solid #007bff;
  box-shadow: 0 2px 8px rgba(0, 123, 255, 0.2);
  background-color: #f0f8ff;
  transform: scale(1.02);
  transition: transform 0.2s, box-shadow 0.2s, background-color 0.2s;
}

/* =========================
   No Record Found (Themed)
   ========================= */

.message-container {
  text-align: center;
  padding: 40px 48px;
  animation: fadeIn 0.8s ease-out forwards;
  max-width: 700px;
  width: 90%;
  position: absolute;
  inset: 50% auto auto 50%;
  transform: translate(-50%, -50%);
  color: var(--accent-strong, #0056b3);
  background: #fff;
  border: 1px solid #dcdcdc;
  border-radius: 8px;
  box-shadow: 0 1px 6px rgba(0,0,0,0.08);
}

.message-container h1 {
  font-size: 24px;
  color: #333;
  margin-bottom: 10px;
  animation: slideInDown 0.8s ease-out;
  text-shadow: 0 1px 2px rgba(0,0,0,0.04);
}

.message-container p {
  font-size: 14px;
  color: #555;
  margin-bottom: 18px;
  animation: slideInUp 1s ease-out;
}

.message-container .svg-icon {
  width: 96px;
  height: 96px;
  margin: 0 auto 18px auto;
  display: block;
  fill: currentColor;
  stroke: currentColor;
  color: var(--accent, #007bff);
  animation: fadeInZoom 0.9s ease-out;
  transition: color 0.25s ease, transform 0.2s ease;
}

.message-container .svg-icon:hover {
  color: var(--accent-strong, #0056b3);
  transform: scale(1.04);
}

.cta-btn {
  display: inline-block;
  margin-top: 14px;
  padding: 12px 26px;
  font-size: 14px;
  font-weight: 600;
  color: #ffffff;
  background: linear-gradient(135deg, var(--accent, #007bff), var(--accent-strong, #0056b3));
  border: none;
  border-radius: 6px;
  cursor: pointer;
  text-decoration: none;
  box-shadow: 0 4px 12px rgba(0, 123, 255, 0.18);
  transition: background 0.25s ease, box-shadow 0.25s ease, transform 0.18s ease;
}

.cta-btn:hover {
  background: linear-gradient(135deg, var(--accent-strong, #0056b3), var(--accent, #007bff));
  transform: translateY(-2px);
  box-shadow: 0 6px 18px rgba(0, 123, 255, 0.25);
}

/* Subtle link style inside message (if any) */
.message-container a.inline-link {
  color: var(--accent, #007bff);
  text-decoration: none;
  border-bottom: 1px dashed var(--accent-light, #66b2ff);
}
.message-container a.inline-link:hover {
  color: var(--accent-strong, #0056b3);
  border-bottom-color: var(--accent-strong, #0056b3);
}

/* ===== Animations (unchanged) ===== */
@keyframes fadeIn {
  from { opacity: 0; transform: translate(-50%, calc(-50% + 8px)); }
  to   { opacity: 1; transform: translate(-50%, -50%); }
}
@keyframes slideInDown {
  from { opacity: 0; transform: translateY(-8px); }
  to   { opacity: 1; transform: translateY(0); }
}
@keyframes slideInUp {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}
@keyframes fadeInZoom {
  from { opacity: 0; transform: scale(0.96); }
  to   { opacity: 1; transform: scale(1); }
}

/* Respect reduced motion */
@media (prefers-reduced-motion: reduce) {
  .message-container,
  .message-container h1,
  .message-container p,
  .message-container .svg-icon {
    animation: none !important;
    transition: none !important;
  }
}
~~~

---

## Contract Manifest (Selectors and Property Names)
The AI agent must ensure every new theme keeps these exact selectors and property names.

- `*`
  - box-sizing, margin, padding, font-family

- `body`
  - display, justify-content, align-items, background-color, height, padding, overflow

- `.main-content`
  - width, height, position, inset, margin, display, flex-direction, padding, background, border-radius, box-shadow, overflow

- `.header`
  - display, justify-content, align-items, position, top, background-color, padding, z-index, box-shadow

- `.header .search-bar`
  - width

- `.search-wrapper`
  - position, width

- `.search-wrapper i`
  - position, left, top, transform, color, font-size, pointer-events

- `.search-wrapper input[type='search']`
  - padding, border, border-radius, background-color, color, outline, width, font-size, transition

- `.search-wrapper input[type='search']:focus`
  - border-color, box-shadow, background-color

- `.search-wrapper input[type='search']::placeholder`
  - color, opacity

- `.record-count`
  - display, align-items, font-size, color

- `.record-count .count-number`
  - font-size, font-weight, color, margin-left

- `.add-button button`
  - background-color, color, padding, border, border-radius, cursor, font-size, font-weight

- `.add-button button:hover`
  - background-color

- `.table-container`
  - flex-grow, overflow-y, border-radius, padding

- `.table-wrapper`
  - max-height, overflow-y, border, border-radius, background-color

- `table`
  - width, border-collapse, margin-bottom

- `thead th`
  - position, top, background, font-weight, padding, border-bottom, z-index, color

- `th, td`
  - padding, text-align, font-size, color

- `th`
  - background-color, font-weight

- `tbody tr:nth-child(odd)`
  - background-color

- `tbody tr:nth-child(even)`
  - background-color

- `tr:hover`
  - background-color

- `.selected-row`
  - background-color

- `.status`
  - display, padding, border-radius, font-size, font-weight

- `.status.gray`
  - background-color, color

- `.status.yellow`
  - background-color, color

- `.status.green`
  - background-color, color

- `.status.red`
  - background-color, color

- `.icon`
  - font-size, color, cursor

- `.record-grid`
  - display, grid-template-columns, gap, padding, overflow-y, height, background-color

- `.record-grid-card`
  - background-color, border-radius, box-shadow, overflow, text-align, padding, transition, cursor, height

- `.record-grid-card img`
  - width, height, object-fit, border-radius

- `.record-grid-info h3`
  - font-size, margin-top, white-space, overflow, text-overflow

- `.record-grid-info p`
  - font-size, color, margin-top

- `h2`
  - font-size, font-weight, color, border-bottom, text-transform, letter-spacing

- `.category-header`
  - height, padding-bottom, font-weight, text-transform, color, border-bottom, letter-spacing, text-shadow, font-size

- `.category-header:first-child`
  - margin-top, padding-top

- `.category-header + tr`
  - margin-top, padding-top

- `.category-header td`
  - vertical-align, padding-top, padding-bottom, font-size, color, font-weight

- `:root`
  - --accent, --accent-light, --accent-strong, --row-hover-bg, --row-selected-bg

- `#recordsTable tbody tr.clickable-row`
  - cursor

- `#recordsTable tbody tr.clickable-row:hover`
  - background-color, box-shadow, transition

- `#recordsTable tbody tr.clickable-row:hover td:first-child`
  - box-shadow, transition

- `#recordsTable tbody tr.selected-row`
  - background-color, color, box-shadow, transition

- `#recordsTable tbody tr.selected-row td:first-child`
  - box-shadow, transition

- `#recordsTable tbody tr:hover`
  - background-color

- `.selected-card`
  - outline, box-shadow, background-color, transform, transition

- `.message-container`
  - text-align, padding, animation, max-width, width, position, inset, transform, color, background, border, border-radius, box-shadow

- `.message-container h1`
  - font-size, color, margin-bottom, animation, text-shadow

- `.message-container p`
  - font-size, color, margin-bottom, animation

- `.message-container .svg-icon`
  - width, height, margin, display, fill, stroke, color, animation, transition

- `.message-container .svg-icon:hover`
  - color, transform

- `.cta-btn`
  - display, margin-top, padding, font-size, font-weight, color, background, border, border-radius, cursor, text-decoration, box-shadow, transition

- `.cta-btn:hover`
  - background, transform, box-shadow

- `.message-container a.inline-link`
  - color, text-decoration, border-bottom

- `.message-container a.inline-link:hover`
  - color, border-bottom-color

- `@keyframes fadeIn`
  - from opacity, from transform, to opacity, to transform

- `@keyframes slideInDown`
  - from opacity, from transform, to opacity, to transform

- `@keyframes slideInUp`
  - from opacity, from transform, to opacity, to transform

- `@keyframes fadeInZoom`
  - from opacity, from transform, to opacity, to transform

- `@media (prefers-reduced-motion: reduce)`
  - (keep selectors exactly as in the skeleton)
  - .message-container animation, transition
  - .message-container h1 animation, transition
  - .message-container p animation, transition
  - .message-container .svg-icon animation, transition

---

## Theme Creation Procedure
When creating a new theme:
1. Create a new theme name (unique, Title Case).
2. Copy the canonical List Page CSS skeleton exactly.
3. Change values only, mainly:
   - overall background and base surfaces (`body`, `.main-content`, `.header`, `.table-wrapper`)
   - accent colors used in count, headings, selection, and empty state (`.record-count .count-number`, `h2`, `.category-header`, `:root` variables)
   - focus state glow for the search input (`.search-wrapper input[type='search']:focus`)
   - row hover and selection effects (`tr:hover`, `.selected-row`, `#recordsTable ...`)
   - chip colors (`.status.*`) to match the theme palette
   - shadows and borders to stay sharp and modern (`.main-content`, `.header`, `.table-wrapper`, `.record-grid-card`, `.message-container`, `.cta-btn`)
4. Ensure it looks professional, sharp, compact, modern, cohesive, and remains readable.

---

## FileMaker Script Branch Template
Add a new branch to the List Pages CSS generator script:

~~~text
Else If [ $$Theme = "New Theme Name" ]
  Set Variable [ $styles ; Value: "PASTE FULL UPDATED CSS STRING HERE" ]
End If
~~~

---

## AI Request Template
Create a new List Page theme using the canonical List Page CSS in this document.

Theme Name: <New Theme Name>
Mode: <Light or Dark>
Vibe Keywords: <example: premium, compact, modern, cohesive>

Rules:
- Copy the canonical List Page CSS exactly.
- Change values only.
- Do not add, remove, or rename selectors.
- Do not add or remove properties.

Deliver:
1. Full updated CSS string
2. Full FileMaker `Else If` branch for the new theme


---

# Details Pages CSS Generator (CauferoAppStarter)

## Purpose
This document defines the canonical CSS skeleton for Details Pages in CauferoAppStarter.

A “Details Page” is any WebViewer page that shows one selected record, plus:
- A left summary column (image, status, key fields, buttons)
- A right content column (tabs, forms, subtables, widgets)
- Optional components like wizards, master-detail panels, hierarchical scoring tables, reports, vitals grids, and empty states

The AI agent must generate new Details Page themes by changing CSS values only, while keeping the same selectors and the same property names.

---

## Script Context
- Input: `$$Theme` (or your theme selector variable)
- Output: `$styles` (full CSS stylesheet string)
- Script ends with: `Exit Script [ Text Result: $styles ]`

---

## Hard Rules (CSS Contract)
When creating a new Details Page theme:
1. Keep the exact same selectors.
2. Keep the exact same property names under each selector.
3. Change values only.

Never:
- add selectors
- remove selectors
- rename selectors
- add properties
- remove properties

If a requested visual change would require new selectors or new properties, do not implement it. Achieve the best possible result using value changes only.

---

## Details Page Layout Model
The page is built around a fixed, full-viewport container split into two columns:

1. **Left: Summary Section (25%)**
   - `.summary-section` + `.summary-card`
   - Status badge, image, key fields, QR code
   - Buttons at the bottom (`save`, `cancel`, `third`, and color variants)
   - Optional “summary cards” above buttons

2. **Right: Content Section (75%)**
   - `.content-section` + `.tabs` + `.content`
   - Form grids (1–4 columns)
   - Inputs, selects, textarea, checkbox group
   - Subtables, accordions, alerts, breadcrumbs
   - Optional components (wizard, report layout, master-details, hierarchical scoring table, vitals grid)
   - Optional empty-state message container

All scrolling must be internal (inside cards or content blocks). The viewport must not scroll.

---

## Canonical Details Page CSS Skeleton (Source Of Truth)
The AI must copy this skeleton exactly for every theme and change values only.

~~~css
/* General reset and body styling */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

body {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f3f4f6;
  height: 100vh;
  padding: 20px;
  overflow: hidden;
}

/* ===== Theming tokens (updated for theme) ===== */
:root {
  --accent: #007bff;
  --accent-light: #66b2ff;
  --accent-strong: #0056b3;
  --row-hover-bg: #e9eef2;
  --row-selected-bg: #d1e7ff;
}

/* Main container styling */
.container {
  width: auto;
  height: auto;
  position: fixed;
  inset: 0;
  margin: 20px !important;
  display: flex;
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

/* Left section styling (Summary) */
.summary-section {
  width: 25%;
  padding-right: 20px;
  border-right: 1px solid #e0e0e0;
}

.summary-card {
  background: #edf7ff;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  position: relative;
}

.summary-content {
  overflow-y: auto;
  padding-right: 10px;
  height: calc(100% - 120px);
}

.summary-card h2 {
  font-size: 18px;
  color: #333;
  text-align: center;
  margin-bottom: 20px;
}

.summary-info {
  font-size: 13px;
  color: #333;
  line-height: 1.4;
}

.summary-info p {
  margin: 5px 0;
}

.summary-info strong {
  color: #007BB0;
  font-weight: 600;
}

.summary-image {
  width: 100%;
  height: 200px;
  object-fit: contain;
  background: black;
  margin-bottom: 15px;
  display: block;
  border-radius: 8px;
  box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.2);
}

/* Status Badge Styling */
.status-badge {
  display: inline-block;
  margin: auto;
  margin-top: 0px;
  margin-bottom: 5px;
  padding: 5px 10px;
  font-size: 12px;
  font-weight: bold;
  color: white;
  border-radius: 12px;
  text-align: center;
  width: fit-content;
}

.status-badge.gray { background-color: #6c757d; }
.status-badge.yellow { background-color: #007BFF; }
.status-badge.green { background-color: #28a745; }
.status-badge.red { background-color: #dc3545; }

.qr-code {
  width: 80px;
  height: auto;
  margin: 15px auto;
  display: block;
}

.section-title {
  font-size: 16px;
  margin-top: 15px;
  font-weight: bold;
}

/* Line styling */
hr {
  border: none;
  border-top: 1px solid #ddd;
  margin-top: auto;
  margin-bottom: 10px;
}

/* Button styling */
.button-container {
  margin-top: 10px;
  display: flex;
  justify-content: center;
  gap: 10px;
}

.save-button,
.cancel-button,
.third-button,
.button_red,
.button_green,
.button_blue,
.button_yellow,
.button_purple {
  padding: 10px 15px;
  font-size: 13px;
  font-weight: 500;
  width: 50%;
  height: 40px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  color: white;
  transition: background-color 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
}

.save-button { background-color: #28a745; }
.save-button:hover { background-color: #218838; }

.cancel-button { background-color: #dc3545; }
.cancel-button:hover { background-color: #c82333; }

/* Third Button Styling */
.third-button {
  background-color: #007BFF;
  width: 100%;
  margin-top: 10px;
}
.third-button:hover { background-color: #0056b3; }

/* Add Another Item Button */
.add-another-item-button {
  background-color: #007bff;
  color: white;
  padding: 10px 15px;
  font-size: 14px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-top: 15px;
  width: fit-content;
  display: flex;
  align-items: center;
  gap: 5px;
}
.add-another-item-button:hover { background-color: #0056b3; }

.button1 {
  background-color: #FF9800;
  color: #fff;
  padding: 10px 15px;
  font-size: 14px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-top: 15px;
  width: fit-content;
  display: flex;
  align-items: center;
  gap: 5px;
}
.button1:hover { background-color: #F57C00; }

.button_red { background-color: #FF4C4C; }
.button_red:hover { background-color: #D43A3A; }

.button_green { background-color: #33CC99; }
.button_green:hover { background-color: #29A87C; }

.button_blue { background-color: #6699FF; }
.button_blue:hover { background-color: #4C7AD6; }

.button_yellow { background-color: #FFD700; }
.button_yellow:hover { background-color: #D4B200; }

.button_purple { background-color: #B266FF; }
.button_purple:hover { background-color: #8A4CD6; }

.save-button_,
.cancel-button_,
.third-button_,
.button_red_,
.button_green_,
.button_blue_,
.button_yellow_,
.button_purple_ {
  padding: 10px 15px;
  font-size: 13px;
  font-weight: 500;
  width: 50%;
  height: 40px;
  border-radius: 6px;
  cursor: pointer;
  background-color: transparent;
  border: 2px solid;
  color: inherit;
  transition: color 0.3s, border-color 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
}

/* Outline variants */
.save-button_ { color: #28a745; border-color: #28a745; }
.save-button_:hover { color: #218838; border-color: #218838; }

.cancel-button_ { color: #dc3545; border-color: #dc3545; }
.cancel-button_:hover { color: #c82333; border-color: #c82333; }

.third-button_ {
  color: #007BFF;
  border-color: #007BFF;
  width: 100%;
  margin-top: 10px;
}
.third-button_:hover { color: #0056b3; border-color: #0056b3; }

.button_red_ { color: #FF4C4C; border-color: #FF4C4C; }
.button_red_:hover { color: #D43A3A; border-color: #D43A3A; }

.button_green_ { color: #33CC99; border-color: #33CC99; }
.button_green_:hover { color: #29A87C; border-color: #29A87C; }

.button_blue_ { color: #6699FF; border-color: #6699FF; }
.button_blue_:hover { color: #4C7AD6; border-color: #4C7AD6; }

.button_yellow_ { color: #FFD700; border-color: #FFD700; }
.button_yellow_:hover { color: #D4B200; border-color: #D4B200; }

.button_purple_ { color: #B266FF; border-color: #B266FF; }
.button_purple_:hover { color: #8A4CD6; border-color: #8A4CD6; }

/* Right section styling (Tabs and content) */
.content-section {
  width: 75%;
  padding-left: 20px;
  display: flex;
  flex-direction: column;
  overflow-y: hidden;
  height: 100%;
}

.tabs {
  display: flex;
  border-bottom: 2px solid #e0e0e0;
  margin-bottom: 10px;
}

.tab {
  padding: 10px 20px;
  font-size: 14px;
  color: #555;
  cursor: pointer;
  border-bottom: 3px solid transparent;
  position: relative;
  transition: color 0.2s ease, border-bottom 0.2s ease;
}

.tab.active {
  color: #1e293b;
  font-weight: 600;
  border-bottom-color: #1e293b;
}

/* Badge styling */
.tab.badge-tab::after {
  content: '5';
  position: absolute;
  top: -5px;
  right: 5px;
  background-color: #dc3545;
  color: white;
  font-size: 12px;
  font-weight: bold;
  padding: 3px 6px;
  border-radius: 50%;
  box-shadow: 0px 1px 3px rgba(0, 0, 0, 0.2);
}

.content {
  padding: 10px 0;
  display: none;
  overflow-y: auto;
  max-height: 100%;
}

.content h3 {
  font-size: 16px;
  font-weight: 500;
  color: #333;
  margin-bottom: 5px;
}

.content p {
  color: #666;
  font-size: 14px;
  line-height: 1.5;
  margin-bottom: 15px;
}

.form-grid1 { display: grid; grid-template-columns: repeat(1, 1fr); gap: 15px; }
.form-grid2 { display: grid; grid-template-columns: repeat(2, 1fr); gap: 15px; }
.form-grid3 { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; }
.form-grid4 { display: grid; grid-template-columns: repeat(4, 1fr); gap: 15px; }

.form-group { margin-bottom: 15px; }
.form-group-2 { margin-bottom: 15px; padding: 0 20px; }

/* Instruction Styling */
.instructions1 {
  background: #e3f2fd;
  padding: 10px;
  padding-left: 30px;
  padding-right: 30px;
  border-left: 5px solid #007BFF;
  margin-bottom: 15px;
  border-radius: 5px;
  position: relative;
  font-size: 14px;
}
.instructions2 {
  background: #f3e5f5;
  padding: 10px;
  padding-left: 30px;
  padding-right: 30px;
  border-left: 5px solid #8E24AA;
  margin-bottom: 15px;
  border-radius: 5px;
  position: relative;
  font-size: 14px;
}
.instructions3 {
  background: #e8f5e9;
  padding: 10px;
  padding-left: 30px;
  padding-right: 30px;
  border-left: 5px solid #28a745;
  margin-bottom: 15px;
  border-radius: 5px;
  position: relative;
  font-size: 14px;
}

.instructions1 h2,
.instructions2 h2,
.instructions3 h2 {
  color: #333;
  font-size: 15px;
  margin-bottom: 5px;
}

label {
  font-size: 14px;
  font-weight: 500;
  color: #333;
  display: block;
  margin-bottom: 5px;
}

input[type='text']:not(.combined-inputs-input),
input[type='file']:not(.combined-inputs-input),
input[type='number']:not(.combined-inputs-input),
input[type='email']:not(.combined-inputs-input),
input[type='date']:not(.combined-inputs-input),
textarea {
  width: 100%;
  height: 38px;
  padding: 10px;
  font-size: 14px;
  color: #333;
  border: 1px solid #ddd;
  border-radius: 5px;
  outline: none;
  background-color: #f9f9f9;
}

textarea {
  height: 150px;
  resize: none;
}

/* Dropdown Styling */
select:not(.combined-inputs-dropdown) {
  width: 100%;
  height: 38px;
  padding: 10px;
  font-size: 14px;
  color: #333;
  border: 1px solid #ddd;
  border-radius: 5px;
  outline: none;
  appearance: none;
  background-image: url('https://img.playbook.com/EiJ_h1h3ku1Tk5o2UUdY2GR36EdGRYp_FFrAGJtJ9BA/Z3M6Ly9wbGF5Ym9v/ay1hc3NldHMtcHVi/bGljLzU5ZmU3Zjk4/LTliMWEtNDZjYy1h/MTA4LWQ5ODI4YmJl/ZDZlZQ');
  background-repeat: no-repeat;
  background-position: right 10px center;
  background-size: 10px;
}

/* Container for the Checkbox Group */
.checkbox-group {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(30px, 1fr));
  padding: 15px;
  border: 1px solid #E0E0E0;
  border-radius: 8px;
  background-color: #F9FAFB;
  box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.1);
  margin-top: 10px;
  font-family: Arial, sans-serif;
}

.sub-table-container {
  border: 1px solid #ddd;
  border-radius: 0px;
  overflow-y: auto;
}

/* Another Item Table */
.sub-table { width: 100%; border-collapse: collapse; }
.sub-table th,
.sub-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
  font-size: 12px;
}
.sub-table th { background-color: #f2f2f2; font-weight: bold; }
.sub-table tr:nth-child(even) { background-color: #f9f9f9; }

/* Specific column widths */
.sub-table-column-style1 { width: 30%; }
.sub-table-column-style2 { width: 15%; }
.sub-table-column-style3 { width: 10%; }
.sub-table-column-style4 { width: 5%; }

.sub_table_footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
}

/* Sub Table Footer Styling */
.sub_table_footer .total {
  font-size: 18px;
  font-weight: bold;
  color: #dc3545;
}

.sub-table td img {
  width: 50px;
  height: 50px;
  object-fit: cover;
}

.icon-group {
  display: flex;
  gap: 15px;
  justify-content: center;
  align-items: center;
}

.view-icon {
  color: #007BFF;
  font-size: 16px;
  cursor: pointer;
  transition: color 0.3s;
}
.view-icon:hover { color: #0056b3; }

.edit-icon, .delete-icon {
  font-size: 16px;
  cursor: pointer;
  transition: color 0.3s;
}

.edit-icon { color: #8E24AA; }
.edit-icon:hover { color: #6A1B9A; }

.delete-icon { color: #dc3545; }
.delete-icon:hover { color: #c82333; }

/* Accordion */
.accordion-container { margin-top: 10px; }
.accordion { border: 1px solid #ddd; border-radius: 5px; margin-bottom: 10px; }
.accordion-header {
  padding: 10px;
  background: #f2f2f2;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 500;
}
.accordion-header:hover { background: #e0e0e0; }
.accordion-content { padding: 10px; display: none; background: #fafafa; }

/* Improved Inline Alert */
.alert {
  padding: 15px 20px;
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
  font-weight: 500;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 15px;
}
.alert-title { font-weight: bold; margin-right: 10px; }
.alert i { cursor: pointer; color: #721c24; }
.alert i:hover { color: #491116; }

/* Breadcrumbs Styles */
.breadcrumbs {
  display: flex;
  align-items: center;
  padding: 10px 20px;
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.1);
  position: relative;
  margin-bottom: 20px;
}
.breadcrumbs::before {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  height: 8px;
  background: linear-gradient(135deg, #4a90e2, #50e3c2);
  border-radius: 10px 10px 0 0;
  z-index: -1;
}
.breadcrumbs a {
  text-decoration: none;
  color: #4a90e2;
  padding: 10px 15px;
  border-radius: 5px;
  position: relative;
  transition: background-color 0.3s, color 0.3s;
  display: flex;
  align-items: center;
}
.breadcrumbs a:hover { background-color: rgba(74, 144, 226, 0.1); color: #007BFF; }
.breadcrumbs a::after {
  content: '>';
  margin-left: 10px;
  margin-right: 10px;
  color: #aaa;
  font-weight: bold;
}
.breadcrumbs a:last-child::after { content: ''; }
.current {
  color: #555;
  font-weight: bold;
  pointer-events: none;
  position: relative;
  z-index: 1;
}
.icon { margin-right: 5px; font-size: 18px; transition: transform 0.3s; }
.breadcrumbs a:hover .icon { transform: scale(1.2); }

/* Example icon styles - using Font Awesome */
.fa-home:before { content: '\f015'; }
.fa-folder:before { content: '\f07b'; }
.fa-file:before { content: '\f15b'; }
.fa-chevron-right:before { content: '\f054'; }

/* Custom Sign-Up Button for Tab 4 */
.big_button {
  display: flex;
  align-items: center;
  background-color: #4A90E2;
  color: white;
  font-size: 16px;
  font-weight: bold;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  margin-top: 15px;
}
.big_button span { margin-right: 10px; }
.divider { height: 24px; width: 1px; background-color: white; margin: 0 10px; }
.checkmark { font-size: 16px; }

/* Basic styling for the iframe container */
.map-container {
  display: flex;
  justify-content: center;
  align-items: top;
  align-items: left;
}
iframe {
  width: 100%;
  height: 100%;
  border: none;
  border-radius: 8px;
  box-shadow: 0px 0px 10px rgba(0,0,0,0.1);
}

/* Gallery Styles */
.gallery-container { width: 45%; margin-left: 30px; }
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 10px;
}
.gallery-item {
  position: relative;
  overflow: hidden;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.gallery-item img {
  width: 100%;
  height: 100px;
  object-fit: cover;
  transition: transform 0.3s ease;
}
.gallery-item:hover img { transform: scale(1.05); }
.gallery-item .caption {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background: rgba(0, 0, 0, 0.6);
  color: white;
  text-align: center;
  padding: 5px 0;
  font-size: 12px;
  display: none;
}
.gallery-item:hover .caption { display: block; }

/* Message container */
.message-container {
  text-align: center;
  padding: 50px 60px;
  animation: fadeIn 0.8s ease-out forwards;
  width: 700px;
  max-width: 100%;
  color: var(--accent-strong, #0056b3);
  background: #ffffff;
  border: none !important;
  outline: none !important;
  box-shadow: none !important;
  border-radius: 16px;
  margin: auto;
}

/* Headline */
.message-container h1 {
  font-size: 20px;
  color: #333;
  margin-bottom: 10px;
  animation: slideInDown 0.8s ease-out;
}

/* Paragraph */
.message-container p {
  font-size: 14px;
  color: #555;
  margin-bottom: 20px;
  animation: slideInUp 1s ease-out;
}

/* SVG icon */
.message-container .svg-icon {
  width: 100px;
  height: 100px;
  margin-bottom: 40px;
  display: block;
  margin-left: auto;
  margin-right: auto;
  fill: currentColor;
  stroke: currentColor;
  color: var(--accent, #007bff) !Important;
  animation: fadeInZoom 1s ease-out;
  transition: color 0.3s ease, transform 0.2s ease;
}
.message-container .svg-icon:hover { color: var(--accent-strong, #0056b3); transform: scale(1.04); }

/* Button */
.cta-btn {
  display: inline-block;
  margin-top: 20px;
  padding: 14px 30px;
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  background: linear-gradient(135deg, var(--accent, #007bff), var(--accent-strong, #0056b3));
  border: none;
  border-radius: 8px;
  cursor: pointer;
  text-decoration: none;
  transition: background 0.3s ease, transform 0.2s ease;
}
.cta-btn:hover {
  background: linear-gradient(135deg, var(--accent-strong, #0056b3), var(--accent, #007bff));
  transform: translateY(-2px);
}

@keyframes fadeIn {
  0% { opacity: 0; transform: translateY(20px); }
  100% { opacity: 1; transform: translateY(0); }
}
@keyframes fadeInZoom {
  0% { opacity: 0; transform: scale(0.8); }
  100% { opacity: 1; transform: scale(1); }
}
@keyframes slideInDown {
  0% { opacity: 0; transform: translateY(-20px); }
  100% { opacity: 1; transform: translateY(0); }
}
@keyframes slideInUp {
  0% { opacity: 0; transform: translateY(20px); }
  100% { opacity: 1; transform: translateY(0); }
}

/* Row and Card Styling */
.cards_row { display: flex; gap: 20px; margin-top: 10px; }
.note-card {
  flex: 1;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  background-color: #e0f7fa;
  display: flex;
  flex-direction: column;
}
.note-header {
  font-weight: bold;
  margin-bottom: 10px;
  font-size: 16px;
  display: flex;
  justify-content: space-between;
}
.note-content { flex-grow: 1; overflow-y: auto; max-height: 300px; }
.note-content ul { list-style-type: none; padding: 0; margin: 0; }
.note-content li {
  padding: 10px;
  margin: 10px 0;
  background-color: #ffffff;
  border-radius: 6px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}
.note-content li:hover { transform: scale(1.02); }
.note-content button {
  background: transparent;
  border: none;
  font-size: 18px;
  color: #007bff;
  cursor: pointer;
}
.note-content button:hover { color: #0056b3; }

/* Rating */
.rating {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 5px;
  direction: rtl;
}
.rating input { display: none; }
.rating label {
  font-size: 30px;
  color: #ddd;
  cursor: pointer;
  transition: color 0.3s;
}
.rating label:hover,
.rating label:hover ~ label { color: #ffc107; }
.rating input:checked ~ label { color: #ffc107; }
.rating input:checked + label:hover,
.rating input:checked + label:hover ~ label { color: #ffab00; }
.rating label:hover ~ label { color: #ddd; }

/* Wizard */
.wizard-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}
.stepper {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 30px;
  background: linear-gradient(135deg, #007bff, #4c9fff);
  color: #fff;
  position: relative;
}
.step { text-align: center; flex: 1; position: relative; margin-bottom: -25px; }
.step .icon {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background-color: #e0e0e0;
  color: #007bff;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 18px;
  font-weight: bold;
  margin: 0 auto 8px;
  transition: background-color 0.3s ease, color 0.3s ease, transform 0.3s ease, box-shadow 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}
.step.active .icon {
  background-color: #fff;
  color: #007bff;
  border: 2px solid #007bff;
  transform: scale(1.2);
  box-shadow: 0 0 10px 3px rgba(0, 123, 255, 0.6);
}
.step .line {
  position: absolute;
  top: 25px;
  left: 50%;
  width: 100%;
  height: 4px;
  background-color: #e0e0e0;
  z-index: -1;
  transition: background-color 0.3s ease;
}
.step.active + .line { background-color: #fff; }
.step p { font-size: 14px; font-weight: bold; margin-top: 8px; color: #fff; }

.step-wizard_content { display: none; }
.step-wizard_content.active { display: block; }

.wizard_content {
  padding: 20px;
  animation: fadeIn 0.3s ease;
  flex: 1;
  overflow-y: auto;
}
.wizard_content > .step-wizard_content > div.table-top-legend { display: flex !important; }
.wizard_content div.active { display: block; }

.wizard_buttons {
  display: flex;
  justify-content: space-between;
  padding: 20px;
  background-color: #f9f9f9;
}
.wizard_button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  background-color: #007bff;
  color: #fff;
  cursor: pointer;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
}
.wizard_button:hover {
  background-color: #0056b3;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}
.wizard_button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
  box-shadow: none;
}

/* Toggles */
.toggle-container { display: flex; align-items: center; gap: 10px; font-size: 16px; color: #333; }
.toggle-switch { display: inline-block; width: 50px; height: 25px; position: relative; }
.toggle-switch input { opacity: 0; width: 0; height: 0; }
.slider {
  position: absolute;
  cursor: pointer;
  top: 0; left: 0; right: 0; bottom: 0;
  background-color: #ccc;
  border-radius: 25px;
  transition: background-color 0.3s;
}
.slider::before {
  content: '';
  position: absolute;
  height: 21px; width: 21px;
  left: 2px; bottom: 2px;
  background-color: white;
  border-radius: 50%;
  transition: transform 0.3s;
}
input:checked + .slider { background-color: #4caf50; }
input:checked + .slider::before { transform: translateX(25px); }
.status { font-weight: bold; }

/* Segmented Control */
.segmented-control {
  display: flex;
  background-color: #ddd;
  border-radius: 5px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 100%;
  height: 38px;
}
.segmented-control button {
  flex: 1;
  padding: 10px 20px;
  background-color: transparent;
  border: none;
  outline: none;
  cursor: pointer;
  font-size: 16px;
  color: #333;
  transition: background-color 0.3s, color 0.3s;
}
.segmented-control button.active { background-color: #4caf50; color: white; }
.segmented-control button:not(:last-child) { border-right: 1px solid #ccc; }
.segmented-control button:hover { background-color: #bfbfbf; }

/* Report */
.report_header {
  border-bottom: 2px solid #d6d6d6;
  padding-bottom: 15px;
  margin-bottom: 25px;
  text-align: center;
  font-family: Arial, sans-serif;
}
.report_header .company-info { font-size: 14px; color: #555; margin-bottom: 10px; }
.report_header .title { font-size: 20px; font-weight: bold; color: #1e293b; text-transform: uppercase; }
.report_header .date { font-size: 14px; color: #666; }

.report_table {
  width: 100%;
  border-collapse: collapse;
  margin: 15px 0;
  font-size: 13px;
  color: #333;
}
.report_table th,
.report_table td {
  border: 1px solid #e0e0e0;
  padding: 8px 12px;
  text-align: left;
}
.report_table th {
  background-color: #edf7ff;
  font-weight: bold;
  color: #2c3e50;
  text-transform: uppercase;
  font-size: 13px;
}
.report_cell_highlight { font-weight: bold; color: #2c3e50; }
.report_table_footer { text-align: right; margin-top: 25px; font-size: 16px; font-weight: bold; color: #444; }
.report_table_footer span { color: #28a745; }

.report_footer {
  text-align: center;
  font-size: 13px;
  color: #555;
  padding: 20px 0;
  background-color: #f7f9f9;
  border-top: 1px solid #e0e0e0;
}
.report_footer p { margin: 0px 0; }
.report_footer a { color: #007BFF; text-decoration: none; font-weight: bold; }
.report_footer a:hover { text-decoration: underline; }

/* Grid Numbers */
.grid-numbers-container {
  display: grid;
  margin-top: 20px;
  gap: 10px;
  justify-content: center;
  align-items: center;
}
.grid-number {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #007BFF;
  color: #fff;
  font-size: 1.5rem;
  height: 60px;
  width: 60px;
  border-radius: 6px;
  border: 1px solid #0056B3;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease, transform 0.2s ease;
}
.grid-number:hover { background-color: #0056B3; transform: scale(1.05); }

/* Image + Text Grid */
.imagetext-grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(25%, 1fr));
  gap: 20px;
  padding: 20px;
  width: 100%;
  max-width: 100%;
}
.imagetext-grid-item {
  display: flex;
  align-items: center;
  padding: 8px;
  border-radius: 6px;
  background: linear-gradient(135deg, #ffffff, #e9ebef);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  cursor: pointer;
}
.imagetext-grid-item:hover { box-shadow: 0 6px 10px rgba(0, 0, 0, 0.1); }
.imagetext-grid-photo {
  flex-shrink: 0;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  overflow: hidden;
  margin-right: 10px;
  border: 2px solid #007bff;
}
.imagetext-grid-photo img { width: 100%; height: 100%; object-fit: cover; }
.imagetext-grid-info { display: flex; flex-direction: column; gap: 2px; }
.imagetext-grid-info1 { font-size: 0.85rem; font-weight: bold; color: #1e293b; }
.imagetext-grid-info2 { font-size: 0.75rem; color: #4b5563; }
.imagetext-grid-info3 { font-size: 0.7rem; color: #374151; }
.email {
  font-size: 0.75rem;
  color: #2563eb;
  text-decoration: none;
  transition: color 0.2s ease;
  word-break: break-word;
}
.email:hover { color: #1d4ed8; text-decoration: underline; }

/* Combined Inputs */
.combined-inputs-container {
  width: 100%;
  height: 38px;
  padding: 10px;
  font-size: 14px;
  color: #333;
  border: 1px solid #ddd;
  border-radius: 5px;
  outline: none;
  background-color: #f9f9f9;
  display: flex;
  align-items: center;
  box-sizing: border-box;
}
.combined-inputs-input {
  border: none;
  outline: none;
  text-align: center;
  font-size: 14px;
  background-color: transparent;
  color: #333;
}
.combined-inputs-dropdown {
  padding: 10px;
  border: none;
  outline: none;
  font-size: 12px;
  text-align: center;
  color: #333;
  background-color: transparent;
  appearance: none;
}

/* Flatpickr Top Banner Styling */
.flatpickr-months { display: flex; align-items: center; justify-content: center; padding: 8px 0; gap: 12px; }
.flatpickr-current-month { display: flex; align-items: center; justify-content: center; gap: 12px; line-height: 1.2; }
.flatpickr-monthDropdown-months {
  width: 120px;
  height: 36px;
  padding: 4px 8px;
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 14px;
  line-height: 1.5;
}
.flatpickr-yearDropdown-years {
  position: relative;
  width: 90px;
  height: 36px;
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 14px;
  line-height: 1.5;
  display: flex;
  align-items: center;
  justify-content: center;
}
.flatpickr-current-month input.cur-year {
  width: 100%;
  height: 100%;
  text-align: center;
  font-size: 14px;
  border: none;
  box-sizing: border-box;
  -moz-appearance: textfield;
}
.flatpickr-current-month input.cur-year::-webkit-inner-spin-button,
.flatpickr-current-month input.cur-year::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
.flatpickr-yearDropdown-years::after {
  content: '▲';
  position: absolute;
  right: 4px;
  top: 4px;
  font-size: 10px;
  cursor: pointer;
  pointer-events: all;
}
.flatpickr-yearDropdown-years::before {
  content: '▼';
  position: absolute;
  right: 4px;
  bottom: 4px;
  font-size: 10px;
  cursor: pointer;
  pointer-events: all;
}
.flatpickr-prev-month,
.flatpickr-next-month {
  font-size: 18px;
  cursor: pointer;
  padding: 4px;
  line-height: 1;
}
.flatpickr-prev-month { margin-right: 8px; }
.flatpickr-next-month { margin-left: 8px; }
.flatpickr-monthDropdown-months,
.flatpickr-yearDropdown-years,
.flatpickr-current-month input.cur-year {
  display: flex;
  align-items: center;
  justify-content: center;
  vertical-align: middle;
}
.flatpickr-calendar {
  margin: 0;
  padding: 0;
  border-radius: 8px;
  overflow: hidden;
}

/* Master Details */
.master-details-container { display: flex; border: 1px solid #ddd; border-radius: 8px; overflow: hidden; }
.master { background: #f7f9fc; border-right: 1px solid #ddd; overflow-y: auto; }
.master-item {
  display: flex;
  align-items: center;
  padding: 5px 5px;
  border-bottom: 1px solid #ececec;
  cursor: pointer;
  transition: background-color 0.3s ease, border-left 0.3s ease;
  border-left: 4px solid transparent;
}
.master-item:hover { background-color: #f1f7ff; }
.master-item.selected {
  background-color: #e8f0fe;
  border-left: 4px solid #007BFF;
  font-weight: 600;
}
.master-item img {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  margin-right: 10px;
  object-fit: cover;
  border: 1px solid #ccc;
}
.master-item-text { display: flex; flex-direction: column; }
.master-item-text1 { font-weight: 600; font-size: 0.85em; color: #2c3e50; }
.master-item-text2 { font-size: 0.75em; color: #666; }

.details { padding: 20px; background-color: #fff; overflow-y: auto; }
.details h2 { margin-top: 0; font-size: 1.2em; color: #2c3e50; }
.details p { font-size: 0.8em; line-height: 1.5; color: #444; }
.details img { max-width: 100%; border-radius: 3px; border: 1px solid #ddd; }

/* Responsive Design */
@media (max-width: 768px) {
  .container { flex-direction: column; }
  .master, .details { width: 100%; height: 50%; }
}

/* Master Header Styling */
.master-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px;
  border-bottom: 1px solid #ececec;
  background-color: #f7f9fc;
}
.master-header-search-field {
  padding: 8px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

/* Score Labels */
.table-top-legend {
  display: flex;
  justify-content: space-between;
  margin: 10px auto;
  width: 100%;
  font-size: 0.9em;
  text-align: center;
  gap: 12px;
  flex-wrap: wrap;
}
.table-top-legend div {
  padding: 10px 15px;
  border-radius: 6px;
  flex: 1;
  margin: 5px;
  background-color: #f9fafc;
  color: #333;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}
.table-top-legend .score-1 { background: #f8d7da; color: #721c24; }
.table-top-legend .score-2 { background: #fff3cd; color: #856404; }
.table-top-legend .score-3 { background: #d4edda; color: #155724; }
.table-top-legend .score-4 { background: #d1ecf1; color: #0c5460; }
.table-top-legend .score-5 { background: #f5e6fe; color: #6f42c1; }

/* Checkmark Radio Buttons */
.radio-cell { text-align: center; position: relative; }
.radio-cell input[type='radio'] { display: none; }
.radio-cell label {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 35px;
  height: 35px;
  border: 2px solid #007BFF;
  border-radius: 50%;
  cursor: pointer;
  font-size: 14px;
  font-weight: bold;
  color: #fff;
  text-align: center;
  box-sizing: border-box;
  transition: background-color 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
}
.radio-cell input[type='radio']:checked + label {
  background-color: #007BFF;
  border-color: #0056b3;
  box-shadow: 0 0 8px rgba(0, 123, 255, 0.3);
}
.radio-cell label::after {
  content: '';
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  color: white;
  opacity: 0;
  transform: scale(0.5);
  transition: opacity 0.3s ease-in-out, transform 0.3s ease-in-out;
  font-family: 'Font Awesome 6 Free';
  font-weight: 900;
  position: absolute;
  width: 100%;
  height: 100%;
}
.radio-cell input[type='radio']:checked + label::after {
  content: '\f00c';
  color: white;
  opacity: 1;
  transform: scale(1);
}
.radio-cell label:hover {
  background-color: #0056b3;
  border-color: #0056b3;
  box-shadow: 0 0 10px rgba(0, 86, 179, 0.4);
}

/* Hierarchical Table Container */
.hierarchical_table_container {
  width: 100%;
  overflow-y: auto;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  background: #fff;
  position: relative;
}
.hierarchical_table { width: 100%; border-collapse: collapse; }
.hierarchical_table thead {
  position: sticky;
  top: 0;
  background-color: #007BFF;
  z-index: 10;
}
.hierarchical_table th {
  background-color: #007BFF;
  color: #fff;
  text-align: center;
  padding: 12px;
  font-size: 14px;
  font-weight: bold;
}
.hierarchical_table td {
  padding: 12px;
  border-bottom: 1px solid #f1f1f1;
  text-align: center;
  vertical-align: middle;
  font-size: 13px;
}
.hierarchical_table_record_category {
  font-weight: bold;
  text-align: left;
  padding: 15px;
  color: #6A1B9A;
  background-color: #f9fafc;
  font-size: 14px;
  border-left: 5px solid #8E24AA;
}
.hierarchical_table_row { background-color: #ffffff; }
.hierarchical_table_record { text-align: left; padding: 12px; font-size: 13px; }
.hierarchical_table_record_title {
  font-weight: bold;
  font-size: 13px;
  color: #333;
  display: block;
  margin-bottom: 4px;
}
.hierarchical_table_record_description { font-size: 12px; color: #7f8c8d; }
.left-align { text-align: left !important; }
.hierarchical_table_action_icons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 10px;
  margin-right: 5%;
}
.hierarchical_table_action_icons button {
  border: none;
  cursor: pointer;
  font-size: 1.2em;
  color: #007BFF;
  background: none;
  transition: color 0.3s ease;
}
.hierarchical_table_action_icons button:hover { color: #0056b3; }
.hierarchical_table tr:hover { background-color: #f0f8ff; transition: background-color 0.3s ease; }

/* Summary Cards Above Buttons */
.summary-cards { display: flex; flex-direction: column; gap: 15px; margin: 5px 0; }
.summary-card-item {
  background: #f9f9f9;
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  text-align: center;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.summary-card-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}
.summary-card-item h3 {
  font-size: 16px;
  font-weight: 600;
  color: #007BFF;
  margin-bottom: 8px;
}
.summary-card-item p { font-size: 14px; color: #555; margin: 0; }
.summary-card-item strong { font-size: 18px; color: #333; }
.summary-section hr {
  margin: 20px 0;
  border: none;
  border-top: 1px solid #ddd;
}

/* Status Chip Styling */
.status { display: inline-block; padding: 5px 10px; border-radius: 12px; font-size: 12px; font-weight: 500; }
.status.gray { background-color: #6c757d; color: #fff; }
.status.yellow { background-color: #f0ad4e; color: #fff; }
.status.green { background-color: #5cb85c; color: #fff; }
.status.red { background-color: #dc3545; color: #fff; }

/* List Container */
.list-container {
  background: #F9FAFB;
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  height: 100%;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
.list-item {
  display: flex;
  align-items: flex-start;
  padding: 12px 16px;
  border-bottom: 1px solid #E0E0E0;
  transition: background-color 0.3s, box-shadow 0.3s;
  cursor: pointer;
}
.list-item:last-child { border-bottom: none; }
.list-item:hover {
  background: #EDF7FF;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}
.list-item.selected {
  background: #E8F5E9;
  box-shadow: inset 0 0 5px rgba(40, 167, 69, 0.2);
}

/* Checkbox Styling */
.list-checkbox { margin-top: 15px; margin-right: 12px; }
.list-checkbox input { visibility: hidden; position: absolute; }
.list-checkbox .checkbox-label {
  width: 20px;
  height: 20px;
  border: 2px solid #D1D5DB;
  border-radius: 4px;
  display: inline-block;
  position: relative;
  transition: all 0.3s ease;
}
.list-checkbox input:checked + .checkbox-label {
  background: #4A90E2;
  border-color: #4A90E2;
}
.list-checkbox .checkbox-label::after {
  content: '';
  position: absolute;
  top: 0px;
  left: 4px;
  width: 5px;
  height: 10px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg) scale(0);
  transition: transform 0.2s ease;
}
.list-checkbox input:checked + .checkbox-label::after {
  transform: rotate(45deg) scale(1);
}

/* Record Details Styling */
.item-details { flex-grow: 1; }
.item-title { font-size: 16px; font-weight: bold; margin-bottom: 4px; color: #333; }
.item-subtitle { font-size: 14px; color: #555; }
.item-description { font-size: 12px; color: #888; margin-top: 4px; }

/* Footer */
.list-footer {
  margin-top: auto;
  text-align: center;
  padding: 12px;
  font-size: 14px;
  background: #FFFFFF;
  color: #666;
  border-top: 1px solid #E0E0E0;
  border-radius: 0 0 8px 8px;
}
.list-footer-highlight { color: #28A745; font-weight: bold; }

/* Grid Layout */
.vitals_grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; justify-content: center; }
@media (max-width: 1024px) { .vitals_grid { grid-template-columns: repeat(2, 1fr); } }
@media (max-width: 768px) { .vitals_grid { grid-template-columns: repeat(1, 1fr); } }

/* Vital Card Styling */
.vital_card {
  background: #fff;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.vital_card:hover { box-shadow: 0px 6px 12px rgba(0, 0, 0, 0.12); }

.vital-input {
  background: transparent;
  border: none;
  outline: none;
  font-size: 1.8rem;
  font-weight: bold;
  width: 75px;
  text-align: center;
  color: #007BFF;
  transition: color 0.3s ease-in-out;
}
.vital-label { font-size: 0.9rem; font-weight: 600; color: #333; }
.vital-unit { font-size: 0.8rem; color: #666; }

.vital_card input[type='number'],
.vital_card input[type='text'] {
  border: none !important;
  background: transparent !important;
  box-shadow: none !important;
  padding: 0 !important;
  font-size: 1.8rem !important;
  font-weight: bold !important;
  text-align: center !important;
  color: #007BFF !important;
  width: 100px !important;
}

.vital_card .bp-slash {
  font-size: 1.8rem;
  font-weight: bold;
  color: #007BFF;
  padding: 0 4px;
}

.vital_icon { font-size: 3rem; padding: 8px; border-radius: 8px; }
.height-icon { color: #007BFF; background-color: #E3F2FD; }
.weight-icon { color: #28A745; background-color: #E8F5E9; }
.bmi-icon { color: #8E44AD; background-color: #F3E5F5; }
.temp-icon { color: #E74C3C; background-color: #FDECEA; }
.bp-icon { color: #D35400; background-color: #FBE7DC; }
.pulse-icon { color: #F39C12; background-color: #FDF3D9; }
.resp-icon { color: #16A085; background-color: #E0F5F5; }
.oxygen-icon { color: #3498DB; background-color: #E4F0FA; }
.glucose-icon { color: #E67E22; background-color: #FDF2E0; }
~~~

---

## Contract Manifest (Selectors and Property Names)
When generating a new theme, the AI must ensure every selector below exists and keeps the same property names.

### 1) Global Base
- `*`: box-sizing, margin, padding, font-family
- `body`: display, justify-content, align-items, background-color, height, padding, overflow
- `:root`: --accent, --accent-light, --accent-strong, --row-hover-bg, --row-selected-bg

### 2) Container + Two-Column Layout
- `.container`: width, height, position, inset, margin, display, padding, background, border-radius, box-shadow, overflow
- `.summary-section`: width, padding-right, border-right
- `.content-section`: width, padding-left, display, flex-direction, overflow-y, height

### 3) Summary Card
- `.summary-card`: background, padding, border-radius, box-shadow, height, display, flex-direction, overflow, position
- `.summary-content`: overflow-y, padding-right, height
- `.summary-card h2`: font-size, color, text-align, margin-bottom
- `.summary-info`: font-size, color, line-height
- `.summary-info p`: margin
- `.summary-info strong`: color, font-weight
- `.summary-image`: width, height, object-fit, background, margin-bottom, display, border-radius, box-shadow

### 4) Status Badge + QR + Divider
- `.status-badge`: display, margin, margin-top, margin-bottom, padding, font-size, font-weight, color, border-radius, text-align, width
- `.status-badge.gray/.yellow/.green/.red`: background-color
- `.qr-code`: width, height, margin, display
- `.section-title`: font-size, margin-top, font-weight
- `hr`: border, border-top, margin-top, margin-bottom

### 5) Buttons
- `.button-container`: margin-top, display, justify-content, gap
- `.save-button, .cancel-button, .third-button, .button_red, .button_green, .button_blue, .button_yellow, .button_purple`
  - padding, font-size, font-weight, width, height, border, border-radius, cursor, color, transition, display, align-items, justify-content, gap
- Each filled button + hover selector keeps background-color changes
- `.add-another-item-button`: background-color, color, padding, font-size, border, border-radius, cursor, transition, margin-top, width, display, align-items, gap
- `.button1`: background-color, color, padding, font-size, border, border-radius, cursor, transition, margin-top, width, display, align-items, gap
- Outline button group: `.save-button_ ... .button_purple_` with their color/border-color + hover

### 6) Tabs + Content Scroll
- `.tabs`: display, border-bottom, margin-bottom
- `.tab`: padding, font-size, color, cursor, border-bottom, position, transition
- `.tab.active`: color, font-weight, border-bottom-color
- `.tab.badge-tab::after`: content, position, top, right, background-color, color, font-size, font-weight, padding, border-radius, box-shadow
- `.content`: padding, display, overflow-y, max-height
- `.content h3`: font-size, font-weight, color, margin-bottom
- `.content p`: color, font-size, line-height, margin-bottom

### 7) Form Grid + Inputs
- `.form-grid1/2/3/4`: display, grid-template-columns, gap
- `.form-group`: margin-bottom
- `.form-group-2`: margin-bottom, padding
- `.instructions1/2/3`: background, padding, padding-left, padding-right, border-left, margin-bottom, border-radius, position, font-size
- `.instructions1 h2, .instructions2 h2, .instructions3 h2`: color, font-size, margin-bottom
- `label`: font-size, font-weight, color, display, margin-bottom
- `input[type=...]:not(.combined-inputs-input), textarea`: width, height, padding, font-size, color, border, border-radius, outline, background-color
- `textarea`: height, resize
- `select:not(.combined-inputs-dropdown)`: width, height, padding, font-size, color, border, border-radius, outline, appearance, background-image, background-repeat, background-position, background-size
- `.checkbox-group`: display, grid-template-columns, padding, border, border-radius, background-color, box-shadow, margin-top, font-family

### 8) Subtables + Action Icons
- `.sub-table-container`: border, border-radius, overflow-y
- `.sub-table`: width, border-collapse
- `.sub-table th, .sub-table td`: border, padding, text-align, font-size
- `.sub-table th`: background-color, font-weight
- `.sub-table tr:nth-child(even)`: background-color
- `.sub-table-column-style1/2/3/4`: width
- `.sub_table_footer`: display, justify-content, align-items, margin-top
- `.sub_table_footer .total`: font-size, font-weight, color
- `.sub-table td img`: width, height, object-fit
- `.icon-group`: display, gap, justify-content, align-items
- `.view-icon` + hover: color, font-size, cursor, transition
- `.edit-icon` + hover: font-size, cursor, transition, color
- `.delete-icon` + hover: font-size, cursor, transition, color

### 9) Accordions + Alerts + Breadcrumbs
- `.accordion-container`, `.accordion`, `.accordion-header`, `.accordion-content`
- `.alert`, `.alert-title`, `.alert i`
- `.breadcrumbs` plus pseudo-element and link styling
- `.current`, `.icon`, Font Awesome `:before` rules

### 10) Media / Gallery / Empty State
- `.map-container`, `iframe`
- `.gallery-container`, `.gallery`, `.gallery-item`, `.gallery-item img`, `.caption`
- `.message-container`, `.message-container h1`, `.message-container p`, `.message-container .svg-icon`, `.cta-btn`
- `@keyframes fadeIn`, `fadeInZoom`, `slideInDown`, `slideInUp`

### 11) Cards, Rating, Wizard, Toggles, Segments
- `.cards_row`, `.note-card`, `.note-header`, `.note-content` and list items
- `.rating` group (inputs, labels, hover and checked behavior)
- `.wizard-container`, `.stepper`, `.step`, `.step .icon`, `.step.active .icon`, `.step .line`, `.step.active + .line`, `.wizard_content`, `.wizard_buttons`, `.wizard_button`
- Toggle group: `.toggle-container`, `.toggle-switch`, `.slider`, `.slider::before`, `input:checked + .slider`
- `.segmented-control` and its buttons

### 12) Reports + Grid Numbers + ImageText Grid
- `.report_header`, `.report_table`, `.report_footer`, and sub-elements
- `.grid-numbers-container`, `.grid-number`
- `.imagetext-grid-container` and its parts
- `.combined-inputs-*`

### 13) Flatpickr Styling
- `.flatpickr-months`, `.flatpickr-current-month`, month/year dropdown classes, `::before/::after` arrows, nav arrows, `.flatpickr-calendar`

### 14) Master-Details + Score Legend + Radio Checkmarks
- `.master-details-container`, `.master`, `.master-item` and states, `.details`
- `.master-header`, `.master-header-search-field`
- `.table-top-legend` + `.score-1..5`
- `.radio-cell` group + checkmark animation

### 15) Hierarchical Table
- `.hierarchical_table_container`, `.hierarchical_table`, sticky header, category row, record title/description, action icons, hover

### 16) List Selector Panel
- `.list-container`, `.list-item` and selected, `.list-checkbox` + label, `.item-*`, `.list-footer`

### 17) Vitals Grid
- `.vitals_grid` + media queries
- `.vital_card`, `.vital-input`, `.vital-label`, `.vital-unit`, overrides for inputs, `.vital_icon` and icon color classes

---

## Theme Creation Procedure
When creating a new Details Page theme:
1. Choose a unique theme name (Title Case).
2. Copy the canonical Details Page CSS skeleton exactly.
3. Adjust values in this order:
   - `:root` tokens first (accent family and selection/hover washes)
   - Base surfaces (`body`, `.container`, `.summary-card`, `.content-section`)
   - Buttons (filled and outline variants)
   - Tabs (active border and text)
   - Form controls (input/select backgrounds, borders, focus feel if any)
   - Tables and cards (subtables, hierarchical table, wizard header gradient)
   - Special components (breadcrumbs gradient, empty state, vitals icon colors)
4. Ensure:
   - readability is strong
   - hover and selection are obvious
   - internal scrolling works (content areas scroll, viewport does not)
   - spacing and alignment stay clean in both summary and content columns

---

## FileMaker Script Branch Template
Add a new branch in the Details Page CSS generator:

~~~text
Else If [ $$Theme = "New Details Theme Name" ]
  Set Variable [ $styles ; Value: "PASTE FULL UPDATED DETAILS CSS STRING HERE" ]
End If
~~~

---

## AI Request Template
Create a new Details Page theme using the canonical Details Page CSS in this document.

Theme Name: <New Theme Name>
Mode: <Light or Dark>
Vibe Keywords: <example: premium, calm, modern, sharp>

Rules:
- Copy the canonical Details Page CSS exactly.
- Change values only.
- Do not add, remove, or rename selectors.
- Do not add or remove properties.

Deliver:
1. Full updated CSS string
2. Full FileMaker `Else If` branch for the new theme

---

# Modal Pages CSS Generator (CauferoAppStarter)

## Purpose
This document defines the canonical CSS skeleton for Modal Pages in CauferoAppStarter.

A “Modal Page” is any WebViewer page that fills the viewport and behaves like a focused overlay screen, usually used for:
- creating or editing a record
- quick workflows (wizard steps)
- confirmations and approvals
- pickers and selectors
- embedded master-detail panels
- short reports or summary dashboards inside a modal shell

The AI agent must generate new Modal Page themes by changing CSS values only, while keeping the same selectors and the same property names.

---

## Script Context
- Input: `$$Theme` (or your theme selector variable)
- Output: `$styles` (full CSS stylesheet string)
- Script ends with: `Exit Script [ Text Result: $styles ]`

---

## Hard Rules (CSS Contract)
When creating a new Modal Page theme:
1. Keep the exact same selectors.
2. Keep the exact same property names under each selector.
3. Change values only.

Never:
- add selectors
- remove selectors
- rename selectors
- add properties
- remove properties

If a requested visual change would require new selectors or new properties, do not implement it. Achieve the best possible result using value changes only.

---

## Modal Page Layout Model
The page is built around a full-viewport modal shell:

1. **Body (Viewport)**
   - `body` is locked to viewport height.
   - The viewport must not scroll.

2. **Modal Shell**
   - `.modal` fills the viewport and contains header, body, footer.
   - `.modal` uses internal layout and internal scrolling only.

3. **Header**
   - `.modal-header` is the title zone.
   - Decorative gradient underline is done via `.modal-header::after`.

4. **Scrollable Content**
   - `.modal-body` is the primary scroll container (`overflow-y: auto`).

5. **Footer**
   - `.modal-footer` holds modal action buttons and stays visible while content scrolls.

6. **Optional Embedded Components**
   This canonical skeleton also includes shared components that can be used inside modals, such as:
   - two-column summary + content split (`.container`, `.summary-section`, `.content-section`)
   - tabs
   - form grids and inputs
   - subtables and icons
   - alerts, accordions, breadcrumbs
   - gallery, empty state, wizard, rating
   - flatpickr styling
   - master-detail panels
   - list selector panel
   - vitals grid

All scrolling must be internal. The modal page must not allow viewport scrolling.

---

## Canonical Modal Page CSS Skeleton (Source Of Truth)
The AI must copy this skeleton exactly for every theme and change values only.

~~~css
/* General reset and body styling */
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    font-family: Arial, sans-serif;
}

body {
    background-color: #f3f4f6;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    overflow: hidden;
}

.modal {
    width: 100%;
    height: 100%;
    background: #fff;
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
    display: flex;
    flex-direction: column;
    padding: 20px;
    overflow: hidden;
}

.modal-header {
    color: #4A90E2;
    text-align: center;
    padding: 20px 0;
    font-size: 20px;
    font-weight: bold;
    letter-spacing: 0.5px;
    position: relative;
}

.modal-header::after {
    content: '';
    display: block;
    width: 100%;
    height: 3px;
    background: linear-gradient(90deg, #4A90E2, #50E3C2);
    position: absolute;
    bottom: 0;
    left: 0;
}

.modal-body {
    flex: 1;
    padding: 20px;
    background-color: #F9FAFB;
    overflow-y: auto;
}

.modal-footer {
    padding: 20px;
    display: flex;
    justify-content: space-between;
    gap: 15px;
    border-top: 1px solid #E0E0E0;
    background: #fff;
}

/* Button Styling */
.modal-footer .button {
    padding: 10px 20px;
    font-size: 14px;
    font-weight: 500;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    color: white;
    transition: background-color 0.3s, box-shadow 0.3s;
}

.modal-footer .button.save {
    background-color: #28A745;
}

.modal-footer .button.save:hover {
    background-color: #218838;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.modal-footer .button.cancel {
    background-color: #DC3545;
}

.modal-footer .button.cancel:hover {
    background-color: #C82333;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.modal-footer .button.secondary {
    background-color: #007BFF;
}

.modal-footer .button.secondary:hover {
    background-color: #0056B3;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}


      /* Main container styling */
      .container {
         width: 100%;
         height: 100%;
         display: flex;
         padding: 20px;
         background: #fff;
         border-radius: 8px;
         box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
         overflow: hidden;
      }

      /* Left section styling (Summary) */
      .summary-section {
         width: 25%;
         padding-right: 20px;
         border-right: 1px solid #e0e0e0;
      }

      .summary-card {
         background: #edf7ff;
         padding: 15px;
         border-radius: 8px;
         box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
         height: 100%;
         display: flex;
         flex-direction: column;
	     overflow: hidden;
	     position: relative;
      }

.summary-content {
	overflow-y: auto;
	padding-right: 10px;
	height: calc(100% - 120px); /* Adjusted for additional spacing */
}

      .summary-card h2 {
         font-size: 18px;
         color: #333;
         text-align: center;
         margin-bottom: 20px;
      }

      .summary-info {
         font-size: 13px;
         color: #333;
         line-height: 1.4;
      }

      .summary-info p {
         margin: 5px 0;
      }

      .summary-info strong {
         color: #007BB0;
         font-weight: 600;
      }

      .summary-image {
         width: 300px;
         height: 200px;
         object-fit: contain;
         background: black;
         margin-bottom: 15px;
         display: block;
         border-radius: 8px;
         box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.2);
      }

/* Status Badge Styling */
.status-badge {
	display: inline-block;
	margin: auto;
	margin-top: 0px;
	margin-bottom: 5px;
	padding: 5px 10px;
	font-size: 12px;
	font-weight: bold;
	color: white;
	border-radius: 12px;
	text-align: center;
	width: fit-content;
}

.status-badge.gray {
    background-color: #6c757d; /* Neutral gray for draft */
}

.status-badge.yellow {
	background-color: #007BFF; /* Blue for pending */
}

.status-badge.green {
	background-color: #28a745; /* Green for approved */
}

.status-badge.red {
	background-color: #dc3545; /* Red for rejected */
}

      .qr-code {
         width: 80px;
         height: auto;
         margin: 15px auto;
         display: block;
      }

      .section-title {
         font-size: 16px;
         margin-top: 15px;
         font-weight: bold;
      }

      /* Line styling */
      hr {
         border: none;
         border-top: 1px solid #ddd;
         margin-top: auto;
         margin-bottom: 10px;
      }

      /* Button styling */
      .button-container {
         margin-top: 10px;
         display: flex;
         justify-content: center;
         gap: 10px;
      }

.save-button,
.cancel-button, 
.third-button,
.button_red,
.button_green,
.button_blue,
.button_yellow,
.button_purple {
         padding: 10px 15px;
         font-size: 13px;
         font-weight: 500;
         width: 50%;
         height: 40px;
         border: none;
         border-radius: 6px;
         cursor: pointer;
         color: white;
         transition: background-color 0.3s;
         display: flex;
         align-items: center;
         justify-content: center;
         gap: 5px;
}

.save-button {
         background-color: #28a745;
}

.save-button:hover {
         background-color: #218838;
}

.cancel-button {
         background-color: #dc3545;
}

.cancel-button:hover {
         background-color: #c82333;
}


/* Third Button Styling */
.third-button {
	background-color: #007BFF; /* Consistent blue for third button */
	width: 100%;
	margin-top: 10px;
}

.third-button:hover {
	background-color: #0056b3; /* Darker blue for hover */
}

      /* Add Another Item Button */
      .add-another-item-button {
         background-color: #007bff;
         color: white;
         padding: 10px 15px;
         font-size: 14px;
         border: none;
         border-radius: 6px;
         cursor: pointer;
         transition: background-color 0.3s;
         margin-top: 15px;
         width: fit-content;
         display: flex;
         align-items: center;
         gap: 5px;
      }

      .add-another-item-button:hover {
         background-color: #0056b3;
      }

.button1 {
    background-color: #FF9800; /* Bright orange to complement the blue and green theme */
    color: #fff;
    padding: 10px 15px;
    font-size: 14px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: background-color 0.3s;
    margin-top: 15px;
    width: fit-content;
    display: flex;
    align-items: center;
    gap: 5px;
}

.button1:hover {
    background-color: #F57C00; /* Darker orange for hover effect */
}

.button_red {
    background-color: #FF4C4C; /* Bright and bold red for high visibility */
}

.button_red:hover {
    background-color: #D43A3A; /* Slightly darker red for hover effect */
}

.button_green {
    background-color: #33CC99; /* Vibrant mint green for a fresh look */
}

.button_green:hover {
    background-color: #29A87C; /* Deeper green for hover effect */
}

.button_blue {
    background-color: #6699FF; /* Bright periwinkle blue for distinction */
}

.button_blue:hover {
    background-color: #4C7AD6; /* Slightly muted blue for hover effect */
}

.button_yellow {
    background-color: #FFD700; /* Radiant golden yellow for prominence */
}

.button_yellow:hover {
    background-color: #D4B200; /* Rich mustard yellow for hover effect */
}

.button_purple {
    background-color: #B266FF; /* Bright lavender purple for uniqueness */
}

.button_purple:hover {
    background-color: #8A4CD6; /* Softer but rich purple for hover effect */
}

/* Right section styling (Tabs and content) */
      .content-section {
         width: 75%;
         padding-left: 20px;
	     display: flex;
	     flex-direction: column;
	     overflow-y: hidden;
         height: 100%;
}

      .tabs {
         display: flex;
         border-bottom: 2px solid #e0e0e0;
         margin-bottom: 10px;
      }

      .tab {
         padding: 10px 20px;
         font-size: 14px;
         color: #555;
         cursor: pointer;
         border-bottom: 3px solid transparent;
         position: relative;
         transition: color 0.2s ease, border-bottom 0.2s ease;
      }

      .tab.active {
         color: #1e293b;
         font-weight: 600;
         border-bottom-color: #1e293b;
      }

      /* Badge styling */
      .tab.badge-tab::after {
         content: '5';
         position: absolute;
         top: -5px;
         right: 5px;
         background-color: #dc3545;
         color: white;
         font-size: 12px;
         font-weight: bold;
         padding: 3px 6px;
         border-radius: 50%;
         box-shadow: 0px 1px 3px rgba(0, 0, 0, 0.2);
      }

      .content {
         padding: 10px 0;
         display: none;
         overflow-y: auto; /* Allow vertical scrolling */
         max-height: 80vh; /* Limit the height and make it scrollable */
      }

      .content h3 {
         font-size: 16px;
         font-weight: 500;
         color: #333;
         margin-bottom: 5px;
      }

      .content p {
         color: #666;
         font-size: 14px;
         line-height: 1.5;
         margin-bottom: 15px;
      }

.form-grid1 {
         display: grid;
         grid-template-columns: repeat(1, 1fr);
         gap: 15px;
}

.form-grid2 {
         display: grid;
         grid-template-columns: repeat(2, 1fr);
         gap: 15px;
}

.form-grid3 {
         display: grid;
         grid-template-columns: repeat(3, 1fr);
         gap: 15px;
}

.form-grid4 {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 15px;
}

      .form-group {
         margin-bottom: 15px;
      }

      .form-group-2 {
         margin-bottom: 15px;
         padding: 0 20px;
      }

/* Instruction Styling */
.instructions1 {
	background: #e3f2fd;
	padding: 10px;
	padding-left: 30px;
	padding-right: 30px;
	border-left: 5px solid #007BFF;
	margin-bottom: 15px;
	border-radius: 5px;
	position: relative;
	font-size: 14px;
}

.instructions2 {
	background: #f3e5f5;
	padding: 10px;
	padding-left: 30px;
	padding-right: 30px;
	border-left: 5px solid #8E24AA;
	margin-bottom: 15px;
	border-radius: 5px;
	position: relative;
	font-size: 14px;
}

.instructions3 {
	background: #e8f5e9;
	padding: 10px;
	padding-left: 30px;
	padding-right: 30px;
	border-left: 5px solid #28a745;
	margin-bottom: 15px;
	border-radius: 5px;
	position: relative;
	font-size: 14px;
}

.instructions1 h2, 
.instructions2 h2, 
.instructions3 h2 {
	color: #333; /* Consistent dark text color for titles */
	font-size: 15px;
	margin-bottom: 5px;
}


      label {
         font-size: 14px;
         font-weight: 500;
         color: #333;
         display: block;
         margin-bottom: 5px;
      }

      input[type='text']:not(.combined-inputs-input),
      input[type='file']:not(.combined-inputs-input),
      input[type='number']:not(.combined-inputs-input),
      input[type='email']:not(.combined-inputs-input),
      input[type='date']:not(.combined-inputs-input),
      textarea {
         width: 100%;
         height: 38px;
         padding: 10px;
         font-size: 14px;
         color: #333;
         border: 1px solid #ddd;
         border-radius: 5px;
         outline: none;
         background-color: #f9f9f9;
      }

      textarea {
         height: 150px;
         resize: none;
      }

      /* Dropdown Styling */
      select:not(.combined-inputs-dropdown) {
         width: 100%;
         height: 38px;
         padding: 10px;
         font-size: 14px;
         color: #333;
         border: 1px solid #ddd;
         border-radius: 5px;
         outline: none;
         appearance: none;
         background-image: url('https://img.playbook.com/EiJ_h1h3ku1Tk5o2UUdY2GR36EdGRYp_FFrAGJtJ9BA/Z3M6Ly9wbGF5Ym9v/ay1hc3NldHMtcHVi/bGljLzU5ZmU3Zjk4/LTliMWEtNDZjYy1h/MTA4LWQ5ODI4YmJl/ZDZlZQ');
         background-repeat: no-repeat;
         background-position: right 10px center;
         background-size: 10px;
      }


/* Container for the Checkbox Group */
.checkbox-group {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(30px, 1fr));
    padding: 15px;
    border: 1px solid #E0E0E0;
    border-radius: 8px;
    background-color: #F9FAFB;
    box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.1);
    margin-top: 10px;
    font-family: Arial, sans-serif;
}

/* Main label */
label.main-label {
    font-weight: bold;
    margin-bottom: 10px;
    display: block;
    font-size: 16px;
    color: #333;
}

/* Individual Checkbox Wrapper */
.checkbox-group label {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 14px;
    color: #555;
    cursor: pointer;
    padding: 8px 12px;
    border-radius: 6px;
    transition: background-color 0.2s ease, box-shadow 0.2s ease;
}

/* Hover effect for checkbox label */
.checkbox-group label:hover {
    background-color: #EDF7FF;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* Checkbox Input Styling */
.checkbox-group input[type='checkbox'] {
    accent-color: #4A90E2; /* Matches the modal-header gradient */
    width: 18px;
    height: 18px;
    flex-shrink: 0;
    cursor: pointer;
}

/* Checked State */
.checkbox-group input[type='checkbox']:checked + span {
    font-weight: bold;
    color: #4A90E2;
}

/* Focus State */
.checkbox-group input[type='checkbox']:focus {
    outline: 2px solid #50E3C2;
    border-radius: 4px;
}

/* Disabled State */
.checkbox-group input[type='checkbox']:disabled + span {
    color: #CCC;
    cursor: not-allowed;
}

/* Checkbox Text Span */
.checkbox-group label span {
    font-size: 14px;
    color: #333;
    flex-grow: 1;
}


      /* Another Item Table */
      .sub-table {
	     width: 100%;
	     border-collapse: collapse;
      }

      .sub-table th,
      .sub-table td {
         border: 1px solid #ddd;
         padding: 8px;
         text-align: left;
         font-size: 12px;
      }

      .sub-table th {
         background-color: #f2f2f2;
         font-weight: bold;
      }

      .sub-table tr:nth-child(even) {
         background-color: #f9f9f9;
      }

      /* Specific column widths */
      .sub-table-column-style1 {
         width: 30%;
      }

      .sub-table-column-style2 {
         width: 15%;
      }

      .sub-table-column-style3 {
         width: 10%;
      }

.sub-table-column-style4 {
	width: 5%;
}

.sub_table_footer {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-top: 10px;
}


/* Sub Table Footer Styling */
.sub_table_footer .total {
	font-size: 18px;
	font-weight: bold;
	color: #dc3545; /* Red for total indicator */
}

.sub-table td img {
 width: 50px;
 height: 50px;
 object-fit: cover;
}

.icon-group {
   display: flex;
   gap: 15px; /* Space between icons */
   justify-content: center;
   align-items: center;
}

.view-icon {
   color: #007BFF; /* Consistent blue */
   font-size: 16px;
   cursor: pointer;
   transition: color 0.3s;
}

.view-icon:hover {
   color: #0056b3; /* Darker blue on hover */
}

.edit-icon, .delete-icon {
   font-size: 16px;
   cursor: pointer;
   transition: color 0.3s;
}

.edit-icon {
   color: #8E24AA; /* Purple for edit */
}

.edit-icon:hover {
   color: #6A1B9A; /* Darker purple on hover */
}

.delete-icon {
   color: #dc3545; /* Red for delete */
}

.delete-icon:hover {
   color: #c82333; /* Darker red on hover */
}


      /* Accordion */
      .accordion-container {
         margin-top: 10px;
      }

      .accordion {
         border: 1px solid #ddd;
         border-radius: 5px;
         margin-bottom: 10px;
      }

      .accordion-header {
         padding: 10px;
         background: #f2f2f2;
         cursor: pointer;
         display: flex;
         justify-content: space-between;
         align-items: center;
         font-weight: 500;
      }

      .accordion-header:hover {
         background: #e0e0e0;
      }

      .accordion-content {
         padding: 10px;
         display: none;
         background: #fafafa;
      }

      /* Improved Inline Alert */
      .alert {
         padding: 15px 20px;
         background-color: #f8d7da;
         color: #721c24;
         border: 1px solid #f5c6cb;
         border-radius: 8px;
         display: flex;
         justify-content: space-between;
         align-items: center;
         font-size: 14px;
         font-weight: 500;
         box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
         margin-bottom: 15px;
      }

      .alert-title {
         font-weight: bold;
         margin-right: 10px;
      }

      .alert i {
         cursor: pointer;
         color: #721c24;
      }

      .alert i:hover {
         color: #491116;
      }

      /* Breadcrumbs Styles */
      .breadcrumbs {
         display: flex;
         align-items: center;
         padding: 10px 20px;
         background: #fff;
         border-radius: 10px;
         box-shadow: 0 6px 15px rgba(0, 0, 0, 0.1);
         position: relative;
         margin-bottom: 20px;
      }

      .breadcrumbs::before {
         content: '';
         position: absolute;
         left: 0;
         right: 0;
         top: 0;
         height: 8px;
         background: linear-gradient(135deg, #4a90e2, #50e3c2);
         border-radius: 10px 10px 0 0;
         z-index: -1;
      }

      .breadcrumbs a {
         text-decoration: none;
         color: #4a90e2;
         padding: 10px 15px;
         border-radius: 5px;
         position: relative;
         transition: background-color 0.3s, color 0.3s;
         display: flex;
         align-items: center;
      }

      .breadcrumbs a:hover {
         background-color: rgba(74, 144, 226, 0.1);
         color: #007BFF;
      }

      .breadcrumbs a::after {
         content: '>';
         margin-left: 10px;
         margin-right: 10px;
         color: #aaa;
         font-weight: bold;
      }

      .breadcrumbs a:last-child::after {
         content: '';
      }

      .current {
         color: #555;
         font-weight: bold;
         pointer-events: none;
         position: relative;
         z-index: 1;
      }

      .icon {
         margin-right: 5px;
         font-size: 18px;
         transition: transform 0.3s;
      }

      .breadcrumbs a:hover .icon {
         transform: scale(1.2);
      }

      /* Example icon styles - using Font Awesome */
      .fa-home:before {
         content: '\f015';
      }

      .fa-folder:before {
         content: '\f07b';
      }

      .fa-file:before {
         content: '\f15b';
      }

      .fa-chevron-right:before {
         content: '\f054';
      }

      /* Custom Sign-Up Button for Tab 4 */
      .big_button {
         display: flex;
         align-items: center;
         background-color: #4A90E2;
         color: white;
         font-size: 16px;
         font-weight: bold;
         padding: 10px 20px;
         border: none;
         border-radius: 5px;
         box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
         cursor: pointer;
         margin-top: 15px;
      }

      .big_button span {
         margin-right: 10px;
      }

      .divider {
         height: 24px;
         width: 1px;
         background-color: white;
         margin: 0 10px;
      }

      .checkmark {
         font-size: 16px;
      }


    /* Basic styling for the iframe container */
    .map-container {
      width: 50%;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: top;
      align-items: left;
    }

    iframe {
      width: 100%;
      height: 70%;
      border: none;
      border-radius: 8px;
      box-shadow: 0px 0px 10px rgba(0,0,0,0.1);
    }

    /* Gallery Styles */
    .gallery-container {
      width: 45%;
      margin-left: 30px;
    }

    .gallery {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
      gap: 10px;
    }

    .gallery-item {
      position: relative;
      overflow: hidden;
      border-radius: 8px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    .gallery-item img {
      width: 100%;
      height: 100px;
      object-fit: cover;
      transition: transform 0.3s ease;
    }

    .gallery-item:hover img {
      transform: scale(1.05);
    }

    .gallery-item .caption {
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      background: rgba(0, 0, 0, 0.6);
      color: white;
      text-align: center;
      padding: 5px 0;
      font-size: 12px;
      display: none;
    }

    .gallery-item:hover .caption {
      display: block;
    }

    /* No Product Found Message Styles */
   .message-container {
      text-align: center;
      background: #ffffff;
      padding: 50px 60px;
      border-radius: 16px;
      box-shadow: 0 8px 30px rgba(0, 0, 0, 0.1);
      animation: fadeIn 0.8s ease-out forwards;
      max-width: 500px;
      position: absolute;
      top: 25%;
      left: 45%;
      transform: translate(-50%, -50%);
    }

      .message-container h1 {
         font-size: 28px;
         color: #333;
         margin-bottom: 10px;
         animation: slideInDown 0.8s ease-out;
      }

      .message-container p {
         font-size: 16px;
         color: #555;
         margin-bottom: 20px;
         animation: slideInUp 1s ease-out;
      }

      .message-container img {
         width: 100px;
         margin-bottom: 20px;
         animation: fadeInZoom 1s ease-out;
      }

      .cta-btn {
         display: inline-block;
         margin-top: 20px;
         padding: 14px 30px;
         font-size: 16px;
         font-weight: 600;
         color: #ffffff;
         background: linear-gradient(135deg, #007BFF, #0056D2);
         border: none;
         border-radius: 8px;
         cursor: pointer;
         text-decoration: none;
         box-shadow: 0 4px 12px rgba(0, 123, 255, 0.3);
         transition: background 0.3s ease, box-shadow 0.3s ease, transform 0.2s ease;
      }

      .cta-btn:hover {
         background: linear-gradient(135deg, #0056D2, #007BFF);
         transform: translateY(-2px);
         box-shadow: 0 6px 20px rgba(0, 123, 255, 0.4);
      }

      @keyframes fadeIn {
         0% {
            opacity: 0;
            transform: translateY(20px);
         }

         100% {
            opacity: 1;
            transform: translateY(0);
         }
      }

      @keyframes fadeInZoom {
         0% {
            opacity: 0;
            transform: scale(0.8);
         }

         100% {
            opacity: 1;
            transform: scale(1);
         }
      }

      @keyframes slideInDown {
         0% {
            opacity: 0;
            transform: translateY(-20px);
         }

         100% {
            opacity: 1;
            transform: translateY(0);
         }
      }

      @keyframes slideInUp {
         0% {
            opacity: 0;
            transform: translateY(20px);
         }

         100% {
            opacity: 1;
            transform: translateY(0);
         }
      }

      /* Row and Card Styling */
      .cards_row {
         display: flex;
         gap: 20px;
         margin-top: 10px;
      }

      .note-card {
         flex: 1;
         padding: 20px;
         border-radius: 8px;
         box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
         background-color: #e0f7fa;
         display: flex;
         flex-direction: column;
      }

      .note-header {
         font-weight: bold;
         margin-bottom: 10px;
         font-size: 16px;
         display: flex;
         justify-content: space-between;
      }

.note-content {
    flex-grow: 1;
    overflow-y: auto; /* Allow vertical scrolling */
    max-height: 300px; /* Set the max height as per your requirement */
}

      .note-content ul {
         list-style-type: none;
         padding: 0;
         margin: 0;
      }

      .note-content li {
         padding: 10px;
         margin: 10px 0;
         background-color: #ffffff;
         border-radius: 6px;
         display: flex;
         justify-content: space-between;
         align-items: center;
         box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
         transition: transform 0.2s;
      }

      .note-content li:hover {
         transform: scale(1.02);
      }

      .note-content button {
         background: transparent;
         border: none;
         font-size: 18px;
         color: #007bff;
         cursor: pointer;
      }

      .note-content button:hover {
         color: #0056b3;
      }


    .rating {
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 5px;
      direction: rtl; /* Corrects the display order for right-to-left alignment */
    }
    .rating input {
      display: none;
    }
    .rating label {
      font-size: 30px;
      color: #ddd;
      cursor: pointer;
      transition: color 0.3s;
    }
    .rating label:hover,
    .rating label:hover ~ label {
      color: #ffc107;
    }
    .rating input:checked ~ label {
      color: #ffc107;
    }
    .rating input:checked + label:hover,
    .rating input:checked + label:hover ~ label {
      color: #ffab00;
    }
    .rating label:hover ~ label {
      color: #ddd; /* Reverts stars on the left side */
    }

    .wizard-container {
      width: 600px;
      background-color: #fff;
      border-radius: 12px;
      box-shadow: 0 6px 16px rgba(0, 0, 0, 0.1);
      overflow: hidden;
    }

    .stepper {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 20px 30px;
      background: linear-gradient(135deg, #007bff, #4c9fff);
      color: #fff;
      position: relative;
    }

    .step {
      text-align: center;
      flex: 1;
      position: relative;
    }

    .step .icon {
      width: 50px;
      height: 50px;
      border-radius: 50%;
      background-color: #e0e0e0;
      color: #007bff;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 18px;
      font-weight: bold;
      margin: 0 auto 8px;
      transition: background-color 0.3s ease, color 0.3s ease, transform 0.3s ease, box-shadow 0.3s ease;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
    }

    .step.active .icon {
      background-color: #fff;
      color: #007bff;
      border: 2px solid #007bff;
      transform: scale(1.2);
      box-shadow: 0 0 10px 3px rgba(0, 123, 255, 0.6);
    }

    .step .line {
      position: absolute;
      top: 25px;
      left: 50%;
      width: 100%;
      height: 4px;
      background-color: #e0e0e0;
      z-index: -1;
      transition: background-color 0.3s ease;
    }

    .step.active + .line {
      background-color: #fff;
    }

    .step p {
      font-size: 14px;
      font-weight: bold;
      margin-top: 8px;
      color: #fff;
    }

    .wizard_content {
      padding: 20px;
      animation: fadeIn 0.3s ease;
    }

    .wizard_content div {
      display: none;
    }

    .wizard_content div.active {
      display: block;
    }

    .wizard_buttons {
      display: flex;
      justify-content: space-between;
      padding: 20px;
      background-color: #f9f9f9;
    }

    .wizard_button {
      padding: 10px 20px;
      border: none;
      border-radius: 4px;
      background-color: #007bff;
      color: #fff;
      cursor: pointer;
      transition: background-color 0.3s ease, box-shadow 0.3s ease;
    }

    .wizard_button:hover {
      background-color: #0056b3;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
    }

    .wizard_button:disabled {
      background-color: #ccc;
      cursor: not-allowed;
      box-shadow: none;
    }

    .toggle-container {
      display: flex;
      align-items: center;
      gap: 10px;
      font-size: 16px;
      color: #333;
    }

    .toggle-switch {
      display: inline-block;
      width: 50px;
      height: 25px;
      position: relative;
    }

    .toggle-switch input {
      opacity: 0;
      width: 0;
      height: 0;
    }

    .slider {
      position: absolute;
      cursor: pointer;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-color: #ccc;
      border-radius: 25px;
      transition: background-color 0.3s;
    }

    .slider::before {
      content: '';
      position: absolute;
      height: 21px;
      width: 21px;
      left: 2px;
      bottom: 2px;
      background-color: white;
      border-radius: 50%;
      transition: transform 0.3s;
    }

    input:checked + .slider {
      background-color: #4caf50;
    }

    input:checked + .slider::before {
      transform: translateX(25px);
    }

    .status {
      font-weight: bold;
    }

    .segmented-control {
      display: flex;
      background-color: #ddd;
      border-radius: 8px;
      overflow: hidden;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      width: 400px;
    }

    .segmented-control button {
      flex: 1;
      padding: 10px 20px;
      background-color: transparent;
      border: none;
      outline: none;
      cursor: pointer;
      font-size: 16px;
      color: #333;
      transition: background-color 0.3s, color 0.3s;
    }

    .segmented-control button.active {
      background-color: #4caf50;
      color: white;
    }

    .segmented-control button:not(:last-child) {
      border-right: 1px solid #ccc;
    }

    .segmented-control button:hover {
      background-color: #bfbfbf;
    }

.icon {
	font-size: 16px;
	color: #999;
	cursor: pointer;
}

.combined-inputs-container {
    width: 100%;
    height: 38px;
    padding: 10px;
    font-size: 14px;
    color: #333;
    border: 1px solid #ddd;
    border-radius: 5px;
    outline: none;
    background-color: #f9f9f9;
    display: flex;
    align-items: center;
    box-sizing: border-box;
}

.combined-inputs-input {
    border: none;
    outline: none;
    text-align: center;
    font-size: 14px;
    background-color: transparent;
    color: #333;
}

.combined-inputs-dropdown {
    padding: 10px;
    border: none;
    outline: none;
    font-size: 12px;
    text-align: center;
    color: #333;
    background-color: transparent;
    appearance: none; /* Removes default dropdown styling */
}

/* 📅 Flatpickr Top Banner Styling */
.flatpickr-months {
  display: flex;
  align-items: center; /* Align vertically */
  justify-content: center; /* Align horizontally */
  padding: 8px 0;
  gap: 12px; /* Increased spacing for clarity */
}

/* 📆 Month and Year Container Styling */
.flatpickr-current-month {
  display: flex;
  align-items: center; /* Vertical alignment */
  justify-content: center; /* Horizontal alignment */
  gap: 12px; /* Space between month and year */
  line-height: 1.2; /* Prevent content cutoff */
}

/* 📅 Month Dropdown Styling */
.flatpickr-monthDropdown-months {
  width: 120px; /* Ensure enough space for full month names */
  height: 36px; /* Matching heights */
  padding: 4px 8px;
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 14px;
  line-height: 1.5;
}

/* 📆 Year Dropdown Container */
.flatpickr-yearDropdown-years {
  position: relative; /* Allows precise arrow placement */
  width: 90px; /* Consistent width */
  height: 36px; /* Match the month dropdown height */
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 14px;
  line-height: 1.5;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 📆 Year Input Styling */
.flatpickr-current-month input.cur-year {
  width: 100%;
  height: 100%;
  text-align: center;
  font-size: 14px;
  border: none;
  box-sizing: border-box;
  -moz-appearance: textfield; /* Firefox */
}

.flatpickr-current-month input.cur-year::-webkit-inner-spin-button,
.flatpickr-current-month input.cur-year::-webkit-outer-spin-button {
  -webkit-appearance: none; /* Chrome, Safari, Edge */
  margin: 0;
}

/* 🠕🠗 Custom Year Arrows on Right Side */
.flatpickr-yearDropdown-years::after {
  content: '▲';
  position: absolute;
  right: 4px;
  top: 4px;
  font-size: 10px;
  cursor: pointer;
  pointer-events: all;
}

.flatpickr-yearDropdown-years::before {
  content: '▼';
  position: absolute;
  right: 4px;
  bottom: 4px;
  font-size: 10px;
  cursor: pointer;
  pointer-events: all;
}

/* ⬅️➡️ Navigation Arrows Styling */
.flatpickr-prev-month,
.flatpickr-next-month {
  font-size: 18px;
  cursor: pointer;
  padding: 4px;
  line-height: 1;
}

.flatpickr-prev-month {
  margin-right: 8px;
}

.flatpickr-next-month {
  margin-left: 8px;
}

/* ✨ Ensure Uniform Alignment for Both Fields */
.flatpickr-monthDropdown-months,
.flatpickr-yearDropdown-years,
.flatpickr-current-month input.cur-year {
  display: flex;
  align-items: center;
  justify-content: center;
  vertical-align: middle;
}

/* 🛠️ Overall Calendar Adjustment */
.flatpickr-calendar {
  margin: 0;
  padding: 0;
  border-radius: 8px;
  overflow: hidden;
}

.master-details-container {
  display: flex;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

/* Master Section */
.master {
  background: #f7f9fc;
  border-right: 1px solid #ddd;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.master-item {
  display: flex;
  align-items: center;
  padding: 5px 5px;
  border-bottom: 1px solid #ececec;
  cursor: pointer;
  transition: background-color 0.3s ease, border-left 0.3s ease;
  border-left: 4px solid transparent;
}

.master-item:hover {
  background-color: #f1f7ff;
}

.master-item.selected {
  background-color: #e8f0fe;
  border-left: 4px solid #007BFF;
  font-weight: 600;
}

.master-item img {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  margin-right: 10px;
  object-fit: cover;
  border: 1px solid #ccc;
}

.master-item-text {
  display: flex;
  flex-direction: column;
}

.master-item-text1 {
  font-weight: 600;
  font-size: 0.85em;
  color: #2c3e50;
}

.master-item-text2 {
  font-size: 0.75em;
  color: #666;
}

/* Details Section */
.details {
  padding: 20px;
  background-color: #fff;
  overflow-y: auto;
}

.details h2 {
  margin-top: 0;
  font-size: 1.2em;
  color: #2c3e50;
}

.details p {
  font-size: 0.8em;
  line-height: 1.5;
  color: #444;
}

.details img {
  max-width: 100%;
  border-radius: 3px;
  border: 1px solid #ddd;
}


/* Responsive Design */
@media (max-width: 768px) {
.container {
  flex-direction: column;
}

.master, .details {
  width: 100%;
  height: 50%;
}
}

/* Master Header Styling */
.master-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px;
  border-bottom: 1px solid #ececec;
  background-color: #f7f9fc;
  position: sticky; /* Keeps the header fixed */
  top: 0; /* Ensures the header stays at the top */
  z-index: 1; /* Ensures it stays above the list items */
}

.master-header-search-field {
   padding: 8px 10px;
   border: 1px solid #ddd;
   border-radius: 4px;
   font-size: 14px;
}

/* List Container */
.list-container {
    background: #F9FAFB; /* Matches modal body */
    flex: 1;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    height: 100%;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* List Item */
.list-item {
    display: flex;
    align-items: flex-start;
    padding: 12px 16px;
    border-bottom: 1px solid #E0E0E0;
    transition: background-color 0.3s, box-shadow 0.3s;
    cursor: pointer;
}

.list-item:last-child {
    border-bottom: none;
}

.list-item:hover {
    background: #EDF7FF; /* Matches summary card hover effect */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.list-item.selected {
    background: #E8F5E9; /* Soft green to match modal-footer save button */
    box-shadow: inset 0 0 5px rgba(40, 167, 69, 0.2);
}

/* Checkbox Styling */
.list-checkbox {
    margin-top: 15px;
    margin-right: 12px;
}

.list-checkbox input {
    visibility: hidden;
    position: absolute;
}

.list-checkbox .checkbox-label {
    width: 20px;
    height: 20px;
    border: 2px solid #D1D5DB;
    border-radius: 4px;
    display: inline-block;
    position: relative;
    transition: all 0.3s ease;
}

.list-checkbox input:checked + .checkbox-label {
    background: #4A90E2; /* Matches modal-header */
    border-color: #4A90E2;
}

.list-checkbox .checkbox-label::after {
    content: '';
    position: absolute;
    top: 0px;
    left: 4px;
    width: 5px;
    height: 10px;
    border: solid white;
    border-width: 0 2px 2px 0;
    transform: rotate(45deg) scale(0);
    transition: transform 0.2s ease;
}

.list-checkbox input:checked + .checkbox-label::after {
    transform: rotate(45deg) scale(1);
}

/* Record Details Styling */
.item-details {
    flex-grow: 1;
}

.item-title {
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 4px;
    color: #333; /* Matches section-title */
}

.item-subtitle {
    font-size: 14px;
    color: #555;
}

.item-description {
    font-size: 12px;
    color: #888;
    margin-top: 4px;
}

/* Footer */
.list-footer {
    margin-top: auto; /* Pushes the footer to the bottom */
    text-align: center;
    padding: 12px;
    font-size: 14px;
    background: #FFFFFF; /* Matches modal-footer */
    color: #666;
    border-top: 1px solid #E0E0E0;
    border-radius: 0 0 8px 8px;
}

.list-footer-highlight {
    color: #28A745; /* Matches save button */
    font-weight: bold;
}

/* Grid Layout */
.vitals_grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
    justify-content: center;
}

/* Responsive: Stack cards on smaller screens */
@media (max-width: 1024px) {
    .vitals_grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

@media (max-width: 768px) {
    .vitals_grid {
        grid-template-columns: repeat(1, 1fr);
    }
}

/* Vital Card Styling */
.vital_card {
    background: #fff;
    border-radius: 8px;
    padding: 16px;
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.vital_card:hover {
    box-shadow: 0px 6px 12px rgba(0, 0, 0, 0.12);
}

/* Input Fields */
.vital-input {
    background: transparent;
    border: none;
    outline: none;
    font-size: 1.8rem;
    font-weight: bold;
    width: 75px;
    text-align: center;
    color: #007BFF;
    transition: color 0.3s ease-in-out;
}

.vital-label {
    font-size: 0.9rem;
    font-weight: 600;
    color: #333;
}

.vital-unit {
    font-size: 0.8rem;
    color: #666;
}

/* Override general input styles */
.vital_card input[type='number'],
.vital_card input[type='text'] {
    border: none !important;
    background: transparent !important;
    box-shadow: none !important;
    padding: 0 !important;
    font-size: 1.8rem !important;
    font-weight: bold !important;
    text-align: center !important;
    color: #007BFF !important;
    width: 100px !important;
}

/* Blood pressure inputs */
.vital_card .bp-slash {
    font-size: 1.8rem;
    font-weight: bold;
    color: #007BFF;
    padding: 0 4px;
}

/* Icon Styling */
.vital_icon {
    font-size: 3rem;
    padding: 8px;
    border-radius: 8px;
}

/* Icon Colors */
.height-icon { color: #007BFF; background-color: #E3F2FD; }
.weight-icon { color: #28A745; background-color: #E8F5E9; }
.bmi-icon { color: #8E44AD; background-color: #F3E5F5; }
.temp-icon { color: #E74C3C; background-color: #FDECEA; }
.bp-icon { color: #D35400; background-color: #FBE7DC; }
.pulse-icon { color: #F39C12; background-color: #FDF3D9; }
.resp-icon { color: #16A085; background-color: #E0F5F5; }
.oxygen-icon { color: #3498DB; background-color: #E4F0FA; }
.glucose-icon { color: #E67E22; background-color: #FDF2E0; }
~~~

---

## Contract Manifest (Selectors and Property Names)
When generating a new theme, the AI must ensure every selector below exists and keeps the same property names.

### 1) Global Base
- `*`: box-sizing, margin, padding, font-family
- `body`: background-color, height, display, flex-direction, justify-content, align-items, overflow

### 2) Modal Shell
- `.modal`: width, height, background, box-shadow, display, flex-direction, padding, overflow
- `.modal-header`: color, text-align, padding, font-size, font-weight, letter-spacing, position
- `.modal-header::after`: content, display, width, height, background, position, bottom, left
- `.modal-body`: flex, padding, background-color, overflow-y
- `.modal-footer`: padding, display, justify-content, gap, border-top, background

### 3) Modal Footer Buttons
- `.modal-footer .button`: padding, font-size, font-weight, border, border-radius, cursor, color, transition
- `.modal-footer .button.save`: background-color
- `.modal-footer .button.save:hover`: background-color, box-shadow
- `.modal-footer .button.cancel`: background-color
- `.modal-footer .button.cancel:hover`: background-color, box-shadow
- `.modal-footer .button.secondary`: background-color
- `.modal-footer .button.secondary:hover`: background-color, box-shadow

### 4) Optional Embedded Two-Column Section
- `.container`: width, height, display, padding, background, border-radius, box-shadow, overflow
- `.summary-section`: width, padding-right, border-right
- `.summary-card`: background, padding, border-radius, box-shadow, height, display, flex-direction, overflow, position
- `.summary-content`: overflow-y, padding-right, height
- `.summary-card h2`: font-size, color, text-align, margin-bottom
- `.summary-info`: font-size, color, line-height
- `.summary-info p`: margin
- `.summary-info strong`: color, font-weight
- `.summary-image`: width, height, object-fit, background, margin-bottom, display, border-radius, box-shadow

### 5) Status Badge + QR + Divider
- `.status-badge`: display, margin, margin-top, margin-bottom, padding, font-size, font-weight, color, border-radius, text-align, width
- `.status-badge.gray/.yellow/.green/.red`: background-color
- `.qr-code`: width, height, margin, display
- `.section-title`: font-size, margin-top, font-weight
- `hr`: border, border-top, margin-top, margin-bottom

### 6) Shared Button Set (Inside Summary Card)
- `.button-container`: margin-top, display, justify-content, gap
- `.save-button, .cancel-button, .third-button, .button_red, .button_green, .button_blue, .button_yellow, .button_purple`
  - padding, font-size, font-weight, width, height, border, border-radius, cursor, color, transition, display, align-items, justify-content, gap
- `.save-button` + hover: background-color
- `.cancel-button` + hover: background-color
- `.third-button` + hover: background-color, width, margin-top
- `.add-another-item-button` + hover:
  - background-color, color, padding, font-size, border, border-radius, cursor, transition, margin-top, width, display, align-items, gap
- `.button1` + hover:
  - background-color, color, padding, font-size, border, border-radius, cursor, transition, margin-top, width, display, align-items, gap
- `.button_red/.green/.blue/.yellow/.purple` + hover: background-color

### 7) Tabs + Content
- `.content-section`: width, padding-left, display, flex-direction, overflow-y, height
- `.tabs`: display, border-bottom, margin-bottom
- `.tab`: padding, font-size, color, cursor, border-bottom, position, transition
- `.tab.active`: color, font-weight, border-bottom-color
- `.tab.badge-tab::after`: content, position, top, right, background-color, color, font-size, font-weight, padding, border-radius, box-shadow
- `.content`: padding, display, overflow-y, max-height
- `.content h3`: font-size, font-weight, color, margin-bottom
- `.content p`: color, font-size, line-height, margin-bottom

### 8) Form Grid + Inputs + Select
- `.form-grid1/2/3/4`: display, grid-template-columns, gap
- `.form-group`: margin-bottom
- `.form-group-2`: margin-bottom, padding
- `.instructions1/2/3`: background, padding, padding-left, padding-right, border-left, margin-bottom, border-radius, position, font-size
- `.instructions1 h2, .instructions2 h2, .instructions3 h2`: color, font-size, margin-bottom
- `label`: font-size, font-weight, color, display, margin-bottom
- `input[type=...]:not(.combined-inputs-input), textarea`: width, height, padding, font-size, color, border, border-radius, outline, background-color
- `textarea`: height, resize
- `select:not(.combined-inputs-dropdown)`: width, height, padding, font-size, color, border, border-radius, outline, appearance, background-image, background-repeat, background-position, background-size

### 9) Checkbox Group (Extended)
- `.checkbox-group`: display, grid-template-columns, padding, border, border-radius, background-color, box-shadow, margin-top, font-family
- `label.main-label`: font-weight, margin-bottom, display, font-size, color
- `.checkbox-group label`: display, align-items, gap, font-size, color, cursor, padding, border-radius, transition
- `.checkbox-group label:hover`: background-color, box-shadow
- `.checkbox-group input[type='checkbox']`: accent-color, width, height, flex-shrink, cursor
- `.checkbox-group input[type='checkbox']:checked + span`: font-weight, color
- `.checkbox-group input[type='checkbox']:focus`: outline, border-radius
- `.checkbox-group input[type='checkbox']:disabled + span`: color, cursor
- `.checkbox-group label span`: font-size, color, flex-grow

### 10) Subtable + Icons
- `.sub-table`: width, border-collapse
- `.sub-table th, .sub-table td`: border, padding, text-align, font-size
- `.sub-table th`: background-color, font-weight
- `.sub-table tr:nth-child(even)`: background-color
- `.sub-table-column-style1/2/3/4`: width
- `.sub_table_footer`: display, justify-content, align-items, margin-top
- `.sub_table_footer .total`: font-size, font-weight, color
- `.sub-table td img`: width, height, object-fit
- `.icon-group`: display, gap, justify-content, align-items
- `.view-icon` + hover: color, font-size, cursor, transition
- `.edit-icon` + hover: font-size, cursor, transition, color
- `.delete-icon` + hover: font-size, cursor, transition, color

### 11) Accordions + Alerts + Breadcrumbs
- `.accordion-container`: margin-top
- `.accordion`: border, border-radius, margin-bottom
- `.accordion-header`: padding, background, cursor, display, justify-content, align-items, font-weight
- `.accordion-header:hover`: background
- `.accordion-content`: padding, display, background
- `.alert`: padding, background-color, color, border, border-radius, display, justify-content, align-items, font-size, font-weight, box-shadow, margin-bottom
- `.alert-title`: font-weight, margin-right
- `.alert i` + hover: cursor, color
- `.breadcrumbs`: display, align-items, padding, background, border-radius, box-shadow, position, margin-bottom
- `.breadcrumbs::before`: content, position, left, right, top, height, background, border-radius, z-index
- `.breadcrumbs a`: text-decoration, color, padding, border-radius, position, transition, display, align-items
- `.breadcrumbs a:hover`: background-color, color
- `.breadcrumbs a::after`: content, margin-left, margin-right, color, font-weight
- `.breadcrumbs a:last-child::after`: content
- `.current`: color, font-weight, pointer-events, position, z-index
- `.icon`: margin-right, font-size, transition
- `.breadcrumbs a:hover .icon`: transform
- `.fa-home:before`, `.fa-folder:before`, `.fa-file:before`, `.fa-chevron-right:before`: content

### 12) Media, Gallery, Empty State, Animations
- `.map-container`: width, height, display, justify-content, align-items (two align-items lines exist)
- `iframe`: width, height, border, border-radius, box-shadow
- `.gallery-container`: width, margin-left
- `.gallery`: display, grid-template-columns, gap
- `.gallery-item`: position, overflow, border-radius, box-shadow
- `.gallery-item img`: width, height, object-fit, transition
- `.gallery-item:hover img`: transform
- `.gallery-item .caption`: position, bottom, left, width, background, color, text-align, padding, font-size, display
- `.gallery-item:hover .caption`: display
- `.message-container`: text-align, background, padding, border-radius, box-shadow, animation, max-width, position, top, left, transform
- `.message-container h1`: font-size, color, margin-bottom, animation
- `.message-container p`: font-size, color, margin-bottom, animation
- `.message-container img`: width, margin-bottom, animation
- `.cta-btn` + hover: display, margin-top, padding, font-size, font-weight, color, background, border, border-radius, cursor, text-decoration, box-shadow, transition, transform
- `@keyframes fadeIn`: opacity, transform
- `@keyframes fadeInZoom`: opacity, transform
- `@keyframes slideInDown`: opacity, transform
- `@keyframes slideInUp`: opacity, transform

### 13) Cards and Rating
- `.cards_row`: display, gap, margin-top
- `.note-card`: flex, padding, border-radius, box-shadow, background-color, display, flex-direction
- `.note-header`: font-weight, margin-bottom, font-size, display, justify-content
- `.note-content`: flex-grow, overflow-y, max-height
- `.note-content ul`: list-style-type, padding, margin
- `.note-content li`: padding, margin, background-color, border-radius, display, justify-content, align-items, box-shadow, transition
- `.note-content li:hover`: transform
- `.note-content button` + hover: background, border, font-size, color, cursor
- `.rating`: display, justify-content, align-items, gap, direction
- `.rating input`: display
- `.rating label`: font-size, color, cursor, transition
- `.rating label:hover, .rating label:hover ~ label`: color
- `.rating input:checked ~ label`: color
- `.rating input:checked + label:hover, .rating input:checked + label:hover ~ label`: color
- `.rating label:hover ~ label`: color

### 14) Wizard, Toggles, Segmented Control
- `.wizard-container`: width, background-color, border-radius, box-shadow, overflow
- `.stepper`: display, justify-content, align-items, padding, background, color, position
- `.step`: text-align, flex, position
- `.step .icon`: width, height, border-radius, background-color, color, display, justify-content, align-items, font-size, font-weight, margin, transition, box-shadow
- `.step.active .icon`: background-color, color, border, transform, box-shadow
- `.step .line`: position, top, left, width, height, background-color, z-index, transition
- `.step.active + .line`: background-color
- `.step p`: font-size, font-weight, margin-top, color
- `.wizard_content`: padding, animation
- `.wizard_content div`: display
- `.wizard_content div.active`: display
- `.wizard_buttons`: display, justify-content, padding, background-color
- `.wizard_button` + hover + disabled: padding, border, border-radius, background-color, color, cursor, transition, box-shadow
- `.toggle-container`: display, align-items, gap, font-size, color
- `.toggle-switch`: display, width, height, position
- `.toggle-switch input`: opacity, width, height
- `.slider`: position, cursor, top, left, right, bottom, background-color, border-radius, transition
- `.slider::before`: content, position, height, width, left, bottom, background-color, border-radius, transition
- `input:checked + .slider`: background-color
- `input:checked + .slider::before`: transform
- `.status`: font-weight
- `.segmented-control`: display, background-color, border-radius, overflow, box-shadow, width
- `.segmented-control button`: flex, padding, background-color, border, outline, cursor, font-size, color, transition
- `.segmented-control button.active`: background-color, color
- `.segmented-control button:not(:last-child)`: border-right
- `.segmented-control button:hover`: background-color
- `.icon`: font-size, color, cursor

### 15) Combined Inputs
- `.combined-inputs-container`: width, height, padding, font-size, color, border, border-radius, outline, background-color, display, align-items, box-sizing
- `.combined-inputs-input`: border, outline, text-align, font-size, background-color, color
- `.combined-inputs-dropdown`: padding, border, outline, font-size, text-align, color, background-color, appearance

### 16) Flatpickr Styling
- `.flatpickr-months`: display, align-items, justify-content, padding, gap
- `.flatpickr-current-month`: display, align-items, justify-content, gap, line-height
- `.flatpickr-monthDropdown-months`: width, height, padding, text-align, border, border-radius, box-sizing, font-size, line-height
- `.flatpickr-yearDropdown-years`: position, width, height, text-align, border, border-radius, box-sizing, font-size, line-height, display, align-items, justify-content
- `.flatpickr-current-month input.cur-year`: width, height, text-align, font-size, border, box-sizing, -moz-appearance
- `.flatpickr-current-month input.cur-year::-webkit-inner-spin-button`, `.flatpickr-current-month input.cur-year::-webkit-outer-spin-button`: -webkit-appearance, margin
- `.flatpickr-yearDropdown-years::after`: content, position, right, top, font-size, cursor, pointer-events
- `.flatpickr-yearDropdown-years::before`: content, position, right, bottom, font-size, cursor, pointer-events
- `.flatpickr-prev-month, .flatpickr-next-month`: font-size, cursor, padding, line-height
- `.flatpickr-prev-month`: margin-right
- `.flatpickr-next-month`: margin-left
- `.flatpickr-monthDropdown-months, .flatpickr-yearDropdown-years, .flatpickr-current-month input.cur-year`: display, align-items, justify-content, vertical-align
- `.flatpickr-calendar`: margin, padding, border-radius, overflow

### 17) Master-Details
- `.master-details-container`: display, border, border-radius, overflow
- `.master`: background, border-right, overflow-y, display, flex-direction
- `.master-item`: display, align-items, padding, border-bottom, cursor, transition, border-left
- `.master-item:hover`: background-color
- `.master-item.selected`: background-color, border-left, font-weight
- `.master-item img`: width, height, border-radius, margin-right, object-fit, border
- `.master-item-text`: display, flex-direction
- `.master-item-text1`: font-weight, font-size, color
- `.master-item-text2`: font-size, color
- `.details`: padding, background-color, overflow-y
- `.details h2`: margin-top, font-size, color
- `.details p`: font-size, line-height, color
- `.details img`: max-width, border-radius, border
- `@media (max-width: 768px) { .container { flex-direction } .master, .details { width, height } }`

### 18) Master Header
- `.master-header`: display, align-items, justify-content, padding, border-bottom, background-color, position, top, z-index
- `.master-header-search-field`: padding, border, border-radius, font-size

### 19) List Selector Panel
- `.list-container`: background, flex, overflow-y, display, flex-direction, height, box-shadow
- `.list-item`: display, align-items, padding, border-bottom, transition, cursor
- `.list-item:last-child`: border-bottom
- `.list-item:hover`: background, box-shadow
- `.list-item.selected`: background, box-shadow
- `.list-checkbox`: margin-top, margin-right
- `.list-checkbox input`: visibility, position
- `.list-checkbox .checkbox-label`: width, height, border, border-radius, display, position, transition
- `.list-checkbox input:checked + .checkbox-label`: background, border-color
- `.list-checkbox .checkbox-label::after`: content, position, top, left, width, height, border, border-width, transform, transition
- `.list-checkbox input:checked + .checkbox-label::after`: transform
- `.item-details`: flex-grow
- `.item-title`: font-size, font-weight, margin-bottom, color
- `.item-subtitle`: font-size, color
- `.item-description`: font-size, color, margin-top
- `.list-footer`: margin-top, text-align, padding, font-size, background, color, border-top, border-radius
- `.list-footer-highlight`: color, font-weight

### 20) Vitals Grid
- `.vitals_grid`: display, grid-template-columns, gap, justify-content
- `@media (max-width: 1024px) { .vitals_grid { grid-template-columns } }`
- `@media (max-width: 768px) { .vitals_grid { grid-template-columns } }`
- `.vital_card`: background, border-radius, padding, box-shadow, display, justify-content, align-items, transition
- `.vital_card:hover`: box-shadow
- `.vital-input`: background, border, outline, font-size, font-weight, width, text-align, color, transition
- `.vital-label`: font-size, font-weight, color
- `.vital-unit`: font-size, color
- `.vital_card input[type='number'], .vital_card input[type='text']`: border, background, box-shadow, padding, font-size, font-weight, text-align, color, width
- `.vital_card .bp-slash`: font-size, font-weight, color, padding
- `.vital_icon`: font-size, padding, border-radius
- Icon classes: `.height-icon`, `.weight-icon`, `.bmi-icon`, `.temp-icon`, `.bp-icon`, `.pulse-icon`, `.resp-icon`, `.oxygen-icon`, `.glucose-icon` with color + background-color

---

## Theme Creation Procedure
When creating a new Modal Page theme:
1. Choose a unique theme name (Title Case).
2. Copy the canonical Modal Page CSS skeleton exactly.
3. Adjust values in this order:
   - Modal surfaces first (`body`, `.modal`, `.modal-body`, `.modal-footer`)
   - Header color and underline gradient (`.modal-header`, `.modal-header::after`)
   - Footer buttons (`.modal-footer .button.*` + hover)
   - Shared components used inside the modal (forms, tabs, tables, alerts, wizard, list panel, vitals)
4. Ensure:
   - readability is strong
   - the primary scroll area is `.modal-body`
   - the footer remains clear and visible
   - hover and selection states are obvious inside lists and tables

---

## FileMaker Script Branch Template
Add a new branch in the Modal Page CSS generator:

~~~text
Else If [ $$Theme = "New Modal Theme Name" ]
  Set Variable [ $styles ; Value: "PASTE FULL UPDATED MODAL CSS STRING HERE" ]
End If
~~~

---

## AI Request Template
Create a new Modal Page theme using the canonical Modal Page CSS in this document.

Theme Name: <New Theme Name>
Mode: <Light or Dark>
Vibe Keywords: <example: premium, calm, modern, sharp>

Rules:
- Copy the canonical Modal Page CSS exactly.
- Change values only.
- Do not add, remove, or rename selectors.
- Do not add or remove properties.

Deliver:
1. Full updated CSS string
2. Full FileMaker `Else If` branch for the new theme

---

# Splash Screen Page Documentation (CauferoAppStarter)
Version: 2.0  
Objective: Train an AI Agent to generate the Splash Screen HTML page in CauferoAppStarter using FileMaker scripts, consistent variable contracts, and a CSS-first architecture.  
Output: A complete HTML document stored in `$$Splash`, rendered by the Startup script in the Web Viewer object named `Splash`.

---

## 1) Page Identity
- Page Name: Splash Screen
- Output Variable: `$$Splash`
- Rendering Target: Web Viewer object name `Splash`
- Rendering Ownership: Startup script (Startup loads `$$Splash` into the Web Viewer)
- JavaScript: None (Splash contains no JavaScript)

---

## 2) Hard Rules (Non-Negotiable)

### 2.1 Storage and Rendering
- The generator script must write the final HTML document into `$$Splash`.
- The splash Web Viewer content must be `$$Splash` only.
- The generator script is called by Startup.
- The generator script does not rely on JavaScript to navigate or run actions.

### 2.2 CSS Source
- Splash CSS must be obtained by running:
  - `Perform Script [ “🖌️ Use Splash Screen CSS” ]`
  - `Set Variable [ $styles ; Get ( ScriptResult ) ]`
- The CSS script must return its CSS as Script Result (Exit Script with Text Result).

### 2.3 PNG Handling
- Splash background photo is always a PNG base64 string that is injected as:
  - `data:image/png;base64,` + `$$Splash Background Photo`
- The AI Agent must not assume JPEG or any other mime type for the splash background.

### 2.4 No JavaScript
- Do not generate `$Scripts`.
- Do not add `<script>` tags.
- Do not add click handlers or fmp URLs.

### 2.5 Global Variable Contract
The Splash page depends on specific global variables already set by Startup.  
The generator must treat them as required inputs and must not set them here.

---

## 3) Splash Generator Script (Canonical Behavior)

### 3.1 Script Purpose
Generate and store the splash HTML document in `$$Splash`.  
Startup will render it after generation.

### 3.2 Canonical Step Order
The AI Agent must generate the script in this exact sequence:

1. CSS
2. HTML Fragment (banner block)
3. JavaScript section comment (explicitly none)
4. Full HTML Document (assigned to `$$Splash`)
5. WebDirect Pause
6. Refresh Window

---

## 4) Required Scripts

### 4.1 CSS Script
- `🖌️ Use Splash Screen CSS`  
Returns CSS through Script Result.

### 4.2 WebDirect Script
- `WebDirect Pause`  
Used to stabilize page load.

---

## 5) Required Global Variables (Set By Startup)
These variables must exist before the splash generator runs.

### 5.1 Branding
- `$$App Name`  
Displayed in banner and in main center title.
- `$$App Slogan`  
Displayed in banner and in main center subtitle.

### 5.2 Layout / Context
- `$$Layout Name`  
Displayed in the banner title area.

### 5.3 Splash Visual Assets
- `$$Splash Background Photo`  
Base64 PNG string used for the `<img class='background'>`.

### 5.4 Partnerships
- `$$Partners`  
Optional text. If not empty, the splash shows the partnership line.

### 5.5 Loading Message
- `$$Splash Screen Loading Statement`  
Text displayed under the loading bar.

### 5.6 Footer Metadata
- `$$App Launch Year`  
Used in the copyright.

---

## 6) HTML Fragment ($HTML) Specification
`$HTML` is a banner block used by this page pattern.  
It is defined exactly as follows:

~~~filemaker
Set Variable [ $HTML ; Value:
"
<div class='banner'>
  <!-- Logo Section -->
  <div class='banner-logo'>
    <div>
      <div class='logo-text'>" & $$App Name & "</div>
      <div class='logo-subtitle'>" & $$App Slogan & "</div>
    </div>
  </div>
  <!-- Page Title -->
  <div class='banner-title'>" & $$Layout Name & "</div>
</div>
"
]
~~~

### 6.1 Rules For $HTML
- Keep single quotes inside HTML attributes.
- Inject FileMaker globals by concatenation using `&`.
- Preserve the HTML comment labels (they guide human maintainers and AI training).
- Do not add click handlers.
- Do not add additional sections unless the system defines them.

---

## 7) Full HTML Document ($$Splash) Specification
The full document stored in `$$Splash` is defined exactly as follows:

~~~filemaker
Set Variable [ $$Splash ; Value:
"<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='UTF-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>
    <title>Splash Screen</title>
    <link href='https://fonts.googleapis.com/css2?family=Montserrat:wght@600&display=swap' rel='stylesheet'>
    <style> " & $styles & " </style>
</head>
<body>

    <!-- Background image -->
    <img class='background' src='data:image/png;base64," & $$Splash Background Photo & "' alt='Background'>

    <!-- Overlay -->
    <div class='overlay'></div>

    <!-- Gradient overlay -->
    <div class='colour'></div>

    <!-- Content -->
    <div class='container'>
        <div class='title'>" & $$App Name & "</div>
        <div class='subtitle'>" & $$App Slogan & "</div>
        <div class='partners'>
            Developed by <span class='developers'>Caufero Inc.</span><br>
            " & If ( not IsEmpty ( $$Partners ) ; "In partnership with <span class='developers'>" & $$Partners & "</span>" ) & "
        </div>
    </div>
    <div class='loading-bar'>
        <div class='loading-bar-inner'></div>
    </div>
    <div class='loading-message'>" & $$Splash Screen Loading Statement & "</div>
    <div class='footer'>
        &copy; " & $$App Launch Year & " Caufero Inc" & If ( not IsEmpty ( $$Partners ) ; " & " & $$Partners ) & ". All rights reserved.
    </div>

</body>
</html>
"
]
~~~

### 7.1 Rules For $$Splash Assembly
- Must include the Google Fonts Montserrat link exactly as shown.
- Must inject `$styles` inside `<style> ... </style>`.
- Must embed the background image as a base64 PNG using:
  - `src='data:image/png;base64," & $$Splash Background Photo & "'`
- Must include the overlay layers:
  - `.overlay`
  - `.colour`
- Must include:
  - `.container` with title, subtitle, partners
  - `.loading-bar` with `.loading-bar-inner`
  - `.loading-message`
  - `.footer`

### 7.2 Conditional Partnership Rendering
Two separate `$$Partners` conditions exist and must remain exactly as designed:

1) Partnership line in the content:
~~~filemaker
If ( not IsEmpty ( $$Partners ) ;
  "In partnership with <span class='developers'>" & $$Partners & "</span>"
)
~~~

2) Partnership append in the footer:
~~~filemaker
If ( not IsEmpty ( $$Partners ) ;
  " & " & $$Partners
)
~~~

The AI Agent must not “simplify” these unless instructed, because they express the precise intended rendering.

---

## 8) CSS Script Requirements (🖌️ Use Splash Screen CSS)

### 8.1 CSS Contract
The CSS script must:
- Produce a single CSS string
- Exit with `Text Result` so the caller can read it using `Get ( ScriptResult )`

### 8.2 CSS Must Style These Classes
The CSS returned by the script must define styling for:

Banner:
- `.banner`
- `.banner-logo`
- `.logo-text`
- `.logo-subtitle`
- `.banner-title`

Background layers:
- `.background`
- `.overlay`
- `.colour`

Main content:
- `.container`
- `.title`
- `.subtitle`
- `.partners`
- `.developers`

Loading:
- `.loading-bar`
- `.loading-bar-inner`
- `.loading-message`

Footer:
- `.footer`

### 8.3 Font Consistency
Because the `<head>` links Montserrat, CSS should set:
- `font-family: 'Montserrat', ...`

If external fonts later become disallowed, change must be made in the shared template or by replacing the `<link>` in `$$Splash`. The AI Agent must not change it independently.

---

## 9) Generator Script Full Implementation (Training Reference)
This is the recommended full script layout for training the AI Agent.  
Keep section labels and sequence exactly.

~~~filemaker
# ******************************************
# ******************************************
# GENERATE PAGE (START)
# ******************************************
# ******************************************
# 
# CSS
Perform Script [ Specified: From list ; “🖌️ Use Splash Screen CSS” ; Parameter:    ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]
# 
# 
# HTML
Set Variable [ $HTML ; Value:
"
<div class='banner'>
  <!-- Logo Section -->
  <div class='banner-logo'>
    <div>
      <div class='logo-text'>" & $$App Name & "</div>
      <div class='logo-subtitle'>" & $$App Slogan & "</div>
    </div>
  </div>
  <!-- Page Title -->
  <div class='banner-title'>" & $$Layout Name & "</div>
</div>
"
]
# 
# 
# Javascript (No JavaScript on this page)
# 
# 
# Full Page
Set Variable [ $$Splash ; Value:
"<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='UTF-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>
    <title>Splash Screen</title>
    <link href='https://fonts.googleapis.com/css2?family=Montserrat:wght@600&display=swap' rel='stylesheet'>
    <style> " & $styles & " </style>
</head>
<body>

    <!-- Background image -->
    <img class='background' src='data:image/png;base64," & $$Splash Background Photo & "' alt='Background'>

    <!-- Overlay -->
    <div class='overlay'></div>

    <!-- Gradient overlay -->
    <div class='colour'></div>

    <!-- Content -->
    <div class='container'>
        <div class='title'>" & $$App Name & "</div>
        <div class='subtitle'>" & $$App Slogan & "</div>
        <div class='partners'>
            Developed by <span class='developers'>Caufero Inc.</span><br>
            " & If ( not IsEmpty ( $$Partners ) ; "In partnership with <span class='developers'>" & $$Partners & "</span>" ) & "
        </div>
    </div>
    <div class='loading-bar'>
        <div class='loading-bar-inner'></div>
    </div>
    <div class='loading-message'>" & $$Splash Screen Loading Statement & "</div>
    <div class='footer'>
        &copy; " & $$App Launch Year & " Caufero Inc" & If ( not IsEmpty ( $$Partners ) ; " & " & $$Partners ) & ". All rights reserved.
    </div>

</body>
</html>
"
]
# 
# 
# ******************************************
# ******************************************
# GENERATE PAGE (END)
# ******************************************
# ******************************************
# 
# 
Perform Script [ Specified: From list ; “WebDirect Pause” ; Parameter:    ]
Refresh Window [ Flush cached join results ; Flush cached external data ]
~~~

---

## 10) Startup Script Integration Rules (What Startup Must Guarantee)
Startup must set all global variables first, then call splash generation, then render.

### 10.1 Required Startup Order
1. Set:
   - `$$App Name`
   - `$$App Slogan`
   - `$$Layout Name`
   - `$$Splash Background Photo`
   - `$$Splash Screen Loading Statement`
   - `$$App Launch Year`
   - `$$Partners` (optional)

2. Run the generator script.

3. Render in Web Viewer `Splash` using `$$Splash`.

If Startup renders directly, the canonical rendering step is:

~~~filemaker
Set Web Viewer [
  Object Name: "Splash" ;
  Action: "Set HTML" ;
  HTML: $$Splash
]
~~~

---

## 11) QA Checklist (AI Agent Must Validate)

### 11.1 Data Presence Checks
- `$$Splash Background Photo` is not empty (else background will be broken image).
- `$$App Launch Year` is not empty (else footer may show blank year).
- `$$Splash Screen Loading Statement` is not empty (else loading message shows blank).

### 11.2 Render Checks
- `$$Splash` contains `<!DOCTYPE html>` and closes `</html>`.
- `<style>` contains non-empty `$styles`.
- Background image loads visually.
- Overlay and gradient layers render above background.
- Title and subtitle render on top of overlays.
- Loading bar appears.
- Footer appears.

### 11.3 Conditional Checks For Partners
- If `$$Partners` is empty:
  - The partnership line inside `.partners` must not render extra text.
  - The footer must not append `" & "`.
- If `$$Partners` has text:
  - Both partnership locations render correctly.

---

## 12) Known Constraints (Do Not Break)
- No JavaScript on splash.
- Must remain fast: no heavy libraries.
- Background must remain base64 PNG.
- The overlay stack order must remain:
  1) `.background`
  2) `.overlay`
  3) `.colour`
  4) `.container` + loading + footer

The CSS must enforce that stacking order (usually via positioning and z-index).


---

# Banner Page Documentation (CauferoAppStarter)
Version: 1.0  
Purpose: Train an AI Agent to generate the Banner page used in CauferoAppStarter (HTML + CSS + JavaScript bridge + FileMaker scripts).  
Scope: Banner page only. This page is rendered inside a Web Viewer and communicates with FileMaker via `FileMaker.PerformScript()`.

---

## 1) What the Banner Page Is
The Banner page is a reusable HTML component that sits at the top of layouts.

It typically contains:
- A left section (branding or app identity).
- A center section (layout title or context title).
- A right section (icon buttons that trigger FileMaker scripts).
- An optional marquee at the bottom of the banner (only shown for specific dev conditions).

The Banner page is generated fully in FileMaker as a single HTML document string and stored in a global variable (commonly `$$Banner`). The calling layout then loads `$$Banner` into a Web Viewer.

---

## 2) Design Goals
The AI Agent must generate Banner code that is:
1. Deterministic: same inputs produce same HTML output.
2. Scriptable: every interactive element calls a named FileMaker script.
3. Maintainable: CSS is centralized and reusable.
4. Safe: no script name injection, no broken JS, no broken HTML.
5. Responsive: works on FileMaker Pro and FileMaker Go (iPhone and iPad).

---

## 3) Dependencies and Assumptions
### 3.1 FileMaker Environment
- Banner renders in a Web Viewer object.
- JavaScript must be enabled in Web Viewer.
- `FileMaker.PerformScript(scriptName, parameter)` must be available inside the Web Viewer context.

### 3.2 Required Scripts (FileMaker)
The Banner generator script calls:
- `🖌️ Use Banner CSS` to fetch base banner CSS.
- `Get Alerts` to update alerts state before rendering.

The Banner must also be able to call these scripts via buttons:
- `Go Back To App`
- `Go To Claris`
- `Go To Claris Partners`
- `Go To Mail`
- `Go To Google` (if used)
- `Go To Canva`
- `Go To Globe`
- `Go To YT`
- `Go To IG`
- `Go To Fb`
- `Go To LI`
- `Go To WA`
- `Go To NF`
- `Go To Music`
- `Click On Alerts Icon`
- `Show Notes`
- `Go To Notes`
- `Show More`
- `Go To Menu`

If any of these scripts do not exist in the file, the UI action will fail. The AI Agent must treat this as a strict dependency.

### 3.3 Required Globals (FileMaker)
The sample code references:
- `$$Currently Playing Song Title`

The banner may also rely on additional globals from your system (branding, current layout name, etc). If so, those must be declared explicitly in this document and set before rendering.

---

## 4) Inputs, Outputs, and State
### 4.1 Inputs (what the generator uses)
The generator script uses:
- Device context: `Get ( Device )`
- Account context: `Get ( UserName )`
- File context: `Get ( FileName )`
- Optional text content: `$$Currently Playing Song Title`
- Script names to call when icons are clicked

### 4.2 Output
The generator sets:
- `$$Banner` = Full HTML document (includes CSS and JavaScript).

### 4.3 Side Effects
- Runs `Get Alerts` (likely sets alert-related globals or fields).
- No records should be created or modified by the Banner generator itself, unless your alert logic requires it.

---

## 5) Banner Generation Pipeline (Exact Build Order)
The AI Agent must follow this order. Do not shuffle it.

1. Resolve username logic (dev-only logic can alter what renders).
2. Set FileMaker script name variables (strings).
3. Refresh alerts state by calling `Get Alerts`.
4. Fetch base CSS from `🖌️ Use Banner CSS`.
5. Append additional CSS overrides that belong to the banner instance (example: marquee CSS).
6. Build the banner HTML fragment (`$HTML`).
7. Build JavaScript bridge functions. Each function calls a FileMaker script.
8. Concatenate everything into a full HTML document and store in `$$Banner`.

---

## 6) Canonical Script Skeleton (FileMaker Pseudocode)
This is the minimum structure the AI Agent must produce when building the Banner generator script.

~~~filemaker
# ------------------------------------------------------------
# Banner Generator (Canonical Structure)
# ------------------------------------------------------------

# 1) Context
Set Variable [ $The Username ; Value: If ( Get ( Device ) = 4 ; "iPhone" ; "Cyril Amegah" ) ]
Set Variable [ $Click On Alerts Icon Script ; Value: "Click On Alerts Icon" ]
Set Variable [ $Open Show Notes FM Script ; Value: "Show Notes" ]
Set Variable [ $Open Show More FM Script ; Value: "Show More" ]

# 2) Refresh state (alerts)
Perform Script [ Specified: From list ; “Get Alerts” ; Parameter: "" ]

# 3) CSS (central + appended overrides)
Perform Script [ Specified: From list ; “🖌️ Use Banner CSS” ; Parameter: "" ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]

# Append banner-specific CSS overrides (example: marquee)
Set Variable [ $styles ; Value:
    $styles &
    " .marquee { ... } " &
    " .marquee .track { ... } "
]

# 4) HTML fragment (banner structure)
Set Variable [ $HTML ; Value:
    "<div class='banner'>" &
        ... &
    "</div>"
]

# 5) JavaScript bridge (FileMaker.PerformScript wrappers)
Set Variable [ $Go Back To App Script ; Value: "function goBackToApp(){ FileMaker.PerformScript('Go Back To App'); }" ]
... more function strings ...

# Combine all JS functions into one <script> block
Set Variable [ $Scripts ; Value: "<script>" & $Go Back To App Script & " " & ... & "</script>" ]

# 6) Full document
Set Variable [ $$Banner ; Value:
    "<!DOCTYPE html>" &
    "<html lang='en'>" &
    "<head>" &
        "<meta charset='UTF-8'>" &
        "<meta name='viewport' content='width=device-width, initial-scale=1.0'>" &
        "<title>Banner</title>" &
        "<link href='https://fonts.googleapis.com/css2?family=Montserrat:wght@600&display=swap' rel='stylesheet'>" &
        "<style>" & $styles & "</style>" &
    "</head>" &
    "<body>" &
        $HTML &
        $Scripts &
    "</body>" &
    "</html>"
]
~~~


---

## 7) Banner HTML Structure (Recommended)
The Banner must use stable class names so CSS can target them reliably.

### 7.1 Required container
- `.banner` is the root of the UI.

### 7.2 Suggested internal layout
Use three zones:
- `.banner-left`
- `.banner-center`
- `.banner-right`

Example skeleton:

~~~html
<div class="banner">
  <div class="banner-left">
    <!-- Brand area -->
  </div>

  <div class="banner-center">
    <!-- Page title or context -->
  </div>

  <div class="banner-right">
    <!-- Icon buttons -->
  </div>

  <!-- Optional: marquee -->
  <div class="marquee">
    <div class="track" data-text="..."></div>
  </div>
</div>
~~~

### 7.3 Buttons and click bindings
Each clickable icon must map to a JavaScript function, and that function must call a FileMaker script.

Example:

~~~html
<button class="icon-btn" onclick="goToMail()" title="Mail">
  <span class="icon">✉️</span>
</button>
~~~

---

## 8) Conditional Marquee Logic (Dev-Only Display)
The sample shows marquee only when:
- `Get ( UserName ) = $The Username`
- `Lower ( Get ( FileName ) ) = "cauferoappstarter"`

This is a dev-only visual that should not appear for other users or other file names.

### 8.1 Canonical condition
The AI Agent must implement it exactly as defined in your system.

~~~filemaker
If ( Get ( UserName ) = $The Username and Lower ( Get ( FileName ) ) = "cauferoappstarter" ;
    /* include marquee HTML */
    "" /* else */
)
~~~

### 8.2 Marquee content
The sample uses:
- A fixed phrase: `I want to do too many apps`
- A music glyph
- `$$Currently Playing Song Title`

Example construction pattern:

~~~filemaker
"<div class='marquee'>" &
  "<div class='track' data-text='I want to do too many apps  [ ♫ " & $$Currently Playing Song Title & " ]'></div>" &
"</div>"
~~~

### 8.3 Why `data-text`
A common marquee pattern duplicates the same text repeatedly via JS or CSS. Using `data-text` allows JS to read it and generate repeated spans if needed.

If your current banner uses a JS-based marquee animation, document the exact algorithm here.

---

## 9) CSS Architecture
### 9.1 Central CSS retrieval
Banner CSS is retrieved via:

~~~filemaker
Perform Script [ Specified: From list ; “🖌️ Use Banner CSS” ; Parameter: "" ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]
~~~

The AI Agent must treat `🖌️ Use Banner CSS` as the source of truth for core banner styles.

### 9.2 Local overrides
After fetching core CSS, append instance-specific styles.

The sample appends marquee CSS:

~~~filemaker
Set Variable [ $styles ; Value:
  $styles &
  " .marquee { position:absolute; left:24px; right:24px; bottom:24px; height:28px; display:flex; align-items:center; overflow:hidden; pointer-events:none; background:transparent; } "
]
~~~

### 9.3 Recommended CSS rules
Minimum recommended rule categories:
1. Layout: flex rows, spacing, alignment
2. Typography: font family, weight, size
3. Buttons: hit area, hover states, active states
4. Dark mode or theme tokens (if your app uses them)
5. Responsive scaling for iPhone and iPad
6. Z-index layering and safe padding

---

## 10) JavaScript Bridge (FileMaker.PerformScript Wrappers)
### 10.1 Rule
Do not call `FileMaker.PerformScript()` inline inside HTML. Wrap it in named functions. This keeps HTML clean and makes debugging easier.

### 10.2 Canonical function pattern
Each function must:
- Be named clearly
- Call a single FileMaker script
- Pass parameters only if needed

Example:

~~~js
function goToMail() {
  FileMaker.PerformScript('Go To Mail');
}
~~~

### 10.3 Script name variables (when required)
The sample sometimes stores script names in FileMaker variables and injects them into JS.

Example:

~~~filemaker
Set Variable [ $Click On Alerts Icon Script ; Value: "Click On Alerts Icon" ]
Set Variable [ $Click On Alerts Icon Script ; Value:
  "function clickOnAlertsIcon(){ FileMaker.PerformScript('" & $Click On Alerts Icon Script & "'); }"
]
~~~

This pattern reuses the same variable name for two different meanings:
1. Script name
2. Function source code string

This is risky. The AI Agent should prefer separate variables:

- `$FM_ClickOnAlerts_ScriptName`
- `$JS_ClickOnAlerts_Function`

Example improved pattern:

~~~filemaker
Set Variable [ $FM_ClickOnAlerts_ScriptName ; Value: "Click On Alerts Icon" ]
Set Variable [ $JS_ClickOnAlerts_Function ; Value:
  "function clickOnAlertsIcon(){ FileMaker.PerformScript('" & $FM_ClickOnAlerts_ScriptName & "'); }"
]
~~~

If you want the AI Agent to preserve your current style, say so explicitly. Otherwise, adopt the safer naming convention.

### 10.4 JS concatenation strategy
The sample concatenates function strings separated by `¶¶`.

The AI Agent can use:
- literal spaces
- line breaks
- `Char ( 10 )` for newlines

Example:

~~~filemaker
Set Variable [ $Scripts ; Value:
  "<script>" &
  $JS_GoBackToApp_Function & Char ( 10 ) &
  $JS_GoToMail_Function & Char ( 10 ) &
  "</script>"
]
~~~

---

## 11) Full HTML Document Assembly
The AI Agent must generate a complete HTML document, not a fragment.

Minimum required head elements:
- `meta charset`
- `meta viewport`
- `title`
- font include (Montserrat 600 as shown)
- `<style>` block with `$styles`

Minimum required body elements:
- `$HTML` banner fragment
- `$Scripts` script block

Example assembly (pattern):

~~~filemaker
Set Variable [ $$Banner ; Value:
  "<!DOCTYPE html>" &
  "<html lang='en'>" &
  "<head>" &
    "<meta charset='UTF-8'>" &
    "<meta name='viewport' content='width=device-width, initial-scale=1.0'>" &
    "<title>Banner</title>" &
    "<link href='https://fonts.googleapis.com/css2?family=Montserrat:wght@600&display=swap' rel='stylesheet'>" &
    "<style>" & $styles & "</style>" &
  "</head>" &
  "<body>" &
    $HTML &
    $Scripts &
  "</body>" &
  "</html>"
]
~~~

---

## 12) Web Viewer Integration (How the Banner is Loaded)
Your layout must have a Web Viewer object dedicated to the banner.

Recommended object name:
- `wvBanner`

Common load options:
1. Set Web Viewer to `$$Banner` (HTML)
2. Set Web Viewer to a calculation that returns `$$Banner`

Example:

~~~filemaker
Set Web Viewer [ Object Name: "wvBanner" ; Action: "Set HTML" ; HTML: $$Banner ]
~~~

If your system uses a different mechanism, document it here explicitly.

---

## 13) Test Plan (AI Agent Must Validate These)
The AI Agent should verify:

### 13.1 Rendering
- Banner loads without blank screen.
- No broken HTML (missing closing tags).
- CSS applies (fonts, spacing, alignment).

### 13.2 Script bindings
Click each icon and confirm the FileMaker script runs:
- Mail icon triggers `Go To Mail`
- Menu icon triggers `Go To Menu`
- Alerts icon triggers `Click On Alerts Icon`
- Notes icon triggers `Show Notes` or `Go To Notes`
- Show More triggers `Show More`

### 13.3 Dev-only marquee
- Marquee appears only when condition is true.
- Marquee does not block clicks (`pointer-events:none` should ensure that).
- Marquee text includes `$$Currently Playing Song Title` when available.

### 13.4 Device checks
- iPhone sizing looks correct.
- iPad sizing looks correct.
- If the UI scales, confirm it does not overflow.

---

## 14) Troubleshooting Guide
### 14.1 Banner is blank
Likely causes:
- Web Viewer did not load `$$Banner`
- HTML string is malformed
- Quotes were not escaped correctly
- CSS script returned empty result

Actions:
- Temporarily set Web Viewer HTML to a simple `<div>Test</div>`
- Show `Length ( $$Banner )` in a field to confirm content exists
- Log `$styles` length and preview it

### 14.2 Clicking buttons does nothing
Likely causes:
- Web Viewer context does not support `FileMaker.PerformScript`
- Function name mismatch in `onclick`
- Script name mismatch in FileMaker

Actions:
- Add a JS test: `alert('clicked')` to confirm click fires
- Confirm script exists and runs from Script Workspace
- Confirm spelling and casing

### 14.3 Fonts do not load
Likely causes:
- No internet access to Google Fonts
- Network restrictions

Actions:
- Bundle fonts locally if required, or choose a fallback stack.

---

## 15) Implementation Rules for the AI Agent
When the AI Agent is asked to build or modify the Banner generator, it must:
1. Identify all scripts the banner calls.
2. Confirm which are called from FileMaker (pre-render) vs from Web Viewer (on click).
3. Keep CSS centralized and append only banner-specific overrides.
4. Generate full HTML document and store it in `$$Banner`.
5. Avoid variable name reuse that changes meaning mid-script, unless your standard requires it.
6. Keep conditions explicit, especially dev-only UI.

---

## 16) Ambiguities, Contradictions, and Clarifications Needed
These items must be clarified to make the documentation fully deterministic for the AI Agent.

### 16.1 `Get ( Device ) = 4` meaning
- You used: `If ( Get ( Device ) = 4 ; "iPhone" ; "Cyril Amegah" )`
Questions:
- What does value `4` represent in your environment?
- Is it always iPhone, or could it represent FileMaker Go generally?

### 16.2 Username logic looks inverted or unclear
- `$The Username` becomes `"iPhone"` on one branch and `"Cyril Amegah"` on the other.
Questions:
- Do you really want `Get ( UserName ) = "iPhone"` to ever be true?
- Is `"iPhone"` intended to match `Get ( UserName )` on FileMaker Go devices in your setup?
- If this is a dev hack, what is the exact goal?

### 16.3 `$Go To Google Script` is referenced but not defined in the sample
In your `$Scripts` concatenation you include `$Go To Google Script`, but the sample does not show it being set.
Questions:
- Is there a `Go To Google` script and JS function?
- If yes, provide the exact JS wrapper string you expect.

### 16.4 `Set Variable []` appears as an empty step
The sample shows `Set Variable []` after `Perform Script [ “Get Alerts” ]`.
Questions:
- Is this a copy artifact, or is there a real variable that should be set here?
- If real, what variable and what value?

### 16.5 Marquee CSS string is truncated
The sample shows the appended CSS cut off:
- `"background: transparen…"`
Questions:
- Provide the full marquee CSS you want appended.
- Confirm if marquee animation is CSS-only or JS-assisted.

### 16.6 HTML fragment is truncated
The sample `$HTML` value is cut off mid-string:
- `"<div class='marquee'>   <div class='track' data-text='I want to do too many apps  [ ♫ " & $$Currently Playing Song Title & " …" ]`
Questions:
- Provide the full intended banner HTML structure.
- List all icons and their exact class names and click functions.

### 16.7 Script name variables are reused for function strings
Example:
- `$Click On Alerts Icon Script` is first a script name, then becomes function source code.
Questions:
- Do you want to keep this pattern as your standard?
- Or do you want separate variables for script names vs JS function strings?

### 16.8 Where does layout title come from
The banner usually shows a title (layout name or section name). The sample does not show how that title is inserted.
Questions:
- Do you want to use `$$Layout Name`, `Get ( LayoutName )`, or something else?
- What is the exact field or global variable?

### 16.9 Alerts UI state
You run `Get Alerts` before rendering.
Questions:
- What does `Get Alerts` output or set that the banner needs?
- Does the banner show an unread count badge?
- If yes, what variable or field contains the count?

---

# CauferoAppStarter AI Training Documentation
## Dashboard Development (WebViewer + ChartJS)

> Purpose  
> Train an AI Agent to reliably generate a complete dashboard page inside CauferoAppStarter using the established pattern: **Theme-driven CSS + HTML blocks + JavaScript blocks + Page assembly**.  
> The agent must output FileMaker script steps (Set Variable, Perform Script, etc.) and the HTML/CSS/JS payloads in a way that matches CauferoAppStarter conventions.

---

## 1) Dashboard Definition in CauferoAppStarter

A **Dashboard** in CauferoAppStarter is a **single HTML page** rendered inside a WebViewer / WebDirect context, composed from:

1. **Theme-driven CSS**  
2. **HTML blocks** (cards, charts, tables, timelines, etc.)  
3. **JavaScript blocks** (ChartJS initializations + small UI animations)  
4. **External JS preload** (ChartJS library or any dashboard JS library stored in a table)  
5. **Final Page HTML** stored in `$$Page`

The whole dashboard is generated by a FileMaker script that:
- sets `$$App Section = "Dashboard"`
- pulls reusable JS payload from a table via ExecuteSQL (example: `ChartJS::JS`)
- builds CSS variables based on `$$Theme`
- builds multiple HTML blocks into `$HTML`
- builds multiple JS blocks into `$Scripts`
- assembles `$$Page` as a full HTML document and refreshes the window

---

## 2) Hard Requirements (AI Must Follow)

### 2.1 Use the Standard Variable Names

The AI must use these variable scopes as a baseline:

- `$$App Section`  
- `$$Theme`  
- `$Dashboard JS`  
- `$Styles`  
- `$HTML`  
- `$Scripts`  
- `$$Page`  

If additional variables are needed, keep them consistent and descriptive:
- `$Pie Chart HTML`, `$Pie Chart JS`, `$KPI Card Colours`, etc.

### 2.2 Use the Standard Build Order

The AI must generate in this exact order:

1. **Context**
2. **External JS preload** (ExecuteSQL or other retrieval)
3. **Theme variables (colors, sizes, chart settings)**
4. **CSS (including KPI nth-child mapping)**
5. **Call shared Dashboard CSS script**
6. **HTML blocks**
7. **JS blocks**
8. **Assemble `$$Page`**
9. **Pause / refresh for WebDirect**

### 2.3 Keep Chart IDs and JS Selectors Perfectly Matched

If the HTML defines:
- `<canvas id='pieChart'></canvas>`

Then the JS must reference:
- `document.getElementById('pieChart')`

Zero mismatches allowed.

### 2.4 Use Self-Invoking JS Wrappers Per Chart

Dashboard JS blocks must use a safe scope, example:

~~~js
(function () {
  const ctx = document.getElementById('pieChart').getContext('2d');
  new Chart(ctx, {...});
})();
~~~

This avoids variable collisions when multiple charts exist.

### 2.5 Theme Must Drive Colors Everywhere

The AI must generate theme-mapped values for:
- KPI card backgrounds
- Pie chart colors
- Line graph dataset colors
- Axis colors
- Grid colors
- Bar chart colors
- Stacked bars colors
- Gauge colors
- Progress circle stroke

If the theme is unknown, provide a default palette.

---

## 3) Data Sources and Retrieval Pattern

### 3.1 External ChartJS / Dashboard JS Retrieval via ExecuteSQL

Dashboard JS libraries or shared JS can be stored in a table (example: `ChartJS`) and loaded like:

~~~filemaker
Set Variable [ $Dashboard JS ; 
  Value: Let ( 
    [
      sql = "Select JS From ChartJS" ;
      fieldseparator = "xxxxxxxxxx" ;
      rowseparator = "yyyyyyyyyy" ;
      result = ExecuteSQL ( sql ; fieldseparator ; rowseparator )
    ] ;
    result
  )
]
~~~

#### Notes for the AI
- `fieldseparator` and `rowseparator` are hard-coded tokens used to avoid collisions in retrieved text.
- If multiple rows are returned, the agent must decide whether:
  - to use the first row only, or
  - to concatenate rows, or
  - to filter by name/version.
- If the table is empty, the agent must still build the dashboard page, but charts may not render unless ChartJS is otherwise included.

### 3.2 Dashboard Data Strategy

Dashboards can use one of these data strategies:

1. **Static demo data** (as in the sample code)  
2. **FileMaker data injected into JS** using:
   - `JSONSetElement()` to create a JSON payload
   - `Substitute()` to inject JSON into HTML/JS template
3. **AJAX bridge** (advanced)  
   - A JavaScript call that triggers FileMaker script via URL scheme or `Perform Script` bridge pattern (implementation depends on platform)

For AI training, default to:
- **Static demo data** for initial dashboard templates
- Provide clear placeholders where real data gets injected

---

## 4) Theme Mapping System

### 4.1 Themes Are Selected via `$$Theme`

The dashboard script reads the global theme:

- `$$Theme = "AquaView"`
- `$$Theme = "Aurora"`
- `$$Theme = "ContrastCard"`
- `$$Theme = "Dokondo"`
- etc.

### 4.2 Theme Variables the AI Must Produce

The AI must generate these variables (even if simplified):

- `$KPI Card Colours` (List)
- `$PieBorderWidth`
- `$PieColours`
- `$Line Graph Colours`
- `$AxisColours _X`
- `$GridColours`
- `$Bar Chart Colours`
- `$Stacked Bars Colours`
- `$Gauge Colours`
- `$Progress Circle Stroke`

#### Example: KPI Card Colors as a List

~~~filemaker
Set Variable [ $KPI Card Colours ;
  Value: Case ( 
    $$Theme = "AquaView" ;
      List ( "#26A69A" ; "#4CAF50" ; "#FF9800" ; "#FF7043" ; "#9C27B0" ; "#00BCD4" ) ;
    $$Theme = "Aurora" ;
      List ( "#EF476F" ; "#06D6A0" ; "#118AB2" ; "#FFD166" ; "#073B4C" ; "#8D99AE" ) ;
    $$Theme = "ContrastCard" ;
      List ( "#4D3A70" ; "#596B8C" ; "#1F4D79" ; "#EB5757" ; "#27AE60" ; "#B0BEC5" ) ;
    $$Theme = "Dokondo" ;
      List ( "#D51E26" ; "#0E3344" ; "#5FA4A1" ; "#41C5E0" ; "#D8743D" ; "#D5432E" ) ;
    List ( "#4D3A70" ; "#596B8C" ; "#1F4D79" ; "#EB5757" ; "#27AE60" ; "#B0BEC5" )
  )
]
~~~

### 4.3 KPI nth-child Mapping Must Be Generated

After choosing the palette, generate a CSS string:

~~~filemaker
Set Variable [ $$KPI Card Colours ;
  Value:
    ".kpi-card:nth-child(1) { background: " & GetValue ( $KPI Card Colours ; 1 ) & "; }" & ¶ &
    ".kpi-card:nth-child(2) { background: " & GetValue ( $KPI Card Colours ; 2 ) & "; }" & ¶ &
    ".kpi-card:nth-child(3) { background: " & GetValue ( $KPI Card Colours ; 3 ) & "; }" & ¶ &
    ".kpi-card:nth-child(4) { background: " & GetValue ( $KPI Card Colours ; 4 ) & "; }" & ¶ &
    ".kpi-card:nth-child(5) { background: " & GetValue ( $KPI Card Colours ; 5 ) & "; }" & ¶ &
    ".kpi-card:nth-child(6) { background: " & GetValue ( $KPI Card Colours ; 6 ) & "; }"
]
~~~

This `$$KPI Card Colours` string must be used inside the final CSS.

---

## 5) CSS Generation Pattern

### 5.1 Use the Shared Dashboard CSS Script

The dashboard must call the shared style generator:

~~~filemaker
Perform Script [ Specified: From list ; “🖌️ Use Dashboard CSS” ; Parameter: "" ]
Set Variable [ $Styles ; Value: Get ( ScriptResult ) ]
Set Variable [ $Styles ; Value: "<style> " & $Styles & " </style>" ]
~~~

### 5.2 Inject Dynamic Theme CSS into Shared CSS

The shared CSS script must support injection points or you must append dynamic CSS after it.

Minimum dynamic CSS to append:
- `$$KPI Card Colours`
- chart container sizing overrides (if needed)
- progress circle stroke color
- grid/axis colors may be purely JS-driven

If the shared CSS does not support injection, the AI must do:

~~~filemaker
Set Variable [ $Styles ;
  Value: "<style> " & Get ( ScriptResult ) & ¶ & $$KPI Card Colours & " </style>"
]
~~~

---

## 6) HTML Block System

### 6.1 Dashboard HTML Is Built as Multiple Blocks

Each dashboard component is generated as its own variable:

- `$Cards HTML`
- `$Pie Chart HTML`
- `$Line Graph HTML`
- `$Bar Graph HTML`
- `$DataTable HTML`
- `$Timeline HTML`
- etc.

Then all are concatenated into one `$HTML`:

~~~filemaker
Set Variable [ $HTML ;
  Value:
    $Cards HTML & ¶ &
    $Pie Chart HTML & ¶ &
    $Pie Chart 2 HTML & ¶ &
    $Doughnut Chart HTML & ¶ &
    $Line Graph HTML & ¶ &
    $Bar Graph HTML & ¶ &
    $Timeline HTML
]
~~~

### 6.2 Positioning Strategy

The sample uses **absolute positioning with percentage layout**:

- top: `1.5%`, `22.5%`, `69%`, etc.
- left: `0.5%`, `21%`, `62%`, etc.
- width: `20%`, `30.2%`, `37.2%`, `49%`, etc.

The AI must preserve this style for consistency unless instructed to switch to CSS Grid/Flex.

### 6.3 KPI Cards Block Requirements

- KPI cards are links (`<a href='#...'>`)
- They carry class `kpi-card`
- They contain icon (often SVG) + value + label
- They rely on `$$KPI Card Colours` nth-child mapping

Minimum required structure:

~~~html
<div class='cards' style='position:absolute; top:1.5%; left:0.5%; gap:0.5%; width:99%; height:20%;'>
  <a href='#kpi-1' class='kpi-card'>
    <div class='kpi-title'>Total Products</div>
    <div class='kpi-value' id='kpiTotalProducts'>1,245</div>
  </a>
</div>
~~~

### 6.4 Chart Card Block Requirements

Every chart must be inside a `.chart-card` container and include:

- `<h2>Title</h2>`
- `.chart-container`
- `<canvas id='...'></canvas>`

Example:

~~~html
<div class="chart-card" style="position:absolute; top:22.5%; left:0.5%; width:20%; height:45%">
  <h2>Customer Satisfaction</h2>
  <div class="chart-container" style="width:100%; height:70%;">
    <canvas id="pieChart"></canvas>
  </div>
</div>
~~~

### 6.5 Non-Chart Visuals (Progress, Counter, Timeline)

These are still treated as HTML blocks and must have matching JS:

- Progress Circle: `<canvas id='progressCircle'>`
- Counter: `<div class='counter' id='counter'>0</div>`
- Progress Bar: `<div class='progress-bar' id='progressBar'>`
- Timeline Fill: `<div id='timelineFill'>` (or similar)

---

## 7) JavaScript Block System

### 7.1 Dashboard JS Is Built as Multiple Blocks

Each component’s JS is built into its own variable:

- `$Pie Chart JS`
- `$Line Graph JS`
- `$Progress Circle JS`
- `$Counter JS`
- etc.

Then concatenated into `$Scripts`:

~~~filemaker
Set Variable [ $Scripts ;
  Value:
    "<script>" & ¶ &
    $Pie Chart JS & ¶ &
    $Line Graph JS & ¶ &
    $Progress Circle JS & ¶ &
    $Counter JS & ¶ &
    "</script>"
]
~~~

### 7.2 ChartJS Must Be Available Before Any Chart Code Runs

The final page must load ChartJS before `$Scripts`.

Your pattern options:

1. Inline `<script>` injection from `$Dashboard JS` in `<head>`
2. External CDN `<script src='...'>` in `<head>` (if allowed)
3. A stored JS blob in FileMaker (as shown)

The sample uses:

~~~filemaker
"<script>" & $Dashboard JS & "</script>"
~~~

This implies `$Dashboard JS` contains ChartJS library code or loader code.

### 7.3 Theme-Driven Colors in JS

The AI must use the theme variables in JS strings, especially:

- pie colors (`$PieColours`)
- border widths (`$PieBorderWidth`)
- line dataset colors (`$Line Graph Colours`)
- axis colors (`$AxisColours _X`)
- grid colors (`$GridColours`)
- bar colors (`$Bar Chart Colours`)
- stacked colors (`$Stacked Bars Colours`)
- gauge colors (`$Gauge Colours`)

Example snippet shape (string-building idea):

~~~js
options: {
  scales: {
    x: { ticks: { color: 'AXIS_X_COLOR' }, grid: { color: 'GRID_COLOR' } },
    y: { ticks: { color: 'AXIS_X_COLOR' }, grid: { color: 'GRID_COLOR' } }
  }
}
~~~

Then replace placeholders in FileMaker, or concatenate values directly into the JS string.

### 7.4 Defensive JS: Wait for DOM Ready (Optional but Recommended)

If WebDirect timing issues occur, wrap chart creation inside:

~~~js
(function () {
  const run = () => { /* create charts */ };
  if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", run);
  } else {
    run();
  }
})();
~~~

The sample uses immediate execution. The AI may use the DOM-ready pattern if dashboards fail to render in some contexts.

---

## 8) Final Page Assembly Pattern

### 8.1 `$$Page` Must Be Full HTML Document

The final page must include:

- `<!DOCTYPE html>`
- `<html lang="en">`
- `<head>`
  - charset
  - viewport
  - title
  - ChartJS preload script
  - `$Styles`
- `</head>`
- `<body>`
  - `$HTML`
  - `$Scripts` (or scripts appended after HTML)
- `</body>`
- `</html>`

### 8.2 Example Assembly (Pattern)

~~~filemaker
Set Variable [ $$Page ;
  Value:
    "<!DOCTYPE html>" & 
    "<html lang='en'>" &
    "<head>" &
      "<meta charset='UTF-8'>" &
      "<meta name='viewport' content='width=device-width, initial-scale=1.0'>" &
      "<title>Dashboard</title>" &
      "<script>" & $Dashboard JS & "</script>" &
      $Styles &
    "</head>" &
    "<body>" &
      $HTML &
      $Scripts &
    "</body>" &
    "</html>"
]
~~~

### 8.3 WebDirect Stabilization

After setting `$$Page`, the script should:

~~~filemaker
Perform Script [ Specified: From list ; “WebDirect Pause” ; Parameter: "" ]
Refresh Window [ Flush cached join results ; Flush cached external data ]
~~~

---

## 9) Full Script Blueprint (AI Output Template)

> The AI Agent should generate a dashboard script using this skeleton, filling in the theme maps, HTML blocks, and JS blocks.

~~~filemaker
Set Variable [ $$App Section ; Value: "Dashboard" ]

/* External JS preload */
Set Variable [ $Dashboard JS ;
  Value: Let (
    [
      sql = "Select JS From ChartJS" ;
      fieldseparator = "xxxxxxxxxx" ;
      rowseparator = "yyyyyyyyyy" ;
      result = ExecuteSQL ( sql ; fieldseparator ; rowseparator )
    ] ;
    result
  )
]

/* Theme variables */
Set Variable [ $KPI Card Colours ; Value: Case ( /* theme mapping */ ) ]
Set Variable [ $PieBorderWidth ; Value: If ( /* theme conditions */ ; 2 ; 1 ) ]
Set Variable [ $PieColours ; Value: Case ( /* theme mapping */ ) ]
Set Variable [ $Line Graph Colours ; Value: Case ( /* theme mapping */ ) ]
Set Variable [ $AxisColours _X ; Value: Case ( /* theme mapping */ ) ]
Set Variable [ $GridColours ; Value: Case ( /* theme mapping */ ) ]
Set Variable [ $Bar Chart Colours ; Value: Case ( /* theme mapping */ ) ]
Set Variable [ $Stacked Bars Colours ; Value: Case ( /* theme mapping */ ) ]
Set Variable [ $Gauge Colours ; Value: Case ( /* theme mapping */ ) ]
Set Variable [ $Progress Circle Stroke ; Value: Case ( /* theme mapping */ ) ]

/* KPI dynamic CSS */
Set Variable [ $$KPI Card Colours ;
  Value:
    ".kpi-card:nth-child(1) { background: " & GetValue ( $KPI Card Colours ; 1 ) & "; }" & ¶ &
    ".kpi-card:nth-child(2) { background: " & GetValue ( $KPI Card Colours ; 2 ) & "; }" & ¶ &
    ".kpi-card:nth-child(3) { background: " & GetValue ( $KPI Card Colours ; 3 ) & "; }" & ¶ &
    ".kpi-card:nth-child(4) { background: " & GetValue ( $KPI Card Colours ; 4 ) & "; }" & ¶ &
    ".kpi-card:nth-child(5) { background: " & GetValue ( $KPI Card Colours ; 5 ) & "; }" & ¶ &
    ".kpi-card:nth-child(6) { background: " & GetValue ( $KPI Card Colours ; 6 ) & "; }"
]

/* Overall CSS */
Perform Script [ Specified: From list ; “🖌️ Use Dashboard CSS” ; Parameter: "" ]
Set Variable [ $Styles ; Value: Get ( ScriptResult ) ]
Set Variable [ $Styles ; Value: "<style>" & $Styles & ¶ & $$KPI Card Colours & "</style>" ]

/* HTML blocks */
Set Variable [ $Cards HTML ; Value: "..." ]
Set Variable [ $Pie Chart HTML ; Value: "..." ]
Set Variable [ $Line Graph HTML ; Value: "..." ]
Set Variable [ $DataTable HTML ; Value: "..." ]
Set Variable [ $Timeline HTML ; Value: "..." ]

Set Variable [ $HTML ;
  Value: $Cards HTML & ¶ & $Pie Chart HTML & ¶ & $Line Graph HTML & ¶ & $DataTable HTML & ¶ & $Timeline HTML
]

/* JS blocks */
Set Variable [ $Pie Chart JS ; Value: "..." ]
Set Variable [ $Line Graph JS ; Value: "..." ]
Set Variable [ $Progress Circle JS ; Value: "..." ]
Set Variable [ $Counter JS ; Value: "..." ]
Set Variable [ $Timeline JS ; Value: "..." ]

Set Variable [ $Scripts ;
  Value: "<script>" & ¶ &
    $Pie Chart JS & ¶ &
    $Line Graph JS & ¶ &
    $Progress Circle JS & ¶ &
    $Counter JS & ¶ &
    $Timeline JS & ¶ &
  "</script>"
]

/* Full page */
Set Variable [ $$Page ;
  Value:
    "<!DOCTYPE html>" &
    "<html lang='en'>" &
    "<head>" &
      "<meta charset='UTF-8'>" &
      "<meta name='viewport' content='width=device-width, initial-scale=1.0'>" &
      "<title>Dashboard</title>" &
      "<script>" & $Dashboard JS & "</script>" &
      $Styles &
    "</head>" &
    "<body>" &
      $HTML &
      $Scripts &
    "</body>" &
    "</html>"
]

Perform Script [ Specified: From list ; “WebDirect Pause” ; Parameter: "" ]
Refresh Window [ Flush cached join results ; Flush cached external data ]
~~~

---

## 10) Component Recipes (AI Must Be Able to Generate)

This section teaches the AI how to generate each dashboard component in a repeatable way.

### 10.1 KPI Cards Recipe

**HTML requirements**
- `div.cards`
- children: `a.kpi-card`
- each card contains:
  - icon (svg optional)
  - `.kpi-title`
  - `.kpi-value` with `id`

**JS requirements**
- optional: animate numbers or update from data

**Example HTML (string form shape)**
~~~html
<div class='cards' style='position:absolute; top:1.5%; left:0.5%; gap:0.5%; width:99%; height:20%;'>
  <a href='#total-products' class='kpi-card'>
    <div class='kpi-title'>Total Products</div>
    <div class='kpi-value' id='kpiTotalProducts'>1245</div>
  </a>
  <a href='#total-customers' class='kpi-card'>
    <div class='kpi-title'>Total Customers</div>
    <div class='kpi-value' id='kpiTotalCustomers'>320</div>
  </a>
</div>
~~~

### 10.2 Pie Chart Recipe

**HTML**
- `canvas id="pieChart"`

**JS**
- new Chart(ctx, { type: 'pie', data: {...}, options: {...} })

Theme dependencies:
- `$PieColours`
- `$PieBorderWidth`

### 10.3 Doughnut Chart Recipe

Same as pie, type = `doughnut`.

### 10.4 Line Graph Recipe

Theme dependencies:
- `$Line Graph Colours` (dataset style chunks)
- `$AxisColours _X`
- `$GridColours`

The AI must generate dataset color blocks as raw JS fragments, example style:

~~~js
datasets: [
  { label: 'Sales', data: [..], borderColor: '...', backgroundColor: 'rgba(...)' },
  { label: 'Orders', data: [..], borderColor: '...', backgroundColor: 'rgba(...)' }
]
~~~

### 10.5 Bar Chart Recipe

Theme dependencies:
- `$Bar Chart Colours` (backgroundColor + borderColor arrays)

### 10.6 Horizontal Bar Chart Recipe

ChartJS modern versions use:
- type: 'bar'
- options: `indexAxis: 'y'`

### 10.7 Stacked Bar Chart Recipe

Use:
- `scales: { x: { stacked: true }, y: { stacked: true } }`

Theme dependencies:
- `$Stacked Bars Colours` (dataset style chunks)

### 10.8 Gauge Recipe

If implemented using ChartJS doughnut with partial ring:
- type: 'doughnut'
- rotation and circumference control
- theme gauge colors from `$Gauge Colours`

### 10.9 Progress Circle Recipe

HTML:
- `<canvas id='progressCircle' width='150' height='150'></canvas>`
- `<div id='progress_circleValue'></div>` (if used)

JS:
- draw arc based on percentage
- use `$Progress Circle Stroke`

### 10.10 Counter Recipe

HTML:
- `<div class='counter' id='counter'>0</div>`

JS:
- animate number from 0 to target over duration
- use requestAnimationFrame (or interval)

### 10.11 Progress Bar Recipe

HTML:
- `<div class='progress-bar' id='progressBar'></div>`

JS:
- animate width until target percentage

### 10.12 Data Table Recipe

HTML:
- `<table>` with `<thead>` and `<tbody>`
- table must be readable within `.chart-card`

Optional JS:
- none, can be static

### 10.13 Gantt Chart Recipe (HTML Table)

HTML:
- table rows with bars created via CSS classes

Optional JS:
- none, can be static

### 10.14 Timeline Recipe

HTML:
- progress line + fill element with `id='timelineFill'`

JS:
- set width based on progress percentage

---

## 11) Quality Rules for AI Output

### 11.1 Single Responsibility Per Variable

- One variable per block (HTML or JS).
- Do not create one mega variable for everything unless assembling.

### 11.2 Use Predictable IDs

Use semantic IDs:
- `pieChart`
- `pieChart2`
- `doughnutChart`
- `lineChart`
- `lineChartAlt`
- `barChart`
- `barChartAlt`
- `horizontalBarChart`
- `horizontalBarChartAlt`
- `stackedBarChart`
- `gaugeChart`
- `progressCircle`
- `counter`
- `progressBar`
- `timelineFill`

### 11.3 Keep Inline Styles Minimal and Consistent

If using inline absolute positioning, do it consistently:
- top/left/width/height in percentages
- keep `max-width` consistent with `width`

### 11.4 Fail Gracefully

If ChartJS is missing:
- dashboard should still load
- non-chart blocks should still display
- optional: show a message in chart containers

### 11.5 Avoid Colliding Globals

Prefer:
- self-invoking wrappers
- local const variables

---

## 12) Ambiguities and Clarifications Needed (You Must Answer)

The sample code implies several rules, but they are not fully defined. Please clarify these so the AI training can be exact.

### 12.1 What Exactly Is in `ChartJS::JS`?

- Is it the full ChartJS library minified code?
- Or is it a loader that loads ChartJS from CDN?
- Or is it your own wrapper functions?

The AI needs to know whether to:
- embed the library, or
- reference it externally, or
- assume it exists already.

### 12.2 ExecuteSQL Returns Multiple Rows

Your SQL is:

- `Select JS From ChartJS`

If ChartJS table contains multiple records:
- Which record should be used?
- Do you want:
  - the first row only
  - a specific row by `Name` or `Version`
  - concatenation of all rows

If there is a field like `ChartJS::Name` or `ChartJS::Key`, the AI should filter.

### 12.3 Where Does `$$Page` Get Rendered?

After setting `$$Page`, what script or WebViewer object consumes it?

- Is there a standard object name (example: `"wvMain"`)?
- Is there a standard script step: `Set Web Viewer [ Object Name: ... ; Action: "Set HTML" ; HTML: $$Page ]`?

The sample code stops at `$$Page` + `WebDirect Pause` + `Refresh Window`, but does not show the WebViewer injection step.

### 12.4 What Does “🖌️ Use Dashboard CSS” Return?

- Does it return only base CSS, and you append dynamic parts?
- Or does it accept parameters (theme, dynamic CSS, etc.)?

In the sample, the parameter is empty. If it supports parameters, the AI should use them.

### 12.5 Theme Canon: What Are the Official Theme Names?

The sample references many themes (some truncated). Please confirm the official list used in CauferoAppStarter, for example:

- AquaView
- Aurora
- ContrastCard
- Dokondo
- Electric Eclipse
- Ethereal Workspace
- Frosted Horizons
- Lumen Clean UI
- Minimal Elegance
- Modern Flex UI
- Modern Minimalist
- KoolTool
- NeatFrame
- (others)

The AI must match theme names exactly.

### 12.6 Chart Colors Format Is Mixed (List vs JS String)

Some theme variables are:
- FileMaker `List()` values
Some are:
- JS literal arrays inside strings like: `"'#00796B', '#80CBC4', ...'"`

Please confirm the standard:
- Should `$PieColours` be a JS array literal string only?
- Or should it be a FileMaker list that the AI converts to JS?

### 12.7 The ¶ Character Usage

Your code uses the `¶` symbol as a line delimiter inside strings.

- Is the AI required to use `¶` always for JS/CSS strings?
- Or is `& ¶ &` the standard only for FileMaker readability?

If you want strict consistency, define it.

---

## 13) AI Completion Checklist (Must Pass Before Output Is Accepted)

Before the AI finalizes any dashboard generation script, it must verify:

1. `$$App Section` set to `"Dashboard"`
2. `$Dashboard JS` retrieval defined and used in `<head>`
3. Theme variables computed from `$$Theme`
4. `$$KPI Card Colours` CSS mapping generated
5. `🖌️ Use Dashboard CSS` called and included in `$Styles`
6. Every `<canvas id='X'>` has matching JS `getElementById('X')`
7. `$HTML` contains all component blocks
8. `$Scripts` concatenates all JS blocks inside `<script>...</script>`
9. `$$Page` includes: doctype, html, head, body
10. WebDirect stabilization is executed (pause + refresh)
11. No missing variable references
12. No mismatched quotes or broken HTML tags

---

## 14) Minimal Working Example (Compressed Concept)

> This is not the full sample, but a small canonical example the AI can always generate as a starting dashboard: 2 KPI cards + 1 pie chart.

~~~filemaker
Set Variable [ $$App Section ; Value: "Dashboard" ]

Set Variable [ $Dashboard JS ;
  Value: Let (
    [
      sql = "Select JS From ChartJS" ;
      fieldseparator = "xxxxxxxxxx" ;
      rowseparator = "yyyyyyyyyy" ;
      result = ExecuteSQL ( sql ; fieldseparator ; rowseparator )
    ] ;
    result
  )
]

Set Variable [ $KPI Card Colours ;
  Value: Case (
    $$Theme = "Dokondo" ; List ( "#D51E26" ; "#0E3344" ; "#5FA4A1" ; "#41C5E0" ; "#D8743D" ; "#D5432E" ) ;
    List ( "#4D3A70" ; "#596B8C" ; "#1F4D79" ; "#EB5757" ; "#27AE60" ; "#B0BEC5" )
  )
]

Set Variable [ $$KPI Card Colours ;
  Value:
    ".kpi-card:nth-child(1) { background: " & GetValue ( $KPI Card Colours ; 1 ) & "; }" & ¶ &
    ".kpi-card:nth-child(2) { background: " & GetValue ( $KPI Card Colours ; 2 ) & "; }"
]

Perform Script [ Specified: From list ; “🖌️ Use Dashboard CSS” ; Parameter: "" ]
Set Variable [ $Styles ; Value: Get ( ScriptResult ) ]
Set Variable [ $Styles ; Value: "<style>" & $Styles & ¶ & $$KPI Card Colours & "</style>" ]

Set Variable [ $Cards HTML ;
  Value:
    "<div class='cards' style='position:absolute; top:1.5%; left:0.5%; gap:0.5%; width:99%; height:20%;'>" &
      "<a href='#kpi-1' class='kpi-card'>" &
        "<div class='kpi-title'>Total Products</div>" &
        "<div class='kpi-value' id='kpiTotalProducts'>1245</div>" &
      "</a>" &
      "<a href='#kpi-2' class='kpi-card'>" &
        "<div class='kpi-title'>Total Customers</div>" &
        "<div class='kpi-value' id='kpiTotalCustomers'>320</div>" &
      "</a>" &
    "</div>"
]

Set Variable [ $Pie Chart HTML ;
  Value:
    "<div class='chart-card' style='position:absolute; top:22.5%; left:0.5%; width:30%; height:45%'>" &
      "<h2>Customer Satisfaction</h2>" &
      "<div class='chart-container' style='width:100%; height:80%;'>" &
        "<canvas id='pieChart'></canvas>" &
      "</div>" &
    "</div>"
]

Set Variable [ $HTML ; Value: $Cards HTML & ¶ & $Pie Chart HTML ]

Set Variable [ $Pie Chart JS ;
  Value:
    "(function () {" &
      "const el = document.getElementById('pieChart');" &
      "if (!el || typeof Chart === 'undefined') return;" &
      "const ctx = el.getContext('2d');" &
      "new Chart(ctx, {" &
        "type: 'pie'," &
        "data: {" &
          "labels: ['Very Satisfied','Satisfied','Neutral','Unsatisfied','Very Unsatisfied']," &
          "datasets: [{" &
            "data: [45,25,15,10,5]," &
            "backgroundColor: ['#D51E26','#0E3344','#5FA4A1','#41C5E0','#D8743D']," &
            "borderWidth: 1" &
          "}]" &
        "}," &
        "options: { responsive: true, maintainAspectRatio: false }" &
      "});" &
    "})();"
]

Set Variable [ $Scripts ; Value: "<script>" & ¶ & $Pie Chart JS & ¶ & "</script>" ]

Set Variable [ $$Page ;
  Value:
    "<!DOCTYPE html><html lang='en'><head>" &
      "<meta charset='UTF-8'>" &
      "<meta name='viewport' content='width=device-width, initial-scale=1.0'>" &
      "<title>Dashboard</title>" &
      "<script>" & $Dashboard JS & "</script>" &
      $Styles &
    "</head><body>" &
      $HTML &
      $Scripts &
    "</body></html>"
]

Perform Script [ Specified: From list ; “WebDirect Pause” ; Parameter: "" ]
Refresh Window [ Flush cached join results ; Flush cached external data ]

---

# Side Menu Page Documentation (CauferoAppStarter)
Version: 1.0  
Purpose: Train an AI Agent to build a complete Side Menu page in CauferoAppStarter using FileMaker + WebViewer (HTML/CSS/JS) + ExecuteSQL.

---

## 0) What the Side Menu Page Is
The Side Menu page is a WebViewer-driven UI panel that:
- Shows navigation links a user is allowed to see, based on App + Role.
- Supports parent links and sub links (accordion).
- Highlights the currently selected link.
- Can insert section dividers between link groups.
- Has a search input to filter visible links.
- Shows user profile photo and user actions (example: Change Password).
- Can expose Theme switching UI based on the app’s available themes.

This is not a generic menu. It is driven by your database and current user context.

---

## 1) Output Contract (What This Script Must Produce)
When the “Build Side Menu” script finishes, these must be true:

### 1.1 Global Variables Produced
- `$$My Links`  
  A fully concatenated HTML snippet containing all menu items (parents plus optional accordion submenus), including any `<hr>` dividers.

- `$$Menu`  
  The full HTML document string that can be pushed into the WebViewer.

### 1.2 The WebViewer Shows the Menu
The layout hosting the side menu contains a WebViewer object, for example:
- Object Name: `wvMenu`

The script ends by pushing `$$Menu` into that object.

~~~filemaker
Set Web Viewer [
  Object Name: "wvMenu" ;
  Action: "Set HTML" ;
  HTML: $$Menu
]
~~~

---

## 2) Data Model Requirements
The side menu relies on a specific model of Apps, Roles, and Links.

### 2.1 Tables
- `Apps`
- `Links`
- `Role Links`

### 2.2 Required Fields (Minimum)
#### Apps
- `Apps::ID`
- `Apps::Available Themes`  
  A return-delimited list of theme names. Example:
  - NeatFrame
  - KoolTool
  - Slate

#### Links
- `Links::ID`
- `Links::Link`  
  Display name. Example: `Dashboard`
- `Links::Order`  
  Numeric ordering inside the menu.
- `Links::SVG Icon`  
  The SVG `path d="..."` value (the actual path string).
- `Links::Note`  
  Used for special behavior. Example: `After Section Divider`
- `Links::Parent Link ID`  
  Null for top-level parent links. Populated for sub links.
- Optional fields that often exist in real builds:
  - `Links::Layout Name` or `Links::Target Layout`
  - `Links::Script Name` (if navigation is script-driven)
  - `Links::Pending` (boolean) used to hide links still being built

#### Role Links
- `Role Links::App ID`
- `Role Links::Role ID`
- `Role Links::Link ID`

### 2.3 Relationship Concept (Mental Model)
A user has a Role in an App.  
Role Links defines which Links that role is allowed to see.

---

## 3) Runtime Context Inputs (Globals and Fields)
The menu builder depends on current context. These values must already exist before building the menu.

### 3.1 Required Context Values
- `Settings::App ID`  
  The current App ID.
- Current Role ID (wherever you store it)  
  Example patterns:
  - `$$Role ID`
  - `Users::Role ID`
  - `Settings::Role ID`

### 3.2 Required Globals Used by the Menu
- `$$Link ID`  
  The currently selected Link ID (used to add class `selected`).
- `$$Parent Link ID`  
  The currently selected parent Link ID (used to expand accordion).
- `$$My Photo`  
  User profile photo base64 (png). Can be empty.

### 3.3 App Name and Slogan (Optional, If You Show Them)
If your menu header includes app identity:
- `$$App Name`
- `$$App Slogan`

---

## 4) High-Level Build Flow
The script is built in distinct phases:

1) Query allowed parent links (App + Role, parent only).  
2) Loop through parent links:
   - Split record into fields
   - Build SVG icon HTML
   - Inject divider if Note says so
   - Fetch sub links using a sub-script
   - Build link HTML:
     - Direct anchor if no sub links
     - Accordion if sub links exist
   - Append into `$$My Links`
3) Build Themes list from Apps table.
4) Pull Menu CSS from centralized script.
5) Build JS functions as strings.
6) Assemble final `$$Menu` HTML document string.
7) Push `$$Menu` into the menu WebViewer.

---

## 5) Phase 1: Query Parent Links Using ExecuteSQL
### 5.1 Goal
Return a list of top-level Links the user can see:
- Allowed by Role Links
- Belonging to current App
- Parent only: `Links::Parent Link ID IS NULL`
- Excluding pending links (if you have that field)

### 5.2 Query Output Shape
Each row must return these columns in this exact order:
1. Link ID
2. Link Name
3. Order
4. SVG Icon path
5. Note

Your loop expects those 5 values per row.

### 5.3 ExecuteSQL Template
You will build a Let() block with:
- `sql`
- `fieldseparator`
- `rowseparator`
- parameters for app_id and role_id

Important: Your loop later does this:
- `GetValue ( $Link Record As List ; 1..5 )`

That only works if each row can be converted into a value list, one value per line.

### 5.4 Recommended Separator Strategy
- `fieldseparator = "xxxxx"`
- `rowseparator = "¶"`

Then later:
- Convert `"xxxxx"` into `¶` so each field becomes a value.
This matches your sample.

Why this strategy exists:
- ExecuteSQL returns a row as a single text line.
- You want to split fields safely without conflicts.
- A weird separator like `xxxxx` avoids accidental collisions with normal text.

### 5.5 Example SQL (Adjust Field Names to Your Schema)
~~~filemaker
Set Variable [ $List ;
  Value:
  Let ( [
    sql =
      "SELECT
         t2.ID,
         t2.Link,
         t2.\"Order\",
         t2.\"SVG Icon\",
         t2.\"Note\"
       FROM \"Role Links\" t1
       LEFT JOIN Links t2 ON ( t1.\"Link ID\" = t2.ID )
       WHERE
         t1.\"App ID\" = ?
         AND t1.\"Role ID\" = ?
         AND t2.\"Parent Link ID\" IS NULL
         AND ( t2.Pending IS NULL OR t2.Pending = 0 )
       ORDER BY t2.\"Order\" " ;

    fieldseparator = "xxxxx" ;
    rowseparator   = "¶" ;

    app_id  = Settings::App ID ;
    role_id = $$Role ID ;

    result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; app_id ; role_id )
  ] ; result )
]
~~~

### 5.6 Count Rows
~~~filemaker
Set Variable [ $Total ; Value: ValueCount ( $List ) ]
Set Variable [ $$My Links ; Value: "" ]
~~~

---

## 6) Phase 2: Loop Through Parent Links
### 6.1 Loop Skeleton
Initialize `$i` properly before incrementing.

~~~filemaker
Set Variable [ $i ; Value: 0 ]

If [ $Total > 0 ]
  Loop [ Flush: Always ]
    Set Variable [ $i ; Value: $i + 1 ]
    Set Variable [ $Link Record ; Value: GetValue ( $List ; $i ) ]

    Exit Loop If [ $i ≥ $Total ]
  End Loop
End If
~~~

### 6.2 Convert Row to Value List
Your sample uses:
- `$Link Record` is one row: `IDxxxxxLinkxxxxxOrderxxxxxSVGxxxxxNote`
- Convert fields into separate lines:

~~~filemaker
Set Variable [ $Link Record As List ;
  Value: Substitute ( $Link Record ; "xxxxx" ; ¶ )
]
~~~

### 6.3 Extract Columns
~~~filemaker
Set Variable [ $ID        ; Value: GetValue ( $Link Record As List ; 1 ) ]
Set Variable [ $Link Name ; Value: GetValue ( $Link Record As List ; 2 ) ]
Set Variable [ $Order     ; Value: GetValue ( $Link Record As List ; 3 ) ]
Set Variable [ $SVG       ; Value: GetValue ( $Link Record As List ; 4 ) ]
Set Variable [ $Note      ; Value: GetValue ( $Link Record As List ; 5 ) ]
~~~

---

## 7) SVG Icon Rendering
### 7.1 Data Format
`Links::SVG Icon` stores only the `d="..."` path content.
Example:
`M12 2C6.48 2 2 6.48 2 12...`

### 7.2 HTML Wrapper
You create the actual SVG wrapper in script:

~~~filemaker
Set Variable [ $SVG Icon HTML ;
  Value:
  " <svg class='icon' viewBox='0 0 24 24'>
      <path d='" & $SVG & "'></path>
    </svg> "
]
~~~

### 7.3 CSS Expectation
The CSS must style `.icon` so icons look consistent:
- width/height
- fill color tied to menu state (normal vs selected)

---

## 8) Section Dividers
### 8.1 Trigger Rule
If a parent link has:
- `Links::Note = "After Section Divider"`

then append a divider immediately after this link entry.

### 8.2 Divider HTML
Your sample:
~~~filemaker
If [ $Note = "After Section Divider" ]
  Set Variable [ $$My Links ;
    Value: $$My Links & " <hr class='divider always-visible-divider' /> " & ¶
  ]
End If
~~~

### 8.3 Divider Filtering Rule
The class `always-visible-divider` is used so the divider still shows (or can be controlled) during search filtering.
Your JS must deliberately decide how dividers behave during filtering.

---

## 9) Fetch Sub Links (Accordion Support)
### 9.1 Sub Script Contract
You call:
- `Perform Script [ “Get Sub Menu Links 1” ; Parameter: $ID ]`
and then:
- `$Sub Links 1 = Get ( ScriptResult )`

Therefore, the sub-script must return a string that is either:
- empty, meaning no sub links
- non-empty, meaning sub links exist (and the returned value is embedded into HTML)

### 9.2 Strong Recommendation
The sub-script should return the final HTML for sub links, already wrapped correctly.

Example returned snippet:
~~~html
<div class='submenu'>
  <a class='submenu-link' data-id='123' data-name='Reports' onclick="goToLink('123')">Reports</a>
  <a class='submenu-link' data-id='124' data-name='Settings' onclick="goToLink('124')">Settings</a>
</div>
~~~

### 9.3 Parent Script Call
~~~filemaker
Perform Script [ Specified: From list ; “Get Sub Menu Links 1” ; Parameter: $ID ]
Set Variable [ $Sub Links 1 ; Value: Get ( ScriptResult ) ]
~~~

---

## 10) Build HTML for Each Parent Link
There are two cases:
1) Parent link has no sub links: direct navigation link
2) Parent link has sub links: accordion container

### 10.1 Direct Link Case (No Sub Links)
Behavior:
- Clicking takes you directly to a target
- You mark it selected if `$ID = $$Link ID`

HTML must:
- include `data-name` for search
- include `class='selected'` when active
- include an onclick hook that calls FileMaker scripts

Example pattern:
~~~filemaker
Set Variable [ $isSelected ;
  Value: If ( $ID = $$Link ID ; " class='selected' " ; "" )
]

Set Variable [ $slug ;
  Value: Substitute ( Lower ( $Link Name ) ; " " ; "-" )
]

Set Variable [ $Link Record's HTML ;
  Value:
  "<a href='#" & $slug & "' data-name='" & $Link Name & "' " &
    $isSelected &
    " onclick=""goToLink('" & $ID & "')"" >" &
      "<span class='link-left'>" & $SVG Icon HTML & "<span class='link-text'>" & $Link Name & "</span></span>" &
  "</a>"
]
~~~

### 10.2 Accordion Case (Has Sub Links)
Behavior:
- Parent row is a button
- Clicking toggles accordion open/closed
- It is expanded when `$ID = $$Parent Link ID`

Example pattern:
~~~filemaker
Set Variable [ $accordionClass ;
  Value: If ( $ID = $$Parent Link ID ; "accordion expanded" ; "accordion" )
]

Set Variable [ $Link Record's HTML ;
  Value:
  " <div class='menu_item'>
      <button class='" & $accordionClass & "' data-name='" & $Link Name & "' onclick=""toggleAccordion(this)"">
        <span class='link-left'>" & $SVG Icon HTML & "<span class='link-text'>" & $Link Name & "</span></span>
        <span class='chev'></span>
      </button>
      " & $Sub Links 1 & "
    </div> "
]
~~~

### 10.3 Append into `$$My Links`
~~~filemaker
Set Variable [ $$My Links ; Value: $$My Links & $Link Record's HTML & ¶ ]
~~~

---

## 11) Themes List (Apps::Available Themes)
This section builds values used by the menu UI (example: a theme switch modal).

### 11.1 Pull Themes From Apps
~~~filemaker
Set Variable [ $List ;
  Value:
  Let ( [
    sql = "SELECT \"Available Themes\" FROM \"Apps\" WHERE ID = ?" ;
    fieldseparator = "|" ;
    rowseparator   = "¶" ;
    app_id = Settings::App ID ;
    result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; app_id )
  ] ; result )
]
~~~

### 11.2 Normalize Themes
- Sort
- Remove empties
~~~filemaker
Set Variable [ $List ; Value: RemoveEmptyValues ( SortValues ( $List ) ) ]
~~~

### 11.3 Build `'Theme1','Theme2'` String (If Needed In JS)
~~~filemaker
Set Variable [ $Themes ; Value: "'" & Substitute ( $List ; "¶" ; "',¶'" ) & "'" ]
~~~

### 11.4 Modal Height Rule
~~~filemaker
Set Variable [ $Total Available Themes ; Value: ValueCount ( $List ) ]
Set Variable [ $Modal Height ;
  Value: Case ( $Total Available Themes ≤ 8 ; "30%" ; "60%" )
]
~~~

### 11.5 User Profile Photo (Fallback)
If `$$My Photo` is base64, build a data URI. Else use a placeholder URL.

~~~filemaker
Set Variable [ $User Profile Photo ;
  Value:
  If ( not IsEmpty ( $$My Photo ) ;
    "data:image/png;base64," & $$My Photo ;
    "https://upload.wikimedia.org/wikipedia/commons/8/89/Portrait_Placeholder.png"
  )
]
~~~

---

## 12) CSS Source of Truth: “🖌️ Use Menu CSS”
### 12.1 Contract
The CSS must come from a centralized script:
- Script name: `🖌️ Use Menu CSS`
- Result: full CSS text, no `<style>` tags

### 12.2 Call Pattern
~~~filemaker
Perform Script [ Specified: From list ; “🖌️ Use Menu CSS” ; Parameter: "" ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]
~~~

### 12.3 CSS Must Cover These Selectors
Minimum selectors expected by generated HTML and JS:
- Layout and containers:
  - `body`
  - `.menu`
  - `.menu-header`
  - `#menuLinks` (scroll container)
- Links:
  - `a`
  - `a.selected`
  - `.menu_item`
  - `.link-left`
  - `.link-text`
- Icons:
  - `.icon`
- Accordion:
  - `.accordion`
  - `.accordion.expanded`
  - `.submenu`
  - `.submenu-link`
  - `.chev`
- Divider:
  - `.divider`
  - `.always-visible-divider`
- Search:
  - `#menuSearch`
- Profile area:
  - `.profile`
  - `.profile-photo`

---

## 13) JavaScript (WebViewer Side)
### 13.1 Guiding Rule
All actions must route through FileMaker scripts using:
- `FileMaker.PerformScript(scriptName, parameters)`

### 13.2 Build Script Strings in FileMaker
You store JS code in FileMaker variables, then inject into HTML.

#### Change Password
~~~filemaker
Set Variable [ $Change Password Script ; Value: "Change Password" ]
Set Variable [ $Change Password Script ;
  Value:
  "function changePassword(parameters) {
     FileMaker.PerformScript('" & $Change Password Script & "', parameters);
   }"
]
~~~

#### Search Filter
Your sample implies:
- An input `menuSearch`
- A container `menuLinks`
- Filtering by matching `data-name` values

A robust filtering function should:
- Read input value
- Loop through:
  - anchors
  - accordion buttons
  - submenu links
- Show/hide based on text match
- Decide what happens to dividers

Example JS (concept):
~~~javascript
function filterMenu() {
  const input = document.getElementById('menuSearch').value.toLowerCase().trim();

  const container = document.getElementById('menuLinks');
  if (!container) return;

  const items = container.querySelectorAll('a, button.accordion, .submenu a');
  items.forEach(el => {
    const name = (el.getAttribute('data-name') || el.textContent || '').toLowerCase();
    const match = name.includes(input);
    const wrapper = el.closest('.menu_item') || el;
    if (input === '') {
      wrapper.style.display = '';
    } else {
      wrapper.style.display = match ? '' : 'none';
    }
  });

  const dividers = container.querySelectorAll('.always-visible-divider');
  dividers.forEach(hr => {
    if (input === '') hr.style.display = '';
    else hr.style.display = '';
  });
}
~~~

#### Center Selected Item
Your sample includes:
- `#menuLinks` container
- `.selected` item within it

Behavior:
- Scroll selected item into the middle of the scroll area on load

Example JS:
~~~javascript
function centerSelectedInMenu() {
  var container = document.getElementById('menuLinks');
  if (!container) return;

  var selected = container.querySelector('.selected');
  if (!selected) return;

  var containerRect = container.getBoundingClientRect();
  var selectedRect = selected.getBoundingClientRect();

  var containerCenter = containerRect.top + (containerRect.height / 2);
  var selectedCenter  = selectedRect.top + (selectedRect.height / 2);

  var delta = selectedCenter - containerCenter;
  container.scrollTop += delta;
}
~~~

#### Accordion Toggle
If your HTML uses `toggleAccordion(this)`:
~~~javascript
function toggleAccordion(btn) {
  if (!btn) return;

  btn.classList.toggle('expanded');

  var parent = btn.closest('.menu_item');
  if (!parent) return;

  var submenu = parent.querySelector('.submenu');
  if (!submenu) return;

  if (btn.classList.contains('expanded')) submenu.style.display = '';
  else submenu.style.display = 'none';
}
~~~

### 13.3 Navigation Function (Required)
You must define a single point of navigation. Example: `goToLink(id)`

This function should call a FileMaker script like:
- `Navigate To Link`

~~~javascript
function goToLink(linkId) {
  FileMaker.PerformScript('Navigate To Link', linkId);
}
~~~

In FileMaker, `Navigate To Link`:
- Receives `linkId`
- Sets:
  - `$$Link ID = linkId`
  - Possibly sets `$$Parent Link ID`
- Navigates to target layout or performs target script
- Rebuilds menu so highlight updates

---

## 14) Assemble the Full HTML Document `$$Menu`
### 14.1 Required Document Parts
Your `$$Menu` must include:
- `<!DOCTYPE html>`
- `<head>` with CSS injection
- optional CDN icons (Font Awesome)
- `<body>` that contains:
  - profile header
  - search input
  - menu links container with `$$My Links`
- `<script>` injection containing your JS functions
- onload hooks (center selected)

### 14.2 Example Assembly Pattern
~~~filemaker
Set Variable [ $$Menu ;
  Value:
  "<!DOCTYPE html>
  <html lang='en'>
  <head>
    <meta charset='UTF-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>

    <link rel='stylesheet' href='https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css'>

    <style>" & $styles & "</style>
  </head>

  <body onload='centerSelectedInMenu()'>

    <div class='menu'>

      <div class='menu-header'>
        <div class='profile'>
          <img class='profile-photo' src='" & $User Profile Photo & "' alt='' />
          <div class='profile-meta'>
            <div class='profile-name'>" & $$App Name & "</div>
            <div class='profile-sub'>" & $$App Slogan & "</div>
          </div>
        </div>

        <div class='profile-actions'>
          <button onclick=""changePassword('')"">Change Password</button>
        </div>

        <input id='menuSearch' placeholder='Search...' oninput='filterMenu()' />
      </div>

      <div id='menuLinks'>
        " & $$My Links & "
      </div>

    </div>

    <script>
      " & $Change Password Script & "
      " & $Scroll To Selected Menu Item Script & "
      " & $Search Script & "
      function goToLink(linkId) { FileMaker.PerformScript('Navigate To Link', linkId); }
      function toggleAccordion(btn) { /* implement toggle */ }
    </script>

  </body>
  </html>"
]
~~~

---

## 15) Selection State Rules (Critical)
### 15.1 Parent Link Highlight
Direct link parent:
- Add `class='selected'` if `$ID = $$Link ID`

Accordion parent:
- Add `expanded` class if `$ID = $$Parent Link ID`

### 15.2 Sub Link Highlight
If sub links exist, sub link HTML should also mark selected.
Example: add `selected` class to the sub link anchor when:
- subLinkId = $$Link ID

This requires the sub-links script to know `$$Link ID`.

---

## 16) Performance Rules
- ExecuteSQL once for parent links.
- Sub links can be:
  - fetched per parent (simple, may be slower)
  - fetched once and grouped (faster, more complex)

If the menu is big, avoid one ExecuteSQL per parent. Prefer a single query that returns all links, then group inside FileMaker.

---

## 17) Security and Data Integrity Rules
- Never trust UI state alone.
- FileMaker scripts must validate:
  - user role
  - link permission
before navigating.

Even if the menu hides a link, a user could still try to call a script manually.
Permissions must be enforced in FileMaker.

---

## 18) Debugging Checklist
When the menu looks wrong, check in this order:

1) Is `Settings::App ID` correct?
2) Is role ID correct?
3) Does ExecuteSQL return rows?
4) Are separators correct?  
   If `xxxxx` appears in output HTML, your Substitute step failed or separators mismatch.
5) Is `$$My Links` populated?
6) Does CSS script return valid CSS?
7) Does the WebViewer allow external CSS from CDNs (Font Awesome)?
8) Is `FileMaker.PerformScript` firing?  
   Test with a tiny script that logs parameters.

---

## 19) Test Cases (Must Pass)
1) Role has zero links  
   Result: menu shows empty state or shows only header.
2) Role has parent links only  
   Result: direct links work, selected highlight works.
3) Role has parent links with sub links  
   Result: accordion toggles, expands correct parent, sub links navigate.
4) Dividers  
   Result: divider appears after the flagged link.
5) Search  
   Result: typing filters by `data-name` correctly.
6) Theme list  
   Result: correct modal height, correct theme list.
7) User photo empty  
   Result: placeholder image displays.
8) Selected centering  
   Result: on load, selected item is brought into view.

---

## 20) Clarifications Needed (Ambiguities Found In The Sample)
These are not optional. The AI Agent must be told the exact answers so it can generate correct code every time.

### 20.1 The SQL In Your Sample Is Truncated
Your sample begins:
`Select t2.ID, t2.Link, t2."Order", t2."SVG Icon", t2."Note" ... and t2.Pending…`
- What is the exact filter on `t2.Pending`?
- Is it `= 0`, `Is Null`, or a text status?

### 20.2 What Exactly Does “Get Sub Menu Links 1” Return?
Your parent script checks:
`If [ IsEmpty ( $Sub Links 1 ) ]`
So `$Sub Links 1` is text.  
Clarify which is true:
- A) It returns raw HTML to embed directly.
- B) It returns a list of sub link records that the parent script converts into HTML.
- C) It returns JSON, and JS builds the submenu.

Pick one and lock it.

### 20.3 Separator Choice “xxxxx”
Your loop converts `xxxxx` into `¶`.
Clarify:
- Is `xxxxx` always the field separator in ExecuteSQL for menus?
- Is there a shared constant for it in your framework?

### 20.4 Link Target Behavior For Direct Links
Your sample has `href='#slug'` and an onclick stub.
Clarify the official navigation contract:
- A) Navigation is always `FileMaker.PerformScript('Navigate To Link', linkId)`
- B) Navigation is layout-based using `Go to Layout [ Links::Layout Name ]`
- C) Some links call scripts, some go to layouts

If it is mixed, define the fields in Links table that drive it.

### 20.5 Global Variables Ownership
These globals are used:
- `$$Link ID`
- `$$Parent Link ID`
- `$$My Photo`
Clarify:
- Which script sets them?
- When do they refresh?
- Is menu rebuilt every navigation or only sometimes?

### 20.6 WebViewer Object Name
Your sample shows building `$$Menu` but does not show the final Set Web Viewer step.
Clarify the official object name(s) used across CauferoAppStarter:
- Example: `wvMenu`, `wvSideMenu`, `wvLinks`

Lock one standard.

### 20.7 Theme Switching UI
You generate:
- `$Themes`
- `$Modal Height`
Clarify:
- Does the menu actually render a theme picker UI now?
- Which FileMaker script applies a theme after selection?

---

## 21) Implementation Recipe (AI Agent Step List)
When the AI Agent is asked to build a new side menu page, it must follow this exact recipe:

1) Confirm required context variables exist: App ID, Role ID, selected Link IDs, user photo.
2) Run ExecuteSQL to get allowed parent links with strict field order.
3) Loop parents:
   - extract ID, name, order, svg, note
   - build icon HTML
   - insert divider if note matches
   - get sub links output using the sub links script
   - build either direct link or accordion block
   - append into `$$My Links`
4) Query `Apps::Available Themes`, normalize list, compute modal height.
5) Call `🖌️ Use Menu CSS`, store `$styles`.
6) Build JS strings:
   - changePassword
   - filterMenu
   - centerSelectedInMenu
   - toggleAccordion
   - goToLink
7) Assemble full `$$Menu` HTML document.
8) Set WebViewer HTML to `$$Menu`.
9) Validate test cases.

---

## 22) Naming Conventions (Recommended)
- Build script: `🧭 Build Side Menu`
- Sub links script: `🧭 Get Sub Menu Links 1`
- CSS script: `🖌️ Use Menu CSS`
- Navigation script: `🧭 Navigate To Link`
- WebViewer object: `wvMenu`
- Globals:
  - `$$Link ID`
  - `$$Parent Link ID`
  - `$$My Links`
  - `$$Menu`

---

## 23) Minimal Deliverables Checklist
A Side Menu page is considered complete only when all of these exist:

- [ ] Links table contains SVG paths for icons
- [ ] Role Links defines allowed links for a role
- [ ] Build script runs without errors
- [ ] `$$Menu` renders correctly in WebViewer
- [ ] Direct links navigate correctly
- [ ] Accordion links expand/collapse
- [ ] Selected link highlight works
- [ ] Search works
- [ ] Dividers work
- [ ] Theme list builds correctly (even if UI is not shown yet)
- [ ] Profile photo fallback works

---

# CauferoAppStarter Documentation
## Building a List Page for Any Entity Type (AI Agent Training Spec)

### Purpose
This document trains an AI Agent to create a fully working **List Page generator script** for any **Entity Type** inside the FileMaker file **CauferoAppStarter**.

A "List Page" in this system is generated by:
1) Setting a standard set of script variables (SQL, headers, column behavior, UI labels, empty-state content).
2) Calling a shared renderer script (`+++ Display List HTML` or legacy `+++ Display List HTML_`) with a strict parameter order.
3) Setting any post-render UI state globals (example: `$$Tab To Show`).

This document is written as a build spec. Follow it exactly.

---

# 1. Definitions

## 1.1 Entity Type
An Entity Type is a table-backed concept that has:
- A FileMaker table (example: `Departments`, `Staff`, `Job Titles`)
- A "List Page generator script" (example: `+++ Staff List Page`)
- A "Details Page script" (example: `+++ Staff Details Page`)
- Optional "Add New Record script" (example: `Add New Staff`)

## 1.2 List Page Generator Script
A List Page generator script:
- Sets `$$App Section`
- Defines SQL (first selected column must be the record ID)
- Defines column headers and optional layout rules
- Defines search behavior
- Defines empty-state visuals and messages
- Calls the shared list renderer script with a strict parameter order

---

# 2. FileMaker Script Naming Rules

Use these conventions consistently:
- List Page script name: `+++ <Entity Name> List Page`
- Details Page script name: `+++ <Entity Name> Details Page`
- Add New script name: `Add New <Entity Name>`

Examples:
- `+++ Department List Page`
- `+++ Job Title Details Page`
- `Add New Staff`

---

# 3. Shared Renderer Scripts (Critical)

There are two renderer scripts seen in the samples:

- `+++ Display List HTML`
- `+++ Display List HTML_`

They use different parameter signatures.

## 3.1 Preferred Renderer (New Signature)
Use:
- `+++ Display List HTML`

It supports column widths and categorization (grouping).

## 3.2 Legacy Renderer (Old Signature)
Use:
- `+++ Display List HTML_`

It appears to be an older signature that does not accept column widths and categorization.

You must choose the renderer that matches the parameter list you are passing.

---

# 4. Data Contract: SQL Output Requirements (Critical)

The SQL query must follow these rules:

## 4.1 First Selected Column Must Be Record ID
The very first selected field must be the primary key (ID) of the entity record you want to view.

Examples:
- `SELECT t1.ID, t1.Name, ... FROM Departments t1 ...`
- `SELECT t1.ID, t1."Staff Number", ... FROM Staff t1 ...`

The renderer depends on the first column being ID so it can route row actions to the correct record.

## 4.2 Output Columns After ID Must Match List UI Columns (With 2 Important Exceptions)
The SQL returns a dataset. The UI will show:
- A computed `S/N` column
- The columns you provide headers for (data columns)
- An `Actions` column on the far right (row actions)

So your `Column Headers` must describe the data columns that appear in the list, plus `S/N` and `Actions`.

### Exception A: Status must be the last data column (when used)
If the SQL query returns a status value AND the list page will display a status column, then:
- Status must be the **last data column** in the list (right before `Actions`)
- Therefore, status must appear as the **last data header** in `$Column Headers` before `Actions`

Example (good):
- `"S/N,Name,Role,Status,Actions"`

Example (bad):
- `"S/N,Status,Name,Role,Actions"`

### Exception B: Categorization field must be returned by SQL but hidden from displayed data columns
If the list will be categorized/grouped, the categorization value **must be present in the SQL recordset**, but it **must NOT be displayed as a visible data column**.

That means:
- You include the category field in the SQL `SELECT` list
- But you do **not** include the category label in `$Column Headers`

This is how the renderer can group rows by the category value without showing it as a column.

**Key concept:**
- SQL can return more columns than the UI displays.
- The categorization column index points to the position of the category field in the SQL resultset (see indexing rules below).

Example:
SQL selected fields:
- `ID, Name, Role, Category`

Displayed headers:
- `"S/N,Name,Role,Actions"`

Categorization column:
- The category is the **4th selected field in the SQL recordset** (after ID is 1st, Name is 2nd, Role is 3rd, Category is 4th)
- So `$Categorization Column = 4`

Result:
- Category does not show as a visible column
- But the list groups by category

## 4.3 Aggregations Require GROUP BY
If SQL uses `COUNT(...)` or any aggregation, add a correct `GROUP BY` to match non-aggregated fields.

Example pattern:
- `COUNT(t4."Staff ID") AS NumberOfStaff`
- `GROUP BY t1.ID, t1.Name, t2."First Name", t2."Last Name"`

## 4.4 Consistent Aliasing
Use table aliases in the form:
- `t1`, `t2`, `t3`, `t4`

This keeps SQL consistent and predictable.

## 4.5 Friendly Display Fields
Use computed fields for readable UI text:
- Full name: `t1."First Name" || ' ' || t1."Last Name" AS FullName`
- Status mapping: `CASE WHEN ... THEN ... ELSE ... END AS Status`

---

# 5. List Page Variable Set (Master Spec)

This section defines the variables your List Page script must set.

There are two variants:
- New variant for `+++ Display List HTML` (recommended)
- Legacy variant for `+++ Display List HTML_`

---

## 5.1 Variables for New Renderer: `+++ Display List HTML` (Recommended)

### Required
- `$$App Section`
- `$SQL`
- `$SQL Condition`
- `$Column Headers`
- `$Status Column`
- `$Date Columns`
- `$Number Columns`
- `$Categorization Column`
- `$Searchable Columns`
- `$Add New Record Script Name`
- `$View Record Script Name`
- `$Search Field Placeholder`
- `$Total Records Label`
- `$Add New Record Button Label`
- Empty-state icon (either SVG path or URL depending on your renderer implementation)
- Empty-state title
- Empty-state body

### Optional
- `$Column Widths` (highly recommended for stable UI sizing)

---

## 5.2 Variables for Legacy Renderer: `+++ Display List HTML_` (Old)

### Required
- `$$App Section`
- `$SQL`
- `$SQL Condition`
- `$Column Headers`
- `$Status Column`
- `$Date Columns`
- `$Number Columns`
- `$Searchable Columns`
- `$Add New Record Script Name`
- `$View Record Script Name`
- `$Search Field Placeholder`
- `$Total Records Label`
- `$Add New Record Button Label`
- Empty-state icon SVG path
- Empty-state title
- Empty-state body

This legacy signature does not include:
- `$Column Widths`
- `$Categorization Column`

---

# 6. Parameter Order (Strict)

The renderer script reads a single parameter string separated by `¶`. Order matters.

## 6.1 Parameter Order for `+++ Display List HTML` (New)

Pass parameters in this exact order:

1. `$SQL`
2. `$SQL Condition`
3. `$Column Headers`
4. `$Column Widths`
5. `$Status Column`
6. `$Date Columns`
7. `$Number Columns`
8. `$Categorization Column`
9. `$Searchable Columns`
10. `$Add New Record Script Name`
11. `$View Record Script Name`
12. `$Search Field Placeholder`
13. `$Total Records Label`
14. `$Add New Record Button Label`
15. `$No Record Found Icon` (SVG path or URL, depends on your implementation)
16. `$No Record Found Message Title`
17. `$No Record Found Message Body`

### Canonical call shape
~~~filemaker
Perform Script [
  Specified: From list ;
  “+++ Display List HTML” ;
  Parameter:
    $SQL & ¶ &
    $SQL Condition & ¶ &
    $Column Headers & ¶ &
    $Column Widths & ¶ &
    $Status Column & ¶ &
    $Date Columns & ¶ &
    $Number Columns & ¶ &
    $Categorization Column & ¶ &
    $Searchable Columns & ¶ &
    $Add New Record Script Name & ¶ &
    $View Record Script Name & ¶ &
    $Search Field Placeholder & ¶ &
    $Total Records Label & ¶ &
    $Add New Record Button Label & ¶ &
    $No Record Found Icon & ¶ &
    $No Record Found Message Title & ¶ &
    $No Record Found Message Body
]
~~~

## 6.2 Parameter Order for `+++ Display List HTML_` (Legacy)

Pass parameters in this exact order:

1. `$SQL`
2. `$SQL Condition`
3. `$Column Headers`
4. `$Status Column`
5. `$Date Columns`
6. `$Number Columns`
7. `$Searchable Columns`
8. `$Add New Record Script Name`
9. `$View Record Script Name`
10. `$Search Field Placeholder`
11. `$Total Records Label`
12. `$Add New Record Button Label`
13. `$No Record Found Icon SVG Path`
14. `$No Record Found Message Title`
15. `$No Record Found Message Body`

### Canonical call shape
~~~filemaker
Perform Script [
  Specified: From list ;
  “+++ Display List HTML_” ;
  Parameter:
    $SQL & ¶ &
    $SQL Condition & ¶ &
    $Column Headers & ¶ &
    $Status Column & ¶ &
    $Date Columns & ¶ &
    $Number Columns & ¶ &
    $Searchable Columns & ¶ &
    $Add New Record Script Name & ¶ &
    $View Record Script Name & ¶ &
    $Search Field Placeholder & ¶ &
    $Total Records Label & ¶ &
    $Add New Record Button Label & ¶ &
    $No Record Found Icon SVG Path & ¶ &
    $No Record Found Message Title & ¶ &
    $No Record Found Message Body
]
~~~

---

# 7. Variable Semantics and Allowed Formats

This section defines exact meaning and format rules.

## 7.1 `$$App Section`
String. The human-friendly title of the current section.

Examples:
- `"Staff List"`
- `"Job Titles List"`
- `"Department List"`

## 7.2 `$SQL`
String. SQL SELECT statement.

Rules:
- Must start with `SELECT`
- Must select `ID` first
- Must produce the dataset needed by the list
- If list is categorized, SQL must include the category field in the selected fields even if it is not displayed

Example pattern:
~~~sql
SELECT
  t1.ID,
  t1.Name,
  COUNT(t3.ID) AS TotalChildRecords
FROM SomeTable t1
LEFT JOIN ChildTable t3 ON t1.ID = t3."Parent ID"
GROUP BY t1.ID, t1.Name
~~~

## 7.3 `$SQL Condition`
This variable is used as a condition value to influence list output.

Observed usages:
- Simple list: `1`
- Conditional list by layout: `Case ( $$Layout Name = ... ; ... ; )`

Format rules:
- Can be a number or text
- Must be a single scalar value (no line breaks)

Important:
- The way this condition is applied inside the renderer is not shown in samples. Treat it as a required pipe variable that the renderer expects, even if it is `1`.

## 7.4 `$Column Headers`
Comma-separated list of column header labels.

Rules:
- Must include `S/N` as the first label
- Must include `Actions` as the last label
- Labels must match the displayed columns
- If categorization is enabled, the category field must NOT appear in headers (it is hidden but returned by SQL)
- If status is displayed, `Status` must be the last data header before `Actions`
- The generator must avoid overly cramped tables (see Section 8.1)

Examples:
- `"S/N,Staff #,Full Name,Gender,Job Title,Department,Status,Actions"`
- `"S/N,Job Title,Department,Total Staff,Actions"`

Categorization example (category returned in SQL but hidden in headers):
- SQL selects: `ID, Name, Role, Category`
- Headers: `"S/N,Name,Role,Actions"`

## 7.5 `$Column Widths` (New Renderer)
Comma-separated numeric percentages or proportional widths.

Rules:
- Count must match the number of headers
- Include widths for `S/N` and `Actions`
- Use widths to prevent cramped columns on a 13-inch screen with a side menu

Examples:
- `"5,10,20,10,20,20,10,5"`
- `"5,50,20,20,5"`

If not used, pass an empty string `""` only if the renderer allows it.

## 7.6 `$Status Column`
Index of the status column in the displayed list, using 1-based indexing aligned to `Column Headers`.

Rules:
- Only set this if status is displayed
- If set, status must be the last data column before `Actions`

Examples:
- `"S/N,Name,Role,Status,Actions"` => `$Status Column = 4`
- If no status column exists: `""` (empty string)

## 7.7 `$Date Columns`
Comma-separated indexes (1-based) of columns that should be formatted as dates.

Examples:
- InPatients list: `"2"`
- No date columns: `""`

## 7.8 `$Number Columns`
Comma-separated indexes (1-based) of columns that should be formatted as numbers.

Examples:
- `""` if none
- `"4,7"` if columns 4 and 7 are numeric

## 7.9 `$Categorization Column` (New Renderer)
A single index (1-based) or empty string.

Purpose:
- Used to group or categorize rows by a column value (example: group by date, group by industry).

Critical rule (hidden category field):
- If categorization is enabled, the category value must be present in the SQL selected fields
- The category value should not be included in `$Column Headers` (hidden from the UI)
- `$Categorization Column` should point to the category field position in the SQL resultset (see Open Questions for exact index-space confirmation)

Examples:
- SQL selects: `ID, AdmissionDate, PatientName, Ward`
  - Group by AdmissionDate => `$Categorization Column = 2`
- SQL selects: `ID, Name, Role, Industry`
  - Headers: `"S/N,Name,Role,Actions"`
  - Group by Industry => `$Categorization Column = 4`

No grouping:
- `$Categorization Column = ""`

## 7.10 `$Searchable Columns`
Comma-separated indexes (1-based) that are included in search filtering.

Examples:
- `"1,2,3"`
- `"1,2,3,4,5,6"`

Note:
- Index meanings must match the renderer's internal mapping. In samples, it appears to refer to displayed columns excluding `Actions`. Confirm in Open Questions section.

## 7.11 `$Add New Record Script Name`
Text. Script name to run when user clicks the "Add New" button.

Examples:
- `"Add New Staff"`
- `""` if no add button is needed

## 7.12 `$View Record Script Name`
Text. Script name to run when user clicks a row or a "View" action.

Examples:
- `"+++ Staff Details Page"`
- Conditional: `Case ( $$Layout Name = ... ; "+++ X Details Page" ; ... )`

## 7.13 `$Search Field Placeholder`
Text. Placeholder inside search field.

Examples:
- `"Search staff list"`
- Conditional by layout: `Case ( $$Layout Name = ... ; "Search patients on admission" ; ... )`

## 7.14 `$Total Records Label`
Text. Label next to total record count.

Examples:
- `"Total Staff"`
- `"Business Types"`
- Conditional by layout: `"Discharged Patients"`

## 7.15 `$Add New Record Button Label`
Text. Label of add button.

Examples:
- `"Add New Staff Member"`
- `""` if no add button

## 7.16 Empty-state Content
Used when there are no records.

Supported types observed:
- SVG path string: `$No Record Found Icon SVG Path`
- External image URL: `$No Record Found Icon URL`

Messages observed:
- `$No Record Found Message Title`
- `$No Record Found Message Body`

Rules:
- Provide at least icon + title + body unless the UI intentionally hides them
- Keep title short
- Body should be one sentence

---

# 8. Build Procedure (AI Agent Workflow)

Follow this exact sequence to build a new entity list page script.

## Step 1: Identify the Entity and Its Dataset
Collect:
- Table name (example: `Departments`)
- Primary key field (assumed: `ID`)
- Core display fields (example: `Name`)
- Related joins needed for display (example: HOD Staff name, Department counts)

## Step 2: Decide the Visible Columns
Define the list columns in human terms:
- Example: Department Name, Head of Department, Total Staff

Then map each column to a SQL expression.

Rules to apply:
- If status will be displayed, place it as the last data column (right before Actions)
- If categorization will be used, decide the category field and include it in SQL, but do not display it as a column

## Step 3: Build the SQL
Rules recap:
- First select: `t1.ID`
- Use left joins where needed
- Use group by when using count
- Use aliases for readability
- If categorization is enabled: include the category field in `SELECT` (even though it will be hidden)
- If status is displayed: ensure the status expression is selected such that it becomes the last visible data column in the UI

## Step 4: Create Column Headers
Create a comma-separated string:
- First label: `S/N`
- Last label: `Actions`
- Middle labels match your visible data columns

Rules to apply:
- If status is displayed, `Status` must be the last header before `Actions`
- If categorization is enabled, do not include the category label in headers

## Step 5: Define Optional Column Widths (New Renderer)
Create widths string with same count as headers.

## Step 6: Define Column Behaviors
Set:
- `$Status Column` or empty
- `$Date Columns` or empty
- `$Number Columns` or empty
- `$Categorization Column` or empty
- `$Searchable Columns` indexes

Reminder:
- `$Status Column` index must point to the status header, and status must be the last data header before Actions
- `$Categorization Column` index must point to the category field in the SQL resultset even if that category is not displayed in headers

## Step 7: Define Scripts and Labels
Set:
- `$Add New Record Script Name`
- `$View Record Script Name`
- `$Search Field Placeholder`
- `$Total Records Label`
- `$Add New Record Button Label`

## Step 8: Define Empty-state
Set:
- icon (SVG path or URL)
- title
- body

## Step 9: Call Renderer Script
Choose one:
- New: `+++ Display List HTML`
- Legacy: `+++ Display List HTML_`

Pass parameters in the exact order.

## Step 10: Set Post-render UI State
At minimum:
- `Set Variable [ $$Tab To Show ; Value: 1 ]`

Some modules reset additional globals. Do so only when the module requires it.

---

# 8.1 Column Count and Layout Density Rule (Anti-Cramping Rule)

The displayed list must not feel cramped.

Context:
- The list page is mostly displayed with a **side menu**
- The typical viewing device is a **13-inch screen**

Rule:
- As a general target, the number of **displayed columns** (data columns) should be around **3 to 5**.
- The generator may choose **3, 4, 5**, and in rare cases **6** columns if the data still reads cleanly.
- The generator may choose fewer columns if the fields are long (example: long names, long descriptions).
- The generator must use discretion to prevent tight column widths and unreadable rows.

Important:
- This is a discretion rule, not a hard limit.
- Prioritize readability over "show everything."
- If there are many useful fields, choose the best few for the list and keep the rest for the Details Page.
- Use `$Column Widths` (new renderer) to help protect readability.

Practical heuristics (apply common sense):
- If 1 or more columns will contain long text, reduce the total number of displayed columns.
- Prefer short, high-signal columns in the list (example: Name, Code, Category label if displayed, Key count metric).
- Defer long or secondary fields to the Details Page.

---

# 9. Canonical Templates

## 9.1 New Renderer Template (Recommended)

~~~filemaker
# +++ <Entity Name> List Page

Set Variable [ $$App Section ; Value: "<Entity Name> List" ]

Set Variable [ $SQL ; Value:
"SELECT
  t1.ID,
  <field_expr_1>,
  <field_expr_2>,
  <optional_hidden_category_expr>,
  <optional_status_expr>
FROM <TableName> t1
<joins>
<optional_where_or_grouping>
"
]

Set Variable [ $SQL Condition ; Value: 1 ]

Set Variable [ $Column Headers ; Value: "S/N,<Header 1>,<Header 2>,<optional Status>,Actions" ]
Set Variable [ $Column Widths ; Value: "<w1>,<w2>,<w3>,<w4>,<w5>" ]

Set Variable [ $Status Column ; Value: "" ]
Set Variable [ $Date Columns ; Value: "" ]
Set Variable [ $Number Columns ; Value: "" ]
Set Variable [ $Categorization Column ; Value: "" ]

Set Variable [ $Searchable Columns ; Value: "1,2,3" ]

Set Variable [ $Add New Record Script Name ; Value: "Add New <Entity Name>" ]
Set Variable [ $View Record Script Name ; Value: "+++ <Entity Name> Details Page" ]

Set Variable [ $Search Field Placeholder ; Value: "Search <entity plural> list" ]
Set Variable [ $Total Records Label ; Value: "Total <Entity Plural>" ]
Set Variable [ $Add New Record Button Label ; Value: "Add New <Entity Name>" ]

Set Variable [ $No Record Found Icon ; Value: "<svg_path_or_url>" ]
Set Variable [ $No Record Found Message Title ; Value: "No <Entity Plural>" ]
Set Variable [ $No Record Found Message Body ; Value: "There are no <entity plural> set up." ]

Perform Script [ Specified: From list ; “+++ Display List HTML” ;
  Parameter:
    $SQL & ¶ &
    $SQL Condition & ¶ &
    $Column Headers & ¶ &
    $Column Widths & ¶ &
    $Status Column & ¶ &
    $Date Columns & ¶ &
    $Number Columns & ¶ &
    $Categorization Column & ¶ &
    $Searchable Columns & ¶ &
    $Add New Record Script Name & ¶ &
    $View Record Script Name & ¶ &
    $Search Field Placeholder & ¶ &
    $Total Records Label & ¶ &
    $Add New Record Button Label & ¶ &
    $No Record Found Icon & ¶ &
    $No Record Found Message Title & ¶ &
    $No Record Found Message Body
]

Set Variable [ $$Tab To Show ; Value: 1 ]
~~~

## 9.2 Legacy Renderer Template

~~~filemaker
# +++ <Entity Name> List Page (Legacy Renderer)

Set Variable [ $$App Section ; Value: "<Entity Name> List" ]

Set Variable [ $SQL ; Value:
"SELECT
  t1.ID,
  <field_expr_1>,
  <field_expr_2>
FROM <TableName> t1
<joins>
"
]

Set Variable [ $SQL Condition ; Value: 1 ]

Set Variable [ $Column Headers ; Value: "S/N,<Header 1>,<Header 2>,Actions" ]

Set Variable [ $Status Column ; Value: "" ]
Set Variable [ $Date Columns ; Value: "" ]
Set Variable [ $Number Columns ; Value: "" ]

Set Variable [ $Searchable Columns ; Value: "1,2" ]

Set Variable [ $Add New Record Script Name ; Value: "Add New <Entity Name>" ]
Set Variable [ $View Record Script Name ; Value: "+++ <Entity Name> Details Page" ]

Set Variable [ $Search Field Placeholder ; Value: "Search <entity plural> list" ]
Set Variable [ $Total Records Label ; Value: "Total <Entity Plural>" ]
Set Variable [ $Add New Record Button Label ; Value: "Add New <Entity Name>" ]

Set Variable [ $No Record Found Icon SVG Path ; Value: "<svg_path>" ]
Set Variable [ $No Record Found Message Title ; Value: "No <Entity Plural>" ]
Set Variable [ $No Record Found Message Body ; Value: "There are no <entity plural> set up." ]

Perform Script [ Specified: From list ; “+++ Display List HTML_” ;
  Parameter:
    $SQL & ¶ &
    $SQL Condition & ¶ &
    $Column Headers & ¶ &
    $Status Column & ¶ &
    $Date Columns & ¶ &
    $Number Columns & ¶ &
    $Searchable Columns & ¶ &
    $Add New Record Script Name & ¶ &
    $View Record Script Name & ¶ &
    $Search Field Placeholder & ¶ &
    $Total Records Label & ¶ &
    $Add New Record Button Label & ¶ &
    $No Record Found Icon SVG Path & ¶ &
    $No Record Found Message Title & ¶ &
    $No Record Found Message Body
]

Set Variable [ $$Tab To Show ; Value: 1 ]
~~~

---

# 10. Implementation Rules for Column Indexing (AI Agent Must Apply)

This section defines how to compute column indexes for:
- Status column
- Date columns
- Number columns
- Searchable columns
- Categorization column

## 10.1 Use 1-Based Indexing Aligned to Column Headers (Displayed Columns)
Indexes refer to positions in `$Column Headers` for:
- `$Status Column`
- `$Date Columns`
- `$Number Columns`
- `$Searchable Columns` (pending renderer confirmation)

Example:
`"S/N,Staff #,Full Name,Gender,Job Title,Department,Status,Actions"`

Indices:
1 = S/N  
2 = Staff #  
3 = Full Name  
4 = Gender  
5 = Job Title  
6 = Department  
7 = Status  
8 = Actions  

So:
- `$Status Column = 7`

Rule reinforcement:
- If status is used, it must be the last data column, so it should appear immediately before `Actions`.

## 10.2 Categorization Column Indexing (SQL Resultset Position, Hidden Allowed)
For categorization, the index may refer to the SQL resultset position (including hidden category fields).

Rule:
- If the list is categorized, include the category field in the SQL selected fields.
- Do not include the category in `$Column Headers`.
- Set `$Categorization Column` to the **position of the category field within the SQL selected fields**.

Example:
SQL selects:
1) `ID`
2) `Name`
3) `Role`
4) `Category`

Headers (category hidden):
- `"S/N,Name,Role,Actions"`

Categorization:
- `$Categorization Column = 4`

Important:
- This matches observed behavior in samples where `$Categorization Column` can appear to exceed the number of visible data columns. Final confirmation depends on how the renderer interprets index space. See Open Questions.

---

# 13. AI Agent Verification Checklist (Must Pass)

Before you consider the List Page script complete, verify all items:

## 13.1 SQL Checks
- [ ] First selected column is ID
- [ ] SQL returns all required display fields
- [ ] Aggregations have a valid GROUP BY
- [ ] Field names with spaces are quoted correctly
- [ ] Joins are correct and do not create row explosion
- [ ] If categorization is enabled: SQL includes category field in selected fields even if it is hidden from UI
- [ ] If status is displayed: status is positioned as the last data column before Actions

## 13.2 UI Columns Checks
- [ ] Column headers include `S/N` first and `Actions` last
- [ ] Header count equals widths count (if widths provided)
- [ ] If status is displayed: `Status` appears as the last header before `Actions`
- [ ] Status column index exists in headers (if provided)
- [ ] Date column indices exist in headers (if provided)
- [ ] Searchable columns indices exist in headers
- [ ] If categorization is enabled: category is NOT in headers, but categorization column index points to the correct SQL selected-field position (including hidden field)
- [ ] Column density looks readable on a 13-inch screen with a side menu (not cramped)

## 13.3 Script Routing Checks
- [ ] Add script name exists, or is intentionally empty
- [ ] View script name exists and is correct
- [ ] Search placeholder is meaningful
- [ ] Total label matches entity plural

## 13.4 Empty State Checks
- [ ] Icon is set
- [ ] Title is set
- [ ] Body is set

## 13.5 Renderer Signature Checks
- [ ] You are calling `+++ Display List HTML` with 17 parameters, or
- [ ] You are calling `+++ Display List HTML_` with 15 parameters
- [ ] The parameter order matches the selected renderer

---

# 14. Open Questions, Ambiguities, Contradictions (Needs Your Clarification)

These items must be clarified to make the AI training perfect. Please answer them and the spec will be updated.

## Q1: Which renderer is canonical?
Samples use both:
- `+++ Display List HTML`
- `+++ Display List HTML_`

Questions:
- Is `+++ Display List HTML_` deprecated?
- Should all list pages be migrated to `+++ Display List HTML`?
- If both are valid, what is the decision rule for choosing one?

## Q2: What is the exact meaning and usage of `$SQL Condition` inside the renderer?
Observed:
- Many lists set it to `1`
- InPatients sets it to values like `"On Admission"` and `"Discharged"`

Questions:
- Does the renderer append a WHERE clause using this value?
- Does the renderer do a Substitute into `$SQL` using this value?
- Is `$SQL Condition` used for filtering after query results are returned?
- What is the exact token or mechanism used by the renderer?

## Q3: How does the renderer interpret column indexes?
Observed:
- `$Status Column = 7` in Staff list aligns to headers including `S/N`
- `$Searchable Columns` includes `"1,2,3"` etc

Questions:
- Do indexes include `S/N` and `Actions`, or exclude them?
- Does the renderer internally shift indexes because ID is hidden?
- Is the index space based on displayed headers, or based on SQL result columns excluding ID?

## Q4: Categorization Column index space (SQL resultset vs headers)
Observed pattern you want:
- SQL selects category field
- Headers hide category field
- `$Categorization Column` points to the category field position in the SQL recordset

Questions:
- Confirm: is `$Categorization Column` based on SQL selected-field position (including hidden category field)?
- If yes: does the position count include the ID column as position 1?
- If no: what is the exact index-space rule?

## Q5: Empty state icon type (SVG path vs URL)
Observed:
- Most lists use `$No Record Found Icon SVG Path`
- Business Types uses `$No Record Found Icon URL`

Questions:
- Does `+++ Display List HTML` accept both SVG path and URL in the same parameter slot?
- If yes, how does it detect which one it is?
- If no, which one is the standard for the whole app?

## Q6: Column Widths required or optional for new renderer
Observed:
- Staff, Business Types, InPatients use widths
- Departments and Job Titles do not, and they also use legacy renderer

Questions:
- Is `$Column Widths` mandatory for `+++ Display List HTML`?
- If optional, what is the correct fallback behavior?

## Q7: Actions column behavior
Questions:
- Does the renderer always add Actions column UI automatically?
- Does it rely on the last header being literally `Actions`?
- What actions are included by default (View, Edit, Delete), and how are they configured?

---

# CauferoAppStarter Documentation
## Generating a Details Page (Entity Details UI in WebViewer)

This documentation trains an AI Agent to generate FileMaker scripts that build **details pages** in the CauferoAppStarter style.

A details page is a WebViewer-driven page that:
- Receives a selected record ID (and optional context parameters)
- Fetches record data (usually via `ExecuteSQL`)
- Optionally fetches dropdown data and related subtables
- Builds CSS, HTML, and JavaScript as strings
- Writes a complete HTML document to `$$Page`
- Uses buttons and UI actions that call FileMaker scripts via `FileMaker.PerformScript(...)`

This doc focuses on **how the page is structured and generated**.
Other docs will cover:
- Specific HTML objects (maps, KPI cards, graphs, accordions, segmented buttons, subtables)
- Reusable JavaScript utilities and widgets

---

## 1) Core Outcomes the AI Agent Must Produce

When asked to create a new details page script, the AI Agent must generate a FileMaker script that:

1. Sets global navigation context
   - `$$App Section`
   - `$$Selected Record ID`
   - Entity-specific globals like `$$Department ID`, `$$Staff ID`, `$$Leave Request ID`
   - Optional globals like `$$Search Item`, `$$Tab To Show`

2. Defines FileMaker scripts to be called from the page
   - Save script name
   - Cancel script name
   - Other action scripts (modals, insert image, view related record, add document, fetch attendance, etc.)

3. Fetches entity data
   - Prefer `ExecuteSQL` with a clear SELECT list
   - Parse the SQL result into variables
   - Apply defaults when record does not exist (new record cases)

4. Fetches dropdown data (value lists)
   - Use `Perform Script [ "Get Value List" ; Parameter: ... ]`
   - Store `Get ( ScriptResult )` into a variable used inside HTML or JS

5. Fetches related lists (subtables)
   - Use `Perform Script [ "+++ Display Subtable HTML" ; Parameter: ... ]` and store HTML
   - For documents: `+++ Display Documents Subtable HTML` where required
   - Insert subtable HTML into tabs or sections

6. Builds the page
   - CSS: `Perform Script [ "🖌️ Use Details CSS" ]` then optionally append extra CSS
   - HTML sections:
     - Summary section (left column feel)
     - Content section (tabs + tab bodies, or single body)
   - JavaScript functions:
     - `showContent`, `cancelAction`, `saveItemInfo`
     - Support functions for widgets used in page (toggle, segmented control, datepicker, modal open, etc.)

7. Produces full HTML document and assigns to `$$Page`
   - Full `<!DOCTYPE html> ... </html>`
   - Includes Font Awesome
   - Includes Flatpickr when datepickers are used
   - Includes Chart.js when charts are used

8. Ends with standard refresh behavior
   - `Perform Script [ "WebDirect Pause" ]`
   - `Refresh Window [ Flush cached join results ; Flush cached external data ]`

---

## 2) Details Page Script Layout (Canonical Order)

A CauferoAppStarter details page script must follow this order.

### 2.1) Page identity and primary record selection

Required at the top:

~~~filemaker
Set Variable [ $$App Section ; Value: "<Human-readable page title>" ]
Set Variable [ $$Selected Record ID ; Value: GetValue ( GetScriptParameters ; 1 ) ]
~~~

Then entity-specific globals:

~~~filemaker
Set Variable [ $$<Entity> ID ; Value: GetValue ( GetScriptParameters ; 1 ) ]
Set Variable [ $$Search Item ; Value: GetValue ( GetScriptParameters ; 2 ) ]
~~~

Notes:
- Parameter 1 is the selected record ID for almost every details page.
- Parameter 2 is frequently used as a search context value when returning to list pages.

Optional additional parameters:
- Date range filters
- Tab to show
- Related record IDs for modal detail pages

Example pattern (Staff page):

~~~filemaker
Set Variable [ $From Date ; Value: DefaultIfEmpty ( GetValue ( GetScriptParameters ; 3 ) ; Get ( CurrentDate ) ) ]
Set Variable [ $To Date ; Value: DefaultIfEmpty ( GetValue ( GetScriptParameters ; 4 ) ; Get ( CurrentDate ) ) ]
~~~

### 2.2) Define FileMaker scripts called by the page

Every major button or row action in the HTML must map to a FileMaker script name via a variable.

Typical set:

~~~filemaker
Set Variable [ $Save Record Script ; Value: "Save <Entity>" ]
Set Variable [ $Cancel Script ; Value: "+++ <Entity> List Page" ]
Set Variable [ $Delete Record Script ; Value: "" ]
~~~

Optional additional action scripts (examples from samples):
- Insert image script
- Open modal script
- View child record modal script
- Add document
- Get attendance
- Grant system access

Important rule:
- The JS functions must call `FileMaker.PerformScript('<script name>', parameters)` using these variables.
- Script variables should be defined even if empty to avoid referencing undefined variables.

### 2.3) Fetch the entity record data (usually via ExecuteSQL)

#### 2.3.1) SQL construction standard

Preferred approach:
- Build a SQL string with explicit field list
- Use `?` placeholder(s) for parameters
- Use `ExecuteSQL ( sql ; fieldseparator ; rowseparator ; id )`

Standard separators:
- `fieldseparator = "|"`
- `rowseparator = "¶"`

Example:

~~~filemaker
Set Variable [ $List ; Value:
Let (
[
  sql = "SELECT t1.\"Name\", t1.\"Description\" FROM \"Departments\" t1 WHERE t1.ID = ?" ;
  fieldseparator = "|" ;
  rowseparator = "¶" ;
  id = $$Department ID ;
  result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; id )
] ;
result
)
]
~~~

#### 2.3.2) Parsing pattern

After `ExecuteSQL`, parse like this:

~~~filemaker
Set Variable [ $Total ; Value: ValueCount ( $List ) ]

If [ $Total > 0 ]
  Loop
    Set Variable [ $i ; Value: $i + 1 ]
    Set Variable [ $Link Record ; Value: GetValue ( $List ; $i ) ]
    Set Variable [ $Link Record As List ; Value: Substitute ( $Link Record ; "|" ; ¶ ) ]

    Set Variable [ $Field1 ; Value: GetValue ( $Link Record As List ; 1 ) ]
    Set Variable [ $Field2 ; Value: GetValue ( $Link Record As List ; 2 ) ]

    Exit Loop If [ $i ≥ $Total ]
  End Loop
Else
  /* defaults for new record case */
End If
~~~

Notes:
- Even when a single row is expected, the scripts still use a loop. Keep that style.
- Long text fields should normalize line breaks:
  - `ConvertLineBreakMarkersToReturns ( GetValue ( ... ) )`

#### 2.3.3) Defaults logic

If no record found:
- Set smart defaults that allow creating a new record
- Or leave blank if your design does not support "new record from details page"

Examples:
- Staff: default nationality and status
- Leave request: default dates, staff identity, duration, status

The AI Agent must implement defaults only if the page supports creating new records.

### 2.4) Fetch value lists (dropdowns and JS dictionaries)

Use the dedicated value list script.

#### 2.4.1) Dropdown HTML options

Pattern:

~~~filemaker
Perform Script [ “Get Value List” ; Parameter: "Dropdown|<List Name>|" & <SelectedValueOrID> ]
Set Variable [ $<ListVarName> Dropdown ; Value: Get ( ScriptResult ) ]
~~~

Examples:
- `"Dropdown|Leave Types|" & $Leave Type ID`
- `"Dropdown|Staff|" & $HOD Staff ID`
- `"Dropdown|Nationalities|" & $Nationality`

The returned ScriptResult is HTML that can be inserted directly inside `<select> ... </select>`.

#### 2.4.2) JS dictionary data for dependent dropdowns

Pattern:

~~~filemaker
Perform Script [ “Get Value List” ; Parameter: "JS|<List Name>|" & <SelectedValue> ]
Set Variable [ $<ListVarName> JS ; Value: Get ( ScriptResult ) ]
~~~

In Staff page, these JS datasets drive cascading dropdowns:
- Departments
- Job Titles
- Notches

The JS is later injected like:

~~~javascript
const departments = { <ScriptResult> };
~~~

---

## 3) Related Lists: Subtables (Embedded Related Data)

Details pages often include related lists such as:
- Job Titles in a department
- Staff in a department
- Emergency contacts for a staff member
- Education, payroll, attendance, KPIs, documents

These are rendered as a reusable component that returns HTML.

### 3.1) Subtable generation: Required parameters

Before calling the subtable script, set a block of variables.

Minimum required:
- `$SQL`
- `$SQL Condition`
- `$Column Headers`
- `$Column Widths`
- `$Table ID`
- `$Sub Table Name`
- `$Sub Table Width`
- `$Sub Table Height`
- No-record icon path, title, body

Often used optional parameters:
- `$Status Column` (index of status column)
- `$Date Columns` (indexes)
- `$Number Columns` (indexes)
- `$Inputs` (input field configuration)
- `$Row Action JS Script`
- `$Row Action Icon Class`
- `$Add New Record Button Label`
- `$Add New Record JS Script Name`
- `$Two Decimal Columns`
- `$Timestamp Columns`

Then call:

~~~filemaker
Perform Script [ “+++ Display Subtable HTML” ; Parameter:
  $SQL & ¶ &
  $SQL Condition & ¶ &
  $Column Headers & ¶ &
  $Column Widths & ¶ &
  $Status Column & ¶ &
  $Date Columns & ¶ &
  $Number Columns & ¶ &
  $Inputs & ¶ &
  $Row Action JS Script & ¶ &
  $Row Action Icon Class & ¶ &
  $Table ID & ¶ &
  $Two Decimal Columns & ¶ &
  $Timestamp Columns & ¶ &
  $Sub Table Name & ¶ &
  $Sub Table Width & ¶ &
  $Sub Table Height & ¶ &
  $Add New Record Button Label & ¶ &
  $Add New Record JS Script Name & ¶ &
  $No Record Found Icon SVG Path & ¶ &
  $No Record Found Message Title & ¶ &
  $No Record Found Message Body
]
Set Variable [ $<SubTableVarName> ; Value: Get ( ScriptResult ) ]
~~~

### 3.2) Documents subtable

For documents, your pattern uses:

- Script: `+++ Display Documents Subtable HTML`

The parameter list is the same shape as the normal subtable, but the renderer is specialized for documents.

### 3.3) Inserting subtables into the page

Subtable HTML is inserted in a tab body or content block like:

~~~html
<div id='tab3' class='content'>
  " & $Education Sub Table HTML & "
</div>
~~~

---

## 4) CSS Generation

### 4.1) Details CSS is the base for all details pages

Always call:

~~~filemaker
Perform Script [ “🖌️ Use Details CSS” ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]
~~~

### 4.2) Optional: Append extra CSS

Some pages add extra rules for special widgets (charts, dashboard cards):

~~~filemaker
Set Variable [ $styles ; Value:
  $styles &
  "
    #chart-container { width: 100%; height: 200px; padding: 10px; box-sizing: border-box; }
    canvas { width: 100% !important; height: 100% !important; }
  "
]
~~~

### 4.3) Optional: Merge multiple CSS packs

Example: Details CSS + Dashboard CSS

~~~filemaker
Perform Script [ “🖌️ Use Dashboard CSS” ]
Set Variable [ $Dashboard Styles ; Value: Get ( ScriptResult ) ]

Set Variable [ $styles ; Value:
"
<style> " &
  $styles & " " & $Dashboard Styles &
" </style>
"
]
~~~

Important:
- Decide whether `$styles` should hold raw CSS only, or a `<style>...</style>` wrapper.
- Be consistent within one script. If you wrap with `<style>`, do not wrap again in the final HTML head.

---

## 5) HTML Generation: The Standard Page Structure

All details pages follow this high-level structure:

~~~html
<div class='container'>
  <div class='summary-section'> ... </div>
  <div class='content-section'>
    <!-- tabs optional -->
    <!-- tab bodies or single content -->
  </div>
</div>
~~~

### 5.1) Summary section

The summary section is built as `$Summary Section HTML` and usually includes:

- Title `<h2>`
- Summary metrics via `<p><strong>Label:</strong> value</p>`
- Optional photo block
- Divider `<hr />` blocks
- Button container with save/cancel or other actions

Common patterns:
- Phone link: `<a href='tel:...'>`
- Email link: `<a href='mailto:...'>`

Example pattern:

~~~filemaker
Set Variable [ $Summary Section HTML ; Value:
"
<div class='summary-section'>
 <div class='summary-card'>
  <h2>" & If ( not IsEmpty ( $Name ) ; $Name ; "Entity Information" ) & "</h2>
  <div class='summary-content'>
   <div class='summary-info'>
     <p><strong>Field:</strong> " & $Value & "</p>
   </div>
  </div>
  <hr />
  <div class='button-container'>
    <button class='save-button' onclick=\"saveItemInfo()\"><i class='fas fa-save'></i> Save</button>
    <button class='cancel-button' onclick=\"cancelAction()\"><i class='fas fa-times-circle'></i> Cancel</button>
  </div>
 </div>
</div>
"
]
~~~

### 5.2) Tabs block (optional)

Tabs exist when the entity has multiple sub-sections.

The tabs HTML must:
- Use `.tabs` wrapper
- Each tab uses `.tab`
- Active tab uses `.active`
- Each tab calls `showContent('tabX', this)`

~~~filemaker
Set Variable [ $Tabs HTML ; Value:
"
<div class='tabs'>
 <div class='tab" & If ( $$Tab To Show = 1 ; " active" ) & "' onclick=\"showContent('tab1', this)\">Details</div>
 <div class='tab" & If ( $$Tab To Show = 2 ; " active" ) & "' onclick=\"showContent('tab2', this)\">Other</div>
</div>
"
]
~~~

### 5.3) Tab body blocks (or single content block)

Each tab body:
- Uses `<div id='tabX' class='content'>`
- Shows only the selected tab on load
- The default show condition uses `$$Tab To Show`

~~~filemaker
Set Variable [ $Tab1 HTML ; Value:
"
<div id='tab1' class='content'" & If ( $$Tab To Show = 1 ; " style='display: block;'" ) & ">
 ...
</div>
"
]
~~~

If no tabs:
- You can skip the tabs HTML and place only one content block inside `.content-section`.

### 5.4) Form layout conventions

Your pages consistently use grid classes such as:
- `.form-grid1`
- `.form-grid2`
- `.form-grid3`
- `.form-group`
- `.form-group-2`

Common input patterns:
- `<input type='text' id='field_id' value='...'>`
- `<textarea id='field_id'>...</textarea>`
- `<select id='field_id'> ...options... </select>`

Line break policy:
- Textareas should receive line-break normalized text from FileMaker via `ConvertLineBreakMarkersToReturns(...)`.
- When sending textareas back to FileMaker, JS should encode: `encodeURIComponent(...)`.

### 5.5) Reusable UI objects seen across scripts

These show up repeatedly:
- Save button and Cancel button in the summary section
- Tabs + `showContent` function
- Toggle switch (checkbox + slider) with a status text element
- Segmented control buttons
- Date inputs with calendar icon using Flatpickr
- Subtable HTML blocks inserted in tabs

Other objects appear in specific pages (photo modal, KPI cards, charts). Mention them at the layout level only in this doc.

---

## 6) JavaScript Generation: Required Functions and Assembly

### 6.1) JS functions are built as strings in FileMaker variables

Pattern:

~~~filemaker
Set Variable [ $Cancel Action Script ; Value:
"function cancelAction() {
  FileMaker.PerformScript('" & $Cancel Script & "');
}"
]
~~~

Then assemble:

~~~filemaker
Set Variable [ $Scripts ; Value:
"<script> " &
  $Show Content Script & "¶¶" &
  $Cancel Action Script & "¶¶" &
  $Save Item Info Script &
" </script> "
]
~~~

Important:
- Use `¶¶` to separate blocks for readability.
- Only include scripts that are defined or guaranteed to exist.
- If you keep a standard library of scripts like `$Wizard Script`, `$Toggle Accordion Script`, define them or confirm they exist globally.

### 6.2) Required baseline functions

#### 6.2.1) showContent (tabs)

~~~javascript
function showContent(contentId, tabElement) {
  const contentSections = document.querySelectorAll('.content');
  contentSections.forEach((section) => (section.style.display = 'none'));
  document.getElementById(contentId).style.display = 'block';

  const tabs = document.querySelectorAll('.tab');
  tabs.forEach((tab) => tab.classList.remove('active'));
  tabElement.classList.add('active');
}
~~~

#### 6.2.2) cancelAction

Calls the list page (or previous page) script:

~~~javascript
function cancelAction() {
  FileMaker.PerformScript('<Cancel Script Name>');
}
~~~

#### 6.2.3) saveItemInfo

Collect values from DOM, build pipe-separated parameter string, call save script.

Rules:
- Use `encodeURIComponent` for textareas and any free-text that may contain line breaks or pipes.
- Keep the order of parameters consistent with the save script expectation.
- Validate required fields before calling FileMaker.

Example shape:

~~~javascript
function saveItemInfo() {
  const name = document.getElementById('name').value;
  const description = encodeURIComponent(document.getElementById('description').value);

  const parameters = [ name, description ].join('|');
  FileMaker.PerformScript('<Save Script Name>', parameters);
}
~~~

### 6.3) Common supporting functions (include only if used by the page)

#### 6.3.1) Toggle switch

Used to show and change a status label.

DOM expectations:
- Checkbox: `id='toggle'`
- Label span: `id='status'`

Typical behavior:
- A `preselectToggle(...)` called on load
- A `toggle()` that updates the status text

#### 6.3.2) Segmented control

Used for binary or small options like gender.

DOM expectations:
- `.segmented-control button`
- The clicked button becomes `.active`

#### 6.3.3) Datepicker (Flatpickr)

When the page has date fields:
- Include Flatpickr CSS and JS in the HTML head
- Convert MySQL date strings to readable strings for display
- Provide helper `getMySQLFormattedDate(fieldId)` for saving

Core idea:
- Display: `l, j F, Y`
- Save: `Y-m-d`

#### 6.3.4) Modal open and close (image modal)

If the summary section shows a photo:
- Provide `openImageModal()` and `closeImageModal()`
- The modal uses `display: flex` and closes when clicking outside the image

#### 6.3.5) Related record viewers

Subtables often need row actions like "viewEducation", "viewDocument", "viewEmergencyContactDetails".
These functions take a parameter from the row and call a FileMaker script.

Example shape:

~~~javascript
function viewDocument(parameters) {
  FileMaker.PerformScript('<View Document Script>', parameters);
}
~~~

#### 6.3.6) Dynamic cascading dropdowns

When one dropdown depends on another (department -> job titles -> notches):
- Fetch JS dictionaries via `"Get Value List"` with `JS|...`
- Build dropdown options in JS on DOMContentLoaded
- Preselect based on existing IDs

---

## 7) Full Page Assembly into $$Page

### 7.1) HTML head includes

Minimum:
- meta charset, viewport
- Font Awesome CSS
- `<style> ... </style>` containing your `$styles`

Optional:
- Flatpickr CSS + JS if datepickers exist
- Chart.js script if charts exist

Example shape:

~~~filemaker
Set Variable [ $$Page ; Value:
"<!DOCTYPE html>
<html lang='en'>
<head>
  <meta charset='UTF-8'>
  <meta name='viewport' content='width=device-width, initial-scale=1.0'>
  <title>Item Management</title>
  <link rel='stylesheet' href='https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css'>
  <link rel='stylesheet' href='https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css'>
  <script src='https://cdn.jsdelivr.net/npm/flatpickr'></script>
  <style> " & $styles & " </style>
</head>
<body> " & $HTML & "¶¶" & $Scripts & " </body>
</html>"
]
~~~

### 7.2) HTML body uses your container block

Always set `$HTML` first:

~~~filemaker
Set Variable [ $HTML ; Value:
"
<div class='container'>
  " & $Summary Section HTML & "
  <div class='content-section'>
    " & $Tabs HTML & "
    " & $Tab1 HTML & "
    " & $Tab2 HTML & "
  </div>
</div>
"
]
~~~

Notes:
- If some tabs do not exist, keep their variables empty. FileMaker concatenation should safely ignore empty strings if variables are set to "".
- If you reference `$Tab4 HTML` etc, ensure they exist and are defined as empty when unused.

---

## 8) Page Actions and Parameter Conventions

### 8.1) Parameter passing convention

All actions that call FileMaker scripts use a single parameter string:
- Values joined by `|`
- Encoded where needed

Rules:
1. Use `encodeURIComponent` for:
   - Textareas
   - Any value that can contain line breaks
   - Any value that can contain the pipe character
2. Keep the ordering fixed and documented
3. Save scripts must parse using `GetValue ( Substitute ( $param ; "|" ; "¶" ) ; n )` or an equivalent approach

### 8.2) Record identity and page modes

Most pages operate in one of these modes:
- Existing record mode: ID exists, fields may be editable or partially locked
- New record mode: no record found, defaults applied, some fields enabled for entry

Your Leave Request page uses an important style:
- If an ID value is empty, treat it as a "new request" state.
- In that state:
  - Some fields are enabled
  - The main button label becomes "Submit Request"
  - The button action can route to `saveItemInfo()`

The AI Agent must implement this behavior when the page supports creating a record.

---

## 9) Common HTML Element IDs and Naming Rules

### 9.1) Element ID naming

Use snake_case IDs matching the semantic field name:
- `leave_type`, `start_date`, `end_date`, `reason`
- `name`, `hod`, `description`
- `staff_number`, `first_name`, `last_name`, `dob`, `department`, `job_title`

### 9.2) Icon ID naming for datepickers

For each date input:
- Input: `id='dob'`
- Icon: `id='dob_icon'`

Same pattern for:
- `start_date_icon`
- `end_date_icon`
- `from_date_icon`
- `to_date_icon`

### 9.3) Status and toggle IDs

Toggle switch requirements:
- checkbox: `id='toggle'`
- status text: `id='status'`

Avoid creating conflicting IDs across tabs. Keep IDs unique per page.

---

## 10) Reusable Script Includes Commonly Referenced

Across your samples, the `$Scripts` assembly references scripts that are not shown inside the snippet, for example:
- `$Toggle Accordion Script`
- `$Add Another Item Script`
- `$Close Alert Script`
- `$Move Item Script`
- `$Wizard Script`
- `$Choose File`

The AI Agent must follow this rule:
- If a details page references a script block in `$Scripts`, it must be defined in the script or guaranteed by your app framework.

If the app framework injects these as globals, the documentation for that framework must specify:
- Where they come from
- Their exact variable names
- Whether they are always present

---

## 11) Quality Checklist for the AI Agent

Before finalizing a generated details page script, the AI Agent must verify:

### 11.1) Variable completeness
- Every variable referenced later is set earlier.
- If optional components exist (tabs 4 to 9), define them as empty strings if unused.

### 11.2) SQL safety and correctness
- Correct table and field quoting with `\"Field\"` and `"Table Name"`
- Correct joins and group by when using aggregates
- Parameter placeholders `?` used for IDs

### 11.3) UI correctness
- All buttons have a matching JS function defined
- All JS functions that call FileMaker use the correct script variables
- IDs used in JS match IDs used in HTML

### 11.4) Encoding rules
- Textareas are encoded on save
- Any value containing special characters that can break the `|` join is encoded

### 11.5) Dependencies
- If datepickers exist, include Flatpickr CSS and JS
- If charts exist, include the chart library and the JS uses correct canvas IDs
- If font icons used, include Font Awesome

### 11.6) End-of-script behavior
- `WebDirect Pause`
- `Refresh Window [ Flush cached join results ; Flush cached external data ]`

---

## 12) Known Ambiguities and Possible Contradictions Found in the Samples

The AI Agent needs clarity on the items below because they affect correctness and repeatability.

### 12.1) Undefined script variables included in $Scripts
In multiple scripts, `$Scripts` concatenates variables that are not defined in the pasted script, including:
- `$Toggle Accordion Script`
- `$Add Another Item Script`
- `$Close Alert Script`
- `$Move Item Script`
- `$Wizard Script`
- `$Reset Account Script` (in Leave Request page there is a setter for it, but the variable name is used inside itself)
- `$Choose File` (Department page uses it, Staff page defines it later)

Question for you:
1) Are these defined in a shared global script that always runs before details pages?
2) If yes, what is the canonical list of standard JS blocks that every details page can include safely?

### 12.2) Leave Request page: button style condition looks wrong
This snippet appears inconsistent:

~~~filemaker
... & If ( not IsEmpty ( IsEmpty ( $Leave Type ID ) ) ; " style='width: 100%'" ) & ...
~~~

`IsEmpty ( IsEmpty ( $Leave Type ID ) )` produces a boolean inside `IsEmpty`, which is likely unintended.

Question for you:
- What is the correct intent here?
  - Is it: `If ( IsEmpty ( $Leave Type ID ) ; " style='width: 100%'" ; "" )`?

### 12.3) Leave Request toggle script references an element id that does not exist
In Leave Request page JS:

~~~javascript
const item = document.getElementById('item');
...
item.textContent = 'Approved';
~~~

But the HTML uses:

~~~html
<span class='status' id='status'>...</span>
~~~

Question for you:
- Should the JS always update `id='status'`?
- Is `id='item'` a leftover name from another page?

### 12.4) Staff page: toggle script references a missing FileMaker field placeholder
This appears in the staff toggle script:

~~~javascript
window.onload = function() {
  preselectToggle('" & <Table Missing>::<Field Missing> & "');
};
~~~

Question for you:
- What field controls staff active/inactive status for the toggle?
- What are the correct table and field names used in the real script?

### 12.5) New record detection rule differs by page
- Leave Request page uses empty `$Leave Type ID` as a signal of a new request state.
- Department page does not define defaults for no-record case.
- Staff page defines defaults for some fields.

Question for you:
- What is the standard rule for "new record mode" across your app starter?
  - Is it based on the entity ID being empty?
  - Or based on `ExecuteSQL` returning no rows?
  - Or based on a specific field being empty?

### 12.6) Duration field format in Leave Request saving
In Leave Request:
- Duration input is set as `"X Days"`
- The save function reads `duration = document.getElementById('duration').value;`
- That value includes the word "Days"

Question for you:
- Should the save script expect a pure number, or `"X Days"` text?

### 12.7) Date fields: mix of stored MySQL vs displayed readable strings
The scripts:
- Store dates as MySQL `Y-m-d` in variables
- Convert to readable in Flatpickr on load
- Convert back to MySQL via `getMySQLFormattedDate()` on save

Question for you:
- Are all date fields stored in the database as MySQL date strings, or are some stored as FileMaker dates?
- Is `FMPDateTextToMySQLDate()` always required before placing values into HTML?

---

## 13) Implementation Template the AI Agent Should Follow

When asked to create a new details page script, generate it using this template flow.

### 13.1) Script skeleton (high-level)

~~~filemaker
# 1) Page Identity
Set Variable [ $$App Section ; Value: "<Title>" ]
Set Variable [ $$Selected Record ID ; Value: GetValue ( GetScriptParameters ; 1 ) ]
Set Variable [ $$<Entity> ID ; Value: GetValue ( GetScriptParameters ; 1 ) ]
Set Variable [ $$Search Item ; Value: GetValue ( GetScriptParameters ; 2 ) ]

# 2) Action Scripts
Set Variable [ $Save Record Script ; Value: "Save <Entity>" ]
Set Variable [ $Cancel Script ; Value: "+++ <Entity> List Page" ]
Set Variable [ $Delete Record Script ; Value: "" ]
/* add more if needed */

# 3) Fetch Record Data (ExecuteSQL)
Set Variable [ $List ; Value: Let ( [ sql=... ; fieldseparator="|" ; rowseparator="¶" ; id=$$<Entity> ID ; result=ExecuteSQL(...) ] ; result ) ]
Set Variable [ $Total ; Value: ValueCount ( $List ) ]
If [ $Total > 0 ]
  Loop
    /* parse fields into variables */
    Exit Loop If [ $i ≥ $Total ]
  End Loop
Else
  /* defaults if needed */
End If

# 4) Fetch Value Lists (Dropdown and JS)
Perform Script [ “Get Value List” ; Parameter: "Dropdown|<List>|" & <Selected> ]
Set Variable [ $<DropdownVar> ; Value: Get ( ScriptResult ) ]
/* more lists as needed */

# 5) Generate Subtables
Set Variable [ $SQL ; Value: "SELECT ..." ]
/* set subtable meta vars */
Perform Script [ “+++ Display Subtable HTML” ; Parameter: ... ]
Set Variable [ $<SubTableVar> ; Value: Get ( ScriptResult ) ]
/* repeat as needed */

# 6) CSS
Perform Script [ “🖌️ Use Details CSS” ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]
/* optional extra CSS */

# 7) HTML blocks
Set Variable [ $Summary Section HTML ; Value: "..." ]
Set Variable [ $Tabs HTML ; Value: "..." ] /* optional */
Set Variable [ $Tab1 HTML ; Value: "..." ]
/* more tabs */
Set Variable [ $HTML ; Value: "..." ]

# 8) JS blocks
Set Variable [ $Show Content Script ; Value: "..." ] /* if tabs exist */
Set Variable [ $Cancel Action Script ; Value: "..." ]
Set Variable [ $Save Item Info Script ; Value: "..." ]
/* widget scripts as needed */
Set Variable [ $Scripts ; Value: "<script> " & ... & " </script> " ]

# 9) Full document
Set Variable [ $$Page ; Value: "<!DOCTYPE html>..." ]

# 10) End
Perform Script [ “WebDirect Pause” ]
Refresh Window [ Flush cached join results ; Flush cached external data ]
~~~

---

## 14) Minimum Deliverable: What a "Complete" Details Page Means

A details page script is complete only if:
- It can load without missing variables
- It can display existing record details accurately
- It can save changes via the Save script
- It can cancel back to a list page via the Cancel script
- It can render all required dropdowns and subtables
- It includes all required library references for widgets used

If a page supports creating new records from the details page, it is complete only if:
- Defaults are set properly
- Required fields are validated
- Save creates the record and returns to the correct place

---

## 15) What the AI Agent Must Ask You When It Lacks Required Info

If the AI Agent is missing any of these, it must request clarification:
- Entity table name and primary key field name used in SQL
- The list page script name for cancel action
- The save script name and expected parameter order
- Which tabs exist and what each tab contains
- Which value lists are required and whether they return Dropdown HTML or JS dictionaries
- Which related lists (subtables) exist and their SQL plus column specs
- Which UI widgets are required (datepickers, toggles, segmented control, charts, photo modal)
- Whether the page supports new record creation and what defaults apply

If you provide those, the AI Agent should be able to generate the complete script in your CauferoAppStarter style.

---

# Modal Page Generation in CauferoAppStarter (FileMaker + WebViewer HTML)

This documentation defines the exact pattern used in CauferoAppStarter to generate a modal page (a card window that pops up with details of a selected entity) using HTML, CSS, and JavaScript rendered in a FileMaker WebViewer.

The goal is to train an AI Agent to generate FileMaker scripts that match this pattern exactly.

---

## 1) What a Modal Page Is in CauferoAppStarter

A modal page in CauferoAppStarter is:

- A **FileMaker script** that:
  - Receives an entity ID or mode via Script Parameter(s)
  - Pulls required data with `ExecuteSQL`
  - Builds an HTML string (`$HTML`)
  - Builds JavaScript functions (`$Scripts`)
  - Gets base modal CSS from a shared script (`🖌️ Use Modal CSS`)
  - Assembles a full HTML document string into a global variable `$$Modal`
  - Opens a **Card** window to the **Modal** layout

- A **Modal layout** that:
  - Contains a WebViewer that loads the HTML stored in `$$Modal` (the loading mechanism is part of the app shell and is not repeated inside each modal generator script)

---

## 2) Core Architecture Pattern

Every modal generator script follows this pipeline:

1. **Read parameters**
2. **Define the target FileMaker scripts**
   - Save script name
   - Cancel script name
   - Optional scripts (delete, insert image, row action, administer medication, etc.)
3. **Fetch data**
   - `ExecuteSQL` using `fieldseparator="|"` and `rowseparator="¶"`
   - Parse the result by row and then by column
4. **Generate CSS**
   - `Perform Script [ "🖌️ Use Modal CSS" ]`
   - Optionally append extra CSS for custom controls
5. **Generate HTML**
   - `div.modal`
   - `div.modal-header` (title)
   - `div.modal-body` (form, cards, subtables, instructions)
   - `div.modal-footer` (Save and Cancel buttons)
6. **Generate JavaScript**
   - Always includes `cancelUpdates()`
   - Usually includes `saveUpdates()`
   - Optionally includes helpers (date picker, segmented control, dropdown search, editor logic, BMI calc, list selection)
7. **Assemble full HTML page**
   - `<!DOCTYPE html> ... <style> ... </style> ...`
   - External libraries included only if needed (Flatpickr, Font Awesome)
8. **Store as `$$Modal`**
9. **Open modal window (Card)**
   - Toolbars off, menu bar off, resize off, dim parent on

---

## 3) Standard Conventions and Required Variables

### 3.1 Script parameter handling

Two styles exist in the samples:

- `Get ( ScriptParameter )` for a single parameter
- `GetValue ( GetScriptParameters ; n )` for multi-part parameters

**Rule for AI Agent:**
- Use `Get ( ScriptParameter )` when exactly one value is passed.
- Use `GetValue ( GetScriptParameters ; n )` when multiple values are passed.

Examples:
~~~filemaker
Set Variable [ $$Diagnosis ID ; Value: Get ( ScriptParameter ) ]
Set Variable [ $$Mode ; Value: GetValue ( GetScriptParameters ; 1 ) ]
~~~

### 3.2 Modal control script names

Every modal generator script defines:
- A FileMaker script to save changes
- A FileMaker script to close the modal

Standard variable names used in the scripts:
- `$Save Record Script` or `$Save Updates Script` (string name of FileMaker script)
- `$Cancel Script` (string name of FileMaker script)

Examples:
~~~filemaker
Set Variable [ $Save Record Script ; Value: "Save Diagnosis Details" ]
Set Variable [ $Cancel Script ; Value: "Close Window" ]
~~~

**Important:**
There is a naming collision used in some scripts:
- `$Save Updates Script` is first used as the FileMaker script name
- Later reused as the JavaScript function text

**Rule for AI Agent:**
To avoid confusion, prefer this naming internally:
- `$SaveFM` for FileMaker script name
- `$CancelFM` for FileMaker script name
- `$JS_Save` for JavaScript save function code
- `$JS_Cancel` for JavaScript cancel function code
You can still output scripts in the legacy naming style if the target system expects it, yet keep your internal generation logic clean.

### 3.3 Global variable for modal HTML

All modal pages are stored into:
- `$$Modal`

Example:
~~~filemaker
Set Variable [ $$Modal ; Value: "<!DOCTYPE html> ... </html>" ]
~~~

### 3.4 Standard delimiters

- SQL field separator: `|`
- SQL row separator: `¶`
- JS parameters to FileMaker: `|` joined strings, or JSON string
- Special delimiters used in some complex saves:
  - `vvvvvvvvvv` between row payloads in subtables
  - Prefix labels like `ITEMS:` and `TREATMENT_CHART:` separated by `¶`

Examples:
~~~js
const parameters = [ a, b, c ].join('|');
FileMaker.PerformScript('Save Script Name', parameters);
~~~

~~~js
const tableDataString = updatedData.map(item => `${item.id}|${item.value}`).join('vvvvvvvvvv');
const combinedParameters = `ITEMS:${parameters}¶TREATMENT_CHART:${tableDataString}`;
FileMaker.PerformScript('Save Treatment Chart', combinedParameters);
~~~

---

## 4) Data Retrieval Pattern (ExecuteSQL + Parsing)

### 4.1 SQL select

Typical pattern:
~~~filemaker
Set Variable [ $List ;
  Value:
    Let (
      [
        sql = "SELECT \"Field1\", \"Field2\"
               FROM \"Table\"
               WHERE \"ID\" = ?" ;
        fieldseparator = "|" ;
        rowseparator = "¶" ;
        id = $$Some ID ;
        result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; id )
      ] ;
      result
    )
]
~~~

### 4.2 Parse result rows and columns

Typical pattern:
~~~filemaker
Set Variable [ $Total ; Value: ValueCount ( $List ) ]

If [ $Total > 0 ]
  Loop
    Set Variable [ $i ; Value: $i + 1 ]
    Set Variable [ $Row ; Value: GetValue ( $List ; $i ) ]
    Set Variable [ $RowAsList ; Value: Substitute ( $Row ; "|" ; ¶ ) ]

    Set Variable [ $Field1 ; Value: GetValue ( $RowAsList ; 1 ) ]
    Set Variable [ $Field2 ; Value: GetValue ( $RowAsList ; 2 ) ]

    Exit Loop If [ $i ≥ $Total ]
  End Loop
Else
  # Optional defaults
End If
~~~

### 4.3 Formatting and conversion helpers

These are frequently used:
- `ConvertLineBreakMarkersToReturns ( ... )` for text stored with markers
- `RelativeDateInWords ( ... )` for display-friendly dates
- `MySQLDateToFMPDateText ( ... )` to convert date formats (custom function)
- `DefaultIfEmpty ( x ; "fallback" )` (custom function)

**Rule for AI Agent:**
- Display formatting is handled before HTML generation.
- HTML receives already formatted strings except for interactive date pickers which may store MySQL dates and convert on load.

---

## 5) CSS Generation Pattern

### 5.1 Base modal CSS

Every modal uses:
~~~filemaker
Perform Script [ "🖌️ Use Modal CSS" ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]
~~~

### 5.2 Append extra CSS when needed

Some modals add custom controls and append CSS via:
~~~filemaker
Set Variable [ $styles ; Value: $styles & " ...extra css... " ]
~~~

Common appended CSS blocks in the samples:
- Searchable dropdown styling (`.dropdown-container`, `.dropdown-input`, `.dropdown-list`)
- Rich text editor styling (`.toolbar`, `#editor`)
- Custom cards or grids (vitals cards, KPI cards, etc.)

**Rule for AI Agent:**
- Use base CSS always.
- Append only the required CSS for the controls used in that modal.

---

## 6) HTML Structure Standard

### 6.1 Wrapper skeleton

All modals use:
~~~html
<div class='modal'>
  <div class='modal-header'>TITLE</div>

  <div class='modal-body'>
    <form id='update-form'>
      <!-- content -->
    </form>
  </div>

  <div class='modal-footer'>
    <button type='button' class='save-button' onclick="saveUpdates()">
      <i class='fas fa-save'></i>
      Save
    </button>
    <button type='button' class='cancel-button' onclick="cancelUpdates()">
      <i class='fas fa-times'></i>
      Cancel
    </button>
  </div>
</div>
~~~

### 6.2 Inputs and IDs

Inputs must have stable IDs because JS reads them by `document.getElementById(...)`.

Examples:
- `institution`, `programme`, `result`
- `start_date`, `end_date` and icon IDs `start_date_icon`, `end_date_icon`
- `height`, `weight`, `bmi`, `temperature`, etc.

**Rule for AI Agent:**
- Every field that is saved must have a deterministic `id`.
- JS must read those IDs and serialize values in the agreed format.

### 6.3 Field groups

These CSS classes appear repeatedly:
- `.form-group-2` for two-column grid fields
- `.form-grid1` as a grid wrapper
- `.instructions1`, `.instructions2` for top info blocks

**Rule for AI Agent:**
- Match the visual grouping style:
  - Simple details pages: `.form-grid1` with `.form-group-2`
  - Dashboard or metrics pages: cards in a grid
  - Heavy editor pages: custom editor block, footer stays standard

---

## 7) JavaScript Structure Standard

### 7.1 Required: cancelUpdates()

Every modal includes this exact idea:
~~~js
function cancelUpdates() {
  FileMaker.PerformScript('Close Script Name');
}
~~~

In FileMaker concatenation:
~~~filemaker
Set Variable [ $Cancel Updates Script ;
  Value:
"function cancelUpdates() {

         FileMaker.PerformScript('" & $Cancel Script & "');

}"
]
~~~

### 7.2 Required in edit modals: saveUpdates()

The save function:
- Reads values from DOM
- Encodes values if needed
- Builds either:
  - A pipe-delimited string for simple cases, or
  - JSON for complex structures
- Calls `FileMaker.PerformScript(saveScriptName, parameters)`

Pipe example:
~~~js
function saveUpdates() {
  const a = document.getElementById('a').value;
  const b = document.getElementById('b').value;
  const parameters = [a, b].join('|');
  FileMaker.PerformScript('Save Script Name', parameters);
}
~~~

JSON example:
~~~js
function saveUpdates() {
  const data = { items: selectedItems };
  const parameters = JSON.stringify(data);
  FileMaker.PerformScript('Save Script Name', parameters);
}
~~~

### 7.3 Script bundling into one script tag

The system uses a single variable:
- `$Scripts` containing `<script> ... </script>`

Example:
~~~filemaker
Set Variable [ $Scripts ;
  Value:
"<script> " &
  $Save Updates Script & "¶¶" &
  $Cancel Updates Script & "¶¶" &
  $Some Other Script &
" </script>"
]
~~~

**Rule for AI Agent:**
- Build each JS function as a string.
- Concatenate them into one `<script>` block.
- Separate chunks with `¶¶` for readability (it is used consistently in the sample scripts).

---

## 8) Full Page Assembly Pattern

Every modal writes a full HTML document:

### 8.1 Standard head includes

- `<meta charset='UTF-8'>`
- `<meta name='viewport' content='width=device-width, initial-scale=1.0'>`
- Font Awesome stylesheet
- Optional Flatpickr CSS and JS when date pickers exist
- Inline `<style>` with `$styles`

Example:
~~~filemaker
Set Variable [ $$Modal ;
  Value:
"<!DOCTYPE html>
<html lang='en'>

<head>
   <meta charset='UTF-8'>
   <meta name='viewport' content='width=device-width, initial-scale=1.0'>
   <title>Modal Page</title>
   <link rel='stylesheet' href='https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css'>
   <style> " & $styles & " </style>
</head>

<body> " & $HTML & "¶¶" & $Scripts & " </body>

</html>
"
]
~~~

### 8.2 Flatpickr inclusion rule

Only include Flatpickr when a datepicker is used:
~~~html
<link rel='stylesheet' href='https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css'>
<script src='https://cdn.jsdelivr.net/npm/flatpickr'></script>
~~~

---

## 9) Window Opening Pattern

### 9.1 Standard Card window settings

Most modals do:
~~~filemaker
New Window [
  Style: Card ;
  Height: 750 ;
  Width: 600 ;
  Dim parent window: On ;
  Toolbars: Off ;
  Menu Bar: Off ;
  Close: Off ;
  Minimize: Off ;
  Maximize: Off ;
  Resize: Off ;
  Layout: "Modal"
]
~~~

### 9.2 Refresh or reuse modal window variant

Some scripts check whether a modal window already exists:
~~~filemaker
If [ $$Window = "Modal" ]
  Refresh Window [ Flush cached join results ; Flush cached external data ]
Else
  New Window [ Style: Card ; ... ; Layout: "Modal" ]
End If
~~~

**Rule for AI Agent:**
- Default to opening a new Card window unless the surrounding shell expects reuse.
- If reuse is required, follow the existing convention with `$$Window`.

---

## 10) Common UI Objects Used Across Modals

This documentation does not fully define each object implementation. It defines how they are assembled into the modal.

### 10.1 Simple form fields

- Text inputs
- Textareas
- Select dropdowns

Pattern:
- `<label for='id'>Label</label>`
- `<input id='id' value='...'>`
- `<textarea id='id'>...</textarea>`
- `<select id='id'>...options...</select>`

### 10.2 Date fields with icon-triggered picker

Structure:
~~~html
<div style='position: relative;'>
  <input type='text' id='start_date' value='YYYY-MM-DD'>
  <i class='fas fa-calendar-alt' id='start_date_icon' style='position: absolute; right: 10px; top: 50%; transform: translateY(-50%); ...'></i>
</div>
~~~

JS idea:
- On load, convert MySQL date to readable
- Flatpickr attaches to input
- Icon click opens picker
- A helper returns MySQL format to save

### 10.3 Segmented control buttons

Structure:
~~~html
<div class='segmented-control'>
  <button class='active' onclick="selectSegmentedControl(this)">Cash</button>
  <button onclick="selectSegmentedControl(this)">Insurance</button>
</div>
~~~

JS idea:
- Toggle active class
- Show or hide related fields based on selection

### 10.4 Searchable dropdown (custom)

Structure:
- Visible text input for searching
- Hidden input for UUID
- UL list to show results

JS idea:
- Load network list as JS array
- Filter by typed text
- Show first N results
- Infinite scroll load more
- Preselect default by UUID
- Hide dropdown on outside click

### 10.5 Selection list with checkboxes and row highlight

Structure comes from a Value List helper script (HTML chunk injected into `$HTML`).
JS idea:
- Preselect rows from FileMaker-provided IDs
- Clicking row toggles checkbox
- Save returns JSON list of selected items

### 10.6 Subtable block inside modal

Subtable HTML is generated by a helper FileMaker script and injected into `$HTML`.

The generator script supplies:
- SQL
- Column headers
- Column widths
- Column type hints (date columns, number columns)
- Input-enabled columns
- Table ID used by JS to collect changes
- Add-new button label and JS hook name
- Empty-state icon and message

Save logic for subtables:
- Loop all rows in tbody
- Each row has `data-id`
- Collect inputs per row
- Serialize with `vvvvvvvvvv` delimiter
- Attach to a combined parameter string with a prefix label

### 10.7 Rich text editor block (contenteditable)

Editor HTML injected into modal:
- Toolbar buttons
- `<div id="editor" contenteditable="true">...</div>`

JS idea:
- Uses `document.execCommand`
- Maintains selection restore logic
- Converts links to open in new tab
- On save, returns HTML encoded and sanitized for delimiters

---

## 11) Encoding and Safety Rules

### 11.1 Pipe delimiter protection

If a saved field can contain `|`, the pipe breaks the delimiter protocol.

Observed strategy in the editor script:
- Replace `|` with a safe token before building parameters

Example:
~~~js
editor = editor.replace(/\|/g, 'ppippe');
~~~

**Rule for AI Agent:**
- If saving pipe-delimited parameters, sanitize any field that may contain `|`.
- If saving large or complex text, prefer JSON over pipe delimiting.

### 11.2 URL / text encoding

Observed patterns:
- `encodeURIComponent(text)` for textarea or rich content
- `encodeURIComponent(editorHTML)` inside JSON payload

**Rule for AI Agent:**
- If a field can contain line breaks, quotes, or special characters, encode it.

### 11.3 Disabled mode

Some modals disable editing based on layout context:
~~~html
<textarea id='remarks' ... disabled>...</textarea>
~~~

In FileMaker concatenation:
~~~filemaker
If ( $$Layout Name = "Past Visits" ; "disabled" )
~~~

**Rule for AI Agent:**
- When modal is used for viewing historical records, disable input controls and still allow Cancel.

---

## 12) Modal Script Specification Template (AI Agent Must Follow)

This is the standard blueprint the AI Agent should implement for any new modal page.

### 12.1 Required Inputs (design-time specification)

For each new modal, the AI must be given:

1. Modal purpose and title
2. Entity type and primary key ID
3. Save script name (FileMaker)
4. Cancel script name (FileMaker)
5. Fields:
   - Field label
   - DOM id
   - Input type (text, number, textarea, select, segmented, date)
   - Default value rule
   - Save encoding rule (none, encodeURIComponent, JSON)
6. Data source:
   - Table name(s)
   - ExecuteSQL query
   - Any joins needed
7. UI objects used:
   - Date picker
   - Searchable dropdown
   - Selection list
   - Subtable
   - Rich editor
8. Window size:
   - Height
   - Width
9. View-only rules (if any):
   - Disable fields on certain layouts or modes

### 12.2 Required Output (FileMaker script code)

The generated script must contain these sections in order:

1. Get Script Parameter(s) into `$$...` variables
2. Set FileMaker script name variables
3. Fetch data with ExecuteSQL
4. Parse into `$Field` variables
5. Generate CSS
6. Generate HTML into `$HTML`
7. Generate JS into:
   - `$Cancel Updates Script`
   - `$Save Updates Script`
   - plus any helper scripts
8. Combine JS into `$Scripts`
9. Combine full page into `$$Modal`
10. Open Card window to layout `Modal`

---

## 13) Object-Specific Assembly Guides (How to Combine Parts)

### 13.1 Simple details modal (Staff Education pattern)

- Fetch 1 record by ID
- Show input fields in `.form-grid1`
- Add flatpickr for date fields
- Save returns a pipe-delimited list of values

Key assembly checklist:
- Flatpickr included in `<head>`
- JS includes:
  - `getMySQLFormattedDate(fieldId)`
  - `saveUpdates()` uses it for date fields

### 13.2 Assign list modal (Appraisal template assignment pattern)

- Pull list HTML from `Get Value List` helper
- Preselect items using `$$Selected IDs` (return-separated string)
- Save returns JSON with selected rows details

Key assembly checklist:
- Row elements must expose:
  - `data-id`
  - `input.rowCheckbox`
  - `.item-title`, `.item-subtitle`, `.item-description`
- JS must:
  - Toggle `.selected` class
  - Update selected count UI element if present

### 13.3 Conditional fields modal (Payment option pattern)

- Segmented control chooses a mode
- JS toggles visibility of field containers
- Some dropdowns are populated from JS objects injected from FileMaker

Key assembly checklist:
- Segmented control buttons call `selectSegmentedControl(this)`
- `DOMContentLoaded` runs initial visibility setup
- Insurer and policies dropdowns:
  - `const insurers = { ... }`
  - `const policies = { ... }`
  - Populate insurer options
  - Populate policy options when insurer changes
  - Preselect insurer and policy when IDs exist

### 13.4 Dashboard card modal (Vitals pattern)

- Fetch data by ID, else default to current user and time
- Render a grid of cards with inputs
- Some fields computed live (BMI)
- Save returns pipe-delimited numeric values

Key assembly checklist:
- `calculateBMI()` runs on input events for height and weight
- `bmi` field is readonly

### 13.5 Subtable inside modal (Treatment chart pattern)

- Fetch header details (medication, dosage)
- Generate subtable HTML via helper script `+++ Display Subtable HTML`
- Provide Add New Entry button that calls a FileMaker script (administer medication)
- Save collects row updates from table inputs

Key assembly checklist:
- Subtable table has stable ID (example: `treatmentChartTable`)
- Each `tr` has `data-id`
- Save builds combined parameters:
  - `ITEMS:...`
  - `TREATMENT_CHART:...`

### 13.6 Rich editor modal (Nurse note pattern)

- Modal title based on `$$Mode`
- Uses contenteditable editor and toolbar
- Save returns JSON with encoded HTML

Key assembly checklist:
- Editor uses selection save and restore
- Links open in new tab, prevent navigation inside WebViewer
- Save replaces pipe characters and encodes HTML

### 13.7 Searchable dropdown modal (Diagnosis pattern)

- Data list injected as JS array of objects
- Input filters list and writes selection into hidden field
- Save uses hidden UUID field and encoded remarks

Key assembly checklist:
- Preselect existing UUID on load
- Infinite scroll loads more results in chunks

---

## 14) Helper Scripts and External Dependencies

This modal system assumes the existence of helper scripts that return reusable chunks:

1. `🖌️ Use Modal CSS`
   - Returns the base CSS for `.modal`, `.modal-header`, `.modal-body`, `.modal-footer`, buttons, grids, form groups

2. `Get Value List`
   - Returns either:
     - HTML options for `<select>`
     - Full HTML blocks for checklist selection list
     - JavaScript object literal strings for dynamic dropdowns

3. `+++ Display Subtable HTML`
   - Returns a fully formed HTML table widget with optional inline inputs

4. `WebDirect Pause`
   - A compatibility helper for WebDirect rendering flow

External libraries used in modals:
- Font Awesome (icons)
- Flatpickr (date picker)

---

## 15) Quality Rules the AI Agent Must Follow

1. Keep IDs consistent between HTML and JS.
2. Keep Save and Cancel behavior consistent with the pattern.
3. Use `ExecuteSQL` with parameter binding (`?`) for IDs.
4. Avoid injecting raw unescaped text into HTML when it can break attributes.
   - When placing values inside `value='...'`, ensure they do not contain `'`.
   - If the system does not escape, the AI must apply a safe substitution rule.
5. Prefer JSON for complex payloads.
6. If using pipe-delimited parameters, sanitize pipe characters in user-entered fields.
7. Include only required external dependencies in the page head.
8. Always store final page string in `$$Modal` before opening the modal window.
9. Use a Card window to the Modal layout with dim parent on.

---

## 16) Ambiguities, Contradictions, and Clarifications Needed

The AI Agent can generate correct scripts only if these are clarified.

### 16.1 How the Modal layout loads `$$Modal`
In the provided scripts, `$$Modal` is set and a new card window opens, yet there is no explicit WebViewer action shown (no `Set Web Viewer` step).

Clarification needed:
- Does the Modal layout contain a WebViewer that auto-loads `$$Modal` using a calculation like `GetAsText ( $$Modal )` or a data URL technique?
- Is there an OnLayoutEnter trigger that pushes `$$Modal` into the WebViewer?

### 16.2 Inconsistent parameter function name
Some scripts use `GetScriptParameters` while FileMaker standard is `Get ( ScriptParameter )`.

Clarification needed:
- Is `GetScriptParameters` a custom function in this file?
- If yes, what exactly does it return and how is it structured?

### 16.3 Variable naming collision: `$Save Updates Script`
In multiple scripts, `$Save Updates Script` is used as:
- FileMaker script name string, and later
- JavaScript function body string

Clarification needed:
- Is this a deliberate style choice you want preserved?
- Or should the AI Agent always keep separate names for FM script name and JS function code?

### 16.4 Payment Option modal references `$$Patient ID` without showing it being set
The Payment Option modal validates a patient exists using `$$Patient ID`, yet that variable is not set in the snippet.

Clarification needed:
- Where is `$$Patient ID` set for that modal flow?
- Should every modal that depends on parent entity enforce such validation?

### 16.5 Datepicker script included where date fields are not present
In the Payment Option script, a datepicker script block exists with `date_scheduled` and `date_completed`, yet those IDs do not exist in the HTML.

Clarification needed:
- Is that block a leftover template piece?
- Should the AI Agent include datepicker blocks only when those inputs exist?

### 16.6 Escaping values inserted inside HTML attributes
Values are inserted into:
~~~html
<input value='" & $Value & "'>
~~~
If `$Value` contains a single quote, it can break HTML.

Clarification needed:
- Do you already sanitize values before insertion?
- Do you want the AI Agent to always sanitize `'` into `&#39;` and `"` into `&quot;` when injecting into attributes?

### 16.7 The dropdown naming mismatch in Diagnosis modal
The diagnosis modal uses IDs like `momo_network` for a medical condition selector.

Clarification needed:
- Is this naming intentionally reused to reduce code duplication?
- Or should the AI Agent generate semantically correct IDs per modal?

### 16.8 Special delimiters used for subtable saves
The treatment chart save uses:
- `vvvvvvvvvv` as a row delimiter
- Combined parameter string with labels

Clarification needed:
- Are these delimiters standardized across all subtables?
- Do you want a single canonical delimiter set for all subtable modals?

---

## 17) Minimal Reference Implementation Blueprint (Pseudo)

This section is a compact spec the AI Agent can follow.

~~~text
INPUT:
- ModalName
- Title
- PrimaryIDVar (global $$...)
- SQL query + binds
- Field map: { label, domId, type, default, encoding }
- SaveScriptFM, CancelScriptFM
- Optional widgets: datepicker, segmented, dropdownSearch, selectionList, subtable, editor
- Window size

OUTPUT:
- FileMaker script that:
  - Reads parameter(s) into $$...
  - Sets $SaveFM and $CancelFM
  - Executes SQL, parses fields
  - Calls 🖌️ Use Modal CSS and appends extras
  - Builds $HTML with modal wrapper, form, footer buttons
  - Builds JS functions:
    - cancelUpdates() always
    - saveUpdates() when editable
    - widget helpers based on usage
  - Assembles $$Modal full HTML doc
  - Opens Card window to Modal layout

---

# CauferoAppStarter Documentation
## Generate a FileMaker Script That Creates a New Record for Any Entity Type

### Purpose
This document teaches an AI Agent how to generate a FileMaker script that creates a new record for any entity type in **CauferoAppStarter**.

This script is the standard **entry point** used by a List Page button like:
- Add New Department
- Add New Material
- Add New Job Title
- Add New Logo File
- Add Staff Emergency Contact

### Core Pattern Used in CauferoAppStarter
CauferoAppStarter uses a simple, reliable pattern:

1. **The Add New script does not directly create the record.**
2. It generates a brand new UUID using `Get ( UUID )`.
3. It calls the target **Details Page** script (or **Modal Page** script) and passes that UUID.
4. The target page script uses the UUID to do an **Upsert-by-UUID**:
   - If a record with that UUID exists, it opens it.
   - If it does not exist, it creates a new record and sets its UUID to the passed value.

This means the Add New script is extremely consistent across all entities and very easy to auto-generate.

### Evidence From Existing Samples (Naming and Call Style)
Your samples show this exact call style:

- `Perform Script [ Specified: From list ; "+++ Department Details Page" ; Parameter: Get ( UUID ) ]`
- `Perform Script [ Specified: From list ; "+++ Material Details Page" ; Parameter: Get ( UUID ) ]`
- `Perform Script [ Specified: From list ; "+++ Job Title Details Page" ; Parameter: Get ( UUID ) ]`
- `Perform Script [ Specified: From list ; "+++ Logo File Details Page" ; Parameter: Get ( UUID ) ]`
- `Perform Script [ Specified: From list ; "+++ Staff Emergency Contacts Modal Page" ; Parameter: Get ( UUID ) ]`

So the Add New script is a consistent delegator that provides the new UUID.

---

# 1) What The AI Agent Must Generate

## 1.1 Script Goal
Generate a script that can be attached to an "Add New <Entity>" button on a List Page, and that will result in a new record being created for that entity type and opened on the correct UI (Details Page or Modal Page).

## 1.2 Input
- No required script parameter.
- The script itself generates a new UUID using `Get ( UUID )`.

## 1.3 Output
- The script navigates the user into the correct entity UI.
- The resulting entity record will have a UUID equal to the generated UUID.

The script usually does not return anything. If your framework uses script results, the expected result is the created UUID.

---

# 2) Naming Rules and Conventions

## 2.1 Script Name
The Add New script name should reflect the UI button label and entity.

Recommended naming pattern:
- `Add New <Entity>` as the visible label
- Script name can match label or follow internal naming, as long as it is consistent

Examples:
- `Add New Department`
- `Add New Material`
- `Add New Job Title`
- `Add New Logo File`
- `Add Staff Emergency Contact`

## 2.2 Target Script Name
The Add New script must call one of these target scripts:

### A) Entity Details Page Script
Pattern:
- `+++ <Entity> Details Page`

Examples:
- `+++ Department Details Page`
- `+++ Material Details Page`
- `+++ Job Title Details Page`
- `+++ Logo File Details Page`

### B) Entity Modal Page Script
Pattern:
- `+++ <Entity> Modal Page`

Example:
- `+++ Staff Emergency Contacts Modal Page`

## 2.3 Script Call Context
Use the Perform Script step exactly like your samples:
- `Specified: From list`
- Target script name must match exactly
- Parameter must be `Get ( UUID )`

---

# 3) The Add New Script Specification

## 3.1 High-Level Algorithm
When generating the Add New script, follow these steps:

1. Generate a new UUID for the record.
2. Perform the target page script (Details Page or Modal Page).
3. Pass the new UUID as the script parameter.

That is all.

## 3.2 Script Steps Template
This is the exact pattern the AI Agent must produce.

~~~filemaker
# Script: Add New <Entity>

Perform Script [
    Specified: From list ;
    "+++ <Entity> Details Page" ;
    Parameter: Get ( UUID )
]
~~~

If the entity is shown in a modal, replace the target script:

~~~filemaker
# Script: Add <Entity> (Modal)

Perform Script [
    Specified: From list ;
    "+++ <Entity> Modal Page" ;
    Parameter: Get ( UUID )
]
~~~

## 3.3 Why This Works
Because the target script receives the UUID and performs the Upsert-by-UUID logic:
- Find record by UUID
- Create if missing
- Load the UI

This keeps the Add New scripts uniform, fast to generate, and easy to maintain.

---

# 4) Target Page Script Contract (Required For Record Creation)

Even though this documentation focuses on the Add New script, the record creation only succeeds if the target page script follows this contract.

The AI Agent must assume this contract exists, and must flag it if it is missing.

## 4.1 Contract Summary
Target script receives `Get ( ScriptParameter )` as `$targetUUID`.

Then it must:
1. Validate `$targetUUID` is not empty.
2. Try to locate an existing record by UUID.
3. If not found, create a new record and set UUID to `$targetUUID`.
4. Continue building the page or modal for that record.

## 4.2 Contract Pseudocode
~~~filemaker
# Script: +++ <Entity> Details Page
# Parameter: targetUUID

Set Variable [ $targetUUID ; Get ( ScriptParameter ) ]

If [ IsEmpty ( $targetUUID ) ]
    # Handle error (dialog, exit, log)
    Exit Script [ Result: "" ]
End If

# Attempt to find existing record by UUID
# If found -> load it
# If not found -> create it and set UUID = $targetUUID

# Continue with page rendering logic
Exit Script [ Result: $targetUUID ]
~~~

## 4.3 Contract Concrete Skeleton (Recommended)
The exact steps will depend on your file structure, layouts, and how you manage context, but the creation portion usually follows this shape:

~~~filemaker
# Script: +++ <Entity> Details Page
# Parameter: targetUUID

Set Error Capture [ On ]
Allow User Abort [ Off ]

Set Variable [ $targetUUID ; Get ( ScriptParameter ) ]

If [ IsEmpty ( $targetUUID ) ]
    # Show dialog via your standard utility script
    # Perform Script [ "UTIL | Dialog" ; Parameter: ... ]
    Exit Script [ Result: "" ]
End If

# Go to an entity context where you can search and create
# Go to Layout [ "<Entity>__Context" ( <Entity> ) ]

Enter Find Mode [ Pause: Off ]
Set Field [ <Entity>::UUID ; $targetUUID ]
Perform Find [ ]

If [ Get ( FoundCount ) = 0 ]
    New Record/Request
    Set Field [ <Entity>::UUID ; $targetUUID ]

    # Recommended default fields
    # Set Field [ <Entity>::CreatedTS ; Get ( CurrentTimestamp ) ]
    # Set Field [ <Entity>::CreatedBy ; Get ( AccountName ) ]

    Commit Records/Requests [ Skip data entry validation: Off ]
    If [ Get ( LastError ) ≠ 0 ]
        Revert Record/Request [ No dialog ]
        Exit Script [ Result: "" ]
    End If
End If

# Now load the details UI for the record whose UUID is $targetUUID
# Continue with your Page Builder code
Exit Script [ Result: $targetUUID ]
~~~

This contract is the heart of the system. The Add New script stays clean because the Details Page script owns the creation logic.

---

# 5) How The AI Agent Chooses Details Page vs Modal Page

## 5.1 Decision Rule
Use **Details Page** when the entity is edited on a full details page experience.

Use **Modal Page** when the entity is edited inside a modal window.

## 5.2 How To Determine Which One To Use
Use the existing UI spec for that entity type.

If the UI button label says:
- "Add New <Entity>" and the entity has its own page -> Details Page
- "Add <Entity>" inside a larger parent workflow -> Modal Page

Your samples show:
- Staff Emergency Contacts uses Modal Page
- Department, Material, Job Title, Logo File use Details Page

## 5.3 Output Rule
The Add New script must call exactly one target script and pass `Get ( UUID )`.

---

# 6) Generation Checklist (AI Agent Quality Control)

Before outputting the script, the AI Agent must verify:

1. Target script name starts with `+++`.
2. Target script name ends with either `Details Page` or `Modal Page`.
3. Perform Script uses `Specified: From list`.
4. Parameter is exactly `Get ( UUID )`.
5. No extra steps are added unless a project rule explicitly requires them.

---

# 7) Common Failure Modes and Fixes

## 7.1 Failure: Record gets created with a different UUID
Cause:
- Target script generates a new UUID instead of using the passed UUID.

Fix:
- Target script must use `$targetUUID = Get ( ScriptParameter )` and set the record UUID to `$targetUUID`.

## 7.2 Failure: Page opens but record is missing
Cause:
- Target script does not implement Upsert-by-UUID.

Fix:
- Implement find-by-UUID then create-if-missing.

## 7.3 Failure: Wrong target script used
Cause:
- Modal entity called via Details Page script name, or vice versa.

Fix:
- Use UI spec and naming patterns to choose correctly.

## 7.4 Failure: Parameter is missing or empty
Cause:
- Parameter was not passed, or target script ignored it.

Fix:
- Add New script must pass `Get ( UUID )`.
- Target script must validate `$targetUUID`.

---

# 8) Examples (Directly Matching Your Samples)

## 8.1 Add New Logo File
~~~filemaker
Perform Script [
    Specified: From list ;
    "+++ Logo File Details Page" ;
    Parameter: Get ( UUID )
]
~~~

## 8.2 Add New Department
~~~filemaker
Perform Script [
    Specified: From list ;
    "+++ Department Details Page" ;
    Parameter: Get ( UUID )
]
~~~

## 8.3 Add New Job Title
~~~filemaker
Perform Script [
    Specified: From list ;
    "+++ Job Title Details Page" ;
    Parameter: Get ( UUID )
]
~~~

## 8.4 Add Staff Emergency Contact (Modal)
~~~filemaker
Perform Script [
    Specified: From list ;
    "+++ Staff Emergency Contacts Modal Page" ;
    Parameter: Get ( UUID )
]
~~~

## 8.5 Add New Material
~~~filemaker
Perform Script [
    Specified: From list ;
    "+++ Material Details Page" ;
    Parameter: Get ( UUID )
]
~~~

---

# 9) Clarifications Needed (Ambiguities That Must Be Resolved For Perfect Training)

The samples strongly imply the Upsert-by-UUID pattern, but to make the AI Agent output perfect, these must be confirmed:

1. **UUID Field Name**
   - What is the exact UUID field name used in each entity table?
   - Examples: `UUID`, `z_UUID`, `__k_UUID`, `pk_UUID`

2. **Where Upsert-by-UUID Lives**
   - Confirm that the record creation happens inside:
     - `+++ <Entity> Details Page`
     - `+++ <Entity> Modal Page`
   - Confirm that the Add New scripts remain one-step delegators.

3. **Default Fields On Create**
   - Which standard fields must be set on every new record?
   - Examples: CreatedTS, CreatedBy, UpdatedTS, UpdatedBy, IsActive, SortOrder

4. **Commit Rule**
   - Must the new record be committed immediately?
   - Or do you allow uncommitted creation while the user edits in a modal?

5. **Privilege and Error Handling**
   - Do you have a standard utility script for:
     - Access checks
     - Error dialogs
     - Logging
   - If yes, provide names so the AI Agent can call them consistently.

6. **Context Layout Rule**
   - When the target page script does the find and create, which layout should it use for that context?
   - Is there a standard context layout per entity for safe record operations?

7. **Modal Parent Context**
   - For modal entities like Staff Emergency Contacts, is the new record linked to a parent record immediately?
   - If yes, what parent keys are required and how are they passed?

Once these are clarified, the AI Agent can generate Add New scripts and the supporting Details/Modal scripts with full consistency across all entity types.

---

# Saving the Details of a Selected Entity (CauferoAppStarter)
_Training documentation for an AI Agent to generate FileMaker scripts that save the edited details of a selected entity record that originated from a List Page, whether the Details UI is a normal page or a modal window._

---

## 0) Scope and Goal

This documentation teaches an AI Agent how to generate a **Save Details** script for an entity record that was:

1. Selected from a **List Page**
2. Opened for viewing and editing in a Details UI that is either:
   - a normal **Details Page**, or
   - a **Modal Details Page** (a modal window)
3. Edited by the user
4. Saved back into the database

### 0.1 Key definition: “Selected Entity” in this document
In this document, “selected entity” always means:
- the **main entity record selected from a List Page** and opened for editing.

The Details UI may be a normal page or a modal window. Modal is only a UI container.

### 0.2 Out of scope for this document
This documentation does not cover saving a record that was opened from a **subtable of related records**, even if that related record is shown in a modal window.

That pattern uses a different saving approach and belongs to a separate documentation.

---

## 1) Canonical Save Details Pattern

A Save Details script must follow this flow:

1. Read ordered values from `Get ( ScriptParameter )` via `GetValue`.
2. Normalize inputs (dates, numbers, URL-encoded text).
3. Optional confirmation dialog for submissions.
4. Save using the generic script: **“Save A Record”**.
5. Optionally perform additional saves for related or join tables (only when they are part of saving the main entity record).
6. If the Details UI is a **Modal Details Page**, close the modal window.
7. Restore UI state (example: `$$Tab To Show`).
8. Navigate to the appropriate Details Page or List Page.
9. Show a success message.

This is the exact pattern used in your samples, with one added step for modal pages.

---

## 2) Core System Contracts (Must Be Obeyed)

### 2.1 The record ID is always provided
- Save Details scripts always save using a primary key ID.
- The ID will not be missing.
- The Save Details script must already have the selected record’s ID in a global variable (example: `$$Patient ID`, `$$Department ID`, `$$Staff ID`).

### 2.2 Field names in the save string must match FileMaker schema exactly
- Every field name included in the “Save A Record” parameter must be the exact FileMaker field name.
- No labels, no aliases, no friendly names.

### 2.3 `Settings::Location` is mandatory and always present
- `Settings::Location` indicates the app run mode (Local or On Server).
- “Save A Record” uses it to decide whether to run the final insert/update locally or perform the final script on server.
- If `Settings::Location` is blank, saving must error. Developer must set it.

---

## 3) “Save A Record” Contract (Authoritative)

### 3.1 Parameter string format
“Save A Record” receives one single string parameter using `¶` as the delimiter:

1. Table name (required, first item)
2. `Settings::Location` (required, second item)
3. Primary key field name and value (required, third pair)
4. Then field name/value pairs

Example shape:

~~~filemaker
Perform Script [ Specified: From list ; “Save A Record” ;
  Parameter:
    "Departments¶" &
    Settings::Location &
    "¶ID¶" & $$Department ID &
    "¶Name¶" & $Name &
    "¶HOD Staff ID¶" & $HOD Staff ID &
    "¶Description¶" & $Description
]
~~~

### 3.2 “Save A Record” behavior
“Save A Record” does this:

1. Goes to the specified table context.
2. Finds by the provided primary key field (example: `ID`) and value.
3. If the record is found, it updates fields.
4. If the record is not found, it creates a new record and sets the primary key field to the provided ID, then sets the remaining fields.

Important implication:
- Even for Save Details, if the ID exists and is valid but not found in the table, the record will be created as new.
- This is allowed and part of the design.

### 3.3 “ID” must always be included
The Save Details script must always include:

- `"¶ID¶" & $$Entity ID`

No exceptions.

---

## 4) Inputs and Normalization Rules

### 4.1 Script parameters are ordered
Your Details UI must pass values in the exact order the Save Details script expects.

Every Save Details script must read with:

- `GetValue ( GetScriptParameters ; 1 )`
- `GetValue ( GetScriptParameters ; 2 )`
- etc.

### 4.2 Text decoding rule (DecodeURIComponent)
If a value may come from a Web Viewer, treat it as URL-encoded unless you are 100 percent sure it is not encoded.

Use:

~~~filemaker
Set Variable [ $Description ; Value: DecodeURIComponent ( GetValue ( GetScriptParameters ; 3 ) ) ]
~~~

Use DecodeURIComponent for:
- Description
- Reason
- Addresses
- Notes
- Any multi-word text
- Any text that can contain punctuation or line breaks

### 4.3 Date conversion rule (MySQLDateToFMPDateText)
Dates are passed in a format that must be converted to FileMaker date format before saving.

Preferred single-step pattern:

~~~filemaker
Set Variable [ $Start Date ; Value: MySQLDateToFMPDateText ( GetValue ( GetScriptParameters ; 2 ) ) ]
~~~

Two-step conversion is allowed if a single step cannot guarantee the correct FileMaker format.

### 4.4 Number casting rule
When a value must be numeric, cast it:

~~~filemaker
Set Variable [ $Duration ; Value: GetAsNumber ( GetValue ( GetScriptParameters ; 4 ) ) ]
~~~

### 4.5 Derived values rule
If the database stores derived values, compute them after normalization.

Example from Staff:
- Age from DOB
- DOB Day
- DOB MonthName
- DOB MonthNumber key

---

## 5) Modal Details Page Rule (Mandatory When the Details UI Is Modal)

If the entity Details UI is a **Modal Details Page**:

### 5.1 The only difference
After all save calls complete, insert a Close Window step before restoring tab state.

Required sequence:

1. Perform the save call(s)
2. Close the modal window
3. Set `$$Tab To Show`
4. Navigate as required

### 5.2 Required step and exact placement
Insert this step immediately before:

- `Set Variable [ $$Tab To Show ; Value: 1 ]`

Required step:

~~~filemaker
Perform Script [ Specified: From list ; “Close Window” ; Parameter:    ]
~~~

---

## 6) Post-save UI and Navigation Rules

### 6.1 Tab restoration
If your Details UI uses tabs, restore the desired tab:

~~~filemaker
Set Variable [ $$Tab To Show ; Value: 1 ]
~~~

If the Details UI is modal:
- Close Window first
- Then set `$$Tab To Show`

### 6.2 Navigation after saving
Choose the destination based on the UX:

- Reload Details Page when user should remain on the record (example: Department, Staff).
- Return to List Page when save completes a task and user should go back (example: Patients).
- Return to a workflow list page after submission (example: Leave Request submission).

---

## 7) Commit Records/Requests Rule (Recommended Standard)

You asked: “Is Commit Records/Requests being used anywhere?”

I cannot see your full file from here, so I cannot confirm usage directly.

### 7.1 Recommended standard
- Do not put `Commit Records/Requests` inside individual Save Details scripts.
- Put commit responsibility inside “Save A Record” (or inside the final insert/update scripts that “Save A Record” calls).

### 7.2 How to verify inside your FileMaker file
In Script Workspace, search for:
- `Commit Records/Requests`

Check:
- “Save A Record”
- Any scripts it performs (local and server variants)
- Any lower-level scripts used for final insert/update

If commit exists there, then Save Details scripts should not add commit.

If commit does not exist anywhere in the save engine, then add commit inside the save engine, not inside each Save Details script.

---

## 8) Month Fields Standardization Rule

You want a standard when similar month tracking is needed.

### 8.1 Recommended standard fields
If a table needs month tracking, store both:

1. Readable label:
   - `Month Created` = `MonthName ( Get ( CurrentDate ) ) & ", " & Year ( Get ( CurrentDate ) )`
2. Sortable numeric key:
   - `Month Created Number` = `Year ( Get ( CurrentDate ) ) * 100 + Month ( Get ( CurrentDate ) )`

If your schema has different field names (example: `Month Name Created`), keep field names, but align the values to the standard approach.

---

## 9) Entity Patterns the AI Agent Must Learn From the Samples

### 9.1 Department (single table save)
Traits:
- Few params
- Decode description
- Save one table
- Reload Details Page
- Success dialog

### 9.2 Leave Request (submission style save)
Traits:
- Confirmation dialog required
- Convert multiple dates
- Decode reason
- Save status
- Go back to workflow list page
- Success dialog

### 9.3 Staff (multi-save plus user/role side effect)
Traits:
- Many params
- Convert DOB
- Compute derived fields
- Generate staff number if empty using a global previous code
- Save Staff main table
- Save Staff Job Titles table
- If role changed, create or update user record

Important correction:
- Use `$$Previous Code`, not `$Previous Code`.

### 9.4 Patient (main save plus related saves)
Traits:
- Convert DOB and compute age
- Save Patients
- Save emergency contact
- Save primary care physician
- Return to Patients list page

---

## 10) Canonical Rule for Role and User Logic (Staff)

### 10.1 Stored meaning of `$$Role ID`
- `$$Role ID` is captured when viewing the staff record.
- It represents the staff’s role before edits.

### 10.2 When to create or update the user
When saving staff:
1. If `$Role ID` is empty, skip user logic.
2. If `$Role ID` is not empty:
   - Find if a user exists for this staff (via Users table lookup).
   - If role did not change, do nothing.
   - If role changed:
     - If user does not exist, create user.
     - If user exists, update role.

This must happen only when the role changed.

### 10.3 Username logic per business
Your sample shows:

~~~filemaker
Set Variable [ $Username ;
  Value:
    Case (
      $$Staff ID = $$My Staff ID ; "admin" ;
      $$Business Name = "DC Clinic" ; $Staff Number ;
      $Email
    )
]
~~~

This is a business rule. The AI Agent must not invent new business rules.

Remaining requirement:
- You should define a single authoritative username rule per business, then the AI Agent must reuse it exactly.

---

## 11) The AI Agent Output Requirements (Hard Requirements)

When asked to generate a Save Details script, the AI Agent must produce a FileMaker script that includes:

1. A clear script name in your naming convention:
   - `Save [Entity] Details` or `Save [Entity]`
2. Parameter extraction for every input using `GetValue`.
3. Correct normalization:
   - `DecodeURIComponent` for encoded text
   - `MySQLDateToFMPDateText` for dates
   - `GetAsNumber` for numbers
4. One or more “Save A Record” calls with this exact ordering:
   - Table name first
   - `Settings::Location` second
   - Primary key pair (`ID` then the ID value) immediately after
   - Field name/value pairs after that
5. Any secondary saves for related or join tables if required.
6. If the Details UI is a modal page:
   - Perform Script: `Close Window` after saving and immediately before setting `$$Tab To Show`
7. Post-save UI state restore (`$$Tab To Show` when applicable).
8. Navigation to Details Page or List Page.
9. A success dialog.

---

## 12) Canonical Save Details Template (AI Agent Reuse Template)

Adapt this template per entity.

~~~filemaker
# Save [Entity] Details

# Get Script Parameters
Set Variable [ $Field 1 ; Value: GetValue ( GetScriptParameters ; 1 ) ]
Set Variable [ $Field 2 ; Value: GetValue ( GetScriptParameters ; 2 ) ]
Set Variable [ $Long Text ; Value: DecodeURIComponent ( GetValue ( GetScriptParameters ; 3 ) ) ]
Set Variable [ $Date Field ; Value: MySQLDateToFMPDateText ( GetValue ( GetScriptParameters ; 4 ) ) ]
Set Variable [ $Number Field ; Value: GetAsNumber ( GetValue ( GetScriptParameters ; 5 ) ) ]

# Derived values (only if schema needs them)
# Set Variable [ $Derived ; Value: ... ]

# Optional confirmation for submissions
# Show Custom Dialog [ "Confirm" ; "Are you sure?" ]
# If [ Get ( LastMessageChoice ) ≠ 1 ]
#   Exit Script [ Result: "" ]
# End If

# Save main record
Perform Script [ Specified: From list ; “Save A Record” ;
  Parameter:
    "[TableName]¶" &
    Settings::Location &
    "¶ID¶" & $$[Entity] ID &
    "¶Field 1¶" & $Field 1 &
    "¶Field 2¶" & $Field 2 &
    "¶Long Text¶" & $Long Text &
    "¶Date Field¶" & $Date Field &
    "¶Number Field¶" & $Number Field
]

# Optional related saves (only if required as part of saving the main entity record)
# Perform Script [ Specified: From list ; “Save A Record” ;
#   Parameter: "RelatedTable¶" & Settings::Location & "¶ID¶" & $$Related ID & ...
# ]

# If Details UI is a Modal Details Page, close it before restoring tab state
# Perform Script [ Specified: From list ; “Close Window” ; Parameter:    ]

# Post-save UI state
Set Variable [ $$Tab To Show ; Value: 1 ]

# Navigate
Perform Script [ Specified: From list ; “+++ [Entity] Details Page” ; Parameter: $$[Entity] ID ]

# Success feedback
Show Custom Dialog [ "Saved" ; "[Entity] saved successfully" ]
~~~

---

## 13) Remaining Ambiguities That Still Need Definition

These are the only items still not fully defined. The AI Agent cannot invent them.

1. Username rule per business:
   - You should define the authoritative logic and where it lives.

2. Month field naming:
   - You want a standard, but field names differ between tables.
   - Define whether you will standardize field names across tables, or keep table-specific field names and standardize values only.

Everything else is deterministic based on your answers.

---

# Saving Records in Batch (CauferoAppStarter)
_Training documentation for an AI Agent to generate FileMaker scripts that save a batch of records in one operation, using list parsing or JSON parsing, typically replacing an existing found set._

---

## 0) Scope and Goal

This documentation teaches an AI Agent how to generate **Batch Save** scripts, meaning:

- The UI collects many items (answers, questions, chart rows, results, costs).
- The script receives them as a single payload.
- The script then saves **multiple records** in a loop.

Batch Save scripts in this system commonly use a **replace pattern**:

1. Find the existing related records for a parent context.
2. Delete all those existing records.
3. Recreate the entire set from the incoming payload.

This documentation covers that batch replace style.

---

## 1) Canonical Batch Save Pattern

A Batch Save script must follow this flow:

1. Turn on `Set Error Capture [ On ]` (mandatory).
2. Read the payload from `Get ( ScriptParameter )` or ordered parameters.
3. Convert payload into a loopable structure:
   - return-delimited list, or
   - JSON array iteration.
4. Open a Card window when the script must isolate context (recommended for destructive operations).
5. Go to the target table layout.
6. Perform a find that isolates the target record set (usually `Perform Find [ Restore ]`).
7. If found set has records, delete them all:
   - `Delete All Records [ With dialog: Off ]`
8. Loop through the incoming items:
   - parse one item
   - validate minimal integrity
   - create record
   - set fields
9. Close the Card window if one was opened.
10. Restore UI state and navigate if required.

---

## 2) When to Use Batch Save

Batch Save is preferred when:

- The UI edits many rows at once.
- The user expects one “Save” action to apply to all rows.
- The database rows are “dependent children” of a parent record (answers for an appraisal, lab results for a test, costs for a payroll item, treatment chart rows for a visit).
- Partial incremental updates are not needed, and a full replace is acceptable.

---

## 3) Batch Save Strategies Supported

This system uses two batch save strategies:

### 3.1 Replace-All (delete existing, reinsert all)
This is the dominant strategy in your samples.

Traits:
- A find is performed to get the old set.
- The found set is deleted.
- New records are inserted from the payload.

Used by:
- Save Appraisal Answers
- Save Treatment Chart
- Save Lab Test Results
- Save Payroll Item Costs

### 3.2 Upsert Per Item (find-by-ID, create if missing, update if found)
This is used when incoming items include IDs that must map to persistent records and you must avoid deleting everything.

Used by:
- Save Appraisal Template Questions (categories and questions)

---

## 4) Payload Formats the AI Agent Must Handle

### 4.1 Return-delimited list payloads (common)
The payload is converted into a return-delimited list.
Each line represents one record item.
Each line is then split using a secondary delimiter (often `|`, sometimes `^`).

Examples:
- Lab results: comma separated items -> replaced with `¶`, each item split by `|`
- Treatment chart: items separated by custom markers then split by `|`
- Payroll costs: items separated by commas -> `¶`, then split by `|` and `^`

Canonical parsing pattern:

~~~filemaker
Set Variable [ $Items ; Value: <payload> ]
Set Variable [ $Items ; Value: Substitute ( $Items ; "," ; ¶ ) ]
Set Variable [ $Total Items ; Value: ValueCount ( $Items ) ]

If [ $Total Items > 0 ]
  Loop [ Flush: Always ]
    Set Variable [ $i ; Value: $i + 1 ]
    Set Variable [ $Item ; Value: GetValue ( $Items ; $i ) ]
    Set Variable [ $Item As List ; Value: Substitute ( $Item ; "|" ; ¶ ) ]
    /* Extract fields using GetValue */
    Exit Loop If [ $i ≥ $Total Items ]
  End Loop
End If
~~~

### 4.2 JSON array payloads (structured)
The payload is JSON and includes an array key (example: `records`).
Each record object contains multiple keys.

Canonical parsing pattern:

~~~filemaker
Set Variable [ $jsonData ; Value: GetValue ( GetScriptParameters ; 1 ) ]
Set Variable [ $recordCount ; Value: JSONListKeys ( $jsonData ; "records" ) ]
Set Variable [ $counter ; Value: 0 ]

Loop [ Flush: Always ]
  Exit Loop If [ $counter >= ValueCount ( $recordCount ) ]
  Set Variable [ $currentRecord ; Value: JSONGetElement ( $jsonData ; "records[" & $counter & "]" ) ]

  /* Read keys with JSONGetElement */
  Set Variable [ $counter ; Value: $counter + 1 ]
End Loop
~~~

---

## 5) Window and Layout Rules

### 5.1 When to open a Card window (recommended)
Open a Card window when the script will:
- delete records,
- create many records,
- switch layouts multiple times,
- or should not affect user context.

Pattern:

~~~filemaker
New Window [ Style: Card ; Using layout: <Current Layout> ]
/* work in the card */
Perform Script [ Specified: From list ; “Close Window” ; Parameter:    ]
~~~

This is used in:
- Save Appraisal Answers
- Save Lab Test Results
- Save Payroll Item Costs

### 5.2 Layout switching rule
Batch saves are layout-driven.
You must go to the table layout that matches the records you are editing before:
- finding,
- deleting,
- creating,
- setting fields.

---

## 6) Found Set Isolation and Delete Rules (Replace-All Strategy)

### 6.1 Found set must represent only the target records
Before deleting, the script must isolate only the records that belong to the current parent context.

Your samples use:

~~~filemaker
Perform Find [ Restore ]
If [ Get ( FoundCount ) > 0 ]
  Delete All Records [ With dialog: Off ]
End If
~~~

Contract:
- The layout used must have a stored find request that filters correctly for the parent context (example: `$$Staff Appraisal ID`, `$$Patient Visit Lab Test ID`, `$$Payroll Item ID`, etc.).
- The AI Agent must not delete records unless it is certain the found set is the intended subset.

### 6.2 Delete confirmation must be off
Batch saves must not prompt the user during deletes:

~~~filemaker
Delete All Records [ With dialog: Off ]
~~~

---

## 7) Loop Insert Rules

### 7.1 Each iteration must create a record only when valid
Batch payloads can contain blank lines, placeholders, or items that must be skipped.
Therefore, each loop must have validation logic.

Examples of validation in your scripts:

- Save Appraisal Answers:
  - only create if question id looks valid:
    - `If [ Length ( $Appraisal Template Question ID ) < 10 ]`
- Save Payroll Item Costs:
  - only create if amount is not empty:
    - `If [ not IsEmpty ( $Amount ) ]`

Canonical rule:
- Always validate that the record item contains the minimum required fields before creating a record.

### 7.2 New record and Set Field sequence
When inserting:

~~~filemaker
New Record/Request
Set Field [ Table::Parent ID ; $$Parent ID ]
Set Field [ Table::Child Key ; $Some ID ]
Set Field [ Table::Value ; $Value ]
~~~

### 7.3 Exit loop condition
Always exit cleanly:

~~~filemaker
Exit Loop If [ $i ≥ $Total Items ]
~~~

or for JSON loops:

~~~filemaker
Exit Loop If [ $counter >= ValueCount ( $recordCount ) ]
~~~

---

## 8) Upsert Per Item Rules (Find-by-ID Strategy)

Used when:
- incoming records have stable IDs
- deleting everything is not acceptable
- category and question records must persist and be updated or created as needed

Canonical pattern:

1. Go to table layout
2. Enter Find Mode
3. Set field `ID` to incoming ID
4. Perform Find
5. If not found, create record and set required foreign keys
6. Set/update the editable fields

Example structure:

~~~filemaker
Go to Layout [ “Some Table” (Some Table) ; Animation: None ]
Enter Find Mode [ Pause: Off ]
Set Field [ Some Table::ID ; $incomingId ]
Perform Find [ ]

If [ Get ( FoundCount ) = 0 ]
  New Record/Request
  Set Field [ Some Table::ID ; $incomingId ]
  Set Field [ Some Table::Parent ID ; $$Parent ID ]
End If

Set Field [ Some Table::Name ; $name ]
Set Field [ Some Table::Description ; $description ]
~~~

---

## 9) Normalization Rules in Batch Saves

Batch saves must apply the same normalization rules as single saves, but inside loops.

### 9.1 DecodeURIComponent rule
If JSON contains URL-encoded content:

~~~filemaker
Set Variable [ $questionDescription ; Value: DecodeURIComponent ( $questionDescription ) ]
~~~

### 9.2 Date conversion rule
If a date comes in non-FileMaker format:

~~~filemaker
Set Variable [ $Date ; Value: DateInShortWordsToFMPDate ( $Date ) ]
~~~

or use the correct conversion function for that payload type.

### 9.3 Numeric duplication rule (when schema needs it)
If schema includes both text and numeric fields for the same value, set both:

~~~filemaker
Set Field [ Table::Quantity ; $Quantity ]
Set Field [ Table::Quantity Number ; $Quantity ]
~~~

---

## 10) Post-save UX Rules

### 10.1 Closing Card windows
If a Card window was opened, close it:

~~~filemaker
Perform Script [ Specified: From list ; “Close Window” ; Parameter:    ]
~~~

### 10.2 Tab restoration and navigation (when applicable)
Some batch saves return the user to a specific tab and page.

Example (Treatment Chart):

~~~filemaker
Set Variable [ $$Tab To Show ; Value: 5 ]
Perform Script [ Specified: From list ; “+++ InPatient Details Page” ; Parameter: $$Appointment ID ]
~~~

Rule:
- Only set `$$Tab To Show` and navigate if the batch save is part of a page workflow that requires it.

---

## 11) The AI Agent Output Requirements (Hard Requirements)

When asked to generate a Batch Save script, the AI Agent must produce a FileMaker script that includes:

1. `Set Error Capture [ On ]`
2. Payload ingestion and conversion to a loopable structure:
   - return-delimited list, or
   - JSON iteration
3. Safe isolation of the target found set before deleting (if Replace-All strategy is used)
4. Delete existing records with dialog off (only when Replace-All strategy is used)
5. Loop with:
   - counter increment
   - item parsing
   - validation
   - `New Record/Request`
   - `Set Field` assignments
   - exit condition
6. Window management:
   - Card window isolation when destructive operations are used
   - `Close Window` when finished
7. Optional UI restoration and navigation as required by the workflow

---

## 12) Canonical Batch Save Templates (AI Agent Reuse Templates)

### 12.1 Template A: Replace-All from a Return-Delimited List

~~~filemaker
# Save [Entity] Batch Items

Set Error Capture [ On ]

# Get payload
Set Variable [ $Items ; Value: Get ( ScriptParameter ) ]
Set Variable [ $Items ; Value: Substitute ( $Items ; "," ; ¶ ) ]
Set Variable [ $Total Items ; Value: ValueCount ( $Items ) ]

# Isolate in Card window (recommended)
New Window [ Style: Card ; Using layout: <Current Layout> ]

# Go to target layout and isolate found set
Go to Layout [ “[Target Layout]” ([Target Table]) ; Animation: None ]
Perform Find [ Restore ]

If [ Get ( FoundCount ) > 0 ]
  Delete All Records [ With dialog: Off ]
End If

# Insert new set
If [ $Total Items > 0 ]
  Loop [ Flush: Always ]
    Set Variable [ $i ; Value: $i + 1 ]
    Set Variable [ $Item ; Value: GetValue ( $Items ; $i ) ]
    Set Variable [ $Item As List ; Value: Substitute ( $Item ; "|" ; ¶ ) ]

    # Extract fields
    Set Variable [ $Child ID ; Value: GetValue ( $Item As List ; 1 ) ]
    Set Variable [ $Value ; Value: GetValue ( $Item As List ; 2 ) ]

    # Validate
    If [ not IsEmpty ( $Child ID ) ]
      New Record/Request
      Set Field [ [Target Table]::Parent ID ; $$Parent ID ]
      Set Field [ [Target Table]::Child ID ; $Child ID ]
      Set Field [ [Target Table]::Value ; $Value ]
    End If

    Exit Loop If [ $i ≥ $Total Items ]
  End Loop
End If

# Close window
Perform Script [ Specified: From list ; “Close Window” ; Parameter:    ]
~~~

### 12.2 Template B: Upsert from JSON Array

~~~filemaker
# Save [Entity] Records From JSON

Set Error Capture [ On ]

Set Variable [ $jsonData ; Value: GetValue ( GetScriptParameters ; 1 ) ]
Set Variable [ $recordKeys ; Value: JSONListKeys ( $jsonData ; "records" ) ]
Set Variable [ $counter ; Value: 0 ]

Loop [ Flush: Always ]
  Exit Loop If [ $counter >= ValueCount ( $recordKeys ) ]

  Set Variable [ $currentRecord ; Value: JSONGetElement ( $jsonData ; "records[" & $counter & "]" ) ]

  # Read fields
  Set Variable [ $id ; Value: JSONGetElement ( $currentRecord ; "id" ) ]
  Set Variable [ $name ; Value: JSONGetElement ( $currentRecord ; "name" ) ]
  Set Variable [ $desc ; Value: DecodeURIComponent ( JSONGetElement ( $currentRecord ; "description" ) ) ]

  # Upsert
  Go to Layout [ “[Target Layout]” ([Target Table]) ; Animation: None ]
  Enter Find Mode [ Pause: Off ]
  Set Field [ [Target Table]::ID ; $id ]
  Perform Find [ ]

  If [ Get ( FoundCount ) = 0 ]
    New Record/Request
    Set Field [ [Target Table]::ID ; $id ]
    Set Field [ [Target Table]::Parent ID ; $$Parent ID ]
  End If

  Set Field [ [Target Table]::Name ; $name ]
  Set Field [ [Target Table]::Description ; $desc ]

  Set Variable [ $counter ; Value: $counter + 1 ]
End Loop

Go to Layout [ original layout ; Animation: None ]
~~~

---

## 13) Clarifications the AI Agent Must Not Guess

These must be defined per batch save use case:

1. What the stored find request filters by when using `Perform Find [ Restore ]`.
   - The script must know the parent context variable(s) that the restore find depends on.

2. Whether the batch save is Replace-All or Upsert.
   - The AI Agent must not choose randomly.

3. The delimiter rules for non-JSON payloads.
   - Example: `|`, `^`, `vvvvvvvvvv`, `ITEMS:`, `TREATMENT_CHART:`.

The AI Agent must replicate the delimiter protocol exactly as required by the calling UI.

---

# CauferoAppStarter Documentation
## Text Fields in Pages (How to Add Them and How They Work)

> Purpose of this doc  
> Train an AI Agent to correctly add, render, prefill, read, and save **text fields** inside CauferoAppStarter WebViewer pages (HTML built in FileMaker scripts).

---

## 1) What a "Text Field" Means in CauferoAppStarter

In CauferoAppStarter pages, a "text field" is a standard HTML field element rendered in the WebViewer, then read by JavaScript, then sent back to FileMaker via `FileMaker.PerformScript()`.

Text fields include:

1. **Single line text input**
   - `<input type='text' ...>`
2. **Multi-line text input**
   - `<textarea ...>...</textarea>`
3. **Read-only or computed text**
   - `<input type='text' ... disabled>`
4. **Text inputs that are controlled by another UI behavior**
   - Example: a date field still uses `<input type='text'>` but is controlled by Flatpickr

This document focuses on text fields only.

---

## 2) The Full Lifecycle of a Text Field

Every text field follows the same lifecycle:

### Step A: FileMaker prepares the value
- Data is fetched from SQL and assigned to FileMaker variables like:
  - `$Staff Number`, `$First Name`, `$Residential Address`, etc.
- Any special formatting is handled in FileMaker:
  - Example: `ConvertLineBreakMarkersToReturns ( ... )`

### Step B: FileMaker injects the value into the HTML
- The value is injected into the `value='...'` attribute for inputs
- Or inserted between `<textarea> ... </textarea>`

### Step C: User edits the field in the WebViewer
- User types into the input or textarea

### Step D: JavaScript reads the value
- Example:
  - `document.getElementById('first_name').value`

### Step E: JavaScript builds a parameter string and calls FileMaker
- Parameters are joined with a delimiter
- Example delimiter used in your page:
  - `|`
- Then:
  - `FileMaker.PerformScript('Save Staff', parameters)`

### Step F: FileMaker receives, parses, and saves
- The FileMaker script receiving the parameters splits and assigns fields

---

## 3) Core Rules (Non-Negotiable)

### 3.1 Every text field MUST have a stable `id`
The `id` is the primary key used by JavaScript.

Bad:
- random IDs
- duplicate IDs
- IDs that change

Good:
- `staff_number`
- `first_name`
- `residential_address`

### 3.2 Every text field MUST have a matching `<label for='...'>`
The label improves usability and helps the AI Agent place fields consistently.

Example:
~~~html
<label for='phone'>Phone</label>
<input type='text' id='phone' value='...'>
~~~

### 3.3 Always use one of the standard layout containers
From your page:
- `form-grid1`
- `form-grid2`
- `form-grid3`
- `form-group-2`
- `combined-inputs-container`

These containers control spacing and responsiveness.

### 3.4 Parameter delimiter risk: encode any value that can break the delimiter
Your save script joins values using `|`.

If a text field can contain `|`, line breaks, or other special characters, you must encode it with `encodeURIComponent()` before joining.

Your sample already does this for:
- `residential_address`
- `postal_address`
- `role`

That is correct behavior.

---

## 4) Field Naming Convention

Use these rules for IDs:

- **snake_case**
- **lowercase**
- **descriptive**
- match FileMaker field meaning

Examples from your page:
- `staff_number`
- `first_name`
- `other_names`
- `last_name`
- `hometown`
- `phone`
- `email`
- `gps`
- `bank`
- `branch`
- `account_number`
- `residential_address`
- `postal_address`

---

## 5) Adding a Single Line Text Field

### 5.1 Pattern
~~~html
<div class='form-group-2'>
  <label for='FIELD_ID'>LABEL</label>
  <input type='text' id='FIELD_ID' placeholder='OPTIONAL_PLACEHOLDER' value='PRE_FILLED_VALUE'>
</div>
~~~

### 5.2 Example from your page (Phone)
~~~html
<div class='form-group-2'>
  <label for='phone'>Phone</label>
  <input type='text' id='phone' value='{{PHONE}}'>
</div>
~~~

### 5.3 FileMaker injection pattern
In FileMaker, build the HTML string like:
~~~filemaker
Set Variable [ $Tab2 HTML ;
"
...
<input type='text' id='phone' value='" & $Phone & "'>
...
"
]
~~~

### 5.4 When to use `placeholder`
Use placeholder when you expect the field can be empty and you want guidance:
~~~html
<input type='text' id='hometown' placeholder='Type hometown...' value='{{HOMETOWN}}'>
~~~

---

## 6) Adding a Multi-Line Text Field (Textarea)

Textarea is used for longer text, addresses, notes, and descriptions.

### 6.1 Pattern
~~~html
<div class='form-group-2'>
  <label for='FIELD_ID'>LABEL</label>
  <textarea id='FIELD_ID' placeholder='OPTIONAL_PLACEHOLDER'>PRE_FILLED_TEXT</textarea>
</div>
~~~

### 6.2 Example from your page (Residential Address)
~~~html
<div class='form-group-2'>
  <label for='residential_address'>Residential Address</label>
  <textarea id='residential_address' placeholder='Type the residential address here...'>{{RESIDENTIAL_ADDRESS}}</textarea>
</div>
~~~

### 6.3 Pre-fill rule for textarea
- Inputs use `value='...'`
- Textarea uses inner text between tags

Correct:
~~~html
<textarea id='postal_address'>{{POSTAL_ADDRESS}}</textarea>
~~~

Wrong:
~~~html
<textarea id='postal_address' value='{{POSTAL_ADDRESS}}'></textarea>
~~~

### 6.4 Line breaks rule
If FileMaker stores text with custom break markers, convert them before injecting into HTML.

Your page does:
~~~filemaker
Set Variable [ $Residential Address ;
  Value: ConvertLineBreakMarkersToReturns ( GetValue ( $Link Record As List ; 14 ) )
]
~~~

This is good. It means the injected textarea text looks like real address lines.

---

## 7) Adding a Read-Only Text Field (Computed Display)

A read-only text field is a normal text input that the user cannot edit.

### 7.1 Pattern
~~~html
<div class='form-group-2'>
  <label for='age'>Age</label>
  <input type='text' id='age' value='{{AGE}}' disabled>
</div>
~~~

### 7.2 Example from your page (Age)
~~~html
<div class='form-group-2'>
  <label for='age'>Age</label>
  <input type='text' id='age' placeholder='0 years' value='{{AGE}}' disabled>
</div>
~~~

### 7.3 Save rule
Disabled fields are usually not read and saved. If you do read them, they are often derived values anyway.

In your save logic, you do not read `age`. That is correct.

---

## 8) Adding Text Fields Inside "Combined Inputs" UI

Your page has a "Full Name" area where one label covers multiple inputs arranged together.

### 8.1 Structure used in your page
~~~html
<div class='form-group form-group-2'>
  <label for='size'>Full Name</label>

  <div class='combined-inputs-container'>
    <select class='combined-inputs-dropdown' id='title' ...>...</select>
    <input class='combined-inputs-input' type='text' id='first_name' ...>
    <input class='combined-inputs-input' type='text' id='other_names' ...>
    <input class='combined-inputs-input' type='text' id='last_name' ...>
  </div>
</div>
~~~

### 8.2 Text-field rule inside combined inputs
- Each input still needs its own unique `id`
- Each input still must be read individually in `saveItemInfo()`

---

## 9) Reading Text Field Values in JavaScript

### 9.1 Standard rule
Every text field is read using:
~~~js
const value = document.getElementById('FIELD_ID').value;
~~~

### 9.2 Examples from your save function
From your page:
~~~js
const staff_number = document.getElementById('staff_number').value;
const first_name = document.getElementById('first_name').value;
const last_name = document.getElementById('last_name').value;
const phone = document.getElementById('phone').value;
const email = document.getElementById('email').value;
const gps = document.getElementById('gps').value;
const bank = document.getElementById('bank').value;
~~~

### 9.3 Encoding rule (critical)
If a field can contain:
- line breaks
- pipes `|`
- commas are fine
- long paragraphs
then encode it before joining:

~~~js
const residential_address = encodeURIComponent(
  document.getElementById('residential_address').value
);
~~~

Your page does exactly this. That is correct.

---

## 10) Saving Text Field Values (Parameter Construction)

### 10.1 The standard pattern in your pages
1. Collect field values
2. Join them with `|`
3. Call FileMaker script

~~~js
const parameters = [ a, b, c, d ].join('|');
FileMaker.PerformScript('Save Staff', parameters);
~~~

### 10.2 Why the delimiter matters
Your delimiter is `|`.

If a user enters `|` into any text field, FileMaker will mis-split the parameter list and your data will land in wrong variables.

That is why encoding is mandatory for any risky field.

---

## 11) Where Text Fields Live in the Page Layout

### 11.1 Tabs contain fields
Your page uses tabs (`Details`, `Contact`, etc.). Each tab HTML can contain text fields.

Examples:
- Tab 1 (Details) contains:
  - `staff_number`
  - `first_name`, `other_names`, `last_name`
  - `hometown`
- Tab 2 (Contact) contains:
  - `phone`, `email`, `gps`
  - `residential_address`, `postal_address`
- Tab 5 (Remunerations) contains:
  - `bank`, `branch`, `account_number`

### 11.2 Use grids to control width and alignment
From your page:
- `form-grid3` used for 3 columns
- `form-grid2` used for 2 columns

Example:
~~~html
<div class='form-grid3'>
  <div class='form-group-2'>...</div>
  <div class='form-group-2'>...</div>
  <div class='form-group-2'>...</div>
</div>
~~~

---

## 12) Text Fields That Look Like Text Fields but Behave Differently

Some fields are still `<input type='text'>` but are controlled by scripts.

### 12.1 Date fields (still text inputs)
In your page:
- `dob`
- `from_date`
- `to_date`

They are text inputs, but Flatpickr controls them and converts to MySQL date format via `getMySQLFormattedDate()`.

This doc treats them as text fields structurally, but their behavior belongs to a separate doc for "Date Fields".

---

## 13) Quality Checklist for the AI Agent

When the AI Agent adds a new text field, it MUST verify:

1. The field is placed inside the correct grid container
2. The field has a stable `id`
3. The label `for` matches the `id`
4. The field is prefilled properly from FileMaker variables
5. The field is read in JS with `getElementById(id).value`
6. The field is included in `parameters` in the correct order
7. If the field can contain risky characters, it is encoded with `encodeURIComponent()`
8. If it is a textarea, prefill text is inside the element body
9. If it is read-only, it uses `disabled` and is excluded from save unless explicitly required

---

## 14) Copy-Paste Templates

### 14.1 Single line text field template
~~~html
<div class='form-group-2'>
  <label for='FIELD_ID'>LABEL</label>
  <input type='text' id='FIELD_ID' placeholder='PLACEHOLDER' value='{{VALUE}}'>
</div>
~~~

### 14.2 Textarea template
~~~html
<div class='form-group-2'>
  <label for='FIELD_ID'>LABEL</label>
  <textarea id='FIELD_ID' placeholder='PLACEHOLDER'>{{VALUE}}</textarea>
</div>
~~~

### 14.3 Disabled text field template
~~~html
<div class='form-group-2'>
  <label for='FIELD_ID'>LABEL</label>
  <input type='text' id='FIELD_ID' value='{{VALUE}}' disabled>
</div>
~~~

### 14.4 JS read template
~~~js
const FIELD_ID = document.getElementById('FIELD_ID').value;
~~~

### 14.5 JS encode template
~~~js
const FIELD_ID = encodeURIComponent(document.getElementById('FIELD_ID').value);
~~~

---

## 15) Common Failure Modes and Fixes

### 15.1 Field does not save
Cause:
- Field ID not included in `saveItemInfo()`

Fix:
- Add:
  - `const x = document.getElementById('your_field').value;`
  - include `x` in the joined parameter list

### 15.2 Data lands in wrong FileMaker variable
Cause:
- A user typed `|` into a field that was not encoded

Fix:
- Encode that field with `encodeURIComponent()`

### 15.3 Textarea loses line breaks
Cause:
- FileMaker value injected without converting markers properly
- Or the value is stored with a custom marker system

Fix:
- Convert in FileMaker before injection
- Example:
  - `ConvertLineBreakMarkersToReturns ( ... )`

### 15.4 Field displays wrong prefill value
Cause:
- Wrong FileMaker variable injected
- Or quoting breaks the HTML attribute

Fix:
- Ensure the correct FileMaker variable is used
- If you have apostrophes and quoting issues, switch to safe escaping rules in your HTML builder (see Clarifications section)

---

## 16) Clarifications Needed (Ambiguities the AI Agent Cannot Guess)

These must be clarified by you because the AI Agent cannot safely assume them.

1. **Encoding policy**
   - Right now, you only encode some fields (addresses, role).
   - Question: Should the standard be:
     - encode only multi-line fields
     - or encode every text field before joining
   - Recommendation for safety:
     - encode every field that can contain `|` or line breaks
     - optionally encode all fields to eliminate delimiter risk completely

2. **HTML escaping policy for values**
   - Values injected into `value='...'` can break HTML if they contain `'` or special characters.
   - Question: Do you have an existing escaping function or standard for:
     - escaping `'` to `&#39;`
     - escaping `&` to `&amp;`
     - escaping `<` and `>`
   - If there is a standard, the AI Agent must follow it.

3. **Canonical "Text Field Component"**
   - In the sample, text fields are raw HTML tags.
   - Question: Do you have a reusable component function or XML snippet for text fields, or should the AI Agent always generate raw HTML tags inside `$TabX HTML`?

4. **Validation**
   - Question: Do you have a standard validation style for required text fields?
     - Example: required attribute, inline red border, message block, or server-side only

5. **Field order in parameters**
   - The save function depends on strict parameter order.
   - Question: Is there a documented contract for the parameter order per page, or does each page define its own order?

---

## 17) Minimum Example (End to End)

This is an end-to-end reference for adding one new text field called "Nickname".

### 17.1 FileMaker: set variable
~~~filemaker
Set Variable [ $Nickname ; Value: GetValue ( $Link Record As List ; 99 ) ]
~~~

### 17.2 HTML: add field to a grid
~~~html
<div class='form-group-2'>
  <label for='nickname'>Nickname</label>
  <input type='text' id='nickname' placeholder='Optional nickname' value='{{NICKNAME}}'>
</div>
~~~

### 17.3 JS: read field value
~~~js
const nickname = document.getElementById('nickname').value;
~~~

### 17.4 JS: add to parameters (encode if needed)
~~~js
const nicknameSafe = encodeURIComponent(nickname);
const parameters = [ staff_number, first_name, nicknameSafe ].join('|');
FileMaker.PerformScript('Save Staff', parameters);
~~~

---

## 18) Summary for the AI Agent (Do This Every Time)

When adding a text field:

1. Decide if it is input or textarea
2. Choose a clean snake_case `id`
3. Add `<label for='id'>`
4. Add the field inside a `form-gridX` container
5. Pre-fill it using the correct FileMaker variable
6. Read it in JS using `getElementById(id).value`
7. Encode it if it can break parameter joining
8. Append it to the `parameters` list in correct order
9. Confirm FileMaker save script expects it in that position

---


# CauferoAppStarter Documentation
## Number Fields and How They Are Implemented and Used

> Purpose of this doc  
> Train an AI Agent to correctly implement **number fields** in CauferoAppStarter WebViewer pages, including:
> 1) The HTML patterns used for numeric inputs  
> 2) Formatting rules (integers, decimals, currency, percent)  
> 3) Validation rules (min, max, required, decimals allowed)  
> 4) How values are preloaded from FileMaker variables  
> 5) How values are collected in JavaScript and sent back to FileMaker  
> 6) How the FileMaker Save script should parse and store the number correctly  
> 7) Common pitfalls and correct fixes

This document is written to match the architectural patterns in your sample page:
- fields are identified by stable `id`
- values are preloaded by string injection into HTML
- Save functions collect values via `document.getElementById('...').value`
- script parameters are sent using `|` delimiter: `[ ... ].join('|')`
- some values must be encoded to protect delimiter safety

Even though the sample page does not show a dedicated numeric input field, the AI Agent must implement number fields using the same CauferoAppStarter conventions.

---

## 1) What a Number Field Means in CauferoAppStarter

A number field is any input where the intended stored value is numeric:
- quantity
- price
- cost
- percentage
- duration (minutes/hours)
- scores
- IDs that are numeric (only if they behave like numbers, otherwise treat as text)
- rates (tax, discount)
- balances and totals

In this system, number fields are:
- rendered as HTML `<input>`
- validated in JS (recommended)
- parsed and validated again in FileMaker (required)
- passed as script parameters joined by `|`

---

## 2) Mandatory Rules (Non-Negotiable)

### 2.1 Every number field MUST have a stable `id`
All Save and action scripts depend on IDs.

Example:
- `quantity`
- `unit_price`
- `discount_percent`

### 2.2 Number fields MUST be validated before script call (recommended)
At minimum:
- prevent empty required numeric fields
- prevent non-numeric text from being sent

FileMaker validation still must exist as the final gate.

### 2.3 Never trust client input
Even if JS validates, FileMaker must validate again.
Users can paste weird text, and WebViewer can be manipulated.

### 2.4 Delimiter safety
Numbers usually do not contain `|`, so encoding is often optional.
But if you allow formatted values like `1,200.50`, currency symbols, or spaces, your parsing must be strict.

Standard approach:
- send raw numeric strings only (recommended)
- avoid commas and symbols in the actual input value

---

## 3) HTML Implementation Patterns

There are two acceptable HTML patterns:

### Pattern A: `<input type='number'>` (preferred)
Use when you want native numeric keyboard support on mobile and browser-level constraints.

~~~html
<div class='form-group-2'>
  <label for='quantity'>Quantity</label>
  <input type='number' id='quantity' placeholder='0' value='{{QUANTITY}}' min='0' step='1' inputmode='numeric'>
</div>
~~~

### Pattern B: `<input type='text'>` with numeric rules (fallback)
Use when:
- you want custom formatting
- or your WebViewer environment behaves inconsistently with `type='number'`

~~~html
<div class='form-group-2'>
  <label for='unit_price'>Unit Price</label>
  <input type='text' id='unit_price' placeholder='0.00' value='{{UNIT_PRICE}}' inputmode='decimal'>
</div>
~~~

### Rule
If you do not need special formatting:
- use `type='number'`

If you need formatting, thousands separators, or currency display:
- use `type='text'` and implement strict parsing.

---

## 4) Numeric Types You Must Support

The AI Agent must be able to implement these common numeric types:

### 4.1 Integer
Examples:
- quantity
- number of children
- count fields

HTML:
~~~html
<input type='number' id='qty' step='1' min='0'>
~~~

JS parsing:
~~~js
const qty = parseInt(document.getElementById('qty').value, 10);
~~~

### 4.2 Decimal
Examples:
- prices
- weights
- measurements

HTML:
~~~html
<input type='number' id='price' step='0.01' min='0'>
~~~

JS parsing:
~~~js
const price = parseFloat(document.getElementById('price').value);
~~~

### 4.3 Currency
Two approaches exist:

#### Approach 1: Store raw decimal, display label outside input (recommended)
~~~html
<label for='amount'>Amount ({{CURRENCY}})</label>
<input type='number' id='amount' step='0.01' min='0'>
~~~

#### Approach 2: Store formatted string and parse (not recommended unless required)
If you allow `1,200.00`, then you must strip commas before parsing.

### 4.4 Percentage
Store as numeric percent or decimal fraction depending on your database standard.

Common percent-as-0-100 approach:
~~~html
<label for='discount'>Discount (%)</label>
<input type='number' id='discount' step='0.01' min='0' max='100'>
~~~

---

## 5) Preloading Number Values (FileMaker Side)

### 5.1 Standard: preload numeric values as plain text
When inserting into HTML, numeric values must be converted to text.

Recommended pattern:
~~~filemaker
Set Variable [ $Quantity ; Value: GetAsText ( table::Quantity ) ]
Set Variable [ $Unit Price ; Value: GetAsText ( table::UnitPrice ) ]
~~~

### 5.2 Default values
If empty, use `0` or blank based on field type and UX requirements:

~~~filemaker
Set Variable [ $Quantity ; Value: DefaultIfEmpty ( GetAsText ( table::Quantity ) ; "0" ) ]
~~~

### Rule
Do not preload values with commas or currency symbols unless your JS is explicitly designed to parse them.

---

## 6) Collecting Number Values (JavaScript)

### 6.1 Basic collection pattern (string)
Even if you use `type='number'`, `.value` returns a string.
You can send the raw string to FileMaker if it is already clean.

~~~js
const quantity_raw = document.getElementById('quantity').value;
~~~

### 6.2 Clean parse pattern (recommended)
Always normalize before sending:

#### Integer normalize
~~~js
function getIntValue(id) {
  const v = document.getElementById(id).value.trim();
  if (!v) return null;
  const n = parseInt(v, 10);
  return Number.isFinite(n) ? n : null;
}
~~~

#### Decimal normalize
~~~js
function getFloatValue(id) {
  const v = document.getElementById(id).value.trim();
  if (!v) return null;
  const n = parseFloat(v);
  return Number.isFinite(n) ? n : null;
}
~~~

### 6.3 Sending values to FileMaker
Your system joins parameters with `|`.
Numbers must be converted back to strings.

~~~js
const qty = getIntValue('quantity');
const price = getFloatValue('unit_price');

const parameters = [ qty, price ].join('|');
FileMaker.PerformScript('Save Something', parameters);
~~~

### Rule
If a value is `null`, decide your system standard:
- send empty string
- or send `0`
- or send explicit keyword like `NULL`

Your current pages commonly send strings. The AI Agent needs your canonical standard clarified.

---

## 7) Validation Rules (JS + UX)

### 7.1 Required numeric field
~~~js
function requireNumber(id, label) {
  const v = document.getElementById(id).value.trim();
  if (!v) {
    alert(label + ' is required.');
    return false;
  }
  const n = parseFloat(v);
  if (!Number.isFinite(n)) {
    alert(label + ' must be a number.');
    return false;
  }
  return true;
}
~~~

### 7.2 Min/Max constraints
~~~js
function enforceRange(id, label, min, max) {
  const v = document.getElementById(id).value.trim();
  if (!v) return true;
  const n = parseFloat(v);
  if (!Number.isFinite(n)) return false;
  if (n < min || n > max) {
    alert(label + ' must be between ' + min + ' and ' + max + '.');
    return false;
  }
  return true;
}
~~~

### 7.3 Integer only
~~~js
function enforceInteger(id, label) {
  const v = document.getElementById(id).value.trim();
  if (!v) return true;
  const n = Number(v);
  if (!Number.isInteger(n)) {
    alert(label + ' must be a whole number.');
    return false;
  }
  return true;
}
~~~

### Rule
Use JS validation as the first wall.
FileMaker validation remains the final wall.

---

## 8) FileMaker Parsing and Storage Rules

### 8.1 FileMaker receives parameters as text
Numbers arrive inside a delimited string:
- `param1|param2|param3`

So FileMaker must:
1. split parameters
2. validate numeric
3. store into numeric fields

### 8.2 FileMaker numeric conversion
Recommended pattern:
- `GetAsNumber ( value )` for numeric conversion

Example:
~~~filemaker
Set Variable [ $qtyText ; Value: GetValue ( $params ; 1 ) ]
Set Variable [ $qty ; Value: GetAsNumber ( $qtyText ) ]
~~~

### 8.3 Empty handling
If the string is empty:
- decide whether to store 0 or leave empty

Example:
~~~filemaker
Set Variable [ $qty ;
  Value:
    Case (
      IsEmpty ( $qtyText ) ; "" ;
      GetAsNumber ( $qtyText )
    )
]
~~~

### Rule
The Save script must enforce:
- numeric fields do not store non-numeric strings
- min/max rules that matter to business logic

---

## 9) Formatting Rules (Display vs Stored Value)

### 9.1 Stored value should be raw numeric
Store:
- `1200.5`
not:
- `1,200.50`
not:
- `GHS 1,200.50`

### 9.2 Display formatting should be optional UI sugar
If you want formatted display, use one of these:
- format on blur event (and strip formatting before save)
- display formatted value in a separate label/span next to input
- store raw number and render formatted totals elsewhere (KPI cards, summary sections, tables)

The AI Agent must not introduce formatted input values unless instructed, because it complicates parsing.

---

## 10) Implementation Templates (Copy-Paste)

### 10.1 Integer field template
HTML:
~~~html
<div class='form-group-2'>
  <label for='quantity'>Quantity</label>
  <input type='number' id='quantity' placeholder='0' value='{{QUANTITY}}' min='0' step='1' inputmode='numeric'>
</div>
~~~

JS read:
~~~js
const quantity = document.getElementById('quantity').value.trim();
~~~

### 10.2 Decimal field template
HTML:
~~~html
<div class='form-group-2'>
  <label for='unit_price'>Unit Price</label>
  <input type='number' id='unit_price' placeholder='0.00' value='{{UNIT_PRICE}}' min='0' step='0.01' inputmode='decimal'>
</div>
~~~

JS read:
~~~js
const unit_price = document.getElementById('unit_price').value.trim();
~~~

### 10.3 Percent field template
HTML:
~~~html
<div class='form-group-2'>
  <label for='discount_percent'>Discount (%)</label>
  <input type='number' id='discount_percent' placeholder='0' value='{{DISCOUNT}}' min='0' max='100' step='0.01' inputmode='decimal'>
</div>
~~~

JS read:
~~~js
const discount_percent = document.getElementById('discount_percent').value.trim();
~~~

---

## 11) Quality Checklist for the AI Agent

When implementing a number field, the AI Agent must verify:

### HTML correctness
1. input has stable `id`
2. label exists and points to id
3. appropriate type is chosen:
   - `number` preferred
   - `text` only when needed
4. step/min/max reflect numeric type

### Data correctness
5. FileMaker preloads raw numeric text
6. default is applied if required (0 or blank)

### Save correctness
7. JS reads value by id
8. JS validates numeric (recommended)
9. parameters are joined in correct order

### FileMaker correctness
10. Save script converts using `GetAsNumber`
11. business rules validation enforced (min/max, required)

---

## 12) Common Failure Modes and Fixes

### 12.1 User enters commas and save fails
Cause:
- you allowed `1,200` and FileMaker `GetAsNumber` may interpret it unexpectedly depending on locale
Fix:
- prevent commas in input, or strip commas in JS before sending:
~~~js
const raw = document.getElementById('amount').value;
const cleaned = raw.replace(/,/g, '').trim();
~~~

### 12.2 Empty numeric field becomes 0 but should be blank
Cause:
- `GetAsNumber("")` results in 0
Fix:
- handle empty explicitly before conversion

### 12.3 Decimal field stores too many decimals
Cause:
- no rounding rule
Fix:
- round in FileMaker or JS:
~~~js
const rounded = Math.round(n * 100) / 100;
~~~
Use the correct factor based on required decimals.

### 12.4 Negative values allowed accidentally
Cause:
- missing `min='0'`
Fix:
- set min in HTML and validate in FileMaker

---

## 13) Clarifications Needed (Ambiguities the AI Agent Cannot Guess)

To make number field implementation perfect, confirm these:

1. **Null vs 0 standard**
   - When a number field is empty, do you store:
     - empty
     - 0
     - or keep existing value

2. **Decimal precision standard**
   - For currency, do you always enforce 2 decimals?
   - For measurements, do you allow more?

3. **Thousands separator policy**
   - Do you allow users to type commas?
   - If yes, should JS strip them automatically?

4. **Locale and decimal separator**
   - Do you assume dot `.` as decimal separator always?

5. **Validation responsibility**
   - Which rules must be enforced in JS vs FileMaker?
   - Example: min/max, required, rounding, negative restrictions.

---

## 14) Implementation Standard (What the AI Agent Must Produce)

When asked to implement a number field in CauferoAppStarter, the AI Agent must produce:

1. HTML numeric input with stable id and label
2. appropriate numeric constraints (step, min, max)
3. preload raw numeric values from FileMaker variables
4. JS logic that reads the value and validates it (recommended)
5. JS parameter packaging using `|`
6. FileMaker Save script conversion and validation using numeric-safe rules

---

# CauferoAppStarter Documentation
## Datepickers (Flatpickr) How They Are Implemented and Used

> Purpose of this doc  
> Train an AI Agent to correctly implement **datepickers** in CauferoAppStarter WebViewer pages using **Flatpickr**, including:
> 1) How date fields are rendered in HTML  
> 2) How calendar icons are wired to open the picker  
> 3) How existing values are preloaded and shown in human-readable format  
> 4) How dates are converted back into **MySQL format (Y-m-d)** before sending to FileMaker  
> 5) How date ranges (From/To) are used to filter data and trigger FileMaker scripts  
> 6) Required IDs, patterns, and failure mode fixes

This documentation is based on your sample page, which uses Flatpickr from CDN and implements:
- `dob` (Date of Birth)
- `from_date` (Attendance filter)
- `to_date` (Attendance filter)
- Icon IDs:
  - `dob_icon`
  - `from_date_icon`
  - `to_date_icon`
- Helper function:
  - `getMySQLFormattedDate(fieldId)`

---

## 1) What a Datepicker Means in CauferoAppStarter

A datepicker is an enhanced text input that:
- displays a human-readable date string on-screen
- stores (or converts to) a database-friendly date string when saving or filtering
- is opened by:
  - clicking the input
  - clicking a calendar icon next to it

In CauferoAppStarter, datepickers are built using:
- Flatpickr CSS + JS included in the HTML `<head>`
- A shared initialization script that:
  - binds flatpickr to selected fields
  - converts initial MySQL dates into readable date strings
  - provides a helper to convert readable dates back into MySQL format

---

## 2) Required Dependencies (Must Exist in the Full Page)

Your full page includes these CDN resources:

~~~html
<link rel='stylesheet' href='https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css'>
<script src='https://cdn.jsdelivr.net/npm/flatpickr'></script>
~~~

### Rules
1. Flatpickr CSS must be loaded before rendering fields (for correct UI)
2. Flatpickr JS must be loaded before the initialization script runs
3. The initialization script must run after DOM content is ready

---

## 3) HTML Implementation Pattern

Each datepicker field uses:
- an `<input type='text'>`
- a stable `id`
- an icon element placed absolutely within the same container

### 3.1 Canonical HTML for a date input with icon
From your page:

~~~html
<div style='position: relative;'>
  <input type='text' id='dob' value='{{DATE_VALUE}}' />
  <i class='fas fa-calendar-alt' id='dob_icon'
     style='position: absolute; right: 10px; top: 50%; transform: translateY(-50%); color: #666; cursor: pointer;'></i>
</div>
~~~

Same structure for range filters:

~~~html
<div style='position: relative;'>
  <input type='text' id='from_date' value='{{FROM_DATE_MYSQL}}' />
  <i class='fas fa-calendar-alt' id='from_date_icon' style='position: absolute; right: 10px; top: 50%; transform: translateY(-50%); color: #666; cursor: pointer;'></i>
</div>

<div style='position: relative;'>
  <input type='text' id='to_date' value='{{TO_DATE_MYSQL}}' />
  <i class='fas fa-calendar-alt' id='to_date_icon' style='position: absolute; right: 10px; top: 50%; transform: translateY(-50%); color: #666; cursor: pointer;'></i>
</div>
~~~

### 3.2 Mandatory IDs
A datepicker field must have:
- input ID: `FIELD_ID`
- icon ID: `FIELD_ID_icon` (or explicit mapping inside JS)

In your system you use explicit mapping objects, so the icon ID does not have to follow a naming formula, but it must match what JS expects.

---

## 4) FileMaker Preprocessing of Date Values

Your page uses FileMaker variables to generate two styles of date values:

### 4.1 Date of Birth (DOB)
DOB appears to be stored as a date text string (likely in MySQL format already) and injected directly:
~~~filemaker
Set Variable [ $Date of Birth ; Value: GetValue ( $Link Record As List ; 8 ) ]
~~~

### 4.2 Date range filter values (From / To)
You set defaults and also create MySQL versions:

~~~filemaker
Set Variable [ $From Date ; Value: DefaultIfEmpty ( GetValue ( GetScriptParameters ; 3 ) ; Get ( CurrentDate ) ) ]
Set Variable [ $To Date ; Value: DefaultIfEmpty ( GetValue ( GetScriptParameters ; 4 ) ; Get ( CurrentDate ) ) ]

Set Variable [ $From Date MySQL ; Value: FMPDateTextToMySQLDate ( $From Date ) ]
Set Variable [ $To Date MySQL ; Value: FMPDateTextToMySQLDate ( $To Date ) ]
~~~

### Rule
When the page loads, date fields can start with a MySQL-style date string:
- `YYYY-MM-DD`
Then JS converts it into a readable format for the user.

---

## 5) Datepicker Initialization Script (Flatpickr)

Your implementation uses a single script that:
- declares date fields to initialize
- converts initial MySQL date strings to readable date strings
- creates flatpickr instances
- binds the calendar icon click to open the picker

### 5.1 Canonical initialization script (from your page)
This is the core pattern:

~~~js
document.addEventListener('DOMContentLoaded', function () {
  const dateFields = [
    { id: 'dob', iconId: 'dob_icon' },
    { id: 'from_date', iconId: 'from_date_icon' },
    { id: 'to_date', iconId: 'to_date_icon' }
  ];

  function formatMySQLDateToReadable(mysqlDate) {
    const parsedDate = flatpickr.parseDate(mysqlDate, 'Y-m-d');
    return parsedDate ? flatpickr.formatDate(parsedDate, 'l, j F, Y') : '';
  }

  dateFields.forEach(field => {
    const dateInput = document.getElementById(field.id);
    const dateIcon = document.getElementById(field.iconId);

    const initialDate = dateInput.value && dateInput.value !== '--' ? dateInput.value : null;

    if (initialDate) {
      dateInput.value = formatMySQLDateToReadable(initialDate);
    }

    const flatpickrInstance = flatpickr(dateInput, {
      dateFormat: 'l, j F, Y',
      defaultDate: initialDate || null,
      allowInput: true,
      onChange: function (selectedDates, dateStr) {
        dateInput.value = dateStr;
      }
    });

    if (dateIcon) {
      dateIcon.addEventListener('click', function () {
        flatpickrInstance.open();
      });
    }
  });
});
~~~

### 5.2 What this script guarantees
1. If the input value starts as `YYYY-MM-DD`, it becomes readable like:
   - `Sunday, 7 January, 2026` (example format)
2. The flatpickr instance uses readable date format for UI
3. Clicking the icon opens the datepicker
4. The input value remains readable after picking

---

## 6) Converting User-Selected Dates Back to MySQL Format

Your system provides a helper:

~~~js
function getMySQLFormattedDate(fieldId) {
  const dateInput = document.getElementById(fieldId).value;

  if (!dateInput || dateInput === '--') {
    return null;
  }

  const parsedDate = flatpickr.parseDate(dateInput, 'l, j F, Y');
  return parsedDate ? flatpickr.formatDate(parsedDate, 'Y-m-d') : null;
}
~~~

### Rule
Whenever the system needs to pass a date back to FileMaker:
- convert readable input value to `YYYY-MM-DD`
- pass that string in script parameters

---

## 7) How Datepicker Values Are Used in Page Actions

There are two common action types:

### 7.1 Save action (Saving entity data)
In your page, DOB is captured during Save as MySQL date:

~~~js
const dob = getMySQLFormattedDate('dob');
~~~

Then included in save parameters.

### 7.2 Filter action (Date range filtering)
Your attendance section reads:

~~~js
function getAttendance() {

  const from_date = getMySQLFormattedDate('from_date');
  const to_date = getMySQLFormattedDate('to_date');

  const parameters = [ from_date, to_date ].join('|');

  FileMaker.PerformScript('Get Staff Attendance', parameters);
}
~~~

### Rule
For date range filters:
- both dates must be converted to MySQL format
- parameters are sent to FileMaker script that regenerates the page or subtable

---

## 8) Standard ID and Icon Wiring Rules

### 8.1 Input ID rule
Every datepicker must have a unique, stable input ID.

### 8.2 Icon ID rule
Every datepicker may have an icon ID.
If present, it must be mapped in the `dateFields` array.

Example mapping entry:
~~~js
{ id: 'my_date_field', iconId: 'my_date_field_icon' }
~~~

### 8.3 If icon is missing
The script checks:
~~~js
if (dateIcon) { ... }
~~~
So icon is optional, but recommended.

---

## 9) Copy-Paste Implementation Templates

### 9.1 Template: Add one datepicker field
HTML:
~~~html
<div class='form-group-2'>
  <label for='my_date'>My Date</label>
  <div style='position: relative;'>
    <input type='text' id='my_date' value='{{MY_DATE_MYSQL}}' />
    <i class='fas fa-calendar-alt' id='my_date_icon'
       style='position: absolute; right: 10px; top: 50%; transform: translateY(-50%); color: #666; cursor: pointer;'></i>
  </div>
</div>
~~~

JS additions:
1) Add entry in the dateFields array:
~~~js
{ id: 'my_date', iconId: 'my_date_icon' }
~~~

2) Read value when needed:
~~~js
const my_date_mysql = getMySQLFormattedDate('my_date');
~~~

### 9.2 Template: Date range picker (From / To)
HTML:
~~~html
<input type='text' id='from_date' value='{{FROM_DATE_MYSQL}}' />
<i class='fas fa-calendar-alt' id='from_date_icon' ...></i>

<input type='text' id='to_date' value='{{TO_DATE_MYSQL}}' />
<i class='fas fa-calendar-alt' id='to_date_icon' ...></i>
~~~

JS usage:
~~~js
const from_date = getMySQLFormattedDate('from_date');
const to_date = getMySQLFormattedDate('to_date');
~~~

---

## 10) Quality Checklist for the AI Agent

When implementing a datepicker, the AI Agent must verify:

### Dependencies
1. Flatpickr CSS is included
2. Flatpickr JS is included

### HTML
3. The field is an `<input type='text'>`
4. The input has a stable ID
5. The icon exists and has a stable ID (recommended)

### Initialization
6. Field ID is included in `dateFields` array
7. Script runs on `DOMContentLoaded`

### Formatting
8. Initial MySQL date is converted to readable format on load
9. On selection, the input stores readable format

### Output
10. When sending to FileMaker, use `getMySQLFormattedDate()`
11. Null is returned for empty or `--` values

---

## 11) Common Failure Modes and Fixes

### 11.1 Datepicker does not show calendar
Cause:
- Flatpickr JS not loaded
Fix:
- Ensure:
  - `<script src='https://cdn.jsdelivr.net/npm/flatpickr'></script>`

### 11.2 Icon click does nothing
Cause:
- icon ID mismatch
Fix:
- Ensure `iconId` in dateFields matches HTML

### 11.3 Initial date shows as raw MySQL (YYYY-MM-DD)
Cause:
- format conversion not applied
Fix:
- Ensure:
  - `dateInput.value = formatMySQLDateToReadable(initialDate);`

### 11.4 getMySQLFormattedDate returns null unexpectedly
Cause:
- input is not in readable format expected by parseDate
Fix:
- Ensure flatpickr `dateFormat` matches parseDate format:
  - parse uses `'l, j F, Y'`
  - input is stored as `'l, j F, Y'`

### 11.5 Saving breaks because date contains commas and spaces
Cause:
- you passed readable date instead of MySQL date
Fix:
- always pass `getMySQLFormattedDate('fieldId')` result to FileMaker

---

## 12) Clarifications Needed (Ambiguities the AI Agent Cannot Guess)

These must be clarified to make the AI Agent implementation perfect:

1. **Date storage standard in FileMaker**
   - Are all stored dates in MySQL `YYYY-MM-DD` text format?
   - Or are some dates stored as FileMaker date type and later converted?

2. **Locale and display format**
   - Your readable format is: `'l, j F, Y'`
   - Must this always be the same across the whole app?
   - Any pages that must show a different format?

3. **Timezone behavior**
   - Do you treat dates as pure dates (no timezone meaning)?
   - Or do you sometimes use timestamp pickers?

4. **Validation rules**
   - Do you restrict:
     - future dates for DOB
     - from_date <= to_date
     - required fields
   - If yes, where do you enforce it (JS or FileMaker)?

5. **Empty date marker**
   - Your script checks for `'--'`
   - Confirm what the standard empty marker is in your pages.

---

## 13) Implementation Standard (What the AI Agent Must Produce)

When asked to implement a datepicker in CauferoAppStarter, the AI Agent must produce:

1. `<input type='text'>` with stable ID and label
2. calendar icon next to it, wired by ID mapping
3. Flatpickr initialization script:
   - converts initial MySQL date to readable
   - sets flatpickr to readable date format
4. Helper `getMySQLFormattedDate(fieldId)`
5. Usage rule:
   - always convert to MySQL format before passing to FileMaker scripts

---

# CauferoAppStarter Documentation
## Dropdown Fields (Select Inputs) and How They Are Implemented

> Purpose of this doc  
> Train an AI Agent to correctly implement dropdown fields (HTML `<select>`) in CauferoAppStarter WebViewer pages, including:
> 1) Standard dropdowns populated as HTML `<option>` tags from FileMaker  
> 2) Dropdowns populated dynamically in JavaScript from FileMaker-provided JSON objects  
> 3) Dependent dropdowns (Department -> Job Title -> Notch)  
> 4) How selected values are preselected, read, and saved back to FileMaker

This documentation is based on patterns in your sample page:
- Standard dropdowns:
  - `title`
  - `nationality`
  - `hometown_region`
  - `role`
- Dynamic dependent dropdowns:
  - `department`
  - `job_title`
  - `notch`
- Data sources:
  - `Perform Script [ "Get Value List" ; Parameter: "Dropdown|..." ]`
  - `Perform Script [ "Get Value List" ; Parameter: "JS|..." ]`

---

## 1) What a Dropdown Means in CauferoAppStarter

A dropdown is an HTML `<select>` element that:
1. renders on the page
2. may be preselected to the entity's existing value
3. is read in JavaScript on Save
4. is sent back to FileMaker via `FileMaker.PerformScript()`

There are two implementation modes in your framework:

### Mode A: HTML options generated in FileMaker
- FileMaker returns a string of `<option>` tags
- The `<select>` is rendered already filled

### Mode B: Options generated in JavaScript
- FileMaker returns a JavaScript object string (JSON-like)
- JavaScript builds `<option>` tags and appends them to `<select>`
- Used for larger datasets and dependent dropdowns

---

## 2) Mandatory Rules (Non-Negotiable)

### 2.1 Every dropdown MUST have a stable `id`
JavaScript reads `.value` using the ID.

Examples:
- `title`
- `nationality`
- `hometown_region`
- `department`
- `job_title`
- `notch`
- `role`

### 2.2 Every dropdown MUST have a label
Use `<label for='...'>`.

### 2.3 Do not hardcode large option lists in HTML
If the list is long or depends on other selections, use Mode B (JS object approach).

### 2.4 Saving dropdown values is the same as text fields
Read:
~~~js
const nationality = document.getElementById('nationality').value;
~~~
Then include in parameter list.

---

## 3) Dropdown Types in Your Page

### 3.1 Simple dropdowns (FileMaker-generated HTML options)
From your page:
- Titles
- Nationalities
- Regions
- Roles

These are rendered as:
~~~html
<select id='nationality'> {{OPTIONS_HTML}} </select>
~~~

### 3.2 Dynamic dropdowns (JS-generated options)
From your page:
- Department
- Job Title
- Notch

These are rendered as empty selects with a placeholder option:
~~~html
<select id='department'> <option value=''>Select Department</option> </select>
~~~

Then JS fills them.

---

## 4) Mode A: Dropdown Options Generated in FileMaker (HTML Options)

### 4.1 Data source pattern (FileMaker script call)
Your page calls a central script named `Get Value List`:

~~~filemaker
Perform Script [ “Get Value List” ; Parameter: "Dropdown|Titles|" & $Title ]
Set Variable [ $Titles HTML ; Value: Get ( ScriptResult ) ]
~~~

Same pattern:
~~~filemaker
Perform Script [ “Get Value List” ; Parameter: "Dropdown|Nationalities|" & $Nationality ]
Set Variable [ $Nationalities HTML ; Value: Get ( ScriptResult ) ]

Perform Script [ “Get Value List” ; Parameter: "Dropdown|Regions|" & $Hometown Region ]
Set Variable [ $Hometown Regions HTML ; Value: Get ( ScriptResult ) ]

Perform Script [ “Get Value List” ; Parameter: "Dropdown|Roles|" & $Role ID ]
Set Variable [ $Roles Dropdown HTML ; Value: Get ( ScriptResult ) ]
~~~

### 4.2 Contract of `Get Value List` for Dropdown mode
The AI Agent must assume this contract:

Input parameter format:
- `"Dropdown|<ValueListName>|<SelectedValue>"`

Output:
- A string containing one or more `<option>` elements
- The matching selected item is marked as selected

Example output shape:
~~~html
<option value='Mr'>Mr</option>
<option value='Mrs' selected>Mrs</option>
<option value='Dr'>Dr</option>
~~~

If your actual script uses different option value rules (IDs vs labels), clarify it in the Clarifications section.

### 4.3 HTML injection pattern
Inject the returned HTML into the `<select>`:

~~~html
<select id='nationality'> {{NATIONALITIES_HTML}} </select>
~~~

From your page:
~~~html
<select id='nationality'> " & $Nationalities HTML & " </select>
~~~

### 4.4 Example: Titles dropdown in combined inputs
Your page uses:
~~~html
<select class='combined-inputs-dropdown' id='title' aria-label='Title' style='width:50%'>
  {{TITLES_HTML}}
</select>
~~~

---

## 5) Mode B: Dropdown Options Generated in JavaScript (JS Objects)

This mode is used when:
- the dataset is large
- the dataset needs dependent filtering
- you want to avoid shipping huge HTML strings

### 5.1 FileMaker data source pattern
Your page fetches JS objects through `Get Value List` in JS mode:

~~~filemaker
Perform Script [ “Get Value List” ; Parameter: "JS|Departments|" & $Department ]
Set Variable [ $Departments JS ; Value: Get ( ScriptResult ) ]

Perform Script [ “Get Value List” ; Parameter: "JS|Job Titles|" & $Job Title ]
Set Variable [ $Job Titles JS ; Value: Get ( ScriptResult ) ]

Perform Script [ “Get Value List” ; Parameter: "JS|Notches|" & $Notch ID ]
Set Variable [ $Notches JS ; Value: Get ( ScriptResult ) ]
~~~

### 5.2 Contract of `Get Value List` for JS mode
The AI Agent must assume this contract:

Input parameter format:
- `"JS|<DatasetName>|<SelectedDisplayValueOrId>"`

Output:
- A string that can be placed directly into JS object literal context
- It becomes a valid object when wrapped like:
  - `const departments = { <output> };`

Example expected shape:
~~~js
{
  "UUID-1": { "c2": "Dept Name" },
  "UUID-2": { "c2": "Another Dept" }
}
~~~

Your dependent dropdown script uses keys like:
- `details.c2`
- `details.c3`

So the AI Agent must preserve the same field mapping:
- For Departments:
  - `details.c2` = department name
- For Job Titles:
  - `details.c2` = department ID
  - `details.c3` = job title name
- For Notches:
  - `details.c2` = job title ID
  - `details.c3` = notch label

If those c-values differ in your real data, clarify it.

---

## 6) Dependent Dropdowns (Department -> Job Title -> Notch)

### 6.1 HTML skeleton (empty selects with placeholders)
From your Tab 4 HTML:

~~~html
<select id='department'> <option value=''>Select Department</option> </select>
<select id='job_title'> <option value=''>Select Job Title</option> </select>
<select id='notch'> <option value=''>Select Notch</option> </select>
~~~

### 6.2 Preselected values come from FileMaker IDs
Your script sets these variables earlier:
- `$Department ID`
- `$Job Title ID`
- `$Notch ID`

Then injects them into JS:
~~~js
const selectedDepartment = '{{DEPARTMENT_ID}}';
const selectedJobTitle = '{{JOB_TITLE_ID}}';
const selectedNotch = '{{NOTCH_ID}}';
~~~

### 6.3 JavaScript population pattern (from your page)
Your page uses:

~~~js
document.addEventListener('DOMContentLoaded', function() {

  const departments = { {{DEPARTMENTS_JS}} };
  const jobTitles = { {{JOB_TITLES_JS}} };
  const notches = { {{NOTCHES_JS}} };

  const departmentDropdown = document.getElementById('department');
  const jobTitleDropdown = document.getElementById('job_title');
  const notchDropdown = document.getElementById('notch');

  const selectedDepartment = '{{DEPARTMENT_ID}}';
  const selectedJobTitle = '{{JOB_TITLE_ID}}';
  const selectedNotch = '{{NOTCH_ID}}';

  /* Populate departments dropdown */
  for (const [id, details] of Object.entries(departments)) {
    let option = document.createElement('option');
    option.value = id;
    option.textContent = details.c2; /* c2 holds department name */
    if (id === selectedDepartment) option.selected = true;
    departmentDropdown.appendChild(option);
  }

  function populateJobTitles(deptId) {
    jobTitleDropdown.innerHTML = '<option value=\"\">Select Job Title</option>';
    notchDropdown.innerHTML = '<option value=\"\">Select Notch</option>';

    for (const [id, details] of Object.entries(jobTitles)) {
      /* c2 holds department ID, c3 holds job title name */
      if (details.c2 === deptId) {
        let option = document.createElement('option');
        option.value = id;
        option.textContent = details.c3;
        if (id === selectedJobTitle) option.selected = true;
        jobTitleDropdown.appendChild(option);
      }
    }
    if (selectedJobTitle) populateNotches(selectedJobTitle);
  }

  function populateNotches(jobId) {
    notchDropdown.innerHTML = '<option value=\"\">Select Notch</option>';
    for (const [id, details] of Object.entries(notches)) {
      /* c2 holds job title ID, c3 holds notch label */
      if (details.c2 === jobId) {
        let option = document.createElement('option');
        option.value = id;
        option.textContent = details.c3;
        if (id === selectedNotch) option.selected = true;
        notchDropdown.appendChild(option);
      }
    }
  }

  if (selectedDepartment) populateJobTitles(selectedDepartment);

  departmentDropdown.addEventListener('change', function() {
    populateJobTitles(this.value);
  });

  jobTitleDropdown.addEventListener('change', function() {
    populateNotches(this.value);
  });
});
~~~

### 6.4 Core rules for dependent dropdowns
1. Department options are always loaded first
2. When Department changes:
   - Job Titles reset
   - Notches reset
   - Job Titles repopulate for that department
3. When Job Title changes:
   - Notches repopulate for that job title
4. Preselected values must be applied during population

---

## 7) Reading Dropdown Values in Save Function

### 7.1 Standard read
Use `.value`:

~~~js
const title = document.getElementById('title').value;
const nationality = document.getElementById('nationality').value;
const hometown_region = document.getElementById('hometown_region').value;

const department = document.getElementById('department').value;
const job_title = document.getElementById('job_title').value;
const notch = document.getElementById('notch').value;
~~~

### 7.2 Encoding rule when saving
In your page you encode:
- `role`

~~~js
const role = encodeURIComponent(document.getElementById('role').value);
~~~

This can be standard for dropdowns too if:
- option labels contain risky characters
- you are using `|` delimiter
- you want maximum safety

---

## 8) Preselecting Dropdown Values

### 8.1 Mode A preselect
FileMaker does it inside `Get Value List` output.
You pass the selected value in the parameter string, and the script outputs `selected` on the matching `<option>`.

### 8.2 Mode B preselect
JavaScript does it during option creation:

~~~js
if (id === selectedDepartment) option.selected = true;
~~~

That means:
- FileMaker must supply correct selected IDs
- the JS object keys must match those IDs

---

## 9) Where Dropdown Data Comes From

Your page uses two sources:

### 9.1 Value lists turned into HTML options (Mode A)
Examples:
- Titles
- Nationalities
- Regions
- Roles

### 9.2 Data lists turned into JS objects (Mode B)
Examples:
- Departments
- Job Titles
- Notches

---

## 10) Copy-Paste Implementation Templates

### 10.1 Template A: FileMaker-generated HTML options dropdown
HTML:
~~~html
<div class='form-group-2'>
  <label for='FIELD_ID'>LABEL</label>
  <select id='FIELD_ID'> {{OPTIONS_HTML}} </select>
</div>
~~~

FileMaker pattern:
~~~filemaker
Perform Script [ "Get Value List" ; Parameter: "Dropdown|VALUE_LIST_NAME|" & $SelectedValue ]
Set Variable [ $OptionsHTML ; Value: Get ( ScriptResult ) ]
~~~

Inject:
~~~filemaker
Set Variable [ $TabX HTML ;
"
<select id='FIELD_ID'> " & $OptionsHTML & " </select>
"
]
~~~

### 10.2 Template B: JS-generated dropdown from JS object
HTML skeleton:
~~~html
<div class='form-group-2'>
  <label for='FIELD_ID'>LABEL</label>
  <select id='FIELD_ID'><option value=''>Select LABEL</option></select>
</div>
~~~

JS population:
~~~js
const data = { {{DATA_JS}} };
const dropdown = document.getElementById('FIELD_ID');

for (const [id, details] of Object.entries(data)) {
  let option = document.createElement('option');
  option.value = id;
  option.textContent = details.c2;
  dropdown.appendChild(option);
}
~~~

---

## 11) Quality Checklist for the AI Agent

When adding a dropdown field, the AI Agent must verify:

### Structure
1. `<select>` has a stable ID
2. `<label for>` matches the ID
3. Field sits inside standard grid containers (`form-gridX`, `form-group-2`)

### Data source
4. Mode A:
   - `Get Value List` called with `"Dropdown|..."` format
   - HTML options string injected into `<select>`
5. Mode B:
   - `Get Value List` called with `"JS|..."` format
   - output becomes valid JS object inside `{ }`
   - JS populates `<select>` with `document.createElement('option')`

### Preselect
6. Preselected value is applied correctly:
   - Mode A uses `selected` option output
   - Mode B compares IDs and sets `option.selected = true`

### Save
7. Dropdown value is read on Save
8. Value is included in parameter list in correct order
9. Encode if needed to protect delimiter-based parameter parsing

---

## 12) Common Failure Modes and Fixes

### 12.1 Dropdown shows empty even though data exists
Cause:
- You forgot to inject `$OptionsHTML`
Fix:
- Ensure:
  - `select id='x'> " & $OptionsHTML & " </select>`

### 12.2 Wrong item is preselected in Mode A
Cause:
- The selected value passed to `Get Value List` does not match any option value
Fix:
- Ensure the script expects label vs ID and you pass the correct one

### 12.3 Dependent dropdown does not update
Cause:
- missing change event listener
Fix:
- Ensure:
  - `departmentDropdown.addEventListener('change', ...)`

### 12.4 Dependent dropdown filters wrong
Cause:
- incorrect c-mapping assumptions (`c2`, `c3`)
Fix:
- verify:
  - jobTitles entry has department ID in `c2`
  - notch entry has job title ID in `c2`

### 12.5 Save script receives wrong dropdown data
Cause:
- parameter order mismatch
Fix:
- match `parameters = [ ... ].join('|')` order to the FileMaker save script parsing order

---

## 13) Clarifications Needed (Ambiguities the AI Agent Cannot Guess)

These must be clarified to make the AI Agent implementation perfect:

1. **Option values: IDs or labels**
   - For Titles, Nationalities, Regions:
     - does the `<option value>` equal the label text
     - or a UUID behind the scenes
   - For Roles:
     - looks like it may be ID-based (`$Role ID`)
   - The AI Agent needs the rule per value list type.

2. **Exact output format of `Get Value List`**
   - For Dropdown mode:
     - does it return only `<option>` tags
     - or full `<select>` markup sometimes
   - For JS mode:
     - does it return strict JSON
     - or JSON-like object literal without outer braces
   - The AI Agent must generate compatible wrappers.

3. **Selection key in JS mode**
   - Are selected values always IDs (`$Department ID`, `$Job Title ID`, `$Notch ID`)?
   - Or do you sometimes preselect by display name?

4. **Sorting**
   - Should dropdown options be sorted alphabetically
   - or in a custom order defined by the dataset?

5. **Empty selection rules**
   - When value is empty, should the first option always be:
     - `<option value=''>Select X</option>`
   - Any fields that must never be blank?

---

## 14) Implementation Standard (What the AI Agent Must Produce)

When asked to implement a dropdown in CauferoAppStarter, the AI Agent must produce:

1. HTML `<select>` with stable ID and label
2. One of:
   - Mode A: FileMaker-generated `<option>` HTML injected
   - Mode B: JS object injected and JS populates options
3. Preselect behavior:
   - Mode A via `selected` option in HTML output
   - Mode B via ID comparison and `option.selected = true`
4. Save integration:
   - read `.value`
   - include in parameter list
   - encode if needed for delimiter safety

---

# CauferoAppStarter Documentation
## Text Areas (Multi-line Text Fields) and How They Are Implemented

> Purpose of this doc  
> Train an AI Agent to correctly implement **text areas** in CauferoAppStarter WebViewer pages, including:
> 1) HTML `<textarea>` rendering patterns used in pages  
> 2) How values are preloaded from FileMaker variables  
> 3) How user input is collected in JavaScript and sent back to FileMaker  
> 4) Encoding rules for safe parameter passing  
> 5) Line break handling between FileMaker and HTML  
> 6) Common pitfalls and the correct fixes

This documentation is based on patterns in your sample page:
- Text areas used:
  - `residential_address`
  - `postal_address`
- Data pre-processing:
  - `ConvertLineBreakMarkersToReturns(...)`
- Save logic:
  - `encodeURIComponent(document.getElementById('residential_address').value)`
  - `encodeURIComponent(document.getElementById('postal_address').value)`

---

## 1) What a Text Area Means in CauferoAppStarter

A text area is used for multi-line values such as:
- addresses
- descriptions
- notes
- remarks
- long comments

In CauferoAppStarter, a text area is implemented using:
- an HTML `<textarea>` element with a stable `id`
- FileMaker variable injection to preload existing value
- JavaScript collection on Save
- URL encoding to protect delimiters and line breaks

---

## 2) Mandatory Rules (Non-Negotiable)

### 2.1 Always use a stable `id`
JavaScript reads the value by ID.

Example IDs:
- `residential_address`
- `postal_address`

### 2.2 Always use a label
The label must match `for='ID'`.

### 2.3 Never use `value='...'` attribute on `<textarea>`
Textareas do not use `value` attribute.
Their value is the inner text content:

Correct:
~~~html
<textarea id='residential_address'>Existing text</textarea>
~~~

Wrong:
~~~html
<textarea id='residential_address' value='Existing text'></textarea>
~~~

### 2.4 Always encode textarea values on Save
Because:
- you join parameters using `|`
- textarea content can contain line breaks, pipes, and special characters

So do:
~~~js
encodeURIComponent(document.getElementById('residential_address').value)
~~~

---

## 3) Data Flow Overview (End-to-End)

### Step A: Get stored text from database
You query and assign values to variables like:
- `$Residential Address`
- `$Postal Address`

### Step B: Normalize line breaks for HTML insertion
Your page does:
- `ConvertLineBreakMarkersToReturns(...)`

### Step C: Inject into `<textarea>` inner text
Your page injects:
~~~html
<textarea id='residential_address'>{{RES_ADDRESS}}</textarea>
~~~

### Step D: Read on Save and encode
Your save script does:
- `encodeURIComponent(textarea.value)`

### Step E: Pass to FileMaker via parameters
Parameters are joined with `|`
FileMaker receives and decodes if needed.

---

## 4) Preloading Text Area Values (FileMaker Side)

### 4.1 Canonical pattern (from your page)
You set values like:

~~~filemaker
Set Variable [ $Residential Address ;
  Value: ConvertLineBreakMarkersToReturns ( GetValue ( $Link Record As List ; 14 ) )
]

Set Variable [ $Postal Address ;
  Value: ConvertLineBreakMarkersToReturns ( GetValue ( $Link Record As List ; 16 ) )
]
~~~

### 4.2 What `ConvertLineBreakMarkersToReturns` is doing (assumed)
The AI Agent must assume your stored text may contain markers such as:
- `¶`
- `\n`
- other placeholders

The function converts them into actual returns suitable for HTML insertion.

If your storage uses a different marker system, the AI Agent needs it clarified.

---

## 5) Rendering Text Areas (HTML)

### 5.1 Canonical HTML pattern (from your page)
Your Tab 2 uses:

~~~html
<div class='form-group-2'>
  <label for='residential_address'>Residential Address</label>
  <textarea id='residential_address' placeholder='Type the residential address here...'>{{RES_ADDRESS}}</textarea>
</div>

<div class='form-group-2'>
  <label for='postal_address'>Postal Address</label>
  <textarea id='postal_address' placeholder='Type the postal address here...'>{{POSTAL_ADDRESS}}</textarea>
</div>
~~~

### 5.2 Layout pattern
Text areas typically sit in a 2-column grid container:
- `form-grid2`
- each text area uses `form-group-2`

Example container:
~~~html
<div class='form-grid2'>
  ...two textarea blocks...
</div>
~~~

### 5.3 Placeholder guidelines
Placeholders must be:
- short
- instructive
- consistent with your page tone

Example:
- `placeholder='Type the residential address here...'`

---

## 6) Collecting Text Area Values on Save (JavaScript)

### 6.1 Canonical pattern (from your page)
Your save script uses:

~~~js
const residential_address = encodeURIComponent(document.getElementById('residential_address').value);
const postal_address = encodeURIComponent(document.getElementById('postal_address').value);
~~~

### 6.2 Why encodeURIComponent is required
Your parameter system uses:
~~~js
const parameters = [ ... ].join('|');
~~~

If textarea content contains:
- pipes `|`
- returns
- commas
- quotes
- Unicode characters

Then encoding is mandatory to avoid breaking parsing in FileMaker.

### 6.3 Standard saving rule
Always encode textareas.
Even if you think "it is just an address".
Addresses can contain:
- line breaks
- commas
- special characters

---

## 7) Parameter Passing Rules (Delimiter Safety)

### 7.1 Standard join method
Your system uses:
~~~js
const parameters = [ field1, field2, field3, ... ].join('|');
FileMaker.PerformScript('Save Staff', parameters);
~~~

### 7.2 Implication for text areas
FileMaker receives encoded values.
So the FileMaker Save script must decode them (if your standard requires it).

Two possibilities exist:

#### Option A: Decode in FileMaker Save Script
- Use `GetAsText` and decode percent encoding
- Example utility might exist in your system (unknown name)

#### Option B: Store encoded value as-is
Not recommended for text fields because it stores `%0A` sequences and reduces readability.

The AI Agent needs your canonical approach clarified.

---

## 8) Recommended Standard: Text Areas Must Be Decoded Before Saving

This is the clean standard the AI Agent should follow unless you say otherwise:

1. JS encodes textarea:
   - `encodeURIComponent(...)`
2. FileMaker Save script decodes to real text
3. Save decoded text into the field

If you already have a standard function like:
- `URLDecode`
- `PercentDecode`
- `DecodeURIComponent`
the AI Agent must be told the exact function name.

---

## 9) Copy-Paste Implementation Templates

### 9.1 Template: Text area in a form group
~~~html
<div class='form-group-2'>
  <label for='FIELD_ID'>LABEL</label>
  <textarea id='FIELD_ID' placeholder='PLACEHOLDER'>{{PRELOADED_VALUE}}</textarea>
</div>
~~~

### 9.2 Template: JS read and encode
~~~js
const FIELD_ID_value = encodeURIComponent(document.getElementById('FIELD_ID').value);
~~~

### 9.3 Template: Save parameter inclusion
~~~js
const parameters = [ ..., FIELD_ID_value, ... ].join('|');
FileMaker.PerformScript('SAVE_SCRIPT_NAME', parameters);
~~~

---

## 10) Quality Checklist for the AI Agent

When implementing text areas, the AI Agent must verify:

### HTML correctness
1. `<textarea>` has an ID
2. `<label for>` matches ID
3. Preloaded value is inside textarea inner text, not an attribute
4. Placeholder exists (optional but recommended)

### Data correctness
5. FileMaker variable is normalized for line breaks before injection
6. The injected value does not break HTML structure

### Save correctness
7. Textarea value is read using `.value`
8. Value is always passed through `encodeURIComponent`
9. Encoded value is included in the correct parameter order

### Integration correctness
10. FileMaker Save script correctly decodes the encoded text area values before storing (if that is your standard)

---

## 11) Common Failure Modes and Fixes

### 11.1 Text area appears empty even though record has text
Cause:
- The value was injected into `value=''` attribute instead of inner text
Fix:
- Use:
  - `<textarea>VALUE</textarea>`

### 11.2 Text area breaks the page HTML
Cause:
- Injected text contains raw HTML-like characters (`<`, `>`, `&`)
Fix:
- Escape the text before injection, or ensure your page builder does escaping.
This is not shown in your sample, so the AI Agent needs your standard:
- Do you HTML-escape text values before injecting?

### 11.3 Save script receives partial address only
Cause:
- address contains `|`
Fix:
- Always `encodeURIComponent` before joining parameters

### 11.4 Saved address contains `%0A` and `%2C`
Cause:
- You encoded but never decoded in FileMaker
Fix:
- Decode in FileMaker before saving into address field

### 11.5 Line breaks vanish after save
Cause:
- FileMaker decoding or normalization removed returns
Fix:
- Ensure decode restores actual returns
- Ensure you store in a text field that supports returns
- Ensure you do not strip returns during parsing

---

## 12) Clarifications Needed (Ambiguities the AI Agent Cannot Guess)

These must be clarified to make the AI Agent implementation perfect:

1. **Decoding standard in FileMaker**
   - After JS encodes textarea values, what exact method do you use in FileMaker to decode them?
   - Provide the function name or script step pattern.

2. **HTML escaping standard**
   - Do you escape user text before injecting into HTML?
   - If yes, what is the standard function used in your system?
   - Without escaping, `<` and `&` can break HTML.

3. **Line break storage**
   - When you store addresses in FileMaker, do you store:
     - real returns
     - or markers that later need conversion
   - Confirm what `ConvertLineBreakMarkersToReturns` expects.

4. **Maximum length and validation**
   - Do you limit textarea length for certain fields?
   - Do you sanitize content (remove scripts, etc.)?

5. **Multi-language and unicode**
   - Should the system preserve all unicode characters exactly?

---

## 13) Implementation Standard (What the AI Agent Must Produce)

When asked to implement a textarea field in CauferoAppStarter, the AI Agent must produce:

1. A `<textarea>` with:
   - stable ID
   - matching label
   - placeholder (recommended)
   - preloaded inner text value from FileMaker

2. FileMaker preprocessing:
   - normalize line breaks before injection (example: `ConvertLineBreakMarkersToReturns(...)`)

3. Save integration:
   - read `.value`
   - apply `encodeURIComponent`
   - include in parameter list in correct order
   - FileMaker side decodes before saving (if that is your standard)

---

# CauferoAppStarter Documentation
## Text Editors (Rich Text Fields) in WebViewer Pages

> Scope: This document is only for text fields implemented as a rich text editor inside WebViewer pages (HTML + CSS + JavaScript).  
> Non-scope: number fields, dropdowns, date pickers, and other inputs (covered in separate docs).

---

## 1. Purpose

A “Text Editor” in CauferoAppStarter is a rich-text input that lets the user type and format content (bold, italics, lists, links, alignment, highlight, etc.) inside a WebViewer page, then save the formatted HTML back into FileMaker.

This editor is used when:
- You want more than plain text (formatting, lists, links).
- You want to store the content as HTML in a FileMaker text field.

---

## 2. High-Level Architecture

### 2.1 Data Flow (Round Trip)

1. **FileMaker loads record**
   - FileMaker reads the stored value for a rich text field (example: `Businesses::Notes`).
   - The stored value is treated as **HTML**.

2. **FileMaker generates HTML page**
   - Page HTML includes the editor markup:
     - Toolbar buttons
     - A `<div id="editor" contenteditable="true">...</div>`
   - The editor’s initial value is inserted directly inside the editor container:
     - `"<div id=\"editor\" ...>" & $Notes & "</div>"`

3. **User edits and formats in the WebViewer**
   - Buttons trigger formatting commands using `document.execCommand(...)`.

4. **User clicks Save**
   - JavaScript collects editor HTML via:
     - `document.getElementById('editor').innerHTML`
   - JavaScript encodes the content and sends it to FileMaker using `FileMaker.PerformScript(...)`.

5. **FileMaker Save script receives the HTML**
   - FileMaker decodes and normalizes the value.
   - FileMaker stores the HTML into the correct field.

---

## 3. Implementation Components

A Text Editor implementation has 4 parts:

1. **CSS styles** (toolbar and editor area)
2. **HTML markup** (toolbar + contenteditable editor)
3. **JavaScript logic** (formatting commands, selection restore, link handling, toolbar state)
4. **Save pipeline** (JS encode + FileMaker decode + store)

The sample code shows all four.

---

## 4. Editor UI and Features

### 4.1 Supported Formatting Controls (Sample)

Toolbar buttons and inputs included in the sample:

- Bold
- Italic
- Underline
- Strikethrough
- Align left / center / right
- Ordered list / unordered list
- Indent / outdent (also via Tab handling)
- Insert link (prompts for URL)
- Text color
- Highlight color
- Insert code tag (wrap prompt result in `<code>...</code>`)
- Insert blockquote

These map to `document.execCommand` commands like:
- `bold`, `italic`, `underline`, `strikethrough`
- `justifyLeft`, `justifyCenter`, `justifyRight`
- `insertOrderedList`, `insertUnorderedList`
- `indent`, `outdent`
- `createLink`
- `foreColor`, `hiliteColor`
- `insertHTML`, `formatBlock`

---

## 5. FileMaker-Side Preparation

### 5.1 Read the Field Value

Your FileMaker script typically loads the field using SQL or normal fields, then assigns it to a variable like `$Notes`.

In the sample, `$Notes` is read from SQL:
- `"Notes"` is selected as column 29 in the first SQL.
- `$Notes` is later prepared for safe insertion into the HTML page.

### 5.2 Decode Placeholder Tokens (If Used)

The sample shows a decode step that replaces placeholder tokens with real characters:

- `lessthan` → `<`
- `greaterthan` → `>`
- `forwardslash` → `/`
- `backslash` → `\\`
- `hashhhh` → `#`
- `singlequote` → `'`
- `doublequotes` → `"`
- `sometab` → (tab)
- `equalto` → `=`
- `doublecolon` → `::`
- `semicolon` → `;`
- `ampersand` → `&`
- `ppippe` → `|`

This implies the stored HTML may have been saved with these replacements earlier.

**Rule for the AI Agent:**  
If the system uses placeholder tokens for HTML-safe storage, always apply the same decode mapping before inserting HTML into the editor.

### 5.3 Store a Plain-Text Version (Optional)

The sample also computes:
- `$Notes No Tags = HTMLToTextStripTags ( $Notes )`

This is useful for:
- Prompt building
- Search indexing
- Summaries

It is separate from the editor’s stored HTML.

---

## 6. CSS Requirements

### 6.1 Minimum Required Styles

The sample styles define:
- `.texteditor` wrapper
- `.toolbar` layout and button styling
- `#editor` sizing, padding, typography
- `.color_icon` for color pickers
- Explicit list styling in `#editor ol`, `#editor ul`, `#editor li`
- Link behavior styling in `#editor a`

**Key requirement:**  
Make sure lists show bullets and numbering inside the editor. The sample enforces this explicitly.

### 6.2 Example CSS (Template)

~~~css
.texteditor {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  background: #fff;
}

.toolbar {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  align-items: center;
  background: #f4f4f4;
  border-bottom: 1px solid #ddd;
  padding: 8px;
  gap: 8px;
}

.toolbar button,
.toolbar select,
.toolbar input {
  margin: 0;
  padding: 6px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: #fff;
  color: #333;
  font-size: 14px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.toolbar button:hover,
.toolbar select:hover,
.toolbar input:hover {
  background: #e0e0e0;
}

.toolbar button.active {
  background: #d1e7fd;
  color: #0056b3;
  border-color: #0056b3;
}

#editor {
  height: 65vh;
  margin: 0;
  padding: 12px;
  width: 100%;
  background: #fff;
  font-size: 16px;
  line-height: 1.6;
  color: #333;
  resize: none;
  overflow: auto;
}

.color_icon[type="color"] {
  padding: 0;
  border: none;
  height: 27px;
  width: 30px;
  cursor: pointer;
  border-radius: 4px;
  background: transparent;
}

.color_icon[type="color"]::-webkit-color-swatch {
  border-radius: 4px;
}

#editor a {
  pointer-events: auto;
  color: #007bff;
  text-decoration: underline;
}

#editor ol {
  list-style-type: decimal;
  padding-left: 20px;
}

#editor ul {
  list-style-type: disc;
  padding-left: 20px;
}

#editor li {
  margin-bottom: 5px;
}
~~~

---

## 7. HTML Markup Requirements

### 7.1 Editor Markup Structure

The editor is a wrapper that contains:
- Toolbar (`.toolbar`)
- Editable content area (`#editor` with `contenteditable="true"`)

The sample uses Font Awesome icons inside toolbar buttons:
- `<i class="fas fa-bold"></i>` etc.

**Dependency requirement:**  
Font Awesome must be included in the page header or globally in the WebViewer HTML. If it is missing, buttons still work but icons may not render.

### 7.2 Example HTML (Template)

Important: Insert FileMaker value inside the editor container as raw HTML.

~~~html
<div class="texteditor">

  <div class="toolbar">
    <button id="boldBtn" onclick="toggleFormat('bold')"><i class="fas fa-bold"></i></button>
    <button id="italicBtn" onclick="toggleFormat('italic')"><i class="fas fa-italic"></i></button>
    <button id="underlineBtn" onclick="toggleFormat('underline')"><i class="fas fa-underline"></i></button>
    <button id="strikeBtn" onclick="toggleFormat('strikethrough')"><i class="fas fa-strikethrough"></i></button>

    <button id="justifyLeftBtn" onclick="toggleFormat('justifyLeft')"><i class="fas fa-align-left"></i></button>
    <button id="justifyCenterBtn" onclick="toggleFormat('justifyCenter')"><i class="fas fa-align-center"></i></button>
    <button id="justifyRightBtn" onclick="toggleFormat('justifyRight')"><i class="fas fa-align-right"></i></button>

    <button id="orderedListBtn" onclick="toggleFormat('insertOrderedList')"><i class="fas fa-list-ol"></i></button>
    <button id="unorderedListBtn" onclick="toggleFormat('insertUnorderedList')"><i class="fas fa-list-ul"></i></button>

    <button id="indentBtn" onclick="toggleFormat('indent')"><i class="fas fa-indent"></i></button>
    <button id="outdentBtn" onclick="toggleFormat('outdent')"><i class="fas fa-outdent"></i></button>

    <button id="linkBtn" onclick="addLink()"><i class="fas fa-link"></i></button>

    <input type="color" class="color_icon" id="textColor"
      onchange="formatText('foreColor', this.value)" title="Text Color">

    <input type="color" class="color_icon" id="highlightColor"
      onchange="formatText('hiliteColor', this.value)" title="Highlight Color">

    <button id="codeBtn"
      onclick="formatText('insertHTML', '<code>'+prompt('Enter code:')+'</code>')">
      <i class="fas fa-code"></i>
    </button>

    <button id="quoteBtn" onclick="formatText('formatBlock', '<blockquote>')">
      <i class="fas fa-quote-right"></i>
    </button>
  </div>

  <div id="editor"
       contenteditable="true"
       onkeydown="handleListTab(event)"
       onmouseup="updateToolbar()">
    <!-- FileMaker inserts existing HTML here -->
  </div>

</div>
~~~

### 7.3 FileMaker Concatenation Pattern

The sample builds `$TextEditor HTML` like this:

- HTML string in a FileMaker calculation
- Insert `$Notes` into the editor div:
  - `...>" & $Notes & "</div>`

**Rule for the AI Agent:**  
Treat the stored editor field as HTML. Insert it raw inside the contenteditable container.

---

## 8. JavaScript Requirements

### 8.1 Why Selection Restore Exists

In contenteditable editors, clicking a toolbar button steals focus from the editor and the selection can collapse.

The sample fixes this using:
- Save selection on mouseup
- Restore selection before executing commands
- Refocus the editor after formatting

### 8.2 Core Functions (Sample Behavior)

1. `formatText(command, value)`
   - Restores selection
   - Runs `document.execCommand(command, false, value)`
   - Focuses editor

2. `toggleFormat(command)`
   - Restores selection
   - Runs `execCommand`
   - Updates toolbar active states
   - Focuses editor

3. `updateToolbar()`
   - Uses `document.queryCommandState(cmd)`
   - Toggles `.active` class on toolbar buttons

4. `addLink()`
   - Prompts for URL
   - Forces `https://` if missing
   - Creates link via `createLink`
   - Sets `target="_blank"` and `rel="noopener noreferrer"`
   - Also adds click handler so links open in new window

5. `handleListTab(event)`
   - Tab: indent
   - Shift + Tab: outdent
   - Backspace at start of list item: outdent

### 8.3 Example JS (Template)

Use block comments in code examples.

~~~javascript
let lastSelection;

document.getElementById('editor').addEventListener('mouseup', () => {
  lastSelection = saveSelection();
});

function formatText(command, value = null) {
  restoreSelection();
  document.execCommand(command, false, value);
  document.getElementById('editor').focus();
}

function toggleFormat(command) {
  restoreSelection();
  document.execCommand(command, false, null);
  updateToolbar();
  document.getElementById('editor').focus();
}

function updateToolbar() {
  const commands = [
    { cmd: 'bold', buttonId: 'boldBtn' },
    { cmd: 'italic', buttonId: 'italicBtn' },
    { cmd: 'underline', buttonId: 'underlineBtn' },
    { cmd: 'strikethrough', buttonId: 'strikeBtn' },
    { cmd: 'justifyLeft', buttonId: 'justifyLeftBtn' },
    { cmd: 'justifyCenter', buttonId: 'justifyCenterBtn' },
    { cmd: 'justifyRight', buttonId: 'justifyRightBtn' },
    { cmd: 'insertOrderedList', buttonId: 'orderedListBtn' },
    { cmd: 'insertUnorderedList', buttonId: 'unorderedListBtn' },
    { cmd: 'indent', buttonId: 'indentBtn' },
    { cmd: 'outdent', buttonId: 'outdentBtn' }
  ];

  commands.forEach(({ cmd, buttonId }) => {
    const isActive = document.queryCommandState(cmd);
    document.getElementById(buttonId).classList.toggle('active', isActive);
  });
}

function addLink() {
  let url = prompt('Enter the URL:');
  if (!url) return;

  if (!url.startsWith('http://') && !url.startsWith('https://')) {
    url = 'https://' + url;
  }

  formatText('createLink', url);

  const links = document.getElementById('editor').getElementsByTagName('a');
  for (let link of links) {
    link.setAttribute('target', '_blank');
    link.setAttribute('rel', 'noopener noreferrer');
  }
}

function handleListTab(event) {
  if (event.key === 'Tab') {
    event.preventDefault();
    if (event.shiftKey) {
      /* Outdent if Shift + Tab */
      document.execCommand('outdent');
    } else {
      /* Indent if Tab */
      document.execCommand('indent');
    }
  }

  if (event.key === 'Backspace') {
    const selection = window.getSelection();
    if (selection.rangeCount > 0) {
      const range = selection.getRangeAt(0);
      const parent = range.startContainer.parentNode;
      if (parent.tagName === 'LI' && range.startOffset === 0) {
        /* Outdent if backspace is pressed at start of list item */
        event.preventDefault();
        document.execCommand('outdent');
      }
    }
  }
}

function saveSelection() {
  const selection = window.getSelection();
  if (selection.rangeCount > 0) return selection.getRangeAt(0);
  return null;
}

function restoreSelection() {
  const selection = window.getSelection();
  if (lastSelection) {
    selection.removeAllRanges();
    selection.addRange(lastSelection);
  }
}

document.getElementById('editor').addEventListener('click', function (event) {
  const target = event.target;
  if (target.tagName === 'A') {
    event.preventDefault();
    window.open(target.href, '_blank', 'noopener,noreferrer');
  }
});
~~~

---

## 9. Saving to FileMaker (Critical)

### 9.1 Extract Editor HTML

The sample does:

- `let editor_ = document.getElementById('editor').innerHTML;`

This preserves formatting and tags.

### 9.2 Prevent Delimiter Collisions

The sample replaces pipes before encoding:

- `editor_ = editor_.replace(/\|/g, 'ppippe');`

This implies the system’s parameter serialization uses `|` somewhere (common in your SQL parsing and likely in your script parameter encoding).

**Rule for the AI Agent:**  
Whatever delimiter is used to join the parameters, replace it inside editor content before sending to FileMaker.

### 9.3 Encode for Script Parameter Transport

The sample then does:

- `const editor = encodeURIComponent(editor_);`

This makes the payload safe for transport inside a single script parameter string.

### 9.4 Add to Parameters List

The editor value is appended to the parameters array.

Later (not shown due to truncation), your code likely:
- Joins parameters using a delimiter (often `|` or `¶`)
- Calls `FileMaker.PerformScript('Save Business', joinedParams)`

**Rule for the AI Agent:**  
The editor HTML must be included as a separate parameter, encoded.

### 9.5 FileMaker Save Script Responsibilities

In the target FileMaker script (example: `Save Business`):

1. Read the editor parameter.
2. URL-decode it.
3. Reverse delimiter replacements:
   - `ppippe` → `|`
4. (If your system uses placeholder tokens) re-encode unsafe characters consistently for storage.

---

## 10. Implementation Checklist (AI Agent)

When the AI Agent implements a text editor in a new page:

### 10.1 Variables (FileMaker script)

- Ensure a variable exists for the field value (example: `$Notes`).
- If placeholder tokens are used in storage, decode them before HTML insertion.
- Build `$TextEditor HTML`:
  - Contains toolbar
  - Contains `#editor`
  - Inserts the existing value inside `#editor`

### 10.2 CSS

- Add editor CSS into `$styles`.
- Ensure lists render properly.
- Ensure the editor height fits your page and tab context.

### 10.3 JavaScript

- Inject the editor JS logic into the page scripts bundle.
- Ensure functions are globally accessible (toolbar uses `onclick="..."`).
- Ensure selection restore is enabled.

### 10.4 Save Pipeline

- On Save:
  - `innerHTML` captured
  - delimiter-safe replacements
  - `encodeURIComponent`
  - included in parameters
  - sent to FileMaker

### 10.5 FileMaker Save Script

- Decode and store editor HTML into the correct field.
- Keep the placeholder-token strategy consistent with the rest of the system.

---

## 11. Patterns and Naming Conventions

### 11.1 IDs

The sample uses fixed IDs:
- `editor`
- `boldBtn`, `italicBtn`, etc.

**Rule for the AI Agent:**  
If a page has only one editor, fixed IDs are acceptable.  
If a page will have multiple editors, IDs must be unique and the JS must be refactored to target the correct editor instance.

### 11.2 Reusable Component Strategy (Recommended)

For future scalability, implement the editor as a reusable component:

- Unique editor container id: `editor_notes`, `editor_description`, etc.
- Pass the editor id into all formatting functions:
  - `toggleFormat(editorId, command)`
  - `formatText(editorId, command, value)`
- Store last selection per editor id.

This is optional for the current sample, required if you add multiple editors to one page.

---

## 12. WebViewer Compatibility Notes

- `document.execCommand` is deprecated in modern browsers, yet it still works widely inside embedded webviews.
- FileMaker WebViewer (WKWebView on macOS/iOS) generally supports this pattern well for internal tools.

**Rule for the AI Agent:**  
Keep this implementation for consistency across CauferoAppStarter until a planned migration to a modern editor engine (like ProseMirror, TipTap, Quill) is approved.

---

## 13. Security and Content Hygiene

Rich text editors can accept pasted HTML.

Minimum hygiene steps:
- In FileMaker save scripts, reject `<script>` tags or strip them.
- If users paste content from external sites, consider sanitizing HTML to allowed tags:
  - `b`, `strong`, `i`, `em`, `u`, `s`, `ol`, `ul`, `li`, `a`, `blockquote`, `code`, `p`, `br`, `div`, `span`

**Rule for the AI Agent:**  
If the page is internal-only and used by trusted staff, keep hygiene minimal. If the page is public-facing or client-facing, sanitize strictly.

---

## 14. Known Ambiguities and Clarifications Needed

The AI Agent can implement the editor as shown, yet these items should be clarified to make the round-trip fully deterministic:

1. **What exactly is `JSNormalizeTextChain`**
   - The save script uses `...value" & JSNormalizeTextChain & ";`
   - This chain likely replaces characters that break delimiters or script parameters.
   - Clarification needed: What replacements does it perform, and does it already handle:
     - quotes
     - ampersands
     - less-than / greater-than
     - pipes
     - line breaks

2. **What delimiter is used to serialize the parameters**
   - The editor replaces `|` with `ppippe`, which suggests `|` is used as a delimiter somewhere.
   - Clarification needed: When sending to FileMaker, do you join parameters with `|`, with `¶`, or with something else?

3. **What is the canonical storage format for rich-text fields**
   - The load pipeline decodes tokens (`lessthan`, `doublequotes`, etc.), which suggests the stored value is tokenized HTML.
   - The save pipeline shown only replaces pipes and then URL-encodes.
   - Clarification needed: On save, do you re-tokenize the HTML into the same placeholders before storing? If yes, where does it happen (JavaScript or FileMaker script)?

4. **Which FileMaker field holds the HTML**
   - The sample reads `"Notes"` into `$Notes`.
   - Clarification needed: Is `"Notes"` always the HTML field, or do some tables store rich text in a different field naming convention (example: `Notes HTML`, `Notes_RichText`)?

5. **Do you want to preserve or strip certain tags**
   - Example: the code button inserts `<code>...</code>`.
   - Clarification needed: Are there tags you want to disallow in saved HTML (example: `<img>`, `<iframe>`)?

Until these are clarified, the AI Agent should:
- Follow the sample pipeline exactly.
- Keep delimiter replacement for `|`
- Keep encodeURIComponent
- Expect FileMaker scripts to finalize the storage normalization.

---

## 15. Minimum “Done” Definition for an AI-Generated Editor

The text editor feature is considered complete when:

- Existing saved HTML loads correctly inside the editor.
- Toolbar actions apply formatting correctly.
- Links open in a new tab/window inside the WebViewer context.
- Ordered and unordered lists display correctly.
- Save captures HTML and persists it in FileMaker.
- Reload shows the same formatting.

---

# CauferoAppStarter Documentation
## File Upload Fields and Image Display Sections (How They Work and How to Implement Them)

> Purpose of this doc  
> Train an AI Agent to correctly implement:
> 1) A **file upload field** (the UI element that looks like a file picker)  
> 2) An **image display section** (thumbnail or profile image shown on the page)  
> 3) A **modal image viewer** (click image to enlarge)  
> 4) The **FileMaker bridging logic** where the browser UI triggers FileMaker scripts to do the real file selection and saving

This documentation is based on the patterns in your sample page:
- File upload field: `<input type='file' id='materialPhoto' />`
- Image display section: `<img src='data:image/png;base64,...' class='summary-image' />`
- File dialog interception: JS prevents native browser picker and calls FileMaker script
- Modal: `openImageModal()`, `closeImageModal()`

---

## 1) Key Concept: WebViewer Cannot Reliably Own the File System

In CauferoAppStarter, the WebViewer UI does not own the file system workflow end-to-end.

Instead:
- The WebViewer provides the UI surface (file input control for familiarity)
- JavaScript intercepts the interaction
- FileMaker does the real work:
  - select file
  - read file container
  - store base64
  - update record
  - re-render the page

This design is intentional and consistent with FileMaker WebViewer realities.

---

## 2) Components of the System

### 2.1 File Upload UI Component
A file upload field is represented in HTML as:

~~~html
<input type='file' id='materialPhoto' />
~~~

But in this framework:
- the click is intercepted
- FileMaker script opens the file picker and handles import

### 2.2 Image Display Component
The image is displayed as a normal `<img>` tag whose `src` is a base64 data URI:

~~~html
<img src='data:image/png;base64,XXXXX' class='summary-image' />
~~~

### 2.3 Click-to-View Modal
An overlay modal is shown when the user clicks the thumbnail.

### 2.4 FileMaker Script Bridge
JavaScript triggers FileMaker scripts using:

~~~js
FileMaker.PerformScript('Insert Staff Photo');
~~~

---

## 3) Image Data Pipeline (How the Image Gets Onto the Page)

This is the canonical pipeline used in your page:

### Step A: Fetch stored image base64 from the database
Your page uses SQL:

~~~filemaker
Set Variable [ $Image ;
Let (
  [
    sql = "Select \"Base64 Image\"
From Images
Where \"Reference\" = ?" ;
    fieldseparator = "xxxxx" ;
    rowseparator = "¶" ;
    id = $$Staff ID ;
    result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; id )
  ] ;
  result
)
]
~~~

### Step B: Use a placeholder when no image exists
Your page sets a default base64 placeholder:

~~~filemaker
Set Variable [ $Image Placeholder ; Value: "iVBORw0K...RK5CYII=" ]
~~~

### Step C: Construct an HTML-safe image source string
Your page builds:

~~~filemaker
Set Variable [ $Image Source HTML ;
  Value: "data:image/png;base64," & If ( IsEmpty ( $Image ) ; $Image Placeholder ; $Image )
]
~~~

### Step D: Inject it into HTML `<img src='...'>`
Example from your summary section:

~~~html
<img src='{{IMAGE_SOURCE_HTML}}' alt='Staff Photo' class='summary-image' />
~~~

---

## 4) Implementing the Image Display Section (Summary Card Pattern)

### 4.1 Canonical HTML Pattern
This is the exact structure pattern your page uses:

~~~html
<div class='image-container' onclick='openImageModal()'>
  <img src='{{IMAGE_SOURCE_HTML}}' alt='Staff Photo' class='summary-image' />
  <i class='fas fa-search-plus summary-image-icon'></i>
</div>
~~~

### 4.2 Behaviors this provides
- Shows a thumbnail in a fixed summary section
- Displays a magnifier icon overlay
- Clicking the container triggers `openImageModal()`

### 4.3 Mandatory class names
The AI Agent must preserve these class names because the CSS and JS assume them:
- `image-container`
- `summary-image`
- `summary-image-icon`

If you rename classes, update the CSS and JS selectors accordingly.

---

## 5) Implementing the Modal Viewer

### 5.1 Modal HTML Pattern
Your page uses:

~~~html
<div class='image-modal' id='imageModal'>
  <span class='image-modal-close-btn' onclick='closeImageModal()'>&times;</span>
  <img id='modalImage' src='' alt='Enlarged Image'>
</div>
~~~

### 5.2 Modal JavaScript Pattern
Your page uses:

~~~js
function openImageModal() {
  const modal = document.getElementById('imageModal');
  const modalImage = document.getElementById('modalImage');
  const thumbnail = document.querySelector('.summary-image');

  modalImage.src = thumbnail.src;
  modal.style.display = 'flex';
}

function closeImageModal() {
  const modal = document.getElementById('imageModal');
  modal.style.display = 'none';
}

window.onclick = function (event) {
  const modal = document.getElementById('imageModal');
  if (event.target === modal) {
    modal.style.display = 'none';
  }
}
~~~

### 5.3 Critical rules
1. The thumbnail must be selectable:
   - `.summary-image` must exist
2. Modal must have:
   - container id `imageModal`
   - image id `modalImage`
3. The modal display style must match CSS assumptions:
   - typically `display: none` by default
   - show via `flex`

---

## 6) Implementing the File Upload Field

### 6.1 Canonical HTML Pattern
Your page places the upload field in the form:

~~~html
<div class='form-group form-group-2'>
  <label for='materialPhoto'>Photo</label>
  <input type='file' id='materialPhoto' />
</div>
~~~

### 6.2 Why it is not used as a normal file input
In FileMaker WebViewer contexts, letting the browser open a native file dialog can be unreliable or inconsistent depending on:
- platform
- WebViewer implementation
- security and sandbox restrictions

So your solution:
- intercepts click
- uses FileMaker script to do the file selection and import

---

## 7) Intercepting the File Input and Calling FileMaker

### 7.1 Canonical JavaScript Intercept Pattern
Your page does:

~~~js
/* Prevent the file dialog from opening by intercepting the 'click' event */
document.getElementById('materialPhoto').addEventListener('click', function(event) {
  /* Prevent the default click behavior (opening the file dialog) */
  event.preventDefault(); 

  /* Trigger FileMaker to handle the file selection */
  FileMaker.PerformScript('Insert Staff Photo');
});

/* Add an event listener to the file input for the 'change' event */
document.getElementById('materialPhoto').addEventListener('change', function(event) {
  /* Prevent default file handling behavior */
  event.preventDefault(); 

  /* Optional debug */
  alert('Change event triggered, but file dialog didn’t open.');
});
~~~

### 7.2 Core rule
In this framework, the file input is a trigger widget, not a real uploader.

Meaning:
- It exists to look like upload UI
- It exists to give user an obvious entry point
- The real upload occurs in FileMaker

### 7.3 Script variable mapping rule
Your page uses:

~~~filemaker
Set Variable [ $Insert Image Script ; Value: "Insert Staff Photo" ]
~~~

Then injects that name into the JS:

~~~js
FileMaker.PerformScript('" & $Insert Image Script & "');
~~~

The AI Agent must always follow this approach:
- store the FM script name in a FileMaker variable
- inject into JS
- never hardcode script names deep inside JS unless your standard says otherwise

---

## 8) FileMaker Side: What the "Insert Photo" Script Must Do

This doc describes the contract, not the exact internal script steps (your implementation may differ).

### 8.1 Script contract (must be true)
When JavaScript calls the FM script:
- FileMaker must prompt user to select an image
- FileMaker must associate the image with the current entity (example: staff record)
- FileMaker must store the image in a predictable way
- FileMaker must refresh the WebViewer page

### 8.2 Data storage pattern implied by your page
Your page fetches from a table called `Images`:

- It selects `"Base64 Image"`
- It filters by `"Reference" = $$Staff ID`

So the implied contract is:
- The Insert Photo script stores:
  - `Images::Reference = $$Staff ID`
  - `Images::Base64 Image = <base64 of selected image>`

If your system uses a different table or reference strategy, the AI Agent needs that clarified.

### 8.3 Refresh rule
After inserting, FileMaker must regenerate the page (or refresh) so the new base64 appears in `$Image Source HTML`.

Typical approaches:
- re-run the page generation script
- refresh window / flush caches
- re-set `$$Page` and load WebViewer again

---

## 9) Putting It Together: Minimum End-to-End Implementation

### 9.1 FileMaker variables (script names)
~~~filemaker
Set Variable [ $Insert Image Script ; Value: "Insert Staff Photo" ]
~~~

### 9.2 HTML (upload field + image preview)
~~~html
<div class='form-group form-group-2'>
  <label for='materialPhoto'>Photo</label>
  <input type='file' id='materialPhoto' />
</div>

<div class='image-container' onclick='openImageModal()'>
  <img src='{{IMAGE_SOURCE_HTML}}' alt='Staff Photo' class='summary-image' />
  <i class='fas fa-search-plus summary-image-icon'></i>
</div>
~~~

### 9.3 JS (intercept + modal)
~~~js
document.getElementById('materialPhoto').addEventListener('click', function(event) {
  event.preventDefault();
  FileMaker.PerformScript('Insert Staff Photo');
});

function openImageModal() {
  const modal = document.getElementById('imageModal');
  const modalImage = document.getElementById('modalImage');
  const thumbnail = document.querySelector('.summary-image');
  modalImage.src = thumbnail.src;
  modal.style.display = 'flex';
}

function closeImageModal() {
  document.getElementById('imageModal').style.display = 'none';
}
~~~

---

## 10) Quality Checklist for the AI Agent

When implementing file upload + image display, the AI Agent must verify:

### Upload trigger
1. A file input exists with a stable ID (example: `materialPhoto`)
2. A JS click handler intercepts the input and calls a FileMaker script
3. The FileMaker script name is injected from a FileMaker variable (preferred standard)
4. The native file dialog does not open

### Display
5. The image `src` is built as a data URI
6. A placeholder base64 is used when no image exists
7. The image is visible in the summary section with correct classes

### Modal
8. Modal HTML exists (`imageModal`, `modalImage`)
9. Clicking thumbnail opens modal
10. Close button and outside click closes modal

### Refresh
11. After inserting photo, FileMaker refreshes page so the new image appears

---

## 11) Common Failure Modes and Fixes

### 11.1 Clicking the file input does nothing
Cause:
- No click listener attached
- Wrong element ID in JS
Fix:
- Ensure `document.getElementById('materialPhoto')` matches HTML

### 11.2 Browser file picker still opens
Cause:
- missing `event.preventDefault()`
Fix:
- add `event.preventDefault()` at top of click handler

### 11.3 FileMaker script does not run
Cause:
- wrong script name
- script not available in file context
Fix:
- ensure `$Insert Image Script` is correct
- ensure script exists and is accessible

### 11.4 Image never updates after inserting
Cause:
- Insert script stored the image but page did not refresh
Fix:
- re-run page generator and refresh WebViewer

### 11.5 Modal opens but shows blank image
Cause:
- `.summary-image` selector not found
- modal image ID mismatch
Fix:
- ensure thumbnail has class `summary-image`
- ensure modal image id is `modalImage`

---

## 12) Clarifications Needed (Ambiguities the AI Agent Cannot Guess)

These must be clarified to make the AI Agent implementation perfect:

1. **Exact FileMaker Insert Photo script behavior**
   - Does it:
     - open dialog using Insert Picture / Open File?
     - store into container first?
     - then convert to base64 and store in `Images::Base64 Image`?
   - The AI Agent needs the canonical internal approach.

2. **Image formats**
   - Are you standardizing on PNG only?
   - Or do you support JPG, JPEG, WEBP?
   - If multiple formats, how do you store MIME type and build the data URI prefix?

3. **Reference strategy**
   - You use `Images::Reference = $$Staff ID`.
   - Is `Reference` always the entity ID?
   - Or do you combine it with type (example: `StaffPhoto|<ID>`)?

4. **Single image vs multiple images per entity**
   - Is there one profile photo per staff record?
   - Or can staff have multiple images?

5. **Security rules**
   - Do you validate file size?
   - Do you validate image type?
   - Do you compress or resize images before saving?

6. **Where to place the upload UI**
   - Your page places it in Tab 1 and shows preview in Summary.
   - Is that always your standard pattern?

---

## 13) Implementation Standard (What the AI Agent Must Produce)

When asked to implement "Upload Photo + Show Photo" on a page, the AI Agent must produce:

1. FileMaker variables:
   - `$Insert Image Script`
   - `$Image`, `$Image Placeholder`, `$Image Source HTML`

2. HTML:
   - `<input type='file' id='materialPhoto'>`
   - `<img src='{{IMAGE_SOURCE_HTML}}' class='summary-image'>`
   - modal markup

3. JavaScript:
   - intercept click and call FileMaker script
   - modal open/close functions

4. FileMaker contract:
   - Insert script stores base64 image for the current entity reference
   - page refreshes after insert

---

# CauferoAppStarter Documentation
## Combined Fields (Grouped Inputs in One Visual Control) How They Are Implemented and Used

> Purpose of this doc  
> Train an AI Agent to correctly implement **combined fields** in CauferoAppStarter WebViewer pages, including:
> 1) What a combined field is in this framework  
> 2) The HTML structure used to render multiple inputs as one grouped control  
> 3) How dropdowns and text inputs are mixed inside the combined control  
> 4) How values are preloaded from FileMaker variables  
> 5) How values are read and saved back to FileMaker  
> 6) Layout rules, IDs, and common failure mode fixes

This documentation is based on the pattern in your sample page under Tab 1, where **Full Name** is implemented as a combined field:
- `title` (dropdown)
- `first_name` (text)
- `other_names` (text)
- `last_name` (text)

The combined control is rendered using:
- `combined-inputs-container`
- `combined-inputs-dropdown`
- `combined-inputs-input`

---

## 1) What a Combined Field Means in CauferoAppStarter

A combined field is a UI pattern where multiple related fields are grouped so the user experiences them as one unit.

Examples:
- Full Name: Title + First + Other + Last
- Address: Street + City + Region + Country (possible future use)
- Dimensions: Length + Width + Height + Unit (possible future use)
- Currency Amount: Amount + Currency (possible future use)

In your framework, combined fields are not a special input type.
They are standard HTML controls arranged as a single grouped layout.

---

## 2) Why Combined Fields Exist (UI and Data Reasons)

Combined fields are used when:
1. the fields are logically one thing (Full Name)
2. users should not mentally treat them as separate sections
3. you want one label for the group, not one label per input
4. you want a compact layout that looks intentional and modern

---

## 3) Canonical Combined Field Pattern (Full Name Example)

### 3.1 HTML structure from the sample page
Your page uses this exact structure:

~~~html
<div class='form-group form-group-2'>
  <label for='size'>Full Name</label>

  <div class='combined-inputs-container'>
    <select class='combined-inputs-dropdown' id='title' aria-label='Title' style='width:50%'>
      {{TITLES_HTML}}
    </select>

    <input class='combined-inputs-input' type='text' id='first_name' placeholder='First Name' aria-label='Length' style='width:50%' value='{{FIRST_NAME}}'>

    <input class='combined-inputs-input' type='text' id='other_names' placeholder='Other Names' aria-label='Length' style='width:50%' value='{{OTHER_NAMES}}'>

    <input class='combined-inputs-input' type='text' id='last_name' placeholder='Last Name' aria-label='Length' style='width:50%' value='{{LAST_NAME}}'>
  </div>
</div>
~~~

### 3.2 Combined field components
A combined field has:
1. One wrapper form group
2. One label (for the whole group)
3. One `combined-inputs-container`
4. Multiple inputs inside, each with its own ID

---

## 4) Mandatory Rules (Non-Negotiable)

### 4.1 Every inner input MUST have a stable `id`
Even though the UI groups fields, the data model is separate.
Saving depends on input IDs.

Example IDs:
- `title`
- `first_name`
- `other_names`
- `last_name`

### 4.2 Only one label for the combined group
Do not create separate `<label>` tags for each internal input.
Use placeholders and aria-labels instead.

### 4.3 Each internal input MUST have a placeholder or aria-label
Because there is one label for the group, each internal control must identify itself.

Example:
- `placeholder='First Name'`
- `aria-label='Title'`

### 4.4 Combined fields must be saved as separate values
Even if the combined UI looks like one field, the save logic treats each piece independently.

---

## 5) Preloading Combined Field Values (FileMaker Side)

### 5.1 Values are prepared as separate variables
Your script sets:
~~~filemaker
Set Variable [ $Title ; Value: GetValue ( $Link Record As List ; 2 ) ]
Set Variable [ $First Name ; Value: GetValue ( $Link Record As List ; 3 ) ]
Set Variable [ $Other Names ; Value: GetValue ( $Link Record As List ; 5 ) ]
Set Variable [ $Last Name ; Value: GetValue ( $Link Record As List ; 4 ) ]
~~~

### 5.2 Dropdown options for title are generated by `Get Value List`
Your script:
~~~filemaker
Perform Script [ “Get Value List” ; Parameter: "Dropdown|Titles|" & $Title ]
Set Variable [ $Titles HTML ; Value: Get ( ScriptResult ) ]
~~~

Then injects:
~~~html
<select id='title'> " & $Titles HTML & " </select>
~~~

---

## 6) Saving Combined Field Values (JavaScript)

### 6.1 Canonical save pattern from your page
Your save function reads each internal field separately:

~~~js
const staff_number = document.getElementById('staff_number').value;

const title = document.getElementById('title').value;
const first_name = document.getElementById('first_name').value;
const other_names = document.getElementById('other_names').value;
const last_name = document.getElementById('last_name').value;
~~~

Then joins parameters:
~~~js
const parameters = [ staff_number, title, first_name, other_names, last_name, ... ].join('|');
FileMaker.PerformScript('Save Staff', parameters);
~~~

### 6.2 Encoding rules for combined fields
In your sample page, the name inputs are not encoded.
This is acceptable if:
- names do not contain the delimiter `|`
- you trust the input

But safest approach:
- encode everything that could contain delimiter characters

Recommended safe pattern:
~~~js
const first_name = encodeURIComponent(document.getElementById('first_name').value);
const other_names = encodeURIComponent(document.getElementById('other_names').value);
const last_name = encodeURIComponent(document.getElementById('last_name').value);
~~~

If your system has a strict standard for what must be encoded, clarify it.

---

## 7) Layout and Styling Rules

### 7.1 Grid container placement
Your combined field sits in:
- `form-grid1`
and the internal inputs align in a grouped row/stack based on CSS.

Example wrapper:
~~~html
<div class='form-grid1'>
  <div class='form-group form-group-2'>
    ...
  </div>
</div>
~~~

### 7.2 CSS contract (assumed)
The combined control relies on CSS classes:
- `.combined-inputs-container`
- `.combined-inputs-dropdown`
- `.combined-inputs-input`

The AI Agent must assume:
- these classes already exist in your CSS scripts (`🖌️ Use Details CSS` / `🖌️ Use Dashboard CSS`)
- they are designed to create a unified visual group (shared border radius, spacing, background, etc.)

If these classes vary by theme, the agent should not override them unless required.

### 7.3 Width usage
Your sample uses inline `style='width:50%'` on each input.
This suggests:
- you intentionally enforce sizes at the field level

Rule for the AI Agent:
- match the local pattern of width control (inline style) when adding new combined fields to existing pages.

If you later standardize widths purely in CSS, update this rule.

---

## 8) Combined Fields With Mixed Control Types

A combined field can include:
- dropdown + text inputs (your Full Name example)
- text + datepicker
- number + unit dropdown
- prefix + main value + suffix

### 8.1 Rules for mixing types
1. Each internal control must have:
   - stable ID
   - placeholder or aria-label
2. Special controls keep their own scripts:
   - datepicker: must still be listed in dateFields initialization
   - dropdown: must still be populated via Mode A or Mode B
3. The combined control does not change how each input is wired, only how it is laid out

---

## 9) Copy-Paste Templates

### 9.1 Template A: Dropdown + 2 text inputs
~~~html
<div class='form-group form-group-2'>
  <label>Group Label</label>

  <div class='combined-inputs-container'>
    <select class='combined-inputs-dropdown' id='field_a' aria-label='Field A' style='width:30%'>
      {{OPTIONS_HTML}}
    </select>

    <input class='combined-inputs-input' type='text' id='field_b' placeholder='Field B' aria-label='Field B' style='width:35%' value='{{FIELD_B}}'>

    <input class='combined-inputs-input' type='text' id='field_c' placeholder='Field C' aria-label='Field C' style='width:35%' value='{{FIELD_C}}'>
  </div>
</div>
~~~

JS read example:
~~~js
const field_a = document.getElementById('field_a').value;
const field_b = document.getElementById('field_b').value;
const field_c = document.getElementById('field_c').value;
~~~

### 9.2 Template B: 2 text inputs + datepicker
~~~html
<div class='form-group form-group-2'>
  <label>Group Label</label>

  <div class='combined-inputs-container'>
    <input class='combined-inputs-input' type='text' id='x' placeholder='X' aria-label='X' style='width:35%' value='{{X}}'>

    <input class='combined-inputs-input' type='text' id='y' placeholder='Y' aria-label='Y' style='width:35%' value='{{Y}}'>

    <div style='position: relative; width:30%'>
      <input class='combined-inputs-input' type='text' id='z_date' aria-label='Z Date' value='{{Z_DATE_MYSQL}}' style='width:100%'>
      <i class='fas fa-calendar-alt' id='z_date_icon' style='position: absolute; right: 10px; top: 50%; transform: translateY(-50%); color: #666; cursor: pointer;'></i>
    </div>
  </div>
</div>
~~~

JS date integration:
- add `{ id: 'z_date', iconId: 'z_date_icon' }` into dateFields
- read with `getMySQLFormattedDate('z_date')`

---

## 10) Quality Checklist for the AI Agent

When implementing combined fields, the AI Agent must verify:

### Structure
1. One label covers the group
2. One `combined-inputs-container` wraps all internal controls
3. Every internal control has:
   - stable ID
   - placeholder or aria-label

### Data
4. Each internal control is preloaded correctly:
   - text inputs via `value='...'`
   - dropdown via injected `<option>` HTML or JS population

### Save
5. Each internal control is read separately in JS
6. Values are included in parameters in correct order
7. Values are encoded if needed for delimiter safety

### Integration
8. Special controls still obey their own required scripts:
   - datepicker fields must be registered for initialization
   - dynamic dropdowns must be populated

---

## 11) Common Failure Modes and Fixes

### 11.1 Only the first control saves, others are ignored
Cause:
- JS save function reads only one field
Fix:
- ensure all internal IDs are read and included in parameter list

### 11.2 Dropdown inside combined field appears empty
Cause:
- forgot to inject `$Options HTML`
Fix:
- ensure:
  - `<select ...> " & $OptionsHTML & " </select>`

### 11.3 Combined field looks broken (spacing and borders wrong)
Cause:
- missing CSS class names or mismatched structure
Fix:
- ensure:
  - wrapper is exactly `combined-inputs-container`
  - inner controls use `combined-inputs-input` or `combined-inputs-dropdown`

### 11.4 Accessibility confusion
Cause:
- multiple inputs with only one label
Fix:
- ensure each input has placeholder or aria-label

---

## 12) Clarifications Needed (Ambiguities the AI Agent Cannot Guess)

These must be clarified for perfect AI implementation:

1. **Encoding rule**
   - Do you want combined field text inputs encoded like textareas, or only certain fields?
   - Your sample does not encode name fields, but encoding is safer.

2. **Required vs optional segments**
   - In Full Name:
     - is `title` required?
     - are `other_names` optional?
   - The AI Agent needs validation rules.

3. **Width standard**
   - Your sample uses inline widths (50% each).
   - Is this required always, or can the agent rely on CSS only?

4. **CSS class contract**
   - Are `.combined-inputs-*` classes guaranteed across all themes?
   - If not, what is the fallback pattern?

---

## 13) Implementation Standard (What the AI Agent Must Produce)

When asked to implement a combined field in CauferoAppStarter, the AI Agent must produce:

1. A wrapper form group with one label
2. A `combined-inputs-container` holding multiple inputs
3. Each internal input:
   - has stable ID
   - has placeholder or aria-label
   - is preloaded correctly from FileMaker variables
4. Save logic reads and saves each component separately
5. Any special controls inside the combined group still obey their own initialization rules

---

# CauferoAppStarter Documentation
## Buttons and How They Are Implemented and Used

> Purpose of this doc  
> Train an AI Agent to correctly implement **buttons** in CauferoAppStarter WebViewer pages, including:
> 1) Button categories used in the framework  
> 2) The HTML patterns and class contracts for each category  
> 3) How buttons call JavaScript functions  
> 4) How JavaScript functions call FileMaker scripts with `FileMaker.PerformScript`  
> 5) How parameters are built, encoded, and delimited  
> 6) How conditional buttons are shown or hidden based on FileMaker variables  
> 7) Common pitfalls and correct fixes

This documentation is based on the patterns in your sample page, which uses buttons for:
- Save
- Cancel
- Print Staff ID Card
- Show Attendance Records
- Grant System Access
- Segmented control buttons (covered in a separate doc)
- Summary image modal click (clickable container behaves like a button)

---

## 1) What a Button Means in CauferoAppStarter

A button is any clickable UI element that triggers an action:
- Save record
- Navigate back to list page
- Open a modal page
- Load a subtable section
- Run a server-side calculation
- Print
- Grant access
- View record details

In this framework, buttons are implemented as:
- HTML `<button>` elements styled using theme CSS classes
- Each button calls a JavaScript function via `onclick="..."`
- JavaScript then calls FileMaker scripts using `FileMaker.PerformScript(scriptName, parameters)`

---

## 2) Button Categories in the Framework

The AI Agent must recognize and implement these categories:

1. **Primary action buttons**  
   Examples: Save, Submit, Create, Confirm

2. **Secondary action buttons**  
   Examples: Cancel, Back, Close, Reset

3. **Context buttons**  
   Examples: Print ID, Show Records, Refresh Metrics

4. **Conditional buttons**  
   Buttons that appear only when a condition is true  
   Example: "Give access" appears only when `$Username` is empty

5. **Icon buttons**  
   Buttons with Font Awesome icons for quick recognition

6. **Non-button clickable elements**  
   Containers or images that act like buttons  
   Example: `.image-container` clickable to open image modal

---

## 3) Core Rules (Non-Negotiable)

### 3.1 Buttons do not call FileMaker scripts directly
Buttons call JavaScript functions.
JavaScript functions call FileMaker scripts.

This creates a consistent architecture:
- UI layer: HTML
- orchestration layer: JavaScript
- server/data layer: FileMaker scripts

### 3.2 Button actions must be declared in variables (FileMaker side)
Script names are set as variables so the page generator can swap them easily.

Example from your script:
~~~filemaker
Set Variable [ $Save Record Script ; Value: "Save Staff" ]
Set Variable [ $Cancel Script ; Value: "+++ Staff List Page" ]
Set Variable [ $Grant System Access Script ; Value: "Grant System Access To Staff" ]
Set Variable [ $Get Attendance FM Script ; Value: "Get Staff Attendance" ]
~~~

### 3.3 Button click must be deterministic
The AI Agent must ensure every button:
- has a clear label
- calls a defined function
- function exists in the `$Scripts` bundle
- function calls the correct FileMaker script name variable

### 3.4 Parameter delimiter is `|`
Your system uses:
~~~js
const parameters = [ a, b, c ].join('|');
~~~

Therefore:
- never allow raw user input containing `|`
- encode values that can contain special characters

---

## 4) HTML Implementation Patterns

### 4.1 Standard button with CSS class
From your summary section:

~~~html
<button class='save-button' onclick="saveItemInfo()">
  <i class='fas fa-save'></i> Save
</button>

<button class='cancel-button' onclick="cancelAction()">
  <i class='fas fa-times-circle'></i> Cancel
</button>
~~~

Key components:
- `class`: determines styling based on theme CSS
- `onclick`: calls JS function
- icon: `<i class='...'></i>`
- label text

### 4.2 Full-width primary button (context action)
From your page:

~~~html
<button class='button_blue' style='width: 100%; justify-content: center;' onclick="printStaffIDCard()">
  <i class='fa-solid fa-file'></i> Print Staff ID Card
</button>
~~~

### 4.3 Small context button (inline action)
From attendance section:

~~~html
<button class='button_blue_' style='width: auto' onclick="getAttendance()">
  <i class='fas fa-plus'></i> Show Records
</button>
~~~

### 4.4 Conditional button
From Tab 9 (Access), the button appears only if `$Username` is empty:

~~~html
<button class='button_green_' style='width: auto; position: relative;' onclick="grantSystemAccess()">
  <i class='fas fa-plus'></i> Give him/her access to {{APP_NAME}}
</button>
~~~

This is injected conditionally using FileMaker `If ( IsEmpty ( $Username ) ; ... ; ... )`.

---

## 5) JavaScript Implementation Patterns

### 5.1 Navigation or cancel action button
From your page:

~~~js
function cancelAction() {
  FileMaker.PerformScript('+++ Staff List Page');
}
~~~

In the script generator, the name is injected using `$Cancel Script`:
~~~filemaker
Set Variable [ $Cancel Action Script ;
"function cancelAction() {
  FileMaker.PerformScript('" & $Cancel Script & "');
}"
]
~~~

### 5.2 Save action button (collect inputs and send to FileMaker)
From your page:

~~~js
function saveItemInfo() {

  const staff_number = document.getElementById('staff_number').value;
  const title = document.getElementById('title').value;
  const first_name = document.getElementById('first_name').value;

  const genderButton = document.querySelector('.segmented-control button.active');
  const gender = genderButton.textContent.trim();

  const dob = getMySQLFormattedDate('dob');

  const residential_address = encodeURIComponent(document.getElementById('residential_address').value);

  const parameters = [
    staff_number,
    title,
    first_name,
    gender,
    dob,
    residential_address
  ].join('|');

  FileMaker.PerformScript('Save Staff', parameters);
}
~~~

### 5.3 Button that runs a FileMaker action with computed parameters
From your page (Grant Access):

~~~js
function grantSystemAccess() {

  const staff_number = encodeURIComponent(document.getElementById('staff_number').value);
  const first_name = encodeURIComponent(document.getElementById('first_name').value);
  const last_name = encodeURIComponent(document.getElementById('last_name').value);
  const email = encodeURIComponent(document.getElementById('email').value);

  const parameters = [ staff_number, first_name, last_name, email ].join('|');

  FileMaker.PerformScript('Grant System Access To Staff', parameters);
}
~~~

### 5.4 Button that refreshes data using datepicker helper
From your page (Attendance):

~~~js
function getAttendance() {

  const from_date = getMySQLFormattedDate('from_date');
  const to_date = getMySQLFormattedDate('to_date');

  const parameters = [ from_date, to_date ].join('|');

  FileMaker.PerformScript('Get Staff Attendance', parameters);
}
~~~

---

## 6) Parameter Packaging Rules

### 6.1 Use `|` join for multi-parameter payloads
Standard:
~~~js
const parameters = [ p1, p2, p3 ].join('|');
~~~

### 6.2 Encoding policy (critical)
Encode any value that can include:
- line breaks
- commas
- special characters
- spaces (optional)
- or delimiter `|` (must)

Your sample encodes:
- textareas: residential_address, postal_address
- role: role

Your sample does not encode many text fields like names.
This is workable but unsafe if users can type `|`.

Recommended framework rule:
- encode every user-entered string except numeric fields, unless you have a strict reason not to

Safe pattern:
~~~js
const safe = encodeURIComponent(document.getElementById('field').value);
~~~

### 6.3 Decode policy in FileMaker
If you encode in JS, FileMaker must decode.
If your framework has a dedicated decode function, the AI Agent must use it.
If not, you must define the canonical decode method.

---

## 7) Script Name Injection Pattern (FileMaker Side)

Buttons do not hardcode script names.
Script names are set in variables and injected into JS.

Example:
~~~filemaker
Set Variable [ $Get Attendance FM Script ; Value: "Get Staff Attendance" ]

Set Variable [ $Get Attendance Script ;
"function getAttendance() {
  const from_date = getMySQLFormattedDate('from_date');
  const to_date = getMySQLFormattedDate('to_date');
  const parameters = [ from_date, to_date ].join('|');
  FileMaker.PerformScript('" & $Get Attendance FM Script & "', parameters);
}"
]
~~~

### Rule
Every button that triggers a FileMaker script must:
1. define a `$... Script` variable containing the script name
2. define a `$... JS` variable containing the JavaScript function
3. include that JS in the `$Scripts` bundle
4. reference the JS function in HTML `onclick`

---

## 8) Conditional Buttons (Display Logic)

### 8.1 FileMaker decides if a button should exist
In Tab 9:
- if `$Username` is empty: show "Give access" button
- else: show user details and role dropdown

Pattern:
~~~filemaker
If (
  IsEmpty ( $Username ) ;
  "<button ... onclick='grantSystemAccess()'>...</button>" ;
  "<div>... user details ...</div>"
)
~~~

### Rule
Conditional buttons should be controlled by FileMaker page generation, not by JS hiding.
This prevents hidden actions from being accidentally triggered.

---

## 9) Button Placement Patterns Used in Pages

### 9.1 Summary section action buttons
Your summary section uses a `.button-container` with Save and Cancel.

~~~html
<div class='button-container'>
  <button class='save-button' onclick="saveItemInfo()"><i class='fas fa-save'></i> Save</button>
  <button class='cancel-button' onclick="cancelAction()"><i class='fas fa-times-circle'></i> Cancel</button>
</div>
~~~

### 9.2 Context button inside a form grid row
Attendance tab shows "Show Records" aligned with the date filters.

~~~html
<div class='form-group-2'>
  <label for='gps'> . </label>
  <button class='button_blue_' style='width: auto' onclick="getAttendance()">
    <i class='fas fa-plus'></i> Show Records
  </button>
</div>
~~~

Rule:
- Use an empty label placeholder where needed to keep grid alignment consistent.

---

## 10) Non-Button Clickable Elements (Button Behavior Without `<button>`)

Your summary photo uses a clickable container:

~~~html
<div class='image-container' onclick='openImageModal()'>
  <img src='...' alt='Staff Photo' class='summary-image' />
  <i class='fas fa-search-plus summary-image-icon'></i>
</div>
~~~

Rule:
- Any clickable container must still call a JS function
- Keep the same architecture: HTML event triggers JS function

---

## 11) Copy-Paste Templates

### 11.1 Template A: Primary Save button
~~~html
<button class='save-button' onclick="saveItemInfo()">
  <i class='fas fa-save'></i> Save
</button>
~~~

### 11.2 Template B: Secondary Cancel button
~~~html
<button class='cancel-button' onclick="cancelAction()">
  <i class='fas fa-times-circle'></i> Cancel
</button>
~~~

### 11.3 Template C: Context button that calls FileMaker script with params
~~~html
<button class='button_blue_' style='width:auto' onclick="doActionX()">
  <i class='fas fa-play'></i> Run Action
</button>
~~~

JS:
~~~js
function doActionX() {
  const a = encodeURIComponent(document.getElementById('a').value);
  const b = encodeURIComponent(document.getElementById('b').value);
  const parameters = [ a, b ].join('|');
  FileMaker.PerformScript('{{FM_SCRIPT_NAME}}', parameters);
}
~~~

### 11.4 Template D: Conditional button block (FileMaker)
~~~filemaker
Set Variable [ $HTML ;
If (
  {{condition}} ;
  "<button class='button_green_' onclick=\"runSpecialAction()\"><i class='fas fa-plus'></i> Do Special Thing</button>" ;
  ""
)
]
~~~

---

## 12) Quality Checklist for the AI Agent

When implementing buttons, the AI Agent must verify:

### HTML
1. button has correct class for style
2. button text describes action clearly
3. button uses `onclick="functionName()"`
4. icons use Font Awesome classes already loaded on the page

### JS
5. function exists in `$Scripts`
6. function builds params using `|` delimiter
7. values are encoded when needed
8. function calls `FileMaker.PerformScript(scriptName, params)`

### FileMaker
9. script name is stored in a `$... Script` variable
10. HTML and JS strings inject that variable correctly
11. conditional button logic is generated by FileMaker when required

---

## 13) Common Failure Modes and Fixes

### 13.1 Button does nothing on click
Cause:
- JS function name mismatch between HTML and JS
Fix:
- ensure `onclick="saveItemInfo()"` matches `function saveItemInfo() { ... }`

### 13.2 FileMaker script does not run
Cause:
- wrong script name injected
Fix:
- confirm `$Save Record Script` contains the exact script name

### 13.3 Parameters arrive broken in FileMaker
Cause:
- user input contains `|`
Fix:
- encode values before join
- decode in FileMaker

### 13.4 Page loads but JS errors stop all buttons
Cause:
- one script block has syntax error
Fix:
- ensure every `$... Script` string is valid JS
- avoid unescaped quotes inside injected strings

### 13.5 Conditional button appears when it should not
Cause:
- FileMaker condition is wrong
Fix:
- confirm the actual variable (example: `$Username`) and its empty state

---

## 14) Clarifications Needed (Ambiguities the AI Agent Cannot Guess)

To make button implementation perfect, clarify these:

1. **Encoding standard**
   - Should the AI Agent encode all user-entered values before joining parameters?
   - If yes, what is the canonical decode method in FileMaker?

2. **Button class standard**
   - Which classes are official and must be used:
     - `save-button`, `cancel-button`, `button_blue`, `button_blue_`, `button_green_`
   - Are there additional button classes for danger actions (delete)?

3. **Disable and loading state**
   - Do you want the AI Agent to implement:
     - disabling buttons during script calls
     - showing a spinner or "Saving..." state

4. **Confirm dialogs**
   - Do you want confirmation prompts for destructive actions (delete, remove access)?

---

## 15) Implementation Standard (What the AI Agent Must Produce)

When asked to add a button in CauferoAppStarter, the AI Agent must produce:

1. HTML `<button>` with the correct CSS class and icon
2. `onclick` calling a named JavaScript function
3. JavaScript function that:
   - collects any required values by id
   - encodes values where necessary
   - builds parameters using `|`
   - calls `FileMaker.PerformScript` with an injected script name variable
4. FileMaker variables:
   - one variable holding the target script name
   - one variable holding the JS function code
5. Conditional logic in FileMaker for buttons that must appear only in specific states

---

# CauferoAppStarter Documentation
## Segmented Control Buttons and How They Are Implemented and Used

> Purpose of this doc  
> Train an AI Agent to correctly implement **segmented control buttons** in CauferoAppStarter WebViewer pages, including:
> 1) The HTML structure used to render segmented controls  
> 2) How default (preselected) state is applied from FileMaker data  
> 3) The JavaScript logic that toggles the active selection  
> 4) How the selected value is read and sent back to FileMaker on Save  
> 5) Styling rules and class contracts  
> 6) Common failure modes and correct fixes

This documentation is based on your sample page implementation for **Gender**:
- segmented control container: `.segmented-control`
- segment buttons: `<button ...>`
- selected state: `class='active'`
- click handler: `onclick="selectSegmentedControl(this)"`
- save logic reads: `.segmented-control button.active`

---

## 1) What a Segmented Control Means in CauferoAppStarter

A segmented control is a UI control that looks like a single grouped toggle, but it is actually multiple buttons.
Only one option is selected at a time (single-select).

Use it when:
- options are few (2–5)
- options are mutually exclusive
- you want a fast tap/click selection
- dropdown would feel slow or unnecessary

Examples:
- Gender: Male / Female
- Status: Active / Inactive
- Type: Individual / Company
- Payment mode: Cash / MoMo / Card
- Yes/No decisions

In this framework, segmented controls are implemented using:
- plain HTML buttons
- a CSS class `active` for the selected state
- a shared JS function `selectSegmentedControl(button)` that sets active state

---

## 2) Canonical HTML Pattern

### 2.1 Basic structure
A segmented control is composed of:
1. A wrapper with class `.segmented-control`
2. Multiple `<button>` elements inside the wrapper
3. Each button has:
   - display text (the value)
   - onclick handler `selectSegmentedControl(this)`
   - optional `class='active'` when preselected

### 2.2 Example from the sample page (Gender)
~~~html
<div class='form-group form-group-2'>
  <label for='gender'>Gender</label>

  <div class='segmented-control'>
    <button class='active' onclick="selectSegmentedControl(this)">Male</button>
    <button onclick="selectSegmentedControl(this)">Female</button>
  </div>
</div>
~~~

### 2.3 Preselecting the active button (FileMaker-driven)
Your sample uses FileMaker string concatenation:

~~~html
<button " & If ( $Gender = "Male" ; "class='active'" ) & " onclick="selectSegmentedControl(this)">Male</button>
<button " & If ( $Gender = "Female" ; "class='active'" ) & " onclick="selectSegmentedControl(this)">Female</button>
~~~

### Rule
Preselect by injecting `class='active'` onto exactly one button if a stored value exists.

---

## 3) FileMaker Preprocessing for Default Selection

### 3.1 The field value is loaded from the record
Your script sets:
~~~filemaker
Set Variable [ $Gender ; Value: GetValue ( $Link Record As List ; 7 ) ]
~~~

### 3.2 The HTML uses `$Gender` to determine which button gets `active`
~~~filemaker
If ( $Gender = "Male" ; "class='active'" )
If ( $Gender = "Female" ; "class='active'" )
~~~

### Rule
Segmented control defaults must be determined server-side (FileMaker) during page build, not by guessing in JS.

---

## 4) JavaScript Toggle Logic

### 4.1 Canonical toggle function from your page
~~~js
function selectSegmentedControl(button) {
  const buttons = document.querySelectorAll('.segmented-control button');
  buttons.forEach(btn => btn.classList.remove('active'));
  button.classList.add('active');
}
~~~

### What this function does
1. Finds all segmented control buttons (globally)
2. Removes active from all
3. Adds active to the clicked button

---

## 5) Important Constraint: Multiple Segmented Controls on One Page

### Problem in current implementation
Your sample uses:
~~~js
document.querySelectorAll('.segmented-control button')
~~~
This means:
- if there are multiple segmented controls on the page, clicking one will clear the active state in all segmented controls

### Required implementation rule for scalable pages
The AI Agent must scope the selection to the clicked control only.

### 5.1 Correct scoped version (recommended standard)
~~~js
function selectSegmentedControl(button) {
  const container = button.closest('.segmented-control');
  if (!container) return;

  const buttons = container.querySelectorAll('button');
  buttons.forEach(btn => btn.classList.remove('active'));
  button.classList.add('active');
}
~~~

### Rule
If the page can ever have more than one segmented control, always use the scoped version.

If you intentionally support only one segmented control per page, keep the global version.
The safer standard is scoped.

---

## 6) Reading the Selected Value (Save and Actions)

### 6.1 Canonical read pattern from your page
Your save function does:

~~~js
const genderButton = document.querySelector('.segmented-control button.active');
const gender = genderButton.textContent.trim();
~~~

Then includes `gender` in parameters.

### 6.2 Mandatory robustness rule
If no button is selected, `genderButton` will be null and Save will crash.

A safe pattern:
~~~js
const genderButton = document.querySelector('.segmented-control button.active');
const gender = genderButton ? genderButton.textContent.trim() : '';
~~~

### Rule
When reading a segmented control value:
- default to empty string if none is selected
- enforce required logic before calling FileMaker script if necessary

---

## 7) Passing the Segmented Value to FileMaker

### 7.1 Standard parameter join pattern
~~~js
const parameters = [ ..., gender, ... ].join('|');
FileMaker.PerformScript('Save Staff', parameters);
~~~

### Rule
Segment values must be stable strings.
Prefer using values that match your database enums exactly.

Example:
- store "Male" / "Female"
not:
- store "M" / "F" unless your database expects it

If the displayed text differs from stored value, use a data attribute.

---

## 8) Using Data Attributes (When Display Text Is Not the Stored Value)

If you want button text to be user-friendly but store an internal code:
- use `data-value`

Example:
~~~html
<div class='segmented-control' id='payment_mode'>
  <button data-value='cash' onclick="selectSegmentedControl(this)">Cash</button>
  <button data-value='momo' onclick="selectSegmentedControl(this)">MoMo</button>
  <button data-value='card' onclick="selectSegmentedControl(this)">Card</button>
</div>
~~~

Read value:
~~~js
const activeBtn = document.querySelector('#payment_mode button.active');
const payment_mode = activeBtn ? activeBtn.getAttribute('data-value') : '';
~~~

### Rule
Use `data-value` whenever:
- displayed label is not the stored value
- you may change button text later without changing database values

---

## 9) Preselecting by Value With Data Attributes (FileMaker)

If using `data-value`, FileMaker must set the `active` class by comparing the stored value.

Example pattern:
~~~filemaker
<button " & If ( $PaymentMode = "cash" ; "class='active'" ) & " data-value='cash' onclick="selectSegmentedControl(this)">Cash</button>
~~~

### Rule
Preselect logic must always reference the stored enum, not display text.

---

## 10) Styling Rules and Class Contract

### 10.1 Required classes
Segmented control depends on these:
- wrapper: `.segmented-control`
- active state: `.active`

### 10.2 CSS source
Your page pulls CSS via:
- `Perform Script [ “🖌️ Use Details CSS” ]`
- `Perform Script [ “🖌️ Use Dashboard CSS” ]`

The AI Agent must:
- reuse `.segmented-control` and `.active`
- not invent new class names unless the CSS is extended intentionally

---

## 11) Copy-Paste Templates

### 11.1 Template A: Simple 2-option segmented control
~~~html
<div class='form-group-2'>
  <label>Status</label>
  <div class='segmented-control' id='status_control'>
    <button class='active' onclick="selectSegmentedControl(this)">Active</button>
    <button onclick="selectSegmentedControl(this)">Inactive</button>
  </div>
</div>
~~~

JS:
~~~js
function selectSegmentedControl(button) {
  const container = button.closest('.segmented-control');
  if (!container) return;
  container.querySelectorAll('button').forEach(btn => btn.classList.remove('active'));
  button.classList.add('active');
}
~~~

Read value:
~~~js
const statusBtn = document.querySelector('#status_control button.active');
const status = statusBtn ? statusBtn.textContent.trim() : '';
~~~

### 11.2 Template B: Stored value differs from label
~~~html
<div class='form-group-2'>
  <label>Payment Mode</label>
  <div class='segmented-control' id='payment_mode'>
    <button data-value='cash' onclick="selectSegmentedControl(this)">Cash</button>
    <button data-value='momo' onclick="selectSegmentedControl(this)">MoMo</button>
    <button data-value='card' onclick="selectSegmentedControl(this)">Card</button>
  </div>
</div>
~~~

Read value:
~~~js
const btn = document.querySelector('#payment_mode button.active');
const paymentMode = btn ? btn.getAttribute('data-value') : '';
~~~

---

## 12) Quality Checklist for the AI Agent

When implementing segmented controls, the AI Agent must verify:

### HTML
1. wrapper uses `.segmented-control`
2. buttons are inside wrapper
3. each button calls `selectSegmentedControl(this)`
4. exactly one button is preselected when a stored value exists

### JS
5. toggle function exists
6. toggle function is scoped (recommended) to avoid multi-control collisions

### Save
7. selected value is read safely (null-safe)
8. value is included in script parameters in correct order

### Data consistency
9. stored values are stable and match database enums
10. if using `data-value`, save uses `data-value` not text

---

## 13) Common Failure Modes and Fixes

### 13.1 Clicking one segmented control breaks another
Cause:
- global selector: `.segmented-control button`
Fix:
- scope to closest container using `button.closest('.segmented-control')`

### 13.2 Save crashes because active button is null
Cause:
- no preselected value and user never clicked
Fix:
- use null-safe read and enforce required selection if needed

### 13.3 Preselect never works
Cause:
- FileMaker comparison value mismatch
Fix:
- ensure `$StoredValue` exactly matches the string being compared
- if using `data-value`, compare against the internal stored enum

### 13.4 Saved value changes when UI label changes
Cause:
- saving `textContent`
Fix:
- use `data-value` for stored value stability

---

## 14) Clarifications Needed (Ambiguities the AI Agent Cannot Guess)

To make segmented controls implementation perfect, confirm:

1. **Multiple segmented controls per page**
   - Is it guaranteed that there will be only one segmented control per page?
   - If not, the scoped JS version must be the standard.

2. **Stored value policy**
   - Do you store the visible label text (Male/Female)?
   - Or do you want internal values (M/F)?

3. **Validation policy**
   - Are segmented controls always required?
   - If required, do you want JS to block Save with an alert?

4. **Theme styling**
   - Are `.segmented-control` and `.active` styles consistent across all themes?

---

## 15) Implementation Standard (What the AI Agent Must Produce)

When asked to implement segmented control buttons in CauferoAppStarter, the AI Agent must produce:

1. A `.segmented-control` container with buttons
2. Preselect logic injected by FileMaker using `class='active'`
3. A JS function `selectSegmentedControl(button)` that toggles active state
4. Save logic that reads selected button safely and sends correct stored value to FileMaker
5. Scoped behavior if multiple segmented controls can exist on the same page

---

# CauferoAppStarter Documentation
## Toggle Switches and How They Are Implemented (WebViewer Pages)

> Purpose of this doc  
> Train an AI Agent to implement a **Toggle Switch** UI control in CauferoAppStarter WebViewer pages, using the provided **User Details** page script as the reference pattern.  
> This doc covers **text states only** (example: `Active` and `Inactive`).  
> Number fields, dropdown-backed toggles, and other variants will be documented separately.

---

## 1) What a Toggle Switch Means Here (CauferoAppStarter Definition)

A **Toggle Switch** is a UI control that:
- lets the user flip between two states (ON/OFF)
- shows the state as a **human-readable text label** beside the switch
- stores the selected state in a DOM element (the label), then includes it in the Save payload

In the sample, the switch controls `Status` and displays:
- `Active` when ON
- `Inactive` when OFF

Key elements in the sample:
- Checkbox input: `<input type='checkbox' id='toggle' onchange="toggle()">`
- State label: `<span class='status' id='status'>...</span>`
- JS initializes the checkbox based on `$Status` (`preselectToggle`)
- JS updates the label when user changes toggle (`toggle()`)

---

## 2) When to Use a Toggle Switch

Use a toggle switch when:
- there are exactly **two** states
- the UI should be fast and visual
- the choice is reversible

Good examples:
- Status: Active / Inactive
- Enabled: Yes / No
- Access: Allowed / Blocked
- Notify: On / Off

AI Agent rule:
- If you have more than 2 states, do not use a toggle. Use segmented control, radio buttons, or dropdown.

---

## 3) Required Pieces (HTML + CSS Contract + JS)

Toggle Switch implementation has 3 layers:

1) HTML markup  
2) CSS classes (assumed to exist in shared CSS)  
3) JavaScript (init + change handler)

### 3.1 HTML (Required Contract)

Reference pattern from the sample:

~~~html
<div class='toggle-container'>
  <label class='toggle-switch'>
    <input type='checkbox' id='toggle' onchange="toggle()">
    <span class='slider'></span>
  </label>
  <span class='status' id='status'>Active</span>
</div>
~~~

Where:
- `toggle-container` controls layout
- `toggle-switch` wraps the checkbox and the slider UI
- `slider` is the styled track/knob element
- `status` is the visible text state

AI Agent rules:
- Use `type='checkbox'`.
- `id='toggle'` and `id='status'` must match the JavaScript selectors unless you update the JS accordingly.
- The label must be present because this implementation stores the state as text for saving.

### 3.2 FileMaker Placement Pattern (Inside Form)

In the sample, the toggle sits inside a form-group:

~~~html
<div class='form-group form-group-2'>
  <label for='status'>Status</label>

  <div class='toggle-container'>
    <label class='toggle-switch'>
      <input type='checkbox' id='toggle' onchange="toggle()">
      <span class='slider'></span>
    </label>
    <span class='status' id='status'>Active</span>
  </div>

</div>
~~~

AI Agent rules:
- The outer `<label>` is the field label. Keep it separate from the switch label wrapper.
- Keep the toggle inside the same grid cell as the field it represents.

### 3.3 CSS Contract (Shared Styles)

The sample loads page CSS via:

~~~filemaker
Perform Script [ “🖌️ Use Details CSS” ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]
~~~

So the toggle relies on shared CSS defining:
- `.toggle-container`
- `.toggle-switch`
- `.slider`
- `.status`

AI Agent rules:
- Do not inline toggle CSS per page unless requested.
- If toggle looks broken, fix the shared CSS script, not the page script.

---

## 4) JavaScript Behavior (Initialization + User Change)

Toggle requires two behaviors:
1) Preselect the checkbox based on FileMaker state (`preselectToggle`)
2) Update the visible text state on change (`toggle()`)

### 4.1 Initialization (Preselect)

The goal:
- If `$Status` is `Active`, checkbox should be ON and label should show `Active`
- Else checkbox should be OFF and label should show `Inactive`

Correct reference logic (conceptual):

~~~javascript
function preselectToggle(statusValue) {
  const toggle = document.getElementById('toggle');
  const status = document.getElementById('status');

  if (statusValue === 'Active') {
    toggle.checked = true;
    status.textContent = 'Active';
  } else {
    toggle.checked = false;
    status.textContent = 'Inactive';
  }
}

window.onload = function() {
  preselectToggle('Active'); /* injected from FileMaker */
};
~~~

#### Important note about the provided script
Your pasted code contains a mismatch:

- It sets: `const item = document.getElementById('item');`
- But the HTML uses: `id='status'`

So the preselect code as pasted will not update the label correctly unless `item` is corrected to `status`.

AI Agent rule:
- Always ensure the JS id matches the HTML id.

### 4.2 Change Handler (When user flips switch)

Reference pattern:

~~~javascript
function toggle() {
  const status = document.getElementById('status');
  const toggle = document.getElementById('toggle');
  status.textContent = toggle.checked ? 'Active' : 'Inactive';
}
~~~

AI Agent rules:
- Always update the label text.
- Keep the label text as the single source of truth for saving (in this pattern).

---

## 5) FileMaker Integration Pattern

### 5.1 Inject initial state into HTML

The label is injected from FileMaker:

~~~filemaker
"<span class='status' id='status'>" & $Status & "</span>"
~~~

AI Agent rules:
- Ensure `$Status` is always defined. If missing, set a default (`Active` or `Inactive`) before page generation.
- Keep `$Status` values consistent with your toggle mapping.

### 5.2 Inject initial state into JS on load

The sample passes `$Status` into `preselectToggle()`:

~~~filemaker
window.onload = function() {
 preselectToggle('" & $Status & "');
};
~~~

AI Agent rules:
- The injected value must exactly match expected strings (`Active` / `Inactive`) unless you change mapping logic.

---

## 6) Saving the Toggle Value (Text Fields Only)

In this pattern, the Save script reads the state from the label:

~~~javascript
const status = document.getElementById('status').textContent.trim();
~~~

Then it joins it into parameters:

~~~javascript
const parameters = [ first_name, last_name, email, role, status ].join('|');
FileMaker.PerformScript('Save User', parameters);
~~~

AI Agent rules:
- Always trim the text to avoid leading/trailing spaces.
- Always include status in the correct position in the pipe-delimited payload.
- The receiving FileMaker script must parse the parameter order exactly.

---

## 7) Recommended Toggle Naming Standard (For AI Agent)

If the page has a single toggle:
- checkbox id: `toggle`
- label id: `status`
- functions: `preselectToggle()` and `toggle()`

If the page will have multiple toggles:
- Use unique ids per toggle (required)
- Use a generic handler that targets specific label ids

Example multi-toggle standard:
- checkbox id: `toggle_status`, label id: `status_label`
- checkbox id: `toggle_notify`, label id: `notify_label`

Generic JS pattern:

~~~javascript
function preselectToggleByIds(toggleId, labelId, stateValue, onText, offText) {
  const toggle = document.getElementById(toggleId);
  const label = document.getElementById(labelId);

  if (!toggle || !label) return;

  if (stateValue === onText) {
    toggle.checked = true;
    label.textContent = onText;
  } else {
    toggle.checked = false;
    label.textContent = offText;
  }
}

function toggleByIds(toggleId, labelId, onText, offText) {
  const toggle = document.getElementById(toggleId);
  const label = document.getElementById(labelId);

  if (!toggle || !label) return;

  label.textContent = toggle.checked ? onText : offText;
}
~~~

AI Agent rule:
- Only implement multi-toggle logic if needed. Otherwise keep the simple single-toggle pattern.

---

## 8) Common Failure Modes (Must Avoid)

### 8.1 ID mismatch (Seen in the sample)
JS references `item` id, HTML uses `status` id.

AI Agent rule:
- Enforce id consistency before returning output.

### 8.2 Missing default state
If `$Status` is empty, toggle may display blank or incorrect.

AI Agent rule:
- Always ensure `$Status` is set server-side before HTML generation.

### 8.3 Saving checkbox state instead of label
If saving uses `toggle.checked` but display uses text, you can desync.

AI Agent rule:
- In this pattern, label text is the source of truth for saving.

---

## 9) Ambiguities and Clarifications Needed (Must Ask User)

To implement this perfectly across the codebase, these must be clarified:

1) What are the canonical text values for status?
   - Is it always `Active` and `Inactive`?
   - Or do some modules use `Enabled/Disabled`, `Yes/No`, `True/False`?

2) Are status values case-sensitive in your database?
   - Example: `active` vs `Active`

3) Do you want the toggle to immediately save on change for some pages?
   - The current pattern only changes UI and saves later via Save button.

4) Should toggles support a third state (unknown/unset)?
   - If yes, toggle is the wrong control and needs a different UI.

AI Agent default behavior if not clarified:
- Use `Active` / `Inactive`
- Treat them as case-sensitive
- Update UI only, save only when user clicks Save

---

## 10) Copy-Paste Templates (AI Agent Output Blocks)

### 10.1 Template: Toggle HTML (Single)
~~~html
<div class='form-group form-group-2'>
  <label for='status'>Status</label>

  <div class='toggle-container'>
    <label class='toggle-switch'>
      <input type='checkbox' id='toggle' onchange="toggle()">
      <span class='slider'></span>
    </label>
    <span class='status' id='status'>Active</span>
  </div>
</div>
~~~

### 10.2 Template: Toggle JS (Single)
~~~javascript
function preselectToggle(statusValue) {
  const toggle = document.getElementById('toggle');
  const status = document.getElementById('status');

  if (!toggle || !status) return;

  if (statusValue === 'Active') {
    toggle.checked = true;
    status.textContent = 'Active';
  } else {
    toggle.checked = false;
    status.textContent = 'Inactive';
  }
}

window.onload = function() {
  preselectToggle('Active'); /* inject FileMaker value */
};

function toggle() {
  const status = document.getElementById('status');
  const toggle = document.getElementById('toggle');

  if (!toggle || !status) return;

  status.textContent = toggle.checked ? 'Active' : 'Inactive';
}
~~~

### 10.3 Template: Save Extraction (Text state)
~~~javascript
const status = document.getElementById('status').textContent.trim();
~~~

---

## 11) Minimal Acceptance Checklist (AI Agent)

A Toggle Switch implementation is correct only if:

- HTML includes:
  - a checkbox input
  - a slider span
  - a visible label span storing the text state
- JavaScript includes:
  - init logic (preselect) based on a FileMaker-provided value
  - change logic updating the label text
- Save logic reads the label text and includes it in the parameter payload
- All ids used in JS exist in HTML and match exactly

---

# CauferoAppStarter Documentation
## Breadcrumbs and How They Are Implemented (WebViewer Pages)

> Purpose of this doc  
> Train an AI Agent to implement **Breadcrumbs** in CauferoAppStarter WebViewer pages using the provided sample (Tab 4).  
> Breadcrumbs show the user “where they are” in the app’s navigation hierarchy and provide quick jump links back to parent levels.

---

## 1) What Breadcrumbs Mean in CauferoAppStarter

Breadcrumbs are a compact navigation trail, usually placed near the top of a page or section, that:

- Shows the hierarchy of the current page (Home → Category → Subcategory → Current Page)
- Makes earlier levels clickable (optional but recommended)
- Keeps the last segment as a non-clickable “current” label

In this framework:
- The breadcrumb container uses `.breadcrumbs`
- Each clickable segment is an `<a>` element
- The final segment uses `<span class='current'>`

---

## 2) When to Use Breadcrumbs

Use breadcrumbs when:
- The user can arrive at the page through multiple paths
- The page is a “deep detail page” (Record Details, Staff Details, Item Details, KPI Details)
- The user needs an easy way to move back without relying only on Cancel or Back buttons
- The system uses multiple nested groupings (Modules → Lists → Records → Subrecords)

AI Agent rule:
- Add breadcrumbs to pages where the user needs to quickly orient and backtrack.
- Breadcrumbs are most valuable on detail pages and “drill-down” pages.

---

## 3) Breadcrumbs Anatomy (The Required HTML Pattern)

The sample implementation:

~~~html
<div class='breadcrumbs'>
  <a href='#'><i class='fas fa-home icon'></i>Home</a>
  <a href='#'><i class='fas fa-folder icon'></i>Category</a>
  <a href='#'><i class='fas fa-file icon'></i>Subcategory</a>
  <span class='current'>Current Page</span>
</div>
~~~

### 3.1 Required elements

- Outer container: `<div class='breadcrumbs'>`
- One or more intermediate links: `<a ...>Label</a>`
- Final current marker: `<span class='current'>Label</span>`

### 3.2 Icons

- Icons are Font Awesome classes and are optional.
- When used, the icon is inside the `<a>` tag.

Example:
~~~html
<i class='fas fa-home icon'></i>
~~~

AI Agent rule:
- If icons are used, standardize the icon class name `.icon` for spacing and styling.

---

## 4) Breadcrumb Data Model (What the AI Agent Must Build)

Breadcrumbs should be generated from a list of segments.

Each segment has:
- `label` (string shown to user)
- `action` (optional: what happens when clicked)
- `iconClass` (optional)
- `isCurrent` (true for the last segment)

Recommended internal representation (conceptual):
- Home (clickable)
- Parent page (clickable)
- List page (clickable)
- Current record (not clickable)

AI Agent rule:
- Breadcrumbs must always end with the current page label as a non-link.

---

## 5) How Breadcrumb Clicks Should Work in CauferoAppStarter

In a WebViewer page, `<a href='#'>` does nothing useful by itself.  
Breadcrumbs must trigger FileMaker navigation, typically through:

- `FileMaker.PerformScript('Some Script', 'param')`
- or `fmp://$/FileName?script=ScriptName&param=...` (less preferred if PerformScript is available)

### 5.1 Best practice: use onclick handlers calling FileMaker.PerformScript

Pattern:
~~~html
<a href='#' onclick="goBreadcrumb('HOME')">
  <i class='fas fa-home icon'></i>Home
</a>
~~~

Then define:
~~~javascript
function goBreadcrumb(target) {
  FileMaker.PerformScript('+++ Some Page Script', target);
}
~~~

AI Agent rule:
- Breadcrumb navigation must be explicit and deterministic.
- Do not rely on browser history inside FileMaker WebViewer.

### 5.2 Breadcrumb navigation scripts (FileMaker side)

Breadcrumbs normally route to page scripts such as:
- `+++ Records List Page`
- `+++ Staff List Page`
- `+++ Item Details Page`
- `+++ Module Home Page`

In the sample script, `$Cancel Script` is already set:
- `$Cancel Script = "+++ Records List Page"`

AI Agent rule:
- If a breadcrumb link navigates “back to list”, reuse the same page script used by Cancel when appropriate.
- If breadcrumbs are deeper than Cancel, define dedicated scripts per breadcrumb level.

---

## 6) Implementation Pattern (Step-by-step)

### Step 1: Decide breadcrumb hierarchy for the page

Example (Item Details):
- Home
- Inventory
- Items
- Item Details (current)

Example (Staff KPIs detail context):
- Home
- HR
- Staff
- Staff Details (current)

AI Agent rule:
- Breadcrumb labels must be user-friendly and match the app’s module vocabulary.

### Step 2: Set breadcrumb target scripts in FileMaker variables

Recommended variables:
- `$Breadcrumb Home Script`
- `$Breadcrumb Level1 Script`
- `$Breadcrumb Level2 Script`

Example:
~~~filemaker
Set Variable [ $Breadcrumb Home Script ; Value: "+++ Home Page" ]
Set Variable [ $Breadcrumb Level1 Script ; Value: "+++ Inventory Module" ]
Set Variable [ $Breadcrumb Level2 Script ; Value: "+++ Items List Page" ]
~~~

AI Agent rule:
- Keep breadcrumb routing script names in variables, same style as Save/Cancel/Open Modal patterns.

### Step 3: Generate Breadcrumb HTML in FileMaker

Recommended approach:
- Build a variable `$Breadcrumbs HTML`
- Inject it into the page where needed (top of tab content or top of content-section)

Example:
~~~filemaker
Set Variable [ $Breadcrumbs HTML ; Value:
"
<div class='breadcrumbs'>
  <a href='#' onclick=\"goHome()\"><i class='fas fa-home icon'></i>Home</a>
  <a href='#' onclick=\"goLevel1()\"><i class='fas fa-folder icon'></i>Inventory</a>
  <a href='#' onclick=\"goLevel2()\"><i class='fas fa-file icon'></i>Items</a>
  <span class='current'>Item Details</span>
</div>
"
]
~~~

Then place it inside a tab:
~~~filemaker
Set Variable [ $TabX HTML ; Value:
"
<div id='tabX' class='content' ...>

 " & $Breadcrumbs HTML & "

 ...rest of content...

</div>
"
]
~~~

AI Agent rule:
- Breadcrumbs should appear near the top of the tab/page content, before heavy UI objects.

### Step 4: Add JavaScript handlers for breadcrumb clicks

Example:
~~~javascript
function goHome() {
  FileMaker.PerformScript('+++ Home Page');
}

function goLevel1() {
  FileMaker.PerformScript('+++ Inventory Module');
}

function goLevel2() {
  FileMaker.PerformScript('+++ Items List Page');
}
~~~

If parameters are required:
~~~javascript
function goLevel2() {
  const param = 'some|pipe|params';
  FileMaker.PerformScript('+++ Items List Page', param);
}
~~~

AI Agent rule:
- Breadcrumb handlers must be included in `$Scripts` just like Cancel/Save/Wizard scripts.

---

## 7) CSS Expectations for Breadcrumbs

Breadcrumbs depend on `.breadcrumbs` and `.current` styling.

Minimum behavior:
- Breadcrumb trail is readable
- Links look clickable
- Current item stands out and is not a link
- Spacing between segments is consistent

Recommended CSS (if not already in `🖌️ Use Details CSS`):

~~~css
.breadcrumbs {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: center;
  font-size: 13px;
}

.breadcrumbs a {
  text-decoration: none;
}

.breadcrumbs a:hover {
  text-decoration: underline;
}

.breadcrumbs .icon {
  margin-right: 6px;
}

.breadcrumbs .current {
  font-weight: 600;
  opacity: 0.85;
}
~~~

Optional: add separators using CSS only (recommended):
~~~css
.breadcrumbs a::after {
  content: "›";
  margin-left: 10px;
  opacity: 0.5;
}

.breadcrumbs a:last-of-type::after {
  content: "›";
}
~~~

Alternative: build separators in HTML (less preferred):
~~~html
<a ...>Home</a>
<span class='sep'>›</span>
<a ...>Category</a>
<span class='sep'>›</span>
<span class='current'>Current</span>
~~~

AI Agent rule:
- Prefer CSS separators to avoid cluttering generated HTML.

---

## 8) Breadcrumb Placement Rules (In This Framework)

Breadcrumbs can be placed:
- Inside a specific tab (as in the sample Tab 4)
- At the top of the `content-section` so they remain visible across tabs
- Directly under the summary card, before tabs

Recommended placements:
1) For single-page flows: place under summary section
2) For tabbed pages: place inside each deep tab that needs context, or place once above tabs

AI Agent rule:
- If breadcrumbs represent overall page location (not tab location), place them once above the tabs.
- If breadcrumbs represent a special sub-context inside a tab, place them inside that tab.

---

## 9) Breadcrumbs vs Cancel Button (Policy)

Breadcrumbs are not the same as Cancel:

- Cancel is a form action (“leave without saving”)
- Breadcrumbs are navigation (“go back up the hierarchy”)

AI Agent rule:
- Do not replace Cancel with breadcrumbs.
- Breadcrumbs complement Cancel, they do not replace it.

---

## 10) Dynamic Breadcrumbs (Using Record Names)

Breadcrumbs often should include the record name:

Example:
- Home → Inventory → Items → Paracetamol

Implementation:
- FileMaker calculates the record label (name/title)
- Inject into the current segment

Example:
~~~filemaker
Set Variable [ $Current Label ; Value: $Item Name ]
Set Variable [ $Breadcrumbs HTML ; Value:
"
<div class='breadcrumbs'>
  <a href='#' onclick=\"goHome()\"><i class='fas fa-home icon'></i>Home</a>
  <a href='#' onclick=\"goLevel1()\"><i class='fas fa-folder icon'></i>Inventory</a>
  <a href='#' onclick=\"goLevel2()\"><i class='fas fa-file icon'></i>Items</a>
  <span class='current'>" & $Current Label & "</span>
</div>
"
]
~~~

AI Agent rule:
- If a page is “Record Details”, the current breadcrumb should be the record’s human label, not a generic “Details”.

---

## 11) Common Failure Modes and Fixes

### 11.1 Breadcrumb links do nothing
Cause:
- `<a href='#'>` has no onclick handler and no JS functions
Fix:
- Add onclick handlers and implement the functions in `$Scripts`

### 11.2 Breadcrumb navigation breaks parameters
Cause:
- scripts need parameters but breadcrumb handlers do not pass them
Fix:
- explicitly pass the required parameters, usually record IDs or tab targets

### 11.3 Breadcrumbs show wrong hierarchy
Cause:
- hardcoded labels reused across pages without updating
Fix:
- generate breadcrumb segments from page context (module, list, record)

### 11.4 Breadcrumbs clutter small screens
Cause:
- too many segments with long labels
Fix:
- keep labels short or hide middle segments on mobile via CSS

---

## 12) Required Clarifications (Ambiguities the AI Agent Cannot Guess)

To standardize breadcrumbs across your system, clarify:

1) Breadcrumb click behavior
   - Should breadcrumb links always call FileMaker scripts, or is browser history acceptable anywhere?

2) Standard breadcrumb levels
   - Do you always want: Home → Module → List → Record
   - Or do some modules add extra levels: Home → Module → Submodule → List → Record

3) Script naming conventions for breadcrumbs
   - What is your official Home page script name?
   - Do all module landing pages exist as scripts?

4) Breadcrumb persistence with tabs
   - When a page has tabs, should breadcrumbs reflect:
     - only the page hierarchy, or
     - the active tab too (rare but possible)

5) Icons standard
   - Do you want icons on every breadcrumb, or only the first one?

---

## 13) AI Agent Output Standard (What Must Be Produced)

When asked to add breadcrumbs to a page, the AI Agent must output:

1) FileMaker variables
   - breadcrumb target scripts
   - breadcrumb labels (module, list, record)
2) Breadcrumb HTML block (stored in a variable, injected into page)
3) JavaScript functions that call FileMaker scripts
4) CSS expectations for `.breadcrumbs`, `.current`, and `.icon`
5) Placement decision (where to inject breadcrumbs in the page)

---

# CauferoAppStarter Documentation
## Alert Boxes and How They Are Implemented (WebViewer Pages)

> Purpose of this doc  
> Train an AI Agent to implement **Alert Boxes** in CauferoAppStarter WebViewer pages, using the provided sample as the reference pattern.  
> In this codebase, the Alert Box is represented by a block like:
> - `<div id='alertBox' class='alert'> ... </div>`  
> and closed via:
> - `function closeAlert() { document.getElementById('alertBox').style.display = 'none'; }`

---

## 1) What an Alert Box Means Here (CauferoAppStarter Definition)

An **Alert Box** is an **inline, dismissible notification panel** that appears inside the page content.

It is:
- **visible inside the page layout** (not a browser alert dialog)
- **dismissible** by clicking an X icon
- **controlled by simple DOM show/hide**, not by reloading the page

In the sample, it appears inside Tab 4 and communicates a short notice:
- `"This application is being accessed on the server."`

Reference block from the sample:

~~~html
<div id='alertBox' class='alert'>
  <span class='alert-title'>Note:</span>This application is being accessed on the server.
  <i class='fas fa-times' onclick="closeAlert()"></i>
</div>
~~~

And the close handler:

~~~javascript
function closeAlert() {
  document.getElementById('alertBox').style.display = 'none';
}
~~~

---

## 2) When to Use Alert Boxes

Use an Alert Box when you need to display **short, important, time-relevant information** that:
- should be seen immediately
- is not a full-page error state
- should not block the user
- can be dismissed once understood

Examples:
- "You are currently on the server"
- "Offline mode is active"
- "Changes are not yet saved"
- "This record is locked by another user"
- "Readonly mode enabled"

AI Agent rule:
- If the message must interrupt the user, use a modal or a confirm flow instead.
- If the message is just helpful background, use an Info Panel instead.

---

## 3) Core Components of the Alert Box Pattern

An Alert Box implementation has 3 parts:

1. **HTML container**
2. **Close icon** (user action)
3. **JavaScript close function** (behavior)

### 3.1 HTML Container (Required)

Minimum required structure:

~~~html
<div id='alertBox' class='alert'>
  <span class='alert-title'>Note:</span> Alert message goes here.
  <i class='fas fa-times' onclick="closeAlert()"></i>
</div>
~~~

Required attributes:
- `id='alertBox'`  
- `class='alert'`

Required inner elements:
- A title label: `<span class='alert-title'>...</span>`
- A dismiss icon: `<i class='fas fa-times' onclick="closeAlert()"></i>`

AI Agent rules:
- The id must match what JavaScript expects (`alertBox`), unless you also update the JS to match.
- Keep content short (1 to 2 lines).
- Do not place form fields inside alert boxes.

---

## 4) JavaScript Behavior

### 4.1 Close Function (Required)

The sample uses a simple hide approach:

~~~javascript
function closeAlert() {
  document.getElementById('alertBox').style.display = 'none';
}
~~~

AI Agent rules:
- Use `style.display = 'none'` to close the alert.
- Do not remove the element from the DOM unless required.
- If multiple alerts are required, do not reuse a single fixed id (see Multi-Alert section).

---

## 5) CSS Contract (Styling Expectations)

The sample uses:

~~~filemaker
Perform Script [ “🖌️ Use Details CSS” ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]
~~~

So the classes below are assumed to be defined in the shared CSS:
- `.alert`
- `.alert-title`

AI Agent rules:
- Do not invent new class names for alert styling unless explicitly requested.
- If the alert does not look right, the fix is usually in the shared CSS script, not in random inline styles per page.

---

## 6) Placement Rules (Where to Put Alerts)

Alerts should be placed:
- near the top of the area they relate to
- inside the relevant tab or section container
- before the content the alert affects

Sample placement is inside Tab 4 content, directly after breadcrumbs:

~~~html
<!-- BREADCRUMBS -->
<div class='breadcrumbs'> ... </div>

<!-- ALERT BOX -->
<div id='alertBox' class='alert'> ... </div>
~~~

AI Agent rules:
- Place alert boxes above the UI that is impacted by the alert message.
- Avoid burying alert boxes at the bottom of long pages.

---

## 7) Dynamic Alert Content (Text Fields Only)

This documentation is for **text content** only.

### 7.1 Static alert message
Hardcode the message text into the HTML string:

~~~filemaker
Set Variable [ $AlertHTML ; Value:
"
<div id='alertBox' class='alert'>
  <span class='alert-title'>Note:</span>This application is being accessed on the server.
  <i class='fas fa-times' onclick=\"closeAlert()\"></i>
</div>
" ]
~~~

### 7.2 Dynamic alert message (FileMaker variables/fields)
Use FileMaker concatenation to inject the message:

~~~filemaker
Set Variable [ $AlertHTML ; Value:
"
<div id='alertBox' class='alert'>
  <span class='alert-title'>" & $AlertTitle & ":</span>" & $AlertMessage & "
  <i class='fas fa-times' onclick=\"closeAlert()\"></i>
</div>
" ]
~~~

AI Agent rules:
- If `$AlertMessage` comes from user-entered text, it should be HTML-escaped to avoid breaking markup.
- If no escaping utility exists, flag it as required (see Ambiguities).

---

## 8) Implementation Pattern Inside a Page Script

Alert Boxes follow the same pattern as other HTML objects:
1. Build the HTML chunk in a FileMaker variable
2. Insert the chunk into the page (tab HTML or summary/content section)
3. Ensure the JS function exists inside `$Scripts`

Example (Tab 4 pattern):

~~~filemaker
Set Variable [ $Tab4 HTML ; Value:
"
<div id='tab4' class='content'" & If ( $$Tab To Show = 4 ; " style='display: block;'" ) & ">

  <!-- ALERT BOX -->
  <div id='alertBox' class='alert'>
    <span class='alert-title'>Note:</span>This application is being accessed on the server.
    <i class='fas fa-times' onclick=\"closeAlert()\"></i>
  </div>

</div>
" ]
~~~

And the JavaScript must be included:

~~~filemaker
Set Variable [ $Close Alert Script ; Value:
"function closeAlert() {
  document.getElementById('alertBox').style.display = 'none';
}" ]
~~~

Then `$Close Alert Script` must be appended into the page script bundle:

~~~filemaker
Set Variable [ $Scripts ; Value:
"<script> " &
  ... &
  $Close Alert Script &
" </script> " ]
~~~

AI Agent rules:
- If you add the alert HTML, you must also ensure `closeAlert()` is present in the final JS bundle.
- If the alert is inside a tab that can be switched to, the function must be available globally (page-level script bundle is fine).

---

## 9) Multi-Alert Support (Optional Extension)

The sample implements one alert using a fixed id: `alertBox`.

If the page needs multiple alerts simultaneously, fixed ids will collide.

### 9.1 Multi-alert approach using unique ids
Use unique ids and pass the id into the close function:

~~~html
<div id='alertBox_server' class='alert'>
  <span class='alert-title'>Note:</span>Server access detected.
  <i class='fas fa-times' onclick="closeAlertById('alertBox_server')"></i>
</div>

<div id='alertBox_readonly' class='alert'>
  <span class='alert-title'>Warning:</span>Readonly mode enabled.
  <i class='fas fa-times' onclick="closeAlertById('alertBox_readonly')"></i>
</div>
~~~

~~~javascript
function closeAlertById(id) {
  const el = document.getElementById(id);
  if (el) el.style.display = 'none';
}
~~~

AI Agent rule:
- Only implement multi-alert support if required by the page spec.
- If not required, default to the single-alert pattern from the sample.

---

## 10) Quality Rules for Alert Text

Alert text must be:
- short
- specific
- actionable when needed

Guidelines:
- Title should be a single word or short label: `Note`, `Warning`, `Error`, `Info`
- Message should be one sentence or less when possible

AI Agent rule:
- If you need to explain a process, do not use an alert. Use an Info Panel.

---

## 11) Ambiguities and Clarifications Needed (Must Ask User)

The sample shows one alert pattern, but it does not specify some system contracts.

### 11.1 Are there multiple alert styles?
- Do you have variants like `alert-warning`, `alert-error`, `alert-success`?
- Or is everything `.alert` and the title changes only?

### 11.2 Is there a standard rule for when alerts should appear automatically?
- Should alerts appear based on FileMaker conditions (server vs local, permissions, record locked)?
- If yes, do you have a naming convention for alert variables like `$AlertTitle`, `$AlertMessage`, `$ShowAlert`?

### 11.3 Is there an HTML escaping utility in the codebase?
If dynamic text is used:
- What is the official escape/sanitize method to prevent HTML injection or broken markup?

AI Agent default behavior if not clarified:
- Implement a single `.alert` box with id `alertBox`
- Use `closeAlert()` with `style.display='none'`
- Keep content static or controlled by trusted variables only

---

## 12) Copy-Paste Templates (AI Agent Output Blocks)

### 12.1 Template: Basic Alert (Single)
~~~html
<div id='alertBox' class='alert'>
  <span class='alert-title'>Note:</span> Alert message goes here.
  <i class='fas fa-times' onclick="closeAlert()"></i>
</div>
~~~

~~~javascript
function closeAlert() {
  document.getElementById('alertBox').style.display = 'none';
}
~~~

### 12.2 Template: Multi-Alert (Optional)
~~~html
<div id='alertBox_1' class='alert'>
  <span class='alert-title'>Warning:</span> Message 1.
  <i class='fas fa-times' onclick="closeAlertById('alertBox_1')"></i>
</div>

<div id='alertBox_2' class='alert'>
  <span class='alert-title'>Info:</span> Message 2.
  <i class='fas fa-times' onclick="closeAlertById('alertBox_2')"></i>
</div>
~~~

~~~javascript
function closeAlertById(id) {
  const el = document.getElementById(id);
  if (el) el.style.display = 'none';
}
~~~

---

## 13) Minimal Acceptance Checklist (For the AI Agent)

An Alert Box implementation is correct only if:

- The HTML includes `class='alert'`
- The container has an id that matches the JavaScript logic
- The dismiss icon calls a close function
- The close function hides the alert with `style.display = 'none'`
- The alert is placed near the content it affects
- The styling is assumed to be provided by the shared CSS script

---

# CauferoAppStarter Documentation
## Accordions and How They Are Implemented and Used (SRD Tab)

> Purpose of this doc  
> Train an AI Agent to correctly implement **accordions** inside CauferoAppStarter WebViewer pages, using the SRD tab implementation shown in the sample script.  
> This doc focuses on the accordion container, headers, content blocks, toggle behavior, default open behavior, and how accordion content is assembled from FileMaker variables and subtables.

---

## 1) What an Accordion Means in CauferoAppStarter

An accordion is a UI pattern used to:
- group large content into collapsible sections
- reduce scrolling and visual noise
- allow focused reading and editing
- optionally open a specific section by default

In this framework:
- accordion sections are generated as HTML strings inside FileMaker
- each section has a header and a content block
- clicking the header toggles the next content block
- optional: a specific section can be auto-opened on page load using a FileMaker variable `$$Accordion Section`

---

## 2) Accordion Implementation Overview

The implementation is split into 3 layers:

1. **FileMaker Page Generator (HTML assembly)**
   - builds `$Tab5 HTML` (the SRD tab) which includes the accordion markup
   - inserts dynamic content (images, statements, paragraphs, subtable HTML, etc.)
   - decides which accordion section should open by default using `$$Accordion Section`

2. **CSS (accordion look and feel)**
   - comes from theme script `🖌️ Use Details CSS`
   - may be extended inside `$styles` with additional rules
   - accordion-specific classes must exist in the base theme CSS or be added if missing

3. **JavaScript (toggle logic + auto-open logic)**
   - `toggleAccordion(element)` toggles the next sibling content block
   - optional `DOMContentLoaded` handler opens a specific section by matching header text

---

## 3) Naming and Class Contracts (Must Be Followed)

The accordion relies on these class names:

- `.accordion`  
  Wrapper around all accordion sections

- `.accordion-item`  
  A single accordion section (header + content)

- `.accordion-header`  
  The clickable header element

- `.accordion-content`  
  The collapsible block. It must be the *immediate next sibling* of `.accordion-header`

### Why the “next sibling” rule matters
The toggle function uses:
`element.nextElementSibling`

So the HTML must be:
- header
- then content
- with nothing else between them

Correct:
~~~html
<div class="accordion-item">
  <div class="accordion-header" onclick="toggleAccordion(this)">Functional Requirements</div>
  <div class="accordion-content"> ... </div>
</div>
~~~

Incorrect:
~~~html
<div class="accordion-item">
  <div class="accordion-header" onclick="toggleAccordion(this)">Functional Requirements</div>
  <hr>
  <div class="accordion-content"> ... </div>
</div>
~~~
Because `nextElementSibling` would be the `<hr>`, not the content.

---

## 4) Accordion Markup Pattern (HTML)

The SRD tab uses an accordion to present sections like:
- Introduction
- Business Context
- System Overview
- Functional Requirements
- Non-Functional Requirements
- UI Guidelines
- Suggested Improvements
- Customization Options
- Phase Prioritization
- Future Enhancements
- Cost & Timelines
- Payment Terms

### Canonical HTML structure (repeat for each section)
~~~html
<div class="accordion">
  
  <div class="accordion-item">
    <div class="accordion-header" onclick="toggleAccordion(this)">
      Introduction
    </div>
    <div class="accordion-content" style="display:none;">
      <!-- content -->
    </div>
  </div>

  <div class="accordion-item">
    <div class="accordion-header" onclick="toggleAccordion(this)">
      Functional Requirements
    </div>
    <div class="accordion-content" style="display:none;">
      <!-- content -->
    </div>
  </div>

</div>
~~~

Notes:
- `style="display:none;"` may be set inline or via CSS default
- the sample JS toggles between `'block'` and `'none'`, so the “open” display type is `block`

---

## 5) JavaScript Toggle Logic

The sample script defines:

~~~js
function toggleAccordion(element) {
  const content = element.nextElementSibling;
  content.style.display = content.style.display === 'block' ? 'none' : 'block';
}
~~~

### Behavior contract
- input: the clicked `.accordion-header` element
- finds: its immediate next sibling element (must be `.accordion-content`)
- toggles: `display` between `'block'` and `'none'`

### Practical consequences
- do not rely on CSS transitions unless you rewrite the toggle logic
- if you want animations, you must implement a height-based transition pattern instead of simple display toggling

---

## 6) Default Open Section (Auto-Expand on Load)

The page supports opening a specific accordion section by using:
- FileMaker variable `$$Accordion Section`
- JavaScript that runs on `DOMContentLoaded`
- a text match against `.accordion-header` label

Sample blocks:

~~~js
window.addEventListener('DOMContentLoaded', function () {
  const headers = document.querySelectorAll('.accordion-header');
  headers.forEach(function (header) {
    if (header.textContent.trim() === 'Functional Requirements') {
      const content = header.nextElementSibling;
      if (content) {
        content.style.display = 'block';
      }
    }
  });
});
~~~

And similarly for:
- `Cost & Timelines`

### How FileMaker chooses which section opens
In the sample:
- the `DOMContentLoaded` block is appended conditionally based on:

- `If ( $$Accordion Section = "Functional Requirements" ; <open functional req section> )`
- `If ( $$Accordion Section = "Cost & Timelines" ; <open cost section> )`

### AI Agent rule
When implementing a new default-open section:
1. ensure the header’s visible text matches exactly the string being compared
2. append another `DOMContentLoaded` block for that section, or generalize the logic (recommended)

---

## 7) Recommended Improvement (Generalize Default Open Logic)

The current implementation repeats code for each section. The AI Agent may generate a general solution:

### General solution pattern
- store the desired header label in a JS variable injected from FileMaker
- open only that label

~~~js
window.addEventListener('DOMContentLoaded', function () {
  const target = '{{ACCORDION_SECTION}}'; /* injected from FileMaker */
  if (!target) return;

  const headers = document.querySelectorAll('.accordion-header');
  headers.forEach(function (header) {
    if (header.textContent.trim() === target) {
      const content = header.nextElementSibling;
      if (content) content.style.display = 'block';
    }
  });
});
~~~

FileMaker injects:
- `{{ACCORDION_SECTION}} = $$Accordion Section`

This eliminates the duplicated `If ( $$Accordion Section = ... )` blocks.

---

## 8) Accordion Content Assembly (FileMaker)

The accordion’s value is in how each `.accordion-content` block is generated from:
- ExecuteSQL results stored in variables (e.g., `$Introduction_Background`)
- default fallback values when no SRD record exists
- subtables generated by `+++ Display Subtable HTML` scripts

### Example content types used inside accordion sections
1. **Hero image**
   - uses URLs such as `$Cover Image URL`, `$Introduction_Image URL`, etc.
   - each URL is defaulted via `DefaultIfEmpty(...)`

2. **Section statement**
   - single-paragraph summary like `$Business Context_Statement`

3. **Bullet lists / paragraphs**
   - fields converted with `ConvertLineBreakMarkersToReturns(...)`

4. **Subtable HTML**
   - functional requirements are rendered using a generated subtable:
     - `$Functional Requirements Sub Table HTML`
   - cost items rendered similarly:
     - `$Price Breakdown Sub Table HTML`

### AI Agent rule
Accordion content must be built as HTML using already-available helper patterns:
- image blocks
- paragraph blocks
- subtable blocks
- button blocks (copy to clipboard, generate prompt, etc.)

Do not place raw FileMaker returns inside HTML without conversion if the content is intended to preserve line breaks.

---

## 9) Accordion Section Labels Must Be Stable

Because the default-open logic matches header text exactly, labels like:
- `Functional Requirements`
- `Cost & Timelines`

must remain stable.

### AI Agent rule
If a label changes, you must also update:
- the default-open matching string
- any routing logic that sets `$$Accordion Section`

---

## 10) Accordion and Tabs Relationship

The accordion is inside a tabbed page structure.

Relevant pattern:
- `$Tabs HTML` defines tabs
- `$Tab5 HTML` contains the SRD tab content
- the accordion exists inside `$Tab5 HTML`

JavaScript:
- `showContent(contentId, tabElement)` hides all `.content` blocks and shows the requested one

### AI Agent rule
If you want an accordion section to open by default, ensure:
1. Tab 5 is visible (either active by default or selected)
2. Accordion auto-open runs after DOM load
3. The accordion markup exists inside the loaded DOM

---

## 11) Minimum CSS Requirements for Accordion

The sample shows no explicit accordion CSS added in `$styles`. That means accordion styling is likely included in:
- `🖌️ Use Details CSS`

### AI Agent rule
If accordion CSS is missing in a theme, add at least:

~~~css
.accordion { width: 100%; }
.accordion-item { border-radius: 10px; margin-bottom: 10px; overflow: hidden; }
.accordion-header { cursor: pointer; padding: 12px 14px; font-weight: 600; }
.accordion-content { display: none; padding: 12px 14px; }
~~~

Do not over-style in the page script unless the framework expects local overrides.

---

## 12) Copy-Paste Accordion Template (Framework-Compatible)

This template is what the AI Agent should generate inside a `$TabX HTML` calculation.

~~~text
<div class="accordion">

  <div class="accordion-item">
    <div class="accordion-header" onclick="toggleAccordion(this)">Introduction</div>
    <div class="accordion-content" style="display:none;">
      
      <div class="section-image">
        <img src="{{Introduction_Image URL}}" alt="">
      </div>

      <p class="section-statement">{{Introduction_Statement}}</p>

      <div class="section-body">
        {{Introduction_Background}}
      </div>

    </div>
  </div>

  <div class="accordion-item">
    <div class="accordion-header" onclick="toggleAccordion(this)">Functional Requirements</div>
    <div class="accordion-content" style="display:none;">
      {{Functional Requirements Sub Table HTML}}
    </div>
  </div>

</div>
~~~

Replace placeholders with FileMaker variables concatenated into the HTML string.

---

## 13) JS Injection Pattern (How Accordion JS Gets Into the Page)

The accordion JS is injected via `$Scripts`, which is assembled as:

~~~filemaker
Set Variable [ $Scripts ;
  Value:
  "<script> " &
  $Show Content Script & "¶¶" &
  $Cancel Action Script & "¶¶" &
  $Toggle Accordion Script &
  " </script> "
]
~~~

### AI Agent rule
If you add an accordion to a page, you must ensure:
- `$Toggle Accordion Script` is included in `$Scripts`
- the HTML uses `onclick="toggleAccordion(this)"`

---

## 14) Common Failure Modes and Fixes

### 14.1 Clicking header does nothing
Cause:
- `toggleAccordion` not injected into `$Scripts`
Fix:
- ensure `$Toggle Accordion Script` is in the `$Scripts` bundle

### 14.2 Wrong content opens or nothing opens
Cause:
- header’s `nextElementSibling` is not `.accordion-content`
Fix:
- remove any elements between header and content
- keep strict header then content ordering

### 14.3 Default open does not work
Cause:
- header text mismatch (extra spaces, different punctuation)
Fix:
- ensure exact label match
- `header.textContent.trim()` is compared to the exact string

### 14.4 Default open works but user sees different tab
Cause:
- Tab 5 not active
Fix:
- set `$$Tab To Show = 5` when routing to SRD tab
- ensure `showContent('tab5', ...)` is invoked if needed

---

## 15) Required Clarifications (Ambiguities the AI Agent Cannot Guess)

These affect whether the AI Agent can implement accordions perfectly:

1. **Accordion CSS source of truth**
   - Is `.accordion` styling always guaranteed in `🖌️ Use Details CSS` for every theme?
   - If not, should the AI Agent inject a minimal accordion CSS block whenever missing?

2. **Open behavior for multiple sections**
   - Should multiple sections be able to stay open at the same time?
   - Or should opening one section close others automatically (classic accordion)?

   Current logic allows multiple open sections.

3. **Default open routing**
   - Where is `$$Accordion Section` set from?
   - Is it passed as a script parameter, or set by a calling script before page generation?

4. **Display mode**
   - Must accordion content open with `display: block` only?
   - Or do some sections need `display: flex` or `grid` when open?

5. **Section labels localization**
   - Are section titles always English strings?
   - Are they ever dynamic or pulled from table fields?

---

## 16) AI Agent Output Standard (What Must Be Produced)

When asked to add an accordion section to a CauferoAppStarter page, the AI Agent must output:

1. HTML:
   - `.accordion` wrapper
   - `.accordion-item` per section
   - `.accordion-header` with `onclick="toggleAccordion(this)"`
   - `.accordion-content` immediately after the header

2. JavaScript:
   - `toggleAccordion(element)` function injected into `$Scripts`
   - optional auto-open on `DOMContentLoaded` using `$$Accordion Section`

3. FileMaker assembly:
   - content variables prepared (from ExecuteSQL or defaults)
   - any subtable HTML already generated and inserted into the relevant accordion content

4. Consistency:
   - stable header labels if used for default open
   - no DOM structure between header and content\

---

# CauferoAppStarter Documentation
## Maps and How They Are Implemented and Used (Google Maps Embed)

> Purpose of this doc  
> Train an AI Agent to implement **maps** in CauferoAppStarter WebViewer pages using the sample Vendor Details page.  
> The map is implemented as a **Google Maps embed iframe** driven by a FileMaker text field (Location).  
> This doc covers the HTML structure, data preparation, URL formatting, CSS container rules, tab placement, and failure modes.

---

## 1) What “Map” Means in CauferoAppStarter

A map is a UI section that shows the physical location of an entity (Vendor, Business, Customer, Site, Warehouse, etc.).

In this framework, the map is:
- not an interactive JavaScript map SDK
- not a Google Maps API key integration
- simply an **iframe embed** pointing to Google Maps with a query string built from FileMaker data

This gives a fast implementation with:
- zero JS dependencies
- minimal maintenance
- predictable behavior in WebViewer

---

## 2) Where the Map Lives in the Page

In the sample, the page is tabbed:
- Tab 1: Vendor Details
- Tab 2: Supplies Catalog
- Tab 3: Transaction History

The map is placed inside **Tab 1** (Vendor Details), after the form fields and a divider:

~~~text
Tab 1 content
  - text fields + dropdown + segmented control + toggle
  - horizontal rule
  - MAP (iframe embed)
~~~

### AI Agent rule
If implementing a map:
- put it in the relevant tab (usually the “Details” tab)
- keep it near other address/location fields
- ensure it is inside the correct `.content` block so it shows only when that tab is active

---

## 3) Core Implementation Pattern (HTML)

### Map container + iframe (as used in sample)
The sample map block is:

~~~html
<!-- MAP (START) -->
<div class='map-container' style='width: 100%; height: 40vh; padding: 0px 20px'>
  <iframe
    src='https://local.google.com/maps?q={{LOCATION_QUERY}}&output=embed'
    allowfullscreen
    loading='lazy'>
  </iframe>
</div>
<!-- MAP (END) -->
~~~

Where `{{LOCATION_QUERY}}` is built from FileMaker:

- `Substitute ( $Location ; ", " ; "+" )`

So the actual generation used is:

~~~filemaker
"<iframe src='https://local.google.com/maps?q=" &
Substitute ( $Location ; ", " ; "+" ) &
"&output=embed' allowfullscreen loading='lazy'> </iframe>"
~~~

### AI Agent rule
The map must be implemented using an `<iframe>` with:
- a `src` that includes a `q=` query parameter
- `output=embed`
- `loading='lazy'` for performance

---

## 4) Data Source and Preparation (FileMaker)

### Source field
The map is driven from:

- `$Location` (Vendor Location) retrieved via ExecuteSQL:

~~~filemaker
sql = "Select \"Name\", \"Location\", ... From \"Vendors\" Where ID = ?"
~~~

### Formatting rule used in the sample
The sample uses:

- `Substitute ( $Location ; ", " ; "+" )`

This converts typical address formatting:
- `"Accra, Greater Accra Region"`  
into:
- `"Accra+Greater Accra Region"`

### AI Agent rule
The map query must be derived from a readable location string:
- city, region, country
- or an address line
- or a landmark name

---

## 5) Recommended Encoding Rules (Stronger Than Sample)

The sample only replaces `", "` with `"+"`. That works for many cases, but it is not a full URL encoding strategy.

### Recommended minimal encoding (safe baseline)
At minimum, replace:
- space with `+`

Example pattern:

~~~filemaker
Set Variable [ $Location Query ;
  Value: Substitute ( $Location ;
    [ " " ; "+" ]
    [ "¶" ; "+" ]
  )
]
~~~

### Recommended stronger encoding (if your data has symbols)
If locations may include `&`, `#`, or `?`, you must sanitize because it can break the URL.

Suggested replacement set:

~~~filemaker
Set Variable [ $Location Query ;
  Value: Substitute ( $Location ;
    [ " " ; "+" ] ;
    [ "¶" ; "+" ] ;
    [ "&" ; "" ] ;
    [ "#" ; "" ] ;
    [ "?" ; "" ] ;
    [ "\"" ; "" ] ;
    [ "'" ; "" ]
  )
]
~~~

### AI Agent rule
If your location text can contain URL-breaking characters:
- sanitize them before concatenating into the iframe `src`

---

## 6) CSS Contract for Map Rendering

The sample uses an inline style on the container:

- `width: 100%`
- `height: 40vh`
- `padding: 0px 20px`

This ensures the iframe gets a visible height.

### Required CSS / inline rules
The iframe must have a height. If not set, the map can render as a thin line.

Recommended baseline styling:

~~~css
.map-container iframe {
  width: 100%;
  height: 100%;
  border: 0;
  border-radius: 12px;
}
~~~

### AI Agent rule
A map container must define height, either by:
- inline style `height: ...`
- or CSS class with a height

Then the iframe must fill:
- `width: 100%`
- `height: 100%`

---

## 7) Implementation Steps (AI Agent Checklist)

When adding a map to a page:

1. **Ensure the page has a location variable**
   - `$Location` or `$Address` is loaded from ExecuteSQL or set to defaults

2. **Create a query-friendly string**
   - `$Location Query` derived from `$Location` using Substitute rules

3. **Insert a map container into the correct tab**
   - typically after location fields and before the tab ends

4. **Generate iframe src**
   - `https://local.google.com/maps?q={{Location Query}}&output=embed`

5. **Ensure container height**
   - inline `height: 40vh` or CSS `min-height`

6. **Add iframe styling**
   - width/height 100%, border 0

7. **Handle empty locations gracefully**
   - show a message if no location is available (see section 9)

---

## 8) Copy-Paste Map Template (Framework-Compatible)

Use this block inside a `$TabX HTML` calculation:

~~~text
<!-- MAP (START) -->
<div class='map-container' style='width: 100%; height: 40vh; padding: 0px 20px'>
  <iframe
    src='https://local.google.com/maps?q={{LOCATION_QUERY}}&output=embed'
    allowfullscreen
    loading='lazy'>
  </iframe>
</div>
<!-- MAP (END) -->
~~~

And build `{{LOCATION_QUERY}}` from FileMaker:

~~~filemaker
Set Variable [ $Location Query ;
  Value: Substitute ( $Location ; [ " " ; "+" ] [ ", " ; "+" ] [ "¶" ; "+" ] )
]
~~~

Then inject:

~~~filemaker
"<iframe src='https://local.google.com/maps?q=" & $Location Query & "&output=embed' allowfullscreen loading='lazy'></iframe>"
~~~

---

## 9) Empty Location Handling (Must Not Show a Broken Map)

If `$Location` is empty, the embed may show:
- a random world view
- a blank map
- an error inside the iframe

### Recommended behavior
If location is empty:
- display a friendly message instead of an iframe

Template:

~~~text
<div class='map-container' style='width: 100%; height: 40vh; padding: 0px 20px'>
  {{MAP_HTML}}
</div>
~~~

Where `{{MAP_HTML}}` becomes either:
- iframe embed when `$Location` is not empty
- a placeholder card when `$Location` is empty

Example placeholder:

~~~html
<div class="no-record-card">
  <div class="no-record-title">No Location Provided</div>
  <div class="no-record-body">Add a location to view this vendor on the map.</div>
</div>
~~~

### AI Agent rule
Never embed a map with an empty `q=` query. Use a placeholder UI.

---

## 10) Security and WebViewer Constraints

This implementation:
- does not require API keys
- does not require external JS
- works by embedding a third-party site

Potential constraints:
- some environments may restrict iframe access to Google domains
- some WebDirect deployments may enforce content security rules

### AI Agent rule
If iframe embeds are blocked in a target environment, you must:
- switch to a different embed domain policy
- or open maps externally via a clickable link instead of iframe

Fallback link pattern:

~~~html
<a href="https://www.google.com/maps/search/?api=1&query={{LOCATION_QUERY}}" target="_blank">
  Open in Google Maps
</a>
~~~

---

## 11) Performance and UX Rules

Recommended options used in the sample:
- `loading='lazy'` reduces initial load cost
- map is placed lower on the page after form fields
- uses a moderate height (40vh)

### AI Agent rule
- always use `loading='lazy'`
- do not place a heavy map at the very top unless the page is a “Location Page”
- keep the height between `30vh` and `60vh` depending on the page density

---

## 12) Common Failure Modes and Fixes

### 12.1 Map shows but is tiny / invisible
Cause:
- container has no height
Fix:
- set container height and iframe height to 100%

### 12.2 Map query is wrong or too broad
Cause:
- location string is incomplete
Fix:
- include city + region + country, or add a more specific address

### 12.3 Map breaks when location has special characters
Cause:
- URL not sanitized
Fix:
- remove or encode `&`, `#`, `?`, quotes

### 12.4 Map does not load in WebDirect
Cause:
- iframe blocked by environment policy
Fix:
- use “Open in Google Maps” link fallback
- or whitelist the domain in deployment policy where possible

---

## 13) Required Clarifications (Ambiguities the AI Agent Cannot Guess)

These items affect “perfect” map generation:

1. **Canonical location format**
   - What is the expected structure of `$Location` in your system?
   - Examples: `"East Legon, Accra"`, `"P. O. Box ..."`, `"GPS Address ..."`, `"Street + City + Country"`

2. **Preferred map provider**
   - Must it always be Google Maps embed, or do you also support OpenStreetMap?

3. **Offline / low-connectivity fallback**
   - If map cannot load, should we show:
     - a link only
     - a static image (screenshot map)
     - or a simple text-only panel?

4. **Multiple locations**
   - If an entity has multiple addresses, which one should the map display?
   - Primary address only, or selectable?

5. **Exact CSS contract**
   - Is `.map-container` styled inside `🖌️ Use Details CSS` in all themes?
   - If not, should the AI Agent always inject map CSS locally?

---

## 14) AI Agent Output Standard (What Must Be Produced)

When asked to implement a map in CauferoAppStarter, the AI Agent must produce:

1. FileMaker variables:
   - `$Location` loaded (or defaulted)
   - `$Location Query` derived by sanitizing `$Location`

2. HTML block:
   - `.map-container` with a defined height
   - iframe embed with `q=` and `output=embed`
   - placeholder UI if location is empty

3. CSS (if needed):
   - iframe fill rules: width/height 100%, border 0, optional radius

4. Optional fallback:
   - “Open in Google Maps” link when iframe is blocked

---

# CauferoAppStarter Documentation
## Transfer Lists and How They Are Implemented and Used (Two-List Assignment UI)

> Purpose of this doc  
> Train an AI Agent to implement **transfer lists** (move items from one list to another) in CauferoAppStarter WebViewer pages using the Role Details sample.  
> A transfer list is used to assign or unassign many items quickly (example: assign menu links to a role).  
> This doc covers the data contract, HTML structure, required attributes, move logic, sorting, and how to save back to FileMaker.

---

## 1) What “Transfer List” Means in CauferoAppStarter

A transfer list is a UI component that displays:

- **List A (Unassigned / Available items)**
- **List B (Assigned / Selected items)**

Each row is an item (usually an `<li>`), and the user can move an item between the two lists.

Typical use cases:
- Assign menu links to a role
- Assign permissions to a role
- Assign tags to a record
- Assign users to a group
- Assign features/modules to a subscription

In this framework, the transfer list is implemented with:
- plain HTML `<ul>` lists
- `<li>` rows generated by FileMaker scripts (value list scripts)
- a JavaScript `moveItem()` function
- a save function that collects item IDs from the “assigned” list and calls a FileMaker script

---

## 2) Where the Transfer List Lives in the Page

In the sample:
- The page has a Summary Section (Save/Cancel)
- The transfer list lives inside Tab1 HTML after the basic fields

Placement pattern:
- Header text (e.g., "Menu Assignments")
- `.cards_row` container
- two `.note-card` cards
- each card has a header and a `<ul>` list

AI Agent rule:
- Put transfer lists inside the most relevant “Details” tab, below the identifier fields (Role Name) so the user knows what entity they are editing.

---

## 3) Data Contracts (What FileMaker Must Provide)

The transfer list requires two sets of list items:

1) **All Items Not Assigned** (List A)
2) **Items Assigned** (List B)

In the sample these are produced by `Get Value List` scripts:

~~~filemaker
Perform Script [ “Get Value List” ; Parameter: "Table In First Card|Links|" & $$Role ID ]
Set Variable [ $All Links ; Value: Get ( ScriptResult ) ]

Perform Script [ “Get Value List” ; Parameter: "Table In Second Card|Selected Role's Links|" & $$Role ID ]
Set Variable [ $Role's Links ; Value: Get ( ScriptResult ) ]
~~~

AI Agent rule:
- The transfer list does not fetch data in JS.
- FileMaker must inject HTML list rows into `$All Links` and `$Role's Links`.

---

## 4) Required HTML Structure

### 4.1 Outer layout (two cards side by side)

The sample pattern:

~~~html
<div class='cards_row'>

  <!-- Card 1 -->
  <div class='note-card'>
    <div class='note-header'>Menu Links Not Assigned To {{ROLE_NAME}}</div>
    <div class='note-content'>
      <ul id='list1'>
        {{ALL_LINKS_LI_HTML}}
      </ul>
    </div>
  </div>

  <!-- Card 2 -->
  <div class='note-card'>
    <div class='note-header'>Menu Links Assigned To {{ROLE_NAME}}</div>
    <div class='note-content'>
      <ul id='list2'>
        {{ASSIGNED_LINKS_LI_HTML}}
      </ul>
    </div>
  </div>

</div>
~~~

### 4.2 Each item row must be an `<li>` with attributes

The move logic and saving depend on `<li>` attributes.

Minimum required attributes per `<li>`:
- `data-id` : the record ID to save
- `data-order` : numeric order used for sorting when moved

Example `<li>` template (recommended):
~~~html
<li data-id="{{ID}}" data-order="{{ORDER}}">
  <span class="item-label">{{LABEL}}</span>
  <button class="transfer-btn" onclick="moveItem(this, 'list2', 'right')">
    <i class="fas fa-arrow-right"></i>
  </button>
</li>
~~~

For an item already in the assigned list, the button should point left:

~~~html
<li data-id="{{ID}}" data-order="{{ORDER}}">
  <span class="item-label">{{LABEL}}</span>
  <button class="transfer-btn" onclick="moveItem(this, 'list1', 'left')">
    <i class="fas fa-arrow-left"></i>
  </button>
</li>
~~~

AI Agent rule:
- Each list item MUST include `data-id`.
- Each list item MUST include `data-order` (or sorting will be unpredictable).
- Each list item MUST include a move button that calls `moveItem(button, targetListId, direction)`.

---

## 5) The Move Logic (JavaScript)

The transfer list is powered by the `moveItem()` function in the sample.

### 5.1 Move item between lists and flip arrow direction

Sample function behavior:
- Identify the `<li>` via `button.parentElement`
- Append the `<li>` to the target list
- Change the arrow icon to match direction
- Update onclick to reverse direction next time
- Sort the destination list by `data-order`

Reference logic (as used):

~~~javascript
function moveItem(button, targetListId, direction) {
    const item = button.parentElement;
    const targetList = document.getElementById(targetListId);
    const iconClass = direction === 'left' ? 'fa-arrow-left' : 'fa-arrow-right';

    button.innerHTML = `<i class='fas ${iconClass}'></i>`;
    button.onclick = () => moveItem(
        button,
        targetListId === 'list1' ? 'list2' : 'list1',
        direction === 'left' ? 'right' : 'left'
    );

    targetList.appendChild(item);

    /* Sort the target list by the order field */
    sortList(targetList);
}
~~~

### 5.2 Sort list after moving

Sorting uses `data-order`:

~~~javascript
function sortList(list) {
    const items = Array.from(list.children);

    items.sort((a, b) => {
        const orderA = parseInt(a.getAttribute('data-order'), 10);
        const orderB = parseInt(b.getAttribute('data-order'), 10);
        return orderA - orderB;
    });

    items.forEach(item => list.appendChild(item));
}
~~~

AI Agent rule:
- Always re-sort after move to keep a stable predictable UI.
- Sorting depends on numeric `data-order` values. Ensure your FileMaker generated `<li>` rows include valid numeric order.

---

## 6) Saving the Assigned Items Back to FileMaker

The save process must collect IDs from the assigned list (List B).

In sample:
- `list2` is the assigned list
- it collects `data-id` from each `<li>` in `#list2`
- then passes `role` and `list2Ids` to a FileMaker script

Sample save logic:

~~~javascript
function saveItemInfo() {

    const role = document.getElementById('role').value;

    const list2Ids = Array.from(document.querySelectorAll('#list2 li'))
      .map(item => item.getAttribute('data-id'));

    const parameters = [ role, list2Ids ].join('|');

    FileMaker.PerformScript('Save Role', parameters);
}
~~~

### Important parameter formatting rule
`list2Ids` is a JavaScript array. When coerced to string, it becomes a comma-separated list:

Example:
~~~text
["12","55","88"] -> "12,55,88"
~~~

AI Agent rule:
- Your FileMaker save script must expect the assigned IDs as a comma-separated string OR you must explicitly join with a delimiter you control.

Recommended explicit join (safer):
~~~javascript
const list2Ids = Array.from(document.querySelectorAll('#list2 li'))
  .map(item => item.getAttribute('data-id'))
  .join(',');
~~~

Then:
~~~javascript
const parameters = [ role, list2Ids ].join('|');
~~~

---

## 7) CSS Expectations

The transfer list layout relies on these structural classes:
- `.cards_row` for the two-card row layout
- `.note-card` for each list card
- `.note-header` and `.note-content` for card structure

Additionally, transfer list item styling should support:
- horizontal row alignment inside `<li>`
- right-aligned move button
- clear hover states

Recommended CSS (if not already in `🖌️ Use Details CSS`):

~~~css
.cards_row { display: flex; gap: 20px; align-items: flex-start; }
.note-card { flex: 1; }
.note-content ul { list-style: none; padding: 0; margin: 0; }
.note-content li { display: flex; align-items: center; justify-content: space-between; padding: 10px; border-bottom: 1px solid rgba(0,0,0,0.08); }
.transfer-btn { border: 0; background: transparent; cursor: pointer; }
~~~

AI Agent rule:
- If the global theme CSS does not style `ul/li` rows cleanly, add local CSS for list readability and button alignment.

---

## 8) Build Pattern (FileMaker + WebViewer)

### Step-by-step build contract
To implement a transfer list in a new page, the AI Agent must do:

1) Decide the entity and assignment relationship
   - Example: Role to Links
   - Example: User to Roles
   - Example: Item to Categories

2) Create the two list datasets in FileMaker
   - Available items (not assigned)
   - Assigned items (already assigned)

3) Use `Get Value List` scripts to return `<li>` HTML for each list
   - Each `<li>` must include:
     - `data-id`
     - `data-order`
     - button calling `moveItem(...)`

4) Inject those HTML strings into:
   - `<ul id='list1'>...</ul>`
   - `<ul id='list2'>...</ul>`

5) Include `moveItem()` and `sortList()` JS
6) Implement `saveItemInfo()` to collect IDs from list2 and call a FileMaker save script

---

## 9) Empty List Handling

When one list is empty:
- It should still render as an empty `<ul>`
- Optionally show a “No items” placeholder row

Recommended placeholder `<li>`:
~~~html
<li class="empty-state" data-order="999999" data-id="">
  <span>No items found</span>
</li>
~~~

AI Agent rule:
- Do not break JS by inserting rows without `data-order` if your sort function assumes it exists.
- If you add placeholders, exclude them from saving by ensuring `data-id` is blank and filtering them out in JS.

Example filter:
~~~javascript
const list2Ids = Array.from(document.querySelectorAll('#list2 li'))
  .map(li => li.getAttribute('data-id'))
  .filter(id => id && id.trim().length);
~~~

---

## 10) Common Failure Modes and Fixes

### 10.1 Buttons stop working after move
Cause:
- the button is not inside the `<li>` as direct parent-child
Fix:
- enforce `<button>` as direct child of `<li>` so `button.parentElement` is correct

### 10.2 Sorting becomes random or throws NaN
Cause:
- missing `data-order` or non-numeric value
Fix:
- ensure every `<li>` has a numeric `data-order`

### 10.3 Save script receives weird formatting
Cause:
- passing JS array directly without join
Fix:
- always `.join(',')` in JS, and parse in FileMaker

### 10.4 Duplicate items appear in both lists
Cause:
- FileMaker list generation includes the same item in both outputs
Fix:
- enforce clean separation in SQL or logic used to build the two lists

---

## 11) Required Clarifications (Ambiguities the AI Agent Cannot Guess)

These must be clarified for a perfect, consistent transfer list generator:

1) **Exact `<li>` row HTML returned by `Get Value List`**
   - Does it already include the move button HTML?
   - If yes: what is the exact onclick signature and direction convention?

2) **Sorting rules**
   - Is `data-order` derived from:
     - menu position order
     - alphabetical order
     - table field `Order`
   - Should the UI always preserve the original system order?

3) **Save script parameter contract**
   - Does `Save Role` expect:
     - comma-separated IDs
     - return-delimited IDs
     - JSON array
   - If return-delimited is preferred, the JS must join with `'\n'` instead of comma.

4) **Uniqueness and duplicates**
   - Are links unique by ID always, or can “same name different link” exist?
   - If duplicates can exist, the UI label must include a unique descriptor.

5) **Multi-card layout styling**
   - Is `.cards_row` always available in theme CSS?
   - If not, should the AI Agent inject a standard cards_row CSS block every time?

---

## 12) AI Agent Output Standard (What Must Be Produced)

When asked to implement a transfer list, the AI Agent must output:

1) FileMaker steps:
   - Variables holding HTML list rows for List A and List B
   - Each list row must carry `data-id` and `data-order`

2) HTML:
   - Two `<ul>` elements with stable IDs (example: list1, list2)
   - Two card containers, consistent with the system UI patterns

3) JavaScript:
   - `moveItem(button, targetListId, direction)`
   - `sortList(list)`
   - `saveItemInfo()` collecting assigned IDs and calling `FileMaker.PerformScript(...)`

4) Save contract:
   - Document what delimiter is used for IDs (comma, return, JSON)

---

# CauferoAppStarter Documentation
## Wizards and How They Are Implemented (Stepper + Step Content + Next/Prev Logic)

> Purpose of this doc  
> Train an AI Agent to implement a **Wizard** (multi-step flow) in CauferoAppStarter WebViewer pages using the "My Appraisal Details" sample.  
> A wizard is used when one “page” is too dense and must be broken into steps, with a stepper UI at the top and Next/Previous navigation at the bottom.  
> This doc covers: data prep, HTML structure, CSS expectations, JS state machine, last-step actions, and FileMaker script contracts.

---

## 1) What “Wizard” Means in CauferoAppStarter

A wizard is a structured multi-step UI that:

- Shows a **Stepper** (steps 1..N) so users know where they are
- Shows **only one step’s content at a time**
- Provides **Previous** and **Next** buttons
- On the last step, **Next becomes Submit/OK** and triggers a final action (save, submit, close, cancel)

In this framework:
- Steps are `.step` elements
- Step content sections are `.step-wizard_content`
- JavaScript manages `currentStep` and toggles the `active` class

---

## 2) When to Use a Wizard

Use a wizard when:
- The user must complete a process in a strict sequence
- There are too many fields or sections to present at once
- You need to reduce cognitive load and improve completion rates
- The final step needs confirmation and a final action

Examples:
- Appraisal form: Questions → KPIs → Remarks → Submit
- New employee onboarding: Bio → Permissions → Review → Create
- Purchase request: Vendor → Items → Approvals → Submit
- Stock transfer: Source → Items → Destination → Confirm

AI Agent rule:
- Use a wizard for “completion workflows” where it is helpful to guide users through a linear path.

---

## 3) Wizard Component Anatomy (High Level)

A wizard has 3 major blocks:

1) **Stepper UI**
   - visual steps with numbers/labels
2) **Wizard Content Area**
   - multiple step panels, only one visible at a time
3) **Wizard Navigation Buttons**
   - Previous and Next, with Next label changing on last step

In the sample, these blocks are wrapped in:

~~~html
<div class='wizard-container'>
  <div class='stepper'> ...steps... </div>
  <div class='wizard_content'> ...step-wizard_content... </div>
  <div class='wizard_buttons'> ...prev/next... </div>
</div>
~~~

---

## 4) FileMaker Data Preparation Requirements

A wizard page usually depends on significant data prep before HTML generation.

In the sample, the wizard depends on:
- appraisal header fields (period name, status, totals)
- hierarchical questions HTML (`$Questions HTML`)
- score headers HTML (`$Scores HTML`)
- KPI subtable HTML (`$Sub Table HTML _KPIs`)
- preloaded answers JS (`$Answers JS`)
- status flags (e.g., `$$Status`, `$$Disabled`, `$Total Questions`)

AI Agent rule:
- Prepare all wizard-dependent HTML/JS fragments in FileMaker variables before building `$Tab1 HTML`.
- Never try to fetch these in the browser; WebViewer page should be fully assembled.

---

## 5) Required HTML Structure

### 5.1 Wizard container

Recommended base:

~~~html
<div class='wizard-container' style='max-height: 105%;'>
  ...
</div>
~~~

### 5.2 Stepper (must match number of content steps)

Sample stepper:

~~~html
<div class='stepper'>

  <div class='step active'>
    <div class='icon'>1</div>
    <p>Start</p>
    <div class='line'></div>
  </div>

  <div class='step'>
    <div class='icon'>2</div>
    <p>KPIs</p>
    <div class='line'></div>
  </div>

  <div class='step'>
    <div class='icon'>3</div>
    <p>Remarks</p>
  </div>

</div>
~~~

Key rules:
- Each `.step` represents one wizard stage.
- The first `.step` starts as `active` (optional; JS will correct it).
- A `.line` element is used for the visual connector between steps (optional on last step).
- Labels must match the content intention.

AI Agent rule:
- The number of `.step` elements MUST equal the number of `.step-wizard_content` elements.

### 5.3 Wizard content panels

Sample content container:

~~~html
<div class='wizard_content'>

  <div class='step-wizard_content active'>
    ... Step 1 content ...
  </div>

  <div class='step-wizard_content'>
    ... Step 2 content ...
  </div>

  <div class='step-wizard_content'>
    ... Step 3 content ...
  </div>

</div>
~~~

Key rules:
- Only one panel should have `active` at a time.
- The first panel starts as `active`.

AI Agent rule:
- Each step panel must map to a stepper item by index.
- Each step panel should include:
  - a short title `<h3>`
  - a short instruction `<p>`
  - the actual UI objects (tables, inputs, subtables)

### 5.4 Wizard navigation buttons (Prev/Next)

Sample buttons:

~~~html
<div class='wizard_buttons'>
  <button class='wizard_button' id='prev' disabled>Previous</button>
  <button class='wizard_button' id='next'>Next</button>
</div>
~~~

Key rules:
- Buttons must have stable IDs:
  - `prev`
  - `next`
- `prev` starts disabled
- `next` starts as Next

AI Agent rule:
- Wizard JS must query by these IDs, so do not change them unless you also change the JS.

---

## 6) JavaScript Wizard State Machine

### 6.1 Core variables and selection

The wizard script begins by selecting:
- `.step` elements
- `.step-wizard_content` panels
- buttons `#prev` and `#next`
- `currentStep` index state

Sample:

~~~javascript
const steps = document.querySelectorAll('.step');
const contents = document.querySelectorAll('.step-wizard_content');
const prevBtn = document.getElementById('prev');
const nextBtn = document.getElementById('next');
let currentStep = 0;
~~~

AI Agent rule:
- `currentStep` must always be an integer between 0 and steps.length-1.

### 6.2 updateWizard() function (the brain)

This function enforces:
- stepper active state (<= currentStep)
- content active state (== currentStep)
- prev button enabled/disabled
- next button text changes on last step

Sample logic:

~~~javascript
function updateWizard() {
  steps.forEach((step, index) => {
    step.classList.toggle('active', index <= currentStep);
  });

  contents.forEach((content, index) => {
    content.classList.toggle('active', index === currentStep);
  });

  prevBtn.disabled = currentStep === 0;

  nextBtn.textContent =
    currentStep === steps.length - 1
      ? 'Submit' /* or OK */
      : 'Next';
}
~~~

In your framework, the last button label is computed from FileMaker:

- If status is Pending/Not Completed: label is Submit
- Else: label is OK

Sample injection:

~~~filemaker
nextBtn.textContent = currentStep === steps.length - 1
  ? '" & If ( $$Status = "Pending" or $$Status = "Not Completed" ; "Submit" ; "OK" ) & "'
  : 'Next';
~~~

AI Agent rule:
- Wizard UI state must always be derived from `currentStep` and `steps.length`.
- The last step label must be controlled by FileMaker business logic when needed.

### 6.3 Next button behavior

Rules:
- If not last step: increment step and update
- If last step: run final action

Sample:

~~~javascript
nextBtn.addEventListener('click', () => {
  if (currentStep < steps.length - 1) {
    currentStep++;
    updateWizard();
  } else {
    /* Last step action */
    submitRecordsInHierarchicalTable();
  }
});
~~~

In the sample, last action is injected based on status:

~~~filemaker
" & If ( $$Status = "Pending" or $$Status = "Not Completed" ;
  "submitRecordsInHierarchicalTable()" ;
  "cancelAction()" ) & ";
~~~

AI Agent rule:
- Last step must always trigger a single “final action”.
- Final action should be:
  - submit script when editable
  - cancel/close when already submitted or read-only

### 6.4 Prev button behavior

Rules:
- If currentStep > 0: decrement step and update

Sample:

~~~javascript
prevBtn.addEventListener('click', () => {
  if (currentStep > 0) {
    currentStep--;
    updateWizard();
  }
});
~~~

### 6.5 Initialization call

Always call:

~~~javascript
updateWizard();
~~~

AI Agent rule:
- Without initialization, UI state will mismatch initial markup.

---

## 7) Wizard and “Read-Only / Disabled” State

In the sample, fields become disabled based on status:

~~~filemaker
Set Variable [ $$Disabled ; Value: If ( $$Status = "Submitted" or $$Status = "Appraised" ; " disabled" ) ]
~~~

Usage in HTML:

~~~html
<textarea id='staff_remarks' ... {{DISABLED}}>{{REMARKS}}</textarea>
~~~

AI Agent rule:
- Wizard does not manage permissions, FileMaker does.
- Wizard content panels must respect `$$Disabled` when injected into input elements.

---

## 8) Wizard and Business Logic Coupling (Submit vs OK)

This wizard changes behavior at the end:

- Editable state: final button is Submit, calls submit function
- Non-editable state: final button is OK, calls cancel action or closes page

AI Agent rule:
- Make wizard final step “meaningful”.
- Do not leave Next on last step. It must become Submit/OK/Finish based on business rules.

---

## 9) Wizard Integration With Other Page Scripts

In the sample, `$Scripts` includes many scripts:

- `$Wizard Script` (wizard navigation)
- `$Store Scores Scripts` (radio selection tracking + progress card updates)
- `$Hierarchical Table Scripts` (save and submit functions)
- `$Cancel Action Script` (exit page)
- `$View KPI Script` (open KPI modal)

Composition pattern:

~~~filemaker
Set Variable [ $Scripts ; Value:
"<script> " & 
  $Show Content Script & "¶¶" &
  $Cancel Action Script & "¶¶" &
  $Wizard Script & "¶¶" &
  $Store Scores Scripts & "¶¶" &
  $View KPI Script & "¶¶" &
  $Hierarchical Table Scripts &
" </script> "
]
~~~

AI Agent rule:
- Wizard is a navigation shell; it depends on other scripts to save/submit.
- Ensure functions called by the wizard exist before runtime:
  - `submitRecordsInHierarchicalTable()`
  - `cancelAction()`

---

## 10) CSS Expectations for Wizards

The wizard depends on these classes being styled:

- `.wizard-container`
- `.stepper`
- `.step`
- `.step.active`
- `.icon`
- `.line`
- `.wizard_content`
- `.step-wizard_content`
- `.step-wizard_content.active`
- `.wizard_buttons`
- `.wizard_button`

Minimum behavior:
- Only `.step-wizard_content.active` should be visible
- Inactive panels should be hidden (`display: none`)

Recommended CSS (if not already in `🖌️ Use Details CSS`):

~~~css
.step-wizard_content { display: none; }
.step-wizard_content.active { display: block; }

.step.active .icon { /* highlight current and completed */ }
.step.active p { /* highlight label */ }

.wizard_buttons { display: flex; justify-content: space-between; gap: 12px; }
.wizard_button[disabled] { opacity: 0.5; cursor: not-allowed; }
~~~

AI Agent rule:
- If CSS is missing, wizard will show all steps at once and break the UX.
- Always verify that `.step-wizard_content` is hidden unless active.

---

## 11) Standard Build Pattern (How the AI Agent Should Implement a Wizard)

When asked to implement a wizard, the AI Agent must do:

1) Decide steps and titles
   - Example: Start, KPIs, Remarks
2) Build stepper HTML with `.step` items
3) Build content panels with `.step-wizard_content` in the same order
4) Add navigation buttons with IDs `prev` and `next`
5) Implement wizard JS:
   - `currentStep`
   - `updateWizard()`
   - next/prev event handlers
6) Define the last-step action based on FileMaker business logic
7) Ensure any functions referenced on last step exist (submit/save/cancel)
8) Ensure CSS supports hiding inactive steps

---

## 12) Common Failure Modes and Fixes

### 12.1 Clicking Next does nothing
Cause:
- `#next` button ID is different
Fix:
- ensure `id='next'` exists

### 12.2 All step contents show at once
Cause:
- missing CSS that hides `.step-wizard_content`
Fix:
- ensure `.step-wizard_content { display: none; }` and `.active { display: block; }`

### 12.3 Stepper highlights incorrectly
Cause:
- active logic uses `index <= currentStep` but you want only current step
Fix:
- decide desired behavior:
  - completed style for earlier steps (<=)
  - current-only style (==)
- adjust accordingly

### 12.4 Last step runs wrong action
Cause:
- status logic inconsistent (`Pending`, `Not Completed`, `Submitted`, `Appraised`)
Fix:
- standardize status values and the condition used in wizard final action

### 12.5 Wizard references functions that do not exist
Cause:
- last step calls `submitRecordsInHierarchicalTable()` but scripts not included
Fix:
- always include the submit/save functions in `$Scripts`

---

## 13) Required Clarifications (Ambiguities the AI Agent Cannot Guess)

For perfect consistency across pages, clarify:

1) Status vocabulary
   - What are the exact allowed values for `$$Status`?
   - In this sample it checks: Pending, Not Completed, Submitted, Appraised
   - Are there others like Not Submitted, Completed, Approved?

2) Desired stepper behavior
   - Should completed steps stay highlighted (index <= currentStep), or should only the current step highlight?

3) Last-step action policy
   - When status is not editable, should final action always be `cancelAction()`?
   - Or should it call a different script, like close modal, go back, or show a confirmation?

4) Wizard clickability
   - Should clicking on step circles allow jumping steps?
   - Current sample does not support click-to-jump.
   - If required, AI Agent must implement step click handlers and validation rules.

5) Validation rule placement
   - Should validation occur:
     - when moving Next
     - only on Submit
     - both
   - Current sample validates “all questions answered” only on submit.

---

## 14) AI Agent Output Standard (What Must Be Produced)

When asked for a wizard implementation, the AI Agent must output:

1) HTML
   - `.wizard-container`
   - `.stepper` with `.step` items
   - `.wizard_content` with `.step-wizard_content` panels
   - `.wizard_buttons` with `#prev` and `#next`

2) JavaScript
   - `currentStep` state
   - `updateWizard()` toggling `active`
   - next/prev event listeners
   - last-step final action calling a known function

3) FileMaker contracts
   - Where the last-step label and action are derived from FileMaker variables (status)
   - Which FileMaker scripts are called (Save, Submit, Cancel)

4) CSS expectations
   - Ensure inactive steps are hidden and active is visible

---

# CauferoAppStarter Documentation
## Info Panels and How They Are Implemented (WebViewer Pages)

> Purpose of this doc  
> Train an AI Agent to implement **Info Panels** in CauferoAppStarter WebViewer pages, using the provided sample as the reference pattern.  
> In this codebase, the Info Panels are represented by blocks like:  
> - `div class='instructions1'`  
> - `div class='instructions2'`  
> - (and optionally `div class='instructions3'`)

---

## 1) What an Info Panel Means Here (CauferoAppStarter Definition)

An **Info Panel** is a **content card** used to display explanatory text to the user.

It is not an "alert" (toast or warning).
It is not a modal.
It is a **panel** that sits inside the page layout and stays visible while the user reads or fills a section.

In your sample, the Info Panels are used to present:
- A heading plus a short list of rules (`instructions1`)
- A heading plus an explanatory paragraph (`instructions2`)
- Additional notes / extra info (`instructions3`)

Reference blocks from the sample (Tab 8 and Tab 9):

~~~html
<div class='instructions1'>
  <h2>Instructions</h2>
  <ul>
    <li>Requester fills in all cells highlighted in YELLOW.</li>
    <li>Requester provides overall description of PO (below).</li>
    <li>Requester emails the form to <strong>Purchasing@translarity.com</strong>.</li>
  </ul>
</div>

<div class='instructions2'>
  <h2>Overall Description</h2>
  This purchase order is intended to procure essential equipment to support the lamination tables for the Guide Plate project. The requested items include extension cords, cord covers, and crucible clamps, ensuring a safer and more efficient workspace.
</div>

<div class='instructions3'>
  <h2>Additional Notes</h2>
  Extend Power to Lamination Tables
</div>
~~~

AI Agent rule:
- When you see `instructions1`, `instructions2`, `instructions3`, treat them as **Info Panels**.

---

## 2) When to Use Info Panels

Use an Info Panel when the user needs **context before acting**.

Examples:
- Explaining how to fill a form (rules, checklist)
- Explaining what a section means (definition, purpose)
- Explaining what the system is about to do (before submission)
- Giving short help text that should not be hidden behind tooltips

AI Agent rule:
- If removing the text would increase user mistakes or support questions, it belongs in an Info Panel.

---

## 3) The Info Panel Types in This Pattern

This sample implies 3 tiers of Info Panels (by class name):

### 3.1 `instructions1`
Used for: **rules + steps + checklists**  
Most often contains:
- `<h2>` title
- `<ul>` list of bullet instructions
- Optional `<strong>` emphasis and inline links/email

### 3.2 `instructions2`
Used for: **explanation block**  
Most often contains:
- `<h2>` title
- A paragraph-style body (plain text or wrapped in `<p>`)

### 3.3 `instructions3`
Used for: **additional notes / special remarks**  
Most often contains:
- `<h2>` title
- A short note, summary, or one-line instruction

AI Agent rule:
- The class name communicates the intended structure.
- Use the matching structure consistently.

---

## 4) Required HTML Structure for Each Info Panel

### 4.1 `instructions1` (Rules Panel)

Minimum structure:

~~~html
<div class='instructions1'>
  <h2>Instructions</h2>
  <ul>
    <li>Instruction line 1.</li>
    <li>Instruction line 2.</li>
    <li>Instruction line 3.</li>
  </ul>
</div>
~~~

Allowed elements:
- `<strong>` inside list items
- `<a>` links inside list items
- Small inline `<span>` styling if required (example: coloring “YELLOW”)

AI Agent rules:
- Must contain a title: `<h2>...</h2>`
- Must contain a list: `<ul>...</ul>`
- Must avoid long paragraphs. If you need paragraphs, use `instructions2`.

---

### 4.2 `instructions2` (Explanation Panel)

Minimum structure (exactly like the sample):

~~~html
<div class='instructions2'>
  <h2>Overall Description</h2>
  Short explanatory text that introduces what this section is for.
</div>
~~~

Recommended structure (more consistent markup):

~~~html
<div class='instructions2'>
  <h2>Overall Description</h2>
  <p>Short explanatory text that introduces what this section is for.</p>
</div>
~~~

AI Agent rules:
- Must contain a title: `<h2>...</h2>`
- Must contain body text as either plain text or a `<p>`
- Must be short and scannable (1 to 5 lines in normal desktop view)

---

### 4.3 `instructions3` (Notes Panel)

Minimum structure:

~~~html
<div class='instructions3'>
  <h2>Additional Notes</h2>
  Short note content goes here.
</div>
~~~

AI Agent rules:
- Must contain a title
- Must contain short content
- Must not be used for multi-step instructions (use `instructions1`)

---

## 5) Placement Rules (Where to Put Info Panels)

Info Panels are most effective when placed:
- Above the section they explain
- Immediately before the table/form they relate to
- Near the top of a tab if they apply to the whole tab

From the sample (Tab 8), the flow is:

1. `instructions1` (rules)
2. `instructions2` (overall description)
3. section title (`<h3>Items</h3>`)
4. table
5. footer controls
6. `instructions3` (notes)

AI Agent rule:
- Place panels in reading order, before the objects they refer to.

---

## 6) CSS Contract (What the AI Agent Must Assume)

In the sample, CSS is pulled from:

~~~filemaker
Perform Script [ “🖌️ Use Details CSS” ]
Set Variable [ $styles ; Value: Get ( ScriptResult ) ]
~~~

That means `instructions1`, `instructions2`, `instructions3` are expected to already be styled by that CSS.

AI Agent rules:
- Do not invent new CSS class names for Info Panels unless explicitly requested.
- Use the existing `instructions1/2/3` classes to inherit theme styling.
- If the page is missing styling, the correct fix is to ensure the CSS script includes these classes, not to inline random CSS per page.

---

## 7) Data Injection Rules (Text Fields Only)

This documentation is for **text content** only.

That means:
- Titles are plain text
- Body is plain text
- Links and emphasis are allowed, but no form inputs belong in these panels

### 7.1 Static text (like sample)
Use literal HTML strings exactly as written.

### 7.2 Dynamic text (from FileMaker fields or variables)
Use FileMaker concatenation inside the HTML string:

~~~filemaker
Set Variable [ $InfoPanelHTML ; Value:
"
<div class='instructions2'>
  <h2>Overall Description</h2>
  <p>" & $OverallDescription & "</p>
</div>
" ]
~~~

AI Agent rules:
- Escape or sanitize user-entered text if it can contain `<` or `&`.
- If there is no sanitization utility yet, flag it as a required helper function (see ambiguities section).

---

## 8) Implementation Pattern Inside a Page Script

The standard pattern in your sample is:

1. Build HTML chunks (`$Tab8 HTML`, `$Tab9 HTML`, etc.)
2. Combine chunks into `$HTML`
3. Inject `$HTML` into `$$Page`

So Info Panels live **inside the tab HTML** (or inside the page section they belong to).

Example (Tab 8 pattern):

~~~filemaker
Set Variable [ $Tab8 HTML ; Value:
"
<div id='tab8' class='content'" & If ( $$Tab To Show = 8 ; " style='display: block;'" ) & ">
 <h3>Tab 8</h3>
 <p>Tab 8 description</p>

 <div class='instructions1'>
  <h2>Instructions</h2>
  <ul>
   <li>Requester fills in all cells highlighted in YELLOW.</li>
   <li>Requester provides overall description of PO (below).</li>
   <li>Requester emails the form to <strong>Purchasing@translarity.com</strong>.</li>
  </ul>
 </div>

 <div class='instructions2'>
  <h2>Overall Description</h2>
  This purchase order is intended to procure essential equipment...
 </div>

 <!-- rest of tab content -->
</div>
" ]
~~~

AI Agent rules:
- Info Panels are constructed inside the same HTML string as the section they explain.
- Avoid placing Info Panels outside the container structure unless you want them global to the page.

---

## 9) Quality Rules for Info Panel Text (Training Rules)

Because these panels are guidance and context, the text must be:
- Specific
- Short
- Unambiguous
- Written in plain English
- Easy to skim

Rules:
- Titles should be 1 to 4 words (example: "Instructions", "Overall Description", "Additional Notes")
- Lists should be 3 to 7 bullets, not 20
- Each bullet should be one instruction only
- Avoid mixing unrelated topics inside one panel

AI Agent rule:
- If content grows beyond 7 bullets or becomes multi-paragraph, split into multiple panels.

---

## 10) Ambiguities and Clarifications Needed (Must Ask User)

The sample proves the Info Panels exist, but it does not specify certain contracts the AI Agent must follow.

### 10.1 Are `instructions1/2/3` the official naming convention?
- Should the AI Agent always use exactly these class names?
- Or can there be other panel class names (example: `infoPanel`, `helpPanel`) elsewhere in the system?

### 10.2 Is there an official HTML sanitizer/escape function?
If `$OverallDescription` comes from a user-entered field, it can break HTML.
- Do you have a standard utility for escaping text to safe HTML?
- If yes, what is the function name and how is it called?

### 10.3 Is there a consistent design meaning between `instructions1` vs `instructions2` vs `instructions3`?
The sample implies layout differences, but does not define them.
- Do they differ by color, border, background, icon, or spacing?
- Should `instructions1` always render with an icon or special highlight?

### 10.4 Should Info Panels ever be collapsible?
This sample does not show collapsible panels.
- Do you want them always open, or optionally collapsible?

AI Agent rule:
- If these clarifications are not provided, default to using `instructions1/2/3` exactly as shown and keep panels always visible.

---

## 11) Copy-Paste Templates (AI Agent Output Blocks)

### 11.1 Template: Rules Panel (`instructions1`)
~~~html
<div class='instructions1'>
  <h2>Instructions</h2>
  <ul>
    <li>Write instruction 1.</li>
    <li>Write instruction 2.</li>
    <li>Write instruction 3.</li>
  </ul>
</div>
~~~

### 11.2 Template: Explanation Panel (`instructions2`)
~~~html
<div class='instructions2'>
  <h2>Overall Description</h2>
  <p>Write a short explanation that makes the next section obvious.</p>
</div>
~~~

### 11.3 Template: Notes Panel (`instructions3`)
~~~html
<div class='instructions3'>
  <h2>Additional Notes</h2>
  Short note here.
</div>
~~~

---

## 12) Minimal Acceptance Checklist (For the AI Agent)

A generated Info Panel implementation is correct only if:

- It uses `instructions1` for bullet rules and includes `<h2>` + `<ul>`
- It uses `instructions2` for explanation and includes `<h2>` + body text
- It uses `instructions3` for short notes and includes `<h2>` + short body
- It is placed before the content it explains
- It remains within the tab/page container structure
- It does not introduce unrelated class names without instructions

---

# Startup Script (CauferoAppStarter)

## Purpose

The **Startup** script is the canonical boot sequence for CauferoAppStarter.

It runs at application launch and is responsible for:

- Setting the UI into the expected “app mode” (kiosk-like look and feel).
- Loading and caching global session variables for:
  - App identity and configuration
  - Business identity and branding
  - Current user identity, role, and preferences
- Applying device and platform UI rules (zoom and WebViewer load strategy).
- Preparing and rendering the WebViewer-based shell (Page and Menu).
- Selecting the initial navigation destination for the current user based on role.
- Logging the user as “Logged In”.
- Resetting startup-only globals to leave the system in a clean state.

This script does not “build the app”. It configures the running session so every other script can rely on a stable set of global variables and a consistent UI environment.

---

## Key Concepts

### Session Globals (Canonical)
Startup establishes a set of `$$` variables that act as the system’s runtime cache. The system expects these globals to be available after Startup:

- App identity: `$$App ID`, `$$App Name`, `$$App Slogan`, `$$Partners`, `$$App Launch Year`
- Splash content: `$$Splash Screen Loading Statement`, `$$Splash Background Photo`
- Theme: `$$App Default Theme`, `$$My Preferred Theme`, `$$Theme`
- Business: `$$Business Name`, `$$Business Phone`, `$$Business Location`, `$$Website`, `$$Business Logo URL`, `$$Business Logo Icon URL`, `$$Menu Mode`
- User: `$$My User ID`, `$$My Staff ID`, `$$My Staff Number`, `$$My Fullname`, `$$My First Name`, `$$My Role ID`, `$$My Role`, `$$My Photo`
- Preferences: `$$Preferred Currency`
- UI/Mode: `$$Work Mode`, `$$Layout Name`
- Navigation: `$$Startup` (temporary flag used during initial navigation)
- Utilities: `$$No Image URL`
- OAuth: `$$Google Client ID`, `$$Google Client Secret`, `$$Google Scope`

### WebViewer Shell Model
CauferoAppStarter uses a WebViewer-based shell for the main UI.

Key WebViewer objects:
- `Page` (the main content area)
- `Links` (the menu area, used when menu mode is persistent)

The Startup script ensures the correct shell layout is used:
- `Page With Menu` layout when `$$Menu Mode = "Persistent Side Menu"`
- `Page` layout otherwise

### Platform-Specific WebViewer Loading
Different platforms may require different approaches to loading HTML into WebViewers.

Startup uses:
- `data:text/html;base64,` + `Base64Encode ( $$Page )` and `Base64Encode ( $$Menu )` when `Get ( SystemPlatform ) = 4`
- Raw HTML strings otherwise

The script treats `Get ( SystemPlatform ) = 4` as the special case platform.

---

## Script Responsibilities (High-Level)

1. **Enter Splash / Pre-UI State**
2. **Apply UI concealment (toolbars/menubar/ruler)**
3. **Enable developer-friendly syntax coloring (MBS)**
4. **Load core session data (App, Business, User)**
5. **Set OAuth configuration for Google Drive**
6. **Resolve theme and basic runtime defaults**
7. **Apply privilege-based UI locking**
8. **Maximize window**
9. **Run splash animation (for non-Bridge apps)**
10. **Log user as “Logged In”**
11. **Render banner**
12. **Load the shell layout and WebViewers**
13. **Compute role-based first link and navigate**
14. **Apply special post-load behavior for The Bridge**
15. **Cleanup startup-only globals**

---

## Preconditions

Startup assumes:

- `Settings::App ID` and `Settings::Business ID` are already set (or are set by earlier file-open logic).
- A valid user session exists (FileMaker account login already occurred).
- Layouts exist and are named exactly:
  - `Splash Screen` (Settings)
  - `Page With Menu` (Settings)
  - `Page` (Settings)
- WebViewer object names exist as expected:
  - `Page`
  - `Links`
- Auxiliary scripts exist and are callable by name:
  - `Get App Details`
  - `Get Business Details`
  - `Preferred Currency`
  - `Get My Details`
  - `Set Zoom Level Based On Device`
  - `+++ Splash Page`
  - `+++ Banner Page`
  - `Select Link`
  - `Save A Record`
- Tables and fields referenced by auxiliary scripts exist (Apps, Businesses, Users, Roles, Images, Role Links, Links).

---

## Postconditions

After Startup completes:

- The UI is in the correct shell layout.
- The banner and menu (if applicable) are loaded.
- The user is navigated to the first permitted link for their role.
- Session globals are populated and can be used by downstream scripts.
- Temporary startup variables are cleared (`$$Startup`, `$$Splash`, `$$Splash Background Photo`, `$$Selected Record ID`).

---

## Detailed Walkthrough (Step-by-Step)

### 1) Enter Startup UI Context (Splash Screen)

Startup begins by switching to the splash layout and hiding UI chrome:

- Go to layout: `Splash Screen` (Settings)
- Hide toolbars
- Hide and configure menubar visibility/locking
- Hide text ruler

**Why this exists**
- Ensures a clean launch state.
- Prevents the user from seeing FileMaker UI elements during initialization.
- Makes the solution feel like a real product.

---

### 2) Enable Syntax Coloring (MBS Plugin)

Startup calls MBS plugin functions to enable and configure syntax coloring.

It enables syntax coloring:
- `MBS( "SyntaxColoring.Enable"; "1" )`

Then it adds tags for Script Steps such as:
- `Perform Script`
- `Go To Layout`
- `Pause/Resume Script`
- comment marker `#`
- `Set Variable`
- `If`, `Else If`, `Else`, `End If`
- `Loop`, `Exit Loop If`, `End Loop`

It also adds tags for:
- Functions: `If`, `Case`
- Formula token coloring (example tags):
  - `variable color`
  - `text color`

**Why this exists**
- The system likely contains WebViewer-based text editors, code panels, or script viewers.
- Syntax coloring improves readability of generated scripts or formulas shown in the UI.
- This is a global setup, so it is done once at startup.

**Agent guidance**
- Treat these as **cosmetic developer UX configuration**, not business logic.
- Do not remove or reorder these unless you are intentionally changing syntax coloring behavior across the app.

---

### 3) Load App Details (Auxiliary Script)

Startup calls:
- `Perform Script [ “Get App Details” ]`

This script is responsible for populating global variables for app identity and configuration.

Key results:
- `$$App ID`
- `$$App Name`
- `$$App Slogan`
- `$$Splash Screen Loading Statement`
- `$$Partners`
- `$$App Launch Year`
- `$$Splash Background Photo`
- `$$App Default Theme`
- `$$Report Header` (hardcoded here in example)
- Flush caches via `Refresh Window [ Flush cached join results ; Flush cached external data ]`

**Why this exists**
- AppStarter can host multiple apps (or multiple configurations) and needs a single boot step that reads “what app am I?”
- These globals support:
  - window title
  - splash page rendering
  - theme selection
  - partner labels
  - report header output

---

### 4) Apply Window Title

Startup sets the window title:
- `Set Window Title [ $$App Name ]`

**Why this exists**
- This is branding and a usability cue, especially when multiple FileMaker windows are open.

---

### 5) Apply Zoom Based on Device

Startup calls:
- `Perform Script [ “Set Zoom Level Based On Device” ]`

This auxiliary script applies zoom rules such as:
- If `Get ( Device ) = 3`, set zoom to 75%
- Else if a specific IP address matches, set zoom to 75%
- Else if `$$Menu Mode = "Persistent Side Menu"`, set zoom to 100%

**Why this exists**
- Different devices and screen densities require different zoom to make the UI feel consistent.
- Menu mode influences available screen space, so it can influence zoom.

**Agent guidance**
- Zoom is a presentation concern. Do not mix business logic into zoom scripts.
- Maintain a single place for zoom rules (this script) to avoid drift.

---

### 6) Load Current User Details (Auxiliary Script)

Startup calls:
- `Perform Script [ “Get My Details” ]`

This script uses ExecuteSQL to retrieve user + role metadata and assigns:

- `$$My User ID`
- `$$My Staff ID`
- `$$My Staff Number`
- `$$My Fullname`
- `$$My First Name`
- `$$My Role ID`
- `$$My Role`
- `$$My Preferred Theme`
- `$$My Photo` (Base64 Image from Images where Reference = Staff ID)

It also flushes caches via:
- `Refresh Window [ Flush cached join results ; Flush cached external data ]`

**Why this exists**
- The app uses role-based navigation and permissions. Role is required early.
- Theme preference is user-specific, so it must be loaded before theme resolution.
- User photo is needed for profile UI components.

---

### 7) Set Google OAuth Globals (Drive Integration)

Startup sets Google OAuth configuration:

- `$$Google Client ID`
- `$$Google Client Secret`
- `$$Google Scope`

The scope includes:
- Drive file scope
- userinfo email
- userinfo profile

**Why this exists**
- The app can integrate with Google Drive (upload, file operations).
- Centralizing these values in Startup makes them accessible to any script that needs OAuth.

**Agent guidance**
- Treat these as environment configuration.
- If the solution is distributed, secrets should be stored safely (avoid embedding secrets in scripts). In this file, they are stored in globals for convenience during runtime.

---

### 8) Resolve Theme and Initialize Defaults

Startup sets:
- `$$Theme` using:
  - user preferred theme if available
  - else app default theme
- `$$Work Mode = "Relaxing"`
- `$$No Image URL` placeholder

**Why this exists**
- Theme must be resolved before rendering HTML/CSS for WebViewers.
- Work mode is a runtime state used by UI logic.
- Placeholder image URL is used where images may be missing.

---

### 9) Privilege-Based UI Locking

Startup checks:
- `Get ( AccountPrivilegeSetName ) = "[Full Access]"`

If Full Access:
- Menubar is unlocked (but hidden)
- Toolbars hidden

Else:
- Menubar locked and hidden
- Toolbars locked and hidden

**Why this exists**
- Prevents non-admin users from accessing FileMaker UI features.
- Allows admins to access needed menus when debugging or maintaining.

**Agent guidance**
- This is a core security posture for “app-like” FileMaker systems.
- Preserve this behavior unless the solution intentionally changes security stance.

---

### 10) Maximize Window

Startup:
- `Adjust Window [ Maximize ]`

**Why this exists**
- Ensures consistent layout proportions.
- Avoids the user seeing partial UI or unrendered regions on launch.

---

### 11) Splash Experience (Conditional)

Startup checks:
- If `$$App Name ≠ "The Bridge"`

Then:
- Runs `+++ Splash Page`
- Pauses 6 seconds

**Why this exists**
- Provides a branded loading screen experience for most apps.
- “The Bridge” is treated specially and may skip or alter splash timing.

**Agent guidance**
- Use app-name branching only when justified by real UI differences.
- Keep the splash logic isolated in `+++ Splash Page` so Startup remains mostly orchestration.

---

### 12) Log User as Logged In

Startup calls:
- `Perform Script [ “Save A Record” ; Parameter: "Users¶" & Settings::Location & "¶ID¶" & $$My User ID & "¶Logged In¶1" ]`

This indicates a generic “Save A Record” utility that can set fields based on a structured parameter.

**Why this exists**
- Tracks active sessions / user status.
- Supports operational visibility (“who is logged in”) and may support access rules.

**Agent guidance**
- Treat this as an audit/state update step.
- Maintain consistent parameter formats so generic scripts remain reusable.

---

### 13) Set Initial Layout Name and Render Banner

Startup sets:
- `$$Layout Name = "Dashboard"`

Then runs:
- `+++ Banner Page`

**Why this exists**
- `$$Layout Name` likely informs the banner’s title and/or context.
- Banner is a persistent UI component that must exist before navigating.

---

### 14) Load Shell Layout + WebViewers (Non-Platform-4 Early Load)

Startup includes an early branch:

- If `Get ( SystemPlatform ) ≠ 4`
  - If `$$Menu Mode = "Persistent Side Menu"`
    - Go to `Page With Menu`
    - Set WebViewer `Page` to rendered HTML (`$$Page`)
    - Set WebViewer `Links` to rendered HTML (`$$Menu`)
  - Else
    - Go to `Page`

**Why this exists**
- On non-platform-4 systems, the shell can be initialized immediately using raw HTML or direct WebViewer loading.
- Persistent side menu requires both `Page` and `Links` objects to be loaded.

**Agent guidance**
- The shell must be ready before selecting a navigation link.
- The Page/Menu HTML must be available in globals (`$$Page`, `$$Menu`) by the time these steps run. Those globals are expected to be prepared by other scripts in the system.

---

### 15) Compute Role-Based First Link

Startup builds `$Links` via ExecuteSQL.

It selects link IDs allowed for the user’s role:

- Source tables:
  - `Role Links` (mapping Role → Link)
  - `Links` (metadata for link ordering and layout target)
- Filters:
  - `Role ID = ?`
  - `Order` is not null
  - `Layout` is not null
- Ordering:
  - by `Links::Order`

Then it selects:
- `$First Link = GetValue ( $Links ; 1 )`

**Why this exists**
- The app’s navigation is role-based.
- Startup must land the user on a valid first page that they are allowed to access.

**Agent guidance**
- The first permitted link is chosen by explicit ordering, not by creation date or random selection.
- Links without Order or Layout are not eligible for startup routing.

---

### 16) Trigger Initial Navigation (Select Link)

Startup sets a temporary flag:

- `$$Startup = 1`
- Perform Script: `Select Link` with parameter `$First Link`
- Clear: `$$Startup = ""`

**Why this exists**
- `Select Link` likely behaves differently during startup (for example: skipping animations, preventing extra logging, or preventing “back” behavior).
- The temporary global is used to inform downstream behavior.

**Agent guidance**
- Treat `$$Startup` as a transient “boot mode” flag.
- Clear it immediately after the initial link selection to avoid affecting later navigation.

---

### 17) Post-Load Layout Handling (Bridge vs Non-Bridge)

Startup then branches again:

- If `$$App Name = "The Bridge"`
  - Go to `Page With Menu`
  - Set `Page` WebViewer from `$$Page`
  - Set `Links` WebViewer from `$$Menu`
  - Refresh objects: `Page` and `Links`

Else:
- If `Get ( SystemPlatform ) = 4`
  - If `$$Menu Mode = "Persistent Side Menu"`
    - Go to `Page With Menu`
    - Set `Page` and `Links` WebViewers using base64 strategy
  - Else
    - Go to `Page`

**Why this exists**
- “The Bridge” requires a forced re-render + refresh for both web viewers, likely due to complex rendering or initialization timing.
- Platform-4 requires base64 webviewer loading and may require layout switching after the initial navigation.
- The system separates “desktop/non-4” behavior from “platform-4” behavior.

**Agent guidance**
- Maintain the distinction between:
  - non-platform-4 early shell load
  - platform-4 post-navigation shell load
- The Bridge special case is deliberate and should be preserved unless Bridge rendering logic changes.

---

### 18) Cleanup Startup Globals

At the end, Startup resets:

- `$$Splash = ""`
- `$$Splash Background Photo = ""`
- `$$Selected Record ID = ""`

**Why this exists**
- Splash variables should not persist after launch.
- Selected record must be blank at startup to avoid stale context.
- The runtime should enter the app in a neutral state.

---

## Auxiliary Scripts (Responsibilities)

### Get App Details (Responsibilities)

This script loads “app identity” and related configuration.

Key operations:
- Runs `Preferred Currency`
- Runs `Get Business Details`
- Uses ExecuteSQL to read from `Apps` by `Settings::App ID`
- Assigns:
  - `$$App ID`
  - `$$App Name`
  - `$$App Slogan`
  - `$$Splash Screen Loading Statement`
  - `$$Partners`
  - `$$App Launch Year`
  - `$$Splash Background Photo` (from Apps)
  - `$$App Default Theme` (derived from Apps::Available Themes)
  - `$$Report Header` (hard-coded in shown sample)
- Flushes caches via `Refresh Window`

**Key rule**
- This script is authoritative for app identity. Other scripts should rely on `$$App Name` and `$$App ID` after Startup.

---

### Preferred Currency (Responsibilities)

Loads the business’s preferred currency into:
- `$$Preferred Currency`

Source:
- `Businesses::Preferred Currency` filtered by `Settings::Business ID`

---

### Get Business Details (Responsibilities)

Loads business identity and branding into globals:

- `$$Business Name`
- `$$Business Phone`
- `$$Business Location`
- `$$Website`
- `$$Business Logo URL`
- `$$Business Logo Icon URL` (with placeholder fallback)
- `$$Menu Mode`

Source:
- `Businesses` filtered by `Settings::Business ID`

**Key rule**
- `$$Menu Mode` is the authoritative selector for which shell layout should be used.

---

### Set Zoom Level Based On Device (Responsibilities)

Applies zoom rules based on device/platform/system conditions.

Outputs:
- Zoom level

This script does not set globals. It only configures the window.

---

### Get My Details (Responsibilities)

Loads current user identity, role, preferences, and profile image into globals.

Outputs:
- `$$My User ID`
- `$$My Staff ID`
- `$$My Staff Number`
- `$$My Fullname`
- `$$My First Name`
- `$$My Role ID`
- `$$My Role`
- `$$My Preferred Theme`
- `$$My Photo` (base64)

Flushes caches after loading.

**Key rule**
- Role and theme preference must be available before menu building and theme selection.

---

## Data Dependencies (Tables / Fields)

Startup and its auxiliary scripts depend on these conceptual records:

### Apps
Must contain (at minimum):
- ID
- Name
- Slogan
- Splash Screen Loading Statement
- Partners
- App Launch Year
- Splash Background
- Available Themes

### Businesses
Must contain (at minimum):
- ID
- Name
- Phone
- Location
- City
- Website
- Logo URL
- Logo Icon URL
- Preferred Currency
- App Menu Mode

### Users / Roles / Staff
Must allow retrieval of:
- User identity
- Full name
- First name
- Role ID
- Role name
- Preferred colour theme
- Linked staff record identity

### Images
Must allow retrieval of:
- Base64 Image by reference (staff ID)

### Links / Role Links
Must allow retrieval of:
- Link IDs per role
- Ordering and layout validity

---

## Agent Operating Rules for Startup Documentation

This section defines how the AI agent should interpret and use this documentation.

### Rule 1: Startup is Orchestration
Startup coordinates other scripts and configures the session. It should not contain heavy business logic.

### Rule 2: Globals are the Contract
After Startup, downstream scripts should assume required globals exist. Do not duplicate the loading logic elsewhere.

### Rule 3: Menu Mode Drives Shell Layout
- If `$$Menu Mode = "Persistent Side Menu"` use `Page With Menu` and load both `Page` and `Links`.
- Otherwise use `Page` layout.

### Rule 4: Role Determines First Page
The first page is computed by:
- Role ID → Role Links → Links
- ordered by Links::Order
- must have Layout and Order set

### Rule 5: Platform Differences are Deliberate
Maintain separate handling for:
- `Get ( SystemPlatform ) ≠ 4`
- `Get ( SystemPlatform ) = 4`

Platform-4 uses base64 `data:` URLs for WebViewers.

### Rule 6: Bridge is a Special Case
If app name equals `"The Bridge"`, force re-render + refresh of `Page` and `Links`.

### Rule 7: Clean State at End
Startup must clear transient variables so the system begins with a neutral context.

---

## Canonical Script Listing (Reference Only)

Below is a simplified, structural outline of Startup. It is not intended to be executed from this documentation, only to reflect the sequence of responsibilities.

~~~filemaker
Startup:
- Go to Splash Screen
- Hide UI chrome
- Enable MBS syntax coloring
- Perform Script: Get App Details
- Set Window Title to $$App Name
- Perform Script: Set Zoom Level Based On Device
- Perform Script: Get My Details
- Set Google OAuth globals
- Resolve $$Theme and defaults
- Privilege-based menu/toolbar locking
- Maximize window
- Optional splash page + pause (non-Bridge)
- Log user as Logged In
- Set $$Layout Name = "Dashboard"
- Render banner page
- Load shell layout + WebViewers (non-platform-4)
- Query role links, pick first
- Set $$Startup flag, Perform Script: Select Link, clear flag
- Special post-load handling for Bridge and platform-4
- Clear splash and selected-record globals
~~~

---

## Expected Outcomes (Verification Checklist)

After Startup:

- [ ] Window title matches `$$App Name`
- [ ] Toolbars are hidden
- [ ] Menubar is hidden and locked for non-admin users
- [ ] App identity globals are set (`$$App Name`, `$$App ID`, etc.)
- [ ] Business branding globals are set (`$$Business Name`, logos, menu mode)
- [ ] User globals are set (`$$My User ID`, role, theme preference, photo)
- [ ] `$$Theme` is resolved correctly (preferred theme overrides default)
- [ ] Banner is visible
- [ ] Shell layout matches menu mode
- [ ] User lands on the first permitted link for their role
- [ ] `$$Startup` is cleared
- [ ] `$$Selected Record ID` is blank
- [ ] Splash globals are cleared

---

## Notes and Constraints

- The Startup script references global variables `$$Page` and `$$Menu` when setting WebViewers. These must be prepared by other scripts in the system. Startup assumes they exist and are valid HTML strings at the time of rendering.
- The role-based first link logic assumes:
  - Roles are correctly mapped in Role Links
  - Links are correctly ordered and have valid target layouts
- The “Bridge” special case should only be removed if Bridge-specific rendering timing issues are resolved elsewhere.

---

# Select Link Script Documentation (CauferoAppStarter)

## Purpose

`Select Link` is the central navigation router for CauferoAppStarter.

It receives a Link ID, looks up the Link’s intended destination in the `Links` table, applies global UI cleanup rules (modal handling, banner refresh, menu behavior), then routes the user by calling the correct page script based on the link’s configured `"Layout"` value.

This script exists so the app can:
- Navigate consistently from anywhere (menus, buttons, list items).
- Control banner + menu synchronization from one place.
- Keep navigation mostly data-driven (Link record defines a target layout name).
- Map layout names to canonical “page scripts” that actually render pages.

---

## Script Name

- **Name:** `Select Link`

---

## What This Script Does (High-Level Flow)

1. **Capture the Link ID** from the Script Parameter into `$$Link ID`.
2. **Read destination metadata** from the `Links` table using `ExecuteSQL`:
   - `"Layout"` → stored in `$$Layout Name`
   - `"Parent Link ID"` → stored in `$$Parent Link ID`
3. **Apply pre-navigation UI rules**:
   - Special handling if current layout is `"Modal"`.
   - Close window logic depending on `$$Startup` and `$$Menu Mode`.
   - Always refresh the banner.
   - Ensure menu state is correct for Persistent Side Menu mode.
4. **Route to the correct destination** by calling the appropriate `+++ <Page Name>` script based on `$$Layout Name` (and sometimes `$$App Name`).
5. **Fallback handling** if no route matched.

---

## Inputs

### Script Parameter (Required)

This script expects a **value list** style parameter, where:

- **Value 1:** Link ID (primary key of the `Links` table record)

Example parameter:
- `Get ( ScriptParameter )` = `12345`

If you need to pass more values in the future, keep **Link ID as Value 1** to preserve compatibility.

---

## Outputs

This script does not return a meaningful Text Result in most cases.

It changes global variables and performs navigation by running page scripts.

---

## Global Variables Used

### Set by This Script

- `$$Link ID`
  - The selected Link record ID.
  - Always set at the start of the script.

- `$$Search Item`
  - Reset to empty string.
  - Used elsewhere in the app as a search context value (cleared on navigation).

- `$$Layout Name`
  - The Link’s configured layout label from the `Links` table.
  - Used to determine which destination route to execute.

- `$$Parent Link ID`
  - The Link’s parent grouping ID from the `Links` table.
  - Used for menu hierarchy / navigation context.

### Read by This Script (Must Exist in the System)

- `$$Startup`
  - Used to detect first-time app startup behavior.
  - If `$$Startup ≠ 1`, navigation behaves differently (may close windows).

- `$$Menu Mode`
  - Controls how the menu behaves.
  - This script checks for `$$Menu Mode = "Persistent Side Menu"`.

- `$$App Name`
  - Used to apply app-specific routing overrides (example: “The Bridge”).

- `$$Page`
  - Cleared in fallback case.

---

## Tables and Fields Required

### Table: `Links`

This script queries the `Links` table using `ExecuteSQL`.

Required fields (exact names in SQL):
- `Links::ID`
- `Links::Layout`
- `Links::Parent Link ID`

The SQL statement used:
- Select `"Layout", "Parent Link ID"` From `"Links"` Where `ID = ?`

Important notes:
- Field names are quoted, so the field names must match exactly.
- Table name is quoted as `"Links"`.

---

## SQL Query Details

### SQL

~~~filemaker
Let ( [
  sql = "Select \"Layout\", \"Parent Link ID\" From \"Links\" Where ID = ?" ;
  fieldseparator = "|" ;
  rowseparator = "¶" ;
  id = $$Link ID ;
  result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; id )
] ; result )
~~~

### Result Format

- Column separator: `|`
- Row separator: `¶`

So one returned row looks like:
- `Dashboard|1001`

This is converted into a list by replacing `|` with `¶` so it can be read with `GetValue()`.

---

## Detailed Logic Walkthrough

### 1. Capture the Link ID

- `$$Link ID = GetValue ( Get ( ScriptParameter ) ; 1 )`
- `$$Search Item = ""`

This ensures navigation resets any cross-page search state.

---

### 2. Load Link Metadata From Links Table

- The script queries the `Links` table for `"Layout"` and `"Parent Link ID"`.
- It loops over results, but it effectively uses only the returned row values to set:
  - `$$Layout Name`
  - `$$Parent Link ID`

Even if multiple rows were returned (not expected for a unique ID), the loop would end with the last row’s values. In normal use, Link ID should return exactly one row.

---

### 3. Modal Layout Handling

If current layout is `"Modal"`:
- Perform Script: `Close Window`
- Perform Script: `+++ Banner Page`
- Exit Script

Meaning:
- Modal windows are treated as temporary overlays.
- Clicking a link while on Modal closes it first, refreshes the banner, then stops.

This prevents navigation while a modal window remains open.

---

### 4. Close Window Logic (Non-Modal)

If not startup (`$$Startup ≠ 1`):
- If menu mode is NOT Persistent Side Menu:
  - Perform Script: `Close Window`

Meaning:
- In non-persistent menu mode, menu clicks may come from a separate menu window.
- This step closes it so the UI returns to normal after selection.

---

### 5. Always Refresh Banner

- Perform Script: `+++ Banner Page`

Banner refresh happens regardless of destination.
This keeps navigation consistent and ensures top UI reflects the new context.

---

### 6. Persistent Side Menu Sync

If `$$Menu Mode = "Persistent Side Menu"`:
- Perform Script: `Go To Menu`

Meaning:
- The side menu is persistent and needs to update selection/highlights or menu context.
- `Go To Menu` likely re-renders or refreshes the menu state.

---

### 7. Routing Logic (The Router)

This is the core of the script:
- A long `If / Else If` chain maps:
  - `$$Layout Name` → `Perform Script: “+++ <Destination Page>”`

Some routes depend on both:
- `$$Layout Name`
- `$$App Name`

Example:
- If `$$Layout Name = "Dashboard"` AND `$$App Name ≠ "The Bridge"` → `Go to Dashboard`
- If `$$App Name = "The Bridge"` AND `$$Layout Name = "Process Manager"` → `+++ Process Manager List Page`

This exists because some layout names are shared across apps, or some destinations are app-specific.

---

### 8. Fallback (Unknown Route)

If no route matches:
- `$$Page = ""`
- `Refresh Window [ Flush cached join results ; Flush cached external data ]`

Meaning:
- If the link’s configured layout name does not match a known destination, do not navigate.
- Reset page state and refresh caches to avoid stale UI issues.

---

## Routing Contract (Critical Rules)

### Rule 1: `Links::Layout` Must Match Router Keys Exactly

The router compares text:
- `If [ $$Layout Name = "Patients Database" ]`

So `Links::Layout` value must exactly match the expected string:
- Case-sensitive matching is recommended.
- Extra spaces will break routing.

### Rule 2: Destinations Are Scripts (Not Layout Names)

Even though the field is called `"Layout"`, it is really a **route key**.
It maps to a “page script” that handles page rendering.

Examples:
- `"Patients Database"` routes to `+++ Patients List Page`
- `"Payroll Periods"` routes to `+++ Payroll Periods List Page`

### Rule 3: Banner Refresh Happens Before Routing

Do not move banner refresh into destination scripts unless you intentionally want inconsistent behavior.
This script guarantees banner refresh across all navigation.

### Rule 4: Menu Mode Controls Window Behavior

- Non-persistent menus likely open in separate windows.
- Persistent menus stay on-screen and must be synced.

Therefore, new routes must not break:
- Modal close behavior
- Menu close behavior
- Banner refresh behavior
- Persistent menu sync

---

## How To Add a New Navigation (Step-by-Step)

This section is written as a procedure for an AI agent (and humans) to safely extend the router.

### Step A: Decide the Route Key

Pick a `Links::Layout` value that will represent the destination.

Example:
- `"Suppliers Database"`

This is the exact string that `$$Layout Name` will be compared against.

### Step B: Create or Confirm the Destination Page Script Exists

Destination scripts follow a common naming style:
- `+++ <Something> List Page`
- `+++ <Something> Page`
- `+++ <Something> Filter Page`
- Sometimes a non-+++ script exists (example: `Go to Dashboard`)

Create your new destination script first if it does not exist.

Example:
- `+++ Suppliers List Page`

### Step C: Add the Route to Select Link

Add a new `Else If` block in the routing section.

Example route:
~~~filemaker
Else If [ $$Layout Name = "Suppliers Database" ]
  Perform Script [ Specified: From list ; “+++ Suppliers List Page” ; Parameter:    ]
~~~

### Step D: Add/Update the Link Record in `Links`

Create a new record in `Links`:

- `Links::ID` = auto-generated
- `Links::Layout` = `Suppliers Database`
- `Links::Parent Link ID` = appropriate parent menu link ID

Ensure the menu item or UI element passes the Link ID into `Select Link`.

### Step E: Test in All Menu Modes

Test in:
- Modal context (open modal, click link)
- Non-persistent menu mode (menu window closes)
- Persistent side menu mode (menu stays, sync occurs)

Confirm:
- Banner updates
- Correct destination loads
- No extra windows remain open
- No routing falls into fallback

---

## Where To Insert New Routes (Ordering Rules)

This script uses a single `If / Else If` chain.
Order matters when conditions overlap.

### Overlap Example

Some conditions use OR:
- `If [ $$Layout Name = "Inpatients" or $$Layout Name = "Discharged Inpatients" ]`

If you add a new route that partially overlaps an existing condition, ensure:
- More specific conditions appear earlier.
- General catch-all combinations appear later.

### App-Specific Overrides

Routes for `$$App Name = "The Bridge"` should be placed near the end where other Bridge routes exist, unless they must override earlier generic routes.

---

## Naming Conventions Used

### Link Route Keys (Links::Layout)
- Human-readable titles (e.g., `"Patients Database"`, `"Leave Types"`, `"Lab Tests Setup"`)
- Often match menu item labels.
- Sometimes represent a category rather than a literal FileMaker layout name.

### Destination Scripts
- Typically start with `+++` to indicate page scripts.
- Suffix patterns:
  - `List Page`
  - `Page`
  - `Filter Page`
  - `Report Filter`

Follow existing patterns when adding new navigation.

---

## Common Patterns in Existing Routes

### One Key → One Page
~~~filemaker
Else If [ $$Layout Name = "Patients Database" ]
  Perform Script [ Specified: From list ; “+++ Patients List Page” ; Parameter:    ]
~~~

### Multiple Keys → Same Page
~~~filemaker
Else If [ $$Layout Name = "Preventive Maintenance" or $$Layout Name = "Maintenance" ]
  Perform Script [ Specified: From list ; “+++ PM Schedule Sheets List Page” ; Parameter:    ]
~~~

### Key + App Name Condition
~~~filemaker
Else If [ $$App Name = "The Bridge" and $$Layout Name = "Process Manager" ]
  Perform Script [ Specified: From list ; “+++ Process Manager List Page” ; Parameter:    ]
~~~

### Key + App Name Exception
~~~filemaker
If [ $$Layout Name = "Dashboard" and $$App Name ≠ "The Bridge" ]
  Perform Script [ Specified: From list ; “Go to Dashboard” ; Parameter:    ]
~~~

---

## Troubleshooting Guide

### Symptom: Clicking Link Does Nothing
Likely causes:
- `Links::Layout` value does not match any route key.
- Link ID passed in is empty or wrong.
- SQL query returned empty results.

Check:
- Is `$$Link ID` set?
- Does the `Links` table contain that ID?
- Does it have a `Layout` value?
- Does the router contain that exact string?

### Symptom: You Always Hit Fallback
Likely causes:
- Typos or extra spaces in the `Links::Layout` value.
- Route added to wrong place, never matched due to earlier condition.

### Symptom: Menu Window Stays Open
Likely causes:
- `$$Startup` incorrectly set to 1.
- `$$Menu Mode` set unexpectedly to `"Persistent Side Menu"`.

### Symptom: Banner Does Not Update
Likely causes:
- `+++ Banner Page` fails or exits early.
- Script exited due to Modal logic and did not proceed.

---

## AI Agent Extension Checklist (Must Follow)

When adding a new navigation route, the AI agent must:

1. Use the **Links table** and keep Link ID as parameter value 1.
2. Ensure the route key string is **exact** and consistent.
3. Add a matching `Else If` branch:
   - `$$Layout Name = "<Route Key>"`
4. Route to an existing destination script using `Perform Script`.
5. Preserve:
   - Modal handling section
   - Startup/menu close logic
   - Banner refresh call
   - Persistent side menu sync
   - Fallback behavior
6. Test at least three contexts:
   - Modal
   - Non-persistent menu mode
   - Persistent side menu mode

---

## Suggested Improvement Pattern (Optional Refactor Note)

This router is a large conditional chain. As it grows, consider:
- Moving route mappings into a table or JSON config.
- Returning a target script name and executing it dynamically.
- Centralizing app-specific overrides cleanly.

This is not required for current operation. The current script’s behavior is correct and intentionally explicit.

---

## Reference: Router Keys Already Present

The script already routes for keys including (partial examples):
- Dashboard (with app exception)
- Patients Database
- Consultation
- Prescriptions
- Lab Tests Setup
- Specimens Setup
- Vital Signs Monitoring
- Preventive Maintenance / Maintenance
- Purchase Requests
- Staff Profiles
- Social Media Posts
- Website
- The Bridge: Tasks, Process Manager

When adding new navigation, follow these established patterns.

---

# Script Documentation: App Section Router (Return To Previous App Page)

## Purpose

The name of the script is **Go To App Section**. This script is a centralized navigation router used by **CauferoAppStarter** to return the user to the correct **internal WebViewer app page** after the WebViewer has been used to load an **external website**.

In CauferoAppStarter, the UI is rendered inside a WebViewer. Sometimes that same WebViewer is used to open external websites. When this happens, the WebViewer’s browser history is not a reliable way to return to the correct internal app state. This script solves that problem by reloading the correct internal page based on the last known app navigation state.

This script is typically executed by a UI button such as:

- **Back**
- **Return**
- **Go Back To App**
- **Exit Website**
- **Resume App**

It uses global variables to determine which internal app page should be re-rendered and which record should be opened when returning to a details page.

---

## Core Concept

### Internal App Pages Versus External Websites

The WebViewer can display:

1. **Internal app pages** generated by your own WebViewer HTML rendering scripts (the “+++ … Page” scripts).
2. **External websites** loaded by URL navigation.

When the WebViewer is showing an external website, the app must use a controlled navigation mechanism to return to the last internal page. This script performs that controlled navigation.

---

## Inputs

This script does not take a script parameter. It depends entirely on global variables.

### Primary Routing Variable

- `$$App Section`

This global variable holds the name of the last internal app section/page that the user was on before leaving the app context (for example, to visit an external website).

### Record Context Variables

For details pages, the router also depends on a corresponding global ID variable, such as:

- `$$Patient ID`
- `$$Appointment ID`
- `$$Specimen ID`
- `$$Analyte ID`
- `$$Lab Test ID`
- `$$Equipment ID`
- `$$Leave Request ID`
- `$$Payroll Period ID`
- etc.

These IDs must already be set when the user was on that details page, so the router can return to the exact same record.

### Special Case Variable

- `$$App Name`

There is one special case in the script:

- `$$App Name = "The Bridge"` combined with `$$App Section = "Tasks"`

This supports routing differences when the same starter file is used across apps/modules.

---

## Outputs

The script produces no explicit result. Its effect is navigation.

It routes the user back to the correct internal page by running the appropriate “+++ … Page” script.

---

## Primary Responsibility

### What This Script Guarantees

When executed, this script will:

1. Inspect `$$App Section`.
2. Match it to a known internal app page key.
3. Run the corresponding page script.
4. Pass the required record ID (if the target is a details page).

This restores internal app UI state after leaving the app UI context inside the WebViewer.

---

## How Routing Works

### Router Pattern

This script is an `If / Else If` chain that maps:

- `$$App Section` (human-readable page key)
to:
- `Perform Script [ “+++ <Page Script>” ]` (the canonical page renderer)

There are two routing types:

#### 1) List Page Routing

List pages usually do not require a record ID.

Example routing shape:

~~~filemaker
Else If [ $$App Section = "Patients List" ]
    Perform Script [ Specified: From list ; “+++ Patients List Page” ; Parameter:    ]
End If
~~~

#### 2) Details Page Routing

Details pages require the ID of the selected record so the page script can render the correct record.

Example routing shape:

~~~filemaker
Else If [ $$App Section = "Patient Details" ]
    Perform Script [ Specified: From list ; “+++ Patient Details Page” ; Parameter: $$Patient ID ]
End If
~~~

---

## Usage Scenario

### Typical Flow: Opening an External Website

1. User is on an internal page (example: “Patient Details”).
2. App sets:
   - `$$App Section = "Patient Details"`
   - `$$Patient ID = <current record ID>`
3. User clicks a button that loads an external website in the WebViewer.
4. User clicks a Back button to return to the app.
5. Back button runs this router script.
6. Router script detects:
   - `$$App Section = "Patient Details"`
7. Router runs:
   - `+++ Patient Details Page` with parameter `$$Patient ID`
8. Internal WebViewer UI is restored to where the user was.

This avoids dependence on WebViewer browser history.

---

## Naming Convention and Contract

### Page Scripts Are Canonical

All destination scripts follow a consistent naming convention:

- `"+++ <Page Name> Page"`

This indicates a canonical internal page-rendering script.

### Page Keys Must Match Exactly

The router depends on exact string equality checks like:

- `$$App Section = "Patient Details"`

These strings must match whatever the app sets into `$$App Section`.

If the app sets a page key that does not exist in this router, the router cannot return to that page.

---

## Script Structure Summary

The script checks many possible sections. Each branch follows one of these patterns:

### Pattern A: Go To Dashboard

- Checks for `"Dashboard"`
- Runs `"Go to Dashboard"`

This appears as a special script name without the `+++` prefix, indicating a legacy or differently structured page entrypoint.

### Pattern B: List Page

- Checks for `"X List"`
- Runs `"+++ X List Page"`
- No parameter

### Pattern C: Details Page

- Checks for `"X Details"`
- Runs `"+++ X Details Page"`
- Parameter: `$$X ID`

### Pattern D: Role-Based / Portal Variants

Some page keys include portal identifiers such as:

- `"(Doctor's Portal) InPatient Details"`

These route to scripts like:

- `"+++ Doctor - InPatient Details Page"`

This allows the same record type to be rendered with a different UI, depending on portal context.

### Pattern E: App Name Conditional

One branch includes:

- `$$App Name = "The Bridge"` and `$$App Section = "Tasks"`

This supports app-specific routing within the shared starter framework.

---

## Dependencies and Preconditions

### Required Preconditions Before Calling This Router

Before navigating away to an external website, the app should:

1. Ensure `$$App Section` is set to the current internal page key.
2. Ensure any required record ID global is set (for details pages).

If these are missing, the router may:

- Route to the wrong page
- Open a details page with an empty parameter
- Fail to restore context correctly

### Details Pages Must Always Have Their ID Global Set

Example: If `$$App Section = "Specimen Details"` then `$$Specimen ID` must be non-empty.

If it is empty, the details page script may:
- Show an empty state
- Show the wrong record
- Break page logic depending on how the page script is implemented

---

## Error Handling Behavior

This script does not include explicit error handling. It assumes:

- `$$App Section` always contains a valid value.
- For details pages, the corresponding `$$<Entity> ID` is available.

If no condition matches, the script ends without routing anywhere.

---

## Maintenance Notes

### Adding a New Page

To support a new internal page in the router:

1. Choose the canonical page key string for `$$App Section`.
2. Create the internal page script: `"+++ <New Page> Page"`.
3. Add a new `Else If` branch to this router.
4. If it is a details page, ensure there is a corresponding global ID variable (example: `$$New Entity ID`) and pass it as the parameter.

Recommended branch pattern:

~~~filemaker
Else If [ $$App Section = "<New Page Key>" ]
    Perform Script [ Specified: From list ; “+++ <New Page Script>” ; Parameter: $$<Relevant ID> ]
End If
~~~

### Avoid Duplicates

This router contains some overlapping concepts (example: “KPIs List” and “KPIs Management” both route to the same page). That is allowed, but it must be intentional.

If you add a new page key that is semantically the same as an existing one, consider mapping it to the same page script instead of creating a separate page.

---

## AI Agent Rules For Understanding This Script

The AI Agent must interpret this script as:

1. A **navigation router**.
2. A **return-to-app mechanism** after leaving internal UI context inside a WebViewer.
3. A **state-restoration tool** driven by global variables.

### The AI Agent Must Learn These Behaviors

- `$$App Section` is the single source of truth for the last internal page.
- Details pages require an ID parameter. That ID comes from a corresponding global variable.
- This script does not build UI directly. It delegates UI rendering to page scripts.
- The “+++ … Page” scripts are the canonical entrypoints for each internal page.

### The AI Agent Must Not Infer Browser History

The AI Agent should not assume WebViewer back/forward behavior is used. This script replaces browser history for internal app navigation restoration.

---

## Script Snippet (Reference Shape Only)

This is the routing pattern repeated across the script:

~~~filemaker
If [ $$App Section = "<Section Key>" ]
    Perform Script [ “+++ <Page Script>” ; Parameter: <optional ID global> ]
Else If [ $$App Section = "<Another Section Key>" ]
    Perform Script [ “+++ <Page Script>” ; Parameter: <optional ID global> ]
...
End If
~~~

This confirms the script is a dispatcher and not a page renderer.

---

## Why This Script Exists In CauferoAppStarter

Because CauferoAppStarter uses WebViewer UI:

- Internal pages are not FileMaker layouts in the traditional sense.
- Navigation must be explicit and controlled.
- External websites disrupt internal WebViewer routing.

This router restores the internal app state reliably and consistently without relying on WebViewer browser navigation.

---

## Implementation Insight

This router is a standard pattern in CauferoAppStarter:

- One script sets `$$App Section` and the relevant ID globals.
- Another script may open an external website.
- This router returns the UI to the correct internal page by re-running the correct page script.

It is effectively the app’s internal routing table.

---

# Populate Available Links Script Documentation (CauferoAppStarter)

## Purpose

The **Populate Available Links** script generates the app’s **Available Links** payload, called **Link Records**, used to build navigation menus (sidebar, hamburger menu, tiles, quick links, and any page that lists modules and screens).

This script must output a single **text blob** that contains many link records. Each link record describes either:
- A **Section Header** (top level group), or
- A **Link Item** (a clickable screen under a section)

The output follows a strict delimiter contract so downstream scripts and WebViewer rendering can parse it deterministically.

This document defines the canonical structure and rules an AI Agent must follow to generate link records correctly.

---

## Where This Script Fits In The App

### Typical Flow

1. User logs in or app loads a module selector.
2. App determines the user’s role, permissions, or enabled modules.
3. **Populate Available Links** generates the navigation payload based on these permissions.
4. UI scripts render menus using the payload (often inside WebViewer HTML/JS).

### Why The Contract Matters

Many scripts depend on the exact separators and field positions. If the payload shape changes, UI rendering and click actions break.

Hard rule: **Keep the delimiter contract stable. Change values only.**

---

## Output Data Contract (Link Records)

### Record Separator

- Each link record is one line.
- Records are separated by the paragraph mark delimiter **¶**.

Notes:
- Multiple consecutive ¶ may exist to visually separate groups.
- Empty records (blank lines) must be ignored by parsers.

### Field Separator

- Fields within each record are separated by a pipe **|**.

### Canonical Field Order (Version 1)

Each record has 5 required fields, plus up to 2 optional fields.

| Position | Field Name | Required | Description |
|---:|---|:---:|---|
| 1 | LinkID | Yes | Unique ID for this link record (UUID recommended) |
| 2 | OrderKey | Yes | Sort and hierarchy code (examples: `3`, `3.1`, `6.2`) |
| 3 | Caption | Yes | Display title in the menu (short label) |
| 4 | Subtitle | Yes | Secondary label or expanded title (may be empty) |
| 5 | IconPath | Yes | SVG path string (the `M...` path data) |
| 6 | ParentLinkID | No | The LinkID of the section header this item belongs to |
| 7 | Flags | No | UI directives (example: `After Section Divider`) |

### Record Shape Rules

#### Section Header Record (Top Level)
- ParentLinkID is empty.
- Flags may exist.

Canonical shapes:
- `LinkID|OrderKey|Caption|Subtitle|IconPath¶`
- `LinkID|OrderKey|Caption|Subtitle|IconPath||After Section Divider¶`

#### Link Item Record (Child Under Section)
- ParentLinkID is populated with the section’s LinkID.
- Flags may exist.

Canonical shapes:
- `LinkID|OrderKey|Caption|Subtitle|IconPath|ParentLinkID¶`
- `LinkID|OrderKey|Caption|Subtitle|IconPath|ParentLinkID|After Section Divider¶`

---

## Hierarchy Rules

### Parent-Child Relationship

- A record becomes a child when **ParentLinkID** is present and matches a section’s LinkID.
- A record becomes a section when **ParentLinkID** is empty.

### OrderKey Meaning

OrderKey controls display order and implies hierarchy.

Examples:
- `2` is a section
- `2.1`, `2.2`, `2.3` are children under section `2`
- `6` is a section
- `6.1`, `6.2`, `6.3` are children under section `6`

Hard rule: The OrderKey must be consistent with ParentLinkID grouping.
- If a record has ParentLinkID pointing to section `2`, its OrderKey should start with `2.`

---

## Flags Rules

### Supported Flag: After Section Divider

- `After Section Divider` is a UI directive used to insert a divider line after a section group in navigation.

Placement:
- Typically placed on the **Section Header** record using field 7.
- If placed on a child record, UI behavior becomes ambiguous and should be avoided unless explicitly required.

Hard rule: Use flags only from an approved list. Do not invent new flags inside this script.

Approved flag list:
- `After Section Divider`

---

## IconPath Rules (SVG Path Data)

- IconPath is a string containing SVG `path d` instructions.
- It must not contain the record separator ¶.
- It must not contain unescaped pipe `|`.

If a pipe is needed (rare), it must be replaced prior to payload generation or avoided by choosing a different icon path.

---

## Script Inputs and Outputs

### Inputs (Typical)

The script usually depends on:

- Current User ID (or logged in staff record)
- Current User Role(s)
- Enabled Modules
- Permission list
- Optional: device context (desktop vs mobile) if link availability differs

These values can be sourced from:
- $$ variables (session state)
- fields on current user record
- a permissions table
- script parameters

Hard rule: If the script uses Script Parameter, document the exact JSON or delimited structure and keep it stable.

### Output

The script returns:
- `$LinkRecords` (text) containing all link records

Typical end step:
- `Exit Script [ Text Result: $LinkRecords ]`

---

## Generation Logic

This section defines how the AI Agent must generate link records.

### Step 1: Determine Which Links Are Available

Build a list of allowed sections and allowed items based on user permissions.

Core rule:
- If a section has zero allowed children, the section should not appear, unless a product rule says section headers can exist without children.

### Step 2: Build Section Header Records

For each allowed section:
- Create a LinkID (UUID)
- Assign OrderKey (example: `3`)
- Assign Caption (example: `Staff Management`)
- Subtitle can be empty for sections
- Assign IconPath for the section icon
- ParentLinkID must be empty
- Optional: add Flags, such as `After Section Divider`

### Step 3: Build Child Link Item Records

For each allowed child:
- Create a LinkID (UUID)
- Assign OrderKey (example: `3.1`)
- Assign Caption (menu label)
- Subtitle (expanded label, often same as caption)
- IconPath for the item icon
- ParentLinkID must equal the LinkID of the parent section
- Optional: flags only if explicitly required

### Step 4: Sort Records

Sort order is based on OrderKey.

Recommended approach:
- Sort sections by numeric value of OrderKey where possible.
- Sort children under each section by numeric value of the fractional part.
- Preserve the final output order exactly as sorted.

### Step 5: Join Into One Payload

Join each record line with ¶.

Hard rules:
- Every record must end with ¶, including the last record.
- Do not add extra spaces around pipes.
- Keep empty Subtitle as empty field, not a literal word like `null`.

---

## Validation Rules (Must Pass Before Exit Script)

The script must validate the output before returning it.

### Record-level Validation

For each non-empty record line:
- It must contain at least 5 pipe-separated fields.
- Field 1 (LinkID) must be non-empty.
- Field 2 (OrderKey) must be non-empty.
- Field 3 (Caption) must be non-empty.
- Field 5 (IconPath) must be non-empty.

### UUID Shape Validation (Recommended)

LinkID and ParentLinkID should be UUIDs.

Recommended shape:
- 36 characters with hyphens: `8-4-4-4-12`

If legacy IDs exist, document the acceptable formats. Do not mix random malformed IDs in generated output.

### Parent Validation

If ParentLinkID is present:
- It must match a section LinkID in the same payload.

### OrderKey Consistency

If ParentLinkID points to section `X`, the child OrderKey should start with `X.`

Example:
- Parent is `2` section
- Child must be `2.1`, `2.2`, etc

### Flag Validation

If field 7 exists:
- It must be one of the approved flags.

---

## Common Mistakes And What To Avoid

### Mistake: Inconsistent numbering vs parent

Bad:
- Parent section OrderKey is `2`
- Child has ParentLinkID pointing to section `2`
- Child OrderKey is `3.4`

This creates incorrect grouping and sorting.

### Mistake: Malformed IDs

Bad:
- LinkID missing segments
- LinkID contains extra characters
- ParentLinkID cannot match any section due to typos

Result: UI cannot resolve parent, click routing fails.

### Mistake: Pipes inside IconPath

Bad:
- IconPath contains `|`

Result: parser splits fields incorrectly.

### Mistake: Changing the output contract

Bad:
- Adding new fields
- Removing fields
- Renaming fields
- Changing separators

Result: dependent scripts break.

---

## Parsing Guidance (How Downstream Code Must Read It)

This section exists so an AI Agent understands why the output must be stable.

### Parser Expectations

Downstream parsers typically do:

1. Split by ¶ to get record lines
2. Ignore empty lines
3. Split each line by `|`
4. Read fields by index:
   - [1] LinkID
   - [2] OrderKey
   - [3] Caption
   - [4] Subtitle
   - [5] IconPath
   - [6] ParentLinkID (optional)
   - [7] Flags (optional)

Hard rule: Always preserve the first 5 fields.

---

## Canonical Examples

### Example A: One Section With Two Items

This example shows the canonical shape. IDs are illustrative.

~~~text
7bd28a51-3f49-4f7c-b8c5-13e7c3eae4f8|2|My Stuff||M4 4h5l3 3h8v12H4z¶
a7e6f9c1-4c38-4692-a1f3-25d4b6e7a3c9|2.1|My Profile|My Profile|M20,19a1.5,1.5,0,0,1-1.5,1.5H5.5A1.5,1.5,0,0,1,4,19a5,5,0,0,1,5-5h6A5,5,0,0,1,20,19Zm-8-7A4,4,0,1,0,8,8,4,4,0,0,0,12,12Z|7bd28a51-3f49-4f7c-b8c5-13e7c3eae4f8¶
b9f3d4a7-1e62-4a6c-b8f1-d7c25e93a4c7|2.2|My Leave Requests|My Leave Requests|M14 1v3h-3v-3h-6v3h-3v-3h-2v15h16v-15h-2z|7bd28a51-3f49-4f7c-b8c5-13e7c3eae4f8¶
~~~

### Example B: Section Divider Flag

~~~text
b9f4d2a7-6c3e-4a8f-a1b5-7e9c13d6f2a8|3|Company||M2 5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v11a2 2 0 0 1-2 2h-7v2h3a1 1 0 1 1 0 2H8a1 1 0 1 1 0-2h3v-2H4a2 2 0 0 1-2-2V5zm18 11V5H4v11h16z||After Section Divider¶
~~~

---

## FileMaker Implementation Specification

This section defines how the script should be implemented in FileMaker so an AI Agent can generate it.

### Recommended Script Outline

1. Set context variables (current user, roles, permissions).
2. Query allowed sections and items.
3. Generate records into `$LinkRecords`.
4. Validate.
5. Exit Script with `$LinkRecords`.

### Data Source Options

The script can generate links from:

Option 1: Static configuration (hardcoded in script)
- Fast to start
- Harder to maintain across many apps

Option 2: Configuration tables (recommended)
- A `Links` table that stores all potential links and metadata
- A `Permissions` table mapping roles to links
- The script queries allowed links

Hard rule: This documentation assumes a configuration-driven approach unless your app explicitly uses hardcoded links.

---

## SQL Contract (If Using ExecuteSQL)

If the script uses ExecuteSQL, use a stable select list.

### Suggested SQL Output Fields (Ordered)

The SQL query should return fields in this exact order:

1. LinkID
2. OrderKey
3. Caption
4. Subtitle
5. IconPath
6. ParentLinkID
7. Flags

Return delimiter rule:
- Each row returned as a single text line
- Columns separated by `|`
- Rows separated by ¶

### Example SQL Pseudocode

~~~filemaker
/* Example only: table and field names must match your file */
Let ( [
  sql =
    "SELECT
       Links.ID,
       Links.OrderKey,
       Links.Caption,
       Links.Subtitle,
       Links.IconPath,
       Links.ParentID,
       Links.Flags
     FROM Links
     WHERE Links.IsActive = 1
       AND Links.ID IN ( /* permitted link ids */ )
     ORDER BY Links.OrderKey" ;
  result = ExecuteSQL ( sql ; "|" ; "¶" )
] ;
result
)
~~~

Hard rule: If Flags is empty for most records, still include the column so the output shape stays stable.

---

## Record Assembly Rules (If Building In A Loop)

When generating records in a loop, assemble each line using strict concatenation.

### Canonical Assembly Pattern

- `$Line = LinkID & "|" & OrderKey & "|" & Caption & "|" & Subtitle & "|" & IconPath`
- If ParentLinkID exists, append `|` & ParentLinkID
- If Flags exists, append `|` & Flags
- Append to `$LinkRecords` using ¶ separators.

### Example Loop Pseudocode

~~~filemaker
/* Loop example: assumes you have current row values in $ID, $OrderKey, $Caption, $Subtitle, $IconPath, $ParentID, $Flags */

Set Variable [ $Line ;
  Value:
    $ID & "|" &
    $OrderKey & "|" &
    $Caption & "|" &
    $Subtitle & "|" &
    $IconPath
]

If [ not IsEmpty ( $ParentID ) ]
  Set Variable [ $Line ; Value: $Line & "|" & $ParentID ]
End If

If [ not IsEmpty ( $Flags ) ]
  /* Ensure Parent field exists if Flags exists */
  If [ IsEmpty ( $ParentID ) ]
    Set Variable [ $Line ; Value: $Line & "|" ]
  End If
  Set Variable [ $Line ; Value: $Line & "|" & $Flags ]
End If

Set Variable [ $LinkRecords ;
  Value:
    $LinkRecords &
    If ( IsEmpty ( $LinkRecords ) ; "" ; "¶" ) &
    $Line
]

/* After loop ends */
Set Variable [ $LinkRecords ; Value: $LinkRecords & "¶" ]
~~~

Hard rule: Final payload must end with ¶.

---

## Required Downstream Behaviors (For Compatibility)

Any UI renderer that consumes Link Records typically does:

- Split records by ¶
- Build sections first
- Map children by ParentLinkID
- Render sections in OrderKey order
- Render children under their section in OrderKey order
- Apply divider where flag says `After Section Divider`

This means:
- Section LinkIDs must exist before child records can correctly attach.
- ParentLinkID must be correct.

---

## Testing Checklist

Run these checks on the output:

1. Output is not empty for a user with permissions.
2. Every non-empty record has at least 5 fields.
3. All LinkIDs are unique.
4. All ParentLinkIDs match a section LinkID.
5. Children are grouped under the correct section.
6. OrderKey sorting results in expected order.
7. Divider appears only where expected.
8. No record contains an unescaped `|` inside IconPath.
9. Payload ends with ¶.

---

## Ambiguities And Clarifications Needed

The AI Agent needs these points clarified to generate a perfect script for your CauferoAppStarter file.

1. What is the authoritative data source for links in CauferoAppStarter?
   - A Links configuration table
   - A hardcoded list inside the script
   - A mixed approach

2. What is the exact name of the script output variable?
   - `$LinkRecords`
   - `$AvailableLinks`
   - `$GeneratedList`
   - Something else

3. What is the expected return behavior?
   - Exit Script with Text Result
   - Set a global variable like `$$AvailableLinks`
   - Write to a field on a settings record

4. Are empty group separator blank lines allowed in final output?
   - Allowed for readability
   - Disallowed to keep payload compact

5. Is OrderKey always numeric-like (example `6.2`) or can it contain letters (example `6A`)?

6. Are there additional flags beyond `After Section Divider` that CauferoAppStarter already supports?

7. How should the script handle a section with zero allowed children?
   - Omit the section
   - Include the section anyway

8. Does the app require subtitles for child items to always match captions, or do some items use different subtitles?

9. Do you require strict UUID validation, or can LinkID be any unique string?

10. Are there app-specific routing fields missing from this contract?
   - Example: layout name, webviewer page key, script name to call, URL, module code
   If those exist, they must be included in a separate stable contract, or the menu click handler must infer routing from LinkID.

---

## Definition Of Done

This documentation is considered implemented when:

- The Populate Available Links script returns Link Records that conform to the contract above.
- The navigation UI renders correctly for multiple roles.
- No dependent scripts require changes due to output shape drift.

---

# ExecuteSQL Patterns in CauferoAppStarter
## Generate ExecuteSQL Queries and Save Returned Results in Variables

## Purpose
This document defines the canonical way CauferoAppStarter scripts:
1. build an `ExecuteSQL()` query that matches the goal of the script,
2. run it safely with parameters,
3. store the raw result in a variable,
4. parse the result into individual variables for use in UI, logic, and WebViewer payloads.

The AI Agent must generate scripts that follow these formatting and parsing rules exactly.

---

## Hard Rules (Non Negotiable)

### 1. Always Quote All Table Names and Field Names
All table names and field names in the SQL string must be wrapped in escaped quotes: `\"...\"`.

Examples:
- Table: `\"Departments\"`
- Field: `t1.\"Department ID\"`
- ID field: `t1.\"ID\"`

Hard rule:
- Quote everything, even if there are no spaces.

### 2. Reserved SQL Syntax Words Must Be Uppercase
Examples:
- `SELECT`, `FROM`, `LEFT JOIN`, `INNER JOIN`, `WHERE`, `AND`, `OR`, `GROUP BY`, `ORDER BY`, `HAVING`, `AS`

Hard rule:
- Do not lowercase these keywords.

### 3. SQL Clause Spacing and Arrangement Must Be Predictable
The SQL string must be arranged with clear spacing between major sections:
- Space between selected fields and tables: between `SELECT ...` and `FROM ...`
- Space between tables and conditions: between `FROM ...` and `WHERE ...`
- Space between `WHERE ...` and later clauses like `GROUP BY`, `HAVING`, `ORDER BY`

Hard rule:
- There must be at least one whitespace character separating each major clause boundary: `SELECT ... FROM ... WHERE ...`
- The final SQL must read cleanly with single spaces separating major clauses.

### 4. GROUP BY and ORDER BY Spacing Rule
Special rule:
- `GROUP BY` and `ORDER BY` do not require an extra blank line or “visual spacing separation” between them in the SQL string layout.
- They can be adjacent clauses in the same concatenated segment, or in consecutive segments, without inserting an extra “spacer segment”.

Hard rule:
- Do not insert any extra spacing-only segments between `GROUP BY` and `ORDER BY`.
- Normal clause separation still applies: the end of the `GROUP BY` clause must include a space (or the next clause must begin with a leading space) so the SQL remains valid.

### 5. The Query Must Match the Goal
Conditions could be anything. The query structure must be based on what it is trying to achieve.
The only hard rules are the formatting rules above and the Let pattern below.

### 6. Let Pattern Must Match CauferoAppStarter Style
All ExecuteSQL blocks must follow the Let pattern used in your examples:
- `sql = "..." ;`
- `fieldseparator = "|" ;`
- `rowseparator = "¶" ;`
- parameters assigned as Let variables
- `result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; ...params... )`
- return `result`

---

## FileMaker ExecuteSQL Contract

### Function Signature
`ExecuteSQL ( sql ; fieldSeparator ; rowSeparator ; parameter1 ; parameter2 ; ... )`

### Separators Contract
- `fieldseparator = "|"`
- `rowseparator = "¶"`

Return format:
- Each row separated by `¶`
- Each column separated by `|`

---

## Canonical Script Flow
1. Run `ExecuteSQL()` inside a `Let()` block.
2. Save the returned result into `$List`.
3. Compute `$Total = ValueCount ( $List )`.
4. If `$Total > 0`, loop through rows and parse into variables.
5. Else, set defaults (if needed).

Canonical shell:
~~~filemaker
Set Variable [ $List ; Value: Let ( 
[
  sql = "SELECT ... FROM ... WHERE ... = ?" ;
  fieldseparator = "|" ;
  rowseparator = "¶" ;
  p1 = $$Some ID ;
  result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; p1 )
] ;
result
) ]

Set Variable [ $Total ; Value: ValueCount ( $List ) ]

If [ $Total > 0 ]
  Set Variable [ $i ; Value: 0 ]
  Loop [ Flush: Always ]
    Set Variable [ $i ; Value: $i + 1 ]
    Set Variable [ $Row ; Value: GetValue ( $List ; $i ) ]
    Set Variable [ $RowAsList ; Value: Substitute ( $Row ; "|" ; ¶ ) ]

    /* Map columns to variables here */

    Exit Loop If [ $i ≥ $Total ]
  End Loop
Else
  /* Defaults here */
End If
~~~

Hard rule:
- Always initialize `$i` to 0 before the loop.

---

## SQL Formatting Template (Mandatory Style)

### Recommended Clause Layout
Build the SQL string in segments so the final output always has correct spacing for:
- `SELECT` to `FROM`
- `FROM` to `WHERE`
- `WHERE` to subsequent clauses

At the same time, do not insert any extra spacing-only segments between `GROUP BY` and `ORDER BY`.

~~~filemaker
sql =
  "SELECT " &
    "t1.\"Field 1\", " &
    "t1.\"Field 2\", " &
    "t2.\"Field 3\" " &

  "FROM \"Table 1\" t1 " &
  "LEFT JOIN \"Table 2\" t2 ON t1.\"Link Field\" = t2.\"ID\" " &

  "WHERE t1.\"ID\" = ? " &

  "GROUP BY t1.\"Field 1\", t1.\"Field 2\", t2.\"Field 3\" " &
  "ORDER BY t1.\"Field 1\" ASC"
~~~

Hard rules enforced by this template:
- Keywords uppercase
- Table and field names quoted with `\"...\"`
- Major clause spacing is clean
- No extra spacer segments between `GROUP BY` and `ORDER BY`

---

## Conditions and Parameters (Flexible, Goal Driven)

### WHERE Conditions Can Be Anything
The AI Agent can generate any valid condition set based on the goal, for example:
- Single ID filter
- Multi condition filters with `AND` and `OR`
- Date range filters
- Status filters
- Search text filters
- Null checks

Examples (all valid patterns):

#### A. Single parameter
~~~sql
WHERE t1.\"ID\" = ?
~~~

#### B. Multiple parameters
~~~sql
WHERE t1.\"Department ID\" = ? AND t1.\"Status\" = ?
~~~

#### C. OR logic
~~~sql
WHERE t1.\"Status\" = ? OR t1.\"Status\" = ?
~~~

#### D. Date range
~~~sql
WHERE t1.\"Date Created\" >= ? AND t1.\"Date Created\" <= ?
~~~

#### E. Search style filter
~~~sql
WHERE t1.\"Name\" LIKE ?
~~~

### Parameter Binding Rules
Hard rules:
- Every `?` must have a matching parameter passed into `ExecuteSQL()`, in the exact order the `?` placeholders appear.
- Never inject values directly into the SQL string.

Example with multiple parameters:
~~~filemaker
Set Variable [ $List ; Value: Let ( 
[
  sql =
    "SELECT " &
      "t1.\"ID\", " &
      "t1.\"Name\" " &
    "FROM \"Departments\" t1 " &
    "WHERE t1.\"Status\" = ? AND t1.\"Date Created\" >= ? " &
    "ORDER BY t1.\"Name\" ASC" ;

  fieldseparator = "|" ;
  rowseparator = "¶" ;

  pStatus = $$Status ;
  pStartDate = $$Start Date ;

  result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; pStatus ; pStartDate )
] ;
result
) ]
~~~

---

## Parsing Rules

### Row and Column Parsing
1. Get row N:
- `$Row = GetValue ( $List ; $i )`

2. Convert `|` into returns so each column becomes a line:
- `$RowAsList = Substitute ( $Row ; "|" ; ¶ )`

3. Extract columns by position:
- `GetValue ( $RowAsList ; 1 )`
- `GetValue ( $RowAsList ; 2 )`
- etc.

Hard rule:
- Column order is controlled only by the `SELECT` order.

### Description Field Cleanup (Always)
Hard rule:
- Every Description field must be converted consistently:

~~~filemaker
Set Variable [ $Description ; Value: ConvertLineBreakMarkersToReturns ( GetValue ( $RowAsList ; N ) ) ]
~~~

---

## Aggregate Rules (COUNT, DISTINCT, GROUP BY)

### When to Use COUNT(DISTINCT)
Use `COUNT(DISTINCT ...)` when you want a unique entity count and joins can multiply rows.

Example:
~~~sql
COUNT(DISTINCT t4.\"Staff ID\") AS TotalStaff
~~~

### When to Use COUNT (Non Distinct)
Use `COUNT(...)` when you want row counts (events, notches, logs, line items).

Example:
~~~sql
COUNT(t4.\"ID\") AS TotalNotches
~~~

### GROUP BY Rule
If `SELECT` contains aggregates, every non aggregate selected field must appear in `GROUP BY`.

---

## Date Handling Rule
Dates are stored as FileMaker dates, then returned by ExecuteSQL as MySQL style date text.

Hard rule:
- Convert when you need FileMaker date logic.
- Keep SQL text when you need `YYYY-MM-DD` as text.

Canonical pattern:
~~~filemaker
Set Variable [ $Sql Some Date ; Value: GetValue ( $RowAsList ; N ) ]
Set Variable [ $Some Date ; Value: MySQLDateToFMPDateText ( $Sql Some Date ) ]
~~~

---

## Canonical Example Snippet (Matches All Hard Rules)
This snippet demonstrates the formatting rules clearly, including `GROUP BY` followed immediately by `ORDER BY` with no extra spacing-only segments.

~~~filemaker
Set Variable [ $List ; Value: Let ( 
[
  sql =
    "SELECT " &
      "t1.\"Name\", " &
      "t1.\"Description\", " &
      "COUNT(DISTINCT t2.\"ID\") AS TotalRelated " &

    "FROM \"Departments\" t1 " &
    "LEFT JOIN \"Staff\" t2 ON t1.\"HOD Staff ID\" = t2.\"ID\" " &

    "WHERE t1.\"ID\" = ? " &

    "GROUP BY t1.\"Name\", t1.\"Description\" " &
    "ORDER BY t1.\"Name\" ASC" ;

  fieldseparator = "|" ;
  rowseparator = "¶" ;

  pID = $$Department ID ;

  result = ExecuteSQL ( sql ; fieldseparator ; rowseparator ; pID )
] ;
result
) ]

Set Variable [ $Total ; Value: ValueCount ( $List ) ]

If [ $Total > 0 ]
  Set Variable [ $i ; Value: 0 ]
  Loop [ Flush: Always ]
    Set Variable [ $i ; Value: $i + 1 ]
    Set Variable [ $Row ; Value: GetValue ( $List ; $i ) ]
    Set Variable [ $RowAsList ; Value: Substitute ( $Row ; "|" ; ¶ ) ]

    Set Variable [ $Name ; Value: GetValue ( $RowAsList ; 1 ) ]
    Set Variable [ $Description ; Value: ConvertLineBreakMarkersToReturns ( GetValue ( $RowAsList ; 2 ) ) ]
    Set Variable [ $Total Related ; Value: GetValue ( $RowAsList ; 3 ) ]

    Exit Loop If [ $i ≥ $Total ]
  End Loop
Else
  /* Defaults if needed */
End If
~~~

---

## Agent Output Checklist
Before finalizing any generated ExecuteSQL script, the AI Agent must verify:
- Every table name is wrapped in `\"...\"`
- Every field name is wrapped in `\"...\"`
- All SQL keywords are uppercase
- SQL has clean clause spacing between `SELECT`, `FROM`, `WHERE`, and later clauses
- `GROUP BY` and `ORDER BY` are consecutive without any extra spacing-only segments
- SQL is built using the Let pattern used in CauferoAppStarter
- `$i` is initialized to 0 before looping
- Every `?` has a bound parameter in the correct order
- Every Description field is assigned using `ConvertLineBreakMarkersToReturns()`
- Date conversion is applied only when FileMaker date logic is needed
