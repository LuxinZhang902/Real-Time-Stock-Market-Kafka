# Setting up EC2

kafka-stock-project
https://us-east-2.console.aws.amazon.com/ec2/home?region=us-east-2#Home:

1. Instance -> Running
2. Choose Amazon Lunix -> t2.micro[Free tier elgible]
3. Create key pair for EC2 to connecting with my local computer. RSA, .pem -> (keep all other things as default) launch instance
4. After it's status changes to "running", click the instance id and [connect] -> click SSH client -> copy the line under "Example"
5. Enter into the folder whereit contains the key [,pem] file, paste the line to connect to the ec2 instance

```python
# Mac permission error
chmod 400 kafka-stock-project.pem
```

- Command line successful pic
  ![Success Image](/img/aws_success.png)

## Kafka

```python
# Note on changing the version of kafka - Downloading all required files onto the AWS ec2 machine
wget https://downloads.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz
# Checking whether the kafka is here
ls

# unpack it
tar -xvf kafka_2.13-3.6.1.tgz
# Checking it
ls # it should have kafka_2.13-3.6.1  kafka_2.13-3.6.1.tgz shown up
```

Since Apache Kafka runs on top of jvm[Java Virtual Machine], thus, Java is also needed to be installed on our PC

```python
# Checking java environment
java --version
# Install Java
sudo yum install java # I delete the version for this commend since the version can't find a match
```

## Start Zookeeper, kafka Server

```python
cd kafka_2.13-3.6.1
# Start the zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Open other window and start EC2 server. One window is the Zookeeper, and the other is Kafka Server

```

### Start Kafka Server

```python
# increase the memory for Kafka server: It will allocate some of the memory in our Kafka Server
export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"


# Start Kafka Server
cd kafka_2.13-3.6.1

# Start the Kafka Server - THE NAME CHANGED !
bin/kafka-server-start.sh config/server.properties

# Change Private DNS IP to Public IP - You cannot access your private DNS from your local computer unless you are in the same network
# 1. Stop both servers - Ctrl + C on both terminals
# 2. Edit the properties file
sudo nano config/server.properties
# 3. Remove the # before [advertised.listeners=PLAINTEXT://your.host.name:9092], replace your.hose.name with public address
# 4. Ctrl + X and Y and Enter


# Start the Zookeeper Server -> Start the Kafka Server, now the Kafka Server should run on the top of the public IP address
```

### Providing the security access from our local machine

1. In EC2 Instance page, scroll down and find the Security Section
2. Click into the security groups -> scroll down and click "Edit Inbound Rules"
3. Add Rule -> Allow All Traffic -> Anywhere IPV4 [Or MyIP, it will work as well] -> Save Rule
   \*\* DO NOT allow all traffic! it is not the most secure way \*\*

### Topic

```python
cd kafka_2.13-3.6.1/
# 1. Create the Topic
## New Terminal, connect with ec2
# Topic name right after --topic
bin/kafka-topics.sh --create --topic demo_test --bootstrap-server 3.16.162.213:9092 --replication-factor 1 --partitions 1
# My public address (Ignore the waning of naming)


# 2. Start Producer Kafka - Make sure the topic names are same
bin/kafka-console-producer.sh --topic demo_test --bootstrap-server 3.16.162.213:9092
# Start '>' It is my producer

# 3. Start Consumer
## New Terminal, connect with ec2 - Specific process in in the local doc
cd kafka_2.13-3.6.1/
bin/kafka-console-consumer.sh --topic demo_test --bootstrap-server 3.16.162.213:9092
```

![Consumer Image](/img/connect_consumer.png)
Now whatever you type in the Topic terminal shows back into the Consumer Terminal
FINISH THE PRODUCER-KAFKA-CONSUMER part, we can see the data in real time!

# Writing Code on the Python side

- Push the stock market data in real time, and we will consume that data
- Then upload that data onto Amazon S3 do the further operations

1. Build Python file in the folder where key exists

```python
# In the Git terminal
jupyter notebook
# Open new notebook

## On the Producer side
!pip install kafka-python
```

# Restart the whole thing

```python
# KAFKA-ZooKeeper
cd key
ssh -i "kafka-stock-project.pem" xxxxxxxxx[Found in the .md file outside]

cd kafka_2.13-3.6.1
bin/zookeeper-server-start.sh config/zookeeper.properties

# KAFKA-Server
cd key
ssh -i "kafka-stock-project.pem" xxxxxxxxx[Found in the .md file outside]
export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
cd kafka_2.13-3.6.1
bin/kafka-server-start.sh config/server.properties

# KAFKA-Topic
cd key
ssh -i "kafka-stock-project.pem" xxxxxxxxx[Found in the .md file outside]
cd kafka_2.13-3.6.1/

# 1. Create the Topic
## New Terminal, connect with ec2
cd key
ssh -i "kafka-stock-project.pem" xxxxxxxxx[Found in the .md file outside]
cd kafka_2.13-3.6.1/
# Topic name right after --topic
bin/kafka-topics.sh --create --topic demo_test_new --bootstrap-server 3.16.162.213:9092 --replication-factor 1 --partitions 1
# My public address (Ignore the waning of naming)


# 2. Start Producer Kafka - Make sure the topic names are same
bin/kafka-console-producer.sh --topic demo_test_new --bootstrap-server 3.16.162.213:9092
# Start '>' It is my producer

# 3. Start Consumer
## New Terminal, connect with ec2 - Specific process in in the local doc
cd key
ssh -i "kafka-stock-project.pem" xxxxxxxxx[Found in the .md file outside]
cd kafka_2.13-3.6.1/
bin/kafka-console-consumer.sh --topic demo_test_new --bootstrap-server 3.16.162.213:9092
```
