# Credit Cards ETL Project

## Overview

This project implements an ETL (Extract, Transform, Load) process for credit card data using Informatica Data Engineering Integration and Oracle SQL. The ETL pipeline reads data from multiple source tables, applies transformations, and loads it into target tables with business rules such as data masking, filtering, and surrogate key generation.

## Technologies Used

- Informatica Data Engineering Integration

- Oracle SQL


## Data Sources

#### The project extracts data from the following source tables:

DEMO_ACCOUNT

DEMO_CREDIT_CARD

DEMO_EMPLOYEE

DEMO_DEPT

## Target Tables

#### The transformed data is loaded into the following tables in the TGT_DWH schema:

CREDIT_CARDS_ARCHIVE: Stores records created before a configurable date.

CREDIT_CARDS_CURRENT: Stores records created on or after a configurable date.

REJECTED_DATA: Stores records with invalid credit card numbers (less than 16 digits).

## Business Logic and Transformations

### 1. Data Cleansing and Transformation

Whitespace Removal:

A variable CC_NUMBER_VAR is created in the Expression Transformation to remove spaces from CC_NUMBER.

Expression: REPLACECHR(0,CC_NUMBER,' ','')

Credit Card Masking:

Another port in the Expression Transformation masks the credit card number while preserving the first 3 and last 3 digits.

Expression: rpad(substr(CC_NUMBER_VAR,0,3),13,'X') || substr(CC_NUMBER_VAR,14,3)

### 2. Data Filtering and Routing

Router Transformation is used to:

Send records with CC_OPEN_DATE before the configured threshold to CREDIT_CARDS_ARCHIVE.

Send records with CC_OPEN_DATE on or after the threshold to CREDIT_CARDS_CURRENT.

Reject records where LENGTH(CC_NUMBER_VAR) < 16 and send them to REJECTED_DATA.

### 3. Surrogate Key Generation

A Sequence Generator Transformation (Sequence_SURR) is used to create a CC_SUR surrogate key that increments sequentially and persists across executions.

### 4. Data Loading (Insert/Update Logic)

Lookup Transformations are used to check for existing records in both CREDIT_CARDS_ARCHIVE and CREDIT_CARDS_CURRENT.

Update Strategy Transformations:

If a match is found, the record is updated.

If no match is found, a new record is inserted.

## Execution Control

Readiness Check: The ETL execution does not start unless a predefined "ready file" is available on the server.

Parameterization: The date threshold for splitting records is stored in a configuration file (config.properties).

Error Handling: Records failing validation (e.g., invalid CC numbers) are redirected to REJECTED_DATA.csv.

## Deployment and Execution

### 1- Run the check_ready_file.sh script to ensure execution is permitted.

### 2- Execute the SQL scripts to create schema, tables, and sequence.

### 3- Configure config.properties with the required date threshold.

### 4- Import the Informatica mapping (mapping.xml) and workflow (workflow.xml) into Informatica Developer.

### 5- Execute the CREDIT_CARDS_WF workflow to load data.
