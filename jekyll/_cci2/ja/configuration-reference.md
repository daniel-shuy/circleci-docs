---
layout: classic-docs
title: "CircleCI を設定する"
short-title: "CircleCI を設定する"
description: ".circleci/config.yml の設定リファレンス"
categories:
  - configuring-jobs
order: 20
---
このページでは `config.yml` ファイル内で使用する CircleCI 2.0 のコンフィグ用のキーについて解説しています。 CircleCI と連携したリポジトリのブランチに `.circleci/config.yml` ファイルが含まれていれば、それは CircleCI 2.x の環境で利用するということを意味します。

`config.yml` ファイルの全容は、このページ内の[サンプルコード](#full-example)で紹介しています。

**※**CircleCI 1.0 を利用していた場合、新たに `config.yml` ファイルを使うことで、以前とは異なるブランチで 2.x 環境におけるビルドを試すことができます。 従来の `circle.yml` の設定内容は残るため、`.circleci/config.yml` のないブランチとなっている CircleCI 1.0 環境の実行には支障はありません。

* * *

## 目次
{:.no_toc}

- 目次
{:toc}

* * *

## **`version`**

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- version | ○ | String | `2`、`2.0`、`2.1` のいずれかの値をとる。`.circleci/config.yml` ファイルの単純化やコードの再利用、パラメーター付きジョブを実現する 2.1 の新しいキーの解説については「[コンフィグを再利用する]({{ site.baseurl }}/2.0/reusing-config/)」を参照。
{: class="table table-striped"}

`version` フィールドは、非推奨になった場合、もしくは大きな変更があった場合に警告するかどうかの判断に用いられます。

## **`orbs`**（version: 2.1 が必須）

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- orbs | - | Map | ユーザー指定の名前によるマップ。Orb の参照名（文字列）または Orb の定義名（マップ) Orb の定義は、バージョン 2.1 のコンフィグにおける Orb 関連のサブセットとなる。詳細は「[Orb を作成する]({{ site.baseurl }}/2.0/creating-orbs/)」を参照。 executors | - | Map | Executor の定義文字列の参照名。 後述の [executors]({{ site.baseurl }}/2.0/configuration-reference/#executors-requires-version-21) のセクションを参照。 commands | - | Map | command 定義に対するコマンド名のマップ。 下記 [commands]({{ site.baseurl }}/2.0/configuration-reference/#commands-requires-version-21) のセクションを参照。
{: class="table table-striped"}

以下の例は認証済みの名前空間 `circleci` 配下にある `hello-build` という Orb のものです。

    version: 2.1
    orbs:
        hello: circleci/hello-build@0.0.5
    workflows:
        "Hello Workflow":
            jobs:
              - hello/hello-build
    

`circleci/hello-build@0.0.5` がそもそもの Orb の参照先ですが、この例では `hello` がその Orb の参照名となります。

## **`commands`**（version: 2.1 が必須）

commands は、ステップシーケンスをジョブ内で実行するマップの形で定義します。これを活用することで、複数ジョブ間で [コマンド定義の再利用]({{ site.baseurl }}/2.0/reusing-config/)が可能になります。

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- steps | ○ | Sequence | コマンド呼び出し元のジョブ内で実行するステップシーケンス parameters | - | Map | パラメーターキーのマップ。 詳細は「[コンフィグを再利用する]({{ site.baseurl }}/2.0/reusing-config/)」内の「[パラメーター構文]({{ site.baseurl }}/2.0/reusing-config/#parameter-syntax)」を参照 description | - | String | コマンドの内容を説明する文章
{: class="table table-striped"}

例

```yaml
commands:
  sayhello:
    description: "デモ用のごく単純なコマンドです"
    parameters:
      to:
        type: string
        default: "Hello World"
    steps:
      - run: echo << parameters.to >>
```

## **`executors`**（version: 2.1 が必須）

executors は、ジョブにおけるステップの実行環境を定義します。Executor は 1 つ定義するだけで複数のジョブで再利用可能です。

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- docker | ○<sup>(1)</sup> | List | docker executor を使用する。指定可能なオプションは[こちら](#docker) resource_class | - | String | ジョブにおいて各コンテナに割り当てられた CPU の数とメモリ容量 （`docker` Executor の時のみ有効）。**※**この機能を利用するには有償アカウントが必要です。 有償プランをお使いの方は[サポートチケット](https://support.circleci.com/hc/en-us/requests/new)を利用してリクエストしてください。 machine | ○<sup>(1)</sup> | Map | machine Executor を使用する。指定可能なオプションは[こちら](#machine) macos | ○<sup>(1)</sup> | Map | macOS Executor を使用する。指定可能なオプションは[こちら](#macos) shell | - | String | ステップ内のコマンド実行に用いるシェル。 ステップごとに使用する `shell` を変えることも可能（デフォルト値については「[デフォルトのシェルオプション](#default-shell-options)」を参照） environment | - | Map | 観光変数の名前と値のマップ
{: class="table table-striped"}

例

```yaml
version: 2.1
executors:
  my-executor:
    docker:
      - image: circleci/ruby:2.5.1-node-browsers

jobs:
  my-job:
    executor: my-executor
    steps:
      - run: echo Executor の“外”で定義しました
```

Executor の並列処理させ型については「[コンフィグを再利用する]({{ site.baseurl }}/2.0/reusing-config/)」のなかの「[Executor でパラメーターを使う](https://circleci.com/docs/2.0/reusing-config/#using-parameters-in-executors)」をご覧ください。

## **`jobs`**

実行処理は 1 つ以上のジョブで構成され、 それらのジョブの指定は `jobs` マップで行います。「[config.yml のサンプル]({{ site.baseurl }}/2.0/sample-config/)」では `job` マップの 2 通りの例を確認できます。 マップにおけるキーがジョブの名前となり、値はジョブの中身を記述するマップとします。

[Workflows]({{ site.baseurl }}/2.0/workflows/) を利用する際は、`.circleci/config.yml` ファイル内でユニークなジョブ名を設定しなければなりません。

Workflows を使わない場合は、`jobs` マップ内に `build` という名前のジョブを用意する必要があります。 `build` ジョブは、GitHub など VCS によるプッシュをトリガーとして実行する際のデフォルトのエントリーポイントとなります。 あるいは、CircleCI API を利用して別のジョブを実行することも可能です。

**※**ジョブの実行可能時間は最大5時間となります。 ジョブがタイムアウトするようなら、ジョブを並列処理させることも検討してください。

### **<`job_name`>**

1 つ 1 つのジョブはそれぞれ名前となるキーと、値となるマップからなります。 ジョブの名前は現在ある `jobs` リスト内でユニークでなければなりません。 値となるマップでは、下記の属性を使用できます。

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- docker | ○<sup>(1)</sup> | List | docker Executor を使用する。指定可能なオプションは[こちら](#docker) machine | ○<sup>(1)</sup> | Map | machine Executor を使用する。指定可能なオプションは[こちら](#machine) macos | ○<sup>(1)</sup> | Map | macOS Executor を使用する。指定可能なオプションは[こちら](#macos) shell | - | String | すべてのステップ内のコマンド実行に用いるシェル。 ステップごとに使用する `shell` を変えることも可能（デフォルト値については「[デフォルトのシェルオプション](#default-shell-options)」を参照） steps | ○ | List | [実行内容](#steps)のリスト working_directory | - | String | steps を実行するディレクトリ。 デフォルトは `~/project` となる（この `project` は文字列リテラルで、特定のプロジェクト名ではないことに注意）。 ジョブ内の実行プロセスは、このディレクトリを参照するために環境変数 `$CIRCLE_WORKING_DIRECTORY` を利用できる。 **※**YAML ファイルに記述したパスは展開*されません*。仮に `store_test_results.path` が `$CIRCLE_WORKING_DIRECTORY/tests` と設定されていたとしても、CircleCI はそのまま `$CIRCLE_WORKING_DIRECTORY` という `$` 記号付きの文字列のディレクトリ内に、サブディレクトリ `test` を格納しようとします。 parallelism | – | Integer | このジョブの並列処理の数（デフォルトは 1）environment | - | Map | 環境変数の名前と値のマップ branches | - | Map | Workflows でも、バージョン 2.1 のコンフィグでも**ない**単一のジョブにおいて、ホワイトリスト・ブラックリスト方式で特定のブランチの実行ルールを決めるためのマップ（デフォルトはホワイトリスト全体）。 Workflows やバージョン 2.1 のコンフィグで、ジョブにおいてブランチの実行を設定する方法については [Workflows](#workflows) を参照 resource_class | - | String | ジョブの各コンテナに割り当てるCPU数とメモリ容量 （`docker` Executor の時のみ有効）。**※**この機能を利用するには有償アカウントが必要です。 有償プランをお使いの方は[サポートチケット](https://support.circleci.com/hc/en-us/requests/new)を利用してリクエストしてください。
{: class="table table-striped"}

<sup>(1)</sup> これらのうちいずれか 1 つを必ず指定します。 2 つ以上指定した場合はエラーとなります。

#### `environment`

環境変数の名前と値のマップです。 CircleCI でセットした環境変数を上書きするのに使えます。

#### `parallelism`

`parallelism` の値を 2 以上に設定すると、独立した Executor が設定した数だけ起動し、そのジョブのステップを並列実行します。 ただし、並列処理を設定していても並列処理にならず、単一の Executor でしか実行されない場合もあります（[`deploy` ステップ](#deploy) がその一例です）。 詳しくは[パラレルジョブ]({{ site.baseurl }}/2.0/parallelism-faster-jobs/)を参照してください。

`working_directory` で指定したディレクトリが存在しないときは自動で作成されます。

例

```yaml
jobs:
  build:
    docker:
      - image: buildpack-deps:trusty
    environment:
      FOO: bar
    parallelism: 3
    resource_class: large
    working_directory: ~/my-app
    branches:
      only:
        - master
        - /rc-.*/
    steps:
      - run: make test
      - run: make
```

#### **`docker`** / **`machine`** / **`macos`**(*Executor*)

「Executor」は、端的に言えば「ステップを処理する場所」です。 CircleCI 2.0 は一度に必要な分の Docker コンテナを起動して、あるいは仮想マシンを利用して、目的とする環境をビルドできます。 詳しくは「[Executor タイプを選択する]({{ site.baseurl }}/2.0/executor-types/)」を参照してください。

#### `docker`
{:.no_toc}

`docker` キーは下記リストの要素を用いて設定します。

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- image | ○ | String | 使用するカスタム Docker イメージの名前 name | - | String | 他から参照する際のコンテナの名前。 デフォルトではコンテナサービスは `localhost` 経由でアクセスされる entrypoint | - | String / List | コンテナ起動時に実行するコマンド command | - | String / List | コンテナ起動時にルートプロセスとなる PID 1（または entrypoint の引数）にするコマンド user | - | String | Docker コンテナにおいてコマンドを実行するユーザー environment | - | Map | 環境変数の名前と値のマップ auth | - | Map | 標準の `docker login` 証明書によるレジストリの認証情報 aws_auth | - | Map | AWS EC2 Container Registry（ECR）の認証情報
{: class="table table-striped"}

一番最初に記述した `image` は、すべてのステップを実行するプライマリコンテナとなります。

`entrypoint` は Dockerfile のデフォルトのエントリーポイントを上書きします。

`command` は、（Dockerfile で指定していれば）イメージのエントリーポイントに対する引数として使われます。もしくは、（このスコープや Dockerfile 内にエントリーポイントがない場合は）実行形式として扱われます。

[プライマリコンテナ]({{ site.baseurl }}/2.0/glossary/#primary-container)（最初に宣言されたもの）において `command` の指定がない場合は、`command` とイメージエントリーポイントは無視されます。これは、エントリーポイントの実行可能ファイルがリソースを過大に消費したり、予期せず終了したりするのを防ぐためです。 今のところは、すべての `steps` はプライマリコンテナ内でのみ実行されます。

`name` では、セカンダリサービスコンテナを利用する際の名前を定義します。  デフォルトはどのサービスも `localhost` 上で直接見える状態になっています。 これは、例えば同じサービスのバージョン違いを複数立ち上げるときなど、localhost とは別のホスト名を使いたい場合に役立ちます。

`environment` の設定は、初期化用の `command` も含め、この Executor におけるすべてのコマンド実行で有効です。 `environment` による設定はジョブのマップにおいて何よりも優先されます。

タグやハッシュ値でイメージのバージョンを指定することもできます。 公式の Docker レジストリ（デフォルトは Docker Hub）のパブリックイメージはどんなものでも自由に使えます。 詳しくは 「[Executor タイプ]({{ site.baseurl }}/2.0/executor-types)」を参照してください。

例

```yaml
obs:
  build:
    docker:
      - image: buildpack-deps:trusty # プライマリコンテナ
        environment:
          ENV: CI

      - image: mongo:2.6.8
        command: [--smallfiles]

      - image: postgres:9.4.1
        environment:
          POSTGRES_USER: root

      - image: redis@sha256:54057dd7e125ca41afe526a877e8bd35ec2cdd33b9217e022ed37bdcf7d09673
```

プライベートな Docker イメージを利用するときは、`auth` を使ってユーザー名とパスワードを指定できます。 パスワード保護を目的に、下記のようにプロジェクトの設定値としてセットしておくこともできます。

```yaml
jobs:
  build:
    docker:
      - image: acme-private/private-image:321
        auth:
          username: mydockerhub-user  # 文字列リテラル値を指定するか
          password: $DOCKERHUB_PASSWORD  # プロジェクト設定の環境変数を指定する
```

[AWS ECR](https://aws.amazon.com/ecr/) にホストしているイメージを使う際は、AWS 証明書での認証が必要です。 デフォルトでは、CircleCI のプロジェクト設定画面にある「AWS Permissions」に追加した AWS 証明書、またはプロジェクト環境変数 `AWS_ACCESS_KEY_ID` と `AWS_SECRET_ACCESS_KEY` を使います。 下記のように `aws_auth` フィールドを用いて証明書をセットすることも可能です。

    jobs:
      build:
        docker:
          - image: account-id.dkr.ecr.us-east-1.amazonaws.com/org/repo:0.1
            aws_auth:
              aws_access_key_id: AKIAQWERVA  # 文字列リテラル値を指定するか
              aws_secret_access_key: $ECR_AWS_SECRET_ACCESS_KEY  # プロジェクト設定の環境変数を指定する
    

バージョン 2.1を使う場合は、ジョブにおいて[宣言済みコマンド]({{ site.baseurl }}/2.0/reusing-config/)の再利用が可能です。下記の例では `sayhello` コマンドを呼び出しています。

    jobs:
      myjob:
        docker:
          - image: "circleci/node:9.6.1"
        steps:
          - sayhello:
              to: "Lev"
    

#### **`machine`**
{:.no_toc}

[machine Executor]({{ site.baseurl }}/2.0/executor-types) は `machine` キーとともに下記リストの要素を用いて設定します。

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- enabled | - | Boolean | `machine` Executor の利用時は必ず true にする。 他に指定している値がないときは必須 image | – | String | 使用するイメージ（デフォルトは `circleci/classic:latest</0）。 <strong>※</strong>このキーはオンプレミス版の CircleCI ではサポートして<strong>いません</strong>。 プライベート環境における <code>michine` Executor イメージのカスタマイズに関する詳細は、[VM サービス]({{ site.baseurl }}/2.0/vm-service) を参照してください。 docker_layer_caching | - | Boolean | [Docker レイヤーキャッシュ]({{ site.baseurl }}/2.0/docker-layer-caching)を有効にするには `true` とする。 **※**Docker レイヤーキャッシュの利用には追加の料金がかかります。この機能を有効にするためにはサポートチケットを使って CircleCI のセールスチームに問い合わせる必要があります。
{: class="table table-striped"}

`machine` キーに `true` をセットする簡単な方法があります。

例

```YAML
jobs:
  build:
    machine:
      enabled: true

# もしくは単に以下のようにします

jobs:
  build:
    machine: true
```

CircleCI は `image` フィールドにおいて複数の machine イメージの指定をサポートしています。

- `circleci/classic:latest`（デフォルト）：Docker v`17.03.0-ce` と docker-compose v`1.9.0`、それと CircleCI 1.0 のビルドイメージに含まれる共通言語ツールを含んだ Ubuntu v`14.04` のイメージです。 `latest` というチャネルを指定することで、最新の検証イメージが使えます。チャネルに更新があるときは、1 週間前までに[アナウンス](https://discuss.circleci.com/t/how-to-subscribe-to-announcements-and-notifications-from-circleci-email-rss-json/5616)されます。
- `circleci/classic:edge`：Docker v`17.06.0-ce` と docker-compose v`1.14.0`、それと CircleCI 1.0 のビルドイメージに含まれる共通言語ツールを含んだ Ubuntu v`14.04` のイメージです。 `edge` というチャネルを指定することで、最終的に `classic:latest` に格上げされる予定のリリース候補版を使えます。
- `circleci/classic:201703-01` – docker 17.03.0-ce, docker-compose 1.9.0
- `circleci/classic:201707-01` – docker 17.06.0-ce, docker-compose 1.14.0
- `circleci/classic:201708-01` – docker 17.06.1-ce, docker-compose 1.14.0
- `circleci/classic:201709-01` – docker 17.07.0-ce, docker-compose 1.14.0
- `circleci/classic:201710-01` – docker 17.09.0-ce, docker-compose 1.14.0
- `circleci/classic:201710-02` – docker 17.10.0-ce, docker-compose 1.16.1
- `circleci/classic:201711-01` – docker 17.11.0-ce, docker-compose 1.17.1
- `circleci/classic:201808-01` – docker 18.06.0-ce, docker-compose 1.22.0

ジョブで使うイメージのバージョンを一定にするために、`year-month` の体裁でバージョン指定することもできます。 新しいイメージは続々とリリースされています。ぜひ[お知らせに登録](https://discuss.circleci.com/t/how-to-subscribe-to-announcements-and-notifications-from-circleci-email-rss-json/5616)して最新情報を受け取ってください。

**参考例：**Docker v`17.06.1-ce` と docker-compose v`1.14.0` を含む Ubuntu v`14.04` のイメージを使う場合

```yaml
version: 2
jobs:
  build:
    machine:
      image: circleci/classic:201708-01
```

machine Executor は、ジョブや Workflows で Docker イメージをビルドする際に効果的な [Docker レイヤーキャッシュ]({{ site.baseurl }}/2.0/docker-layer-caching)をサポートしています。

**例**

```yaml
version: 2
jobs:
  build:
    machine:
      docker_layer_caching: true    # デフォルト：false
```

#### **`macos`**
{:.no_toc}

CircleCI は、[macOS](https://developer.apple.com/macos/) 上でのジョブ実行をサポートしています。macOS アプリケーションや [iOS](https://developer.apple.com/ios/) アプリ、[tvOS](https://developer.apple.com/tvos/) アプリ、さらには[watchOS](https://developer.apple.com/watchos/) アプリのビルド、テスト、デプロイが可能です。 macOS 仮想マシン上でジョブを実行するには、ジョブ設定の最上位に `macos` キーを追加し、使いたい Xcode のバージョンを指定します。

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- xcode | ○ | String | 仮想マシンにインストールしている Xcode のバージョンを指定する。利用可能なバージョンについては、「iOS アプリをテストする」内の「[サポートしている Xcode のバージョン]({{ site.baseurl }}/2.0/testing-ios/#supported-xcode-versions)」を参照
{: class="table table-striped"}

**参考例：**macOS 仮想マシンを Xcode v`9.0` で使う場合

```yaml
jobs:
  build:
    macos:
      xcode: "9.0"
```

#### **`ブランチ`**

Workflows を利用**せず**、バージョン 2.0（2.1 は不可）のコンフィグを使っているケースでは、ブランチの実行をホワイトリスト・ブラックリスト方式で定義します。[Workflows]({{ site.baseurl }}/2.0/workflows/#using-contexts-and-filtering-in-your-workflows) を利用している場合は、ジョブレベルの branches は無視されるため、Workflows セクション内で設定します。 バージョン 2.1 のコンフィグにおいては、Workflows を追加することでフィルタリングを行います。 詳しくは [workflows](#workflows) を参照してください。 ジョブレベルの `branches` キーは、下記リストの要素を用いて設定します。

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- only | - | List | 実行するブランチのみを列挙する ignore | - | List | 実行しないブランチを列挙する
{: class="table table-striped"}

`only` と `ignore` に記述する内容は、完全一致のフルネームと正規表現で表せます。 正規表現では文字列**全体**にマッチさせる形にしなければなりません。 例えば下記のようにします。

```YAML
jobs:
  build:
    branches:
      only:
        - master
        - /rc-.*/
```

このケースでは、「master」ブランチと正規表現「rc-.*」にマッチするブランチのみが実行されます。

```YAML
jobs:
  build:
    branches:
      ignore:
        - develop
        - /feature-.*/
```

上記の例では、「develop」と正規表現「feature-.*」にマッチしたもの以外の全てのブランチが実行されます。

`ignore` と `only` の両方が同時に指定されていた場合は、`ignore` に関するフィルターのみが考慮されます。

コンフィグのルール設定によって実行されなかったジョブは、実行がスキップされたとして CircleCI 上に履歴表示されます。

#### **`resource_class`**

**※**Docker レイヤーキャッシュの利用には追加の料金がかかります。この機能を有効にするためには[サポートチケットを使って](https://support.circleci.com/hc/en-us/requests/new) CircleCI のセールスチームに問い合わせる必要があります。

有償プランにこの機能が追加されると、ジョブごとに CPUの数 とメモリ容量を設定できるようになります。利用可能なマシンリソースは下記の表の通りです。 `resource_class` を指定しない場合、もしくは正しい指定の仕方でなかったときは、デフォルトの `resource_class: medium` が指定されたものとみなされます。 `resource_class` キーは現在のところ `docker` Executor との組み合わせのみサポートしています。

クラス | 仮想CPU数 | メモリ容量 \---\---\---\---|\---\---\-----|\---\--- small | 1 | 2GB medium (default) | 2 | 4GB medium+ | 3 | 6GB large | 4 | 8GB xlarge | 8 | 16GB
{: class="table table-striped"}

CPU 数を取得するのに`/proc` ディレクトリをチェックする Java や Erlang などの開発言語においては、CircleCI 2.0 の resource_class 機能の使用時にパフォーマンスが低下する問題があることから、これを回避するため追加の設定が必要になる場合があります。 この問題は使用する CPU コアを 32 個要求したときに発生するもので、1 コアをリクエストしたときよりも実行速度が低下します。 該当する言語を使用しているユーザーは、問題の起こらない CPU リソースの使い方になるよう CPU コア数を決まった範囲に固定するなどして対処してください。

#### **`steps`**

ジョブにおける `steps` の設定は、キーと値のペアを 1 つずつ列挙する形で行います。キーはステップのタイプを表し、 値は設定内容を記述するマップか文字列（ステップのタイプによって異なる）のどちらかになります。 下記はマップを記述する場合の例です。

```yaml
jobs:
  build:
    working_directory: ~/canary-python
    environment:
      FOO: bar
    steps:
      - run:
          name: Running tests
          command: make test
```

ここでは `run` がステップのタイプとなります。 `name` 属性は CircleCI 上での表示に使われるものです。 `command` 属性は `run` ステップに特有の、実行するコマンドを定義するものです。

場合によっては steps はより簡便に記述することもできます。 例えば `run` ステップを下記のように記述可能です。

    jobs:
      build:
        steps:
          - run: make test
    

簡略化した表記方法では、実行する `command` を文字列値のようにして、`run` ステップをダイレクトに指定できるようになります。 このとき、省略された他の属性に対してはデフォルトの値が自動で設定されます（例えば `name` 属性には `command` と同じ値が設定されます）。

もう 1 つの簡略化の例としては、ステップ名をキーと値のペアの代わりとなる文字列として利用するシンプルな方法もあります。

    jobs:
      build:
        steps:
          - checkout
    

この例の `checkout` ステップは、プロジェクトのソースコードをジョブの [`working_directory`](#jobs) にチェックアウトします。

一般的にはステップは下記のように記述することになります。

キー | 必須 | 型 | 説明 \----|\---\---\-----|\---\---|\---\---\---\--- <step_type> | ○ | Map / String | ステップの設定マップ、またはステップで定義された内容を表す文字列
{: class="table table-striped"}

ステップのなかで利用可能な要素ごとの詳細は下記の通りです。

##### **`走らせる`**

Used for invoking all command-line programs, taking either a map of configuration values, or, when called in its short-form, a string that will be used as both the `command` and `name`. Run commands are executed using non-login shells by default, so you must explicitly source any dotfiles as part of the command.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- command | Y | String | Command to run via the shell name | N | String | Title of the step to be shown in the CircleCI UI (default: full `command`) shell | N | String | Shell to use for execution command (default: See [Default Shell Options](#default-shell-options)) environment | N | Map | Additional environmental variables, locally scoped to command background | N | Boolean | Whether or not this step should run in the background (default: false) working_directory | N | String | In which directory to run this step (default: [`working_directory`](#jobs) of the job) no_output_timeout | N | String | Elapsed time the command can run without output. The string is a decimal with unit suffix, such as "20m", "1.25h", "5s" (default: 10 minutes) when | N | String | [Specify when to enable or disable the step](#the-when-attribute). Takes the following values: `always`, `on_success`, `on_fail` (default: `on_success`)
{: class="table table-striped"}

Each `run` declaration represents a new shell. It's possible to specify a multi-line `command`, each line of which will be run in the same shell:

```YAML
- run:
    command: |
      echo Running test
      mkdir -p /tmp/test-results
      make test
```

###### *Default shell options*

The default value of shell option is `/bin/bash -eo pipefail` if `/bin/bash` is present in the build container. Otherwise it is `/bin/sh -eo pipefail`. The default shell is not a login shell (`--login` or `-l` are not specified by default). Hence, the default shell will **not** source your `~/.bash_profile`, `~/.bash_login`, `~/.profile` files. Descriptions of the `-eo pipefail` options are provided below.

`-e`

> Exit immediately if a pipeline (which may consist of a single simple command), a subshell command enclosed in parentheses, or one of the commands executed as part of a command list enclosed by braces exits with a non-zero status.

So if in the previous example `mkdir` failed to create a directory and returned a non-zero status, then command execution would be terminated, and the whole step would be marked as failed. If you desire the opposite behaviour, you need to add `set +e` in your `command` or override the default `shell` in your configuration map of `run`. 例えば下記のようにします。

```YAML
- run:
    command: |
      echo Running test
      set +e
      mkdir -p /tmp/test-results
      make test

- run:
    shell: /bin/sh
    command: |
      echo Running test
      mkdir -p /tmp/test-results
      make test
```

`-o pipefail`

> If pipefail is enabled, the pipeline’s return status is the value of the last (rightmost) command to exit with a non-zero status, or zero if all commands exit successfully. The shell waits for all commands in the pipeline to terminate before returning a value.

例えば下記のようにします。

```YAML
- run: make test | tee test-output.log
```

If `make test` fails, the `-o pipefail` option will cause the whole step to fail. Without `-o pipefail`, the step will always run successfully because the result of the whole pipeline is determined by the last command (`tee test-output.log`), which will always return a zero status.

Note that even if `make test` fails the rest of pipeline will be executed.

If you want to avoid this behaviour, you can specify `set +o pipefail` in the command or override the whole `shell` (see example above).

In general, we recommend using the default options (`-eo pipefail`) because they show errors in intermediate commands and simplify debugging job failures. For convenience, the UI displays the used shell and all active options for each `run` step.

For more information, see the [Using Shell Scripts]({{ site.baseurl }}/2.0/using-shell-scripts/) document.

###### *Background commands*

The `background` attribute enables you to configure commands to run in the background. Job execution will immediately proceed to the next step rather than waiting for return of a command with the `background` attribute set to `true`. The following example shows the config for running the X virtual framebuffer in the background which is commonly required to run Selenium tests:

```YAML
- run:
    name: Running X virtual framebuffer
    command: Xvfb :99 -screen 0 1280x1024x24
    background: true

- run: make test
```

###### *Shorthand syntax*

`run` has a very convenient shorthand syntax:

```YAML
- run: make test

# shorthanded command can also have multiple lines
- run: |
    mkdir -p /tmp/test-results
    make test
```

In this case, `command` and `name` become the string value of `run`, and the rest of the config map for that `run` have their default values.

###### The `when` Attribute

By default, CircleCI will execute job steps one at a time, in the order that they are defined in `config.yml`, until a step fails (returns a non-zero exit code). After a command fails, no further job steps will be executed.

Adding the `when` attribute to a job step allows you to override this default behaviour, and selectively run or skip steps depending on the status of the job.

The default value of `on_success` means that the step will run only if all of the previous steps have been successful (returned exit code 0).

A value of `always` means that the step will run regardless of the exit status of previous steps. This is useful if you have a task that you want to run regardless of whether the previous steps are successful or not. For example, you might have a job step that needs to upload logs or code-coverage data somewhere.

A value of `on_fail` means that the step will run only if one of the preceding steps has failed (returns a non-zero exit code). It is common to use `on_fail` if you want to store some diagnostic data to help debug test failures, or to run custom notifications about the failure, such as sending emails or triggering alerts in chatrooms.

###### 例

```yaml
steps:
  - run:
      name: Testing application
      command: make test
      shell: /bin/bash
      working_directory: ~/my-app
      no_output_timeout: 30m
      environment:
        FOO: bar

  - run: echo 127.0.0.1 devhost | sudo tee -a /etc/hosts

  - run: |
      sudo -u root createuser -h localhost --superuser ubuntu &&
      sudo createdb -h localhost test_db

  - run:
      name: Upload Failed Tests
      command: curl --data fail_tests.log http://example.com/error_logs
      when: on_fail
```

##### **The `when` Step** (requires version: 2.1)

A conditional step consists of a step with the key `when` or `unless`. Under the `when` key are the subkeys `condition` and `steps`. The purpose of the `when` step is customizing commands and job configuration to run on custom conditions (determined at config-compile time) that are checked before a workflow runs. See the [Conditional Steps section of the Reusing Config document]({{ site.baseurl }}/2.0/reusing-config/#defining-conditional-steps) for more details.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- condition | Y | String | A parameter value steps | Y | Sequence | A list of steps to execute when the condition is true
{: class="table table-striped"}

###### *例*

    version: 2.1
    
    jobs: # conditional steps may also be defined in `commands:`
      job_with_optional_custom_checkout:
        parameters:
          custom_checkout:
            type: string
            default: ""
        machine: true
        steps:
          - when:
              condition: <<parameters.custom_checkout>>
              steps:
                - run: echo "my custom checkout"
          - unless:
              condition: <<parameters.custom_checkout>>
              steps:
                - checkout
    workflows:
      build-test-deploy:
        jobs:
          - job_with_optional_custom_checkout:
              custom_checkout: "any non-empty string is truthy"
          - job_with_optional_custom_checkout
    

##### **`checkout`**

Special step used to check out source code to the configured `path` (defaults to the `working_directory`). The reason this is a special step is because it is more of a helper function designed to make checking out code easy for you. If you require doing git over HTTPS you should not use this step as it configures git to checkout over ssh.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- path | N | String | Checkout directory (default: job's [`working_directory`](#jobs))
{: class="table table-striped"}

If `path` already exists and is: * a git repo - step will not clone whole repo, instead will pull origin * NOT a git repo - step will fail.

In the case of `checkout`, the step type is just a string with no additional attributes:

```YAML
- checkout
```

**Note:** CircleCI does not check out submodules. If your project requires submodules, add `run` steps with appropriate commands as shown in the following example:

```YAML
- checkout
- run: git submodule sync
- run: git submodule update --init
```

**Note:** The `checkout` step will configure Git to skip automatic garbage collection. If you are caching your `.git` directory with [restore_cache](#restore_cache) and would like to use garbage collection to reduce its size, you may wish to use a [run](#run) step with command `git gc` before doing so.

##### **`setup_remote_docker`**

Creates a remote Docker environment configured to execute Docker commands. See [Running Docker Commands]({{ site.baseurl }}/2.0/building-docker-images/) for details.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- docker_layer_caching | N | boolean | set this to `true` to enable [Docker Layer Caching]({{ site.baseurl }}/2.0/docker-layer-caching/) in the Remote Docker Environment (default: `false`)
{: class="table table-striped"}

***Notes***: - A paid account is required to access Docker Layer Caching. 有償プランをお使いの方は[サポートチケット](https://support.circleci.com/hc/en-us/requests/new)を利用してリクエストしてください。 Please include a link to the project on CircleCI) with your request. - `setup_remote_docker` is not compatible with the `machine` executor. See [Docker Layer Caching in Machine Executor]({{ site.baseurl }}/2.0/docker-layer-caching/#machine-executor) for information on how to enable DLC with the `machine` executor.

##### **`save_cache`**

Generates and stores a cache of a file or directory of files such as dependencies or source code in our object storage. Later jobs can [restore this cache](#restore_cache). Learn more in [the caching documentation]({{ site.baseurl }}/2.0/caching/).

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- paths | Y | List | List of directories which should be added to the cache key | Y | String | Unique identifier for this cache name | N | String | Title of the step to be shown in the CircleCI UI (default: "Saving Cache") when | N | String | [Specify when to enable or disable the step](#the-when-attribute). Takes the following values: `always`, `on_success`, `on_fail` (default: `on_success`)
{: class="table table-striped"}

The cache for a specific `key` is immutable and cannot be changed once written.

**Note** If the cache for the given `key` already exists it won't be modified, and job execution will proceed to the next step.

When storing a new cache, the `key` value may contain special templated values for your convenience:

テンプレート | 解説 \----|\---\---\----
{% raw %}`{{ .Branch }}`
{% endraw %}

| 現在ビルドを実行しているバージョン管理システムのブランチ名。
{% raw %}`{{ .BuildNum }}`
{% endraw %}

| The CircleCI build number for this build.
{% raw %}`{{ .Revision }}`
{% endraw %}

| 現在ビルドを実行しているバージョン管理システムのリビジョン。
{% raw %}`{{ .CheckoutKey }}`{% endraw %} | The SSH key used to checkout the repo.
{% raw %}`{{ .Environment.variableName }}`{% endraw %} | `variableName`で示される環境変数 ([定義済み環境変数](https://circleci.com/docs/2.0/env-vars/#circleci-environment-variable-descriptions) 、もしくは[コンテキスト](https://circleci.com/docs/2.0/contexts)を指定できますが、ユーザー定義の環境変数は使えません)。
{% raw %}`{{ checksum "filename" }}`{% endraw %} | A base64 encoded SHA256 hash of the given filename's contents. This should be a file committed in your repo and may also be referenced as a path that is absolute or relative from the current working directory. Good candidates are dependency manifests, such as `package.json`, `pom.xml` or `project.clj`. It's important that this file does not change between `restore_cache` and `save_cache`, otherwise the cache will be saved under a cache key different than the one used at `restore_cache` time.
{% raw %}`{{ epoch }}`{% endraw %} | The current time in seconds since the unix epoch.
{% raw %}`{{ arch }}`{% endraw %} | OS と CPU の種類。 Useful when caching compiled binaries that depend on OS and CPU architecture, for example, `darwin amd64` versus `linux i386/32-bit`.
{: class="table table-striped"}

ステップの処理では、以上のようなテンプレートの部分は実行時に値が置き換えられ、その置換後の文字列が`キー`の値として使われます。

Template examples: *

{% raw %}
`myapp-{{ checksum "package.json" }}`{% endraw %} - cache will be regenerated every time something is changed in `package.json` file, different branches of this project will generate the same cache key. *

{% raw %}
`myapp-{{ .Branch }}-{{ checksum "package.json" }}`{% endraw %} - same as the previous one, but each branch will generate separate cache *{% raw %}
`myapp-{{ epoch }}`{% endraw %} - every run of a job will generate a separate cache

While choosing suitable templates for your cache `key`, keep in mind that cache saving is not a free operation, because it will take some time to upload the cache to our storage. So it make sense to have a `key` that generates a new cache only if something actually changed and avoid generating a new one every run of a job.

<div class="alert alert-info" role="alert">
<b>Tip:</b> Given the immutability of caches, it might be helpful to start all your cache keys with a version prefix <code class="highlighter-rouge">v1-...</code>. That way you will be able to regenerate all your caches just by incrementing the version in this prefix.
</div>

###### *例*

{% raw %}

```YAML
- save_cache:
    key: v1-myapp-{{ arch }}-{{ checksum "project.clj" }}
    paths:
      - /home/ubuntu/.m2
```

{% endraw %}

##### **`restore_cache`**

Restores a previously saved cache based on a `key`. Cache needs to have been saved first for this key using [`save_cache` step](#save_cache). Learn more in [the caching documentation]({{ site.baseurl }}/2.0/caching/).

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- key | Y <sup>(1)</sup> | String | Single cache key to restore keys | Y <sup>(1)</sup> | List | List of cache keys to lookup for a cache to restore. Only first existing key will be restored. name | N | String | Title of the step to be shown in the CircleCI UI (default: "Restoring Cache")
{: class="table table-striped"}

<sup>(1)</sup> at least one attribute has to be present. If `key` and `keys` are both given, `key` will be checked first, and then `keys`.

A key is searched against existing keys as a prefix.

**Note**: When there are multiple matches, the **most recent match** will be used, even if there is a more precise match.

例えば下記のようにします。

```YAML
steps:
  - save_cache:
      key: v1-myapp-cache
      paths:
        - ~/d1

  - save_cache:
      key: v1-myapp-cache-new
      paths:
        - ~/d2

  - run: rm -f ~/d1 ~/d2

  - restore_cache:
      key: v1-myapp-cache
```

In this case cache `v1-myapp-cache-new` will be restored because it's the most recent match with `v1-myapp-cache` prefix even if the first key (`v1-myapp-cache`) has exact match.

For more information on key formatting, see the `key` section of [`save_cache` step](#save_cache).

When CircleCI encounters a list of `keys`, the cache will be restored from the first one matching an existing cache. Most probably you would want to have a more specific key to be first (for example, cache for exact version of `package.json` file) and more generic keys after (for example, any cache for this project). If no key has a cache that exists, the step will be skipped with a warning.

A path is not required here because the cache will be restored to the location from which it was originally saved.

###### 例

{% raw %}

```YAML
- restore_cache:
    keys:
      - v1-myapp-{{ arch }}-{{ checksum "project.clj" }}
      # if cache for exact version of `project.clj` is not present then load any most recent one
      - v1-myapp-

# ... Steps building and testing your application ...

# cache will be saved only once for each version of `project.clj`
- save_cache:
    key: v1-myapp-{{ arch }}-{{ checksum "project.clj" }}
    paths:
      - /foo
```

{% endraw %}

##### **`deploy`**

Special step for deploying artifacts.

`deploy` uses the same configuration map and semantics as [`run`](#run) step. Jobs may have more than one `deploy` step.

In general `deploy` step behaves just like `run` with one exception - in a job with `parallelism`, the `deploy` step will only be executed by node #0 and only if all nodes succeed. Nodes other than #0 will skip this step.

###### 例

```YAML
- deploy:
    command: |
      if [ "${CIRCLE_BRANCH}" == "master" ]; then
        ansible-playbook site.yml
      fi
```

##### **`store_artifacts`**

Step to store artifacts (for example logs, binaries, etc) to be available in the web app or through the API. See the [Uploading Artifacts]({{ site.baseurl }}/2.0/artifacts/) document for more information.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- path | Y | String | Directory in the primary container to save as job artifacts destination | N | String | Prefix added to the artifact paths in the artifacts API (default: the directory of the file specified in `path`)
{: class="table table-striped"}

There can be multiple `store_artifacts` steps in a job. Using a unique prefix for each step prevents them from overwriting files.

###### 例

```YAML
- store_artifacts:
    path: /code/test-results
    destination: prefix
```

##### **`store_test_results`**

Special step used to upload test results so they display in builds' Test Summary section and can be used for timing analysis. To also see test result as build artifacts, please use [the **store_artifacts** step](#store_artifacts).

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- path | Y | String | Path (absolute, or relative to your `working_directory`) to directory containing subdirectories of JUnit XML or Cucumber JSON test metadata files
{: class="table table-striped"}

**Note:** Please write your tests to **subdirectories** of your `store_test_results` path, ideally named to match the names of your particular test suites, in order for CircleCI to correctly infer the names of your reports. If you do not write your reports to subdirectories, you will see reports in your "Test Summary" section such as `Your build ran 71 tests in unknown`, instead of, for example, `Your build ran 71 tests in rspec`.

###### *例*

Directory structure:

    test-results
    ├── jest
    │   └── results.xml
    ├── mocha
    │   └── results.xml
    └── rspec
        └── results.xml
    

`config.yml` syntax:

```YAML
- store_test_results:
    path: test-results
```

##### **`persist_to_workspace`**

Special step used to persist a temporary file to be used by another job in the workflow.

**Note:** Workspaces are stored for up to 30 days after being created. All jobs that try to use a Workspace older than 30 days, including partial reruns of a Workflow and SSH reruns of individual jobs, will fail.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- root | Y | String | Either an absolute path or a path relative to `working_directory` paths | Y | List | Glob identifying file(s), or a non-glob path to a directory to add to the shared workspace. Interpreted as relative to the workspace root. Must not be the workspace root itself.
{: class="table table-striped"}

The root key is a directory on the container which is taken to be the root directory of the workspace. The paths values are all relative to the root.

##### *Example for root Key*

For example, the following step syntax persists the specified paths from `/tmp/dir` into the workspace, relative to the directory `/tmp/dir`.

```YAML
- persist_to_workspace:
    root: /tmp/dir
    paths:
      - foo/bar
      - baz
```

After this step completes, the following directories are added to the workspace:

    /tmp/dir/foo/bar
    /tmp/dir/baz
    

###### *Example for paths Key*

```YAML
- persist_to_workspace:
    root: /tmp/workspace
    paths:
      - target/application.jar
      - build/*
```

The `paths` list uses `Glob` from Go, and the pattern matches [filepath.Match](https://golang.org/pkg/path/filepath/#Match).

    pattern:
        { term }
term:
            '*' matches any sequence of non-Separator characters
            '?' matches any single non-Separator character
            '[' [ '^' ] { character-range }
        ']' character class (must be non-empty)
            c matches character c (c != '*', '?', '\\', '[')
            '\\' c matches character c
    character-range:
            c matches character c (c != '\\', '-', ']')
            '\\' c matches character c
            lo '-' hi matches character c for lo <= c <= hi
    

The Go documentation states that the pattern may describe hierarchical names such as `/usr/*/bin/ed` (assuming the Separator is '/'). **Note:** Everything must be relative to the work space root directory.

##### **`attach_workspace`**

Special step used to attach the workflow's workspace to the current container. The full contents of the workspace are downloaded and copied into the directory the workspace is being attached at.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- at | Y | String | Directory to attach the workspace to.
{: class="table table-striped"}

###### *例*

```YAML
- attach_workspace:
    at: /tmp/workspace
```

Each workflow has a temporary workspace associated with it. The workspace can be used to pass along unique data built during a job to other jobs in the same workflow. Jobs can add files into the workspace using the `persist_to_workspace` step and download the workspace content into their file system using the `attach_workspace` step. The workspace is additive only, jobs may add files to the workspace but cannot delete files from the workspace. Each job can only see content added to the workspace by the jobs that are upstream of it. When attaching a workspace the "layer" from each upstream job is applied in the order the upstream jobs appear in the workflow graph. When two jobs run concurrently the order in which their layers are applied is undefined. If multiple concurrent jobs persist the same filename then attaching the workspace will error.

If a workflow is re-run it inherits the same workspace as the original workflow. When re-running failed jobs only the re-run jobs will see the same workspace content as the jobs in the original workflow.

Note the following distinctions between Artifacts, Workspaces, and Caches:

| Type | lifetime | Use | Example | |\---\---\-----|\---\---\---\---\-----|\---\---\---\---\---\---\---\---\---\---\---\---|\---\---\--- | Artifacts | Months | Preserve long-term artifacts. | Available in the Artifacts tab of the **Job page** under the `tmp/circle-artifacts.<hash>/container`   or similar directory.   | | Workspaces | Duration of workflow | Attach the workspace in a downstream container with the `attach_workspace:` step. | The `attach_workspace` copies and re-creates the entire workspace content when it runs. | | Caches | Months | Store non-vital data that may help the job run faster, for example npm or Gem packages. | The `save_cache` job step with a `path` to a list of directories to add and a `key` to uniquely identify the cache (for example, the branch, build number, or revision). Restore the cache with `restore_cache` and the appropriate `key`. |
{: class="table table-striped"}

Refer to the [Persisting Data in Workflows: When to Use Caching, Artifacts, and Workspaces](https://circleci.com/blog/persisting-data-in-workflows-when-to-use-caching-artifacts-and-workspaces/) for additional conceptual information about using workspaces, caching, and artifacts.

##### **`add_ssh_keys`**

Special step that adds SSH keys from a project's settings to a container. Also configures SSH to use these keys.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- fingerprints | N | List | List of fingerprints corresponding to the keys to be added (default: all keys added)
{: class="table table-striped"}

```yaml
steps:
  - add_ssh_keys:
      fingerprints:
        - "b7:35:a6:4e:9b:0d:6d:d4:78:1e:9a:97:2a:66:6b:be"
```

**Note:** Even though CircleCI uses `ssh-agent` to sign all added SSH keys, you **must** use the `add_ssh_keys` key to actually add keys to a container.

## **`workflows`**

Used for orchestrating all jobs. Each workflow consists of the workflow name as a key and a map as a value. A name should be unique within the current `config.yml`. The top-level keys for the Workflows configuration are `version` and `jobs`.

### **`version`**

The Workflows `version` field is used to issue warnings for deprecation or breaking changes during Beta.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- version | Y | String | Should currently be `2`
{: class="table table-striped"}

### **<`workflow_name`>**

A unique name for your workflow.

#### **`triggers`**

Specifies which triggers will cause this workflow to be executed. Default behavior is to trigger the workflow when pushing to a branch.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- triggers | N | Array | Should currently be `schedule`.
{: class="table table-striped"}

##### **`schedule`**

A workflow may have a `schedule` indicating it runs at a certain time, for example a nightly build that runs every day at 12am UTC:

    workflows:
       version: 2
       nightly:
         triggers:
           - schedule:
               cron: "0 0 * * *"
               filters:
                 branches:
                   only:
                     - master
                     - beta
         jobs:
           - test
    

###### **`cron`**

The `cron` key is defined using POSIX `crontab` syntax.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- cron | Y | String | See the [crontab man page](http://pubs.opengroup.org/onlinepubs/7908799/xcu/crontab.html).
{: class="table table-striped"}

###### **`filters`**

Filters can have the key `branches`.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- filters | Y | Map | A map defining rules for execution on specific branches
{: class="table table-striped"}

###### **`ブランチ`**
{:.no_toc}

The `branches` key controls whether the *current* branch should have a schedule trigger created for it, where *current* branch is the branch containing the `config.yml` file with the `trigger` stanza. That is, a push on the `master` branch will only schedule a [workflow]({{ site.baseurl }}/2.0/workflows/#using-contexts-and-filtering-in-your-workflows) for the `master` branch.

Branches can have the keys `only` and `ignore` which either map to a single string naming a branch. You may also use regular expressions to match against branches by enclosing them with `/`'s, or map to a list of such strings. 正規表現では文字列**全体**にマッチさせる形にしなければなりません。

- Any branches that match `only` will run the job.
- Any branches that match `ignore` will not run the job.
- If neither `only` nor `ignore` are specified then all branches will run the job.
- If both `only` and `ignore` are specified the `only` is considered before `ignore`.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- branches | Y | Map | A map defining rules for execution on specific branches only | Y | String, or List of Strings | Either a single branch specifier, or a list of branch specifiers ignore | N | String, or List of Strings | Either a single branch specifier, or a list of branch specifiers
{: class="table table-striped"}

#### **`jobs`**

A job can have the keys `requires`, `filters`, and `context`.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- jobs | Y | List | A list of jobs to run with their dependencies
{: class="table table-striped"}

##### **<`job_name`>**

A job name that exists in your `config.yml`.

###### **`requires`**

Jobs are run in parallel by default, so you must explicitly require any dependencies by their job name.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- requires | N | List | A list of jobs that must succeed for the job to start name | N | String | A replacement for the job name. Useful when calling a job multiple times. If you want to invoke the same job multiple times and a job requires one of the duplicate jobs, this is required. (2.1 only)
{: class="table table-striped"}

###### **`context`**

Jobs may be configured to use global environment variables set for an organization, see the [Contexts]({{ site.baseurl }}/2.0/contexts) document for adding a context in the application settings.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- context | N | String | The name of the context. The initial default name was `org-global`. With the ability to use multiple contexts, each context name must be unique.
{: class="table table-striped"}

###### **`type`**

A job may have a `type` of `approval` indicating it must be manually approved before downstream jobs may proceed. Jobs run in the dependency order until the workflow processes a job with the `type: approval` key followed by a job on which it depends, for example:

          - hold:
              type: approval
              requires:
                - test1
                - test2
          - deploy:
              requires:
                - hold
    

**Note:** The `hold` job name must not exist in the main configuration.

###### **`filters`**

Filters can have the key `branches` or `tags`. **Note** Workflows will ignore job-level branching. If you use job-level branching and later add workflows, you must remove the branching at the job level and instead declare it in the workflows section of your `config.yml`, as follows:

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- filters | N | Map | A map defining rules for execution on specific branches
{: class="table table-striped"}

###### **`ブランチ`**
{:.no_toc}
Branches can have the keys `only` and `ignore` which either map to a single string naming a branch. You may also use regular expressions to match against branches by enclosing them with '/s', or map to a list of such strings. 正規表現では文字列**全体**にマッチさせる形にしなければなりません。

- Any branches that match `only` will run the job.
- Any branches that match `ignore` will not run the job.
- If neither `only` nor `ignore` are specified then all branches will run the job.
- If both `only` and `ignore` are specified the `only` is considered before `ignore`.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- branches | N | Map | A map defining rules for execution on specific branches only | N | String, or List of Strings | Either a single branch specifier, or a list of branch specifiers ignore | N | String, or List of Strings | Either a single branch specifier, or a list of branch specifiers
{: class="table table-striped"}

###### **`tags`**
{:.no_toc}

CircleCI does not run workflows for tags unless you explicitly specify tag filters. Additionally, if a job requires any other jobs (directly or indirectly), you must specify tag filters for those jobs.

Tags can have the keys `only` and `ignore` keys. You may also use regular expressions to match against tags by enclosing them with '/s', or map to a list of such strings. 正規表現では文字列**全体**にマッチさせる形にしなければなりません。 Both lightweight and annotated tags are supported.

- Any tags that match `only` will run the job.
- Any tags that match `ignore` will not run the job.
- If neither `only` nor `ignore` are specified then the job is skipped for all tags.
- If both `only` and `ignore` are specified the `only` is considered before `ignore`.

Key | Required | Type | Description \----|\---\---\-----|\---\---|\---\---\---\--- tags | N | Map | A map defining rules for execution on specific tags only | N | String, or List of Strings | Either a single tag specifier, or a list of tag specifiers ignore | N | String, or List of Strings | Either a single tag specifier, or a list of tag specifiers
{: class="table table-striped"}

For more information, see the [Executing Workflows For a Git Tag]({{ site.baseurl }}/2.0/workflows/#executing-workflows-for-a-git-tag) section of the Workflows document.

###### *例*

    workflows:
      version: 2
    
      build_test_deploy:
        jobs:
          - flow
          - downstream:
              requires:
                - flow
              filters:
                branches:
                  only: master
    

Refer to the [Orchestrating Workflows]({{ site.baseurl }}/2.0/workflows) document for more examples and conceptual information.

## Full Example
{:.no_toc}

{% raw %}

```yaml
version: 2
jobs:
  build:
    docker:
      - image: ubuntu:14.04

      - image: mongo:2.6.8
        command: [mongod, --smallfiles]

      - image: postgres:9.4.1
        # some containers require setting environment variables
        environment:
          POSTGRES_USER: root

      - image: redis@sha256:54057dd7e125ca41afe526a877e8bd35ec2cdd33b9217e022ed37bdcf7d09673

      - image: rabbitmq:3.5.4

    environment:
      TEST_REPORTS: /tmp/test-reports

    working_directory: ~/my-project

    steps:
      - checkout

      - run:
          command: echo 127.0.0.1 devhost | sudo tee -a /etc/hosts

      # Create Postgres users and database
      # Note the YAML heredoc '|' for nicer formatting
      - run: |
          sudo -u root createuser -h localhost --superuser ubuntu &&
          sudo createdb -h localhost test_db

      - restore_cache:
          keys:
            - v1-my-project-{{ checksum "project.clj" }}
            - v1-my-project-

      - run:
          environment:
            SSH_TARGET: "localhost"
            TEST_ENV: "linux"
          command: |
            set -xu
            mkdir -p ${TEST_REPORTS}
            run-tests.sh
            cp out/tests/*.xml ${TEST_REPORTS}

      - run: |
          set -xu
          mkdir -p /tmp/artifacts
          create_jars.sh ${CIRCLE_BUILD_NUM}
          cp *.jar /tmp/artifacts

      - save_cache:
          key: v1-my-project-{{ checksum "project.clj" }}
          paths:
            - ~/.m2

      # Save artifacts
      - store_artifacts:
          path: /tmp/artifacts
          destination: build

      # Upload test results
      - store_test_results:
          path: /tmp/test-reports

  deploy-stage:
    docker:
      - image: ubuntu:14.04
    working_directory: /tmp/my-project
    steps:
      - run:
          name: Deploy if tests pass and branch is Staging
          command: ansible-playbook site.yml -i staging

  deploy-prod:
    docker:
      - image: ubuntu:14.04
    working_directory: /tmp/my-project
    steps:
      - run:
          name: Deploy if tests pass and branch is Master
          command: ansible-playbook site.yml -i production

workflows:
  version: 2
  build-deploy:
    jobs:
      - build:
          filters:
            branches:
              ignore:
                - develop
                - /feature-.*/
      - deploy-stage:
          requires:
            - build
          filters:
            branches:
              only: staging
      - deploy-prod:
          requires:
            - build
          filters:
            branches:
              only: master
```

{% endraw %}

## 関連情報
{:.no_toc}

[Config Introduction]({{site.baseurl}}/2.0/config-intro/)