# 02 - Ítems de Documentos Presupuestales

## Descripción General

Los **Ítems de Documentos Presupuestales** son las unidades básicas de información dentro de cada documento presupuestal. Cada ítem representa una línea específica del documento que contiene información detallada sobre clasificaciones presupuestales, valores y metadatos asociados.

## Conceptos Fundamentales

### ¿Qué es un Ítem de Documento?

Un ítem de documento presupuestal es:

- **Línea de detalle**: Cada ítem representa una línea específica dentro del documento
- **Portador de clasificaciones**: Contiene los códigos presupuestales que definen la clasificación
- **Contenedor de valores**: Almacena los valores monetarios en columnas específicas según el tipo de documento
- **Unidad de trazabilidad**: Permite seguir el rastro de recursos específicos a través del ciclo presupuestario
- **Elemento configurable**: Su estructura se adapta según el tipo de documento que lo contiene

### Características Principales

- **Estructura normalizada**: Cada valor de columna se almacena por separado
- **Flexibilidad**: Las columnas disponibles dependen del tipo de documento
- **Valores no editables**: Los valores de las columnas nunca se editan directamente
- **Valores monetarios**: Todas las columnas de valores son exclusivamente cantidades de dinero
- **Gestión por movimientos**: Los valores se establecen y modifican únicamente mediante movimientos presupuestales
- **Control de valores**: Cada ítem mantiene su valor inicial y saldo actual
- **Integridad**: Validaciones automáticas según las reglas de negocio
- **Trazabilidad**: Historial completo de cambios y movimientos

---

## Estructura de Datos

### Tabla Principal: item_documento_presupuestal

```sql
CREATE TABLE item_documento_presupuestal (
    id SERIAL PRIMARY KEY,
    documento_id INTEGER NOT NULL REFERENCES documento_presupuestal(id),
    numero_item INTEGER NOT NULL,
    
    -- Descripción del ítem
    descripcion_item VARCHAR(500),
    observaciones TEXT,
    
    -- Valores consolidados del ítem (calculados automáticamente)
    valor_inicial DECIMAL(15,2) DEFAULT 0.00, -- Valor del primer movimiento
    saldo_actual DECIMAL(15,2) DEFAULT 0.00,  -- Saldo actual después de todos los movimientos
    
    -- Metadatos del ítem
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER REFERENCES usuario(id),
    usuario_actualizacion INTEGER REFERENCES usuario(id),
    
    -- Datos adicionales en formato JSON para flexibilidad
    datos_adicionales JSON,
    
    -- Índice único por documento
    UNIQUE(documento_id, numero_item)
);

-- Índices para optimizar consultas
CREATE INDEX idx_item_documento_documento_id ON item_documento_presupuestal(documento_id);
CREATE INDEX idx_item_documento_activo ON item_documento_presupuestal(es_activo) WHERE es_activo = true;
```

### Tabla: item_documento_codigo

```sql
CREATE TABLE item_documento_codigo (
    id SERIAL PRIMARY KEY,
    item_documento_id INTEGER NOT NULL REFERENCES item_documento_presupuestal(id),
    tipo_codigo_id INTEGER NOT NULL REFERENCES tipo_codigo(id),
    codigo_id INTEGER NOT NULL REFERENCES codigo(id),
    
    -- Metadatos de la asignación
    es_principal BOOLEAN DEFAULT FALSE, -- Si es el código principal para este tipo
    orden_presentacion INTEGER DEFAULT 1,
    fecha_asignacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_asignacion INTEGER REFERENCES usuario(id),
    
    -- Índice único para evitar duplicados
    UNIQUE(item_documento_id, tipo_codigo_id, codigo_id)
);

-- Índices para optimizar consultas
CREATE INDEX idx_item_codigo_item_id ON item_documento_codigo(item_documento_id);
CREATE INDEX idx_item_codigo_tipo_codigo ON item_documento_codigo(tipo_codigo_id);
CREATE INDEX idx_item_codigo_codigo_id ON item_documento_codigo(codigo_id);
```

---

## Columnas Específicas por Tipo de Documento

Cada tipo de documento tiene columnas específicas que almacenan valores monetarios y otros datos relevantes para ese tipo particular de operación presupuestal.

### Tabla: tipo_documento_columna

```sql
CREATE TABLE tipo_documento_columna (
    id SERIAL PRIMARY KEY,
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id),
    codigo_columna VARCHAR(50) NOT NULL,
    nombre_columna VARCHAR(100) NOT NULL,
    descripcion_columna TEXT,
    
    -- Configuración de la columna (SIEMPRE monetaria)
    tipo_dato VARCHAR(20) DEFAULT 'DECIMAL' CHECK (tipo_dato = 'DECIMAL'),
    precision_decimal INTEGER DEFAULT 18,
    escala_decimal INTEGER DEFAULT 2,
    valor_defecto DECIMAL(18,2) DEFAULT 0,
    
    -- Comportamiento de la columna
    es_calculada BOOLEAN DEFAULT FALSE,
    formula_calculo TEXT, -- Fórmula para columnas calculadas
    es_editable BOOLEAN DEFAULT FALSE, -- NUNCA editable directamente
    es_visible BOOLEAN DEFAULT TRUE,
    es_requerida BOOLEAN DEFAULT FALSE,
    
    -- Configuración de interfaz
    orden_presentacion INTEGER DEFAULT 1,
    ancho_columna INTEGER DEFAULT 150,
    alineacion VARCHAR(10) DEFAULT 'right', -- left, center, right
    formato_presentacion VARCHAR(50) DEFAULT 'currency', -- Formato monetario
    
    -- Validaciones
    valor_minimo DECIMAL(18,2),
    valor_maximo DECIMAL(18,2),
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER REFERENCES usuario(id),
    
    -- Índice único por tipo de documento y código
    UNIQUE(tipo_documento_id, codigo_columna)
);
```

### Configuración de Columnas por Tipo de Documento

#### Presupuesto de Gasto (PG)

```sql
INSERT INTO tipo_documento_columna VALUES
(1, 1, 'presupuesto_inicial', 'Presupuesto Inicial', 'Valor inicial aprobado para la vigencia', 
 'DECIMAL', 18, 2, 0, false, null, true, true, false, 1, 150, 'right', 'currency', null, null, null, true, NOW(), 1),

(2, 1, 'adiciones', 'Adiciones', 'Suma de todas las adiciones presupuestales realizadas', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 2, 150, 'right', 'currency', 0, null, null, true, NOW(), 1),

(3, 1, 'recortes', 'Recortes', 'Suma de todos los recortes presupuestales realizados', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 3, 150, 'right', 'currency', 0, null, null, true, NOW(), 1),

(4, 1, 'traslados', 'Traslados', 'Neto de traslados (entradas menos salidas)', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 4, 150, 'right', 'currency', null, null, null, true, NOW(), 1),

(5, 1, 'presupuesto_definitivo', 'Presupuesto Definitivo', 'Presupuesto vigente después de modificaciones', 
 'DECIMAL', 18, 2, 0, true, 'presupuesto_inicial + adiciones - recortes + traslados', false, true, false, 5, 150, 'right', 'currency', null, null, null, true, NOW(), 1),

(6, 1, 'valor_reservado', 'Valor Reservado', 'Valor comprometido en CDPs vigentes', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 6, 150, 'right', 'currency', 0, null, null, true, NOW(), 1),

(7, 1, 'saldo_apropiacion', 'Saldo Apropiación', 'Saldo disponible para nuevos compromisos', 
 'DECIMAL', 18, 2, 0, true, 'presupuesto_definitivo - valor_reservado', false, true, false, 7, 150, 'right', 'currency', null, null, null, true, NOW(), 1);
```

#### Certificado de Disponibilidad Presupuestal (CDP)

```sql
INSERT INTO tipo_documento_columna VALUES
(8, 5, 'valor_inicial', 'Valor Inicial', 'Valor inicial del CDP', 
 'DECIMAL', 18, 2, 0, false, null, true, true, true, 1, 150, 'right', 'currency', 1, null, null, true, NOW(), 1),

(9, 5, 'valor_comprometido', 'Valor Comprometido', 'Valor comprometido en RPs', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 2, 150, 'right', 'currency', 0, null, null, true, NOW(), 1),

(10, 5, 'valor_liberado', 'Valor Liberado', 'Valor liberado y devuelto al PG', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 3, 150, 'right', 'currency', 0, null, null, true, NOW(), 1),

(11, 5, 'saldo_comprometer', 'Saldo por Comprometer', 'Saldo disponible para comprometer', 
 'DECIMAL', 18, 2, 0, true, 'valor_inicial - valor_comprometido - valor_liberado', false, true, false, 4, 150, 'right', 'currency', null, null, null, true, NOW(), 1),

(12, 5, 'porcentaje_ejecutado', 'Porcentaje Ejecutado', 'Porcentaje de ejecución del CDP', 
 'DECIMAL', 5, 2, 0, true, '(valor_comprometido / valor_inicial) * 100', false, true, false, 5, 120, 'right', 'percentage', null, null, null, true, NOW(), 1);
```

#### Registro Presupuestal (RP)

```sql
INSERT INTO tipo_documento_columna VALUES
(13, 6, 'valor_inicial', 'Valor Inicial', 'Valor inicial del RP', 
 'DECIMAL', 18, 2, 0, false, null, true, true, true, 1, 150, 'right', 'currency', 1, null, null, true, NOW(), 1),

(14, 6, 'valor_obligaciones', 'Valor Obligaciones', 'Valor causado como obligaciones', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 2, 150, 'right', 'currency', 0, null, null, true, NOW(), 1),

(15, 6, 'valor_liquidaciones', 'Valor Liquidaciones', 'Valor liquidado y devuelto al CDP', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 3, 150, 'right', 'currency', 0, null, null, true, NOW(), 1),

(16, 6, 'saldo_obligar', 'Saldo por Obligar', 'Saldo disponible para causar obligaciones', 
 'DECIMAL', 18, 2, 0, true, 'valor_inicial - valor_obligaciones - valor_liquidaciones', false, true, false, 4, 150, 'right', 'currency', null, null, null, true, NOW(), 1);
```

#### Orden de Pago (OP)

```sql
INSERT INTO tipo_documento_columna VALUES
(17, 7, 'valor_inicial', 'Valor Inicial', 'Valor inicial de la orden de pago', 
 'DECIMAL', 18, 2, 0, false, null, true, true, true, 1, 150, 'right', 'currency', 1, null, null, true, NOW(), 1),

(18, 7, 'valor_pagos', 'Valor Pagos', 'Valor efectivamente pagado', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 2, 150, 'right', 'currency', 0, null, null, true, NOW(), 1),

(19, 7, 'valor_anulaciones', 'Valor Anulaciones', 'Valor anulado de la orden', 
 'DECIMAL', 18, 2, 0, false, null, false, true, false, 3, 150, 'right', 'currency', 0, null, null, true, NOW(), 1),

(20, 7, 'saldo_pagar', 'Saldo por Pagar', 'Saldo pendiente de pago', 
 'DECIMAL', 18, 2, 0, true, 'valor_inicial - valor_pagos - valor_anulaciones', false, true, false, 4, 150, 'right', 'currency', null, null, null, true, NOW(), 1);
```

---

## Almacenamiento Normalizado de Valores

Los valores de las columnas se almacenan de forma normalizada, donde cada valor de cada columna para cada ítem se guarda en una fila separada.

**IMPORTANTE**: Los valores almacenados en esta tabla son únicamente para consulta y reportes. **NUNCA se editan directamente**. Todos los cambios de valores se realizan exclusivamente mediante el sistema de movimientos presupuestales, que garantiza la trazabilidad y auditabilidad completa.

### Tabla: item_documento_columna_valor

```sql
CREATE TABLE item_documento_columna_valor (
    id SERIAL PRIMARY KEY,
    item_documento_id INTEGER NOT NULL REFERENCES item_documento_presupuestal(id),
    tipo_documento_columna_id INTEGER NOT NULL REFERENCES tipo_documento_columna(id),
    
    -- Valor monetario de la columna (TODAS las columnas son valores monetarios)
    valor_columna DECIMAL(18,2) DEFAULT 0,
    
    -- Metadatos del valor (generados automáticamente por movimientos)
    fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_actualizacion INTEGER REFERENCES usuario(id),
    observaciones_cambio TEXT,
    
    -- Índice único para evitar duplicados
    UNIQUE(item_documento_id, tipo_documento_columna_id)
);

-- Índices para optimizar consultas
CREATE INDEX idx_item_columna_valor_item ON item_documento_columna_valor(item_documento_id);
CREATE INDEX idx_item_columna_valor_columna ON item_documento_columna_valor(tipo_documento_columna_id);
CREATE INDEX idx_item_columna_valor_valor ON item_documento_columna_valor(valor_columna) WHERE valor_columna IS NOT NULL;
```

---

## Funciones de Gestión de Ítems

### Función: crear_item_documento

```sql
CREATE OR REPLACE FUNCTION crear_item_documento(
    p_documento_id INTEGER,
    p_descripcion_item VARCHAR(500),
    p_codigos JSON, -- Array de objetos {tipo_codigo_id, codigo_id}
    p_valores JSON, -- Objeto con {codigo_columna: valor}
    p_usuario_id INTEGER
)
RETURNS INTEGER AS $$
DECLARE
    v_item_id INTEGER;
    v_numero_item INTEGER;
    v_codigo RECORD;
    v_valor RECORD;
    v_tipo_documento_id INTEGER;
    v_columna_id INTEGER;
BEGIN
    -- Obtener tipo de documento
    SELECT tipo_documento_id INTO v_tipo_documento_id
    FROM documento_presupuestal
    WHERE id = p_documento_id;
    
    -- Obtener siguiente número de ítem
    SELECT COALESCE(MAX(numero_item), 0) + 1 INTO v_numero_item
    FROM item_documento_presupuestal
    WHERE documento_id = p_documento_id;
    
    -- Crear el ítem
    INSERT INTO item_documento_presupuestal 
        (documento_id, numero_item, descripcion_item, usuario_creacion, usuario_actualizacion)
    VALUES 
        (p_documento_id, v_numero_item, p_descripcion_item, p_usuario_id, p_usuario_id)
    RETURNING id INTO v_item_id;
    
    -- Insertar códigos asociados
    FOR v_codigo IN SELECT * FROM json_populate_recordset(null::RECORD, p_codigos) AS x(tipo_codigo_id INTEGER, codigo_id INTEGER)
    LOOP
        -- Validar que el tipo de código esté permitido para este tipo de documento
        IF NOT EXISTS (
            SELECT 1 FROM tipo_documento_codigo_permitido
            WHERE tipo_documento_id = v_tipo_documento_id 
              AND tipo_codigo_id = v_codigo.tipo_codigo_id
              AND es_activo = true
        ) THEN
            RAISE EXCEPTION 'Tipo de código % no permitido para este tipo de documento', v_codigo.tipo_codigo_id;
        END IF;
        
        INSERT INTO item_documento_codigo 
            (item_documento_id, tipo_codigo_id, codigo_id, usuario_asignacion)
        VALUES 
            (v_item_id, v_codigo.tipo_codigo_id, v_codigo.codigo_id, p_usuario_id);
    END LOOP;
    
    -- Insertar valores de columnas
    FOR v_valor IN SELECT * FROM json_each_text(p_valores)
    LOOP
        -- Obtener ID de la columna
        SELECT id INTO v_columna_id
        FROM tipo_documento_columna
        WHERE tipo_documento_id = v_tipo_documento_id 
          AND codigo_columna = v_valor.key
          AND es_activo = true;
        
        IF FOUND THEN
            INSERT INTO item_documento_columna_valor 
                (item_documento_id, tipo_documento_columna_id, valor_columna, usuario_actualizacion)
            VALUES 
                (v_item_id, v_columna_id, v_valor.value::DECIMAL, p_usuario_id);
        END IF;
    END LOOP;
    
    -- Calcular columnas calculadas
    PERFORM recalcular_columnas_item(v_item_id);
    
    RETURN v_item_id;
END;
$$ LANGUAGE plpgsql;
```

### Función: actualizar_valor_columna_item (OBSOLETA)

**IMPORTANTE**: Esta función está obsoleta y no debe utilizarse. Los valores de las columnas se actualizan únicamente mediante el sistema de movimientos presupuestales.

```sql
-- FUNCIÓN OBSOLETA - NO USAR
-- Los valores se actualizan únicamente mediante movimientos presupuestales
CREATE OR REPLACE FUNCTION actualizar_valor_columna_item(
    p_item_documento_id INTEGER,
    p_codigo_columna VARCHAR(50),
    p_nuevo_valor DECIMAL(18,2),
    p_usuario_id INTEGER,
    p_observaciones TEXT DEFAULT NULL
)
RETURNS BOOLEAN AS $$
BEGIN
    RAISE EXCEPTION 'FUNCIÓN OBSOLETA: Los valores se actualizan únicamente mediante movimientos presupuestales. Use el sistema de movimientos en su lugar.';
    RETURN FALSE;
END;
$$ LANGUAGE plpgsql;
```

**Reemplazo**: Utilice el sistema de movimientos presupuestales que se documenta en el siguiente capítulo. 
        (item_documento_id, columna_id, valor_anterior, valor_nuevo, usuario_id, observaciones)
    VALUES 
        (p_item_documento_id, v_columna_id, v_valor_anterior, p_nuevo_valor, p_usuario_id, p_observaciones);
    
    RETURN TRUE;
END;
$$ LANGUAGE plpgsql;
```

### Función: recalcular_columnas_item (MODIFICADA)

Esta función ahora únicamente actualiza los campos consolidados del ítem basándose en los movimientos existentes.

```sql
CREATE OR REPLACE FUNCTION recalcular_columnas_item(
    p_item_documento_id INTEGER
)
RETURNS VOID AS $$
DECLARE
    v_saldo_total DECIMAL(18,2) := 0;
    v_valor_inicial DECIMAL(18,2) := 0;
BEGIN
    -- Calcular valor inicial (primer movimiento)
    SELECT COALESCE(valor_movimiento, 0) INTO v_valor_inicial
    FROM movimiento_presupuestal
    WHERE item_documento_id = p_item_documento_id
    ORDER BY fecha_movimiento ASC, id ASC
    LIMIT 1;
    
    -- Calcular saldo actual (suma de todos los movimientos)
    SELECT COALESCE(SUM(valor_movimiento), 0) INTO v_saldo_total
    FROM movimiento_presupuestal
    WHERE item_documento_id = p_item_documento_id;
    
    -- Actualizar campos consolidados en el ítem
    UPDATE item_documento_presupuestal 
    SET 
        valor_inicial = v_valor_inicial,
        saldo_actual = v_saldo_total,
        fecha_actualizacion = NOW()
    WHERE id = p_item_documento_id;
    
    -- Recalcular valores de columnas desde movimientos
    -- (Este proceso se detallará en el documento de movimientos)
    PERFORM actualizar_valores_columnas_desde_movimientos(p_item_documento_id);
END;
$$ LANGUAGE plpgsql;
```

### Función: evaluar_formula_columna

```sql
CREATE OR REPLACE FUNCTION evaluar_formula_columna(
    p_item_documento_id INTEGER,
    p_formula TEXT
)
RETURNS DECIMAL(18,2) AS $$
DECLARE
    v_columna RECORD;
    v_formula_evaluada TEXT;
    v_valor_columna DECIMAL(18,2);
    v_resultado DECIMAL(18,2);
BEGIN
    -- Inicializar fórmula
    v_formula_evaluada := p_formula;
    
    -- Obtener todas las columnas del tipo de documento
    FOR v_columna IN 
        SELECT tdc.codigo_columna
        FROM tipo_documento_columna tdc
        JOIN item_documento_presupuestal idp ON idp.documento_id IN (
            SELECT d.id FROM documento_presupuestal d WHERE d.tipo_documento_id = tdc.tipo_documento_id
        )
        WHERE idp.id = p_item_documento_id
          AND tdc.es_activo = true
          AND v_formula_evaluada LIKE '%' || tdc.codigo_columna || '%'
    LOOP
        -- Obtener valor actual de la columna
        SELECT COALESCE(idcv.valor_columna, 0) INTO v_valor_columna
        FROM item_documento_columna_valor idcv
        JOIN tipo_documento_columna tdc ON idcv.tipo_documento_columna_id = tdc.id
        WHERE idcv.item_documento_id = p_item_documento_id 
          AND tdc.codigo_columna = v_columna.codigo_columna;
        
        -- Reemplazar en la fórmula
        v_formula_evaluada := REPLACE(v_formula_evaluada, v_columna.codigo_columna, COALESCE(v_valor_columna, 0)::TEXT);
    END LOOP;
    
    -- Evaluar expresión matemática
    BEGIN
        EXECUTE 'SELECT (' || v_formula_evaluada || ')::DECIMAL(18,2)' INTO v_resultado;
    EXCEPTION 
        WHEN OTHERS THEN
            -- En caso de error, retornar 0
            v_resultado := 0;
    END;
    
    RETURN COALESCE(v_resultado, 0);
END;
$$ LANGUAGE plpgsql;
```

---

## Vistas para Consulta de Ítems

### Vista: vista_items_completos

```sql
CREATE VIEW vista_items_completos AS
SELECT 
    idp.id as item_id,
    idp.documento_id,
    d.numero_documento,
    d.tipo_documento_id,
    tdp.codigo as tipo_documento_codigo,
    tdp.nombre as tipo_documento_nombre,
    idp.numero_item,
    idp.descripcion_item,
    
    -- Códigos asociados (JSON)
    (
        SELECT json_agg(
            json_build_object(
                'tipo_codigo_id', idc.tipo_codigo_id,
                'tipo_codigo', tc.codigo,
                'tipo_codigo_nombre', tc.nombre,
                'codigo_id', idc.codigo_id,
                'codigo', c.codigo,
                'codigo_nombre', c.nombre
            )
        )
        FROM item_documento_codigo idc
        JOIN tipo_codigo tc ON idc.tipo_codigo_id = tc.id
        JOIN codigo c ON idc.codigo_id = c.id
        WHERE idc.item_documento_id = idp.id
    ) as codigos_asociados,
    
    -- Valores de columnas (JSON)
    (
        SELECT json_object_agg(
            tdc.codigo_columna,
            COALESCE(idcv.valor_columna, tdc.valor_defecto)
        )
        FROM tipo_documento_columna tdc
        LEFT JOIN item_documento_columna_valor idcv ON 
            idcv.item_documento_id = idp.id AND 
            idcv.tipo_documento_columna_id = tdc.id
        WHERE tdc.tipo_documento_id = d.tipo_documento_id
          AND tdc.es_activo = true
    ) as valores_columnas,
    
    idp.fecha_creacion,
    idp.fecha_actualizacion,
    u_creacion.nombre as usuario_creacion,
    u_actualizacion.nombre as usuario_actualizacion
    
FROM item_documento_presupuestal idp
JOIN documento_presupuestal d ON idp.documento_id = d.id
JOIN tipo_documento_presupuestal tdp ON d.tipo_documento_id = tdp.id
LEFT JOIN usuario u_creacion ON idp.usuario_creacion = u_creacion.id
LEFT JOIN usuario u_actualizacion ON idp.usuario_actualizacion = u_actualizacion.id
WHERE idp.es_activo = true;
```

### Vista: vista_items_por_rubro

```sql
CREATE VIEW vista_items_por_rubro AS
SELECT 
    c_rubro.codigo as codigo_rubro,
    c_rubro.nombre as nombre_rubro,
    COUNT(DISTINCT idp.id) as total_items,
    COUNT(DISTINCT idp.documento_id) as total_documentos,
    SUM(CASE WHEN tdc.codigo_columna = 'valor_inicial' THEN idcv.valor_columna ELSE 0 END) as total_valor_inicial,
    SUM(CASE WHEN tdc.codigo_columna = 'presupuesto_definitivo' THEN idcv.valor_columna ELSE 0 END) as total_presupuesto_definitivo,
    SUM(CASE WHEN tdc.codigo_columna = 'saldo_apropiacion' THEN idcv.valor_columna ELSE 0 END) as total_saldo_disponible
    
FROM item_documento_presupuestal idp
JOIN documento_presupuestal d ON idp.documento_id = d.id
JOIN item_documento_codigo idc ON idp.id = idc.item_documento_id
JOIN tipo_codigo tc ON idc.tipo_codigo_id = tc.id AND tc.codigo = 'RUBRO'
JOIN codigo c_rubro ON idc.codigo_id = c_rubro.id
LEFT JOIN item_documento_columna_valor idcv ON idp.id = idcv.item_documento_id
LEFT JOIN tipo_documento_columna tdc ON idcv.tipo_documento_columna_id = tdc.id

WHERE idp.es_activo = true
  AND d.estado IN ('APROBADO', 'EXPEDIDO', 'MODIFICADO')
  AND EXTRACT(YEAR FROM d.fecha_documento) = EXTRACT(YEAR FROM CURRENT_DATE)

GROUP BY c_rubro.codigo, c_rubro.nombre
ORDER BY c_rubro.codigo;
```

---

## Tabla de Auditoría

### Tabla: auditoria_cambios_columnas

```sql
CREATE TABLE auditoria_cambios_columnas (
    id SERIAL PRIMARY KEY,
    item_documento_id INTEGER NOT NULL REFERENCES item_documento_presupuestal(id),
    columna_id INTEGER NOT NULL REFERENCES tipo_documento_columna(id),
    valor_anterior DECIMAL(18,2),
    valor_nuevo DECIMAL(18,2),
    tipo_cambio VARCHAR(20) DEFAULT 'MANUAL', -- MANUAL, AUTOMATICO, CALCULO, MOVIMIENTO
    usuario_id INTEGER REFERENCES usuario(id),
    movimiento_id INTEGER, -- Si el cambio fue por un movimiento presupuestal
    observaciones TEXT,
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_usuario INET,
    datos_contexto JSON
);

-- Índices para auditoría
CREATE INDEX idx_auditoria_item ON auditoria_cambios_columnas(item_documento_id);
CREATE INDEX idx_auditoria_fecha ON auditoria_cambios_columnas(fecha_cambio);
CREATE INDEX idx_auditoria_usuario ON auditoria_cambios_columnas(usuario_id);
```

---

## Ejemplos Prácticos

### Ejemplo 1: Crear ítem de CDP

```sql
-- Crear un ítem de CDP con códigos y valores
SELECT crear_item_documento(
    100, -- documento_id (CDP)
    'Contratación servicios profesionales', -- descripción
    '[
        {"tipo_codigo_id": 1, "codigo_id": 1001}, 
        {"tipo_codigo_id": 2, "codigo_id": 2001},
        {"tipo_codigo_id": 6, "codigo_id": 3001}
    ]'::json, -- códigos (rubro, fuente, tercero)
    '{"valor_inicial": 50000000}'::json, -- valores
    123 -- usuario_id
);
-- Retorna: ID del ítem creado
```

### Ejemplo 2: Actualizar valor de columna (OBSOLETO)

**IMPORTANTE**: Este ejemplo muestra el método OBSOLETO. Los valores se actualizan únicamente mediante movimientos presupuestales.

```sql
-- MÉTODO OBSOLETO - NO USAR
-- Anteriormente se podía actualizar valores directamente:
SELECT actualizar_valor_columna_item(
    1001, -- item_documento_id
    'valor_comprometido', -- codigo_columna
    30000000.00, -- nuevo_valor
    456, -- usuario_id
    'Compromiso parcial por RP-2025-001' -- observaciones
);
-- Ahora retorna: ERROR - Función obsoleta

-- MÉTODO CORRECTO - Usar sistema de movimientos:
-- (Se detallará en el documento 04-MOVIMIENTOS_PRESUPUESTALES.md)
SELECT crear_movimiento_presupuestal(
    'COMPROMISO_CDP', -- tipo_movimiento
    1001, -- item_documento_id
    30000000.00, -- valor_movimiento
    456, -- usuario_id
    'Compromiso parcial por RP-2025-001' -- observaciones
);
```

### Ejemplo 3: Consultar ítem completo

```sql
-- Obtener información completa de un ítem
SELECT 
    item_id,
    numero_documento,
    tipo_documento_nombre,
    numero_item,
    descripcion_item,
    codigos_asociados,
    valores_columnas
FROM vista_items_completos
WHERE item_id = 1001;

-- Resultado esperado:
-- {
--   "item_id": 1001,
--   "numero_documento": "CDP-2025-001",
--   "tipo_documento_nombre": "Certificado de Disponibilidad Presupuestal",
--   "numero_item": 1,
--   "descripcion_item": "Contratación servicios profesionales",
--   "codigos_asociados": [
--     {"tipo_codigo": "RUBRO", "codigo": "2104", "codigo_nombre": "Servicios Profesionales"},
--     {"tipo_codigo": "FUENTE", "codigo": "01", "codigo_nombre": "Recursos Propios"}
--   ],
--   "valores_columnas": {
--     "valor_inicial": 50000000,
--     "valor_comprometido": 30000000,
--     "valor_liberado": 0,
--     "saldo_comprometer": 20000000
--   }
-- }
```

### Ejemplo 4: Consultar resumen por rubro

```sql
-- Obtener resumen consolidado por rubro presupuestal
SELECT 
    codigo_rubro,
    nombre_rubro,
    total_items,
    total_documentos,
    total_presupuesto_definitivo,
    total_saldo_disponible,
    ROUND(
        CASE 
            WHEN total_presupuesto_definitivo > 0 
            THEN ((total_presupuesto_definitivo - total_saldo_disponible) / total_presupuesto_definitivo) * 100 
            ELSE 0 
        END, 2
    ) as porcentaje_ejecucion
FROM vista_items_por_rubro
WHERE total_presupuesto_definitivo > 0
ORDER BY porcentaje_ejecucion DESC;
```

---

## Próximos Pasos

Este documento estableció la estructura completa de los ítems de documentos presupuestales con un enfoque moderno basado en movimientos. El siguiente documento cubrirá:

**03 - Estados de Documentos**: Sistema de estados, transiciones automáticas y manuales, con evaluación mediante JavaScript para condiciones complejas.

Los ítems están ahora completamente definidos con:
- ✅ Estructura normalizada de almacenamiento
- ✅ Columnas configurables por tipo de documento (únicamente monetarias)
- ✅ Valores controlados exclusivamente por movimientos presupuestales
- ✅ Campos consolidados (`valor_inicial` y `saldo_actual`) para consultas rápidas
- ✅ Asociación flexible con códigos del módulo de configuración
- ✅ Funciones adaptadas al sistema de movimientos
- ✅ Vistas para consultas eficientes
- ✅ Sistema de auditoría completo

---

## Transición al Sistema de Movimientos

### Principios Fundamentales

Este documento establece la estructura básica de los ítems de documentos presupuestales, pero es importante entender que:

1. **Los valores nunca se editan directamente**: Todas las columnas de valores son de solo lectura para los usuarios finales.

2. **Gestión exclusiva por movimientos**: Los valores se establecen y modifican únicamente a través del sistema de movimientos presupuestales.

3. **Valores consolidados**: Los campos `valor_inicial` y `saldo_actual` en el ítem principal proporcionan un resumen rápido, pero la fuente de verdad son los movimientos.

4. **Trazabilidad completa**: Cada cambio de valor queda registrado como un movimiento individual con metadatos completos.

### Próximos Pasos

En los siguientes documentos de esta serie se detallará:

- **03-ESTADOS_DOCUMENTOS.md**: Sistema de estados y transiciones de documentos
- **04-MOVIMIENTOS_PRESUPUESTALES.md**: Sistema completo de movimientos que gestiona todos los cambios de valores
- **05-VALIDACIONES_Y_CONTROLES.md**: Reglas de negocio y validaciones automáticas

---
