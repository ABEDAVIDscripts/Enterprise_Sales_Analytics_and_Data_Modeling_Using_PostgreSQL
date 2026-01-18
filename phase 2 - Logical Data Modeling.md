## Phase 2: Logical Data Modeling

### Modeling Philosophy
This phase emphasizes that a production database is more than stored data. <br>
The database was designed around clear structure, enforced rules, and explicit relationships. <br>

**Database = Structure + Rules + Relationships** 

<br>


#### ETL Implementation (SQL-Based)
- Extract: Raw CSV data was ingested into staging tables to preserve source fidelity and enable validation.
- Transform: Data was cleansed and reshaped during SQL-based inserts, including data type casting, deduplication, and mapping of textual attributes to surrogate keys.
- Load: Transformed data was loaded into a normalized core schema with enforced foreign key relationships.
- Validate: Post-load verification ensured record-level consistency between staging and core schemas.

<br>

#### Core Schema Data Modeling Principles
- Normalized to Third Normal Form (3NF)
- Textual attributes centralized to eliminate duplication
- No calculated or aggregated values stored
- Referential integrity enforced using foreign keys
- Business rules enforced at the database level


<br>


#### Entity Relationship Diagram (ERD)
An Entity Relationship Diagram (ERD) was created to validate and communicate the logical model. <br>
The following entities form the foundation of the core schema : *customers, country, channels, products, sales, salesitems, campaigns, stock*. 

<br>

**Diagram** <br>
<img height="450" alt="er diagram" src="https://github.com/user-attachments/assets/68c5d42f-baef-41d9-a505-877eb0209fed" />



<br>
<br>