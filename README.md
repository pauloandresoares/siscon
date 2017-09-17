# Aplicação para estudo do .NET CORE

# Criar
**1 - Criar o projeto**
Se não definir o nome do projeto ele irá pegar o diretório atual
```sh
dotnet new mvc <name> 
```

**2 - Instalar o code generator tools**
```sh
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Tools
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
```
Caso no arquivo do projeto .csproj não tiver a linha abaixo, adiciona-la:
```xml
  <ItemGroup>
    <DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.0" />
  </ItemGroup>
```  

**3 - Instalar todas as dependencias**
```
dotnet restore
```
**4 -Comando usado para copilar**
```
dotnet build 
```
**5 - Gerar uma controller:**
É importante informar a opção "--relativeFolderPath Controllers" pq caso contrario ele irá gera na raiz do projeto
```
dotnet aspnet-codegenerator controller  -name <Nome da Controller. Ex.: OrderController> --relativeFolderPath Controllers
```
**6 - Gerar uma view**
```
dotnet aspnet-codegenerator view <Nome> <tipo: Empty|Create|Edit|Delete|Details|List> -m <Model> --relativeFolderPath Views\Order  --useDefaultLayout
```
O -m deverá ser utilizado quando vc precisar utilizar um modelo. Por exemplo uma página de listagem.

| Tipo |  |
| ------ | ------ |
| Empty | Página vazia |
| Create | Pagina com um formulário de cadastro |
| Edit |  Pagina com um formulário de edição |
| Delete | Página de exclusão |
| Details | Página de detalhes |
| List | Página de listagem |

**7 - Instalar o Entity Framework Core e as suas dependencias**
Instalar o Entity Framework Core:
```
dotnet add package Microsoft.EntityFrameworkCore
```
Instalar o Entity Framework Core Tools:
```
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

Instalar o provider o Postgres SQL:
```
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

Pacote para facilitar a migração utilizando o provider do Postgres SQL:
```
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL.Design
```
**8 - Adicionar o Entity Framework para ser utilizado por linha de comando**
```
dotnet add package Microsoft.EntityFrameworkCore.Tools.DotNe
```

Abra o arquivo <Nome do Projeto>.csproj e adicione a seguinte linha:
```xml
<DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" />
```
Para ficar dessa forma:
```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="2.0.0" />
    <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="2.0.0" />
    <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.0" />
    <PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="2.0.0" />
    <PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL.Design" Version="1.1.1" />
  </ItemGroup>
  <ItemGroup>
    <DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.0" />
  
	// >>>>> AQUI O EXEMPLO <<<<<<<<
	<DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" />
    
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="2.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" />
  </ItemGroup>	
```  

**9 -  Caso não exista a pasta Models na raiz do projeto criar e criar a classe como exemplo**
```csharp
// Models/Person.cs
namespace Sisdem.Models
{
    public class Person
    {
        public int Id { get; set;}
        public string Name { get; set;}
        public int Age { get; set;}
        public string Description { get; set;}
    }
}
``` 



**10 - Criar o diretorio Context e modificar o <Nome do Projeto>.csproj:**
Adicionando uma nova tag ItemGroup:
```xml 
  <!-- Checar a real necessidade da linha abaixo: -->
   <ItemGroup>
      <folder Include="/Models" />
      <folder Include="/Context" />
   </ItemGroup>
``` 

**11 - Criar o aqruivo DBContext :**
```csharp 
//SisdemDBContext:

using Microsoft.EntityFrameworkCore;
using Sisdem.Models;

namespace Sisdem.Context
{
    public class SisdemDBContext : DbContext
    {
        public DbSet<Person> Person { get; set; }

        public SisdemDBContext(DbContextOptions<SisdemDBContext> opts)  
            :  base (opts)
        {

        }
    }
}  
```

**12 - Modificar o arquivo appsettings.json**
```json 
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  
  "ConnectionStrings": {
    "SisdemDBContext":  "User ID=postgres; Password=abcd1234; Host=localhost; Port=5432; Database=Sisdem"
  }
}
``` 
**13 - Modificar a classe: Startup.cs e o metodo: ConfigureServices para adicionar o novo contexto**
Adicionar estes namespace:
```csharp 
using Sisdem.Context;
using Microsoft.EntityFrameworkCore;
``` 

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
	
	// --> Inicio
    string connectionString = Configuration.GetConnectionString("SisdemDBContext");
    services.AddDbContext<SisdemDBContext>((optBuilder) => 
    {
        optBuilder.UseNpgsql(connectionString);
    });
	// --> Fim
}
```   
**14 - Fazer uma migração**	
``` 
dotnet ef migrations add create_person
``` 
**15 - Executar uma migração**
``` 
dotnet ef database update
``` 

**16 - Gerar uma controller com uma model**
``` 
dotnet aspnet-codegenerator controller  -name PersonController --model Sisdem.Models.Person --dataContext Sisdem.Context.SisdemDBContext --relativeFolderPath Controllers --useDefaultLayout
``` 
**17 - Modificar o arquivo _Layout.cshtml**
```html 
<!-- e adicionar o link: -->
<li><a asp-area="" asp-controller="Person" asp-action="Index">Person</a></li>
``` 

# Autenticação
