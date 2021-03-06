---
title: "AWS上でのCI/CD環境の作成例"
emoji: "🏭"
type: "tech"
topics: ["AWS", "CodeBuild", "Amazon EventBridge"]
published: true
---

## この記事で記述する範囲

- 以下では具体的にどのようにしてCodeBuildプロジェクトおよびCI/CD環境を構築するかを記述します。
- CodeCommitリポジトリの作成までの方法や、S3バケット配置後のコンテンツの配信については記述しません。

## サービス構成

- CodeCommit
- CodeBuild
- S3
- Amazon EventBridge

### 処理フロー

- CodeCommit上の特定ブランチへのマージを契機として、CodeBuildによるビルドを実行し、S3バケットに配置するまでを扱います。

## 基本的な考え方

- CI/CD構成を作成するにあたっての基本的なフェーズと、対応するAWSサービスは以下の通りです。

| フェーズ | 対応するサービス                       | 概要                                                                         |
| :--- | :----------------------------- | :------------------------------------------------------------------------- |
| トリガー | Amazon EventBridge, CodeCommit | CodeCommit上の変化がCloudWatch Eventとして発行されるので、その中からフィルタして目的のイベントが発生した場合に処理を開始する |
| ビルド  | CodeBuild, CodeCommit          | CodeCommitの特定リポジトリ・特定ブランチからソースを拾い、ビルド用のdockerイメージを立ち上げて、ビルド処理を実行する         |
| デプロイ | CodeBuild, S3                  | dockerイメージ内のビルド成果物（静的コンテンツ）をS3バケットに配置する                                    |

- 処理フローとしては上述のようになりますが、実際にプロジェクトを作成する順序としては、（ソースの作成→）デプロイ先の作成→ビルドの構成→トリガーの指定となります。
- 以下では、「ソースの作成」（CodeCommitリポジトリの作成）はできている前提で説明します。

## ビルド・デプロイフェーズ

### デプロイ先S3バケットの作成

- AWSマネジメントコンソールのサービス一覧から、S3を選択します。
- 「バケットを作成」を押して、新規バケットを作成します。（詳細割愛）

### buildspec.ymlの作成

- buildspec.ymlとは、CodeBuildでのビルドフローを記述したyml形式のファイルです。
- CodeBuildはデフォルトでリポジトリのルートにあるbuildspec.ymlを読み取りにいきます。特にこだわりがなければルートにこのファイル名で配置しましょう。
- 以下は、Node.js 14系を使用したwebpackビルドの一例です。npm scriptsについては適宜読み替えてください。

```yaml
    version: 0.2

    phases:
      install:
        runtime-versions:
          nodejs: 14
      pre_build:
        commands:
          - echo NPMパッケージをインストールします
          - npm i
        finally:
          - echo インストールが完了しました
      build:
        commands:
          - echo ビルドを開始します
          - npm run build
        finally:
          - echo ビルドが終了しました
      post_build:
        commands:
          - echo AWS S3にデプロイします
    artifacts:
      files:
        - 'dist/*'
      discard-paths: yes
```

- buildspec.ymlの仕様については、 <https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/build-spec-ref.html> をご参照ください。

### CodeBuildプロジェクトの作成

- AWSマネジメントコンソールのサービス一覧から、CodeBuildを選択します。
- 「ビルドプロジェクトを作成する」をクリックして、設定を記入します。
- 以下、特筆すべき入力項目について解説します。(デフォルトのままで良い部分は省略しています）

| 分類        | 項目名          | 内容                                                                                                                                       |
| :-------- | :----------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| プロジェクトの設定 | プロジェクト名      | 実施するCI/CDフローに対して適切な命名を行う（レポジトリ名＋ブランチ名など）                                                                                                 |
| ソース       | ソースプロバイダ     | CodeCommitを選択                                                                                                                            |
| ソース       | リポジトリ        | ビルド対象のソースが存在するリポジトリを選択                                                                                                                   |
| ソース       | ブランチ         | ビルド対象のブランチを選択                                                                                                                            |
| 環境        | 環境イメージ       | 特別なことをしない限りは「マネージド型イメージ」で良いと思います                                                                                                         |
| 環境        | オペレーティングシステム | 各言語ランタイム・バージョンでサポートされているOSランタイムが異なりますので、<https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html> で状況を確認の上選択 |
| アーティファクト  | タイプ          | Amazon S3を選択                                                                                                                             |
| アーティファクト  | バケット名        | 「デプロイ先S3の作成」で作ったS3バケットを指定                                                                                                                |

- 上記ほか、設定を入力し終えたら「ビルドプロジェクトを作成する」を押して完了です。
- ビルドプロジェクトの一覧から作成したビルドプロジェクトを選択して「ビルドを開始」を押すことで、ビルドを実行することができます。エラーが発生した場合は、ログを読んで修正しましょう。

### EventBridgeによるトリガーの作成

- AWS　マネジメントコンソールのサービス一覧から、Amazon EventBridgeを選択します。
- 「ルールを作成」をクリック
- 以下、特筆すべき入力項目について解説します。(デフォルトのままで良い部分は省略しています）

| 分類」     | 項目名        | 内容                                                            |
| :------ | :--------- | :------------------------------------------------------------ |
| 名前と説明   | 名前         | わかりやすく明確な名前をつける                                               |
| パターンを定義 | -        | 「イベントパターン」を選択。                                                |
| パターンを定義 | イベント一致パターン | 「サービスごとの事前定義パターン」を選択                                          |
| パターンを定義 | サービスプロバイダー | 「AWS」を選択                                                      |
| パターンを定義 | サービス名      | 「CodeCommit」を選択                                               |
| パターンを定義 | イベントタイプ    | 今回はマージトリガーを例に考えるので、「CodeCommit Pull Request State Change」を選択。 |

- イベントタイプを選択後、右のJSONエディターにデフォルトが表示されるので、「編集」を押してパターンを記述する。
- 以下は、developブランチへのマージをフィルタする例

```json
    {
      "source": [
        "aws.codecommit"
      ],
      "detail-type": [
        "CodeCommit Pull Request State Change"
      ],
      "detail": {
        "isMerged": [
          "True"
        ],
        "repositoryNames": [
          "リポジトリ名"
        ],
        "destinationReference": [
          "refs/heads/develop"
        ]
      }
    }
```

- 続けて以下を入力。

| 分類       | 項目名       | 内容                                                           |
| :------- | :-------- | :----------------------------------------------------------- |
| ターゲットを選択 | ターゲット     | 「CodeBuild プロジェクト」を選択                                        |
| ターゲットを選択 | プロジェクトARN | CodeBuildのページから作成したプロジェクトを開き、ARNをコピーして貼り付け(ビルドの詳細→プロジェクトARN) |

- 以上を入力後、「作成」をクリック。

以上で設定は完了です。
かなりざっくりとした設定方法で恐縮ですが、一例として参考にしていただけたらと思います。ありがとうございました。
