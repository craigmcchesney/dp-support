FROM openjdk:21-jdk-slim
WORKDIR /app
COPY ./lib/dp-service.jar /app/dp-service.jar
# COPY src/main/resources/application.yml /config/application.yml
# COPY src/main/resources/log4j2.xml /config/log4j2.xml
EXPOSE 60051
# CMD java -Ddp.config=/config/application.yml -Dlog4j.configurationFile=/config/log4j2.xml -cp /app/dp-service-1.12.0-shaded.jar com.ospreydcs.dp.service.ingest.benchmark.BenchmarkIngestionGrpcServer
# CMD java -cp /app/dp-service-1.12.0-shaded.jar com.ospreydcs.dp.service.ingest.benchmark.BenchmarkIngestionGrpcServer
CMD java -cp /app/dp-service.jar -Ddp.MongoClient.dbHost=10.0.1.60 com.ospreydcs.dp.service.ingest.benchmark.BenchmarkIngestionGrpcServer
