# Asset外の静的ファイル

G*Advent Calendar(Groovy,Grails,Gradle,Spock...) Advent Calendar 2016 8日目の記事です。  
前日 -> [私です。Grails3でのES6とSCSSについて書きました。](http://koji-k.github.io/grails-tutorial/documents/asset-pipeline/additional.html)  
翌日 -> [@nobeansさん](http://qiita.com/nobeans)  

## 概要

Grails3では、JavaScript、CSS、そして画像と言った静的ファイルはAsset Pipelineプラグイン経由で利用するようになっています。  
なので静的ファイルの格納場所は`grails-app/assets/`配下になります。  

しかし、何らかの理由でAsset Pipelineを経由せずに、直接静的ファイルにアクセスしたい場合も当然ありますよね。  
Grails3でも当然その手段が提供されています。  
Asset Pipeline外で普通の画像などの静的リソースを取り扱いたい場合、  

- `src/main/webapp/`
- `src/main/resources/public/`

の2つのディレクトリが利用できます。  
利用方法は単純で、上記のディレクトリのどちらかにファイルを置くだけです。  
そうすると、デフォルトでは、`/static`というURLでアクセスできるようになっています。  

`src/main/webapp`の方は **WARには含まれるけど、JARに含まれません。**   


## resources/publicの利用
`src/main/resources`が存在しないので、`src/main/resources/public`というディレクトリまで手動で作成します。  

この`src/main/resources/public`の下に、画像やテキスト等の静的ファイルを置いておけば、`/static/`というURLでアクセスできます。  

このURLの仕様はGrails 3.1/3.0.13からのもののようです。  
例えば、`src/main/resources/public/test.jpg`というファイルがある場合、アクセスするためのURLは`/static/test.jpg`となります。  

このURLは`application.yml`で変更できます。  
以下の最後の2行を追加してGrailsを再起動すれば、`/test.jpg`で先ほどのファイルにアクセスできるようになります。  

```yaml
grails:
    profile: web
    codegen:
        defaultPackage: tran
    spring:
        transactionManagement:
            proxies: false
    # 以下の２行を追加
    resources:
        pattern: '/**' # '/hoge/**' とすれば、/hoge/test.jpgになる
```


1点注意することがあります。  
`src/main/resources`ディレクトリは、Grailsの起動時に1度読み込まれるだけなので、コーディング中に **ファイルを追加したり、内容を修正してもGrailsを再起動するまで反映されません。**  
Grailsのインタラクティブシェルで`compile`を実行することで反映させることも出来ますが、いずれにせよ再起動同様に処理に時間がかかるので注意です。  


### プログラムからResourcesディレクトリ内のファイルにアクセス
`resources/public`には、ブラウザからアクセスできますが、`resources`ディレクトリにある、それ以外のファイルやディレクトリにはブラウザからはアクセスできません。  
ですが、プログラムからアクセスすることは可能です。  
そのため、Grailsアプリケーションで利用するプログラム以外のデータ（例えばPDFを生成するならその際に利用するフォントファイル等）を格納しておくのに適しています。  

Grailsアプリケーションからリソースファイルにアクセスする方法は単純で、`src/main/resources`からの相対パスで指定して、以下のようにファイルの内容を取得できます。  

```groovy
// src/main/resources直下にあるdata.csv
grailsApplication.classLoader.getResourceAsStream('data.csv').text

// src/main/resources/public/test.txtにアクセスする場合
grailsApplication.classLoader.getResourceAsStream('public/test.txt').text
```

なお、`grailsApplication.classLoader.getResourceAsStream`は`InputStream`を返してくれますので、後は煮るなり焼くなり！  


## webappの利用

`src/main/webapp`も利用できます。こちらは最初からディレクトリが用意されていますので特に自分でディレクトリを作る必要はありません。  
Grails2系までは、プロジェクトディレクトリ直下で`web-app`というディレクトリ名でしたが、Grails3からはパスもディレクトリ名も新しくなっていますので、Grails2系からのユーザの方は注意です。  

さて、こちらの`webapp`ディレクトリも、普通に画像などの静的ファイルを保存するだけでOKです。  
URLの仕様は、`recources/public`同様に`/static/`となります。  
例えば、`src/main/webapp/test.jpg`というファイルがある場合、アクセスするためのURLは`/static/test.jpg`となります。  

このURLは`application.yml`で変更できます。  
以下の最後の2行を追加してGrailsを再起動すれば、`/test.jpg`で先ほどのファイルにアクセスできるようになります。  

```yaml
grails:
    profile: web
    codegen:
        defaultPackage: tran
    spring:
        transactionManagement:
            proxies: false
    # 以下の２行を追加
    resources:
        pattern: '/**' # '/hoge/**' とすれば、/hoge/test.jpgになる
```

`resources/public`と違い、`webapp`ディレクトリへのファイルの追加や内容の修正は、 **Grailsの再起動なしにリアルタイムに反映されます。**  

1点注意することがあります。  
それは、冒頭で述べている通り、`src/main/webapp`の方は **WARには含まれるけど、JARに含まれません。**   
この部分はGrailsをデプロイ用にリリースする際にwarを利用する用にすれば回避できます。（Grailsは標準でwarを生成する`war`コマンドがあります）  


## 注意点
さて、Grails3.1以降、`webapp`も`resources/public`も、同じURLを利用するようになりました。  

つまり、`webapp`と`resources/public`では **URLが被る可能性があります**  
もしURLがかぶった場合は、 **webappディレクトリの方が優先されます。**  

上記の`resource/public`と`webapp`の利用方法を見ていただければ解ると思いますが、 **URLの変更（デフォルトでは`/static/`）は、`resources/public`と`webapp`共に同じ設定を共有しています。**  

`resoruces/`ディレクトリは、恐らくGrails起動時に一旦すべてのファイルを走査していて、`webapp`の方は走査していないようなので、大量のファイルがある場合は、`webapp`を利用するほうが、メモリの節約や起動時の高速化が測れるかもしれません（未確認）  

一応、公式ドキュメントの、[Grails2.xからGrails3へのアップグレードに関する部分で、webappはオススメしないとあります。](http://docs.grails.org/latest/guide/single.html#upgradingApps)(Step 7 - Migrate Static Assets not handled by Asset Pipelineの部分)  


```
メモ：
Grails3.0.xでは、`webapp`のみが`/static/`というURLにマッピングされていた模様。
Grails3.1/Grails3.0.13から、`resources/public`も、`/static/`というURLにマッピングされ始めた。
Grails3.0の時点でも、application.ymlに設定を記述すれば、`webapp`のURLも同様に変更できる。
```


## どういう時に利用するのか

これはもうアプリケーションによりけりです。  
私が運用しているGrails製のWebアプリケーションの場合は、まずNginxがリクエスを受け付けていて、基本的にはそのまま裏で動いているGrails（Tomcat)に処理を流しています。  
しかし、アプリケーションには管理者がブラウザから画像をアップロードする機能が有ります。  
そのため、そのGrails(Tomcat)アプリケーション経由でアップロードされるファイルは、Nginxがアクセスできる場所にアップロードするようにしておき、Nginx側の設定で、`/images`というURLへのアクセスは、Nginxに処理させるようにしています。  
これは、 **warに固めた`assets/images`に直接ファイルを追加する方法はないためです。**  

例えば、Grails(Tomcat）はアップロードされた画像を`/usr/share/nginx/my-application/images/`配下に保存するようにして、Nginxの設定を以下のようにします。  

```nginx
location /images {
    alias /usr/share/nginx/my-application/images;
}
```

これで、HTML上の`/images/hoge.jpg`は、Grails（Tomcat）に処理は来ずに、Nginxの時点で画像が返されます。  
そして問題になるのが開発機ですね。  
Grailsコマンド一発で全ての環境が整うのは魅力的ですが、この画像の取り扱いが難しい。  
管理者がブラウザからアップロードする画像は当然`assets/images/`に格納されていません。  

ココでやっと今回の`resources/public`、もしくは`webapp`ディレクトリの登場ですね。  
単純にサーバ上の`images`ディレクトリをこの2つのどちらかにコピーしてきて、application.ymlで`/static`無しでアクセスできるようにしてあげれば、開発環境でも本番サーバ同様に画像にアクセスすることが出来るようになります。  

本番サーバではNginxが画像を返し、開発環境ではGrailsが`resources/public`、`webapp`に有る画像を返す、という流れです。  

注意する点としては、このまま開発機で`grails war`を実行すると、本番サーバでは必要ない画像ファイルが一緒にwarに含まれてしまう点のみです。  
ビルドサーバやら本番サーバ上でwarに固めることで回避できます。  



## まとめ
いかがでしたでしょうか。  
どういう用途で利用するかでどちらのディレクトリ（`resources/public` or `webapp`）を利用するかは変わってくると思います。  
基本的には、まずは`resources/public`を利用してみて、速度面などでもし`webapp`の方が明らかに早いのであれば`webapp`の利用を検討する、という形がベターかなと思います。  


## 参考
[8.2.5 Static Resources](http://docs.grails.org/latest/guide/single.html#resources)  
[Grails 3.1 リリース!!](http://d.hatena.ne.jp/mottsnite/20160129/1454032206)
