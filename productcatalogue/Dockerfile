# =========================
# Stage 1: Build application
# =========================
FROM maven:3.8.8-eclipse-temurin-8 AS builder

WORKDIR /app

COPY pom.xml .

RUN mvn dependency:go-offline

COPY src ./src

RUN mvn clean package -DskipTests


# =========================
# Stage 2: Runtime
# =========================
FROM eclipse-temurin:8-jre

WORKDIR /app

COPY --from=builder /app/target/productcatalogue-0.0.1-SNAPSHOT.jar app.jar

COPY product-catalogue.yml app-config.yml

EXPOSE 8020

ENTRYPOINT ["java","-jar","app.jar","server","app-config.yml"]