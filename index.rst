.. PHP 5.4 Advent Calendar - Day 3 documentation master file, created by
   sphinx-quickstart on Wed Nov 30 21:46:39 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

===============================
PHP 5.4 Advent Calendar - 3日目
===============================

:Author: Shogo Kawahara <kawahara@bucyou.net> Twitter: `@ooharabucyou`_
:Date: 2011-12-03
:License: `Creative Commons Attribution 3.0 Unported License <http://creativecommons.org/licenses/by/3.0/>`_


.. _`@ooharabucyou`: http://twitter.com/ooharabucyou

`<-2日目`_ **3日目(今ここ)** 4日目->

.. _`<-2日目`: http://blog.ohgaki.net/php-5-4-arrays
.. _`4日目->`: http://example.com

概要
====

`PHP5.4 Advent Calendar`_ ということで、おそらくほかデキル方々は目玉機能である、
[] による配列表現や、Build-in Http Server、Traint などを取り扱う
と思うので、私は地味に PHP5.4 のチマい関数仕様変更を取り上げたいと思います。

.. _`PHP5.4 Advent Calendar`: http://atnd.org/events/22473

関数増減(?)
===========

コンパイルオプションを --prefix 以外何も付けない状態で、PHP5.3.8, PHP5.4RC2
関数の差分を見て見ました。

Mac OS X Lion 上にて、こんな感じで調べて見ました。

::

  $ php538 -r '$funcs = get_defined_functions(); echo implode("\n", $funcs["internal"]);' > php538func
  $ php54 -r 'echo implode("\n", get_defined_functions()["internal"]);' > php54rc2func
  $ diff php538func php54rc2func

`cakephper さんの、ワンライナー`_ を参考にしましたが、PHP5.4 は
変数に代入しなくても、配列にアクセスできる array dereference が使えるようになったので、
ちょっとだけシンプルになりました。

.. _`cakephper さんの、ワンライナー`: http://d.hatena.ne.jp/cakephper/20101213/1292209176

::

  20a21
  > trait_exists
  36a38
  > get_declared_traits
  110a113
  > libxml_set_external_entity_loader
  157d159
  < ob_iconv_handler
  178a181
  > class_uses
  226,228d228
  < session_register
  < session_unregister
  < session_is_registered
  238a239,240
  > session_status
  > session_register_shutdown
  243,281d244
  < sqlite_open
  < sqlite_popen
  < sqlite_close
  < sqlite_query
  < sqlite_exec
  < sqlite_array_query
  < sqlite_single_query
  < sqlite_fetch_array
  < sqlite_fetch_object
  < sqlite_fetch_single
  < sqlite_fetch_string
  < sqlite_fetch_all
  < sqlite_current
  < sqlite_column
  < sqlite_libversion
  < sqlite_libencoding
  < sqlite_changes
  < sqlite_last_insert_rowid
  < sqlite_num_rows
  < sqlite_num_fields
  < sqlite_field_name
  < sqlite_seek
  < sqlite_rewind
  < sqlite_next
  < sqlite_prev
  < sqlite_valid
  < sqlite_has_more
  < sqlite_has_prev
  < sqlite_escape_string
  < sqlite_busy_timeout
  < sqlite_last_error
  < sqlite_error_string
  < sqlite_unbuffered_query
  < sqlite_create_aggregate
  < sqlite_create_function
  < sqlite_factory
  < sqlite_udf_encode_binary
  < sqlite_udf_decode_binary
  < sqlite_fetch_column_types
  283a247
  > hex2bin
  303a268
  > getimagesizefromstring
  487a453
  > header_register_callback
  493d458
  < import_request_variables
  530a496
  > http_response_code
  626a593
  > stream_set_chunk_size
  655d621
  < chroot
  698d663
  < define_syslog_variables

なるほどなるほど。PHP5.3時点で、非推奨とされていた関数が
いくつか退場しているように見えます。


おもな退場関数 ::

  ob_iconv_handler
  session_register
  session_unregister
  session_is_registered
  import_request_variables
  define_syslog_variables

ext/sqlite は、 PECL に移動したらしいのでPHP単体のデフォルトのビルドでは
sqlite関数が退場。

.. note::

  なぜか、chroot() が退場した…。
  多分、環境がよくない。

そして、PHP5.4 には新たに、以下の関数が入場したようです。いらっしゃいませ。 ::

  trait_exists
  get_declared_traits
  libxml_set_external_entity_loader
  class_uses
  session_status
  session_register_shutdown
  hex2bin
  getimagesizefromstring
  header_register_callback
  http_response_code
  stream_set_chunk_size

幾つか、面白い使い方ができそうな関数に焦点を当てると共に、
仕様変更のあった関数も紹介します。

[New] http_response_code()
==========================

発行予定の Http Response Code の設定、取得ができます。

404や、その他のレスポンスコードの発行が若干わかりやすくなりますかね。

今まで

.. code-block:: php

  <?php

  header("HTTP/1.0 404 Not Found");

PHP5.4

.. code-block:: php

  <?php

  http_response_code(404);

引数を省略することで、発行予定のレスポンスコードの取得が可能なので、
発行予定のレスポンスコードによって処理を変えるというのが、PHPの機能によって
できるようになりました。

.. code-block:: php

  <?php

  $code = http_response_code();

今まで、送信予定のレスポンスコードについて言語レベルで保証するような
関数はなかったと思うので、なかなか便利な改善だと思います。

`http_response_code ドキュメント <http://php.net/manual/ja/function.http-response-code.php>`_

[New] header_register_callback()
================================

ヘッダ送信直前に行う動作についての関数を登録できるようになりました。

`http_register_callback ドキュメント <http://php.net/manual/ja/function.header-register-callback.php>`_

debug_backtrace()
=================

``debug_backtrace()`` に仕様変更がありました。第2引数として、取得するスタックフレーム数の制限を行うことができるようになりました。

.. code-block:: php

  <?php

  class Foo
  {
      function a($s)
      {
          $this->b($s);
      }

      function b($s)
      {
          $this->c($s);
      }

      function c($s) {
          $this->d($s);
      }

      function d($s) {
          $this->e($s);
      }

      function e($s) {
          var_dump(debug_backtrace(DEBUG_BACKTRACE_PROVIDE_OBJECT, 2));
      }
  }

  (new Foo())->a('Hello');


出力結果はこうなります。::

  array(2) {
    [0]=>
    array(7) {
      ["file"]=>
      string(31) "/Users/kawahara/dev/doc/foo.php"
      ["line"]=>

    .. 省略 ..

    [1]=>
    array(7) {
      ["file"]=>
      string(31) "/Users/kawahara/dev/doc/foo.php"
      ["line"]=>
      int(17)
      ["function"]=>
      string(1) "d"
      ["class"]=>
      string(3) "Foo"
      ["object"]=>
      object(Foo)#1 (0) {
      }
      ["type"]=>
      string(2) "->"
      ["args"]=>
      array(1) {
        [0]=>
        &string(5) "Hello"
      }
    }
  }

第2引数に指定した数を上限として、バックトレースを出すようになります。

あまりに結果が長くなりそうなときなどに、使えそうです。

JSON
====

JSON拡張モジュールのほうで、エンコード周りで便利な機能追加がありました。

json_encode() オプション
------------------------

いくつかのオプションが追加されました。::

  JSON_PRETTY_PRINT
  JSON_UNESCAPED_SLASHES
  JSON_UNESCAPED_UNICODE

``JSON_PRETTY_PRINT`` は、スペースや改行などを挿入して、可読性の高い JSON を出力するためのオプションです。デバックなどで便利そうですね。

``JSON_UNESCAPED_SLASHES`` は、スラッシュをエスケープしなくなります。

``JSON_UNESCAPED_UNICODE`` は、マルチバイト文字をエスケープしなくなります。

試してみましょう。

.. code-block:: php

  <?php

  $a = [
    'こんにちは',
    'hello/',
  ];

  var_dump(json_encode($a));
  var_dump(json_encode($a, JSON_PRETTY_PRINT));
  var_dump(json_encode($a, JSON_UNESCAPED_SLASHES));
  var_dump(json_encode($a, JSON_UNESCAPED_UNICODE));

結果::

  string(44) "["\u3053\u3093\u306b\u3061\u306f","hello\/"]"
  string(55) "[
      "\u3053\u3093\u306b\u3061\u306f",
      "hello\/"
  ]"
  string(43) "["\u3053\u3093\u306b\u3061\u306f","hello/"]"
  string(29) "["こんにちは","hello\/"]"

JsonSerializable
----------------

``JsonSerializable`` という interface が増えました。

これは、 ``json_encode()`` されたときの挙動を定義できる便利なものです。


.. code-block:: php

  <?php

  class Udon implements JsonSerializable, ArrayAccess
  {
      protected
          $id = null,
          $data = null;

      /**
       * JsonSerializable::jsonSerialize 実装
       */
      public function jsonSerialize()
      {
          // この結果を json_encode する
          return ['id' => $this->id, 'data' => $this->data];
      }

      public function __construct()
      {
          $this->data = [];
      }

      public function setId($id)
      {
          $this->id = $id;
      }

      public function getId()
      {
          return $this->id;
      }

      public function setData(array $data)
      {
          $this->data = $data;
      }

      public function getData()
      {
          return $this->data;
      }

      public function offsetSet($offset, $value)
      {
          $this->data[$offset] = $value;
      }

      public function offsetGet($offset)
      {
          return $this->data[$offset];
      }

      public function offsetExists($offset)
      {
          return isset($this->data[$offset]);
      }

      public function offsetUnset($offset)
      {
          unset($this->data[$offset]);
      }
  }

  $udon = new Udon();
  $udon->setId(1);
  $udon['sanuki']   = '讃岐';
  $udon['inaniwa']  = '稲庭';
  $udon['mizusawa'] = '水沢';
  var_dump(json_encode($udon, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));

結果::

  string(122) "{
    "id": 1,
    "data": {
        "sanuki": "讃岐",
        "inaniwa": "稲庭",
        "mizusawa": "水沢"
    }
  }"


まとめ
======

PHP5.4 以上用のPHPフレームワークとかが出たら、ココらへんの機能が
駆使されるのでしょうか?

ひとまず、みなさんにとって、この文章が気づきになったのであれば
幸いであります。

明日のアドベントカレンダーは `@rsky`_ さんの回です。

今日の時点では、まだ参加枠に空きがあるので、ぜひともご参加ください。

それでは、ごきげんよう。

.. _`@rsky`: http://twitter.com/rsky
