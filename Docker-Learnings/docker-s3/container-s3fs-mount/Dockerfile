#Multi-Stage Build Image.

################
# Envs Image (Image containing all base environment requirements)
################

FROM python:3.8-slim as envs
LABEL maintainer="Pavan_Gandham"

# Set arguments to be used throughout the image
ARG OPERATOR_HOME="/home/op"
# Bitbucket-pipelines uses docker v18, which doesn't allow variables in COPY with --chown, so it has been statically set where needed.
# If the user is changed here, it also needs to be changed where COPY with --chown appears
ARG OPERATOR_USER="op"
ARG OPERATOR_UID="50000"

# Attach Labels to the image to help identify the image in the future
LABEL docker=true
LABEL docker.distro="debian"
LABEL docker.module="s3fs-test"
LABEL docker.component="docker-s3fs-test"
LABEL docker.uid="${OPERATOR_UID}"

# Set arguments for access s3 bucket to mount using s3fs
ARG BUCKET_NAME
ARG S3_ENDPOINT="https://s3.us-east-1.amazonaws.com"

# Add environment variables based on arguments
ENV OPERATOR_HOME ${OPERATOR_HOME}
ENV OPERATOR_USER ${OPERATOR_USER}
ENV OPERATOR_UID ${OPERATOR_UID}
ENV BUCKET_NAME ${BUCKET_NAME}
ENV S3_ENDPOINT ${S3_ENDPOINT}

# Add user for code to be run as (we don't want to be using root)
RUN useradd -ms /bin/bash -d ${OPERATOR_HOME} --uid ${OPERATOR_UID} ${OPERATOR_USER}

################
# Dist Image
################
FROM envs as dist

# install s3fs
RUN set -ex && \
    apt-get update && \
    apt install s3fs -y

# setup s3fs configs
RUN echo "s3fs#${BUCKET_NAME} ${OPERATOR_HOME}/s3_bucket fuse _netdev,allow_other,nonempty,umask=000,uid=${OPERATOR_UID},gid=${OPERATOR_UID},passwd_file=${OPERATOR_HOME}/.s3fs-creds,use_cache=/tmp,url=${S3_ENDPOINT} 0 0" >> /etc/fstab
RUN sed -i '/user_allow_other/s/^#//g' /etc/fuse.conf

# Set our user to the operator user
USER ${OPERATOR_USER}
WORKDIR ${OPERATOR_HOME}
COPY main.py .

RUN printf '#!/usr/bin/env bash  \n\
echo ${ACCESS_KEY_ID}:${SECRET_ACCESS_KEY} > ${OPERATOR_HOME}/.s3fs-creds \n\
chmod 400 ${OPERATOR_HOME}/.s3fs-creds \n\
mkdir ${OPERATOR_HOME}/s3_bucket \n\
mount -a \n\
exec python ${OPERATOR_HOME}/main.py \
' >> ${OPERATOR_HOME}/entrypoint.sh

RUN chmod 700 ${OPERATOR_HOME}/entrypoint.sh
ENTRYPOINT [ "/home/op/entrypoint.sh" ]