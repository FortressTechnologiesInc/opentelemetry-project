# Use Eclipse Temurin base image for building the application
FROM eclipse-temurin:21-jdk as builder

WORKDIR /usr/src/app/

# Copy gradle and project files
COPY ./src/adservice/gradlew* ./src/adservice/settings.gradle* ./src/adservice/build.gradle ./
COPY ./src/adservice/gradle ./gradle

# Execute gradle commands to set up the build environment
RUN ./gradlew
RUN ./gradlew downloadRepos

# Copy source code and proto files
COPY ./src/adservice/ ./
COPY ./pb/ ./proto

# List contents of the build context to ensure files are present
RUN ls -R /usr/src/app

# Build the application
RUN ./gradlew installDist -PprotoSourceDir=./proto

# Use Eclipse Temurin base image for running the application
FROM eclipse-temurin:21-jre

ARG version=2.3.0
WORKDIR /usr/src/app/

# Copy built application from builder stage
COPY --from=builder /usr/src/app/ ./

# Add OpenTelemetry agent
ADD --chmod=644 https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v$version/opentelemetry-javaagent.jar /usr/src/app/opentelemetry-javaagent.jar
ENV JAVA_TOOL_OPTIONS=-javaagent:/usr/src/app/opentelemetry-javaagent.jar

# Expose port and set entrypoint
EXPOSE ${AD_SERVICE_PORT}
ENTRYPOINT [ "./build/install/opentelemetry-demo-ad-service/bin/AdService" ]
