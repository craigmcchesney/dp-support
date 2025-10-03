FROM openjdk:21-jdk-slim
WORKDIR /app
COPY ./lib/dp-service.jar /app/dp-service.jar
# CMD java -cp /app/dp-service.jar com.ospreydcs.dp.service.ingest.utility.TestDataGenerator
# ENTRYPOINT ["java", "-jar", "dp-service.jar"]
# ENTRYPOINT exec java -cp /app/dp-service.jar "$@"
CMD java -cp /app/dp-service.jar -Ddp.GrpcClient.ingestionConnectString=dp-eco-ingestion:50051 com.ospreydcs.dp.service.ingest.utility.TestDataGenerator
