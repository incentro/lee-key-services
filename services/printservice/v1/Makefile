NAME := $(shell cd ..; basename $$(pwd))
VERSION := $(shell basename $$(pwd))

SERVICE_NAME := $(NAME)-$(VERSION)
SERVICE_ACCOUNT = $(SERVICE_NAME)@$(PROJECT).iam.gserviceaccount.com

REGION := europe-west1
HOST_TAIL := $(PROJ_HASH)-ew.a.run.app

# Docker-related
IMAGE_NAME := $(NAME)_$(VERSION)
TAG := latest

PORT := 8084


.PHONY:

all: build push-gcr service-account deploy

check:
ifndef PROJECT
	$(error PROJECT not specified)
endif


clean:
	go clean

build:
	docker build -t $(IMAGE_NAME) .

run:
	export PORT=$(PORT); \
	export LOCAL_ENVIRONMENT=1; \
	export WEBSITE_SERVICE=http://localhost:8080; \
	export ORDER_SERVICE=http://localhost:8081; \
	export PAYMENT_SERVICE=http://localhost:8082; \
	export NUMBER_SERVICE=http://localhost:8083; \
	export PRINT_SERVICE=http://localhost:8084; \
	go run *.go

docker-run:
	docker run \
		--network="host" \
		--env PORT=$(PORT) \
		--env LOCAL_ENVIRONMENT=1 \
		--env WEBSITE_SERVICE=http://localhost:8080 \
		--env ORDER_SERVICE=http://localhost:8081 \
		--env PAYMENT_SERVICE=http://localhost:8082 \
		--env NUMBER_SERVICE=http://localhost:8083 \
		--env PRINT_SERVICE=http://localhost:8084 \
		$(IMAGE_NAME)

push-gcr: check
	docker tag $(IMAGE_NAME) gcr.io/$(PROJECT)/$(IMAGE_NAME):$(TAG)
	docker push gcr.io/$(PROJECT)/$(IMAGE_NAME)

service-account:
	-gcloud iam service-accounts create $(SERVICE_NAME) \
		--project=$(PROJECT)

deploy: check
	gcloud run deploy $(SERVICE_NAME) \
		--project=$(PROJECT) \
		--region=$(REGION) \
		--platform=managed \
		--memory=128Mi \
		--image=gcr.io/$(PROJECT)/$(IMAGE_NAME) \
		--no-allow-unauthenticated \
		--service-account=$(SERVICE_ACCOUNT)

destroy: check
	gcloud iam service-accounts delete $(SERVICE_ACCOUNT)
	gcloud run services delete $(SERVICE_NAME) \
		--project=$(PROJECT) \
		--region=$(REGION) \
		--platform=managed
