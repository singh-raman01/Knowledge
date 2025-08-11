Makefile is a list of rules:
```make
target: dependencies
<TAB>command
```

- **target**: The thing you want to make (a file, binary, or pseudo-command)
- **dependencies**: Things that must be built/updated before the target
- **command**: Shell commands (MUST start with a tab, unless using `.RECIPEPREFIX`)

Variable defintion in a makefile:
```make
NAME = value      # simple assignment
NAME := value     # immediate assignment (expanded immediately)
NAME ?= value     # assign if not already set
NAME += value     # append

// EXAMPLE
BINARY_NAME = myapp
GO_FILES := $(wildcard *.go)

```

Make has some built-in functions:
```make
$(wildcard pattern)   # match files
$(patsubst from,to,text) # replace substrings
$(shell command)      # run a shell command
$(subst from,to,text) # string replace
$(dir names)          # get directory
$(notdir names)       # strip directory

// EXAMPLES
SRC := $(wildcard src/*.go)
OBJ := $(patsubst src/%.go,build/%.o,$(SRC))

```

We also have special built-in variables in makefile:
`$@` -> name of the target
`$<`-> first dependency
`$^`-> all dependencies
`$?`-> dependencies newer than the target

```make
output.txt: input.txt
	cp $< $@

```
Then input.txt is the first dependency, and output.txt is the target

Understanding targets and dependencies
```make
myapp: main.go utils.go
	go build -o myapp main.go utils.go
```
This will only rebuild if any `.go` file changes

- **Phony Targets**: if a target doesn't represent a file, then we mark it as phony to avoid conflicts.
```make
.PHONY: run clean
```
Without this, if a file name clean exists, make clean won't run.

**Pattern rules**:
They let us define generic recipes.
Ex. 
```make
%.o: %.c
	gcc -c $< -o $@

```
This code matches any .c file and builds a .o file

**Conditionals**:
```make
ifeq ($(ENV),prod)
	GO_FLAGS = -ldflags="-s -w"
else
	GO_FLAGS =
endif
```

We can include multiple makefiles, and split our makefiles.
`include configs.mk`

```make
# ========= CONFIGURATION =========
BINARY_NAME := myapp
SRC_DIR := .
GO_FILES := $(shell find $(SRC_DIR) -name '*.go')
GO_FLAGS :=
PKG := ./...
ENV ?= dev

# ========= PHONY TARGETS =========
.PHONY: all run build test clean lint fmt tidy deps help

# Default target
all: build

# ========= COMMANDS =========

run: ## Run the application
	@echo "Running $(BINARY_NAME) in $(ENV) mode..."
	ENV=$(ENV) go run $(SRC_DIR)

build: ## Build the binary
	@echo "Building $(BINARY_NAME)..."
	go build $(GO_FLAGS) -o $(BINARY_NAME) $(SRC_DIR)

test: ## Run tests
	go test -v $(PKG)

lint: ## Lint the code
	golangci-lint run ./...

fmt: ## Format code
	go fmt $(PKG)

tidy: ## Clean and sync go.mod
	go mod tidy

deps: ## Install dependencies
	go mod download

clean: ## Remove binary
	@echo "Cleaning up..."
	rm -f $(BINARY_NAME)

# ========= HELP =========
help: ## Show available commands
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | \
		awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-12s\033[0m %s\n", $$1, $$2}'

```
This is an example of a MAKEFILE which can help us achieve different goals

WE can extend this MAKEFILE to include production grade commands:
```make
###############################################################################
# PRODUCTION-READY Makefile for Go projects
#
# Features:
# - build, run, test, lint, fmt, tidy
# - embed version metadata from git (tag/commit/date)
# - cross-compile for multiple OS/ARCH
# - create release archives + checksums
# - Docker multi-stage build
# - optional GPG signing
# - auto-rebuild/watch using reflex (if installed)
###############################################################################

# ----------------- Configuration -----------------
BINARY_NAME ?= myapp
PKG := ./...
SRC_DIR := .
DIST_DIR := dist
BUILD_DIR := build

# Cross compile matrix - extend as needed
TARGETS := linux/amd64 linux/arm64 darwin/amd64 windows/amd64

# Go build flags - empty by default
GO_FLAGS ?=
# Stripped production flags
STRIP_FLAGS := -ldflags="-s -w"

# Tools (can be installed with `go install` or package manager)
REFLEX := $(shell command -v reflex 2>/dev/null || echo)
GOLANGCI_LINT := $(shell command -v golangci-lint 2>/dev/null || echo)

# ----------------- Version info (from git) -----------------
GIT_TAG := $(shell git describe --tags --dirty --always 2>/dev/null || echo "v0.0.0")
GIT_COMMIT := $(shell git rev-parse --short HEAD 2>/dev/null || echo "unknown")
BUILD_DATE := $(shell date -u +%Y-%m-%dT%H:%M:%SZ)
VERSION := $(GIT_TAG)

LDFLAGS := -X 'main.version=$(VERSION)' -X 'main.commit=$(GIT_COMMIT)' -X 'main.date=$(BUILD_DATE)'

# Full ldflags used in production build (strip + embed)
PROD_LDFLAGS := $(STRIP_FLAGS) -ldflags="$(LDFLAGS)"

# ----------------- Helpers -----------------
SHASUM := $(shell command -v shasum 2>/dev/null || echo)
SHA256 := $(if $(SHASUM),$(SHASUM) -a 256,sha256sum)

.PHONY: all help run build test lint fmt tidy deps clean cross release docker watch install-tools

# Default target
all: build

# ----------------- Development -----------------
run: ## Run locally (dev)
	@echo "ENV=dev go run $(SRC_DIR)"
	ENV=dev go run $(SRC_DIR)

test: ## Run tests
	go test -v $(PKG)

fmt: ## go fmt
	go fmt $(PKG)

tidy: ## go mod tidy
	go mod tidy

deps: ## download dependencies
	go mod download

lint: ## run linter (golangci-lint if available)
ifeq ($(GOLANGCI_LINT),)
	@echo "golangci-lint not found. Install via: go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest"
else
	$(GOLANGCI_LINT) run
endif

# ----------------- Build -----------------
# Simple local build (development)
build: ## Build (local, unstripped)
	@echo "Building $(BINARY_NAME) (dev) ... "
	go build $(GO_FLAGS) -o $(BINARY_NAME) $(SRC_DIR)

# Production build: embed version info and strip symbols
build-prod: ## Build production binary with version metadata (stripped)
	@echo "Building $(BINARY_NAME) (prod) version=$(VERSION) commit=$(GIT_COMMIT)"
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build $(PROD_LDFLAGS) -o $(BUILD_DIR)/$(BINARY_NAME)-linux-amd64 $(SRC_DIR)

# ----------------- Cross compile -----------------
# Cross compile for each OS/ARCH in TARGETS
cross: $(TARGETS:%=cross-%)
.PHONY: $(TARGETS:%=cross-%)
$(TARGETS:%=cross-%):
	@osarch=$(@:cross-%=%); os=$${osarch%/*}; arch=$${osarch#*/}; \
	echo "Cross-compiling for $$os/$$arch ..."; \
	out=$(BUILD_DIR)/$(BINARY_NAME)-$$os-$$arch; \
	stripflags="$(PROD_LDFLAGS)"; \
	CGO_ENABLED=0 GOOS=$$os GOARCH=$$arch go build $$stripflags -o $$out $(SRC_DIR); \
	if [ "$$os" = "windows" ]; then mv $$out $$out.exe; fi

# ----------------- Release packaging -----------------
# Create dist directory, cross compile, package .tar.gz/.zip and checksums
release: clean-dist cross package-checksums
	@echo "Release packages created in $(DIST_DIR)"

clean-dist:
	@rm -rf $(DIST_DIR) $(BUILD_DIR)
	@mkdir -p $(DIST_DIR) $(BUILD_DIR)

package-checksums:
	@echo "Packaging artifacts..."
	@for f in $(BUILD_DIR)/*; do \
		if [ -f $$f ]; then \
			base=$$(basename $$f); \
			# choose extension
			if echo $$base | grep -q '\.exe$$'; then \
				zipfile=$(DIST_DIR)/$$base.zip; zip -j $$zipfile $$f; else \
				tarfile=$(DIST_DIR)/$$base.tar.gz; tar -czf $$tarfile -C $(BUILD_DIR) $$base; fi; \
		fi; \
	done
	@echo "Generating checksums..."
	@for pkg in $(DIST_DIR)/*; do echo "sha256  $$pkg"; $(SHA256) $$pkg > $$pkg.sha256; done

# Optional: sign with gpg (requires gpg and private key)
sign-release: ## Sign release artifacts with GPG (optional)
	@for f in $(DIST_DIR)/*; do \
		if [ -f $$f ] && [ "$${f##*.}" != "sha256" ]; then \
			echo "Signing $$f"; gpg --armor --detach-sign $$f || echo "gpg sign failed for $$f"; fi; \
	done

# ----------------- Docker -----------------
# Build a small scratch or alpine final image via multi-stage build
docker: ## Build docker image (local)
	docker build --build-arg BINARY=$(BINARY_NAME) -t $(BINARY_NAME):$(VERSION) .

docker-push: ## Push docker image (requires login)
	docker push $(BINARY_NAME):$(VERSION)

# Example Dockerfile expected to be present in repo (multi-stage) ---
# FROM golang:1.20 AS builder
# WORKDIR /app
# COPY . .
# RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o app .
# FROM scratch
# COPY --from=builder /app/app /app
# ENTRYPOINT ["/app"]
# ---------------------------------------------------------------

# ----------------- Watch / Live reload -----------------
# Requires reflex (go install github.com/cespare/reflex@latest)
watch: ## Watch files and rerun `make run` (requires reflex)
ifeq ($(REFLEX),)
	@echo "reflex not found. Install: go install github.com/cespare/reflex@latest"
	@exit 1
else
	@reflex -r '\.go$$' -s -- sh -c 'echo "changed: restarting..."; make run'
endif

# ----------------- Helpers -----------------
clean: ## clean local artifacts
	@echo "Cleaning..."
	@rm -f $(BINARY_NAME)
	@rm -rf $(BUILD_DIR)

install-tools: ## helper - install common go tools
	go install github.com/cespare/reflex@latest || true
	go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest || true

help: ## Show this help
	@grep -E '^[a-zA-Z0-9_\-]+:.*?## .*$$' $(MAKEFILE_LIST) | \
		awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

```