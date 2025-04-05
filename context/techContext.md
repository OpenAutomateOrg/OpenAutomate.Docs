# Technical Context

## Architecture Overview
OpenAutomate follows a modern, scalable architecture with a clear separation of concerns:

1. **Backend**: ASP.NET Core API with a clean architecture approach
2. **Frontend**: Next.js application with React and TypeScript
3. **Automation Agents**: Python-based automation bots that can be deployed to various environments
4. **Communication Layer**: RESTful APIs and WebSockets for real-time updates

## Tech Stack
### Backend
- **Framework**: ASP.NET Core 7.0
- **Language**: C# 11
- **Data Access**: Entity Framework Core 7.0
- **Authentication**: JWT with refresh tokens
- **API Documentation**: Swagger/OpenAPI
- **Pattern**: Clean Architecture with CQRS
- **Database**: PostgreSQL (planned)
- **Deployment**: Docker containers

### Frontend
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **UI Components**: Custom components with Tailwind CSS
- **State Management**: React Context API
- **Authentication**: JWT with refresh tokens and HTTP-only cookies
- **API Integration**: Custom fetch-based client
- **SSR Strategy**: Server components with selective client hydration
- **Deployment**: Docker containers or Vercel

### Automation Agents
- **Language**: Python 3.11+
- **Dependencies**: Standard libraries, custom agent SDK
- **Communication**: REST API with the central OpenAutomate server
- **Packaging**: Docker containers or Python packages
- **Execution**: Local or cloud environments

## Core Technologies

### Backend Components
- **Database**: Entity Framework Core with PostgreSQL
- **API Layer**: ASP.NET Core Web API
- **Authentication**: JWT tokens with refresh token rotation
- **Validation**: FluentValidation
- **Dependency Injection**: Built-in .NET DI container
- **Middleware**: Custom tenant resolution, error handling, and auth middleware
- **Testing**: xUnit, Moq

### Frontend Components
- **Rendering**: Next.js with both server and client components
- **Styling**: Tailwind CSS for utility-first styling
- **API Client**: Custom fetch wrapper with authentication handling
- **Authentication**: Provider components with context for auth state management
- **Routing**: Next.js App Router with tenant-specific paths
- **Form Handling**: React Hook Form with Zod validation
- **Testing**: React Testing Library, Jest

### Deployment Infrastructure
- **Container Orchestration**: Kubernetes (planned)
- **CI/CD**: GitHub Actions
- **Cloud Provider**: Azure or AWS (to be determined)
- **Monitoring**: Prometheus and Grafana (planned)
- **Logging**: Serilog with structured logging (planned)

## Key Implementation Details

### Multi-Tenant Architecture
- Path-based tenant identification (/tenant/resource)
- Shared database with tenant filtering
- Tenant context propagation across service boundaries
- Data isolation at repository layer

### Authentication Implementation
#### Backend
- JWT tokens with short expiration (15 minutes)
- Refresh tokens with longer expiration (7 days)
- HTTP-only cookies for refresh tokens
- Token rotation on each refresh
- EF Core optimized queries for token validation

#### Frontend
- In-memory token storage with sessionStorage fallback
- Automatic token refresh before expiration
- Focus event listeners for refreshing tokens on tab activation
- AuthProvider component for centralized auth state management
- TenantProvider component for tenant context across the application
- SSR-compatible authentication with hydration protection

### Data Access Strategy
- Repository pattern with specification pattern
- Unit of work for transaction management
- Tenant-aware queries with automatic filtering
- Query optimization with proper indexing

### Cross-Cutting Concerns
- Global exception handling middleware
- Request and response logging
- Performance monitoring
- API versioning
- CORS configuration for secure cross-origin requests

## Development Environment
- Visual Studio 2022 or VS Code for backend development
- VS Code with appropriate extensions for frontend development
- Docker Desktop for containerization
- Git for version control
- Node.js 18+ for frontend development
- .NET SDK 7.0 for backend development
- Python 3.11+ for bot development

## Third-Party Services
- None currently integrated, but planning for:
  - Email service for notifications
  - Object storage for file uploads
  - AI services for automation enhancement

## Development Setup
### Prerequisites
- .NET 7/8 SDK
- Node.js and npm/yarn
- Python 3.8+
- SQL Server/PostgreSQL
- Visual Studio 2022 or VS Code
- Git

### Installation Steps
1. Clone the repository
2. Set up backend services (API and Worker)
3. Configure database connections
4. Install frontend dependencies and build
5. Configure Python environment for automation

### Environment Variables
```env
# Required environment variables
DATABASE_CONNECTION_STRING=connection_string
JWT_SECRET=secret_key
JWT_ISSUER=OpenAutomate
JWT_AUDIENCE=OpenAutomateUsers
JWT_ACCESS_TOKEN_EXPIRATION_MINUTES=15
JWT_REFRESH_TOKEN_EXPIRATION_DAYS=7
ALLOWED_ORIGINS=http://localhost:3000
PYTHON_RUNTIME_PATH=path_to_python
ENABLE_TENANT_ISOLATION=true
DEFAULT_TENANT_SLUG=default
```

## Dependencies
### Production Dependencies
- Entity Framework Core
- ASP.NET Core Identity
- SignalR
- MediatR
- AutoMapper
- Hangfire (for scheduling)
- Serilog (for logging)
- Swagger/OpenAPI (for API documentation)
- Microsoft.AspNetCore.Authentication.JwtBearer

### Development Dependencies
- xUnit (for testing)
- Moq (for mocking)
- Entity Framework Core in-memory database (for testing)
- ESLint/Prettier (for frontend code quality)

## Technical Constraints
- Must support Windows and Linux environments
- Real-time monitoring requires stable WebSocket connections
- Python runtime must be available on bot agent machines
- Network connectivity between components is essential
- Security considerations for package distribution
- Token-based authentication with refresh token mechanism
- Tenant isolation must be enforced at the data access layer

## Multi-Tenant Architecture
- Shared database with tenant filtering approach
- Path-based tenant identification (/{tenant-slug}/api/...)
- Tenant context maintained in HttpContext
- Global query filters for automatic tenant data isolation
- Entity Framework Core shadow properties for tenant tracking
- Cross-tenant operations restricted to system administrators

## Performance Requirements
- Low latency for real-time monitoring updates
- Scalability for large numbers of bot agents
- Efficient database access for reporting
- Optimized package distribution to minimize network traffic
- Responsive UI even with large datasets
- Minimal overhead from tenant filtering in queries

## Security Requirements
- Secure authentication and authorization with JWT and refresh tokens
- Protection of sensitive automation credentials
- Encrypted communication between components
- Secure storage of automation packages
- Comprehensive audit logging for compliance
- Protection against common web vulnerabilities (XSS, CSRF, etc.)
- Token rotation and revocation capabilities
- Secure handling and storage of refresh tokens
- Strict tenant isolation preventing cross-tenant data access
- Protection against tenant identification tampering 