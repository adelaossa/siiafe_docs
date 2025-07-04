# Budget Module / Módulo de Presupuesto

## English

### Overview
The Budget Module is the central core of the SIIAFE system, responsible for managing the entire budget lifecycle of Colombian governmental entities. This module handles everything from initial budget formulation to execution and control of public resources, ensuring compliance with Colombian regulations and transparency in public fund management.

### Key Features
- **Comprehensive document management**: Complete lifecycle of budget documents with configurable types and workflows
- **Movement-driven architecture**: All value changes through auditable movements, never direct edits
- **State-controlled workflow**: Strict document state transitions with prerequisite validations
- **Realm-based organization**: Multi-realm support with independent numbering and access control
- **Code inheritance system**: Automatic code inheritance from precedent documents with validation
- **Real-time availability**: Instant budget availability queries and balance calculations
- **Colombian compliance**: Full compliance with Colombian budget regulations (CHIP, CGN standards)

### Documentation Structure
1. **[01 - Budget Documents](./es/01-DOCUMENTOS_PRESUPUESTALES.md)**: Document types, realm associations, and precedent relationships
2. **[02 - Document Items](./es/02-ITEMS_DOCUMENTOS_PRESUPUESTALES.md)**: Normalized item model with movement-only value changes
3. **[03 - Budget Movements](./es/03-MOVIMIENTOS_PRESUPUESTALES.md)**: Movement system, types, and automatic counterparts
4. **[04 - Document States](./es/04-ESTADOS_DOCUMENTOS_PRESUPUESTALES.md)**: State management with JavaScript-based transitions
5. **[05 - State Prerequisites](./es/05-PREREQUISITOS_ESTADOS_DOCUMENTOS.md)**: Prerequisite validation system for document operations
6. **[06 - Item Codes](./es/06-CODIGOS_ITEMS_DOCUMENTOS.md)**: Code assignment and inheritance from precedent documents

### Technical Documentation / Documentación Técnica
- **[📊 Architecture Diagram](./ARCHITECTURE_DIAGRAM.md)**: Complete system architecture with entity relationships and data flows

---

## Español

### Descripción General
El Módulo de Presupuesto es el núcleo central del sistema SIIAFE, responsable de gestionar todo el ciclo de vida presupuestario de las entidades gubernamentales colombianas. Este módulo maneja desde la formulación inicial del presupuesto hasta la ejecución y control de los recursos públicos, garantizando el cumplimiento de las normativas colombianas y la transparencia en el manejo de los fondos públicos.

### Características Principales
- **Gestión integral de documentos**: Ciclo completo de documentos presupuestales con tipos y flujos configurables
- **Arquitectura basada en movimientos**: Todos los cambios de valor mediante movimientos auditables, nunca ediciones directas
- **Flujo controlado por estados**: Transiciones estrictas de estados con validaciones de prerequisitos
- **Organización por ámbitos**: Soporte multi-ámbito con numeración independiente y control de acceso
- **Sistema de herencia de códigos**: Herencia automática de códigos desde documentos precedentes con validación
- **Disponibilidad en tiempo real**: Consultas instantáneas de disponibilidad y cálculos de saldos
- **Cumplimiento colombiano**: Cumplimiento total con regulaciones presupuestales colombianas (CHIP, estándares CGN)

### Estructura de Documentación
1. **[01 - Documentos Presupuestales](./es/01-DOCUMENTOS_PRESUPUESTALES.md)**: Tipos de documentos, asociaciones con ámbitos y relaciones precedentes
2. **[02 - Ítems de Documentos](./es/02-ITEMS_DOCUMENTOS_PRESUPUESTALES.md)**: Modelo normalizado de ítems con cambios de valor solo por movimientos
3. **[03 - Movimientos Presupuestales](./es/03-MOVIMIENTOS_PRESUPUESTALES.md)**: Sistema de movimientos, tipos y contrapartidas automáticas
4. **[04 - Estados de Documentos](./es/04-ESTADOS_DOCUMENTOS_PRESUPUESTALES.md)**: Gestión de estados con transiciones basadas en JavaScript
5. **[05 - Prerequisitos de Estados](./es/05-PREREQUISITOS_ESTADOS_DOCUMENTOS.md)**: Sistema de validación de prerequisitos para operaciones
6. **[06 - Códigos de Ítems](./es/06-CODIGOS_ITEMS_DOCUMENTOS.md)**: Asignación y herencia de códigos desde documentos precedentes

### Documentación Técnica
- **[📊 Diagrama de Arquitectura](./ARCHITECTURE_DIAGRAM.md)**: Arquitectura completa del sistema con relaciones de entidades y flujos de datos

## Architecture / Arquitectura

**📊 [Complete Architecture Diagram](./ARCHITECTURE_DIAGRAM.md)** - Comprehensive visual representation of the system architecture with entity relationships, data flows, and technical specifications.

The Budget Module is built around a movement-driven, state-controlled, and realm-based architecture with the following core entities:

### Core Entities / Entidades Principales:

#### Configuration Layer / Capa de Configuración:
1. **Budget Document Type** (Tipo Documento Presupuestal): Document type definitions and workflow configurations
2. **Code Config Document Type** (Configuración Código Tipo Documento): Code requirements and inheritance rules per document type  
3. **Realm** (Ámbito): Organizational scopes with independent numbering and access control
4. **Realm-Document Type Association** (Tipo Ámbito Documento): Many-to-many associations between realms and document types
5. **State Prerequisites** (Prerequisitos Estado Documento): Required states for document operations and transitions

#### Document Layer / Capa de Documentos:
6. **Budget Document** (Documento Presupuestal): Main budget document records with realm assignment
7. **Budget Document Item** (Ítem Documento Presupuestal): Individual items within documents with movement-only balance changes
8. **Budget Item Coding** (Codificación Ítem Presupuestal): Code assignments to items with inheritance from precedents
9. **Budget Document Relation** (Relación Documento Presupuestal): Many-to-many relationships between documents

#### Movement Layer / Capa de Movimientos:
10. **Budget Movement** (Movimiento Presupuestal): Movement transactions with automatic counterpart generation
11. **Movement Document Detail** (Detalle Movimiento Documento): Document-level movement effects
12. **Movement Item Detail** (Detalle Movimiento Ítem): Item-level movement details and balance updates

#### State Management Layer / Capa de Gestión de Estados:
13. **Document State** (Estado Documento Presupuestal): State tracking with timestamp and user information
14. **State Flow Transition** (Transición Flujo Estado Documento): JavaScript-based automatic and manual transitions
15. **State Annexes** (Anexo Estado Documento): Additional state-specific data and configurations

## Document Types / Tipos de Documento

### 1. PG - Expenditure Budget / Presupuesto de Gasto
Base document establishing annual appropriations / Documento base que establece apropiaciones anuales

### 2. CDP - Budget Availability Certificate / Certificado de Disponibilidad Presupuestal  
Certifies existence of budget availability / Certifica existencia de disponibilidad presupuestal

### 3. RP - Budget Registration / Registro Presupuestal
Formalizes resource commitment / Formaliza compromiso de recursos

### 4. Payment Order / Orden de Pago
Authorizes obligation payment / Autoriza pago de obligación

### 5. Disbursement / Egreso
Records actual resource outflow / Registra salida efectiva de recursos

## Technology Stack / Stack Tecnológico
- **Backend**: NestJS with TypeScript (recommended)
- **Database**: PostgreSQL with comprehensive constraints and triggers
- **Architecture**: Movement-driven, state-controlled, realm-based design
- **Integration**: Configuration module for code structure management
- **API**: RESTful with real-time validation and movement processing
- **Validation**: Real-time budget availability, state transitions, and coding validation
- **State Management**: JavaScript-based condition evaluation for automatic transitions
- **Audit**: Complete movement-based audit trail with timestamp and user tracking

## Development Status / Estado de Desarrollo

� **Documentation Complete - Implementation Ready / Documentación Completa - Listo para Implementación**

### ✅ Completed / Completado
1. **Complete modular documentation** / Documentación modular completa
2. **Normalized data model design** / Diseño de modelo de datos normalizado
3. **Movement-driven architecture specification** / Especificación de arquitectura basada en movimientos
4. **State management system design** / Diseño de sistema de gestión de estados
5. **Code inheritance logic definition** / Definición de lógica de herencia de códigos
6. **Prerequisite validation framework** / Marco de validación de prerequisitos
7. **Realm-based organization structure** / Estructura de organización por ámbitos
8. **SQL examples and validation functions** / Ejemplos SQL y funciones de validación

### 🔄 Ready for Implementation / Listo para Implementación
1. **Database schema implementation** / Implementación del esquema de base de datos
2. **Core API development** / Desarrollo de API principal  
3. **Document workflow engine** / Motor de flujo de documentos
4. **Real-time validation system** / Sistema de validación en tiempo real
5. **Movement processing engine** / Motor de procesamiento de movimientos
6. **State transition automation** / Automatización de transiciones de estado
7. **Code inheritance automation** / Automatización de herencia de códigos

### 📋 Next Development Phases / Próximas Fases de Desarrollo

#### Phase 1: Core Implementation / Fase 1: Implementación Central
- Database schema creation with all tables and constraints
- Basic CRUD operations for all entities
- Movement processing with automatic counterparts
- State validation and transition logic

#### Phase 2: Advanced Features / Fase 2: Características Avanzadas  
- Real-time balance calculation engine
- JavaScript-based state condition evaluation
- Code inheritance automation
- Prerequisite validation enforcement

#### Phase 3: Integration & Optimization / Fase 3: Integración y Optimización
- Integration with configuration module for code management
- Performance optimization for large datasets
- Advanced reporting and analytics
- Comprehensive audit trail implementation

### Dependencies / Dependencias
- **Configuration Module**: Required for code structure and classification
- **Authentication Module**: Required for user management and authorization
- **Audit Module**: Integration for comprehensive audit trails

### Contributing / Contribuir
This module is part of the SIIAFE project. Please refer to the main project documentation for contribution guidelines.

Este módulo es parte del proyecto SIIAFE. Por favor, consulte la documentación principal del proyecto para las pautas de contribución.

### Recent Updates / Actualizaciones Recientes

#### Version 2.0 - Complete Modular Restructure / Reestructuración Modular Completa

**English**: 
- **COMPLETE REWRITE**: All documentation restructured into modular, conceptual components
- **MOVEMENT-DRIVEN ARCHITECTURE**: All value changes through auditable movements, zero direct edits
- **REALM-BASED ORGANIZATION**: Multi-realm support with mandatory ámbito assignment for all documents
- **STATE-CONTROLLED WORKFLOW**: Strict state transitions with JavaScript-based conditions and prerequisite validation
- **CODE INHERITANCE SYSTEM**: Automatic code inheritance from precedent documents with configurable requirements
- **ENHANCED AUDITABILITY**: Complete audit trail for all operations with normalized data model
- **MODULAR DOCUMENTATION**: Six specialized documents covering each core concept independently

**Español**:
- **REESCRITURA COMPLETA**: Toda la documentación reestructurada en componentes modulares y conceptuales
- **ARQUITECTURA BASADA EN MOVIMIENTOS**: Todos los cambios de valor a través de movimientos auditables, cero ediciones directas
- **ORGANIZACIÓN POR ÁMBITOS**: Soporte multi-ámbito con asignación obligatoria de ámbito para todos los documentos
- **FLUJO CONTROLADO POR ESTADOS**: Transiciones estrictas de estados con condiciones basadas en JavaScript y validación de prerequisitos
- **SISTEMA DE HERENCIA DE CÓDIGOS**: Herencia automática de códigos desde documentos precedentes con requisitos configurables
- **AUDITABILIDAD MEJORADA**: Rastro completo de auditoría para todas las operaciones con modelo de datos normalizado
- **DOCUMENTACIÓN MODULAR**: Seis documentos especializados cubriendo cada concepto central independientemente

**New Features / Nuevas Características:**
- ✅ **Ámbito (Realm) System**: Multi-organizational support with independent document numbering
- ✅ **Code Inheritance**: Automatic propagation of codes from precedent documents  
- ✅ **Strict Prerequisites**: Non-bypassable validation of required states for operations
- ✅ **Movement-Only Changes**: All balance modifications through auditable movement transactions
- ✅ **JavaScript State Logic**: Flexible, configurable state transition conditions
- ✅ **Enhanced Relationships**: Many-to-many document associations with value tracking

**Removed Features / Características Eliminadas:**
- ❌ **Direct Balance Edits**: All value changes now require movement transactions
- ❌ **Exception Logic**: No bypass mechanisms for business rule validation
- ❌ **Manual State Transitions**: All transitions now follow configured rules
- ❌ **Single Document Origins**: Replaced with flexible many-to-many relationships

---

## Practical Examples / Ejemplos Prácticos

### Complete Budget Flow Example / Ejemplo de Flujo Presupuestal Completo

**English**: [Budget Flow Example](./BUDGET_FLOW_EXAMPLE.md)
- Step-by-step walkthrough of a complete budget process
- From code structure setup to final payment and CDP release
- Detailed SQL examples showing data changes at each step
- Real-world scenario: Municipal consulting contract for $120M

**Español**: [Ejemplo de Flujo Presupuestal](./EJEMPLO_FLUJO_PRESUPUESTAL.md)
- Proceso paso a paso de un flujo presupuestal completo
- Desde la configuración de códigos hasta el pago final y liberación de CDP
- Ejemplos SQL detallados mostrando cambios de datos en cada paso
- Escenario real: Contrato de consultoría municipal por $120M

### Key Learning Points / Puntos Clave de Aprendizaje

**English**:
- How many-to-many relationships work in practice
- Budget availability calculations and updates
- Document state transitions and validations
- Complete traceability from appropriation to payment

**Español**:
- Cómo funcionan las relaciones muchos-a-muchos en la práctica
- Cálculos y actualizaciones de disponibilidad presupuestal
- Transiciones de estado y validaciones de documentos
- Trazabilidad completa desde la apropiación hasta el pago

### Specialized Documentation / Documentación Especializada

#### Budget Flow Examples / Ejemplos de Flujo Presupuestal
- **English**: [Complete Budget Flow Example](./BUDGET_FLOW_EXAMPLE.md)
- **Español**: [Ejemplo Completo de Flujo Presupuestal](./es/EJEMPLO_FLUJO_PRESUPUESTAL.md)

#### Document and Movement Management / Gestión de Documentos y Movimientos
- **English**: [Budget Documents and Movements](./en/BUDGET_DOCUMENTS_AND_MOVEMENTS.md)
- **Español**: [Documentos y Movimientos Presupuestales](./es/DOCUMENTOS_Y_MOVIMIENTOS_PRESUPUESTALES.md)

#### Advanced Features / Características Avanzadas
- **Movement-Driven Architecture**: All value changes through auditable movements, never direct edits
- **Realm-Based Organization**: Multi-realm support with independent numbering and access control  
- **Code Inheritance System**: Automatic code propagation from precedent documents with validation
- **State-Controlled Workflow**: JavaScript-based automatic transitions with prerequisite validation
- **Many-to-Many Document Relations**: Flexible document relationships supporting complex business scenarios
- **Automatic Counterparts**: Automatic generation of movement counterparts for complete audit trails
- **Prerequisite Validation**: Strict enforcement of required states for document operations
- **Real-time Balance Tracking**: Instant calculation of budget balances and availability through movements
