# spark-kafka-airflow-etl-pipelines



## Building the Data Pipeline: Step-by-Step
1. Set Up Kafka Cluster
Start your Kafka cluster with the following commands:

```
docker network create docker_streaming
docker-compose -f docker-compose.yml up -d
```
2. Create the topic for Kafka (`http://localhost:8888/`)
Access the Kafka UI at `http://localhost:8888/`.
Observe the active cluster.
Navigate to 'Topics'.
Create a new topic named "names_topic".
Set the replication factor to 3.
3. Configure Airflow User
Create an Airflow user with admin privileges:

`docker-compose run airflow_webserver airflow users create --role Admin --username admin --email admin --firstname admin --lastname admin --password admin`
4. Access Airflow Bash & Install Dependencies
Use the provided script to access the Airflow bash and install required packages:

`docker-compose exec airflow_webserver /bin/bash`
`pip install -r ./requirements.txt`
5. Validate DAGs
Ensure there are no errors with your DAGs:

`airflow dags list`
6. Start Airflow Scheduler
To initiate the DAG, run the scheduler:

`airflow scheduler`
6. Verify the data is uploaded to Kafka Cluster
Access the Kafka UI at http://localhost:8888/ and verify that the data is uploaded for the topic
8. Transfer Spark Script
Copy your Spark script into the Docker container:

`docker cp spark_processing.py spark_master:/opt/bitnami/spark/`
9. Initiate Spark Master & Download JARs
Access Spark bash, navigate to the jars directory, and download essential JAR files. After downloading, submit the Spark job:


```
docker exec -it spark_master /bin/bash
cd jars


curl -O https://repo1.maven.org/maven2/org/apache/kafka/kafka-clients/3.5.0/kafka-clients-3.5.0.jar
curl -O https://repo1.maven.org/maven2/org/apache/spark/spark-sql-kafka-0-10_2.13/3.5.0/spark-sql-kafka-0-10_2.13-3.5.0.jar
curl -O https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.3.6/hadoop-aws-3.3.6.jar
curl -O https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-s3/1.9.9/aws-java-sdk-s3-1.9.9.jar
curl -O https://repo1.maven.org/maven2/org/apache/commons/commons-pool2/2.9.0/commons-pool2-2.9.0.jar



cd ..


spark-submit \
--master local[2] \
--jars /opt/bitnami/spark/jars/kafka-clients-3.5.0.jar,\
/opt/bitnami/spark/jars/spark-sql-kafka-0-10_2.13-3.5.0.jar,\
/opt/bitnami/spark/jars/hadoop-aws-3.3.6.jar,\
/opt/bitnami/spark/jars/aws-java-sdk-s3-1.9.9.jar,\
/opt/bitnami/spark/jars/commons-pool2-2.9.0.jar \
spark_processing.py

spark-submit  --master local[2] --jars /opt/bitnami/spark/jars/kafka-clients-3.5.0.jar,/opt/bitnami/spark/jars/spark-sql-kafka-0-10_2.13-3.5.0.jar,/opt/bitnami/spark/jars/hadoop-aws-3.3.6.jar,/opt/bitnami/spark/jars/aws-java-sdk-s3-1.9.9.jar,/opt/bitnami/spark/jars/commons-pool2-2.9.0.jar spark_processing.py

```
10. Verify Data on S3
After executing the steps, check your S3 bucket to ensure data has been uploaded

Challenges and Troubleshooting
Configuration Challenges: Ensuring environment variables and configurations (like in the docker-compose.yaml file) are correctly set can be tricky. An incorrect setting might prevent services from starting or communicating.
Service Dependencies: Services like Kafka or Airflow have dependencies on other services (e.g., Zookeeper for Kafka). Ensuring the correct order of service initialization is crucial.
Airflow DAG Errors: Syntax or logical errors in the DAG file (kafka_stream_dag.py) can prevent Airflow from recognizing or executing the DAG correctly.
Data Transformation Issues: The data transformation logic in the Python script might not always produce expected results, especially when handling various data inputs from the Random Name API.
Spark Dependencies: Ensuring all required JARs are available and compatible is essential for Spark's streaming job. Missing or incompatible JARs can lead to job failures.
Kafka Topic Management: Creating topics with the correct configuration (like replication factor) is essential for data durability and fault tolerance.
Networking Challenges: Docker networking, as set up in the docker-compose.yaml, must correctly facilitate communication between services, especially for Kafka brokers and Zookeeper.
S3 Bucket Permissions: Ensuring correct permissions when writing to S3 is crucial. Misconfigured permissions can prevent Spark from saving data to the bucket.
Deprecation Warnings: The provided logs show deprecation warnings, indicating that some methods or configurations used might become obsolete in future versions.



