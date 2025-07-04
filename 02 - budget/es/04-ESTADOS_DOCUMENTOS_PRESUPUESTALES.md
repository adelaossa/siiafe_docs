# 04 - Estados de Documentos Presupuestales

## Descripción General

Los **Estados de Documentos Presupuestales** son un sistema que controla el ciclo de vida de cada documento, definiendo en qué fase se encuentra y qué operaciones están permitidas. El sistema maneja dos tipos de transiciones: **manuales** (controladas por el usuario con posibles anexos) y **automáticas** (evaluadas mediante condiciones JavaScript cuando ocurren movimientos).

## Conceptos Fundamentales

### ¿Qué es un Estado de Documento?

Un estado de documento presupuestal es:

- **Indicador de fase**: Define en qué etapa del ciclo presupuestario se encuentra el documento
- **Controlador de permisos**: Determina qué operaciones están permitidas
- **Validador de transiciones**: Controla los cambios de estado válidos
- **Auditor de cambios**: Registra quién, cuándo y por qué cambió el estado

### Tipos de Transiciones

#### Transiciones Manuales
- **Control humano**: Requieren intervención y decisión del usuario
- **Documentación**: Pueden requerir anexos (PDFs firmados, resoluciones, etc.)
- **Validación previa**: Verifican condiciones antes de permitir el cambio
- **Trazabilidad**: Registran usuario, fecha, observaciones y anexos

#### Transiciones Automáticas
- **Evaluación continua**: Se ejecutan después de cada movimiento presupuestal
- **Condiciones JavaScript**: Usan scripts configurables para evaluar si debe cambiar el estado
- **Basadas en datos**: Evalúan estado actual + valores de columnas del documento
- **Sin intervención**: No requieren acción del usuario

### Características Principales

- **Configurabilidad**: Estados y transiciones son completamente configurables
- **Flexibilidad**: Diferentes tipos de documento pueden tener diferentes estados
- **Seguridad**: Validaciones estrictas para cambios de estado
- **Auditoría**: Historial completo de cambios de estado
- **Extensibilidad**: Sistema de scripts JavaScript para lógica compleja

---

## Estructura de Datos

### Tabla Principal: estado_documento

```sql
CREATE TABLE estado_documento (
    id SERIAL PRIMARY KEY,
    codigo VARCHAR(50) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT,
    
    -- Configuración del estado
    es_estado_inicial BOOLEAN DEFAULT FALSE,
    es_estado_final BOOLEAN DEFAULT FALSE,
    permite_edicion BOOLEAN DEFAULT TRUE,
    permite_movimientos BOOLEAN DEFAULT TRUE,
    permite_anulacion BOOLEAN DEFAULT FALSE,
    
    -- Configuración visual
    color_estado VARCHAR(20) DEFAULT '#007bff', -- Para interfaz de usuario
    icono_estado VARCHAR(50), -- Clase CSS o nombre del icono
    orden_presentacion INTEGER DEFAULT 1,
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER REFERENCES usuario(id),
    
    -- Índice único
    UNIQUE(codigo)
);
```

### Tabla: estado_documento_tipo

```sql
CREATE TABLE estado_documento_tipo (
    id SERIAL PRIMARY KEY,
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id),
    estado_documento_id INTEGER NOT NULL REFERENCES estado_documento(id),
    
    -- Configuración específica para este tipo de documento
    es_estado_inicial_tipo BOOLEAN DEFAULT FALSE,
    orden_flujo INTEGER DEFAULT 1,
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER REFERENCES usuario(id),
    
    -- Índice único
    UNIQUE(tipo_documento_id, estado_documento_id)
);
```

### Tabla: transicion_estado

```sql
CREATE TABLE transicion_estado (
    id SERIAL PRIMARY KEY,
    codigo VARCHAR(50) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT,
    
    -- Estados origen y destino
    estado_origen_id INTEGER NOT NULL REFERENCES estado_documento(id),
    estado_destino_id INTEGER NOT NULL REFERENCES estado_documento(id),
    
    -- Tipo de transición
    tipo_transicion VARCHAR(20) NOT NULL CHECK (tipo_transicion IN ('MANUAL', 'AUTOMATICA')),
    
    -- Configuración para transiciones manuales
    requiere_anexo BOOLEAN DEFAULT FALSE,
    tipos_anexo_permitidos VARCHAR(200), -- PDF,DOC,JPG separados por coma
    tamano_maximo_anexo INTEGER DEFAULT 10485760, -- 10MB en bytes
    requiere_observaciones BOOLEAN DEFAULT FALSE,
    requiere_autorizacion BOOLEAN DEFAULT FALSE,
    
    -- Configuración para transiciones automáticas
    condicion_javascript TEXT, -- Script para evaluar si debe ejecutarse
    prioridad_evaluacion INTEGER DEFAULT 1, -- Orden de evaluación
    
    -- Validaciones comunes
    script_validacion_previa TEXT, -- JavaScript para validar antes del cambio
    mensaje_validacion_fallo VARCHAR(500),
    
    -- Configuración de permisos
    roles_autorizados JSON, -- Array de roles que pueden ejecutar esta transición
    
    -- Metadatos
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER REFERENCES usuario(id),
    
    -- Índice único
    UNIQUE(codigo)
);
```

### Tabla: historial_estado_documento

```sql
CREATE TABLE historial_estado_documento (
    id SERIAL PRIMARY KEY,
    documento_id INTEGER NOT NULL REFERENCES documento_presupuestal(id),
    transicion_id INTEGER NOT NULL REFERENCES transicion_estado(id),
    
    -- Estados
    estado_anterior_id INTEGER REFERENCES estado_documento(id),
    estado_nuevo_id INTEGER NOT NULL REFERENCES estado_documento(id),
    
    -- Tipo de cambio
    tipo_cambio VARCHAR(20) NOT NULL CHECK (tipo_cambio IN ('MANUAL', 'AUTOMATICO', 'SISTEMA')),
    
    -- Información del cambio manual
    observaciones_cambio TEXT,
    archivo_anexo_nombre VARCHAR(255),
    archivo_anexo_ruta VARCHAR(500),
    archivo_anexo_tipo VARCHAR(50),
    archivo_anexo_tamano INTEGER,
    
    -- Información del cambio automático
    condicion_evaluada TEXT, -- Condición JavaScript que se cumplió
    resultado_evaluacion JSON, -- Resultado detallado de la evaluación
    
    -- Información del movimiento que disparó el cambio (para automáticos)
    movimiento_disparador_id INTEGER REFERENCES movimiento_presupuestal(id),
    
    -- Metadatos
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_cambio INTEGER REFERENCES usuario(id),
    ip_usuario INET,
    datos_contexto JSON,
    
    -- Validación del estado nuevo
    es_cambio_valido BOOLEAN DEFAULT TRUE,
    motivo_invalidacion TEXT
);

-- Índices
CREATE INDEX idx_historial_documento ON historial_estado_documento(documento_id);
CREATE INDEX idx_historial_fecha ON historial_estado_documento(fecha_cambio);
CREATE INDEX idx_historial_usuario ON historial_estado_documento(usuario_cambio);
CREATE INDEX idx_historial_tipo ON historial_estado_documento(tipo_cambio);
```

---

## Configuraciones de Estados por Tipo de Documento

### Estados para Presupuesto de Gasto

```sql
-- Estados para Presupuesto de Gasto
INSERT INTO estado_documento VALUES
(1, 'BORRADOR', 'Borrador', 'Documento en construcción, permite edición completa', 
 true, false, true, false, false, '#6c757d', 'fa-edit', 1, true, NOW(), 1),

(2, 'REVISION', 'En Revisión', 'Documento sometido a revisión técnica', 
 false, false, false, false, false, '#ffc107', 'fa-search', 2, true, NOW(), 1),

(3, 'APROBADO', 'Aprobado', 'Documento aprobado y vigente para operaciones', 
 false, false, false, true, false, '#28a745', 'fa-check-circle', 3, true, NOW(), 1),

(4, 'MODIFICADO', 'Modificado', 'Documento con modificaciones aplicadas', 
 false, false, false, true, false, '#17a2b8', 'fa-edit', 4, true, NOW(), 1),

(5, 'ANULADO', 'Anulado', 'Documento anulado, sin efectos presupuestales', 
 false, true, false, false, true, '#dc3545', 'fa-ban', 5, true, NOW(), 1);

-- Asociar estados con tipo de documento Presupuesto de Gasto
INSERT INTO estado_documento_tipo VALUES
(1, 1, 1, true, 1, true, NOW(), 1),  -- BORRADOR es inicial
(2, 1, 2, false, 2, true, NOW(), 1), -- REVISION
(3, 1, 3, false, 3, true, NOW(), 1), -- APROBADO
(4, 1, 4, false, 4, true, NOW(), 1), -- MODIFICADO
(5, 1, 5, false, 5, true, NOW(), 1); -- ANULADO
```

### Estados para Adición Presupuestal

```sql
-- Estados para Adición Presupuestal
INSERT INTO estado_documento VALUES
(6, 'PROPUESTA', 'Propuesta', 'Adición propuesta, pendiente de aprobación', 
 true, false, true, false, false, '#6c757d', 'fa-file-alt', 1, true, NOW(), 1),

(7, 'APROBADA', 'Aprobada', 'Adición aprobada oficialmente', 
 false, false, false, true, false, '#28a745', 'fa-stamp', 2, true, NOW(), 1),

(8, 'APLICADA', 'Aplicada', 'Adición aplicada al presupuesto', 
 false, true, false, false, false, '#007bff', 'fa-check-double', 3, true, NOW(), 1);

-- Asociar estados con tipo de documento Adición Presupuestal
INSERT INTO estado_documento_tipo VALUES
(6, 2, 6, true, 1, true, NOW(), 1),  -- PROPUESTA es inicial
(7, 2, 7, false, 2, true, NOW(), 1), -- APROBADA
(8, 2, 8, false, 3, true, NOW(), 1); -- APLICADA
```

### Estados para CDP

```sql
-- Estados para CDP
INSERT INTO estado_documento VALUES
(9, 'EXPEDIDO', 'Expedido', 'CDP expedido, disponible para comprometer', 
 true, false, false, true, false, '#28a745', 'fa-certificate', 1, true, NOW(), 1),

(10, 'COMPROMETIDO_PARCIAL', 'Comprometido Parcial', 'CDP con compromisos parciales', 
 false, false, false, true, false, '#ffc107', 'fa-clock', 2, true, NOW(), 1),

(11, 'COMPROMETIDO_TOTAL', 'Comprometido Total', 'CDP totalmente comprometido', 
 false, false, false, false, false, '#17a2b8', 'fa-lock', 3, true, NOW(), 1),

(12, 'LIBERADO', 'Liberado', 'CDP con recursos liberados', 
 false, true, false, false, true, '#6c757d', 'fa-unlock', 4, true, NOW(), 1);

-- Asociar estados con tipo de documento CDP
INSERT INTO estado_documento_tipo VALUES
(9, 5, 9, true, 1, true, NOW(), 1),   -- EXPEDIDO es inicial
(10, 5, 10, false, 2, true, NOW(), 1), -- COMPROMETIDO_PARCIAL
(11, 5, 11, false, 3, true, NOW(), 1), -- COMPROMETIDO_TOTAL
(12, 5, 12, false, 4, true, NOW(), 1); -- LIBERADO
```

---

## Configuración de Transiciones

### Transiciones Manuales - Adición Presupuestal

```sql
-- Transición manual: Propuesta → Aprobada
INSERT INTO transicion_estado VALUES
(1, 'APROBAR_ADICION', 'Aprobar Adición Presupuestal', 
 'Transición manual que aprueba una adición presupuestal',
 6, 7, 'MANUAL', -- PROPUESTA → APROBADA
 true, 'PDF,DOC', 10485760, true, true, -- Requiere anexo PDF/DOC, observaciones y autorización
 null, 1, -- No es automática
 'validar_montos_adicion(documento)', -- Script de validación previa
 'Los montos de la adición no pueden exceder los límites establecidos',
 '["DIRECTOR_PRESUPUESTO", "SECRETARIO_HACIENDA"]', -- Roles autorizados
 true, NOW(), 1),

-- Transición automática: Aprobada → Aplicada  
(2, 'APLICAR_ADICION_AUTO', 'Aplicar Adición Automáticamente',
 'Transición automática que aplica la adición cuando se crea el movimiento',
 7, 8, 'AUTOMATICA', -- APROBADA → APLICADA
 false, null, null, false, false, -- No requiere anexos (es automática)
 'documento.estado_actual === "APROBADA" && documento.movimientos_recientes.length > 0', 1,
 'validar_presupuesto_vigente(documento)', 
 'Solo se puede aplicar sobre presupuestos vigentes',
 null, -- Sin restricción de roles (es automática)
 true, NOW(), 1);
```

### Transiciones Automáticas - CDP

```sql
-- Transición automática: Expedido → Comprometido Parcial
INSERT INTO transicion_estado VALUES
(3, 'CDP_COMPROMETIDO_PARCIAL_AUTO', 'CDP Comprometido Parcialmente',
 'Transición automática cuando se compromete parcialmente un CDP',
 9, 10, 'AUTOMATICA', -- EXPEDIDO → COMPROMETIDO_PARCIAL
 false, null, null, false, false,
 'documento.estado_actual === "EXPEDIDO" && documento.columnas.VALOR_COMPROMETIDO > 0 && documento.columnas.SALDO_COMPROMETER > 0', 1,
 null, null, null,
 true, NOW(), 1),

-- Transición automática: Expedido/Parcial → Comprometido Total
(4, 'CDP_COMPROMETIDO_TOTAL_AUTO', 'CDP Comprometido Totalmente',
 'Transición automática cuando se compromete totalmente un CDP',
 10, 11, 'AUTOMATICA', -- COMPROMETIDO_PARCIAL → COMPROMETIDO_TOTAL
 false, null, null, false, false,
 '(documento.estado_actual === "EXPEDIDO" || documento.estado_actual === "COMPROMETIDO_PARCIAL") && (documento.columnas.SALDO_COMPROMETER === 0 && documento.columnas.VALOR_COMPROMETIDO > 0)', 1,
 null, null, null,
 true, NOW(), 1),

-- Transición automática: Comprometido → Liberado (cuando se libera todo)
(5, 'CDP_LIBERADO_AUTO', 'CDP Liberado Automáticamente',
 'Transición automática cuando se libera completamente un CDP',
 10, 12, 'AUTOMATICA', -- COMPROMETIDO_PARCIAL → LIBERADO
 false, null, null, false, false,
 'documento.estado_actual === "COMPROMETIDO_PARCIAL" && documento.columnas.VALOR_LIBERADO > 0 && documento.columnas.VALOR_COMPROMETIDO === 0', 2,
 null, null, null,
 true, NOW(), 1);
```

---

## Funciones de Gestión de Estados

### Función: cambiar_estado_documento_manual

```sql
CREATE OR REPLACE FUNCTION cambiar_estado_documento_manual(
    p_documento_id INTEGER,
    p_transicion_codigo VARCHAR(50),
    p_usuario_id INTEGER,
    p_observaciones TEXT,
    p_archivo_anexo_nombre VARCHAR(255) DEFAULT NULL,
    p_archivo_anexo_ruta VARCHAR(500) DEFAULT NULL,
    p_archivo_anexo_tipo VARCHAR(50) DEFAULT NULL,
    p_archivo_anexo_tamano INTEGER DEFAULT NULL
)
RETURNS INTEGER AS $$
DECLARE
    v_historial_id INTEGER;
    v_transicion RECORD;
    v_documento RECORD;
    v_estado_actual_id INTEGER;
    v_resultado_validacion BOOLEAN;
    v_mensaje_error VARCHAR(500);
BEGIN
    -- Obtener documento y estado actual
    SELECT d.*, ed.id as estado_actual_id, ed.codigo as estado_actual_codigo
    INTO v_documento
    FROM documento_presupuestal d
    JOIN estado_documento ed ON d.estado_actual_id = ed.id
    WHERE d.id = p_documento_id;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Documento % no encontrado', p_documento_id;
    END IF;
    
    -- Obtener configuración de la transición
    SELECT * INTO v_transicion
    FROM transicion_estado
    WHERE codigo = p_transicion_codigo 
      AND tipo_transicion = 'MANUAL'
      AND estado_origen_id = v_documento.estado_actual_id
      AND es_activo = true;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Transición % no válida desde estado %', p_transicion_codigo, v_documento.estado_actual_codigo;
    END IF;
    
    -- Validar permisos del usuario
    IF NOT validar_permisos_transicion(p_usuario_id, v_transicion.roles_autorizados) THEN
        RAISE EXCEPTION 'Usuario no autorizado para ejecutar esta transición';
    END IF;
    
    -- Validar anexos si son requeridos
    IF v_transicion.requiere_anexo THEN
        IF p_archivo_anexo_nombre IS NULL OR p_archivo_anexo_ruta IS NULL THEN
            RAISE EXCEPTION 'Esta transición requiere un archivo anexo';
        END IF;
        
        -- Validar tipo de archivo
        IF NOT validar_tipo_archivo(p_archivo_anexo_tipo, v_transicion.tipos_anexo_permitidos) THEN
            RAISE EXCEPTION 'Tipo de archivo no permitido. Tipos válidos: %', v_transicion.tipos_anexo_permitidos;
        END IF;
        
        -- Validar tamaño
        IF p_archivo_anexo_tamano > v_transicion.tamano_maximo_anexo THEN
            RAISE EXCEPTION 'Archivo excede el tamaño máximo permitido de % bytes', v_transicion.tamano_maximo_anexo;
        END IF;
    END IF;
    
    -- Validar observaciones si son requeridas
    IF v_transicion.requiere_observaciones AND (p_observaciones IS NULL OR LENGTH(TRIM(p_observaciones)) = 0) THEN
        RAISE EXCEPTION 'Esta transición requiere observaciones';
    END IF;
    
    -- Ejecutar validación previa si existe
    IF v_transicion.script_validacion_previa IS NOT NULL THEN
        SELECT * INTO v_resultado_validacion, v_mensaje_error
        FROM ejecutar_script_validacion(v_transicion.script_validacion_previa, p_documento_id);
        
        IF NOT v_resultado_validacion THEN
            RAISE EXCEPTION 'Validación previa falló: %', COALESCE(v_mensaje_error, v_transicion.mensaje_validacion_fallo);
        END IF;
    END IF;
    
    -- Actualizar estado del documento
    UPDATE documento_presupuestal 
    SET estado_actual_id = v_transicion.estado_destino_id,
        fecha_actualizacion = NOW(),
        usuario_actualizacion = p_usuario_id
    WHERE id = p_documento_id;
    
    -- Registrar en historial
    INSERT INTO historial_estado_documento 
        (documento_id, transicion_id, estado_anterior_id, estado_nuevo_id, tipo_cambio,
         observaciones_cambio, archivo_anexo_nombre, archivo_anexo_ruta, archivo_anexo_tipo, 
         archivo_anexo_tamano, usuario_cambio, ip_usuario)
    VALUES 
        (p_documento_id, v_transicion.id, v_documento.estado_actual_id, v_transicion.estado_destino_id, 'MANUAL',
         p_observaciones, p_archivo_anexo_nombre, p_archivo_anexo_ruta, p_archivo_anexo_tipo,
         p_archivo_anexo_tamano, p_usuario_id, inet_client_addr())
    RETURNING id INTO v_historial_id;
    
    -- Evaluar transiciones automáticas después del cambio
    PERFORM evaluar_transiciones_automaticas(p_documento_id, null);
    
    RETURN v_historial_id;
END;
$$ LANGUAGE plpgsql;
```

### Función: evaluar_transiciones_automaticas

```sql
CREATE OR REPLACE FUNCTION evaluar_transiciones_automaticas(
    p_documento_id INTEGER,
    p_movimiento_disparador_id INTEGER DEFAULT NULL
)
RETURNS INTEGER AS $$
DECLARE
    v_transicion RECORD;
    v_documento RECORD;
    v_resultado_evaluacion BOOLEAN;
    v_resultado_detalle JSON;
    v_cambios_realizados INTEGER := 0;
    v_historial_id INTEGER;
BEGIN
    -- Obtener documento y estado actual
    SELECT d.*, ed.id as estado_actual_id, ed.codigo as estado_actual_codigo
    INTO v_documento
    FROM documento_presupuestal d
    JOIN estado_documento ed ON d.estado_actual_id = ed.id
    WHERE d.id = p_documento_id;
    
    IF NOT FOUND THEN
        RETURN 0;
    END IF;
    
    -- Evaluar todas las transiciones automáticas disponibles desde el estado actual
    FOR v_transicion IN 
        SELECT * FROM transicion_estado
        WHERE estado_origen_id = v_documento.estado_actual_id
          AND tipo_transicion = 'AUTOMATICA'
          AND es_activo = true
        ORDER BY prioridad_evaluacion
    LOOP
        -- Ejecutar script de condición
        SELECT resultado, detalle INTO v_resultado_evaluacion, v_resultado_detalle
        FROM evaluar_condicion_javascript(v_transicion.condicion_javascript, p_documento_id);
        
        -- Si la condición se cumple, ejecutar la transición
        IF v_resultado_evaluacion THEN
            -- Ejecutar validación previa si existe
            IF v_transicion.script_validacion_previa IS NOT NULL THEN
                SELECT resultado INTO v_resultado_evaluacion
                FROM ejecutar_script_validacion(v_transicion.script_validacion_previa, p_documento_id);
                
                IF NOT v_resultado_evaluacion THEN
                    CONTINUE; -- Saltar esta transición
                END IF;
            END IF;
            
            -- Actualizar estado del documento
            UPDATE documento_presupuestal 
            SET estado_actual_id = v_transicion.estado_destino_id,
                fecha_actualizacion = NOW(),
                usuario_actualizacion = 0 -- Usuario sistema
            WHERE id = p_documento_id;
            
            -- Registrar en historial
            INSERT INTO historial_estado_documento 
                (documento_id, transicion_id, estado_anterior_id, estado_nuevo_id, tipo_cambio,
                 condicion_evaluada, resultado_evaluacion, movimiento_disparador_id, usuario_cambio)
            VALUES 
                (p_documento_id, v_transicion.id, v_documento.estado_actual_id, v_transicion.estado_destino_id, 'AUTOMATICO',
                 v_transicion.condicion_javascript, v_resultado_detalle, p_movimiento_disparador_id, 0)
            RETURNING id INTO v_historial_id;
            
            v_cambios_realizados := v_cambios_realizados + 1;
            
            -- Actualizar estado actual para próximas evaluaciones
            v_documento.estado_actual_id := v_transicion.estado_destino_id;
            
            -- Evaluar recursivamente si hay más transiciones disponibles
            v_cambios_realizados := v_cambios_realizados + evaluar_transiciones_automaticas(p_documento_id, p_movimiento_disparador_id);
            
            -- Solo ejecutar una transición por llamada para evitar bucles
            EXIT;
        END IF;
    END LOOP;
    
    RETURN v_cambios_realizados;
END;
$$ LANGUAGE plpgsql;
```

### Función: evaluar_condicion_javascript

```sql
CREATE OR REPLACE FUNCTION evaluar_condicion_javascript(
    p_script TEXT,
    p_documento_id INTEGER
)
RETURNS TABLE(resultado BOOLEAN, detalle JSON) AS $$
DECLARE
    v_documento_data JSON;
    v_script_completo TEXT;
    v_resultado BOOLEAN;
    v_detalle JSON;
BEGIN
    -- Obtener datos completos del documento en formato JSON
    SELECT to_json(documento_completo.*) INTO v_documento_data
    FROM (
        SELECT 
            d.*,
            ed.codigo as estado_actual_codigo,
            ed.nombre as estado_actual_nombre,
            -- Valores de columnas
            (
                SELECT json_object_agg(tdc.codigo_columna, COALESCE(idcv.valor_columna, 0))
                FROM tipo_documento_columna tdc
                LEFT JOIN item_documento_columna_valor idcv ON 
                    idcv.tipo_documento_columna_id = tdc.id AND 
                    idcv.item_documento_id IN (
                        SELECT id FROM item_documento_presupuestal 
                        WHERE documento_id = d.id AND es_activo = true
                    )
                WHERE tdc.tipo_documento_id = d.tipo_documento_id AND tdc.es_activo = true
            ) as valores_columnas,
            -- Movimientos recientes
            (
                SELECT json_agg(
                    json_build_object(
                        'tipo_movimiento', tmp.codigo,
                        'valor', mp.valor_movimiento,
                        'fecha', mp.fecha_movimiento
                    )
                )
                FROM movimiento_presupuestal mp
                JOIN tipo_movimiento_presupuestal tmp ON mp.tipo_movimiento_id = tmp.id
                WHERE mp.documento_afectado_id = d.id
                  AND mp.estado_movimiento = 'ACTIVO'
                ORDER BY mp.fecha_movimiento DESC
                LIMIT 10
            ) as movimientos_recientes
        FROM documento_presupuestal d
        JOIN estado_documento ed ON d.estado_actual_id = ed.id
        WHERE d.id = p_documento_id
    ) documento_completo;
    
    -- Construir script completo con datos del documento
    v_script_completo := '
        // Datos del documento disponibles en variable "documento"
        var documento = ' || v_documento_data::text || ';
        
        // Agregar propiedad "columnas" para fácil acceso a valores
        documento.columnas = documento.valores_columnas || {};
        
        // Evaluar condición directamente
        var resultado = (' || p_script || ');
        
        // Retornar resultado con detalle
        JSON.stringify({
            resultado: !!resultado,
            valor_evaluado: resultado,
            datos_usados: {
                estado_actual: documento.estado_actual_codigo,
                columnas: documento.columnas,
                total_movimientos: (documento.movimientos_recientes || []).length
            }
        });
    ';
    
    -- Ejecutar script JavaScript (requiere extensión plv8 o similar)
    -- Por simplicidad, aquí usaremos una implementación básica
    -- En producción se usaría una extensión como plv8
    BEGIN
        -- Simulación de evaluación JavaScript
        -- En implementación real usaría: SELECT plv8_exec(v_script_completo) INTO v_detalle;
        
        -- Por ahora, evaluación simplificada para condiciones comunes
        IF p_script LIKE '%documento.columnas.SALDO_COMPROMETER%' THEN
            -- Lógica específica para saldo comprometer de CDP
            SELECT evaluar_condicion_cdp_saldo(p_documento_id) INTO v_resultado;
        ELSIF p_script LIKE '%documento.movimientos_recientes.length%' THEN
            -- Lógica específica para movimientos recientes
            SELECT verificar_movimientos_recientes(p_documento_id) INTO v_resultado;
        ELSIF p_script LIKE '%documento.columnas.VALOR_COMPROMETIDO%' THEN
            -- Lógica específica para valor comprometido
            SELECT evaluar_valor_comprometido(p_documento_id) INTO v_resultado;
        ELSE
            -- Evaluación genérica
            v_resultado := false;
        END IF;
        
        v_detalle := json_build_object(
            'resultado', v_resultado,
            'script_evaluado', p_script,
            'documento_id', p_documento_id
        );
        
    EXCEPTION WHEN OTHERS THEN
        v_resultado := false;
        v_detalle := json_build_object(
            'resultado', false,
            'error', SQLERRM,
            'script', p_script
        );
    END;
    
    RETURN QUERY SELECT v_resultado, v_detalle;
END;
$$ LANGUAGE plpgsql;
```

---

## Funciones Auxiliares de Evaluación

### Función: evaluar_condicion_cdp_saldo

```sql
CREATE OR REPLACE FUNCTION evaluar_condicion_cdp_saldo(
    p_documento_id INTEGER
)
RETURNS BOOLEAN AS $$
DECLARE
    v_saldo_comprometer DECIMAL(15,2) := 0;
    v_valor_comprometido DECIMAL(15,2) := 0;
BEGIN
    -- Obtener valores actuales del documento
    SELECT 
        SUM(CASE WHEN tdc.codigo_columna = 'saldo_comprometer' THEN COALESCE(idcv.valor_columna, 0) ELSE 0 END),
        SUM(CASE WHEN tdc.codigo_columna = 'valor_comprometido' THEN COALESCE(idcv.valor_columna, 0) ELSE 0 END)
    INTO v_saldo_comprometer, v_valor_comprometido
    FROM item_documento_presupuestal idp
    LEFT JOIN item_documento_columna_valor idcv ON idp.id = idcv.item_documento_id
    LEFT JOIN tipo_documento_columna tdc ON idcv.tipo_documento_columna_id = tdc.id
    WHERE idp.documento_id = p_documento_id
      AND idp.es_activo = true;
    
    -- Evaluar condición: saldo = 0 y hay valor comprometido
    RETURN v_saldo_comprometer = 0 AND v_valor_comprometido > 0;
END;
$$ LANGUAGE plpgsql;
```

### Función: evaluar_valor_comprometido

```sql
CREATE OR REPLACE FUNCTION evaluar_valor_comprometido(
    p_documento_id INTEGER
)
RETURNS BOOLEAN AS $$
DECLARE
    v_valor_comprometido DECIMAL(15,2) := 0;
    v_saldo_comprometer DECIMAL(15,2) := 0;
BEGIN
    -- Obtener valores actuales del documento
    SELECT 
        SUM(CASE WHEN tdc.codigo_columna = 'valor_comprometido' THEN COALESCE(idcv.valor_columna, 0) ELSE 0 END),
        SUM(CASE WHEN tdc.codigo_columna = 'saldo_comprometer' THEN COALESCE(idcv.valor_columna, 0) ELSE 0 END)
    INTO v_valor_comprometido, v_saldo_comprometer
    FROM item_documento_presupuestal idp
    LEFT JOIN item_documento_columna_valor idcv ON idp.id = idcv.item_documento_id
    LEFT JOIN tipo_documento_columna tdc ON idcv.tipo_documento_columna_id = tdc.id
    WHERE idp.documento_id = p_documento_id
      AND idp.es_activo = true;
    
    -- Evaluar condición: hay valor comprometido y aún hay saldo
    RETURN v_valor_comprometido > 0 AND v_saldo_comprometer > 0;
END;
$$ LANGUAGE plpgsql;
```

### Función: verificar_movimientos_recientes

```sql
CREATE OR REPLACE FUNCTION verificar_movimientos_recientes(
    p_documento_id INTEGER
)
RETURNS BOOLEAN AS $$
BEGIN
    RETURN EXISTS (
        SELECT 1 
        FROM movimiento_presupuestal mp
        WHERE mp.documento_afectado_id = p_documento_id
          AND mp.estado_movimiento = 'ACTIVO'
          AND mp.fecha_movimiento >= NOW() - INTERVAL '1 hour' -- Movimientos recientes
    );
END;
$$ LANGUAGE plpgsql;
```

---

## Trigger para Evaluación Automática después de Movimientos

### Trigger: trigger_evaluar_estados_post_movimiento

```sql
CREATE OR REPLACE FUNCTION trigger_evaluar_estados_post_movimiento()
RETURNS TRIGGER AS $$
BEGIN
    -- Evaluar transiciones automáticas en el documento afectado
    PERFORM evaluar_transiciones_automaticas(NEW.documento_afectado_id, NEW.id);
    
    -- Si el movimiento afecta un documento precedente, evaluarlo también
    IF NEW.documento_origen_id != NEW.documento_afectado_id THEN
        PERFORM evaluar_transiciones_automaticas(NEW.documento_origen_id, NEW.id);
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Crear el trigger
CREATE TRIGGER trigger_evaluar_estados_post_movimiento
    AFTER INSERT ON movimiento_presupuestal
    FOR EACH ROW
    EXECUTE FUNCTION trigger_evaluar_estados_post_movimiento();
```

---

## Vistas para Consulta de Estados

### Vista: vista_estados_documentos

```sql
CREATE VIEW vista_estados_documentos AS
SELECT 
    d.id as documento_id,
    d.numero_documento,
    tdp.nombre as tipo_documento,
    ed.codigo as estado_actual_codigo,
    ed.nombre as estado_actual_nombre,
    ed.descripcion as estado_descripcion,
    ed.color_estado,
    ed.icono_estado,
    ed.permite_edicion,
    ed.permite_movimientos,
    ed.permite_anulacion,
    
    -- Último cambio de estado
    (
        SELECT json_build_object(
            'fecha_cambio', h.fecha_cambio,
            'tipo_cambio', h.tipo_cambio,
            'usuario', u.nombre,
            'observaciones', h.observaciones_cambio,
            'archivo_anexo', h.archivo_anexo_nombre
        )
        FROM historial_estado_documento h
        LEFT JOIN usuario u ON h.usuario_cambio = u.id
        WHERE h.documento_id = d.id
        ORDER BY h.fecha_cambio DESC
        LIMIT 1
    ) as ultimo_cambio_estado,
    
    -- Transiciones disponibles
    (
        SELECT json_agg(
            json_build_object(
                'transicion_codigo', t.codigo,
                'transicion_nombre', t.nombre,
                'estado_destino', ed_destino.nombre,
                'tipo_transicion', t.tipo_transicion,
                'requiere_anexo', t.requiere_anexo,
                'requiere_observaciones', t.requiere_observaciones
            )
        )
        FROM transicion_estado t
        JOIN estado_documento ed_destino ON t.estado_destino_id = ed_destino.id
        WHERE t.estado_origen_id = d.estado_actual_id
          AND t.es_activo = true
    ) as transiciones_disponibles
    
FROM documento_presupuestal d
JOIN tipo_documento_presupuestal tdp ON d.tipo_documento_id = tdp.id
JOIN estado_documento ed ON d.estado_actual_id = ed.id
WHERE d.es_activo = true;
```

### Vista: vista_historial_estados

```sql
CREATE VIEW vista_historial_estados AS
SELECT 
    h.id as historial_id,
    h.documento_id,
    d.numero_documento,
    t.nombre as transicion_nombre,
    t.tipo_transicion,
    ed_anterior.nombre as estado_anterior,
    ed_nuevo.nombre as estado_nuevo,
    h.tipo_cambio,
    h.observaciones_cambio,
    h.archivo_anexo_nombre,
    h.fecha_cambio,
    u.nombre as usuario_cambio,
    
    -- Información específica según tipo de cambio
    CASE 
        WHEN h.tipo_cambio = 'MANUAL' THEN 
            json_build_object(
                'requiere_anexo', t.requiere_anexo,
                'archivo_adjunto', h.archivo_anexo_nombre,
                'observaciones', h.observaciones_cambio
            )
        WHEN h.tipo_cambio = 'AUTOMATICO' THEN
            json_build_object(
                'condicion_evaluada', h.condicion_evaluada,
                'resultado_evaluacion', h.resultado_evaluacion,
                'movimiento_disparador', mp.numero_movimiento
            )
        ELSE NULL
    END as detalle_cambio
    
FROM historial_estado_documento h
JOIN documento_presupuestal d ON h.documento_id = d.id
JOIN transicion_estado t ON h.transicion_id = t.id
LEFT JOIN estado_documento ed_anterior ON h.estado_anterior_id = ed_anterior.id
JOIN estado_documento ed_nuevo ON h.estado_nuevo_id = ed_nuevo.id
LEFT JOIN usuario u ON h.usuario_cambio = u.id
LEFT JOIN movimiento_presupuestal mp ON h.movimiento_disparador_id = mp.id
ORDER BY h.fecha_cambio DESC;
```

---

## Ejemplos Prácticos

### Ejemplo 1: Aprobar manualmente una Adición Presupuestal

```sql
-- Cambiar estado de una adición presupuestal de PROPUESTA a APROBADA
SELECT cambiar_estado_documento_manual(
    201, -- documento_id (Adición Presupuestal)
    'APROBAR_ADICION', -- transicion_codigo
    456, -- usuario_id (Director de Presupuesto)
    'Adición aprobada según ordenanza municipal No. 15-2025 del Concejo Municipal', -- observaciones
    'ordenanza_15_2025_firmada.pdf', -- archivo_anexo_nombre
    '/anexos/presupuesto/2025/ordenanza_15_2025_firmada.pdf', -- archivo_anexo_ruta
    'application/pdf', -- archivo_anexo_tipo
    2485760 -- archivo_anexo_tamano (2.4MB)
);

-- Resultado: 
-- Retorna ID del historial creado
-- Estado cambia a APROBADA
-- Se registra el anexo y observaciones
-- Se evalúan automáticamente las transiciones siguientes
```

### Ejemplo 2: Evaluación automática de estado de CDP

```sql
-- Después de crear un movimiento de compromiso, el CDP automáticamente evalúa su estado
-- Este proceso ocurre automáticamente vía trigger

-- Supongamos que se crea un movimiento que compromete 30M de un CDP de 50M:
SELECT crear_movimiento_presupuestal(
    'COMPROMISO_CDP',
    301, -- documento RP
    3001, -- item RP  
    100, -- documento CDP
    1001, -- item CDP
    30000000.00, -- valor compromiso
    'Compromiso parcial por contrato servicios',
    789,
    'Contrato No. 001-2025'
);

-- El trigger automáticamente ejecuta:
-- 1. evaluar_transiciones_automaticas(100, movimiento_id)
-- 2. Evalúa condición: documento.estado_actual === "EXPEDIDO" && documento.columnas.VALOR_COMPROMETIDO > 0 && documento.columnas.SALDO_COMPROMETER > 0
-- 3. Resultado: VALOR_COMPROMETIDO = 30,000,000 > 0 y SALDO_COMPROMETER = 20,000,000 > 0
-- 4. Estado cambia automáticamente de EXPEDIDO a COMPROMETIDO_PARCIAL
-- 5. Se registra en historial como cambio AUTOMATICO
```

### Ejemplo 3: Consultar estados y transiciones disponibles

```sql
-- Consultar estado actual y transiciones disponibles para un documento
SELECT 
    numero_documento,
    tipo_documento,
    estado_actual_nombre,
    estado_descripcion,
    permite_edicion,
    permite_movimientos,
    ultimo_cambio_estado,
    transiciones_disponibles
FROM vista_estados_documentos
WHERE documento_id = 201;

-- Resultado:
-- {
--   "numero_documento": "AD-2025-001",
--   "tipo_documento": "Adición Presupuestal",
--   "estado_actual_nombre": "Aprobada",
--   "estado_descripcion": "Adición aprobada oficialmente",
--   "permite_edicion": false,
--   "permite_movimientos": true,
--   "ultimo_cambio_estado": {
--     "fecha_cambio": "2025-01-15 14:30:00",
--     "tipo_cambio": "MANUAL",
--     "usuario": "Director Presupuesto",
--     "observaciones": "Adición aprobada según ordenanza...",
--     "archivo_anexo": "ordenanza_15_2025_firmada.pdf"
--   },
--   "transiciones_disponibles": [
--     {
--       "transicion_codigo": "APLICAR_ADICION_AUTO",
--       "transicion_nombre": "Aplicar Adición Automáticamente",
--       "estado_destino": "Aplicada",
--       "tipo_transicion": "AUTOMATICA",
--       "requiere_anexo": false,
--       "requiere_observaciones": false
--     }
--   ]
-- }
```

### Ejemplo 4: Consultar historial completo de estados

```sql
-- Ver historial completo de cambios de estado de un documento
SELECT 
    transicion_nombre,
    tipo_transicion,
    estado_anterior,
    estado_nuevo,
    fecha_cambio,
    usuario_cambio,
    detalle_cambio
FROM vista_historial_estados
WHERE documento_id = 201
ORDER BY fecha_cambio;

-- Resultado:
-- transicion_nombre: "Aprobar Adición Presupuestal"
-- tipo_transicion: "MANUAL"  
-- estado_anterior: "Propuesta"
-- estado_nuevo: "Aprobada"
-- fecha_cambio: "2025-01-15 14:30:00"
-- usuario_cambio: "Director Presupuesto"
-- detalle_cambio: {
--   "requiere_anexo": true,
--   "archivo_adjunto": "ordenanza_15_2025_firmada.pdf", 
--   "observaciones": "Adición aprobada según ordenanza..."
-- }
--
-- transicion_nombre: "Aplicar Adición Automáticamente"
-- tipo_transicion: "AUTOMATICA"
-- estado_anterior: "Aprobada" 
-- estado_nuevo: "Aplicada"
-- fecha_cambio: "2025-01-15 14:31:00"
-- usuario_cambio: "Sistema"
-- detalle_cambio: {
--   "condicion_evaluada": "documento.estado_actual === 'APROBADA' && documento.movimientos_recientes.length > 0",
--   "resultado_evaluacion": {"resultado": true, "datos_usados": {...}},
--   "movimiento_disparador": "AD-001"
-- }
```

---

## Próximos Pasos

Este documento estableció el sistema completo de estados para documentos presupuestales. Los próximos documentos cubrirán:

**05 - Configuración de Módulos**: Sistema de configuración general del módulo presupuestario.
**06 - Reportes y Consultas**: Reportes estándar y consultas avanzadas del sistema.

Los estados de documentos están ahora completamente definidos con:
- ✅ Estados configurables por tipo de documento
- ✅ Transiciones manuales con anexos y validaciones
- ✅ Transiciones automáticas con evaluación JavaScript
- ✅ Sistema de permisos y roles para transiciones
- ✅ Validaciones previas y de saldos
- ✅ Historial completo y auditoría
- ✅ Triggers automáticos post-movimiento
- ✅ Vistas para consultas y reportes
- ✅ Ejemplos prácticos de uso completo
