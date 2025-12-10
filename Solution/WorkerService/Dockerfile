# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0-alpine AS build
ARG BUILD_CONFIGURATION=Release
ARG VERSION=1.0.0
WORKDIR /src

# Copy project files and restore dependencies (cached layer)
COPY ["Domain/Domain.csproj", "Domain/"]
COPY ["Application/Application.csproj", "Application/"]
COPY ["Infrastructure/Infrastructure.csproj", "Infrastructure/"]
COPY ["IoC/IoC.csproj", "IoC/"]
COPY ["WorkerService/WorkerService.csproj", "WorkerService/"]

# Restore dependencies
RUN dotnet restore "WorkerService/WorkerService.csproj"

# Copy source code
COPY . .

# Publish optimized build
WORKDIR /src/WorkerService
RUN dotnet publish "WorkerService.csproj" \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    --no-restore \
    --self-contained false \
    /p:DebugType=None \
    /p:DebugSymbols=false

# Final runtime image - using minimal runtime Alpine image
FROM mcr.microsoft.com/dotnet/runtime:10.0-alpine AS final
ARG VERSION=1.0.0

# Add metadata labels
LABEL org.opencontainers.image.title="WorkerService Payment Processor" \
      org.opencontainers.image.description="Payment Worker Service with Clean Architecture" \
      org.opencontainers.image.version="${VERSION}" \
      org.opencontainers.image.vendor="Payment Service"

WORKDIR /app

# Create non-root user for security
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -s /bin/sh -D appuser && \
    chown -R appuser:appgroup /app

USER appuser

# Copy only published files
COPY --from=build --chown=appuser:appgroup /app/publish .

# Set environment variables for optimization
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=true \
    DOTNET_EnableDiagnostics=0 \
    DOTNET_GCHeapHardLimit=0x8000000

# Health check - verifica se o processo está rodando
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD pgrep -f "dotnet WorkerService.dll" || exit 1

ENTRYPOINT ["dotnet", "WorkerService.dll"]