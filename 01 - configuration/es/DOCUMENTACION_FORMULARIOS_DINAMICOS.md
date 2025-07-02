# Documentación del Sistema de Formularios Dinámicos

## Descripción General

El Sistema de Formularios Dinámicos es un componente fundamental que permite la captura, presentación y modificación de códigos de manera flexible y configurable. Este sistema trabaja estrechamente integrado con la estructura de códigos para proporcionar una interfaz de usuario adaptable a las necesidades específicas de cada tipo de código y ámbito.

El sistema está diseñado para manejar la complejidad de las relaciones entre códigos mientras mantiene una experiencia de usuario intuitiva y eficiente para los funcionarios gubernamentales.

## Tipos de Formularios

### 1. Formulario Tipo Índice

**Propósito**: Formulario principal que muestra todos los posibles valores para un tipo de código (Code Type) dentro del ámbito (Realm) actual.

**Características Principales**:
- **Vista Tabular**: Presenta cada código como una fila independiente
- **Columnas Relacionales**: Muestra códigos relacionados como columnas adicionales
- **Configuración Flexible**: Las columnas mostradas, su orden, visibilidad y capacidad de ordenamiento se definen mediante configuración
- **Operaciones CRUD**: Permite ejecutar todas las operaciones de Crear, Leer, Actualizar y Eliminar sobre los registros

**Ejemplo Práctico**:
```
Formulario Índice: Rubros Presupuestarios 2025

| Código Rubro | Nombre Rubro           | Sector      | Programa         | Proyecto    | Acciones |
|--------------|------------------------|-------------|------------------|-------------|----------|
| 01.02.03.001 | Capacitación Docente   | Educación   | Educación Super. | EDU-2025-001| [E][M][D] |
| 01.02.03.002 | Infraestructura Aulas  | Educación   | Educación Básica | EDU-2025-002| [E][M][D] |
| 02.01.01.001 | Equipos Médicos        | Salud       | Atenc. Primaria  | SAL-2025-001| [E][M][D] |
```

### 2. Formulario Tipo Prompt de Selección

**Propósito**: Formulario modal para la selección de códigos que será utilizado por otros formularios o procesos.

**Características Principales**:
- **Interfaz Modal**: Se presenta como una ventana emergente
- **Vista Similar al Índice**: Mantiene la misma estructura visual del formulario índice
- **Selección Simple o Múltiple**: Configurable según las necesidades del formulario que lo invoca
- **Solo Lectura**: No permite operaciones CRUD, únicamente selección
- **Filtros y Búsqueda**: Incluye herramientas para facilitar la búsqueda de códigos específicos

**Caso de Uso Típico**:
- Un formulario de captura de presupuesto necesita seleccionar un sector de inversión
- Se abre el prompt modal mostrando todos los sectores disponibles
- El usuario selecciona el sector deseado y el valor se retorna al formulario origen

### 3. Formulario Tipo Captura/Modificación

**Propósito**: Formulario dedicado para crear nuevos códigos o modificar códigos existentes.

**Características Principales**:
- **Campos Configurables**: Los campos mostrados se definen mediante configuración
- **Validaciones Flexibles**: Permite definir restricciones como campos obligatorios, formatos, etc.
- **Métodos de Selección Variables**: 
  - Auto-completado simple
  - Prompts modales de selección
  - Campos de texto libre
- **Layout Personalizable**: 
  - Organización en secciones
  - Control de colspan para campos específicos
  - Orden y agrupación de campos

**Ejemplo de Configuración**:
```
Formulario Captura: Nuevo Rubro Presupuestario

Sección: Información Básica
├── Código Rubro [Texto, Obligatorio, Validación: ##.##.##.##]
├── Nombre [Texto, Obligatorio, Colspan: 2]
└── Descripción [Área de Texto, Opcional, Colspan: 2]

Sección: Clasificaciones
├── Tipo de Partida [Selección Modal, Obligatorio]
├── Sector de Inversión [Auto-completado, Condicional]
└── Programa de Inversión [Selección Modal, Condicional]
```

## Modelo de Datos

### Diagrama de Relación de Entidades

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ FORM_TYPE_CONFIG│    │ FORM_COLUMN_    │    │ FORM_FIELD_     │
│                 │    │    CONFIG       │    │    CONFIG       │
│ • id            │◄───┤                 │    │                 │
│ • code_type_id  │    │ • form_config_id├───►│ • form_config_id│
│ • realm_id      │    │ • column_code_  │    │ • field_name    │
│ • form_type     │    │   type_id       │    │ • field_type    │
│ • name          │    │ • column_name   │    │ • is_required   │
│ • description   │    │ • display_order │    │ • validation_   │
│ • is_active     │    │ • is_visible    │    │   rules         │
│ • created_at    │    │ • is_sortable   │    │ • selection_    │
│ • updated_at    │    │ • is_hidden_    │    │   method        │
└─────────────────┘    │   default       │    │ • section_name  │
                       │ • column_width  │    │ • field_order   │
                       │ • is_active     │    │ • colspan       │
                       │ • created_at    │    │ • is_active     │
                       │ • updated_at    │    │ • created_at    │
                       └─────────────────┘    │ • updated_at    │
                                              └─────────────────┘
```

### Definiciones de Tablas

#### 1. CONFIGURACION_FORMULARIO_TIPO
Define la configuración principal de formularios por tipo de código y ámbito.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID/INT | Llave primaria |
| tipo_codigo_id | UUID/INT | Llave foránea hacia tipo_codigo |
| ambito_id | UUID/INT | Llave foránea hacia ambito |
| tipo_formulario | ENUM | Tipo de formulario (INDICE, PROMPT_SELECCION, CAPTURA) |
| nombre | VARCHAR(255) | Nombre del formulario |
| descripcion | TEXT | Descripción detallada |
| configuracion_adicional | JSON | Configuraciones específicas (paginación, filtros, etc.) |
| es_activo | BOOLEAN | Estado activo |
| creado_en | TIMESTAMP | Marca de tiempo de creación |
| actualizado_en | TIMESTAMP | Marca de tiempo de última actualización |

**Ejemplos de Registros**:
```sql
INSERT INTO configuracion_formulario_tipo VALUES 
(1, 1, 1, 'INDICE', 'Índice Rubros Presupuestarios 2025', 'Formulario principal para gestión de rubros presupuestarios', '{"pagination": true, "default_page_size": 50, "enable_filters": true}', true),
(2, 1, 1, 'PROMPT_SELECCION', 'Selector Rubros Presupuestarios', 'Modal de selección de rubros presupuestarios', '{"selection_mode": "single", "show_search": true}', true),
(3, 1, 1, 'CAPTURA', 'Captura/Edición Rubro Presupuestario', 'Formulario para crear/editar rubros presupuestarios', '{"auto_save": false, "show_preview": true}', true);
```

#### 2. CONFIGURACION_COLUMNA_FORMULARIO
Define las columnas para formularios tipo índice y prompt de selección.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID/INT | Llave primaria |
| config_formulario_id | UUID/INT | Llave foránea hacia configuracion_formulario_tipo |
| tipo_codigo_columna_id | UUID/INT | Llave foránea hacia tipo_codigo (para códigos relacionados) |
| nombre_columna | VARCHAR(100) | Nombre de la columna |
| titulo_columna | VARCHAR(255) | Título mostrado al usuario |
| orden_visualizacion | INT | Orden de aparición de la columna |
| es_visible | BOOLEAN | Si la columna es visible por defecto |
| es_ordenable | BOOLEAN | Si la columna permite ordenamiento |
| es_oculta_defecto | BOOLEAN | Si la columna está oculta por defecto |
| ancho_columna | VARCHAR(20) | Ancho de la columna (px, %, auto) |
| tipo_datos | VARCHAR(50) | Tipo de datos de la columna |
| formato_visualizacion | JSON | Reglas de formato para la visualización |
| es_activo | BOOLEAN | Estado activo |
| creado_en | TIMESTAMP | Marca de tiempo de creación |
| actualizado_en | TIMESTAMP | Marca de tiempo de última actualización |

**Ejemplos de Registros**:
```sql
INSERT INTO configuracion_columna_formulario VALUES 
(1, 1, 1, 'codigo_rubro', 'Código Rubro', 1, true, true, false, '120px', 'TEXT', '{"align": "left"}', true),
(2, 1, 1, 'nombre_rubro', 'Nombre Rubro', 2, true, true, false, '300px', 'TEXT', '{"align": "left", "truncate": 30}', true),
(3, 1, 4, 'sector_inversion', 'Sector', 3, true, true, false, '150px', 'RELATION', '{"show_code": true, "show_name": true}', true),
(4, 1, 5, 'programa_inversion', 'Programa', 4, true, false, false, '200px', 'RELATION', '{"show_name_only": true}', true),
(5, 1, 6, 'proyecto_mga', 'Proyecto MGA', 5, true, false, true, '150px', 'RELATION', '{"show_code_only": true}', true);
```

#### 3. CONFIGURACION_CAMPO_FORMULARIO
Define los campos para formularios tipo captura/modificación.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID/INT | Llave primaria |
| config_formulario_id | UUID/INT | Llave foránea hacia configuracion_formulario_tipo |
| nombre_campo | VARCHAR(100) | Nombre interno del campo |
| etiqueta_campo | VARCHAR(255) | Etiqueta mostrada al usuario |
| tipo_campo | ENUM | Tipo de campo (TEXT, TEXTAREA, SELECT, AUTOCOMPLETE, etc.) |
| es_requerido | BOOLEAN | Si el campo es obligatorio |
| reglas_validacion | JSON | Reglas de validación específicas |
| metodo_seleccion | ENUM | Método de selección (MANUAL, AUTOCOMPLETE, MODAL_PROMPT) |
| tipo_codigo_relacionado_id | UUID/INT | Llave foránea hacia tipo_codigo (si aplica) |
| nombre_seccion | VARCHAR(100) | Nombre de la sección donde aparece el campo |
| orden_campo | INT | Orden dentro de la sección |
| colspan | INT | Número de columnas que ocupa el campo |
| configuracion_campo | JSON | Configuraciones específicas del campo |
| es_activo | BOOLEAN | Estado activo |
| creado_en | TIMESTAMP | Marca de tiempo de creación |
| actualizado_en | TIMESTAMP | Marca de tiempo de última actualización |

**Ejemplos de Registros**:
```sql
INSERT INTO configuracion_campo_formulario VALUES 
(1, 3, 'codigo_rubro', 'Código del Rubro', 'TEXT', true, '{"pattern": "##.##.##.##", "min_length": 11, "max_length": 11}', 'MANUAL', NULL, 'Información Básica', 1, 1, '{"placeholder": "Ej: 01.02.03.001"}', true),
(2, 3, 'nombre_rubro', 'Nombre del Rubro', 'TEXT', true, '{"min_length": 5, "max_length": 255}', 'MANUAL', NULL, 'Información Básica', 2, 2, '{"placeholder": "Ingrese el nombre del rubro"}', true),
(3, 3, 'descripcion_rubro', 'Descripción', 'TEXTAREA', false, '{"max_length": 1000}', 'MANUAL', NULL, 'Información Básica', 3, 2, '{"rows": 3}', true),
(4, 3, 'tipo_partida', 'Tipo de Partida', 'SELECT', true, '{"required_message": "Debe seleccionar un tipo de partida"}', 'MODAL_PROMPT', 3, 'Clasificaciones', 1, 1, '{"show_description": true}', true),
(5, 3, 'sector_inversion', 'Sector de Inversión', 'AUTOCOMPLETE', false, '{"conditional_required": "if_investment"}', 'AUTOCOMPLETE', 4, 'Clasificaciones', 2, 1, '{"min_chars": 2}', true);
```

## Comportamientos y Reglas de Negocio

### Formularios Tipo Índice

1. **Filtrado Dinámico**: Los registros mostrados se filtran automáticamente según el ámbito actual
2. **Columnas Relacionales**: Solo se muestran las columnas de códigos que tienen relación definida con el tipo de código principal
3. **Acciones Contextuales**: Las acciones disponibles (Crear, Editar, Eliminar) dependen de los permisos del usuario y el estado del código
4. **Paginación Configurable**: El tamaño de página y opciones de navegación se configuran por formulario

### Formularios Tipo Prompt de Selección

1. **Filtrado por Contexto**: Solo muestra códigos aplicables al contexto desde donde se invoca
2. **Validación de Selección**: Valida que la selección cumple con las restricciones definidas
3. **Búsqueda Avanzada**: Incluye capacidades de búsqueda por múltiples campos
4. **Preselección**: Puede recibir valores preseleccionados del formulario que lo invoca

### Formularios Tipo Captura/Modificación

1. **Validación en Tiempo Real**: Las validaciones se ejecutan mientras el usuario completa los campos
2. **Dependencias Condicionales**: Los campos pueden habilitarse/deshabilitarse según valores de otros campos
3. **Auto-completado Inteligente**: Las sugerencias se basan en códigos existentes y relaciones definidas
4. **Preservación de Relaciones**: Mantiene automáticamente las relaciones entre códigos según las reglas definidas

## Casos de Uso Específicos

### Caso 1: Gestión de Rubros Presupuestarios

**Formulario Índice**:
- Muestra todos los rubros del ámbito "Presupuesto 2025"
- Columnas: Código, Nombre, Tipo Partida, Sector, Programa, Proyecto
- Permite ordenar por código y nombre
- Acciones: Crear nuevo rubro, Editar, Eliminar, Ver detalles

**Formulario Captura**:
- Campos obligatorios: Código, Nombre, Tipo de Partida
- Campos condicionales: Sector (solo si Tipo = Inversión)
- Validaciones: Formato de código, unicidad, coherencia de relaciones

### Caso 2: Selección de Sectores para Proyectos

**Formulario Prompt**:
- Modal activado desde formulario de proyectos
- Filtrado: Solo sectores activos del ámbito actual
- Selección simple con búsqueda por nombre
- Retorna código y nombre del sector seleccionado

## Consideraciones de Implementación

### Frontend (Componentes React/Angular/Vue)

```typescript
// Ejemplo de configuración de formulario
interface FormConfiguration {
  id: string;
  formType: 'INDEX' | 'SELECTION_PROMPT' | 'CAPTURE';
  codeTypeId: string;
  realmId: string;
  columns?: ColumnConfiguration[];
  fields?: FieldConfiguration[];
  layout?: LayoutConfiguration;
}

interface ColumnConfiguration {
  name: string;
  title: string;
  dataType: string;
  isVisible: boolean;
  isSortable: boolean;
  width: string;
  relationshipType?: string;
}

interface FieldConfiguration {
  name: string;
  label: string;
  fieldType: 'TEXT' | 'TEXTAREA' | 'SELECT' | 'AUTOCOMPLETE';
  isRequired: boolean;
  validationRules: ValidationRule[];
  selectionMethod: 'MANUAL' | 'AUTOCOMPLETE' | 'MODAL_PROMPT';
  section: string;
  order: number;
  colspan: number;
}
```

### Backend (API Endpoints)

```typescript
// Endpoints para gestión de formularios
GET /api/forms/config/:codeTypeId/:realmId/:formType
POST /api/forms/config
PUT /api/forms/config/:id
DELETE /api/forms/config/:id

// Endpoints para datos de formularios
GET /api/forms/data/:configId
POST /api/forms/data/:configId
PUT /api/forms/data/:configId/:recordId
DELETE /api/forms/data/:configId/:recordId

// Endpoints para prompts de selección
GET /api/forms/selection/:codeTypeId/:realmId
POST /api/forms/selection/validate
```

### Base de Datos

```sql
-- Índices recomendados
CREATE INDEX idx_form_config_type_realm ON configuracion_formulario_tipo(tipo_codigo_id, ambito_id);
CREATE INDEX idx_form_column_config_order ON configuracion_columna_formulario(config_formulario_id, orden_visualizacion);
CREATE INDEX idx_form_field_section_order ON configuracion_campo_formulario(config_formulario_id, nombre_seccion, orden_campo);
```

## Integración con Estructura de Códigos

### Dependencias Clave

1. **Validación de Relaciones**: Los formularios validan automáticamente que las relaciones entre códigos sean coherentes con las definidas en `relacion_tipo_codigo`
2. **Filtrado por Ámbito**: Solo se muestran códigos válidos para el ámbito actual según `ambito_tipo_codigo`
3. **Jerarquías**: Los formularios respetan las jerarquías definidas en los tipos de código
4. **Metadatos de Relación**: Utilizan los metadatos almacenados en `relacion_codigo` para validaciones adicionales

### Sincronización de Cambios

- Los cambios en la estructura de códigos se reflejan automáticamente en los formularios
- Las configuraciones de formulario se validan contra la estructura actual de códigos
- Se mantiene un log de cambios para rastrear modificaciones en configuraciones

## Casos de Error y Manejo

### Errores de Configuración
- Columnas referenciando tipos de código inexistentes
- Campos obligatorios sin validación adecuada
- Relaciones inconsistentes entre formularios

### Errores de Datos
- Códigos duplicados
- Relaciones inválidas
- Violaciones de restricciones de formato

### Recuperación de Errores
- Validación previa a la persistencia
- Rollback automático en caso de falla
- Mensajes de error específicos y accionables

## Mejoras Futuras

1. **Editor Visual de Formularios**: Interfaz drag-and-drop para configurar formularios
2. **Plantillas de Formulario**: Configuraciones predefinidas para casos comunes
3. **Formularios Anidados**: Capacidad de gestionar códigos con múltiples niveles de relación
4. **Importación/Exportación**: Herramientas para migrar configuraciones entre entornos
5. **Versionado de Configuraciones**: Control de versiones para configuraciones de formulario
6. **Analytics de Uso**: Métricas sobre el uso y eficiencia de los formularios
7. **Validaciones Avanzadas**: Reglas de negocio complejas y validaciones cruzadas
8. **Formularios Responsive**: Adaptación automática a diferentes tamaños de pantalla
