# syntax=docker/dockerfile:1

############################################
# STAGE 1: Build the application
############################################
FROM eclipse-temurin:21-jdk-alpine AS builder

LABEL maintainer="Shop2Many Team <dev@shop2many.com>"

# Install necessary tools
RUN apk add --no-cache bash git

# Set workdir
WORKDIR /app

# Copy Gradle wrapper and settings
COPY gradlew .
COPY gradle gradle

# Copy dependency declarations
COPY build.gradle.kts settings.gradle.kts ./

# Download dependencies (cache)
RUN chmod +x ./gradlew \
    && ./gradlew --no-daemon dependencies \
    && rm -rf ~/.gradle/caches/modules-2/*/cache.properties

# Copy source code
COPY src src

# Build JAR
RUN ./gradlew --no-daemon bootJar \
    && mv build/libs/shop2many-backend-*.jar shop2many-backend.jar

############################################
# STAGE 2: Create minimal runtime image
############################################
FROM eclipse-temurin:21-jre-alpine AS runtime

LABEL maintainer="Shop2Many Team <dev@shop2many.com>"

# Create a non-root user
RUN addgroup -S shop2many && adduser -S shop2many -G shop2many

# Copy JAR from builder
WORKDIR /home/shop2many
COPY --from=builder /app/shop2many-backend.jar .

# Expose application port
EXPOSE 8080

# Use non-root user
USER shop2many

# Default entrypoint
ENTRYPOINT ["java","-jar","shop2many-backend.jar"]
