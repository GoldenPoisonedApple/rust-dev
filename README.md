# Rustのよさげな開発環境！
それぞれのプロジェクトで独立したコンテナを作って開発しよう！

**ベースイメージ(`rust-dev`)とクレートのキャッシュ(`.cargo/registry`, `.cargo/git`)を共有しているからストレージに優しい！**

## 開発用イメージ！
開発用イメージ: `rust-dev`: 元イメージ(rust:bookworm)
- [Dockerfile](./docker/rust-dev/Dockerfile): ユーザをホストと同じユーザに揃える、開発に必要なツールのインストール
- [Makefile](./docker/rust-dev/Makefile): 必ず最初にここで開発用イメージをビルド、各プロジェクトはこのイメージを参照、元イメージのバージョンもここで指定

## つかいかた！
1. `_template/` をコピーしてプロジェクトを作成
2. 当該プロジェクトの`.devcontainer/compose.yml` のnameを修正
3. 当該プロジェクトの`.devcontainer/devcontainer.json` のimageを修正
4. 当該プロジェクトの`.devcontainer/` で `make build` でコンテナをビルド
5. 煮るなり焼くなり自由に開発！

- `cargo deny init`: all-features = true 推奨

### プロジェクト固有のOS依存を足すとき！
1. 当該プロジェクトの`.devcontainer/Dockerfile` を編集
2. 当該プロジェクトの`.devcontainer/compose.yml` の`image: `を`build: .`に変更
3. 上に同じ

## 所感
- 最初のみMakefileでベースイメージをビルドする必要がある

- 非rootユーザーでの開発環境構築にしたらDockerfileがえげちい事になった。
非rootでやる理由: ホストと同じユーザに揃えることで、ホストからも編集が可能になる
=> 所感としてはほぼやらんくて良かった、複雑になるだけ

- 最初に.devcontainer/Makefileからビルドしてからコンテナに入らないとdevcontainerの機能でキショいimage名になるのでやった方がいい
