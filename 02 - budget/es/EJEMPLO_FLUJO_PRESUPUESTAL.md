# Ejemplo de Flujo Presupuestal Completo

## Descripción del Escenario

**Entidad**: Alcaldía Municipal de Ejemplo  
**Vigencia**: 2025  
**Proceso**: Contratación de servicios de consultoría para fortalecimiento institucional

---

## PASO 1: Estructura de Códigos

### 1.1 Configuración de Ámbitos y Tipos de Código

```sql
-- Ámbito: Presupuesto Municipal
-- Columnas: id, codigo, nombre, descripcion, es_activo
INSERT INTO ambito VALUES 
(1, 'PRESUPUESTO_MUNICIPAL', 'Presupuesto Municipal', 'Clasificación presupuestal municipal', true);

-- Tipos de Código
-- Columnas: id, ambito_id, codigo, nombre, descripcion, es_activo
INSERT INTO tipo_codigo VALUES 
(1, 1, 'RUBRO', 'Rubro Presupuestal', 'Clasificación por rubro de gasto', true),
(2, 1, 'FUENTE', 'Fuente de Financiación', 'Origen de los recursos', true);
```

### 1.2 Estructura de Rubros

```sql
-- Códigos de Rubros
-- Columnas: id, tipo_codigo_id, codigo_padre_id, codigo, nombre, descripcion, es_activo
INSERT INTO codigo VALUES 
(1, 1, NULL, '2.2', 'GASTOS GENERALES', 'Gastos de funcionamiento y operación', true),
(2, 1, 1, '2.2.01', 'GASTOS DE PERSONAL', 'Remuneración y prestaciones del personal', true),
(3, 1, 1, '2.2.02', 'GASTOS GENERALES', 'Gastos diversos de funcionamiento', true),
(4, 1, 3, '2.2.02.01', 'SERVICIOS PROFESIONALES', 'Contratación de servicios profesionales', true),
(5, 1, 3, '2.2.02.02', 'SERVICIOS TÉCNICOS', 'Contratación de servicios técnicos', true);
```

### 1.3 Estructura de Fuentes

```sql
-- Códigos de Fuentes
-- Columnas: id, tipo_codigo_id, codigo_padre_id, codigo, nombre, descripcion, es_activo
INSERT INTO codigo VALUES 
(10, 2, NULL, '11', 'RECURSOS PROPIOS', 'Recursos propios de la entidad', true),
(11, 2, 10, '11.1', 'INGRESOS CORRIENTES', 'Ingresos corrientes de libre destinación', true),
(12, 2, 10, '11.2', 'RECURSOS DE CAPITAL', 'Recursos de capital propios', true),
(20, 2, NULL, '12', 'TRANSFERENCIAS', 'Transferencias del nivel nacional', true),
(21, 2, 20, '12.1', 'SGP - PROPÓSITO GENERAL', 'Sistema General de Participaciones', true);
```

---

## PASO 2: Apropiación Inicial del Presupuesto

### 2.1 Configuración de Documento Tipo PG

```sql
-- Configuración de códigos requeridos para PG
-- Columnas: id, tipo_documento_id, tipo_codigo_id, es_obligatorio, orden_captura, validaciones_adicionales, es_activo
INSERT INTO configuracion_codigo_tipo_documento VALUES 
(1, 1, 1, true, 1, '{"validar_jerarquia": true}', true),  -- Rubro obligatorio
(2, 1, 2, true, 2, '{"validar_coherencia": true}', true); -- Fuente obligatoria
```

### 2.2 Creación del Presupuesto General

```sql
-- Documento Presupuesto General
-- Columnas: id, tipo_documento_id, numero_documento, fecha_documento, fecha_vencimiento, estado_actual, valor_total_inicial, valor_total_actual, observaciones, usuario_creacion_id, usuario_aprobacion_id, fecha_aprobacion, metadatos, es_activo, creado_en, actualizado_en
INSERT INTO documento_presupuestal VALUES 
(1, 1, 'PG-2025-001', '2025-01-01', NULL, 'VIGENTE', 500000000.00, 500000000.00, 
 'Presupuesto de funcionamiento vigencia 2025', 1, 2, '2025-01-15 10:00:00', 
 '{"decreto": "001-2025", "fecha_sancion": "2025-01-15"}', true, '2025-01-01 08:00:00', '2025-01-15 10:00:00');
```

### 2.3 Ítems del Presupuesto

```sql
-- Ítem 1: Servicios Profesionales con Recursos Propios
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(1, 1, 1, 'Servicios profesionales y consultoría', 200000000.00, 200000000.00, 
 NULL, '{"descripcion_detallada": "Contratación de servicios profesionales para fortalecimiento institucional"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');

-- Ítem 2: Servicios Técnicos con Recursos Propios  
INSERT INTO item_documento_presupuestal VALUES 
(2, 1, 2, 'Servicios técnicos especializados', 150000000.00, 150000000.00, 
 NULL, '{"descripcion_detallada": "Contratación de servicios técnicos especializados"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');

-- Ítem 3: Servicios Profesionales con SGP
INSERT INTO item_documento_presupuestal VALUES 
(3, 1, 3, 'Servicios profesionales - Fortalecimiento', 100000000.00, 100000000.00, 
 NULL, '{"descripcion_detallada": "Servicios profesionales financiados con SGP"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');

-- Ítem 4: Servicios Técnicos con SGP
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(4, 1, 4, 'Servicios técnicos - Capacitación', 50000000.00, 50000000.00, 
 NULL, '{"descripcion_detallada": "Servicios técnicos de capacitación con SGP"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');
```

### 2.4 Codificación de Ítems

```sql
-- Codificación Ítem 1: Servicios Profesionales + Recursos Propios
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(1, 1, 1, 4, '2025-01-01 08:00:00', '2025-01-01 08:00:00'), -- Rubro: Servicios Profesionales
(2, 1, 2, 11, '2025-01-01 08:00:00', '2025-01-01 08:00:00'); -- Fuente: Ingresos Corrientes

-- Codificación Ítem 2: Servicios Técnicos + Recursos Propios
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(3, 2, 1, 5, '2025-01-01 08:00:00', '2025-01-01 08:00:00'), -- Rubro: Servicios Técnicos
(4, 2, 2, 11, '2025-01-01 08:00:00', '2025-01-01 08:00:00'); -- Fuente: Ingresos Corrientes

-- Codificación Ítem 3: Servicios Profesionales + SGP
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(5, 3, 1, 4, '2025-01-01 08:00:00', '2025-01-01 08:00:00'), -- Rubro: Servicios Profesionales
(6, 3, 2, 21, '2025-01-01 08:00:00', '2025-01-01 08:00:00'); -- Fuente: SGP

-- Codificación Ítem 4: Servicios Técnicos + SGP
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(7, 4, 1, 5, '2025-01-01 08:00:00', '2025-01-01 08:00:00'), -- Rubro: Servicios Técnicos
(8, 4, 2, 21, '2025-01-01 08:00:00', '2025-01-01 08:00:00'); -- Fuente: SGP
```

### 2.5 Estado Después del Paso 2

**Resumen de Disponibilidad:**
- **Servicios Profesionales + Recursos Propios**: $200,000,000
- **Servicios Técnicos + Recursos Propios**: $150,000,000  
- **Servicios Profesionales + SGP**: $100,000,000
- **Servicios Técnicos + SGP**: $50,000,000
- **TOTAL PRESUPUESTO**: $500,000,000

---

## PASO 3: Expedición de Certificado de Disponibilidad Presupuestal (CDP)

### 3.1 Solicitud de CDP

**Necesidad**: Contratar consultoría para fortalecimiento institucional por $180,000,000
**Rubros a afectar**: 
- Servicios Profesionales + Recursos Propios: $120,000,000
- Servicios Profesionales + SGP: $60,000,000

### 3.2 Configuración CDP

```sql
-- Configuración de códigos para CDP (hereda del PG)
-- Columnas: id, tipo_documento_id, tipo_codigo_id, es_obligatorio, orden_captura, validaciones_adicionales, es_activo
INSERT INTO configuracion_codigo_tipo_documento VALUES 
(3, 2, 1, true, 1, '{"heredar_de_pg": true}', true),  -- Rubro obligatorio
(4, 2, 2, true, 2, '{"heredar_de_pg": true}', true); -- Fuente obligatoria
```

### 3.3 Creación del CDP

```sql
-- Documento CDP
-- Columnas: id, tipo_documento_id, numero_documento, fecha_documento, fecha_vencimiento, estado_actual, valor_total_inicial, valor_total_actual, observaciones, usuario_creacion_id, usuario_aprobacion_id, fecha_aprobacion, metadatos, es_activo, creado_en, actualizado_en
INSERT INTO documento_presupuestal VALUES 
(2, 2, 'CDP-2025-001', '2025-02-15', '2025-05-15', 'EXPEDIDO', 180000000.00, 180000000.00, 
 'CDP para contratación de consultoría fortalecimiento institucional', 3, NULL, NULL, 
 '{"proceso": "LP-001-2025", "objeto": "Consultoría fortalecimiento institucional"}', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
```

### 3.4 Ítems del CDP

```sql
-- Ítem 1 CDP: Servicios Profesionales con Recursos Propios
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(5, 2, 1, 'Consultoría fortalecimiento - Fase 1', 120000000.00, 120000000.00, 
 1, '{"fase": "1", "descripcion": "Diagnóstico y formulación estratégica"}', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');

-- Ítem 2 CDP: Servicios Profesionales con SGP
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(6, 2, 2, 'Consultoría fortalecimiento - Fase 2', 60000000.00, 60000000.00, 
 3, '{"fase": "2", "descripcion": "Implementación y seguimiento"}', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
```

### 3.5 Codificación CDP

```sql
-- Codificación Ítem 1 CDP: Servicios Profesionales + Recursos Propios
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(9, 5, 1, 4, '2025-02-15 09:00:00', '2025-02-15 09:00:00'), -- Rubro: Servicios Profesionales
(10, 5, 2, 11, '2025-02-15 09:00:00', '2025-02-15 09:00:00'); -- Fuente: Ingresos Corrientes

-- Codificación Ítem 2 CDP: Servicios Profesionales + SGP
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(11, 6, 1, 4, '2025-02-15 09:00:00', '2025-02-15 09:00:00'), -- Rubro: Servicios Profesionales
(12, 6, 2, 21, '2025-02-15 09:00:00', '2025-02-15 09:00:00'); -- Fuente: SGP
```

### 3.6 Relación CDP con PG

```sql
-- Relaciones del CDP con el PG
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(1, 1, 2, 'ORIGINA', 180000000.00, 36.00, 
 '{"tipo_operacion": "RESERVA", "conceptos": ["Servicios Profesionales RP", "Servicios Profesionales SGP"]}', 
 '2025-02-15', true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
```

### 3.7 Estado Después del Paso 3

**Actualización de Saldos PG:**
```sql
-- Actualizar saldos del PG después del CDP
-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = 80000000.00,  -- $200M - $120M = $80M
    actualizado_en = '2025-02-15 09:00:00'
WHERE id = 1; -- Servicios Profesionales + Recursos Propios

UPDATE item_documento_presupuestal SET 
    valor_actual = 40000000.00,  -- $100M - $60M = $40M
    actualizado_en = '2025-02-15 09:00:00'
WHERE id = 3; -- Servicios Profesionales + SGP
```

**Resumen de Disponibilidad:**
- **Servicios Profesionales + Recursos Propios**: $80,000,000 (reservado: $120,000,000)
- **Servicios Técnicos + Recursos Propios**: $150,000,000 (sin cambios)
- **Servicios Profesionales + SGP**: $40,000,000 (reservado: $60,000,000)
- **Servicios Técnicos + SGP**: $50,000,000 (sin cambios)
- **CDP EXPEDIDO**: $180,000,000

---

## PASO 4: Expedición de Registro Presupuestal (RP)

### 4.1 Adjudicación del Contrato

**Resultado**: Contrato adjudicado por $120,000,000 (66.67% del CDP)
**Contratista**: Consultores Asociados SAS
**Contrato**: CNT-2025-001

### 4.2 Configuración RP

```sql
-- Configuración de códigos para RP (hereda del CDP)
-- Columnas: id, tipo_documento_id, tipo_codigo_id, es_obligatorio, orden_captura, validaciones_adicionales, es_activo
INSERT INTO configuracion_codigo_tipo_documento VALUES 
(5, 3, 1, true, 1, '{"heredar_de_cdp": true}', true),  -- Rubro obligatorio
(6, 3, 2, true, 2, '{"heredar_de_cdp": true}', true); -- Fuente obligatoria
```

### 4.3 Creación del RP

```sql
-- Documento RP
-- Columnas: id, tipo_documento_id, numero_documento, fecha_documento, fecha_vencimiento, estado_actual, valor_total_inicial, valor_total_actual, observaciones, usuario_creacion_id, usuario_aprobacion_id, fecha_aprobacion, metadatos, es_activo, creado_en, actualizado_en
INSERT INTO documento_presupuestal VALUES 
(3, 3, 'RP-2025-001', '2025-03-01', NULL, 'EXPEDIDO', 120000000.00, 120000000.00, 
 'RP para contrato consultoría fortalecimiento institucional', 4, 5, '2025-03-01 10:30:00', 
 '{"contrato": "CNT-2025-001", "contratista": "Consultores Asociados SAS", "plazo": "6 meses"}', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');
```

### 4.4 Ítems del RP

```sql
-- Ítem 1 RP: Servicios Profesionales con Recursos Propios (distribución proporcional)
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(7, 3, 1, 'Contrato consultoría - Recursos Propios', 80000000.00, 80000000.00, 
 5, '{"contrato": "CNT-2025-001", "porcentaje_cdp": 66.67}', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');

-- Ítem 2 RP: Servicios Profesionales con SGP (distribución proporcional)
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(8, 3, 2, 'Contrato consultoría - SGP', 40000000.00, 40000000.00, 
 6, '{"contrato": "CNT-2025-001", "porcentaje_cdp": 66.67}', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');
```

### 4.5 Codificación RP

```sql
-- Codificación Ítem 1 RP: Servicios Profesionales + Recursos Propios
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(13, 7, 1, 4, '2025-03-01 10:00:00', '2025-03-01 10:00:00'), -- Rubro: Servicios Profesionales
(14, 7, 2, 11, '2025-03-01 10:00:00', '2025-03-01 10:00:00'); -- Fuente: Ingresos Corrientes

-- Codificación Ítem 2 RP: Servicios Profesionales + SGP
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(15, 8, 1, 4, '2025-03-01 10:00:00', '2025-03-01 10:00:00'), -- Rubro: Servicios Profesionales
(16, 8, 2, 21, '2025-03-01 10:00:00', '2025-03-01 10:00:00'); -- Fuente: SGP
```

### 4.6 Relación RP con CDP

```sql
-- Relación del RP con el CDP
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(2, 2, 3, 'INCORPORA', 120000000.00, 66.67, 
 '{"tipo_operacion": "COMPROMISO", "contrato": "CNT-2025-001", "adjudicacion": "2025-02-28"}', 
 '2025-03-01', true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');
```

### 4.7 Estado Después del Paso 4

**Actualización de Estados:**
```sql
-- CDP cambia a COMPROMETIDO parcialmente
-- Columnas en UPDATE: estado_actual, valor_actual, actualizado_en
UPDATE documento_presupuestal SET 
    estado_actual = 'COMPROMETIDO',
    valor_actual = 60000000.00,  -- $180M - $120M = $60M disponible
    actualizado_en = '2025-03-01 10:30:00'
WHERE id = 2; -- CDP-2025-001

-- Actualizar ítems del CDP
-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = 40000000.00,  -- $120M - $80M = $40M disponible
    actualizado_en = '2025-03-01 10:30:00'
WHERE id = 5; -- Ítem 1 CDP

UPDATE item_documento_presupuestal SET 
    valor_actual = 20000000.00,  -- $60M - $40M = $20M disponible
    actualizado_en = '2025-03-01 10:30:00'
WHERE id = 6; -- Ítem 2 CDP
```

**Resumen de Estados:**
- **PG**: $500,000,000 (sin cambios en total)
- **CDP**: $180,000,000 total, $120,000,000 comprometido, $60,000,000 disponible
- **RP**: $120,000,000 expedido y activo
- **Contrato**: CNT-2025-001 por $120,000,000

---

## PASO 5: Orden de Pago

### 5.1 Causación de la Obligación

**Fecha**: 2025-04-15
**Concepto**: Primer pago del contrato (50% = $60,000,000)
**Documentos**: Acta de recibo satisfactorio, factura, paz y salvo

### 5.2 Configuración Orden de Pago

```sql
-- Configuración de códigos para OP (hereda del RP)
-- Columnas: id, tipo_documento_id, tipo_codigo_id, es_obligatorio, orden_captura, validaciones_adicionales, es_activo
INSERT INTO configuracion_codigo_tipo_documento VALUES 
(7, 4, 1, true, 1, '{"heredar_de_rp": true}', true),  -- Rubro obligatorio
(8, 4, 2, true, 2, '{"heredar_de_rp": true}', true); -- Fuente obligatoria
```

### 5.3 Creación de la Orden de Pago

```sql
-- Documento OP
-- Columnas: id, tipo_documento_id, numero_documento, fecha_documento, fecha_vencimiento, estado_actual, valor_total_inicial, valor_total_actual, observaciones, usuario_creacion_id, usuario_aprobacion_id, fecha_aprobacion, metadatos, es_activo, creado_en, actualizado_en
INSERT INTO documento_presupuestal VALUES 
(4, 4, 'OP-2025-001', '2025-04-15', NULL, 'EXPEDIDO', 60000000.00, 60000000.00, 
 'Orden de pago primer avance contrato consultoría', 6, 7, '2025-04-15 14:30:00', 
 '{"contrato": "CNT-2025-001", "tipo_pago": "PRIMER_AVANCE", "porcentaje": 50, "factura": "FC-001"}', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:30:00');
```

### 5.4 Ítems de la Orden de Pago

```sql
-- Ítem 1 OP: Servicios Profesionales con Recursos Propios
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(9, 4, 1, 'Primer pago contrato - Recursos Propios', 40000000.00, 40000000.00, 
 7, '{"contrato": "CNT-2025-001", "porcentaje_rp": 50}', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:00:00');

-- Ítem 2 OP: Servicios Profesionales con SGP
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(10, 4, 2, 'Primer pago contrato - SGP', 20000000.00, 20000000.00, 
 8, '{"contrato": "CNT-2025-001", "porcentaje_rp": 50}', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:00:00');
```

### 5.5 Codificación OP

```sql
-- Codificación Ítem 1 OP: Servicios Profesionales + Recursos Propios
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(17, 9, 1, 4, '2025-04-15 14:00:00', '2025-04-15 14:00:00'), -- Rubro: Servicios Profesionales
(18, 9, 2, 11, '2025-04-15 14:00:00', '2025-04-15 14:00:00'); -- Fuente: Ingresos Corrientes

-- Codificación Ítem 2 OP: Servicios Profesionales + SGP
-- Columnas: id, item_id, tipo_codigo_id, codigo_id, creado_en, actualizado_en
INSERT INTO codificacion_item_presupuestal VALUES 
(19, 10, 1, 4, '2025-04-15 14:00:00', '2025-04-15 14:00:00'), -- Rubro: Servicios Profesionales
(20, 10, 2, 21, '2025-04-15 14:00:00', '2025-04-15 14:00:00'); -- Fuente: SGP
```

### 5.6 Relación OP con RP

```sql
-- Relación de la OP con el RP
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(3, 3, 4, 'INCORPORA', 60000000.00, 50.00, 
 '{"tipo_operacion": "OBLIGACION", "factura": "FC-001", "acta": "AR-001"}', 
 '2025-04-15', true, '2025-04-15 14:00:00', '2025-04-15 14:00:00');
```

### 5.7 Estado Después del Paso 5

**Actualización de Estados:**
```sql
-- RP cambia a OBLIGADO parcialmente
-- Columnas en UPDATE: estado_actual, valor_actual, actualizado_en
UPDATE documento_presupuestal SET 
    estado_actual = 'OBLIGADO',
    valor_actual = 60000000.00,  -- $120M - $60M = $60M por pagar
    actualizado_en = '2025-04-15 14:30:00'
WHERE id = 3; -- RP-2025-001

-- Actualizar ítems del RP
-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = 40000000.00,  -- $80M - $40M = $40M por pagar
    actualizado_en = '2025-04-15 14:30:00'
WHERE id = 7; -- Ítem 1 RP

-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = 20000000.00,  -- $40M - $20M = $20M por pagar
    actualizado_en = '2025-04-15 14:30:00'
WHERE id = 8; -- Ítem 2 RP
```

**Resumen de Estados:**
- **PG**: $500,000,000 (sin cambios en total)
- **CDP**: $180,000,000 total, $120,000,000 comprometido, $60,000,000 disponible
- **RP**: $120,000,000 total, $60,000,000 obligado, $60,000,000 por obligar
- **OP**: $60,000,000 expedida y pendiente de pago

---

## PASO 6: Liberación del CDP Restante

### 6.1 Situación

**Fecha**: 2025-05-10
**Motivo**: El contrato se ejecutó por $120,000,000 pero el CDP era por $180,000,000
**Saldo a liberar**: $60,000,000 ($40M Recursos Propios + $20M SGP)

### 6.2 Movimiento de Liberación

```sql
-- Movimiento de liberación del CDP
-- Columnas: id, numero_movimiento, tipo_movimiento, documento_origen_id, documento_destino_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(1, 'MOV-2025-001', 'LIBERACION', NULL, NULL, 60000000.00, '2025-05-10', 
 'Liberación saldo no ejecutado CDP-2025-001', 'Memo-001-2025', 6, '2025-05-10 16:00:00', 
 'APROBADO', true, '2025-05-10 15:30:00', '2025-05-10 16:00:00');
```

### 6.3 Detalle del Movimiento

```sql
-- Detalle del movimiento por documento
-- Columnas: id, movimiento_id, documento_id, item_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_documento VALUES 
(1, 1, 2, NULL, 60000000.00, 'Liberación total del saldo disponible CDP-2025-001', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00');

-- Detalle del movimiento por ítems
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(1, 1, 5, NULL, 40000000.00, 'Liberación saldo Recursos Propios', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00'),
(2, 1, 6, NULL, 20000000.00, 'Liberación saldo SGP', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00');
```

### 6.4 Estado Final Después del Paso 6

**Actualización de Estados:**
```sql
-- CDP cambia a LIBERADO parcialmente
-- Columnas en UPDATE: estado_actual, valor_actual, actualizado_en
UPDATE documento_presupuestal SET 
    estado_actual = 'LIBERADO',
    valor_actual = 0.00,  -- Todo el saldo disponible se liberó
    actualizado_en = '2025-05-10 16:00:00'
WHERE id = 2; -- CDP-2025-001

-- Actualizar ítems del CDP a cero
-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = 0.00,
    actualizado_en = '2025-05-10 16:00:00'
WHERE id IN (5, 6); -- Ambos ítems del CDP

-- Restaurar disponibilidad en el PG
-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = valor_actual + 40000000.00,  -- $80M + $40M = $120M
    actualizado_en = '2025-05-10 16:00:00'
WHERE id = 1; -- Servicios Profesionales + Recursos Propios

-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = valor_actual + 20000000.00,  -- $40M + $20M = $60M
    actualizado_en = '2025-05-10 16:00:00'
WHERE id = 3; -- Servicios Profesionales + SGP
```

---

## RESUMEN FINAL DEL FLUJO

### Estado Final de Documentos

| Documento | Estado | Valor Total | Valor Actual | Descripción |
|-----------|--------|-------------|--------------|-------------|
| **PG-2025-001** | VIGENTE | $500,000,000 | $500,000,000 | Presupuesto base restaurado |
| **CDP-2025-001** | LIBERADO | $180,000,000 | $0 | CDP liberado después de uso parcial |
| **RP-2025-001** | OBLIGADO | $120,000,000 | $60,000,000 | RP con obligación parcial |
| **OP-2025-001** | EXPEDIDO | $60,000,000 | $60,000,000 | Orden de pago pendiente |

### Disponibilidad Final por Rubro/Fuente

| Rubro/Fuente | Apropiación | Comprometido | Obligado | Disponible |
|--------------|-------------|--------------|-----------|------------|
| **Servicios Profesionales + RP** | $200,000,000 | $80,000,000 | $40,000,000 | $120,000,000 |
| **Servicios Técnicos + RP** | $150,000,000 | $0 | $0 | $150,000,000 |
| **Servicios Profesionales + SGP** | $100,000,000 | $40,000,000 | $20,000,000 | $60,000,000 |
| **Servicios Técnicos + SGP** | $50,000,000 | $0 | $0 | $50,000,000 |

### Trazabilidad Completa

```
PG-2025-001 ($500M)
├── CDP-2025-001 ($180M) → LIBERADO
│   ├── RP-2025-001 ($120M) → OBLIGADO
│   │   └── OP-2025-001 ($60M) → EXPEDIDO
│   └── Liberación ($60M) → Devuelto al PG
└── Disponible ($380M) → Disponible para nuevos CDPs
```

### Consultas de Verificación

```sql
-- Verificar disponibilidad actual
-- Columnas: id, descripcion_item, valor_inicial, valor_actual, comprometido_obligado (calculado)
SELECT 
    i.id,
    i.descripcion_item,
    i.valor_inicial,
    i.valor_actual,
    i.valor_inicial - i.valor_actual as comprometido_obligado
FROM item_documento_presupuestal i
WHERE i.documento_id = 1 AND i.es_activo = true;

-- Verificar trazabilidad completa
-- Columnas: origen (numero_documento), destino (numero_documento), tipo_relacion, valor_relacion, fecha_relacion
SELECT 
    do.numero_documento as origen,
    dd.numero_documento as destino,
    rd.tipo_relacion,
    rd.valor_relacion,
    rd.fecha_relacion
FROM relacion_documento_presupuestal rd
JOIN documento_presupuestal do ON rd.documento_origen_id = do.id
JOIN documento_presupuestal dd ON rd.documento_destino_id = dd.id
WHERE rd.es_activo = true
ORDER BY rd.fecha_relacion;
```

Este ejemplo demuestra cómo el sistema maneja la flexibilidad real del manejo presupuestal, permitiendo compromisos parciales, liberaciones y manteniendo la trazabilidad completa en todo momento.
