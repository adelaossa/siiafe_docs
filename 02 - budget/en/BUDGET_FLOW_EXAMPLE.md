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

### 3.6 CDP Relationship with PG

```sql
-- CDP relationships with PG
INSERT INTO budget_document_relation VALUES 
(1, 1, 2, 'ORIGINATES', 180000000.00, 36.00, 
 '{"operation_type": "RESERVE", "concepts": ["Professional Services OR", "Professional Services SGP"]}', 
 '2025-02-15', true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
```

### 3.7 State After Step 3

**PG Balance Update:**
```sql
-- Update PG balances after CDP
UPDATE budget_document_item SET 
    current_value = 80000000.00,  -- $200M - $120M = $80M
    updated_at = '2025-02-15 09:00:00'
WHERE id = 1; -- Professional Services + Own Resources

UPDATE budget_document_item SET 
    current_value = 40000000.00,  -- $100M - $60M = $40M
    updated_at = '2025-02-15 09:00:00'
WHERE id = 3; -- Professional Services + SGP
```

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

### 4.6 RP Relationship with CDP

```sql
-- RP relationship with CDP
INSERT INTO budget_document_relation VALUES 
(2, 2, 3, 'INCORPORATES', 120000000.00, 66.67, 
 '{"operation_type": "COMMITMENT", "contract": "CNT-2025-001", "award": "2025-02-28"}', 
 '2025-03-01', true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');
```

### 4.7 State After Step 4

**State Updates:**
```sql
-- CDP changes to partially COMMITTED
UPDATE budget_document SET 
    current_state = 'COMMITTED',
    current_total_value = 60000000.00,  -- $180M - $120M = $60M available
    updated_at = '2025-03-01 10:30:00'
WHERE id = 2; -- CDP-2025-001

-- Update CDP items
UPDATE budget_document_item SET 
    current_value = 40000000.00,  -- $120M - $80M = $40M available
    updated_at = '2025-03-01 10:30:00'
WHERE id = 5; -- CDP Item 1

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
|-------------|---------------|-----------|-----------|-----------|
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
