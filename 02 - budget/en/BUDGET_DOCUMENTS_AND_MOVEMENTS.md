# Budget Documents and Movements

## Overview

This document explains the detailed operation of **Budget Documents** and **Budget Movements** in the SIIAFE system, defining the specific columns that each document type handles and how different movement types affect these columns.

## Fundamental Concepts

### Budget Documents
Budget documents are entities that represent different states and operations within the budget cycle. Each document type has a specific set of columns that reflect its nature and purpose.

### Budget Movements
Budget movements are transactions that modify the columns of budget documents, following specific business rules according to the movement type and the affected document type.

---

## Types of Budget Documents and their Columns

### 1. Expenditure Budget (PG)

**Purpose**: Main document that establishes annual budget appropriations.

**Item Columns**:
- `initial_budget`: Initial value approved for the fiscal year
- `additions`: Sum of all budget additions made
- `reductions`: Sum of all budget reductions made
- `transfers`: Net transfers (inbound minus outbound)
- `definitive_budget`: Current budget after modifications
- `reserved_value`: Value committed in active CDPs
- `appropriation_balance`: Available balance for new commitments

**Calculation Formulas**:
```
definitive_budget = initial_budget + additions - reductions + transfers
appropriation_balance = definitive_budget - reserved_value
```

### 2. Budget Addition (AD)

**Purpose**: Document that increases the total budget.

**Item Columns**:
- `addition_value`: Value of the budget addition

### 3. Budget Reduction (RC)

**Purpose**: Document that reduces the total budget.

**Item Columns**:
- `reduction_value`: Value of the budget reduction

### 4. Budget Transfer (TR)

**Purpose**: Document that moves resources between items without changing the total.

**Item Columns**:
- `transfer_outbound_value`: Value leaving the item
- `transfer_inbound_value`: Value entering the item

### 5. Budget Availability Certificate (CDP)

**Purpose**: Document that certifies resource availability.

**Item Columns**:
- `initial_value`: Initial CDP value
- `commitments`: Value committed in RPs
- `releases`: Value released and returned to PG
- `balance_to_commit`: Available balance for commitment

**Calculation Formulas**:
```
balance_to_commit = initial_value - commitments - releases
```

### 6. Budget Registration (RP)

**Purpose**: Document that formalizes resource commitment.

**Item Columns**:
- `initial_value`: Initial RP value
- `obligations`: Value accrued as obligations
- `liquidations`: Value liquidated and returned to CDP
- `balance_to_obligate`: Available balance for obligations

**Calculation Formulas**:
```
balance_to_obligate = initial_value - obligations - liquidations
```

### 7. Payment Order (OP)

**Purpose**: Document that authorizes payment of obligations.

**Item Columns**:
- `initial_value`: Initial order value
- `payments`: Value actually paid
- `cancellations`: Cancelled order value
- `balance_to_pay`: Pending payment balance

**Calculation Formulas**:
```
balance_to_pay = initial_value - payments - cancellations
```

---

## Types of Budget Movements

### 1. Initial Appropriation

**Description**: Establishes the initial budget for the fiscal year.

**Affected Documents**:
- **Expenditure Budget**: 
  - ✅ Increases: `initial_budget`
  - ✅ Increases: `definitive_budget`
  - ✅ Increases: `appropriation_balance`

**Example**:
```sql
-- Movement: Initial Appropriation $500,000,000
UPDATE expenditure_budget_item SET
    initial_budget = initial_budget + 500000000,
    definitive_budget = definitive_budget + 500000000,
    appropriation_balance = appropriation_balance + 500000000
WHERE document_id = :pg_id AND item_id = :item_id;
```

### 2. Budget Addition

**Description**: Increases budget with new resources.

**Affected Documents**:
- **Expenditure Budget**:
  - ✅ Increases: `additions`
  - ✅ Increases: `definitive_budget`
  - ✅ Increases: `appropriation_balance`
- **Budget Addition**:
  - ✅ Increases: `addition_value`

**Example**:
```sql
-- Movement: Budget Addition $100,000,000
-- Affects Expenditure Budget
UPDATE expenditure_budget_item SET
    additions = additions + 100000000,
    definitive_budget = definitive_budget + 100000000,
    appropriation_balance = appropriation_balance + 100000000
WHERE document_id = :pg_id AND item_id = :item_id;

-- Affects Addition Document
UPDATE budget_addition_item SET
    addition_value = addition_value + 100000000
WHERE document_id = :ad_id AND item_id = :item_id;
```

### 3. Budget Reduction

**Description**: Reduces budget due to lower resource availability.

**Affected Documents**:
- **Expenditure Budget**:
  - ✅ Increases: `reductions`
  - ❌ Reduces: `definitive_budget`
  - ❌ Reduces: `appropriation_balance`
- **Budget Reduction**:
  - ✅ Increases: `reduction_value`

**Example**:
```sql
-- Movement: Budget Reduction $50,000,000
-- Affects Expenditure Budget
UPDATE expenditure_budget_item SET
    reductions = reductions + 50000000,
    definitive_budget = definitive_budget - 50000000,
    appropriation_balance = appropriation_balance - 50000000
WHERE document_id = :pg_id AND item_id = :item_id;

-- Affects Reduction Document
UPDATE budget_reduction_item SET
    reduction_value = reduction_value + 50000000
WHERE document_id = :rc_id AND item_id = :item_id;
```

### 4. Budget Transfer

**Description**: Moves resources between items without changing the total.

**Affected Documents**:
- **Expenditure Budget (Source Item)**:
  - ❌ Reduces: `transfers`
  - ❌ Reduces: `definitive_budget`
  - ❌ Reduces: `appropriation_balance`
- **Expenditure Budget (Target Item)**:
  - ✅ Increases: `transfers`
  - ✅ Increases: `definitive_budget`
  - ✅ Increases: `appropriation_balance`
- **Budget Transfer**:
  - ✅ Increases: `transfer_outbound_value` (source item)
  - ✅ Increases: `transfer_inbound_value` (target item)

### 5. CDP Value (CDP Issuance)

**Description**: Reserves budget resources in a CDP.

**Affected Documents**:
- **Expenditure Budget**:
  - ✅ Increases: `reserved_value`
  - ❌ Reduces: `appropriation_balance`
- **Budget Availability Certificate**:
  - ✅ Increases: `initial_value`
  - ✅ Increases: `balance_to_commit`

**Example**:
```sql
-- Movement: CDP Issuance $80,000,000
-- Affects Expenditure Budget
UPDATE expenditure_budget_item SET
    reserved_value = reserved_value + 80000000,
    appropriation_balance = appropriation_balance - 80000000
WHERE document_id = :pg_id AND item_id = :item_id;

-- Affects CDP
UPDATE cdp_item SET
    initial_value = initial_value + 80000000,
    balance_to_commit = balance_to_commit + 80000000
WHERE document_id = :cdp_id AND item_id = :item_id;
```

### 6. RP Value (RP Issuance)

**Description**: Commits CDP resources in an RP.

**Affected Documents**:
- **Budget Availability Certificate**:
  - ✅ Increases: `commitments`
  - ❌ Reduces: `balance_to_commit`
- **Budget Registration**:
  - ✅ Increases: `initial_value`
  - ✅ Increases: `balance_to_obligate`

### 7. CDP Release

**Description**: Releases unused CDP resources back to PG.

**Affected Documents**:
- **Expenditure Budget**:
  - ❌ Reduces: `reserved_value`
  - ✅ Increases: `appropriation_balance`
- **Budget Availability Certificate**:
  - ✅ Increases: `releases`
  - ❌ Reduces: `balance_to_commit`

### 8. Accrual (OP Value)

**Description**: Accrues an RP obligation in an OP.

**Affected Documents**:
- **Budget Registration**:
  - ✅ Increases: `obligations`
  - ❌ Reduces: `balance_to_obligate`
- **Payment Order**:
  - ✅ Increases: `initial_value`
  - ✅ Increases: `balance_to_pay`

---

## Database Structure

### Table: budget_document_item

**Base Columns (for all types)**:
```sql
CREATE TABLE budget_document_item (
    id SERIAL PRIMARY KEY,
    document_id INTEGER NOT NULL,
    item_number INTEGER NOT NULL,
    item_description VARCHAR(500),
    
    -- Document type specific columns (JSON or specific columns)
    specific_columns JSONB,
    
    -- Metadata
    item_metadata JSON,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Column Configuration by Document Type

```sql
CREATE TABLE document_type_columns (
    id SERIAL PRIMARY KEY,
    document_type_id INTEGER NOT NULL,
    column_name VARCHAR(100) NOT NULL,
    data_type VARCHAR(50) NOT NULL,
    default_value DECIMAL(18,2) DEFAULT 0,
    is_calculated BOOLEAN DEFAULT FALSE,
    calculation_formula TEXT,
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE
);

-- Configuration for Expenditure Budget
INSERT INTO document_type_columns VALUES
(1, 1, 'initial_budget', 'DECIMAL', 0, false, null, 'Initial approved budget'),
(2, 1, 'additions', 'DECIMAL', 0, false, null, 'Sum of budget additions'),
(3, 1, 'reductions', 'DECIMAL', 0, false, null, 'Sum of budget reductions'),
(4, 1, 'transfers', 'DECIMAL', 0, false, null, 'Net transfers'),
(5, 1, 'definitive_budget', 'DECIMAL', 0, true, 'initial_budget + additions - reductions + transfers', 'Current budget'),
(6, 1, 'reserved_value', 'DECIMAL', 0, false, null, 'Value reserved in CDPs'),
(7, 1, 'appropriation_balance', 'DECIMAL', 0, true, 'definitive_budget - reserved_value', 'Available balance');

-- Configuration for CDP
INSERT INTO document_type_columns VALUES
(8, 2, 'initial_value', 'DECIMAL', 0, false, null, 'Initial CDP value'),
(9, 2, 'commitments', 'DECIMAL', 0, false, null, 'Value committed in RPs'),
(10, 2, 'releases', 'DECIMAL', 0, false, null, 'Released value'),
(11, 2, 'balance_to_commit', 'DECIMAL', 0, true, 'initial_value - commitments - releases', 'Balance to commit');
```

---

## Movement Type Affectation Configuration

### Table: movement_type_affectation

```sql
CREATE TABLE movement_type_affectation (
    id SERIAL PRIMARY KEY,
    movement_type_id INTEGER NOT NULL,
    document_type_id INTEGER NOT NULL,
    affected_column VARCHAR(100) NOT NULL,
    affectation_type VARCHAR(20) NOT NULL, -- 'INCREASES', 'REDUCES'
    multiplier_factor DECIMAL(10,4) DEFAULT 1.0,
    is_source BOOLEAN DEFAULT FALSE, -- If it's the movement source document
    is_target BOOLEAN DEFAULT FALSE, -- If it's the movement target document
    execution_order INTEGER DEFAULT 1,
    is_active BOOLEAN DEFAULT TRUE
);

-- Configuration for Initial Appropriation
INSERT INTO movement_type_affectation VALUES
(1, 1, 1, 'initial_budget', 'INCREASES', 1.0, false, true, 1, true),
(2, 1, 1, 'definitive_budget', 'INCREASES', 1.0, false, true, 2, true),
(3, 1, 1, 'appropriation_balance', 'INCREASES', 1.0, false, true, 3, true);

-- Configuration for Budget Addition
INSERT INTO movement_type_affectation VALUES
(4, 2, 1, 'additions', 'INCREASES', 1.0, false, true, 1, true),
(5, 2, 1, 'definitive_budget', 'INCREASES', 1.0, false, true, 2, true),
(6, 2, 1, 'appropriation_balance', 'INCREASES', 1.0, false, true, 3, true),
(7, 2, 3, 'addition_value', 'INCREASES', 1.0, true, false, 4, true);

-- Configuration for CDP Value
INSERT INTO movement_type_affectation VALUES
(8, 5, 1, 'reserved_value', 'INCREASES', 1.0, true, false, 1, true),
(9, 5, 1, 'appropriation_balance', 'REDUCES', 1.0, true, false, 2, true),
(10, 5, 2, 'initial_value', 'INCREASES', 1.0, false, true, 3, true),
(11, 5, 2, 'balance_to_commit', 'INCREASES', 1.0, false, true, 4, true);
```

---

## Movement Processing

### Function: process_budget_movement

```sql
CREATE OR REPLACE FUNCTION process_budget_movement(
    p_movement_id INTEGER,
    p_movement_value DECIMAL(18,2)
)
RETURNS BOOLEAN AS $$
DECLARE
    v_movement_type_id INTEGER;
    v_source_document_id INTEGER;
    v_target_document_id INTEGER;
    v_affectation RECORD;
    v_apply_value DECIMAL(18,2);
BEGIN
    -- Get movement information
    SELECT movement_type_id, source_document_id, target_document_id
    INTO v_movement_type_id, v_source_document_id, v_target_document_id
    FROM budget_movement
    WHERE id = p_movement_id;

    -- Process each configured affectation
    FOR v_affectation IN 
        SELECT * FROM movement_type_affectation 
        WHERE movement_type_id = v_movement_type_id 
          AND is_active = true
        ORDER BY execution_order
    LOOP
        -- Calculate value to apply
        v_apply_value := p_movement_value * v_affectation.multiplier_factor;
        
        -- Determine document to affect
        IF v_affectation.is_source AND v_source_document_id IS NOT NULL THEN
            -- Affect source document
            PERFORM update_document_column(
                v_source_document_id,
                v_affectation.affected_column,
                v_affectation.affectation_type,
                v_apply_value
            );
        ELSIF v_affectation.is_target AND v_target_document_id IS NOT NULL THEN
            -- Affect target document
            PERFORM update_document_column(
                v_target_document_id,
                v_affectation.affected_column,
                v_affectation.affectation_type,
                v_apply_value
            );
        END IF;
    END LOOP;

    RETURN TRUE;
END;
$$ LANGUAGE plpgsql;
```

### Function: update_document_column

```sql
CREATE OR REPLACE FUNCTION update_document_column(
    p_document_id INTEGER,
    p_column VARCHAR(100),
    p_affectation_type VARCHAR(20),
    p_value DECIMAL(18,2)
)
RETURNS VOID AS $$
DECLARE
    v_sql TEXT;
    v_operator TEXT;
BEGIN
    -- Determine operator based on affectation type
    v_operator := CASE 
        WHEN p_affectation_type = 'INCREASES' THEN '+'
        WHEN p_affectation_type = 'REDUCES' THEN '-'
        ELSE '+'
    END;

    -- Build and execute dynamic SQL
    v_sql := 'UPDATE budget_document_item SET ' ||
             'specific_columns = specific_columns || ' ||
             'jsonb_build_object($1, ' ||
             'COALESCE((specific_columns->>$1)::decimal, 0) ' || v_operator || ' $2) ' ||
             'WHERE document_id = $3';
             
    EXECUTE v_sql USING p_column, p_value, p_document_id;
END;
$$ LANGUAGE plpgsql;
```

---

## Complete Practical Example

### Scenario: Complete process from PG to OP

```sql
-- 1. Initial PG Appropriation
INSERT INTO budget_movement VALUES 
(1, 'MOV-001', 1, NULL, 1, 500000000.00, '2025-01-01', 'Initial appropriation fiscal year 2025');

-- Process movement
SELECT process_budget_movement(1, 500000000.00);

-- PG state after step 1:
-- initial_budget: $500,000,000
-- definitive_budget: $500,000,000
-- appropriation_balance: $500,000,000

-- 2. CDP Issuance
INSERT INTO budget_movement VALUES 
(2, 'MOV-002', 5, 1, 2, 100000000.00, '2025-02-01', 'CDP-001 issuance');

-- Process movement
SELECT process_budget_movement(2, 100000000.00);

-- State after step 2:
-- PG: appropriation_balance: $400,000,000, reserved_value: $100,000,000
-- CDP: initial_value: $100,000,000, balance_to_commit: $100,000,000

-- 3. RP Issuance
INSERT INTO budget_movement VALUES 
(3, 'MOV-003', 6, 2, 3, 80000000.00, '2025-02-15', 'RP-001 issuance');

-- Process movement
SELECT process_budget_movement(3, 80000000.00);

-- State after step 3:
-- CDP: commitments: $80,000,000, balance_to_commit: $20,000,000
-- RP: initial_value: $80,000,000, balance_to_obligate: $80,000,000
```

---

## System Advantages

1. **Flexibility**: Configuration of specific columns per document type
2. **Consistency**: Clear affectation rules per movement type
3. **Traceability**: Each movement records exactly which columns it modifies
4. **Scalability**: Easy to add new document and movement types
5. **Integrity**: Automatic validations in each processing
6. **Audit**: Complete record of all changes made

This system provides a robust and flexible framework for handling budget documents and movements, maintaining data integrity and facilitating detailed tracking of all budget operations.
