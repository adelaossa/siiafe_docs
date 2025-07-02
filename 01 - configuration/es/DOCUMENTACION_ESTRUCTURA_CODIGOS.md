# Documentación del Sistema de Estructura de Códigos

## Descripción General

Las entidades gubernamentales manejan sistemas de códigos complejos que son esenciales para la administración financiera, gestión presupuestaria y cumplimiento regulatorio. Estas estructuras de códigos son sistemas de clasificación jerárquicos que organizan la información financiera según varios criterios como partidas presupuestarias, fuentes de ingresos, sectores de inversión y divisiones administrativas.

El sistema de estructura de códigos está diseñado para manejar la naturaleza dinámica de los requerimientos de codificación gubernamental, donde diferentes períodos fiscales pueden requerir esquemas de codificación diferentes, y las entidades pueden necesitar gestionar múltiples sistemas de códigos simultáneamente.

## Conceptos Clave

### Organización Basada en Ámbitos (Realms)
Cada estructura de códigos pertenece a un **Ámbito**, que representa un contexto específico o alcance de aplicación dentro de la entidad gubernamental. Los ámbitos están típicamente asociados con:
- Períodos fiscales (ej., Año Presupuestal 2025)
- Divisiones administrativas (ej., Administración Central, Entidades Descentralizadas)
- Áreas funcionales (ej., Gestión de Ingresos, Control de Gastos)

### Flexibilidad de Tipos de Código
El sistema soporta múltiples **Tipos de Código** que pueden ser configurados independientemente y reutilizados en diferentes ámbitos. Este enfoque proporciona flexibilidad mientras mantiene consistencia en los estándares de codificación.

### Estructura Jerárquica
La mayoría de los sistemas de códigos gubernamentales son jerárquicos, permitiendo clasificación detallada en múltiples niveles (ej., Capítulo > Artículo > Concepto > Sub-concepto).

### Relaciones entre Tipos de Código
Los tipos de código pueden tener **relaciones muchos-a-muchos** con otros tipos de código, creando estructuras de codificación complejas e interconectadas. Esto permite sistemas de clasificación sofisticados donde un tipo de código puede estar relacionado con múltiples otros tipos de código simultáneamente.

Por ejemplo:
- Un tipo de código "Partida Presupuestaria" puede estar relacionado con "Tipos de Partida", "Sectores de Inversión", "Programas de Inversión" y "Proyectos MGA"
- Cada uno de estos tipos de código relacionados puede, a su vez, tener sus propias relaciones con otros tipos de código
- Esto crea una red flexible de relaciones de codificación que refleja las necesidades reales de clasificación gubernamental

## Reglas de Negocio

1. **Flexibilidad Temporal**: Diferentes años fiscales pueden usar estructuras de códigos completamente diferentes
2. **Soporte Multi-Ámbito**: Una sola entidad puede gestionar múltiples ámbitos de codificación simultáneamente
3. **Reutilización de Tipos de Código**: Los tipos de código pueden ser compartidos entre múltiples ámbitos
4. **Relaciones entre Tipos de Código**: Los tipos de código pueden tener relaciones muchos-a-muchos con otros tipos de código
5. **Organización Jerárquica**: Soporte para códigos jerárquicos multi-nivel
6. **Cumplimiento Regulatorio**: Todas las estructuras de códigos deben cumplir con los estándares de contabilidad gubernamental colombiana

## Modelo de Datos

### Diagrama de Relación de Entidades

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     AMBITO      │    │  AMBITO_TIPO_   │    │   TIPO_CODIGO   │
│                 │    │    CODIGO       │    │                 │
│ • id            │◄───┤                 ├───►│ • id            │
│ • nombre        │    │ • ambito_id     │    │ • nombre        │
│ • descripcion   │    │ • tipo_codigo_id│    │ • descripcion   │
│ • año_fiscal    │    │ • es_activo     │    │ • es_jerarquico │
│ • div_entidad   │    │ • creado_en     │    │ • max_niveles   │
│ • es_activo     │    │ • actualizado_en│    │ • formato_codigo│
│ • creado_en     │    └─────────────────┘    │ • es_activo     │
│ • actualizado_en│                           │ • creado_en     │
└─────────────────┘                           │ • actualizado_en│
                                              └─────────┬───────┘
                                                        │       │
                                         ┌──────────────┘       │
                                         │                      │
                                         ▼                      ▼
                               ┌─────────────────┐    ┌─────────────────┐
                               │ REL_TIPO_CODIGO │    │     CODIGO      │◄┐
                               │                 │    │                 │ │
                               │ • id            │    │ • id            │ │
                               │ • tipo_origen_id│    │ • tipo_codigo_id│ │
                               │ • tipo_destino_id│   │ • valor_codigo  │ │
                               │ • relacion      │    │ • nombre        │ │
                               │ • es_requerido  │    │ • descripcion   │ │
                               │ • es_activo     │    │ • padre_id      │ │
                               │ • creado_en     │    │ • nivel         │ │
                               │ • actualizado_en│    │ • es_activo     │ │
                               └─────────────────┘    │ • orden_sort    │ │
                                                      │ • creado_en     │ │
                                                      │ • actualizado_en│ │
                                                      └─────────┬───────┘ │
                                                                │         │
                                                                ▼         │
                                                      ┌─────────────────┐ │
                                                      │ RELACION_CODIGO │ │
                                                      │                 │ │
                                                      │ • id            │ │
                                                      │ • codigo_origen_id├─┤
                                                      │ • codigo_destino_id├─┘
                                                      │ • tipo_relacion │
                                                      │ • es_activo     │
                                                      │ • creado_en     │
                                                      │ • actualizado_en│
                                                      └─────────────────┘
```

### Definiciones de Tablas

#### 1. AMBITO
Representa el contexto principal o alcance para una estructura de códigos dentro de la entidad gubernamental.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID/INT | Llave primaria |
| nombre | VARCHAR(255) | Nombre del ámbito (ej., "Presupuesto de Gastos - 2025 - Administración Central") |
| descripcion | TEXT | Descripción detallada del ámbito |
| año_fiscal | INT | Año fiscal asociado |
| division_entidad | VARCHAR(100) | Subdivisión de entidad (Central, Descentralizada, etc.) |
| tipo_ambito | VARCHAR(50) | Tipo de ámbito (PRESUPUESTO, INGRESOS, INVERSION, etc.) |
| es_activo | BOOLEAN | Estado activo |
| creado_en | TIMESTAMP | Marca de tiempo de creación |
| actualizado_en | TIMESTAMP | Marca de tiempo de última actualización |

**Ejemplos de Registros:**
```sql
INSERT INTO ambito VALUES 
(1, 'Presupuesto de Gastos - 2025 - Administración Central', 'Presupuesto principal de gastos para administración central', 2025, 'CENTRAL', 'PRESUPUESTO', true),
(2, 'Fuentes de Ingresos - 2025', 'Sistema de clasificación de ingresos para año fiscal 2025', 2025, 'TODAS', 'INGRESOS', true),
(3, 'Sectores de Inversión - Multi-anual', 'Clasificación de sectores de inversión', NULL, 'TODAS', 'INVERSION', true);
```

#### 2. TIPO_CODIGO
Define la estructura y características de diferentes tipos de códigos que pueden ser usados en varios ámbitos.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID/INT | Llave primaria |
| nombre | VARCHAR(255) | Nombre del tipo de código |
| descripcion | TEXT | Descripción del tipo de código |
| es_jerarquico | BOOLEAN | Si el tipo de código soporta jerarquía |
| max_niveles | INT | Máximo de niveles jerárquicos (si es jerárquico) |
| formato_codigo | VARCHAR(50) | Patrón de formato (ej., "##.##.##.##") |
| reglas_validacion | JSON | Reglas de validación para códigos |
| es_activo | BOOLEAN | Estado activo |
| creado_en | TIMESTAMP | Marca de tiempo de creación |
| actualizado_en | TIMESTAMP | Marca de tiempo de última actualización |

**Ejemplos de Registros:**
```sql
INSERT INTO tipo_codigo VALUES 
(1, 'Partidas Presupuestarias 2025', 'Clasificación jerárquica de partidas presupuestarias', true, 4, '##.##.##.##', '{"longitud_min": 2, "longitud_max": 11}', true),
(2, 'Fuentes de Financiación', 'Códigos de fuentes de financiación no jerárquicos', false, 1, '###', '{"longitud_min": 3, "longitud_max": 3}', true),
(3, 'Tipos de Partida', 'Clasificación de tipos de partida presupuestaria', false, 1, '##', '{"longitud_min": 2, "longitud_max": 2}', true),
(4, 'Sectores de Inversión', 'Clasificación de sectores de inversión', true, 3, '##.##.##', '{"longitud_min": 2, "longitud_max": 8}', true);
```

#### 3. AMBITO_TIPO_CODIGO
Tabla de unión que asocia ámbitos con sus tipos de código aplicables.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID/INT | Llave primaria |
| ambito_id | UUID/INT | Llave foránea hacia ambito |
| tipo_codigo_id | UUID/INT | Llave foránea hacia tipo_codigo |
| es_requerido | BOOLEAN | Si este tipo de código es obligatorio para el ámbito |
| es_activo | BOOLEAN | Estado activo |
| creado_en | TIMESTAMP | Marca de tiempo de creación |
| actualizado_en | TIMESTAMP | Marca de tiempo de última actualización |

#### 4. RELACION_TIPO_CODIGO
Tabla de unión que define relaciones muchos-a-muchos entre tipos de código.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID/INT | Llave primaria |
| tipo_origen_id | UUID/INT | Llave foránea hacia tipo_codigo (origen) |
| tipo_destino_id | UUID/INT | Llave foránea hacia tipo_codigo (destino) |
| nombre_relacion | VARCHAR(100) | Nombre/descripción de la relación |
| es_requerido | BOOLEAN | Si la relación es obligatoria |
| reglas_validacion | JSON | Reglas para validar la relación |
| es_activo | BOOLEAN | Estado activo |
| creado_en | TIMESTAMP | Marca de tiempo de creación |
| actualizado_en | TIMESTAMP | Marca de tiempo de última actualización |

**Ejemplos de Registros:**
```sql
INSERT INTO relacion_tipo_codigo VALUES 
(1, 1, 3, 'Clasificación de Partida Presupuestaria', true, '{"forzar_seleccion_unica": true}', true),
(2, 1, 4, 'Asignación de Sector de Inversión', false, '{"permitir_multiples": true}', true),
(3, 1, 5, 'Asignación de Programa de Inversión', false, '{"requerido_condicional": "si_inversion"}', true),
(4, 1, 6, 'Asignación de Proyecto MGA', false, '{"depende_de": "programa_inversion"}', true),
(5, 4, 5, 'Sector a Programa', true, '{"restriccion_jerarquica": true}', true);
```

#### 5. CODIGO
Contiene los valores de código reales y sus relaciones jerárquicas.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID/INT | Llave primaria |
| tipo_codigo_id | UUID/INT | Llave foránea hacia tipo_codigo |
| valor_codigo | VARCHAR(50) | El valor de código real |
| nombre | VARCHAR(255) | Nombre/título del código |
| descripcion | TEXT | Descripción detallada |
| padre_id | UUID/INT | Llave foránea auto-referencial para jerarquía |
| nivel | INT | Nivel jerárquico (1 = nivel raíz) |
| es_activo | BOOLEAN | Estado activo |
| orden_sort | INT | Orden de visualización dentro del mismo nivel |
| creado_en | TIMESTAMP | Marca de tiempo de creación |
| actualizado_en | TIMESTAMP | Marca de tiempo de última actualización |

#### 6. RELACION_CODIGO
Tabla de unión que define relaciones muchos-a-muchos entre códigos individuales.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID/INT | Llave primaria |
| codigo_origen_id | UUID/INT | Llave foránea hacia codigo (origen) |
| codigo_destino_id | UUID/INT | Llave foránea hacia codigo (destino) |
| tipo_relacion_id | UUID/INT | Llave foránea hacia relacion_tipo_codigo |
| metadatos | JSON | Datos adicionales de la relación |
| es_activo | BOOLEAN | Estado activo |
| creado_en | TIMESTAMP | Marca de tiempo de creación |
| actualizado_en | TIMESTAMP | Marca de tiempo de última actualización |

**Ejemplos de Registros:**
```sql
-- Relaciones de Partida Presupuestaria "01.02.03.001 - Programa de Capacitación Docente"
INSERT INTO relacion_codigo VALUES 
-- Se relaciona con Tipo de Partida "3 - Inversión"
(1, 101, 301, 1, '{"clasificacion": "obligatorio"}', true),
-- Se relaciona con Sector de Inversión "01 - Educación"
(2, 101, 401, 2, '{"prioridad": "alta"}', true),
-- Se relaciona con Programa de Inversión "01.02 - Educación Superior"
(3, 101, 501, 3, '{"porcentaje_presupuesto": 85}', true),
-- Se relaciona con Proyecto MGA "EDU-2025-001"
(4, 101, 601, 4, '{"fase_proyecto": "ejecucion"}', true);
```

## Ejemplos de Casos de Uso

### Ejemplo 1: Jerarquía de Partidas Presupuestarias
```
01 - Servicios Personales
├── 01.01 - Sueldos Básicos
├── 01.02 - Compensación Adicional
│   ├── 01.02.01 - Horas Extras
│   └── 01.02.02 - Bonificaciones
└── 01.03 - Contribuciones a la Seguridad Social
```

### Ejemplo 2: Fuentes de Financiación (No jerárquico)
```
001 - Tesoro Nacional
002 - Recursos Propios
003 - Operaciones de Crédito
004 - Cooperación Internacional
```

### Ejemplo 3: Configuración de Ámbito
**Ámbito**: "Presupuesto de Gastos - 2025 - Administración Central"
- **Tipos de Código**:
  - Partidas Presupuestarias 2025 (jerárquico)
  - Fuentes de Financiación (plano)
  - Tipos de Partida (plano)
  - Sectores de Inversión (jerárquico)
  - Programas de Inversión (jerárquico)
  - Proyectos MGA (plano)

- **Relaciones entre Tipos de Código**:
  - Partidas Presupuestarias → Tipos de Partida (requerido)
  - Partidas Presupuestarias → Sectores de Inversión (condicional)
  - Partidas Presupuestarias → Programas de Inversión (condicional)
  - Partidas Presupuestarias → Proyectos MGA (condicional)
  - Sectores de Inversión → Programas de Inversión (jerárquico)
  - Programas de Inversión → Proyectos MGA (dependencia)

**Tipo de Código de Partida Presupuestaria** está relacionado con:
- **Tipos de Partida** (relación requerida):
  - 1 - Funcionamiento
  - 2 - Servicio a la Deuda
  - 3 - Inversión

- **Sectores de Inversión** (opcional, solo para partidas de inversión):
  - 01 - Educación
  - 02 - Salud
  - 03 - Infraestructura
  - 04 - Desarrollo Social

- **Programas de Inversión** (opcional, depende del sector de inversión):
  - 01.01 - Educación Básica
  - 01.02 - Educación Superior
  - 02.01 - Atención Primaria en Salud
  - 02.02 - Infraestructura Hospitalaria

- **Proyectos MGA** (opcional, depende del programa de inversión):
  - Códigos específicos de proyectos del sistema MGA (Gestión y Proyectos de Inversión)

**Ejemplo de Estructura de Codificación:**
```
Partida Presupuestaria: 01.02.03.001 - "Programa de Capacitación Docente"
├── Tipo de Partida: 3 - Inversión
├── Sector de Inversión: 01 - Educación
├── Programa de Inversión: 01.02 - Educación Superior
└── Proyecto MGA: EDU-2025-001 - "Desarrollo Profesional Docente"
```

**Ejemplos de Relaciones de Código:**
```sql
-- Códigos de muestra de diferentes tipos de código
INSERT INTO codigo VALUES 
-- Partidas Presupuestarias
(101, 1, '01.02.03.001', 'Programa de Capacitación Docente', 'Desarrollo profesional para docentes', NULL, 4, true, 1),
-- Tipos de Partida
(301, 3, '3', 'Inversión', 'Gastos de inversión', NULL, 1, true, 3),
-- Sectores de Inversión
(401, 4, '01', 'Educación', 'Inversiones del sector educativo', NULL, 1, true, 1),
-- Programas de Inversión
(501, 5, '01.02', 'Educación Superior', 'Programas universitarios y de educación superior', 401, 2, true, 2),
-- Proyectos MGA
(601, 6, 'EDU-2025-001', 'Desarrollo Profesional Docente', 'Iniciativa nacional de capacitación docente', NULL, 1, true, 1);

-- Relaciones entre estos códigos
INSERT INTO relacion_codigo VALUES 
(1, 101, 301, 1, '{"obligatorio": true, "regla_validacion": "clasificacion_inversion"}', true),
(2, 101, 401, 2, '{"asignacion_sector": 100, "nivel_prioridad": "alta"}', true),
(3, 101, 501, 3, '{"componente_programa": "desarrollo_docente", "participacion_presupuesto": 0.85}', true),
(4, 101, 601, 4, '{"fase_proyecto": "ejecucion", "fecha_inicio": "2025-01-01"}', true);
```

## Consideraciones de Implementación

### Índices de Base de Datos
- Llaves primarias en todas las tablas
- Índices de llaves foráneas para relaciones
- Índice compuesto en (ambito_id, tipo_codigo_id) en ambito_tipo_codigo
- Índice compuesto en (tipo_origen_id, tipo_destino_id) en relacion_tipo_codigo
- Índice compuesto en (codigo_origen_id, codigo_destino_id) en relacion_codigo
- Índice en valor_codigo para búsquedas rápidas
- Índice en padre_id para consultas de jerarquía
- Índice en tipo_relacion_id para consultas de relación

### Diseño de API
- Endpoints RESTful para operaciones CRUD
- Endpoints especializados para operaciones de jerarquía
- Endpoints para gestionar relaciones entre tipos de código
- Endpoints para gestionar relaciones entre códigos individuales
- Operaciones masivas para importación de códigos
- Endpoints de validación para cumplimiento de formato de códigos
- Endpoints de validación de relaciones
- Endpoints de travesía de grafo para consultas de relaciones complejas

### Consideraciones de Rendimiento
- Usar rutas materializadas o conjuntos anidados para jerarquías profundas
- Cache para estructuras de códigos y relaciones frecuentemente accedidas
- Implementar eliminaciones suaves para preservación de datos históricos
- Considerar réplicas de lectura para consultas de reportes
- Optimizar consultas para travesía de relaciones
- Usar consultas basadas en grafos para patrones de relación complejos
- Considerar desnormalización para relaciones de códigos frecuentemente accedidas
- Implementar cache de validación de relaciones

## Cumplimiento Regulatorio Colombiano

Este sistema de estructura de códigos está diseñado para cumplir con:
- **CHIP** (Clasificador de Ingresos y Gastos Públicos)
- **Estándares CGN** (Contaduría General de la Nación)
- **Reglamentos de Ejecución Presupuestal** (Estatuto Orgánico de Presupuesto)
- **Estándares de Contabilidad Pública** (Régimen de Contabilidad Pública)

## Mejoras Futuras

- **Sistema de Versionado**: Rastrear cambios en estructuras de códigos a lo largo del tiempo
- **Herramientas de Importación/Exportación**: Operaciones masivas para gestión de códigos
- **Motor de Validación**: Reglas de validación avanzadas y restricciones
- **Rastro de Auditoría**: Historial completo de cambios para cumplimiento
- **Soporte Multi-idioma**: Soporte para lenguas indígenas según requerido por ley
- **Visualización de Relaciones**: Representación gráfica de relaciones entre tipos de código
- **Validación Inteligente**: Validación contextual basada en relaciones entre tipos de código
- **Plantillas de Relación**: Patrones de relación preconfigurados para casos de uso comunes
