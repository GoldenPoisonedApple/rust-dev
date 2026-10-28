# rust-dev:1.99-bookworm
# タグは rust-toolchain.toml の channel と揃える
# ホストの UID は渡さない。起動時に updateRemoteUserUID が合わせる
RUST_VERSION := 1.99
IMAGE := rust-dev:$(RUST_VERSION)-bookworm

.PHONY: init
init:
	docker volume create cargo-registry
	docker volume create cargo-git

.PHONY: build
build:
	docker build \
		--build-arg RUST_VERSION=$(RUST_VERSION) \
		-t $(IMAGE) \
		.
