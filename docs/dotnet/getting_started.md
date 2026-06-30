## Creating Dotnet App using Dotnet Core CLI

### 1. Create the sln

```bash
dotnet new sln --name <PROJECT_NAME>
```

### 2. Create The Project

Using clean architecture we want to create project for each layer
1. Presentation 
Here we use webapi as the presentation layer.  
```bash
dotnet new webapi -o <PROJECT_NAME>.Api
```
2. Infrastructure 
The Infrastructure layer represent connection to outside system (DB, Email, etc).  
```bash
dotnet new classlib -o <PROJECT_NAME>.Infrastructure
```
3. Application
The Application layer is where logic of the program lives. it contains all use cases that the application needs.  
```bash
dotnet new classlib -o <PROJECT_NAME>.Application
```
4. Domain 
The Domain layer is the core of the application which will be referenced by other layers. It contains entities, etc.  
```bash
dotnet new classlib -o <PROJECT_NAME>.Domain
```

### 3. Add all the project to the sln

In Windows Powershell
```bash
dotnet sln <PROJECT_NAME>.sln add (ls -r **\*.csproj)
```

or in mac
```bash
dotnet sln <PROJECT_NAME>.sln add **/*.csproj
```

### 4. Configure dependency for each project

Presentation layer should depend on application layer
```bash
dotnet add <PROJECT_NAME>.Api reference <PROJECT_NAME>.Application
```
   
Infrastructure layer should als depend on application layer
```bash
dotnet add <PROJECT_NAME>.Infrastructure reference <PROJECT_NAME>.Application
```

Application layer should depend on Domain layer
```bash
dotnet add <PROJECT_NAME>.Application reference <PROJECT_NAME>.Domain
```

And the Domain layer should not depend on any other layers.

Notes:
Presentation layer need to depend on Infrastructure layer to register the dependency.
```bash
dotnet add <PROJECT_NAME>.Api reference <PROJECT_NAME>.Infrastructure
```

### 5. Configure Dependency Injection
In Api project configure the Program.
In each layer setup dependency injection.


## Build the project
To build the projects run this command
```bash
dotnet build
```

## Run the project
To run api 
```bash
dotnet run --project <PROJECT_NAME>.Api
```

# Connect to existing database

```bash
dotnet ef dbcontext scaffold "<connection_string>" Npgsql.EntityFrameworkCore.PostgreSQL  --project LibraryApp.Api --output-dir Models --force
```

## Create migrations

```bash
dotnet ef migrations add InitialCreate -p LibraryApp.Infrastructure -s LibraryApp.Web -o <output-folder>
```

## Apply migrations

```bash
dotnet ef database update
```