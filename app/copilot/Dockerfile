FROM mcr.microsoft.com/openjdk/jdk:17-ubuntu AS build

WORKDIR /workspace/app
EXPOSE 3100

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
COPY copilot-backend copilot-backend
COPY copilot-common copilot-common
COPY langchain4j-agents langchain4j-agents

RUN chmod +x ./mvnw
# Convert CRLF to LF
RUN sed -i 's/\r$//' ./mvnw
RUN ./mvnw package -DskipTests
RUN mkdir -p copilot-backend/target/dependency && (cd copilot-backend/target/dependency; jar -xf ../*.jar)

#install curl
RUN apt-get update && apt-get install -y curl
RUN curl -LJ -o applicationinsights-agent-3.5.4.jar https://github.com/microsoft/ApplicationInsights-Java/releases/download/3.5.4/applicationinsights-agent-3.5.4.jar
COPY applicationinsights.json .

#for production deployment use mcr.microsoft.com/openjdk/jdk:17-distroless
FROM mcr.microsoft.com/openjdk/jdk:17-distroless

ARG DEPENDENCY=/workspace/app/copilot-backend/target/dependency/
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app
COPY --from=build /workspace/app/applicationinsights-agent-3.5.4.jar /app
COPY --from=build /workspace/app/applicationinsights.json /app
EXPOSE 8080

ENTRYPOINT ["java","-javaagent:/app/applicationinsights-agent-3.5.4.jar","-noverify", "-XX:MaxRAMPercentage=70", "-XX:+UseParallelGC", "-XX:ActiveProcessorCount=2", "-cp","app:app/lib/*","com.microsoft.openai.samples.assistant.CopilotApplication"]