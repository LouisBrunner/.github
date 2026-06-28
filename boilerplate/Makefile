BIOME_CMD ?= biome
ifeq ($(shell command -v $(BIOME_CMD) 2>/dev/null),)
	BIOME_CMD = bunx @biomejs/biome
endif

.DEFAULT_GOAL := all

# ── Aggregates ─────────────────────────────────────────────────────────────────

.PHONY: all
all: vet

# ── Lint / format ──────────────────────────────────────────────────────────────

.PHONY: vet
vet: lint-yaml lint-biome lint-toml lint-md lint-sh

# Go
.PHONY: lint-go
lint-go:
	gofmt -d -e -s . | tee /dev/stderr | awk 'END { if (NR > 0) { exit 1 } }'
	go vet ./...
	go tool honnef.co/go/tools/cmd/staticcheck ./...

.PHONY: format-go
format-go:
	gofmt -w -s .

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

# Biome (JSON / HTML / JS / TS / CSS)
.PHONY: lint-biome
lint-biome:
	$(BIOME_CMD) ci .

.PHONY: format-biome
format-biome:
	$(BIOME_CMD) check --write .

# YAML
.PHONY: lint-yaml
lint-yaml:
	yamllint .

.PHONY: format-yaml
format-yaml:
	@echo "yamllint has no auto-fix; manually correct flagged files"

# TOML
.PHONY: lint-toml
lint-toml:
	tombi format --check --diff
	tombi lint --error-on-warnings

.PHONY: format-toml
format-toml:
	tombi format

# Markdown
.PHONY: lint-md
lint-md:
	markdownlint-cli2 '**/*.{md,markdown}'

.PHONY: format-md
format-md:
	markdownlint-cli2 --fix '**/*.{md,markdown}'

# Shell
.PHONY: lint-sh
lint-sh:
	shfmt --diff .
	shellcheck $$(find . -name '*.sh')

.PHONY: format-sh
format-sh:
	shfmt --write .

# Rust
.PHONY: lint-rust
lint-rust:
	cargo fmt --check
	cargo clippy -- -D warnings

.PHONY: format-rust
format-rust:
	cargo fmt

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
