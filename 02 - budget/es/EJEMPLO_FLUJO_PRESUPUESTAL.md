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
(13, 'LIQUIDACION_RP', 'Liquidación RP', 'REDUCE', 'Liquidación y devolución de saldo no ejecutado al CDP', true, NOW(), NOW()),

-- Tipos de movimiento para Orden de Pago (OP)
(14, 'VALOR_INICIAL_OP', 'Valor Inicial OP', 'INCREMENTA', 'Asignación inicial de la OP', true, NOW(), NOW()),

-- Tipos de movimiento para CDP (desde liquidación RP)
(15, 'RESTAURACION_POR_LIQUIDACION_RP', 'Restauración por Liquidación RP', 'INCREMENTA', 'Incremento en CDP por liquidación de RP', true, NOW(), NOW());
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
(11, 15, 2, true, NOW(), NOW()), -- RESTAURACION_POR_LIQUIDACION_RP

-- Registro Presupuestal (tipo_documento_id = 3)
(12, 11, 3, true, NOW(), NOW()), -- VALOR_INICIAL_RP
(13, 12, 3, true, NOW(), NOW()), -- REDUCCION_COMPROMISO
(14, 13, 3, true, NOW(), NOW()), -- LIQUIDACION_RP

-- Orden de Pago (tipo_documento_id = 4)
(15, 14, 4, true, NOW(), NOW()); -- VALOR_INICIAL_OP
```

### Estructura Actualizada de Movimientos con Contrapartidas

```sql
-- Tabla movimiento_presupuestal actualizada (cabecera del movimiento)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, valor_total_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
CREATE TABLE movimiento_presupuestal (
    id SERIAL PRIMARY KEY,
    numero_movimiento VARCHAR(20) NOT NULL UNIQUE,
    movimiento_tipo_id INTEGER NOT NULL REFERENCES movimiento_tipo(id),
    documento_origen_id INTEGER REFERENCES documento_presupuestal(id),
    valor_total_movimiento DECIMAL(15,2) NOT NULL,
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

-- Tabla detalle_movimiento_presupuestal actualizada (líneas del movimiento)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
CREATE TABLE detalle_movimiento_presupuestal (
    id SERIAL PRIMARY KEY,
    movimiento_id INTEGER NOT NULL REFERENCES movimiento_presupuestal(id),
    linea_numero INTEGER NOT NULL,
    movimiento_tipo_id INTEGER NOT NULL REFERENCES movimiento_tipo(id),
    documento_afectado_id INTEGER NOT NULL REFERENCES documento_presupuestal(id),
    valor_linea DECIMAL(15,2) NOT NULL,
    observaciones_linea TEXT,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(movimiento_id, linea_numero)
);
```

### Función para Crear Movimientos con Contrapartidas Automáticas

```sql
-- Función que crea automáticamente las contrapartidas al insertar un movimiento
CREATE OR REPLACE FUNCTION crear_contrapartidas_automaticas()
RETURNS TRIGGER AS $$
DECLARE
    contrapartida RECORD;
    siguiente_linea INTEGER;
BEGIN
    -- Obtener el siguiente número de línea
    SELECT COALESCE(MAX(linea_numero), 0) + 1
    INTO siguiente_linea
    FROM detalle_movimiento_presupuestal
    WHERE movimiento_id = NEW.movimiento_id;
    
    -- Buscar contrapartidas automáticas para este tipo de movimiento
    FOR contrapartida IN
        SELECT 
            mtc.movimiento_tipo_contrapartida_id,
            mtc.descripcion,
            mt.efecto
        FROM movimiento_tipo_contrapartida mtc
        JOIN movimiento_tipo mt ON mtc.movimiento_tipo_contrapartida_id = mt.id
        WHERE mtc.movimiento_tipo_principal_id = NEW.movimiento_tipo_id
          AND mtc.es_automatico = true
          AND mtc.es_activo = true
    LOOP
        -- Determinar el documento afectado por la contrapartida
        -- (Lógica específica según el tipo de movimiento)
        INSERT INTO detalle_movimiento_presupuestal (
            movimiento_id,
            linea_numero,
            movimiento_tipo_id,
            documento_afectado_id,
            valor_linea,
            observaciones_linea
        ) VALUES (
            NEW.movimiento_id,
            siguiente_linea,
            contrapartida.movimiento_tipo_contrapartida_id,
            -- Aquí se determina el documento afectado según la lógica de negocio
            CASE 
                WHEN contrapartida.movimiento_tipo_contrapartida_id = 7 THEN -- AFECTACION_POR_CDP
                    (SELECT documento_origen_id FROM movimiento_presupuestal WHERE id = NEW.movimiento_id)
                ELSE NEW.documento_afectado_id
            END,
            NEW.valor_linea,
            'Contrapartida automática: ' || contrapartida.descripcion
        );
        
        siguiente_linea := siguiente_linea + 1;
    END LOOP;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER crear_contrapartidas_automaticas_trigger
    AFTER INSERT ON detalle_movimiento_presupuestal
    FOR EACH ROW EXECUTE FUNCTION crear_contrapartidas_automaticas();
```

### Configuración de Documentos Precedentes

```sql
-- Tabla que define los documentos precedentes válidos para cada tipo de documento
-- Columnas: id, tipo_documento_id, tipo_documento_precedente_id, es_obligatorio, orden_precedencia, descripcion, es_activo, creado_en, actualizado_en
CREATE TABLE documento_tipo_precedente (
    id SERIAL PRIMARY KEY,
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento(id),
    tipo_documento_precedente_id INTEGER NOT NULL REFERENCES tipo_documento(id),
    es_obligatorio BOOLEAN DEFAULT TRUE,
    orden_precedencia INTEGER DEFAULT 1,
    descripcion TEXT,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(tipo_documento_id, tipo_documento_precedente_id, orden_precedencia)
);

-- Configuración de precedencias estándar
INSERT INTO documento_tipo_precedente VALUES 
-- CDP debe estar precedido por PG
(1, 2, 1, true, 1, 'CDP debe originarse desde un Presupuesto de Gasto', true, NOW(), NOW()),

-- RP debe estar precedido por CDP
(2, 3, 2, true, 1, 'RP debe originarse desde un CDP', true, NOW(), NOW()),

-- OP debe estar precedida por RP
(3, 4, 3, true, 1, 'OP debe originarse desde un RP', true, NOW(), NOW()),

-- Egreso puede estar precedido por OP (opción 1)
(4, 5, 4, false, 1, 'Egreso puede originarse desde una OP', true, NOW(), NOW()),

-- Egreso puede estar precedido por Cuenta por Pagar (opción 2)
(5, 5, 6, false, 2, 'Egreso puede originarse desde una Cuenta por Pagar de vigencia anterior', true, NOW(), NOW()),

-- Adición presupuestal puede estar precedida por PG (para traslados)
(6, 7, 1, false, 1, 'Adición presupuestal puede originarse desde PG para traslados', true, NOW(), NOW());

-- Nota: Los IDs de tipo_documento se asumen como:
-- 1 = PG, 2 = CDP, 3 = RP, 4 = OP, 5 = Egreso, 6 = Cuenta por Pagar, 7 = Adición Presupuestal
```

### Configuración de Contrapartidas de Movimientos

```sql
-- Tabla que define las contrapartidas automáticas para cada tipo de movimiento
-- Columnas: id, movimiento_tipo_principal_id, movimiento_tipo_contrapartida_id, orden_ejecucion, es_automatico, descripcion, es_activo, creado_en, actualizado_en
CREATE TABLE movimiento_tipo_contrapartida (
    id SERIAL PRIMARY KEY,
    movimiento_tipo_principal_id INTEGER NOT NULL REFERENCES movimiento_tipo(id),
    movimiento_tipo_contrapartida_id INTEGER NOT NULL REFERENCES movimiento_tipo(id),
    orden_ejecucion INTEGER DEFAULT 1,
    es_automatico BOOLEAN DEFAULT TRUE,
    descripcion TEXT,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(movimiento_tipo_principal_id, movimiento_tipo_contrapartida_id)
);

-- Configuración de contrapartidas automáticas
INSERT INTO movimiento_tipo_contrapartida VALUES 
-- VALOR_INICIAL_CDP (8) genera automáticamente AFECTACION_POR_CDP (7)
(1, 8, 7, 1, true, 'Al crear un CDP, se debe afectar automáticamente el PG', true, NOW(), NOW()),

-- VALOR_INICIAL_RP (11) genera automáticamente REDUCCION_DISPONIBILIDAD (10)
(2, 11, 10, 1, true, 'Al crear un RP, se debe reducir automáticamente el CDP', true, NOW(), NOW()),

-- VALOR_INICIAL_OP (14) genera automáticamente REDUCCION_COMPROMISO (12)
(3, 14, 12, 1, true, 'Al crear una OP, se debe reducir automáticamente el RP', true, NOW(), NOW()),

-- LIBERACION_CDP (9) genera automáticamente RESTAURACION_DISPONIBILIDAD (6)
(4, 9, 6, 1, true, 'Al liberar un CDP, se debe restaurar automáticamente el PG', true, NOW(), NOW()),

-- LIQUIDACION_RP (13) genera automáticamente RESTAURACION_POR_LIQUIDACION_RP (15)
(5, 13, 15, 1, true, 'Al liquidar un RP, se debe restaurar automáticamente el CDP', true, NOW(), NOW());
```

### Validación de Documentos Precedentes

```sql
-- Función para validar que un documento tenga el precedente correcto
CREATE OR REPLACE FUNCTION validar_documento_precedente()
RETURNS TRIGGER AS $$
BEGIN
    -- Solo validar si el documento tiene un origen definido
    IF NEW.documento_origen_id IS NOT NULL THEN
        IF NOT EXISTS (
            SELECT 1 FROM documento_tipo_precedente dtp
            JOIN documento_presupuestal dp_origen ON dp_origen.id = NEW.documento_origen_id
            JOIN documento_presupuestal dp_destino ON dp_destino.id = NEW.documento_afectado_id
            WHERE dtp.tipo_documento_id = dp_destino.tipo_documento_id
              AND dtp.tipo_documento_precedente_id = dp_origen.tipo_documento_id
              AND dtp.es_activo = true
        ) THEN
            RAISE EXCEPTION 'El documento origen no es un precedente válido para este tipo de documento';
        END IF;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER validar_documento_precedente_trigger
    BEFORE INSERT OR UPDATE ON movimiento_presupuestal
    FOR EACH ROW EXECUTE FUNCTION validar_documento_precedente();
```

### Consulta de Precedentes Válidos

```sql
-- Función para consultar los precedentes válidos de un tipo de documento
CREATE OR REPLACE FUNCTION obtener_precedentes_validos(p_tipo_documento_id INTEGER)
RETURNS TABLE (
    precedente_id INTEGER,
    precedente_nombre VARCHAR,
    es_obligatorio BOOLEAN,
    orden_precedencia INTEGER,
    descripcion TEXT
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        dtp.tipo_documento_precedente_id,
        td.nombre,
        dtp.es_obligatorio,
        dtp.orden_precedencia,
        dtp.descripcion
    FROM documento_tipo_precedente dtp
    JOIN tipo_documento td ON dtp.tipo_documento_precedente_id = td.id
    WHERE dtp.tipo_documento_id = p_tipo_documento_id
      AND dtp.es_activo = true
    ORDER BY dtp.orden_precedencia;
END;
$$ LANGUAGE plpgsql;

-- Ejemplo de uso:
-- SELECT * FROM obtener_precedentes_validos(5); -- Para tipo documento 'Egreso'
-- Resultado podría ser:
-- precedente_id | precedente_nombre | es_obligatorio | orden_precedencia | descripcion
-- 4             | Orden de Pago     | false          | 1                 | Egreso puede originarse desde una OP
-- 6             | Cuenta por Pagar  | false          | 2                 | Egreso puede originarse desde una Cuenta por Pagar
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
-- Movimiento de apropiación inicial del presupuesto (cabecera)
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, valor_total_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(1, 'MOV-2025-001', 1, NULL, 500000000.00, '2025-01-01', 
 'Apropiación inicial del presupuesto de funcionamiento 2025', 'Acuerdo-001-2025', 2, '2025-01-15 10:00:00', 
 'APROBADO', true, '2025-01-01 08:00:00', '2025-01-15 10:00:00');

-- Detalle del movimiento (línea única para apropiación)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(1, 1, 1, 1, 1, 500000000.00, 'Apropiación inicial PG-2025-001', 
 true, '2025-01-01 08:00:00', '2025-01-15 10:00:00');
 
-- Nota: movimiento_tipo_id = 1 corresponde a 'APROPIACION_INICIAL' que tiene efecto 'INCREMENTA'
-- Nota: La apropiación inicial no tiene contrapartida automática
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

### 3.6 Movimientos de Afectación del CDP (Con Contrapartidas Automáticas)

```sql
-- MOVIMIENTO ÚNICO: Expedición del CDP con contrapartida automática
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, valor_total_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(2, 'MOV-2025-002', 8, 1, 180000000.00, '2025-02-15', 
 'Expedición CDP-2025-001 con afectación automática del PG-2025-001', 'CDP-2025-001', 3, '2025-02-15 09:00:00', 
 'APROBADO', true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');

-- LÍNEA 1: Incremento en el CDP (movimiento principal)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(2, 2, 1, 8, 2, 180000000.00, 'Valor inicial del CDP-2025-001', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');

-- LÍNEA 2: Reducción en el PG (contrapartida automática)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(3, 2, 2, 7, 1, 180000000.00, 'Contrapartida automática: Afectación del PG-2025-001', 
 true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
 
-- Nota: movimiento_tipo_id = 8 'VALOR_INICIAL_CDP' genera automáticamente tipo_id = 7 'AFECTACION_POR_CDP'
-- Nota: Un solo movimiento (MOV-2025-002) con dos líneas que afectan documentos diferentes
```

### 3.7 Detalle de Movimientos por Ítem - CDP

```sql
-- Detalle por ítem del movimiento CDP (MOV-002)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
-- Ítems afectados del CDP (línea 1 del movimiento)
(5, 2, 5, 1, 120000000.00, 'CDP Fase 1 desde Servicios Profesionales RP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00'),
(6, 2, 6, 3, 60000000.00, 'CDP Fase 2 desde Servicios Profesionales SGP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00'),

-- Ítems afectados del PG (línea 2 del movimiento - contrapartida)
(7, 2, 1, NULL, 120000000.00, 'Reducción: Servicios Profesionales RP por CDP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00'),
(8, 2, 3, NULL, 60000000.00, 'Reducción: Servicios Profesionales SGP por CDP', 
 '2025-02-15 09:00:00', '2025-02-15 09:00:00');

-- Nota: Ambas líneas del movimiento comparten el mismo detalle por ítem
-- Nota: item_destino_id indica el ítem origen cuando es una transferencia
```

### 3.8 Relación CDP with PG

```sql
-- Relaciones del CDP con el PG (movimiento único con contrapartida)
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(1, 1, 2, 'ORIGINA', 180000000.00, 36.00, 
 '{"tipo_operacion": "RESERVA", "conceptos": ["Servicios Profesionales RP", "Servicios Profesionales SGP"], "movimiento_id": 2, "lineas": [1, 2]}', 
 '2025-02-15', true, '2025-02-15 09:00:00', '2025-02-15 09:00:00');
 
-- Nota: metadatos_relacion incluye el ID del movimiento único y las líneas que lo componen
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

## PASO 4: Expedición de Registro Presupestal (RP)

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

### 4.6 Movimientos de Afectación del RP (Con Contrapartidas Automáticas)

```sql
-- MOVIMIENTO ÚNICO: Expedición del RP con contrapartida automática
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, valor_total_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(3, 'MOV-2025-003', 11, 3, 120000000.00, '2025-03-01', 
 'Expedición RP-2025-001 con afectación automática del CDP-2025-001', 'RP-2025-001', 4, '2025-03-01 10:30:00', 
 'APROBADO', true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');

-- LÍNEA 1: Incremento en el RP (movimiento principal)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(4, 3, 1, 11, 3, 120000000.00, 'Valor inicial del RP-2025-001', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');

-- LÍNEA 2: Reducción en el CDP (contrapartida automática)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(5, 3, 2, 10, 2, 120000000.00, 'Contrapartida automática: Reducción disponibilidad CDP-2025-001', 
 true, '2025-03-01 10:00:00', '2025-03-01 10:30:00');
 
-- Nota: movimiento_tipo_id = 11 'VALOR_INICIAL_RP' genera automáticamente tipo_id = 10 'REDUCCION_DISPONIBILIDAD'
-- Nota: Un solo movimiento (MOV-2025-003) con dos líneas que afectan documentos diferentes
```

### 4.7 Detalle de Movimientos por Ítem - RP

```sql
-- Detalle por ítem del movimiento RP (MOV-003)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
-- Ítems afectados del RP (línea 1 del movimiento)
(9, 3, 7, 5, 80000000.00, 'RP Recursos Propios desde CDP Fase 1', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00'),
(10, 3, 8, 6, 40000000.00, 'RP SGP desde CDP Fase 2', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00'),

-- Ítems afectados del CDP (línea 2 del movimiento - contrapartida)
(11, 3, 5, NULL, 80000000.00, 'Reducción: CDP Fase 1 por RP', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00'),
(12, 3, 6, NULL, 40000000.00, 'Reducción: CDP Fase 2 por RP', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00');

-- Nota: Ambas líneas del movimiento comparten el mismo detalle por ítem
-- Nota: item_destino_id indica el ítem origen cuando es una transferencia
```

### 4.8 Relación RP con CDP

```sql
-- Relación del RP con el CDP (movimiento único con contrapartida)
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(2, 2, 3, 'INCORPORA', 120000000.00, 66.67, 
 '{"tipo_operacion": "COMPROMISO", "contrato": "CNT-2025-001", "adjudicacion": "2025-02-28", "movimiento_id": 3, "lineas": [1, 2]}', 
 '2025-03-01', true, '2025-03-01 10:00:00', '2025-03-01 10:00:00');
 
-- Nota: metadatos_relacion incluye el ID del movimiento único y las líneas que lo componen
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

-- Actualizar ítems del CDP después del movimiento
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
- **CDP**: $180,000,000 total, $60,000,000 disponible (restaurado por liquidación)
- **RP**: $120,000,000 total, $60,000,000 ejecutado, $60,000,000 liquidado
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

### 5.6 Movimientos de Afectación de la OP (Con Contrapartidas Automáticas)

```sql
-- MOVIMIENTO ÚNICO: Expedición de la OP con contrapartida automática
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, valor_total_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(6, 'MOV-2025-004', 14, 4, 60000000.00, '2025-04-15', 
 'Expedición OP-2025-001 con afectación automática del RP-2025-001', 'OP-2025-001', 6, '2025-04-15 14:30:00', 
 'APROBADO', true, '2025-04-15 14:00:00', '2025-04-15 14:30:00');

-- LÍNEA 1: Incremento en la OP (movimiento principal)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(6, 6, 1, 14, 4, 60000000.00, 'Valor inicial de la OP-2025-001', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:30:00');

-- LÍNEA 2: Reducción en el RP (contrapartida automática)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(7, 6, 2, 12, 3, 60000000.00, 'Contrapartida automática: Reducción compromiso RP-2025-001', 
 true, '2025-04-15 14:00:00', '2025-04-15 14:30:00');
 
-- Nota: movimiento_tipo_id = 14 'VALOR_INICIAL_OP' genera automáticamente tipo_id = 12 'REDUCCION_COMPROMISO'
-- Nota: Un solo movimiento (MOV-2025-004) con dos líneas que afectan documentos diferentes
```

### 5.7 Detalle de Movimientos por Ítem - OP

```sql
-- Detalle por ítem del movimiento OP (MOV-004)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
-- Ítems afectados de la OP (línea 1 del movimiento)
(13, 6, 9, 7, 40000000.00, 'OP Recursos Propios desde RP', 
 '2025-04-15 14:00:00', '2025-04-15 14:00:00'),
(14, 6, 10, 8, 20000000.00, 'OP SGP desde RP', 
 '2025-04-15 14:00:00', '2025-04-15 14:00:00'),

-- Ítems afectados del RP (línea 2 del movimiento - contrapartida)
(15, 6, 7, NULL, 40000000.00, 'Reducción: RP Recursos Propios por OP', 
 '2025-04-15 14:00:00', '2025-04-15 14:00:00'),
(16, 6, 8, NULL, 20000000.00, 'Reducción: RP SGP por OP', 
 '2025-04-15 14:00:00', '2025-04-15 14:00:00');

-- Nota: Ambas líneas del movimiento comparten el mismo detalle por ítem
-- Nota: item_destino_id indica el ítem origen cuando es una transferencia
```

### 5.8 Relación OP con RP

```sql
-- Relación de la OP con el RP (movimiento único con contrapartida)
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(3, 3, 4, 'INCORPORA', 60000000.00, 50.00, 
 '{"tipo_operacion": "OBLIGACION", "factura": "FC-001", "acta": "AR-001", "movimiento_id": 6, "lineas": [1, 2]}', 
 '2025-04-15', true, '2025-04-15 14:00:00', '2025-04-15 14:00:00');
 
-- Nota: metadatos_relacion incluye el ID del movimiento único y las líneas que lo componen
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
- **CDP**: $180,000,000 total, $60,000,000 disponible (restaurado por liquidación)
- **RP**: $120,000,000 total, $60,000,000 ejecutado, $60,000,000 liquidado
- **OP**: $60,000,000 expedida y pendiente de pago

---

## PASO 6: Liberación del CDP Restante

### 6.1 Situación

**Fecha**: 2025-05-10
**Motivo**: El contrato se ejecutó por $120,000,000 pero el CDP era por $180,000,000
**Saldo a liberar**: $60,000,000 ($40M Recursos Propios + $20M SGP)

### 6.2 Movimientos de Liberación del CDP (Con Contrapartidas Automáticas)

```sql
-- MOVIMIENTO ÚNICO: Liberación del CDP con contrapartida automática
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, valor_total_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(8, 'MOV-2025-005', 9, NULL, 60000000.00, '2025-05-10', 
 'Liberación CDP-2025-001 con restauración automática al PG', 'Memo-001-2025', 6, '2025-05-10 16:00:00', 
 'APROBADO', true, '2025-05-10 15:30:00', '2025-05-10 16:00:00');

-- LÍNEA 1: Reducción en el CDP (movimiento principal)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(8, 8, 1, 9, 2, 60000000.00, 'Liberación de saldo no ejecutado CDP-2025-001', 
 true, '2025-05-10 15:30:00', '2025-05-10 16:00:00');

-- LÍNEA 2: Incremento en el PG (contrapartida automática)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(9, 8, 2, 6, 1, 60000000.00, 'Contrapartida automática: Restauración disponibilidad PG', 
 true, '2025-05-10 15:30:00', '2025-05-10 16:00:00');
 
-- Nota: movimiento_tipo_id = 9 'LIBERACION_CDP' genera automáticamente tipo_id = 6 'RESTAURACION_DISPONIBILIDAD'
-- Nota: Un solo movimiento (MOV-2025-005) con dos líneas que afectan documentos diferentes
```

### 6.3 Detalle de Movimientos de Liberación

```sql
-- Detalle por ítem del movimiento de liberación (MOV-005)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
-- Ítems afectados del CDP (línea 1 del movimiento)
(17, 8, 5, NULL, 40000000.00, 'Liberación: CDP Fase 1', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00'),
(18, 8, 6, NULL, 20000000.00, 'Liberación: CDP Fase 2', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00'),

-- Ítems afectados del PG (línea 2 del movimiento - contrapartida)
(19, 8, 1, 5, 40000000.00, 'Restauración: Servicios Profesionales RP desde CDP', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00'),
(20, 8, 3, 6, 20000000.00, 'Restauración: Servicios Profesionales SGP desde CDP', 
 '2025-05-10 15:30:00', '2025-05-10 15:30:00');

-- Nota: Ambas líneas del movimiento comparten el mismo detalle por ítem
-- Nota: item_destino_id indica el ítem origen cuando es una restauración
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

## DIAGRAMA DE FLUJO CON MOVIMIENTOS ÚNICOS Y CONTRAPARTIDAS AUTOMÁTICAS

```
APROPIACION_INICIAL (MOV-001)
    ↓ Línea 1: INCREMENTA PG +$500M
PG-2025-001 ($500M disponible)
    ↓ 
MOV-002: EXPEDICION_CDP
    ├── Línea 1: VALOR_INICIAL_CDP → CDP +$180M
    └── Línea 2: AFECTACION_POR_CDP → PG -$180M (contrapartida automática)
    ↓
CDP-2025-001 ($180M disponible)
    ↓
MOV-003: EXPEDICION_RP
    ├── Línea 1: VALOR_INICIAL_RP → RP +$120M
    └── Línea 2: REDUCCION_DISPONIBILIDAD → CDP -$120M (contrapartida automática)
    ↓
RP-2025-001 ($120M disponible)
    ├── MOV-004: EXPEDICION_OP
    │   ├── Línea 1: VALOR_INICIAL_OP → OP +$60M
    │   └── Línea 2: REDUCCION_COMPROMISO → RP -$60M (contrapartida automática)
    │   ↓
    │   OP-2025-001 ($60M disponible)
    │
    └── MOV-006: LIQUIDACION_RP
        ├── Línea 1: LIQUIDACION_RP → RP -$60M
        └── Línea 2: RESTAURACION_POR_LIQUIDACION_RP → CDP +$60M (contrapartida automática)
        ↑
        CDP-2025-001 ($60M restaurado)
        ↓
MOV-005: LIBERACION_CDP
    ├── Línea 1: LIBERACION_CDP → CDP -$60M
    └── Línea 2: RESTAURACION_DISPONIBILIDAD → PG +$60M (contrapartida automática)
    ↑
PG-2025-001 ($440M disponible)
```

**Ventajas del Nuevo Modelo:**
- **6 movimientos únicos** en lugar de 10 movimientos separados
- **Contrapartidas automáticas** garantizan equilibrio
- **Numeración simplificada** sin sufijos A/B
- **Atomicidad** de las operaciones presupuestales

**Comparación con Modelo Anterior:**
| Operación | Modelo Anterior | Modelo Nuevo |
|-----------|----------------|--------------|
| Expedición CDP | MOV-002A + MOV-002B | MOV-002 (2 líneas) |
| Expedición RP | MOV-003A + MOV-003B | MOV-003 (2 líneas) |
| Expedición OP | MOV-004A + MOV-004B | MOV-004 (2 líneas) |
| Liberación CDP | MOV-005A + MOV-005B | MOV-005 (2 líneas) |
| Liquidación RP | MOV-006A + MOV-006B | MOV-006 (2 líneas) |
