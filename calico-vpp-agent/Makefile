include ../.ci/common.mk

.PHONY: all build gobgp image push proto

TAG ?= latest # Tag images with :$(TAG)
ALSO_LATEST ?= n # If 'y' also tag images with :latest
GENERATE_LOG_FILE=../vpplink/generated/generate.log
VERSION_FILE=version

all: build gobgp image

export GOOS=linux

# We make felix-api-proxy a static executable as it will run in the calico container
# for which we have less control on the env and glibc version
.PHONY: felix-api-proxy
felix-api-proxy: CGO_ENABLED=0
felix-api-proxy:
	${DOCKER_RUN} go build -o ./cmd/felix-api-proxy ./cmd/api-proxy

build: felix-api-proxy
	${DOCKER_RUN} go build -o ./cmd/calico-vpp-agent ./cmd
	${DOCKER_RUN} go build -o ./cmd/debug ./cmd/debug-state

gobgp: GOBGP_DIR:=$(shell ${DOCKER_RUN} go list -f '{{.Dir}}' -m github.com/osrg/gobgp/v3)
gobgp:
	${DOCKER_RUN} go build -o ./dep/gobgp $(GOBGP_DIR)/cmd/gobgp/

image: build gobgp
	@echo "Image tag                   : $(TAG)"                         > $(VERSION_FILE)
	@echo "VPP-dataplane version       : $(shell git log -1 --oneline)" >> $(VERSION_FILE)
	@cat $(GENERATE_LOG_FILE)                                           >> $(VERSION_FILE)
	docker build --pull -t calicovpp/agent:$(TAG) .
	@if [ "${ALSO_LATEST}" = "y" ]; then \
		docker tag calicovpp/agent:$(TAG) calicovpp/agent:latest; \
	fi

push: ${PUSH_DEP}
	docker push calicovpp/agent:$(TAG)
	@if [ "${ALSO_LATEST}" = "y" ]; then \
		docker push calicovpp/agent:latest; \
	fi

dev: image

proto:
	$(MAKE) -C proto $@
