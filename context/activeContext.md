# Active Context

## Current Focus
The current focus is on refining the authentication system and addressing issues with JWT and refresh token implementation. This involves fixing EF Core query translation errors, improving token refresh handling, and enhancing frontend-backend integration. We're also implementing proper provider components for the frontend to handle authentication and tenant context management.

## Recent Changes
### April 4, 2025
- Fixed EF Core query translation error in TokenService by separating database queries from computed property checks
- Modified RefreshToken and RevokeToken methods to avoid using computed properties in LINQ queries
- Implemented AuthProvider component in frontend for centralized authentication management
- Created TenantProvider component for tenant context across the application
- Updated app layout to use the new provider architecture
- Fixed SSR compatibility issues in auth components with the 'use client' directive
- Enhanced token storage with memory-first and sessionStorage fallback strategy
- Updated root layout to include AuthProvider and TenantProvider

### April 3, 2025
- Implemented JWT authentication with refresh token support
- Created `TokenService` and `UserService` for handling authentication
- Refined token management to use direct repository access instead of navigation properties
- Created `AuthController` with endpoints for login, register, and token management
- Cleaned up redundant code, including removing unused methods in `TokenService`
- Improved error handling in token-related operations
- Implemented multi-tenant architecture with path-based tenant resolution
- Created technical design document outlining the system architecture and components
- Generated comprehensive task breakdown for project implementation
- Decided to implement refresh token authentication for improved security

### April 2, 2025
- Created initial project structure with OpenAutomate.API, OpenAutomate.Core, and OpenAutomate.Infrastructure projects
- Set up repository in GitHub for version control
- Initialized backend solution structure

## Next Steps
1. Resolve the 400 (Bad Request) error when refreshing token immediately after login
2. Add roles and permissions to the authentication system
3. Implement token cleanup strategy to periodically remove expired tokens
4. Add database indexes for optimizing token lookups
5. Create API endpoints for managing organizations (tenants)
6. Implement integration tests for authentication and multi-tenant scenarios
7. Implement bot agent registration and management endpoints
8. Create automation package management endpoints
9. Add WebSocket support for real-time monitoring
10. Implement scheduling system for automation tasks
11. Update documentation to reflect latest implementation details

## Active Decisions
### Frontend Authentication Implementation
- Context: Need to determine the best way to handle authentication in the Next.js frontend
- Options:
  - HTTP-only cookies with backend refresh token rotation
  - Local storage with short-lived tokens and manual refresh
  - In-memory token with optional sessionStorage fallback
- Status: Decision made to use in-memory token storage with sessionStorage fallback for better UX while maintaining security

### Token Storage Strategy in Backend
- Context: Need to determine the best way to store and manage refresh tokens
- Options:
  - Store as properties on User entity (navigation properties)
  - Store independently and access directly via repository
- Status: Decision made to use direct repository access for better performance and to avoid concurrency issues

### EF Core Query Optimization
- Context: Need to address EF Core query translation errors with computed properties
- Options:
  - Materialize query results before using computed properties
  - Store computed values in database columns
  - Split queries into two parts (database query + in-memory filtering)
- Status: Decision made to split queries - first fetch data from database without computed properties, then check computed properties in memory

### Frontend Provider Architecture
- Context: Need a consistent way to manage auth state and tenant context across the application
- Options:
  - Use individual hooks
  - Implement provider components with context
  - Use global state management
- Status: Decision made to use provider components with React context for better organization and state sharing

### Token Data Type
- Context: Need to decide how to store token hashes and salts
- Options:
  - Binary storage using byte[] in the database
  - String storage with Base64 encoding
- Status: Decision made to use string storage with Base64 encoding for better portability and easier debugging

### Multi-Tenant Architecture Approach
- Context: Need to implement multi-tenancy for the platform
- Options:
  - Database per tenant
  - Schema per tenant
  - Shared database with tenant filtering
- Status: Decision made to use shared database with tenant filtering for simpler management and better resource utilization

### Tenant Identification Strategy
- Context: Need to determine how tenants are identified in requests
- Options:
  - Subdomain-based (tenant.domain.com)
  - Path-based (domain.com/tenant)
  - Header-based (X-Tenant header)
- Status: Decision made to use path-based tenant identification for better URL readability and compatibility

### Database Provider
- Context: Need to select a database provider for the project
- Options: 
  - SQL Server (familiar, good tooling support)
  - PostgreSQL (open-source, cross-platform)
- Status: Under consideration, leaning toward PostgreSQL for cross-platform support

### Authentication Mechanism
- Context: Need to implement secure authentication for API access
- Options:
  - Basic JWT authentication
  - JWT with refresh token mechanism
  - OAuth 2.0 with identity provider
- Status: Implemented JWT with refresh tokens for enhanced security and user experience

## Current Considerations
- Addressing the immediate token refresh error after login
- Implementing token cleanup strategy (background job vs. on-demand)
- Addressing potential redundant fields in entities (e.g., Created/CreateAt in RefreshToken)
- Optimizing database queries with proper indexes for token lookups
- Implementing proper error handling in authentication flow
- Enhancing tenant data isolation enforcement across the application
- Adding monitoring and logging for authentication failures
- Implementing rate limiting for authentication endpoints
- Ensuring secure storage and transmission of tokens
- Adding authorization policies for resource access control
- Cross-tenant operations for system administrators
- URL structure and routing for tenant-specific resources
- Performance impact of tenant filtering on queries

## Blockers
- Need to fix the token refresh error occurring immediately after login
- Need to implement token cleanup strategy
- Need to define exact communication protocol between API and bot agents
- Need to determine deployment strategy for bot agents

## Dependencies
- Entity Framework Core for data access
- JWT authentication middleware for token validation
- Multi-tenant infrastructure for proper data isolation
- Unit of Work pattern for transaction management
- Tenant context implementation for data isolation
- Next.js frontend with React context providers for authentication and tenant management 