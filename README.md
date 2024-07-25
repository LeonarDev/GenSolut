# GenSolut
Criação de solution genérica automatizada

> [!WARNING]
> Em construção.

<br>
<hr>

<details>
  <summary>**sh script** (clique para expandir)</summary>
  
```sh
#!/bin/bash

# Nome da Solution
SOLUTION_NAME="GenericSolution"

# Criar a Solution
dotnet new sln -n $SOLUTION_NAME

# Criar os projetos
dotnet new webapi -n Api
dotnet new classlib -n Application
dotnet new classlib -n Domain
dotnet new classlib -n DataTransfer
dotnet new classlib -n Infrastructure
dotnet new classlib -n IOC

# Adicionar os projetos à Solution
dotnet sln add Api/Api.csproj
dotnet sln add Application/Application.csproj
dotnet sln add Domain/Domain.csproj
dotnet sln add DataTransfer/DataTransfer.csproj
dotnet sln add Infrastructure/Infrastructure.csproj
dotnet sln add IOC/IOC.csproj

# Adicionar referências entre os projetos
dotnet add Api/Api.csproj reference Application/Application.csproj
dotnet add Api/Api.csproj reference IOC/IOC.csproj
dotnet add Application/Application.csproj reference Domain/Domain.csproj
dotnet add Application/Application.csproj reference Infrastructure/Infrastructure.csproj
dotnet add Application/Application.csproj reference DataTransfer/DataTransfer.csproj
dotnet add Domain/Domain.csproj reference DataTransfer/DataTransfer.csproj
dotnet add Domain/Domain.csproj reference Infrastructure/Infrastructure.csproj
dotnet add Infrastructure/Infrastructure.csproj reference IOC/IOC.csproj

# Criar a estrutura e arquivos para a camada Api
cd Api
mkdir -p Controllers
cat <<EOT > Program.cs
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using IOC;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddInfrastructureServices(builder.Configuration);
builder.Services.AddIOCServices();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
EOT

cat <<EOT > appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "YourConnectionStringHere"
  }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada Application
cd Application
mkdir -p Services
cat <<EOT > Services/ApplicationService.cs
namespace Application.Services
{
    public class ApplicationService
    {
        // Implementação dos serviços de aplicação
    }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada Domain
cd Domain
mkdir -p Entities Repositories Services
cat <<EOT > Entities/Entity.cs
namespace Domain.Entities
{
    public class Entity
    {
        public int Id { get; set; }
        // Outras propriedades
    }
}
EOT

cat <<EOT > Repositories/IRepository.cs
namespace Domain.Repositories
{
    public interface IRepository<T>
    {
        Task<T> GetByIdAsync(int id);
        Task<IEnumerable<T>> GetAllAsync();
        Task AddAsync(T entity);
        Task UpdateAsync(T entity);
        Task DeleteAsync(int id);
    }
}
EOT

cat <<EOT > Services/DomainService.cs
namespace Domain.Services
{
    public class DomainService
    {
        // Implementação dos serviços de domínio
    }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada DataTransfer
cd DataTransfer
mkdir -p Requests Responses
cat <<EOT > Requests/ExampleRequest.cs
namespace DataTransfer.Requests
{
    public class ExampleRequest
    {
        public string Property { get; set; }
        // Outras propriedades
    }
}
EOT

cat <<EOT > Responses/ExampleResponse.cs
namespace DataTransfer.Responses
{
    public class ExampleResponse
    {
        public string Property { get; set; }
        // Outras propriedades
    }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada Infrastructure
cd Infrastructure
mkdir -p Repositories Data
cat <<EOT > Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using Domain.Entities;

namespace Infrastructure.Data
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

        public DbSet<Entity> Entities { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);
            // Configurações adicionais do modelo
        }
    }
}
EOT

cat <<EOT > Repositories/Repository.cs
using Domain.Repositories;
using Microsoft.EntityFrameworkCore;

namespace Infrastructure.Repositories
{
    public class Repository<T> : IRepository<T> where T : class
    {
        private readonly AppDbContext _context;
        private readonly DbSet<T> _dbSet;

        public Repository(AppDbContext context)
        {
            _context = context;
            _dbSet = _context.Set<T>();
        }

        public async Task<T> GetByIdAsync(int id)
        {
            return await _dbSet.FindAsync(id);
        }

        public async Task<IEnumerable<T>> GetAllAsync()
        {
            return await _dbSet.ToListAsync();
        }

        public async Task AddAsync(T entity)
        {
            await _dbSet.AddAsync(entity);
            await _context.SaveChangesAsync();
        }

        public async Task UpdateAsync(T entity)
        {
            _dbSet.Update(entity);
            await _context.SaveChangesAsync();
        }

        public async Task DeleteAsync(int id)
        {
            var entity = await _dbSet.FindAsync(id);
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}
EOT

cat <<EOT > ServiceCollectionExtensions.cs
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.EntityFrameworkCore;
using Infrastructure.Data;
using Domain.Repositories;

namespace Infrastructure
{
    public static class ServiceCollectionExtensions
    {
        public static IServiceCollection AddInfrastructureServices(this IServiceCollection services, IConfiguration configuration)
        {
            services.AddDbContext<AppDbContext>(options =>
                options.UseSqlServer(configuration.GetConnectionString("DefaultConnection")));
            services.AddScoped(typeof(IRepository<>), typeof(Repository<>));

            return services;
        }
    }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada IOC
cd IOC
cat <<EOT > ServiceCollectionExtensions.cs
using Microsoft.Extensions.DependencyInjection;
using Application.Services;
using Domain.Services;

namespace IOC
{
    public static class ServiceCollectionExtensions
    {
        public static IServiceCollection AddIOCServices(this IServiceCollection services)
        {
            // Configuração dos serviços da aplicação
            services.AddScoped<ApplicationService>();
            services.AddScoped<DomainService>();

            // Outros serviços podem ser registrados aqui

            return services;
        }
    }
}
EOT
cd ..

echo "Setup concluído com sucesso!"
```

</details>

<hr>
<br>

Torne o arquivo executável:

```ps
chmod +x setup.sh
```

Execute o script:

```ps
./setup.sh
```
