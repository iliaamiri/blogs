**Deploy a .NET app on Railway**
**Before starting, make sure you have: **
- Installed .NET 7 sdk.
- Installed Docker and ran its service.
- Installed Railway CLI from https://docs.railway.app/develop/cli

Dockerfile (Auto-generated by Rider IDE):
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["MyApp/MyApp.csproj", "MyApp/"]
RUN dotnet restore "MyApp/MyApp.csproj"
COPY . .
WORKDIR "/src/MyApp"
RUN dotnet build "MyApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

Imagining this is our app's source code:
/MyApp
--/MyApp
----/[*Your MyApp project source code*]
--/MyApp.sln

make a `railway.toml` file under `/MyApp` and with this content:
```toml
[build]
builder = "dockerfile"
dockerfilePath = "./MyApp/Dockerfile"

[deploy]
startCommand = "dotnet MyApp.dll"
restartPolicyType = "never"
```

Go to your `Program.cs` at `/MyApp/MyApp/Program.cs`. And, add these two lines before `var app = builder.Build();` :
```csharp
var port = Environment.GetEnvironmentVariable("PORT") ?? "8081";
builder.WebHost.UseUrls($"http://0.0.0.0:{port}");
```
:warning:**NOTE**: comment out the line that forces HTTPS redirection. (`app.UseHttpsRedirection();`) in your `Program.cs`.

**2. Railway Moment**

2.0 navigate to `/MyApp` folder in your command line shell.

2.1 run `railway login` and follow the instructions to login on your railway account.

2.2 (optional) run `railway run dotnet run --project ./MyApp` and make sure that your app is running (local)

2.3 On your railway, "Start a New Project" and then choose the "Empty project" from the dropdown menu.

2.4 Add a Service. and choose "Empty Service".

2.5 On you terminal, (staying in the `/MyApp` working directory), run `railway link` and then choose your project that you want to link your .NET app to. (There will be a list of your railway projects by their names.) 

2.6 Finally, run `railway up` and wait until your deployment builds, and deploys on railway. You can checkout the deployment starting on your Railway project (service) deployment page.

2.7 And maybe generate a new domain so that you can access your app via that.

**Resources, if you're curious**
- https://rendle.dev/posts/deploying-to-railway-with-dotnet/
- https://docs.railway.app/develop/cli
- https://stackoverflow.com/questions/70502758/how-do-i-inject-herokus-port-value-into-a-dotnet-core-6-0-web-api
- https://stackoverflow.com/questions/38755516/how-to-change-the-port-number-for-asp-net-core-app
- https://docs.railway.app/deploy/config-as-code

---

if anyone is having their solution and project in the same folder, here's how your Dockerfile should almost look like:
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["BallApp.csproj", "./"]
RUN dotnet restore "BallApp.csproj"
COPY . .
WORKDIR "/src/"
RUN dotnet build "BallApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "BallApp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "BallApp.dll"]
```