pages   := $(shell find . -type f -name '*.adoc')
out_dir := ./_public

docker_cmd  ?= docker
docker_opts ?= --rm --tty --user "$$(id -u)"

antora_cmd  ?= $(docker_cmd) run $(docker_opts) --volume "$${PWD}":/antora vshn/antora:2.3.3-1
antora_opts ?= --cache-dir=.cache/antora

asciidoctor_cmd  ?= $(docker_cmd) run $(docker_opts) --volume "$${PWD}":/documents/ asciidoctor/docker-asciidoctor asciidoctor
asciidoctor_pdf_cmd  ?= $(docker_cmd) run $(docker_opts) --volume "$${PWD}":/documents/ vshn/asciidoctor-pdf:1.14.0 --attribute toclevels=1
asciidoctor_epub3_cmd  ?= $(docker_cmd) run $(docker_opts) --volume "$${PWD}":/documents/ vshn/asciidoctor-epub3:1.14.0 --attribute toclevels=1
asciidoctor_opts ?= --destination-dir=$(out_dir)
asciidoctor_kindle_opts ?= --attribute ebook-format=kf8

vale_cmd ?= $(docker_cmd) run $(docker_opts) --volume "$${PWD}"/docs/modules/ROOT/pages:/pages vshn/vale:2.10.5.1 --minAlertLevel=error /pages
hunspell_cmd ?= $(docker_cmd) run $(docker_opts) --volume "$${PWD}":/spell vshn/hunspell:1.7.0 -d en,vshn -l -H _public/**/*.html
htmltest_cmd ?= $(docker_cmd) run $(docker_opts) --volume "$${PWD}"/_public:/test wjdp/htmltest:v0.12.0
preview_cmd ?= $(docker_cmd) run --rm --publish 35729:35729 --publish 2020:2020 --volume "${PWD}":/preview/antora vshn/antora-preview:2.3.14 --antora=docs --style=vshn

.PHONY: all
all: html pdf
documents: pdf manpage kindle epub

# This will clean the Antora Artifacts, not the npm artifacts
.PHONY: clean
clean:
	rm -rf $(out_dir) '?' .cache

.PHONY: check
check:
	$(vale_cmd) && cd docs && ./check_orphans.py

.PHONY: syntax
syntax: html
	$(hunspell_cmd)

.PHONY: htmltest
htmltest: html pdf epub kindle manpage
	$(htmltest_cmd)

.PHONY: preview
preview:
	$(preview_cmd)

.PHONY: html pdf manpage epub kindle
html:    $(out_dir)/index.html
pdf:     $(out_dir)/{{cookiecutter.project_slug}}.pdf
manpage: $(out_dir)/{{cookiecutter.project_slug}}.1
epub:    $(out_dir)/{{cookiecutter.project_slug}}.epub
kindle:  $(out_dir)/{{cookiecutter.project_slug}}-kf8.epub

$(out_dir)/index.html: playbook.yml $(pages)
	$(antora_cmd) $(antora_opts) $<

$(out_dir)/%.1: docs/%.adoc $(pages)
	$(asciidoctor_cmd) --backend=manpage --attribute doctype=manpage $(asciidoctor_opts) $<

$(out_dir)/%.pdf: docs/%.adoc $(pages)
	$(asciidoctor_pdf_cmd) $(asciidoctor_opts) $<

$(out_dir)/%.epub: docs/%.adoc $(pages)
	$(asciidoctor_epub3_cmd) $(asciidoctor_opts) $<

$(out_dir)/%-kf8.epub: docs/%.adoc $(pages)
	$(asciidoctor_epub3_cmd) $(asciidoctor_kindle_opts) $(asciidoctor_opts) $<

