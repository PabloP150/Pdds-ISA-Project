# Stage 1: Build the application
FROM gradle:jdk17 AS build
WORKDIR /home/gradle/project
COPY . .
RUN gradle build --no-daemon

# Stage 2: Create the final image
FROM eclipse-temurin:17-jre-focal
WORKDIR /app
COPY --from=build /home/gradle/project/build/libs/*.jar /app/petclinic.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "petclinic.jar"]
