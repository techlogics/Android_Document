

*環境*

* Mac 	OSX 10.10.1

* Vagrant 1.6.3

##はじめに

このドキュメントはwebAPIを用いて、androdアプリを作成するための基本をまとめたものです。

このドキュメントで作るアプリは
**「偉人の名言をタイマーで告知する」 **
というシンプルなアプリです。

今回のこのドキュメントでは以下の順番でアプリを作成していきます。

1. nodejsの環境構築
2. expressを用いたwebAPIの構築
3. androidのコーディング

##1. nodejsの環境構築
####1.0 Vagrant ??
VirtualBox・Vagrant
について少し勉強しておきましょう。

以下にリンクを張っておきます。

僕が書くよりもわかりやすくて、よっぽどわかりやすいでしょう！

* [「Vagrant」って何ぞ？（・o・）](http://www.atmarkit.co.jp/ait/articles/1307/22/news076.html)
* [ドットインストール Vagrant入門](http://dotinstall.com/lessons/basic_vagrant)
* [Vagrant 日本語ドキュメント](http://lab.raqda.com/vagrant/)
* [Vagrant Docs](https://docs.vagrantup.com/v2/)
* [Qiita Vagrant入門](http://qiita.com/kidachi_/items/e63c1607705178aa257c)

####1.1 vagrant・virtualboxのインストール ※1

まずはVagrantをインストールしましょう。

<http://www.vagrantup.com/downloads.html>

VagrantはVirtualBoxを利用しているので、VirtualBoxもインストールしましょう。

<https://www.virtualbox.org/wiki/Downloads>


※1 これを書いてる時のバージョンは

* Vagrant 1.6.3

* VirtualBox 4.8.14
です

####1.2 Vagrantを使って Node.ja,MongoDB の環境を作ってみよう
まずはVagrantを設置するディレクトリを作成します

	$ cd
	~$mkdir vagrant && cd vagrant

次に有志の方が用意してくれたBoxファイルをローカルに持ってきます。

このBoxははじめから Nodejs MongoDB redis が動いています。なのでnodeプログラムをコーディングしてすぐに実行することができます。

	$ clone https://github.com/joaquimserafim/vagrant-nodejs-redis-mongodb

そしたらVagrantfileを見てみましょう。面白いのはこの一行です。

	....

	config.vm.synced_folder "~/Projects", "/vagrant"
	
	....
	
これは、ホスト(Mac)の~/Projects と ゲスト(ubuntu)の/vagrant を同期させるという設定です。
Boxを立ち上げる前に、共有フォルダを作成しておきましょう。

	$mkdir ~/Projects

それでは仮想マシンを立ち上げてみましょう。

	$vagrant up
	...
	...
	...
	==> default: Machine booted and ready!
	==> default: Checking for host entries
	$

これで立ち上がりました。この立ち上げたマシンに接続してみましょう。

	$vagrant ssh
	Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-40-generic x86_64)
	...
	...
	...
	vagrant@vagrant-ubuntu-trusty-64:~$
	
つながりました！
vagrant ssh でリモートのサーバーにsshで繋いでいるのと、同じように操作することができます。
nodeのバージョンを見てみましょう
	
	$ node -v
	v0.10.33
	
mongoは動いてるのか？？
	
	$ mongo
	MongoDB shell version: 2.6.6
	...
	...
	>
動いてますね〜

実際に簡単なnodeプログラムを実行してみましょう。
	
	// ~/Projects/sample.js

	var http = require('http');
	http.createServer(function (req, res) {
	  res.writeHead(200, {'Content-Type': 'text/plain'});
	  res.end('Hello World\n');	
	}).listen(5000);
	console.log('start');

上記のファイルを~/Projects/sample.jsとして保存しましょう。保存したファイルを確認してみましょう。まずはホスト(Mac)側

	$cd ~/Projects
	~/Projects$ ls
	sample.js
	
次はゲスト(ubuntu)
	
	$cd /vagrant
	/vagrant$ ls
	sample.js
	
ちゃんと同期できていますね。
そしたらゲスト(ubuntu)の方で先ほど作ったプログラムを実行してみます。

	/vagrant$ node sample.js
	start!

これで実行されました。

ブラウザで <http://192.168.33.10:5000/> にアクセスしてください。

	Hello World!

と表示されたら成功です！

このようにホストの ~/Project/ 以下にnodeファイルを保存して、ゲストの /vagrant/ 以下でプログラムを実行してテストする。というのが今回の開発の流れです。

Vagrantを使うと、使いたい環境に合ったBoxをダウンロードしてきてたちあげるだけですぐに開発を始めることができます。

さらにチームで開発するときは、今やったようにチームで同じイメージを使うことで環境の違いを吸収してくれるというわけですね♪

次の項からは実際に「役に立つ」プログラムを作っていきたいと思います。


##2. Expressを用いたwebAPIの構築

この項での最終目的はMongoに入っている名言集を取得することです。

[DEMO](http://)

####2.0.1 Expressとは？
expressとはnodejsのwebフレームワークです。単純な処理をサクッと書くことができます。
本来ならばMVCフレームワークとして紹介したいのですが、今回はAPIとして利用するので、ビューの説明は省きます。

仕組みがシンプルなので、触っていじってみて覚えたほうが早いかと思います。

[ドットインストール Express入門](http://dotinstall.com/lessons/basic_expressjs)

####2.0.2 MongoDBとは？
MongoDBとは、MySQLやpostgreのようなRDBMSではなく、NoSQLというデータベースの一つです。
「ドキュメント」と呼ばれる構造的データをJSONライクな形式で表現し、そのドキュメントの集合を「コレクション」として管理すします。

データ構造がJSONライクなので直感的にわかりやすく、またORMとも非常に相性が良いのが特徴です。

[MongoDBの薄い本](http://www.cuspy.org/diary/2012-04-17/the-little-mongodb-book-ja.pdf)

[node.js から MongoDB にアクセス (Mongoose の紹介)](http://krdlab.hatenablog.com/entry/20110317/1300367785)

####2.1 簡単なExpressアプリ

ゲストで express-generator をインストールします。
express-generatorとはExpressのテンプレートを生成するツールです。

	$sudo npm install -g express-generator
	
npm は Node.js のパッケージマネージャーです。

普段 npm を使ってパッケージを入れるときはプロジェクトごとに(インストールしたパッケージはプロジェクトディレクトリの直下においてある node_modules に入る)パッケージを管理するのですが、-g オプションをつけるとグローバルな領域にインストールします。

	$express great_phrase_server
	   create : great_phrase_server
	   create : great_phrase_server/package.json
	   create : great_phrase_server/app.js
	   create : great_phrase_server/public
	   create : great_phrase_server/public/images
	   create : great_phrase_server/routes
	   create : great_phrase_server/routes/index.js
	   create : great_phrase_server/routes/users.js
	   create : great_phrase_server/views
	   create : great_phrase_server/views/index.jade
	   create : great_phrase_server/views/layout.jade
	   create : great_phrase_server/views/error.jade
	   create : great_phrase_server/public/javascripts
	   create : great_phrase_server/public/stylesheets
	   create : great_phrase_server/public/stylesheets/style.css
	   create : great_phrase_server/bin
	   create : great_phrase_server/bin/www

	   install dependencies:
	     $ cd great_phrase_server && npm install

	   run the app:
		     $ DEBUG=great_phrase_server ./bin/www
		     
	$

これでひな形が出来ました。
プロジェクトフォルダに移動しましょう

	$cd great_phrase_server

これからは プロジェクトフォルダ(/great_phrase_server)直下にいるとして説明します。


まず実行するために、依存するパッケージをインストールしましょう。

	$npm install
	
これでパッケージをインストール出来ました。
npm install これは何をしているのかというと
package.jsonを見に行って、インストールされていないパッケージをインストールしてます。

実行してみましょう。

	$npm start
	
そしたらブラウザで見てみましょう。

<http://localhost:3000/>

	Welcome to Express

成功です！

####2.2 プリセットを用意する

予め利用する「名言」を用意してDBに入れてみたいと思います。

MongoDB ではデータを入れるために mongoimport というコマンドがあるのですが、データ形式が独自(json風)で使いづらいので、はじめにDBを初期するコードを書こうと思います。

流し込むファイルを用意します。
以下からダウンロードしてきてプロジェクトフォルダに移してください。
<phraseList.json>

早速コーディングに入ります。
コーディングする際はホストの ~/Pojects/great_phrase_server/ をお好きなエディタで編集してください。(ちなみに私のおすすめは JetBrains の WebStorm です。学生の皆さん〜。無料なので使ってみてください！)

#####まずはホスト(Mac)側

	// insert.js
	// DB初期化スクリプト
	
	var mongoose = require('mongoose');
	
	//DBへ接続
	var db = mongoose.createConnection("mongodb://localhost/greate_phrase", function(error, res){
			if (error)
				console.log(error);
		});
	
	//スキーマ宣言
	var ProverbSchema = mongoose.Schema({
    	parson : { type : String, required: true },
    	phrase : { type : String, required: true },
    	url : { type : String, required: true }
	});
	
	//スキーマ保存
	var Proverb = db.model("Proverb", ProverbSchema);
	
	//何回もこのスクリプトを実行すると、ドキュメントが重複してちまうので、
	//実行するたびに Proverb コレクションを全削除
	Proverb.remove({}, function (err) {
        if(err)console.log(err);
    });
    
    //json読み込み
    var json = require('./phraseList.json');
    
    //保存
    json.forEach(function (one) {
        var proverb = new Proverb();
        proverb.phrase = one.phrase;
        proverb.parson = one.parson;
        proverb.url = one.url;
        proverb.save(function(err){
            if(err){
                console.log(err);
            }
        });
    });
    console.log('SUCCESS!');
    
このコードを insert.js として phraseList.json と同じディレクトリに保存してください。

requireでjsonファイルを指定すると勝手にパースしてくれます。

	var json = require('./phraseList.json');
    
便利便利


mongooseの使い方は以下がわかりやすいです。


[node.js から MongoDB にアクセス (Mongoose の紹介)](http://krdlab.hatenablog.com/entry/20110317/1300367785)



#####次はゲスト(ubuntu)側

実行してみましょう。

	$node insert.js
	SUCCESS!
	
SUCCESS! が表示されたら成功です。





	
    
####データの確認
今 入れたデータの確認をします。
mongo コマンドでコンソールに入ります。

	$mongo
	MongoDB shell version: 2.6.6
	connecting to: test
	>
データベース 一覧を見てみましょう

	> show dbs
	admin        (empty)
	greate_phrase  0.078GB
	local        0.078GB
	>

しっかり出来てますね。次に コレクション(collection) 一覧

	> use  greate_phrase
	switched to db greate_phrase
	> show collections
	proverbs
	system.indexes
	
次に ドキュメント(documents)一覧

	> db.proverbs.find({})
	{ "_id" : ObjectId("5491992e96440655366a69c3"), "url" : "http://ja.wikipedia.org/wiki/%E3%82%B9%E3%83%86%E3%82%A3%E3%83%BC%E3%83%96%E3%83%BB%E3%82%B8%E3%83%A7%E3%83%96%E3%82%BA", "parson" : "スティーブ・ジョブズ", "phrase" : "墓場で一番の金持ちになることは私には重要ではない。夜眠るとき、我々は素晴らしいことをしたと言えること、それが重要だ。", "__v" : 0 }
	...
	...
	..
	.
	Type "it" for more
	>

しっかり入ってますね！
それではコンソールとさよならしましょう。

	> exit
	bye
	
プリセットの用意が終わりました。
次は今準備した「名言」を取り出すAPIを実装していきたいと思います。

####2.3 実装

まずは models フォルダを作成しましょう。

	$mkdir models
	
この中にスキーマを保存していくことになります。

#####mongooseの基本
まず

	//index.js
	
	var mongoose = require('mongoose');

	var db = mongoose.createConnection("mongodb://localhost/greatphrase",
	    function (error, res){
	        if (error) {
	            console.log(error);
	        }
	    });

	module.exports = db;
	
これを models/index.js として保存しましょう。
次に

	//proverb.js
	
	var db = require('./index.js');
	var mongoose = require('mongoose');

	var ProverbSchema = mongoose.Schema({
	    parson : { type : String, required: true },
	    phrase : { type : String, required: true },
	    url : { type : String, required: true }
	});

	db.model("Proverb", ProverbSchema);

	module.exports = db.model('Proverb');
	
これを models/proverb.js として保存しましょう。

これで Proverb モデルの出来上がりです。
このようにしてできたモデルは、

	var Proverb = require('./path/to/models/proverb);
	
	//全件検索
	Proverb.find({}, function(error, docs) {
		if (error) {
			console.log(error);
		}
	});
	
	// index が 2 のドキュメント
	// select * from Proverb where index = 2
	Proverb.find({ index : 2 }, function(error, docs) {
		if (error) {
			console.log(error);
		}
		console.log(docs);
	});
	
	//indexが 2 と 5 のドキュメント
	//select * from Proverb where index in ( 2, 5 );
	Proverb.find({ $or : [
		{ index : 2 },{ index : 5 }
	],function(error, docs) {
		if (error) {
			console.log(error);
		}
		console.log(docs);
	});
	
	//indexが2かつ、nameがhoge
	//select * from Proverb where index = 2 and name = "hoge";
	Proverb.find({ $and : [
		{ index : 2 },{ name : "hoge" }
	],function(error, docs) {
		if (error) {
			console.log(error);
		}
	});
	
	//100件目から130件目までdateについて降順で
	//select * from Proverb limit 30 offset 100 order by desc;
	//.sort({ date : 1 })で昇順
	Proverb.find({})
		.skip(100)
		.limit(30)
		.sort({ date : -1 })
		.exec(function (error, docs) {
			if (error) {
				console.log(error);
			}
			console.log(docs);
		}
	);
	
	//like検索 正規表現も入れることができます。
	//	select * from object where a like %query%
    //                    or b like %query%
    //                    or c like %query%;
	Proverb.find( { $or:[
    {a: new RegExp( ".*"+query+".*")},
    {b: new RegExp( ".*"+query+".*")},
    {c: new RegExp( ".*"+query+".*")}
    ] } function(err, docs){
        if(err)console.log(err);
        res.json(docs);
    });

のように使います。

#####ルーティング

次にroutesフォルダを作成しましょう。

	$cd ../
	$mkdir routes
	
そしてこのファイルを作成しましょう。

	// routes/proverb.js
	    var Proverb = require('../models/proverb');

    module.exports.find = function (req, res) {
        if(req.param('query') === undefined) {
        
            Proverb.find({}, function (err, docs) {
                res.json(docs);
            });
        
        } else {
        
            var param = req.param('query');
            
            var query = new RegExp( ".*"+param+".*");
            Proverb.find({ $or: [
                { parson : query },
                { phrase : query },
                { url : query }
            ]}, function (err, docs) {
                res.json(docs);
            });
            
        }
    };

次にプロジェクト直下のapp.jsを編集しましょう。付け足すのはこの3行です。

	//app.js
	
	...
	
	var proverb = require('./routes/proverb');
	
	...
	
	app.get('/proverb', proverb.find);
	app.get('/proverb/:query', proverb.find);
	
	...
	
app.get('/from', to) という感じにルーティングします。
この例で言うと http://あなた/proverb　にアクセスすると proverb.findが実行されます。

/proverb/:query の :query のように「:」で始まるのはパラメータになります。

	var param = req.param('query');

のように取得します。

http://あなた/proverb/hogehoge とアクセスをすると

	var param = req.param('query');
	console.log(param); // hogehoge
	
となります。

では実際に実行してみましょう。ゲストで実行し
	
	$npm start
	
	> GreatPhrase@0.0.1 start /vagrant/GreatPhrase_server
	> node ./bin/www
	
<http://localhost:3000/proverb/>へアクセス！

うまく取れましたか？

<http://localhost:3000/proverb/スティーブ・ジョブス>へアクセス！

ジョブス！！！

シンプルなAPIはすぐ出来ましたね。Express + Mongoose はAPIをすぐに作るのに最適ではないかと思います。



