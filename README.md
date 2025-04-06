# OpenAutomate

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

OpenAutomate is an open-source business process automation management platform that provides a cost-effective alternative to commercial automation solutions. It leverages Python for automation execution, ASP.NET Core for backend services, and Next.js for the frontend interface.

## Overview

OpenAutomate enables organizations to create, deploy, monitor, and manage automation processes without vendor lock-in or expensive licensing costs. The platform is built with a multi-tenant architecture, allowing multiple organizations to use the system while ensuring data isolation and security between tenants.

### Key Features

- **User Authentication & Authorization**: JWT-based authentication with refresh tokens
- **Multi-Tenant Architecture**: Shared database with tenant isolation
- **Bot Agent Management**: Register and manage automation agents
- **Automation Package Management**: Create, edit, and deploy automation packages
- **Real-Time Monitoring**: Monitor bot agent status and activities
- **Execution Logging**: Detailed logging of all automation executions
- **Scheduling**: Schedule automation tasks to run at specific times
- **Notifications**: Alerts for automation success, failures, and issues
- **Performance Analytics**: Metrics and visualizations for automation processes
- **Multi-Environment Support**: Development, testing, and production environments

## Architecture

OpenAutomate follows a modern, scalable architecture with clean separation of concerns:

- **Backend**: ASP.NET Core 7.0 API with clean architecture (Core, Infrastructure, API layers)
- **Frontend**: Next.js 14 with App Router, TypeScript, and React
- **Authentication**: JWT with refresh tokens and HTTP-only cookies
- **Multi-Tenant**: Path-based tenant identification with global query filters
- **Database**: Entity Framework Core with PostgreSQL

## Getting Started

### Prerequisites

- .NET SDK 7.0+
- Node.js 18+
- SQL Server or PostgreSQL
- Visual Studio 2022 or VS Code with C# extension

### Backend Setup

```bash
# Clone the repository
git clone https://github.com/yourusername/openautomate.git

# Navigate to backend directory
cd OpenAutomate.Backend

# Restore packages
dotnet restore

# Set up the database (update connection string in appsettings.json first)
dotnet ef database update -p OpenAutomate.Infrastructure -s OpenAutomate.API

# Run the API
dotnet run --project OpenAutomate.API
```

### Frontend Setup

```bash
# Navigate to frontend directory
cd OpenAutomate.Frontend/openautomate-frontend

# Install dependencies
npm install

# Run development server
npm run dev
```

### Environment Variables

Create a `.env` file in the root directory with the following variables:

```
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

## Project Structure

### Backend

The backend solution consists of three main projects:

1. **OpenAutomate.Core**: Domain models, interfaces, and business logic
2. **OpenAutomate.Infrastructure**: Implementations of interfaces, database context
3. **OpenAutomate.API**: API controllers and application configuration

### Frontend

The frontend follows the Next.js App Router structure:

1. **app**: Page routes and layouts
2. **components**: Reusable React components
3. **lib**: Utilities and services
4. **types**: TypeScript type definitions

## Development

Refer to the [Development Guide](./Development_Guide.md) for detailed instructions on:

- Adding new entities
- Creating API endpoints
- Working with multi-tenant architecture
- Creating frontend components and pages
- Best practices for authentication and authorization
- Error handling patterns
- And more

## Testing

### Backend Testing

```bash
# Run unit tests
dotnet test
```

### Frontend Testing

```bash
# Run component tests
npm test

# Run end-to-end tests
npm run cypress
```

## Contributing

We welcome contributions to OpenAutomate! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Run tests to ensure they pass
5. Commit your changes (`git commit -m 'Add some amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

Please make sure to update tests as appropriate and adhere to the existing coding standards.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Thanks to all contributors who have helped shape this project
- Special appreciation to the open-source community for the tools and libraries used 