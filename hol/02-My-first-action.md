# 🔨 ハンズオン: 初めてのAction

このハンズオンラボでは、dockerアクションの作成方法、パラメータの受け渡し、ワークフローへの戻り値の返し方を学習します。また、CIビルドでアクションをローカルでテストする方法も学習します。

このハンズオンラボは以下のステップで構成されています：
- [アクションの作成](#アクションの作成)
- [アクションのテスト](#アクションのテスト)
- オプション: [アクションのリリースと使用](#オプション-アクションのリリースと使用)


## アクションの作成

1. 新しいファイル[`hello-world-docker-action/action.yml`](/../../new/main?filename=hello-world-docker-action%2Faction.yml)を作成します：
<img width="400" alt="image" src="https://user-images.githubusercontent.com/5276337/174234628-14f58066-3188-42a6-9204-99c577558c08.png">

2. `action.yml`ファイルにコンテンツを追加します（[テンプレート](https://github.com/actions/hello-world-docker-action)と[ヘルプ](https://github.com/actions/hello-world-docker-action)を参照）。アクションが`Dockerfile`を実行し、デフォルト値`world`を持つパラメータ`who-to-greet`を渡すようにします。また、現在時刻を返すアウトプットも追加します。


<details>
  <summary>解答</summary>

```YAML
name: 'Hello World Docker Action'
description: 'Say hello to a user or the world.'
inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
    default: 'world'
outputs:
  time:
    description: 'The time we said hello.'
runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.who-to-greet }}
```

</details>

3. ファイルをコミットします（まだビルドを実行しないよう`[skip ci]`を付けます）。
4. `hello-world-docker-action`フォルダ内に[`Dockerfile`](/../../new/main?filename=hello-world-docker-action%2FDockerfile)を作成します。コンテナは`FROM alpine:3.22`を継承し、`entrypoint.sh`ファイルをコピーして実行する必要があります。`RUN chmod +x entrypoint.sh`でスクリプトを実行可能にすることを忘れないでください。

<details>
  <summary>解答</summary>

```dockerfile
FROM alpine:3.22

# Set the working directory inside the container
WORKDIR /usr/src

# Copy any source file(s) required for the action
COPY entrypoint.sh .
RUN chmod +x entrypoint.sh

# Configure the container to be run as an executable
ENTRYPOINT ["/usr/src/entrypoint.sh"]
```

</details>

5. ファイルをコミットします（今はビルドをスキップするため`[skip ci]`を付けます）。
6. フォルダ内にファイル[`entrypoint.sh`](/../../new/main?filename=hello-world-docker-action%2Fentrypoint.sh)を作成します。これはコンソールにメッセージを書き込み、アウトプットパラメータを設定する単純なbashスクリプトです。

<details>
  <summary>解答</summary>

```bash
#!/bin/sh -l

echo "hello $1"

echo "time=$(date)" >> $GITHUB_OUTPUT
```

</details>

7. ファイルをコミットします（まだビルドを実行しないよう`[skip ci]`を付けます）。

## アクションのテスト

1. アクションをテストするため、新しいワークフローファイル[`.github/workflows/hello-world-docker-ci.yml`](/../../new/main?filename=.github%2Fworkflows%2Fhello-world-docker-ci.yml&workflow_template=blank)を作成します。
2. ワークフローはアクションが変更された場合に全てのプッシュで実行されるようにします。また、手動でビルドを開始するための手動トリガーも追加します。
   リポジトリをチェックアウトして、gitリファレンスなしでローカルでアクションを参照します。

<details>
  <summary>解答</summary>

```YAML
name: CI Build for Docker Action
on:
  push:
    branches: [ main ]
    paths: [ hello-world-docker-action/** ]
  workflow_dispatch:

jobs:
  test-action:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3.3.0

      - name: Run my own container action
        id: hello-action
        uses: ./hello-world-docker-action
        with:
          who-to-greet: '@wulfland'

      - name: Output time set in the container
        run: echo "The time was ${{ steps.hello-action.outputs.time }} when the action said hello"

```

</details>

3. ワークフローを実行し、パラメータがアクションに渡され、ワークフローに戻される様子を確認します。

<img width="600" alt="image" src="https://user-images.githubusercontent.com/5276337/174239255-262a8014-4b66-40df-aa17-6f043f948342.png">

## オプション: アクションのリリースと使用

時間に余裕がある場合は、リリースを作成し、別のリポジトリのワークフローでアクションを使用できます。

> **注意**
> GitHub Actionはリポジトリのルートに存在する場合のみ公開できます。

1. 新しいパブリックリポジトリ`hello-world-docker-action`を作成し、[hello-world-docker-action](../hello-world-docker-action)から全てのファイルをコピーします。

2. タグ`v1`とタイトル`v1`で[新しいリリース](/../..releases/new)を作成します。`Generate release notes`をクリックしてリリースを公開します。

<img width="600" alt="image" src="https://user-images.githubusercontent.com/5276337/174241482-6d3d0c34-9d55-4e3d-86fa-8ac28055cea8.png">

3. Settings > Actions > General > Accessに移動し、`Accessible from repositories owned by the user '<your-github-username>`が選択されていることを確認します。

4. 新しいリポジトリを作成するか既存のものを使用し、バージョン`v1`で作成したアクションを呼び出すシンプルなワークフローを作成します。

<details>
  <summary>解答</summary>

```YAML
name: Test
on: [workflow_dispatch]

jobs:
  test-action:
    runs-on: ubuntu-latest
    steps:
      - name: Say hello
        uses: <your-github-username>/hello-world-docker-action@v1
        with:
          who-to-greet: '@octocat'
```

</details>

## まとめ

このハンズオンラボでは、dockerアクションの作成方法、パラメータの受け渡し、ワークフローへの戻り値の返し方、そしてCIビルドでアクションをローカルでテストする方法を学習しました。

次は[段階的デプロイメント](03-Staged-deployments.md)に進むことができます。
