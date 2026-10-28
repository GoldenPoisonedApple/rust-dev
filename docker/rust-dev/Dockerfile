# Rustプロジェクト共通開発ベースイメージ
# バージョンは各プロジェクトの rust-toolchain.toml の channel と揃える
# ビルド: make build
ARG RUST_VERSION=dummy
FROM rust:${RUST_VERSION}-bookworm

# ユーザー名 devuser は各 .devcontainer が前提にするため固定
# ホストと同一ユーザで所有権を揃える(DevContainerのupdateRemoteUserUID が起動時に非rootユーザを付け替える)
ARG USERNAME=devuser
ARG USER_UID=1000
ARG USER_GID=1000

# 共通依存
# - pkg-config, libssl-dev: Rustのコンパイルに必要
# - mold, clang: 高速なリンカー
# - sudo: updateRemoteUserUID が UID を付け替えるために使う
# DEBIAN_FRONTEND=noninteractive: 非対話式でインストールする(全てデフォルト選択) ビルドが止まるのを阻止
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        pkg-config \
        libssl-dev \
        mold \
        clang \
        sudo \
    && rm -rf /var/lib/apt/lists/*

# cargo-deny, cargo-watch をインストール
RUN cargo install cargo-deny cargo-watch --locked

# デフォルトのCARGO_HOMEはroot所有の /usr/local/cargo のため、設定とキャッシュはホームに置く
# それに合わせてPATHも最高優先度で追加
# PATHは左側から検索 A:B:C の場合、Aが優先度が高い
ENV CARGO_HOME=/home/${USERNAME}/.cargo \
    PATH=/home/${USERNAME}/.cargo/bin:${PATH}

# rustup と cargo は補助グループ rust へグループ書き込みにする
# UID 付け替え後も所有者がビルド時のまま rustup できる
RUN set -eux; \
		# ユーザー作成
    groupadd --gid "${USER_GID}" ${USERNAME}; \	
    useradd --uid "${USER_UID}" --gid "${USER_GID}" --create-home --shell /bin/bash ${USERNAME}; \
		# ツールチェーン(root所有)をユーザにもR/W可能にするためのグループ作成/追加
    groupadd --system rust; \
    usermod -aG rust ${USERNAME}; \
		# devuserはパスワード無しでsudoできるように
    echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME}; \
		# セキュリティ要件、構文チェック
    chmod 0440 /etc/sudoers.d/${USERNAME}; \
    visudo -cf /etc/sudoers.d/${USERNAME}; \
		# ツールチェーン(root所有)の所有グループを変更し、ユーザーにR/W可能にする(更新も可能にする)
    chgrp -R rust /usr/local/rustup /usr/local/cargo; \
    chmod -R g+rwX /usr/local/rustup /usr/local/cargo; \
    find /usr/local/rustup /usr/local/cargo -type d -exec chmod g+s {} +; \
		# ユーザ空間にcargo用ディレクトリ作成
    mkdir -p /home/${USERNAME}/.cargo/registry /home/${USERNAME}/.cargo/git; \
    chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}

# 共通cargo設定読み込み
COPY --chown=${USERNAME}:${USERNAME} cargo-config.toml /home/${USERNAME}/.cargo/config.toml

# ログインシェルは /etc/profile が PATH を標準値に戻す
# profile.d と bashrc の両方で cargo を戻す
RUN printf '%s\n' \
        'case ":${PATH}:" in' \
        '  *:/usr/local/cargo/bin:*) ;;' \
        '  *) PATH="/home/${USERNAME}/.cargo/bin:/usr/local/cargo/bin:${PATH}" ;;' \
        'esac' \
        'export PATH' \
        'export CARGO_HOME="${CARGO_HOME:-/home/${USERNAME}/.cargo}"' \
        > /etc/profile.d/rust.sh \
    && chmod 644 /etc/profile.d/rust.sh \
		&& printf '\n%s\n' '. /etc/profile.d/rust.sh' >> /home/${USERNAME}/.bashrc \
    && chown ${USERNAME}:${USERNAME} /home/${USERNAME}/.bashrc

# ユーザー切り替え
USER ${USERNAME}

# ホームディレクトリ移動
WORKDIR /home/${USERNAME}
