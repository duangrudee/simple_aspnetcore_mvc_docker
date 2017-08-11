# simple_aspnetcore_mvc_docker

This project was created by following the guideline in [Quickstart: Compose and ASP.NET Core with SQL Server](https://docs.docker.com/compose/aspnet-mssql-compose/)

## Prerequisite
1. Docker install 


## Running instruction

1. Clone this repo

`$ git clone [url] `

2. Build a docker container using the configuration provided in a Dockerfile and docker-compose.yml

`$ docker-compose build`

3. Run the container

`$ docker-compose up`

4. Browse the website through [http://localhost:8000](http://localhost:8000)

## How I got here
#### Bunch of dockers command to get the working directory where it is rightnow (just in case the above link isn't working)
1. Create a Docker container ASP.NET core build image and create ASP.NET Core project using MVC template

``$ docker run -v ${PWD}:/app --workdir /app microsoft/aspnetcore-build:lts dotnet new mvc --auth Individual``

2. Create a Dockerfile within your app directory and add the following content:

```
FROM microsoft/aspnetcore-build:lts
COPY . /app
WORKDIR /app
RUN ["dotnet", "restore"]
RUN ["dotnet", "build"]
EXPOSE 80/tcp
RUN chmod +x ./entrypoint.sh
CMD /bin/bash ./entrypoint.sh
```

3. The Dockerfile makes use of an entrypoint to your webapp Docker image. Create this script in a file called entrypoint.sh and paste the contents below.
```
#!/bin/bash

set -e
run_cmd="dotnet run --server.urls http://*:80"

until dotnet ef database update; do
>&2 echo "SQL Server is starting up"
sleep 1
done

>&2 echo "SQL Server is up - executing command"
exec $run_cmd
```

4. Create a docker-compose.yml file. Write the following in the file, and make sure to replace the password in the SA_PASSWORD environment variable under db below. This file will define the way the images will interact as independent services.

```
version: "3"
services:
    web:
        build: .
        ports:
            - "8000:80"
        depends_on:
            - db
    db:
        image: "microsoft/mssql-server-linux"
        environment:
            SA_PASSWORD: "your_password"
            ACCEPT_EULA: "Y"
```

5. Go to Startup.cs and locate the function called ConfigureServices (Hint: it should be under line 42). Replace the entire function to use the following code (watch out for the brackets!).            
```

public void ConfigureServices(IServiceCollection services)
{
    // Database connection string.
    // Make sure to update the Password value below from "your_password" to your actual password.
    var connection = @"Server=db;Database=master;User=sa;Password=your_password;";

    // This line uses 'UseSqlServer' in the 'options' parameter
    // with the connection string defined above.
    services.AddDbContext<ApplicationDbContext>(
        options => options.UseSqlServer(connection));

    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.AddMvc();

    // Add application services.
    services.AddTransient<IEmailSender, AuthMessageSender>();
    services.AddTransient<ISmsSender, AuthMessageSender>();
}
```

6. Go to app.csproj. add the following just after sqlite package reference

<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="1.1.2" />

