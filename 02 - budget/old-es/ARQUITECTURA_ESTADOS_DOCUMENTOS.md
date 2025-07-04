# Arquitectura de Estados de Documentos Presupuestales

## Descripción General

Este documento especifica la arquitectura y el sistema de gestión de estados para los documentos presupuestales en SIIAFE, definiendo tanto los cambios de estado automáticos (basados en valores de columnas) como los cambios manuales (iniciados por usuarios).

## Conceptos Fundamentales

### Estados de Documentos
Los estados representan las diferentes fases del ciclo de vida de un documento presupuestal, desde su creación hasta su finalización. Cada estado determina qué operaciones son permitidas sobre el documento.

### Tipos de Cambios de Estado

#### 1. **Cambios Automáticos**
- Ocurren después de movimientos presupuestales que modifican valores de columnas
- Se evalúan mediante reglas configurables basadas en condiciones
- Son disparados por el sistema sin intervención humana

#### 2. **Cambios Manuales**
- Requieren acción explícita de un usuario autorizado
- Pueden requerir anexos de prueba o documentación de soporte
- Representan hitos administrativos o legales del proceso

---

## Modelo de Datos de Estados

### Tabla: tipo_documento_estado

```sql
CREATE TABLE tipo_documento_estado (
    id SERIAL PRIMARY KEY,
    tipo_documento_id INTEGER NOT NULL REFERENCES tipo_documento(id),
    codigo_estado VARCHAR(50) NOT NULL,
    nombre_estado VARCHAR(100) NOT NULL,
    descripcion TEXT,
    es_inicial BOOLEAN DEFAULT FALSE,
    es_final BOOLEAN DEFAULT FALSE,
    es_automatico BOOLEAN DEFAULT FALSE, -- Si el estado solo se alcanza automáticamente
    requiere_anexos BOOLEAN DEFAULT FALSE,
    color_interfaz VARCHAR(20), -- Para representación visual
    orden_presentacion INTEGER DEFAULT 1,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(tipo_documento_id, codigo_estado)
);
```

### Tabla: transicion_estado

```sql
CREATE TABLE transicion_estado (
    id SERIAL PRIMARY KEY,
    estado_origen_id INTEGER NOT NULL REFERENCES tipo_documento_estado(id),
    estado_destino_id INTEGER NOT NULL REFERENCES tipo_documento_estado(id),
    tipo_transicion VARCHAR(20) NOT NULL, -- 'MANUAL', 'AUTOMATICA'
    requiere_autorizacion BOOLEAN DEFAULT FALSE,
    requiere_anexos BOOLEAN DEFAULT FALSE,
    descripcion_transicion TEXT,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(estado_origen_id, estado_destino_id)
);
```

### Tabla: condicion_estado_automatico

```sql
CREATE TABLE condicion_estado_automatico (
    id SERIAL PRIMARY KEY,
    transicion_estado_id INTEGER NOT NULL REFERENCES transicion_estado(id),
    nombre_condicion VARCHAR(100) NOT NULL,
    descripcion_condicion TEXT,
    codigo_javascript TEXT NOT NULL, -- Código JS que evalúa la condición
    version_codigo INTEGER DEFAULT 1,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    creado_por INTEGER REFERENCES usuario(id),
    
    UNIQUE(transicion_estado_id, nombre_condicion)
);
```

### Tabla: historial_estado_documento

```sql
CREATE TABLE historial_estado_documento (
    id SERIAL PRIMARY KEY,
    documento_id INTEGER NOT NULL REFERENCES documento_presupuestal(id),
    estado_anterior_id INTEGER REFERENCES tipo_documento_estado(id),
    estado_nuevo_id INTEGER NOT NULL REFERENCES tipo_documento_estado(id),
    tipo_cambio VARCHAR(20) NOT NULL, -- 'MANUAL', 'AUTOMATICO'
    usuario_id INTEGER, -- Solo para cambios manuales
    movimiento_id INTEGER REFERENCES movimiento_presupuestal(id), -- Solo para cambios automáticos
    observaciones TEXT,
    anexos_adjuntos JSON, -- Información de archivos adjuntos
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_usuario INET,
    datos_adicionales JSON
);
```

---

## Configuración de Estados por Tipo de Documento

### Estados del Presupuesto de Gasto (PG)

```sql
-- Estados del PG
INSERT INTO tipo_documento_estado VALUES
(1, 1, 'BORRADOR', 'Borrador', 'Presupuesto en elaboración', true, false, false, false, '#FFC107', 1, true, NOW(), NOW()),
(2, 1, 'APROBADO', 'Aprobado', 'Presupuesto aprobado y vigente', false, false, false, false, '#4CAF50', 2, true, NOW(), NOW()),
(3, 1, 'MODIFICADO', 'Modificado', 'Presupuesto con modificaciones', false, false, true, false, '#FF9800', 3, true, NOW(), NOW()),
(4, 1, 'CERRADO', 'Cerrado', 'Presupuesto cerrado para la vigencia', false, true, false, false, '#9E9E9E', 4, true, NOW(), NOW());

-- Transiciones del PG
INSERT INTO transicion_estado VALUES
(1, 1, 2, 'MANUAL', true, true, 'Aprobación del presupuesto por autoridad competente', true, NOW()),
(2, 2, 3, 'AUTOMATICA', false, false, 'Cambio automático por modificaciones presupuestales', true, NOW()),
(3, 3, 4, 'MANUAL', true, false, 'Cierre manual de vigencia presupuestal', true, NOW());
```

### Estados del CDP

```sql
-- Estados del CDP
INSERT INTO tipo_documento_estado VALUES
(5, 5, 'BORRADOR', 'Borrador', 'CDP en elaboración', true, false, false, false, '#FFC107', 1, true, NOW(), NOW()),
(6, 5, 'EXPEDIDO', 'Expedido', 'CDP impreso, firmado y expedido', false, false, false, true, '#2196F3', 2, true, NOW(), NOW()),
(7, 5, 'COMPROMETIDO_PARCIAL', 'Comprometido Parcial', 'CDP con compromisos parciales', false, false, true, false, '#FF9800', 3, true, NOW(), NOW()),
(8, 5, 'COMPROMETIDO_TOTAL', 'Comprometido Total', 'CDP totalmente comprometido', false, false, true, false, '#9C27B0', 4, true, NOW(), NOW()),
(9, 5, 'LIBERADO_PARCIAL', 'Liberado Parcial', 'CDP con liberaciones parciales', false, false, true, false, '#00BCD4', 5, true, NOW(), NOW()),
(10, 5, 'LIBERADO_TOTAL', 'Liberado Total', 'CDP totalmente liberado', false, true, true, false, '#607D8B', 6, true, NOW(), NOW()),
(11, 5, 'ANULADO', 'Anulado', 'CDP anulado', false, true, false, true, '#F44336', 7, true, NOW(), NOW());

-- Transiciones del CDP
INSERT INTO transicion_estado VALUES
-- Transiciones manuales
(4, 5, 6, 'MANUAL', false, true, 'Expedición manual del CDP (requiere impresión y firma)', true, NOW()),
(5, 6, 11, 'MANUAL', true, true, 'Anulación del CDP expedido', true, NOW()),
(6, 7, 11, 'MANUAL', true, true, 'Anulación del CDP parcialmente comprometido', true, NOW()),

-- Transiciones automáticas
(7, 6, 7, 'AUTOMATICA', false, false, 'Cambio automático a comprometido parcial', true, NOW()),
(8, 6, 8, 'AUTOMATICA', false, false, 'Cambio automático a comprometido total', true, NOW()),
(9, 7, 8, 'AUTOMATICA', false, false, 'Cambio automático de parcial a total', true, NOW()),
(10, 7, 9, 'AUTOMATICA', false, false, 'Cambio automático a liberado parcial', true, NOW()),
(11, 8, 9, 'AUTOMATICA', false, false, 'Cambio automático a liberado parcial desde total', true, NOW()),
(12, 9, 10, 'AUTOMATICA', false, false, 'Cambio automático a liberado total', true, NOW());
```

---

## Condiciones Automáticas de Estado con JavaScript

### Condiciones para CDP

```sql
-- Condición para EXPEDIDO -> COMPROMETIDO_PARCIAL
-- Si: saldo_comprometer > 0 AND valor_comprometido > 0
INSERT INTO condicion_estado_automatico VALUES
(1, 7, 'comprometido_parcial', 'Cambio a comprometido parcial cuando hay compromisos pero queda saldo',
'
// Función que evalúa si el CDP debe cambiar a COMPROMETIDO_PARCIAL
function evaluar(columnas, estadoActual) {
    // Valores disponibles en el objeto columnas:
    // columnas.saldo_comprometer, columnas.valor_comprometido, etc.
    
    if (estadoActual !== "EXPEDIDO") {
        return false;
    }
    
    return columnas.saldo_comprometer > 0 && columnas.valor_comprometido > 0;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);

-- Condición para EXPEDIDO -> COMPROMETIDO_TOTAL
-- Si: saldo_comprometer = 0 AND liberado = 0 AND valor_comprometido > 0
INSERT INTO condicion_estado_automatico VALUES
(2, 8, 'comprometido_total', 'Cambio a comprometido total cuando no queda saldo y no hay liberaciones',
'
function evaluar(columnas, estadoActual) {
    if (estadoActual !== "EXPEDIDO") {
        return false;
    }
    
    return columnas.saldo_comprometer === 0 && 
           columnas.liberado === 0 && 
           columnas.valor_comprometido > 0;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);

-- Condición para COMPROMETIDO_PARCIAL -> COMPROMETIDO_TOTAL
-- Si: saldo_comprometer = 0 AND liberado = 0
INSERT INTO condicion_estado_automatico VALUES
(3, 9, 'parcial_a_total', 'Cambio de parcial a total cuando se agota el saldo',
'
function evaluar(columnas, estadoActual) {
    if (estadoActual !== "COMPROMETIDO_PARCIAL") {
        return false;
    }
    
    return columnas.saldo_comprometer === 0 && columnas.liberado === 0;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);

-- Condición para COMPROMETIDO_PARCIAL -> LIBERADO_PARCIAL
-- Si: liberado > 0 AND valor_comprometido > 0
INSERT INTO condicion_estado_automatico VALUES
(4, 10, 'liberacion_parcial', 'Cambio a liberado parcial cuando hay liberaciones pero quedan compromisos',
'
function evaluar(columnas, estadoActual) {
    const estadosPermitidos = ["COMPROMETIDO_PARCIAL", "COMPROMETIDO_TOTAL"];
    
    if (!estadosPermitidos.includes(estadoActual)) {
        return false;
    }
    
    return columnas.liberado > 0 && columnas.valor_comprometido > 0;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);

-- Condición para LIBERADO_TOTAL
-- Si: valor_comprometido = 0 AND liberado = valor_inicial
INSERT INTO condicion_estado_automatico VALUES
(5, 12, 'liberacion_total', 'Cambio a liberado total cuando todo está liberado',
'
function evaluar(columnas, estadoActual) {
    const estadosPermitidos = ["LIBERADO_PARCIAL", "COMPROMETIDO_PARCIAL", "COMPROMETIDO_TOTAL"];
    
    if (!estadosPermitidos.includes(estadoActual)) {
        return false;
    }
    
    return columnas.valor_comprometido === 0 && 
           columnas.liberado === columnas.valor_inicial;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);
```

### Condiciones Complejas para Presupuesto de Gasto

```sql
-- Condición para PG: APROBADO -> MODIFICADO
-- Cuando hay adiciones, recortes o traslados
INSERT INTO condicion_estado_automatico VALUES
(6, 2, 'presupuesto_modificado', 'PG pasa a modificado cuando hay cambios presupuestales',
'
function evaluar(columnas, estadoActual) {
    if (estadoActual !== "APROBADO") {
        return false;
    }
    
    // Si hay cualquier tipo de modificación presupuestal
    return columnas.adiciones > 0 || 
           columnas.recortes > 0 || 
           Math.abs(columnas.traslados) > 0;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);
```

### Condiciones para RP

```sql
-- Condición para RP: EXPEDIDO -> OBLIGADO_PARCIAL
INSERT INTO condicion_estado_automatico VALUES
(7, 15, 'rp_obligado_parcial', 'RP pasa a obligado parcial cuando hay obligaciones pero queda saldo',
'
function evaluar(columnas, estadoActual) {
    if (estadoActual !== "EXPEDIDO") {
        return false;
    }
    
    return columnas.obligaciones > 0 && columnas.saldo_obligar > 0;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);

-- Condición para RP: OBLIGADO_TOTAL
INSERT INTO condicion_estado_automatico VALUES
(8, 16, 'rp_obligado_total', 'RP pasa a obligado total cuando no queda saldo por obligar',
'
function evaluar(columnas, estadoActual) {
    const estadosPermitidos = ["EXPEDIDO", "OBLIGADO_PARCIAL"];
    
    if (!estadosPermitidos.includes(estadoActual)) {
        return false;
    }
    
    return columnas.saldo_obligar === 0 && 
           columnas.liquidaciones === 0 && 
           columnas.obligaciones > 0;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);
```

---

## Funciones de Procesamiento de Estados con JavaScript

### Función: evaluar_cambio_estado_automatico

```sql
CREATE OR REPLACE FUNCTION evaluar_cambio_estado_automatico(
    p_documento_id INTEGER,
    p_movimiento_id INTEGER DEFAULT NULL
)
RETURNS BOOLEAN AS $$
DECLARE
    v_documento RECORD;
    v_estado_actual_id INTEGER;
    v_transicion RECORD;
    v_cumple_condiciones BOOLEAN;
    v_nuevo_estado_id INTEGER;
    v_columnas_json JSON;
BEGIN
    -- Obtener información del documento
    SELECT d.*, tde.id as estado_actual_id, tde.codigo_estado as estado_codigo
    INTO v_documento
    FROM documento_presupuestal d
    JOIN tipo_documento_estado tde ON d.estado = tde.codigo_estado AND tde.tipo_documento_id = d.tipo_documento_id
    WHERE d.id = p_documento_id;
    
    v_estado_actual_id := v_documento.estado_actual_id;
    
    -- Obtener valores actuales de todas las columnas del documento
    v_columnas_json := obtener_columnas_documento_json(p_documento_id);
    
    -- Evaluar todas las transiciones automáticas posibles desde el estado actual
    FOR v_transicion IN 
        SELECT te.*, tde_destino.codigo_estado as estado_destino_codigo
        FROM transicion_estado te
        JOIN tipo_documento_estado tde_destino ON te.estado_destino_id = tde_destino.id
        WHERE te.estado_origen_id = v_estado_actual_id 
          AND te.tipo_transicion = 'AUTOMATICA'
          AND te.es_activo = true
        ORDER BY te.id
    LOOP
        -- Evaluar condiciones JavaScript para esta transición
        v_cumple_condiciones := evaluar_condiciones_javascript(
            v_transicion.id,
            v_columnas_json,
            v_documento.estado_codigo
        );
        
        -- Si cumple las condiciones, realizar el cambio de estado
        IF v_cumple_condiciones THEN
            v_nuevo_estado_id := v_transicion.estado_destino_id;
            
            -- Ejecutar cambio de estado
            PERFORM cambiar_estado_documento(
                p_documento_id,
                v_nuevo_estado_id,
                'AUTOMATICO',
                NULL, -- usuario_id
                p_movimiento_id,
                'Cambio automático por condiciones JavaScript: ' || v_transicion.estado_destino_codigo
            );
            
            RETURN TRUE;
        END IF;
    END LOOP;
    
    RETURN FALSE;
END;
$$ LANGUAGE plpgsql;
```

### Función: obtener_columnas_documento_json

```sql
CREATE OR REPLACE FUNCTION obtener_columnas_documento_json(
    p_documento_id INTEGER
)
RETURNS JSON AS $$
DECLARE
    v_columnas JSON;
BEGIN
    -- Construir objeto JSON con todas las columnas del documento
    SELECT json_object_agg(
        tdc.nombre_columna, 
        COALESCE(idcv.valor_columna, tdc.valor_defecto)
    ) INTO v_columnas
    FROM item_documento_presupuestal idp
    JOIN documento_presupuestal d ON idp.documento_id = d.id
    JOIN tipo_documento_columnas tdc ON tdc.tipo_documento_id = d.tipo_documento_id
    LEFT JOIN item_documento_columna_valor idcv ON 
        idp.id = idcv.item_documento_id AND 
        idcv.tipo_documento_columna_id = tdc.id
    WHERE idp.documento_id = p_documento_id
      AND tdc.es_activo = true
    GROUP BY idp.id;
    
    RETURN COALESCE(v_columnas, '{}'::json);
END;
$$ LANGUAGE plpgsql;
```

### Función: evaluar_condiciones_javascript

```sql
CREATE OR REPLACE FUNCTION evaluar_condiciones_javascript(
    p_transicion_id INTEGER,
    p_columnas_json JSON,
    p_estado_actual VARCHAR(50)
)
RETURNS BOOLEAN AS $$
DECLARE
    v_condicion RECORD;
    v_resultado BOOLEAN := FALSE;
BEGIN
    -- Evaluar todas las condiciones JavaScript para esta transición
    FOR v_condicion IN 
        SELECT * FROM condicion_estado_automatico
        WHERE transicion_estado_id = p_transicion_id 
          AND es_activo = true
        ORDER BY id
    LOOP
        -- Evaluar código JavaScript usando la extensión plv8
        BEGIN
            SELECT plv8_execute_js(
                v_condicion.codigo_javascript,
                p_columnas_json,
                p_estado_actual
            ) INTO v_resultado;
            
            -- Si cualquier condición se cumple, retornar true
            IF v_resultado THEN
                RETURN TRUE;
            END IF;
            
        EXCEPTION
            WHEN OTHERS THEN
                -- Log del error y continuar con la siguiente condición
                INSERT INTO log_errores_estados 
                    (documento_id, transicion_id, condicion_id, error_mensaje, fecha_error)
                VALUES 
                    (NULL, p_transicion_id, v_condicion.id, SQLERRM, NOW());
                CONTINUE;
        END;
    END LOOP;
    
    RETURN FALSE;
END;
$$ LANGUAGE plpgsql;
```

### Función Auxiliar: plv8_execute_js

```sql
-- Función que ejecuta JavaScript usando plv8 (requiere extensión plv8)
CREATE OR REPLACE FUNCTION plv8_execute_js(
    p_codigo_js TEXT,
    p_columnas JSON,
    p_estado_actual VARCHAR(50)
)
RETURNS BOOLEAN AS $$
    // Crear contexto para la evaluación
    var columnas = p_columnas;
    var estadoActual = p_estado_actual;
    
    // Funciones utilitarias disponibles en el contexto
    function log(mensaje) {
        plv8.elog(NOTICE, "JS Estado: " + mensaje);
    }
    
    function redondear(valor, decimales = 2) {
        return Math.round(valor * Math.pow(10, decimales)) / Math.pow(10, decimales);
    }
    
    function esNulo(valor) {
        return valor === null || valor === undefined;
    }
    
    function porcentaje(parte, total) {
        return total > 0 ? (parte / total) * 100 : 0;
    }
    
    // Evaluar el código JavaScript proporcionado
    try {
        return eval(p_codigo_js);
    } catch (error) {
        log("Error en evaluación JS: " + error.message);
        return false;
    }
$$ LANGUAGE plv8;
```

### Tabla de Log de Errores

```sql
CREATE TABLE log_errores_estados (
    id SERIAL PRIMARY KEY,
    documento_id INTEGER,
    transicion_id INTEGER REFERENCES transicion_estado(id),
    condicion_id INTEGER REFERENCES condicion_estado_automatico(id),
    error_mensaje TEXT,
    fecha_error TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    codigo_javascript TEXT,
    columnas_contexto JSON
);
```

### Función: cambiar_estado_documento

```sql
CREATE OR REPLACE FUNCTION cambiar_estado_documento(
    p_documento_id INTEGER,
    p_nuevo_estado_id INTEGER,
    p_tipo_cambio VARCHAR(20),
    p_usuario_id INTEGER DEFAULT NULL,
    p_movimiento_id INTEGER DEFAULT NULL,
    p_observaciones TEXT DEFAULT NULL,
    p_anexos JSON DEFAULT NULL
)
RETURNS BOOLEAN AS $$
DECLARE
    v_estado_anterior_id INTEGER;
    v_codigo_nuevo_estado VARCHAR(50);
BEGIN
    -- Obtener estado actual
    SELECT tde.id INTO v_estado_anterior_id
    FROM documento_presupuestal d
    JOIN tipo_documento_estado tde ON d.estado = tde.codigo_estado AND tde.tipo_documento_id = d.tipo_documento_id
    WHERE d.id = p_documento_id;
    
    -- Obtener código del nuevo estado
    SELECT codigo_estado INTO v_codigo_nuevo_estado
    FROM tipo_documento_estado
    WHERE id = p_nuevo_estado_id;
    
    -- Actualizar estado del documento
    UPDATE documento_presupuestal 
    SET estado = v_codigo_nuevo_estado,
        actualizado_en = NOW()
    WHERE id = p_documento_id;
    
    -- Registrar en historial
    INSERT INTO historial_estado_documento 
        (documento_id, estado_anterior_id, estado_nuevo_id, tipo_cambio, 
         usuario_id, movimiento_id, observaciones, anexos_adjuntos, fecha_cambio)
    VALUES 
        (p_documento_id, v_estado_anterior_id, p_nuevo_estado_id, p_tipo_cambio,
         p_usuario_id, p_movimiento_id, p_observaciones, p_anexos, NOW());
    
    RETURN TRUE;
END;
$$ LANGUAGE plpgsql;
```

---

## Ejemplos Avanzados de Condiciones JavaScript

### Condiciones Complejas para CDP

```sql
-- Condición avanzada para manejo de múltiples escenarios
INSERT INTO condicion_estado_automatico VALUES
(20, 7, 'cdp_logica_compleja', 'Lógica compleja para estados de CDP con múltiples validaciones',
'
function evaluar(columnas, estadoActual) {
    // Funciones auxiliares
    function calcularPorcentajeComprometido() {
        return columnas.valor_inicial > 0 ? 
            (columnas.valor_comprometido / columnas.valor_inicial) * 100 : 0;
    }
    
    function hayMovimientosRecientes() {
        // En un caso real, podríamos recibir información de fechas
        return true; // Simplificado para el ejemplo
    }
    
    // Lógica principal
    switch (estadoActual) {
        case "EXPEDIDO":
            // Solo puede pasar a comprometido si hay valor comprometido
            if (columnas.valor_comprometido <= 0) {
                return false;
            }
            
            // Determinar si es parcial o total
            const porcentajeComprometido = calcularPorcentajeComprometido();
            
            if (porcentajeComprometido >= 100 && columnas.liberado === 0) {
                return "COMPROMETIDO_TOTAL";
            } else if (porcentajeComprometido > 0) {
                return "COMPROMETIDO_PARCIAL";
            }
            break;
            
        case "COMPROMETIDO_PARCIAL":
            // Verificar si debe pasar a total
            if (columnas.saldo_comprometer === 0 && columnas.liberado === 0) {
                return "COMPROMETIDO_TOTAL";
            }
            
            // Verificar si hay liberaciones
            if (columnas.liberado > 0) {
                return "LIBERADO_PARCIAL";
            }
            break;
            
        default:
            return false;
    }
    
    return false;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);
```

### Condiciones con Validaciones de Negocio

```sql
-- Condición para RP con validaciones complejas
INSERT INTO condicion_estado_automatico VALUES
(21, 15, 'rp_validaciones_negocio', 'Validaciones de negocio para RP con reglas específicas',
'
function evaluar(columnas, estadoActual) {
    // Constantes de negocio
    const PORCENTAJE_MINIMO_OBLIGACION = 10; // 10%
    const VALOR_MINIMO_OBLIGACION = 1000000; // $1,000,000
    
    // Funciones de validación
    function validarMontos() {
        const suma = columnas.obligaciones + columnas.liquidaciones + columnas.saldo_obligar;
        const diferencia = Math.abs(suma - columnas.valor_inicial);
        
        // Tolerancia de $1 peso para diferencias de redondeo
        return diferencia <= 1;
    }
    
    function calcularPorcentajeObligado() {
        return columnas.valor_inicial > 0 ? 
            (columnas.obligaciones / columnas.valor_inicial) * 100 : 0;
    }
    
    function puedeOblegar() {
        return columnas.saldo_obligar >= VALOR_MINIMO_OBLIGACION;
    }
    
    // Validar integridad de datos
    if (!validarMontos()) {
        log("Error: Los montos del RP no cuadran. Valor inicial: " + 
            columnas.valor_inicial + ", Suma: " + 
            (columnas.obligaciones + columnas.liquidaciones + columnas.saldo_obligar));
        return false;
    }
    
    // Lógica principal
    if (estadoActual === "EXPEDIDO") {
        const porcentajeObligado = calcularPorcentajeObligado();
        
        if (columnas.obligaciones > 0 && porcentajeObligado >= PORCENTAJE_MINIMO_OBLIGACION) {
            if (columnas.saldo_obligar === 0 || !puedeOblegar()) {
                return "OBLIGADO_TOTAL";
            } else {
                return "OBLIGADO_PARCIAL";
            }
        }
    }
    
    return false;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);
```

### Condición con Cálculos Temporales

```sql
-- Condición que considera fechas y períodos
INSERT INTO condicion_estado_automatico VALUES
(22, 3, 'pg_cierre_automatico', 'Cierre automático de PG por fechas y ejecución',
'
function evaluar(columnas, estadoActual, contexto) {
    // En un caso real, recibiríamos información adicional en contexto
    // como fechas, configuración de vigencia, etc.
    
    const fechaActual = new Date();
    const finVigencia = new Date(fechaActual.getFullYear(), 11, 31); // 31 de diciembre
    const diasParaFinVigencia = Math.ceil((finVigencia - fechaActual) / (1000 * 60 * 60 * 24));
    
    function calcularPorcentajeEjecucion() {
        return columnas.presupuesto_definitivo > 0 ? 
            (columnas.valor_reservado / columnas.presupuesto_definitivo) * 100 : 0;
    }
    
    function esFinalDeVigencia() {
        return diasParaFinVigencia <= 30; // Últimos 30 días
    }
    
    if (estadoActual === "MODIFICADO") {
        const porcentajeEjecucion = calcularPorcentajeEjecucion();
        
        // Reglas para cierre automático
        if (esFinalDeVigencia() && porcentajeEjecucion >= 98) {
            log("Iniciando cierre automático: " + porcentajeEjecucion + "% ejecutado, " + 
                diasParaFinVigencia + " días para fin de vigencia");
            return "CERRADO";
        }
        
        // Cierre por ejecución completa
        if (columnas.saldo_apropiacion === 0 && porcentajeEjecucion >= 99.9) {
            log("Cierre por ejecución completa: " + porcentajeEjecucion + "%");
            return "CERRADO";
        }
    }
    
    return false;
}

evaluar(columnas, estadoActual, contexto);
', 1, true, NOW(), NOW(), 1);
```

---

## Herramientas de Desarrollo y Testing

### Función de Testing de Condiciones

```sql
CREATE OR REPLACE FUNCTION test_condicion_javascript(
    p_condicion_id INTEGER,
    p_columnas_test JSON,
    p_estado_test VARCHAR(50)
)
RETURNS JSON AS $$
DECLARE
    v_condicion RECORD;
    v_resultado BOOLEAN;
    v_tiempo_inicio TIMESTAMP;
    v_tiempo_fin TIMESTAMP;
    v_duracion_ms INTEGER;
BEGIN
    v_tiempo_inicio := clock_timestamp();
    
    -- Obtener la condición
    SELECT * INTO v_condicion
    FROM condicion_estado_automatico
    WHERE id = p_condicion_id;
    
    IF NOT FOUND THEN
        RETURN json_build_object(
            'error', 'Condición no encontrada',
            'condicion_id', p_condicion_id
        );
    END IF;
    
    -- Ejecutar la condición
    BEGIN
        SELECT plv8_execute_js(
            v_condicion.codigo_javascript,
            p_columnas_test,
            p_estado_test
        ) INTO v_resultado;
        
        v_tiempo_fin := clock_timestamp();
        v_duracion_ms := EXTRACT(MILLISECONDS FROM v_tiempo_fin - v_tiempo_inicio)::INTEGER;
        
        RETURN json_build_object(
            'condicion_id', p_condicion_id,
            'nombre_condicion', v_condicion.nombre_condicion,
            'resultado', v_resultado,
            'tiempo_ejecucion_ms', v_duracion_ms,
            'columnas_entrada', p_columnas_test,
            'estado_entrada', p_estado_test,
            'success', true
        );
        
    EXCEPTION
        WHEN OTHERS THEN
            v_tiempo_fin := clock_timestamp();
            v_duracion_ms := EXTRACT(MILLISECONDS FROM v_tiempo_fin - v_tiempo_inicio)::INTEGER;
            
            RETURN json_build_object(
                'condicion_id', p_condicion_id,
                'nombre_condicion', v_condicion.nombre_condicion,
                'error', SQLERRM,
                'tiempo_ejecucion_ms', v_duracion_ms,
                'columnas_entrada', p_columnas_test,
                'estado_entrada', p_estado_test,
                'success', false
            );
    END;
END;
$$ LANGUAGE plpgsql;
```

### Suite de Tests Automatizados

```sql
-- Tabla para casos de prueba
CREATE TABLE casos_prueba_estados (
    id SERIAL PRIMARY KEY,
    nombre_caso VARCHAR(100) NOT NULL,
    descripcion TEXT,
    tipo_documento_id INTEGER NOT NULL,
    condicion_id INTEGER REFERENCES condicion_estado_automatico(id),
    columnas_entrada JSON NOT NULL,
    estado_entrada VARCHAR(50) NOT NULL,
    resultado_esperado BOOLEAN NOT NULL,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Casos de prueba para CDP
INSERT INTO casos_prueba_estados VALUES
(1, 'CDP_Comprometido_Parcial_Basico', 'Caso básico de CDP con compromiso parcial', 5, 1,
'{"valor_inicial": 100000000, "valor_comprometido": 60000000, "saldo_comprometer": 40000000, "liberado": 0}',
'EXPEDIDO', true, true, NOW()),

(2, 'CDP_Comprometido_Total_Basico', 'Caso básico de CDP comprometido totalmente', 5, 2,
'{"valor_inicial": 100000000, "valor_comprometido": 100000000, "saldo_comprometer": 0, "liberado": 0}',
'EXPEDIDO', true, true, NOW()),

(3, 'CDP_Sin_Compromisos', 'CDP sin compromisos no debe cambiar estado', 5, 1,
'{"valor_inicial": 100000000, "valor_comprometido": 0, "saldo_comprometer": 100000000, "liberado": 0}',
'EXPEDIDO', false, true, NOW());

-- Función para ejecutar suite de tests
CREATE OR REPLACE FUNCTION ejecutar_tests_estados(
    p_tipo_documento_id INTEGER DEFAULT NULL
)
RETURNS TABLE (
    caso_id INTEGER,
    nombre_caso VARCHAR(100),
    resultado_obtenido BOOLEAN,
    resultado_esperado BOOLEAN,
    test_passed BOOLEAN,
    tiempo_ms INTEGER,
    error_mensaje TEXT
) AS $$
DECLARE
    v_caso RECORD;
    v_resultado_test JSON;
BEGIN
    FOR v_caso IN 
        SELECT * FROM casos_prueba_estados
        WHERE (p_tipo_documento_id IS NULL OR tipo_documento_id = p_tipo_documento_id)
          AND es_activo = true
        ORDER BY id
    LOOP
        -- Ejecutar test
        SELECT test_condicion_javascript(
            v_caso.condicion_id,
            v_caso.columnas_entrada,
            v_caso.estado_entrada
        ) INTO v_resultado_test;
        
        -- Retornar resultado
        RETURN QUERY SELECT
            v_caso.id,
            v_caso.nombre_caso,
            (v_resultado_test->>'resultado')::BOOLEAN,
            v_caso.resultado_esperado,
            (v_resultado_test->>'resultado')::BOOLEAN = v_caso.resultado_esperado,
            (v_resultado_test->>'tiempo_ejecucion_ms')::INTEGER,
            v_resultado_test->>'error';
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

### Función de Depuración

```sql
CREATE OR REPLACE FUNCTION debug_evaluacion_estados(
    p_documento_id INTEGER,
    p_incluir_todas_transiciones BOOLEAN DEFAULT FALSE
)
RETURNS JSON AS $$
DECLARE
    v_documento RECORD;
    v_columnas_json JSON;
    v_transiciones JSON[];
    v_transicion RECORD;
    v_resultado_condicion JSON;
    v_debug_info JSON;
BEGIN
    -- Obtener información del documento
    SELECT d.*, tde.codigo_estado
    INTO v_documento
    FROM documento_presupuestal d
    JOIN tipo_documento_estado tde ON d.estado = tde.codigo_estado AND tde.tipo_documento_id = d.tipo_documento_id
    WHERE d.id = p_documento_id;
    
    -- Obtener columnas actuales
    v_columnas_json := obtener_columnas_documento_json(p_documento_id);
    
    -- Evaluar cada transición posible
    FOR v_transicion IN 
        SELECT te.*, 
               tde_origen.codigo_estado as estado_origen,
               tde_destino.codigo_estado as estado_destino,
               cea.id as condicion_id,
               cea.nombre_condicion,
               cea.codigo_javascript
        FROM transicion_estado te
        JOIN tipo_documento_estado tde_origen ON te.estado_origen_id = tde_origen.id
        JOIN tipo_documento_estado tde_destino ON te.estado_destino_id = tde_destino.id
        LEFT JOIN condicion_estado_automatico cea ON te.id = cea.transicion_estado_id
        WHERE (p_incluir_todas_transiciones OR tde_origen.codigo_estado = v_documento.codigo_estado)
          AND te.tipo_transicion = 'AUTOMATICA'
          AND tde_origen.tipo_documento_id = v_documento.tipo_documento_id
        ORDER BY te.id, cea.id
    LOOP
        IF v_transicion.condicion_id IS NOT NULL THEN
            -- Evaluar condición específica
            SELECT test_condicion_javascript(
                v_transicion.condicion_id,
                v_columnas_json,
                v_documento.codigo_estado
            ) INTO v_resultado_condicion;
            
            v_transiciones := array_append(v_transiciones, json_build_object(
                'transicion_id', v_transicion.id,
                'estado_origen', v_transicion.estado_origen,
                'estado_destino', v_transicion.estado_destino,
                'condicion_id', v_transicion.condicion_id,
                'nombre_condicion', v_transicion.nombre_condicion,
                'resultado_condicion', v_resultado_condicion
            ));
        END IF;
    END LOOP;
    
    -- Construir respuesta de debug
    v_debug_info := json_build_object(
        'documento_id', p_documento_id,
        'numero_documento', v_documento.numero_documento,
        'tipo_documento_id', v_documento.tipo_documento_id,
        'estado_actual', v_documento.codigo_estado,
        'columnas_actuales', v_columnas_json,
        'transiciones_evaluadas', array_to_json(v_transiciones),
        'fecha_evaluacion', NOW()
    );
    
    RETURN v_debug_info;
END;
$$ LANGUAGE plpgsql;
```

### Función Modificada: procesar_movimiento_presupuestal

```sql
CREATE OR REPLACE FUNCTION procesar_movimiento_presupuestal(
    p_movimiento_id INTEGER,
    p_valor_movimiento DECIMAL(18,2)
)
RETURNS BOOLEAN AS $$
DECLARE
    v_movimiento_tipo_id INTEGER;
    v_documento_origen_id INTEGER;
    v_documento_destino_id INTEGER;
    v_afectacion RECORD;
    v_valor_aplicar DECIMAL(18,2);
    v_item_documento_id INTEGER;
BEGIN
    -- Obtener información del movimiento
    SELECT movimiento_tipo_id, documento_origen_id, documento_destino_id
    INTO v_movimiento_tipo_id, v_documento_origen_id, v_documento_destino_id
    FROM movimiento_presupuestal
    WHERE id = p_movimiento_id;

    -- Procesar cada afectación configurada
    FOR v_afectacion IN 
        SELECT * FROM movimiento_tipo_afectacion mta
        JOIN tipo_documento_columnas tdc ON mta.tipo_documento_columna_id = tdc.id
        WHERE mta.movimiento_tipo_id = v_movimiento_tipo_id 
          AND mta.es_activo = true
        ORDER BY mta.orden_ejecucion
    LOOP
        -- Calcular valor a aplicar
        v_valor_aplicar := p_valor_movimiento * v_afectacion.factor_multiplicador;
        
        -- Determinar el item del documento a afectar
        IF v_afectacion.es_origen AND v_documento_origen_id IS NOT NULL THEN
            -- Buscar item del documento origen
            SELECT id INTO v_item_documento_id
            FROM item_documento_presupuestal
            WHERE documento_id = v_documento_origen_id
            LIMIT 1;
            
            -- Afectar documento origen
            PERFORM actualizar_columna_item_normalizada(
                v_item_documento_id,
                v_afectacion.tipo_documento_columna_id,
                v_afectacion.tipo_afectacion,
                v_valor_aplicar
            );
            
        ELSIF v_afectacion.es_destino AND v_documento_destino_id IS NOT NULL THEN
            -- Buscar item del documento destino
            SELECT id INTO v_item_documento_id
            FROM item_documento_presupuestal
            WHERE documento_id = v_documento_destino_id
            LIMIT 1;
            
            -- Afectar documento destino
            PERFORM actualizar_columna_item_normalizada(
                v_item_documento_id,
                v_afectacion.tipo_documento_columna_id,
                v_afectacion.tipo_afectacion,
                v_valor_aplicar
            );
        END IF;
    END LOOP;

    -- **NUEVA FUNCIONALIDAD: Evaluar cambios de estado automáticos**
    
    -- Evaluar estado del documento origen (si existe)
    IF v_documento_origen_id IS NOT NULL THEN
        PERFORM evaluar_cambio_estado_automatico(v_documento_origen_id, p_movimiento_id);
    END IF;
    
    -- Evaluar estado del documento destino (si existe)
    IF v_documento_destino_id IS NOT NULL THEN
        PERFORM evaluar_cambio_estado_automatico(v_documento_destino_id, p_movimiento_id);
    END IF;

    RETURN TRUE;
END;
$$ LANGUAGE plpgsql;
```

---

## API para Cambios Manuales de Estado

### Función: cambiar_estado_manual

```sql
CREATE OR REPLACE FUNCTION cambiar_estado_manual(
    p_documento_id INTEGER,
    p_nuevo_estado_codigo VARCHAR(50),
    p_usuario_id INTEGER,
    p_observaciones TEXT DEFAULT NULL,
    p_anexos JSON DEFAULT NULL
)
RETURNS JSON AS $$
DECLARE
    v_documento RECORD;
    v_estado_actual_id INTEGER;
    v_nuevo_estado_id INTEGER;
    v_transicion_valida BOOLEAN := FALSE;
    v_resultado JSON;
BEGIN
    -- Obtener información del documento y estado actual
    SELECT d.*, tde.id as estado_actual_id 
    INTO v_documento
    FROM documento_presupuestal d
    JOIN tipo_documento_estado tde ON d.estado = tde.codigo_estado AND tde.tipo_documento_id = d.tipo_documento_id
    WHERE d.id = p_documento_id;
    
    IF NOT FOUND THEN
        RETURN json_build_object(
            'success', false,
            'error', 'Documento no encontrado'
        );
    END IF;
    
    v_estado_actual_id := v_documento.estado_actual_id;
    
    -- Obtener ID del nuevo estado
    SELECT id INTO v_nuevo_estado_id
    FROM tipo_documento_estado
    WHERE codigo_estado = p_nuevo_estado_codigo 
      AND tipo_documento_id = v_documento.tipo_documento_id;
    
    IF NOT FOUND THEN
        RETURN json_build_object(
            'success', false,
            'error', 'Estado destino no válido para este tipo de documento'
        );
    END IF;
    
    -- Verificar si la transición manual es válida
    SELECT true INTO v_transicion_valida
    FROM transicion_estado
    WHERE estado_origen_id = v_estado_actual_id 
      AND estado_destino_id = v_nuevo_estado_id
      AND tipo_transicion = 'MANUAL'
      AND es_activo = true;
    
    IF NOT v_transicion_valida THEN
        RETURN json_build_object(
            'success', false,
            'error', 'Transición de estado no permitida'
        );
    END IF;
    
    -- Ejecutar cambio de estado
    PERFORM cambiar_estado_documento(
        p_documento_id,
        v_nuevo_estado_id,
        'MANUAL',
        p_usuario_id,
        NULL,
        p_observaciones,
        p_anexos
    );
    
    RETURN json_build_object(
        'success', true,
        'message', 'Estado cambiado exitosamente',
        'estado_anterior', v_documento.estado,
        'estado_nuevo', p_nuevo_estado_codigo
    );
    
EXCEPTION
    WHEN OTHERS THEN
        RETURN json_build_object(
            'success', false,
            'error', 'Error interno: ' || SQLERRM
        );
END;
$$ LANGUAGE plpgsql;
```

## Ejemplos Prácticos con JavaScript

### Escenario 1: Testing de Condiciones

```sql
-- 1. Probar condición de CDP comprometido parcial
SELECT test_condicion_javascript(
    1, -- condicion_id
    '{"valor_inicial": 100000000, "valor_comprometido": 60000000, "saldo_comprometer": 40000000, "liberado": 0}'::json,
    'EXPEDIDO'
);

-- Resultado esperado:
-- {
--   "condicion_id": 1,
--   "nombre_condicion": "comprometido_parcial",
--   "resultado": true,
--   "tiempo_ejecucion_ms": 2,
--   "success": true
-- }

-- 2. Ejecutar suite completa de tests para CDP
SELECT * FROM ejecutar_tests_estados(5);

-- Resultado esperado:
-- caso_id | nombre_caso                    | resultado_obtenido | resultado_esperado | test_passed | tiempo_ms
-- --------|--------------------------------|-------------------|-------------------|-------------|----------
-- 1       | CDP_Comprometido_Parcial_Basico| true              | true              | true        | 2
-- 2       | CDP_Comprometido_Total_Basico  | true              | true              | true        | 1
-- 3       | CDP_Sin_Compromisos            | false             | false             | true        | 1
```

### Escenario 2: Desarrollo y Debug de Condiciones

```sql
-- 1. Crear nueva condición JavaScript para OP
INSERT INTO condicion_estado_automatico VALUES
(25, 20, 'op_estado_pagado', 'OP pasa a estado pagado cuando se completa el pago',
'
function evaluar(columnas, estadoActual) {
    // Validar estado actual
    if (!["EXPEDIDA", "PAGADO_PARCIAL"].includes(estadoActual)) {
        return false;
    }
    
    // Calcular porcentaje pagado
    const porcentajePagado = columnas.valor_inicial > 0 ? 
        (columnas.pagos / columnas.valor_inicial) * 100 : 0;
    
    // Log para debugging
    log("OP " + estadoActual + ": " + porcentajePagado.toFixed(2) + "% pagado");
    
    // Determinar nuevo estado
    if (columnas.pagos === columnas.valor_inicial && columnas.anulaciones === 0) {
        return "PAGADO_TOTAL";
    } else if (columnas.pagos > 0 && porcentajePagado >= 10) {
        return "PAGADO_PARCIAL";
    }
    
    return false;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);

-- 2. Probar la nueva condición
SELECT test_condicion_javascript(
    25,
    '{"valor_inicial": 50000000, "pagos": 50000000, "anulaciones": 0, "saldo_pagar": 0}'::json,
    'EXPEDIDA'
);

-- 3. Debug completo de un documento
SELECT debug_evaluacion_estados(100, true);
```

### Escenario 3: Flujo Completo con Estados Automáticos

```sql
-- 1. CDP está en estado EXPEDIDO
-- Columnas iniciales: valor_inicial: $100,000,000, compromisos: $0, saldo_comprometer: $100,000,000

-- 2. Se crea un RP que consume del CDP
INSERT INTO movimiento_presupuestal VALUES 
(200, 'MOV-200', 6, 100, 101, 60000000.00, '2025-07-04', 'Expedición RP-001');

-- 3. Procesar movimiento (automáticamente evalúa cambios de estado con JavaScript)
SELECT procesar_movimiento_presupuestal(200, 60000000.00);

-- El sistema ejecutará automáticamente:
-- a) Actualizar columnas del CDP: compromisos: $60,000,000, saldo_comprometer: $40,000,000
-- b) Evaluar condiciones JavaScript para el CDP
-- c) Ejecutar función JavaScript que retorna true para "COMPROMETIDO_PARCIAL"
-- d) Cambiar estado del CDP de EXPEDIDO a COMPROMETIDO_PARCIAL

-- 4. Verificar el cambio en el historial
SELECT 
    hed.tipo_cambio,
    estado_ant.codigo_estado as estado_anterior,
    estado_nue.codigo_estado as estado_nuevo,
    hed.fecha_cambio,
    hed.observaciones
FROM historial_estado_documento hed
JOIN tipo_documento_estado estado_ant ON hed.estado_anterior_id = estado_ant.id
JOIN tipo_documento_estado estado_nue ON hed.estado_nuevo_id = estado_nue.id
WHERE hed.documento_id = 100
ORDER BY hed.fecha_cambio DESC
LIMIT 1;

-- Resultado:
-- tipo_cambio | estado_anterior | estado_nuevo        | fecha_cambio        | observaciones
-- ------------|-----------------|--------------------|--------------------|------------------
-- AUTOMATICO  | EXPEDIDO        | COMPROMETIDO_PARCIAL| 2025-07-04 10:30:00| Cambio automático por condiciones JavaScript...
```

### Escenario 4: Condición Compleja con Validaciones

```sql
-- Crear condición avanzada para validar integridad antes de cambio de estado
INSERT INTO condicion_estado_automatico VALUES
(30, 12, 'validacion_integridad_cdp', 'Validación completa de integridad antes de cambio de estado',
'
function evaluar(columnas, estadoActual) {
    // Funciones de validación
    function validarIntegridad() {
        const suma = columnas.valor_comprometido + columnas.liberado + columnas.saldo_comprometer;
        const diferencia = Math.abs(suma - columnas.valor_inicial);
        
        if (diferencia > 1) { // Tolerancia de $1 peso
            log("ERROR: Integridad fallida. Inicial: " + columnas.valor_inicial + 
                ", Suma: " + suma + ", Diferencia: " + diferencia);
            return false;
        }
        return true;
    }
    
    function validarRangosValidos() {
        const campos = ["valor_inicial", "valor_comprometido", "liberado", "saldo_comprometer"];
        
        for (let campo of campos) {
            if (columnas[campo] < 0) {
                log("ERROR: Campo " + campo + " no puede ser negativo: " + columnas[campo]);
                return false;
            }
        }
        
        return true;
    }
    
    function validarLogicaNegocio() {
        // El valor comprometido no puede exceder el valor inicial
        if (columnas.valor_comprometido > columnas.valor_inicial) {
            log("ERROR: Valor comprometido excede valor inicial");
            return false;
        }
        
        // El liberado no puede exceder el valor comprometido
        if (columnas.liberado > columnas.valor_comprometido) {
            log("ERROR: Valor liberado excede valor comprometido");
            return false;
        }
        
        return true;
    }
    
    // Ejecutar todas las validaciones
    if (!validarIntegridad() || !validarRangosValidos() || !validarLogicaNegocio()) {
        return false;
    }
    
    // Lógica principal del cambio de estado
    if (estadoActual === "LIBERADO_PARCIAL") {
        return columnas.valor_comprometido === 0 && 
               columnas.liberado === columnas.valor_inicial;
    }
    
    return false;
}

evaluar(columnas, estadoActual);
', 1, true, NOW(), NOW(), 1);

-- Probar con datos válidos
SELECT test_condicion_javascript(
    30,
    '{"valor_inicial": 100000000, "valor_comprometido": 0, "liberado": 100000000, "saldo_comprometer": 0}'::json,
    'LIBERADO_PARCIAL'
);

-- Probar con datos inválidos (debe fallar)
SELECT test_condicion_javascript(
    30,
    '{"valor_inicial": 100000000, "valor_comprometido": 120000000, "liberado": 0, "saldo_comprometer": 0}'::json,
    'LIBERADO_PARCIAL'
);
```

### Escenario 1: Cambio Manual de Estado

```sql
-- 1. Usuario crea un CDP en estado BORRADOR
INSERT INTO documento_presupuestal VALUES 
(100, 'CDP-001', 5, 'BORRADOR', '2025-07-04', 'CDP para contratación', 1, NOW(), NOW());

-- 2. Usuario expide manualmente el CDP (requiere anexos)
SELECT cambiar_estado_manual(
    100, -- documento_id
    'EXPEDIDO', -- nuevo estado
    123, -- usuario_id
    'CDP expedido después de impresión y firma', -- observaciones
    '{"archivo_firmado": "cdp_001_firmado.pdf", "fecha_firma": "2025-07-04T10:30:00Z"}'::json -- anexos
);

-- Resultado: {"success": true, "message": "Estado cambiado exitosamente", "estado_anterior": "BORRADOR", "estado_nuevo": "EXPEDIDO"}
```

### Escenario 2: Cambio Automático de Estado

```sql
-- 1. CDP está en estado EXPEDIDO
-- valor_inicial: $100,000,000, compromisos: $0, saldo_comprometer: $100,000,000

-- 2. Se crea un RP que consume del CDP
INSERT INTO movimiento_presupuestal VALUES 
(200, 'MOV-200', 6, 100, 101, 60000000.00, '2025-07-04', 'Expedición RP-001');

-- 3. Procesar movimiento (automáticamente evalúa cambios de estado)
SELECT procesar_movimiento_presupuestal(200, 60000000.00);

-- Resultado automático:
-- CDP: compromisos: $60,000,000, saldo_comprometer: $40,000,000
-- Estado CDP cambia automáticamente de EXPEDIDO a COMPROMETIDO_PARCIAL

-- 4. Verificar cambio en historial
SELECT 
    hed.tipo_cambio,
    estado_ant.codigo_estado as estado_anterior,
    estado_nue.codigo_estado as estado_nuevo,
    hed.fecha_cambio,
    hed.observaciones
FROM historial_estado_documento hed
JOIN tipo_documento_estado estado_ant ON hed.estado_anterior_id = estado_ant.id
JOIN tipo_documento_estado estado_nue ON hed.estado_nuevo_id = estado_nue.id
WHERE hed.documento_id = 100
ORDER BY hed.fecha_cambio DESC;
```

### Escenario 3: Múltiples Cambios Automáticos

```sql
-- Continuando con el CDP anterior...

-- 1. Se crea otro RP que consume el resto del CDP
INSERT INTO movimiento_presupuestal VALUES 
(201, 'MOV-201', 6, 100, 102, 40000000.00, '2025-07-04', 'Expedición RP-002');

SELECT procesar_movimiento_presupuestal(201, 40000000.00);

-- Resultado automático:
-- CDP: compromisos: $100,000,000, saldo_comprometer: $0, liberado: $0
-- Estado CDP cambia automáticamente de COMPROMETIDO_PARCIAL a COMPROMETIDO_TOTAL

-- 2. Posteriormente se libera parcialmente un RP
INSERT INTO movimiento_presupuestal VALUES 
(202, 'MOV-202', 8, 101, 100, 20000000.00, '2025-07-05', 'Liberación parcial RP-001');

SELECT procesar_movimiento_presupuestal(202, 20000000.00);

-- Resultado automático:
-- CDP: compromisos: $80,000,000, liberado: $20,000,000, saldo_comprometer: $0
-- Estado CDP cambia automáticamente de COMPROMETIDO_TOTAL a LIBERADO_PARCIAL
```

---

## Reportes y Consultas de Estados

### Vista: estado_documentos_consolidado

```sql
CREATE VIEW estado_documentos_consolidado AS
SELECT 
    d.id as documento_id,
    d.numero_documento,
    td.nombre as tipo_documento,
    tde.codigo_estado,
    tde.nombre_estado,
    tde.color_interfaz,
    d.fecha_documento,
    
    -- Información del último cambio de estado
    ultimo_cambio.fecha_cambio as fecha_ultimo_cambio,
    ultimo_cambio.tipo_cambio as tipo_ultimo_cambio,
    usuario.nombre as usuario_ultimo_cambio,
    
    -- Indicadores de estado
    tde.es_inicial,
    tde.es_final,
    tde.es_automatico,
    
    -- Conteos de historial
    (SELECT COUNT(*) FROM historial_estado_documento WHERE documento_id = d.id) as total_cambios_estado
    
FROM documento_presupuestal d
JOIN tipo_documento td ON d.tipo_documento_id = td.id
JOIN tipo_documento_estado tde ON d.estado = tde.codigo_estado AND tde.tipo_documento_id = d.tipo_documento_id
LEFT JOIN LATERAL (
    SELECT * FROM historial_estado_documento hed
    WHERE hed.documento_id = d.id
    ORDER BY hed.fecha_cambio DESC
    LIMIT 1
) ultimo_cambio ON true
LEFT JOIN usuario ON ultimo_cambio.usuario_id = usuario.id;
```

### Consulta: Documentos pendientes de expedición

```sql
-- Documentos en estado BORRADOR que pueden ser expedidos
SELECT 
    d.numero_documento,
    td.nombre as tipo_documento,
    d.fecha_documento,
    EXTRACT(DAYS FROM NOW() - d.fecha_documento) as dias_en_borrador
FROM documento_presupuestal d
JOIN tipo_documento td ON d.tipo_documento_id = td.id
JOIN tipo_documento_estado tde ON d.estado = tde.codigo_estado AND tde.tipo_documento_id = d.tipo_documento_id
WHERE tde.codigo_estado = 'BORRADOR'
  AND EXISTS (
      SELECT 1 FROM transicion_estado te
      JOIN tipo_documento_estado tde_destino ON te.estado_destino_id = tde_destino.id
      WHERE te.estado_origen_id = tde.id 
        AND te.tipo_transicion = 'MANUAL'
        AND tde_destino.codigo_estado = 'EXPEDIDO'
  )
ORDER BY d.fecha_documento;
```

---

## Configuración y Validaciones

### Función: validar_configuracion_estados

```sql
CREATE OR REPLACE FUNCTION validar_configuracion_estados(
    p_tipo_documento_id INTEGER
)
RETURNS TABLE (
    tipo_validacion VARCHAR(50),
    descripcion_error TEXT,
    es_critico BOOLEAN
) AS $$
BEGIN
    -- Validar que existe un estado inicial
    IF NOT EXISTS (
        SELECT 1 FROM tipo_documento_estado 
        WHERE tipo_documento_id = p_tipo_documento_id AND es_inicial = true
    ) THEN
        RETURN QUERY SELECT 'ESTADO_INICIAL', 'No hay estado inicial definido', true;
    END IF;
    
    -- Validar que existe al menos un estado final
    IF NOT EXISTS (
        SELECT 1 FROM tipo_documento_estado 
        WHERE tipo_documento_id = p_tipo_documento_id AND es_final = true
    ) THEN
        RETURN QUERY SELECT 'ESTADO_FINAL', 'No hay estados finales definidos', false;
    END IF;
    
    -- Validar transiciones circulares
    -- (Implementación simplificada)
    
    -- Validar condiciones automáticas sin contradicciones
    -- (Implementación más compleja según reglas de negocio)
    
    RETURN;
END;
$$ LANGUAGE plpgsql;
```

---

## Ventajas de la Arquitectura con JavaScript

### 1. **Flexibilidad Extrema**
- **Lógica expresiva**: JavaScript permite condiciones complejas con sintaxis natural
- **Funciones auxiliares**: Posibilidad de crear funciones de validación y cálculo reutilizables
- **Operaciones matemáticas**: Cálculos avanzados de porcentajes, redondeos, y validaciones
- **Estructuras de control**: Switch, if-else, loops para lógica sofisticada

### 2. **Facilidad de Desarrollo y Mantenimiento**
- **Sintaxis familiar**: JavaScript es ampliamente conocido por desarrolladores
- **Testing integrado**: Funciones de prueba automatizadas con casos de prueba
- **Debug avanzado**: Herramientas de depuración y logging detallado
- **Versionado**: Control de versiones de código de condiciones

### 3. **Rendimiento y Escalabilidad**
- **Ejecución eficiente**: plv8 proporciona excelente rendimiento para JavaScript
- **Evaluación rápida**: Condiciones se evalúan en microsegundos
- **Caching**: Código JavaScript se compila y cachea automáticamente
- **Paralelización**: Evaluaciones pueden ejecutarse en paralelo

### 4. **Robustez y Validación**
- **Manejo de errores**: Try-catch y validaciones comprehensivas
- **Integridad de datos**: Validaciones automáticas de consistencia
- **Logging detallado**: Trazas de ejecución para auditoría y debugging
- **Tolerancia a fallos**: Degradación elegante en caso de errores

### 5. **Capacidades Avanzadas**
- **Validaciones de negocio**: Reglas complejas de sector público
- **Cálculos temporales**: Manejo de fechas y períodos
- **Análisis estadísticos**: Porcentajes, promedios, y métricas
- **Integración con contexto**: Acceso a información adicional del sistema

### 6. **Cumplimiento y Auditoría**
- **Trazabilidad completa**: Cada evaluación queda registrada
- **Historial de cambios**: Versionado de reglas de negocio
- **Reportes detallados**: Análisis de decisiones automáticas
- **Cumplimiento normativo**: Alineado con estándares del sector público

### 7. **Casos de Uso Específicos**

#### **Validaciones Financieras**
```javascript
// Ejemplo: Validar que los montos cuadren
function validarIntegridad(columnas) {
    const tolerancia = 1; // $1 peso
    const suma = columnas.compromisos + columnas.liberado + columnas.saldo;
    return Math.abs(suma - columnas.inicial) <= tolerancia;
}
```

#### **Lógica Temporal**
```javascript
// Ejemplo: Estados basados en fechas
function evaluarPorFecha(columnas, contexto) {
    const diasRestantes = calcularDiasFinVigencia(contexto.fechaActual);
    return diasRestantes <= 30 && columnas.ejecucion >= 95;
}
```

#### **Reglas de Sector Público**
```javascript
// Ejemplo: Cumplimiento de normativas
function cumpleReglamento(columnas, estadoActual) {
    // Decreto 1068 de 2015 - Gestión presupuestal
    const porcentajeEjecucion = (columnas.ejecutado / columnas.aprobado) * 100;
    return porcentajeEjecucion >= 85 || esFinalVigencia();
}
```

### 8. **Comparación con Enfoque Tradicional**

| Aspecto | SQL Tradicional | JavaScript |
|---------|----------------|------------|
| **Expresividad** | Limitada | Alta |
| **Mantenibilidad** | Difícil | Fácil |
| **Testing** | Manual | Automatizado |
| **Debug** | Complejo | Intuitivo |
| **Flexibilidad** | Baja | Muy alta |
| **Validaciones** | Básicas | Comprehensivas |
| **Rendimiento** | Alto | Alto |

Esta arquitectura con JavaScript proporciona un sistema extremadamente flexible, mantenible y robusto para la gestión de estados de documentos presupuestales, superando las limitaciones de los enfoques tradicionales y facilitando el cumplimiento de las complejas normativas del sector público colombiano.
