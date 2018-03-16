.PHONY: build run default debug test doc upstream downstream

IMAGE_OPTIONS = \
    -p 11211:11211

VARIANT := upstream
DISTRO = fedora-26-x86_64
DG := /usr/bin/dg
GOMD2MAN = /usr/bin/go-md2man

DG_EXEC = ${DG} --max-passes 25 --distro ${DISTRO}.yaml --spec specs/configuration.yml --multispec specs/multispec.yml --multispec-selector variant=$(VARIANT)
DISTRO_ID = $(shell ${DG_EXEC} --template "{{ config.os.id }}")

IMAGE_REPOSITORY = $(shell ${DG_EXEC} --template "{{ spec.image_repository }}")

default: run

run: build
	docker run -d $(IMAGE_REPOSITORY)

debug: build
	docker run -t -i $(IMAGE_OPTIONS) -e MEMCACHED_DEBUG_MODE $(IMAGE_REPOSITORY) bash

build: doc
	docker build --tag=$(IMAGE_REPOSITORY) -f Dockerfile.rendered .

test: build
	MODULE=docker DOCKERFILE="../Dockerfile.rendered" URL="docker=$(IMAGE_REPOSITORY)" make -C tests test_mtf
	make -C tests test_conu

doc: dg
	mkdir -p ./root/
	${GOMD2MAN} -in=help/help.md.rendered -out=./root/help.1

upstream:
	make -e doc VARIANT="upstream"
	make VARIANT="upstream"

downstream:
	make -e doc VARIANT="downstream"
	make VARIANT="downstream"

dg:
	${DG_EXEC} --template Dockerfile --output Dockerfile.rendered
	${DG_EXEC} --template help/help.md --output help/help.md.rendered

clean:
	rm -f Dockerfile.*
	rm -f help/help.md.*
	rm -rf root
