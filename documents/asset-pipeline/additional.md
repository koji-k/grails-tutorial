# ES6とSCSS

G*Advent Calendar(Groovy,Grails,Gradle,Spock...) Advent Calendar 2016 7日目の記事です。  
前日 -> [私です。Asset Pipelineに付いて書きました。](index.html)  
翌日 -> [私です](http://qiita.com/saba1024)  

## ES6(JavaScript2015)

JavaScriptは当然ブラウザ側で実行されるのもなので、別にサーバ側のGrailsがどうこう考えるものではありませんね。  
実際に以下のファイルを`grails-app/assets/javascripts/sample.js`として保存します。  

```javascript
class Person {
    constructor(name) {
        this.name = name
    }

    hello() {
        alert("Hello! " + this.name);
    }
}

const obj = new Person('Grails');
obj.hello();
```

このJavaScriptを読み込むGSPを用意してアクセスした場合、ブラウザに返されるJavaScriptも当然上記と同じものになります。  
ただ、やはり古いブラウザやブラウザ間で挙動に差異が合って怖い、なのでまだES6は使えないという環境もあるかと思います。  
Grailsに標準で搭載されているAsset Pipelineプラグインはそんな問題も解決してくれます！  

では、実際に見てみます。  

上記の`sample.js`というファイル名を`sample.es6`にリネームします。  
これだけで、 **Asset Pipelineが自動的にES6で記述されたJavaScriptをES5形式に変換してくれます。**  

このファイルを利用するには、例えば`assets/javascripts/application.js`の中に、  

```css
//= require sample
```

という1行を追加するか、GSPのheadタグの中で直接、  

```html
<asset:javascript src="sample" />
```

と指定するだけです。  
もし拡張子まで指定する場合は、`.es6`のままでも`.js`でもどちらでもOKです。  
（`application.js`の中に`require_tree .`という記述が有る場合には、GSPの中で`application.js`を読み込むだけでOKです。）  

この`sample.es6`は以下のようなJavaScriptに変換されてブラウザに渡されます。  

```javascript
/** @struct @constructor */ var Person = function(name) {
  this.name = name;
};
Person.prototype.hello = function() {
  alert("Hello! " + this.name);
};
/** @const */ var obj = new Person("Grails");
obj.hello();
```

見事にES5に変換してくれていますね！  

ここでさらに嬉しいニュースです。  
例えばES5には無い、定数を扱う`const`ですが、開発段階でこれが上記のJavaScriptの結果のように、`var`に変換されると話にならないですよね。書いてる意味がない。  
でも！Asset Pipelineなら大丈夫です！  
ES6形式（拡張子が.es6）のファイルで、ES6的にありえないコードを書いた場合には、 **ES5にトランスパイルされずES6のままになります。**  
ためしに`sample.js`を以下のように修正してみます。  

```javascript
class Person {
    constructor(name) {
        this.name = name
    }

    hello() {
        alert("Hello! " + this.name);
    }
}

const obj = new Person('Grails');
obj.hello();
obj = 1; // constの値に再代入！これは本来許されない！
alert(obj);
```

`const`で宣言した変数に別の値を再代入していますね。ES6ではこれは許されません、が、もしブラウザに返されるJavaScriptがES5に変換されると、`const`は`var`に置き換えられてしまいます。つまりブラウザ上では普通に動作してしまうことになります。  
ただし、既に述べたようにES6としておかしなコードはES5に変換されない（トランスパイル出来ない）ので、結果的に **ブラウザ側でES6のまま実行されて、constに再代入できない、というエラーがちゃんと発生します。**  
実際にブラウザに返されるコードは  

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  hello() {
    alert("Hello! " + this.name);
  }
}
const obj = new Person("Grails");
obj.hello();
obj = 1;
alert(obj);
```

となっています。実際のソースと変化ありませんね！  
（ちなみにChromeだと`sample.js?compile=false:11 Uncaught TypeError: Assignment to constant variable.(…)`とうエラーがJavaScriptコンソールに表示されます）  

Grailsで開発を進めるという事は、当然常にGrailsが起動しているわけで、その **Grailsのプラグインとして標準で搭載されているAsset PipelineがES6をES5に自動で トランスパイルしてくれる** ので、別途JavaScriptのトランスパイラなどを用意する必要はありません。  
これは非常に便利ですね！  

## SASS/SCSS

さて、CSSを書くとなるとやっぱりSASS/SCSSで書きたくなりますよね！  
Grailsに標準で搭載されているAsset Pipelineでは、さらに専用のSASS/SCSSプラグインも利用できるようになっています。  
利用方法は至って簡単です。  
`build.gradle`の中の`dependencies`に１行プラグインに関する記述を追加するだけです。  

```
dependencies {
    compile "org.springframework.boot:spring-boot-starter-logging"
    compile "org.springframework.boot:spring-boot-autoconfigure"
    compile "org.grails:grails-core"
    compile "org.springframework.boot:spring-boot-starter-actuator"
    compile "org.springframework.boot:spring-boot-starter-tomcat"
    compile "org.grails:grails-dependencies"
    compile "org.grails:grails-web-boot"
    compile "org.grails.plugins:cache"
    compile "org.grails.plugins:scaffolding"
    compile "org.grails.plugins:hibernate5"
    compile "org.hibernate:hibernate-core:5.1.2.Final"
    compile "org.hibernate:hibernate-ehcache:5.1.2.Final"
    console "org.grails:grails-console"
    profile "org.grails.profiles:web"
    runtime "com.bertramlabs.plugins:asset-pipeline-grails:2.11.6"
    runtime "com.h2database:h2"
    testCompile "org.grails:grails-plugin-testing"
    testCompile "org.grails.plugins:geb"
    testRuntime "org.seleniumhq.selenium:selenium-htmlunit-driver:2.47.1"
    testRuntime "net.sourceforge.htmlunit:htmlunit:2.18"

    // これを追加するだけ。
    assets "com.bertramlabs.plugins:sass-asset-pipeline:2.11.6"
}
```

末尾のバージョン名はGrails3に標準で搭載されているAsset Pipelineのものと同じにすればOKです。  
（ちなみにAsset-Pipeline本体は`runtime "com.bertramlabs.plugins:asset-pipeline-grails:2.11.6"`の部分）  

あとは、`assets/stylesheets`の中で自分が作成するCSSのファイルを、拡張子を`.sass`、もしくは`.scss`として保存するだけです！  
`.sass`、`.scss`のCSSへのトランスパイルは全てAsset Pipelineが自動で行ってくれます。  

つまり  
`hoge.scss`

```css
body {
    $myColor: #337ab7;
    
    padding: 10px;
    background-color: #cccccc;
    h1 {
        color: $myColor;
    }
    .content {
        .title {
            font-weight: bold;
        }
        .description {
            font-style: italic;
        }
    }

    /* 画像表示位置の微調整用に以下を追加！ */
    .img-size {
        display: block;
        margin-top: 10px;
        width: 50px;
    }
}
```

というSCSSファイルが合った場合、自動的にCSSへトランスパイルされて、ブラウザには  

```css
@charset "UTF-8";
/* line 1, my-css/my-original.scss */
body {
  padding: 10px;
  background-color: #cccccc;
  /* 画像表示位置の微調整用に以下を追加！ */
}

/* line 6, my-css/my-original.scss */
body h1 {
  color: #337ab7;
}

/* line 10, my-css/my-original.scss */
body .content .title {
  font-weight: bold;
}

/* line 13, my-css/my-original.scss */
body .content .description {
  font-style: italic;
}

/* line 19, my-css/my-original.scss */
body .img-size {
  display: block;
  margin-top: 10px;
  width: 50px;
}
```

というCSSとして返されます。  
このファイルを利用するには、例えば`assets/stylesheets/application.css`の中に、  

```css
*= require hoge
```

という1行を追加するか、GSPのheadタグの中で直接、  

```html
<asset:stylesheet src="hoge" />
```

と指定するだけです。  
もし拡張子まで指定する場合は、`.scss`のままでも`.css`でもどちらでもOKです。  


## まとめ
Grails3では、標準でES6（JavaScript2015）がトランスパイルでき、さらに追加のプラグインでSASS/SCSSをCSSにトランスパイルすることができる事も分かりました。  
Grailsが起動していれば全て自動でトランスパイルが行われるので、別途トランスパイル用のツールを準備、実行する必要はありません。  
さらに、Asset Pipelineによってそれらのファイルも一元管理でき、デプロイ時には一つにまとめてminifyまで全て自動で行ってくれます。  
Grailsには、モダンなWebアプリケーションを構築するために必要な物が全て揃っているだけではなく、それらが高度に統合され、より快適に、便利にWeb開発が出来るようになっています。  

## 参考
[Grailsポータル上のプラグイン情報](http://plugins.grails.org/plugin/asset-pipeline-grails)  
[Github](https://github.com/bertramdev/asset-pipeline)