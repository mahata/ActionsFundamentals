# 🔨 ハンズオン: 再利用可能なワークフロー

このハンズオンラボでは、[再利用可能なワークフロー](https://docs.github.com/en/actions/using-workflows/reusing-workflows#creating-a-reusable-workflow)とそれを利用するワークフローを作成します。再利用可能なワークフローにパラメータを渡し、利用側のワークフローで出力パラメータを使用する方法を学習します。

このハンズオンラボは以下のステップで構成されています：
- [再利用可能なワークフローの作成](#再利用可能なワークフローの作成)
- [出力パラメータの追加](#出力パラメータの追加)
- [再利用可能なワークフローの利用](#再利用可能なワークフローの利用)

## 再利用可能なワークフローの作成

1. [新しいファイル](/../../new/main)`.github/workflows/reusable.yml`を作成します（ボックスにパスを含むファイル名をペーストします）。
2. 名前を`Reusable workflow`に設定します。

<details>
  <summary>解答</summary>

```YAML
name: Reusable workflow
```

</details>

3. `string`型で必須の[入力パラメータ](https://docs.github.com/en/enterprise-cloud@latest/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_call)`who-to-greet`を持つ`workflow_call`トリガーを追加します。デフォルト値を`World`に設定します。

<details>
  <summary>解答</summary>

```YAML
on:
  workflow_call:
    inputs:
      who-to-greet:
        description: 'The person to greet'
        type: string
        required: true
        default: World
```

</details>

4. `ubuntu-latest`で実行され、コンソールに"Hello <入力パラメータ>"をエコーする`reusable-job`という名前のジョブを追加します。

<details>
  <summary>解答</summary>

```YAML
jobs:
  reusable-job:
    runs-on: ubuntu-latest
    steps:
      - name: Greet someone
        run: echo "Hello ${{ inputs.who-to-greet }}"
```

</details>

## 出力パラメータの追加

1. ワークフローコマンドを使用して、現在の日時（`$(date)`を使用）を`current-time`という名前の出力パラメータに設定するid`time`を持つ追加ステップを追加します。

<details>
  <summary>解答</summary>

```YAML
      - name: Set time
        id: time
        run: echo "time=$(date)" >> $GITHUB_OUTPUT
```

</details>

2. `reusable-job`に`current-time`という出力を追加します。

<details>
  <summary>解答</summary>

```YAML
   outputs:
      current-time: ${{ steps.time.outputs.time }}
```

</details>

3. `workflow_call`に`current-time`という出力パラメータを追加し、ワークフローコマンドの出力に設定します。

<details>
  <summary>解答</summary>

```YAML
    outputs:
      current-time:
        description: 'The time when greeting.'
        value: ${{ jobs.reusable-job.outputs.current-time }}
```

</details>


<details>
  <summary>完全解答</summary>

```YAML
name: Reusable workflow

on:
  workflow_call:
    inputs:
      who-to-greet:
        description: 'The person to greet'
        type: string
        required: true
        default: World
    outputs:
      current-time:
        description: 'The time when greeting.'
        value: ${{ jobs.reusable-job.outputs.current-time }}

jobs:
  reusable-job:
    runs-on: ubuntu-latest
    outputs:
      current-time: ${{ steps.time.outputs.time }}
    steps:
      - name: Greet someone
        run: echo "Hello ${{ inputs.who-to-greet }}"
      - name: Set time
        id: time
        run: echo "time=$(date)" >> $GITHUB_OUTPUT
```

</details>

## 再利用可能なワークフローの利用

1. [新しいファイル](/../../new/main)`.github/workflows/reuse.yml`を作成します（ボックスにパスを含むファイル名をペーストします）。
2. 名前を`Reuse other workflow`に設定し、手動トリガーを追加します。

<details>
  <summary>解答</summary>

```YAML
name: Reuse other workflow

on: [workflow_dispatch]
```

</details>

3. 再利用可能なワークフローを使用し、あなたのユーザー名を入力パラメータとして渡す`call-workflow`ジョブを追加します。

<details>
  <summary>解答</summary>

```YAML
jobs:
  call-workflow:
    uses: ./.github/workflows/reusable.yml
    with:
      who-to-greet: '@octocat'
```

</details>

4. 出力パラメータ`current-time`をコンソールに書き込む別のジョブ`use-output`を追加します。（ヒント：出力にアクセスするためにneedsコンテキストを使用します）

<details>
  <summary>解答</summary>

```YAML
  use-output:
    runs-on: ubuntu-latest
    needs: [call-workflow]
    steps:
      - run: echo "Time was ${{ needs.call-workflow.outputs.current-time }}"
```

</details>

5. ワークフローを実行し、出力を確認します。

## まとめ

このラボでは、再利用可能なワークフローとそれを利用するワークフローの作成方法を学習しました。また、再利用可能なワークフローにパラメータを渡し、利用側のワークフローで出力パラメータを使用する方法も学習しました。

[README](../README.md)に進むことができます。
