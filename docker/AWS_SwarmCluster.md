[**Linux**](../Linux.md)

This will drive you through creating a Docker 1.12 Swarm cluster (with Swarm mode) on AWS infrastructure.

## Prerequisites

You need a few things already prepared in order to get started. You need at least Docker 1.12 set up. I was using the stable version of Docker for mac for preparing this guide.
```
$ docker --version
Docker version 1.12.0, build 8eab29e
```
You also need Docker machine installed.
```
$ docker-machine --version
docker-machine version 0.8.0, build b85aac1
```
You need an AWS account. Either you should have you `credentials` file filled:
```
$ cat ~/.aws/credentials
[default]
aws_access_key_id = 
aws_secret_access_key = 
```
Or you need to export these variables before going forward.
```
$ export AWS_ACCESS_KEY_ID=
$ export AWS_SECRET_ACCESS_KEY=
```
Also, you should have AWS CLI installed.
```
$ aws --version
aws-cli/1.10.44 Python/2.7.10 Darwin/15.5.0 botocore/1.4.34
```

## Set up
You should collect the following details from your AWS account.
```
$ VPC=vpc-abcd1234 # the VPC to create your nodes in
$ REGION=eu-west-1 # the region to use
$ SUBNET=subnet-abcd1234 # the subnet to attach your nodes
$ ZONE=b # the zone to use
```

## Steps
Execute these steps one by one. We will create three t2.micro nodes. NOTE: this might cost you some money.

- Create the docker swarm manager node first.
```
$ docker-machine create -d amazonec2 --amazonec2-vpc-id $VPC --amazonec2-region $REGION --amazonec2-zone $ZONE --amazonec2-instance-type t2.micro --amazonec2-subnet-id $SUBNET --amazonec2-security-group demo-swarm demo-swarm-manager
```
- Create the two worker nodes. You can run these commands in parallel with the first one.
```
$ docker-machine create -d amazonec2 --amazonec2-vpc-id $VPC --amazonec2-region $REGION --amazonec2-zone $ZONE --amazonec2-instance-type t2.micro --amazonec2-subnet-id $SUBNET --amazonec2-security-group demo-swarm demo-swarm-node1
$ docker-machine create -d amazonec2 --amazonec2-vpc-id $VPC --amazonec2-region $REGION --amazonec2-zone $ZONE --amazonec2-instance-type t2.micro --amazonec2-subnet-id $SUBNET --amazonec2-security-group demo-swarm demo-swarm-node2
```
- Get the internal IP address of the swarm manager.
```
$ docker-machine ssh demo-swarm-manager ifconfig eth0
```
This should output a bunch of details, but somewhere in the second row you should have the IP address. In my case it is `10.0.0.22`
- Point your docker client to the swarm manager.
```
$ eval $(docker-machine env demo-swarm-manager)
```
- Initialize Swarm mode.
```
$ docker swarm init --advertise-addr 10.0.0.22 # This is the internal IP of manager node.
```
This should output a command which you can use to join on the workers. You will need this in a minute.
- Modify the security group to allow the swarm communication (this is necessary because Docker Machine as of today does not support the new Swarm mode so it doesn't open the right ports)
```
$ aws ec2 describe-security-groups --filter "Name=group-name,Values=demo-swarm"
```
From this command you should get all the details of the security group. Including the GroupId. Copy that information and run the following commands:
```
$ SECURITY_GROUP_ID=sg- #Copy the group id here
$ aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 2377 --source-group $SECURITY_GROUP_ID
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 7946 --source-group $SECURITY_GROUP_ID
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol udp --port 7946 --source-group $SECURITY_GROUP_ID
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 4789 --source-group $SECURITY_GROUP_ID
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol udp --port 4789 --source-group $SECURITY_GROUP_ID
```
- Join the workers to the cluster.
```
$ eval $(docker-machine env demo-swarm-node1)
$ docker swarm join  --token TOKEN 10.0.0.22:2377 # This is the command copied from docker swarm init command's output
$ eval $(docker-machine env demo-swarm-node2)
$ docker swarm join  --token TOKEN 10.0.0.22:2377 # This is the command copied from docker swarm init command's output
```
- Verify the cluster.
```
$ eval $(docker-machine env vizdemo-manager)
$ docker node ls
```

You are done. Enjoy!
