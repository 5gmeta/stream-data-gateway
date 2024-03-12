FROM confluentinc/cp-kafka-connect:7.0.1

USER root
# mkdir -p /opt/ && \
RUN cd /opt/ && \
    wget -c https://github.com/lensesio/stream-reactor/releases/download/3.0.1/stream-reactor-3.0.1-2.5.0.tar.gz && \
    tar xvf stream-reactor-3.0.1-2.5.0.tar.gz && rm stream-reactor-3.0.1-2.5.0.tar.gz 

# using docker exec -it -u 0 CONTAINERID bash 
# and run the command: java -XshowSettings:properties -version
# to  find the JAVA HOME
RUN cd /usr/lib/jvm/zulu11-ca && \
    curl -s -L https://repo1.maven.org/maven2/org/apache/activemq/activemq-client/5.16.3/activemq-client-5.16.3.jar -o activemq-client-5.16.3.jar

RUN cd /usr/lib/ && \
     curl -s -L https://repo1.maven.org/maven2/org/apache/activemq/activemq-client/5.16.3/activemq-client-5.16.3.jar -o activemq-all-5.16.3.jar

RUN cd /usr/share/java && \
    curl -s -L https://repo1.maven.org/maven2/org/apache/activemq/activemq-client/5.16.3/activemq-client-5.16.3.jar -o activemq-client-5.16.3.jar



# RUN cd /usr/lib/ && \
#     wget https://repo1.maven.org/maven2/org/apache/activemq/activemq-all/5.16.3/activemq-all-5.16.3.jar