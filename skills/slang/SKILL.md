---
name: slang
description: SLang language reference — syntax, types, entities, relations, actions, triggers, queues, and compilation to SCode JSON.
---

# SLang Language Reference

SLang is a domain-specific language for defining data models, actions, triggers, queues, and permissions for Sutro applications. It compiles to SCode, a JSON intermediate representation consumed by the Sutro platform.

## Syntax Fundamentals

- **Indentation-significant**: Blocks are delimited by indentation (like Python), not braces.
- **Comments**: `;;` starts a line comment. Comments work at all levels (top-level, inside blocks, etc.).
- **Identifiers**: `[_a-zA-Z0-9]+` (alphanumeric with underscores).
- **Strings**: Double-quoted `"like this"`.
- **Numbers**: Integer literals (`0`, `42`).
- **Booleans**: `true`, `false`.
- **Null**: `null`.
- **Assignment**: `:=` (not `=`).

## Top-Level Constructs

A SLang file contains zero or more of these, in any order:

1. `entity` — define a data model
2. `relation` — define a relationship between two models
3. `action` — define a named operation with a body
4. `trigger` — bind an action to an HTTP endpoint, queue, or event
5. `queue` — declare a named async queue
6. `permissions` — define role-based permission grants

---

## Entity (Model) Definition

```slang
entity ModelName
  description "Human-readable description."
  identity email        ;; marks this as the identity/user model
  subject               ;; marks this as the subject (authenticated user) model
  group @id             ;; scoping/grouping marker
  fields
    FieldName: TYPE
      description "Field description."
      minLength 2
    OptionalField: TYPE?
    FieldWithDefault: TYPE := "default value"
    EnumField: ENUM("val1", "val2", "val3") := "val1"
    ComputedField: BOOLEAN
      computed SomeOtherField == "approved"
```

### Entity Block Keywords

| Keyword | Purpose |
|---------|---------|
| `description` | Human-readable description string |
| `identity` | Marks the model as the identity model (e.g. `identity email`) |
| `subject` | Marks the model as the authenticated-user model |
| `group` | Scoping marker (e.g. `group @id`) |
| `fields` | Begins the field definitions block |

### Field Modifiers

| Modifier | Syntax | Purpose |
|----------|--------|---------|
| Optional | `TYPE?` | Field is not required |
| Default | `TYPE := "value"` | Default value when not provided |
| Description | `description "..."` | Human-readable field description |
| Min length | `minLength N` | Minimum string length constraint |
| Computed | `computed <expression>` | Derived value (read-only) — see Known Limitations |

### Primitive Types

| Type | Description | SCode Mapping |
|------|-------------|---------------|
| `TEXT` | String text | `p_TEXT` |
| `EMAIL` | Email address | `p_TEXT` |
| `PHONE_NUMBER` | Phone number | `p_PHONE_NUMBER` |
| `ADDRESS` | Physical address | `p_ADDRESS` |
| `NUMBER` | Numeric value | `p_NUMBER` |
| `CURRENCY_AMOUNT` | Monetary amount | `p_CURRENCY_AMOUNT` |
| `BOOLEAN` | True/false | `p_BOOLEAN` |
| `DATE` | Calendar date | `p_DATE` |
| `TIME` | Time of day | `p_TIME` |
| `DATE_TIME` | Combined date and time | `p_DATE_TIME` |
| `URL` | Web URL | `p_LINK` |

**Note**: `EMAIL` is a semantic alias that maps to `p_TEXT` in SCode.

### Enum Type

```slang
Status: ENUM("draft", "uploaded", "approved", "archived") := "draft"
```

Enums define a fixed set of allowed string values. An optional default can follow `:=`.

---

## Relation Definition

Relations connect two entities. Syntax:

```slang
relation ModelA[FieldOnA] CardinalityA --- CardinalityB ModelB[FieldOnB]
  description "Description of the relationship."
```

### Cardinality Formats

| Syntax | Meaning |
|--------|---------|
| `1` | Exactly one |
| `*` | Zero or more (unbounded) |
| `0..1` | Zero or one |
| `0..*` | Zero or more |
| `1..*` | One or more |
| `N..M` | Range from N to M |

### Relationship Ownership Rules (how SCode determines the owner)

- **Many-to-many**: Both models are owners.
- **One-to-one**: The left (first) model is the owner.
- **One-to-many / many-to-one**: The "one" side owns the relationship.

### Example

```slang
relation User[Memberships] 1 --- 0..* Membership[Member]
  description "Each Membership belongs to one User."
```

This means:
- A `User` has 0..* `Memberships` (accessed via `user.memberships`)
- A `Membership` belongs to exactly 1 `User` (accessed via `membership.member`)

---

## Action Definition

Actions define named operations with typed parameters and a body of statements.

```slang
action ActionName(param1: TYPE, param2?: TYPE := "default"): ReturnType
  description "What this action does."
  body
    ;; statements here
```

Actions with no parameters use empty parentheses:

```slang
action ListItems(): [Item]
  description "List all items for the current user."
  body
    items := pageOf Item where owner == @subject
    return items
```

### Action-Only Types

The following types are supported for Action parameters and return types, in addition to the Sutro primitives:

| Type | Description |
|------|-------------|
| `BYTE_STREAM` | Binary data / file upload | 
| `VOID` | No value (for action returns) | 

### Action Parameters

- Required: `name: TYPE`
- Optional: `name?: TYPE`
- Optional with default: `name?: TYPE := "default"`
- Model type: `name: ModelName`
- Model field type: `name: ModelName[FieldName]`
- Array return type: `[ModelName]`
- Optional return type: `ReturnType?`

### Action Body Statements

#### Create

```slang
variable := create ModelName {
  field1 := value1
  field2 := value2
}
```

#### Update

```slang
update variable {
  field1 := newValue
}
```

#### Delete

```slang
delete variable
```

#### Query (paginated)

```slang
result := pageOf ModelName where fieldName == value
```

#### Query (single record)

```slang
result := single ModelName where fieldName == value
```

Use `single` when you expect exactly one result (e.g., lookup by ID or unique field).

#### Assert

```slang
assert(description := "Error message shown to user.",
       rule := someCondition)
```

#### Enqueue

```slang
enqueue QueueName with variable
```

#### For Loop

```slang
for item in collection
  ;; loop body using item
```

#### Return

```slang
return expression
```

#### Function Call

```slang
result := SomeService.method(param1 := value1, param2 := value2)
```

---

## Trigger Definition

Triggers bind actions to external event sources.

```slang
trigger ActionName on TriggerType
  description "Description."
  ;; type-specific blocks
  arguments
    param := expression
  auth
    ;; auth rules
```

### Trigger Types

| SLang Name | Internal Type | Purpose |
|------------|---------------|---------|
| `HttpRequest` | `http` | HTTP API endpoint |
| `Queue` | `queue` | Async queue consumer |
| `Event` | `event` | Event bus listener |

### HTTP Trigger

```slang
trigger CreateUser on HttpRequest
  description "Create a new user."
  endpoint POST /users
  arguments
    name := @request.body.name
    email := @request.body.email
  auth
    @subject can "user:create"
```

**HTTP methods**: `GET`, `POST`, `PUT`, `PATCH`, `DELETE` (any valid method).

**Path parameters**: Use `{paramName}` in the path, reference via `@request.path.paramName`.

```slang
trigger GetUser on HttpRequest
  endpoint GET /users/{userId}
  arguments
    userId := @request.path.userId
```

Triggers that don't need arguments (e.g., list endpoints where the action derives data from `@subject`) can omit the `arguments` block:

```slang
trigger ListItems on HttpRequest
  description "List all items."
  endpoint GET /items
  auth
    @subject can "item:read"
```

### Queue Trigger

```slang
trigger ProcessJob on Queue
  description "Background job processor."
  queue jobQueue
  arguments
    job := @message
```

### Event Trigger

```slang
trigger OnJobCreated on Event
  description "Handle job creation event."
  event jobCreated
  arguments
    job := @message
```

### Auth Rules

Auth rules control who can invoke a trigger. They support:

| Rule | Syntax |
|------|--------|
| Permission check | `@subject can "permission:string"` |
| Scoped permission | `@subject can "permission" in GroupName` |
| Role check | `@subject is RoleName` |
| Scoped role | `@subject is RoleName in GroupName` |
| Anonymous access | `@subject is @anonymous` |
| Authenticated access | `@subject is @defined` |


Auth rules can be combined with `and`, `or`, and parenthesized grouping:

```slang
auth
  (@subject can "doc:read" or @subject is Admin) and @subject is @defined
```

---

## Queue Declaration

Queues are declared at the top level and referenced by actions and triggers.

```slang
queue queueName with ModelName
```

- `queueName` is the queue identifier.
- `ModelName` is the type of message the queue carries.
- Actions enqueue with: `enqueue queueName with variable`
- Queue triggers consume with: `trigger ActionName on Queue` + `queue queueName`

---

## Permissions

**Note**: Permissions parse correctly but are not yet fully serialized to SCode.

The syntax:

```slang
permissions MemberModel -> GroupModel
  "permission:action"
  "another:permission"
```

Example:

```slang
permissions User -> Organization
  "doc:read"
  "doc:write"
  "doc:delete"
```

---

## Expressions

### Operators (precedence, lowest to highest)

1. `or` — logical or
2. `and` — logical and
3. `==`, `!=` — equality
4. `>=`, `<=`, `>`, `<` — comparison
5. `in` — membership test
6. `+`, `-` — addition/subtraction
7. `*`, `/` — multiplication/division
8. `!` — logical not (unary)
9. `.` — property access
10. `()` — function call

### Special References

| Reference | Meaning |
|-----------|---------|
| `@subject` | The authenticated user |
| `@request` | The incoming HTTP request |
| `@request.body.*` | Request body fields |
| `@request.path.*` | URL path parameters |
| `@request.files.*` | Uploaded files |
| `@message` | Queue/event message payload |
| `@id` | Internal ID reference |
| `@anonymous` | Anonymous user marker |
| `@defined` | Any authenticated user marker |

### Literals

- Strings: `"hello"`
- Numbers: `42`
- Booleans: `true`, `false`
- Null: `null`
- Objects: `{ key := value }`
- Arrays: `[expr1, expr2]`

---

## Keywords Reference

### Active Keywords

```
action, and, anonymous, arguments, assert, auth, body, can, computed,
create, defined, delete, endpoint, enqueue, entity, event, fields, for, group,
in, is, model, on, of, or, pageOf, params, permissions, queue, relation,
return, role, single, subject, trigger, update, with, where
```

### Reserved (not yet implemented)

```
break, bus, case, concurrent, continue, emit, helper, include, lambda,
publish, switch, topic, while, yield
```

Do not use reserved words as identifiers. If you must use a reserved word as a field name, the SLang generator will automatically wrap it in backticks (e.g., `` `role` ``).

---

## Known Limitations

1. **Backtick identifiers with spaces**: The documentation historically mentioned backtick-quoted identifiers for names with spaces (`` `Display Name` ``), but the current lexer does not support spaces inside backticks. Use camelCase or PascalCase instead.

2. **Computed fields**: The `computed` keyword parses correctly, but computed expressions are not fully serialized to SCode output.

3. **Permissions**: Permission statements parse correctly but are not serialized to the SCode output.

4. **Round-trip field casing**: When generating SLang from SCode, field names are normalized to camelCase. This is valid SLang but may differ from the original casing.

---

## SCode Output Overview

SLang compiles to SCode, a JSON structure with this shape:

```json
{
  "appId": "uuid",
  "version": "1.0.0",
  "userModelId": "urn:sutro:model:<uuid>",
  "groupModelId": "urn:sutro:model:<uuid>",
  "models": [ ... ],
  "actions": [ ... ],
  "triggers": [ ... ],
  "queues": [ ... ],
  "securitySubjects": [ ... ],
  "requirements": [],
  "personas": [],
  "appOverview": null,
  "appDescription": "",
  "appDraft": null,
  "appViews": null,
  "domainModel": null
}
```

### ID Formats

| Entity | Format |
|--------|--------|
| Model | `urn:sutro:model:<uuid>` |
| Field/Edge | `urn:sutro:edge:<uuid>` |
| Action | `urn:sutro:action:<uuid>` |
| Trigger | `urn:sutro:trigger:<uuid>` |
| Effect | `urn:sutro:effect:<uuid>` |

### SCode Models

Each model has:
- `id` — Model ID (`urn:sutro:model:...`)
- `name` — Display name
- `fields` — Array of edges, each with `id`, `name`, `to` (type reference), `min`, `max`, `relationshipOwner`, `accessControl`

Field `to` values: primitive types are `p_TEXT`, `p_NUMBER`, etc. Relation fields point to model IDs.

### SCode Actions

Each action has:
- `id` — Action ID
- `name` — Display name
- `effects` — Array of effect objects (create, update, delete, assert, return, or `@sutro/executeSlang` for SLang bodies)

### SCode Triggers

Each trigger mapping has:
- `actionId` — References an action
- `trigger` — Object with `type` (`http`, `queue`, `event`), `method`, `path`, `initialState`, etc.

---

## Full Example

```slang
entity Organization
  description "Tenant workspace."
  group @id
  fields
    Name: TEXT
      description "Organization name."
      minLength 2
    Plan: ENUM("free", "team", "enterprise") := "free"

entity User
  description "Authenticated user."
  identity email
  subject
  fields
    Email: EMAIL
    DisplayName: TEXT
      minLength 1

entity Membership
  fields
    Role: ENUM("viewer", "admin", "owner")

entity Document
  fields
    Title: TEXT
      minLength 1
    Status: ENUM("draft", "published") := "draft"

relation User[Memberships] 1 --- 0..* Membership[Member]
relation Organization[Members] 1 --- 0..* Membership[Organization]
relation Organization[Documents] 1 --- 0..* Document[Organization]
relation User[OwnedDocuments] 1 --- 0..* Document[Owner]

queue documentQueue with Document

action CreateDocument(organization: Organization, title: TEXT): Document
  description "Create a new document."
  body
    doc := create Document {
      title := title
      organization := organization
      owner := @subject
    }
    return doc

action ProcessDocument(doc: Document): VOID
  description "Background document processor."
  body
    update doc { status := "published" }

trigger CreateDocument on HttpRequest
  endpoint POST /organizations/{organizationId}/documents
  arguments
    organization := @subject.organization
    title := @request.body.title
  auth
    @subject can "doc:create"

trigger ProcessDocument on Queue
  queue documentQueue
  arguments
    doc := @message
```
