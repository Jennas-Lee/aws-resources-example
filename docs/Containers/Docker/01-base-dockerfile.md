---
title: Base Dockerfile
Description: Base Dockerfile templates.
---

# Base Dockerfile

## Alpine

=== "General"
    ``` Dockerfile linenums="1" hl_lines="3 4 6 7 9 27"
    FROM public.ecr.aws/docker/library/alpine:3.21.3

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN apk add --no-cache tini=~0.19.0 && \
        addgroup -g $GROUP_GID $GROUP_NAME && \
        adduser -D -u $USER_UID -G $GROUP_NAME $USER_NAME && \
        chown -R $USER_NAME:$GROUP_NAME .
    
    # RUN apk add --no-cache curl=~8.2.1 && \
    #     curl --version
    
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/sbin/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/_/alpine)

=== "AMD64"
    ``` Dockerfile linenums="1" hl_lines="3 4 6 7 9 27"
    FROM amd64/alpine:3.21.3

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN apk add --no-cache tini=~0.19.0 && \
        addgroup -g $GROUP_GID $GROUP_NAME && \
        adduser -D -u $USER_UID -G $GROUP_NAME $USER_NAME && \
        chown -R $USER_NAME:$GROUP_NAME .
    
    # RUN apk add --no-cache curl=~8.2.1 && \
    #     curl --version
    
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/sbin/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/amd64/alpine)

=== "ARM64"
    ``` Dockerfile linenums="1" hl_lines="3 4 6 7 9 27"
    FROM arm64v8/alpine:3.21.3

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN apk add --no-cache tini=~0.19.0 && \
        addgroup -g $GROUP_GID $GROUP_NAME && \
        adduser -D -u $USER_UID -G $GROUP_NAME $USER_NAME && \
        chown -R $USER_NAME:$GROUP_NAME .
    
    # RUN apk add --no-cache curl=~8.2.1 && \
    #     curl --version
    
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/sbin/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/arm64v8/alpine)

## Amazon Linux 2023

=== "General"
    ``` Dockerfile linenums="1" hl_lines="5 6 8 9 11 31"
    FROM public.ecr.aws/docker/library/amazonlinux:2023.6.20250218.2-minimal

    ENV TINI_VERSION v0.19.0

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN yum update -y && \
        yum install -y shadow-utils && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        if [ "$(arch)" = "x86_64" ]; then export ARCH="amd64"; else export ARCH="arm64"; fi && \
        curl -o /tini -L https://github.com/krallin/tini/releases/download/"${TINI_VERSION}"/tini-"${ARCH}" && \
        chmod +x /tini && \
        chown $USER_NAME:$GROUP_NAME /tini && \
        chown -R $USER_NAME:$GROUP_NAME .

    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...

    ENTRYPOINT [ "/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/_/amazonlinux)

=== "AMD64"
    ``` Dockerfile linenums="1" hl_lines="5 6 8 9 11 31"
    FROM amd64/amazonlinux:2023.6.20250218.2-minimal

    ENV TINI_VERSION v0.19.0

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN yum update -y && \
        yum install -y shadow-utils && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        if [ "$(arch)" = "x86_64" ]; then export ARCH="amd64"; else export ARCH="arm64"; fi && \
        curl -o /tini -L https://github.com/krallin/tini/releases/download/"${TINI_VERSION}"/tini-"${ARCH}" && \
        chmod +x /tini && \
        chown $USER_NAME:$GROUP_NAME /tini && \
        chown -R $USER_NAME:$GROUP_NAME .

    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...

    ENTRYPOINT [ "/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/amd64/amazonlinux/)

=== "ARM64"
    ``` Dockerfile linenums="1" hl_lines="5 6 8 9 11 31"
    FROM arm64v8/amazonlinux:2023.6.20250218.2-minimal

    ENV TINI_VERSION v0.19.0

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN yum update -y && \
        yum install -y shadow-utils && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        if [ "$(arch)" = "x86_64" ]; then export ARCH="amd64"; else export ARCH="arm64"; fi && \
        curl -o /tini -L https://github.com/krallin/tini/releases/download/"${TINI_VERSION}"/tini-"${ARCH}" && \
        chmod +x /tini && \
        chown $USER_NAME:$GROUP_NAME /tini && \
        chown -R $USER_NAME:$GROUP_NAME .

    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...

    ENTRYPOINT [ "/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/arm64v8/amazonlinux)

## CentOS

=== "General"
    ``` Dockerfile linenums="1" hl_lines="5 6 8 9 11 30"
    FROM public.ecr.aws/docker/library/centos:7.9.2009

    ENV TINI_VERSION v0.19.0

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN yum update -y && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        if [ "$(arch)" = "x86_64" ]; then export ARCH="amd64"; else export ARCH="arm64"; fi && \
        curl -o /tini -L https://github.com/krallin/tini/releases/download/"${TINI_VERSION}"/tini-"${ARCH}" && \
        chmod +x /tini && \
        chown $USER_NAME:$GROUP_NAME /tini && \
        chown -R $USER_NAME:$GROUP_NAME .

    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...

    ENTRYPOINT [ "/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/_/centos)

=== "AMD64"
    ``` Dockerfile linenums="1" hl_lines="5 6 8 9 11 30"
    FROM amd64/centos:7.9.2009

    ENV TINI_VERSION v0.19.0

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-amd64 /tini

    RUN yum update -y && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        chmod +x /tini && \
        chown $USER_NAME:$GROUP_NAME /tini && \
        chown -R $USER_NAME:$GROUP_NAME .
        
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/amd64/centos/)

=== "ARM64"
    ``` Dockerfile linenums="1" hl_lines="5 6 8 9 11 30"
    FROM arm64v8/centos:7.9.2009

    ENV TINI_VERSION v0.19.0

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-arm64 /tini

    RUN yum update -y && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        chmod +x /tini && \
        chown $USER_NAME:$GROUP_NAME /tini && \
        chown -R $USER_NAME:$GROUP_NAME .
        
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/arm64v8/centos/)

## Debian

=== "General"
    ``` Dockerfile linenums="1" hl_lines="3 4 6 7 9 29"
    FROM public.ecr.aws/docker/library/debian:bookworm-20250203-slim

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN apt-get update -y && \
        apt-get install -y --no-install-recommends tini=* && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/* && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        chown -R $USER_NAME:$GROUP_NAME .
    
    # RUN apt-get install -y curl=*
    
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/usr/bin/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/_/debian)

=== "AMD64"
    ``` Dockerfile linenums="1" hl_lines="3 4 6 7 9 29"
    FROM amd64/debian:bookworm-20250203-slim

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN apt-get update -y && \
        apt-get install -y --no-install-recommends tini=* && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/* && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        chown -R $USER_NAME:$GROUP_NAME .
    
    # RUN apt-get install -y curl=*
    
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/usr/bin/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/amd64/debian/)

=== "ARM64"
    ``` Dockerfile linenums="1" hl_lines="3 4 6 7 9 29"
    FROM arm64v8/debian:bookworm-20250203-slim

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN apt-get update -y && \
        apt-get install -y --no-install-recommends tini=* && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/* && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        chown -R $USER_NAME:$GROUP_NAME .
    
    # RUN apt-get install -y curl=*
    
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/usr/bin/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/arm64v8/debian/)

## Ubuntu

=== "General"
    ``` Dockerfile linenums="1" hl_lines="3 4 6 7 9 29"
    FROM public.ecr.aws/docker/library/ubuntu:24.04

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN apt-get update -y && \
        apt-get install -y --no-install-recommends tini=* && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/* && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        chown -R $USER_NAME:$GROUP_NAME .
    
    # RUN apt-get install -y curl=*
    
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/usr/bin/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/_/ubuntu)

=== "AMD64"
    ``` Dockerfile linenums="1" hl_lines="3 4 6 7 9 29"
    FROM amd64/ubuntu:24.04

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN apt-get update -y && \
        apt-get install -y --no-install-recommends tini=* && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/* && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        chown -R $USER_NAME:$GROUP_NAME .
    
    # RUN apt-get install -y curl=*
    
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/usr/bin/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/amd64/ubuntu/)

=== "ARM64"
    ``` Dockerfile linenums="1" hl_lines="3 4 6 7 9 29"
    FROM arm64v8/ubuntu:24.04

    ENV USER_NAME <USER NAME>
    ENV USER_UID <UID NUMBER>

    ENV GROUP_NAME <GROUP NAME>
    ENV GROUP_GID <GID NUMBER>

    ENV WORKDIR <WORKDIR PATH>
    
    WORKDIR $WORKDIR

    RUN apt-get update -y && \
        apt-get install -y --no-install-recommends tini=* && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/* && \
        groupadd -g $GROUP_GID $GROUP_NAME && \
        useradd -l -g $GROUP_NAME -u $USER_UID $USER_NAME && \
        chown -R $USER_NAME:$GROUP_NAME .
    
    # RUN apt-get install -y curl=*
    
    USER $USER_NAME


    # COPY [--chown=<user>:<group>] [--chmod=<permission (ex. 755)>] <src> <dest>   # with Docker Buildkit
    # RUN ...
    
    ENTRYPOINT [ "/usr/bin/tini", "--", "<APPLICATION>" ]
    ```

    [Image Information](https://hub.docker.com/r/arm64v8/ubuntu/)