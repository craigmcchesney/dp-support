FROM openjdk:21-jdk-slim
WORKDIR /app
COPY ./lib/dp-service.jar /app/dp-service.jar
CMD java -cp /app/dp-service.jar -Ddp.MongoClient.dbHost=10.0.1.60 com.ospreydcs.dp.service.query.server.QueryGrpcServer
