# Ejemplo de Flujo Presupuestal Completo

## Descripción del Escenario

**Entidad**: Alcaldía Municipal de Ejemplo  
**Vigencia**: 2025  
**Proceso**: Contratación de servicios de consultoría para fortalecimiento institucional

## Principios de la Arquitectura Basada en Movimientos

### 1. **Todos los valores son movimientos**
- Los documentos presupuestales se crean con valores iniciales en **0.00**
- Toda apropiación, afectación o liberación se registra como un **movimiento presupuestal**
- Los valores actuales se calculan y actualizan automáticamente después de cada movimiento

### 2. **Tipos de movimiento normalizados**
- Los tipos de movimiento están normalizados en tablas específicas
- Cada tipo define automáticamente si **INCREMENTA** o **REDUCE** el valor
- Los tipos están asociados a los tipos de documento permitidos
- Esto garantiza consistencia y elimina errores de digitación

**Estructura de normalización:**
- **Tabla `movimiento_tipo`**: Define los tipos de movimiento disponibles
- **Tabla `movimiento_tipo_documento_tipo`**: Relaciona qué tipos de movimiento se pueden usar con cada tipo de documento
- **Campo `efecto`**: Determina automáticamente si el movimiento incrementa o reduce

### 3. **Movimientos dobles para afectaciones**
- Cada afectación genera **DOS movimientos separados**:
  - **Movimiento de reducción** en el documento origen
  - **Movimiento de incremento** en el documento destino
- Ambos movimientos se referencian mutuamente para mantener la trazabilidad

### 4. **Referencias cruzadas**
- Cada movimiento incluye:
  - `documento_origen_id`: Documento que genera la afectación
  - `documento_afectado_id`: Documento que recibe el impacto
  - `movimiento_relacionado_id`: Referencia al movimiento complementario
  - `movimiento_tipo_id`: Referencia al tipo de movimiento normalizado
  - `tipo_impacto`: Se determina automáticamente por el tipo de movimiento

### 5. **Auditabilidad total**
- Cada cambio de valor queda registrado como un movimiento identificable
- Los movimientos tienen fecha, usuario, documento soporte y estado
- Se puede reconstruir el estado de cualquier momento histórico

---

## CONFIGURACIÓN: Normalización de Tipos de Movimiento

### Tabla de Tipos de Movimiento

```sql
-- Tabla maestra de tipos de movimiento
-- Columnas: id, codigo, nombre, efecto, descripcion, es_activo, creado_en, actualizado_en
CREATE TABLE movimiento_tipo (
    id SERIAL PRIMARY KEY,
    codigo VARCHAR(50) NOT NULL UNIQUE,
    nombre VARCHAR(200) NOT NULL,
    efecto VARCHAR(20) NOT NULL CHECK (efecto IN ('INCREMENTA', 'REDUCE')),
    descripcion TEXT,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tipos de movimiento para Presupuesto de Gasto (PG)
INSERT INTO movimiento_tipo VALUES 
(1, 'APROPIACION_INICIAL', 'Apropiación Inicial', 'INCREMENTA', 'Dotación inicial del presupuesto', true, NOW(), NOW()),
(2, 'ADICION_PRESUPUESTAL', 'Adición Presupuestal', 'INCREMENTA', 'Incremento del presupuesto aprobado', true, NOW(), NOW()),
(3, 'REDUCCION_PRESUPUESTAL', 'Reducción Presupuestal', 'REDUCE', 'Disminución del presupuesto aprobado', true, NOW(), NOW()),
(4, 'TRASLADO_PRESUPUESTAL_ORIGEN', 'Traslado Presupuestal - Origen', 'REDUCE', 'Movimiento de traslado que reduce el rubro origen', true, NOW(), NOW()),
(5, 'TRASLADO_PRESUPUESTAL_DESTINO', 'Traslado Presupuestal - Destino', 'INCREMENTA', 'Movimiento de traslado que incrementa el rubro destino', true, NOW(), NOW()),
(6, 'RESTAURACION_DISPONIBILIDAD', 'Restauración de Disponibilidad', 'INCREMENTA', 'Aumento por liberación de CDP', true, NOW(), NOW()),
(7, 'AFECTACION_POR_CDP', 'Afectación por CDP', 'REDUCE', 'Reducción del presupuesto por expedición de CDP', true, NOW(), NOW()),

-- Tipos de movimiento para Certificado de Disponibilidad Presupuestal (CDP)
(8, 'VALOR_INICIAL_CDP', 'Valor Inicial CDP', 'INCREMENTA', 'Asignación inicial del CDP', true, NOW(), NOW()),
(9, 'LIBERACION_CDP', 'Liberación CDP', 'REDUCE', 'Devolución de recursos no utilizados', true, NOW(), NOW()),
(10, 'REDUCCION_DISPONIBILIDAD', 'Reducción Disponibilidad', 'REDUCE', 'Disminución por compromisos', true, NOW(), NOW()),

-- Tipos de movimiento para Registro Presupuestal (RP)
(11, 'VALOR_INICIAL_RP', 'Valor Inicial RP', 'INCREMENTA', 'Asignación inicial del RP', true, NOW(), NOW()),
(12, 'REDUCCION_COMPROMISO', 'Reducción Compromiso', 'REDUCE', 'Disminución por obligaciones', true, NOW(), NOW()),

-- Tipos de movimiento para Orden de Pago (OP)
(13, 'VALOR_INICIAL_OP', 'Valor Inicial OP', 'INCREMENTA', 'Asignación inicial de la OP', true, NOW(), NOW());
```

### Relación Tipo de Movimiento - Tipo de Documento

```sql
-- Tabla que relaciona tipos de movimiento con tipos de documento
-- Columnas: id, movimiento_tipo_id, tipo_documento_id, es_activo, creado_en, actualizado_en
CREATE TABLE movimiento_tipo_documento_tipo (
    id SERIAL PRIMARY KEY,
    movimiento_tipo_id INTEGER NOT NULL REFERENCES movimiento_tipo(id),
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento(id),
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(movimiento_tipo_id, tipo_documento_id)
);

-- Configuración de tipos de movimiento por tipo de documento
INSERT INTO movimiento_tipo_documento_tipo VALUES 
-- Presupuesto de Gasto (tipo_documento_id = 1)
(1, 1, 1, true, NOW(), NOW()),  -- APROPIACION_INICIAL
(2, 2, 1, true, NOW(), NOW()),  -- ADICION_PRESUPUESTAL
(3, 3, 1, true, NOW(), NOW()),  -- REDUCCION_PRESUPUESTAL
(4, 4, 1, true, NOW(), NOW()),  -- TRASLADO_PRESUPUESTAL_ORIGEN
(5, 5, 1, true, NOW(), NOW()),  -- TRASLADO_PRESUPUESTAL_DESTINO
(6, 6, 1, true, NOW(), NOW()),  -- RESTAURACION_DISPONIBILIDAD
(7, 7, 1, true, NOW(), NOW()),  -- AFECTACION_POR_CDP

-- Certificado de Disponibilidad Presupuestal (tipo_documento_id = 2)
(8, 8, 2, true, NOW(), NOW()),  -- VALOR_INICIAL_CDP
(9, 9, 2, true, NOW(), NOW()),  -- LIBERACION_CDP
(10, 10, 2, true, NOW(), NOW()), -- REDUCCION_DISPONIBILIDAD

-- Registro Presupuestal (tipo_documento_id = 3)
(11, 11, 3, true, NOW(), NOW()), -- VALOR_INICIAL_RP
(12, 12, 3, true, NOW(), NOW()), -- REDUCCION_COMPROMISO

-- Orden de Pago (tipo_documento_id = 4)
(13, 13, 4, true, NOW(), NOW()); -- VALOR_INICIAL_OP
```

### Estructura Actualizada de Movimientos

```sql
-- Tabla movimiento_presupuestal actualizada con referencia al tipo normalizado
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
CREATE TABLE movimiento_presupuestal (
    id SERIAL PRIMARY KEY,
    numero_movimiento VARCHAR(20) NOT NULL UNIQUE,
    movimiento_tipo_id INTEGER NOT NULL REFERENCES movimiento_tipo(id),
    documento_origen_id INTEGER REFERENCES documento_presupuestal(id),
    documento_afectado_id INTEGER NOT NULL REFERENCES documento_presupuestal(id),
    movimiento_relacionado_id INTEGER REFERENCES movimiento_presupuestal(id),
    valor_movimiento DECIMAL(15,2) NOT NULL,
    fecha_movimiento DATE NOT NULL,
    observaciones TEXT,
    documento_soporte VARCHAR(100),
    usuario_id INTEGER NOT NULL,
    fecha_aprobacion TIMESTAMP,
    estado VARCHAR(20) DEFAULT 'PENDIENTE',
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

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
-- Documento Presupuesto General (valores iniciales en 0, se llenan por movimientos)
-- Columnas: id, tipo_documento_id, numero_documento, fecha_documento, fecha_vencimiento, estado_actual, valor_total_inicial, valor_total_actual, observaciones, usuario_creacion_id, usuario_aprobacion_id, fecha_aprobacion, metadatos, es_activo, creado_en, actualizado_en
INSERT INTO documento_presupuestal VALUES 
(1, 1, 'PG-2025-001', '2025-01-01', NULL, 'VIGENTE', 0.00, 0.00, 
 'Presupuesto de funcionamiento vigencia 2025', 1, 2, '2025-01-15 10:00:00', 
 '{"decreto": "001-2025", "fecha_sancion": "2025-01-15"}', true, '2025-01-01 08:00:00', '2025-01-15 10:00:00');
```

### 2.3 Ítems del Presupuesto

```sql
-- Ítem 1: Servicios Profesionales con Recursos Propios (valores iniciales en 0)
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(1, 1, 1, 'Servicios profesionales y consultoría', 0.00, 0.00, 
 NULL, '{"descripcion_detallada": "Contratación de servicios profesionales para fortalecimiento institucional"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');

-- Ítem 2: Servicios Técnicos con Recursos Propios  
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(2, 1, 2, 'Servicios técnicos especializados', 0.00, 0.00, 
 NULL, '{"descripcion_detallada": "Contratación de servicios técnicos especializados"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');

-- Ítem 3: Servicios Profesionales con SGP
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(3, 1, 3, 'Servicios profesionales - Fortalecimiento', 0.00, 0.00, 
 NULL, '{"descripcion_detallada": "Servicios profesionales financiados con SGP"}', 
 true, '2025-01-01 08:00:00', '2025-01-01 08:00:00');

-- Ítem 4: Servicios Técnicos con SGP
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(4, 1, 4, 'Servicios técnicos - Capacitación', 0.00, 0.00, 
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

### 2.5 Movimientos de Apropiación Inicial

```sql
-- Movimiento de apropiación inicial del presupuesto (usando tipo normalizado)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(1, 'MOV-2025-001', 1, NULL, 1, NULL, 500000000.00, '2025-01-01', 
 'Apropiación inicial del presupuesto de funcionamiento 2025', 'Acuerdo-001-2025', 2, '2025-01-15 10:00:00', 
 'APROBADO', true, '2025-01-01 08:00:00', '2025-01-15 10:00:00');
 
-- Nota: movimiento_tipo_id = 1 corresponde a 'APROPIACION_INICIAL' que tiene efecto 'INCREMENTA'
```

### 2.6 Detalle de Movimientos por Ítem

```sql
-- Detalle del movimiento por ítems - Apropiación inicial
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(1, 1, 1, NULL, 200000000.00, 'Apropiación inicial - Servicios Profesionales con Recursos Propios', 
 '2025-01-01 08:00:00', '2025-01-01 08:00:00'),
(2, 1, 2, NULL, 150000000.00, 'Apropiación inicial - Servicios Técnicos con Recursos Propios', 
 '2025-01-01 08:00:00', '2025-01-01 08:00:00'),
(3, 1, 3, NULL, 100000000.00, 'Apropiación inicial - Servicios Profesionales con SGP', 
 '2025-01-01 08:00:00', '2025-01-01 08:00:00'),
(4, 1, 4, NULL, 50000000.00, 'Apropiación inicial - Servicios Técnicos con SGP', 
 '2025-01-01 08:00:00', '2025-01-01 08:00:00');
```

### 2.7 Actualización de Valores Después de Apropiación

```sql
-- Actualizar documento con valores de apropiación
-- Columnas en UPDATE: valor_total_inicial, valor_total_actual, actualizado_en
UPDATE documento_presupuestal SET 
    valor_total_inicial = 500000000.00,
    valor_total_actual = 500000000.00,
    actualizado_en = '2025-01-15 10:00:00'
WHERE id = 1; -- PG-2025-001

-- Actualizar ítems con valores de apropiación
-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 200000000.00,
    valor_actual = 200000000.00,
    actualizado_en = '2025-01-15 10:00:00'
WHERE id = 1; -- Servicios Profesionales + Recursos Propios

-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 150000000.00,
    valor_actual = 150000000.00,
    actualizado_en = '2025-01-15 10:00:00'
WHERE id = 2; -- Servicios Técnicos + Recursos Propios

-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 100000000.00,
    valor_actual = 100000000.00,
    actualizado_en = '2025-01-15 10:00:00'
WHERE id = 3; -- Servicios Profesionales + SGP

-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 50000000.00,
    valor_actual = 50000000.00,
    actualizado_en = '2025-01-15 10:00:00'
WHERE id = 4; -- Servicios Técnicos + SGP
```

### 2.8 Estado Después del Paso 2

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
-- Documento CDP (valores iniciales en 0, se llenan por movimientos)
-- Columnas: id, tipo_documento_id, numero_documento, fecha_documento, fecha_vencimiento, estado_actual, valor_total_inicial, valor_total_actual, observaciones, usuario_creacion_id, usuario_aprobacion_id, fecha_aprobacion, metadatos, es_activo, creado_en, actualizado_en
INSERT INTO documento_presupuestal VALUES 
(2, 2, 'CDP-2025-001', '2025-02-15', '2025-05-15', 'EXPEDIDO', 0.00, 0.00, 
 'CDP para contratación de consultoría fortalecimiento institucional', 3, NULL, NULL, 
 '{"proceso": "LP-001-2025", "objeto": "Consultoría fortalecimiento institucional"}', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
```

### 3.4 Ítems del CDP

```sql
-- Ítem 1 CDP: Servicios Profesionales con Recursos Propios (valores iniciales en 0)
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(5, 2, 1, 'Consultoría fortalecimiento - Fase 1', 0.00, 0.00, 
 1, '{"fase": "1", "descripcion": "Diagnóstico y formulación estratégica"}', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');

-- Ítem 2 CDP: Servicios Profesionales con SGP (valores iniciales en 0)
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(6, 2, 2, 'Consultoría fortalecimiento - Fase 2', 0.00, 0.00, 
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

### 3.6 Movimientos de Afectación del CDP (Dobles)

```sql
-- MOVIMIENTO 1: Reducción en el PG (documento origen)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(2, 'MOV-2025-002A', 7, 2, 1, 3, 180000000.00, '2025-02-15', 
 'Reducción de disponibilidad en PG por expedición CDP-2025-001', 'CDP-2025-001', 3, '2025-02-15 09:00:00', 
 'APROBADO', true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');

-- MOVIMIENTO 2: Incremento en el CDP (documento destino)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(3, 'MOV-2025-002B', 8, 2, 2, 2, 180000000.00, '2025-02-15', 
 'Valor inicial del CDP-2025-001 por afectación desde PG-2025-001', 'CDP-2025-001', 3, '2025-02-15 09:00:00', 
 'APROBADO', true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
 
-- Nota: movimiento_tipo_id = 7 corresponde a 'AFECTACION_POR_CDP' que tiene efecto 'REDUCE'
-- Nota: movimiento_tipo_id = 8 corresponde a 'VALOR_INICIAL_CDP' que tiene efecto 'INCREMENTA'
```

### 3.7 Detalle de Movimientos por Ítem - CDP

```sql
-- Detalle del movimiento de reducción en PG (MOV-002A)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(5, 2, 1, NULL, 120000000.00, 'Reducción: Servicios Profesionales RP por CDP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00'),
(6, 2, 3, NULL, 60000000.00, 'Reducción: Servicios Profesionales SGP por CDP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00');

-- Detalle del movimiento de incremento en CDP (MOV-002B)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(7, 3, 5, 1, 120000000.00, 'Incremento: CDP Fase 1 desde Servicios Profesionales RP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00'),
(8, 3, 6, 3, 60000000.00, 'Incremento: CDP Fase 2 desde Servicios Profesionales SGP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00');
```

### 3.8 Relación CDP con PG

```sql
-- Relaciones del CDP con el PG
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(1, 1, 2, 'ORIGINA', 180000000.00, 36.00, 
 '{"tipo_operacion": "RESERVA", "conceptos": ["Servicios Profesionales RP", "Servicios Profesionales SGP"], "movimiento_reduccion_id": 2, "movimiento_incremento_id": 3}', 
 '2025-02-15', true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
```

### 3.9 Estado Después del Paso 3

**Actualización Automática por Movimiento:**
```sql
-- Actualizar valores del CDP después del movimiento
-- Columnas en UPDATE: valor_total_inicial, valor_total_actual, actualizado_en
UPDATE documento_presupuestal SET 
    valor_total_inicial = 180000000.00,
    valor_total_actual = 180000000.00,
    actualizado_en = '2025-02-15 09:00:00'
WHERE id = 2; -- CDP-2025-001

-- Actualizar ítems del CDP después del movimiento
-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 120000000.00,
    valor_actual = 120000000.00,
    actualizado_en = '2025-02-15 09:00:00'
WHERE id = 5; -- Ítem 1 CDP

-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 60000000.00,
    valor_actual = 60000000.00,
    actualizado_en = '2025-02-15 09:00:00'
WHERE id = 6; -- Ítem 2 CDP

-- Reducir disponibilidad del PG automáticamente
-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = valor_actual - 120000000.00,  -- $200M - $120M = $80M
    actualizado_en = '2025-02-15 09:00:00'
WHERE id = 1; -- Servicios Profesionales + Recursos Propios

-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = valor_actual - 60000000.00,  -- $100M - $60M = $40M
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
-- Documento RP (valores iniciales en 0, se llenan por movimientos)
-- Columnas: id, tipo_documento_id, numero_documento, fecha_documento, fecha_vencimiento, estado_actual, valor_total_inicial, valor_total_actual, observaciones, usuario_creacion_id, usuario_aprobacion_id, fecha_aprobacion, metadatos, es_activo, creado_en, actualizado_en
INSERT INTO documento_presupuestal VALUES 
(3, 3, 'RP-2025-001', '2025-03-01', NULL, 'EXPEDIDO', 0.00, 0.00, 
 'RP para contrato consultoría fortalecimiento institucional', 4, 5, '2025-03-01 10:30:00', 
 '{"contrato": "CNT-2025-001", "contratista": "Consultores Asociados SAS", "plazo": "6 meses"}', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');
```

### 4.4 Ítems del RP

```sql
-- Ítem 1 RP: Servicios Profesionales con Recursos Propios (valores iniciales en 0)
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(7, 3, 1, 'Contrato consultoría - Recursos Propios', 0.00, 0.00, 
 5, '{"contrato": "CNT-2025-001", "porcentaje_cdp": 66.67}', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');

-- Ítem 2 RP: Servicios Profesionales con SGP (valores iniciales en 0)
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(8, 3, 2, 'Contrato consultoría - SGP', 0.00, 0.00, 
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

### 4.6 Movimientos de Afectación del RP (Dobles)

```sql
-- MOVIMIENTO 1: Reducción en el CDP (documento origen)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(4, 'MOV-2025-003A', 10, 3, 2, 5, 120000000.00, '2025-03-01', 
 'Reducción de disponibilidad en CDP por expedición RP-2025-001', 'RP-2025-001', 4, '2025-03-01 10:30:00', 
 'APROBADO', true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');
 
-- Nota: movimiento_tipo_id = 10 corresponde a 'REDUCCION_DISPONIBILIDAD' que tiene efecto 'REDUCE'

-- MOVIMIENTO 2: Incremento en el RP (documento destino)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(5, 'MOV-2025-003B', 11, 3, 3, 4, 120000000.00, '2025-03-01', 
 'Valor inicial del RP-2025-001 por afectación desde CDP-2025-001', 'RP-2025-001', 4, '2025-03-01 10:30:00', 
 'APROBADO', true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');
 
-- Nota: movimiento_tipo_id = 11 corresponde a 'VALOR_INICIAL_RP' que tiene efecto 'INCREMENTA'
```

### 4.7 Detalle de Movimientos por Ítem - RP

```sql
-- Detalle del movimiento de reducción en CDP (MOV-003A)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(9, 4, 5, NULL, 80000000.00, 'Reducción: CDP Fase 1 por RP', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00'),
(10, 4, 6, NULL, 40000000.00, 'Reducción: CDP Fase 2 por RP', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00');

-- Detalle del movimiento de incremento en RP (MOV-003B)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(11, 5, 7, 5, 80000000.00, 'Incremento: RP Recursos Propios desde CDP Fase 1', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00'),
(12, 5, 8, 6, 40000000.00, 'Incremento: RP SGP desde CDP Fase 2', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00');
```

### 4.8 Relación RP con CDP

```sql
-- Relación del RP con el CDP
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(2, 2, 3, 'INCORPORA', 120000000.00, 66.67, 
 '{"tipo_operacion": "COMPROMISO", "contrato": "CNT-2025-001", "adjudicacion": "2025-02-28", "movimiento_reduccion_id": 4, "movimiento_incremento_id": 5}', 
 '2025-03-01', true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');
```

### 4.9 Estado Después del Paso 4

**Actualización Automática por Movimiento:**
```sql
-- Actualizar valores del RP después del movimiento
-- Columnas en UPDATE: valor_total_inicial, valor_total_actual, actualizado_en
UPDATE documento_presupuestal SET 
    valor_total_inicial = 120000000.00,
    valor_total_actual = 120000000.00,
    actualizado_en = '2025-03-01 10:30:00'
WHERE id = 3; -- RP-2025-001

-- Actualizar ítems del RP después del movimiento
-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 80000000.00,
    valor_actual = 80000000.00,
    actualizado_en = '2025-03-01 10:30:00'
WHERE id = 7; -- Ítem 1 RP

-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 40000000.00,
    valor_actual = 40000000.00,
    actualizado_en = '2025-03-01 10:30:00'
WHERE id = 8; -- Ítem 2 RP

-- Reducir disponibilidad del CDP automáticamente
-- Columnas en UPDATE: estado_actual, valor_actual, actualizado_en
UPDATE documento_presupuestal SET 
    estado_actual = 'COMPROMETIDO',
    valor_actual = valor_actual - 120000000.00,  -- $180M - $120M = $60M disponible
    actualizado_en = '2025-03-01 10:30:00'
WHERE id = 2; -- CDP-2025-001

-- Actualizar ítems del CDP
-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = valor_actual - 80000000.00,  -- $120M - $80M = $40M disponible
    actualizado_en = '2025-03-01 10:30:00'
WHERE id = 5; -- Ítem 1 CDP

-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = valor_actual - 40000000.00,  -- $60M - $40M = $20M disponible
    actualizado_en = '2025-03-01 10:30:00'
WHERE id = 6; -- Ítem 2 CDP
```

**Resumen de Estados:**
- **PG**: $500,000,000 (sin cambios en total)
- **CDP**: $180,000,000 total, $120,000,000 comprometido, $60,000,000 disponible
- **RP**: $120,000,000 total, $60,000,000 obligado, $60,000,000 por obligar
- **OP**: $60,000,000 expedida y pendiente de pago

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
-- Documento OP (valores iniciales en 0, se llenan por movimientos)
-- Columnas: id, tipo_documento_id, numero_documento, fecha_documento, fecha_vencimiento, estado_actual, valor_total_inicial, valor_total_actual, observaciones, usuario_creacion_id, usuario_aprobacion_id, fecha_aprobacion, metadatos, es_activo, creado_en, actualizado_en
INSERT INTO documento_presupuestal VALUES 
(4, 4, 'OP-2025-001', '2025-04-15', NULL, 'EXPEDIDO', 0.00, 0.00, 
 'Orden de pago primer avance contrato consultoría', 6, 7, '2025-04-15 14:30:00', 
 '{"contrato": "CNT-2025-001", "tipo_pago": "PRIMER_AVANCE", "porcentaje": 50, "factura": "FC-001"}', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:30:00');
```

### 5.4 Ítems de la Orden de Pago

```sql
-- Ítem 1 OP: Servicios Profesionales con Recursos Propios (valores iniciales en 0)
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(9, 4, 1, 'Primer pago contrato - Recursos Propios', 0.00, 0.00, 
 7, '{"contrato": "CNT-2025-001", "porcentaje_rp": 50}', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:00:00');

-- Ítem 2 OP: Servicios Profesionales con SGP (valores iniciales en 0)
-- Columnas: id, documento_id, numero_item, descripcion_item, valor_inicial, valor_actual, item_origen_id, metadatos_item, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(10, 4, 2, 'Primer pago contrato - SGP', 0.00, 0.00, 
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

### 5.6 Movimientos de Afectación de la OP (Dobles)

```sql
-- MOVIMIENTO 1: Reducción en el RP (documento origen)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(6, 'MOV-2025-004A', 12, 4, 3, 7, 60000000.00, '2025-04-15', 
 'Reducción de compromiso en RP por expedición OP-2025-001', 'OP-2025-001', 6, '2025-04-15 14:30:00', 
 'APROBADO', true, '2025-04-15 14:00:00', '2025-04-15 14:30:00');

-- MOVIMIENTO 2: Incremento en la OP (documento destino)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(7, 'MOV-2025-004B', 13, 4, 4, 6, 60000000.00, '2025-04-15', 
 'Valor inicial de la OP-2025-001 por afectación desde RP-2025-001', 'OP-2025-001', 6, '2025-04-15 14:30:00', 
 'APROBADO', true, '2025-04-15 14:00:00', '2025-04-15 14:30:00');
 
-- Nota: movimiento_tipo_id = 12 corresponde a 'REDUCCION_COMPROMISO' que tiene efecto 'REDUCE'
-- Nota: movimiento_tipo_id = 13 corresponde a 'VALOR_INICIAL_OP' que tiene efecto 'INCREMENTA'
```

### 5.7 Detalle de Movimientos por Ítem - OP

```sql
-- Detalle del movimiento de reducción en RP (MOV-004A)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(13, 6, 7, NULL, 40000000.00, 'Reducción: RP Recursos Propios por OP', 
 '2025-04-15 14:00:00', '2025-04-15 14:00:00'),
(14, 6, 8, NULL, 20000000.00, 'Reducción: RP SGP por OP', 
 '2025-04-15 14:00:00', '2025-04-15 14:00:00');

-- Detalle del movimiento de incremento en OP (MOV-004B)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(15, 7, 9, 7, 40000000.00, 'Incremento: OP Recursos Propios desde RP', 
 '2025-04-15 14:00:00', '2025-04-15 14:00:00'),
(16, 7, 10, 8, 20000000.00, 'Incremento: OP SGP desde RP', 
 '2025-04-15 14:00:00', '2025-04-15 14:00:00');
```

### 5.8 Relación OP con RP

```sql
-- Relación de la OP con el RP
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(3, 3, 4, 'INCORPORA', 60000000.00, 50.00, 
 '{"tipo_operacion": "OBLIGACION", "factura": "FC-001", "acta": "AR-001", "movimiento_reduccion_id": 6, "movimiento_incremento_id": 7}', 
 '2025-04-15', true, '2025-04-15 14:00:00', '2025-04-15 14:00:00');
```

### 5.9 Estado Después del Paso 5

**Actualización Automática por Movimiento:**
```sql
-- Actualizar valores de la OP después del movimiento
-- Columnas en UPDATE: valor_total_inicial, valor_total_actual, actualizado_en
UPDATE documento_presupuestal SET 
    valor_total_inicial = 60000000.00,
    valor_total_actual = 60000000.00,
    actualizado_en = '2025-04-15 14:30:00'
WHERE id = 4; -- OP-2025-001

-- Actualizar ítems de la OP después del movimiento
-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 40000000.00,
    valor_actual = 40000000.00,
    actualizado_en = '2025-04-15 14:30:00'
WHERE id = 9; -- Ítem 1 OP

-- Columnas en UPDATE: valor_inicial, valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_inicial = 20000000.00,
    valor_actual = 20000000.00,
    actualizado_en = '2025-04-15 14:30:00'
WHERE id = 10; -- Ítem 2 OP

-- Reducir disponibilidad del RP automáticamente
-- Columnas en UPDATE: estado_actual, valor_actual, actualizado_en
UPDATE documento_presupuestal SET 
    estado_actual = 'OBLIGADO',
    valor_actual = valor_actual - 60000000.00,  -- $120M - $60M = $60M por pagar
    actualizado_en = '2025-04-15 14:30:00'
WHERE id = 3; -- RP-2025-001

-- Actualizar ítems del RP
-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = valor_actual - 40000000.00,  -- $80M - $40M = $40M por pagar
    actualizado_en = '2025-04-15 14:30:00'
WHERE id = 7; -- Ítem 1 RP

-- Columnas en UPDATE: valor_actual, actualizado_en
UPDATE item_documento_presupuestal SET 
    valor_actual = valor_actual - 20000000.00,  -- $40M - $20M = $20M por pagar
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

### 6.2 Movimientos de Liberación del CDP (Dobles)

```sql
-- MOVIMIENTO 1: Liberación en el CDP (documento origen)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(8, 'MOV-2025-005A', 9, NULL, 2, 9, 60000000.00, '2025-05-10', 
 'Liberación de saldo no ejecutado en CDP-2025-001', 'Memo-001-2025', 6, '2025-05-10 16:00:00', 
 'APROBADO', true, '2025-05-10 15:30:00', '2025-05-10 16:00:00');

-- MOVIMIENTO 2: Restauración en el PG (documento destino)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, documento_afectado_id, movimiento_relacionado_id, valor_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(9, 'MOV-2025-005B', 6, NULL, 1, 8, 60000000.00, '2025-05-10', 
 'Restauración de disponibilidad en PG por liberación CDP-2025-001', 'Memo-001-2025', 6, '2025-05-10 16:00:00', 
 'APROBADO', true, '2025-05-10 15:30:00', '2025-05-10 16:00:00');
 
-- Nota: movimiento_tipo_id = 9 corresponde a 'LIBERACION_CDP' que tiene efecto 'REDUCE'
-- Nota: movimiento_tipo_id = 6 corresponde a 'RESTAURACION_DISPONIBILIDAD' que tiene efecto 'INCREMENTA'
```

### 6.3 Detalle de Movimientos de Liberación

```sql
-- Detalle del movimiento de liberación en CDP (MOV-005A)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(17, 8, 5, NULL, 40000000.00, 'Liberación: CDP Fase 1', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00'),
(18, 8, 6, NULL, 20000000.00, 'Liberación: CDP Fase 2', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00');

-- Detalle del movimiento de restauración en PG (MOV-005B)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
(19, 9, 1, 5, 40000000.00, 'Restauración: Servicios Profesionales RP desde CDP', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00'),
(20, 9, 3, 6, 20000000.00, 'Restauración: Servicios Profesionales SGP desde CDP', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00');
```

### 6.4 Estado Final Después del Paso 6

**Actualización Automática por Movimiento de Liberación:**
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

-- Restaurar disponibilidad en el PG automáticamente por movimiento inverso
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

---

## DIAGRAMA DE FLUJO DE MOVIMIENTOS DOBLES CON TIPOS NORMALIZADOS

```
APROPIACION_INICIAL (MOV-001, tipo_id=1, efecto=INCREMENTA)
    ↓ INCREMENTA $500M
PG-2025-001 ($500M disponible)
    ↓ MOV-002A: AFECTACION_POR_CDP (tipo_id=7, efecto=REDUCE) (-$180M)
    ↓ MOV-002B: VALOR_INICIAL_CDP (tipo_id=8, efecto=INCREMENTA) (+$180M)
CDP-2025-001 ($180M disponible)
    ├── MOV-003A: REDUCCION_DISPONIBILIDAD (tipo_id=10, efecto=REDUCE) (-$120M)
    │   MOV-003B: VALOR_INICIAL_RP (tipo_id=11, efecto=INCREMENTA) (+$120M)
    │   ↓ RP-2025-001 ($120M disponible)
    │       ↓ MOV-004A: REDUCCION_COMPROMISO (tipo_id=12, efecto=REDUCE) (-$60M)
    │       ↓ MOV-004B: VALOR_INICIAL_OP (tipo_id=13, efecto=INCREMENTA) (+$60M)
    │       OP-2025-001 ($60M disponible)
    │
    └── MOV-005A: LIBERACION_CDP (tipo_id=9, efecto=REDUCE) (-$60M)
        MOV-005B: RESTAURACION_DISPONIBILIDAD (tipo_id=6, efecto=INCREMENTA) (+$60M)
        ↑ PG-2025-001 (saldo restaurado)
```

## ARQUITECTURA NORMALIZADA DE TIPOS DE MOVIMIENTO

```
┌─────────────────────┐    ┌──────────────────────────┐    ┌───────────────────────┐
│   movimiento_tipo   │    │ movimiento_tipo_documento│    │   tipo_documento      │
│                     │    │         _tipo            │    │                       │
│ id (PK)             │◄───┤ movimiento_tipo_id (FK)  │───►│ id (PK)               │
│ codigo              │    │ tipo_documento_id (FK)   │    │ nombre                │
│ nombre              │    │ es_activo                │    │ descripcion           │
│ efecto              │    │ creado_en                │    │ estado                │
│ descripcion         │    │ actualizado_en           │    │                       │
│ es_activo           │    └──────────────────────────┘    └───────────────────────┘
│ creado_en           │
│ actualizado_en      │
└─────────────────────┘
           │
           │
           ▼
┌─────────────────────┐
│ movimiento_presup.  │
│                     │
│ id (PK)             │
│ numero_movimiento   │
│ movimiento_tipo_id  │◄── Referencia normalizada
│ documento_origen_id │
│ documento_afectado_ │
│ movimiento_relacion │
│ valor_movimiento    │
│ fecha_movimiento    │
│ observaciones       │
│ documento_soporte   │
│ usuario_id          │
│ fecha_aprobacion    │
│ estado              │
│ es_activo           │
│ creado_en           │
│ actualizado_en      │
└─────────────────────┘
```

## PRINCIPIOS DE CONSISTENCIA CON NORMALIZACIÓN

### 1. **Movimientos Dobles Atómicos**
Cada afectación genera **DOS movimientos separados pero relacionados**:
- **Movimiento A**: Reduce el saldo del documento origen
- **Movimiento B**: Incrementa el saldo del documento destino
- Ambos movimientos se referencian mutuamente (`movimiento_relacionado_id`)

### 2. **Tipos de Movimiento Normalizados por Documento**
```sql
-- Consulta de tipos válidos para Presupuesto de Gasto
SELECT mt.id, mt.codigo, mt.nombre, mt.efecto
FROM movimiento_tipo mt
JOIN movimiento_tipo_documento_tipo mtdt ON mt.id = mtdt.movimiento_tipo_id
WHERE mtdt.tipo_documento_id = 1 -- PG
  AND mtdt.es_activo = true;

-- Resultado:
-- id | codigo                        | nombre                        | efecto
-- 1  | APROPIACION_INICIAL          | Apropiación Inicial           | INCREMENTA
-- 2  | ADICION_PRESUPUESTAL         | Adición Presupuestal          | INCREMENTA
-- 3  | REDUCCION_PRESUPUESTAL       | Reducción Presupuestal        | REDUCE
-- 4  | TRASLADO_PRESUPUESTAL_ORIGEN | Traslado Presupuestal Origen  | REDUCE
-- 5  | TRASLADO_PRESUPUESTAL_DESTINO| Traslado Presupuestal Destino | INCREMENTA
-- 6  | RESTAURACION_DISPONIBILIDAD  | Restauración Disponibilidad   | INCREMENTA
-- 7  | AFECTACION_POR_CDP           | Afectación por CDP            | REDUCE
```

### 3. **Efecto Automático por Tipo**
- El efecto (INCREMENTA/REDUCE) se determina automáticamente por `movimiento_tipo.efecto`
- No hay riesgo de error manual en la asignación del efecto
- Garantiza consistencia en todos los movimientos del mismo tipo

### 4. **Trazabilidad Completa con Información Semántica**
Cada movimiento incluye:
- `movimiento_tipo_id`: Referencia al tipo normalizado
- `documento_origen_id`: Documento que inicia la operación
- `documento_afectado_id`: Documento que recibe el impacto
- `movimiento_relacionado_id`: Movimiento complementario
- El efecto se obtiene de `movimiento_tipo.efecto`

### 5. **Validación Automática**
```sql
-- Trigger para validar tipos de movimiento permitidos
CREATE OR REPLACE FUNCTION validar_tipo_movimiento_documento()
RETURNS TRIGGER AS $$
BEGIN
    IF NOT EXISTS (
        SELECT 1 FROM movimiento_tipo_documento_tipo mtdt
        JOIN documento_presupuestal dp ON dp.tipo_documento_id = mtdt.tipo_documento_id
        WHERE mtdt.movimiento_tipo_id = NEW.movimiento_tipo_id
          AND dp.id = NEW.documento_afectado_id
          AND mtdt.es_activo = true
    ) THEN
        RAISE EXCEPTION 'Tipo de movimiento % no permitido para este tipo de documento', NEW.movimiento_tipo_id;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER validar_tipo_movimiento_documento_trigger
    BEFORE INSERT OR UPDATE ON movimiento_presupuestal
    FOR EACH ROW EXECUTE FUNCTION validar_tipo_movimiento_documento();
```

### 6. **Cadena de Afectaciones con Tipos Normalizados**
```sql
-- Ejemplo de cadena completa con tipos normalizados:
PG (Ítem 1): $200M 
    ↓ MOV-002A (tipo_id=7, REDUCE) (-$120M) → $80M 
    ↓ MOV-005B (tipo_id=6, INCREMENTA) (+$40M) → $120M (después liberación)

CDP (Ítem 1): $0 
    ↓ MOV-002B (tipo_id=8, INCREMENTA) (+$120M) → $120M 
    ↓ MOV-003A (tipo_id=10, REDUCE) (-$80M) → $40M 
    ↓ MOV-005A (tipo_id=9, REDUCE) (-$40M) → $0 (después liberación)

RP (Ítem 1): $0 
    ↓ MOV-003B (tipo_id=11, INCREMENTA) (+$80M) → $80M 
    ↓ MOV-004A (tipo_id=12, REDUCE) (-$40M) → $40M (después obligación)

OP (Ítem 1): $0 
    ↓ MOV-004B (tipo_id=13, INCREMENTA) (+$40M) → $40M (pendiente de pago)
```

### 7. **Integridad Referencial con Normalización**
- Cada movimiento referencia un tipo normalizado válido
- Los tipos están relacionados con documentos específicos
- Los detalles de movimiento mapean ítems origen y destino correctamente
- Las relaciones entre documentos incluyen IDs de ambos movimientos con sus tipos

Este enfoque normalizado garantiza la consistencia, reduce errores y facilita el mantenimiento del sistema presupuestal.
