# Setting up a single instance Kafka with basic setup

## Installation setup

### Pre-requisite

1. Create kafka service account
```
groupadd kafka
adduser kafka --system --ingroup kafka
adduser kafka sudo
```

2. Install minimum java requirement of > JRE8
```
apt install openjdk-8-jre-headless
```

### Setup kafka logging directory
1. Create log directory in /var/log
```
mkdir /var/log/kafka
chown -R kafka:kafka /var/log/kafka
```

2. Apply log rotation using logrotate
```
nano /etc/logrotate.d/kafka

## Add the following configuration
Note: to remove the comments.
/var/log/kafka/kafka.log {
    daily               # Rotate the logs weekly
    rotate 7            # Keep 1 week worth of logs
    compress            # Compress (gzip) the log files on rotation
    delaycompress       # Compress the day-old logs
    missingok           # Don't throw errors if the log file is missing
    notifempty          # Don't rotate the log if it's empty
    copytruncate        # Truncate the original log file to zero size in place after creating a copy, instead of moving the old log file and optionally creating a new one
}
```
### Setup kafka data directory
```
mkdir /var/lib/kafka
chown -R kafka:kafka /var/lib/kafka
```

### Installer Download
1. Download kafka 3.6.1 from Apache official site
```
wget https://dlcdn.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz
tar -xzf kafka_2.13-3.6.1.tgz
```
2. Move Kafka installer package to opt
```
mv kafka_2.13-3.6.1 /opt/
chwon -R kafka:kafka kafka_2.13-3.6.1
```

### Installation Steps
1. Replace the following parameter in config/kraft/server.properties
```
advertised.listeners=PLAINTEXT://<ip_or_fwdn_reachable_by_pega>:9092
```
2. Add the following parameters in config/kraft/server.properties required by PEGA
```
auto.create.topics.enable=false
message.max.bytes=5000000
```
3. Generate Cluster UUID 
```
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
```
4. Format kafka log directory
```
sudo -u kafka bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties
```
### Generate systemctl for kafka
1. Create kafka.service
```
sudo nano /etc/systemd/system/kafka.service
```
```
[Unit]
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=kafka
ExecStart=/bin/sh -c '/opt/kafka_2.13-3.6.1/bin/kafka-server-start.sh /opt/kafka_2.13-3.6.1/config/kraft/server.properties > /var/log/kafka/kafka.log 2>&1'
ExecStop=/opt/kafka_2.13-3.6.1/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
2. Test systemctl for kafka
```
systemctl start kafka
```
3. Once no error identified, enable it to start at boot time
```
systemctl enable kafka
```

## Kafka Testing
### Create, read and write topic
1. List topic
```
bin/kafka-topics.sh --list --bootstrap-server <advertised_listener>:9092
```
2. Create topic
```
bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server <advertised_listener>:9092
```
3. Write topic
```
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server <advertised_listener>:9092
>This is my first event
>This is my second event
```
4. Read topic
```
bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server <advertised_listener>:9092
```