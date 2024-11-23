.PHONY: check-docker install-docker network pull-spark start-master start-worker open-ui pyspark spark-shell stop clean

# Create Docker network
network:
	docker network create spark-network || true

# Check if Docker is installed
check-docker:
	@if ! command -v docker >/dev/null 2>&1; then \
		echo "Docker is not installed. Installing Docker..."; \
		make install-docker; \
	else \
		echo "Docker is already installed."; \
	fi

# Install Docker on Mac using Homebrew
install-docker:
	@if ! command -v brew >/dev/null 2>&1; then \
		/bin/bash -c "$$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"; \
	fi
	brew install --cask docker
	@echo "Please open Docker Desktop to complete the installation"

# Pull Spark Docker image
pull-spark:
	docker pull bitnami/spark:latest

# Start Spark master node
start-master:
	docker run -d \
		--name spark-master \
		--hostname spark-master \
		--network spark-network \
		-p 8090:8080 \
		-p 7077:7077 \
		-e SPARK_MODE=master \
		-e SPARK_RPC_AUTHENTICATION_ENABLED=no \
		-e SPARK_RPC_ENCRYPTION_ENABLED=no \
		-e SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no \
		-e SPARK_SSL_ENABLED=no \
		bitnami/spark:latest
	@echo "Waiting for Spark master to be ready..."
	@sleep 20
	@if docker ps -q -f name=spark-master >/dev/null; then \
		echo "Spark master is running."; \
	else \
		echo "Spark master failed to start. Check logs:"; \
		docker logs spark-master; \
		exit 1; \
	fi

# Start Spark worker node
start-worker:
	docker run -d \
		--name spark-worker \
		--hostname spark-worker \
		--network spark-network \
		-e SPARK_MODE=worker \
		-e SPARK_MASTER_URL=spark://spark-master:7077 \
		-e SPARK_WORKER_MEMORY=1G \
		-e SPARK_WORKER_CORES=1 \
		-e SPARK_RPC_AUTHENTICATION_ENABLED=no \
		-e SPARK_RPC_ENCRYPTION_ENABLED=no \
		-e SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no \
		-e SPARK_SSL_ENABLED=no \
		bitnami/spark:latest

# Open Spark UI in default browser
open-ui:
	open http://localhost:8090

# Start PySpark shell
pyspark:
	docker exec -it spark-master /opt/bitnami/spark/bin/pyspark

# Start Scala Spark shell
spark-shell:
	docker exec -it spark-master /opt/bitnami/spark/bin/spark-shell

# Stop all Spark containers
stop:
	docker stop spark-master spark-worker || true

# Remove all Spark containers and network
clean: stop
	docker rm spark-master spark-worker || true
	docker network rm spark-network || true

# Setup everything
setup: check-docker network pull-spark
	@make start-master
	@echo "Starting worker..."
	@make start-worker
	@echo "Waiting for cluster to stabilize..."
	@sleep 20
	@echo "Checking Spark UI availability..."
	@for i in 1 2 3 4 5; do \
		if curl -s http://localhost:8090 >/dev/null; then \
			echo "Spark UI is ready!"; \
			break; \
		fi; \
		if [ $$i -eq 5 ]; then \
			echo "Warning: Spark UI might not be ready. Check logs:"; \
			docker logs spark-master; \
		fi; \
		echo "Waiting for Spark UI to become available... (attempt $$i/5)"; \
		sleep 5; \
	done
	@echo "Opening Spark UI..."
	@make open-ui
