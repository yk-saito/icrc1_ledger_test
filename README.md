# 概要

ICRC-1 Fungible Token Standard を実装した Ledger を使用して、ローカルレプリカをセットアップする。

# ファイル構成

## icrc1.did

Ledger キャニスターの API。

## dfx.json

dfx SDK で使用するキャニスター設定ファイル。

# 手順

## ステップ 0: 準備

```bash
mkdir icrc1_ledger_test
cd icrc1_ledger_test
```

## ステップ 1: Ledger のダウンロード

以下は、2022/10/19 現在の最新バージョン（コミットログ）。
最新版は`dfinity/ic`のコミットログから取得可能。

```bash
export IC_VERSION=bed99acfb65c65a2e84adbe0acdbc48d1b864969
# 確認
env | grep IC_VERSION
```

設定したバージョンで、ファイルを二つ取得する。

```bash
curl -o ic-icrc1-ledger.wasm.gz https://download.dfinity.systems/ic/${IC_VERSION}/canisters/ic-icrc1-ledger.wasm.gz
curl -o icrc1.did https://raw.githubusercontent.com/dfinity/ic/${IC_VERSION}/rs/rosetta-api/icrc1/ledger/icrc1.did
# gzファイルを解凍
gunzip ic-icrc1-ledger.wasm.gz
```

ファイルの確認

```bash
ls -sh1 *

2824 ic-icrc1-ledger.wasm
   8 icrc1.did
```

### 注意

curl コマンドをコピーして実行する際、`curl -o icrc1.did https://raw.githubusercontent.com/dfinity/ic/$\{IC_VERSION\}/rs/rosetta-api/icrc1/ledger/icrc1.did`のように、${IC_VERSION}にエスケープが入ってしまうと、ファイルのダウンロードに失敗するので注意（ダウンロード自体は実行されるが、ファイルの中身がエラー）

## ステップ 2: レプリカを起動する

まず、dfx.json ファイルを作成する。

```bash
touch dfx.json
```

dfx.json ファイルに以下を記載する。バインドするポート番号は適宜修正する。

```json
{
  "canisters": {
    "icrc1-ledger": {
      "type": "custom",
      "wasm": "ic-icrc1-ledger.wasm",
      "candid": "icrc1.did"
    }
  },
  "networks": {
    "local": {
      "bind": "127.0.0.1:8000"
    }
  }
}
```

レプリカを起動する。

```
dfx start --background
```

以下のような出力があれば起動 OK。

```bash
 Oct 19 03:16:14.357 INFO Log Level: INFO
 Oct 19 03:16:14.370 INFO Starting server. Listening on http://127.0.0.1:8000/
```

Leger をデプロイする前に、プリンシパルを取得する必要がある。ローカルでセットアップする場合は、現在使用している ID のプリンシパルを使用することになる。そのため、現在の ID のプリンシパルを環境変数に格納する。

なお、この ID は民とするアカウントとアーカイブキャニスターのコントローラーとして使用される。

```bash
export PRINCIPAL=$(dfx identity get-principal)
# 確認
env | grep PRINCIPAL
PRINCIPAL=YOUR_PRINCIPAL_ID
```

最後にデプロイをする。

```bash
$ dfx deploy icrc1-ledger --argument "(record {
  token_symbol = \"TEX\";
  token_name = \"Token example\";
  minting_account = record { owner = principal \"$PRINCIPAL\"  };
  transfer_fee = 10_000;
  metadata = vec {};
  initial_balances = vec {};
  archive_options = record {
    num_blocks_to_archive = 2000;
    trigger_threshold = 1000;
    controller_id = principal \"$PRINCIPAL\";
  };
},)"
```

## ステップ 3 : トークンをミントする

`icrc1_transfer`で転送、`icrc1_balance_of`でユーザーの残高確認ができる。

```bash
dfx canister call icrc1-ledger icrc1_transfer '(record {
  to = record {owner = principal "PRINCIPAL_ID"};
  amount=1_000_000
},)'
```

```bash
$ dfx canister call icrc1-ledger icrc1_balance_of '(record { owner=principal "PRINCIPAL_ID" },)'

(1_000_000 : nat64)
```

# 参照

https://github.com/dfinity/ic/tree/master/rs/rosetta-api/icrc1/ledger

最後に、Ledger をデプロイする。
