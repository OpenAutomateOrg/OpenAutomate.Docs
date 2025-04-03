# Technical Context

## Technology Stack
### Frontend
- Framework: Next.js
- UI Library: React with Material-UI or similar component library
- State Management: React Context API/Redux
- Build Tools: Webpack, Babel
- Routing: Next.js with tenant path segment support

### Backend
- Language: C#
- Framework: ASP.NET Core
- Database: SQL Server/PostgreSQL
- API: RESTful API with SignalR for real-time communication
- Worker Service: .NET Worker Service
- Authentication: JWT with refresh token implementation
- Multi-tenancy: Path-based tenant resolution with global query filtering

### Automation
- Language: Python
- Packaging: Standard Python package format
- Execution: Python runtime with virtual environments

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