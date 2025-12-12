<!--
Sync Impact Report:
Version change: 1.4.0 → 1.5.0
Modified sections:
  - Added Model Context Protocol (MCP) Standards section (NEW)
  - Expanded AI & Agent Framework with MCP requirements
  - Added MCP tool sourcing and implementation guidelines
Principles:
  - I. Hexagonal Architecture
  - II. Domain-Driven Design
  - III. Dependency Inversion
  - IV. Port & Adapter Pattern
  - V. Test-First Development
  - VI. Microservices Principles
Technology Stack:
  - Backend: C# .NET 10 Minimal APIs
  - Security: JWT token-based authentication
  - Recommended Tools: EF Core, SQL Server, Dapr, Docker, Kubernetes, Azure Agent Framework
  - MCP: Model Context Protocol for AI agent tool integration (NEW)
Last updated: 2025-12-12
-->

# AI Development Factory Constitution

## Core Principles

### I. Hexagonal Architecture (NON-NEGOTIABLE)
All components MUST follow hexagonal (ports and adapters) architecture principles:
- **Domain Core**: Business logic isolated in the center, with no external dependencies
- **Ports**: Define interfaces for inbound (driving) and outbound (driven) interactions
- **Adapters**: Implement ports to connect domain to external systems (UI, DB, APIs, frameworks)
- **Dependency Rule**: Dependencies point inward only; domain knows nothing of adapters
- **Framework Independence**: Business rules testable without UI, database, web server, or external agency

**Rationale**: Hexagonal architecture ensures maintainability, testability, and flexibility by decoupling business logic from technical implementation details.

### II. Domain-Driven Design
Domain model MUST be the primary focus:
- **Ubiquitous Language**: Shared vocabulary between developers and domain experts
- **Bounded Contexts**: Clear boundaries between different domain models
- **Aggregates**: Consistency boundaries for domain entities
- **Value Objects**: Immutable objects representing domain concepts
- **Domain Events**: Capture significant business occurrences

**Rationale**: DDD aligns software structure with business domain, improving communication and reducing translation overhead.

### III. Dependency Inversion
All dependencies MUST follow the dependency inversion principle:
- High-level modules (domain) do not depend on low-level modules (infrastructure)
- Both depend on abstractions (interfaces/ports)
- Abstractions do not depend on details; details depend on abstractions
- Use dependency injection for all cross-boundary dependencies

**Rationale**: Enables flexibility, testability, and prevents coupling to specific implementations.

### IV. Port & Adapter Pattern
External integrations MUST use explicit ports and adapters:
- **Inbound Ports**: Application service interfaces (use cases)
- **Outbound Ports**: Repository, external service, notification interfaces
- **Primary Adapters**: REST controllers, CLI handlers, GraphQL resolvers, message consumers
- **Secondary Adapters**: Database implementations, HTTP clients, file systems, message producers
- Each adapter implements exactly one port

**Rationale**: Clear separation enables independent evolution of domain logic and external integrations.

### V. Test-First Development (NON-NEGOTIABLE)
TDD MUST be followed for all development:
- Tests written and approved by user before implementation
- Domain logic tested through ports (no adapters required)
- Adapter tests use test doubles for external dependencies
- Red-Green-Refactor cycle strictly enforced
- Minimum 80% code coverage, 100% for domain core

**Rationale**: Test-first ensures correctness, drives better design, and provides living documentation.

### VI. Microservices Principles (NON-NEGOTIABLE)
All services MUST adhere to microservices architecture principles:
- **Single Responsibility**: Each microservice owns one bounded context or subdomain
- **Autonomous**: Services can be developed, deployed, and scaled independently
- **Decentralized Data**: Each service manages its own database; no shared databases
- **API-First**: Services communicate only through well-defined APIs (REST, gRPC, events)
- **Resilience**: Services designed for failure; implement circuit breakers, retries, timeouts
- **Observability**: Each service provides health checks, metrics, logs, and distributed tracing
- **Evolutionary**: Services can be versioned and evolved without breaking consumers

**Rationale**: Microservices enable independent scaling, deployment, and team autonomy while maintaining system resilience.

## Microservices Standards

### Service Boundaries
Define clear service boundaries:
- **Bounded Context Alignment**: Each service maps to exactly one DDD bounded context
- **Data Ownership**: Service owns all data related to its domain; no cross-service database queries
- **Business Capability**: Services organized around business capabilities, not technical layers
- **Team Ownership**: Each service owned by a single team with end-to-end responsibility

### Inter-Service Communication
Services MUST communicate through defined patterns:
- **Synchronous**: REST/HTTP or gRPC for request-response patterns
- **Asynchronous**: Message brokers (events, commands) for eventual consistency
- **API Gateway**: Single entry point for external clients (optional for internal services)
- **Service Discovery**: Dynamic service registration and discovery mechanism
- **No Direct Database Access**: Services never access another service's database

### Data Management
Each service MUST manage data independently:
- **Database per Service**: Each service has its own database instance
- **Eventual Consistency**: Accept eventual consistency across service boundaries
- **Saga Pattern**: Distributed transactions implemented as sagas (choreography or orchestration)
- **Event Sourcing**: Consider event sourcing for audit trails and temporal queries
- **CQRS**: Separate read and write models when appropriate for scalability

### Service Deployment
Services MUST be independently deployable:
- **Containerization**: Each service packaged as container (Docker, containerd)
- **Orchestration**: Kubernetes or equivalent for service orchestration
- **CI/CD per Service**: Independent deployment pipelines
- **Zero-Downtime Deployments**: Blue-green, canary, or rolling deployments
- **Backward Compatibility**: API changes maintain backward compatibility or versioning

### Resilience Patterns
Services MUST implement resilience mechanisms:
- **Circuit Breaker**: Prevent cascading failures
- **Retry with Backoff**: Exponential backoff for transient failures
- **Timeout**: All external calls have explicit timeouts
- **Bulkhead**: Isolate critical resources
- **Fallback**: Graceful degradation when dependencies fail

### Observability Requirements
Each service MUST provide:
- **Health Endpoints**: Liveness and readiness probes
- **Metrics**: Service-level metrics (latency, throughput, error rates)
- **Structured Logging**: JSON logs with correlation IDs
- **Distributed Tracing**: OpenTelemetry or equivalent for request tracing
- **Service Mesh**: Consider service mesh for cross-cutting concerns (optional)

### Security Standards
All API access MUST be secured using JWT token-based authentication:
- **Authentication**: JWT Bearer tokens required for all protected endpoints
- **Authorization**: Role-based or claims-based authorization enforced at API level
- **Token Validation**: Signature, expiration, issuer, and audience MUST be validated
- **HTTPS Only**: All API communication MUST use TLS/HTTPS in non-development environments
- **Token Storage**: Tokens stored securely (HttpOnly cookies or secure storage, never localStorage)
- **Token Expiration**: Access tokens MUST have short expiration (15-60 minutes)
- **Refresh Tokens**: Implement refresh token mechanism for long-lived sessions
- **Secrets Management**: API keys, certificates, and secrets stored in secure vaults (Azure Key Vault, HashiCorp Vault)

#### .NET JWT Implementation
```csharp
// Program.cs - JWT configuration
var builder = WebApplication.CreateBuilder(args);

// Add JWT authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

// Secured endpoint example
app.MapGet("/api/orders", 
    [Authorize] async (IOrderService orderService) => 
    {
        return await orderService.GetOrders();
    });

// Role-based authorization example
app.MapPost("/api/orders", 
    [Authorize(Roles = "Admin,Manager")] 
    async (CreateOrderRequest request, IOrderService orderService) => 
    {
        var result = await orderService.CreateOrder(request.ToCommand());
        return Results.Created($"/api/orders/{result.Id}", result);
    });

// Claims-based authorization example
app.MapDelete("/api/orders/{id}", 
    [Authorize(Policy = "CanDeleteOrders")] 
    async (Guid id, IOrderService orderService) => 
    {
        await orderService.DeleteOrder(id);
        return Results.NoContent();
    });

app.Run();
```

#### JWT Security Best Practices
- **Algorithm**: Use RS256 (asymmetric) for production; HS256 acceptable for internal services
- **Claims**: Include only necessary claims (sub, role, permissions); avoid sensitive data
- **Scope**: Implement granular scopes for API access control
- **Revocation**: Implement token revocation mechanism (blacklist or short expiration)
- **Rate Limiting**: Apply rate limiting per user/token to prevent abuse
- **CORS**: Configure CORS policies appropriately; never use wildcard in production
- **Audit Logging**: Log all authentication failures and authorization denials

## Architecture Standards

### Directory Structure
Project structure MUST reflect hexagonal architecture layers within each microservice:
```
/services                - Root for all microservices
  /service-name          - Individual microservice (one bounded context)
    /domain              - Pure business logic, entities, value objects, domain events
    /application         - Use cases, application services (inbound ports)
    /ports               - Interface definitions (both inbound and outbound)
    /adapters            - Implementations of ports
      /primary           - Inbound/driving adapters (REST, CLI, GraphQL, etc.)
      /secondary         - Outbound/driven adapters (repositories, external APIs, etc.)
    /infrastructure      - Cross-cutting concerns (logging, configuration, DI setup)
    /tests               - Mirrors source structure with unit and integration tests
    Dockerfile           - Container definition
    README.md            - Service documentation
/shared                  - Shared libraries (use sparingly)
  /contracts             - API contracts, events schemas
  /common                - Truly shared utilities (avoid domain logic)
```

### Backend Technology Stack
All backend services MUST be developed using:
- **Framework**: .NET 10 (or latest LTS version)
- **API Style**: Minimal APIs (not Controller-based)
- **Language**: C# with nullable reference types enabled
- **Dependency Injection**: Built-in .NET DI container
- **Configuration**: IConfiguration with appsettings.json and environment variables
- **Logging**: ILogger with structured logging (Serilog recommended)
- **Testing**: xUnit or NUnit with FluentAssertions

### Recommended Frameworks & Tools
The following frameworks and tools are RECOMMENDED for development:

#### Data Access & Persistence
- **Entity Framework Core**: ORM for data access (secondary adapters only)
  - Use Code-First approach for database development
  - Migrations managed via EF Core CLI tools
  - DbContext isolated in Infrastructure layer
  - Repository pattern implements outbound ports
- **Microsoft SQL Server**: Primary relational database
  - Each microservice has dedicated database instance
  - Database per service pattern strictly enforced
  - Use Azure SQL Database for cloud deployments

#### Distributed Application Runtime
- **Dapr (Distributed Application Runtime)**: Building block APIs for microservices
  - **Service Invocation**: Service-to-service calls with mTLS
  - **State Management**: Pluggable state stores
  - **Pub/Sub**: Publish/subscribe messaging
  - **Bindings**: Input/output bindings to external systems
  - **Secrets**: Secure secret management integration
  - **Observability**: Built-in distributed tracing and metrics
  - Configure as sidecar pattern alongside each service

#### Containerization & Orchestration
- **Docker**: Container platform for packaging services
  - Multi-stage Dockerfiles for optimized images
  - Base image: mcr.microsoft.com/dotnet/aspnet:10.0
  - Non-root user for security
  - Health check instructions in Dockerfile
- **Kubernetes**: Container orchestration platform
  - Each microservice deployed as Deployment + Service
  - ConfigMaps for configuration management
  - Secrets for sensitive data
  - Horizontal Pod Autoscaler (HPA) for scaling
  - Ingress controllers for external access
  - Use Azure Kubernetes Service (AKS) for cloud deployments

#### AI & Agent Framework
- **Azure Agent Framework**: AI agent orchestration and integration
  - Agent-based patterns for intelligent automation
  - Integration with Azure AI services
  - Implement as secondary adapter when integrating AI capabilities
  - Domain logic remains AI-agnostic; AI decisions communicated through ports

### Model Context Protocol (MCP) Standards
AI Agents MUST use the Model Context Protocol (MCP) for tool integration:

#### MCP Tool Sourcing Policy (NON-NEGOTIABLE)
- **First-Party Only**: Use ONLY MCP tools developed by the company providing the solution
- **No Third-Party Tools**: Third-party MCP tools are EXCLUDED from use
- **Build Over Buy**: If no first-party MCP tool exists, create a new microservice to implement required MCP capabilities
- **Remote Servers Only**: All MCP tools MUST use remote server architecture (no local tools allowed)

**Rationale**: First-party MCP tools ensure security, compliance, maintainability, and control over dependencies.

#### MCP Architecture Requirements
- **Remote MCP Servers**: All MCP servers deployed as remote services
  - Each MCP server is a dedicated microservice
  - Follows hexagonal architecture internally
  - Exposes MCP protocol endpoints
  - Deployed in containerized environment (Docker/Kubernetes)
- **No Local MCP Tools**: Local MCP tools (stdio, file-based) are prohibited
- **Service Discovery**: MCP servers registered in service discovery mechanism
- **Authentication**: MCP servers secured with JWT authentication
- **Observability**: MCP servers provide health checks, metrics, and distributed tracing

#### MCP Microservice Implementation
When creating new MCP tools as microservices:

```csharp
// MCP Server Microservice Structure
MCPToolName/
  MCPToolName.Domain/          - Business logic for tool capabilities
  MCPToolName.Application/     - Use cases, MCP protocol handlers
  MCPToolName.Infrastructure/  - External integrations, data access
  MCPToolName.Api/             - MCP protocol endpoints (SSE, HTTP)
    Program.cs                 - MCP server configuration
    Dockerfile                 - Container definition
  MCPToolName.Tests.Unit/
  MCPToolName.Tests.Integration/

// Program.cs - MCP Server Setup
var builder = WebApplication.CreateBuilder(args);

// Add MCP protocol support
builder.Services.AddMCPProtocol();

// Register tool capabilities (application layer)
builder.Services.AddScoped<IToolCapability, SpecificToolCapability>();

// Add authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* JWT config */ });

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

// MCP protocol endpoints
app.MapMCPEndpoints(); // Server-Sent Events for MCP protocol

// Health checks for MCP server
app.MapHealthChecks("/health");

app.Run();
```

#### MCP Tool Development Guidelines
- **Tool Catalog**: Maintain registry of all first-party MCP tools
- **Versioning**: MCP tools follow semantic versioning
- **Documentation**: Each MCP tool documents capabilities, parameters, and usage
- **Testing**: MCP tools include unit and integration tests for all capabilities
- **Schema Validation**: Input/output schemas strictly validated
- **Error Handling**: MCP tools return structured error responses
- **Rate Limiting**: Implement rate limiting per agent/user

#### MCP Tool Discovery & Registration
```csharp
// MCP Tool Registry Service
public interface IMCPToolRegistry
{
    Task<IEnumerable<MCPToolInfo>> GetAvailableToolsAsync();
    Task<MCPToolInfo> GetToolByNameAsync(string toolName);
    Task RegisterToolAsync(MCPToolInfo toolInfo);
}

public record MCPToolInfo(
    string Name,
    string Description,
    Uri ServerEndpoint,
    string Version,
    IReadOnlyList<ToolCapability> Capabilities,
    bool IsRemote,  // MUST always be true
    string Provider // MUST match company name
);

// AI Agent using MCP tools
public class AIAgentService
{
    private readonly IMCPToolRegistry _toolRegistry;
    private readonly IMCPClient _mcpClient;

    public async Task<AgentResponse> ExecuteAgentTaskAsync(AgentRequest request)
    {
        // Get available first-party MCP tools
        var tools = await _toolRegistry.GetAvailableToolsAsync();
        var firstPartyTools = tools.Where(t => t.Provider == "CompanyName" && t.IsRemote);

        // Agent uses MCP protocol to invoke tools
        var result = await _mcpClient.InvokeToolAsync(
            firstPartyTools.First(), 
            request.ToolParameters);

        return new AgentResponse(result);
    }
}
```

#### MCP Security Requirements
- **Authentication**: All MCP servers require JWT token authentication
- **Authorization**: Role-based or capability-based authorization for tool access
- **Encryption**: MCP communication over HTTPS/TLS only
- **Audit Logging**: Log all MCP tool invocations with agent/user context
- **Input Validation**: Strict validation of all tool parameters
- **Secrets**: Never expose secrets in MCP tool responses

#### Prohibited MCP Patterns
- ❌ Using third-party MCP tools (external providers)
- ❌ Local MCP servers (stdio, file-based)
- ❌ Unversioned MCP tools
- ❌ MCP tools without authentication
- ❌ Direct database access from MCP tools (must use domain services)
- ❌ MCP tools that bypass hexagonal architecture

#### MCP Tool Creation Process
1. **Requirement**: Identify need for new AI agent capability
2. **Evaluation**: Check if first-party MCP tool exists in registry
3. **Decision**: If not exists, create new microservice implementing MCP protocol
4. **Design**: Define tool capabilities, parameters, and schemas
5. **Implementation**: Build microservice following hexagonal architecture
6. **Testing**: Unit tests (domain), integration tests (MCP protocol)
7. **Deployment**: Deploy as containerized remote MCP server
8. **Registration**: Register in MCP tool registry
9. **Documentation**: Document tool capabilities and usage examples
10. **Monitoring**: Enable observability and set up alerts

#### Example: EF Core Code-First Implementation
```csharp
// Infrastructure/Data/OrderDbContext.cs (Secondary Adapter)
public class OrderDbContext : DbContext
{
    public OrderDbContext(DbContextOptions<OrderDbContext> options) 
        : base(options) { }

    public DbSet<OrderEntity> Orders { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<OrderEntity>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.CustomerId).IsRequired();
            entity.Property(e => e.TotalAmount).HasPrecision(18, 2);
        });
    }
}

// Infrastructure/Repositories/SqlOrderRepository.cs (Secondary Adapter)
public class SqlOrderRepository : IOrderRepository // Outbound Port
{
    private readonly OrderDbContext _context;

    public SqlOrderRepository(OrderDbContext context)
    {
        _context = context;
    }

    public async Task<Order> GetByIdAsync(Guid id)
    {
        var entity = await _context.Orders.FindAsync(id);
        return entity?.ToDomain(); // Map to domain model
    }

    public async Task SaveAsync(Order order)
    {
        var entity = OrderEntity.FromDomain(order);
        _context.Orders.Add(entity);
        await _context.SaveChangesAsync();
    }
}

// Program.cs - Configure EF Core
builder.Services.AddDbContext<OrderDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("OrderDatabase"),
        sqlOptions => sqlOptions.EnableRetryOnFailure()));
```

#### Example: Dapr Integration
```csharp
// Program.cs - Configure Dapr
builder.Services.AddControllers().AddDapr();

// Publish event via Dapr Pub/Sub (Secondary Adapter)
public class DaprEventPublisher : IEventPublisher // Outbound Port
{
    private readonly DaprClient _daprClient;

    public DaprEventPublisher(DaprClient daprClient)
    {
        _daprClient = daprClient;
    }

    public async Task PublishAsync(OrderCreatedEvent domainEvent)
    {
        await _daprClient.PublishEventAsync(
            "pubsub", 
            "order-created", 
            domainEvent);
    }
}

// Subscribe to events via Dapr
app.MapPost("/dapr/subscribe/order-created", 
    async (OrderCreatedEvent evt, IOrderService orderService) =>
    {
        await orderService.HandleOrderCreated(evt);
        return Results.Ok();
    });
```

#### Docker Multi-Stage Dockerfile Example
```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY ["ServiceName.Api/ServiceName.Api.csproj", "ServiceName.Api/"]
COPY ["ServiceName.Application/ServiceName.Application.csproj", "ServiceName.Application/"]
COPY ["ServiceName.Domain/ServiceName.Domain.csproj", "ServiceName.Domain/"]
COPY ["ServiceName.Infrastructure/ServiceName.Infrastructure.csproj", "ServiceName.Infrastructure/"]
RUN dotnet restore "ServiceName.Api/ServiceName.Api.csproj"
COPY . .
WORKDIR "/src/ServiceName.Api"
RUN dotnet build -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish -c Release -o /app/publish

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app
EXPOSE 80
EXPOSE 443
COPY --from=publish /app/publish .

# Run as non-root user
USER app
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80/health || exit 1

ENTRYPOINT ["dotnet", "ServiceName.Api.dll"]
```

#### .NET Minimal API Structure
```csharp
// Program.cs - Primary adapter configuration
var builder = WebApplication.CreateBuilder(args);

// Register domain services (application layer)
builder.Services.AddScoped<IOrderService, OrderService>();

// Register ports and adapters
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();

var app = builder.Build();

// Map endpoints (primary adapters)
app.MapPost("/api/orders", async (CreateOrderRequest request, IOrderService orderService) => 
{
    var result = await orderService.CreateOrder(request.ToCommand());
    return Results.Created($"/api/orders/{result.Id}", result);
});

app.Run();
```

#### .NET Project Structure per Service
```
ServiceName/
  ServiceName.Domain/          - Domain entities, value objects, domain events
  ServiceName.Application/     - Use cases, application services (ports)
  ServiceName.Infrastructure/  - Secondary adapters (repositories, external APIs)
  ServiceName.Api/             - Primary adapter (Minimal API endpoints)
    Program.cs                 - API configuration and endpoint mapping
    appsettings.json          - Configuration
    Dockerfile                - Container definition
  ServiceName.Tests.Unit/      - Unit tests for domain and application
  ServiceName.Tests.Integration/ - Integration tests for adapters
```

### Technology Constraints
- **Hexagonal Principles**: Architecture patterns apply regardless of tech stack
- **Framework Isolation**: .NET framework code (HTTP, EF Core) lives in adapters only
- **Database Agnostic Domain**: Domain models do not reference Entity Framework attributes or database types
- **API Agnostic Domain**: Domain services do not reference HttpContext, IResult, or HTTP-specific types
- **Nullable References**: C# nullable reference types MUST be enabled project-wide
- **Async/Await**: All I/O operations MUST use async/await pattern

### Implementation Guidelines
- **Interface Segregation**: Ports define minimal, focused interfaces
- **Single Responsibility**: Each adapter has one reason to change
- **Open/Closed**: New adapters can be added without modifying domain or ports
- **Explicit Dependencies**: All dependencies declared in constructors (no hidden coupling)

### .NET-Specific Guidelines
- **Records for DTOs**: Use record types for data transfer objects and requests/responses
- **Value Objects**: Implement as readonly record structs or sealed classes
- **Domain Events**: Use MediatR or Dapr Pub/Sub for event dispatcher pattern
- **Result Pattern**: Return Result<T> or Result<T, TError> from domain operations (no exceptions for business logic failures)
- **Validation**: FluentValidation for request validation in primary adapters
- **Mapping**: Avoid AutoMapper in domain; use explicit mapping methods or Mapster
- **Entity Framework Core**: Use only in secondary adapters (Infrastructure layer), never in domain
  - Code-First migrations for schema management
  - Repository pattern to abstract EF Core from application layer
- **Health Checks**: Implement IHealthCheck for all external dependencies (database, Dapr, external APIs)
- **API Versioning**: Use URL versioning (/api/v1/) or header-based versioning
- **Dapr Integration**: Use Dapr SDK for service invocation, pub/sub, state management
- **Container Best Practices**: Multi-stage Docker builds, non-root users, explicit health checks

## Development Workflow

### Architecture Review Gates
All changes MUST pass architecture compliance checks:
1. **Layer Dependency Check**: Verify domain has no outward dependencies
2. **Port Interface Check**: Ensure ports are abstract and technology-agnostic
3. **Adapter Isolation Check**: Confirm adapters don't leak into domain
4. **Test Coverage Check**: Domain core at 100%, adapters at 80% minimum
5. **Service Boundary Check**: Verify service doesn't access another service's database
6. **API Contract Check**: Ensure backward compatibility or proper versioning
7. **Resilience Pattern Check**: Confirm circuit breakers, timeouts, and retries implemented

### Code Review Requirements
- Verify hexagonal architecture principles respected
- Check that business logic resides in domain core
- Ensure adapters implement well-defined ports
- Validate test coverage meets minimum thresholds
- Confirm no framework-specific code in domain layer

### Refactoring Policy
When modifying existing code that violates architecture principles:
- Extract business logic to domain core
- Define appropriate ports for external dependencies
- Create adapters to implement ports
- Update tests to reflect new structure
- Document migration rationale in commit messages

## Governance

This constitution supersedes all other development practices and guidelines. All team members MUST:
- Adhere to hexagonal architecture principles in all new development
- Refactor existing code toward compliance when making modifications
- Raise concerns about architecture violations in code reviews
- Propose amendments through formal documentation and team consensus

### Amendment Process
1. Propose changes with clear rationale and impact analysis
2. Review with team leads and architects
3. Update version number according to semantic versioning:
   - **MAJOR**: Backward incompatible principle changes
   - **MINOR**: New principles or expanded guidance
   - **PATCH**: Clarifications, typo fixes, non-semantic refinements
4. Propagate changes to all dependent templates and documentation
5. Communicate changes to all stakeholders

### Compliance Verification
- Architecture compliance tools SHOULD be integrated into CI/CD pipeline
- Regular architecture reviews MUST be conducted quarterly
- Violations MUST be documented as technical debt with remediation plans
- New team members MUST complete architecture training before contributing

**Version**: 1.5.0 | **Ratified**: 2025-12-11 | **Last Amended**: 2025-12-12
