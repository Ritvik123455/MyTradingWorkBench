# ---- Stage 1: build the jar ----
FROM eclipse-temurin:17-jdk AS build
WORKDIR /app

# Copy only the wrapper + pom first so dependency downloads are cached
# as their own layer and don't re-run when only source code changes.
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN chmod +x mvnw && ./mvnw dependency:go-offline --batch-mode

# Now copy source and build the jar (tests already ran in CI).
COPY src/ src/
RUN ./mvnw clean package --batch-mode -DskipTests

# ---- Stage 2: the runtime image ----
FROM eclipse-temurin:17-jre AS runtime
WORKDIR /app

# Create and run as a non-root user instead of root.
RUN groupadd --system spring && useradd --system --gid spring spring
USER spring

# Copy ONLY the finished jar out of the build stage.
COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
