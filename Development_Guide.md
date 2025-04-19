# OpenAutomate Development Guide

This guide provides essential information for developers working on the OpenAutomate project. It covers both backend and frontend development, including how to add new features, follow project conventions, and leverage existing components.

## Project Repositories

OpenAutomate is organized into several repositories:

- [OpenAutomate.Backend](https://github.com/OpenAutomateOrg/OpenAutomate.Backend) - ASP.NET Core backend API and services
- [OpenAutomate.Frontend](https://github.com/OpenAutomateOrg/OpenAutomate.Frontend) - Next.js web application frontend
- [OpenAutomate.Docs](https://github.com/OpenAutomateOrg/OpenAutomate.Docs) - Project documentation (this repository)
- [OpenAutomate.BotAgent](https://github.com/OpenAutomateOrg/OpenAutomate.BotAgent) - Python-based automation agent

## Table of Contents

1. [Project Architecture Overview](#project-architecture-overview)
2. [Backend Development](#backend-development)
   - [Project Structure](#backend-project-structure)
   - [Multi-Tenant Development](#multi-tenant-development)
   - [Adding New Entities](#adding-new-entities)
   - [Creating API Endpoints](#creating-api-endpoints)
   - [Authentication & Authorization](#authentication--authorization)
   - [Error Handling](#error-handling)
   - [EF Core Best Practices](#ef-core-best-practices)
3. [Frontend Development](#frontend-development)
   - [Project Structure](#frontend-project-structure)
   - [Authentication & Tenant Context](#authentication--tenant-context)
   - [Creating New Pages](#creating-new-pages)
   - [Creating New Components](#creating-new-components)
   - [State Management](#state-management)
   - [API Integration](#api-integration)
   - [Server vs. Client Components](#server-vs-client-components)
4. [Development Workflow](#development-workflow)
   - [Setting Up the Development Environment](#setting-up-the-development-environment)
   - [Running the Application](#running-the-application)
   - [Testing](#testing)
   - [Pull Request Process](#pull-request-process)

## Project Architecture Overview

OpenAutomate is an open-source business process automation management platform built with:

- **Backend**: ASP.NET Core 8.0 with clean architecture
- **Frontend**: Next.js 14 with App Router and TypeScript
- **Authentication**: JWT with refresh tokens (HTTP-only cookies)
- **Multi-Tenant Architecture**: Shared database with tenant filtering
- **Database**: Entity Framework Core with global query filters for tenant isolation

The architecture follows these core principles:

1. **Clean Architecture**: Separation of concerns with Core, Infrastructure, and API layers
2. **Multi-Tenancy**: All tenant-specific data is isolated through global query filters
3. **Provider-Based Frontend**: Authentication and tenant state managed through context providers

## Backend Development

### Backend Project Structure

The backend solution consists of three main projects:

1. **OpenAutomate.Core**: Contains domain models, interfaces, and business logic
   - Domain/Entities: Domain models
   - Domain/Interfaces: Interfaces for repositories and services
   - Domain/Dto: Data Transfer Objects

2. **OpenAutomate.Infrastructure**: Contains implementations of interfaces
   - DbContext: Database context and configuration
   - Repositories: Repository implementations
   - Services: Service implementations
   - Middleware: Custom middleware components

3. **OpenAutomate.API**: Contains API controllers and configuration
   - Controllers: API endpoints grouped by resource
   - Middleware: API-specific middleware

### Multi-Tenant Development

When developing for multi-tenant scenarios, follow these guidelines:

1. **Tenant-Aware Entities**: All tenant-specific entities must include:
   ```csharp
   public Guid OrganizationId { get; set; }
   public virtual Organization Organization { get; set; }
   ```

2. **Repository Usage**: Always use the repository pattern to access data. The repositories automatically apply tenant filtering.
   ```csharp
   // Example: Get filtered data for current tenant
   var items = await _repository.GetAllAsync(filter: null);
   ```

3. **Tenant Context**: Use the tenant context to get the current tenant ID:
   ```csharp
   public class MyService
   {
       private readonly ITenantContext _tenantContext;
       
       public MyService(ITenantContext tenantContext)
       {
           _tenantContext = tenantContext;
       }
       
       public void DoSomething()
       {
           var tenantId = _tenantContext.CurrentTenantId;
           // Use tenant ID as needed
       }
   }
   ```

4. **Cross-Tenant Operations**: For admin functions that need to work across tenants:
   ```csharp
   // Use IgnoreTenantFilter to bypass tenant filtering
   using (_tenantContext.IgnoreTenantFilter())
   {
       var allItems = await _repository.GetAllAsync(filter: null);
   }
   ```

### Adding New Entities

Follow these steps to add a new entity:

1. **Create the Entity Class** in `OpenAutomate.Core/Domain/Entities`:
   ```csharp
   public class MyEntity : BaseEntity
   {
       public string Name { get; set; }
       public string Description { get; set; }
       
       // Tenant relationship - required for tenant isolation
       public Guid OrganizationId { get; set; }
       public virtual Organization Organization { get; set; }
       
       // Any other relationships
       public Guid CreatedById { get; set; }
       public virtual User CreatedBy { get; set; }
   }
   ```

2. **Create DTOs** in `OpenAutomate.Core/Domain/Dto`:
   ```csharp
   // Request DTO
   public class CreateMyEntityRequest
   {
       public string Name { get; set; }
       public string Description { get; set; }
   }
   
   // Response DTO
   public class MyEntityResponse
   {
       public Guid Id { get; set; }
       public string Name { get; set; }
       public string Description { get; set; }
       public DateTime CreatedAt { get; set; }
       public Guid CreatedById { get; set; }
   }
   ```

3. **Add DbSet to ApplicationDbContext** in `OpenAutomate.Infrastructure/DbContext/ApplicationDbContext.cs`:
   ```csharp
   public DbSet<MyEntity> MyEntities { get; set; }
   ```

4. **Configure Entity** in `OnModelCreating` if needed:
   ```csharp
   protected override void OnModelCreating(ModelBuilder modelBuilder)
   {
       // Existing code...
       
       modelBuilder.Entity<MyEntity>()
           .HasOne(e => e.CreatedBy)
           .WithMany()
           .HasForeignKey(e => e.CreatedById)
           .OnDelete(DeleteBehavior.Restrict);
   }
   ```

5. **Create Migration**:
   ```
   dotnet ef migrations add AddMyEntity -p OpenAutomate.Infrastructure -s OpenAutomate.API
   ```

6. **Create Interface** in `OpenAutomate.Core/Domain/Interfaces/IServices`:
   ```csharp
   public interface IMyEntityService
   {
       Task<MyEntityResponse> CreateAsync(CreateMyEntityRequest request);
       Task<List<MyEntityResponse>> GetAllAsync();
       Task<MyEntityResponse> GetByIdAsync(Guid id);
       Task<MyEntityResponse> UpdateAsync(Guid id, UpdateMyEntityRequest request);
       Task<bool> DeleteAsync(Guid id);
   }
   ```

7. **Implement Service** in `OpenAutomate.Infrastructure/Services`:
   ```csharp
   public class MyEntityService : IMyEntityService
   {
       private readonly IUnitOfWork _unitOfWork;
       private readonly ITenantContext _tenantContext;
       
       public MyEntityService(IUnitOfWork unitOfWork, ITenantContext tenantContext)
       {
           _unitOfWork = unitOfWork;
           _tenantContext = tenantContext;
       }
       
       // Implement methods from interface
   }
   ```

8. **Register Service** in `Program.cs`:
   ```csharp
   builder.Services.AddScoped<IMyEntityService, MyEntityService>();
   ```

### Implementing Multi-Tenant Entities

When creating entities that belong to a specific tenant (organization unit), follow these steps:

1. **Use the TenantEntity Base Class**:
   ```csharp
   public class MyTenantEntity : TenantEntity
   {
       public string Name { get; set; }
       public string Description { get; set; }
       
       // Add entity-specific properties
       public bool IsActive { get; set; }
       
       // Add relationships (if needed)
       public Guid? RelatedEntityId { get; set; }
       public virtual RelatedEntity RelatedEntity { get; set; }
   }
   ```

2. **Alternative Approach Using ITenantEntity**:
   If you need to inherit from another base class, implement the ITenantEntity interface:
   ```csharp
   public class MySpecialEntity : BaseEntity, ITenantEntity
   {
       // ITenantEntity implementation
       public Guid OrganizationUnitId { get; set; }
       public virtual OrganizationUnit OrganizationUnit { get; set; }
       
       // Entity-specific properties
       public string Name { get; set; }
   }
   ```

3. **Configure Entity in ApplicationDbContext**:
   ```csharp
   protected override void OnModelCreating(ModelBuilder modelBuilder)
   {
       // Existing code...
       
       // Configure tenant entity
       modelBuilder.Entity<MyTenantEntity>()
           .HasOne(e => e.OrganizationUnit)
           .WithMany()
           .HasForeignKey(e => e.OrganizationUnitId)
           .OnDelete(DeleteBehavior.Restrict);
           
       // If using global query filters for tenant isolation:
       modelBuilder.Entity<MyTenantEntity>().HasQueryFilter(
           e => tenantContextAccessor.TenantContext.IgnoreTenantFilter || 
                e.OrganizationUnitId == tenantContextAccessor.TenantContext.CurrentTenantId);
   }
   ```

4. **Register in IUnitOfWork**:
   ```csharp
   public interface IUnitOfWork : IDisposable
   {
       // Existing repositories...
       
       IRepository<MyTenantEntity> MyTenantEntities { get; }
       
       Task<int> CompleteAsync();
   }
   ```

5. **Add Property in UnitOfWork Implementation**:
   ```csharp
   public class UnitOfWork : IUnitOfWork
   {
       // Existing code...
       
       private IRepository<MyTenantEntity> _myTenantEntityRepository;
       
       public IRepository<MyTenantEntity> MyTenantEntities => 
           _myTenantEntityRepository ??= new Repository<MyTenantEntity>(_context);
   }
   ```

By following these patterns, you ensure your entity participates in the multi-tenant architecture and works with the global query filters for automatic tenant data isolation.

### Creating API Endpoints

Follow these steps to add a new controller:

1. **Create Controller** in `OpenAutomate.API/Controllers`:
   ```csharp
   [Route("{tenant}/api/[controller]")]
   [ApiController]
   [Authorize]
   public class MyEntitiesController : ControllerBase
   {
       private readonly IMyEntityService _myEntityService;
       
       public MyEntitiesController(IMyEntityService myEntityService)
       {
           _myEntityService = myEntityService;
       }
       
       [HttpGet]
       public async Task<IActionResult> GetAll()
       {
           try
           {
               var entities = await _myEntityService.GetAllAsync();
               return Ok(entities);
           }
           catch (Exception ex)
           {
               return StatusCode(500, new { message = "An error occurred while processing your request." });
           }
       }
       
       [HttpGet("{id}")]
       public async Task<IActionResult> GetById(Guid id)
       {
           try
           {
               var entity = await _myEntityService.GetByIdAsync(id);
               if (entity == null)
                   return NotFound();
               
               return Ok(entity);
           }
           catch (Exception ex)
           {
               return StatusCode(500, new { message = "An error occurred while processing your request." });
           }
       }
       
       [HttpPost]
       public async Task<IActionResult> Create([FromBody] CreateMyEntityRequest request)
       {
           try
           {
               var entity = await _myEntityService.CreateAsync(request);
               return CreatedAtAction(nameof(GetById), new { id = entity.Id }, entity);
           }
           catch (Exception ex)
           {
               return StatusCode(500, new { message = "An error occurred while processing your request." });
           }
       }
       
       [HttpPut("{id}")]
       public async Task<IActionResult> Update(Guid id, [FromBody] UpdateMyEntityRequest request)
       {
           try
           {
               var entity = await _myEntityService.UpdateAsync(id, request);
               if (entity == null)
                   return NotFound();
               
               return Ok(entity);
           }
           catch (Exception ex)
           {
               return StatusCode(500, new { message = "An error occurred while processing your request." });
           }
       }
       
       [HttpDelete("{id}")]
       public async Task<IActionResult> Delete(Guid id)
       {
           try
           {
               var result = await _myEntityService.DeleteAsync(id);
               if (!result)
                   return NotFound();
               
               return NoContent();
           }
           catch (Exception ex)
           {
               return StatusCode(500, new { message = "An error occurred while processing your request." });
           }
       }
   }
   ```

### Authentication & Authorization

The OpenAutomate backend uses JWT with refresh tokens for authentication:

1. **Securing Endpoints**:
   ```csharp
   // Require authentication
   [Authorize]
   public class MyController : ControllerBase
   {
       // Controller code
   }
   
   // Require specific role
   [Authorize(Roles = "Admin")]
   public IActionResult AdminAction()
   {
       // Action code
   }
   ```

2. **Getting Current User**:
   ```csharp
   public class MyService
   {
       private readonly IHttpContextAccessor _httpContextAccessor;
       
       public MyService(IHttpContextAccessor httpContextAccessor)
       {
           _httpContextAccessor = httpContextAccessor;
       }
       
       public void DoSomething()
       {
           var userId = _httpContextAccessor.HttpContext.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
           // Use user ID as needed
       }
   }
   ```

3. **Refresh Token Considerations**:
   - Refresh tokens are stored in HTTP-only cookies
   - Backend checks and validates tokens automatically
   - Use the EF Core query optimization technique described in the documentation

### JWT Authentication Implementation

OpenAutomate implements a comprehensive JWT authentication system with refresh tokens. Here's a detailed explanation of how it works:

1. **JWT Token Structure**:
   ```csharp
   // Token generation in TokenService
   var claims = new[]
   {
       new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
       new Claim(JwtRegisteredClaimNames.Email, user.Email),
       new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
       new Claim("tenant", tenantId.ToString())  // Include tenant context in token
   };
   
   var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["JWT:Secret"]));
   var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
   
   var token = new JwtSecurityToken(
       issuer: _configuration["JWT:ValidIssuer"],
       audience: _configuration["JWT:ValidAudience"],
       claims: claims,
       expires: DateTime.Now.AddMinutes(Convert.ToDouble(_configuration["JWT:TokenValidityInMinutes"])),
       signingCredentials: creds
   );
   ```

2. **Refresh Token Implementation**:
   ```csharp
   // RefreshToken entity
   public class RefreshToken : BaseEntity
   {
       public string Token { get; set; }
       public DateTime Expires { get; set; }
       public bool IsExpired => DateTime.UtcNow >= Expires;
       public DateTime Created { get; set; }
       public DateTime? Revoked { get; set; }
       public bool IsRevoked => Revoked != null;
       public bool IsActive => !IsRevoked && !IsExpired;
       public string? ReplacedByToken { get; set; }
       public Guid UserId { get; set; }
       public virtual User User { get; set; }
   }
   ```

3. **Token Service Interface**:
   ```csharp
   public interface ITokenService
   {
       string GenerateAccessToken(User user, Guid? tenantId = null);
       RefreshToken GenerateRefreshToken(string ipAddress);
       Task<AuthResponse> RefreshTokenAsync(string token, string ipAddress);
       Task<bool> RevokeTokenAsync(string token, string ipAddress);
       ClaimsPrincipal GetPrincipalFromExpiredToken(string token);
   }
   ```

4. **JWT Authentication Configuration**:
   ```csharp
   // In Program.cs
   builder.Services.AddAuthentication(options =>
   {
       options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
       options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
   }).AddJwtBearer(options =>
   {
       options.SaveToken = true;
       options.RequireHttpsMetadata = false;
       options.TokenValidationParameters = new TokenValidationParameters
       {
           ValidateIssuer = true,
           ValidateAudience = true,
           ValidateLifetime = true,
           ValidateIssuerSigningKey = true,
           ClockSkew = TimeSpan.Zero,
           ValidIssuer = builder.Configuration["JWT:ValidIssuer"],
           ValidAudience = builder.Configuration["JWT:ValidAudience"],
           IssuerSigningKey = new SymmetricSecurityKey(
               Encoding.UTF8.GetBytes(builder.Configuration["JWT:Secret"]))
       };
   });
   ```

5. **Login Workflow**:
   ```csharp
   [HttpPost("login")]
   public async Task<IActionResult> Login([FromBody] LoginRequest model)
   {
       // Validate user credentials
       var user = await _unitOfWork.Users.GetFirstOrDefaultAsync(u => u.Email == model.Email);
       if (user == null || !VerifyPassword(model.Password, user.PasswordHash, user.PasswordSalt))
           return Unauthorized(new { message = "Invalid email or password" });
       
       // Check if user has access to the tenant
       var tenantId = _tenantContext.CurrentTenantId;
       if (tenantId.HasValue)
       {
           var hasAccess = await _unitOfWork.OrganizationUnitUsers.AnyAsync(
               ou => ou.UserId == user.Id && ou.OrganizationUnitId == tenantId.Value);
           
           if (!hasAccess)
               return Unauthorized(new { message = "User does not have access to this tenant" });
       }
       
       // Generate tokens
       var accessToken = _tokenService.GenerateAccessToken(user, tenantId);
       var refreshToken = _tokenService.GenerateRefreshToken(GetIpAddress());
       
       // Save refresh token
       user.RefreshTokens ??= new List<RefreshToken>();
       user.RefreshTokens.Add(refreshToken);
       _unitOfWork.Users.Update(user);
       await _unitOfWork.CompleteAsync();
       
       // Set refresh token cookie
       SetRefreshTokenCookie(refreshToken.Token);
       
       return Ok(new AuthResponse
       {
           AccessToken = accessToken,
           RefreshToken = refreshToken.Token,
           ExpiresIn = int.Parse(_configuration["JWT:TokenValidityInMinutes"]) * 60
       });
   }
   ```

6. **Refresh Token Endpoint**:
   ```csharp
   [HttpPost("refresh-token")]
   public async Task<IActionResult> RefreshToken()
   {
       var refreshToken = Request.Cookies["refreshToken"];
       
       if (string.IsNullOrEmpty(refreshToken))
           return BadRequest(new { message = "Refresh token is required" });
       
       var response = await _tokenService.RefreshTokenAsync(refreshToken, GetIpAddress());
       
       if (response == null)
           return Unauthorized(new { message = "Invalid token" });
       
       SetRefreshTokenCookie(response.RefreshToken);
       
       return Ok(response);
   }
   ```

7. **JWT Authentication Middleware**:
   ```csharp
   public class JwtAuthenticationMiddleware
   {
       private readonly RequestDelegate _next;
       
       public JwtAuthenticationMiddleware(RequestDelegate next)
       {
           _next = next;
       }
       
       public async Task InvokeAsync(HttpContext context, IUnitOfWork unitOfWork, 
           ITokenService tokenService, ITenantContext tenantContext)
       {
           // Extract token from Authorization header
           var token = context.Request.Headers["Authorization"].FirstOrDefault()?.Split(" ").Last();
           
           if (!string.IsNullOrEmpty(token))
           {
               try
               {
                   // Validate token and set user context
                   var principal = tokenService.GetPrincipalFromExpiredToken(token);
                   var userId = principal.FindFirst(ClaimTypes.NameIdentifier)?.Value;
                   
                   if (!string.IsNullOrEmpty(userId) && Guid.TryParse(userId, out var userIdGuid))
                   {
                       // Set authenticated user context
                       context.User = principal;
                       
                       // Optionally check if token has a tenant claim and set tenant context
                       var tenantClaim = principal.FindFirst("tenant")?.Value;
                       if (!string.IsNullOrEmpty(tenantClaim) && Guid.TryParse(tenantClaim, out var tenantId))
                       {
                           // Set tenant context only if it matches the URL tenant
                           var urlTenantId = tenantContext.CurrentTenantId;
                           if (!urlTenantId.HasValue || urlTenantId.Value == tenantId)
                           {
                               tenantContext.SetCurrentTenant(tenantId);
                           }
                       }
                   }
               }
               catch
               {
                   // Token validation error - continue without setting user context
               }
           }
           
           await _next(context);
       }
   }
   ```

By implementing this comprehensive authentication system, OpenAutomate ensures secure access to tenant-specific resources and maintains proper tenant isolation.

### Error Handling

Follow these guidelines for error handling:

1. **Service Layer**:
   ```csharp
   public async Task<MyEntityResponse> GetByIdAsync(Guid id)
   {
       try
       {
           var entity = await _unitOfWork.MyEntities.GetByIdAsync(id);
           if (entity == null)
               throw new NotFoundException($"Entity with ID {id} not found");
           
           return _mapper.Map<MyEntityResponse>(entity);
       }
       catch (Exception ex)
       {
           _logger.LogError(ex, "Error getting entity by ID {Id}", id);
           throw;
       }
   }
   ```

2. **Controller Layer**:
   ```csharp
   [HttpGet("{id}")]
   public async Task<IActionResult> GetById(Guid id)
   {
       try
       {
           var entity = await _myEntityService.GetByIdAsync(id);
           return Ok(entity);
       }
       catch (NotFoundException ex)
       {
           return NotFound(new { message = ex.Message });
       }
       catch (Exception ex)
       {
           return StatusCode(500, new { message = "An error occurred while processing your request." });
       }
   }
   ```

### Tenant Resolution Middleware

OpenAutomate uses a dedicated middleware component to handle tenant resolution in the multi-tenant architecture. Understanding how this works is essential for developing tenant-aware features:

1. **Middleware Registration**:
   The TenantResolutionMiddleware is registered in the application pipeline:
   ```csharp
   app.UseMiddleware<TenantResolutionMiddleware>();
   ```

2. **Tenant Resolution Process**:
   ```csharp
   public class TenantResolutionMiddleware
   {
       private readonly RequestDelegate _next;
       
       public TenantResolutionMiddleware(RequestDelegate next)
       {
           _next = next;
       }
       
       public async Task InvokeAsync(HttpContext context, IUnitOfWork unitOfWork, ITenantContext tenantContext)
       {
           // Extract tenant slug from the route
           var tenantSlug = context.Request.RouteValues["tenant"]?.ToString();
           
           if (!string.IsNullOrEmpty(tenantSlug))
           {
               // Look up the tenant by slug
               var organizationUnit = await unitOfWork.OrganizationUnits
                   .GetFirstOrDefaultAsync(ou => ou.Slug == tenantSlug);
                   
               if (organizationUnit != null)
               {
                   // Set the current tenant context
                   tenantContext.SetCurrentTenant(organizationUnit.Id);
               }
               else
               {
                   // Handle tenant not found
                   context.Response.StatusCode = 404;
                   await context.Response.WriteAsJsonAsync(new { error = "Tenant not found" });
                   return;
               }
           }
           
           // Continue processing the request
           await _next(context);
       }
   }
   ```

3. **Tenant Context Usage**:
   The ITenantContext interface provides access to the current tenant:
   ```csharp
   public interface ITenantContext
   {
       Guid? CurrentTenantId { get; }
       bool IgnoreTenantFilter { get; }
       void SetCurrentTenant(Guid tenantId);
       IDisposable IgnoreTenantFilter();
   }
   ```

4. **Using Tenant Context in Services**:
   ```csharp
   public class MyService
   {
       private readonly ITenantContext _tenantContext;
       private readonly IUnitOfWork _unitOfWork;
       
       public MyService(ITenantContext tenantContext, IUnitOfWork unitOfWork)
       {
           _tenantContext = tenantContext;
           _unitOfWork = unitOfWork;
       }
       
       public async Task<IEnumerable<MyEntity>> GetEntitiesForCurrentTenant()
       {
           // The tenant filter is automatically applied
           return await _unitOfWork.MyEntities.GetAllAsync();
       }
       
       public async Task<IEnumerable<MyEntity>> GetEntitiesAcrossTenants()
       {
           // Temporarily disable tenant filtering
           using (_tenantContext.IgnoreTenantFilter())
           {
               return await _unitOfWork.MyEntities.GetAllAsync();
           }
       }
   }
   ```

5. **URL Structure for Tenant-Specific Routes**:
   The standard URL pattern for tenant-specific routes is:
   ```
   /{tenant-slug}/api/controller
   ```
   
   Example:
   ```
   /finance/api/botAgents
   /hr/api/assets
   ```

Understanding this middleware is crucial for developing features that respect tenant boundaries and for implementing tenant-aware business logic.

### EF Core Best Practices

When working with Entity Framework Core:

1. **Avoid Using Computed Properties in LINQ Queries**:
   ```csharp
   // Incorrect - will cause translation error
   var token = await _unitOfWork.RefreshTokens.FirstOrDefaultAsync(
       t => t.Token == refreshToken && !t.IsRevoked && !t.IsExpired);
   
   // Correct - query database properties, then check computed properties
   var token = await _unitOfWork.RefreshTokens.FirstOrDefaultAsync(
       t => t.Token == refreshToken);
   
   if (token == null || token.IsRevoked || token.IsExpired)
   {
       // Handle invalid token
   }
   ```

2. **Use Include for Loading Related Entities**:
   ```csharp
   var entity = await _unitOfWork.MyEntities.GetFirstOrDefaultAsync(
       e => e.Id == id,
       includes: e => e.CreatedBy);
   ```

3. **Use Repository Methods for Standard Operations**:
   ```csharp
   // Add entity
   await _unitOfWork.MyEntities.AddAsync(entity);
   
   // Update entity
   await _unitOfWork.MyEntities.UpdateAsync(entity);
   
   // Delete entity
   await _unitOfWork.MyEntities.DeleteAsync(entity);
   
   // Save changes
   await _unitOfWork.CompleteAsync();
   ```

### Repository Pattern Implementation Details

The OpenAutomate platform implements the Repository Pattern with a generic `IRepository<TEntity>` interface and a Unit of Work pattern. Here's how to effectively use these patterns:

1. **IRepository<TEntity> Interface**:
   The interface provides methods for common database operations:
   ```csharp
   public interface IRepository<TEntity> where TEntity : class
   {
       Task<TEntity> GetByIdAsync(Guid id);
       Task<TEntity> GetFirstOrDefaultAsync(Expression<Func<TEntity, bool>> filter = null,
           params Expression<Func<TEntity, object>>[] includes);
       Task<IEnumerable<TEntity>> GetAllAsync(Expression<Func<TEntity, bool>> filter = null,
           Func<IQueryable<TEntity>, IOrderedQueryable<TEntity>> orderBy = null,
           params Expression<Func<TEntity, object>>[] includes);
       Task<bool> AnyAsync(Expression<Func<TEntity, bool>> filter = null);
       Task AddAsync(TEntity entity);
       Task AddRangeAsync(IEnumerable<TEntity> entities);
       void Remove(TEntity entity);
       void RemoveRange(IEnumerable<TEntity> entities);
       void Update(TEntity entity);
       // ... other methods
   }
   ```

2. **Querying with LINQ Expressions**:
   ```csharp
   // Get by ID
   var entity = await _unitOfWork.MyEntities.GetByIdAsync(id);
   
   // Get first entity matching a condition
   var user = await _unitOfWork.Users.GetFirstOrDefaultAsync(
       u => u.Email == email);
   
   // Get all entities matching a condition with included related data
   var botAgents = await _unitOfWork.BotAgents.GetAllAsync(
       filter: b => b.Status == "Active" && b.IsActive == true,
       orderBy: q => q.OrderByDescending(b => b.LastConnected),
       includes: b => b.Owner, b => b.AssetBotAgents);
   
   // Check if any entity matches a condition
   bool exists = await _unitOfWork.Users.AnyAsync(u => u.Email == email);
   ```

3. **Saving Changes**:
   All changes made through repositories are tracked by Entity Framework Core but are not persisted until you call `CompleteAsync()` on the Unit of Work:
   ```csharp
   // Add a new entity
   await _unitOfWork.BotAgents.AddAsync(botAgent);
   
   // Make changes to another entity
   user.LastLoginAt = DateTime.UtcNow;
   _unitOfWork.Users.Update(user);
   
   // Both operations are part of the same transaction
   // and will only be saved when calling CompleteAsync
   await _unitOfWork.CompleteAsync();
   ```

4. **Tenant Awareness**:
   When working with tenant-specific entities, the tenant context is automatically applied through global query filters:
   ```csharp
   // This query will automatically filter by the current tenant
   var assets = await _unitOfWork.Assets.GetAllAsync();
   
   // For cross-tenant operations (admin only), use the TenantContext
   using (_tenantContext.IgnoreTenantFilter())
   {
       var allAssets = await _unitOfWork.Assets.GetAllAsync();
   }
   ```

5. **Common Repository Patterns**:
   ```csharp
   // Pagination
   var pageSize = 10;
   var pageNumber = 1;
   var query = _unitOfWork.BotAgents.GetAllAsync(
       filter: null,
       orderBy: q => q.OrderBy(b => b.Name))
       .Skip((pageNumber - 1) * pageSize)
       .Take(pageSize);
   
   // Eager loading related data
   var package = await _unitOfWork.AutomationPackages.GetFirstOrDefaultAsync(
       p => p.Id == packageId,
       includes: p => p.PackageVersions);
   
   // Filtering with multiple conditions
   var executions = await _unitOfWork.Executions.GetAllAsync(
       e => e.Status == "Completed" && e.CompletedAt >= DateTime.UtcNow.AddDays(-7));
   ```

These patterns ensure consistent data access throughout the application while maintaining tenant isolation and proper transaction management.

## Frontend Development

### Frontend Project Structure

The frontend project follows the Next.js App Router structure:

1. **app**: Page routes and layouts
   - `layout.tsx`: Root layout with providers
   - `(auth)`: Authentication routes (login, register)
   - `(tenant)`: Tenant-specific routes

2. **components**: Reusable components
   - `auth`: Authentication components
   - `layouts`: Layout components including app-layout
   - `ui`: UI components like buttons, forms, etc.
   - `providers`: Context providers

3. **lib**: Utilities and services
   - `api`: API client and utilities
   - `hooks`: Custom React hooks
   - `config.ts`: Application configuration

4. **types**: TypeScript type definitions

### Authentication & Tenant Context

The frontend uses provider components for authentication and tenant context:

1. **Using Authentication**:
   ```tsx
   'use client';
   
   import { useAuth } from '@/lib/hooks/use-auth';
   
   export default function MyComponent() {
     const { user, isAuthenticated, isLoading, logout } = useAuth();
     
     if (isLoading) {
       return <div>Loading...</div>;
     }
     
     if (!isAuthenticated) {
       return <div>You need to login</div>;
     }
     
     return (
       <div>
         <h1>Welcome, {user?.name}</h1>
         <button onClick={logout}>Logout</button>
       </div>
     );
   }
   ```

2. **Using Tenant Context**:
   ```tsx
   'use client';
   
   import { useTenant } from '@/components/providers/TenantProvider';
   
   export default function MyComponent() {
     const { currentTenant, isLoading } = useTenant();
     
     if (isLoading) {
       return <div>Loading...</div>;
     }
     
     return (
       <div>
         <h1>Current Tenant: {currentTenant}</h1>
       </div>
     );
   }
   ```

3. **Server-Side Rendering Considerations**:
   - Mark components using hooks with `'use client'`
   - Use the `mounted` state pattern to prevent hydration mismatches
   ```tsx
   'use client';
   
   import { useState, useEffect } from 'react';
   
   export default function MyComponent() {
     const [mounted, setMounted] = useState(false);
     
     useEffect(() => {
       setMounted(true);
     }, []);
     
     // Use the same loading state for both server and client
     if (!mounted) {
       return <div>Loading...</div>;
     }
     
     // Client-side only content
     return <div>Client content</div>;
   }
   ```

### Creating New Pages

To create a new page in the Next.js app:

1. **Create a New Page File** in the appropriate directory:
   ```tsx
   // app/(tenant)/[tenant]/my-feature/page.tsx
   
   import { Metadata } from 'next';
   
   export const metadata: Metadata = {
     title: 'My Feature - OpenAutomate',
     description: 'Description of my feature',
   };
   
   export default function MyFeaturePage() {
     return (
       <div className="container mx-auto p-4">
         <h1 className="text-2xl font-bold mb-4">My Feature</h1>
         <div className="bg-white shadow rounded-lg p-6">
           {/* Page content */}
         </div>
       </div>
     );
   }
   ```

2. **For Client Components** (if needed):
   ```tsx
   'use client';
   
   import { useState } from 'react';
   
   export default function MyClientPage() {
     const [data, setData] = useState([]);
     
     return (
       <div>
         {/* Client component content */}
       </div>
     );
   }
   ```

3. **For Pages with Data Fetching**:
   ```tsx
   // Server Component approach
   import { apiClient } from '@/lib/api/server-client';
   
   export default async function MyDataPage({ params }) {
     const { tenant } = params;
     const data = await apiClient.get(`/${tenant}/api/my-entities`);
     
     return (
       <div>
         {/* Render data */}
       </div>
     );
   }
   
   // Client Component approach
   'use client';
   
   import { useEffect, useState } from 'react';
   import { apiClient } from '@/lib/api/client';
   
   export default function MyDataPage({ params }) {
     const { tenant } = params;
     const [data, setData] = useState([]);
     const [isLoading, setIsLoading] = useState(true);
     
     useEffect(() => {
       const fetchData = async () => {
         try {
           const response = await apiClient.get(`/my-entities`, { tenant });
           setData(response.data);
         } catch (error) {
           console.error('Error fetching data:', error);
         } finally {
           setIsLoading(false);
         }
       };
       
       fetchData();
     }, [tenant]);
     
     if (isLoading) {
       return <div>Loading...</div>;
     }
     
     return (
       <div>
         {/* Render data */}
       </div>
     );
   }
   ```

### Creating New Components

To create a reusable component:

1. **Create a New Component File**:
   ```tsx
   // components/ui/MyComponent.tsx
   
   interface MyComponentProps {
     title: string;
     children: React.ReactNode;
   }
   
   export function MyComponent({ title, children }: MyComponentProps) {
     return (
       <div className="bg-white shadow rounded-lg p-4">
         <h2 className="text-xl font-semibold mb-2">{title}</h2>
         <div>{children}</div>
       </div>
     );
   }
   ```

2. **For Client Components**:
   ```tsx
   'use client';
   
   import { useState } from 'react';
   
   interface MyClientComponentProps {
     initialValue: number;
     onChange?: (value: number) => void;
   }
   
   export function MyClientComponent({ initialValue, onChange }: MyClientComponentProps) {
     const [value, setValue] = useState(initialValue);
     
     const handleClick = () => {
       const newValue = value + 1;
       setValue(newValue);
       onChange?.(newValue);
     };
     
     return (
       <div>
         <p>Value: {value}</p>
         <button 
           onClick={handleClick}
           className="px-4 py-2 bg-blue-500 text-white rounded"
         >
           Increment
         </button>
       </div>
     );
   }
   ```

### State Management

The application uses React Context for global state management:

1. **Creating a Context Provider**:
   ```tsx
   'use client';
   
   import { createContext, useContext, useState, ReactNode } from 'react';
   
   // Define context type
   interface MyContextType {
     value: string;
     setValue: (value: string) => void;
   }
   
   // Create context with default value
   const MyContext = createContext<MyContextType>({
     value: '',
     setValue: () => {},
   });
   
   // Create provider component
   export function MyProvider({ children }: { children: ReactNode }) {
     const [value, setValue] = useState('');
     
     return (
       <MyContext.Provider value={{ value, setValue }}>
         {children}
       </MyContext.Provider>
     );
   }
   
   // Create hook for using the context
   export function useMyContext() {
     return useContext(MyContext);
   }
   ```

2. **Using the Context**:
   ```tsx
   'use client';
   
   import { useMyContext } from '@/components/providers/MyProvider';
   
   export function MyComponent() {
     const { value, setValue } = useMyContext();
     
     return (
       <div>
         <p>Value: {value}</p>
         <input
           type="text"
           value={value}
           onChange={(e) => setValue(e.target.value)}
           className="border p-2 rounded"
         />
       </div>
     );
   }
   ```

3. **Adding Provider to Layout**:
   ```tsx
   // app/layout.tsx
   import { MyProvider } from '@/components/providers/MyProvider';
   
   export default function RootLayout({ children }) {
     return (
       <html lang="en">
         <body>
           <MyProvider>
             {children}
           </MyProvider>
         </body>
       </html>
     );
   }
   ```

### API Integration

The frontend uses a custom API client for backend integration:

1. **Using the API Client**:
   ```tsx
   'use client';
   
   import { useState, useEffect } from 'react';
   import { apiClient } from '@/lib/api/client';
   import { useTenant } from '@/components/providers/TenantProvider';
   
   export function MyDataComponent() {
     const { currentTenant } = useTenant();
     const [data, setData] = useState([]);
     const [isLoading, setIsLoading] = useState(true);
     
     useEffect(() => {
       const fetchData = async () => {
         if (!currentTenant) return;
         
         try {
           const response = await apiClient.get('/my-entities', { tenant: currentTenant });
           setData(response.data);
         } catch (error) {
           console.error('Error fetching data:', error);
         } finally {
           setIsLoading(false);
         }
       };
       
       fetchData();
     }, [currentTenant]);
     
     // Component rendering
   }
   ```

2. **Creating, Updating, and Deleting Data**:
   ```tsx
   // Create
   const handleCreate = async (data) => {
     try {
       const response = await apiClient.post('/my-entities', data, { tenant: currentTenant });
       // Handle successful creation
     } catch (error) {
       // Handle error
     }
   };
   
   // Update
   const handleUpdate = async (id, data) => {
     try {
       const response = await apiClient.put(`/my-entities/${id}`, data, { tenant: currentTenant });
       // Handle successful update
     } catch (error) {
       // Handle error
     }
   };
   
   // Delete
   const handleDelete = async (id) => {
     try {
       await apiClient.delete(`/my-entities/${id}`, { tenant: currentTenant });
       // Handle successful deletion
     } catch (error) {
       // Handle error
     }
   };
   ```

### Server vs. Client Components

Understanding when to use Server vs. Client Components in Next.js:

1. **Use Server Components When**:
   - You need to fetch data
   - You don't need interactivity or browser APIs
   - You want to reduce JavaScript bundle size
   
   ```tsx
   // app/(tenant)/[tenant]/my-feature/page.tsx
   import { MyServerComponent } from '@/components/MyServerComponent';
   
   export default async function Page({ params }) {
     const { tenant } = params;
     const data = await fetchData(tenant);
     
     return <MyServerComponent data={data} />;
   }
   ```

2. **Use Client Components When**:
   - You need interactivity (useState, event handlers)
   - You need browser APIs (localStorage, window)
   - You need lifecycle effects (useEffect)
   
   ```tsx
   'use client';
   
   import { useState } from 'react';
   
   export function MyClientComponent() {
     const [isOpen, setIsOpen] = useState(false);
     
     return (
       <div>
         <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
         {isOpen && <div>Content</div>}
       </div>
     );
   }
   ```

3. **Component Composition**:
   - Server components can import and render client components
   - Client components cannot use async/await directly
   - Pass data from server to client components as props
   
   ```tsx
   // Server component
   import { MyClientComponent } from '@/components/MyClientComponent';
   
   export default async function Page() {
     const data = await fetchData();
     
     return <MyClientComponent initialData={data} />;
   }
   
   // Client component
   'use client';
   
   export function MyClientComponent({ initialData }) {
     const [data, setData] = useState(initialData);
     
     // Client-side logic
     
     return <div>{/* Render data */}</div>;
   }
   ```

## Development Workflow

### Setting Up the Development Environment

1. **Prerequisites**:
   - .NET SDK 8.0
   - Node.js 18+
   - SQL Server or PostgreSQL
   - Visual Studio 2022 or VS Code with C# extension

2. **Backend Setup**:
   ```bash
   # Clone the repository
   git clone https://github.com/OpenAutomateOrg/OpenAutomate.Backend.git
   
   # Navigate to backend directory
   cd OpenAutomate.Backend
   
   # Restore packages
   dotnet restore
   
   # Update database
   dotnet ef database update -p OpenAutomate.Infrastructure -s OpenAutomate.API
   
   # Run the API
   dotnet run --project OpenAutomate.API
   ```

3. **Frontend Setup**:
   ```bash
   # Clone the repository
   git clone https://github.com/OpenAutomateOrg/OpenAutomate.Frontend.git
   
   # Navigate to frontend directory
   cd OpenAutomate.Frontend
   
   # Install dependencies
   npm install
   
   # Run development server
   npm run dev
   ```

4. **Bot Agent Setup** (optional):
   ```bash
   # Clone the repository
   git clone https://github.com/OpenAutomateOrg/OpenAutomate.BotAgent.git
   
   # Navigate to bot agent directory
   cd OpenAutomate.BotAgent
   
   # Follow the setup instructions in the repository README
   ```

### Running the Application

1. **Backend**:
   - Run with Visual Studio: Open the solution and press F5
   - Run with CLI: `dotnet run --project OpenAutomate.API`
   - API will be available at `https://localhost:7240`

2. **Frontend**:
   - Run with CLI: `npm run dev`
   - Frontend will be available at `http://localhost:3000`

### Testing

1. **Backend Testing**:
   - Unit tests: `dotnet test`
   - API testing: Use Swagger UI or Postman

2. **Frontend Testing**:
   - Component tests: `npm test`
   - End-to-end tests: `npm run cypress`

### Pull Request Process

1. Create a new branch for your feature
2. Implement the feature with appropriate tests
3. Run all tests to ensure they pass
4. Create a pull request with a detailed description
5. Ensure the CI pipeline passes
6. Request a code review
7. Address any review comments
8. Merge the pull request once approved 

### API Documentation

The OpenAutomate API uses Swagger/OpenAPI for documentation. To ensure comprehensive API documentation:

1. **Enable XML Documentation**:
   The project is configured to generate XML documentation. Ensure your controller actions and DTOs have proper XML comments:
   ```csharp
   /// <summary>
   /// Creates a new bot agent for the specified tenant
   /// </summary>
   /// <param name="tenant">The tenant slug</param>
   /// <param name="request">The bot agent creation request</param>
   /// <returns>A newly created bot agent</returns>
   /// <response code="201">Returns the newly created bot agent</response>
   /// <response code="400">If the request is invalid</response>
   /// <response code="404">If the tenant is not found</response>
   [HttpPost]
   [ProducesResponseType(typeof(BotAgentResponse), StatusCodes.Status201Created)]
   [ProducesResponseType(StatusCodes.Status400BadRequest)]
   [ProducesResponseType(StatusCodes.Status404NotFound)]
   public async Task<IActionResult> CreateBotAgent([FromRoute] string tenant, [FromBody] CreateBotAgentRequest request)
   {
       // Implementation
   }
   ```

2. **Document Request and Response Models**:
   ```csharp
   /// <summary>
   /// Request model for creating a new bot agent
   /// </summary>
   public class CreateBotAgentRequest
   {
       /// <summary>
       /// The display name of the bot agent
       /// </summary>
       /// <example>Finance-Bot-01</example>
       [Required]
       public string Name { get; set; }
       
       /// <summary>
       /// The name of the machine where the bot agent will run
       /// </summary>
       /// <example>FINANCE-PC-01</example>
       [Required]
       public string MachineName { get; set; }
   }
   ```

3. **Use Status Code Attributes**:
   ```csharp
   [ProducesResponseType(typeof(AssetResponse), StatusCodes.Status200OK)]
   [ProducesResponseType(StatusCodes.Status404NotFound)]
   ```

4. **Add API Endpoint Grouping**:
   ```csharp
   [ApiController]
   [Route("{tenant}/api/[controller]")]
   [ApiExplorerSettings(GroupName = "Bot Agents")]
   public class BotAgentsController : ControllerBase
   ```

5. **Configure API Versions** (if using versioning):
   ```csharp
   [ApiVersion("1.0")]
   [Route("api/v{version:apiVersion}/[controller]")]
   ```

By following these documentation practices, the Swagger UI will provide comprehensive information about your API endpoints, making it easier for developers to understand and consume your API.
