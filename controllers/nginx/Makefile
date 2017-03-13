all: push

BUILDTAGS=

# Use the 0.0 tag for testing, it shouldn't clobber any release builds
RELEASE?=0.9.0-beta.2
PREFIX?=gcr.io/google_containers/nginx-ingress-controller
GOOS?=linux
DOCKER?=gcloud docker --
GOLANG_CONTAINER = endianogino/golang-glide:1.8
BUILD_IN_CONTAINER = 1
DOCKER_RUN = docker run --rm -v $(shell pwd):/go/src/github.com/mrahbar/ingress -w /go/src/github.com/mrahbar/ingress

REPO_INFO=$(shell git config --get remote.origin.url)

ifndef COMMIT
  COMMIT := git-$(shell git rev-parse --short HEAD)
endif

PKG=github.com/mrahbar/ingress/controllers/nginx

build: clean
ifeq ($(BUILD_IN_CONTAINER),1)
	$(DOCKER_RUN) -e CGO_ENABLED=0 $(GOLANG_CONTAINER) CGO_ENABLED=0 GOOS=${GOOS} go build -a -installsuffix cgo \
		-ldflags "-s -w -X ${PKG}/pkg/version.RELEASE=${RELEASE} -X ${PKG}/pkg/version.COMMIT=${COMMIT} -X ${PKG}/pkg/version.REPO=${REPO_INFO}" \
		-o rootfs/nginx-ingress-controller ${PKG}/pkg/cmd/controller
else
	CGO_ENABLED=0 GOOS=${GOOS} go build -a -installsuffix cgo \
		-ldflags "-s -w -X ${PKG}/pkg/version.RELEASE=${RELEASE} -X ${PKG}/pkg/version.COMMIT=${COMMIT} -X ${PKG}/pkg/version.REPO=${REPO_INFO}" \
		-o rootfs/nginx-ingress-controller ${PKG}/pkg/cmd/controller
endif

container: build
	$(DOCKER) build --pull -t $(PREFIX):$(RELEASE) rootfs

push: container
	$(DOCKER) push $(PREFIX):$(RELEASE)

fmt:
	@echo "+ $@"
	@go list -f '{{if len .TestGoFiles}}"gofmt -s -l {{.Dir}}"{{end}}' $(shell go list ${PKG}/... | grep -v vendor) | xargs -L 1 sh -c

lint:
	@echo "+ $@"
	@go list -f '{{if len .TestGoFiles}}"golint {{.Dir}}/..."{{end}}' $(shell go list ${PKG}/... | grep -v vendor) | xargs -L 1 sh -c

test: fmt lint vet
	@echo "+ $@"
	@go test -v -race -tags "$(BUILDTAGS) cgo" $(shell go list ${PKG}/... | grep -v vendor)

cover:
	@echo "+ $@"
	@go list -f '{{if len .TestGoFiles}}"go test -coverprofile={{.Dir}}/.coverprofile {{.ImportPath}}"{{end}}' $(shell go list ${PKG}/... | grep -v vendor) | xargs -L 1 sh -c
	gover
	goveralls -coverprofile=gover.coverprofile -service travis-ci -repotoken ${COVERALLS_TOKEN}

vet:
	@echo "+ $@"
	@go vet $(shell go list ${PKG}/... | grep -v vendor)

clean:
	rm -f nginx-ingress-controller
