# GenSolut
Criação de solution genérica automatizada

> [!WARNING]
> Em construção.

> [!NOTE]
> Todo:
> - Implementação do CRUD genérico
> - Implementação de logs de erro para procedimentos DML

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
dotnet new xunit -n Test

# Adicionar os projetos à Solution
dotnet sln add Api/Api.csproj
dotnet sln add Application/Application.csproj
dotnet sln add Domain/Domain.csproj
dotnet sln add DataTransfer/DataTransfer.csproj
dotnet sln add Infrastructure/Infrastructure.csproj
dotnet sln add IOC/IOC.csproj
dotnet sln add Test/Test.csproj

# Adicionar referências entre os projetos
dotnet add Api/Api.csproj reference Application/Application.csproj
dotnet add Api/Api.csproj reference IOC/IOC.csproj
dotnet add Application/Application.csproj reference Domain/Domain.csproj
dotnet add Application/Application.csproj reference Infrastructure/Infrastructure.csproj
dotnet add Application/Application.csproj reference DataTransfer/DataTransfer.csproj
dotnet add Domain/Domain.csproj reference DataTransfer/DataTransfer.csproj
dotnet add Domain/Domain.csproj reference Infrastructure/Infrastructure.csproj
dotnet add Infrastructure/Infrastructure.csproj reference IOC/IOC.csproj
dotnet add Test/Test.csproj reference Domain/Domain.csproj

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

// Add Swagger services
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddInfrastructureServices(builder.Configuration);
builder.Services.AddIOCServices();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI();
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

cat <<EOT > Controllers/EntityExampleController.cs
using Application.Services;
using Domain.Entities;
using Microsoft.AspNetCore.Mvc;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace Api.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class EntityExampleController : ControllerBase
    {
        private readonly EntityExampleApplicationService _entityExampleApplicationService;

        public EntityExampleController(EntityExampleApplicationService entityExampleApplicationService)
        {
            _entityExampleApplicationService = entityExampleApplicationService;
        }

        [HttpGet("{id}")]
        public async Task<ActionResult<EntityExample>> GetById(int id)
        {
            var entity = await _entityExampleApplicationService.GetByIdAsync(id);
            if (entity == null)
            {
                return NotFound();
            }
            return Ok(entity);
        }

        [HttpGet]
        public async Task<ActionResult<IEnumerable<EntityExample>>> GetAll()
        {
            var entities = await _entityExampleApplicationService.GetAllAsync();
            return Ok(entities);
        }

        [HttpPost]
        public async Task<ActionResult<EntityExample>> Create([FromBody] EntityExample entity)
        {
            if (entity == null)
            {
                return BadRequest();
            }

            await _entityExampleApplicationService.AddAsync(entity);
            return CreatedAtAction(nameof(GetById), new { id = entity.Id }, entity);
        }

        [HttpPut("{id}")]
        public async Task<IActionResult> Update(int id, [FromBody] EntityExample entity)
        {
            if (id != entity.Id)
            {
                return BadRequest();
            }

            var existingEntity = await _entityExampleApplicationService.GetByIdAsync(id);
            if (existingEntity == null)
            {
                return NotFound();
            }

            await _entityExampleApplicationService.UpdateAsync(entity);
            return NoContent();
        }

        [HttpDelete("{id}")]
        public async Task<IActionResult> Delete(int id)
        {
            var existingEntity = await _entityExampleApplicationService.GetByIdAsync(id);
            if (existingEntity == null)
            {
                return NotFound();
            }

            await _entityExampleApplicationService.DeleteAsync(id);
            return NoContent();
        }
    }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada Application
cd Application
mkdir -p Services
cat <<EOT > Services/EntityExampleApplicationService.cs
using Domain.Entities;
using Domain.Repositories;
using Infrastructure.Repositories;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace Application.Services
{
    public class EntityExampleApplicationService
    {
        private readonly IEntityExampleRepository _entityExampleRepository;

        public EntityExampleApplicationService(IEntityExampleRepository entityExampleRepository)
        {
            _entityExampleRepository = entityExampleRepository;
        }

        public async Task<EntityExample> GetByIdAsync(int id)
        {
            return await _entityExampleRepository.GetByIdAsync(id);
        }

        public async Task<IEnumerable<EntityExample>> GetAllAsync()
        {
            return await _entityExampleRepository.GetAllAsync();
        }

        public async Task AddAsync(EntityExample entity)
        {
            await _entityExampleRepository.AddAsync(entity);
        }

        public async Task UpdateAsync(EntityExample entity)
        {
            await _entityExampleRepository.UpdateAsync(entity);
        }

        public async Task DeleteAsync(int id)
        {
            await _entityExampleRepository.DeleteAsync(id);
        }
    }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada Domain
cd Domain
mkdir -p Repositories
cat <<EOT > Entities/EntityExample.cs
using System;

namespace Domain.Entities
{
    public class EntityExample
    {
        public int Id { get; private set; }
        public string Name { get; private set; }
        public DateTime Date { get; private set; }

        public EntityExample(int id, string name, DateTime date)
        {
            SetId(id);
            SetName(name);
            SetDate(date);
        }

        public void SetId(int id)
        {
            if (id <= 0)
                throw new ArgumentException("Id must be greater than zero.");
            Id = id;
        }

        public void SetName(string name)
        {
            if (string.IsNullOrWhiteSpace(name) || name.Length < 3)
                throw new ArgumentException("Name must be at least 3 characters long.");
            Name = name;
        }

        public void SetDate(DateTime date)
        {
            Date = date;
        }
    }
}
EOT

cat <<EOT > Repositories/IEntityExampleRepository.cs
using Domain.Entities;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace Domain.Repositories
{
    public interface IEntityExampleRepository : IRepositoryBase<EntityExample>
    {
    }
}
EOT

cat <<EOT > Services/IEntityExampleDomainService.cs
using Domain.Entities;
using System;

namespace Domain.Services
{
    public interface IEntityExampleDomainService
    {
        EntityExample Instantiate(int id, string name, DateTime date);
    }
}
EOT

cat <<EOT > Services/EntityExampleDomainService.cs
using Domain.Entities;
using System;

namespace Domain.Services
{
    public class EntityExampleDomainService : IEntityExampleDomainService
    {
        public EntityExample Instantiate(int id, string name, DateTime date)
        {
            return new EntityExample(id, name, date);
        }
    }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada Infrastructure
cd Infrastructure
mkdir -p Repositories
cat <<EOT > Repositories/EntityExampleRepository.cs
using Domain.Entities;
using Domain.Repositories;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace Infrastructure.Repositories
{
    public class EntityExampleRepository : IEntityExampleRepository
    {
        // Simulating a data store with an in-memory list
        private readonly List<EntityExample> _dataStore = new();

        public async Task<EntityExample> GetByIdAsync(int id)
        {
            return await Task.FromResult(_dataStore.Find(e => e.Id == id));
        }

        public async Task<IEnumerable<EntityExample>> GetAllAsync()
        {
            return await Task.FromResult(_dataStore.AsEnumerable());
        }

        public async Task AddAsync(EntityExample entity)
        {
            _dataStore.Add(entity);
            await Task.CompletedTask;
        }

        public async Task UpdateAsync(EntityExample entity)
        {
            var existingEntity = _dataStore.Find(e => e.Id == entity.Id);
            if (existingEntity != null)
            {
                _dataStore.Remove(existingEntity);
                _dataStore.Add(entity);
            }
            await Task.CompletedTask;
        }

        public async Task DeleteAsync(int id)
        {
            var entity = _dataStore.Find(e => e.Id == id);
            if (entity != null)
            {
                _dataStore.Remove(entity);
            }
            await Task.CompletedTask;
        }
    }
}
EOT

cat <<EOT > IRepositoryBase.cs
using System.Collections.Generic;
using System.Threading.Tasks;

namespace Infrastructure
{
    public interface IRepositoryBase<T>
    {
        Task<T> GetByIdAsync(int id);
        Task<IEnumerable<T>> GetAllAsync();
        Task AddAsync(T entity);
        Task UpdateAsync(T entity);
        Task DeleteAsync(int id);
    }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada IOC
cd IOC
cat <<EOT > IOCServiceExtensions.cs
using Application.Services;
using Domain.Repositories;
using Infrastructure.Repositories;
using Microsoft.Extensions.DependencyInjection;

namespace IOC
{
    public static class IOCServiceExtensions
    {
        public static void AddInfrastructureServices(this IServiceCollection services, IConfiguration configuration)
        {
            // Configure any additional services here if needed
        }

        public static void AddIOCServices(this IServiceCollection services)
        {
            services.AddScoped<IEntityExampleRepository, EntityExampleRepository>();
            services.AddScoped<EntityExampleApplicationService>();
            services.AddScoped<IEntityExampleDomainService, EntityExampleDomainService>();
        }
    }
}
EOT
cd ..

# Criar a estrutura e arquivos para a camada Test
cd Test
mkdir -p Entities DomainServices
cat <<EOT > Entities/EntityExampleTests.cs
using Domain.Entities;
using System;
using Xunit;

namespace Test.Entities
{
    public class EntityExampleTests
    {
        [Fact]
        public void Constructor_Should_Set_Properties_Correctly()
        {
            // Arrange
            int id = 1;
            string name = "TestName";
            DateTime date = DateTime.Now;

            // Act
            var entity = new EntityExample(id, name, date);

            // Assert
            Assert.Equal(id, entity.Id);
            Assert.Equal(name, entity.Name);
            Assert.Equal(date, entity.Date);
        }

        [Fact]
        public void SetId_Should_Throw_Exception_For_Invalid_Id()
        {
            // Arrange
            var entity = new EntityExample(1, "TestName", DateTime.Now);

            // Act & Assert
            Assert.Throws<ArgumentException>(() => entity.SetId(0));
        }

        [Fact]
        public void SetName_Should_Throw_Exception_For_Invalid_Name()
        {
            // Arrange
            var entity = new EntityExample(1, "TestName", DateTime.Now);

            // Act & Assert
            Assert.Throws<ArgumentException>(() => entity.SetName("AB"));
        }
    }
}
EOT

cat <<EOT > DomainServices/EntityExampleDomainServiceTests.cs
using Domain.Entities;
using Domain.Services;
using System;
using Xunit;

namespace Test.DomainServices
{
    public class EntityExampleDomainServiceTests
    {
        [Fact]
        public void Instantiate_Should_Create_Entity_Correctly()
        {
            // Arrange
            var service = new EntityExampleDomainService();
            int id = 1;
            string name = "TestName";
            DateTime date = DateTime.Now;

            // Act
            var entity = service.Instantiate(id, name, date);

            // Assert
            Assert.Equal(id, entity.Id);
            Assert.Equal(name, entity.Name);
            Assert.Equal(date, entity.Date);
        }
    }
}
EOT
cd ..
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
