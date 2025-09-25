# 🔨 ハンズオン: 初めてのワークフロー

このハンズオンラボでは、初めてのGitHub Actionsワークフローを作成し、ソフトウェア開発ライフサイクルでのタスク自動化にActionsを使用する方法を学習します。より詳しい背景情報については、GitHub Docsの[GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)ページを参照してください。頑張ってください！ 👍

このハンズオンラボは以下のステップで構成されています：
- [リポジトリの作成](#リポジトリの作成)
- [ワークフローの作成](#ワークフローの作成)
- [ワークフロー結果の確認](#ワークフロー結果の確認)

## リポジトリの作成

[ps-actions-sandbox組織のActionsFundamentalsリポジトリ](https://github.com/ps-actions-sandbox/ActionsFundamentals)にアクセスし、<kbd>Use this template</kbd>をクリックします：

<img width="400" alt="2022-09-18_11-24-58" src="https://user-images.githubusercontent.com/5276337/190895393-6fa0fad9-e05c-4fea-8126-a291b087d663.png">

あなたのGitHubユーザーを所有者として選択し、リポジトリに名前を付けます。アクション実行時間を無制限にするため、リポジトリはpublicのままにしておきます：

<img width="400" alt="2022-09-18_11-25-57" src="https://user-images.githubusercontent.com/5276337/190895398-751a1ec9-c1cf-497f-beb7-a6b53d4d911e.png">

新しいリポジトリで続行します。

## ワークフローの作成

**Actions** | [New Workflow](/../../actions/new)に移動し、[set up a workflow yourself](/../../new/main?filename=.github%2Fworkflows%2Fmain.yml&workflow_template=blank)をクリックします。

1. `.github/workflows`ディレクトリの`main.yml`ファイルを`github-actions-demo.yml`にリネームします。
<img width="400" alt="image" src="https://user-images.githubusercontent.com/5276337/174096754-4e2d7219-9caf-42e8-bfd9-c190762886d3.png">

2. テンプレートの内容を削除します - ワークフローをゼロから作成したいためです。
3. <kbd>Ctrl</kbd>+<kbd>Space</kbd>をクリックし、最初の要素としてnameを選択します：

<img width="400" alt="image" src="https://user-images.githubusercontent.com/5276337/174097468-8be92e37-7948-4895-b5ed-20a22c5773bc.png">

4. ワークフロー名を`GitHub Actions Demo`に設定します：

```YAML
name: GitHub Actions Demo
```

5. <kbd>Ctrl</kbd>+<kbd>Space</kbd>とドキュメントを活用して、ワークフローにトリガーを追加します。ワークフローを以下の条件でトリガーしたいと思います：
  - `main`ブランチへの全てのプッシュ（ただし、`.github`フォルダ内のファイルに変更がある場合を除く）
  - `main`をベースブランチとする全てのプルリクエスト
  - 毎週日曜日のUTC 6:15
  - 手動実行

<details>
  <summary>解答</summary>

```YAML
on:
  push:
    branches: [ main ]
    paths-ignore: [.github/**]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '15 6 * * 0'
  workflow_dispatch:
```

</details>

6. GitHubホストランナーの最新Ubuntu イメージで実行される`Build`ジョブを作成します。使用するラベルとバージョンについては、[仮想環境](https://github.com/actions/virtual-environments/)のドキュメントを確認してください。ジョブは以下のことを行う必要があります：
  - ワークフローをトリガーしたイベントの名前を出力
  - リポジトリが現在参照しているブランチの名前を出力
  - リポジトリ内の全ファイルをリスト表示

<details>
  <summary>解答</summary>

```YAML
jobs:
  Build:
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "🎉 The job was triggered by event: ${{ github.event_name }}"
          echo "🔎 The name of your branch is ${{ github.ref }} and your repository is ."

      - uses: actions/checkout@v3.3.0

      - name: List files in the repository
        run: |
          echo "The repository ${{ github.repository }} contains the following files:"
          tree
```

</details>

7. ワークフローファイルをコミットし、手動でワークフローをトリガーします。パスフィルターが機能している場合、自動実行されないはずです。[Action](/../../Actions)に移動し、[GitHub Actions Demo](/../../actions/workflows/github-actions-demo.yml)を選択して`Run workflow`をクリックします：

<img width="600" alt="image" src="https://user-images.githubusercontent.com/5276337/174105162-19f33fd1-8533-4860-9279-88fabec84451.png">


## ワークフロー結果の確認

1. ワークフロー実行をクリックします：

<img width="600" alt="image" src="https://user-images.githubusercontent.com/5276337/174105747-0e205e0d-37cc-464c-905b-5b29be74fc75.png">

2. ジョブ'Build'をクリックします：

<img width="400" alt="image" src="https://user-images.githubusercontent.com/5276337/174105990-a1c204c6-fb7d-44a4-9343-6982899edb25.png">

3. `Set up job`を展開し、行番号付きのログを確認します（行番号はリンクです）。仮想環境と含まれるソフトウェアに関する情報を確認します：

<img width="600" alt="image" src="https://user-images.githubusercontent.com/5276337/174106759-c2a8f933-74cf-42a4-899b-b29dc67eccd7.png">

4. ジョブを展開し、出力が正しいことを確認します。

<img width="400" alt="image" src="https://user-images.githubusercontent.com/5276337/174107136-af9187c1-dbee-4109-9ddc-f2abd4830282.png">

5. [README.md](/README.md)ファイルを変更して、他のトリガーを確認します：
  - 変更してコミット：ビルドをトリガー（`push`）
  - 変更して`[skip ci]`を追加（ワークフローをトリガーしない）：
  <img width="350" alt="image" src="https://user-images.githubusercontent.com/5276337/174110845-93d4a38a-9c8a-4336-9b6a-9089ea9a1cfd.png">

  - [README.md](/README.md)ファイルを変更してプルリクエストを作成（トリガー：`pull_request`）

## まとめ

このハンズオンでは、トリガー、ジョブ、ステップ、式を使用した初めてのワークフローの作成を学習しました。次に、[初めてのGitHub Action](02-My-first-action.md)を作成します。
