# SIIAFE - Financial Information System

## Overview

SIIAFE (Sistema de Información Integral de Administración Financiera Estatal) is a comprehensive Enterprise Resource Planning (ERP) system designed specifically for governmental entities, primarily focused on the Colombian public sector. The system aims to streamline financial administration processes and provide robust information management capabilities for government institutions.

## Project Status

🚧 **In Development** - This project is currently in the planning and initial development phase.

## Technology Stack

### Backend
- **Runtime**: Node.js
- **Framework**: NestJS
- **Language**: TypeScript
- **Architecture**: RESTful API with modular microservices approach

### Database
- **Status**: To be determined (TBD)
- **Options under consideration**: 
  - PostgreSQL (recommended for financial data integrity)
  - MySQL
  - MongoDB (for specific use cases)

### Frontend
- **Status**: To be determined (TBD)
- **Options under consideration**:
  - React with TypeScript
  - Angular
  - Vue.js

### Additional Technologies (Planned)
- **Authentication**: JWT tokens, OAuth 2.0
- **Documentation**: Swagger/OpenAPI
- **Testing**: Jest, Supertest
- **Containerization**: Docker
- **CI/CD**: GitHub Actions / GitLab CI

## Target Audience

The system is primarily designed for:
- Colombian governmental entities
- Public sector financial departments
- Government accounting offices
- Municipal and departmental administrations
- State-owned enterprises
- Ministry of Finance and Public Credit (MinHacienda)

## Core Features (Planned)

### Financial Management
- **Budget Planning & Control**
  - Annual budget creation and approval workflows
  - Budget execution monitoring
  - Budget modifications and transfers
  - Multi-year budget planning

- **Accounting & General Ledger**
  - Double-entry bookkeeping system
  - Chart of accounts management
  - Journal entries and posting
  - Financial statements generation
  - Compliance with Colombian GAAP (GAAP Colombia)

- **Accounts Payable**
  - Vendor management
  - Invoice processing and approval
  - Payment scheduling and execution
  - Tax withholdings calculation

- **Accounts Receivable**
  - Revenue recognition
  - Billing and invoicing
  - Collections management
  - Aging reports

### Procurement & Contracts
- **Public Procurement**
  - SECOP integration (Colombian public procurement system)
  - Bidding process management
  - Contract lifecycle management
  - Supplier evaluation and qualification

### Treasury Management
- **Cash Management**
  - Cash flow forecasting
  - Bank reconciliation
  - Treasury operations
  - Liquidity management

### Reporting & Analytics
- **Financial Reporting**
  - Standard financial statements
  - Regulatory reports (CGN - Contaduría General de la Nación)
  - Custom report builder
  - Dashboard and KPIs

- **Compliance Reporting**
  - CHIP (Clasificador de Ingresos y Gastos Públicos)
  - SIA (Sistema Integrado de Auditoría)
  - Integration with government reporting systems

### Administration
- **User Management**
  - Role-based access control (RBAC)
  - Multi-entity support
  - Audit trails
  - Digital signatures

## System Architecture

### High-Level Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   API Gateway   │    │   Databases     │
│   (TBD)         │◄──►│   NestJS        │◄──►│   (TBD)         │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │  Microservices  │
                    │                 │
                    │ • Auth Service  │
                    │ • Finance API   │
                    │ • Reporting API │
                    │ • Procurement   │
                    │ • Treasury      │
                    └─────────────────┘
```

### Core Modules (Planned)
- **Authentication & Authorization Module**
- **Financial Management Module**
- **Budget Management Module**
- **Procurement Module**
- **Treasury Module**
- **Reporting Module**
- **Configuration Module**
- **Audit Module**

## Compliance & Standards

### Colombian Regulations
- **Ley 1474 de 2011** (Estatuto Anticorrupción)
- **Decreto 1082 de 2015** (Reglamentario del sector administrativo)
- **Resolución 533 de 2015** (Marco Normativo de Información Financiera)
- **CHIP** - Public Income and Expenditure Classifier
- **Manual de Estadísticas de Finanzas Públicas (MEFP)**

### International Standards
- **IPSAS** (International Public Sector Accounting Standards) compatibility
- **ISO 27001** for information security
- **SOX compliance** principles for internal controls

## Security Requirements

- **Data Encryption**: End-to-end encryption for sensitive financial data
- **Access Control**: Multi-factor authentication (MFA)
- **Audit Logging**: Comprehensive audit trails for all transactions
- **Data Backup**: Automated backup and disaster recovery procedures
- **Network Security**: VPN access, firewall configuration
- **Compliance**: GDPR-like data protection (Ley 1581 de 2012 - Habeas Data)

## Development Roadmap

### Phase 1: Foundation (Q3 2025)
- [ ] Technology stack finalization
- [ ] Development environment setup
- [ ] Core architecture implementation
- [ ] Authentication system
- [ ] Basic user management

### Phase 2: Core Financial Features (Q4 2025)
- [ ] Chart of accounts management
- [ ] General ledger implementation
- [ ] Basic budget management
- [ ] Financial transactions processing

### Phase 3: Advanced Features (Q1 2026)
- [ ] Procurement module
- [ ] Treasury management
- [ ] Reporting engine
- [ ] Dashboard implementation

### Phase 4: Integration & Compliance (Q2 2026)
- [ ] SECOP integration
- [ ] Government reporting systems integration
- [ ] Compliance modules
- [ ] Performance optimization

## Installation & Setup

*This section will be updated as development progresses.*

## Contributing

*Guidelines for contributing to the project will be established once the repository is set up.*

## License

*License information to be determined.*

## Contact

- **Project Lead**: [To be assigned]
- **Technical Lead**: [To be assigned]
- **Email**: [To be defined]

## Changelog

### [Unreleased]
- Initial project documentation
- Technology stack planning
- Architecture design

---

**Note**: This is a living document and will be updated as the project evolves and requirements are refined.
