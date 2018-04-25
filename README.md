# サンプルデータを生成するやつ

発表で使用したデータを生成します。
Rakefileの先頭付近の環境変数を空気を読んで設定しつつ実行します。

```
bundle install --path vendor/bundle

# portとパスワードをデフォルトから変更する例
LM_PORT=12345 LM_PASSWORD=fuga bundle exec rake
```

