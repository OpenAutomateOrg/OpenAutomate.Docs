# OpenAutomate

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

OpenAutomate is an open-source business process automation management platform that provides a cost-effective alternative to commercial automation solutions. It leverages Python for automation execution, ASP.NET Core for backend services, and Next.js for the frontend interface.

## Overview

OpenAutomate enables organizations to create, deploy, monitor, and manage automation processes without vendor lock-in or expensive licensing costs. The platform is built with a multi-tenant architecture, allowing multiple organizations to use the system while ensuring data isolation and security between tenants.

### Why OpenAutomate?

- **Cost-Effective**: Eliminate expensive licensing fees from commercial automation platforms
- **Open Source**: Full control over your automation infrastructure with no vendor lock-in
- **Familiar Technologies**: Built on widely-used technologies (Python, ASP.NET Core, Next.js)
- **Scalable**: Designed to grow with your organization's automation needs
- **Secure**: Robust authentication, authorization, and multi-tenant isolation

### Key Features

- User Authentication & Authorization
- Multi-Tenant Architecture
- Bot Agent Management
- Automation Package Management
- Real-Time Monitoring
- Execution Logging
- Scheduling
- Notifications
- Performance Analytics
- Multi-Environment Support

## Documentation

This repository includes comprehensive documentation to help you understand and contribute to the project:

### Technical Documentation

- [Technical Design Document](./techinal/TechnicalDesignDocument.md) - Architecture, data models, and technical details
- [Development Guide](./Development_Guide.md) - How to add new features, follow project conventions, and best practices
- [Multi-Tenant Architecture](./techinal/MultiTenantArchitecture.md) - Details about the multi-tenant implementation
- [Refresh Token Implementation](./techinal/RefreshTokenImplementation.md) - Authentication system details
- [Project Task Breakdown](./techinal/ProjectTaskBreakdown.md) - Detailed development tasks

### Context Documentation

- [Project Brief](./context/projectbrief.md) - Project overview and core requirements
- [Product Context](./context/productContext.md) - Problem statement and user experience goals
- [System Patterns](./context/systemPatterns.md) - Architecture and design patterns
- [Technical Context](./context/techContext.md) - Tech stack and implementation details
- [Active Context](./context/activeContext.md) - Current focus and recent changes
- [Progress](./context/progress.md) - Project status and next steps

## Getting Started

### Prerequisites

- .NET SDK 8.0
- Node.js 18+
- SQL Server or PostgreSQL
- Visual Studio 2022 or VS Code with C# extension
- Git

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

### Environment Configuration

See the [Technical Context](./context/techContext.md) document for details on required environment variables and configuration settings.

## Project Structure

The project follows a clean architecture approach with separate backend and frontend components. For detailed information about the project structure:

- Backend architecture details: [Technical Design Document](./techinal/TechnicalDesignDocument.md)
- Frontend architecture details: [Development Guide](./Development_Guide.md)

## Development Workflow

Before starting development, we recommend reading the following documents:

1. First, review the [Project Brief](./context/projectbrief.md) to understand the core requirements
2. Next, read the [Technical Design Document](./techinal/TechnicalDesignDocument.md) for architecture overview
3. Then review the [Development Guide](./Development_Guide.md) for coding standards and patterns
4. Check [Active Context](./context/activeContext.md) and [Progress](./context/progress.md) for current status

For specific development tasks, refer to the [Development Guide](./Development_Guide.md) which provides detailed instructions for common development scenarios.

## Contributing

We welcome contributions to OpenAutomate! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Run tests to ensure they pass
5. Commit your changes (`git commit -m 'Add some amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

Please make sure to update tests as appropriate and adhere to the existing coding standards described in the [Development Guide](./Development_Guide.md).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details. 
