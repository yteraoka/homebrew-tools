# homebrew-tools

## GitHub App の作成

https://github.com/settings/apps から New GitHub App で GitHub App を作成する

- GitHub App name は任意の値 (Global で unique な値)
- Homepage URL はとりあえずこのプロジェクトの URL を指定
- Webhook の Active チェックボックスのチェックを外す
- 権限設定 (Permissions)
  - Repository permissions
    - Contents: Read and write
    - Metadata: Read-only (外せない)
- Where can this GitHub App be installed?
  - Only on this account

作成したら Generate a private key で秘密鍵を生成、ダウンロード
App ID をメモ

左ペインのメニューから Install App でこのプロジェクトと goreleaser を使ってるプロジェクトにインストール

## goreleaser を実行するプロジェクトにて

### Secrets 設定

Settings → Secrets

- `APP_ID`
  - メモった App ID (数字)
- `APP_PRIVATE_KEY`
  - 秘密鍵の pem ファイルの中身

### GitHub Actions の設定

```yaml
jobs:
  goreleaser:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # GitHub App から一時的なトークンを生成
      - name: Generate Token
        id: app-token
        uses: actions/create-github-app-token@v1
        with:
          app-id: ${{ secrets.APP_ID }}
          private_key: ${{ secrets.APP_PRIVATE_KEY }}
          # 自分のリポジトリと Tap リポジトリの両方に権限を絞る（任意）
          # repositories: "my-app, homebrew-tools"

      - name: Run GoReleaser
        uses: goreleaser/goreleaser-action@v6
        with:
          distribution: goreleaser
          version: latest
          args: release --clean
        env:
          # 生成した一時トークンを GoReleaser に渡す
          GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
```

## goreleaser の設定

```yaml
# .goreleaser.yaml
brews:
  - repository:
      owner: your-org
      name: homebrew-tools
      # env で渡した名前と一致させる（デフォルトは GITHUB_TOKEN）
      token: "{{ .Env.GITHUB_TOKEN }}"
```

