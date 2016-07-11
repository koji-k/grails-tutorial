# ビューの作成
MVCでいうV(ビュー)です。
さて、先ほどControllerを作成しました。  
では引き続きViewを作成してみましょう。  

`grails-app/views`というディレクトリを見てましょう。  
先ほど作成したControllerと同じ名前の`hello`というディレクトリが有りますね。  
先ほど実行した`create-controller`コマンドは、Controllerを作成すると同時に、同名のディレクトリをviewsの中に自動で作成してくれます｡  
そしてGrailsは、ルールとして URLは**・Controller名/Action名** になっています。  
そして、Actionの中でViewを直接指定する、もしくは何も指定しない場合は、このルールに則って、`grails-app/views/{Controller名}/{Action名}.gsp`というファイルが表示されます。  
それでは、今回は`helloコントローラ`の、`with-viewアクション`を、Viewで表示してみましょう。  

まず、`grails-app/views/hello`ディレクトリに、`withView.gsp`というファイルを作成します。  
拡張子は`gsp`です。これはGroovy Server Pagesの略です。様々な便利な拡張機能が用意されています。  
では、中身を以下のようにしてみましょう。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>This is View</title>
</head>
<body>
<p>This is View! nice GSP!!</p>
</body>
</html>
```

では続いて対応するActionを追加しましょう。

```groovy
package hello_grails

class HelloController {

    def index() {
        render "Hello World!"
    }
    
    // コレを追加
    def withView() {
        // 中身は無くても、Grailsの規約に則って該当するViewが表示される。
    }
}
```

これで完了です。  
では[http://localhost:8080/hello/withView](http://localhost:8080/hello/withView)にアクセスしてみましょう。  

![View](images/view.png)

表示できましたね！
もしアクション名と表示したいGSPファイル名が違う場合は、以下のようにrenderにてGSPファイルの名前を指定することも出来ます。

```groovy
    def withView() {
        render(view: '/hello/withView.gsp')
    }
```