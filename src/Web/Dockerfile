# RUN ALL CONTAINERS FROM ROOT (folder with .sln file):
# docker-compose build
# docker-compose up
#
# RUN JUST THIS CONTAINER FROM ROOT (folder with .sln file):
# docker build --pull -t web -f src/Web/Dockerfile .
#
# RUN COMMAND
#  docker run --name eshopweb --rm -it -p 5106:5106 web
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build

ARG Nuget_CustomFeedUserName
ARG Nuget_CustomFeedPassword

WORKDIR /app

# Copy csproj, sln and restore as distinct layers
COPY *.sln .
COPY . .
WORKDIR /app/src/Web
RUN dotnet restore

# publish app and libraries
RUN dotnet publish -c Release -o out  --no-restore 

FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS runtime
WORKDIR /app
COPY --from=build /app/src/Web/out ./

# Optional: Set this here if not setting it from docker-compose.yml
# ENV ASPNETCORE_ENVIRONMENT Development

ENTRYPOINT ["dotnet", "Web.dll"]
