---
name: frappe-doctype-creation
description: Design and implement Frappe DocTypes by identifying business entities, extracting fields, intelligently assigning appropriate Frappe field types, defining relationships, naming conventions, validations, permissions, and generating valid DocType metadata. Use when creating or modifying DocTypes in a Frappe application.
---

# Frappe DocType Architecture and Creation

Design and implement DocTypes based on the user's application requirements. Identify entities, extract their fields, determine appropriate field types, establish relationships, and generate valid Frappe DocType metadata.

## 1. Inspect the App and Version

- Identify the existing Frappe app, module, neighboring DocTypes, and project structure.
- Determine the installed Frappe version.
- Follow the existing DocType file structure and metadata conventions.
- Check existing DocTypes to avoid duplication.
- Refer to documentation and installed framework source for version-specific properties.

## 2. Identify DocTypes

Analyze the user's requirements and identify all business entities.

For each entity:
- Determine whether it requires a separate DocType.
- Identify its purpose and responsibilities.
- Determine whether it is a Standard, Child Table, Single, Tree, or Virtual DocType.
- Identify dependencies and relationships with other DocTypes.

## 3. Field Identification and Type Assignment (MANDATORY)

Analyze every entity and extract all required fields from the user's requirements.

The agent must independently identify each field and assign the most appropriate Frappe field type based on its meaning, data format, and expected usage.

### Field Identification Process

For every identified field:

1. Identify the information the field must store.
2. Understand its data format and expected values.
3. Determine whether it represents text, numeric data, dates, Boolean values, predefined choices, references, or structured data.
4. Select the appropriate Frappe field type.
5. Configure the required field properties.
6. Identify validation, uniqueness, default value, and read-only requirements.

### Field Type Selection Guidelines

| Data Requirement | Frappe Field Type |
|---|---|
| Names, codes, phone numbers | Data |
| Email addresses | Data (Options: Email) |
| Website addresses | Data (Options: URL) |
| Short descriptions | Small Text |
| Long descriptions | Text / Long Text |
| Whole numbers | Int |
| Decimal measurements | Float |
| Monetary values | Currency |
| Percentages | Percent |
| Yes/No values | Check |
| Fixed choices | Select |
| Calendar dates | Date |
| Date and time | Datetime |
| Time values | Time |
| Reference to another DocType | Link |
| Dynamic document reference | Dynamic Link |
| Multiple child records | Table |
| File uploads | Attach |
| Image uploads | Attach Image |
| Rich formatted content | Text Editor |
| Password values | Password |
| Calculated or non-editable values | Read Only property |
| Form layout sections | Section Break / Column Break |

### Field Type Assignment Rules

- Never assign field types based only on field names. Understand their actual meaning.
- Phone numbers, postal codes, registration numbers, and identification numbers should generally use Data rather than Int.
- Use Currency for monetary values, not Float.
- Use Date for calendar dates and Datetime when time information is required.
- Use Select for predefined choices.
- Use Link when referencing another DocType.
- Use Table when a parent document owns multiple child records.
- Use Check for Boolean values.
- Configure appropriate precision for decimal and currency fields.
- Avoid unnecessary or duplicate fields.
- Ask for clarification when ambiguity materially affects the data model.

### Required Field Analysis Output

Before generating DocType metadata, produce a field mapping:

| Field Label | Field Name | Field Type | Options | Mandatory | Reason |
|---|---|---|---|---|---|
| Customer Name | customer_name | Data | - | Yes | Stores customer name |
| Mobile Number | mobile_number | Data | Phone | Yes | Preserves leading zeros |
| Email | email | Data | Email | No | Stores email address |
| Date of Birth | date_of_birth | Date | - | No | Stores calendar date |
| Customer Status | status | Select | Active, Inactive | Yes | Fixed status choices |

## 4. Define Field Properties

Configure fields using appropriate metadata properties:

- fieldname
- label
- fieldtype
- options
- reqd
- unique
- read_only
- hidden
- default
- in_list_view
- in_standard_filter
- in_global_search
- depends_on
- fetch_from
- description
- precision
- length
- permlevel

Only apply properties when required by the workflow and supported by the installed Frappe version.

## 5. Define Relationships

- Use Link for references to another DocType.
- Use Dynamic Link for dynamically selected DocType references.
- Use Table for child records.
- Identify one-to-one, one-to-many, and many-to-many relationships.
- Create association DocTypes when many-to-many relationships require independent fields or lifecycle.
- Verify that all referenced DocTypes exist.
- Ensure Dynamic Link fields have a corresponding field identifying the target DocType.

## 6. Configure Naming and Document Behavior

- Select appropriate naming methods such as naming_series, field-based naming, format, or other supported strategies.
- Configure autoname according to the installed Frappe version.
- Enable Is Submittable only when document submission and cancellation are required.
- Configure Single, Tree, Child Table, and Virtual behavior only when appropriate.
- Define lifecycle hooks when required.

## 7. Define Validation and Permissions

- Identify mandatory, unique, and conditional fields.
- Define data validation requirements.
- Implement authoritative validation in Python controllers.
- Use Client Scripts for form interactions.
- Define role-based permissions.
- Apply field-level permissions and document-level restrictions where necessary.

## 8. Generate DocType Metadata

Generate valid Frappe DocType JSON metadata and associated Python controller files.

Follow the existing application structure.

For related DocTypes:
- Create referenced and child DocTypes before parent DocTypes where required.
- Ensure Link options match the target DocType names.
- Ensure Table options match the child DocType names.
- Maintain consistent fieldnames and naming conventions.
- Use only metadata properties supported by the installed Frappe version.

## 9. Validate the Architecture

Before implementation, verify:

- Every required business entity is identified.
- Every field has an appropriate field type.
- Field types match actual data requirements.
- Field properties are valid.
- Link and Table relationships are correct.
- Naming conventions are valid.
- Dependencies are resolved.
- Permissions match the requirements.
- Metadata follows the installed Frappe version.

## 10. Implementation and Review

- Generate the required DocType files.
- Implement controller logic when required.
- Add client scripts only when needed.
- Add focused automated tests when requested.
- Do not run migrations, install applications, or modify site data without explicit authorization.

Summarize the generated DocTypes, field mappings, relationships, files created, assumptions, and any required follow-up commands.

## Expected Output

The agent must produce:

1. Identified DocTypes.
2. Field identification and field-type mapping.
3. Field properties and configurations.
4. DocType relationships.
5. Naming conventions.
6. Validation requirements.
7. Permissions.
8. Dependency mapping.
9. Validated DocType JSON metadata.
10. Implementation files and verification results.

## Critical Rule

Never generate a DocType schema without first analyzing every identified field and assigning an appropriate Frappe field type. Field identification and type assignment must happen before metadata generation.
