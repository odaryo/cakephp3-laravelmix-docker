# Dockerを使ったCakePHP開発環境の構築方法(laravel mixを添えて)

## ディレクトリ構造

下記の構成として解説する
```
workspace_dir
  + project_root
      + docker ： Dockerファイル一式
          + mysql
          + nginx
          + php
      + cake : cakePHPデータ
      + docs : ドキュメントファイルなど  
```

## 開発環境について
- Windowsを想定
  - 開発環境はDocker for windowsを利用して作成する
  - 32bit OSの場合は公式のものが使えないため、VM（Vagrant+VirtualBox）にLinuxを入れて、dockerを設定する

## バージョンなど
- Nginx 1.14.x
  - 本番ではOS付属のものを使用する予定のため、1.14を使用する（2019/03/13現在）
- PHP 7.3.x
- Mysql 8.0.x
  - ※CakePHP3.7がMysql 8の暗号化方式（caching_sha2_password）に対応していないため、5.7以前のもの（mysql_native_password）に変更する必要がある
  - ユーザ作成時に設定が必要

## 事前準備
- エディタのインストール＆設定
  - VSCode
- Gitのインストール
  - GitBash
  - ※改行コードをCRLFに自動変換しない設定にする
    - 気にせずインストールした場合は、GitBashで下記を実行
      ```sh
      $ git config --global core.autocrlf input
      ```
- Docker for windowsのインストール
  - Hyper-Vが有効でないと起動できない。
  - やり方は検索。
  - できない/したくない場合はDocker Toolboxを使う
- LinuxOSの場合
  - docker, docker-composeをインストール
    - Linuxで行う場合はインストール方法は公式サイトを参照
      - 日本語の情報はバージョンが古い場合があるため
    - docker
      - CentOS
        - https://docs.docker.com/install/linux/docker-ce/centos/
      - Ubuntu
        - https://docs.docker.com/install/linux/docker-ce/ubuntu/
    - docker-compose
      - linuxの項目を参照
        - https://docs.docker.com/compose/install/

## 環境構築
### １．コンテナの起動 ～ ブラウザでの閲覧

1. プロジェクトのダウンロード
   - git でダウンロードする
      ```sh
      $ pwd
      <path to workspace_dir>
      $ git clone [git URL] <project_root>
      ```
2. 設定ファイルの修正
   - env.sample を .envにコピーし、環境に合わせて修正する
      ```sh
      $ cd <project_root>/docker
      $ cp env.sample .env
      $ vi .env
      ```
   - 下記項目を環境に合わせて修正する
      -  COMPOSE_PROJECT_NAME=<container_name>
         -  dockerで複数環境を立てると、コンテナ名称が重複するため内容が上書きされてしまう。それを回避するため、プロジェクトごとの名称を設定する
      - Nginx、PHP、MySQL、Nodeのバージョン
        - 環境に合わせて設定する
        - ※ Nodeはscssのコンパイルに使用する
      - MYSQLの設定
        - MYSQLの項目を設定する（任意）
3. dockerの起動
   - 初回はイメージダウンロード&コンテナ作成のため5~10分程度かかる
    ```sh
    $ pwd
    <path to workspace_dir>/<project_root>/docker/
    $ docker-compose up --build -d

    ※2回目以降は下記コマンドで起動可能
    $ docker-compose up -d
    ```

### ２．CakePHPのインストール ＆ 初期設定
gitでプロジェクトをダウンロードした場合は下記設定済みのため、本章は飛ばしてください
1. CakePHPプロジェクトのインストール
    - <project_root>/cake/ ディレクトリへインストールする場合の書き方
      ```sh
      $ docker-compose exec php composer create-project --prefer-dist cakephp/app ../cake
      ```
    - インストール時に下記のように聞かれるため、Enterを押すとインストール完了
      ```sh
      Set Folder Permissions ? (Default to Y) [Y,n]? Created `config/app.php` file ...
      ```
1. CakePHP初期設定  
  ※今回は.envファイルを設定ファイルとして扱う  
  .envファイル（git管理しない）に環境ごとの設定を記載できるため、本番/テスト/開発環境の分割管理がしやすくなる
   1. config/.envの作成
      - config/.env.default をコピーして .env を作成
      - .envの下記項目を編集
        ```php
        export APP_NAME="__APP_NAME__"  // -> アプリ名を設定（例としてTEST_APPに設定）

        export APP_DEFAULT_LOCALE="en_US"  // -> ja_JP
        export APP_DEFAULT_TIMEZONE="UTC"  // -> Asia/Tokyo
        export SECURITY_SALT="__SALT__"  // -> 後述するsalt値をセット
        ```
        - SECURITY_SALTの値はconfig/app.phpの71行目あたりのsalt値をコピペする
          - salt（ソルト）値とは、セキュリティと暗号化の設定値で、ハッシュ関数で使用されるランダムな文字列です
          ```php
          'Security' => [
              'salt' => env('SECURITY_SALT', '[salt値]'), 
          ],
          ```
        - コピー後はSALT値を削除しておく
          ```php
          'Security' => [
              'salt' => env('SECURITY_SALT', ''),
          ],
          ```
      - .envにデータベースへの接続設定を記述
         ```php
         export DB_HOST="mysql"  # DBコンテナ名を指定
         export DB_SCHEMA="testdb" 
         export DB_USERNAME="testuser"
         export DB_PASSWORD="passwd"
         ```
   2. .envを読み込むよう設定する
      1. config/bootstrap.php の 56行目あたりのコメントアウトを外す
      ```php
       if (!env('APP_NAME') && file_exists(CONFIG . '.env')) {
         $dotenv = new \josegonzalez\Dotenv\Loader([CONFIG . '.env']);
         $dotenv->parse()
           ->putenv()
           ->toEnv()
           ->toServer();
       }
       ```
   3. config/app.phpの編集
      - 環境に合わせて設定ファイルを編集します
      - 編集項目
      ```php
       'Datasources' => [
           'default' => [
               ...  
               'host' => env('DB_HOST', 'localhost'),
               ...
               'username' => env('DB_USERNAME', ''),
               'password' => env('DB_PASSWORD', ''),
               'database' => env('DB_SCHEMA', ''),
               ...
               'encoding' => 'utf8mb4',
               'timezone' => env('APP_DEFAULT_TIMEZONE', 'UTC'),
      ```
    4. .gitignoreの編集
       - 設定ファイルをapp.confから変更したので、.bitignoreからapp.confをコメントアウト
          ```sh
          #/config/app.php
          ```
    5. ブラウザでスタートページが表示されることを確認する
       - http://localhost/ でアクセス
       - Databaseの項目が緑色であることを確認する
2. Laravel-mix インストール  
   ※ javascriptの圧縮、scssのコンパイルに使用するのでLaravel-mixをインストールします
   - 下記コマンドを実行（node.jsでの操作のため、コンテナをnodeで実施する）
      ```sh
      $ docker-compose exec node npx create-laravel-mix react --public-dir webroot
      $ docker-compose exec node yarn install
      ```
   - プロジェクトに下記ファイルが作成されます
      ```
      cake  
        + assets  
            + js  
            + sass  
        + node_modules  
        + package.json
        + webpack.mix.js
      ```
   - node_modulesは重く、git管理したくないため、gitignoreファイルに追記してやります
      ```sh
      $ echo "node_modules" >> <project_root>/cake/.gitignore
      ```
### ３．開発フローなど

1. cakeコマンドの実行
    ```sh
    $ docker-compose exec php bin/cake .....
    ```
1. sassなどのコンパイル
    ```sh
    $ docker-compose exec node yarn run dev

    ※ 常時監視させる場合は、下記を実行しておく
    $ docker-compose exec node yarn run watch
    ```
    