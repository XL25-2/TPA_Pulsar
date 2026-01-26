FROM eclipse-temurin:25-jdk-jammy AS builder

WORKDIR /build

RUN apt-get update && \
    apt-get install -y dos2unix ca-certificates ca-certificates-java && \
    update-ca-certificates && \
    rm -rf /var/lib/apt/lists/*

COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle .

RUN dos2unix gradlew && chmod +x gradlew

RUN ./gradlew dependencies --no-daemon || true

COPY src src

RUN ./gradlew clean bootJar -x test --no-daemon

# runner stage
FROM eclipse-temurin:25-jre-alpine AS runner

WORKDIR /app

RUN apk add --no-cache dumb-init

RUN addgroup -S spring && adduser -S spring -G spring

COPY --from=builder /build/build/libs/*.jar /app/app.jar

RUN chown -R spring:spring /app

USER spring:spring

EXPOSE 8080

ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/./urandom"

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["dumb-init", "--"]

CMD ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
