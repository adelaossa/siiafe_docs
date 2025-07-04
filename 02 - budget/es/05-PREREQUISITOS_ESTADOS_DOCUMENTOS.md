# 05 - Prerequisitos de Estados entre Documentos

## Descripción General

Los **Prerequisitos de Estados entre Documentos** son un sistema de validación que asegura que las operaciones presupuestales sigan el flujo correcto y que el documento inmediatamente precedente esté en el estado adecuado antes de permitir operaciones posteriores. Por ejemplo, una Orden de Pago debe estar "Autorizada por Contabilidad" antes de permitir que se genere un Egreso sobre ella.

## Conceptos Fundamentales

### ¿Qué son los Prerequisitos de Estados?

Los prerequisitos de estados son:

- **Validaciones de flujo**: Garantizan que las operaciones sigan el orden correcto del proceso presupuestario
- **Control de dependencias**: Evitan operaciones sobre documentos en estados inadecuados
- **Reglas de negocio**: Implementan las políticas institucionales sobre flujos presupuestales
- **Auditoría preventiva**: Detectan errores antes de que ocurran las operaciones

### Características del Sistema

Los prerequisitos **SIEMPRE** se validan sobre el **documento inmediatamente precedente**:

- **Solo documento precedente**: Únicamente se valida el documento del cual se deriva directamente
- **Estados requeridos**: El documento precedente debe estar en uno de los estados válidos definidos
- **Sin excepciones**: Los prerequisitos deben cumplirse obligatoriamente, sin posibilidad de bypass
- **Configurabilidad**: Los prerequisitos se definen mediante tablas de configuración
- **Validación automática**: Se ejecuta automáticamente antes de cada operación
- **Mensajes informativos**: Proporciona mensajes claros cuando no se cumplen los prerequisitos

### Flujo de Validación Simplificado

1. **Documento A** (precedente) → **Documento B** (solicitante)
2. Se verifica que el Documento A esté en uno de los estados válidos definidos
3. Si cumple: se permite la operación sobre Documento B
4. Si no cumple: se rechaza con mensaje explicativo

### Ejemplos de Prerequisites

- **RP requiere CDP Expedido**: Para crear un RP, el CDP precedente debe estar "Expedido"
- **OP requiere RP Expedida**: Para crear una OP, el RP precedente debe estar "Expedido"  
- **Egreso requiere OP Autorizada**: Para crear un Egreso, la OP precedente debe estar "Autorizada"

---

## Estructura de Datos

### Tabla Principal: prerequisito_estado_documento

```sql
CREATE TABLE prerequisito_estado_documento (
    id SERIAL PRIMARY KEY,
    codigo VARCHAR(50) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT,
    
    -- Configuración del prerequisito
    tipo_documento_origen_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id), -- Tipo que requiere el prerequisito
    tipo_documento_precedente_id INTEGER NOT NULL REFERENCES tipo_documento_presupuestal(id), -- Tipo del documento precedente requerido
    
    -- Estados requeridos del documento precedente (JSON array con códigos de estados válidos)
    estados_requeridos JSON NOT NULL, -- ["EXPEDIDO", "AUTORIZADA"]
    
    -- Configuración de validación
    aplicar_en_creacion BOOLEAN DEFAULT TRUE, -- Validar al crear documento
    aplicar_en_movimiento BOOLEAN DEFAULT TRUE, -- Validar al crear movimientos
    
    -- Mensajes
    mensaje_validacion_fallo VARCHAR(500) NOT NULL,
    mensaje_ayuda TEXT,
    
    -- Metadatos
    orden_validacion INTEGER DEFAULT 1, -- Orden de aplicación cuando hay múltiples prerequisitos
    es_activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usuario_creacion INTEGER REFERENCES usuario(id),
    
    -- Índice único
    UNIQUE(codigo)
);
```

### Tabla: prerequisito_validacion_log

```sql
CREATE TABLE prerequisito_validacion_log (
    id SERIAL PRIMARY KEY,
    prerequisito_id INTEGER NOT NULL REFERENCES prerequisito_estado_documento(id),
    
    -- Información del documento que requiere validación
    documento_solicitante_id INTEGER NOT NULL REFERENCES documento_presupuestal(id),
    documento_precedente_id INTEGER REFERENCES documento_presupuestal(id),
    
    -- Resultado de la validación
    resultado_validacion BOOLEAN NOT NULL,
    estado_encontrado VARCHAR(50),
    estados_requeridos JSON,
    
    -- Contexto de la validación
    operacion_solicitada VARCHAR(50) NOT NULL, -- 'CREAR_DOCUMENTO', 'CREAR_MOVIMIENTO'
    usuario_operacion INTEGER REFERENCES usuario(id),
    fecha_validacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Información adicional
    mensaje_resultado TEXT,
    datos_contexto JSON
);

-- Índices
CREATE INDEX idx_prerequisito_log_documento ON prerequisito_validacion_log(documento_solicitante_id);
CREATE INDEX idx_prerequisito_log_fecha ON prerequisito_validacion_log(fecha_validacion);
CREATE INDEX idx_prerequisito_log_resultado ON prerequisito_validacion_log(resultado_validacion);
```

---

## Configuraciones de Prerequisitos por Tipo de Documento

### Prerequisitos para Registro Presupuestal (RP)

```sql
-- RP requiere que el CDP precedente esté EXPEDIDO
INSERT INTO prerequisito_estado_documento VALUES
(1, 'RP_REQUIERE_CDP_EXPEDIDO', 'RP requiere CDP Expedido', 
 'Un Registro Presupuestal solo se puede crear sobre un CDP que esté expedido',
 2, 5, -- RP requiere CDP precedente
 '["EXPEDIDO"]', -- Solo estado EXPEDIDO es válido
 true, true, -- Validar en creación y movimientos
 'El CDP precedente debe estar en estado EXPEDIDO para poder crear un Registro Presupuestal',
 'Verifique que el CDP haya sido expedido oficialmente antes de proceder',
 1, true, NOW(), 1);
```

### Prerequisitos para Orden de Pago (OP)

```sql
-- OP requiere que el RP precedente esté EXPEDIDO
INSERT INTO prerequisito_estado_documento VALUES
(2, 'OP_REQUIERE_RP_EXPEDIDA', 'OP requiere RP Expedida',
 'Una Orden de Pago solo se puede crear sobre un RP que esté expedido',
 3, 2, -- OP requiere RP precedente
 '["EXPEDIDO"]', -- Solo estado EXPEDIDO es válido
 true, true, -- Validar en creación y movimientos
 'El Registro Presupuestal precedente debe estar en estado EXPEDIDO para crear una Orden de Pago',
 'Verifique que el RP haya sido expedido antes de proceder con la Orden de Pago',
 1, true, NOW(), 1);
```

### Prerequisitos para Egreso

```sql
-- Egreso requiere que la OP precedente esté AUTORIZADA por contabilidad
INSERT INTO prerequisito_estado_documento VALUES
(3, 'EGRESO_REQUIERE_OP_AUTORIZADA', 'Egreso requiere OP Autorizada',
 'Un Egreso solo se puede crear sobre una Orden de Pago autorizada por contabilidad',
 4, 3, -- Egreso requiere OP precedente
 '["AUTORIZADA_CONTABILIDAD", "AUTORIZADA_TESORERIA"]', -- Estados de autorización válidos
 true, true, -- Validar en creación y movimientos
 'La Orden de Pago precedente debe estar autorizada por contabilidad antes de generar el egreso',
 'Solicite la autorización de la Orden de Pago en el módulo de contabilidad',
 1, true, NOW(), 1);
```
