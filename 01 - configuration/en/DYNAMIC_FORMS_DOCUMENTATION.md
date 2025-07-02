# Dynamic Forms System Documentation

## Overview

The Dynamic Forms System is a fundamental component that enables the capture, presentation, and modification of codes in a flexible and configurable manner. This system works closely integrated with the code structure to provide a user interface adaptable to the specific needs of each code type and realm.

The system is designed to handle the complexity of code relationships while maintaining an intuitive and efficient user experience for government officials.

## Form Types

### 1. Index Type Form

**Purpose**: Main form that displays all possible values for a code type within the current realm.

**Key Features**:
- **Tabular View**: Presents each code as an independent row
- **Relational Columns**: Shows related codes as additional columns
- **Flexible Configuration**: Displayed columns, their order, visibility, and sorting capability are defined through configuration
- **CRUD Operations**: Allows execution of all Create, Read, Update, and Delete operations on records

**Practical Example**:
```
Index Form: Budget Items 2025

| Item Code    | Item Name              | Sector      | Program          | Project     | Actions   |
|--------------|------------------------|-------------|------------------|-------------|-----------|
| 01.02.03.001 | Teacher Training       | Education   | Higher Education | EDU-2025-001| [V][E][D] |
| 01.02.03.002 | Classroom Infrastructure| Education   | Basic Education  | EDU-2025-002| [V][E][D] |
| 02.01.01.001 | Medical Equipment      | Health      | Primary Care     | HLT-2025-001| [V][E][D] |
```

### 2. Selection Prompt Type Form

**Purpose**: Modal form for code selection to be used by other forms or processes.

**Key Features**:
- **Modal Interface**: Presented as a popup window
- **Index-like View**: Maintains the same visual structure as the index form
- **Single or Multiple Selection**: Configurable according to the invoking form's needs
- **Read-only**: Does not allow CRUD operations, only selection
- **Filters and Search**: Includes tools to facilitate searching for specific codes

**Typical Use Case**:
- A budget capture form needs to select an investment sector
- The modal prompt opens showing all available sectors
- User selects the desired sector and the value is returned to the origin form

### 3. Capture/Modification Type Form

**Purpose**: Dedicated form for creating new codes or modifying existing codes.

**Key Features**:
- **Configurable Fields**: Displayed fields are defined through configuration
- **Flexible Validations**: Allows defining restrictions like required fields, formats, etc.
- **Variable Selection Methods**:
  - Simple auto-complete
  - Modal selection prompts
  - Free text fields
- **Customizable Layout**:
  - Organization in sections
  - Colspan control for specific fields
  - Field order and grouping

**Configuration Example**:
```
Capture Form: New Budget Item

Section: Basic Information
├── Item Code [Text, Required, Validation: ##.##.##.##]
├── Name [Text, Required, Colspan: 2]
└── Description [Text Area, Optional, Colspan: 2]

Section: Classifications
├── Item Type [Modal Selection, Required]
├── Investment Sector [Auto-complete, Conditional]
└── Investment Program [Modal Selection, Conditional]
```

## Data Model

### Entity Relationship Diagram

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ FORM_TYPE_CONFIG│    │ FORM_COLUMN_    │    │ FORM_FIELD_     │
│                 │    │    CONFIG       │    │    CONFIG       │
│ • id            │◄───┤                 │    │                 │
│ • code_type_id  │    │ • form_config_id├───►│ • form_config_id│
│ • realm_id      │    │ • column_code_  │    │ • field_name    │
│ • form_type     │    │   type_id       │    │ • field_type    │
│ • name          │    │ • column_name   │    │ • is_required   │
│ • description   │    │ • display_order │    │ • validation_   │
│ • is_active     │    │ • is_visible    │    │   rules         │
│ • created_at    │    │ • is_sortable   │    │ • selection_    │
│ • updated_at    │    │ • is_hidden_    │    │   method        │
└─────────────────┘    │   default       │    │ • section_name  │
                       │ • column_width  │    │ • field_order   │
                       │ • is_active     │    │ • colspan       │
                       │ • created_at    │    │ • is_active     │
                       │ • updated_at    │    │ • created_at    │
                       └─────────────────┘    │ • updated_at    │
                                              └─────────────────┘
```

### Table Definitions

#### 1. FORM_TYPE_CONFIG
Defines the main configuration of forms by code type and realm.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID/INT | Primary key |
| code_type_id | UUID/INT | Foreign key to code_type |
| realm_id | UUID/INT | Foreign key to realm |
| form_type | ENUM | Form type (INDEX, SELECTION_PROMPT, CAPTURE) |
| name | VARCHAR(255) | Form name |
| description | TEXT | Detailed description |
| additional_config | JSON | Specific configurations (pagination, filters, etc.) |
| is_active | BOOLEAN | Active status |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Last update timestamp |

**Example Records**:
```sql
INSERT INTO form_type_config VALUES 
(1, 1, 1, 'INDEX', 'Budget Items Index 2025', 'Main form for budget items management', '{"pagination": true, "default_page_size": 50, "enable_filters": true}', true),
(2, 1, 1, 'SELECTION_PROMPT', 'Budget Items Selector', 'Modal for budget items selection', '{"selection_mode": "single", "show_search": true}', true),
(3, 1, 1, 'CAPTURE', 'Budget Item Capture/Edit', 'Form for creating/editing budget items', '{"auto_save": false, "show_preview": true}', true);
```

#### 2. FORM_COLUMN_CONFIG
Defines columns for index and selection prompt type forms.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID/INT | Primary key |
| form_config_id | UUID/INT | Foreign key to form_type_config |
| column_code_type_id | UUID/INT | Foreign key to code_type (for related codes) |
| column_name | VARCHAR(100) | Column name |
| column_title | VARCHAR(255) | Title displayed to user |
| display_order | INT | Column appearance order |
| is_visible | BOOLEAN | If column is visible by default |
| is_sortable | BOOLEAN | If column allows sorting |
| is_hidden_default | BOOLEAN | If column is hidden by default |
| column_width | VARCHAR(20) | Column width (px, %, auto) |
| data_type | VARCHAR(50) | Column data type |
| display_format | JSON | Format rules for display |
| is_active | BOOLEAN | Active status |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Last update timestamp |

**Example Records**:
```sql
INSERT INTO form_column_config VALUES 
(1, 1, 1, 'item_code', 'Item Code', 1, true, true, false, '120px', 'TEXT', '{"align": "left"}', true),
(2, 1, 1, 'item_name', 'Item Name', 2, true, true, false, '300px', 'TEXT', '{"align": "left", "truncate": 30}', true),
(3, 1, 4, 'investment_sector', 'Sector', 3, true, true, false, '150px', 'RELATION', '{"show_code": true, "show_name": true}', true),
(4, 1, 5, 'investment_program', 'Program', 4, true, false, false, '200px', 'RELATION', '{"show_name_only": true}', true),
(5, 1, 6, 'mga_project', 'MGA Project', 5, true, false, true, '150px', 'RELATION', '{"show_code_only": true}', true);
```

#### 3. FORM_FIELD_CONFIG
Defines fields for capture/modification type forms.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID/INT | Primary key |
| form_config_id | UUID/INT | Foreign key to form_type_config |
| field_name | VARCHAR(100) | Internal field name |
| field_label | VARCHAR(255) | Label displayed to user |
| field_type | ENUM | Field type (TEXT, TEXTAREA, SELECT, AUTOCOMPLETE, etc.) |
| is_required | BOOLEAN | If field is mandatory |
| validation_rules | JSON | Specific validation rules |
| selection_method | ENUM | Selection method (MANUAL, AUTOCOMPLETE, MODAL_PROMPT) |
| related_code_type_id | UUID/INT | Foreign key to code_type (if applicable) |
| section_name | VARCHAR(100) | Section name where field appears |
| field_order | INT | Order within section |
| colspan | INT | Number of columns the field occupies |
| field_config | JSON | Field-specific configurations |
| is_active | BOOLEAN | Active status |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Last update timestamp |

**Example Records**:
```sql
INSERT INTO form_field_config VALUES 
(1, 3, 'item_code', 'Item Code', 'TEXT', true, '{"pattern": "##.##.##.##", "min_length": 11, "max_length": 11}', 'MANUAL', NULL, 'Basic Information', 1, 1, '{"placeholder": "Ex: 01.02.03.001"}', true),
(2, 3, 'item_name', 'Item Name', 'TEXT', true, '{"min_length": 5, "max_length": 255}', 'MANUAL', NULL, 'Basic Information', 2, 2, '{"placeholder": "Enter item name"}', true),
(3, 3, 'item_description', 'Description', 'TEXTAREA', false, '{"max_length": 1000}', 'MANUAL', NULL, 'Basic Information', 3, 2, '{"rows": 3}', true),
(4, 3, 'item_type', 'Item Type', 'SELECT', true, '{"required_message": "Must select an item type"}', 'MODAL_PROMPT', 3, 'Classifications', 1, 1, '{"show_description": true}', true),
(5, 3, 'investment_sector', 'Investment Sector', 'AUTOCOMPLETE', false, '{"conditional_required": "if_investment"}', 'AUTOCOMPLETE', 4, 'Classifications', 2, 1, '{"min_chars": 2}', true);
```

## Behaviors and Business Rules

### Index Type Forms

1. **Dynamic Filtering**: Displayed records are automatically filtered according to the current realm
2. **Relational Columns**: Only shows columns for codes that have defined relationships with the main code type
3. **Contextual Actions**: Available actions (Create, Edit, Delete) depend on user permissions and code status
4. **Configurable Pagination**: Page size and navigation options are configured per form

### Selection Prompt Type Forms

1. **Context Filtering**: Only shows codes applicable to the context from which it's invoked
2. **Selection Validation**: Validates that selection meets defined restrictions
3. **Advanced Search**: Includes search capabilities across multiple fields
4. **Preselection**: Can receive preselected values from the invoking form

### Capture/Modification Type Forms

1. **Real-time Validation**: Validations execute as user completes fields
2. **Conditional Dependencies**: Fields can be enabled/disabled based on other field values
3. **Smart Auto-complete**: Suggestions based on existing codes and defined relationships
4. **Relationship Preservation**: Automatically maintains code relationships according to defined rules

## Specific Use Cases

### Case 1: Budget Items Management

**Index Form**:
- Shows all items from "Budget 2025" realm
- Columns: Code, Name, Item Type, Sector, Program, Project
- Allows sorting by code and name
- Actions: Create new item, Edit, Delete, View details

**Capture Form**:
- Required fields: Code, Name, Item Type
- Conditional fields: Sector (only if Type = Investment)
- Validations: Code format, uniqueness, relationship coherence

### Case 2: Sector Selection for Projects

**Prompt Form**:
- Modal activated from project forms
- Filtering: Only active sectors from current realm
- Single selection with name search
- Returns code and name of selected sector

## Implementation Considerations

### Frontend (React/Angular/Vue Components)

```typescript
// Example form configuration
interface FormConfiguration {
  id: string;
  formType: 'INDEX' | 'SELECTION_PROMPT' | 'CAPTURE';
  codeTypeId: string;
  realmId: string;
  columns?: ColumnConfiguration[];
  fields?: FieldConfiguration[];
  layout?: LayoutConfiguration;
}

interface ColumnConfiguration {
  name: string;
  title: string;
  dataType: string;
  isVisible: boolean;
  isSortable: boolean;
  width: string;
  relationshipType?: string;
}

interface FieldConfiguration {
  name: string;
  label: string;
  fieldType: 'TEXT' | 'TEXTAREA' | 'SELECT' | 'AUTOCOMPLETE';
  isRequired: boolean;
  validationRules: ValidationRule[];
  selectionMethod: 'MANUAL' | 'AUTOCOMPLETE' | 'MODAL_PROMPT';
  section: string;
  order: number;
  colspan: number;
}
```

### Backend (API Endpoints)

```typescript
// Endpoints for form management
GET /api/forms/config/:codeTypeId/:realmId/:formType
POST /api/forms/config
PUT /api/forms/config/:id
DELETE /api/forms/config/:id

// Endpoints for form data
GET /api/forms/data/:configId
POST /api/forms/data/:configId
PUT /api/forms/data/:configId/:recordId
DELETE /api/forms/data/:configId/:recordId

// Endpoints for selection prompts
GET /api/forms/selection/:codeTypeId/:realmId
POST /api/forms/selection/validate
```

### Database

```sql
-- Recommended indexes
CREATE INDEX idx_form_config_type_realm ON form_type_config(code_type_id, realm_id);
CREATE INDEX idx_form_column_config_order ON form_column_config(form_config_id, display_order);
CREATE INDEX idx_form_field_section_order ON form_field_config(form_config_id, section_name, field_order);
```

## Integration with Code Structure

### Key Dependencies

1. **Relationship Validation**: Forms automatically validate that code relationships are coherent with those defined in `code_type_relationship`
2. **Realm Filtering**: Only shows codes valid for the current realm according to `realm_code_type`
3. **Hierarchies**: Forms respect hierarchies defined in code types
4. **Relationship Metadata**: Use metadata stored in `code_relationship` for additional validations

### Change Synchronization

- Changes in code structure are automatically reflected in forms
- Form configurations are validated against current code structure
- Change log maintained to track configuration modifications

## Error Cases and Handling

### Configuration Errors
- Columns referencing non-existent code types
- Required fields without adequate validation
- Inconsistent relationships between forms

### Data Errors
- Duplicate codes
- Invalid relationships
- Format restriction violations

### Error Recovery
- Pre-persistence validation
- Automatic rollback on failure
- Specific and actionable error messages

## Future Enhancements

1. **Visual Form Editor**: Drag-and-drop interface to configure forms
2. **Form Templates**: Predefined configurations for common cases
3. **Nested Forms**: Capability to manage codes with multiple relationship levels
4. **Import/Export**: Tools to migrate configurations between environments
5. **Configuration Versioning**: Version control for form configurations
6. **Usage Analytics**: Metrics on form usage and efficiency
7. **Advanced Validations**: Complex business rules and cross-validations
8. **Responsive Forms**: Automatic adaptation to different screen sizes
