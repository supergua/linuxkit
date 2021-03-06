# This builds the supported LinuxKit kernels. Kernels are wrapped up
# in a scratch container, which contains the bzImage, a tar
# ball with modules, the kernel sources, and in some case, the perf binary.
#
# Each kernel is pushed to hub twice, once as
# linuxkit/kernel:<kernel>.<major>.<minor>-<hash> and once as
# linuxkit/kernel:<kernel>.<major>.x. The <hash> is the git tree hash
# of the current directory. The build will only rebuild the kernel
# image if the git tree hash changed.
#
# For some kernels we also build a separate package containing the perf utility
# which is specific to a given kernel. perf packages are tagged the same way
# kernel packages.

# Git tree hash of this directory. Override to force build
HASH?=$(shell git ls-tree HEAD -- ../$(notdir $(CURDIR)) | awk '{print $$3}')
# Name and Org on Hub
ORG?=linuxkit
IMAGE:=kernel
IMAGE_PERF:=kernel-perf

.PHONY: check tag push sign
# Targets:
# build: builds all kernels
# push:  pushes all tagged kernel images to hub
# sign:  sign and push all kernel images to hub
build:
push:
sign:

# A template for defining kernel build
# Arguments:
# $1: Full kernel version, e.g., 4.9.22
# $2: Kernel "series", e.g., 4.9.x
# $3: Build a debug kernel (used as suffix for image)
# This defines targets like:
# build_4.9.x, push_4.9.x and sign_4.9.x and adds them as dependencies
# to the global targets
# Set $3 to "_dbg", to build debug kernels. This defines targets like
# build_4.9.x_dbg and adds "_dbg" to the hub image name.
define kernel
build_$(2)$(3): Dockerfile Makefile $(wildcard patches-$(2)/*) kernel_config-$(2) kernel_config.debug
	docker pull $(ORG)/$(IMAGE):$(1)$(3)-$(HASH) || \
		docker build \
			--build-arg KERNEL_VERSION=$(1) \
			--build-arg KERNEL_SERIES=$(2) \
			--build-arg DEBUG=$(3) \
			--no-cache -t $(ORG)/$(IMAGE):$(1)$(3)-$(HASH) .

push_$(2)$(3): build_$(2)$(3)
	docker pull $(ORG)/$(IMAGE):$(1)$(3)-$(HASH) || \
		(docker push $(ORG)/$(IMAGE):$(1)$(3)-$(HASH) && \
		 docker tag $(ORG)/$(IMAGE):$(1)$(3)-$(HASH) $(ORG)/$(IMAGE):$(2)$(3) && \
		 docker push $(ORG)/$(IMAGE):$(2)$(3))

sign_$(2)$(3): build_$(2)$(3)
	DOCKER_CONTENT_TRUST=1 docker pull $(ORG)/$(IMAGE):$(1)$(3)-$(HASH) || \
		(DOCKER_CONTENT_TRUST=1 docker push $(ORG)/$(IMAGE):$(1)$(3)-$(HASH) && \
		 docker tag $(ORG)/$(IMAGE):$(1)$(3)-$(HASH) $(ORG)/$(IMAGE):$(2)$(3) && \
		 DOCKER_CONTENT_TRUST=1 docker push $(ORG)/$(IMAGE):$(2)$(3))

build: build_$(2)$(3)
push: push_$(2)$(3)
sign: sign_$(2)$(3)

ifneq ($(2), 4.4.x)
build_perf_$(2)$(3): build_$(2)$(3)
	docker pull $(ORG)/$(IMAGE_PERF):$(1)$(3)-$(HASH) || \
		docker build -f Dockerfile.perf \
			--build-arg IMAGE=$(ORG)/$(IMAGE):$(1)$(3)-$(HASH) \
	 		--no-cache --network=none -t $(ORG)/$(IMAGE_PERF):$(1)$(3)-$(HASH) .

push_perf_$(2)$(3): build_perf_$(2)$(3)
	docker pull $(ORG)/$(IMAGE_PERF):$(1)$(3)-$(HASH) || \
		(docker push $(ORG)/$(IMAGE_PERF):$(1)$(3)-$(HASH) && \
		 docker tag $(ORG)/$(IMAGE_PERF):$(1)$(3)-$(HASH) $(ORG)/$(IMAGE_PERF):$(2)$(3) && \
		 docker push $(ORG)/$(IMAGE_PERF):$(2)$(3))

sign_perf_$(2)$(3): build_perf_$(2)$(3)
	DOCKER_CONTENT_TRUST=1 docker pull $(ORG)/$(IMAGE_PERF):$(1)$(3)-$(HASH) || \
		(DOCKER_CONTENT_TRUST=1 docker push $(ORG)/$(IMAGE_PERF):$(1)$(3)-$(HASH) && \
		 docker tag $(ORG)/$(IMAGE_PERF):$(1)$(3)-$(HASH) $(ORG)/$(IMAGE_PERF):$(2)$(3) && \
		 DOCKER_CONTENT_TRUST=1 docker push $(ORG)/$(IMAGE_PERF):$(2)$(3))

build: build_perf_$(2)$(3)
push: push_perf_$(2)$(3)
sign: sign_perf_$(2)$(3)
endif

endef

#
# Build Targets
# Debug targets only for latest stable and LTS stable
#
$(eval $(call kernel,4.11.6,4.11.x))
$(eval $(call kernel,4.11.6,4.11.x,_dbg))
$(eval $(call kernel,4.9.33,4.9.x))
$(eval $(call kernel,4.9.33,4.9.x,_dbg))
$(eval $(call kernel,4.4.73,4.4.x))
