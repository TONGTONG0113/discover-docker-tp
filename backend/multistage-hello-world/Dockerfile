# Stage 1: Compile the Java application
FROM eclipse-temurin:21-jdk-alpine AS build

WORKDIR /usr/src
COPY Main.java .
RUN javac Main.java

# Stage 2: Run the application
FROM eclipse-temurin:21-jre-alpine

WORKDIR /usr/src
COPY --from=build /usr/src/Main.class .

CMD ["java", "Main"]
