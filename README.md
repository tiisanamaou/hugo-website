### コマンド
ローカルで確認するとき
```
hugo server -D
```

ビルドするとき
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
