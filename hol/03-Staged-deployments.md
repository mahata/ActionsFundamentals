# 🔨 ハンズオン: 段階的デプロイメント

このハンズオンラボでは、環境と承認を含む段階的デプロイメントワークフローを作成します。
> 注意: 無料アカウントではパブリックリポジトリでのみ実行できます。`プライベート`リポジトリへの環境追加は、Teamアカウント（またはそれ以上）を持つ組織とGitHub Proアカウントを持つユーザーのみ利用可能です。

このハンズオンラボは[初めてのワークフロー](01-My-first-workflow.md)をベースとし、以下のステップを追加します：
- [環境の作成と保護](#環境の作成と保護)
- [手動ワークフロートリガーに環境選択用入力を追加](#手動ワークフロートリガーに環境選択用入力を追加)
- [ワークフローステップの連鎖と条件実行](#ワークフローステップの連鎖と条件実行)
- [デプロイメントのレビューと承認](#デプロイメントのレビューと承認)

## 環境の作成と保護

1. [Settings](/../../settings) | [Environments](/../../settings/environments)に移動し、[New environment](/../../settings/environments/new)をクリックします
2. 名前に`Production`を入力し、`Configure environment`をクリックします
3. この環境の`Required reviewer`として自分自身を追加します：

<img width="349" alt="image" src="https://user-images.githubusercontent.com/5276337/174113475-967127de-45a7-4dc9-8477-4de4df62c7e6.png">

4. `main`ブランチのみがこの環境にデプロイできるように許可します：

<img width="350" alt="image" src="https://user-images.githubusercontent.com/5276337/174113782-70a1b18a-0ab9-49fd-a53e-cb2ea78916e1.png">

5. さらに2つの環境を作成します。`Test`と`Load-Test`は制限なしで作成します。

## 手動ワークフロートリガーに環境選択用入力を追加

ハンズオンラボ1（「初めてのワークフロー」）で作成したワークフローymlファイルを変更します。
以前に作成した`workflow_dispatch`トリガーに、environment型の入力を追加します。

<details>
  <summary>解答</summary>

```YAML
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy to'
        type: environment
        required: true
```

</details>

## ワークフローステップの連鎖と条件実行

1. ワークフローファイルに3つのジョブを追加します：
  - Test: `Build`の後に`ubuntu-latest`で実行。ワークフローが手動でトリガーされた場合のみ実行。環境`Test`で実行。ジョブはワークフローログに`Testing...`を書き込みます。
  - Load-Test: `Build`の後に`ubuntu-latest`で実行。ワークフローが手動でトリガーされた場合のみ実行。環境`Load-Test`で実行。ジョブはワークフローログに`Testing...`を書き込み、15秒間スリープします。
  - Production: `Test`と`Load-Test`の後に`ubuntu-latest`で実行。入力パラメータとして選択された場合のみ環境`Production`にデプロイ。環境のURLは`https://writeabout.net`。デプロイメントをシミュレートするため、ジョブは5つのステップを実行。各ステップはワークフローログに`Step x deploying...`を書き込み、10秒間スリープします。

<details>
  <summary>解答</summary>

```YAML
  Test:
    runs-on: ubuntu-latest
    if: github.event_name == 'workflow_dispatch'
    needs: Build
    environment: Test
    steps:
      - run: echo "🧪 Testing..."

  Load-Test:
    runs-on: ubuntu-latest
    if: github.event_name == 'workflow_dispatch'
    needs: Build
    environment: Load-Test
    steps:
      - run: |
          echo "🧪 Testing..."
          sleep 15

  Production:
    runs-on: ubuntu-latest
    needs: [Test, Load-Test]
    environment:
      name: Production
      url: https://writeabout.net
    if: github.event.inputs.environment == 'Production'
    steps:
      - run: |
          echo "🚀 Step 1..."
          sleep 10
      - run: |
          echo "🚀 Step 2..."
          sleep 10
      - run: |
          echo "🚀 Step 3..."
          sleep 10
      - run: |
          echo "🚀 Step 4..."
          sleep 10
      - run: |
          echo "🚀 Step 5..."
          sleep 10
```

</details>

2. ワークフローをトリガーし、環境として`Production`を選択します：

<img width="212" alt="image" src="https://user-images.githubusercontent.com/5276337/174119722-9b76d479-e355-414b-a534-03d8634536ef.png">

## デプロイメントのレビューと承認

1. ワークフロー実行を開き、レビューを開始します。

<img width="600" alt="image" src="https://user-images.githubusercontent.com/5276337/174120029-f395e8ec-5e6e-4350-94c5-130caefaafc2.png">

2. コメントを付けて承認します：

<img width="350" alt="image" src="https://user-images.githubusercontent.com/5276337/174120086-fed98feb-2d7f-476b-a997-1aa099de7d0e.png">

3. デプロイメント中のプログレスバーに注目してください...

<img width="200" alt="image" src="https://user-images.githubusercontent.com/5276337/174120314-c900585c-6b94-4fc2-8fe9-92452b0cf187.png">

4. 結果は次のようになり、承認とプロダクション環境のURLが含まれます：

<img width="800" alt="image" src="https://user-images.githubusercontent.com/5276337/174120381-cef48594-6663-481a-aadd-1ef0dbd50b0a.png">

## まとめ

このラボでは、GitHubで環境を作成・保護し、ワークフローで使用する方法を学習しました。また、条件付きでジョブやステップを実行し、`needs`キーワードを使用してジョブを連鎖させる方法も学習しました。

[再利用可能なワークフロー](04-Reusable-workflows.md)に進むことができます。
