# Budget Module / Módulo de Presupuesto

## English

### Overview
The Budget Module is the central core of the SIIAFE system, responsible for managing the entire budget lifecycle of Colombian governmental entities. This module handles everything from initial budget formulation to execution and control of public resources, ensuring compliance with Colombian regulations and transparency in public fund management.

### Key Features
- **Budget document management**: Complete lifecycle of budget documents (PG, CDP, RP, Payment Orders, Disbursements)
- **Movement tracking**: Comprehensive tracking of all budget movements and modifications
- **Code integration**: Full integration with the configuration module for proper classification
- **State management**: Configurable document states with validation workflows
- **Real-time availability**: Instant budget availability queries and validations
- **Colombian compliance**: Designed to meet Colombian budget regulations and standards

### Core Components
- **Budget Documents**: Main instruments representing different budget operations
- **Document Items**: Detailed breakdown of documents with proper coding classification
- **Budget Movements**: Transactions that modify document values (additions, reductions, transfers)
- **Document Types Configuration**: Flexible configuration of document types and their requirements

### Documentation
- **English**: [Budget Module Documentation](./en/BUDGET_MODULE_DOCUMENTATION.md)
- **Español**: [Documentación del Módulo de Presupuesto](./es/DOCUMENTACION_MODULO_PRESUPUESTO.md)

---

## Español

### Descripción General
El Módulo de Presupuesto es el núcleo central del sistema SIIAFE, responsable de gestionar todo el ciclo de vida presupuestario de las entidades gubernamentales colombianas. Este módulo maneja desde la formulación inicial del presupuesto hasta la ejecución y control de los recursos públicos, garantizando el cumplimiento de las normativas colombianas y la transparencia en el manejo de los fondos públicos.

### Características Principales
- **Gestión de documentos presupuestales**: Ciclo completo de documentos presupuestales (PG, CDP, RP, Órdenes de Pago, Egresos)
- **Seguimiento de movimientos**: Seguimiento integral de todos los movimientos y modificaciones presupuestales
- **Integración con códigos**: Integración completa con el módulo de configuración para clasificación adecuada
- **Gestión de estados**: Estados de documento configurables con flujos de validación
- **Disponibilidad en tiempo real**: Consultas y validaciones instantáneas de disponibilidad presupuestal
- **Cumplimiento colombiano**: Diseñado para cumplir con las regulaciones y estándares presupuestales colombianos

### Componentes Principales
- **Documentos Presupuestales**: Instrumentos principales que representan diferentes operaciones presupuestales
- **Ítems de Documento**: Desglose detallado de documentos con clasificación de códigos apropiada
- **Movimientos Presupuestales**: Transacciones que modifican valores de documentos (adiciones, reducciones, traslados)
- **Configuración de Tipos de Documento**: Configuración flexible de tipos de documento y sus requisitos

### Documentación
- **English**: [Budget Module Documentation](./en/BUDGET_MODULE_DOCUMENTATION.md)
- **Español**: [Documentación del Módulo de Presupuesto](./es/DOCUMENTACION_MODULO_PRESUPUESTO.md)

## Architecture / Arquitectura

The Budget Module is built around seven core entities / El Módulo de Presupuesto está construido alrededor de siete entidades principales:

### Core Entities / Entidades Principales:
1. **Budget Document Type** (Tipo Documento Presupuestal): Document type definitions and configurations
2. **Code Config Document Type** (Configuración Código Tipo Documento): Code requirements per document type
3. **Budget Document** (Documento Presupuestal): Main budget document records
4. **Budget Document Item** (Ítem Documento Presupuestal): Individual items within documents
5. **Budget Item Coding** (Codificación Ítem Presupuestal): Code assignments to items
6. **Budget Document Relation** (Relación Documento Presupuestal): Many-to-many relationships between documents
7. **Budget Movement** (Movimiento Presupuestal): Movement transactions between documents
8. **Movement Document Detail** (Detalle Movimiento Documento): Document-level movement details
9. **Movement Item Detail** (Detalle Movimiento Ítem): Item-level movement details

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
- Backend: NestJS with TypeScript
- Database: TBD (PostgreSQL recommended)
- Integration: Configuration module for code management
- API: RESTful with real-time validation endpoints
- Validation: Real-time budget availability and coding validation

## Development Status / Estado de Desarrollo

🚧 **In Planning Phase / En Fase de Planificación**

### Next Steps / Próximos Pasos
1. Database schema implementation / Implementación del esquema de base de datos
2. Core API development / Desarrollo de API principal  
3. Document workflow engine / Motor de flujo de documentos
4. Real-time validation system / Sistema de validación en tiempo real
5. Integration with configuration module / Integración con módulo de configuración
6. Budget availability calculation engine / Motor de cálculo de disponibilidad presupuestal

### Dependencies / Dependencias
- **Configuration Module**: Required for code structure and classification
- **Authentication Module**: Required for user management and authorization
- **Audit Module**: Integration for comprehensive audit trails

### Contributing / Contribuir
This module is part of the SIIAFE project. Please refer to the main project documentation for contribution guidelines.

Este módulo es parte del proyecto SIIAFE. Por favor, consulte la documentación principal del proyecto para las pautas de contribución.

### Recent Updates / Actualizaciones Recientes

#### Version 1.1 - Many-to-Many Document Relationships / Relaciones Muchos-a-Muchos entre Documentos

**English**: 
- **CORRECTION**: Replaced single document origin reference with many-to-many relationship model
- **NEW TABLE**: `BUDGET_DOCUMENT_RELATION` enables complex document relationships
- **USE CASES**: RP can now incorporate multiple CDPs, Payment Orders can consolidate multiple RPs
- **ENHANCED TRACEABILITY**: Complete audit trail from source documents to final disbursements
- **IMPROVED FLEXIBILITY**: Support for real-world scenarios where documents combine multiple sources

**Español**:
- **CORRECCIÓN**: Reemplazado referencia única de documento origen con modelo de relaciones muchos-a-muchos
- **NUEVA TABLA**: `RELACION_DOCUMENTO_PRESUPUESTAL` permite relaciones complejas entre documentos
- **CASOS DE USO**: RP ahora puede incorporar múltiples CDPs, Órdenes de Pago pueden consolidar múltiples RPs
- **TRAZABILIDAD MEJORADA**: Pista de auditoría completa desde documentos origen hasta egresos finales
- **FLEXIBILIDAD MEJORADA**: Soporte para escenarios del mundo real donde documentos combinan múltiples fuentes

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
