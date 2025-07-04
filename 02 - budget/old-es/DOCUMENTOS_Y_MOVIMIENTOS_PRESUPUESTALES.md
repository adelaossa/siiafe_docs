# Documentos y Movimientos Presupuestales

## Descripción General

Este documento explica el funcionamiento detallado de los **Documentos Presupuestales** y los **Movimientos Presupuestales** en el sistema SIIAFE, definiendo las columnas específicas que maneja cada tipo de documento y cómo los diferentes tipos de movimientos afectan estas columnas.

## Conceptos Fundamentales

### Documentos Presupuestales
Los documentos presupuestales son entidades que representan diferentes estados y operaciones dentro del ciclo presupuestario. Cada tipo de documento tiene un conjunto específico de columnas que reflejan su naturaleza y propósito.

### Movimientos Presupuestales
Los movimientos presupuestales son las transacciones que modifican las columnas de los documentos presupuestales, siguiendo reglas específicas de negocio según el tipo de movimiento y el tipo de documento afectado.

---

## Tipos de Documentos Presupuestales y sus Columnas

### 1. Presupuesto de Gasto (PG)

**Propósito**: Documento principal que establece las apropiaciones presupuestales anuales.

**Columnas de Items**:
- `presupuesto_inicial`: Valor inicial aprobado para la vigencia
- `adiciones`: Suma de todas las adiciones realizadas
- `recortes`: Suma de todos los recortes realizados
- `traslados`: Neto de traslados (entrada menos salida)
- `presupuesto_definitivo`: Presupuesto vigente después de modificaciones
- `valor_reservado`: Valor comprometido en CDPs vigentes
- `saldo_apropiacion`: Saldo disponible para nuevos compromisos

**Fórmulas de Cálculo**:
```
presupuesto_definitivo = presupuesto_inicial + adiciones - recortes + traslados
saldo_apropiacion = presupuesto_definitivo - valor_reservado
```

### 2. Adición Presupuestal (AD)

**Propósito**: Documento que incrementa el presupuesto total.

**Columnas de Items**:
- `valor_adicion`: Valor de la adición presupuestal

### 3. Recorte Presupuestal (RC)

**Propósito**: Documento que reduce el presupuesto total.

**Columnas de Items**:
- `valor_recorte`: Valor del recorte presupuestal

### 4. Traslado Presupuestal (TR)

**Propósito**: Documento que mueve recursos entre rubros sin cambiar el total.

**Columnas de Items**:
- `valor_traslado_salida`: Valor que sale del rubro
- `valor_traslado_entrada`: Valor que entra al rubro

### 5. Certificado de Disponibilidad Presupuestal (CDP)

**Propósito**: Documento que certifica disponibilidad de recursos.

**Columnas de Items**:
- `valor_inicial`: Valor inicial del CDP
- `compromisos`: Valor comprometido en RPs
- `liberaciones`: Valor liberado y devuelto al PG
- `saldo_comprometer`: Saldo disponible para comprometer

**Fórmulas de Cálculo**:
```
saldo_comprometer = valor_inicial - compromisos - liberaciones
```

### 6. Registro Presupuestal (RP)

**Propósito**: Documento que formaliza el compromiso de recursos.

**Columnas de Items**:
- `valor_inicial`: Valor inicial del RP
- `obligaciones`: Valor causado como obligaciones
- `liquidaciones`: Valor liquidado y devuelto al CDP
- `saldo_obligar`: Saldo disponible para causar obligaciones

**Fórmulas de Cálculo**:
```
saldo_obligar = valor_inicial - obligaciones - liquidaciones
```

### 7. Orden de Pago (OP)

**Propósito**: Documento que autoriza el pago de obligaciones.

**Columnas de Items**:
- `valor_inicial`: Valor inicial de la orden
- `pagos`: Valor efectivamente pagado
- `anulaciones`: Valor anulado de la orden
- `saldo_pagar`: Saldo pendiente de pago

**Fórmulas de Cálculo**:
```
saldo_pagar = valor_inicial - pagos - anulaciones
```

---

## Tipos de Movimientos Presupuestales

### 1. Apropiación Inicial

**Descripción**: Establece el presupuesto inicial de la vigencia.

**Documentos Afectados**:
- **Presupuesto de Gasto**: 
  - ✅ Incrementa: `presupuesto_inicial`
  - ✅ Incrementa: `presupuesto_definitivo`
  - ✅ Incrementa: `saldo_apropiacion`



### 2. Adición Presupuestal

**Descripción**: Incrementa el presupuesto con nuevos recursos.

**Documentos Afectados**:
- **Presupuesto de Gasto**:
  - ✅ Incrementa: `adiciones`
  - ✅ Incrementa: `presupuesto_definitivo`
  - ✅ Incrementa: `saldo_apropiacion`
- **Adición Presupuestal**:
  - ✅ Incrementa: `valor_adicion`



### 3. Recorte Presupuestal

**Descripción**: Reduce el presupuesto por menor disponibilidad de recursos.

**Documentos Afectados**:
- **Presupuesto de Gasto**:
  - ✅ Incrementa: `recortes`
  - ❌ Reduce: `presupuesto_definitivo`
  - ❌ Reduce: `saldo_apropiacion`
- **Recorte Presupuestal**:
  - ✅ Incrementa: `valor_recorte`



### 4. Traslado Presupuestal

**Descripción**: Mueve recursos entre rubros sin cambiar el total.

**Documentos Afectados**:
- **Presupuesto de Gasto (Item Origen)**:
  - ❌ Reduce: `traslados`
  - ❌ Reduce: `presupuesto_definitivo`
  - ❌ Reduce: `saldo_apropiacion`
- **Presupuesto de Gasto (Item Destino)**:
  - ✅ Incrementa: `traslados`
  - ✅ Incrementa: `presupuesto_definitivo`
  - ✅ Incrementa: `saldo_apropiacion`
- **Traslado Presupuestal**:
  - ✅ Incrementa: `valor_traslado_salida` (item origen)
  - ✅ Incrementa: `valor_traslado_entrada` (item destino)

### 5. Valor CDP (Expedición de CDP)

**Descripción**: Reserva recursos del presupuesto en un CDP.

**Documentos Afectados**:
- **Presupuesto de Gasto**:
  - ✅ Incrementa: `valor_reservado`
  - ❌ Reduce: `saldo_apropiacion`
- **Certificado de Disponibilidad Presupuestal**:
  - ✅ Incrementa: `valor_inicial`
  - ✅ Incrementa: `saldo_comprometer`



### 6. Valor RP (Expedición de RP)

**Descripción**: Compromete recursos del CDP en un RP.

**Documentos Afectados**:
- **Certificado de Disponibilidad Presupuestal**:
  - ✅ Incrementa: `compromisos`
  - ❌ Reduce: `saldo_comprometer`
- **Registro Presupuestal**:
  - ✅ Incrementa: `valor_inicial`
  - ✅ Incrementa: `saldo_obligar`

### 7. Liberación CDP

**Descripción**: Libera recursos no utilizados del CDP de vuelta al PG.

**Documentos Afectados**:
- **Presupuesto de Gasto**:
  - ❌ Reduce: `valor_reservado`
  - ✅ Incrementa: `saldo_apropiacion`
- **Certificado de Disponibilidad Presupuestal**:
  - ✅ Incrementa: `liberaciones`
  - ❌ Reduce: `saldo_comprometer`

### 8. Causación (Valor OP)

**Descripción**: Causa una obligación del RP en una OP.

**Documentos Afectados**:
- **Registro Presupuestal**:
  - ✅ Incrementa: `obligaciones`
  - ❌ Reduce: `saldo_obligar`
- **Orden de Pago**:
  - ✅ Incrementa: `valor_inicial`
  - ✅ Incrementa: `saldo_pagar`

---

## Estructura de Base de Datos

### Tabla: item_documento_presupuestal

**Columnas Base (para todos los tipos)**:
```sql
CREATE TABLE item_documento_presupuestal (
    id SERIAL PRIMARY KEY,
    documento_id INTEGER NOT NULL,
    numero_item INTEGER NOT NULL,
    descripcion_item VARCHAR(500),
    
    -- Metadatos
    metadatos_item JSON,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Tabla: item_documento_columna_valor

**Estructura Normalizada para Valores de Columnas**:
```sql
CREATE TABLE item_documento_columna_valor (
    id SERIAL PRIMARY KEY,
    item_documento_id INTEGER NOT NULL REFERENCES item_documento_presupuestal(id),
    tipo_documento_columna_id INTEGER NOT NULL REFERENCES tipo_documento_columnas(id),
    valor_columna DECIMAL(18,2) DEFAULT 0,
    
    -- Metadatos
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Índice único para evitar duplicados
    UNIQUE(item_documento_id, tipo_documento_columna_id)
);
```

### Configuración de Columnas por Tipo de Documento

```sql
CREATE TABLE tipo_documento_columnas (
    id SERIAL PRIMARY KEY,
    tipo_documento_id INTEGER NOT NULL,
    nombre_columna VARCHAR(100) NOT NULL,
    tipo_dato VARCHAR(50) NOT NULL,
    valor_defecto DECIMAL(18,2) DEFAULT 0,
    es_calculada BOOLEAN DEFAULT FALSE,
    formula_calculo TEXT,
    descripcion TEXT,
    orden_presentacion INTEGER DEFAULT 1,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Índice único por tipo de documento y nombre de columna
    UNIQUE(tipo_documento_id, nombre_columna)
);

-- Configuración para Presupuesto de Gasto (tipo_documento_id = 1)
INSERT INTO tipo_documento_columnas VALUES
(1, 1, 'presupuesto_inicial', 'DECIMAL', 0, false, null, 'Presupuesto inicial aprobado', 1, true, NOW(), NOW()),
(2, 1, 'adiciones', 'DECIMAL', 0, false, null, 'Suma de adiciones presupuestales', 2, true, NOW(), NOW()),
(3, 1, 'recortes', 'DECIMAL', 0, false, null, 'Suma de recortes presupuestales', 3, true, NOW(), NOW()),
(4, 1, 'traslados', 'DECIMAL', 0, false, null, 'Neto de traslados', 4, true, NOW(), NOW()),
(5, 1, 'presupuesto_definitivo', 'DECIMAL', 0, true, 'presupuesto_inicial + adiciones - recortes + traslados', 'Presupuesto vigente', 5, true, NOW(), NOW()),
(6, 1, 'valor_reservado', 'DECIMAL', 0, false, null, 'Valor reservado en CDPs', 6, true, NOW(), NOW()),
(7, 1, 'saldo_apropiacion', 'DECIMAL', 0, true, 'presupuesto_definitivo - valor_reservado', 'Saldo disponible', 7, true, NOW(), NOW());

-- Configuración para Adición Presupuestal (tipo_documento_id = 2)
INSERT INTO tipo_documento_columnas VALUES
(8, 2, 'valor_adicion', 'DECIMAL', 0, false, null, 'Valor de la adición presupuestal', 1, true, NOW(), NOW());

-- Configuración para Recorte Presupuestal (tipo_documento_id = 3)
INSERT INTO tipo_documento_columnas VALUES
(9, 3, 'valor_recorte', 'DECIMAL', 0, false, null, 'Valor del recorte presupuestal', 1, true, NOW(), NOW());

-- Configuración para Traslado Presupuestal (tipo_documento_id = 4)
INSERT INTO tipo_documento_columnas VALUES
(10, 4, 'valor_traslado_salida', 'DECIMAL', 0, false, null, 'Valor que sale del rubro', 1, true, NOW(), NOW()),
(11, 4, 'valor_traslado_entrada', 'DECIMAL', 0, false, null, 'Valor que entra al rubro', 2, true, NOW(), NOW());

-- Configuración para CDP (tipo_documento_id = 5)
INSERT INTO tipo_documento_columnas VALUES
(12, 5, 'valor_inicial', 'DECIMAL', 0, false, null, 'Valor inicial del CDP', 1, true, NOW(), NOW()),
(13, 5, 'compromisos', 'DECIMAL', 0, false, null, 'Valor comprometido en RPs', 2, true, NOW(), NOW()),
(14, 5, 'liberaciones', 'DECIMAL', 0, false, null, 'Valor liberado', 3, true, NOW(), NOW()),
(15, 5, 'saldo_comprometer', 'DECIMAL', 0, true, 'valor_inicial - compromisos - liberaciones', 'Saldo por comprometer', 4, true, NOW(), NOW());

-- Configuración para RP (tipo_documento_id = 6)
INSERT INTO tipo_documento_columnas VALUES
(16, 6, 'valor_inicial', 'DECIMAL', 0, false, null, 'Valor inicial del RP', 1, true, NOW(), NOW()),
(17, 6, 'obligaciones', 'DECIMAL', 0, false, null, 'Valor causado como obligaciones', 2, true, NOW(), NOW()),
(18, 6, 'liquidaciones', 'DECIMAL', 0, false, null, 'Valor liquidado y devuelto al CDP', 3, true, NOW(), NOW()),
(19, 6, 'saldo_obligar', 'DECIMAL', 0, true, 'valor_inicial - obligaciones - liquidaciones', 'Saldo disponible para obligar', 4, true, NOW(), NOW());

-- Configuración para OP (tipo_documento_id = 7)
INSERT INTO tipo_documento_columnas VALUES
(20, 7, 'valor_inicial', 'DECIMAL', 0, false, null, 'Valor inicial de la orden', 1, true, NOW(), NOW()),
(21, 7, 'pagos', 'DECIMAL', 0, false, null, 'Valor efectivamente pagado', 2, true, NOW(), NOW()),
(22, 7, 'anulaciones', 'DECIMAL', 0, false, null, 'Valor anulado de la orden', 3, true, NOW(), NOW()),
(23, 7, 'saldo_pagar', 'DECIMAL', 0, true, 'valor_inicial - pagos - anulaciones', 'Saldo pendiente de pago', 4, true, NOW(), NOW());
```

---

## Configuración de Afectaciones por Movimiento

### Tabla: movimiento_tipo_afectacion

```sql
CREATE TABLE movimiento_tipo_afectacion (
    id SERIAL PRIMARY KEY,
    movimiento_tipo_id INTEGER NOT NULL,
    tipo_documento_columna_id INTEGER NOT NULL REFERENCES tipo_documento_columnas(id),
    tipo_afectacion VARCHAR(20) NOT NULL, -- 'INCREMENTA', 'REDUCE'
    factor_multiplicador DECIMAL(10,4) DEFAULT 1.0,
    es_origen BOOLEAN DEFAULT FALSE, -- Si es el documento origen del movimiento
    es_destino BOOLEAN DEFAULT FALSE, -- Si es el documento destino del movimiento
    orden_ejecucion INTEGER DEFAULT 1,
    es_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Configuración para Apropiación Inicial (movimiento_tipo_id = 1)
-- Afecta columnas del Presupuesto de Gasto
INSERT INTO movimiento_tipo_afectacion VALUES
(1, 1, 1, 'INCREMENTA', 1.0, false, true, 1, true, NOW(), NOW()), -- presupuesto_inicial
(2, 1, 5, 'INCREMENTA', 1.0, false, true, 2, true, NOW(), NOW()), -- presupuesto_definitivo
(3, 1, 7, 'INCREMENTA', 1.0, false, true, 3, true, NOW(), NOW()); -- saldo_apropiacion

-- Configuración para Adición Presupuestal (movimiento_tipo_id = 2)
-- Afecta columnas del Presupuesto de Gasto + columnas del documento Adición
INSERT INTO movimiento_tipo_afectacion VALUES
(4, 2, 2, 'INCREMENTA', 1.0, false, true, 1, true, NOW(), NOW()), -- adiciones (PG)
(5, 2, 5, 'INCREMENTA', 1.0, false, true, 2, true, NOW(), NOW()), -- presupuesto_definitivo (PG)
(6, 2, 7, 'INCREMENTA', 1.0, false, true, 3, true, NOW(), NOW()), -- saldo_apropiacion (PG)
(7, 2, 8, 'INCREMENTA', 1.0, true, false, 4, true, NOW(), NOW()); -- valor_adicion (Adición)

-- Configuración para Recorte Presupuestal (movimiento_tipo_id = 3)
-- Afecta columnas del Presupuesto de Gasto + columnas del documento Recorte
INSERT INTO movimiento_tipo_afectacion VALUES
(8, 3, 3, 'INCREMENTA', 1.0, false, true, 1, true, NOW(), NOW()), -- recortes (PG)
(9, 3, 5, 'REDUCE', 1.0, false, true, 2, true, NOW(), NOW()), -- presupuesto_definitivo (PG)
(10, 3, 7, 'REDUCE', 1.0, false, true, 3, true, NOW(), NOW()), -- saldo_apropiacion (PG)
(11, 3, 9, 'INCREMENTA', 1.0, true, false, 4, true, NOW(), NOW()); -- valor_recorte (Recorte)

-- Configuración para Valor CDP (movimiento_tipo_id = 5)
-- Afecta columnas del Presupuesto de Gasto + columnas del CDP
INSERT INTO movimiento_tipo_afectacion VALUES
(12, 5, 6, 'INCREMENTA', 1.0, true, false, 1, true, NOW(), NOW()), -- valor_reservado (PG)
(13, 5, 7, 'REDUCE', 1.0, true, false, 2, true, NOW(), NOW()), -- saldo_apropiacion (PG)
(14, 5, 12, 'INCREMENTA', 1.0, false, true, 3, true, NOW(), NOW()), -- valor_inicial (CDP)
(15, 5, 15, 'INCREMENTA', 1.0, false, true, 4, true, NOW(), NOW()); -- saldo_comprometer (CDP)

-- Configuración para Valor OP (movimiento_tipo_id = 7)
-- Afecta columnas del RP + columnas de la OP
INSERT INTO movimiento_tipo_afectacion VALUES
(20, 7, 17, 'INCREMENTA', 1.0, true, false, 1, true, NOW(), NOW()), -- obligaciones (RP)
(21, 7, 19, 'REDUCE', 1.0, true, false, 2, true, NOW(), NOW()), -- saldo_obligar (RP)
(22, 7, 20, 'INCREMENTA', 1.0, false, true, 3, true, NOW(), NOW()), -- valor_inicial (OP)
(23, 7, 23, 'INCREMENTA', 1.0, false, true, 4, true, NOW(), NOW()); -- saldo_pagar (OP)

-- Configuración para Valor RP (movimiento_tipo_id = 6)
-- Afecta columnas del CDP + columnas del RP
INSERT INTO movimiento_tipo_afectacion VALUES
(16, 6, 13, 'INCREMENTA', 1.0, true, false, 1, true, NOW(), NOW()), -- compromisos (CDP)
(17, 6, 15, 'REDUCE', 1.0, true, false, 2, true, NOW(), NOW()), -- saldo_comprometer (CDP)
(18, 6, 16, 'INCREMENTA', 1.0, false, true, 3, true, NOW(), NOW()), -- valor_inicial (RP)
(19, 6, 19, 'INCREMENTA', 1.0, false, true, 4, true, NOW(), NOW()); -- saldo_obligar (RP)
```


---


## Ventajas del Sistema Normalizado

### 1. **Estructura Normalizada y Auditable**
- Cada valor de columna se almacena en una fila independiente
- Elimina redundancia y garantiza integridad referencial
- Facilita auditorías granulares a nivel de columna individual

### 2. **Flexibilidad Total**
- Configuración dinámica de columnas por tipo de documento
- Fácil adición de nuevos tipos de documentos sin cambios estructurales
- Soporte para columnas calculadas con fórmulas personalizables

### 3. **Trazabilidad Completa**
- Registro detallado de cada cambio en cada columna
- Historial completo de movimientos y sus efectos
- Auditoría automática con triggers

### 4. **Consistencia y Validación**
- Reglas claras de afectación por tipo de movimiento
- Validaciones automáticas en cada procesamiento
- Recálculo automático de campos dependientes

### 5. **Escalabilidad y Performance**
- Índices optimizados para consultas frecuentes
- Vistas materializadas para reportes complejos
- Consultas eficientes usando la estructura normalizada

### 6. **Cumplimiento Normativo**
- Alineado con principios contables del sector público
- Cumple estándares de trazabilidad gubernamental
- Facilita generación de reportes oficiales

### 7. **Mantenibilidad**
- Código modular y reutilizable
- Configuración centralizada de reglas de negocio
- Fácil modificación de comportamientos sin cambios de código

Este sistema proporciona un marco robusto, normalizado y completamente auditable para el manejo de documentos y movimientos presupuestales en SIIAFE, garantizando la integridad de los datos y facilitando el seguimiento detallado de todas las operaciones presupuestales del sector público colombiano.
