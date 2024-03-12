# Makefile for 5GMETA 

VERSION = 7.0.1

ksql-cli:
	sudo docker exec -it ksqldb-cli ksql http://ksqldb-server:8088

build-connect-image:
	sudo docker build ./kafka-connect/ -t kafka-connect:$(VERSION)
	sudo docker save kafka-connect:$(VERSION) -o ./kafka-connect/kafka-connect:$(VERSION).tar.gz
	sudo docker tag kafka-connect:$(VERSION) 5gmeta/kafka-connect:5gmetav$(VERSION)
	sudo docker tag kafka-connect:$(VERSION) 5gmeta/kafka-connect:latest

upload-connect-registry:
	sudo docker push 5gmeta/kafka-connect:5gmetav$(VERSION)
	sudo docker push 5gmeta/kafka-connect:latest
	


