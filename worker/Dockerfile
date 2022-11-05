FROM maven:3.8.6

WORKDIR /app 

COPY . . 

RUN mvn package -DskipTests && \
    mv target/worker-jar-with-dependencies.jar /run/worker.jar && \
    rm -rf *
    
CMD java -jar /run/worker.jar
