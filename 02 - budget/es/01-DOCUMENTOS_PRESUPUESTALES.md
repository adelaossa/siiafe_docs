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

### Ejemplo 1: Configurar nuevo tipo de documento

```sql
-- 1. Crear tipo de documento "Solicitud de CDP"
INSERT INTO tipo_documento_presupuestal VALUES
(8, 'SCDP', 'Solicitud de CDP', 'Solicitud previa para crear un CDP', 
 false, false, false, true, 'SCDP', true, '{prefijo}-{numero}', '#00BCD4', 'request_page', 8, true, NOW(), NOW(), 1);

-- 2. Configurar códigos permitidos
INSERT INTO tipo_documento_codigo_permitido VALUES
(15, 8, 1, true, false, 1, 'Rubro Solicitado', '^[0-9]{4}$', 4, 4, true, NOW()),
(16, 8, 2, true, false, 2, 'Fuente de Financiación', '^[0-9]{2}$', 2, 2, true, NOW()),
(17, 8, 10, true, false, 3, 'Solicitante', '^[0-9]{8,11}$', 8, 11, true, NOW());

-- 3. Configurar precedencia (requiere PG)
INSERT INTO tipo_documento_precedente VALUES
(6, 8, 1, true, 'APROBADO,MODIFICADO', true, 100.00, true, true, true, 
 'Debe existir presupuesto aprobado para el rubro solicitado', 1, true, NOW());
```

### Ejemplo 2: Validar precedencia antes de crear documento

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

---

## Próximos Pasos

Este documento estableció los fundamentos de los documentos presupuestales. Los siguientes documentos del módulo cubrirán:

1. **02 - Ítems de Documentos Presupuestales**: Estructura detallada de los ítems y sus columnas específicas
2. **03 - Estados de Documentos**: Sistema de estados y transiciones
3. **04 - Movimientos Presupuestales**: Operaciones que modifican los documentos
4. **05 - Validaciones y Controles**: Reglas de negocio y validaciones avanzadas

Cada documento se construirá sobre los conceptos establecidos aquí, creando un sistema coherente y bien estructurado para la gestión presupuestaria del sector público colombiano.
