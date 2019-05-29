FROM 216483018798.dkr.ecr.us-west-2.amazonaws.com/aws-iot-greengrass:latest

RUN curl -O https://bootstrap.pypa.io/get-pip.py
RUN python get-pip.py
RUN pip install greengrasssdk

COPY config.json /greengrass/config

ENTRYPOINT /greengrass-entrypoint.sh