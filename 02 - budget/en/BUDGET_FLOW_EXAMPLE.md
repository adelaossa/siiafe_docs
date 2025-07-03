# Complete Budget Flow Example

## Scenario Description

**Entity**: Municipality of Example  
**Fiscal Year**: 2025  
**Process**: Contracting consulting services for institutional strengthening

---

## STEP 1: Code Structure

### 1.1 Realm and Code Type Configuration

```sql
-- Realm: Municipal Budget
INSERT INTO realm VALUES 
(1, 'MUNICIPAL_BUDGET', 'Municipal Budget', 'Municipal budget classification', true);

-- Code Types
INSERT INTO code_type VALUES 
(1, 1, 'ITEM', 'Budget Item', 'Classification by expense item', true),
(2, 1, 'SOURCE', 'Funding Source', 'Origin of resources', true);
```

### 1.2 Item Structure

```sql
-- Item Codes
INSERT INTO code VALUES 
(1, 1, NULL, '2.2', 'GENERAL EXPENSES', 'Operating and administration expenses', true),
(2, 1, 1, '2.2.01', 'PERSONNEL EXPENSES', 'Personnel remuneration and benefits', true),
(3, 1, 1, '2.2.02', 'GENERAL EXPENSES', 'Various operating expenses', true),
(4, 1, 3, '2.2.02.01', 'PROFESSIONAL SERVICES', 'Professional services contracting', true),
(5, 1, 3, '2.2.02.02', 'TECHNICAL SERVICES', 'Technical services contracting', true);
```

### 1.3 Source Structure

```sql
-- Source Codes
INSERT INTO code VALUES 
(10, 2, NULL, '11', 'OWN RESOURCES', 'Entity own resources', true),
(11, 2, 10, '11.1', 'CURRENT INCOME', 'Current income unrestricted', true),
(12, 2, 10, '11.2', 'CAPITAL RESOURCES', 'Own capital resources', true),
(20, 2, NULL, '12', 'TRANSFERS', 'National level transfers', true),
(21, 2, 20, '12.1', 'SGP - GENERAL PURPOSE', 'General Participation System', true);
```

---

## STEP 2: Initial Budget Appropriation

### 2.1 PG Document Type Configuration

```sql
-- Required code configuration for PG
INSERT INTO code_config_document_type VALUES 
(1, 1, 1, true, 1, '{"validate_hierarchy": true}', true),  -- Mandatory item
(2, 1, 2, true, 2, '{"validate_coherence": true}', true); -- Mandatory source
```

### 2.2 General Budget Creation

```sql
-- General Budget Document
INSERT INTO budget_document VALUES 
(1, 1, 'PG-2025-001', '2025-01-01', NULL, 'CURRENT', 500000000.00, 500000000.00, 
 'Operating budget fiscal year 2025', 1, 2, '2025-01-15 10:00:00', 
 '{"decree": "001-2025", "sanction_date": "2025-01-15"}', true, '2025-01-01 08:00:00', '2025-01-15 10:00:00');
```

### 2.3 Budget Items

```sql
-- Item 1: Professional Services with Own Resources
INSERT INTO budget_document_item VALUES 
(1, 1, 1, 'Professional services and consulting', 200000000.00, 200000000.00, 
 NULL, '{"detailed_description": "Professional services contracting for institutional strengthening"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');

-- Item 2: Technical Services with Own Resources  
INSERT INTO budget_document_item VALUES 
(2, 1, 2, 'Specialized technical services', 150000000.00, 150000000.00, 
 NULL, '{"detailed_description": "Specialized technical services contracting"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');

-- Item 3: Professional Services with SGP
INSERT INTO budget_document_item VALUES 
(3, 1, 3, 'Professional services - Strengthening', 100000000.00, 100000000.00, 
 NULL, '{"detailed_description": "Professional services financed with SGP"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');

-- Item 4: Technical Services with SGP
INSERT INTO budget_document_item VALUES 
(4, 1, 4, 'Technical services - Training', 50000000.00, 50000000.00, 
 NULL, '{"detailed_description": "Technical training services with SGP"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');
```

### 2.4 Item Coding

```sql
-- Coding Item 1: Professional Services + Own Resources
INSERT INTO budget_item_coding VALUES 
(1, 1, 1, 4, '2025-01-01 08:00:00', '2025-01-01 08:00:00'), -- Item: Professional Services
(2, 1, 2, 11, '2025-01-01 08:00:00', '2025-01-01 08:00:00'); -- Source: Current Income

-- Coding Item 2: Technical Services + Own Resources
INSERT INTO budget_item_coding VALUES 
(3, 2, 1, 5, '2025-01-01 08:00:00', '2025-01-01 08:00:00'), -- Item: Technical Services
(4, 2, 2, 11, '2025-01-01 08:00:00', '2025-01-01 08:00:00'); -- Source: Current Income

-- Coding Item 3: Professional Services + SGP
INSERT INTO budget_item_coding VALUES 
(5, 3, 1, 4, '2025-01-01 08:00:00', '2025-01-01 08:00:00'), -- Item: Professional Services
(6, 3, 2, 21, '2025-01-01 08:00:00', '2025-01-01 08:00:00'); -- Source: SGP

-- Coding Item 4: Technical Services + SGP
INSERT INTO budget_item_coding VALUES 
(7, 4, 1, 5, '2025-01-01 08:00:00', '2025-01-01 08:00:00'), -- Item: Technical Services
(8, 4, 2, 21, '2025-01-01 08:00:00', '2025-01-01 08:00:00'); -- Source: SGP
```

### 2.5 State After Step 2

**Availability Summary:**
- **Professional Services + Own Resources**: $200,000,000
- **Technical Services + Own Resources**: $150,000,000  
- **Professional Services + SGP**: $100,000,000
- **Technical Services + SGP**: $50,000,000
- **TOTAL BUDGET**: $500,000,000

---

## STEP 3: Budget Availability Certificate (CDP) Issuance

### 3.1 CDP Request

**Need**: Contract consulting for institutional strengthening for $180,000,000
**Items to affect**: 
- Professional Services + Own Resources: $120,000,000
- Professional Services + SGP: $60,000,000

### 3.2 CDP Configuration

```sql
-- Code configuration for CDP (inherits from PG)
INSERT INTO code_config_document_type VALUES 
(3, 2, 1, true, 1, '{"inherit_from_pg": true}', true),  -- Mandatory item
(4, 2, 2, true, 2, '{"inherit_from_pg": true}', true); -- Mandatory source
```

### 3.3 CDP Creation

```sql
-- CDP Document
INSERT INTO budget_document VALUES 
(2, 2, 'CDP-2025-001', '2025-02-15', '2025-05-15', 'ISSUED', 180000000.00, 180000000.00, 
 'CDP for institutional strengthening consulting contracting', 3, NULL, NULL, 
 '{"process": "LP-001-2025", "object": "Institutional strengthening consulting"}', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
```

### 3.4 CDP Items

```sql
-- CDP Item 1: Professional Services with Own Resources
INSERT INTO budget_document_item VALUES 
(5, 2, 1, 'Strengthening consulting - Phase 1', 120000000.00, 120000000.00, 
 1, '{"phase": "1", "description": "Diagnosis and strategic formulation"}', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');

-- CDP Item 2: Professional Services with SGP
INSERT INTO budget_document_item VALUES 
(6, 2, 2, 'Strengthening consulting - Phase 2', 60000000.00, 60000000.00, 
 3, '{"phase": "2", "description": "Implementation and monitoring"}', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
```

### 3.5 CDP Coding

```sql
-- Coding CDP Item 1: Professional Services + Own Resources
INSERT INTO budget_item_coding VALUES 
(9, 5, 1, 4, '2025-02-15 09:00:00', '2025-02-15 09:00:00'), -- Item: Professional Services
(10, 5, 2, 11, '2025-02-15 09:00:00', '2025-02-15 09:00:00'); -- Source: Current Income

-- Coding CDP Item 2: Professional Services + SGP
INSERT INTO budget_item_coding VALUES 
(11, 6, 1, 4, '2025-02-15 09:00:00', '2025-02-15 09:00:00'), -- Item: Professional Services
(12, 6, 2, 21, '2025-02-15 09:00:00', '2025-02-15 09:00:00'); -- Source: SGP
```

### 3.6 CDP Affectation Movements (With Automatic Counterparts)

```sql
-- SINGLE MOVEMENT: CDP Issuance with automatic counterpart
-- Columns: id, movement_number, movement_type_id, document_origin_id, total_movement_value, movement_date, observations, document_support, user_id, approval_date, status, is_active, created_at, updated_at
INSERT INTO budget_movement VALUES 
(2, 'MOV-2025-002', 8, 2, 180000000.00, '2025-02-15', 
 'CDP-2025-001 issuance with automatic PG affectation', 'CDP-2025-001', 3, '2025-02-15 09:30:00', 
 'APPROVED', true, '2025-02-15 09:00:00', '2025-02-15 09:30:00');

-- LINE 1: Increase in CDP (main movement)
-- Columns: id, movement_id, line_number, movement_type_id, affected_document_id, line_value, line_observations, is_active, created_at, updated_at
INSERT INTO budget_movement_detail VALUES 
(2, 2, 1, 8, 2, 180000000.00, 'Initial value of CDP-2025-001', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:30:00');

-- LINE 2: Reduction in PG (automatic counterpart)
-- Columns: id, movement_id, line_number, movement_type_id, affected_document_id, line_value, line_observations, is_active, created_at, updated_at
INSERT INTO budget_movement_detail VALUES 
(3, 2, 2, 7, 1, 180000000.00, 'Automatic counterpart: PG affectation by CDP-2025-001', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:30:00');
 
-- Note: movement_type_id = 8 'INITIAL_CDP_VALUE' automatically generates type_id = 7 'CDP_AFFECTATION'
-- Note: Single movement (MOV-2025-002) with two lines affecting different documents
```

### 3.7 Movement Item Detail - CDP

```sql
-- Item detail for CDP movement (MOV-002)
-- Columns: id, movement_id, item_id, target_item_id, detail_value, observations, created_at, updated_at
INSERT INTO movement_item_detail VALUES 
-- Items affected in CDP (line 1 of movement)
(3, 2, 5, 1, 120000000.00, 'CDP Own Resources from PG Phase 1', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00'),
(4, 2, 6, 3, 60000000.00, 'CDP SGP from PG Phase 2', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00'),

-- Items affected in PG (line 2 of movement - counterpart)
(5, 2, 1, NULL, 120000000.00, 'Reduction: PG Phase 1 by CDP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00'),
(6, 2, 3, NULL, 60000000.00, 'Reduction: PG Phase 2 by CDP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00');

-- Note: Both movement lines share the same item detail
-- Note: target_item_id indicates source item when it's a transfer
```

### 3.8 CDP Relationship with PG

```sql
-- CDP relationship with PG (single movement with counterpart)
-- Columns: id, document_origin_id, document_target_id, relation_type, relation_value, relation_percentage, relation_metadata, relation_date, is_active, created_at, updated_at
INSERT INTO budget_document_relation VALUES 
(1, 1, 2, 'ORIGINATES', 180000000.00, 36.00, 
 '{"operation_type": "RESERVE", "concepts": ["Professional Services OR", "Professional Services SGP"], "movement_id": 2, "lines": [1, 2]}', 
 '2025-02-15', true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
 
-- Note: relation_metadata includes the unique movement ID and its composing lines
```

### 3.9 State After Step 3

**Automatic Update by Movement:**
```sql
-- Update PG balances after CDP
-- Columns in UPDATE: current_value, updated_at
UPDATE budget_document_item SET 
    current_value = 80000000.00,  -- $200M - $120M = $80M
    updated_at = '2025-03-01 10:30:00'
WHERE id = 1; -- Professional Services + Own Resources

-- Columns in UPDATE: current_value, updated_at
UPDATE budget_document_item SET 
    current_value = 40000000.00,  -- $100M - $60M = $40M
    updated_at = '2025-03-01 10:30:00'
WHERE id = 3; -- Professional Services + SGP
```

**State Summary:**
- **PG**: $500,000,000 total, $320,000,000 available
- **CDP**: $180,000,000 total, $180,000,000 available

**Availability Summary:**
- **Professional Services + Own Resources**: $80,000,000 (reserved: $120,000,000)
- **Technical Services + Own Resources**: $150,000,000 (no changes)
- **Professional Services + SGP**: $40,000,000 (reserved: $60,000,000)
- **Technical Services + SGP**: $50,000,000 (no changes)
- **CDP ISSUED**: $180,000,000

---

## STEP 4: Budget Registration (RP) Issuance

### 4.1 Contract Award

**Result**: Contract awarded for $120,000,000 (66.67% of CDP)
**Contractor**: Consultores Asociados SAS
**Contract**: CNT-2025-001

### 4.2 RP Configuration

```sql
-- Code configuration for RP (inherits from CDP)
INSERT INTO code_config_document_type VALUES 
(5, 3, 1, true, 1, '{"inherit_from_cdp": true}', true),  -- Mandatory item
(6, 3, 2, true, 2, '{"inherit_from_cdp": true}', true); -- Mandatory source
```

### 4.3 RP Creation

```sql
-- RP Document
INSERT INTO budget_document VALUES 
(3, 3, 'RP-2025-001', '2025-03-01', NULL, 'ISSUED', 120000000.00, 120000000.00, 
 'RP for institutional strengthening consulting contract', 4, 5, '2025-03-01 10:30:00', 
 '{"contract": "CNT-2025-001", "contractor": "Consultores Asociados SAS", "term": "6 months"}', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');
```

### 4.4 RP Items

```sql
-- RP Item 1: Professional Services with Own Resources (proportional distribution)
INSERT INTO budget_document_item VALUES 
(7, 3, 1, 'Consulting contract - Own Resources', 80000000.00, 80000000.00, 
 5, '{"contract": "CNT-2025-001", "cdp_percentage": 66.67}', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');

-- RP Item 2: Professional Services with SGP (proportional distribution)
INSERT INTO budget_document_item VALUES 
(8, 3, 2, 'Consulting contract - SGP', 40000000.00, 40000000.00, 
 6, '{"contract": "CNT-2025-001", "cdp_percentage": 66.67}', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');
```

### 4.5 RP Coding

```sql
-- Coding RP Item 1: Professional Services + Own Resources
INSERT INTO budget_item_coding VALUES 
(13, 7, 1, 4, '2025-03-01 10:00:00', '2025-03-01 10:00:00'), -- Item: Professional Services
(14, 7, 2, 11, '2025-03-01 10:00:00', '2025-03-01 10:00:00'); -- Source: Current Income

-- Coding RP Item 2: Professional Services + SGP
INSERT INTO budget_item_coding VALUES 
(15, 8, 1, 4, '2025-03-01 10:00:00', '2025-03-01 10:00:00'), -- Item: Professional Services
(16, 8, 2, 21, '2025-03-01 10:00:00', '2025-03-01 10:00:00'); -- Source: SGP
```

### 4.6 RP Affectation Movements (With Automatic Counterparts)

```sql
-- SINGLE MOVEMENT: RP Issuance with automatic counterpart
-- Columns: id, movement_number, movement_type_id, document_origin_id, total_movement_value, movement_date, observations, document_support, user_id, approval_date, status, is_active, created_at, updated_at
INSERT INTO budget_movement VALUES 
(3, 'MOV-2025-003', 11, 3, 120000000.00, '2025-03-01', 
 'RP-2025-001 issuance with automatic CDP-2025-001 affectation', 'RP-2025-001', 4, '2025-03-01 10:30:00', 
 'APPROVED', true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');

-- LINE 1: Increase in RP (main movement)
-- Columns: id, movement_id, line_number, movement_type_id, affected_document_id, line_value, line_observations, is_active, created_at, updated_at
INSERT INTO budget_movement_detail VALUES 
(4, 3, 1, 11, 3, 120000000.00, 'Initial value of RP-2025-001', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');

-- LINE 2: Reduction in CDP (automatic counterpart)
-- Columns: id, movement_id, line_number, movement_type_id, affected_document_id, line_value, line_observations, is_active, created_at, updated_at
INSERT INTO budget_movement_detail VALUES 
(5, 3, 2, 10, 2, 120000000.00, 'Automatic counterpart: CDP-2025-001 availability reduction', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');
 
-- Note: movement_type_id = 11 'INITIAL_RP_VALUE' automatically generates type_id = 10 'AVAILABILITY_REDUCTION'
-- Note: Single movement (MOV-2025-003) with two lines affecting different documents
```

### 4.7 Movement Item Detail - RP

```sql
-- Item detail for RP movement (MOV-003)
-- Columns: id, movement_id, item_id, target_item_id, detail_value, observations, created_at, updated_at
INSERT INTO movement_item_detail VALUES 
-- Items affected in RP (line 1 of movement)
(9, 3, 7, 5, 80000000.00, 'RP Own Resources from CDP Phase 1', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00'),
(10, 3, 8, 6, 40000000.00, 'RP SGP from CDP Phase 2', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00'),

-- Items affected in CDP (line 2 of movement - counterpart)
(11, 3, 5, NULL, 80000000.00, 'Reduction: CDP Phase 1 by RP', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00'),
(12, 3, 6, NULL, 40000000.00, 'Reduction: CDP Phase 2 by RP', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00');

-- Note: Both movement lines share the same item detail
-- Note: target_item_id indicates source item when it's a transfer
```

### 4.8 RP Relationship with CDP

```sql
-- RP relationship with CDP (single movement with counterpart)
-- Columns: id, document_origin_id, document_target_id, relation_type, relation_value, relation_percentage, relation_metadata, relation_date, is_active, created_at, updated_at
INSERT INTO budget_document_relation VALUES 
(2, 2, 3, 'INCORPORATES', 120000000.00, 66.67, 
 '{"operation_type": "COMMITMENT", "contract": "CNT-2025-001", "award": "2025-02-28", "movement_id": 3, "lines": [1, 2]}', 
 '2025-03-01', true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');
 
-- Note: relation_metadata includes the unique movement ID and its composing lines
```

### 4.9 State After Step 4

**Automatic Update by Movement:**
```sql
-- CDP changes to partially COMMITTED
-- Columns in UPDATE: current_state, current_total_value, updated_at
UPDATE budget_document SET 
    current_state = 'COMMITTED',
    current_total_value = 60000000.00,  -- $180M - $120M = $60M available
    updated_at = '2025-03-01 10:30:00'
WHERE id = 2; -- CDP-2025-001

-- Update CDP items
-- Columns in UPDATE: current_value, updated_at
UPDATE budget_document_item SET 
    current_value = 40000000.00,  -- $120M - $80M = $40M available
    updated_at = '2025-03-01 10:30:00'
WHERE id = 5; -- CDP Item 1

-- Columns in UPDATE: current_value, updated_at
UPDATE budget_document_item SET 
    current_value = 20000000.00,  -- $60M - $40M = $20M available
    updated_at = '2025-03-01 10:30:00'
WHERE id = 6; -- CDP Item 2
```

**State Summary:**
- **PG**: $500,000,000 (no changes in total)
- **CDP**: $180,000,000 total, $120,000,000 committed, $60,000,000 available
- **RP**: $120,000,000 issued and active
- **Contract**: CNT-2025-001 for $120,000,000

---

## STEP 5: Payment Order

### 5.1 Obligation Accrual

**Date**: 2025-04-15
**Concept**: First contract payment (50% = $60,000,000)
**Documents**: Satisfactory receipt certificate, invoice, compliance certificate

### 5.2 Payment Order Configuration

```sql
-- Code configuration for PO (inherits from RP)
INSERT INTO code_config_document_type VALUES 
(7, 4, 1, true, 1, '{"inherit_from_rp": true}', true),  -- Mandatory item
(8, 4, 2, true, 2, '{"inherit_from_rp": true}', true); -- Mandatory source
```

### 5.3 Payment Order Creation

```sql
-- PO Document
INSERT INTO budget_document VALUES 
(4, 4, 'PO-2025-001', '2025-04-15', NULL, 'ISSUED', 60000000.00, 60000000.00, 
 'Payment order first advance consulting contract', 6, 7, '2025-04-15 14:30:00', 
 '{"contract": "CNT-2025-001", "payment_type": "FIRST_ADVANCE", "percentage": 50, "invoice": "FC-001"}', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:30:00');
```

### 5.4 Payment Order Items

```sql
-- PO Item 1: Professional Services with Own Resources
INSERT INTO budget_document_item VALUES 
(9, 4, 1, 'First contract payment - Own Resources', 40000000.00, 40000000.00, 
 7, '{"contract": "CNT-2025-001", "rp_percentage": 50}', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:00:00');

-- PO Item 2: Professional Services with SGP
INSERT INTO budget_document_item VALUES 
(10, 4, 2, 'First contract payment - SGP', 20000000.00, 20000000.00, 
 8, '{"contract": "CNT-2025-001", "rp_percentage": 50}', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:00:00');
```

### 5.5 PO Coding

```sql
-- Coding PO Item 1: Professional Services + Own Resources
INSERT INTO budget_item_coding VALUES 
(17, 9, 1, 4, '2025-04-15 14:00:00', '2025-04-15 14:00:00'), -- Item: Professional Services
(18, 9, 2, 11, '2025-04-15 14:00:00', '2025-04-15 14:00:00'); -- Source: Current Income

-- Coding PO Item 2: Professional Services + SGP
INSERT INTO budget_item_coding VALUES 
(19, 10, 1, 4, '2025-04-15 14:00:00', '2025-04-15 14:00:00'), -- Item: Professional Services
(20, 10, 2, 21, '2025-04-15 14:00:00', '2025-04-15 14:00:00'); -- Source: SGP
```

### 5.6 PO Relationship with RP

```sql
-- PO relationship with RP
INSERT INTO budget_document_relation VALUES 
(3, 3, 4, 'INCORPORATES', 60000000.00, 50.00, 
 '{"operation_type": "OBLIGATION", "invoice": "FC-001", "certificate": "AR-001"}', 
 '2025-04-15', true, '2025-04-15 14:00:00', '2025-04-15 14:00:00');
```

### 5.7 State After Step 5

**State Updates:**
```sql
-- RP changes to partially OBLIGATED
UPDATE budget_document SET 
    current_state = 'OBLIGATED',
    current_total_value = 60000000.00,  -- $120M - $60M = $60M to pay
    updated_at = '2025-04-15 14:30:00'
WHERE id = 3; -- RP-2025-001

-- Update RP items
UPDATE budget_document_item SET 
    current_value = 40000000.00,  -- $80M - $40M = $40M to pay
    updated_at = '2025-04-15 14:30:00'
WHERE id = 7; -- RP Item 1

UPDATE budget_document_item SET 
    current_value = 20000000.00,  -- $40M - $20M = $20M to pay
    updated_at = '2025-04-15 14:30:00'
WHERE id = 8; -- RP Item 2
```

**State Summary:**
- **PG**: $500,000,000 (no changes in total)
- **CDP**: $180,000,000 total, $120,000,000 committed, $60,000,000 available
- **RP**: $120,000,000 total, $60,000,000 obligated, $60,000,000 to obligate
- **PO**: $60,000,000 issued and pending payment

---

## STEP 6: CDP Remaining Balance Release

### 6.1 Situation

**Date**: 2025-05-10
**Reason**: Contract executed for $120,000,000 but CDP was for $180,000,000
**Balance to release**: $60,000,000 ($40M Own Resources + $20M SGP)

### 6.2 Release Movement

```sql
-- CDP release movement
INSERT INTO budget_movement VALUES 
(1, 'MOV-2025-001', 'RELEASE', NULL, NULL, 60000000.00, '2025-05-10', 
 'Release unexecuted balance CDP-2025-001', 'Memo-001-2025', 6, '2025-05-10 16:00:00', 
 'APPROVED', true, '2025-05-10 15:30:00', '2025-05-10 16:00:00');
```

### 6.3 Movement Detail

```sql
-- Movement detail by document
INSERT INTO movement_document_detail VALUES 
(1, 1, 2, NULL, 60000000.00, 'Total release of available balance CDP-2025-001', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00');

-- Movement detail by items
INSERT INTO movement_item_detail VALUES 
(1, 1, 5, NULL, 40000000.00, 'Release Own Resources balance', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00'),
(2, 1, 6, NULL, 20000000.00, 'Release SGP balance', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00');
```

### 6.4 Final State After Step 6

**State Updates:**
```sql
-- CDP changes to partially RELEASED
UPDATE budget_document SET 
    current_state = 'RELEASED',
    current_total_value = 0.00,  -- All available balance released
    updated_at = '2025-05-10 16:00:00'
WHERE id = 2; -- CDP-2025-001

-- Update CDP items to zero
UPDATE budget_document_item SET 
    current_value = 0.00,
    updated_at = '2025-05-10 16:00:00'
WHERE id IN (5, 6); -- Both CDP items

-- Restore availability in PG
UPDATE budget_document_item SET 
    current_value = current_value + 40000000.00,  -- $80M + $40M = $120M
    updated_at = '2025-05-10 16:00:00'
WHERE id = 1; -- Professional Services + Own Resources

UPDATE budget_document_item SET 
    current_value = current_value + 20000000.00,  -- $40M + $20M = $60M
    updated_at = '2025-05-10 16:00:00'
WHERE id = 3; -- Professional Services + SGP
```

---

## FLOW FINAL SUMMARY

### Final Document State

| Document | State | Total Value | Current Value | Description |
|----------|-------|-------------|---------------|-------------|
| **PG-2025-001** | CURRENT | $500,000,000 | $500,000,000 | Base budget restored |
| **CDP-2025-001** | RELEASED | $180,000,000 | $0 | CDP released after partial use |
| **RP-2025-001** | OBLIGATED | $120,000,000 | $60,000,000 | RP with partial obligation |
| **PO-2025-001** | ISSUED | $60,000,000 | $60,000,000 | Pending payment order |

### Final Availability by Item/Source

| Item/Source | Appropriation | Committed | Obligated | Available |
|-------------|---------------|-----------|-----------|
| **Professional Services + OR** | $200,000,000 | $80,000,000 | $40,000,000 | $120,000,000 |
| **Technical Services + OR** | $150,000,000 | $0 | $0 | $150,000,000 |
| **Professional Services + SGP** | $100,000,000 | $40,000,000 | $20,000,000 | $60,000,000 |
| **Technical Services + SGP** | $50,000,000 | $0 | $0 | $50,000,000 |

### Complete Traceability

```
PG-2025-001 ($500M)
├── CDP-2025-001 ($180M) → RELEASED
│   ├── RP-2025-001 ($120M) → OBLIGATED
│   │   └── PO-2025-001 ($60M) → ISSUED
│   └── Release ($60M) → Returned to PG
└── Available ($380M) → Available for new CDPs
```

### Verification Queries

```sql
-- Verify current availability
SELECT 
    i.id,
    i.item_description,
    i.initial_value,
    i.current_value,
    i.initial_value - i.current_value as committed_obligated
FROM budget_document_item i
WHERE i.document_id = 1 AND i.is_active = true;

-- Verify complete traceability
SELECT 
    so.document_number as origin,
    ta.document_number as target,
    bd.relation_type,
    bd.relation_value,
    bd.relation_date
FROM budget_document_relation bd
JOIN budget_document so ON bd.source_document_id = so.id
JOIN budget_document ta ON bd.target_document_id = ta.id
WHERE bd.is_active = true
ORDER BY bd.relation_date;
```

This example demonstrates how the system handles real budget management flexibility, allowing partial commitments, releases, and maintaining complete traceability at all times.

---

## CONFIGURATION: Movement Type Normalization

### Movement Type Table

```sql
-- Master table for movement types
-- Columns: id, code, name, effect, description, is_active, created_at, updated_at
CREATE TABLE movement_type (
    id SERIAL PRIMARY KEY,
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(200) NOT NULL,
    effect VARCHAR(20) NOT NULL CHECK (effect IN ('INCREASES', 'REDUCES')),
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Movement types for General Budget (PG)
INSERT INTO movement_type VALUES 
(1, 'INITIAL_APPROPRIATION', 'Initial Appropriation', 'INCREASES', 'Initial budget allocation', true, NOW(), NOW()),
(2, 'BUDGET_ADDITION', 'Budget Addition', 'INCREASES', 'Budget increase', true, NOW(), NOW()),
(3, 'BUDGET_REDUCTION', 'Budget Reduction', 'REDUCES', 'Budget decrease', true, NOW(), NOW()),
(4, 'BUDGET_TRANSFER_SOURCE', 'Budget Transfer - Source', 'REDUCES', 'Transfer movement reducing source item', true, NOW(), NOW()),
(5, 'BUDGET_TRANSFER_TARGET', 'Budget Transfer - Target', 'INCREASES', 'Transfer movement increasing target item', true, NOW(), NOW()),
(6, 'AVAILABILITY_RESTORATION', 'Availability Restoration', 'INCREASES', 'Increase due to CDP release', true, NOW(), NOW()),
(7, 'CDP_AFFECTATION', 'CDP Affectation', 'REDUCES', 'Budget reduction due to CDP issuance', true, NOW(), NOW()),

-- Movement types for Budget Availability Certificate (CDP)
(8, 'INITIAL_CDP_VALUE', 'Initial CDP Value', 'INCREASES', 'Initial CDP assignment', true, NOW(), NOW()),
(9, 'CDP_RELEASE', 'CDP Release', 'REDUCES', 'Return of unused resources', true, NOW(), NOW()),
(10, 'AVAILABILITY_REDUCTION', 'Availability Reduction', 'REDUCES', 'Decrease due to commitments', true, NOW(), NOW()),

-- Movement types for Budget Commitment (RP)
(11, 'INITIAL_RP_VALUE', 'Initial RP Value', 'INCREASES', 'Initial RP assignment', true, NOW(), NOW()),
(12, 'COMMITMENT_REDUCTION', 'Commitment Reduction', 'REDUCES', 'Decrease due to obligations', true, NOW(), NOW()),
(13, 'RP_LIQUIDATION', 'RP Liquidation', 'REDUCES', 'Liquidation and return of unexecuted balance to CDP', true, NOW(), NOW()),

-- Movement types for Payment Order (OP)
(14, 'INITIAL_OP_VALUE', 'Initial OP Value', 'INCREASES', 'Initial OP assignment', true, NOW(), NOW()),

-- Movement types for CDP Release (as independent document)
(16, 'INITIAL_CDP_RELEASE_VALUE', 'Initial CDP Release Value', 'INCREASES', 'Initial CDP Release assignment', true, NOW(), NOW()),

-- Movement types for RP Liquidation (as independent document)
(17, 'INITIAL_RP_LIQUIDATION_VALUE', 'Initial RP Liquidation Value', 'INCREASES', 'Initial RP Liquidation assignment', true, NOW(), NOW()),

-- Movement types for CDP (from RP liquidation)
(15, 'RESTORATION_BY_RP_LIQUIDATION', 'Restoration by RP Liquidation', 'INCREASES', 'CDP increase due to RP liquidation', true, NOW(), NOW());
```

### Movement Type - Document Type Relationship

```sql
-- Table relating movement types to document types
-- Columns: id, movement_type_id, document_type_id, is_active, created_at, updated_at
CREATE TABLE movement_type_document_type (
    id SERIAL PRIMARY KEY,
    movement_type_id INTEGER NOT NULL REFERENCES movement_type(id),
    document_type_id INTEGER NOT NULL REFERENCES document_type(id),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(movement_type_id, document_type_id)
);

-- Movement types for each document type
INSERT INTO movement_type_document_type VALUES 
-- General Budget (document_type_id = 1)
(1, 1, 1, true, NOW(), NOW()),  -- INITIAL_APPROPRIATION
(2, 2, 1, true, NOW(), NOW()),  -- BUDGET_ADDITION
(3, 3, 1, true, NOW(), NOW()),  -- BUDGET_REDUCTION
(4, 4, 1, true, NOW(), NOW()),  -- BUDGET_TRANSFER_SOURCE
(5, 5, 1, true, NOW(), NOW()),  -- BUDGET_TRANSFER_TARGET
(6, 6, 1, true, NOW(), NOW()),  -- AVAILABILITY_RESTORATION
(7, 7, 1, true, NOW(), NOW()),  -- CDP_AFFECTATION

-- Budget Availability Certificate (document_type_id = 2)
(8, 8, 2, true, NOW(), NOW()),  -- INITIAL_CDP_VALUE
(9, 9, 2, true, NOW(), NOW()),  -- CDP_RELEASE
(10, 10, 2, true, NOW(), NOW()), -- AVAILABILITY_REDUCTION
(11, 15, 2, true, NOW(), NOW()), -- RESTORATION_BY_RP_LIQUIDATION

-- Budget Commitment (document_type_id = 3)
(12, 11, 3, true, NOW(), NOW()), -- INITIAL_RP_VALUE
(13, 12, 3, true, NOW(), NOW()), -- COMMITMENT_REDUCTION
(14, 13, 3, true, NOW(), NOW()), -- RP_LIQUIDATION

-- Payment Order (document_type_id = 4)
(15, 14, 4, true, NOW(), NOW()), -- INITIAL_OP_VALUE

-- CDP Release (document_type_id = 8)
(16, 16, 8, true, NOW(), NOW()), -- INITIAL_CDP_RELEASE_VALUE

-- RP Liquidation (document_type_id = 9)
(17, 17, 9, true, NOW(), NOW()); -- INITIAL_RP_LIQUIDATION_VALUE
```

-- Note: Document type IDs are assumed as:
-- 1 = PG, 2 = CDP, 3 = RP, 4 = OP, 5 = Disbursement, 6 = Accounts Payable, 7 = Budget Addition
-- 8 = CDP Release, 9 = RP Liquidation

### Automatic Counterpart Configuration

```sql
-- Table defining automatic counterparts for each movement type
-- Columns: id, main_movement_type_id, counterpart_movement_type_id, execution_order, is_automatic, description, is_active, created_at, updated_at
CREATE TABLE movement_type_counterpart (
    id SERIAL PRIMARY KEY,
    main_movement_type_id INTEGER NOT NULL REFERENCES movement_type(id),
    counterpart_movement_type_id INTEGER NOT NULL REFERENCES movement_type(id),
    execution_order INTEGER DEFAULT 1,
    is_automatic BOOLEAN DEFAULT TRUE,
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(main_movement_type_id, counterpart_movement_type_id)
);

-- Automatic counterpart configuration
INSERT INTO movement_type_counterpart VALUES 
-- INITIAL_CDP_VALUE (8) automatically generates CDP_AFFECTATION (7)
(1, 8, 7, 1, true, 'When creating a CDP, PG must be automatically affected', true, NOW(), NOW()),

-- INITIAL_RP_VALUE (11) automatically generates AVAILABILITY_REDUCTION (10)
(2, 11, 10, 1, true, 'When creating an RP, CDP must be automatically reduced', true, NOW(), NOW()),

-- INITIAL_OP_VALUE (14) automatically generates COMMITMENT_REDUCTION (12)
(3, 14, 12, 1, true, 'When creating an OP, RP must be automatically reduced', true, NOW(), NOW()),

-- INITIAL_CDP_RELEASE_VALUE (16) automatically generates CDP_RELEASE (9)
(4, 16, 9, 1, true, 'When creating a CDP Release, CDP must be automatically reduced', true, NOW(), NOW()),

-- INITIAL_RP_LIQUIDATION_VALUE (17) automatically generates RP_LIQUIDATION (13)
(5, 17, 13, 1, true, 'When creating an RP Liquidation, RP must be automatically reduced', true, NOW(), NOW()),

-- CDP_RELEASE (9) automatically generates AVAILABILITY_RESTORATION (6)
(6, 9, 6, 1, true, 'When releasing a CDP, PG must be automatically restored', true, NOW(), NOW()),

-- RP_LIQUIDATION (13) automatically generates RESTORATION_BY_RP_LIQUIDATION (15)
(7, 13, 15, 1, true, 'When liquidating an RP, CDP must be automatically restored', true, NOW(), NOW());
```

### Function to Create Movements with Automatic Counterparts

```sql
-- Function to automatically create movement lines with counterparts
CREATE OR REPLACE FUNCTION create_movement_with_counterparts(
    p_movement_type_id INTEGER,
    p_document_origin_id INTEGER,
    p_total_value DECIMAL(15,2),
    p_date DATE,
    p_observations TEXT,
    p_document_support VARCHAR(100),
    p_user_id INTEGER
) RETURNS INTEGER AS $$
DECLARE
    v_movement_id INTEGER;
    v_counterpart_type_id INTEGER;
    v_line_number INTEGER := 1;
BEGIN
    -- Create main movement
    INSERT INTO budget_movement (
        movement_number, movement_type_id, document_origin_id, total_movement_value,
        movement_date, observations, document_support, user_id, approval_date,
        status, is_active, created_at, updated_at
    ) VALUES (
        'MOV-' || TO_CHAR(CURRENT_DATE, 'YYYY') || '-' || LPAD(NEXTVAL('movement_seq')::TEXT, 3, '0'),
        p_movement_type_id, p_document_origin_id, p_total_value, p_date,
        p_observations, p_document_support, p_user_id, CURRENT_TIMESTAMP,
        'APPROVED', true, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP
    ) RETURNING id INTO v_movement_id;

    -- Create main line
    INSERT INTO budget_movement_detail (
        movement_id, line_number, movement_type_id, affected_document_id,
        line_value, line_observations, is_active, created_at, updated_at
    ) VALUES (
        v_movement_id, v_line_number, p_movement_type_id, p_document_origin_id,
        p_total_value, 'Main movement line', true, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP
    );

    -- Create automatic counterpart lines
    FOR v_counterpart_type_id IN 
        SELECT counterpart_movement_type_id 
        FROM movement_type_counterpart 
        WHERE main_movement_type_id = p_movement_type_id 
        AND is_automatic = true
        ORDER BY execution_order
    LOOP
        v_line_number := v_line_number + 1;
        
        INSERT INTO budget_movement_detail (
            movement_id, line_number, movement_type_id, affected_document_id,
            line_value, line_observations, is_active, created_at, updated_at
        ) VALUES (
            v_movement_id, v_line_number, v_counterpart_type_id, 
            -- Determine affected document based on counterpart type
            CASE 
                WHEN v_counterpart_type_id IN (7, 6) THEN 1  -- PG
                WHEN v_counterpart_type_id IN (9, 10, 15) THEN 2  -- CDP
                WHEN v_counterpart_type_id IN (12, 13) THEN 3  -- RP
                ELSE p_document_origin_id
            END,
            p_total_value, 'Automatic counterpart', true, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP
        );
    END LOOP;

    RETURN v_movement_id;
END;
$$ LANGUAGE plpgsql;
```

---

## NEW MODEL: CDP RELEASES AND RP LIQUIDATIONS AS INDEPENDENT DOCUMENTS

### Document Type Extensions

The updated model treats CDP releases and RP liquidations as independent budget documents, providing better traceability and auditability.

**New Document Types:**
- **CDP Release** (document_type_id = 8) - Independent document
- **RP Liquidation** (document_type_id = 9) - Independent document

### Updated Movement Flow

**CDP Release Process:**
1. **MOV-XXX**: CDP Release document issuance
   - Line 1: +$Amount → CDP Release document (new document)
   - Line 2: -$Amount → CDP (automatic counterpart)
2. **MOV-XXX-B**: Automatic restoration to PG
   - Line 1: +$Amount → PG (restoration)

**RP Liquidation Process:**
1. **MOV-XXX**: RP Liquidation document issuance
   - Line 1: +$Amount → RP Liquidation document (new document)
   - Line 2: -$Amount → RP (automatic counterpart)
2. **MOV-XXX-B**: Automatic restoration to CDP
   - Line 1: +$Amount → CDP (restoration)

### Advantages of the New Model

1. **Complete Traceability**
   - Each release/liquidation has its own document with unique number
   - Enhanced audit trail for administrative decisions
   - Specific reports by document type

2. **Separation of Responsibilities**
   - Document creation: Administrative decision recording
   - Budget affectation: Automatic impact on related documents
   - Restoration: Automatic resource return to origin

3. **Double Counterpart**
   - Document increment + automatic reduction + automatic restoration
   - Maintains budget balance throughout the process

4. **Operational Flexibility**
   - Partial releases: Multiple releases from same CDP
   - Staged liquidations: Liquidation by phases of same RP
   - Cancellations: Reversal of releases/liquidations with traceability

5. **Referential Integrity**
   - Document relationships: Each release/liquidation references its source document
   - Automatic validations: Value and state controls
   - Reconciliation: Automatic balance between documents

6. **Accounting Benefits**
   - Accounts receivable: Releases generate state receivables
   - Accounts payable: Liquidations generate contractor payables
   - Financial statements: Better representation of financial position

### Model Comparison

| Aspect | Previous Model | New Model |
|---------|----------------|-----------|
| **CDP Release** | Simple movement | Document + Movement |
| **RP Liquidation** | Simple movement | Document + Movement |
| **Traceability** | Limited | Complete |
| **Auditability** | Basic | Advanced |
| **Flexibility** | Rigid | High |
| **Reports** | Limited | Complete |
| **Integrity** | Manual | Automatic |

This enhanced model provides a more robust, auditable, and flexible framework while maintaining budget integrity and providing better traceability for administrative operations.
