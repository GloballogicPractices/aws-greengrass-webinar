### Docker

Main article: https://docs.aws.amazon.com/greengrass/latest/developerguide/run-gg-in-docker-container.html

The AWS docs article doesn't include sufficient information to run the code in Docker, thus follow the instructions below. Check the AWS docs if any conflicts happen.


#### Prerequisites
* Docker is installed, up and running.
* You have cert and key files in the current folder.

#### Get the AWS Greengrass docker image
Login to Docker using AWS CLI command line generator (use bash command substitution exactly how it's shown below).
```bash
`aws ecr get-login --registry-ids 216483018798 --no-include-email --region us-west-2`
```
Pull the latest Greengrass docker image from the AWS ECR
```bash
docker pull 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest
```

#### Prepare the certificates volume
Create volume folders in the host file system:
```bash
mkdir -p /tmp/greengrass/docker/certs
mkdir -p /tmp/greengrass/docker/log
```

Copy the certificate and the keys to the volume. It's recommended not to store these files in the container. Change the file names in the following script to your names.
```bash
cp 6e08606ffa-certificate.pem.crt /tmp/greengrass/docker/certs
cp 6e08606ffa-private.pem.key /tmp/greengrass/docker/certs
cp 6e08606ffa-public.pem.key /tmp/greengrass/docker/certs
```

Download the root certificate.
```bash
cd /tmp/greengrass/docker/certs/
curl -o root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
```

#### Greengrass Core Configuration

Go to the `docker/` folder of the current repo.
Modify the `config.json` file:

| Field | Meaning |
| ------------- | :------------- |
| `certPath` | Name of the `.pem.crt` file |
| `keyPath `        | Name of the `-private.pem.key` file |
| `thingArn`        | Core thing ARN. Can be found in "AWS IoT ~> Manage ~> Things ~> (choose the thing) ~> Details ~> `arn:...` |
| `iotHost`        | IoT Host URL. Can be found in "AWS IoT ~> Manage ~> Things ~> (choose the thing) ~> Interact ~> HTTPS ~> `....iot.us-west-2.amazonaws.com` |
| `ggHost`        | `greengrass-ats.iot.us-west-2.amazonaws.com` (change to the respective region) |
| `privateKeyPath`        | `file:///greengrass/certs/`+name of the `-private.pem.key` file |
| `certificatePath`        | `file:///greengrass/certs/`+name of the `.pem.crt` file |

This file will be copied into the image in the following steps.

### Docker Image and Container

Instead of using Amazon-provided Docker image, it's proposed to use `Dockerfile` based on it.
In addition to the provided Docker image, other libraries might be installed, e.g. Python libraries.
Add all the necessary dependencies to the `Dockerfile`.

Build the docker image:
```bash
docker build . -t my-greengrass-docker
```

In the AWS documentation, it's proposed to use `docker run --rm` command.
This requires you to re-deploy the Greengrass Group each time you restart the container. To avoid that, use `docker create` and then `docker start`.

Create the docker container:
```bash
docker create --init -it \
  --entrypoint /greengrass-entrypoint.sh \
  -v /tmp/greengrass/docker/certs:/greengrass/certs \
  -v /tmp/greengrass/docker/log:/greengrass/ggc/var/log \
  -p 8883:8883 \
  --name aws-iot-greengrass \
  my-greengrass-docker:latest
```

Now you can run or stop the container.
```bash
docker start -i aws-iot-greengrass
```

The logs are stored in the volume folder of the host file system: `/tmp/greengrass/docker/log`.

If you need to attach to the container, run the following command:
```bash
docker exec -it $(docker ps -a -q -f "name=aws-iot-greengrass") /bin/bash
```
