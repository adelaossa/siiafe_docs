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

-- Tipos de movimiento para Liberación CDP (como documento independiente)
(16, 'VALOR_INICIAL_LIBERACION_CDP', 'Valor Inicial Liberación CDP', 'INCREMENTA', 'Asignación inicial de la Liberación CDP', true, NOW(), NOW()),

-- Tipos de movimiento para Liquidación RP (como documento independiente)
(17, 'VALOR_INICIAL_LIQUIDACION_RP', 'Valor Inicial Liquidación RP', 'INCREMENTA', 'Asignación inicial de la Liquidación RP', true, NOW(), NOW()),

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
(15, 14, 4, true, NOW(), NOW()), -- VALOR_INICIAL_OP

-- Liberación CDP (tipo_documento_id = 8)
(16, 16, 8, true, NOW(), NOW()), -- VALOR_INICIAL_LIBERACION_CDP

-- Liquidación RP (tipo_documento_id = 9)
(17, 17, 9, true, NOW(), NOW()); -- VALOR_INICIAL_LIQUIDACION_RP
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
-- 8 = Liberación CDP, 9 = Liquidación RP
```

### Configuración de Estados Requeridos para Documentos Precedentes

```sql
-- Tabla que define qué estados específicos deben tener los documentos precedentes
-- para poder crear un nuevo documento. Ejemplo: para crear un RP, el CDP debe estar en "EXPEDIDO" o "COMPROMETIDO"
-- Columnas: id, documento_tipo_precedente_id, estado_requerido, descripcion, es_activo, creado_en, actualizado_en
CREATE TABLE documento_precedente_estado_requerido (
    id SERIAL PRIMARY KEY,
    documento_tipo_precedente_id INTEGER NOT NULL REFERENCES documento_tipo_precedente(id),
    estado_requerido VARCHAR(50) NOT NULL,
    descripcion TEXT,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(documento_tipo_precedente_id, estado_requerido)
);

-- Configuración de estados requeridos para las precedencias estándar
INSERT INTO documento_precedente_estado_requerido VALUES 
-- Para crear CDP desde PG, el PG debe estar VIGENTE
(1, 1, 'VIGENTE', 'El Presupuesto de Gasto debe estar vigente para expedir CDPs', true, NOW(), NOW()),

-- Para crear RP desde CDP, el CDP puede estar EXPEDIDO o COMPROMETIDO
(2, 2, 'EXPEDIDO', 'El CDP debe estar expedido para crear un RP', true, NOW(), NOW()),
(3, 2, 'COMPROMETIDO', 'Un CDP ya comprometido puede generar RPs adicionales si tiene saldo', true, NOW(), NOW()),

-- Para crear OP desde RP, el RP debe estar EXPEDIDO u OBLIGADO
(4, 3, 'EXPEDIDO', 'El RP debe estar expedido para crear una OP', true, NOW(), NOW()),
(5, 3, 'OBLIGADO', 'Un RP obligado puede generar OPs adicionales si tiene saldo pendiente', true, NOW(), NOW()),

-- Para crear Egreso desde OP, la OP debe estar EXPEDIDA
(6, 4, 'EXPEDIDA', 'La OP debe estar expedida para generar el egreso correspondiente', true, NOW(), NOW()),

-- Para crear Egreso desde Cuenta por Pagar, debe estar APROBADA
(7, 5, 'APROBADA', 'La Cuenta por Pagar debe estar aprobada para generar el egreso', true, NOW(), NOW());
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

-- VALOR_INICIAL_LIBERACION_CDP (16) genera automáticamente LIBERACION_CDP (9)
(4, 16, 9, 1, true, 'Al crear una Liberación CDP, se debe reducir automáticamente el CDP', true, NOW(), NOW()),

-- VALOR_INICIAL_LIQUIDACION_RP (17) genera automáticamente LIQUIDACION_RP (13)
(5, 17, 13, 1, true, 'Al crear una Liquidación RP, se debe reducir automáticamente el RP', true, NOW(), NOW()),

-- LIBERACION_CDP (9) genera automáticamente RESTAURACION_DISPONIBILIDAD (6)
(6, 9, 6, 1, true, 'Al liberar un CDP, se debe restaurar automáticamente el PG', true, NOW(), NOW()),

-- LIQUIDACION_RP (13) genera automáticamente RESTAURACION_POR_LIQUIDACION_RP (15)
(7, 13, 15, 1, true, 'Al liquidar un RP, se debe restaurar automáticamente el CDP', true, NOW(), NOW());
```

### Validación de Documentos Precedentes

```sql
-- Función para validar que un documento tenga el precedente correcto y el estado requerido
CREATE OR REPLACE FUNCTION validar_documento_precedente()
RETURNS TRIGGER AS $$
DECLARE
    v_estado_origen VARCHAR(50);
    v_tipo_documento_origen INTEGER;
    v_tipo_documento_destino INTEGER;
    v_precedente_id INTEGER;
BEGIN
    -- Solo validar si el documento tiene un origen definido
    IF NEW.documento_origen_id IS NOT NULL THEN
        -- Obtener información del documento origen y destino
        SELECT dp_origen.estado_actual, dp_origen.tipo_documento_id,
               dp_destino.tipo_documento_id
        INTO v_estado_origen, v_tipo_documento_origen, v_tipo_documento_destino
        FROM documento_presupuestal dp_origen
        JOIN documento_presupuestal dp_destino ON dp_destino.id = NEW.documento_afectado_id
        WHERE dp_origen.id = NEW.documento_origen_id;

        -- Verificar que existe la relación de precedencia
        SELECT dtp.id INTO v_precedente_id
        FROM documento_tipo_precedente dtp
        WHERE dtp.tipo_documento_id = v_tipo_documento_destino
          AND dtp.tipo_documento_precedente_id = v_tipo_documento_origen
          AND dtp.es_activo = true;

        IF v_precedente_id IS NULL THEN
            RAISE EXCEPTION 'El documento origen (tipo %) no es un precedente válido para el tipo de documento destino (%)', 
                           v_tipo_documento_origen, v_tipo_documento_destino;
        END IF;

        -- Verificar que el estado del documento origen es válido
        IF NOT EXISTS (
            SELECT 1 FROM documento_precedente_estado_requerido dper
            WHERE dper.documento_tipo_precedente_id = v_precedente_id
              AND dper.estado_requerido = v_estado_origen
              AND dper.es_activo = true
        ) THEN
            RAISE EXCEPTION 'El documento origen está en estado "%" pero se requiere uno de los estados válidos para esta operación. Consulte los estados requeridos para el tipo de precedencia.', 
                           v_estado_origen;
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

### Consulta de Estados Requeridos para Precedentes

```sql
-- Función para consultar los estados requeridos de documentos precedentes
CREATE OR REPLACE FUNCTION obtener_estados_requeridos_precedente(
    p_tipo_documento_id INTEGER,
    p_tipo_documento_precedente_id INTEGER DEFAULT NULL
)
RETURNS TABLE (
    precedente_id INTEGER,
    precedente_nombre VARCHAR,
    estado_requerido VARCHAR,
    descripcion_estado TEXT,
    descripcion_precedente TEXT
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        dtp.tipo_documento_precedente_id,
        td.nombre,
        dper.estado_requerido,
        dper.descripcion,
        dtp.descripcion
    FROM documento_tipo_precedente dtp
    JOIN tipo_documento td ON dtp.tipo_documento_precedente_id = td.id
    JOIN documento_precedente_estado_requerido dper ON dper.documento_tipo_precedente_id = dtp.id
    WHERE dtp.tipo_documento_id = p_tipo_documento_id
      AND dtp.es_activo = true
      AND dper.es_activo = true
      AND (p_tipo_documento_precedente_id IS NULL OR dtp.tipo_documento_precedente_id = p_tipo_documento_precedente_id)
    ORDER BY dtp.orden_precedencia, dper.estado_requerido;
END;
$$ LANGUAGE plpgsql;

-- Ejemplos de uso:

-- 1. Consultar todos los estados requeridos para crear un RP (tipo_documento_id = 3)
-- SELECT * FROM obtener_estados_requeridos_precedente(3);
-- Resultado:
-- precedente_id | precedente_nombre | estado_requerido | descripcion_estado | descripcion_precedente
-- 2             | CDP               | EXPEDIDO         | El CDP debe estar expedido para crear un RP | RP debe originarse desde un CDP
-- 2             | CDP               | COMPROMETIDO     | Un CDP ya comprometido puede generar RPs adicionales | RP debe originarse desde un CDP

-- 2. Consultar estados específicos para crear un RP desde un CDP
-- SELECT * FROM obtener_estados_requeridos_precedente(3, 2);
-- Resultado:
-- precedente_id | precedente_nombre | estado_requerido | descripcion_estado | descripcion_precedente
-- 2             | CDP               | EXPEDIDO         | El CDP debe estar expedido para crear un RP | RP debe originarse desde un CDP
-- 2             | CDP               | COMPROMETIDO     | Un CDP ya comprometido puede generar RPs adicionales | RP debe originarse desde un CDP

-- 3. Función de utilidad para validar si un documento puede ser usado como precedente
CREATE OR REPLACE FUNCTION puede_usar_documento_como_precedente(
    p_documento_origen_id INTEGER,
    p_tipo_documento_destino_id INTEGER
)
RETURNS BOOLEAN AS $$
DECLARE
    v_estado_origen VARCHAR(50);
    v_tipo_documento_origen INTEGER;
    v_precedente_id INTEGER;
BEGIN
    -- Obtener estado y tipo del documento origen
    SELECT estado_actual, tipo_documento_id 
    INTO v_estado_origen, v_tipo_documento_origen
    FROM documento_presupuestal 
    WHERE id = p_documento_origen_id;

    IF v_estado_origen IS NULL THEN
        RETURN FALSE;
    END IF;

    -- Verificar que existe la relación de precedencia
    SELECT dtp.id INTO v_precedente_id
    FROM documento_tipo_precedente dtp
    WHERE dtp.tipo_documento_id = p_tipo_documento_destino_id
      AND dtp.tipo_documento_precedente_id = v_tipo_documento_origen
      AND dtp.es_activo = true;

    IF v_precedente_id IS NULL THEN
        RETURN FALSE;
    END IF;

    -- Verificar que el estado es válido
    RETURN EXISTS (
        SELECT 1 FROM documento_precedente_estado_requerido dper
        WHERE dper.documento_tipo_precedente_id = v_precedente_id
          AND dper.estado_requerido = v_estado_origen
          AND dper.es_activo = true
    );
END;
$$ LANGUAGE plpgsql;

-- Ejemplo de uso:
-- SELECT puede_usar_documento_como_precedente(123, 3); -- ¿Puede el documento 123 ser precedente para crear un RP?
-- TRUE/FALSE
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
(11, 3, 5, NULL, 80000000.00, 'Reducción: RP Recursos Propios por liquidación', 
 '2025-03-01 10:00:00', '2025-03-01 10:00:00'),
(12, 3, 6, NULL, 40000000.00, 'Reducción: RP SGP por liquidación', 
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

## PASO 5B: Liquidación del RP Restante

### 5B.1 Creación del Documento de Liquidación RP

```sql
-- Crear documento de Liquidación RP
-- Columnas: id, tipo_documento_id, numero_documento, fecha_documento, fecha_vencimiento, estado_actual, valor_total_inicial, valor_total_actual, observaciones, usuario_creacion_id, usuario_aprobacion_id, fecha_aprobacion, metadatos, es_activo, creado_en, actualizado_en
INSERT INTO documento_presupuestal VALUES 
(6, 9, 'LIQ-RP-2025-001', '2025-04-30', NULL, 'ACTIVO', 60000000.00, 60000000.00, 
 'Liquidación de saldo no ejecutado del RP-2025-001', 6, 6, '2025-04-30 17:00:00', 
 '{"motivo": "Saldo no ejecutado", "rp_origen": "RP-2025-001", "valor_ejecutado": 60000000.00}', 
 true, '2025-04-30 16:30:00', '2025-04-30 17:00:00');

-- Crear ítems para la Liquidación RP
-- Columnas: id, documento_id, item_numero, descripcion, valor_inicial, valor_actual, observaciones, es_activo, creado_en, actualizado_en
INSERT INTO item_documento_presupuestal VALUES 
(13, 6, 1, 'Liquidación Servicios Profesionales - Recursos Propios', 40000000.00, 40000000.00, 
 'Liquidación saldo no ejecutado Fase 1', true, '2025-04-30 16:30:00', '2025-04-30 16:30:00'),
(14, 6, 2, 'Liquidación Servicios Profesionales - SGP', 20000000.00, 20000000.00, 
 'Liquidación saldo no ejecutado Fase 2', true, '2025-04-30 16:30:00', '2025-04-30 16:30:00');
```

### 5B.2 Movimientos de Liquidación del RP (Con Contrapartidas Automáticas)

```sql
-- MOVIMIENTO ÚNICO: Expedición de la Liquidación RP con contrapartida automática
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, valor_total_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(7, 'MOV-2025-006', 17, 6, 60000000.00, '2025-04-30', 
 'Expedición Liquidación RP con afectación automática del RP-2025-001', 'LIQ-RP-2025-001', 6, '2025-04-30 17:00:00', 
 'APROBADO', true, '2025-04-30 16:30:00', '2025-04-30 17:00:00');

-- LÍNEA 1: Incremento en la Liquidación RP (movimiento principal)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(11, 7, 1, 17, 6, 60000000.00, 'Valor inicial de la Liquidación RP', 
 true, '2025-04-30 16:30:00', '2025-04-30 17:00:00');

-- LÍNEA 2: Reducción en el RP (contrapartida automática)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(12, 7, 2, 13, 3, 60000000.00, 'Contrapartida automática: Liquidación RP-2025-001', 
 true, '2025-04-30 16:30:00', '2025-04-30 17:00:00');
 
-- Nota: movimiento_tipo_id = 17 'VALOR_INICIAL_LIQUIDACION_RP' genera automáticamente tipo_id = 13 'LIQUIDACION_RP'
-- Nota: Un solo movimiento (MOV-2025-006) con dos líneas que afectan documentos diferentes
```

### 5B.3 Segundo Movimiento: Restauración al CDP

```sql
-- MOVIMIENTO ÚNICO: Restauración automática al CDP
-- Columnas: id, numero_movimiento, movimiento_tipo_id, documento_origen_id, valor_total_movimiento, fecha_movimiento, observaciones, documento_soporte, usuario_id, fecha_aprobacion, estado, es_activo, creado_en, actualizado_en
INSERT INTO movimiento_presupuestal VALUES 
(10, 'MOV-2025-006B', 15, 6, 60000000.00, '2025-04-30', 
 'Restauración automática al CDP por Liquidación RP', 'LIQ-RP-2025-001', 6, '2025-04-30 17:00:00', 
 'APROBADO', true, '2025-04-30 16:30:00', '2025-04-30 17:00:00');

-- LÍNEA 1: Incremento en el CDP (movimiento principal)
-- Columnas: id, movimiento_id, linea_numero, movimiento_tipo_id, documento_afectado_id, valor_linea, observaciones_linea, es_activo, creado_en, actualizado_en
INSERT INTO detalle_movimiento_presupuestal VALUES 
(13, 10, 1, 15, 2, 60000000.00, 'Restauración disponibilidad CDP por liquidación RP', 
 true, '2025-04-30 16:30:00', '2025-04-30 17:00:00');
 
-- Nota: La restauración al CDP se ejecuta automáticamente cuando se procesa la liquidación RP
```

### 5B.4 Detalle de Movimientos de Liquidación

```sql
-- Detalle por ítem del movimiento de liquidación (MOV-006)
-- Columnas: id, movimiento_id, item_id, item_destino_id, valor_detalle, observaciones, creado_en, actualizado_en
INSERT INTO detalle_movimiento_item VALUES 
-- Ítems afectados de la Liquidación RP (línea 1 del movimiento)
(23, 7, 13, 7, 40000000.00, 'Liquidación RP: Recursos Propios', 
 '2025-04-30 16:30:00', '2025-04-30 16:30:00'),
(24, 7, 14, 8, 20000000.00, 'Liquidación RP: SGP', 
 '2025-04-30 16:30:00', '2025-04-30 16:30:00'),

-- Ítems afectados del RP (línea 2 del movimiento - contrapartida)
(25, 7, 7, NULL, 40000000.00, 'Reducción: RP Recursos Propios por liquidación', 
 '2025-04-30 16:30:00', '2025-04-30 16:30:00'),
(26, 7, 8, NULL, 20000000.00, 'Reducción: RP SGP por liquidación', 
 '2025-04-30 16:30:00', '2025-04-30 16:30:00'),

-- Ítems afectados del CDP (restauración)
(27, 10, 5, 13, 40000000.00, 'Restauración: CDP Fase 1 desde Liquidación RP', 
 '2025-04-30 16:30:00', '2025-04-30 16:30:00'),
(28, 10, 6, 14, 20000000.00, 'Restauración: CDP Fase 2 desde Liquidación RP', 
 '2025-04-30 16:30:00', '2025-04-30 16:30:00');

-- Nota: item_destino_id indica el ítem origen cuando es una restauración
```

### 5B.5 Relación Liquidación RP con RP

```sql
-- Relación de la Liquidación RP con el RP original
-- Columnas: id, documento_origen_id, documento_destino_id, tipo_relacion, valor_relacion, porcentaje_relacion, metadatos_relacion, fecha_relacion, es_activo, creado_en, actualizado_en
INSERT INTO relacion_documento_presupuestal VALUES 
(5, 3, 6, 'LIQUIDA', 60000000.00, 50.00, 
 '{"tipo_operacion": "LIQUIDACION", "motivo": "Saldo no ejecutado", "movimiento_id": 7, "lineas": [1, 2]}', 
 '2025-04-30', true, '2025-04-30 16:30:00', '2025-04-30 16:30:00');
 
-- Nota: metadatos_relacion incluye el ID del movimiento único y las líneas que lo componen
```

---

## VENTAJAS DEL NUEVO MODELO: LIBERACIONES Y LIQUIDACIONES COMO DOCUMENTOS

### 1. **Trazabilidad Completa**
- **Cada liberación y liquidación** tiene su propio documento con número único
- **Auditoría mejorada**: Se puede consultar el historial completo de cada liberación/liquidación
- **Reportes específicos**: Informes por tipo de documento (liberaciones, liquidaciones)

### 2. **Separación de Responsabilidades**
- **Creación del documento**: Registro de la decisión administrativa
- **Afectación presupuestal**: Impacto automático en los documentos relacionados
- **Restauración**: Devolución automática de recursos al origen

### 3. **Doble Contrapartida**
```sql
-- Ejemplo: Liberación CDP
MOV-005: Expedición Liberación CDP
├── Línea 1: +$60M → Liberación CDP (documento nuevo)
└── Línea 2: -$60M → CDP (contrapartida automática)

MOV-005B: Restauración automática al PG
├── Línea 1: +$60M → PG (restauración)
└── Línea 2: (procesamiento automático)
```

### 4. **Flexibilidad Operativa**
- **Liberaciones parciales**: Múltiples liberaciones del mismo CDP
- **Liquidaciones escalonadas**: Liquidación por fases del mismo RP
- **Cancelaciones**: Reversa de liberaciones/liquidaciones con trazabilidad

### 5. **Integridad Referencial**
- **Relaciones documentales**: Cada liberación/liquidación referencia su documento origen
- **Validaciones automáticas**: Control de valores y estados
- **Conciliación**: Balance automático entre documentos

### 6. **Beneficios Contables**
- **Cuentas por cobrar**: Las liberaciones generan cuentas por cobrar al Estado
- **Cuentas por pagar**: Las liquidaciones generan cuentas por pagar a contratistas
- **Estados financieros**: Mejor representación de la situación financiera

### 7. **Comparación de Modelos**

| Aspecto | Modelo Anterior | Modelo Nuevo |
|---------|----------------|--------------|
| **Liberación CDP** | Movimiento simple | Documento + Movimiento |
| **Liquidación RP** | Movimiento simple | Documento + Movimiento |
| **Trazabilidad** | Limitada | Completa |
| **Auditabilidad** | Básica | Avanzada |
| **Flexibilidad** | Rígida | Alta |
| **Reportes** | Limitados | Completos |
| **Integridad** | Manual | Automática |
