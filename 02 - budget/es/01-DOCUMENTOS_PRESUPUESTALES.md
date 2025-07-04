# 01 - Documentos Presupuestales

## Descripción General

En el sistema SIIAFE, los **Documentos Presupuestales** son entidades fundamentales que representan las diferentes operaciones y estados dentro del ciclo presupuestario del sector público colombiano. Cada documento tiene una estructura configurable que permite adaptarse a las necesidades específicas de diferentes tipos de operaciones presupuestales.

## Conceptos Fundamentales

### ¿Qué es un Documento Presupuestal?

Un documento presupuestal es una entidad administrativa y contable que:

- **Registra operaciones**: Cada documento representa una operación específica del ciclo presupuestario
- **Tiene estructura definida**: Cada tipo de documento tiene características y campos específicos
- **Maneja ítems**: Cada documento puede contener múltiples ítems con información detallada
- **Sigue flujos**: Los documentos tienen relaciones de precedencia y dependencia
- **Tiene estados**: Cada documento pasa por diferentes estados durante su ciclo de vida

### Tipos de Documentos Presupuestales

El sistema permite definir diferentes **Tipos de Documentos Presupuestales**, cada uno con características específicas según su propósito en el ciclo presupuestario.

### Asociación con Ámbitos (Realms)

Los **Tipos de Documentos Presupuestales** deben asociarse obligatoriamente con **Ámbitos** del módulo de configuración mediante una relación **muchos-a-muchos**. Esta asociación determina en qué contextos organizacionales puede utilizarse cada tipo de documento.

**Características de la asociación:**
- **Relación muchos-a-muchos**: Un tipo de documento puede estar disponible en múltiples ámbitos
- **Configuración obligatoria**: Todo tipo de documento debe estar asociado al menos a un ámbito
- **Control de acceso**: Solo los usuarios con acceso al ámbito pueden utilizar el tipo de documento
- **Numeración por ámbito**: Cada ámbito puede tener configuraciones específicas de numeración

**Asignación obligatoria a ámbito:**
- **Todo documento individual** debe estar asignado a un **ámbito específico** (reino)
- **Es obligatoria**: No puede existir un documento sin ámbito asignado
- **Es única**: Cada documento pertenece a un solo ámbito
- **Determina visibilidad**: El documento solo es visible para usuarios del ámbito correspondiente
- **Afecta numeración**: La numeración es independiente por ámbito

---

## Estructura de Datos

### Tabla Principal: tipo_documento_presupuestal

```sql
CREATE TABLE tipo_documento_presupuestal (
    id SERIAL PRIMARY KEY,
    codigo VARCHAR(10) NOT NULL UNIQUE,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT,
    
    -- Configuración de comportamiento
    requiere_aprobacion BOOLEAN DEFAULT FALSE,
    genera_afectacion_contable BOOLEAN DEFAULT FALSE,
    es_documento_inicial BOOLEAN DEFAULT FALSE,
    permite_anulacion BOOLEAN DEFAULT TRUE,
    
    -- Configuración de numeración
    prefijo_numeracion VARCHAR(10),
    usa_numeracion_automatica BOOLEAN DEFAULT TRUE,
    formato_numero VARCHAR(50) DEFAULT '{prefijo}-{numero}',
    
    -- Configuración de interfaz
    color_interfaz VARCHAR(20) DEFAULT '#2196F3',
    icono_interfaz VARCHAR(50),
    orden_presentacion INTEGER DEFAULT 1,
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    creado_por INTEGER REFERENCES usuario(id)
);
```

### Ejemplos de Tipos de Documentos

```sql
-- Insertar tipos de documentos presupuestales típicos
INSERT INTO tipo_documento_presupuestal VALUES
(1, 'PG', 'Presupuesto de Gasto', 'Documento principal que establece las apropiaciones presupuestales para una vigencia', 
 true, true, true, false, 'PG', true, '{prefijo}-{vigencia}-{numero}', '#4CAF50', 'receipt', 1, true, NOW(), NOW(), 1),

(2, 'AD', 'Adición Presupuestal', 'Documento que incrementa el presupuesto total por recursos adicionales', 
 true, true, false, true, 'AD', true, '{prefijo}-{numero}', '#FF9800', 'add_circle', 2, true, NOW(), NOW(), 1),

(3, 'RC', 'Recorte Presupuestal', 'Documento que reduce el presupuesto por menor disponibilidad de recursos', 
 true, true, false, true, 'RC', true, '{prefijo}-{numero}', '#F44336', 'remove_circle', 3, true, NOW(), NOW(), 1),

(4, 'TR', 'Traslado Presupuestal', 'Documento que transfiere recursos entre diferentes rubros presupuestales', 
 true, true, false, true, 'TR', true, '{prefijo}-{numero}', '#9C27B0', 'swap_horiz', 4, true, NOW(), NOW(), 1),

(5, 'CDP', 'Certificado de Disponibilidad Presupuestal', 'Documento que certifica la disponibilidad de recursos para una operación', 
 false, false, false, true, 'CDP', true, '{prefijo}-{numero}', '#2196F3', 'verified', 5, true, NOW(), NOW(), 1),

(6, 'RP', 'Registro Presupuestal', 'Documento que formaliza el compromiso de recursos presupuestales', 
 false, true, false, true, 'RP', true, '{prefijo}-{numero}', '#3F51B5', 'assignment', 6, true, NOW(), NOW(), 1),

(7, 'OP', 'Orden de Pago', 'Documento que autoriza el pago de una obligación', 
 false, true, false, true, 'OP', true, '{prefijo}-{numero}', '#607D8B', 'payment', 7, true, NOW(), NOW(), 1);
```

---

## Asociación con Tipos de Códigos

Los documentos presupuestales pueden asociarse con diferentes **Tipos de Códigos** del módulo de configuración. Esta asociación determina qué clasificaciones presupuestales pueden utilizarse en los ítems de cada tipo de documento.

### Tabla: tipo_documento_codigo_permitido

```sql
CREATE TABLE tipo_documento_codigo_permitido (
    id SERIAL PRIMARY KEY,
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id),
    tipo_codigo_id INTEGER NOT NULL REFERENCES tipo_codigo(id), -- Del módulo configuration
    
    -- Configuración de uso
    es_obligatorio BOOLEAN DEFAULT FALSE,
    es_multiple BOOLEAN DEFAULT FALSE, -- Si puede tener múltiples valores
    orden_presentacion INTEGER DEFAULT 1,
    etiqueta_campo VARCHAR(100), -- Nombre personalizado para mostrar
    
    -- Validaciones
    patron_validacion VARCHAR(200), -- Regex para validar formato
    longitud_minima INTEGER,
    longitud_maxima INTEGER,
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Configuración de Códigos por Tipo de Documento

```sql
-- Configuración para Presupuesto de Gasto (PG)
INSERT INTO tipo_documento_codigo_permitido VALUES
(1, 1, 1, true, false, 1, 'Rubro Presupuestal', '^[0-9]{4}$', 4, 4, true, NOW()), -- Rubro
(2, 1, 2, true, false, 2, 'Fuente de Financiación', '^[0-9]{2}$', 2, 2, true, NOW()), -- Fuente
(3, 1, 3, false, false, 3, 'Situación de Fondos', '^[0-9]{2}$', 2, 2, true, NOW()), -- Situación

-- Configuración para CDP
INSERT INTO tipo_documento_codigo_permitido VALUES
(4, 5, 1, true, false, 1, 'Rubro Presupuestal', '^[0-9]{4}$', 4, 4, true, NOW()), -- Rubro
(5, 5, 2, true, false, 2, 'Fuente de Financiación', '^[0-9]{2}$', 2, 2, true, NOW()), -- Fuente
(6, 5, 4, false, false, 3, 'Centro de Costo', '^CC[0-9]{4}$', 6, 6, true, NOW()), -- Centro de Costo
(7, 5, 5, false, true, 4, 'Proyecto', '^PRY[0-9]{6}$', 9, 9, true, NOW()); -- Proyecto (múltiple)

-- Configuración para RP
INSERT INTO tipo_documento_codigo_permitido VALUES
(8, 6, 1, true, false, 1, 'Rubro Presupuestal', '^[0-9]{4}$', 4, 4, true, NOW()), -- Rubro
(9, 6, 2, true, false, 2, 'Fuente de Financiación', '^[0-9]{2}$', 2, 2, true, NOW()), -- Fuente
(10, 6, 6, true, false, 3, 'Tercero', '^[0-9]{8,11}$', 8, 11, true, NOW()), -- Tercero/Proveedor
(11, 6, 7, false, false, 4, 'Concepto de Gasto', '^CG[0-9]{4}$', 6, 6, true, NOW()); -- Concepto

-- Configuración para OP
INSERT INTO tipo_documento_codigo_permitido VALUES
(12, 7, 6, true, false, 1, 'Beneficiario', '^[0-9]{8,11}$', 8, 11, true, NOW()), -- Tercero/Beneficiario
(13, 7, 8, true, false, 2, 'Cuenta Bancaria', '^[0-9]{10,20}$', 10, 20, true, NOW()), -- Cuenta bancaria
(14, 7, 9, false, false, 3, 'Concepto de Pago', '^CP[0-9]{4}$', 6, 6, true, NOW()); -- Concepto de pago
```

---

## Asociación con Ámbitos (Realms)

Los **Tipos de Documentos Presupuestales** se asocian con **Ámbitos** del módulo de configuración mediante una relación muchos-a-muchos, y cada documento individual debe estar asignado obligatoriamente a un ámbito específico.

### Tabla: tipo_documento_ambito

```sql
CREATE TABLE tipo_documento_ambito (
    id SERIAL PRIMARY KEY,
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id),
    ambito_id INTEGER NOT NULL REFERENCES ambito(id), -- Del módulo configuration
    
    -- Configuración específica por ámbito
    prefijo_numeracion_ambito VARCHAR(10), -- Prefijo específico para este ámbito
    numeracion_independiente BOOLEAN DEFAULT TRUE, -- Si la numeración es independiente por ámbito
    formato_numero_ambito VARCHAR(50), -- Formato específico para este ámbito
    
    -- Configuración de permisos
    permite_creacion BOOLEAN DEFAULT TRUE,
    permite_modificacion BOOLEAN DEFAULT TRUE,
    permite_anulacion BOOLEAN DEFAULT TRUE,
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    creado_por INTEGER REFERENCES usuario(id),
    
    -- Índice único para evitar duplicados
    UNIQUE(tipo_documento_id, ambito_id)
);
```

### Tabla Principal de Documento (con ámbito obligatorio)

```sql
CREATE TABLE documento_presupuestal (
    id SERIAL PRIMARY KEY,
    numero_documento VARCHAR(50) NOT NULL,
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id),
    ambito_id INTEGER NOT NULL REFERENCES ambito(id), -- OBLIGATORIO: Todo documento debe tener ámbito
    
    -- Información básica del documento
    fecha_documento DATE NOT NULL DEFAULT CURRENT_DATE,
    descripcion TEXT,
    observaciones TEXT,
    
    -- Relación con documento precedente
    documento_precedente_id INTEGER REFERENCES documento_presupuestal(id),
    
    -- Estado actual
    estado_actual_id INTEGER NOT NULL REFERENCES estado_documento(id),
    
    -- Valores totales (calculados automáticamente desde ítems)
    valor_total DECIMAL(18,2) DEFAULT 0.00,
    valor_ejecutado DECIMAL(18,2) DEFAULT 0.00,
    valor_disponible DECIMAL(18,2) DEFAULT 0.00,
    
    -- Información de vigencia
    vigencia INTEGER NOT NULL DEFAULT EXTRACT(YEAR FROM CURRENT_DATE),
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER NOT NULL REFERENCES usuario(id),
    usuario_actualizacion INTEGER REFERENCES usuario(id),
    
    -- Índices
    UNIQUE(numero_documento, ambito_id, vigencia), -- Numeración única por ámbito y vigencia
    
    -- Validar que el tipo de documento esté permitido en el ámbito
    CONSTRAINT fk_tipo_documento_ambito_valido 
        FOREIGN KEY (tipo_documento_id, ambito_id) 
        REFERENCES tipo_documento_ambito(tipo_documento_id, ambito_id)
);
```

### Configuración de Tipos de Documento por Ámbito

```sql
-- Ejemplo: Configurar tipos de documentos para diferentes ámbitos

-- Ámbito "Presupuesto Nacional 2025" (ID: 1)
INSERT INTO tipo_documento_ambito VALUES
(1, 1, 1, 'PGN', true, '{prefijo_ambito}-{vigencia}-{numero}', true, true, true, true, NOW(), 1), -- PG Nacional
(2, 2, 1, 'ADN', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1), -- AD Nacional
(3, 5, 1, 'CDPN', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1), -- CDP Nacional
(4, 6, 1, 'RPN', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1), -- RP Nacional
(5, 7, 1, 'OPN', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1); -- OP Nacional

-- Ámbito "Territorios Étnicos 2025" (ID: 2)
INSERT INTO tipo_documento_ambito VALUES
(6, 5, 2, 'CDPTE', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1), -- CDP Territorial Étnico
(7, 6, 2, 'RPTE', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1), -- RP Territorial Étnico
(8, 7, 2, 'OPTE', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1); -- OP Territorial Étnico

-- Ámbito "Entidades Descentralizadas 2025" (ID: 3)
INSERT INTO tipo_documento_ambito VALUES
(9, 5, 3, 'CDPED', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1), -- CDP Entidad Descentralizada
(10, 6, 3, 'RPED', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1), -- RP Entidad Descentralizada
(11, 7, 3, 'OPED', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1); -- OP Entidad Descentralizada
```

---

## Documentos Precedentes

El sistema permite establecer **relaciones de precedencia** entre tipos de documentos, definiendo qué documentos deben existir y estar en estados específicos antes de poder crear un nuevo documento.

### Tabla: tipo_documento_precedente

```sql
CREATE TABLE tipo_documento_precedente (
    id SERIAL PRIMARY KEY,
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id),
    tipo_documento_precedente_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id),
    
    -- Configuración de precedencia
    es_obligatorio BOOLEAN DEFAULT TRUE,
    estados_permitidos VARCHAR(200), -- Estados válidos del documento precedente
    permite_consumo_parcial BOOLEAN DEFAULT TRUE,
    porcentaje_consumo_maximo DECIMAL(5,2) DEFAULT 100.00,
    
    -- Configuración de validaciones
    valida_saldos BOOLEAN DEFAULT TRUE,
    valida_fechas BOOLEAN DEFAULT TRUE,
    valida_vigencia BOOLEAN DEFAULT TRUE,
    
    -- Configuración de interfaz
    mensaje_validacion TEXT,
    orden_validacion INTEGER DEFAULT 1,
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Configuración de Precedencias

```sql
-- CDP requiere PG aprobado
INSERT INTO tipo_documento_precedente VALUES
(1, 5, 1, true, 'APROBADO,MODIFICADO', true, 100.00, true, true, true, 
 'Debe existir un Presupuesto de Gasto aprobado con saldo disponible', 1, true, NOW());

-- RP requiere CDP expedido
INSERT INTO tipo_documento_precedente VALUES
(2, 6, 5, true, 'EXPEDIDO,COMPROMETIDO_PARCIAL', true, 100.00, true, true, true, 
 'Debe existir un CDP expedido con saldo disponible para comprometer', 1, true, NOW());

-- OP requiere RP comprometido
INSERT INTO tipo_documento_precedente VALUES
(3, 7, 6, true, 'EXPEDIDO,OBLIGADO_PARCIAL', true, 100.00, true, true, true, 
 'Debe existir un RP con saldo disponible para obligar', 1, true, NOW());

-- Adición requiere PG base
INSERT INTO tipo_documento_precedente VALUES
(4, 2, 1, true, 'APROBADO,MODIFICADO', false, 0.00, false, true, true, 
 'Debe existir un Presupuesto de Gasto base para aplicar la adición', 1, true, NOW());

-- Traslado requiere PG con saldos
INSERT INTO tipo_documento_precedente VALUES
(5, 4, 1, true, 'APROBADO,MODIFICADO', true, 100.00, true, true, true, 
 'El rubro origen debe tener saldo disponible para el traslado', 1, true, NOW());
```

---

## Validaciones de Precedencia

### Función: validar_documento_precedente

```sql
CREATE OR REPLACE FUNCTION validar_documento_precedente(
    p_tipo_documento_id INTEGER,
    p_documento_precedente_id INTEGER,
    p_valor_a_consumir DECIMAL(18,2) DEFAULT 0
)
RETURNS JSON AS $$
DECLARE
    v_precedencia RECORD;
    v_documento_precedente RECORD;
    v_saldo_disponible DECIMAL(18,2);
    v_es_valido BOOLEAN := TRUE;
    v_mensajes TEXT[] := ARRAY[]::TEXT[];
BEGIN
    -- Obtener configuración de precedencia
    SELECT * INTO v_precedencia
    FROM tipo_documento_precedente
    WHERE tipo_documento_id = p_tipo_documento_id
      AND es_activo = true;
    
    IF NOT FOUND THEN
        RETURN json_build_object(
            'es_valido', true,
            'mensaje', 'No requiere documento precedente'
        );
    END IF;
    
    -- Obtener información del documento precedente
    SELECT d.*, tdp.codigo as tipo_codigo
    INTO v_documento_precedente
    FROM documento_presupuestal d
    JOIN tipo_documento_presupuestal tdp ON d.tipo_documento_id = tdp.id
    WHERE d.id = p_documento_precedente_id;
    
    IF NOT FOUND THEN
        RETURN json_build_object(
            'es_valido', false,
            'mensaje', 'Documento precedente no encontrado'
        );
    END IF;
    
    -- Validar tipo de documento correcto
    IF v_documento_precedente.tipo_documento_id != v_precedencia.tipo_documento_precedente_id THEN
        v_es_valido := FALSE;
        v_mensajes := array_append(v_mensajes, 'Tipo de documento precedente incorrecto');
    END IF;
    
    -- Validar estado permitido
    IF v_precedencia.estados_permitidos IS NOT NULL THEN
        IF NOT (v_documento_precedente.estado = ANY(string_to_array(v_precedencia.estados_permitidos, ','))) THEN
            v_es_valido := FALSE;
            v_mensajes := array_append(v_mensajes, 
                'Estado del documento precedente no válido. Estados permitidos: ' || v_precedencia.estados_permitidos);
        END IF;
    END IF;
    
    -- Validar saldos si es requerido
    IF v_precedencia.valida_saldos AND p_valor_a_consumir > 0 THEN
        -- Obtener saldo disponible del documento precedente
        v_saldo_disponible := obtener_saldo_disponible_documento(p_documento_precedente_id);
        
        IF v_saldo_disponible < p_valor_a_consumir THEN
            v_es_valido := FALSE;
            v_mensajes := array_append(v_mensajes, 
                'Saldo insuficiente. Disponible: $' || v_saldo_disponible::TEXT || 
                ', Solicitado: $' || p_valor_a_consumir::TEXT);
        END IF;
    END IF;
    
    -- Validar fechas si es requerido
    IF v_precedencia.valida_fechas THEN
        IF v_documento_precedente.fecha_documento > CURRENT_DATE THEN
            v_es_valido := FALSE;
            v_mensajes := array_append(v_mensajes, 'Fecha del documento precedente es futura');
        END IF;
    END IF;
    
    -- Validar vigencia si es requerido
    IF v_precedencia.valida_vigencia THEN
        IF EXTRACT(YEAR FROM v_documento_precedente.fecha_documento) != EXTRACT(YEAR FROM CURRENT_DATE) THEN
            v_es_valido := FALSE;
            v_mensajes := array_append(v_mensajes, 'Documento precedente de vigencia diferente');
        END IF;
    END IF;
    
    RETURN json_build_object(
        'es_valido', v_es_valido,
        'mensajes', array_to_json(v_mensajes),
        'saldo_disponible', v_saldo_disponible,
        'documento_precedente', row_to_json(v_documento_precedente),
        'configuracion_precedencia', row_to_json(v_precedencia)
    );
END;
$$ LANGUAGE plpgsql;
```

---

## Vistas y Consultas Útiles

### Vista: resumen_tipos_documentos

```sql
CREATE VIEW resumen_tipos_documentos AS
SELECT 
    tdp.id,
    tdp.codigo,
    tdp.nombre,
    tdp.descripcion,
    tdp.prefijo_numeracion,
    tdp.color_interfaz,
    tdp.icono_interfaz,
    
    -- Contar ámbitos asociados
    (SELECT COUNT(*) FROM tipo_documento_ambito 
     WHERE tipo_documento_id = tdp.id AND es_activo = true) as total_ambitos_asociados,
     
    -- Listar nombres de ámbitos
    (SELECT STRING_AGG(a.nombre, '; ') 
     FROM tipo_documento_ambito tda
     JOIN ambito a ON tda.ambito_id = a.id
     WHERE tda.tipo_documento_id = tdp.id AND tda.es_activo = true) as ambitos_disponibles,
    
    -- Contar códigos permitidos
    (SELECT COUNT(*) FROM tipo_documento_codigo_permitido 
     WHERE tipo_documento_id = tdp.id AND es_activo = true) as total_codigos_permitidos,
     
    -- Contar códigos obligatorios
    (SELECT COUNT(*) FROM tipo_documento_codigo_permitido 
     WHERE tipo_documento_id = tdp.id AND es_obligatorio = true AND es_activo = true) as codigos_obligatorios,
     
    -- Contar precedencias
    (SELECT COUNT(*) FROM tipo_documento_precedente 
     WHERE tipo_documento_id = tdp.id AND es_activo = true) as total_precedencias,
     
    -- Documentos creados este año
    (SELECT COUNT(*) FROM documento_presupuestal 
     WHERE tipo_documento_id = tdp.id 
       AND EXTRACT(YEAR FROM fecha_documento) = EXTRACT(YEAR FROM CURRENT_DATE)) as documentos_vigencia_actual
       
FROM tipo_documento_presupuestal tdp
WHERE tdp.es_activo = true
ORDER BY tdp.orden_presentacion;
```

### Vista: documentos_por_ambito

```sql
CREATE VIEW documentos_por_ambito AS
SELECT 
    d.id as documento_id,
    d.numero_documento,
    d.fecha_documento,
    d.vigencia,
    d.valor_total,
    d.valor_ejecutado,
    d.valor_disponible,
    
    -- Información del tipo de documento
    tdp.codigo as tipo_documento_codigo,
    tdp.nombre as tipo_documento_nombre,
    
    -- Información del ámbito
    a.id as ambito_id,
    a.nombre as ambito_nombre,
    a.descripcion as ambito_descripcion,
    a.año_fiscal as ambito_año_fiscal,
    a.tipo_ambito as ambito_tipo,
    
    -- Configuración del ámbito para este tipo de documento
    tda.prefijo_numeracion_ambito,
    tda.numeracion_independiente,
    
    -- Información del estado
    ed.codigo as estado_codigo,
    ed.nombre as estado_nombre,
    
    -- Información del documento precedente
    dp.numero_documento as documento_precedente_numero,
    tdp_prec.codigo as documento_precedente_tipo
    
FROM documento_presupuestal d
JOIN tipo_documento_presupuestal tdp ON d.tipo_documento_id = tdp.id
JOIN ambito a ON d.ambito_id = a.id
JOIN tipo_documento_ambito tda ON (tda.tipo_documento_id = d.tipo_documento_id AND tda.ambito_id = d.ambito_id)
JOIN estado_documento ed ON d.estado_actual_id = ed.id
LEFT JOIN documento_presupuestal dp ON d.documento_precedente_id = dp.id
LEFT JOIN tipo_documento_presupuestal tdp_prec ON dp.tipo_documento_id = tdp_prec.id
WHERE d.es_activo = true
ORDER BY d.fecha_creacion DESC;
```

### Consulta: Códigos permitidos por tipo de documento

```sql
-- Consultar configuración completa de un tipo de documento
SELECT 
    tdp.codigo as tipo_documento,
    tdp.nombre as nombre_documento,
    tc.codigo as codigo_permitido,
    tc.nombre as nombre_codigo,
    tdcp.es_obligatorio,
    tdcp.es_multiple,
    tdcp.etiqueta_campo,
    tdcp.patron_validacion,
    tdcp.orden_presentacion
FROM tipo_documento_presupuestal tdp
JOIN tipo_documento_codigo_permitido tdcp ON tdp.id = tdcp.tipo_documento_id
JOIN tipo_codigo tc ON tdcp.tipo_codigo_id = tc.id
WHERE tdp.codigo = 'CDP' -- Ejemplo para CDP
  AND tdcp.es_activo = true
ORDER BY tdcp.orden_presentacion;
```

### Consulta: Configuración de tipos de documento por ámbito

```sql
-- Consultar qué tipos de documentos están disponibles en cada ámbito
SELECT 
    a.nombre as ambito_nombre,
    a.descripcion as ambito_descripcion,
    a.año_fiscal,
    a.tipo_ambito,
    tdp.codigo as tipo_documento_codigo,
    tdp.nombre as tipo_documento_nombre,
    tda.prefijo_numeracion_ambito,
    tda.numeracion_independiente,
    tda.formato_numero_ambito,
    tda.permite_creacion,
    tda.permite_modificacion,
    tda.permite_anulacion,
    
    -- Estadísticas de uso
    (SELECT COUNT(*) FROM documento_presupuestal d 
     WHERE d.tipo_documento_id = tdp.id 
       AND d.ambito_id = a.id 
       AND EXTRACT(YEAR FROM d.fecha_documento) = EXTRACT(YEAR FROM CURRENT_DATE)) as documentos_vigencia_actual
    
FROM ambito a
JOIN tipo_documento_ambito tda ON a.id = tda.ambito_id
JOIN tipo_documento_presupuestal tdp ON tda.tipo_documento_id = tdp.id
WHERE a.es_activo = true 
  AND tda.es_activo = true 
  AND tdp.es_activo = true
ORDER BY a.año_fiscal DESC, a.nombre, tdp.orden_presentacion;
```

### Consulta: Cadena de precedencias

```sql
-- Ver la cadena completa de precedencias desde un tipo de documento
WITH RECURSIVE precedencias_recursivas AS (
    -- Caso base: tipo de documento inicial
    SELECT 
        tdp.id,
        tdp.codigo,
        tdp.nombre,
        0 as nivel,
        ARRAY[tdp.id] as ruta
    FROM tipo_documento_presupuestal tdp
    WHERE tdp.es_documento_inicial = true
    
    UNION ALL
    
    -- Caso recursivo: documentos que dependen de otros
    SELECT 
        tdp.id,
        tdp.codigo,
        tdp.nombre,
        pr.nivel + 1,
        pr.ruta || tdp.id
    FROM tipo_documento_presupuestal tdp
    JOIN tipo_documento_precedente tp ON tdp.id = tp.tipo_documento_id
    JOIN precedencias_recursivas pr ON tp.tipo_documento_precedente_id = pr.id
    WHERE NOT (tdp.id = ANY(pr.ruta)) -- Evitar ciclos
)
SELECT 
    REPEAT('  ', nivel) || codigo as jerarquia_documentos,
    nombre,
    nivel
FROM precedencias_recursivas
ORDER BY nivel, codigo;
```

---

## Ejemplos Prácticos

### Ejemplo 1: Configurar nuevo tipo de documento con ámbitos

```sql
-- 1. Crear tipo de documento "Solicitud de CDP"
INSERT INTO tipo_documento_presupuestal VALUES
(8, 'SCDP', 'Solicitud de CDP', 'Solicitud previa para crear un CDP', 
 false, false, false, true, 'SCDP', true, '{prefijo}-{numero}', '#00BCD4', 'request_page', 8, true, NOW(), NOW(), 1);

-- 2. Asociar el tipo de documento con ámbitos específicos
INSERT INTO tipo_documento_ambito VALUES
(12, 8, 1, 'SCDPN', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1), -- Ámbito Nacional
(13, 8, 2, 'SCDPTE', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1), -- Ámbito Territorial Étnico
(14, 8, 3, 'SCDPED', true, '{prefijo_ambito}-{numero}', true, true, true, true, NOW(), 1); -- Ámbito Entidades Descentralizadas

-- 3. Configurar códigos permitidos
INSERT INTO tipo_documento_codigo_permitido VALUES
(15, 8, 1, true, false, 1, 'Rubro Solicitado', '^[0-9]{4}$', 4, 4, true, NOW()),
(16, 8, 2, true, false, 2, 'Fuente de Financiación', '^[0-9]{2}$', 2, 2, true, NOW()),
(17, 8, 10, true, false, 3, 'Solicitante', '^[0-9]{8,11}$', 8, 11, true, NOW());

-- 4. Configurar precedencia (requiere PG)
INSERT INTO tipo_documento_precedente VALUES
(6, 8, 1, true, 'APROBADO,MODIFICADO', true, 100.00, true, true, true, 
 'Debe existir presupuesto aprobado para el rubro solicitado', 1, true, NOW());
```

### Ejemplo 2: Crear documento asignado a ámbito específico

```sql
-- Crear un CDP en el ámbito "Presupuesto Nacional 2025"
INSERT INTO documento_presupuestal VALUES
(501, 'CDPN-2025-001', 5, 1, -- tipo_documento_id=5 (CDP), ambito_id=1 (Nacional)
 '2025-01-15', 'CDP para programa de capacitación docente', 'Urgente para inicio de vigencia',
 100, -- documento_precedente_id (PG)
 1, -- estado_actual_id (Borrador)
 50000000.00, 0.00, 50000000.00, -- valores
 2025, -- vigencia
 true, NOW(), NOW(), 123, 123); -- metadatos

-- Verificar que el tipo de documento esté permitido en el ámbito
-- La constraint fk_tipo_documento_ambito_valido validará automáticamente esta relación
```

### Ejemplo 3: Consultar documentos por ámbito

```sql
-- Ver todos los CDP del ámbito "Territorios Étnicos 2025"
SELECT 
    numero_documento,
    fecha_documento,
    valor_total,
    ambito_nombre,
    prefijo_numeracion_ambito,
    estado_nombre
FROM documentos_por_ambito
WHERE tipo_documento_codigo = 'CDP'
  AND ambito_nombre LIKE '%Territorios Étnicos%'
  AND vigencia = 2025
ORDER BY fecha_documento DESC;

-- Resultado ejemplo:
-- numero_documento | fecha_documento | valor_total  | ambito_nombre           | prefijo_numeracion_ambito | estado_nombre
-- CDPTE-001        | 2025-01-15      | 25000000.00  | Territorios Étnicos 2025| CDPTE                     | Expedido
-- CDPTE-002        | 2025-01-16      | 40000000.00  | Territorios Étnicos 2025| CDPTE                     | Borrador
```

### Ejemplo 4: Validar precedencia antes de crear documento

```sql
-- Validar si se puede crear un CDP basado en un PG específico
SELECT validar_documento_precedente(
    5, -- tipo_documento_id (CDP)
    100, -- documento_precedente_id (ID del PG)
    50000000.00 -- valor_a_consumir ($50,000,000)
);

-- Resultado ejemplo:
-- {
--   "es_valido": true,
--   "mensajes": [],
--   "saldo_disponible": 150000000.00,
--   "documento_precedente": {...},
--   "configuracion_precedencia": {...}
-- }
```

### Ejemplo 5: Verificar configuración de ámbitos para un tipo de documento

```sql
-- Ver en qué ámbitos está disponible el tipo de documento CDP
SELECT 
    ambito_nombre,
    año_fiscal,
    tipo_ambito,
    prefijo_numeracion_ambito,
    permite_creacion,
    documentos_vigencia_actual
FROM configuracion_tipos_documento_por_ambito
WHERE tipo_documento_codigo = 'CDP'
  AND año_fiscal = 2025
ORDER BY ambito_nombre;

-- Resultado ejemplo:
-- ambito_nombre                | año_fiscal | tipo_ambito   | prefijo_numeracion_ambito | permite_creacion | documentos_vigencia_actual
-- Entidades Descentralizadas  | 2025       | PRESUPUESTO   | CDPED                     | true             | 15
-- Presupuesto Nacional        | 2025       | PRESUPUESTO   | CDPN                      | true             | 45  
-- Territorios Étnicos         | 2025       | PRESUPUESTO   | CDPTE                     | true             | 8
```

---

## Próximos Pasos

Este documento estableció los fundamentos de los documentos presupuestales. Los siguientes documentos del módulo cubrirán:

1. **02 - Ítems de Documentos Presupuestales**: Estructura detallada de los ítems y sus columnas específicas
2. **03 - Estados de Documentos**: Sistema de estados y transiciones
3. **04 - Movimientos Presupuestales**: Operaciones que modifican los documentos
4. **05 - Validaciones y Controles**: Reglas de negocio y validaciones avanzadas

Cada documento se construirá sobre los conceptos establecidos aquí, creando un sistema coherente y bien estructurado para la gestión presupuestaria del sector público colombiano.
