### 環境
- hugo v0.148.2
- Ubuntu 24.04.3 LTS

### コマンド
ローカルで確認するとき
```
hugo server
```

ドラフトを含める場合は下記コマンド
```
hugo server -D
```

ビルドするとき
```
hugo
```

ドラフトを含める場合は下記コマンド
```
hugo -D
```

ビルドすると"public"フォルダにビルドされたファイルが生成される\
中身をコピーして"docs"フォルダに入れる
```
cp -rf ./public/* ./docs/.
```

その後pushをする\
Github Pagesで公開される

### サイトURL
- https://tiisanamaou.github.io/hugo-website/
