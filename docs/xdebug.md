# Xdebugによるデバッグの設定

## VSCodeを使用する場合
1. 開発環境のPHPバージョンに対応するXdebugをインストール
2. php.iniに下記を追加
    ```sh
    [xdebug]
    xdebug.remote_connect_back=1
    xdebug.remote_enable=1
    xdebug.remote_autostart=1
    xdebug.remote_port=9001

    xdebug.remote_log = /tmp/xdebug.log
    xdebug.idekey=xdebug
    ```
    - phpinfo()を表示し、xdebugが設定されていることを確認する
3. VSCodeのlaunch.jsonの"configurations"項目に下記項目を追加する
    ```sh
    {
        "name": "Listen for XDebug",
        "type": "php",
        "request": "launch",
        "port": 9001,
        "pathMappings": {
            "/var/www/html/": "${workspaceRoot}/"
        },
    },
    ```
4. VSCodeでF5キーを押してデバッガスタート
    