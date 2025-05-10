# 📊 Informatica SCD Implementation Workflow
This project demonstrates three approaches for handling Slowly Changing Dimensions (SCD) in Informatica PowerCenter, processing employee data sourced from an HR system.

# 🔁 SCD Types Covered
SCD Type 1: Overwrites existing data with current values (no history)

SCD Type 2: Maintains full historical changes using versioning

SCD Type 3: Maintains limited history (e.g., current and previous values)

# 🛠️ Workflow Overview
Source System
Table: HR.EMPLOYEES (Oracle)

Method: Full extract using Source Qualifier

Key Field: EMPLOYEE_ID

Data Fields: FIRST_NAME, LAST_NAME, JOB_ID, SALARY, DEPARTMENT_ID, etc.

# 🔹 SCD Type 1 (Mapping: M_EMP_SCD_T1)
# ![M_SCD_T1](https://raw.githubusercontent.com/AhmedReda-7/SCD_Informatica/main/M_SCD_T1.png)
Workflow:

scss
Copy
Edit
[Source] → [SQ_EMPLOYEES] → [LKP_EMP_SCD_T1] → [EXP_EMP_SCD_T1] → [RTR_EMP_SCD_T1_INS_UPD]
              │                                       │
              │                                       ├──→ [EMPLOYEES_SCD_T1_INSERT] (new records)
              └──→ [EMPLOYEES_SCD_T1_UPDATE] (existing records)
Logic:

Lookup checks if the employee exists in the target.

Router splits into:

Insert: New employees

Update: Changes in JOB_ID, SALARY, or DEPARTMENT_ID

Existing data is overwritten (no history retained).

# 🔸 SCD Type 2 (Mapping: M_EMP_SCD_T2)
# ![M_SCD_T2](https://raw.githubusercontent.com/AhmedReda-7/SCD_Informatica/main/M_SCD_T2.png)

Workflow:

css
Copy
Edit
[Source] → [SQ_EMPLOYEES] → [SEQ_EMP_SCD_T2_SURR_KEY] → [LKP_EMP_SCD_T2] → [EXP_EMP_SCD_T2] → [RTR_EMP_SCD_T2]
              │                                                              │
              │                                                              ├──→ [EXP_EMP_SCD_T2_INS] → [EMPLOYEES_SCD_T2_INSERT]
              │                                                              └──→ [EXP_EMP_SCD_T2_UPD] → [UPD_EMP_SCD_T2] → [EMPLOYEES_SCD_T2_UPDATE]
Logic:

Generates surrogate keys using a Sequence Generator

Lookup checks current record where CURRENT_FLAG = 1

Router splits into:

Insert: New version with:

New surrogate key

CURRENT_FLAG = 1

START_DATE = SYSDATE

Update: Expire current record by:

Setting END_DATE = SYSDATE - 1

CURRENT_FLAG = 0

Complete history maintained.

# 🔹 SCD Type 3 (Mapping: M_EMP_SCD_T3)
# ![M_SCD_T3](https://raw.githubusercontent.com/AhmedReda-7/SCD_Informatica/main/M_SCD_T3.png)

Workflow:

scss
Copy
Edit
[Source] → [SQ_EMPLOYEES] → [LKPTRANS] → [RTRTRANS]
              │                        │
              │                        ├──→ [EMPLOYEES_SCD_T3_INS] (new records)
              │                        └──→ [UPDTRANS] → [EMPLOYEES_SCD_T3_UPD] (updates)
Logic:

Lookup fetches current record

Router splits into:

Insert: For new employees

Update: If JOB_ID changes

Move current JOB_ID to PREV_JOB

Update CURRENT_JOB with new value

Maintains limited history (current + previous job only)

# 🔄 Execution Flow
Initial Load
Full Extract from source

SCD Type 1: Inserts all rows

SCD Type 2: Inserts with CURRENT_FLAG = 1

SCD Type 3: Inserts with PREV_JOB = NULL

# Incremental Load
SCD Type 1: Overwrites changed values in place

SCD Type 2: Expires old version, inserts new version

SCD Type 3: Updates CURRENT_JOB, stores old in PREV_JOB

# ⚠️ Error Handling
Default Router paths handle unchanged records

Update Strategy transformations manage constraints and DML logic

# 🧰 Key Components
Component	Purpose
Lookup	Identify existing records
Router	Route records based on change detection
Expression	Derive values, flags, and format fields
Update Strategy	Define insert/update logic
Sequence Generator	Create surrogate keys (used in SCD Type 2)

# 📁 Folder Structure
pgsql
Copy
Edit
/SCD-Implementation
├── mappings/
│   ├── M_EMP_SCD_T1.xml
│   ├── M_EMP_SCD_T2.xml
│   └── M_EMP_SCD_T3.xml
├── README.md
└── docs/
    └── scd-diagram.png
