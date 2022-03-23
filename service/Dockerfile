FROM openjdk

LABEL SERVICE=$SERVICE_NAME

ARG UNAME=user
ARG UID=1001
ARG GID=1001

ENV JAVA_OPTS ""

RUN groupadd -g $GID -o $UNAME && \
    useradd -m -u $UID -g $GID -o -s /bin/bash $UNAME  && \
    mkdir -p /home/application/config

RUN chown -R $UNAME:$UNAME /home/application
USER $UNAME

WORKDIR /home/application/

# To utilize the concept of caching, placing entrypoint before Jar Name as entrypoint will not change 
ENTRYPOINT ["/bin/bash", "-c", "java $JAVA_OPTS -jar $JAR_NAME"]

ARG JAR_NAME
ENV JAR_NAME $JAR_NAME

ARG JAR_PATH
ADD $JAR_PATH /home/application/