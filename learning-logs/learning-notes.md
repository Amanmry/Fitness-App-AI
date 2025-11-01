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

---

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

---

## Adding Interservice Communication, Eureka Server

## Setting UP Eureka Server

create a spring-boot project for centralize server

![alt text](markdown-images/image-2.png)

add the configurations needed for server

application.yml
```yml
spring:
  application:
    name: eureka

server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

In the `EurekaApplication.java` also add `@EnableEurekaServer` to make this application as Eureka Server.

Now, run the application

![alt text](markdown-images/image-3.png)

so, now the Eureka Server is up at `http://localhost/8761`

Note: For a standalone Eureka server that is not intended to register itself or discover other Eureka servers, register-with-eureka and fetch-registry is typically set to false.

---

## Now let's register microservices to the Eureka Server 

Step 1 : Add the eureka client dependencies to the microservice.

Step 2 : Add application configuration in Client 

eg:
```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

this will tell the client to register itself at which endpoint.

now, as we can see `User-Service` and `Activity-Service` is registered with Eureka Server.

![alt text](markdown-images/image-4.png)

---

so, far both the microservice is working as a individual service and there's no validation of checking whether there's any user with the passed user id in request to Activity microservice.

now, we have to add that check, so we will be need to make a `Interservice Communication` between `user-service` and `activity-service`, so that whenever a request comes to user-service with a userId field we can check that into user-service whether there exist any user with incoming userId.

we, will be using `Web Client` for the interservice communication.

so, we need to add this dependency in activity-service pom.xml :

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

## Similar to Activity Service we need to setup AI Service

![AI Microservice Initilizer](markdown-images/image-5.png)

`microservices/aiservice/src/main/resources/application.yaml`
```yml
spring:
  application:
    name: ai-service
  data:
    mongodb:
      uri: mongodb://localhost:27017/fitnessrecommendation
      database: fitnessrecommendation

server:
  port: 8083

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

## Event Driven Architecture for (Recommendation - Generation)

so, to do that we can make use of Rabbit MQ, in spring boot project we can use `Spring AMQP`.

## Setting up Rabbit MQ Locally

[Docs for Installation](https://www.rabbitmq.com/docs/download)

Using Docker 

```sh
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4-management
```

Now, we can access the service on web console on [Rabbit MQ Web-Console](http://www.localhost:15672/)

username : guest
password : guest

## Let's Integrate Spring-AMQP

Activity-Service: dependency in pom.xml
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

Activity Service : application.yml
```yml
server:
  port: 8082

spring:
  application:
    name: activity-service
  data:
    mongodb:
      uri: mongodb://localhost:27017/fitnessactivity
      database: fitnessactivity
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

rabbitmq:
  exchange:
    name: fitness.exchange
  queue:
    name: activity.queue
  routing:
    key: activity.tracking
```

## AI Integration

Endpoint in Activity Service: `http://localhost:8082/api/activities`
Request Body : 

```json
{
    "userId": "495b0334-4c50-42e1-b36f-476695ceff0a",
    "type": "RUNNING",
    "duration": 30,
    "caloriesBurned": 300,
    "startTime": "2025-10-31T10:00:00",
    "additionalMetrics": {
        "distance": 5.2,
        "averageSpeed": 10.4,
        "maxHeartRate": 165
    }
}
```


Gemini response for this particular activity:
```json
{
  "analysis": {
    "overall": "This was a very solid 30-minute running session, indicating good cardiovascular fitness and a strong performance. Covering 5.2 km at an average speed of 10.4 km/h is an excellent effort for this duration, pushing into a vigorous intensity zone, which is well-supported by your peak heart rate. You're maintaining a good level of fitness.",
    "pace": "An average pace of approximately 5 minutes 46 seconds per kilometer (10.4 km/h) for a 30-minute run is impressive. This pace suggests a challenging but sustainable effort, well above a conversational pace, contributing significantly to cardiovascular conditioning and demonstrating good running efficiency.",
    "heartRate": "A maximum heart rate of 165 bpm during this run indicates a high level of cardiovascular effort. Depending on your age (which is not provided), this is likely within or approaching your vigorous exercise zone (e.g., 80-90% of maximum heart rate), which is highly effective for improving aerobic capacity and heart health. It shows great engagement of your cardiovascular system.",
    "CaloriesBurned": "Burning 300 calories in 30 minutes aligns perfectly with the intensity and duration of your run (approximately 10 calories per minute). This is an efficient and effective calorie expenditure for a 30-minute session at this pace and intensity, contributing positively to energy balance and fitness goals."
  },
  "improvements": [
    {
      "area": "Endurance & Stamina",
      "recommendation": "To further build endurance, consider gradually increasing your run duration by 5-10 minutes once or twice a week, maintaining a slightly more conversational pace. This will help improve your body's ability to sustain effort over longer periods without excessive fatigue, enhancing your aerobic base."
    },
    {
      "area": "Speed & Pace",
      "recommendation": "Introduce one speed-focused workout per week. This could involve interval training (e.g., alternating bursts of faster running for 1-2 minutes with 2-3 minutes of slower jogging for recovery) or tempo runs (sustaining a faster-than-usual but still comfortably hard pace for 15-20 minutes within your run). This will improve your top-end speed and anaerobic threshold."
    },
    {
      "area": "Cardiovascular Efficiency",
      "recommendation": "While your max HR indicates good effort, explore varying your intensity. Include some runs where you consciously aim for a lower, steady heart rate zone (e.g., 70-80% MHR) for longer durations, and others where you push into higher zones, as suggested with speed work. This diverse training will enhance overall heart health and metabolic flexibility."
    },
    {
      "area": "Running Form & Efficiency",
      "recommendation": "Focus on maintaining good running form: keep your gaze forward, shoulders relaxed and down, arms at a 90-degree angle, and feet landing softly beneath your hips (midfoot strike). Incorporating dynamic warm-ups and post-run stretching can also improve flexibility, mobility, and reduce injury risk, leading to more efficient running."
    }
  ],
  "suggestions": [
    {
      "workout": "Interval Training (Fartlek)",
      "description": "Warm-up with a 5-10 minute easy jog. Then, for the next 20 minutes, alternate between 1-2 minutes of fast running (pushing your pace significantly, aiming for 85-90% effort) and 2-3 minutes of easy jogging or walking to recover. Repeat this cycle 5-6 times. Finish with a 5-minute cool-down jog and thorough stretching. This workout will boost your speed, anaerobic threshold, and cardiovascular fitness."
    },
    {
      "workout": "Progressive Long Run",
      "description": "Warm-up with 5 minutes of walking/easy jogging. Then, embark on a 40-45 minute run where you start at a comfortable, conversational pace for the first 15-20 minutes. Gradually increase your speed every 10 minutes until the final 5-10 minutes are at a moderately challenging pace (similar to your analyzed run or slightly faster). Cool down with 5 minutes of walking and stretching. This builds endurance and teaches your body to handle increasing fatigue."
    },
    {
      "workout": "Strength & Core for Runners",
      "description": "On a non-running day, perform 2-3 sets of 10-15 repetitions for exercises like squats, lunges, glute bridges, planks (holding for 30-60 seconds), and calf raises. Focus on proper form and controlled movements. Strengthening your core, glutes, and leg muscles is crucial for injury prevention, improving running economy, and boosting power."
    }
  ],
  "safety": [
    "Always perform a dynamic warm-up (e.g., leg swings, high knees, butt kicks) for 5-10 minutes before your run and a static cool-down stretch (holding stretches for 20-30 seconds) for 5-10 minutes afterward to prevent injury and improve flexibility.",
    "Listen to your body. If you experience sharp or persistent pain, discomfort beyond typical muscle fatigue, or excessive dizziness, reduce your intensity or stop the activity. Pushing through pain can lead to more serious injuries.",
    "Stay adequately hydrated by drinking water before, during (if necessary for longer runs or hot weather), and after your run. Proper hydration is crucial for performance and recovery.",
    "Wear appropriate running shoes that offer good support and cushioning, and replace them every 500-800 km or 6-9 months, as worn-out shoes can contribute to injuries.",
    "Run in well-lit, safe environments, and if running outdoors, be aware of your surroundings, traffic, and pedestrians. Consider wearing reflective gear and carrying identification if running in low light or remote areas.",
    "Ensure a gradual progression in your training. Avoid drastically increasing your distance, speed, or intensity by more than 10% per week to give your body adequate time to adapt and minimize the risk of overuse injuries."
  ]
}
```

Now, we need to parse the response generated by Gemini and store it into the Database. 