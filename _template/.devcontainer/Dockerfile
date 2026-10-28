# プロジェクト固有の OS 依存を足すとき用
# compose の image を build に差し替える
#   build: .
FROM rust-dev:1.99-bookworm

USER root
# 例:
# RUN apt-get update \
#     && apt-get install -y --no-install-recommends libpq-dev \
#     && rm -rf /var/lib/apt/lists/*

# ベースイメージのユーザー devuser に戻す
USER devuser

WORKDIR /workspace
