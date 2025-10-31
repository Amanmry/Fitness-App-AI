## Notes for stepwise process involved while building this application

1. Setting up services Required
2. Setting up Database Connections (Posgres and mongodb)
3. Start Building the endpoints

## User Microservice Initilizer

![User Microservice Initilizer](markdown-images/image.png)

## Database Connection 

`microservices/userservice/src/main/resources/application.yml`
```yml
server:
  port: 8082

spring:
  application:
    name: userservice

  datasource:
    url: jdbc:postgresql://localhost:5433/fitness_user_db
    username: postgres
    password: postgres

  jpa:
    hibernate:
      ddl-auto: update
    database-platform: org.hibernate.dialect.PostgreSQLDialect
```

## Activity Microservice Initilizer

![Activity Microservice Initilizer](markdown-images/image-1.png)

## Database Connection 

## Mongo DB Install in MAC

```

Download link : https://fastdl.mongodb.org/osx/mongodb-macos-arm64-8.2.1.tgz

sudo cp /Users/Aman/Downloads/mongodb-macos-aarch64--8.2.1/bin/* /usr/local/bin/

sudo ln -s  /Users/Aman/Downloads/mongodb-macos-aarch64--8.2.1/bin/* /usr/local/bin/

sudo chown Aman ~/data/db

sudo chown Aman ~/data/log/mongodb

nohup mongod --dbpath ~/data/db --logpath ~/data/log/mongodb/mongodb.log >/dev/null &

```

Then can install Mongodb Compass for GUI, there create a DB `fitnessactivity` and create one collection `activities`.

`microservices/activityservice/src/main/resources/application.yml`
```yml
server:
  port: 8082

spring:
  application:
    name: activityservice
  data:
    mongodb:
      uri: mongodb://localhost:27017/fitnessactivity
      database: fitnessactivity
```








