# Kerberos Vault with Machine learning 

This is an example repository which shows the inference of a machine learning model (YOLOv3) by developing a Kerberos Vault integration. The source code in this repository receives events from Kerberos Vault, through a Kafka integration, and downloads and runs inference on recordings stored in Kerberos Vault using the existing and open source YOLOv3 ML model. This repository contains examples to illustrate how the project works, how it is bundled as a Docker container and how it is deployed inside a Kubernetes cluster [using the NVIDIA operator](https://github.com/kerberos-io/nvidia-gpu-kubernetes).

![NVIDIA operator Kerberos Vault](https://user-images.githubusercontent.com/1546779/132137679-33fc02df-085f-47cf-8587-301bd3448e63.png)

## Kerberos Vault

[Kerberos Vault](https://doc.kerberos.io/vault/first-things-first/) allows you to persist your recordings in the storage providers you want. These storage providers could live in the cloud such as AWS S3, GCP storage and Azure Blob store, or can be located on premise - at the edge - such as Minio. After storing one or more recordings, integrations can be enabled in Kerberos Vault. You can generate real-time messages in Kafka, SQS, or other messages queues to start the execution of custom code, or machine learning models (as explained in the next paragraph).

### Extend with Machine learning

The purpose of the Kerberos Enterprise Suite, and more specifically Kerberos Vault, is to allow businesses to create custom integrations to support their unique business processes by bringing their own code and programming languages.

An example of a business process could be the execution of a machine learning model or any other computer vision related algorithm to perform a specific task. Once properly done, the application could take the appropriate actions and integrations: sending an alert, saving a DB entry, calling an API, etc.

![NVIDIA integration with kafka](https://user-images.githubusercontent.com/1546779/132138400-61f6b2a3-04e2-461f-aee7-18d8bc28a3e8.png)

As shown above, Kerberos Agents will take care of the scaling and stability of your IP cameras; it will make sure it is recording at all times (according to your configurations). Once recorded by a Kerberos Agent, the recording(s) will be stored in a storage provider (S3, Minio, etc) by Kerberos Vault.

Once stored, Kerberos Vault can execute multiple integrations such as Kafka messaging. As soon as a message hits the message broker (e.g. Kafka), the process is totally your. Hence, what you will learn next is a practical example of how that integration with custom code looks like. 

## Hardware: CPUs and/or GPUs

This project can run on both CPU's and/or GPU's. By using the correct `docker/Dockerfile` or `docker/Dockerfile-gpu` you will create a compatible Docker image for your CPU or GPU. As you might expect, the only difference will be execution time; you will experience much higher performance with a GPU over a CPU.

### CPU

To run the project on a CPU you will only need to have Docker installed on your machine. When running the `./run-cpu.sh` command, it will create, prepare and run the Docker image for running on your host machine using your CPU.

### GPU 

The same can be done as described before, but then using a GPU. As you might expect a bit more has to be done to make this work. Before starting, make sure you have the correct NVIDIA drivers installed on your host machine. Once done install `nvidia-docker` on top of your docker installation. More information about the Docker installation can be found at the official [at the official NVIDIA docker documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#setting-up-docker). Likewise, running `./run-gpu.sh` will create another Docker image but compatible for a GPU.

## Deployments

You can run/deploy this project on your local working station, as a Docker container or inside your Kubernetes cluster. This will make it interesting for distributing your machine learning model, and afterwards scale it amongst your GPUs and nodes.

### Docker

When running as a Docker container make sure you have read the previous mentioned "Hardware: CPU's and/or GPU's" section. There is not much else to do rather than running the Docker container and providing the relevant environment variables; read more in The Project.

### Kubernetes

After preparing your Docker containers and confirming it is running properly, it is time to scale it with Kubernetes. The same Docker container can be scheduled into a Kubernetes cluster, and can benefit from your CPU or GPU enabled nodes.

To run on Kubernetes, a helpful tool is the NVIDIA Kubernetes operator. The operator will help you to schedule your GPU workloads (as we will create in a couple of minutes) on the appropriate Kubernetes nodes (that have a GPU attached). Next to that the operator will load-balance and allocate a random NVIDIA GPU that's available to run your workload.

So to conclude: the only thing you need to do in order to scale is to add more GPUs and deploy more workloads. Isn't that nice? :)
 
## The Project

This project contains a working example that integrates with Kerberos Vault through a Kafka integration.  The source code lives in the `detect.py` file, and from a high level establishes a connection with a Kafka broker and Kerberos Vault. Next to that it will also load the required libraries for running the `YOLOv3` model. The following diagram explains what is happening and which systems are targeted.

![yolov3-integration-kerberos-vault](images/yolov3-integration-kerberos-vault-example.png)

### Prerequisites

As shown in the above architecture, this project has a couple of prerequisites.

1. Make sure you have some camera streams,
2. have set up [a Kerberos Enterprise installation](https://doc.kerberos.io/enterprise/installation/) which is processing camera streams,
3. have set up [a Kerberos Vault installation](https://doc.kerberos.io/vault/installation/) which is able to store recordings in a provider,
4. have a Kafka installation, and [connected through an integration](https://doc.kerberos.io/vault/get-started/#queues) in Kerberos Vault
5. and if running on GPUs have the required NVIDIA drivers and operator (when on Kubernetes) installed.

### Environment variables   

Inside the project we are making extensive use of environment variables. Those environment variables will inject the appropriate settings into the source code, and will make sure the project connects to your Kafka broker and Kerberos Vault installation.

Environment variables can be passed into your `docker run` command.

    docker run --name vault-ml \
    -e QUEUE_SYSTEM="KAFKA" \
    -e QUEUE_NAME="ml-queue" \
    -e QUEUE_TARGET="ml-results" \
    -e KAFKA_BROKER="kafka1.kerberos.live:9094,kafka2.kerberos.xxx:9094" \
    -e KAFKA_GROUP="mygroup" \
    -e KAFKA_USERNAME="xxx" \
    -e KAFKA_PASSWORD="xxx" \
    -e KAFKA_MECHANISM="PLAIN" \
    -e KAFKA_SECURITY="SASL_PLAINTEXT" \
    -e VAULT_API_URL="https://staging.api.vault.xxx.live" \
    -e VAULT_ACCESS_KEY="xxx!@#$%12345" \
    -e VAULT_SECRET_KEY="xxx!@#$%67890" \
    -e NUMBER_OF_PREDICTIONS="5" \
    -e FORWARDING_MEDIA="false" \
    -e FORWARDING_METADATA="false" \
    -e FORWARDING_OBJECT_THRESHOLD="1" \
    -e REMOVE_AFTER_PROCESSED="false" \
    vault-ml

Or with the Kerberos Hub pipeline.

    docker run --name hub-ml \
    -e QUEUE_SYSTEM="KAFKA" \
    -e QUEUE_NAME="kcloud-analysis-queue.fifo" \
    -e QUEUE_TARGET="" \
    -e KAFKA_BROKER="kafka1.kerberos.live:9094,kafka2.kerberos.xxx:9094" \
    -e KAFKA_GROUP="mygroup" \
    -e KAFKA_USERNAME="xxx" \
    -e KAFKA_PASSWORD="xxx" \
    -e KAFKA_MECHANISM="PLAIN" \
    -e KAFKA_SECURITY="SASL_PLAINTEXT" \
    -e FORWARDING_MEDIA="false" \
    -e FORWARDING_METADATA="false" \
    -e FORWARDING_OBJECT_THRESHOLD="1" \
    -e REMOVE_AFTER_PROCESSED="false" \
    hub-ml

Or if you want to deploy into Kubernetes you can use the `kubernetes.yaml` file and specify the environment variables inside the environments section.

    kubectl apply -f kubernetes.yaml

The following variables can be used, and should be aligned with your Kafka broker settings and Kerberos Vault account.

|-------------------------------|---------------------------|---------------------------------------------------------------------------------------|
| Name                        | Example                  | Explanation                                                                     |
| --------------------------- | ------------------------ | ------------------------------------------------------------------------------- |
| QUEUE_SYSTEM                | "KAFKA"                  | Which Queue system is used "KAFKA" or "SQS".                                    |
| QUEUE_NAME                  | "consume_topic"          | The topic that will be consumed, the topic assigned in Kerberos Vault           |
| QUEUE_TARGET                | "produce_topic"          | Once a interference is done, the result will be send to this topic.             |
| KAFKA_BROKER                | "kafka.broker:9092"      | The Kafka Broker URL from which the topic is requested.                         |
| KAFKA_GROUP                 | "group"                  | Kafka uses the concept of groups for consuming, align this with Kerberos Vault. |
| KAFKA_USERNAME              | "KUYE7CHLZJF"            | Kafka username for authentication.                                              |
| KAFKA_PASSWORD              | "CCfiCkB2GmLb0A"         | Kafka password for authentication.                                              |
| KAFKA_MECHANISM             | "PLAIN"                  | Kafka mechanism.                                                                |
| KAFKA_SECURITY              | "SASL_SSL"               | Kafka security method.                                                          |
| VAULT_API_URL               | "https://api.vault.live" | The url of the Kerberos Vault API.                                              |
| VAULT_ACCESS_KEY            | "ABCDEFGH"               | Kerberos Vault access key of your account.                                      |
| VAULT_SECRET_KEY            | "JKLMNOPQRST"            | Kerberos Vault secret key of your account.                                      |
| NUMBER_OF_PREDICTIONS       | "5"                      | The number of frames it will process in a single video.                         |
| FORWARDING_MEDIA            | "true"                   | If the object threshold is reach, forward the media to a remote Kerberos Vault. |
| FORWARDING_METADATA         | "true"                   | If the object threshold is reach, forward the metadata to Kerberos Hub.         |
| FORWARDING_OBJECT_THRESHOLD | 3                        | The number of frames it will process in a single video.                         |
| REMOVE_AFTER_PROCESSED      | "false"                  | Remove the recording from Kerberos Vault after it was processed.                |

## What to expect

Once running you should see some logs printed in your container or console. When running a Docker container you could run `docker logs [container-name]` or on kubernetes `kubectl logs [pod-name]`. Both commands will show you intermediary results and give you a glimpse of what is happening.

At the start you should see the following.

    layer     filters    size              input                output
    0 conv     32  3 x 3 / 1   608 x 608 x   3   ->   608 x 608 x  32  0.639 BFLOPs
    1 conv     64  3 x 3 / 2   608 x 608 x  32   ->   304 x 304 x  64  3.407 BFLOPs
    2 conv     32  1 x 1 / 1   304 x 304 x  64   ->   304 x 304 x  32  0.379 BFLOPs
    3 conv     64  3 x 3 / 1   304 x 304 x  32   ->   304 x 304 x  64  3.407 BFLOPs
    4 res    1                 304 x 304 x  64   ->   304 x 304 x  64
    5 conv    128  3 x 3 / 2   304 x 304 x  64   ->   152 x 152 x 128  3.407 BFLOPs
    6 conv     64  1 x 1 / 1   152 x 152 x 128   ->   152 x 152 x  64  0.379 BFLOPs
    7 conv    128  3 x 3 / 1   152 x 152 x  64   ->   152 x 152 x 128  3.407 BFLOPs
    8 res    5                 152 x 152 x 128   ->   152 x 152 x 128
    9 conv     64  1 x 1 / 1   152 x 152 x 128   ->   152 x 152 x  64  0.379 BFLOPs
    10 conv    128  3 x 3 / 1   152 x 152 x  64   ->   152 x 152 x 128  3.407 BFLOPs
    11 res    8                 152 x 152 x 128   ->   152 x 152 x 128

After running for some time you should see messages being consumed from your Kafka broker, and some results of the inferences.
    
    checking..
    {'provider': 'kstorage', 'source': 'storj', 'request': 'persist', 'payload': {'is_fragmented': False, 'metadata': {'productid': 'Bfuk14xm40eMSxwEEyrd908yzmDIwKp5', 'event-timestamp': '1630567591', 'publickey': 'ABCDEFGHI!@#$%12345', 'event-instancename': 'highway4', 'capture': 'IPCamera', 'event-microseconds': '0', 'uploadtime': '1630567591', 'event-regioncoordinates': '200-200-400-400', 'event-token': '0', 'event-numberofchanges': '24'}, 'key': 'youruser/1630567591_6-967003_highway4_200-200-400-400_24_769.mp4', 'bytes_range_on_time': None, 'fileSize': 6189104, 'bytes_ranges': ''}, 'events': ['monitor', 'sequence', 'analysis', 'throttler', 'notification'], 'date': 1630567591}
    {"provider": "kstorage", "data": {"probabilities": [[0.930047333240509, 0.8236896991729736]], "labels": [["car", "car"]], "boxes": [[[542, 308, 595, 336], [568, 287, 605, 304]]]}, "source": "storj", "request": "persist", "payload": {"is_fragmented": false, "metadata": {"productid": "Bfuk14xm40eMSxwEEyrd908yzmDIwKp5", "event-timestamp": "1630567591", "publickey": "ABCDEFGHI!@#$%12345", "event-instancename": "highway4", "capture": "IPCamera", "event-microseconds": "0", "uploadtime": "1630567591", "event-regioncoordinates": "200-200-400-400", "event-token": "0", "event-numberofchanges": "24"}, "key": "youruser/1630567591_6-967003_highway4_200-200-400-400_24_769.mp4", "bytes_range_on_time": null, "fileSize": 6189104, "bytes_ranges": ""}, "events": ["monitor", "sequence", "analysis", "throttler", "notification"], "date": 1630567591, "operation": "classification"}
    Sending to footages
    next..
    checking..
    {'provider': 'kstorage', 'source': 'storj', 'request': 'persist', 'payload': {'is_fragmented': False, 'metadata': {'productid': 'Bfuk14xm40eMSxwEEyrd908yzmDIwKp5', 'event-timestamp': '1630567626', 'publickey': 'ABCDEFGHI!@#$%12345', 'event-instancename': 'highway4', 'capture': 'IPCamera', 'event-microseconds': '0', 'uploadtime': '1630567626', 'event-regioncoordinates': '200-200-400-400', 'event-token': '0', 'event-numberofchanges': '24'}, 'key': 'youruser/1630567626_6-967003_highway4_200-200-400-400_24_769.mp4', 'bytes_range_on_time': None, 'fileSize': 6389670, 'bytes_ranges': ''}, 'events': ['monitor', 'sequence', 'analysis', 'throttler', 'notification'], 'date': 1630567626}
    {"provider": "kstorage", "data": {"probabilities": [[0.9682833552360535, 0.9092203974723816, 0.5468082427978516]], "labels": [["car", "car", "traffic light"]], "boxes": [[[488, 143, 532, 171], [249, 68, 272, 83], [353, 122, 363, 145]]]}, "source": "storj", "request": "persist", "payload": {"is_fragmented": false, "metadata": {"productid": "Bfuk14xm40eMSxwEEyrd908yzmDIwKp5", "event-timestamp": "1630567626", "publickey": "ABCDEFGHI!@#$%12345", "event-instancename": "highway4", "capture": "IPCamera", "event-microseconds": "0", "uploadtime": "1630567626", "event-regioncoordinates": "200-200-400-400", "event-token": "0", "event-numberofchanges": "24"}, "key": "youruser/1630567626_6-967003_highway4_200-200-400-400_24_769.mp4", "bytes_range_on_time": null, "fileSize": 6389670, "bytes_ranges": ""}, "events": ["monitor", "sequence", "analysis", "throttler", "notification"], "date": 1630567626, "operation": "classification"}
