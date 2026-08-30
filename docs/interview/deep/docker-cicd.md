# Docker и CI/CD — подробный конспект

## Image, layers, container

Image manifest ссылается на immutable content-addressed layers. Container добавляет writable layer и runtime configuration/namespaces/cgroups. Удаление container удаляет writable state, volume живёт отдельно.

Docker build cache инвалидируется с изменившейся instruction/inputs и всеми следующими layers. Копируйте dependency descriptors до source, используйте multi-stage и `.dockerignore`.

## Dockerfile

```dockerfile
FROM eclipse-temurin:21-jdk AS build
WORKDIR /src
COPY gradlew settings.gradle build.gradle ./
COPY gradle gradle
RUN ./gradlew dependencies --no-daemon
COPY src src
RUN ./gradlew bootJar --no-daemon

FROM eclipse-temurin:21-jre
RUN useradd --system app
USER app
COPY --from=build /src/build/libs/app.jar /app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

Pin/update base image policy, non-root, no secrets in layers, exec-form entrypoint для signals. Image scanning не заменяет patch process.

## Networking/volumes

Port publishing host→container. Внутри network services обращаются по DNS name/container port. `localhost` — текущий container.

Named volume управляется Docker, bind mount связывает host path. Database durability требует backup/restore, а не только volume.

## Pipeline

Pipeline быстрый fail-first: format/static/unit → integration/contract → immutable image → SBOM/scan/sign → deploy → verify. Build once, promote digest между средами.

Secrets выдаются runtime/short-lived credentials, маскируются logs. Cache CI не должен подменять reproducibility.

## Deploy strategies

Rolling экономичен, но версии сосуществуют. Blue-green быстро переключает/rollback, требует двойной capacity. Canary сравнивает SLI новой версии на доле traffic.

DB migrations expand-contract: добавить совместимое → deploy code → backfill → переключить reads → удалить старое позже. Rollback app не откатывает уже разрушительную schema.

## Критерий готовности

Вы готовы, если оптимизируете layers, объясняете network/volume, строите build-once pipeline и проектируете canary/rollback с совместимой миграцией.

