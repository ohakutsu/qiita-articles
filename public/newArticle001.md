---
title: GitHub ActionsのReusable workflowを試す
tags:
  - GitHubActions
private: false
updated_at: "2023-07-21T17:18:09+09:00"
id: 20668a407d0cf2131b58
organization_url_name: qiita-inc
slide: false
---

この記事は [お題は不問！Qiita Engineer Festa 2023 で記事投稿！ - Qiita](https://qiita.com/official-events/4f3daca63fb78f16df0b) の参加記事です。

https://qiita.com/official-events/4f3daca63fb78f16df0b

## はじめに

GitHub Actions の機能に Workflow を再利用できるものがあります。

https://docs.github.com/en/actions/using-workflows/reusing-workflows

リポジトリごとに同じような Workflow を記述している場合、この機能を使ってまとめることができそうだったので試しに触ってみました。

## 再利用する Workflow へのアクセス

同一リポジトリにある場合はもちろん、別リポジトリの Workflow を参照することもできます。
また、プライベートリポジトリの Workflow も[設定を許可](https://docs.github.com/en/actions/creating-actions/sharing-actions-and-workflows-from-your-private-repository)すれば可能です。

## 試す

今回は、

- my-github-workflows
- sample-app

のリポジトリがあり、sample-app リポジトリ側で my-github-workflows リポジトリの Workflow ファイルを参照することをやります。

my-github-workflows リポジトリに共通で使いたい Workflow ファイルを用意します。

`on`フックで`workflow_call`を指定する必要があります。
また、`inputs`, `secrets`を設定するとその値を my-github-workflows 側に渡すことができます。

なお、以下の Workflow 実行時の Context は参照元（sample-app）側のものが使用されます。

```.github/workflows/notify-slack.yml
name: Notify Slack

on:
  workflow_call:
    inputs:
      message:
        required: true
        type: string
    secrets:
      slack_bot_token:
        required: true
      slack_channel_ids:
        required: true

jobs:
  notify:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: slackapi/slack-github-action@v1.24.0
        with:
          channel-id: ${{ secrets.slack_channel_ids }}
          payload: |
            {
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "plain_text",
                    "text": "${{ inputs.message }}",
                    "emoji": true
                  }
                },
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": ">*<${{ env.GITHUB_REPOSITORY_URL }}|${{ github.repository }}>*\n>*Job*\n><${{ env.GITHUB_ACTIONS_JOB_URL }}|${{ github.run_id }}>"
                  }
                }
              ],
              "text": "Repository: ${{ env.GITHUB_REPOSITORY_URL }}\nJob: ${{ env.GITHUB_ACTIONS_JOB_URL }}"
            }
        env:
          SLACK_BOT_TOKEN: ${{ secrets.slack_bot_token }}
          GITHUB_REPOSITORY_URL: ${{ github.server_url}}/${{ github.repository }}
          GITHUB_ACTIONS_JOB_URL: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

そして、sample-app リポジトリで my-github-workflows の Workflow を参照すると完成です。

```.github/workflows/sample.yml
name: Sample

on:
  workflow_dispatch:

jobs:
  sample:
    uses: "ohakutsu/my-github-workflows/.github/workflows/notify-slack.yml@main"
    with:
      message: test sample
    secrets:
      slack_bot_token: ${{ secrets.SLACK_BOT_TOKEN }}
      slack_channel_ids: ${{ secrets.SLACK_CHANNEL_IDS }}
```

無事動作確認できました。

## さいごに

簡単にですが、Workflow 再利用を試してみました。

Workflow を入れ子のように呼び出したり、Action を使用するときと同様に`@main`ブランチ指定やタグ指定もできそうなので便利そうです。
他にも、Reusable workflow から output を渡すことができたりもするようで、様々な用途に使えそうです。

詳しくは、↓ に載っています。

https://docs.github.com/en/actions/using-workflows/reusing-workflows
