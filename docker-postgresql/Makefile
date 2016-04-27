all: start

IMAGE_NAME := postgres-$(shell whoami)
CONTAINER_NAME := postgres-$(shell whoami)
DOCKER_EXEC := docker exec $(CONTAINER_NAME)

#needs HADOOP_CONFIG_DIR
include ../Makefile.options

#build has some dependencies on external files which we download once
#so it's easy to rebuild a container without download times
packages_dir := build/packages
packages := 
packages := $(packages:%=$(packages_dir)/%)
.PRECIOUS: $(packages)

$(packages_dir):
	mkdir -p $(packages_dir)

#$(packages_dir)/spark-1.6.0-bin-hadoop2.6.tgz: | $(packages_dir)
	#cd $(packages_dir) && wget http://mirror.its.dal.ca/apache/spark/spark-1.6.0/spark-1.6.0-bin-hadoop2.6.tgz

build := build/image
build: $(build)
$(build): Dockerfile $(packages)
	mkdir -p build/
	docker build -t $(IMAGE_NAME) .
	touch $@

cid_file := build/container_id
$(cid_file): $(build) 
	docker create \
		--privileged=true \
		--pid=host \
	       	--name $(IMAGE_NAME)\
		--net=host \
		-v /mnt/scratch/cam14/postgres-storage/_data:/var/lib/postgresql/data \
	       -e POSTGRES_PASSWORD=hellopostgres \
       	       $(CONTAINER_NAME)

	echo $(CONTAINER_NAME) > $@

start: $(cid_file)
	docker start $(CONTAINER_NAME) 
	docker exec $(CONTAINER_NAME) bash -c "ip addr show dev eth0 | grep -oP '\d+\.\d+\.\d+\.\d+'" | head -n1 > build/ip

stop:
	-docker stop $(CONTAINER_NAME)

clean: stop clean_container clean_image
clean_container: stop
	-docker rm -v $(CONTAINER_NAME) && rm $(cid_file)

clean_image:
	-docker rmi $(IMAGE_NAME) && rm $(build)

shell: 
	docker exec -it $(CONTAINER_NAME) bash

psql:
	docker exec -it $(CONTAINER_NAME) psql

#command that runs a test against the hadoop setup - should work
test: start
	docker exec -it $(CONTAINER_NAME) \
	sh -c 'cd /usr/local/spark && \
	./bin/spark-submit --class org.apache.spark.examples.SparkPi \
	    --master yarn \
	    --deploy-mode cluster \
	    --driver-memory 4g \
	    --executor-memory 2g \
	    --executor-cores 1 \
	    lib/spark-examples*.jar \
	    10'