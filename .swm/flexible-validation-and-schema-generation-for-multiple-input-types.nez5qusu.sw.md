---
title: Flexible Validation and Schema Generation for Multiple Input Types
---
This document outlines how the system enables validation and schema generation for types that accept both custom objects and strings. The process ensures flexible input handling, clear error reporting, and compatibility with serialization and documentation tools. The main steps are:

- Prepare validation logic for multiple input types
- Define a schema that accepts both forms
- Generate a JSON schema for documentation
- Return the schema with custom error messages
- Support serialization for both input types

```mermaid
sequenceDiagram
  participant Developer
  participant ValidationSystem
  Developer->>ValidationSystem: Provide value (object or string)
  ValidationSystem->>ValidationSystem: Prepare validation logic
  ValidationSystem->>ValidationSystem: Define schema for both types
  ValidationSystem->>ValidationSystem: Generate JSON schema
  ValidationSystem->>ValidationSystem: Return schema with custom error messages
  ValidationSystem->>Developer: Support serialization and documentation
```

# Spec

## Detailed View of the Program's Functionality

a. Preparing Validation for <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> Type

The process begins by setting up the validation logic for a type that represents a combination of a name and an email address. Before any schema is defined, the system ensures that the necessary email validation library is available and meets the required version. This step is crucial for the subsequent validation logic to function correctly.

b. Defining the Core Schema for <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>

Next, a schema is constructed that allows the input to be either an instance of the <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type or a string. This is achieved by creating a union schema that accepts both possibilities. The schema is designed to work for both Python objects and JSON representations. Additionally, custom error handling is specified: if the input does not match either allowed type, a specific error type and message are provided to inform the user that the input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.

c. Generating the JSON Schema for Allowed Types

To represent the union of allowed types in JSON Schema, the system iterates over each possible schema in the union (<SwmToken path="pydantic/json_schema.py" pos="165:14:16" line-data="            # if it introduces no ambiguity, i.e., there is only one distinct schema for that DefsRef.">`i.e`</SwmToken>., <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> instance and string). For each choice:

- It attempts to generate the corresponding JSON schema.
- If a choice should be omitted (for example, if it is not relevant for JSON Schema), it is skipped.
- If a choice is invalid for JSON Schema, a warning is emitted and the choice is skipped.
- If the choice is valid, its schema is added to the list of generated schemas.

After processing all choices, the system checks how many valid schemas were generated:

- If only one valid schema remains, that schema is returned directly.
- If multiple valid schemas exist, a union schema (using <SwmToken path="pydantic/json_schema.py" pos="727:2:2" line-data="                &#39;anyOf&#39;: [">`anyOf`</SwmToken>) is returned to represent all possibilities.

d. Returning the Final Schema with Custom Error

Once the union schema is generated, it is wrapped in a higher-level schema that attaches the custom error type and message. This ensures that, during validation, if the input does not match any of the allowed types, the user receives a clear and specific error indicating the nature of the problem.

e. Detailed Flow of JSON Schema Generation

When generating the JSON schema for any core schema (including the union described above), the system follows a detailed process:

- It first checks if the schema is a reference to an existing definition. If so, it returns a reference to avoid duplication.
- If not, it extracts any metadata from the schema, such as updates or extra information to be applied to the JSON schema.
- It applies any updates or extra information found in the metadata, possibly by wrapping the schema generation handler in additional functions.
- For each function in the metadata that modifies the schema, it wraps the handler again, allowing each function to tweak the schema as needed.
- For each annotation function in the metadata, it wraps the handler to allow further customizations.
- After all customizations are applied, it generates the JSON schema.
- If the schema is a core schema (not just a field), it ensures that any necessary definitions are populated.
- Finally, it returns the fully generated and customized JSON schema.

f. Wrapping Up Schema Construction and Serialization

After constructing the union schema and attaching custom error handling, the entire schema is wrapped in a structure that distinguishes between JSON and Python representations. For JSON, the schema expects a string; for Python, it accepts either a <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> instance or a string. Additionally, serialization logic is specified so that, when converting to a string (for example, during JSON serialization), the appropriate representation is used.

g. Selecting the JSON Schema for Output

When the system needs to output the JSON schema (for documentation, validation, or other purposes), it selects the JSON-specific part of the schema and passes it through the JSON schema generation process described above. This ensures that the outputted schema accurately reflects the allowed input types and includes all customizations and error handling specified earlier.

# Rule Definition

| Paragraph Name                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Rule ID | Category          | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | Conditions                                                                                                                                                                                                                                                                                                                               | Remarks                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema**, NameEmail.\_validate                                                                                                                                                                                                                                                             | RL-001  | Data Assignment   | The system must accept input data for the <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type in two forms: (1) as an object/instance with both 'name' and 'email' fields, or (2) as a string, which may be in the format 'Name <email>' or simply 'email'.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Input is provided for a field of type <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.                                                                                                                      | Accepted input forms: object with 'name' (string) and 'email' (string) fields, or string in 'Name <email>' or 'email' format.                                                                                                                                                                                                                                                                                                                                                         |
| NameEmail.\_validate, <SwmToken path="pydantic/networks.py" pos="62:2:2" line-data="    &#39;validate_email&#39;,">`validate_email`</SwmToken>                                                                                                                                                                                                                                                                                                                                              | RL-002  | Conditional Logic | When a string is provided as input, the system must parse the string to extract the name and email components, validate the email address using an external email validation library, and normalize the email. If the string does not contain a name, the 'name' field should be set to the local part of the email.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Input is a string.                                                                                                                                                                                                                                                                                                                       | Uses regex to parse 'Name <email>' or 'email'. If name is missing, set to local part of email. <SwmToken path="pydantic/networks.py" pos="1258:0:0" line-data="MAX_EMAIL_LENGTH = 2048">`MAX_EMAIL_LENGTH`</SwmToken> = 2048. Email must be validated and normalized.                                                                                                                                                                                                                 |
| <SwmToken path="pydantic/networks.py" pos="62:2:2" line-data="    &#39;validate_email&#39;,">`validate_email`</SwmToken>, NameEmail.\_validate                                                                                                                                                                                                                                                                                                                                              | RL-003  | Conditional Logic | When an object/instance is provided as input, the system must ensure both 'name' and 'email' fields are present and are strings, and validate the 'email' field using the external email validation library.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Input is an object/instance with 'name' and 'email' fields.                                                                                                                                                                                                                                                                              | Both fields must be strings. Email must be validated and normalized.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema**, NameEmail.\_validate, <SwmToken path="pydantic/networks.py" pos="62:2:2" line-data="    &#39;validate_email&#39;,">`validate_email`</SwmToken>                                                                                                                                   | RL-004  | Conditional Logic | If the input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> (<SwmToken path="pydantic/networks.py" pos="134:1:3" line-data="        e.g. `https` in `https://user:pass@host:port/path?query#fragment`">`e.g`</SwmToken>., string cannot be parsed, email is invalid, or required fields are missing), the system must reject the input and raise an error with type <SwmToken path="pydantic/networks.py" pos="1069:4:4" line-data="                    custom_error_type=&#39;name_email_type&#39;,">`name_email_type`</SwmToken> and message 'Input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>'. | Input does not match expected formats or fails validation.                                                                                                                                                                                                                                                                               | Custom error type: <SwmToken path="pydantic/networks.py" pos="1069:4:4" line-data="                    custom_error_type=&#39;name_email_type&#39;,">`name_email_type`</SwmToken>. Custom error message: 'Input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>'.                                                                         |
| <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**str**                                                                                                                                                                                                                                                                                                        | RL-005  | Computation       | The system must provide a method to convert the <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type to a string in the format 'name <email>', quoting the name if it matches the pattern of an email address.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> instance is serialized or converted to string.                                                                                                              | Output format: string. If name contains '@', output as '"name" <email>', otherwise as 'name <email>'.                                                                                                                                                                                                                                                                                                                                                                                 |
| <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema**, <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_json_schema**                                                                        | RL-006  | Computation       | The system must generate a schema definition that specifies the accepted input type as either a string or an object with 'name' and 'email' fields (union type), includes a custom error type identifier <SwmToken path="pydantic/networks.py" pos="1069:4:4" line-data="                    custom_error_type=&#39;name_email_type&#39;,">`name_email_type`</SwmToken>, and a custom error message 'Input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>'. For JSON schema generation, if only one valid schema is possible, return that schema; if multiple, return a union of all valid schemas. Apply any schema metadata, updates, or customizations as specified.                                                                             | JSON schema is generated for <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.                                                                                                                               | Schema must specify union of string and object with 'name' and 'email'. Custom error type: <SwmToken path="pydantic/networks.py" pos="1069:4:4" line-data="                    custom_error_type=&#39;name_email_type&#39;,">`name_email_type`</SwmToken>. Custom error message: 'Input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>'. |
| <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema** (uses <SwmToken path="pydantic/networks.py" pos="1065:1:3" line-data="            core_schema.json_or_python_schema(">`core_schema.json_or_python_schema`</SwmToken>), NameEmail.\_validate                                                                                       | RL-007  | Data Assignment   | The system must support both JSON and native (Python) input and output for the <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Input or output is in JSON or native format.                                                                                                                                                                                                                                                                                             | Supports both JSON (string) and Python (object or string) input/output.                                                                                                                                                                                                                                                                                                                                                                                                               |
| <SwmToken path="pydantic/networks.py" pos="1061:1:1" line-data="        import_email_validator()">`import_email_validator`</SwmToken>, <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema**, <SwmToken path="pydantic/networks.py" pos="50:2:2" line-data="    &#39;EmailStr&#39;,">`EmailStr`</SwmToken>.**get_pydantic_core_schema** | RL-008  | Conditional Logic | The system must ensure that the email validation logic (external library) is available and up-to-date before performing any validation or schema generation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Before validating or generating schema for <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> or <SwmToken path="pydantic/networks.py" pos="50:2:2" line-data="    &#39;EmailStr&#39;,">`EmailStr`</SwmToken>. | Requires <SwmToken path="pydantic/networks.py" pos="948:6:8" line-data="        raise ImportError(&quot;email-validator is not installed, run `pip install &#39;pydantic[email]&#39;`&quot;) from e">`email-validator`</SwmToken> package, version >= <SwmToken path="pydantic/networks.py" pos="950:14:16" line-data="        raise ImportError(&#39;email-validator version &gt;= 2.0 required, run pip install -U email-validator&#39;)">`2.0`</SwmToken>.                         |

# User Stories

## User Story 1: Flexible <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> Input and Validation

---

### Story Description:

As a user of the system, I want to provide a name and email address either as a structured object or as a string, so that I can easily input and validate contact information in the format that is most convenient for me, with clear error messages if the input is invalid.

---

### Business Rule Mapping:

| Rule ID | Paragraph Name                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Rule Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| RL-001  | <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema**, NameEmail.\_validate                                                                                                                                                                                                                                                             | The system must accept input data for the <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type in two forms: (1) as an object/instance with both 'name' and 'email' fields, or (2) as a string, which may be in the format 'Name <email>' or simply 'email'.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| RL-004  | <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema**, NameEmail.\_validate, <SwmToken path="pydantic/networks.py" pos="62:2:2" line-data="    &#39;validate_email&#39;,">`validate_email`</SwmToken>                                                                                                                                   | If the input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> (<SwmToken path="pydantic/networks.py" pos="134:1:3" line-data="        e.g. `https` in `https://user:pass@host:port/path?query#fragment`">`e.g`</SwmToken>., string cannot be parsed, email is invalid, or required fields are missing), the system must reject the input and raise an error with type <SwmToken path="pydantic/networks.py" pos="1069:4:4" line-data="                    custom_error_type=&#39;name_email_type&#39;,">`name_email_type`</SwmToken> and message 'Input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>'. |
| RL-002  | NameEmail.\_validate, <SwmToken path="pydantic/networks.py" pos="62:2:2" line-data="    &#39;validate_email&#39;,">`validate_email`</SwmToken>                                                                                                                                                                                                                                                                                                                                              | When a string is provided as input, the system must parse the string to extract the name and email components, validate the email address using an external email validation library, and normalize the email. If the string does not contain a name, the 'name' field should be set to the local part of the email.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| RL-003  | <SwmToken path="pydantic/networks.py" pos="62:2:2" line-data="    &#39;validate_email&#39;,">`validate_email`</SwmToken>, NameEmail.\_validate                                                                                                                                                                                                                                                                                                                                              | When an object/instance is provided as input, the system must ensure both 'name' and 'email' fields are present and are strings, and validate the 'email' field using the external email validation library.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| RL-008  | <SwmToken path="pydantic/networks.py" pos="1061:1:1" line-data="        import_email_validator()">`import_email_validator`</SwmToken>, <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema**, <SwmToken path="pydantic/networks.py" pos="50:2:2" line-data="    &#39;EmailStr&#39;,">`EmailStr`</SwmToken>.**get_pydantic_core_schema** | The system must ensure that the email validation logic (external library) is available and up-to-date before performing any validation or schema generation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

---

### Relevant Functionality:

- **NameEmail.get_pydantic_core_schema**
  1. **RL-001:**
     - When input is provided:
       - If input is an instance of <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>, accept as is.
       - If input is a string, proceed to parsing and validation.
       - If input is an object, ensure both 'name' and 'email' fields are present and are strings.
       - Otherwise, raise an error.
  2. **RL-004:**
     - If input is invalid (wrong type, missing fields, invalid email, etc.):
       - Raise error with type <SwmToken path="pydantic/networks.py" pos="1069:4:4" line-data="                    custom_error_type=&#39;name_email_type&#39;,">`name_email_type`</SwmToken> and message 'Input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>'.
- **NameEmail.\_validate**
  1. **RL-002:**
     - If input is a string:
       - If length > 2048, raise error.
       - Use regex to extract name and email.
       - If name is missing, set name to local part of email.
       - Validate email using external library (<SwmToken path="pydantic/networks.py" pos="948:6:8" line-data="        raise ImportError(&quot;email-validator is not installed, run `pip install &#39;pydantic[email]&#39;`&quot;) from e">`email-validator`</SwmToken>).
       - If validation fails, raise error.
       - Return <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>(name, normalized_email).
- <SwmToken path="pydantic/networks.py" pos="62:2:2" line-data="    &#39;validate_email&#39;,">`validate_email`</SwmToken>
  1. **RL-003:**
     - If input is an object:
       - Check that 'name' and 'email' fields exist and are strings.
       - Validate 'email' using external library.
       - If validation fails, raise error.
       - Return <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>(name, normalized_email).
- <SwmToken path="pydantic/networks.py" pos="1061:1:1" line-data="        import_email_validator()">`import_email_validator`</SwmToken>
  1. **RL-008:**
     - Before validation or schema generation:
       - Check if <SwmToken path="pydantic/networks.py" pos="948:6:8" line-data="        raise ImportError(&quot;email-validator is not installed, run `pip install &#39;pydantic[email]&#39;`&quot;) from e">`email-validator`</SwmToken> is installed and version >= <SwmToken path="pydantic/networks.py" pos="950:14:16" line-data="        raise ImportError(&#39;email-validator version &gt;= 2.0 required, run pip install -U email-validator&#39;)">`2.0`</SwmToken>.
       - If not, raise <SwmToken path="pydantic/networks.py" pos="947:3:3" line-data="    except ImportError as e:">`ImportError`</SwmToken>.

## User Story 2: Consistent <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> Serialization and Multi-format Support

---

### Story Description:

As a user of the system, I want the <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type to support both JSON and native input/output, and to be serialized to a string in the correct format, so that I can reliably store, transmit, and display contact information across different systems.

---

### Business Rule Mapping:

| Rule ID | Paragraph Name                                                                                                                                                                                                                                                                                                                                                                                        | Rule Description                                                                                                                                                                                                                                                                                                                            |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RL-005  | <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**str**                                                                                                                                                                                                                  | The system must provide a method to convert the <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type to a string in the format 'name <email>', quoting the name if it matches the pattern of an email address. |
| RL-007  | <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema** (uses <SwmToken path="pydantic/networks.py" pos="1065:1:3" line-data="            core_schema.json_or_python_schema(">`core_schema.json_or_python_schema`</SwmToken>), NameEmail.\_validate | The system must support both JSON and native (Python) input and output for the <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type.                                                                           |

---

### Relevant Functionality:

- **NameEmail.str**
  1. **RL-005:**
     - When serializing <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>:
       - If '@' in name, output '"name" <email>'.
       - Else, output 'name <email>'.
- **NameEmail.get_pydantic_core_schema** (uses <SwmToken path="pydantic/networks.py" pos="1065:1:3" line-data="            core_schema.json_or_python_schema(">`core_schema.json_or_python_schema`</SwmToken>)
  1. **RL-007:**
     - Accept input as either JSON string or Python object/string.
     - Output as string in specified format.

## User Story 3: Schema Generation and Custom Error Metadata for <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>

---

### Story Description:

As an API consumer or integrator, I want the system to generate accurate schema definitions for the <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type, including support for union types, custom error identifiers, and schema customizations, so that I can validate and document my data models effectively.

---

### Business Rule Mapping:

| Rule ID | Paragraph Name                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Rule Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RL-006  | <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema**, <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_json_schema**                                                                        | The system must generate a schema definition that specifies the accepted input type as either a string or an object with 'name' and 'email' fields (union type), includes a custom error type identifier <SwmToken path="pydantic/networks.py" pos="1069:4:4" line-data="                    custom_error_type=&#39;name_email_type&#39;,">`name_email_type`</SwmToken>, and a custom error message 'Input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>'. For JSON schema generation, if only one valid schema is possible, return that schema; if multiple, return a union of all valid schemas. Apply any schema metadata, updates, or customizations as specified. |
| RL-008  | <SwmToken path="pydantic/networks.py" pos="1061:1:1" line-data="        import_email_validator()">`import_email_validator`</SwmToken>, <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>.**get_pydantic_core_schema**, <SwmToken path="pydantic/networks.py" pos="50:2:2" line-data="    &#39;EmailStr&#39;,">`EmailStr`</SwmToken>.**get_pydantic_core_schema** | The system must ensure that the email validation logic (external library) is available and up-to-date before performing any validation or schema generation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |

---

### Relevant Functionality:

- **NameEmail.get_pydantic_core_schema**
  1. **RL-006:**
     - When generating schema:
       - Specify input as union of string or object with 'name' and 'email'.
       - If only one valid schema, return it; else, return union.
       - Include custom error type and message.
       - Apply schema metadata/customizations if present.
- <SwmToken path="pydantic/networks.py" pos="1061:1:1" line-data="        import_email_validator()">`import_email_validator`</SwmToken>
  1. **RL-008:**
     - Before validation or schema generation:
       - Check if <SwmToken path="pydantic/networks.py" pos="948:6:8" line-data="        raise ImportError(&quot;email-validator is not installed, run `pip install &#39;pydantic[email]&#39;`&quot;) from e">`email-validator`</SwmToken> is installed and version >= <SwmToken path="pydantic/networks.py" pos="950:14:16" line-data="        raise ImportError(&#39;email-validator version &gt;= 2.0 required, run pip install -U email-validator&#39;)">`2.0`</SwmToken>.
       - If not, raise <SwmToken path="pydantic/networks.py" pos="947:3:3" line-data="    except ImportError as e:">`ImportError`</SwmToken>.

# Code Walkthrough

## Building the core schema for email-like validation

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Prepare validation for NameEmail type"] --> node2["Define schema: accept NameEmail instance or string (Python & JSON)"]
    click node1 openCode "pydantic/networks.py:1056:1062"
    node2 --> node3["Generate JSON schema for allowed types"]
    click node2 openCode "pydantic/networks.py:1065:1071"
    node3 --> node4["Return schema with error type name_email_type and message 'Input is not a valid NameEmail'"]
    click node3 openCode "pydantic/json_schema.py:427:427"
    click node4 openCode "pydantic/networks.py:1063:1074"


subgraph node2 [union_schema]
  sgmain_1_node1["Start: Generate union JSON schema"] --> sgmain_1_loop1
  click sgmain_1_node1 openCode "pydantic/json_schema.py:1241:1264"
  subgraph sgmain_1_loop1["For each possible schema in the union"]
  sgmain_1_node2["Try to generate JSON schema for choice"]
  click sgmain_1_node2 openCode "pydantic/json_schema.py:1253:1257"
  sgmain_1_node2 -->|"Omit"| sgmain_1_node3["Skip this choice"]
  click sgmain_1_node3 openCode "pydantic/json_schema.py:1258:1259"
  sgmain_1_node2 -->|"Invalid"| sgmain_1_node4["Emit warning and skip"]
  click sgmain_1_node4 openCode "pydantic/json_schema.py:1260:1262"
  sgmain_1_node2 -->|"Valid"| sgmain_1_node5["Add to generated schemas"]
  click sgmain_1_node5 openCode "pydantic/json_schema.py:1257:1257"
  end
  sgmain_1_loop1 --> sgmain_1_node6{"How many valid schemas?"}
  click sgmain_1_node6 openCode "pydantic/json_schema.py:1262:1264"
  sgmain_1_node6 -->|"One"| sgmain_1_node7["Return the single schema"]
  click sgmain_1_node7 openCode "pydantic/json_schema.py:1263:1263"
  sgmain_1_node6 -->|"Multiple"| sgmain_1_node8["Return union of schemas"]
  click sgmain_1_node8 openCode "pydantic/json_schema.py:1264:1264"
end

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Prepare validation for <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> type"] --> node2["Define schema: accept <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken> instance or string (Python & JSON)"]
%%     click node1 openCode "<SwmPath>[pydantic/networks.py](pydantic/networks.py)</SwmPath>:1056:1062"
%%     node2 --> node3["Generate JSON schema for allowed types"]
%%     click node2 openCode "<SwmPath>[pydantic/networks.py](pydantic/networks.py)</SwmPath>:1065:1071"
%%     node3 --> node4["Return schema with error type <SwmToken path="pydantic/networks.py" pos="1069:4:4" line-data="                    custom_error_type=&#39;name_email_type&#39;,">`name_email_type`</SwmToken> and message 'Input is not a valid <SwmToken path="pydantic/networks.py" pos="1070:14:14" line-data="                    custom_error_message=&#39;Input is not a valid NameEmail&#39;,">`NameEmail`</SwmToken>'"]
%%     click node3 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:427:427"
%%     click node4 openCode "<SwmPath>[pydantic/networks.py](pydantic/networks.py)</SwmPath>:1063:1074"
%% 
%% 
%% subgraph node2 [<SwmToken path="pydantic/networks.py" pos="1067:5:5" line-data="                python_schema=core_schema.union_schema(">`union_schema`</SwmToken>]
%%   sgmain_1_node1["Start: Generate union JSON schema"] --> sgmain_1_loop1
%%   click sgmain_1_node1 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1241:1264"
%%   subgraph sgmain_1_loop1["For each possible schema in the union"]
%%   sgmain_1_node2["Try to generate JSON schema for choice"]
%%   click sgmain_1_node2 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1253:1257"
%%   sgmain_1_node2 -->|"Omit"| sgmain_1_node3["Skip this choice"]
%%   click sgmain_1_node3 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1258:1259"
%%   sgmain_1_node2 -->|"Invalid"| sgmain_1_node4["Emit warning and skip"]
%%   click sgmain_1_node4 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1260:1262"
%%   sgmain_1_node2 -->|"Valid"| sgmain_1_node5["Add to generated schemas"]
%%   click sgmain_1_node5 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1257:1257"
%%   end
%%   sgmain_1_loop1 --> sgmain_1_node6{"How many valid schemas?"}
%%   click sgmain_1_node6 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1262:1264"
%%   sgmain_1_node6 -->|"One"| sgmain_1_node7["Return the single schema"]
%%   click sgmain_1_node7 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1263:1263"
%%   sgmain_1_node6 -->|"Multiple"| sgmain_1_node8["Return union of schemas"]
%%   click sgmain_1_node8 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1264:1264"
%% end
```

<SwmSnippet path="/pydantic/networks.py" line="1056">

---

In <SwmToken path="pydantic/networks.py" pos="1056:3:3" line-data="    def __get_pydantic_core_schema__(">`__get_pydantic_core_schema__`</SwmToken>, we start by importing the email validator and set up a schema that can validate either a class instance or a string, using a union schema for flexibility and custom error messages. Next, we need to call <SwmToken path="pydantic/networks.py" pos="1067:5:5" line-data="                python_schema=core_schema.union_schema(">`union_schema`</SwmToken> to handle both input types.

```python
    def __get_pydantic_core_schema__(
        cls,
        _source: type[Any],
        _handler: GetCoreSchemaHandler,
    ) -> core_schema.CoreSchema:
        import_email_validator()

        return core_schema.no_info_after_validator_function(
            cls._validate,
            core_schema.json_or_python_schema(
                json_schema=core_schema.str_schema(),
                python_schema=core_schema.union_schema(
                    [core_schema.is_instance_schema(cls), core_schema.str_schema()],
                    custom_error_type='name_email_type',
                    custom_error_message='Input is not a valid NameEmail',
                ),
```

---

</SwmSnippet>

### Combining multiple schema choices

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start: Generate union JSON schema"] --> loop1
    click node1 openCode "pydantic/json_schema.py:1241:1264"
    
    subgraph loop1["For each possible schema in the union"]
        node2["Try to generate JSON schema for choice"]
        click node2 openCode "pydantic/json_schema.py:1253:1257"
        node2 -->|"Omit"| node3["Skip this choice"]
        click node3 openCode "pydantic/json_schema.py:1258:1259"
        node2 -->|"Invalid"| node4["Emit warning and skip"]
        click node4 openCode "pydantic/json_schema.py:1260:1262"
        node2 -->|"Valid"| node5["Add to generated schemas"]
        click node5 openCode "pydantic/json_schema.py:1257:1257"
    end
    loop1 --> node6{"How many valid schemas?"}
    click node6 openCode "pydantic/json_schema.py:1262:1264"
    node6 -->|"One"| node7["Return the single schema"]
    click node7 openCode "pydantic/json_schema.py:1263:1263"
    node6 -->|"Multiple"| node8["Return union of schemas"]
    click node8 openCode "pydantic/json_schema.py:1264:1264"

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start: Generate union JSON schema"] --> loop1
%%     click node1 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1241:1264"
%%     
%%     subgraph loop1["For each possible schema in the union"]
%%         node2["Try to generate JSON schema for choice"]
%%         click node2 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1253:1257"
%%         node2 -->|"Omit"| node3["Skip this choice"]
%%         click node3 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1258:1259"
%%         node2 -->|"Invalid"| node4["Emit warning and skip"]
%%         click node4 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1260:1262"
%%         node2 -->|"Valid"| node5["Add to generated schemas"]
%%         click node5 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1257:1257"
%%     end
%%     loop1 --> node6{"How many valid schemas?"}
%%     click node6 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1262:1264"
%%     node6 -->|"One"| node7["Return the single schema"]
%%     click node7 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1263:1263"
%%     node6 -->|"Multiple"| node8["Return union of schemas"]
%%     click node8 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:1264:1264"
```

<SwmSnippet path="/pydantic/json_schema.py" line="1241">

---

<SwmToken path="pydantic/json_schema.py" pos="1241:3:3" line-data="    def union_schema(self, schema: core_schema.UnionSchema) -&gt; JsonSchemaValue:">`union_schema`</SwmToken> iterates over each schema choice, uses <SwmToken path="pydantic/json_schema.py" pos="1257:7:7" line-data="                generated.append(self.generate_inner(choice_schema))">`generate_inner`</SwmToken> to build their JSON schemas, and skips or warns on invalid ones. We call <SwmToken path="pydantic/json_schema.py" pos="1257:7:7" line-data="                generated.append(self.generate_inner(choice_schema))">`generate_inner`</SwmToken> next to actually generate the JSON schema for each choice.

```python
    def union_schema(self, schema: core_schema.UnionSchema) -> JsonSchemaValue:
        """Generates a JSON schema that matches a schema that allows values matching any of the given schemas.

        Args:
            schema: The core schema.

        Returns:
            The generated JSON schema.
        """
        generated: list[JsonSchemaValue] = []

        choices = schema['choices']
        for choice in choices:
            # choice will be a tuple if an explicit label was provided
            choice_schema = choice[0] if isinstance(choice, tuple) else choice
            try:
                generated.append(self.generate_inner(choice_schema))
            except PydanticOmit:
                continue
            except PydanticInvalidForJsonSchema as exc:
                self.emit_warning('skipped-choice', exc.message)
        if len(generated) == 1:
            return generated[0]
        return self.get_flattened_anyof(generated)
```

---

</SwmSnippet>

### Generating JSON schema for a core schema

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is schema a reference to an existing definition?"}
    click node1 openCode "pydantic/json_schema.py:443:447"
    node1 -- Yes --> node9["Return reference to existing JSON schema"]
    click node9 openCode "pydantic/json_schema.py:447:447"
    node1 -- No --> node2["Extract metadata from schema"]
    click node2 openCode "pydantic/json_schema.py:503:503"
    node2 --> node3["Apply js_updates if present"]
    click node3 openCode "pydantic/json_schema.py:507:517"
    node3 --> node4["Apply js_extra if present"]
    click node4 openCode "pydantic/json_schema.py:518:533"
    subgraph loop1["For each js_modify_function in metadata"]
        node4 --> node5["Apply js_modify_function to schema handler"]
        click node5 openCode "pydantic/json_schema.py:534:550"
    end
    subgraph loop2["For each annotation_function in metadata"]
        node5 --> node6["Apply annotation_function to schema handler"]
        click node6 openCode "pydantic/json_schema.py:552:561"
    end
    node6 --> node7["Generate JSON schema"]
    click node7 openCode "pydantic/json_schema.py:563:563"
    node7 --> node8{"Is schema a core schema?"}
    click node8 openCode "pydantic/json_schema.py:564:565"
    node8 -- Yes --> node10["Populate definitions"]
    click node10 openCode "pydantic/json_schema.py:565:565"
    node10 --> node11["Return JSON schema"]
    click node11 openCode "pydantic/json_schema.py:566:566"
    node8 -- No --> node11

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1{"Is schema a reference to an existing definition?"}
%%     click node1 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:443:447"
%%     node1 -- Yes --> node9["Return reference to existing JSON schema"]
%%     click node9 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:447:447"
%%     node1 -- No --> node2["Extract metadata from schema"]
%%     click node2 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:503:503"
%%     node2 --> node3["Apply <SwmToken path="pydantic/json_schema.py" pos="507:3:3" line-data="        if js_updates := metadata.get(&#39;pydantic_js_updates&#39;):">`js_updates`</SwmToken> if present"]
%%     click node3 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:507:517"
%%     node3 --> node4["Apply <SwmToken path="pydantic/json_schema.py" pos="518:3:3" line-data="        if js_extra := metadata.get(&#39;pydantic_js_extra&#39;):">`js_extra`</SwmToken> if present"]
%%     click node4 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:518:533"
%%     subgraph loop1["For each <SwmToken path="pydantic/json_schema.py" pos="534:3:3" line-data="        for js_modify_function in metadata.get(&#39;pydantic_js_functions&#39;, ()):">`js_modify_function`</SwmToken> in metadata"]
%%         node4 --> node5["Apply <SwmToken path="pydantic/json_schema.py" pos="534:3:3" line-data="        for js_modify_function in metadata.get(&#39;pydantic_js_functions&#39;, ()):">`js_modify_function`</SwmToken> to schema handler"]
%%         click node5 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:534:550"
%%     end
%%     subgraph loop2["For each annotation_function in metadata"]
%%         node5 --> node6["Apply annotation_function to schema handler"]
%%         click node6 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:552:561"
%%     end
%%     node6 --> node7["Generate JSON schema"]
%%     click node7 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:563:563"
%%     node7 --> node8{"Is schema a core schema?"}
%%     click node8 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:564:565"
%%     node8 -- Yes --> node10["Populate definitions"]
%%     click node10 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:565:565"
%%     node10 --> node11["Return JSON schema"]
%%     click node11 openCode "<SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath>:566:566"
%%     node8 -- No --> node11
```

<SwmSnippet path="/pydantic/json_schema.py" line="427">

---

In <SwmToken path="pydantic/json_schema.py" pos="427:3:3" line-data="    def generate_inner(self, schema: CoreSchemaOrField) -&gt; JsonSchemaValue:  # noqa: C901">`generate_inner`</SwmToken>, we check for cached schema references, set up nested functions for schema generation and modification, and start wrapping the handler based on metadata keys for custom behavior.

```python
    def generate_inner(self, schema: CoreSchemaOrField) -> JsonSchemaValue:  # noqa: C901
        """Generates a JSON schema for a given core schema.

        Args:
            schema: The given core schema.

        Returns:
            The generated JSON schema.

        TODO: the nested function definitions here seem like bad practice, I'd like to unpack these
        in a future PR. It'd be great if we could shorten the call stack a bit for JSON schema generation,
        and I think there's potential for that here.
        """
        # If a schema with the same CoreRef has been handled, just return a reference to it
        # Note that this assumes that it will _never_ be the case that the same CoreRef is used
        # on types that should have different JSON schemas
        if 'ref' in schema:
            core_ref = CoreRef(schema['ref'])  # type: ignore[typeddict-item]
            core_mode_ref = (core_ref, self.mode)
            if core_mode_ref in self.core_to_defs_refs and self.core_to_defs_refs[core_mode_ref] in self.definitions:
                return {'$ref': self.core_to_json_refs[core_mode_ref]}

        def populate_defs(core_schema: CoreSchema, json_schema: JsonSchemaValue) -> JsonSchemaValue:
            if 'ref' in core_schema:
                core_ref = CoreRef(core_schema['ref'])  # type: ignore[typeddict-item]
                defs_ref, ref_json_schema = self.get_cache_defs_ref_schema(core_ref)
                json_ref = JsonRef(ref_json_schema['$ref'])
                # Replace the schema if it's not a reference to itself
                # What we want to avoid is having the def be just a ref to itself
                # which is what would happen if we blindly assigned any
                if json_schema.get('$ref', None) != json_ref:
                    self.definitions[defs_ref] = json_schema
                    self._core_defs_invalid_for_json_schema.pop(defs_ref, None)
                json_schema = ref_json_schema
            return json_schema

        def handler_func(schema_or_field: CoreSchemaOrField) -> JsonSchemaValue:
            """Generate a JSON schema based on the input schema.

            Args:
                schema_or_field: The core schema to generate a JSON schema from.

            Returns:
                The generated JSON schema.

            Raises:
                TypeError: If an unexpected schema type is encountered.
            """
            # Generate the core-schema-type-specific bits of the schema generation:
            json_schema: JsonSchemaValue | None = None
            if self.mode == 'serialization' and 'serialization' in schema_or_field:
                # In this case, we skip the JSON Schema generation of the schema
                # and use the `'serialization'` schema instead (canonical example:
                # `Annotated[int, PlainSerializer(str)]`).
                ser_schema = schema_or_field['serialization']  # type: ignore
                json_schema = self.ser_schema(ser_schema)

                # It might be that the 'serialization'` is skipped depending on `when_used`.
                # This is only relevant for `nullable` schemas though, so we special case here.
                if (
                    json_schema is not None
                    and ser_schema.get('when_used') in ('unless-none', 'json-unless-none')
                    and schema_or_field['type'] == 'nullable'
                ):
                    json_schema = self.get_flattened_anyof([{'type': 'null'}, json_schema])
            if json_schema is None:
                if _core_utils.is_core_schema(schema_or_field) or _core_utils.is_core_schema_field(schema_or_field):
                    generate_for_schema_type = self._schema_type_to_method[schema_or_field['type']]
                    json_schema = generate_for_schema_type(schema_or_field)
                else:
                    raise TypeError(f'Unexpected schema type: schema={schema_or_field}')

            return json_schema

        current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, handler_func)

        metadata = cast(_core_metadata.CoreMetadata, schema.get('metadata', {}))

        # TODO: I dislike that we have to wrap these basic dict updates in callables, is there any way around this?

        if js_updates := metadata.get('pydantic_js_updates'):

            def js_updates_handler_func(
                schema_or_field: CoreSchemaOrField,
                current_handler: GetJsonSchemaHandler = current_handler,
            ) -> JsonSchemaValue:
                json_schema = {**current_handler(schema_or_field), **js_updates}
                return json_schema

            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, js_updates_handler_func)

        if js_extra := metadata.get('pydantic_js_extra'):

            def js_extra_handler_func(
                schema_or_field: CoreSchemaOrField,
                current_handler: GetJsonSchemaHandler = current_handler,
            ) -> JsonSchemaValue:
                json_schema = current_handler(schema_or_field)
                if isinstance(js_extra, dict):
                    json_schema.update(to_jsonable_python(js_extra))
                elif callable(js_extra):
                    # similar to typing issue in _update_class_schema when we're working with callable js extra
                    js_extra(json_schema)  # type: ignore
                return json_schema

            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, js_extra_handler_func)

        for js_modify_function in metadata.get('pydantic_js_functions', ()):

            def new_handler_func(
                schema_or_field: CoreSchemaOrField,
                current_handler: GetJsonSchemaHandler = current_handler,
                js_modify_function: GetJsonSchemaFunction = js_modify_function,
            ) -> JsonSchemaValue:
                json_schema = js_modify_function(schema_or_field, current_handler)
                if _core_utils.is_core_schema(schema_or_field):
                    json_schema = populate_defs(schema_or_field, json_schema)
                original_schema = current_handler.resolve_ref_schema(json_schema)
                ref = json_schema.pop('$ref', None)
                if ref and json_schema:
                    original_schema.update(json_schema)
                return original_schema

            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, new_handler_func)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="550">

---

Here, we keep wrapping the handler for each metadata key, letting each one add its own tweaks before the schema is finally generated.

```python
            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, new_handler_func)

        for js_modify_function in metadata.get('pydantic_js_annotation_functions', ()):

            def new_handler_func(
                schema_or_field: CoreSchemaOrField,
                current_handler: GetJsonSchemaHandler = current_handler,
                js_modify_function: GetJsonSchemaFunction = js_modify_function,
            ) -> JsonSchemaValue:
                return js_modify_function(schema_or_field, current_handler)

            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, new_handler_func)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="561">

---

Finally, we generate the JSON schema with all customizations applied and return it for use elsewhere.

```python
            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, new_handler_func)

        json_schema = current_handler(schema)
        if _core_utils.is_core_schema(schema):
            json_schema = populate_defs(schema, json_schema)
        return json_schema
```

---

</SwmSnippet>

### Wrapping up schema construction and serialization

<SwmSnippet path="/pydantic/networks.py" line="1065">

---

Back in <SwmToken path="pydantic/networks.py" pos="1056:3:3" line-data="    def __get_pydantic_core_schema__(">`__get_pydantic_core_schema__`</SwmToken>, after <SwmToken path="pydantic/networks.py" pos="1067:5:5" line-data="                python_schema=core_schema.union_schema(">`union_schema`</SwmToken>, we wrap everything in <SwmToken path="pydantic/networks.py" pos="1065:3:3" line-data="            core_schema.json_or_python_schema(">`json_or_python_schema`</SwmToken> to handle both validation and string serialization.

```python
            core_schema.json_or_python_schema(
                json_schema=core_schema.str_schema(),
                python_schema=core_schema.union_schema(
                    [core_schema.is_instance_schema(cls), core_schema.str_schema()],
                    custom_error_type='name_email_type',
                    custom_error_message='Input is not a valid NameEmail',
                ),
                serialization=core_schema.to_string_ser_schema(),
            ),
        )
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="1382">

---

<SwmToken path="pydantic/json_schema.py" pos="1382:3:3" line-data="    def json_or_python_schema(self, schema: core_schema.JsonOrPythonSchema) -&gt; JsonSchemaValue:">`json_or_python_schema`</SwmToken> picks out the <SwmToken path="pydantic/json_schema.py" pos="1395:10:10" line-data="        return self.generate_inner(schema[&#39;json_schema&#39;])">`json_schema`</SwmToken> and sends it to <SwmToken path="pydantic/json_schema.py" pos="1395:5:5" line-data="        return self.generate_inner(schema[&#39;json_schema&#39;])">`generate_inner`</SwmToken> to actually build the JSON schema.

```python
    def json_or_python_schema(self, schema: core_schema.JsonOrPythonSchema) -> JsonSchemaValue:
        """Generates a JSON schema that matches a schema that allows values matching either the JSON schema or the
        Python schema.

        The JSON schema is used instead of the Python schema. If you want to use the Python schema, you should override
        this method.

        Args:
            schema: The core schema.

        Returns:
            The generated JSON schema.
        """
        return self.generate_inner(schema['json_schema'])
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
