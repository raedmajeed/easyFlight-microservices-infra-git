version: "3"

networks:
app-network:
driver: bridge

services:
zookeeper:
image: confluentinc/cp-zookeeper:latest
container_name: zookeeper
environment:
ZOOKEEPER_SERVER_ID: 1
ZOOKEEPER_CLIENT_PORT: 2181
ZOOKEEPER_TICK_TIME: 2000
ports:
- "22181:2181"
networks:
- app-network

broker:
image: confluentinc/cp-kafka:latest
container_name: broker
ports:
- "9092:9092"
depends_on:
- zookeeper
environment:
KAFKA_BROKER_ID: 1
KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT
KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:9092,PLAINTEXT_HOST://localhost:29092
KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
networks:
- app-network

api-service:
image: raedmajeed/api-service:3
container_name: api-service
depends_on:
- airline-service
- booking-service
- notification-service
- frontend-service
ports:
- "8083:8080"
restart: always
networks:
- app-network

airline-service:
image: raedmajeed/airline-service:12
container_name: airline-service
ports:
- "6061:6061"
depends_on:
- mysql
- redis-service
restart: always
environment:
ADMINPORT: 6060
ADMINBOOKINGPORT: 9095
KAFKABROKER: broker:9092
DBPORT: 3306
DBNAME: flight_booking_airline
DBUSER: admin
DBPASSWORD: 12345678
DBHOST: localhost
REDISHOST: redis-service:6379
SECRETKEY: "secret-key"
BUSINESSSURGE: 1.9
networks:
- app-network

booking-service:
image: raedmajeed/booking-service:12
container_name: booking-service
ports:
- "9091:9091"
restart: always
environment:
PORT: 9090
BSERVICEPORT: 9091
ADMINBOOKINGPORT: "airline-service:9095"
DBPORT: 3306
DBNAME: flight_booking_booking
DBUSER: admin
DBPASSWORD: 12345678
DBHOST: localhost
REDISHOST: redis-service:6379
SECRETKEY: "secret-key"
SERVICETOKEN: VA2b78db376a44009f23efc9b5f786f629
SID: ACeed503561915995469a10fd7a8d54c47
TOKEN: 649aac0165591a99b792b8e445d24fc4
RAZORPAYKEYID: rzp_test_3Zm0sCFZcVmktU
RAZORPAYSECRETKEY: WrjpGGCVJ24oYzKfj12tHftK
depends_on:
- mysql
- redis-service
networks:
- app-network

frontend-service:
image: raedmajeed/frontend-service:3
container_name: frontend-service
ports:
- "3030:3030"
restart: always
networks:
- app-network

notification-service:
image: raedmajeed/notification-service:5
container_name: notification-service
environment:
EMAIL: raedam786@gmail.com
PASSWORD: gtuemqhhzagxvjrh
KAFKABROKER: broker:9092
PORT: 8082
networks:
- app-network

mysql:
image: mysql:8
container_name: mysql
environment:
MYSQL_ROOT_PASSWORD: 12345
MYSQL_PASSWORD: 1234
MYSQL_USER: raed
MYSQL_DATABASE: "flight_booking_booking"
ports:
- "3306:3306"
volumes:
- easyflight-volume:/var/lib/mysql
networks:
- app-network

redis-service:
image: redis
container_name: redis-service
ports:
- "6379:6379"
networks:
- app-network

volumes:
easyflight-volume:
