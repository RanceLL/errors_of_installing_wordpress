# errors_of_installing_wordpress
# WSのUbuntuサーバーを使いwordpressサイト構築の際のエラーと対策まとめ


先週、いくつかのチュートリアルを参考にして、AWSのAmazon EC2・Ubuntu・MySQL・Apache2・PHP（あわせて、LAMPともいう）というありふれたアプローチで、WordPressを使いなんとかこのサイトを立ち上げました。

しかし、チュートリアルで全く言及しなかったエラーもたくさん起きたため、何十時間も費やし死ぬほど痛い目に遭いました。

幸い、あちこち検索した結果エラーのすべてを直したが、この経験が皆さんに何らかの役に立つかもしれないと思い、今回、AWSを使いWordPressのサーバを構築するにあたってのエラーとその対策をまとめます。
皆さんのご参考になりますように！

#実装環境
* リモートサーバー：Amazonのインスタンス（AMI）のubuntu 14.04
* ローカルのOS：Mac OS（WindowsでもOK）＊
* ネット環境

＊ WindowsからUbuntuサーバーにSSHで接続するため、SSHクライアントのPuTTYがよく使われますが、筆者の知る限り、PuTTYでのコピペー操作が効かないから、すべてのコマンドラインを手打たないといけません。よって、Mac OSがおすすめです。

#エラーのまとめ

##エラー_1
  sudo apt-get install php5
というコマンドラインを使い、php5などのパッケージをインストールしようとするところ

  perl: warning: Setting locale failed.
  perl: warning: Please check that your locale settings:
     LANGUAGE = "ru_RU.UTF-8",
     LC_ALL = “",
      ・・・・・・・・

が表示されます。
##（分析と）解決策
言語設定についてのエラーが起きますから、ローカル設定を英語に変更します。

  sudo locale-gen en_US en_US.UTF-8
  sudo dpkg-reconfigure locales
  export LC_ALL=en_US.UTF-8
  sudo apt-get install --reinstall language-pack-en-base

もしくは、nanoというエディタを使ってenvironmentファイルに直接的にコードを追加します。

  vi /etc/environment
   (addingCode:   LC_ALL=“en_GB.utf8"   to /etc/environment and
  rebooting.)

  nano /etc/environment
  (addingCode:   LC_ALL=“en_GB.utf8"   to /etc/environment and    rebooting.)


##エラー_2
LAMP環境を無事構築したのに、「ドメイン/phpmyadmin」が開けません（404エラーなど）。
##（分析と）解決策

     sudo sh -c 'echo "Include /etc/phpmyadmin/apache.conf" >> /etc/apache2/apache2.conf' && sudo service apache2 restart


##エラー_3
「ドメイン/phpmyadmin」のページがでたが、ユーザーネームとパスワードがわかりません。
##（分析と）解決策
MySQLをインストールした際のパスワードとユーザーネームそのまま入力すれば結構です。
どうしても無理という場合は、以下のコマンドラインを打てから、「root」（ユーザーネーム）「12345678」（パスワード）でログインしてみてください。

  mysql -u root -p
  CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
  GRANT ALL ON wordpress.* TO 'root'@'localhost' IDENTIFIED BY '12345678'';
  FLUSH PRIVILEGES;
  EXIT;


##エラー_4
wordpressを実装したのに、「ドメイン/wp-admin」ページを開けません。
##（分析と）解決策
「wp-config.php」の設定には問題がある可能性が高い
####データベース情報の更新
「wp-config.php」ファイルを編集するには、まず、既存のデータベースやユーザー情報を間違いないように更新します。

  mysql -u root -p
  CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
  GRANT ALL ON wordpress.* TO 'newuser'@'localhost' IDENTIFIED BY 'password';
  FLUSH PRIVILEGES;
  EXIT;
> 上記のコマンドラインを打ち間違いったら、EXITというコマンドラインでsshに戻れない（もう一度サーバーにコネクトする必要がある）から、気をつけてそのままコピペしてください。
####「wp-config.php」ファイルの編集
nanoというエディタを使い、
nano /var/www/wordpress/wp-config.php
で、「wp-config.php」編集の画面に移ります。
define(‘DB_NAME’)からdefine(‘DB_HOST’)までの内容を以下のように編集します。

######データベース関連内容

  define(‘DB_NAME’, ‘wordpress‘);
  define(‘DB_USER’, ‘newuser‘);
  define(‘DB_HOST’, ‘localhost‘);
######WordpressのAPIキー関連
まず、wordpressの公式で秘密キーをゲットします。
https://api.wordpress.org/secret-key/1.1/salt/

八行目のすべての内容をコピーして、「wp-config.php」の中の
define('AUTH_KEY',...........)からdefine('NONCE_SALT', )までの内容を先コピーした内容に入れ替えることにします。

  define('AUTH_KEY', daskgjlqkasdfasdfasdfwehnoisjd3');
  define('SECURE_AUTH_KEY',  'tasdfasd-qegqwadgasu,D');
  define('LOGGED_IN_KEY',    'asdfqghjtykugfgherteFc');
  define('NONCE_KEY',        '&gqergo0pijhoijhafqwfB');
  define('AUTH_SALT',        '+j12098u8htgnehg408u9X');
  define('SECURE_AUTH_SALT', 'X`nM$+j4{(~t.A'&%T:agh');
  define('NONCE_SALT',     %'&()YG%$&'YHBJJNIJJKHa4?');


##エラー_5
sudo ufw unable
でファイアーウォールを設定したことがあるなら、サイトを開けなくなるかもしれません。
##（分析と）解決策
原因さえ分ければ手軽に解決できます。

  sudo ufw disable


##エラー_6
wordpressでプラグインをインストールしようとする際、hostname/ username/ ftp passwordなどを入力してください、という画面が表示されます。
##（分析と）解決策
データベースフォルダーにアクセスできないのは原因となります。
以下のコマンドラインで解決できるかもしれません。

  sudo chown www-data:www-data -R /var/www/
  sudo chown -R apache:apache /var/www
もしくは、「wp-config.php」ファイル下から二行目のところ（三行目でもよい）で、以下のラインを追加します。

  define('FS_METHOD','direct’);


##エラー_7
wordpressフォルダーはどうバックアップしますか。
##（分析と）解決策
有料プラグインによって実現できるが、wordpressというフォルダーを丸ダウンロードしたほうがよいではと思います。
以下のコマンドラインで実現します。
scp -r ubuntu@ドメイン:/var/www/wordpress /LocalPath
> WARN：リモートサーバーからファイルをダウンロードするには、sshでコマンドラインを打てばエラーが起き、ダウンロードできません！
必ず、ローカルなターミナルでコマンドラインを打ってください。


##エラー_8
wordpressでテーマを設定したのに、ページの表示が狂いまくりです。

##（分析と）解決策
「ドメイン/wp-admin」→「設定」→「一般」で、wordpressアドレスとサイトアドレス をドメインからサイトのURLに置き換えてみましょう！

上記以外のエラーやバグもたくさんあるかもしれません。
Wordpressのサイト構築が、少々煩わしいのですが、あきらめないように！
