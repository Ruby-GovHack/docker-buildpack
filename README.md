docker-buildpack
================

Dokku/Heroku buildpack for Dockerised code

This buildpack looks for a Makefile in the root folder and a docker subfolder.
Makefile should have a "rebuildandrun" target.

Example Makefile:

```
CONTAINER_NAME_DEV=govhack-backend-dev
IMAGE_NAME_DEV=${CONTAINER_NAME_DEV}-image

CONTAINER_NAME_PROD=govhack-backend
IMAGE_NAME_PROD=${CONTAINER_NAME_PROD}-image

# The directory which contains this Makefile.
BUILD_DIR := $(shell dirname $(abspath $(lastword $(MAKEFILE_LIST))))

build-dev:
	cp "${BUILD_DIR}/docker/dev/Dockerfile" "${BUILD_DIR}/Dockerfile"
	-docker build -t ${IMAGE_NAME_DEV} "${BUILD_DIR}"
	rm "${BUILD_DIR}/Dockerfile"
	
build-dev-nocache:
	cp "${BUILD_DIR}/docker/dev/Dockerfile" "${BUILD_DIR}/Dockerfile"
	-docker build --no-cache -t ${IMAGE_NAME_DEV} "${BUILD_DIR}"
	rm "${BUILD_DIR}/Dockerfile"

run:
	docker run -i -t --rm --name="${CONTAINER_NAME_DEV}" --hostname="${CONTAINER_NAME_DEV}" -v "${BUILD_DIR}/code:/code" -v "${BUILD_DIR}/docker/dev/mongodb:/var/lib/mongodb" -p 5556:5556 ${IMAGE_NAME_DEV}

rebuildandrun: build-dev run

build-prod:
	cp "${BUILD_DIR}/docker/prod/Dockerfile" "${BUILD_DIR}/Dockerfile"
	-docker build -t ${IMAGE_NAME_PROD} "${BUILD_DIR}"
	rm "${BUILD_DIR}/Dockerfile"

build-prod-nocache:
	cp "${BUILD_DIR}/docker/prod/Dockerfile" "${BUILD_DIR}/Dockerfile"
	-docker build --no-cache -t ${IMAGE_NAME_PROD} "${BUILD_DIR}"
	rm "${BUILD_DIR}/Dockerfile"

serve:
	docker run -d --name="${CONTAINER_NAME_PROD}" --hostname="${CONTAINER_NAME_PROD}" -v "${BUILD_DIR}/docker/prod/mongodb:/var/lib/mongodb" -p 5556:5556 ${IMAGE_NAME_PROD}

ssh:
	ssh -i "${BUILD_DIR}/docker/prod/${CONTAINER_NAME_PROD}.key" root@$(shell docker inspect --format="{{ .NetworkSettings.IPAddress }}" ${CONTAINER_NAME_PROD})

sshkeygen:
	ssh-keygen -t rsa -f "${BUILD_DIR}/docker/prod/${CONTAINER_NAME_PROD}.key"
	chmod 600 "${BUILD_DIR}/docker/prod/${CONTAINER_NAME_PROD}.key"

stop:
	-docker stop ${CONTAINER_NAME_PROD}

rm: stop
	-docker rm ${CONTAINER_NAME_PROD}

rebuildandserve: build-prod stop rm serve

```
