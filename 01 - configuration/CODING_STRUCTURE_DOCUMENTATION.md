# Coding Structure System Documentation

## Overview

Governmental entities manage complex coding systems that are essential for financial administration, budget management, and regulatory compliance. These coding structures are hierarchical classification systems that organize financial information according to various criteria such as budget items, revenue sources, investment sectors, and administrative divisions.

The coding structure system is designed to handle the dynamic nature of governmental coding requirements, where different fiscal periods may require different coding schemes, and entities may need to manage multiple coding systems simultaneously.

## Key Concepts

### Realm-Based Organization
Each coding structure belongs to a **Realm**, which represents a specific context or scope of application within the governmental entity. Realms are typically associated with:
- Fiscal periods (e.g., Budget Year 2025)
- Administrative divisions (e.g., Central Administration, Decentralized Entities)
- Functional areas (e.g., Revenue Management, Expenditure Control)

### Code Type Flexibility
The system supports multiple **Code Types** that can be configured independently and reused across different realms. This approach provides flexibility while maintaining consistency in coding standards.

### Hierarchical Structure
Most governmental coding systems are hierarchical, allowing for detailed classification at multiple levels (e.g., Chapter > Article > Concept > Sub-concept).

### Code Type Relationships
Code types can have **many-to-many relationships** with other code types, creating complex interconnected coding structures. This allows for sophisticated classification systems where one code type can be related to multiple other code types simultaneously.

For example:
- A "Budget Item" code type can be related to "Item Types", "Investment Sectors", "Investment Programs", and "MGA Projects"
- Each of these related code types can, in turn, have their own relationships with other code types
- This creates a flexible network of coding relationships that reflects real-world governmental classification needs

## Business Rules

1. **Temporal Flexibility**: Different fiscal years may use completely different coding structures
2. **Multi-Realm Support**: A single entity can manage multiple coding realms simultaneously
3. **Code Type Reusability**: Code types can be shared across multiple realms
4. **Code Type Relationships**: Code types can have many-to-many relationships with other code types
5. **Hierarchical Organization**: Support for multi-level hierarchical codes
6. **Regulatory Compliance**: All coding structures must comply with Colombian governmental accounting standards

## Data Model

### Entity Relationship Diagram

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│      REALM      │    │  REALM_CODE_    │    │   CODE_TYPE     │
│                 │    │     TYPE        │    │                 │
│ • id            │◄───┤                 ├───►│ • id            │
│ • name          │    │ • realm_id      │    │ • name          │
│ • description   │    │ • code_type_id  │    │ • description   │
│ • fiscal_year   │    │ • is_active     │    │ • is_hierarchical│
│ • entity_div    │    │ • created_at    │    │ • max_levels    │
│ • is_active     │    │ • updated_at    │    │ • code_format   │
│ • created_at    │    └─────────────────┘    │ • is_active     │
│ • updated_at    │                           │ • created_at    │
└─────────────────┘                           │ • updated_at    │
                                              └─────────┬───────┘
                                                        │       │
                                         ┌──────────────┘       │
                                         │                      │
                                         ▼                      ▼
                               ┌─────────────────┐    ┌─────────────────┐
                               │ CODE_TYPE_REL   │    │      CODE       │◄┐
                               │                 │    │                 │ │
                               │ • id            │    │ • id            │ │
                               │ • source_type_id│    │ • code_type_id  │ │
                               │ • target_type_id│    │ • code_value    │ │
                               │ • relationship  │    │ • name          │ │
                               │ • is_required   │    │ • description   │ │
                               │ • is_active     │    │ • parent_id     │ │
                               │ • created_at    │    │ • level         │ │
                               │ • updated_at    │    │ • is_active     │ │
                               └─────────────────┘    │ • sort_order    │ │
                                                      │ • created_at    │ │
                                                      │ • updated_at    │ │
                                                      └─────────┬───────┘ │
                                                                │         │
                                                                ▼         │
                                                      ┌─────────────────┐ │
                                                      │ CODE_RELATION   │ │
                                                      │                 │ │
                                                      │ • id            │ │
                                                      │ • source_code_id├─┤
                                                      │ • target_code_id├─┘
                                                      │ • relation_type │
                                                      │ • is_active     │
                                                      │ • created_at    │
                                                      │ • updated_at    │
                                                      └─────────────────┘
```

### Table Definitions

#### 1. REALM
Represents the main context or scope for a coding structure within the governmental entity.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID/INT | Primary key |
| name | VARCHAR(255) | Realm name (e.g., "Expenditure Budget - 2025 - Central Administration") |
| description | TEXT | Detailed description of the realm |
| fiscal_year | INT | Associated fiscal year |
| entity_division | VARCHAR(100) | Entity subdivision (Central, Decentralized, etc.) |
| realm_type | VARCHAR(50) | Type of realm (BUDGET, REVENUE, INVESTMENT, etc.) |
| is_active | BOOLEAN | Active status |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Last update timestamp |

**Example Records:**
```sql
INSERT INTO realm VALUES 
(1, 'Expenditure Budget - 2025 - Central Administration', 'Main expenditure budget for central administration', 2025, 'CENTRAL', 'BUDGET', true),
(2, 'Revenue Sources - 2025', 'Revenue classification system for fiscal year 2025', 2025, 'ALL', 'REVENUE', true),
(3, 'Investment Sectors - Multi-year', 'Investment sector classification', NULL, 'ALL', 'INVESTMENT', true);
```

#### 2. CODE_TYPE
Defines the structure and characteristics of different types of codes that can be used across realms.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID/INT | Primary key |
| name | VARCHAR(255) | Code type name |
| description | TEXT | Description of the code type |
| is_hierarchical | BOOLEAN | Whether the code type supports hierarchy |
| max_levels | INT | Maximum hierarchy levels (if hierarchical) |
| code_format | VARCHAR(50) | Format pattern (e.g., "##.##.##.##") |
| validation_rules | JSON | Validation rules for codes |
| is_active | BOOLEAN | Active status |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Last update timestamp |

**Example Records:**
```sql
INSERT INTO code_type VALUES 
(1, 'Budget Items 2025', 'Hierarchical budget item classification', true, 4, '##.##.##.##', '{"min_length": 2, "max_length": 11}', true),
(2, 'Funding Sources', 'Non-hierarchical funding source codes', false, 1, '###', '{"min_length": 3, "max_length": 3}', true),
(3, 'Item Types', 'Budget item type classification', false, 1, '##', '{"min_length": 2, "max_length": 2}', true),
(4, 'Investment Sectors', 'Investment sector classification', true, 3, '##.##.##', '{"min_length": 2, "max_length": 8}', true);
```

#### 3. REALM_CODE_TYPE
Junction table that associates realms with their applicable code types.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID/INT | Primary key |
| realm_id | UUID/INT | Foreign key to realm |
| code_type_id | UUID/INT | Foreign key to code_type |
| is_required | BOOLEAN | Whether this code type is mandatory for the realm |
| is_active | BOOLEAN | Active status |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Last update timestamp |

#### 4. CODE_TYPE_RELATIONSHIP
Junction table that defines many-to-many relationships between code types.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID/INT | Primary key |
| source_type_id | UUID/INT | Foreign key to code_type (source) |
| target_type_id | UUID/INT | Foreign key to code_type (target) |
| relationship_name | VARCHAR(100) | Name/description of the relationship |
| is_required | BOOLEAN | Whether the relationship is mandatory |
| validation_rules | JSON | Rules for validating the relationship |
| is_active | BOOLEAN | Active status |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Last update timestamp |

**Example Records:**
```sql
INSERT INTO code_type_relationship VALUES 
(1, 1, 3, 'Budget Item Classification', true, '{"enforce_single_selection": true}', true),
(2, 1, 4, 'Investment Sector Assignment', false, '{"allow_multiple": true}', true),
(3, 1, 5, 'Investment Program Assignment', false, '{"conditional_required": "if_investment"}', true),
(4, 1, 6, 'MGA Project Assignment', false, '{"depends_on": "investment_program"}', true),
(5, 4, 5, 'Sector to Program', true, '{"hierarchical_constraint": true}', true);
```

#### 5. CODE
Contains the actual code values and their hierarchical relationships.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID/INT | Primary key |
| code_type_id | UUID/INT | Foreign key to code_type |
| code_value | VARCHAR(50) | The actual code value |
| name | VARCHAR(255) | Code name/title |
| description | TEXT | Detailed description |
| parent_id | UUID/INT | Self-referencing foreign key for hierarchy |
| level | INT | Hierarchy level (1 = root level) |
| is_active | BOOLEAN | Active status |
| sort_order | INT | Display order within same level |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Last update timestamp |

#### 6. CODE_RELATIONSHIP
Junction table that defines many-to-many relationships between individual codes.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID/INT | Primary key |
| source_code_id | UUID/INT | Foreign key to code (source) |
| target_code_id | UUID/INT | Foreign key to code (target) |
| relationship_type_id | UUID/INT | Foreign key to code_type_relationship |
| metadata | JSON | Additional relationship data |
| is_active | BOOLEAN | Active status |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Last update timestamp |

**Example Records:**
```sql
-- Budget Item "01.02.03.001 - Teacher Training Program" relationships
INSERT INTO code_relationship VALUES 
-- Relates to Item Type "3 - Investment"
(1, 101, 301, 1, '{"classification": "mandatory"}', true),
-- Relates to Investment Sector "01 - Education"
(2, 101, 401, 2, '{"priority": "high"}', true),
-- Relates to Investment Program "01.02 - Higher Education"
(3, 101, 501, 3, '{"budget_percentage": 85}', true),
-- Relates to MGA Project "EDU-2025-001"
(4, 101, 601, 4, '{"project_phase": "execution"}', true);
```

## Use Cases Examples

### Example 1: Budget Items Hierarchy
```
01 - Personnel Services
├── 01.01 - Basic Salaries
├── 01.02 - Additional Compensation
│   ├── 01.02.01 - Overtime
│   └── 01.02.02 - Bonuses
└── 01.03 - Social Security Contributions
```

### Example 2: Funding Sources (Non-hierarchical)
```
001 - National Treasury
002 - Own Resources
003 - Credit Operations
004 - International Cooperation
```

### Example 5: Realm Configuration
**Realm**: "Expenditure Budget - 2025 - Central Administration"
- **Code Types**:
  - Budget Items 2025 (hierarchical)
  - Funding Sources (flat)
  - Item Types (flat)
  - Investment Sectors (hierarchical)
  - Investment Programs (hierarchical)
  - MGA Projects (flat)

- **Code Type Relationships**:
  - Budget Items → Item Types (required)
  - Budget Items → Investment Sectors (conditional)
  - Budget Items → Investment Programs (conditional)
  - Budget Items → MGA Projects (conditional)
  - Investment Sectors → Investment Programs (hierarchical)
  - Investment Programs → MGA Projects (dependency)
**Budget Item Code Type** is related to:
- **Item Types** (required relationship):
  - 1 - Operations (Funcionamiento)
  - 2 - Debt Service (Servicio a la Deuda)
  - 3 - Investment (Inversión)

- **Investment Sectors** (optional, only for investment items):
  - 01 - Education
  - 02 - Health
  - 03 - Infrastructure
  - 04 - Social Development

- **Investment Programs** (optional, depends on investment sector):
  - 01.01 - Basic Education
  - 01.02 - Higher Education
  - 02.01 - Primary Healthcare
  - 02.02 - Hospital Infrastructure

- **MGA Projects** (optional, depends on investment program):
  - Specific project codes from the MGA (Management and Investment Projects) system

**Example Coding Structure:**
```
Budget Item: 01.02.03.001 - "Teacher Training Program"
├── Item Type: 3 - Investment
├── Investment Sector: 01 - Education  
├── Investment Program: 01.02 - Higher Education
└── MGA Project: EDU-2025-001 - "Teacher Professional Development"
```

**Code Relationship Examples:**
```sql
-- Sample codes from different code types
INSERT INTO code VALUES 
-- Budget Items
(101, 1, '01.02.03.001', 'Teacher Training Program', 'Professional development for teachers', NULL, 4, true, 1),
-- Item Types  
(301, 3, '3', 'Investment', 'Investment expenditures', NULL, 1, true, 3),
-- Investment Sectors
(401, 4, '01', 'Education', 'Education sector investments', NULL, 1, true, 1),
-- Investment Programs
(501, 5, '01.02', 'Higher Education', 'University and higher education programs', 401, 2, true, 2),
-- MGA Projects
(601, 6, 'EDU-2025-001', 'Teacher Professional Development', 'National teacher training initiative', NULL, 1, true, 1);

-- Relationships between these codes
INSERT INTO code_relationship VALUES 
(1, 101, 301, 1, '{"mandatory": true, "validation_rule": "investment_classification"}', true),
(2, 101, 401, 2, '{"sector_allocation": 100, "priority_level": "high"}', true),
(3, 101, 501, 3, '{"program_component": "teacher_development", "budget_share": 0.85}', true),
(4, 101, 601, 4, '{"project_phase": "execution", "start_date": "2025-01-01"}', true);
```

## Implementation Considerations

### Database Indexes
- Primary keys on all tables
- Foreign key indexes for relationships
- Composite index on (realm_id, code_type_id) in realm_code_type
- Composite index on (source_type_id, target_type_id) in code_type_relationship
- Composite index on (source_code_id, target_code_id) in code_relationship
- Index on code_value for fast lookups
- Index on parent_id for hierarchy queries
- Index on relationship_type_id for relationship queries

### API Design
- RESTful endpoints for CRUD operations
- Specialized endpoints for hierarchy operations
- Endpoints for managing code type relationships
- Endpoints for managing individual code relationships
- Bulk operations for code imports
- Validation endpoints for code format compliance
- Relationship validation endpoints
- Graph traversal endpoints for complex relationship queries

### Performance Considerations
- Use materialized paths or nested sets for deep hierarchies
- Cache frequently accessed code structures and relationships
- Implement soft deletes for historical data preservation
- Consider read replicas for reporting queries
- Optimize queries for relationship traversal
- Use graph-based queries for complex relationship patterns
- Consider denormalization for frequently accessed code relationships
- Implement relationship validation caching

## Colombian Regulatory Compliance

This coding structure system is designed to comply with:
- **CHIP** (Clasificador de Ingresos y Gastos Públicos)
- **CGN Standards** (Contaduría General de la Nación)
- **Budget Execution Regulations** (Estatuto Orgánico de Presupuesto)
- **Public Accounting Standards** (Régimen de Contabilidad Pública)

## Future Enhancements

- **Versioning System**: Track changes to coding structures over time
- **Import/Export Tools**: Bulk operations for code management
- **Validation Engine**: Advanced validation rules and constraints
- **Audit Trail**: Complete change history for compliance
- **Multi-language Support**: Support for indigenous languages as required by law
- **Relationship Visualization**: Graphical representation of code type relationships
- **Smart Validation**: Context-aware validation based on code type relationships
- **Relationship Templates**: Pre-configured relationship patterns for common use cases
