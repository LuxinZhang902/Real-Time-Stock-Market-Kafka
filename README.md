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

Since Apache Kafka runs on top of jvm[Java Virtual Machine], thus, Java is also needed to be installed o our PC

```python
# Checking java environment
java --version
# Install Java
sudo yum install java # I delete the version for this commend since the version can't find a match
```

## Start Zookeeper
