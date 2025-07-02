# Configuration Module / Módulo de Configuración

## English

### Overview
The Configuration Module is responsible for managing the complex coding structures that are essential for financial administration in governmental entities. This module provides the foundation for organizing financial information according to various criteria such as budget items, revenue sources, investment sectors, and administrative divisions.

### Key Features
- **Realm-based organization**: Multiple coding contexts for different fiscal periods and administrative divisions
- **Flexible code types**: Hierarchical and flat coding structures
- **Code type relationships**: Complex many-to-many relationships between different code types
- **Dynamic forms system**: Configurable forms for data capture, presentation, and modification
- **Colombian compliance**: Designed to meet CHIP, CGN, and other regulatory requirements

### Documentation
- **English**: 
  - [Coding Structure Documentation](./en/CODING_STRUCTURE_DOCUMENTATION.md)
  - [Dynamic Forms Documentation](./en/DYNAMIC_FORMS_DOCUMENTATION.md)
- **Español**: 
  - [Documentación de Estructura de Códigos](./es/DOCUMENTACION_ESTRUCTURA_CODIGOS.md)
  - [Documentación de Formularios Dinámicos](./es/DOCUMENTACION_FORMULARIOS_DINAMICOS.md)

### Architecture
The module is built around nine core entities:

**Code Structure System:**
1. **Realm** (Ámbito): Defines the context/scope
2. **Code Type** (Tipo de Código): Defines structure characteristics
3. **Realm Code Type** (Ámbito Tipo Código): Associates realms with code types
4. **Code Type Relationship** (Relación Tipo Código): Defines relationships between code types
5. **Code** (Código): Contains actual code values
6. **Code Relationship** (Relación Código): Defines relationships between individual codes

**Dynamic Forms System:**
7. **Form Type Config** (Configuración Formulario Tipo): Defines form configurations
8. **Form Column Config** (Configuración Columna Formulario): Defines column configurations for index/selection forms
9. **Form Field Config** (Configuración Campo Formulario): Defines field configurations for capture/modification forms

### Technology Stack
- Backend: NestJS with TypeScript
- Database: TBD (PostgreSQL recommended)
- Validation: JSON Schema for flexible rule definitions
- API: RESTful with specialized endpoints for hierarchy operations

---

## Español

### Descripción General
El Módulo de Configuración es responsable de gestionar las estructuras de códigos complejas que son esenciales para la administración financiera en entidades gubernamentales. Este módulo proporciona la base para organizar la información financiera según varios criterios como partidas presupuestarias, fuentes de ingresos, sectores de inversión y divisiones administrativas.

### Características Principales
- **Organización basada en ámbitos**: Múltiples contextos de codificación para diferentes períodos fiscales y divisiones administrativas
- **Tipos de código flexibles**: Estructuras de codificación jerárquicas y planas
- **Relaciones entre tipos de código**: Relaciones complejas muchos-a-muchos entre diferentes tipos de código
- **Sistema de formularios dinámicos**: Formularios configurables para captura, presentación y modificación de datos
- **Cumplimiento colombiano**: Diseñado para cumplir con CHIP, CGN y otros requisitos regulatorios

### Documentación
- **English**: 
  - [Coding Structure Documentation](./en/CODING_STRUCTURE_DOCUMENTATION.md)
  - [Dynamic Forms Documentation](./en/DYNAMIC_FORMS_DOCUMENTATION.md)
- **Español**: 
  - [Documentación de Estructura de Códigos](./es/DOCUMENTACION_ESTRUCTURA_CODIGOS.md)
  - [Documentación de Formularios Dinámicos](./es/DOCUMENTACION_FORMULARIOS_DINAMICOS.md)

### Arquitectura
El módulo está construido alrededor de nueve entidades principales:

**Sistema de Estructura de Códigos:**
1. **Realm** (Ámbito): Define el contexto/alcance
2. **Code Type** (Tipo de Código): Define características de estructura
3. **Realm Code Type** (Ámbito Tipo Código): Asocia ámbitos con tipos de código
4. **Code Type Relationship** (Relación Tipo Código): Define relaciones entre tipos de código
5. **Code** (Código): Contiene valores de código reales
6. **Code Relationship** (Relación Código): Define relaciones entre códigos individuales

**Sistema de Formularios Dinámicos:**
7. **Form Type Config** (Configuración Formulario Tipo): Define configuraciones de formularios
8. **Form Column Config** (Configuración Columna Formulario): Define configuraciones de columnas para formularios índice/selección
9. **Form Field Config** (Configuración Campo Formulario): Define configuraciones de campos para formularios captura/modificación

### Stack Tecnológico
- Backend: NestJS con TypeScript
- Base de datos: TBD (PostgreSQL recomendado)
- Validación: JSON Schema para definiciones de reglas flexibles
- API: RESTful con endpoints especializados para operaciones de jerarquía

## Development Status / Estado de Desarrollo

🚧 **In Planning Phase / En Fase de Planificación**

### Next Steps / Próximos Pasos
1. Database schema implementation / Implementación del esquema de base de datos
2. Core API development / Desarrollo de API principal
3. Validation engine / Motor de validación
4. Testing framework / Framework de pruebas
5. Documentation updates / Actualizaciones de documentación

### Contributing / Contribuir
This module is part of the SIIAFE project. Please refer to the main project documentation for contribution guidelines.

Este módulo es parte del proyecto SIIAFE. Por favor, consulte la documentación principal del proyecto para las pautas de contribución.
