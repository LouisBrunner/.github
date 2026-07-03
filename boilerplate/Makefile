MKFILE_PATH := $(abspath $(lastword $(MAKEFILE_LIST)))
MKFILE_DIR := $(dir $(MKFILE_PATH))

# TODO: not needed?
BIOME_CMD ?= biome
ifeq ($(shell command -v $(BIOME_CMD) 2>/dev/null),)
	BIOME_CMD = bunx @biomejs/biome
endif

.DEFAULT_GOAL := all

# ── Aggregates ─────────────────────────────────────────────────────────────────

.PHONY: all
all: lint

# ── Lint / format ──────────────────────────────────────────────────────────────

.PHONY: lint
lint: lint-super lint-biome lint-toml lint-sh lint-docker lint-tf

# --- Common ---

SUPER_LINTER_CONFIG_FILE ?= $(MKFILE_DIR)/super-config.env
SUPER_SETTINGS ?=
EXTRA_ENV_VARS = $(foreach var,$(SUPER_SETTINGS),-e "$(var)")
LOCAL_ENV_FILE ?= $(shell find $(PWD) -name 'super-config.env' -type f)
ifneq ($(LOCAL_ENV_FILE),)
	EXTRA_ENV_VARS += --env-file $(LOCAL_ENV_FILE)
endif
SUPER_DEBUG ?= false
ifeq ($(SUPER_DEBUG),true)
	EXTRA_ENV_VARS += -e LOG_LEVEL=DEBUG
endif
SUPER_CONFIG := $(shell mktemp -d)

.PHONY: lint-super
lint-super:
	@find $(MKFILE_DIR)/..  -depth 1 -type f -exec cp {} $(SUPER_CONFIG)/ \;
	@find $(PWD) -depth 1 -type f -exec cp {} $(SUPER_CONFIG)/ \;
	docker run --rm --platform linux/amd64 \
		-v $(PWD):/tmp/lint -v $(SUPER_CONFIG):/tmp/config:ro \
		-v super-linter-cache:/root/.cache \
		-e RUN_LOCAL=true -e DEFAULT_BRANCH=main -e LINTER_RULES_PATH=../config \
		--env-file $(SUPER_LINTER_CONFIG_FILE) $(EXTRA_ENV_VARS) \
		ghcr.io/super-linter/super-linter:slim-v8.7.0
	rm -rf $(SUPER_CONFIG)

# TOML
.PHONY: lint-toml
lint-toml:
	tombi format --check --diff
	tombi lint --error-on-warnings

.PHONY: format-toml
format-toml:
	tombi format

# Shell
SHELLCHECK_OPTS ?=
ifdef SHELL_ROOT
SHELLCHECK_OPTS += -P $(SHELL_ROOT)
endif
SC_FILES = $(shell find . -name '*.sh' -type f)
SHELL_NO_EXT_FILES != shell find . -type f ! -name "*.*" | xargs grep -lE '\# shellcheck shell=' 2>/dev/null | grep -v Makefile
SHELL_EXTRA_FILES ?=
SHELL_EXTRA_FILES += $(SHELL_NO_EXT_FILES)
SC_FILES += $(SHELL_NO_EXT_FILES)
SHELL_EXTRA_FILES_S = $(strip $(SHELL_EXTRA_FILES))
SC_FILES_S = $(strip $(SC_FILES))
ZSH_EXT_FILES = $(shell find . -name '*.zsh' -type f)
ZSH_SHEBANG_FILES = $(shell find . -type f ! -name '*.zsh' -exec sh -c 'head -n1 "$$1" 2>/dev/null | grep -qE "^\#!.*zsh"' sh {} \; -print)
ZSH_FILES_S = $(strip $(ZSH_EXT_FILES) $(ZSH_SHEBANG_FILES))

.PHONY: lint-sh
lint-sh:
	shfmt --diff .
	test -z "$(SHELL_EXTRA_FILES_S)" || shfmt --language-dialect bash --diff $(SHELL_EXTRA_FILES_S)
	test -z "$(SC_FILES_S)" || SHELLCHECK_OPTS="$(SHELLCHECK_OPTS)" shellcheck $(SC_FILES_S)
	test -z "$(ZSH_FILES_S)" || for f in $(ZSH_FILES_S); do zsh -n "$$f" || exit 1; done

.PHONY: format-sh
format-sh:
	shfmt --write .
	test -z "$(SHELL_EXTRA_FILES_S)" || shfmt --language-dialect bash --write $(SHELL_EXTRA_FILES_S)

# Docker
DOCKER_FILES := $(shell git ls-files | grep -E '(^|/)Dockerfile(\..+)?$$')
DOCKERFMT_OPTS ?= --newline -i 2

.PHONY: lint-docker
lint-docker:
	test -z "$(DOCKER_FILES)" || dockerfmt --check $(DOCKERFMT_OPTS) $(DOCKER_FILES)

.PHONY: format-docker
format-docker:
	test -z "$(DOCKER_FILES)" || dockerfmt --write $(DOCKERFMT_OPTS) $(DOCKER_FILES)

# TODO: replace with super?
# Biome (JSON / HTML / JS / TS / CSS)
.PHONY: lint-biome
lint-biome:
	$(BIOME_CMD) ci .

.PHONY: format-biome
format-biome:
	$(BIOME_CMD) check --write .

# TODO: replace with super?
# Terraform / Terragrunt
.PHONY: lint-tf
lint-tf:
	terraform fmt -recursive -check -diff
	terragrunt hcl fmt --check --diff
	terragrunt hcl validate

.PHONY: format-tf
format-tf:
	terraform fmt -recursive
	terragrunt hcl fmt

# --- Projects ---

# TODO: replace with super?
# Rust
.PHONY: lint-rust
lint-rust:
		cargo fmt --check
		cargo clippy -- -D warnings

.PHONY: format-rust
format-rust:
		cargo fmt

# TODO: replace with super?
# Go
.PHONY: lint-go
lint-go:
			gofmt -d -e -s . | tee /dev/stderr | awk 'END { if (NR > 0) { exit 1 } }'
			go vet ./...
			go tool honnef.co/go/tools/cmd/staticcheck ./...

.PHONY: format-go
format-go:
			gofmt -w -s .

# TODO: replace with super?
# UV / Python
.PHONY: lint-uv
lint-uv:
			uv run ruff check .
			uv run ruff format --diff .
			uv run ty check .

.PHONY: format-uv
format-uv:
			uv run ruff check --fix .
			uv run ruff format .

# Bun / JS / TS
.PHONY: lint-bun
lint-bun:
			bun run vet:ci

.PHONY: format-bun
format-bun:
			bun run format:fix
