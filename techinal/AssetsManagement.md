# Assets Management Feature Implementation

## 1. Overview

The Assets Management feature allows users to create, store, and manage various types of assets in the OpenAutomate platform. Assets can store text or credentials (secrets) that can be referenced by automation processes. This feature provides a secure way for bot agents to retrieve sensitive information during runtime without hardcoding values in automation scripts.

Key capabilities include:
- Creation and management of assets through the web application
- Secure storage of sensitive information in the database
- Authentication-based access for bot agents
- Support for different asset types (initially string and secret)
- Multi-tenant isolation of assets
- Control which bot agents can access which assets

## 2. Domain Model

### 2.1 Asset Entity

```csharp
using OpenAutomate.Core.Domain.Base;
using System;
using System.Text.Json.Serialization;
using System.Collections.Generic;

namespace OpenAutomate.Core.Domain.Entities
{
    public class Asset : BaseEntity
    {
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public string Key { get; set; } = string.Empty;
        public AssetType Type { get; set; } = AssetType.String;
        public string Value { get; set; } = string.Empty;
        
        // Foreign key for Organization Unit (tenant)
        public Guid OrganizationUnitId { get; set; }
        [JsonIgnore]
        public virtual OrganizationUnit? OrganizationUnit { get; set; }
        
        // Foreign key for User (Creator)
        public Guid CreatedById { get; set; }
        [JsonIgnore]
        public virtual User? CreatedBy { get; set; }
        
        // Navigation property for asset bot agent relationships
        [JsonIgnore]
        public virtual ICollection<AssetBotAgent>? AssetBotAgents { get; set; }
    }
    
    public enum AssetType
    {
        String = 0,
        Secret = 1
    }
}
```

### 2.2 BotAgent Entity Update

Add a MachineKey property to the BotAgent entity:

```csharp
using OpenAutomate.Core.Domain.Base;
using System;
using System.Collections.Generic;
using System.Text.Json.Serialization;

namespace OpenAutomate.Core.Domain.Entities
{
    public class BotAgent : BaseEntity
    {
        public string Name { get; set; } = string.Empty;
        public string MachineName { get; set; } = string.Empty;
        public string IpAddress { get; set; } = string.Empty;
        public string Status { get; set; } = string.Empty;
        public string MachineKey { get; set; } = string.Empty; // Unique key for bot agent authentication
        public DateTime RegisteredAt { get; set; }
        public DateTime LastHeartbeat { get; set; }
        
        // Foreign key for User (Owner)
        public Guid OwnerId { get; set; }
        [JsonIgnore]
        public virtual User? Owner { get; set; }
        
        // Foreign key for Organization Unit (tenant)
        public Guid OrganizationUnitId { get; set; }
        [JsonIgnore]
        public virtual OrganizationUnit? OrganizationUnit { get; set; }
        
        // Navigation property for executions
        [JsonIgnore]
        public virtual ICollection<Execution>? Executions { get; set; }
        
        // Navigation property for asset bot agent relationships
        [JsonIgnore]
        public virtual ICollection<AssetBotAgent>? AssetBotAgents { get; set; }
    }
}
```

### 2.3 AssetBotAgent Entity

Create a new entity for the many-to-many relationship between Assets and BotAgents:

```csharp
using OpenAutomate.Core.Domain.Base;
using System;
using System.Text.Json.Serialization;

namespace OpenAutomate.Core.Domain.Entities
{
    public class AssetBotAgent : BaseEntity
    {
        // Foreign key for Asset
        public Guid AssetId { get; set; }
        [JsonIgnore]
        public virtual Asset? Asset { get; set; }
        
        // Foreign key for BotAgent
        public Guid BotAgentId { get; set; }
        [JsonIgnore]
        public virtual BotAgent? BotAgent { get; set; }
        
        // Foreign key for Organization Unit (tenant)
        public Guid OrganizationUnitId { get; set; }
        [JsonIgnore]
        public virtual OrganizationUnit? OrganizationUnit { get; set; }
    }
}
```

## 3. Data Transfer Objects (DTOs)

### 3.1 Request DTOs

```csharp
namespace OpenAutomate.Core.Dto.Asset
{
    public class CreateAssetRequest
    {
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public string Key { get; set; } = string.Empty;
        public string Value { get; set; } = string.Empty;
        public AssetType Type { get; set; } = AssetType.String;
        public List<Guid>? BotAgentIds { get; set; } // Optional bot agent IDs that can access this asset
    }
    
    public class UpdateAssetRequest
    {
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public string Value { get; set; } = string.Empty;
    }
    
    public class BotAgentAssetRequest
    {
        public string Key { get; set; } = string.Empty;
        public string MachineKey { get; set; } = string.Empty; // Changed from MachineName to MachineKey
    }
    
    public class AssetBotAgentRequest
    {
        public Guid AssetId { get; set; }
        public List<Guid> BotAgentIds { get; set; } = new List<Guid>();
    }
    
    public class BotAgentRegistrationRequest
    {
        public string Name { get; set; } = string.Empty;
        public string MachineName { get; set; } = string.Empty;
        public string IpAddress { get; set; } = string.Empty;
        public string MachineKey { get; set; } = string.Empty;
    }
}
```

### 3.2 Response DTOs

```csharp
namespace OpenAutomate.Core.Dto.Asset
{
    public class AssetResponse
    {
        public Guid Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public string Key { get; set; } = string.Empty;
        public AssetType Type { get; set; }
        public string Value { get; set; } = string.Empty;
        public DateTime CreatedAt { get; set; }
        public Guid CreatedById { get; set; }
        public List<BotAgentSummaryResponse>? AuthorizedBotAgents { get; set; }
    }
    
    public class AssetListResponse
    {
        public Guid Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public string Key { get; set; } = string.Empty;
        public AssetType Type { get; set; }
        public DateTime CreatedAt { get; set; }
        public int AuthorizedBotAgentsCount { get; set; }
    }
    
    public class BotAgentAssetResponse
    {
        public string Key { get; set; } = string.Empty;
        public string Value { get; set; } = string.Empty;
    }
    
    public class BotAgentSummaryResponse
    {
        public Guid Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string MachineName { get; set; } = string.Empty;
        public string Status { get; set; } = string.Empty;
    }
    
    public class BotAgentRegistrationResponse
    {
        public Guid Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string MachineName { get; set; } = string.Empty;
        public string IpAddress { get; set; } = string.Empty;
        public string MachineKey { get; set; } = string.Empty;
        public string Status { get; set; } = string.Empty;
        public DateTime RegisteredAt { get; set; }
    }
}
```

## 4. Service Interface

```csharp
using OpenAutomate.Core.Dto.Asset;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace OpenAutomate.Core.IServices
{
    public interface IAssetService
    {
        Task<AssetResponse> CreateAsync(CreateAssetRequest request, Guid userId);
        Task<List<AssetListResponse>> GetAllAsync();
        Task<AssetResponse> GetByIdAsync(Guid id);
        Task<AssetResponse> GetByKeyAsync(string key);
        Task<AssetResponse> UpdateAsync(Guid id, UpdateAssetRequest request);
        Task<bool> DeleteAsync(Guid id);
        Task<BotAgentAssetResponse> GetAssetForBotAgentAsync(BotAgentAssetRequest request);
        
        // Asset-BotAgent relationship methods
        Task<AssetResponse> UpdateAssetBotAgentsAsync(AssetBotAgentRequest request);
        Task<List<BotAgentSummaryResponse>> GetAuthorizedBotAgentsForAssetAsync(Guid assetId);
        Task<bool> AuthorizeBotAgentForAssetAsync(Guid assetId, Guid botAgentId);
        Task<bool> RevokeBotAgentForAssetAsync(Guid assetId, Guid botAgentId);
    }
    
    public interface IBotAgentService
    {
        Task<BotAgentRegistrationResponse> RegisterBotAgentAsync(BotAgentRegistrationRequest request, Guid userId);
        Task<string> GenerateMachineKeyAsync();
        Task<List<AssetResponse>> GetAuthorizedAssetsForBotAgentAsync(Guid botAgentId);
        Task<bool> VerifyBotAgentMachineKeyAsync(string machineKey);
    }
}
```

## 5. Service Implementation

### 5.1 Asset Service

```csharp
using OpenAutomate.Core.Domain.Entities;
using OpenAutomate.Core.Domain.IRepository;
using OpenAutomate.Core.Dto.Asset;
using OpenAutomate.Core.IServices;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Logging;

namespace OpenAutomate.Infrastructure.Services
{
    public class AssetService : IAssetService
    {
        private readonly IUnitOfWork _unitOfWork;
        private readonly ITenantContext _tenantContext;
        private readonly ILogger<AssetService> _logger;
        
        public AssetService(IUnitOfWork unitOfWork, ITenantContext tenantContext, ILogger<AssetService> logger)
        {
            _unitOfWork = unitOfWork;
            _tenantContext = tenantContext;
            _logger = logger;
        }
        
        public async Task<AssetResponse> CreateAsync(CreateAssetRequest request, Guid userId)
        {
            try
            {
                // Check if key already exists
                var existingAsset = await _unitOfWork.Assets.GetFirstOrDefaultAsync(
                    a => a.Key == request.Key && a.OrganizationUnitId == _tenantContext.CurrentTenantId);
                    
                if (existingAsset != null)
                {
                    throw new InvalidOperationException($"Asset with key '{request.Key}' already exists");
                }
                
                var asset = new Asset
                {
                    Name = request.Name,
                    Description = request.Description,
                    Key = request.Key,
                    Type = request.Type,
                    Value = request.Value,
                    CreatedById = userId,
                    OrganizationUnitId = _tenantContext.CurrentTenantId
                };
                
                await _unitOfWork.Assets.AddAsync(asset);
                await _unitOfWork.CompleteAsync();
                
                // If bot agent IDs are provided, authorize them for this asset
                if (request.BotAgentIds != null && request.BotAgentIds.Any())
                {
                    foreach (var botAgentId in request.BotAgentIds)
                    {
                        await AuthorizeBotAgentForAssetAsync(asset.Id, botAgentId);
                    }
                }
                
                return await GetByIdAsync(asset.Id);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error creating asset");
                throw;
            }
        }
        
        public async Task<List<AssetListResponse>> GetAllAsync()
        {
            try
            {
                var assets = await _unitOfWork.Assets.GetAllAsync(
                    includes: a => a.AssetBotAgents);
                
                return assets.Select(a => new AssetListResponse
                {
                    Id = a.Id,
                    Name = a.Name,
                    Description = a.Description,
                    Key = a.Key,
                    Type = a.Type,
                    CreatedAt = a.CreatedAt ?? DateTime.Now,
                    AuthorizedBotAgentsCount = a.AssetBotAgents?.Count ?? 0
                }).ToList();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error getting all assets");
                throw;
            }
        }
        
        public async Task<AssetResponse> GetByIdAsync(Guid id)
        {
            try
            {
                var asset = await _unitOfWork.Assets.GetFirstOrDefaultAsync(
                    a => a.Id == id,
                    includes: a => a.AssetBotAgents);
                
                if (asset == null)
                {
                    throw new KeyNotFoundException($"Asset with ID {id} not found");
                }
                
                // Get authorized bot agents
                var authorizedBotAgents = new List<BotAgentSummaryResponse>();
                if (asset.AssetBotAgents != null && asset.AssetBotAgents.Any())
                {
                    var botAgentIds = asset.AssetBotAgents.Select(aba => aba.BotAgentId).ToList();
                    var botAgents = await _unitOfWork.BotAgents.GetAllAsync(
                        filter: ba => botAgentIds.Contains(ba.Id));
                        
                    authorizedBotAgents = botAgents.Select(ba => new BotAgentSummaryResponse
                    {
                        Id = ba.Id,
                        Name = ba.Name,
                        MachineName = ba.MachineName,
                        Status = ba.Status
                    }).ToList();
                }
                
                return new AssetResponse
                {
                    Id = asset.Id,
                    Name = asset.Name,
                    Description = asset.Description,
                    Key = asset.Key,
                    Type = asset.Type,
                    Value = asset.Value,
                    CreatedAt = asset.CreatedAt ?? DateTime.Now,
                    CreatedById = asset.CreatedById,
                    AuthorizedBotAgents = authorizedBotAgents
                };
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error getting asset by ID {Id}", id);
                throw;
            }
        }
        
        public async Task<AssetResponse> GetByKeyAsync(string key)
        {
            try
            {
                var asset = await _unitOfWork.Assets.GetFirstOrDefaultAsync(
                    a => a.Key == key,
                    includes: a => a.AssetBotAgents);
                
                if (asset == null)
                {
                    throw new KeyNotFoundException($"Asset with key '{key}' not found");
                }
                
                return await GetByIdAsync(asset.Id);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error getting asset by key {Key}", key);
                throw;
            }
        }
        
        public async Task<AssetResponse> UpdateAsync(Guid id, UpdateAssetRequest request)
        {
            try
            {
                var asset = await _unitOfWork.Assets.GetByIdAsync(id);
                
                if (asset == null)
                {
                    throw new KeyNotFoundException($"Asset with ID {id} not found");
                }
                
                asset.Name = request.Name;
                asset.Description = request.Description;
                asset.Value = request.Value;
                asset.LastModifyAt = DateTime.Now;
                
                await _unitOfWork.Assets.UpdateAsync(asset);
                await _unitOfWork.CompleteAsync();
                
                return await GetByIdAsync(id);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error updating asset with ID {Id}", id);
                throw;
            }
        }
        
        public async Task<bool> DeleteAsync(Guid id)
        {
            try
            {
                var asset = await _unitOfWork.Assets.GetByIdAsync(id);
                
                if (asset == null)
                {
                    return false;
                }
                
                // Delete related asset-bot agent relationships
                var assetBotAgents = await _unitOfWork.AssetBotAgents.GetAllAsync(
                    filter: aba => aba.AssetId == id);
                    
                foreach (var assetBotAgent in assetBotAgents)
                {
                    await _unitOfWork.AssetBotAgents.DeleteAsync(assetBotAgent);
                }
                
                await _unitOfWork.Assets.DeleteAsync(asset);
                await _unitOfWork.CompleteAsync();
                
                return true;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error deleting asset with ID {Id}", id);
                throw;
            }
        }
        
        public async Task<BotAgentAssetResponse> GetAssetForBotAgentAsync(BotAgentAssetRequest request)
        {
            try
            {
                // Verify bot agent exists and is active by machine key
                var botAgent = await _unitOfWork.BotAgents.GetFirstOrDefaultAsync(
                    b => b.MachineKey == request.MachineKey && b.Status == "Active");
                    
                if (botAgent == null)
                {
                    throw new UnauthorizedAccessException($"Bot agent with machine key '{request.MachineKey}' not found or not active");
                }
                
                // Get tenant ID from bot agent
                var tenantId = botAgent.OrganizationUnitId;
                
                // Get asset using tenant ID and key
                var asset = await _unitOfWork.Assets.GetFirstOrDefaultAsync(
                    a => a.Key == request.Key && a.OrganizationUnitId == tenantId,
                    includes: a => a.AssetBotAgents);
                    
                if (asset == null)
                {
                    throw new KeyNotFoundException($"Asset with key '{request.Key}' not found");
                }
                
                // Check if bot agent is authorized to access this asset
                if (asset.AssetBotAgents == null || !asset.AssetBotAgents.Any(aba => aba.BotAgentId == botAgent.Id))
                {
                    throw new UnauthorizedAccessException($"Bot agent is not authorized to access asset with key '{request.Key}'");
                }
                
                return new BotAgentAssetResponse
                {
                    Key = asset.Key,
                    Value = asset.Value
                };
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error getting asset for bot agent");
                throw;
            }
        }
        
        public async Task<AssetResponse> UpdateAssetBotAgentsAsync(AssetBotAgentRequest request)
        {
            try
            {
                var asset = await _unitOfWork.Assets.GetByIdAsync(request.AssetId);
                
                if (asset == null)
                {
                    throw new KeyNotFoundException($"Asset with ID {request.AssetId} not found");
                }
                
                // Delete existing asset-bot agent relationships
                var existingAssetBotAgents = await _unitOfWork.AssetBotAgents.GetAllAsync(
                    filter: aba => aba.AssetId == request.AssetId);
                    
                foreach (var existingAssetBotAgent in existingAssetBotAgents)
                {
                    await _unitOfWork.AssetBotAgents.DeleteAsync(existingAssetBotAgent);
                }
                
                // Add new asset-bot agent relationships
                foreach (var botAgentId in request.BotAgentIds)
                {
                    // Verify bot agent exists
                    var botAgent = await _unitOfWork.BotAgents.GetByIdAsync(botAgentId);
                    if (botAgent == null)
                    {
                        _logger.LogWarning("Bot agent with ID {BotAgentId} not found", botAgentId);
                        continue;
                    }
                    
                    var assetBotAgent = new AssetBotAgent
                    {
                        AssetId = request.AssetId,
                        BotAgentId = botAgentId,
                        OrganizationUnitId = _tenantContext.CurrentTenantId
                    };
                    
                    await _unitOfWork.AssetBotAgents.AddAsync(assetBotAgent);
                }
                
                await _unitOfWork.CompleteAsync();
                
                return await GetByIdAsync(request.AssetId);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error updating asset bot agents for asset ID {AssetId}", request.AssetId);
                throw;
            }
        }
        
        public async Task<List<BotAgentSummaryResponse>> GetAuthorizedBotAgentsForAssetAsync(Guid assetId)
        {
            try
            {
                var asset = await _unitOfWork.Assets.GetFirstOrDefaultAsync(
                    a => a.Id == assetId,
                    includes: a => a.AssetBotAgents);
                
                if (asset == null)
                {
                    throw new KeyNotFoundException($"Asset with ID {assetId} not found");
                }
                
                var authorizedBotAgents = new List<BotAgentSummaryResponse>();
                if (asset.AssetBotAgents != null && asset.AssetBotAgents.Any())
                {
                    var botAgentIds = asset.AssetBotAgents.Select(aba => aba.BotAgentId).ToList();
                    var botAgents = await _unitOfWork.BotAgents.GetAllAsync(
                        filter: ba => botAgentIds.Contains(ba.Id));
                        
                    authorizedBotAgents = botAgents.Select(ba => new BotAgentSummaryResponse
                    {
                        Id = ba.Id,
                        Name = ba.Name,
                        MachineName = ba.MachineName,
                        Status = ba.Status
                    }).ToList();
                }
                
                return authorizedBotAgents;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error getting authorized bot agents for asset ID {AssetId}", assetId);
                throw;
            }
        }
        
        public async Task<bool> AuthorizeBotAgentForAssetAsync(Guid assetId, Guid botAgentId)
        {
            try
            {
                // Verify asset exists
                var asset = await _unitOfWork.Assets.GetByIdAsync(assetId);
                if (asset == null)
                {
                    throw new KeyNotFoundException($"Asset with ID {assetId} not found");
                }
                
                // Verify bot agent exists
                var botAgent = await _unitOfWork.BotAgents.GetByIdAsync(botAgentId);
                if (botAgent == null)
                {
                    throw new KeyNotFoundException($"Bot agent with ID {botAgentId} not found");
                }
                
                // Check if relationship already exists
                var existingRelationship = await _unitOfWork.AssetBotAgents.GetFirstOrDefaultAsync(
                    aba => aba.AssetId == assetId && aba.BotAgentId == botAgentId);
                    
                if (existingRelationship != null)
                {
                    return true; // Already authorized
                }
                
                // Create new relationship
                var assetBotAgent = new AssetBotAgent
                {
                    AssetId = assetId,
                    BotAgentId = botAgentId,
                    OrganizationUnitId = _tenantContext.CurrentTenantId
                };
                
                await _unitOfWork.AssetBotAgents.AddAsync(assetBotAgent);
                await _unitOfWork.CompleteAsync();
                
                return true;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error authorizing bot agent {BotAgentId} for asset {AssetId}", botAgentId, assetId);
                throw;
            }
        }
        
        public async Task<bool> RevokeBotAgentForAssetAsync(Guid assetId, Guid botAgentId)
        {
            try
            {
                // Find the relationship
                var assetBotAgent = await _unitOfWork.AssetBotAgents.GetFirstOrDefaultAsync(
                    aba => aba.AssetId == assetId && aba.BotAgentId == botAgentId);
                    
                if (assetBotAgent == null)
                {
                    return false; // Relationship doesn't exist
                }
                
                await _unitOfWork.AssetBotAgents.DeleteAsync(assetBotAgent);
                await _unitOfWork.CompleteAsync();
                
                return true;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error revoking bot agent {BotAgentId} for asset {AssetId}", botAgentId, assetId);
                throw;
            }
        }
    }
}
```

### 5.2 Bot Agent Service

```csharp
using OpenAutomate.Core.Domain.Entities;
using OpenAutomate.Core.Domain.IRepository;
using OpenAutomate.Core.Dto.Asset;
using OpenAutomate.Core.IServices;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Security.Cryptography;
using Microsoft.Extensions.Logging;

namespace OpenAutomate.Infrastructure.Services
{
    public class BotAgentService : IBotAgentService
    {
        private readonly IUnitOfWork _unitOfWork;
        private readonly ITenantContext _tenantContext;
        private readonly ILogger<BotAgentService> _logger;
        
        public BotAgentService(IUnitOfWork unitOfWork, ITenantContext tenantContext, ILogger<BotAgentService> logger)
        {
            _unitOfWork = unitOfWork;
            _tenantContext = tenantContext;
            _logger = logger;
        }
        
        public async Task<BotAgentRegistrationResponse> RegisterBotAgentAsync(BotAgentRegistrationRequest request, Guid userId)
        {
            try
            {
                // Generate machine key if not provided
                if (string.IsNullOrEmpty(request.MachineKey))
                {
                    request.MachineKey = await GenerateMachineKeyAsync();
                }
                
                // Check if machine key already exists
                var existingBotAgent = await _unitOfWork.BotAgents.GetFirstOrDefaultAsync(
                    b => b.MachineKey == request.MachineKey);
                    
                if (existingBotAgent != null)
                {
                    throw new InvalidOperationException($"Bot agent with machine key '{request.MachineKey}' already exists");
                }
                
                var botAgent = new BotAgent
                {
                    Name = request.Name,
                    MachineName = request.MachineName,
                    IpAddress = request.IpAddress,
                    MachineKey = request.MachineKey,
                    Status = "Active",
                    RegisteredAt = DateTime.Now,
                    LastHeartbeat = DateTime.Now,
                    OwnerId = userId,
                    OrganizationUnitId = _tenantContext.CurrentTenantId
                };
                
                await _unitOfWork.BotAgents.AddAsync(botAgent);
                await _unitOfWork.CompleteAsync();
                
                return new BotAgentRegistrationResponse
                {
                    Id = botAgent.Id,
                    Name = botAgent.Name,
                    MachineName = botAgent.MachineName,
                    IpAddress = botAgent.IpAddress,
                    MachineKey = botAgent.MachineKey,
                    Status = botAgent.Status,
                    RegisteredAt = botAgent.RegisteredAt
                };
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error registering bot agent");
                throw;
            }
        }
        
        public async Task<string> GenerateMachineKeyAsync()
        {
            // Generate a cryptographically secure random key
            var key = new byte[32]; // 256 bits
            using (var rng = RandomNumberGenerator.Create())
            {
                rng.GetBytes(key);
            }
            
            // Convert to Base64 string
            var machineKey = Convert.ToBase64String(key);
            
            // Verify it doesn't already exist
            var existingBotAgent = await _unitOfWork.BotAgents.GetFirstOrDefaultAsync(
                b => b.MachineKey == machineKey);
                
            if (existingBotAgent != null)
            {
                // If it does (extremely unlikely), recursively try again
                return await GenerateMachineKeyAsync();
            }
            
            return machineKey;
        }
        
        public async Task<List<AssetResponse>> GetAuthorizedAssetsForBotAgentAsync(Guid botAgentId)
        {
            try
            {
                // Verify bot agent exists
                var botAgent = await _unitOfWork.BotAgents.GetByIdAsync(botAgentId);
                if (botAgent == null)
                {
                    throw new KeyNotFoundException($"Bot agent with ID {botAgentId} not found");
                }
                
                // Get all asset-bot agent relationships for this bot agent
                var assetBotAgents = await _unitOfWork.AssetBotAgents.GetAllAsync(
                    filter: aba => aba.BotAgentId == botAgentId);
                    
                if (!assetBotAgents.Any())
                {
                    return new List<AssetResponse>();
                }
                
                var assetIds = assetBotAgents.Select(aba => aba.AssetId).ToList();
                
                // Get all assets for these IDs
                var assets = await _unitOfWork.Assets.GetAllAsync(
                    filter: a => assetIds.Contains(a.Id));
                    
                return assets.Select(a => new AssetResponse
                {
                    Id = a.Id,
                    Name = a.Name,
                    Description = a.Description,
                    Key = a.Key,
                    Type = a.Type,
                    Value = a.Value,
                    CreatedAt = a.CreatedAt ?? DateTime.Now,
                    CreatedById = a.CreatedById
                }).ToList();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error getting authorized assets for bot agent ID {BotAgentId}", botAgentId);
                throw;
            }
        }
        
        public async Task<bool> VerifyBotAgentMachineKeyAsync(string machineKey)
        {
            try
            {
                if (string.IsNullOrEmpty(machineKey))
                {
                    return false;
                }
                
                var botAgent = await _unitOfWork.BotAgents.GetFirstOrDefaultAsync(
                    b => b.MachineKey == machineKey && b.Status == "Active");
                    
                return botAgent != null;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error verifying bot agent machine key");
                throw;
            }
        }
    }
}
```

## 6. API Controllers

### 6.1 Assets Controller (Web Application)

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using OpenAutomate.API.Controllers;
using OpenAutomate.Core.Dto.Asset;
using OpenAutomate.Core.IServices;
using System;
using System.Threading.Tasks;

namespace OpenAutomate.API.Controllers
{
    [Route("{tenant}/api/[controller]")]
    [ApiController]
    [Authorize]
    public class AssetsController : CustomControllerBase
    {
        private readonly IAssetService _assetService;
        
        public AssetsController(IAssetService assetService)
        {
            _assetService = assetService;
        }
        
        [HttpGet]
        public async Task<IActionResult> GetAll()
        {
            try
            {
                var assets = await _assetService.GetAllAsync();
                return Ok(assets);
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
                var asset = await _assetService.GetByIdAsync(id);
                return Ok(asset);
            }
            catch (KeyNotFoundException)
            {
                return NotFound(new { message = $"Asset with ID {id} not found" });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
        
        [HttpGet("key/{key}")]
        public async Task<IActionResult> GetByKey(string key)
        {
            try
            {
                var asset = await _assetService.GetByKeyAsync(key);
                return Ok(asset);
            }
            catch (KeyNotFoundException)
            {
                return NotFound(new { message = $"Asset with key '{key}' not found" });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
        
        [HttpPost]
        public async Task<IActionResult> Create([FromBody] CreateAssetRequest request)
        {
            try
            {
                var asset = await _assetService.CreateAsync(request, currentUser.Id);
                return CreatedAtAction(nameof(GetById), new { id = asset.Id }, asset);
            }
            catch (InvalidOperationException ex)
            {
                return BadRequest(new { message = ex.Message });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
        
        [HttpPut("{id}")]
        public async Task<IActionResult> Update(Guid id, [FromBody] UpdateAssetRequest request)
        {
            try
            {
                var asset = await _assetService.UpdateAsync(id, request);
                return Ok(asset);
            }
            catch (KeyNotFoundException)
            {
                return NotFound(new { message = $"Asset with ID {id} not found" });
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
                var result = await _assetService.DeleteAsync(id);
                if (!result)
                    return NotFound(new { message = $"Asset with ID {id} not found" });
                    
                return NoContent();
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
        
        [HttpGet("{id}/bot-agents")]
        public async Task<IActionResult> GetAuthorizedBotAgents(Guid id)
        {
            try
            {
                var botAgents = await _assetService.GetAuthorizedBotAgentsForAssetAsync(id);
                return Ok(botAgents);
            }
            catch (KeyNotFoundException)
            {
                return NotFound(new { message = $"Asset with ID {id} not found" });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
        
        [HttpPost("{id}/bot-agents")]
        public async Task<IActionResult> UpdateBotAgents(Guid id, [FromBody] AssetBotAgentRequest request)
        {
            try
            {
                request.AssetId = id; // Ensure the asset ID matches the route
                var asset = await _assetService.UpdateAssetBotAgentsAsync(request);
                return Ok(asset);
            }
            catch (KeyNotFoundException ex)
            {
                return NotFound(new { message = ex.Message });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
        
        [HttpPost("{assetId}/bot-agents/{botAgentId}")]
        public async Task<IActionResult> AuthorizeBotAgent(Guid assetId, Guid botAgentId)
        {
            try
            {
                var result = await _assetService.AuthorizeBotAgentForAssetAsync(assetId, botAgentId);
                return Ok(new { success = result });
            }
            catch (KeyNotFoundException ex)
            {
                return NotFound(new { message = ex.Message });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
        
        [HttpDelete("{assetId}/bot-agents/{botAgentId}")]
        public async Task<IActionResult> RevokeBotAgent(Guid assetId, Guid botAgentId)
        {
            try
            {
                var result = await _assetService.RevokeBotAgentForAssetAsync(assetId, botAgentId);
                if (!result)
                    return NotFound(new { message = "Asset or bot agent not found or no relationship exists" });
                    
                return NoContent();
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
    }
}
```

### 6.2 Bot Agent Assets Controller

```csharp
using Microsoft.AspNetCore.Mvc;
using OpenAutomate.Core.Dto.Asset;
using OpenAutomate.Core.IServices;
using System;
using System.Threading.Tasks;

namespace OpenAutomate.API.Controllers
{
    [Route("api/bot-agent/[controller]")]
    [ApiController]
    public class AssetsController : ControllerBase
    {
        private readonly IAssetService _assetService;
        private readonly IBotAgentService _botAgentService;
        
        public AssetsController(IAssetService assetService, IBotAgentService botAgentService)
        {
            _assetService = assetService;
            _botAgentService = botAgentService;
        }
        
        [HttpPost("get-value")]
        public async Task<IActionResult> GetAssetValue([FromBody] BotAgentAssetRequest request)
        {
            try
            {
                if (string.IsNullOrEmpty(request.Key) || string.IsNullOrEmpty(request.MachineKey))
                {
                    return BadRequest(new { message = "Key and MachineKey are required" });
                }
                
                // Verify machine key is valid
                var isValidMachineKey = await _botAgentService.VerifyBotAgentMachineKeyAsync(request.MachineKey);
                if (!isValidMachineKey)
                {
                    return Unauthorized(new { message = "Invalid machine key" });
                }
                
                var asset = await _assetService.GetAssetForBotAgentAsync(request);
                return Ok(asset);
            }
            catch (UnauthorizedAccessException ex)
            {
                return Unauthorized(new { message = ex.Message });
            }
            catch (KeyNotFoundException ex)
            {
                return NotFound(new { message = ex.Message });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
    }
}
```

### 6.3 Bot Agents Controller

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using OpenAutomate.API.Controllers;
using OpenAutomate.Core.Dto.Asset;
using OpenAutomate.Core.IServices;
using System;
using System.Threading.Tasks;

namespace OpenAutomate.API.Controllers
{
    [Route("{tenant}/api/[controller]")]
    [ApiController]
    [Authorize]
    public class BotAgentsController : CustomControllerBase
    {
        private readonly IBotAgentService _botAgentService;
        
        public BotAgentsController(IBotAgentService botAgentService)
        {
            _botAgentService = botAgentService;
        }
        
        [HttpPost]
        public async Task<IActionResult> Register([FromBody] BotAgentRegistrationRequest request)
        {
            try
            {
                var botAgent = await _botAgentService.RegisterBotAgentAsync(request, currentUser.Id);
                return Ok(botAgent);
            }
            catch (InvalidOperationException ex)
            {
                return BadRequest(new { message = ex.Message });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
        
        [HttpGet("generate-machine-key")]
        public async Task<IActionResult> GenerateMachineKey()
        {
            try
            {
                var machineKey = await _botAgentService.GenerateMachineKeyAsync();
                return Ok(new { machineKey });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
        
        [HttpGet("{id}/assets")]
        public async Task<IActionResult> GetAuthorizedAssets(Guid id)
        {
            try
            {
                var assets = await _botAgentService.GetAuthorizedAssetsForBotAgentAsync(id);
                return Ok(assets);
            }
            catch (KeyNotFoundException)
            {
                return NotFound(new { message = $"Bot agent with ID {id} not found" });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "An error occurred while processing your request." });
            }
        }
    }
} 
```

## 7. Database Changes

### 7.1 Entity Registration in DbContext

Add the Asset and AssetBotAgent entities to the ApplicationDbContext and modify the BotAgent entity:

```csharp
// OpenAutomate.Infrastructure/DbContext/ApplicationDbContext.cs

public DbSet<Asset> Assets { get; set; }
public DbSet<AssetBotAgent> AssetBotAgents { get; set; }

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Existing code...
    
    // Configure Asset entity
    modelBuilder.Entity<Asset>()
        .HasOne(a => a.CreatedBy)
        .WithMany()
        .HasForeignKey(a => a.CreatedById)
        .OnDelete(DeleteBehavior.Restrict);
        
    modelBuilder.Entity<Asset>()
        .HasOne(a => a.OrganizationUnit)
        .WithMany()
        .HasForeignKey(a => a.OrganizationUnitId)
        .OnDelete(DeleteBehavior.Cascade);
        
    // Apply global query filter for tenant isolation
    modelBuilder.Entity<Asset>().HasQueryFilter(a => 
        !_tenantContext.HasTenant || a.OrganizationUnitId == _tenantContext.CurrentTenantId);
    
    // Configure AssetBotAgent entity
    modelBuilder.Entity<AssetBotAgent>()
        .HasOne(aba => aba.Asset)
        .WithMany(a => a.AssetBotAgents)
        .HasForeignKey(aba => aba.AssetId)
        .OnDelete(DeleteBehavior.Cascade);
        
    modelBuilder.Entity<AssetBotAgent>()
        .HasOne(aba => aba.BotAgent)
        .WithMany(ba => ba.AssetBotAgents)
        .HasForeignKey(aba => aba.BotAgentId)
        .OnDelete(DeleteBehavior.Cascade);
        
    modelBuilder.Entity<AssetBotAgent>()
        .HasOne(aba => aba.OrganizationUnit)
        .WithMany()
        .HasForeignKey(aba => aba.OrganizationUnitId)
        .OnDelete(DeleteBehavior.Restrict);
    
    // Apply global query filter for tenant isolation to AssetBotAgent
    modelBuilder.Entity<AssetBotAgent>().HasQueryFilter(aba => 
        !_tenantContext.HasTenant || aba.OrganizationUnitId == _tenantContext.CurrentTenantId);
    
    // Add unique index for Asset Key within a tenant
    modelBuilder.Entity<Asset>()
        .HasIndex(a => new { a.Key, a.OrganizationUnitId })
        .IsUnique();
    
    // Add unique index for BotAgent MachineKey
    modelBuilder.Entity<BotAgent>()
        .HasIndex(ba => ba.MachineKey)
        .IsUnique();
}
```

### 7.2 Migrations

Create migrations for the new entities and modifications:

```bash
# Add MachineKey to BotAgent
dotnet ef migrations add AddBotAgentMachineKey -p OpenAutomate.Infrastructure -s OpenAutomate.API

# Add Asset entity
dotnet ef migrations add AddAssetEntity -p OpenAutomate.Infrastructure -s OpenAutomate.API

# Add AssetBotAgent entity
dotnet ef migrations add AddAssetBotAgentEntity -p OpenAutomate.Infrastructure -s OpenAutomate.API
``` 