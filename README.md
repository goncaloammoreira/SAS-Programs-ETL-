# SAS-Programs-ETL-

SAP Spend & Accounting Mill SAS Script

**Overview**

This repository contains SAS scripts to process and consolidate SAP data, including:

Concatenation of multiple rows from the ZTEKPO table into aggregated string fields.

Creation of the main tables (sap_spend and/or sap_accounting_mill) by joining VBFA, EKPO, EKKO, VBAP, EKET, EKBE, VBAK, LFA1, BSEG/BKPF, and the aggregated ZTEKPO data.

Aggregation of numeric and financial fields with conditional sums and max operations.

**Structure**

Macros:

formatCol(myCol): formats numeric identifiers with CATS(INPUT(...,10.)).

formatDat(myCol): parses date fields with input(..., yymmdd8.).

ZTEKPO concatenation:

PROC SQL creates ztekpo_concat ordered by EBELN, EBELP, ZZCOMPOS.

DATA step (ztekpo_final) concatenates multiple rows per EBELN+EBELP into string fields.

Main table creation (sap_spend or sap_accounting_mill):

PROC SQL with multiple LEFT JOINs to source tables.

GROUP BY keys (e.g., VBELV, POSNR, optional other keys).

Aggregation functions (MAX, SUM, MIN) with conditional logic.

Replacement of missing or "." values (e.g., s_soItem) handled after aggregation in a DATA step when needed.

Cleanup:

PROC DATASETS to delete temporary tables (ztekpo_concat, ztekpo_final).

**Usage**

Update library references (e.g., LIBNAME hana or LIBNAME sourcing) to match your environment.

Run the ZTEKPO concatenation steps:

PROC SQL;
  CREATE TABLE ztekpo_concat AS ...;
QUIT;

DATA ztekpo_final;
  SET ztekpo_concat;
  ...;
RUN;

Run the main PROC SQL to create sap_spend or sap_accounting_mill:

PROC SQL;
  CREATE TABLE work.sap_spend AS
  SELECT ...
  FROM hana.VBFA AS vbfa
  LEFT JOIN hana.EKPO AS ekpo ON ...
  LEFT JOIN ztekpo_final AS ztc ON ...
  WHERE ...;
QUIT;

(Optional) Replace missing or '.' in s_soItem:

DATA work.sap_spend;
  SET work.sap_spend;
  IF s_soItem = '.' THEN s_soItem = s_poItem;
RUN;

Clean up temporary tables:

PROC DATASETS LIBRARY=WORK NOLIST;
  DELETE ztekpo_concat ztekpo_final;
QUIT;

**Key Points**

Aggregation using MAX, SUM, MIN on grouped data may hide multiple values; ensure grouping matches requirements.

Avoid nesting summary functions inside CASE. Use a single MAX(CASE ...) pattern.

Handle missing or placeholder values ('.') after aggregation in a separate DATA step.

Ensure library references (LIBNAME) are defined before creating permanent tables.

Adapt filters (e.g., BUKRS, LIFNR, date ranges) according to your business rules.

**Customization**

Modify WHERE clauses to filter source data as needed.

Enable or disable commented fields for additional metrics.

Adjust GROUP BY keys if different granularity is required.

Use formatCol and formatDat macros when formatting identifiers or dates.
