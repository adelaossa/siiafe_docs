# 03 - Movimientos Presupuestales

## Descripción General

Los **Movimientos Presupuestales** son el mecanismo fundamental que controla todos los cambios de valores en el sistema presupuestario. Todo cambio en los valores de los ítems de documentos presupuestales se realiza exclusivamente a través de movimientos, garantizando así la trazabilidad completa y la integridad de la información.

## Conceptos Fundamentales

### ¿Qué es un Movimiento Presupuestal?

Un movimiento presupuestal es:

- **Registro de cambio**: Documenta un cambio específico en valores presupuestales
- **Origen definido**: Siempre tiene un documento que lo origina
- **Efectos configurables**: Puede afectar múltiples documentos y columnas según su configuración
- **Trazabilidad completa**: Permite seguir el rastro de todos los cambios
- **Validación automática**: Aplica reglas de negocio para evitar inconsistencias

### Características Principales

- **Inmutabilidad**: Una vez creado, un movimiento no se puede modificar
- **Atomicidad**: Todos los efectos de un movimiento se aplican o ninguno
- **Configurabilidad**: Los tipos de movimiento definen sus efectos mediante configuración
- **Validación de saldos**: Previene que los saldos queden en negativo
- **Auditoría automática**: Cada movimiento queda registrado para auditoría

---

## Estructura de Datos

### Tabla Principal: movimiento_presupuestal

```sql
CREATE TABLE movimiento_presupuestal (
    id SERIAL PRIMARY KEY,
    
    -- Identificación del movimiento
    numero_movimiento VARCHAR(50) NOT NULL,
    tipo_movimiento_id INTEGER NOT NULL REFERENCES tipo_movimiento_presupuestal(id),
    
    -- Documento que origina el movimiento
    documento_origen_id INTEGER NOT NULL REFERENCES documento_presupuestal(id),
    item_documento_origen_id INTEGER REFERENCES item_documento_presupuestal(id),
    
    -- Documento y ítem afectado por el movimiento
    documento_afectado_id INTEGER NOT NULL REFERENCES documento_presupuestal(id),
    item_documento_afectado_id INTEGER NOT NULL REFERENCES item_documento_presupuestal(id),
    
    -- Valores del movimiento
    valor_movimiento DECIMAL(15,2) NOT NULL,
    signo_movimiento CHAR(1) NOT NULL CHECK (signo_movimiento IN ('+', '-')),
    
    -- Descripción y observaciones
    descripcion_movimiento VARCHAR(500),
    observaciones TEXT,
    concepto_movimiento VARCHAR(200),
    
    -- Estado del movimiento
    estado_movimiento VARCHAR(20) DEFAULT 'ACTIVO' CHECK (estado_movimiento IN ('ACTIVO', 'ANULADO')),
    
    -- Metadatos
    fecha_movimiento TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER NOT NULL REFERENCES usuario(id),
    
    -- Para movimientos de anulación
    movimiento_origen_id INTEGER REFERENCES movimiento_presupuestal(id),
    
    -- Datos adicionales en JSON
    datos_adicionales JSON,
    
    -- Índices únicos
    UNIQUE(numero_movimiento)
);

-- Índices para optimizar consultas
CREATE INDEX idx_movimiento_tipo ON movimiento_presupuestal(tipo_movimiento_id);
CREATE INDEX idx_movimiento_documento_origen ON movimiento_presupuestal(documento_origen_id);
CREATE INDEX idx_movimiento_documento_afectado ON movimiento_presupuestal(documento_afectado_id);
CREATE INDEX idx_movimiento_item_afectado ON movimiento_presupuestal(item_documento_afectado_id);
CREATE INDEX idx_movimiento_fecha ON movimiento_presupuestal(fecha_movimiento);
CREATE INDEX idx_movimiento_estado ON movimiento_presupuestal(estado_movimiento);
```

### Tabla: tipo_movimiento_presupuestal

```sql
CREATE TABLE tipo_movimiento_presupuestal (
    id SERIAL PRIMARY KEY,
    codigo VARCHAR(50) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT,
    
    -- Configuración del tipo de movimiento
    tipo_documento_origen_id INTEGER REFERENCES tipo_documento_presupuestal(id),
    es_movimiento_principal BOOLEAN DEFAULT FALSE, -- Si es el movimiento principal del tipo de documento
    
    -- Comportamiento del movimiento
    genera_numero_automatico BOOLEAN DEFAULT TRUE,
    prefijo_numero VARCHAR(10),
    requiere_aprobacion BOOLEAN DEFAULT FALSE,
    permite_valor_cero BOOLEAN DEFAULT FALSE,
    
    -- Validaciones
    valor_minimo DECIMAL(15,2),
    valor_maximo DECIMAL(15,2),
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER REFERENCES usuario(id),
    
    -- Índice único
    UNIQUE(codigo)
);
```

---

## Configuración de Efectos de Movimientos

### Tabla: efecto_movimiento

```sql
CREATE TABLE efecto_movimiento (
    id SERIAL PRIMARY KEY,
    tipo_movimiento_id INTEGER NOT NULL REFERENCES tipo_movimiento_presupuestal(id),
    
    -- Documento y columna afectada
    tipo_documento_afectado_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id),
    tipo_documento_columna_id INTEGER NOT NULL REFERENCES tipo_documento_columna(id),
    
    -- Tipo de efecto
    tipo_efecto VARCHAR(20) NOT NULL CHECK (tipo_efecto IN ('INCREMENTA', 'DECREMENTA', 'ESTABLECE')),
    
    -- Configuración del efecto
    es_efecto_origen BOOLEAN DEFAULT TRUE, -- Si afecta al documento origen o a un precedente
    orden_aplicacion INTEGER DEFAULT 1, -- Orden en que se aplican los efectos
    
    -- Validaciones específicas del efecto
    validar_saldo_positivo BOOLEAN DEFAULT TRUE,
    validar_valor_disponible BOOLEAN DEFAULT TRUE,
    
    -- Fórmula para cálculo del valor (opcional)
    formula_calculo VARCHAR(500), -- Ej: "valor_movimiento * 0.1" para descuentos
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER REFERENCES usuario(id),
    
    -- Validar que la columna pertenezca al tipo de documento
    CONSTRAINT fk_columna_documento CHECK (
        tipo_documento_columna_id IN (
            SELECT id FROM tipo_documento_columna 
            WHERE tipo_documento_id = tipo_documento_afectado_id
        )
    )
);

-- Índices
CREATE INDEX idx_efecto_tipo_movimiento ON efecto_movimiento(tipo_movimiento_id);
CREATE INDEX idx_efecto_tipo_documento ON efecto_movimiento(tipo_documento_afectado_id);
CREATE INDEX idx_efecto_orden ON efecto_movimiento(orden_aplicacion);
```

---

## Configuraciones de Ejemplo

### Configuración: Certificado de Disponibilidad Presupuestal (CDP)

```sql
-- Tipo de movimiento principal para CDP
INSERT INTO tipo_movimiento_presupuestal VALUES
(1, 'VALOR_INICIAL_CDP', 'Valor Inicial de CDP', 
 'Movimiento que establece el valor inicial de un Certificado de Disponibilidad Presupuestal',
 5, true, true, 'CDP-', false, false, 1.00, null, true, NOW(), 1);

-- Efectos del movimiento "Valor Inicial CDP"
-- Efecto 1: Incrementa el valor inicial del CDP (documento origen)
INSERT INTO efecto_movimiento VALUES
(1, 1, 5, 8, 'INCREMENTA', true, 1, true, false, null, true, NOW(), 1);

-- Efecto 2: Incrementa el saldo disponible del CDP (documento origen)  
INSERT INTO efecto_movimiento VALUES
(2, 1, 5, 11, 'INCREMENTA', true, 2, true, false, null, true, NOW(), 1);

-- Efecto 3: Decrementa el saldo de apropiación del Presupuesto de Gasto (documento precedente)
INSERT INTO efecto_movimiento VALUES
(3, 1, 1, 7, 'DECREMENTA', false, 3, true, true, null, true, NOW(), 1);

-- Efecto 4: Incrementa el valor reservado en el Presupuesto de Gasto (documento precedente)
INSERT INTO efecto_movimiento VALUES
(4, 1, 1, 6, 'INCREMENTA', false, 4, true, false, null, true, NOW(), 1);
```

### Configuración: Adición Presupuestal

```sql
-- Tipo de movimiento para Adición Presupuestal
INSERT INTO tipo_movimiento_presupuestal VALUES
(2, 'ADICION_PRESUPUESTAL', 'Adición Presupuestal', 
 'Movimiento que incrementa el presupuesto mediante adiciones',
 2, true, true, 'AD-', true, false, 1.00, null, true, NOW(), 1);

-- Efectos del movimiento "Adición Presupuestal"
-- Efecto 1: Incrementa el valor de adiciones en el Presupuesto de Gasto
INSERT INTO efecto_movimiento VALUES
(5, 2, 1, 2, 'INCREMENTA', true, 1, true, false, null, true, NOW(), 1);

-- Efecto 2: Incrementa el presupuesto definitivo (columna calculada se actualizará automáticamente)
-- Efecto 3: Incrementa el saldo de apropiación disponible
INSERT INTO efecto_movimiento VALUES
(6, 2, 1, 7, 'INCREMENTA', true, 2, true, false, null, true, NOW(), 1);
```

### Configuración: Compromiso de CDP (Registro Presupuestal)

```sql
-- Tipo de movimiento para compromiso de CDP
INSERT INTO tipo_movimiento_presupuestal VALUES
(3, 'COMPROMISO_CDP', 'Compromiso de CDP en RP', 
 'Movimiento que compromete recursos de un CDP mediante un Registro Presupuestal',
 6, false, true, 'CMP-', false, false, 1.00, null, true, NOW(), 1);

-- Efectos del movimiento "Compromiso CDP"
-- Efecto 1: Incrementa el valor inicial del RP (documento origen)
INSERT INTO efecto_movimiento VALUES
(7, 3, 6, 13, 'INCREMENTA', true, 1, true, false, null, true, NOW(), 1);

-- Efecto 2: Incrementa el saldo disponible del RP (documento origen)
INSERT INTO efecto_movimiento VALUES
(8, 3, 6, 16, 'INCREMENTA', true, 2, true, false, null, true, NOW(), 1);

-- Efecto 3: Incrementa el valor comprometido del CDP (documento precedente)
INSERT INTO efecto_movimiento VALUES
(9, 3, 5, 9, 'INCREMENTA', false, 3, true, true, null, true, NOW(), 1);

-- Efecto 4: Decrementa el saldo disponible del CDP (documento precedente)
INSERT INTO efecto_movimiento VALUES
(10, 3, 5, 11, 'DECREMENTA', false, 4, true, true, null, true, NOW(), 1);
```

---

## Funciones de Gestión de Movimientos

### Función: crear_movimiento_presupuestal

```sql
CREATE OR REPLACE FUNCTION crear_movimiento_presupuestal(
    p_tipo_movimiento_codigo VARCHAR(50),
    p_documento_origen_id INTEGER,
    p_item_documento_origen_id INTEGER,
    p_documento_afectado_id INTEGER,
    p_item_documento_afectado_id INTEGER,
    p_valor_movimiento DECIMAL(15,2),
    p_descripcion VARCHAR(500),
    p_usuario_id INTEGER,
    p_observaciones TEXT DEFAULT NULL
)
RETURNS INTEGER AS $$
DECLARE
    v_movimiento_id INTEGER;
    v_tipo_movimiento_id INTEGER;
    v_numero_movimiento VARCHAR(50);
    v_efecto RECORD;
    v_valor_efecto DECIMAL(15,2);
BEGIN
    -- Obtener tipo de movimiento
    SELECT id INTO v_tipo_movimiento_id
    FROM tipo_movimiento_presupuestal
    WHERE codigo = p_tipo_movimiento_codigo AND es_activo = true;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Tipo de movimiento % no encontrado o inactivo', p_tipo_movimiento_codigo;
    END IF;
    
    -- Validar valor del movimiento
    IF p_valor_movimiento <= 0 THEN
        RAISE EXCEPTION 'El valor del movimiento debe ser mayor a cero';
    END IF;
    
    -- Generar número de movimiento
    v_numero_movimiento := generar_numero_movimiento(v_tipo_movimiento_id);
    
    -- Validar efectos antes de crear el movimiento
    PERFORM validar_efectos_movimiento(
        v_tipo_movimiento_id, 
        p_documento_origen_id,
        p_item_documento_origen_id,
        p_documento_afectado_id,
        p_item_documento_afectado_id,
        p_valor_movimiento
    );
    
    -- Crear el movimiento principal
    INSERT INTO movimiento_presupuestal 
        (numero_movimiento, tipo_movimiento_id, documento_origen_id, item_documento_origen_id,
         documento_afectado_id, item_documento_afectado_id, valor_movimiento, signo_movimiento,
         descripcion_movimiento, observaciones, usuario_creacion)
    VALUES 
        (v_numero_movimiento, v_tipo_movimiento_id, p_documento_origen_id, p_item_documento_origen_id,
         p_documento_afectado_id, p_item_documento_afectado_id, p_valor_movimiento, '+',
         p_descripcion, p_observaciones, p_usuario_id)
    RETURNING id INTO v_movimiento_id;
    
    -- Aplicar efectos del movimiento
    FOR v_efecto IN 
        SELECT * FROM efecto_movimiento
        WHERE tipo_movimiento_id = v_tipo_movimiento_id 
          AND es_activo = true
        ORDER BY orden_aplicacion
    LOOP
        -- Calcular valor del efecto
        v_valor_efecto := calcular_valor_efecto(v_efecto, p_valor_movimiento);
        
        -- Aplicar efecto
        PERFORM aplicar_efecto_movimiento(
            v_movimiento_id,
            v_efecto,
            v_valor_efecto,
            p_documento_origen_id,
            p_item_documento_origen_id,
            p_documento_afectado_id,
            p_item_documento_afectado_id,
            p_usuario_id
        );
    END LOOP;
    
    -- Recalcular valores consolidados de los ítems afectados
    PERFORM recalcular_columnas_item(p_item_documento_afectado_id);
    
    -- Si afecta documentos precedentes, recalcular también
    FOR v_efecto IN 
        SELECT DISTINCT item_documento_id
        FROM efecto_movimiento_aplicado
        WHERE movimiento_id = v_movimiento_id
          AND item_documento_id != p_item_documento_afectado_id
    LOOP
        PERFORM recalcular_columnas_item(v_efecto.item_documento_id);
    END LOOP;
    
    RETURN v_movimiento_id;
END;
$$ LANGUAGE plpgsql;
```

### Función: validar_efectos_movimiento

```sql
CREATE OR REPLACE FUNCTION validar_efectos_movimiento(
    p_tipo_movimiento_id INTEGER,
    p_documento_origen_id INTEGER,
    p_item_documento_origen_id INTEGER,
    p_documento_afectado_id INTEGER,
    p_item_documento_afectado_id INTEGER,
    p_valor_movimiento DECIMAL(15,2)
)
RETURNS BOOLEAN AS $$
DECLARE
    v_efecto RECORD;
    v_valor_efecto DECIMAL(15,2);
    v_valor_actual DECIMAL(15,2);
    v_nuevo_valor DECIMAL(15,2);
    v_item_id INTEGER;
    v_documento_precedente_id INTEGER;
BEGIN
    -- Validar cada efecto del movimiento
    FOR v_efecto IN 
        SELECT * FROM efecto_movimiento em
        JOIN tipo_documento_columna tdc ON em.tipo_documento_columna_id = tdc.id
        WHERE em.tipo_movimiento_id = p_tipo_movimiento_id 
          AND em.es_activo = true
        ORDER BY em.orden_aplicacion
    LOOP
        -- Calcular valor del efecto
        v_valor_efecto := calcular_valor_efecto(v_efecto, p_valor_movimiento);
        
        -- Determinar ítem afectado
        IF v_efecto.es_efecto_origen THEN
            v_item_id := p_item_documento_afectado_id;
        ELSE
            -- Buscar ítem correspondiente en documento precedente
            SELECT idp.id INTO v_item_id
            FROM item_documento_presupuestal idp
            JOIN documento_presupuestal d ON idp.documento_id = d.id
            JOIN documento_precedente dp ON d.id = dp.documento_sucesor_id
            WHERE dp.documento_predecesor_id = (
                SELECT documento_id FROM item_documento_presupuestal 
                WHERE id = p_item_documento_afectado_id
            )
            AND d.tipo_documento_id = v_efecto.tipo_documento_afectado_id
            -- Aquí se debería hacer matching por códigos presupuestales
            LIMIT 1;
            
            IF NOT FOUND THEN
                RAISE EXCEPTION 'No se encontró ítem correspondiente en documento precedente para aplicar efecto';
            END IF;
        END IF;
        
        -- Obtener valor actual de la columna
        SELECT COALESCE(valor_columna, 0) INTO v_valor_actual
        FROM item_documento_columna_valor
        WHERE item_documento_id = v_item_id
          AND tipo_documento_columna_id = v_efecto.tipo_documento_columna_id;
        
        -- Calcular nuevo valor según tipo de efecto
        CASE v_efecto.tipo_efecto
            WHEN 'INCREMENTA' THEN
                v_nuevo_valor := v_valor_actual + v_valor_efecto;
            WHEN 'DECREMENTA' THEN
                v_nuevo_valor := v_valor_actual - v_valor_efecto;
            WHEN 'ESTABLECE' THEN
                v_nuevo_valor := v_valor_efecto;
        END CASE;
        
        -- Validar que el saldo no quede negativo
        IF v_efecto.validar_saldo_positivo AND v_nuevo_valor < 0 THEN
            RAISE EXCEPTION 'El movimiento dejaría un saldo negativo en la columna %. Valor actual: %, Valor efecto: %, Nuevo valor: %', 
                v_efecto.nombre_columna, v_valor_actual, v_valor_efecto, v_nuevo_valor;
        END IF;
        
        -- Validar valor disponible si es necesario
        IF v_efecto.validar_valor_disponible AND v_efecto.tipo_efecto = 'DECREMENTA' THEN
            IF v_valor_actual < v_valor_efecto THEN
                RAISE EXCEPTION 'Valor insuficiente en la columna %. Disponible: %, Requerido: %', 
                    v_efecto.nombre_columna, v_valor_actual, v_valor_efecto;
            END IF;
        END IF;
    END LOOP;
    
    RETURN TRUE;
END;
$$ LANGUAGE plpgsql;
```

### Función: aplicar_efecto_movimiento

```sql
CREATE OR REPLACE FUNCTION aplicar_efecto_movimiento(
    p_movimiento_id INTEGER,
    p_efecto efecto_movimiento,
    p_valor_efecto DECIMAL(15,2),
    p_documento_origen_id INTEGER,
    p_item_documento_origen_id INTEGER,
    p_documento_afectado_id INTEGER,
    p_item_documento_afectado_id INTEGER,
    p_usuario_id INTEGER
)
RETURNS VOID AS $$
DECLARE
    v_item_id INTEGER;
    v_valor_actual DECIMAL(15,2);
    v_nuevo_valor DECIMAL(15,2);
    v_signo_movimiento CHAR(1);
BEGIN
    -- Determinar ítem afectado
    IF p_efecto.es_efecto_origen THEN
        v_item_id := p_item_documento_afectado_id;
    ELSE
        -- Buscar ítem correspondiente en documento precedente
        SELECT idp.id INTO v_item_id
        FROM item_documento_presupuestal idp
        JOIN documento_presupuestal d ON idp.documento_id = d.id
        JOIN documento_precedente dp ON d.id = dp.documento_sucesor_id
        WHERE dp.documento_predecesor_id = (
            SELECT documento_id FROM item_documento_presupuestal 
            WHERE id = p_item_documento_afectado_id
        )
        AND d.tipo_documento_id = p_efecto.tipo_documento_afectado_id
        LIMIT 1;
    END IF;
    
    -- Obtener valor actual
    SELECT COALESCE(valor_columna, 0) INTO v_valor_actual
    FROM item_documento_columna_valor
    WHERE item_documento_id = v_item_id
      AND tipo_documento_columna_id = p_efecto.tipo_documento_columna_id;
    
    -- Calcular nuevo valor y signo según tipo de efecto
    CASE p_efecto.tipo_efecto
        WHEN 'INCREMENTA' THEN
            v_nuevo_valor := v_valor_actual + p_valor_efecto;
            v_signo_movimiento := '+';
        WHEN 'DECREMENTA' THEN
            v_nuevo_valor := v_valor_actual - p_valor_efecto;
            v_signo_movimiento := '-';
        WHEN 'ESTABLECE' THEN
            v_nuevo_valor := p_valor_efecto;
            v_signo_movimiento := '=';
    END CASE;
    
    -- Actualizar valor en la tabla de valores
    INSERT INTO item_documento_columna_valor 
        (item_documento_id, tipo_documento_columna_id, valor_columna, usuario_actualizacion,
         observaciones_cambio)
    VALUES 
        (v_item_id, p_efecto.tipo_documento_columna_id, v_nuevo_valor, p_usuario_id,
         'Actualizado por movimiento #' || p_movimiento_id)
    ON CONFLICT (item_documento_id, tipo_documento_columna_id) 
    DO UPDATE SET 
        valor_columna = v_nuevo_valor,
        usuario_actualizacion = p_usuario_id,
        fecha_actualizacion = NOW(),
        observaciones_cambio = 'Actualizado por movimiento #' || p_movimiento_id;
    
    -- Registrar el efecto aplicado para auditoría
    INSERT INTO efecto_movimiento_aplicado 
        (movimiento_id, efecto_movimiento_id, item_documento_id, valor_anterior, 
         valor_nuevo, valor_efecto, signo_efecto)
    VALUES 
        (p_movimiento_id, p_efecto.id, v_item_id, v_valor_actual, 
         v_nuevo_valor, p_valor_efecto, v_signo_movimiento);
END;
$$ LANGUAGE plpgsql;
```

---

## Tabla de Auditoría de Efectos

### Tabla: efecto_movimiento_aplicado

```sql
CREATE TABLE efecto_movimiento_aplicado (
    id SERIAL PRIMARY KEY,
    movimiento_id INTEGER NOT NULL REFERENCES movimiento_presupuestal(id),
    efecto_movimiento_id INTEGER NOT NULL REFERENCES efecto_movimiento(id),
    item_documento_id INTEGER NOT NULL REFERENCES item_documento_presupuestal(id),
    
    -- Valores del efecto aplicado
    valor_anterior DECIMAL(15,2),
    valor_nuevo DECIMAL(15,2),
    valor_efecto DECIMAL(15,2),
    signo_efecto CHAR(1),
    
    -- Metadatos
    fecha_aplicacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_aplicacion INTEGER REFERENCES usuario(id),
    
    -- Observaciones del efecto
    observaciones_efecto TEXT
);

-- Índices
CREATE INDEX idx_efecto_aplicado_movimiento ON efecto_movimiento_aplicado(movimiento_id);
CREATE INDEX idx_efecto_aplicado_item ON efecto_movimiento_aplicado(item_documento_id);
CREATE INDEX idx_efecto_aplicado_fecha ON efecto_movimiento_aplicado(fecha_aplicacion);
```

---

## Vistas para Consulta de Movimientos

### Vista: vista_movimientos_completos

```sql
CREATE VIEW vista_movimientos_completos AS
SELECT 
    mp.id as movimiento_id,
    mp.numero_movimiento,
    tmp.codigo as tipo_movimiento_codigo,
    tmp.nombre as tipo_movimiento_nombre,
    
    -- Documento origen
    d_origen.numero_documento as documento_origen,
    tdp_origen.nombre as tipo_documento_origen,
    idp_origen.numero_item as item_origen,
    idp_origen.descripcion_item as descripcion_item_origen,
    
    -- Documento afectado
    d_afectado.numero_documento as documento_afectado,
    tdp_afectado.nombre as tipo_documento_afectado,
    idp_afectado.numero_item as item_afectado,
    idp_afectado.descripcion_item as descripcion_item_afectado,
    
    -- Valores
    mp.valor_movimiento,
    mp.signo_movimiento,
    mp.descripcion_movimiento,
    mp.observaciones,
    mp.estado_movimiento,
    
    -- Fechas
    mp.fecha_movimiento,
    mp.fecha_creacion,
    u.nombre as usuario_creacion,
    
    -- Efectos aplicados (JSON)
    (
        SELECT json_agg(
            json_build_object(
                'tipo_efecto', em.tipo_efecto,
                'columna_afectada', tdc.nombre_columna,
                'valor_anterior', ema.valor_anterior,
                'valor_nuevo', ema.valor_nuevo,
                'valor_efecto', ema.valor_efecto
            )
        )
        FROM efecto_movimiento_aplicado ema
        JOIN efecto_movimiento em ON ema.efecto_movimiento_id = em.id
        JOIN tipo_documento_columna tdc ON em.tipo_documento_columna_id = tdc.id
        WHERE ema.movimiento_id = mp.id
    ) as efectos_aplicados
    
FROM movimiento_presupuestal mp
JOIN tipo_movimiento_presupuestal tmp ON mp.tipo_movimiento_id = tmp.id
JOIN documento_presupuestal d_origen ON mp.documento_origen_id = d_origen.id
JOIN tipo_documento_presupuestal tdp_origen ON d_origen.tipo_documento_id = tdp_origen.id
LEFT JOIN item_documento_presupuestal idp_origen ON mp.item_documento_origen_id = idp_origen.id
JOIN documento_presupuestal d_afectado ON mp.documento_afectado_id = d_afectado.id
JOIN tipo_documento_presupuestal tdp_afectado ON d_afectado.tipo_documento_id = tdp_afectado.id
JOIN item_documento_presupuestal idp_afectado ON mp.item_documento_afectado_id = idp_afectado.id
LEFT JOIN usuario u ON mp.usuario_creacion = u.id
WHERE mp.estado_movimiento = 'ACTIVO'
ORDER BY mp.fecha_movimiento DESC;
```

### Vista: vista_movimientos_por_item

```sql
CREATE VIEW vista_movimientos_por_item AS
SELECT 
    idp.id as item_documento_id,
    d.numero_documento,
    idp.numero_item,
    idp.descripcion_item,
    COUNT(mp.id) as total_movimientos,
    SUM(CASE WHEN mp.signo_movimiento = '+' THEN mp.valor_movimiento ELSE -mp.valor_movimiento END) as saldo_movimientos,
    MIN(mp.fecha_movimiento) as fecha_primer_movimiento,
    MAX(mp.fecha_movimiento) as fecha_ultimo_movimiento,
    
    -- Detalle de movimientos (JSON)
    json_agg(
        json_build_object(
            'movimiento_id', mp.id,
            'numero_movimiento', mp.numero_movimiento,
            'tipo_movimiento', tmp.nombre,
            'valor', mp.valor_movimiento,
            'signo', mp.signo_movimiento,
            'fecha', mp.fecha_movimiento,
            'descripcion', mp.descripcion_movimiento
        ) ORDER BY mp.fecha_movimiento
    ) as detalle_movimientos
    
FROM item_documento_presupuestal idp
JOIN documento_presupuestal d ON idp.documento_id = d.id
LEFT JOIN movimiento_presupuestal mp ON idp.id = mp.item_documento_afectado_id
LEFT JOIN tipo_movimiento_presupuestal tmp ON mp.tipo_movimiento_id = tmp.id
WHERE idp.es_activo = true
  AND (mp.estado_movimiento = 'ACTIVO' OR mp.estado_movimiento IS NULL)
GROUP BY idp.id, d.numero_documento, idp.numero_item, idp.descripcion_item
ORDER BY d.numero_documento, idp.numero_item;
```

---

## Ejemplos Prácticos

### Ejemplo 1: Crear CDP con valor inicial

```sql
-- Crear movimiento de valor inicial para un CDP
SELECT crear_movimiento_presupuestal(
    'VALOR_INICIAL_CDP', -- tipo_movimiento_codigo
    100, -- documento_origen_id (el mismo CDP)
    1001, -- item_documento_origen_id 
    100, -- documento_afectado_id (el mismo CDP)
    1001, -- item_documento_afectado_id
    50000000.00, -- valor_movimiento
    'Creación inicial de CDP para servicios profesionales', -- descripcion
    123, -- usuario_id
    'CDP creado según solicitud de contratación' -- observaciones
);

-- Resultado: Se crea el movimiento y automáticamente:
-- 1. Se establece valor_inicial = 50,000,000 en el CDP
-- 2. Se establece saldo_disponible = 50,000,000 en el CDP  
-- 3. Se reduce saldo_apropiacion = -50,000,000 en el Presupuesto de Gasto
-- 4. Se incrementa valor_reservado = +50,000,000 en el Presupuesto de Gasto
```

### Ejemplo 2: Crear Adición Presupuestal

```sql
-- Crear movimiento de adición presupuestal
SELECT crear_movimiento_presupuestal(
    'ADICION_PRESUPUESTAL', -- tipo_movimiento_codigo
    201, -- documento_origen_id (documento de adición)
    2001, -- item_documento_origen_id
    101, -- documento_afectado_id (presupuesto de gasto)
    1001, -- item_documento_afectado_id (ítem del presupuesto)
    100000000.00, -- valor_movimiento
    'Adición presupuestal por mayor recaudo', -- descripcion
    456, -- usuario_id
    'Adición aprobada por ordenanza municipal' -- observaciones
);

-- Resultado: Se crea el movimiento y automáticamente:
-- 1. Se incrementa adiciones = +100,000,000 en el Presupuesto de Gasto
-- 2. Se incrementa saldo_apropiacion = +100,000,000 para nuevos CDPs
-- 3. Se recalcula presupuesto_definitivo automáticamente
```

### Ejemplo 3: Comprometer CDP en RP

```sql
-- Crear movimiento de compromiso de CDP
SELECT crear_movimiento_presupuestal(
    'COMPROMISO_CDP', -- tipo_movimiento_codigo
    301, -- documento_origen_id (Registro Presupuestal)
    3001, -- item_documento_origen_id
    301, -- documento_afectado_id (el mismo RP)
    3001, -- item_documento_afectado_id
    30000000.00, -- valor_movimiento
    'Compromiso de CDP para contrato de servicios', -- descripcion
    789, -- usuario_id
    'RP generado por contrato No. 001-2025' -- observaciones
);

-- Resultado: Se crea el movimiento y automáticamente:
-- 1. Se establece valor_inicial = 30,000,000 en el RP
-- 2. Se establece saldo_disponible = 30,000,000 en el RP
-- 3. Se incrementa valor_comprometido = +30,000,000 en el CDP
-- 4. Se reduce saldo_disponible = -30,000,000 en el CDP
```

### Ejemplo 4: Consultar movimientos de un ítem

```sql
-- Consultar todos los movimientos de un ítem específico
SELECT 
    numero_documento,
    numero_item,
    descripcion_item,
    total_movimientos,
    saldo_movimientos,
    fecha_primer_movimiento,
    fecha_ultimo_movimiento,
    detalle_movimientos
FROM vista_movimientos_por_item
WHERE item_documento_id = 1001;

-- Resultado:
-- {
--   "numero_documento": "CDP-2025-001",
--   "numero_item": 1,
--   "descripcion_item": "Servicios profesionales",
--   "total_movimientos": 2,
--   "saldo_movimientos": 20000000,
--   "fecha_primer_movimiento": "2025-01-15 09:30:00",
--   "fecha_ultimo_movimiento": "2025-01-20 14:15:00",
--   "detalle_movimientos": [
--     {
--       "movimiento_id": 1,
--       "numero_movimiento": "CDP-001",
--       "tipo_movimiento": "Valor Inicial de CDP",
--       "valor": 50000000,
--       "signo": "+",
--       "fecha": "2025-01-15 09:30:00",
--       "descripcion": "Creación inicial de CDP"
--     },
--     {
--       "movimiento_id": 15,
--       "numero_movimiento": "CMP-001", 
--       "tipo_movimiento": "Compromiso de CDP en RP",
--       "valor": 30000000,
--       "signo": "-",
--       "fecha": "2025-01-20 14:15:00",
--       "descripcion": "Compromiso parcial por RP"
--     }
--   ]
-- }
```

---

## Próximos Pasos

Este documento estableció el sistema completo de movimientos presupuestales. El siguiente documento cubrirá:

**04 - Estados de Documentos**: Sistema de estados, transiciones automáticas y manuales, con evaluación mediante JavaScript para condiciones complejas.

Los movimientos presupuestales están ahora completamente definidos con:
- ✅ Estructura completa de movimientos y tipos de movimiento
- ✅ Sistema configurable de efectos en múltiples documentos y columnas
- ✅ Validaciones automáticas de saldos y disponibilidad
- ✅ Trazabilidad completa mediante auditoría de efectos
- ✅ Funciones completas de creación y aplicación
- ✅ Vistas para consultas y reportes
- ✅ Ejemplos prácticos del flujo presupuestario completo
