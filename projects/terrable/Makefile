#!/usr/bin/env make

## SETTINGS ###################################################################
.EXPORT_ALL_VARIABLES:
.DELETE_ON_ERROR:
.ONESHELL:
.DEFAULT: help
.PHONY: \
		asciidoctor-render \
		clean \
		clobber \
		debug \
		help \
		publish-confluence \
		terraform-docs \
		terragrunt-dependency-graph \
		tflint \
		validate

help:
	@grep -B1 -E "^[a-zA-Z0-9_-]+\:([^\=]|$$)" Makefile \
		| grep -v -- -- \
		| sed 'N;s/\n/###/' \
		| sed -n 's/^#: \(.*\)###\(.*\):.*/\2###\1/p' \
		| column -t  -s '###'

#: Prints out info needed for debugging results
debug:
	@echo "MODULES: ${MODULES}"

MODULES := $(shell find modules/ -maxdepth 1 -mindepth 1 -type d -not -name '_\*')

## CLEANING ###################################################################

#: Removes build output and state.
clean:
	rm -rf \
		.build/ \
		modules/*/.terraform/ \
		modules/*/.terragrunt-cache/ \
		modules/*/.build/ \
		modules/*/_backend.tf \
		modules/*/_providers.tf \
		modules/*/terraform.log \
		modules/*/README.adoc \
		modules/module-dependencies.dot \
		module-dependencies.svg \
		README.html

#: Wipes out any and all untracked and ignored files.
clobber:
	git clean -dx \
		--force

## Directories ################################################################

.build/docs/: \
		| .build/
	mkdir ${@}
.build/:
	mkdir ${@}

## Tasks ######################################################################

#: Renders asciidoc to html.
asciidoctor-render: .build/docs/index.html
.build/docs/index.html: \
		README.adoc \
		modules/module-dependencies.dot \
		| .build/docs/
	asciidoctor \
		--require asciidoctor-diagram \
		--backend xhtml \
		--destination-dir .build/docs \
		--out-file index.html \
		README.adoc

#: Generates terraform asciidocs.
terraform-docs: $(foreach module,${MODULES},${module}/README.adoc)
$(foreach module,${MODULES},${module}/README.adoc): \
		$(wildcard $(dir ${@})/*.tf)
	cd $(dirname ${@})
	terraform-docs asciidoc table . \
	> $(basename ${@})

#: Generates graph from terragrunt module dependencies.
terragrunt-dependency-graph: modules/module-dependencies.dot
modules/module-dependencies.dot: \
		modules/terragrunt.hcl \
		$(foreach module,${MODULES},${module}/terragrunt.hcl)
	terragrunt graph-dependencies \
	| sed -r 's|"\S+?/modules/(\S+)"|"\1"|g' \
	> ${@}

#: Validates terraform configuration.
validate: $(foreach module,${MODULES},${module}/validation.json)
$(foreach module,${MODULES},${module}/validation.json): \
		$(wildcard $(dir ${@})/*.tf)
	terragrunt validate --json \
		--terragrunt-working-dir $(dir ${@}) \
	> ${@}

#: Lints the terraform code.
tflint: $(foreach module,${MODULES},${module}/checkstyle.xml)
$(foreach module,${MODULES},${module}/checkstyle.xml): \
		.tflint.hcl \
		$(wildcard $(dir ${@})/*.tf) \
		| $(realpath ~/.tflint.d/)
	tflint \
		--config=${<} \
		--format=checkstyle \
		$(dir ${@}) \
		> ${@}
$(realpath ~/.tflint.d/): \
		.tflint.hcl
	tflint \
		--config=${<} \
		--init

