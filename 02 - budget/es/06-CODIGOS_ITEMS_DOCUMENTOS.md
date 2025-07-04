# 06 - Códigos de Ítems de Documentos Presupuestales

## Descripción General

En el sistema SIIAFE, cada **ítem de documento presupuestal** debe estar clasificado mediante diferentes **códigos** que permiten la organización, seguimiento y control de los recursos presupuestales. Los códigos son elementos de clasificación que se obtienen del módulo de configuración y se asignan a los ítems según reglas específicas definidas para cada tipo de documento.

## Conceptos Fundamentales

### ¿Qué son los Códigos de Ítems?

Los códigos de ítems son clasificaciones presupuestales que:

- **Clasifican el ítem**: Proporcionan contexto presupuestario, contable y administrativo
- **Facilitan el seguimiento**: Permiten agrupar y analizar información por diferentes criterios
- **Controlan el flujo**: Establecen restricciones y validaciones en las operaciones
- **Heredan información**: Pueden heredarse automáticamente desde documentos precedentes
- **Se configuran por tipo**: Cada tipo de documento define qué códigos requiere

### Tipos de Asignación de Códigos

El sistema permite diferentes estrategias para asignar códigos a los ítems:

1. **Obligatorio Manual**: El usuario debe seleccionar el código manualmente
2. **Opcional**: El código puede asignarse o dejarse vacío
3. **Heredado del Precedente**: Se copia automáticamente del documento precedente
4. **Heredado con Validación**: Se hereda pero permite modificación con validaciones

### Configuración por Tipo de Documento

Cada **tipo de documento** define qué códigos están disponibles y cómo deben comportarse para sus ítems mediante la tabla `tipo_documento_codigo_permitido`.

---

## Estructura de Datos

### Tabla de Configuración: tipo_documento_codigo_permitido

```sql
CREATE TABLE tipo_documento_codigo_permitido (
    id SERIAL PRIMARY KEY,
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id),
    tipo_codigo_id INTEGER NOT NULL REFERENCES tipo_codigo(id), -- Del módulo configuration
    
    -- Configuración de uso en ítems
    es_obligatorio BOOLEAN DEFAULT FALSE, -- Si es obligatorio asignar este código
    es_multiple BOOLEAN DEFAULT FALSE, -- Si puede tener múltiples valores
    hereda_de_precedente BOOLEAN DEFAULT FALSE, -- Si se hereda automáticamente del documento precedente
    permite_modificar_herencia BOOLEAN DEFAULT TRUE, -- Si se permite modificar el valor heredado
    
    -- Configuración de presentación
    orden_presentacion INTEGER DEFAULT 1,
    etiqueta_campo VARCHAR(100), -- Nombre personalizado para mostrar en la interfaz
    ayuda_usuario TEXT, -- Texto de ayuda para el usuario
    
    -- Validaciones
    patron_validacion VARCHAR(200), -- Regex para validar formato
    longitud_minima INTEGER,
    longitud_maxima INTEGER,
    valores_permitidos JSON, -- Lista de valores específicos permitidos
    
    -- Configuración de herencia
    tipo_codigo_precedente_id INTEGER REFERENCES tipo_codigo(id), -- Qué tipo de código del precedente usar para herencia
    mapeo_herencia JSON, -- Reglas para mapear códigos entre documentos diferentes
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    creado_por INTEGER REFERENCES usuario(id),
    
    -- Índice único
    UNIQUE(tipo_documento_id, tipo_codigo_id)
);
```

### Tabla de Asignación: item_documento_codigo

```sql
CREATE TABLE item_documento_codigo (
    id SERIAL PRIMARY KEY,
    item_documento_id INTEGER NOT NULL REFERENCES item_documento_presupuestal(id),
    tipo_codigo_id INTEGER NOT NULL REFERENCES tipo_codigo(id),
    codigo_id INTEGER NOT NULL REFERENCES codigo(id), -- Del módulo configuration
    
    -- Información de la asignación
    valor_codigo VARCHAR(50) NOT NULL, -- Valor del código asignado
    nombre_codigo VARCHAR(255) NOT NULL, -- Nombre del código para referencia rápida
    
    -- Control de herencia
    es_heredado BOOLEAN DEFAULT FALSE, -- Si este código fue heredado del precedente
    item_precedente_id INTEGER REFERENCES item_documento_presupuestal(id), -- Ítem del cual se heredó
    fue_modificado_usuario BOOLEAN DEFAULT FALSE, -- Si el usuario modificó el valor heredado
    
    -- Información adicional
    metadatos_codigo JSON, -- Información adicional del código
    observaciones TEXT, -- Observaciones específicas sobre esta asignación
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_asignacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_asignacion INTEGER NOT NULL REFERENCES usuario(id),
    fecha_modificacion TIMESTAMP,
    usuario_modificacion INTEGER REFERENCES usuario(id),
    
    -- Índices
    UNIQUE(item_documento_id, tipo_codigo_id), -- Un tipo de código por ítem
    INDEX idx_item_codigo_valor (valor_codigo),
    INDEX idx_item_codigo_heredado (es_heredado),
    INDEX idx_item_codigo_precedente (item_precedente_id)
);
```

### Tabla de Historial: item_codigo_historial

```sql
CREATE TABLE item_codigo_historial (
    id SERIAL PRIMARY KEY,
    item_documento_id INTEGER NOT NULL REFERENCES item_documento_presupuestal(id),
    tipo_codigo_id INTEGER NOT NULL REFERENCES tipo_codigo(id),
    
    -- Valores anteriores y nuevos
    codigo_anterior_id INTEGER REFERENCES codigo(id),
    valor_anterior VARCHAR(50),
    codigo_nuevo_id INTEGER REFERENCES codigo(id),
    valor_nuevo VARCHAR(50),
    
    -- Información del cambio
    tipo_cambio VARCHAR(50) NOT NULL, -- 'ASIGNACION_INICIAL', 'MODIFICACION_MANUAL', 'HERENCIA_AUTOMATICA', 'CORRECCION'
    motivo_cambio TEXT,
    es_herencia BOOLEAN DEFAULT FALSE,
    
    -- Metadatos
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_cambio INTEGER NOT NULL REFERENCES usuario(id),
    
    -- Índices
    INDEX idx_historial_item (item_documento_id),
    INDEX idx_historial_fecha (fecha_cambio),
    INDEX idx_historial_tipo_cambio (tipo_cambio)
);
```

---

## Configuraciones por Tipo de Documento

### Configuración para Certificado de Disponibilidad Presupuestal (CDP)

```sql
-- Configurar códigos para CDP
INSERT INTO tipo_documento_codigo_permitido VALUES

-- Rubro Presupuestal (obligatorio, manual)
(1, 5, 1, true, false, false, false, 1, 'Rubro Presupuestal', 
 'Seleccione el rubro presupuestal específico para esta operación',
 '^[0-9]{2}\.[0-9]{2}\.[0-9]{2}\.[0-9]{3}$', 11, 11, null, null, null,
 true, NOW(), 1),

-- Fuente de Financiación (obligatorio, manual)
(2, 5, 2, true, false, false, false, 2, 'Fuente de Financiación',
 'Indique la fuente de donde provienen los recursos',
 '^[0-9]{3}$', 3, 3, null, null, null,
 true, NOW(), 1),

-- Centro de Costo (opcional, manual)
(3, 5, 4, false, false, false, true, 3, 'Centro de Costo',
 'Centro de costo al cual se asigna el gasto (opcional)',
 '^CC[0-9]{4}$', 6, 6, null, null, null,
 true, NOW(), 1),

-- Proyecto (opcional, múltiple)
(4, 5, 5, false, true, false, true, 4, 'Proyectos Asociados',
 'Proyectos de inversión relacionados con este CDP',
 '^PRY[0-9]{6}$', 9, 9, null, null, null,
 true, NOW(), 1);
```

### Configuración para Registro Presupuestal (RP)

```sql
-- Configurar códigos para RP (hereda algunos del CDP precedente)
INSERT INTO tipo_documento_codigo_permitido VALUES

-- Rubro Presupuestal (obligatorio, heredado del CDP)
(5, 6, 1, true, false, true, false, 1, 'Rubro Presupuestal',
 'Rubro heredado automáticamente del CDP precedente',
 '^[0-9]{2}\.[0-9]{2}\.[0-9]{2}\.[0-9]{3}$', 11, 11, null, 1, null,
 true, NOW(), 1),

-- Fuente de Financiación (obligatorio, heredado del CDP)
(6, 6, 2, true, false, true, false, 2, 'Fuente de Financiación',
 'Fuente heredada del CDP precedente',
 '^[0-9]{3}$', 3, 3, null, 2, null,
 true, NOW(), 1),

-- Tercero/Proveedor (obligatorio, manual)
(7, 6, 6, true, false, false, false, 3, 'Tercero/Proveedor',
 'Identificación del tercero beneficiario del registro presupuestal',
 '^[0-9]{8,11}$', 8, 11, null, null, null,
 true, NOW(), 1),

-- Centro de Costo (opcional, heredado con modificación permitida)
(8, 6, 4, false, false, true, true, 4, 'Centro de Costo',
 'Centro de costo heredado del CDP, puede modificarse si es necesario',
 '^CC[0-9]{4}$', 6, 6, null, 4, null,
 true, NOW(), 1),

-- Concepto de Gasto (opcional, manual)
(9, 6, 7, false, false, false, true, 5, 'Concepto de Gasto',
 'Concepto específico del gasto a realizar',
 '^CG[0-9]{4}$', 6, 6, null, null, null,
 true, NOW(), 1);
```

### Configuración para Orden de Pago (OP)

```sql
-- Configurar códigos para OP (hereda algunos del RP precedente)
INSERT INTO tipo_documento_codigo_permitido VALUES

-- Tercero/Beneficiario (obligatorio, heredado del RP)
(10, 7, 6, true, false, true, true, 1, 'Beneficiario del Pago',
 'Tercero beneficiario heredado del RP, puede modificarse en casos específicos',
 '^[0-9]{8,11}$', 8, 11, null, 6, null,
 true, NOW(), 1),

-- Cuenta Bancaria (obligatorio, manual)
(11, 7, 8, true, false, false, false, 2, 'Cuenta Bancaria',
 'Cuenta bancaria donde se depositará el pago',
 '^[0-9]{10,20}$', 10, 20, null, null, null,
 true, NOW(), 1),

-- Concepto de Pago (opcional, manual)
(12, 7, 9, false, false, false, true, 3, 'Concepto de Pago',
 'Concepto específico para el pago',
 '^CP[0-9]{4}$', 6, 6, null, null, null,
 true, NOW(), 1),

-- Modalidad de Pago (obligatorio, manual)
(13, 7, 10, true, false, false, false, 4, 'Modalidad de Pago',
 'Forma de pago: transferencia, cheque, etc.',
 '^MP[0-9]{2}$', 4, 4, 
 '["MP01", "MP02", "MP03", "MP04"]', null, null,
 true, NOW(), 1);
```

---

## Funciones de Gestión de Códigos

### Función: asignar_codigo_item

```sql
CREATE OR REPLACE FUNCTION asignar_codigo_item(
    p_item_documento_id INTEGER,
    p_tipo_codigo_id INTEGER,
    p_codigo_id INTEGER,
    p_usuario_id INTEGER,
    p_observaciones TEXT DEFAULT NULL
)
RETURNS INTEGER AS $$
DECLARE
    v_configuracion RECORD;
    v_codigo RECORD;
    v_asignacion_id INTEGER;
    v_validacion_resultado BOOLEAN;
BEGIN
    -- Obtener configuración del tipo de código para este tipo de documento
    SELECT tdcp.*, tdp.codigo as tipo_documento_codigo
    INTO v_configuracion
    FROM tipo_documento_codigo_permitido tdcp
    JOIN tipo_documento_presupuestal tdp ON tdcp.tipo_documento_id = tdp.id
    JOIN item_documento_presupuestal idp ON idp.documento_id IN (
        SELECT d.id FROM documento_presupuestal d WHERE d.tipo_documento_id = tdp.id
    )
    WHERE tdcp.tipo_codigo_id = p_tipo_codigo_id
      AND idp.id = p_item_documento_id
      AND tdcp.es_activo = true;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Tipo de código % no está permitido para este tipo de documento', p_tipo_codigo_id;
    END IF;
    
    -- Obtener información del código
    SELECT * INTO v_codigo FROM codigo WHERE id = p_codigo_id;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Código % no encontrado', p_codigo_id;
    END IF;
    
    -- Validar el formato del código
    IF v_configuracion.patron_validacion IS NOT NULL THEN
        SELECT v_codigo.valor_codigo ~ v_configuracion.patron_validacion INTO v_validacion_resultado;
        
        IF NOT v_validacion_resultado THEN
            RAISE EXCEPTION 'El código % no cumple con el patrón requerido %', 
                v_codigo.valor_codigo, v_configuracion.patron_validacion;
        END IF;
    END IF;
    
    -- Verificar si ya existe una asignación para este tipo de código
    IF EXISTS (SELECT 1 FROM item_documento_codigo 
               WHERE item_documento_id = p_item_documento_id 
                 AND tipo_codigo_id = p_tipo_codigo_id 
                 AND es_activo = true) THEN
        
        -- Si no permite múltiples valores, actualizar la asignación existente
        IF NOT v_configuracion.es_multiple THEN
            UPDATE item_documento_codigo
            SET codigo_id = p_codigo_id,
                valor_codigo = v_codigo.valor_codigo,
                nombre_codigo = v_codigo.nombre,
                fue_modificado_usuario = true,
                observaciones = p_observaciones,
                fecha_modificacion = NOW(),
                usuario_modificacion = p_usuario_id
            WHERE item_documento_id = p_item_documento_id 
              AND tipo_codigo_id = p_tipo_codigo_id 
              AND es_activo = true
            RETURNING id INTO v_asignacion_id;
        ELSE
            RAISE EXCEPTION 'Ya existe una asignación para este tipo de código y no permite múltiples valores';
        END IF;
    ELSE
        -- Crear nueva asignación
        INSERT INTO item_documento_codigo 
            (item_documento_id, tipo_codigo_id, codigo_id, valor_codigo, nombre_codigo,
             es_heredado, observaciones, usuario_asignacion)
        VALUES 
            (p_item_documento_id, p_tipo_codigo_id, p_codigo_id, v_codigo.valor_codigo, v_codigo.nombre,
             false, p_observaciones, p_usuario_id)
        RETURNING id INTO v_asignacion_id;
    END IF;
    
    -- Registrar en historial
    INSERT INTO item_codigo_historial
        (item_documento_id, tipo_codigo_id, codigo_nuevo_id, valor_nuevo,
         tipo_cambio, motivo_cambio, usuario_cambio)
    VALUES
        (p_item_documento_id, p_tipo_codigo_id, p_codigo_id, v_codigo.valor_codigo,
         'ASIGNACION_MANUAL', p_observaciones, p_usuario_id);
    
    RETURN v_asignacion_id;
END;
$$ LANGUAGE plpgsql;
```

### Función: heredar_codigos_precedente

```sql
CREATE OR REPLACE FUNCTION heredar_codigos_precedente(
    p_item_documento_id INTEGER,
    p_item_precedente_id INTEGER,
    p_usuario_id INTEGER
)
RETURNS INTEGER AS $$
DECLARE
    v_config_herencia RECORD;
    v_codigo_precedente RECORD;
    v_total_heredados INTEGER := 0;
BEGIN
    -- Obtener configuraciones que requieren herencia para este tipo de documento
    FOR v_config_herencia IN
        SELECT tdcp.*, tdp.codigo as tipo_documento_codigo
        FROM tipo_documento_codigo_permitido tdcp
        JOIN tipo_documento_presupuestal tdp ON tdcp.tipo_documento_id = tdp.id
        JOIN item_documento_presupuestal idp ON idp.documento_id IN (
            SELECT d.id FROM documento_presupuestal d WHERE d.tipo_documento_id = tdp.id
        )
        WHERE idp.id = p_item_documento_id
          AND tdcp.hereda_de_precedente = true
          AND tdcp.es_activo = true
    LOOP
        -- Buscar el código correspondiente en el ítem precedente
        SELECT idc.*, c.valor_codigo, c.nombre
        INTO v_codigo_precedente
        FROM item_documento_codigo idc
        JOIN codigo c ON idc.codigo_id = c.id
        WHERE idc.item_documento_id = p_item_precedente_id
          AND idc.tipo_codigo_id = COALESCE(
              v_config_herencia.tipo_codigo_precedente_id, 
              v_config_herencia.tipo_codigo_id
          )
          AND idc.es_activo = true;
        
        -- Si se encuentra el código en el precedente, heredarlo
        IF FOUND THEN
            -- Verificar si ya existe asignación para este tipo de código
            IF NOT EXISTS (SELECT 1 FROM item_documento_codigo 
                          WHERE item_documento_id = p_item_documento_id 
                            AND tipo_codigo_id = v_config_herencia.tipo_codigo_id 
                            AND es_activo = true) THEN
                
                -- Crear la asignación heredada
                INSERT INTO item_documento_codigo 
                    (item_documento_id, tipo_codigo_id, codigo_id, valor_codigo, nombre_codigo,
                     es_heredado, item_precedente_id, usuario_asignacion)
                VALUES 
                    (p_item_documento_id, v_config_herencia.tipo_codigo_id, v_codigo_precedente.codigo_id,
                     v_codigo_precedente.valor_codigo, v_codigo_precedente.nombre,
                     true, p_item_precedente_id, p_usuario_id);
                
                -- Registrar en historial
                INSERT INTO item_codigo_historial
                    (item_documento_id, tipo_codigo_id, codigo_nuevo_id, valor_nuevo,
                     tipo_cambio, motivo_cambio, es_herencia, usuario_cambio)
                VALUES
                    (p_item_documento_id, v_config_herencia.tipo_codigo_id, v_codigo_precedente.codigo_id,
                     v_codigo_precedente.valor_codigo, 'HERENCIA_AUTOMATICA', 
                     'Heredado automáticamente del ítem precedente', true, p_usuario_id);
                
                v_total_heredados := v_total_heredados + 1;
            END IF;
        END IF;
    END LOOP;
    
    RETURN v_total_heredados;
END;
$$ LANGUAGE plpgsql;
```

### Función: validar_codigos_obligatorios

```sql
CREATE OR REPLACE FUNCTION validar_codigos_obligatorios(
    p_item_documento_id INTEGER
)
RETURNS TABLE(
    es_valido BOOLEAN,
    codigos_faltantes TEXT[],
    mensaje_validacion TEXT
) AS $$
DECLARE
    v_config_obligatorio RECORD;
    v_codigos_faltantes TEXT[] := ARRAY[]::TEXT[];
    v_mensaje TEXT := '';
BEGIN
    -- Verificar todos los códigos obligatorios para este tipo de documento
    FOR v_config_obligatorio IN
        SELECT tdcp.*, tc.nombre as nombre_tipo_codigo
        FROM tipo_documento_codigo_permitido tdcp
        JOIN tipo_codigo tc ON tdcp.tipo_codigo_id = tc.id
        JOIN tipo_documento_presupuestal tdp ON tdcp.tipo_documento_id = tdp.id
        JOIN item_documento_presupuestal idp ON idp.documento_id IN (
            SELECT d.id FROM documento_presupuestal d WHERE d.tipo_documento_id = tdp.id
        )
        WHERE idp.id = p_item_documento_id
          AND tdcp.es_obligatorio = true
          AND tdcp.es_activo = true
    LOOP
        -- Verificar si existe asignación para este tipo de código
        IF NOT EXISTS (SELECT 1 FROM item_documento_codigo 
                      WHERE item_documento_id = p_item_documento_id 
                        AND tipo_codigo_id = v_config_obligatorio.tipo_codigo_id 
                        AND es_activo = true) THEN
            
            v_codigos_faltantes := array_append(v_codigos_faltantes, v_config_obligatorio.nombre_tipo_codigo);
        END IF;
    END LOOP;
    
    -- Construir mensaje de validación
    IF array_length(v_codigos_faltantes, 1) > 0 THEN
        v_mensaje := 'Faltan códigos obligatorios: ' || array_to_string(v_codigos_faltantes, ', ');
        RETURN QUERY SELECT false, v_codigos_faltantes, v_mensaje;
    ELSE
        RETURN QUERY SELECT true, ARRAY[]::TEXT[], 'Todos los códigos obligatorios están asignados';
    END IF;
END;
$$ LANGUAGE plpgsql;
```

---

## Vistas para Consulta de Códigos

### Vista: item_codigos_completos

```sql
CREATE VIEW item_codigos_completos AS
SELECT 
    idp.id as item_documento_id,
    d.numero_documento,
    d.fecha_documento,
    tdp.codigo as tipo_documento_codigo,
    tdp.nombre as tipo_documento_nombre,
    
    -- Información del ítem
    idp.descripcion as item_descripcion,
    idp.valor_inicial as item_valor,
    
    -- Información del código asignado
    tc.id as tipo_codigo_id,
    tc.nombre as tipo_codigo_nombre,
    idc.valor_codigo,
    idc.nombre_codigo,
    
    -- Control de herencia
    idc.es_heredado,
    idc.fue_modificado_usuario,
    ip.id as item_precedente_id,
    dp.numero_documento as documento_precedente_numero,
    
    -- Configuración
    tdcp.es_obligatorio,
    tdcp.hereda_de_precedente,
    tdcp.permite_modificar_herencia,
    
    -- Metadatos
    idc.fecha_asignacion,
    u.nombre as usuario_asignacion
    
FROM item_documento_presupuestal idp
JOIN documento_presupuestal d ON idp.documento_id = d.id
JOIN tipo_documento_presupuestal tdp ON d.tipo_documento_id = tdp.id
LEFT JOIN item_documento_codigo idc ON idp.id = idc.item_documento_id AND idc.es_activo = true
LEFT JOIN tipo_codigo tc ON idc.tipo_codigo_id = tc.id
LEFT JOIN tipo_documento_codigo_permitido tdcp ON (tdcp.tipo_documento_id = tdp.id AND tdcp.tipo_codigo_id = tc.id)
LEFT JOIN item_documento_presupuestal ip ON idc.item_precedente_id = ip.id
LEFT JOIN documento_presupuestal dp ON ip.documento_id = dp.id
LEFT JOIN usuario u ON idc.usuario_asignacion = u.id
WHERE idp.es_activo = true
ORDER BY d.fecha_documento DESC, idp.id, tdcp.orden_presentacion;
```

### Vista: resumen_codigos_por_documento

```sql
CREATE VIEW resumen_codigos_por_documento AS
SELECT 
    d.id as documento_id,
    d.numero_documento,
    d.fecha_documento,
    tdp.codigo as tipo_documento_codigo,
    
    -- Estadísticas de códigos
    (SELECT COUNT(*) FROM item_documento_presupuestal idp
     JOIN item_documento_codigo idc ON idp.id = idc.item_documento_id
     WHERE idp.documento_id = d.id AND idp.es_activo = true AND idc.es_activo = true
    ) as total_codigos_asignados,
    
    (SELECT COUNT(*) FROM item_documento_presupuestal idp
     JOIN item_documento_codigo idc ON idp.id = idc.item_documento_id
     WHERE idp.documento_id = d.id AND idc.es_heredado = true AND idp.es_activo = true AND idc.es_activo = true
    ) as total_codigos_heredados,
    
    (SELECT COUNT(*) FROM item_documento_presupuestal idp
     JOIN item_documento_codigo idc ON idp.id = idc.item_documento_id
     WHERE idp.documento_id = d.id AND idc.fue_modificado_usuario = true AND idp.es_activo = true AND idc.es_activo = true
    ) as total_codigos_modificados,
    
    -- Validación de completitud
    (SELECT COUNT(*) FROM tipo_documento_codigo_permitido tdcp
     WHERE tdcp.tipo_documento_id = tdp.id AND tdcp.es_obligatorio = true AND tdcp.es_activo = true
    ) as total_codigos_obligatorios,
    
    -- Lista de códigos faltantes
    (SELECT STRING_AGG(tc.nombre, '; ')
     FROM tipo_documento_codigo_permitido tdcp
     JOIN tipo_codigo tc ON tdcp.tipo_codigo_id = tc.id
     WHERE tdcp.tipo_documento_id = tdp.id 
       AND tdcp.es_obligatorio = true 
       AND tdcp.es_activo = true
       AND NOT EXISTS (
           SELECT 1 FROM item_documento_presupuestal idp
           JOIN item_documento_codigo idc ON idp.id = idc.item_documento_id
           WHERE idp.documento_id = d.id 
             AND idc.tipo_codigo_id = tdcp.tipo_codigo_id 
             AND idp.es_activo = true 
             AND idc.es_activo = true
       )
    ) as codigos_obligatorios_faltantes
    
FROM documento_presupuestal d
JOIN tipo_documento_presupuestal tdp ON d.tipo_documento_id = tdp.id
WHERE d.es_activo = true;
```

---

## Triggers para Automatización

### Trigger: heredar_codigos_automaticamente

```sql
CREATE OR REPLACE FUNCTION trigger_heredar_codigos_automaticamente()
RETURNS TRIGGER AS $$
DECLARE
    v_item_precedente_id INTEGER;
    v_codigos_heredados INTEGER;
BEGIN
    -- Solo actuar en inserción de nuevos ítems
    IF TG_OP = 'INSERT' THEN
        -- Buscar ítem correspondiente en el documento precedente
        SELECT idp_prec.id INTO v_item_precedente_id
        FROM item_documento_presupuestal idp_prec
        JOIN documento_presupuestal d_prec ON idp_prec.documento_id = d_prec.id
        JOIN documento_presupuestal d_actual ON d_prec.id = d_actual.documento_precedente_id
        WHERE d_actual.id = NEW.documento_id
          AND idp_prec.numero_item = NEW.numero_item -- Asumiendo que se mantiene el número de ítem
          AND idp_prec.es_activo = true
        LIMIT 1;
        
        -- Si se encuentra ítem precedente, heredar códigos automáticamente
        IF v_item_precedente_id IS NOT NULL THEN
            SELECT heredar_codigos_precedente(NEW.id, v_item_precedente_id, NEW.usuario_creacion)
            INTO v_codigos_heredados;
            
            -- Registrar en logs si se requiere
            RAISE NOTICE 'Se heredaron % códigos automáticamente para el ítem %', v_codigos_heredados, NEW.id;
        END IF;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Crear el trigger
CREATE TRIGGER trigger_heredar_codigos_automaticamente
    AFTER INSERT ON item_documento_presupuestal
    FOR EACH ROW
    EXECUTE FUNCTION trigger_heredar_codigos_automaticamente();
```

### Trigger: validar_codigos_obligatorios_antes_aprobacion

```sql
CREATE OR REPLACE FUNCTION trigger_validar_codigos_antes_aprobacion()
RETURNS TRIGGER AS $$
DECLARE
    v_estado_aprobacion INTEGER;
    v_validacion RECORD;
    v_items_invalidos INTEGER := 0;
BEGIN
    -- Obtener ID del estado de aprobación (asumiendo que existe)
    SELECT id INTO v_estado_aprobacion FROM estado_documento WHERE codigo = 'APROBADO' LIMIT 1;
    
    -- Solo validar si se está cambiando a estado aprobado
    IF NEW.estado_actual_id = v_estado_aprobacion AND OLD.estado_actual_id != v_estado_aprobacion THEN
        
        -- Validar que todos los ítems tengan códigos obligatorios
        FOR v_validacion IN
            SELECT idp.id, idp.descripcion, vco.*
            FROM item_documento_presupuestal idp
            CROSS JOIN LATERAL validar_codigos_obligatorios(idp.id) vco
            WHERE idp.documento_id = NEW.id
              AND idp.es_activo = true
              AND NOT vco.es_valido
        LOOP
            v_items_invalidos := v_items_invalidos + 1;
            RAISE NOTICE 'Ítem % tiene códigos faltantes: %', 
                v_validacion.descripcion, v_validacion.mensaje_validacion;
        END LOOP;
        
        -- Si hay ítems inválidos, impedir la aprobación
        IF v_items_invalidos > 0 THEN
            RAISE EXCEPTION 'No se puede aprobar el documento. % ítems tienen códigos obligatorios faltantes', 
                v_items_invalidos;
        END IF;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Crear el trigger
CREATE TRIGGER trigger_validar_codigos_antes_aprobacion
    BEFORE UPDATE ON documento_presupuestal
    FOR EACH ROW
    EXECUTE FUNCTION trigger_validar_codigos_antes_aprobacion();
```

---

## Ejemplos Prácticos

### Ejemplo 1: Crear RP heredando códigos del CDP precedente

```sql
-- 1. Crear ítem de RP
INSERT INTO item_documento_presupuestal VALUES
(201, 201, 1, 'Honorarios consultoría técnica', 25000000.00, 25000000.00, 0.00, 25000000.00,
 true, NOW(), NOW(), 456, 456);

-- 2. Heredar códigos automáticamente del ítem del CDP precedente
SELECT heredar_codigos_precedente(
    201, -- item_documento_id (nuevo ítem RP)
    101, -- item_precedente_id (ítem del CDP)
    456  -- usuario_id
);

-- Resultado: Se crean automáticamente las asignaciones de código heredadas

-- 3. Asignar código específico del RP (Tercero/Proveedor)
SELECT asignar_codigo_item(
    201, -- item_documento_id
    6,   -- tipo_codigo_id (Tercero)
    1234, -- codigo_id (NIT del proveedor)
    456,  -- usuario_id
    'Proveedor seleccionado según estudio de mercado'
);
```

### Ejemplo 2: Validar códigos obligatorios antes de expedir documento

```sql
-- Verificar que todos los ítems de un documento tengan códigos obligatorios
SELECT 
    idp.id as item_id,
    idp.descripcion,
    vco.es_valido,
    vco.codigos_faltantes,
    vco.mensaje_validacion
FROM item_documento_presupuestal idp
CROSS JOIN LATERAL validar_codigos_obligatorios(idp.id) vco
WHERE idp.documento_id = 201 -- ID del documento RP
  AND idp.es_activo = true
ORDER BY idp.numero_item;

-- Resultado muestra qué ítems están completos y cuáles necesitan códigos adicionales
```

### Ejemplo 3: Consultar códigos de un ítem específico

```sql
-- Ver todos los códigos asignados a un ítem específico
SELECT 
    tipo_codigo_nombre,
    valor_codigo,
    nombre_codigo,
    es_heredado,
    fue_modificado_usuario,
    es_obligatorio,
    fecha_asignacion,
    usuario_asignacion
FROM item_codigos_completos
WHERE item_documento_id = 201
ORDER BY tipo_codigo_nombre;

-- Resultado:
-- tipo_codigo_nombre    | valor_codigo  | nombre_codigo           | es_heredado | fue_modificado_usuario | es_obligatorio
-- Centro de Costo       | CC0105        | Secretaría de Educación | true        | false                  | false
-- Fuente de Financiación| 001           | Tesoro Nacional         | true        | false                  | true
-- Rubro Presupuestal    | 01.02.03.001  | Capacitación Docente    | true        | false                  | true
-- Tercero/Proveedor     | 900123456     | Universidad Nacional    | false       | false                  | true
```

### Ejemplo 4: Modificar código heredado

```sql
-- Modificar un código que fue heredado pero permite modificación
SELECT asignar_codigo_item(
    201, -- item_documento_id
    4,   -- tipo_codigo_id (Centro de Costo)
    2567, -- codigo_id (nuevo centro de costo)
    456,  -- usuario_id
    'Cambio de centro de costo por reasignación administrativa'
);

-- El sistema registrará que fue modificado por el usuario
-- y mantendrá el historial del cambio
```

### Ejemplo 5: Reporte de herencia de códigos por documento

```sql
-- Ver resumen de códigos por documento
SELECT 
    numero_documento,
    tipo_documento_codigo,
    total_codigos_asignados,
    total_codigos_heredados,
    total_codigos_modificados,
    total_codigos_obligatorios,
    CASE 
        WHEN codigos_obligatorios_faltantes IS NULL THEN 'COMPLETO'
        ELSE 'FALTAN: ' || codigos_obligatorios_faltantes
    END as estado_codigos
FROM resumen_codigos_por_documento
WHERE fecha_documento >= '2025-01-01'
ORDER BY fecha_documento DESC;

-- Resultado:
-- numero_documento | tipo_documento_codigo | total_codigos_asignados | total_codigos_heredados | total_codigos_modificados | total_codigos_obligatorios | estado_codigos
-- RP-2025-001      | RP                     | 4                       | 3                       | 1                         | 3                          | COMPLETO
-- CDP-2025-001     | CDP                    | 3                       | 0                       | 0                         | 2                          | COMPLETO
-- OP-2025-001      | OP                     | 2                       | 1                       | 0                         | 3                          | FALTAN: Modalidad de Pago
```

---

## Próximos Pasos

Este documento estableció el sistema completo de códigos para ítems de documentos presupuestales. Los próximos documentos cubrirán:

**07 - Reportes y Consultas Avanzadas**: Reportes estándar y consultas complejas del sistema presupuestario.
**08 - Integración y Workflows**: Flujos de trabajo completos y integraciones con otros módulos.

Los códigos de ítems están ahora completamente definidos con:
- ✅ Configuración flexible de códigos por tipo de documento
- ✅ Sistema de herencia automática desde documentos precedentes
- ✅ Validaciones de códigos obligatorios y formatos
- ✅ Historial completo de cambios y asignaciones
- ✅ Triggers automáticos para herencia y validación
- ✅ Vistas para consultas y reportes
- ✅ Funciones especializadas para gestión de códigos
- ✅ Ejemplos prácticos de todos los escenarios
- ✅ Control de modificaciones sobre códigos heredados
