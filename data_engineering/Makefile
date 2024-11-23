.PHONY: check-docker install-docker network pull-spark start-master start-worker open-ui pyspark spark-shell stop clean version

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

# Pull Spark Docker image with retry logic
pull-spark:
	@for i in 1 2 3 4 5; do \
		echo "Attempting to pull Spark image (attempt $$i/5)..."; \
		if docker pull bitnami/spark:3.5.3; then \
			echo "Successfully pulled Spark image"; \
			break; \
		fi; \
		if [ $$i -eq 5 ]; then \
			echo "Failed to pull Spark image after 5 attempts. Try these fixes:"; \
			echo "1. Check your internet connection"; \
			echo "2. Verify Docker Desktop is running"; \
			echo "3. Try running: docker system prune"; \
			echo "4. Try with a VPN if behind a corporate firewall"; \
			exit 1; \
		fi; \
		echo "Pull failed, waiting before retry..."; \
		sleep 10; \
	done

# Start Spark master node with Python support
start-master:
	docker run -d \
		--name spark-master \
		--hostname spark-master \
		--network spark-network \
		-p 8080:8080 \
		-p 7077:7077 \
		-e SPARK_MODE=master \
		-e SPARK_RPC_AUTHENTICATION_ENABLED=no \
		-e SPARK_RPC_ENCRYPTION_ENABLED=no \
		-e SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no \
		-e SPARK_SSL_ENABLED=no \
		bitnami/spark:3.5.3
	@echo "Waiting for Spark master to be ready..."
	@sleep 15
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
		bitnami/spark:3.5.3

# Open Spark UI in default browser
open-ui:
	open http://localhost:8080

# Start PySpark shell
pyspark:
	docker exec -it spark-master bash -c 'export PATH=/opt/bitnami/spark/bin:$$PATH && \
		export PYTHONPATH=/opt/bitnami/spark/python/lib/py4j-0.10.9.7-src.zip:/opt/bitnami/spark/python:$$PYTHONPATH && \
		export SPARK_HOME=/opt/bitnami/spark && \
		cd /opt/bitnami/spark && \
		bin/pyspark'

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
		if curl -s http://localhost:8080 >/dev/null; then \
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

# Check Spark version
version:
	docker exec spark-master spark-submit --version
