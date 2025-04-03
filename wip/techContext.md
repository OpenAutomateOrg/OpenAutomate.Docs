# Technical Context

## Technology Stack
### Frontend
- Framework: Next.js
- UI Library: React with Material-UI or similar component library
- State Management: React Context API/Redux
- Build Tools: Webpack, Babel

### Backend
- Language: C#
- Framework: ASP.NET Core
- Database: SQL Server/PostgreSQL
- API: RESTful API with SignalR for real-time communication
- Worker Service: .NET Worker Service

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
ALLOWED_ORIGINS=http://localhost:3000
PYTHON_RUNTIME_PATH=path_to_python
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

## Performance Requirements
- Low latency for real-time monitoring updates
- Scalability for large numbers of bot agents
- Efficient database access for reporting
- Optimized package distribution to minimize network traffic
- Responsive UI even with large datasets

## Security Requirements
- Secure authentication and authorization
- Protection of sensitive automation credentials
- Encrypted communication between components
- Secure storage of automation packages
- Comprehensive audit logging for compliance
- Protection against common web vulnerabilities (XSS, CSRF, etc.) 