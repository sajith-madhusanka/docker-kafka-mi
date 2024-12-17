# Deploying WSO2 MI 4.2.0 with Kafka and Confluent Schema Registry

This repository contains the Docker Compose artifacts needed to deploy the following services:

- WSO2 Micro Integrator (MI) 4.2.0
- Kafka Server
- ZooKeeper Server
- Confluent Schema Registry Server
- Kafka UI

## Overview

The WSO2 MI 4.2.0 Docker image will be built automatically when deploying the containers. If the image already exists in your local Docker registry, it won't be rebuilt. This repository includes all the necessary additional JAR files to connect with the Kafka server via the WSO2 MI server, and it contains a sample Carbon application with both a message publisher and a consumer.

## Deployment Diagram
![kafka_diagram](https://github.com/user-attachments/assets/75875fbc-6478-47b2-b8ad-661d9454f003)


## Setup Instructions

### 1. Clone the Git Repository

Clone the repository to your local machine:

```bash
git clone https://github.com/sajith-madhusanka/docker-kafka-mi.git
```

## 2. Setting Up the MI Server

### Download the WSO2 MI Product Package:
Download the WSO2 MI product package from [WSO2 Micro Integrator](https://wso2.com/micro-integrator/).

### Update the MI Server:
Update the MI server to the desired update level by following the [WSO2 documentation](https://apim.docs.wso2.com/en/4.2.0/install-and-setup/setup/mi-setup/updating-mi/).

### Prepare the JAR Files:

- Copy the JAR files from `<MI_HOME>/lib` and `<MI_HOME>/dropins` directories to the corresponding directories:
  - `<MI_HOME>/lib` to `<PATH_TO_GIT_REPO_CLONED_DIRECTORY>/docker-kafka-mi/resources/MI/lib`
  - `<MI_HOME>/dropins` to `<PATH_TO_GIT_REPO_CLONED_DIRECTORY>/docker-kafka-mi/resources/MI/dropins`

### Compress the Updated MI Product:
Compress the updated MI product. The compressed file should be named `wso2mi-4.2.0.zip` and placed under the `<PATH_TO_GIT_REPO_CLONED_DIRECTORY>/docker-kafka-mi/wso2MI` directory.

## 3. Deploy the Servers

Navigate to the `docker-compose` directory and run the following command to deploy the servers:

```bash
cd <PATH_TO_GIT_REPO_CLONED_DIRECTORY>/docker-kafka-mi/docker-compose
docker-compose up -d
```
## 4. Viewing Server Logs

To tail the logs of each server, use the following commands:

- **WSO2 MI Server:**
    ```bash
    docker-compose logs -f wso2mi
    ```

- **Kafka Server:**
    ```bash
    docker-compose logs -f kafka
    ```

- **ZooKeeper Server:**
    ```bash
    docker-compose logs -f zookeeper
    ```

- **Confluent Schema Registry:**
    ```bash
    docker-compose logs -f schema-registry
    ```

- **Kafka UI:**
    ```bash
    docker-compose logs -f kafka-ui
    ```

## 5. Stopping the Servers

To stop all servers, use the command:

```bash
docker-compose down
```

## Testing Use Cases

### Using Kafka UI

Use the Kafka UI to create Kafka topics and register schemas under the Confluent Schema Registry. Additionally, you can check consumers and messages on the topics.

- **Kafka UI:** [http://localhost:8080/ui](http://localhost:8080/ui)

### Accessing Kafka Container

To execute Kafka commands directly, access the Kafka container and navigate to the `/opt/kafka` location:

```bash
docker-compose exec kafka bash
```

## Sample Use Case: Testing Message Consumer and Publisher

### Create a Kafka Topic

Create a topic named `sample`.

![image](https://github.com/user-attachments/assets/928a120b-a934-42b1-91f9-adfe75d29847)



### Register a Schema

Register a schema named `sample` with the following content:

```json
{
    "type": "record",
    "name": "logicaltypes",
    "namespace": "sample",
    "doc": "logicaltypes test Schema",
    "fields": [
        {
            "name": "id",
            "type": {
                "type": "string",
                "logicalType": "uuid"
            },
            "default": "99999999-9999-9999-9999-999999999999"
        }
    ]
}
```
You can register this schema via the `Kafka` UI:

![image](https://github.com/user-attachments/assets/d488f98b-eab6-4e70-8de9-95b46846e2be)


You can register this schema via a `curl` command:

```bash
curl -X POST -H "Content-Type: application/json" \
--data '{
    "schema": "{\"type\": \"record\", \
                \"name\": \"logicaltypes\", \
                \"namespace\": \"sample\", \
                \"doc\": \"logicaltypes test Schema\", \
                \"fields\": [ \
                    { \
                        \"name\": \"id\", \
                        \"type\": { \
                            \"type\": \"string\", \
                            \"logicalType\": \"uuid\" \
                        }, \
                        \"default\": \"99999999-9999-9999-9999-999999999999\" \
                    } \
                ] \
              }"
}' http://localhost:8081/subjects/sample/versions
```
## Publish a Message to Kafka

Execute the following `curl` command to publish a message to the Kafka topic through the MI server:

```bash
curl --location --request POST 'http://localhost:8290/sample' \
--header 'kafka_schema: sample' \
--header 'kafka_topic: sample' \
--header 'kafka_partitionNo: 0' \
--header 'Content-Type: application/json' \
--data-raw '{
  "id": "99999999-9999-9999-9999-999999999999"
}'
```

### Deploying Own Carbon Applications

You can test your own use cases by deploying a Carbon application. Place your Carbon application under `<PATH_TO_GIT_REPO_CLONED_DIRECTORY>/docker-kafka-mi/resources/MI/carbonapps`. This can be done while the server is running, and the application will be deployed automatically as the `carbonapps` directory is mounted.

### Troubleshooting

To enable live debugging, set the required `JAVA_OPTS` before running the `docker-compose up -d` command:

```bash
export JAVA_OPTS=-agentlib:jdwp=transport=dt_socket,server=y,address=*:5005
```

Then, in the same terminal, execute `docker-compose up -d` to deploy the servers and start the debug listener using your development IDE.

## Accessing Docker Containers

You can access the required Docker containers by executing:

```bash
docker-compose exec <service_name> bash
```
Eg:
```bash
docker-compose exec kafka bash
```


















