FROM maven:3.6.1-alpine

WORKDIR /app 

COPY . .

RUN mvn package -DskipTests

RUN mv target/worker-jar-with-dependencies.jar /run/worker.jar

RUN rm -r /app 

#WORKDIR /run


CMD java -jar /run/worker.jar