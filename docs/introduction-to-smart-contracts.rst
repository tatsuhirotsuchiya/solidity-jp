###############################
Introduction to Smart Contractsスマートコントラクト入門
###############################

.. _simple-smart-contract:

***********************
A Simple Smart Contract簡単なスマートコントラクト
***********************

Let us begin with a basic example that sets the value of a variable and exposes
it for other contracts to access. It is fine if you do not understand
everything right now, we will go into more detail later.

まず，変数の値を設定して他のコントラクトからアクセスできるようにそれを
公開する基本的な例から始めましょう．
この時点ではすべてを理解していなくても問題ありません．
追ってより詳細についてみていきます．

Storage
=======

::

    pragma solidity >=0.4.0 <0.6.0;

    contract SimpleStorage {
        uint storedData;

        function set(uint x) public {
            storedData = x;
        }

        function get() public view returns (uint) {
            return storedData;
        }
    }

The first line simply tells that the source code is written for
Solidity version 0.4.0 or anything newer that does not break functionality
(up to, but not including, version 0.6.0). This is to ensure that the
contract is not compilable with a new (breaking) compiler version, where it could behave differently.
So-called pragmas are common instructions for compilers about how to treat the
source code (e.g. `pragma once <https://en.wikipedia.org/wiki/Pragma_once>`_).

最初の行は単にソースコードがSolidityバージョン0.4.0か，機能性を破壊しないより新しい
バージョン(バージョン0.6.0まで，ただしこれ自身は含まない)で書かれていることを述べています．
これは，コントラクトが異なった動作をする可能性がある新しい（破壊的な）コンパイラの
バージョンと互換性がないことをはっきりさせるためです．
いわゆるpragmaとは，コンパイラに対して，ソースコードをどのように扱うかということ
についての一般的な指示のことです（例． `pragma once <https://en.wikipedia.org/wiki/Pragma_once>`_)

A contract in the sense of Solidity is a collection of code (its *functions*) and
data (its *state*) that resides at a specific address on the Ethereum
blockchain. The line ``uint storedData;`` declares a state variable called ``storedData`` of
type ``uint`` (*u*nsigned *int*eger of *256* bits). You can think of it as a single slot
in a database that can be queried and altered by calling functions of the
code that manages the database. In the case of Ethereum, this is always the owning
contract. And in this case, the functions ``set`` and ``get`` can be used to modify
or retrieve the value of the variable.

Solidityの意味でコントラクトとはコードの集まり（その*関数*）と
イーサリアムブロックチェーン上の特定のアドレスに存在するデータ(*状態*)のことです．
行 ``uint storedData;``は状態変数として``uint``型 (*256*ビット*u*nsigned（符号なし）
 *int*eger（整数）)の``storedData`` を宣言しています．
これは，クエリーを投げたりデータベースを管理するコードの関数を呼ぶことで
変化させることができるデータベース上の単一スロットとして考えることができます．
イーサリアムの場合，これはいつもそれを保持するコントラクトによってなされます．
そして，この場合，関数``set``と``get``は変数の値を変更したり，取り出したりするのに
利用できます．

To access a state variable, you do not need the prefix ``this.`` as is common in
other languages.

状態変数にアクセスするのには，他の言語ではよくあるようにプリフェックスとして``this``を
付ける必要はありません．

This contract does not do much yet apart from (due to the infrastructure
built by Ethereum) allowing anyone to store a single number that is accessible by
anyone in the world without a (feasible) way to prevent you from publishing
this number. Of course, anyone could just call ``set`` again with a different value
and overwrite your number, but the number will still be stored in the history
of the blockchain. Later, we will see how you can impose access restrictions
so that only you can alter the number.

このコントラクトはこの値を公示するのを制限する(実現可能な)方法を装備しておらず，
(イーサリアムによって築かれたインフラストラクチャーによって)
世界のだれもがアクセス可能な一つの数をだれもが記憶することができるのと，まだほとんど違いません．
もちろん，だれもが単に``set``を違う値で再度呼び出すことであなたの値を上書きすることが
できますが，その値はブロックチェーンのヒストリ中にまだ記録されることになります．
後程，どのようにアクセス制限を課して，あなた自身しか値を変えることができないように
するかを見ます．

.. note::
    All identifiers (contract names, function names and variable names) are restricted to
    the ASCII character set. It is possible to store UTF-8 encoded data in string variables.

.. note::
    すべての識別子(コントラクト名，関数名，変数名)はASCII文字に制限されています．
    文字列変数にはUTF-8にエンコードされたデータを記憶することが可能です．

.. warning::
    Be careful with using Unicode text, as similar looking (or even identical) characters can
    have different code points and as such will be encoded as a different byte array.

.. warning::
    Unicodeのテキストを使うには注意が必要です．同じような外見の（あるいは意味も）文字でも
    違うコードポイントをもっており，それらは違うバイト配列にエンコードされるためです．

.. index:: ! subcurrency

Subcurrency Example
===================

The following contract will implement the simplest form of a
cryptocurrency. It is possible to generate coins out of thin air, but
only the person that created the contract will be able to do that (it is easy
to implement a different issuance scheme).
Furthermore, anyone can send coins to each other without a need for
registering with username and password — all you need is an Ethereum keypair.

以下のコントラクトは暗号通貨のもっとも簡単な形を実装します．
何もないところからコインを生成することが可能ですが，
それができるのはコントラクトを作成した人物のみです
（違う発行方法を実装することも簡単です）．
さらに，ユーザ名やパスワードを登録する必要なしにだれでもコインを送ることが可能です
- 必要なのはイーサリアムキーペアだけです．

::

    pragma solidity >0.4.99 <0.6.0;

    contract Coin {
        // The keyword "public" makes those variables
        // easily readable from outside.
        address public minter;
        mapping (address => uint) public balances;

        // Events allow light clients to react to
        // changes efficiently.
        event Sent(address from, address to, uint amount);

        // This is the constructor whose code is
        // run only when the contract is created.
        constructor() public {
            minter = msg.sender;
        }

        function mint(address receiver, uint amount) public {
            require(msg.sender == minter);
            require(amount < 1e60);
            balances[receiver] += amount;
        }

        function send(address receiver, uint amount) public {
            require(amount <= balances[msg.sender], "Insufficient balance.");
            balances[msg.sender] -= amount;
            balances[receiver] += amount;
            emit Sent(msg.sender, receiver, amount);
        }
    }

This contract introduces some new concepts, let us go through them one by one.

このコントラクトではいくつかの新しい概念を導入しています．一つ一つ見ていきましょう．

The line ``address public minter;`` declares a state variable of type address
that is publicly accessible. The ``address`` type is a 160-bit value
that does not allow any arithmetic operations. It is suitable for
storing addresses of contracts or of keypairs belonging to external
persons. The keyword ``public`` automatically generates a function that
allows you to access the current value of the state variable
from outside of the contract.
Without this keyword, other contracts have no way to access the variable.
The code of the function generated by the compiler is roughly equivalent
to the following (ignore ``external`` and ``view`` for now)::

``address public minter;`` の行はアドレス型で公にアクセスできる状態変数を
宣言しています．``address'' 型は160ビットの，算術演算ができない値です．
これはコントラクトのアドレスを保存したり外部の人物に属するキーペアを保存するのに
向いています．``public'' というキーワードはコントラクトの外部から
状態変数の現在の値にアクセスすることができる関数を自動的に生成します．
このキーワードが無い場合は，他のコントラクトは変数にアクセスする方法はありません．
コンパイラが生成した関数のコードはほぼ以下と等価です (ひとまず``external`` と``view`` は無視します)::


    function minter() external view returns (address) { return minter; }

Of course, adding a function exactly like that will not work
because we would have a
function and a state variable with the same name, but hopefully, you
get the idea - the compiler figures that out for you.

もちろん，これとまったく同じような関数を付け加えても，
関数とそれと同じ名前の状態変数があるので，うまくはいきませんが，
おそらく，意図は分かっていただけるでしょう．コンパイラはあなたに代わってこれを
やってくれます．

.. index:: mapping

The next line, ``mapping (address => uint) public balances;`` also
creates a public state variable, but it is a more complex datatype.
The type maps addresses to unsigned integers.
Mappings can be seen as `hash tables <https://en.wikipedia.org/wiki/Hash_table>`_ which are
virtually initialized such that every possible key exists from the start and is mapped to a
value whose byte-representation is all zeros. This analogy does not go
too far, though, as it is neither possible to obtain a list of all keys of
a mapping, nor a list of all values. So either keep in mind (or
better, keep a list or use a more advanced data type) what you
added to the mapping or use it in a context where this is not needed.
The :ref:`getter function<getter-functions>` created by the ``public`` keyword
is a bit more complex in this case. It roughly looks like the
following::

function balances(address _account) external view returns (uint) {
    return balances[_account];
}

次の行, ``mapping (address => uint) public balances;`` も
またpublicな状態変数を生成しますが，こちらはもっと複雑なデータ型です．
この型はアドレスを符号なし整数にマップします．
マッピングは `hash tables <https://en.wikipedia.org/wiki/Hash_table>`_
と見なすことができます．これは，
どの可能なキーも最初から存在して，すべて0からなるバイト表現の値にマップされている
といった形に，初期化されます．
このアナロジーは，しかし，それ程よくは当てはまりません，なぜならマッピングのすべてのキーを
得ることも，すべての値の一覧を得ることもできないからです．
なので，マッピングに何を加えたかを覚えておくか（あるいは，一覧を保存しておくか
もっと高度なデータ型を使う方がよいです），そうする必要がない場面で使いましょう．
キーワード``public'' で生成された :ref:`getter function<getter-functions>`
はこの場合少しだけより複雑です．
これは大雑把には以下のようになります::

    function balances(address _account) external view returns (uint) {
        return balances[_account];
    }

As you see, you can use this function to easily query the balance of a
single account.

見ての通り，この関数を用いて用意に一つのアカウントの残額について問い合わせることができます．

.. index:: event

The line ``event Sent(address from, address to, uint amount);`` declares
a so-called "event" which is emitted in the last line of the function
``send``. User interfaces (as well as server applications of course) can
listen for those events being emitted on the blockchain without much
cost. As soon as it is emitted, the listener will also receive the
arguments ``from``, ``to`` and ``amount``, which makes it easy to track
transactions. In order to listen for this event, you would use the following
JavaScript code (which assumes that ``Coin`` is a contract object created via
web3.js or a similar module)::


    Coin.Sent().watch({}, '', function(error, result) {
        if (!error) {
            console.log("Coin transfer: " + result.args.amount +
                " coins were sent from " + result.args.from +
                " to " + result.args.to + ".");
            console.log("Balances now:\n" +
                "Sender: " + Coin.balances.call(result.args.from) +
                "Receiver: " + Coin.balances.call(result.args.to));
        }
    })


``event Sent(address from, address to, uint amount);`` の行は
いわゆる "event" を宣言しており，これは関数
``send`` の最後の行でエミットされています．
ユーザインタフェース (もちろんサーバアプリケーションも)は
ブロックチェーン上にエミットされたこれらのイベントを
多くのコストをかけることなく聞くことができます．
エミットされ次第，リスナーはまた引数 ``from``, ``to``, ``amount``
を受け取り，これによりトランザクションを追跡することが容易になります．
イベントを聞くために，次のJavaScriptのコード (``Coin`` はコントラクトオブジェクトで
web3.js あるいは類似のモジュールを通して生成されたことを仮定)を用いることができます::

    Coin.Sent().watch({}, '', function(error, result) {
        if (!error) {
            console.log("Coin transfer: " + result.args.amount +
                " coins were sent from " + result.args.from +
                " to " + result.args.to + ".");
            console.log("Balances now:\n" +
                "Sender: " + Coin.balances.call(result.args.from) +
                "Receiver: " + Coin.balances.call(result.args.to));
        }
    })


Note how the automatically generated function ``balances`` is called from
the user interface.

どのように自動生成された関数``balances`` がユーザインタフェースから呼び出されているか
に注意しましょう．

.. index:: coin

The constructor is a special function which is run during creation of the contract and
cannot be called afterwards. It permanently stores the address of the person creating the
contract: ``msg`` (together with ``tx`` and ``block``) is a special global variable that
contains some properties which allow access to the blockchain. ``msg.sender`` is
always the address where the current (external) function call came from.

コンストラクタは特別な関数でコントラクトの生成の際に実行され，その後は呼び出す
ことはできません．これはコントラクトを生成した人物のアドレスを永久に保存します.
``msg`` (``tx`` と ``block`` と共に)は特別な大域変数でブロックチェーンに
アクセスするためのいくつかのプロパティを有しています．
``msg.sender`` は常に，現在の（外部の）関数呼び出しが来た元のアドレスです．

Finally, the functions that will actually end up with the contract and can be called
by users and contracts alike are ``mint`` and ``send``.
If ``mint`` is called by anyone except the account that created the contract,
nothing will happen. This is ensured by the special function ``require`` which
causes all changes to be reverted if its argument evaluates to false.
The second call to ``require`` ensures that there will not be too many coins,
which could cause overflow errors later.

最後に，コントラクトを実際に締めくくる，ユーザやコントラクトなどから呼び出すことが
できる関数は，``mint`` と ``send`` です．
もし``mint`` がこのコントラクトを生成したアカウント以外のだれかから呼ばれた場合は，
何もおきません．これは，特別な関数である``require`` によって保証されています．
``require`` はすべての変化を，引数が偽に評価される場合，引き戻しますreverted．
２番目の``require`` の呼び出しは，オーバフローエラーを後で引き起こすかもしれない
コインが多くなりすぎることないのを保証します

On the other hand, ``send`` can be used by anyone (who already
has some of these coins) to send coins to anyone else. If you do not have
enough coins to send, the ``require`` call will fail and also provide the
user with an appropriate error message string.

一方，``send`` は（これらのコインのいくつかを既に所有している）だれもが
使ってコインを別のだれかに送ることができます．もし，送るのに十分なコインを
持っていない場合は，この``require`` への呼び出しは失敗し，それと共に
ユーザに対応するエラーメッセージ文字列が渡されます．

.. note::
    If you use
    this contract to send coins to an address, you will not see anything when you
    look at that address on a blockchain explorer, because the fact that you sent
    coins and the changed balances are only stored in the data storage of this
    particular coin contract. By the use of events it is relatively easy to create
    a "blockchain explorer" that tracks transactions and balances of your new coin,
    but you have to inspect the coin contract address and not the addresses of the
    coin owners.
    もしこのコントラクトをあるアドレスにコインを送るのに使うのなら，
    ブロックチェーンエクスプローラーでそのアドレスを見ても何も見ることができません．
    なぜならコインを送ったという事実と変更された残高はこの特定のコインコントラクト
    のデータストレージに記憶されるだけだからです．イベントを用いることで，
    トランザクションを追跡し新しいコインの残高を追跡できる``ブロックチェーンエクスプローラー``
    を作成することは比較的容易ですが，コインの所持者のアドレスではなく，コインコントラクトのアドレスを
    調べなければなりません．

.. _blockchain-basics:

*****************
Blockchain Basicsブロックチェーンの基礎
*****************

Blockchains as a concept are not too hard to understand for programmers. The reason is that
most of the complications (mining, `hashing <https://en.wikipedia.org/wiki/Cryptographic_hash_function>`_, `elliptic-curve cryptography <https://en.wikipedia.org/wiki/Elliptic_curve_cryptography>`_, `peer-to-peer networks <https://en.wikipedia.org/wiki/Peer-to-peer>`_, etc.)
are just there to provide a certain set of features and promises for the platform. Once you accept these
features as given, you do not have to worry about the underlying technology - or do you have
to know how Amazon's AWS works internally in order to use it?

ブロックチェーンは概念としてはプログラマにとって難しすぎて理解できないということはありません．
理由は，複雑なほとんどのこと（マイニング，`ハッシング <https://en.wikipedia.org/wiki/Cryptographic_hash_function>`_,
`楕円曲線暗号 <https://en.wikipedia.org/wiki/Elliptic_curve_cryptography>`_,
`ピア・ツー・ピアネットワーク <https://en.wikipedia.org/wiki/Peer-to-peer>`_, など)
はプラットフォームのための，ある種の機能と約束を提供するためにあるだけです．
これらの機能を所与のものとして受け入れれば，その背後にあるテクノロジーについて心配する必要はありません -
そうでなければ，アマゾンのAWSが内部的にどう動いているか，それを使うのに知らないといけないでしょうか．

.. index:: transaction

Transactionsトランザクション
============

A blockchain is a globally shared, transactional database.
This means that everyone can read entries in the database just by participating in the network.
If you want to change something in the database, you have to create a so-called transaction
which has to be accepted by all others.
The word transaction implies that the change you want to make (assume you want to change
two values at the same time) is either not done at all or completely applied. Furthermore,
while your transaction is being applied to the database, no other transaction can alter it.

ブロックチェーンは全域的に共有された，トランザクションデータベースです．
このことはだれもがデータベース内のエントリをネットワークに参加するだけで読むことができることを意味しています．
もしデータベース中の何かを変更したければ，他の全員が受け入れないといけない
いわゆるトランザクションを生成する必要があります．
このトランザクションという言葉は，行いたい変更（同時に二つの値を変更したいものと仮定）が
まったく行われないか，完全に適用されるかのいずれかとなることを意味します．
さらに，トランザクションがデータベースに適用されている間は，他のトランザクションは
それを変更することはできません．

As an example, imagine a table that lists the balances of all accounts in an
electronic currency. If a transfer from one account to another is requested,
the transactional nature of the database ensures that if the amount is
subtracted from one account, it is always added to the other account. If due
to whatever reason, adding the amount to the target account is not possible,
the source account is also not modified.

例として，すべての口座の預金高が電子通貨の額で一覧になっている表を想像してください．
ある口座から別の口座へと送金が必要となった場合，データベースのトランザクション面での特性によって，
ある口座から引かれた額は，かならずもう一方の口座へ足されます．もし，何らかの理由にのって，
対象の口座へ足すことが可能でないならば，元の口座も変更されることはありません．

Furthermore, a transaction is always cryptographically signed by the sender (creator).
This makes it straightforward to guard access to specific modifications of the
database. In the example of the electronic currency, a simple check ensures that
only the person holding the keys to the account can transfer money from it.

さらに，トランザクションは常に送り手（作り手）によって電子署名されます．
これにより，データベースの特定の変更をするアクセスを保護することが容易になります．
電子通貨の例では，単純なチェックによって口座に対する鍵をもっている人物だけが，
そこから送金することが可能になります．

.. index:: ! block

Blocksブロック
======

One major obstacle to overcome is what (in Bitcoin terms) is called a "double-spend attack":
What happens if two transactions exist in the network that both want to empty an account?
Only one of the transactions can be valid, typically the one that is accepted first.
The problem is that "first" is not an objective term in a peer-to-peer network.

克服すべき最大の障害のひとつは，（ビットコインの用語で）double-spend 攻撃と呼ばれるものです．
あるアカウントを同時に空にする二つのトランザクションがネットワーク上に存在した場合，何が起こるでしょうか．
一つのトランザクションだけが有効であり，通常は，最初に受理された方がそうなります．
問題は，ピア・ツー・ピアネットワークにおいては「最初」は客観的な用語ではないことです．

The abstract answer to this is that you do not have to care. A globally accepted order of the transactions
will be selected for you, solving the conflict. The transactions will be bundled into what is called a "block"
and then they will be executed and distributed among all participating nodes.
If two transactions contradict each other, the one that ends up being second will
be rejected and not become part of the block.

この問題に対する抽象的な答えは，気にする必要がない，というものです．
トランザクションの大域的な受理順序はやがて決定され，競合が解消されます．
トランザクションは「ブロック」と呼ばれるものに束ねられ，実行されてすべての参加ノードに分散されます．
二つのトランザクションが互いに矛盾する場合，2番目のトランザクションになってしまった方は，
拒否されてブロックの一部とはなりません．

These blocks form a linear sequence in time and that is where the word "blockchain"
derives from. Blocks are added to the chain in rather regular intervals - for
Ethereum this is roughly every 17 seconds.

これらのブロックは時間の経過に従って一つの列を形成し，それが
「ブロックチェーン」という言葉が生じた由来です．
ブロックはチェーンに，大体一定の間隔で追加されます．イーサリアムの場合，これは
ほぼ17秒毎です．

As part of the "order selection mechanism" (which is called "mining") it may happen that
blocks are reverted from time to time, but only at the "tip" of the chain. The more
blocks are added on top of a particular block, the less likely this block will be reverted. So it might be that your transactions
are reverted and even removed from the blockchain, but the longer you wait, the less
likely it will be.

「順番選択機構」（「マイニング」と呼ばれる）の一環で，
ブロックはしばしば取り消されますが，それはチェーンの「先」だけです．
より多くのブロックがある特定のブロックの先に追加されれば，そのブロックが取り消される
可能性はより小さくなります．なので，トランザクションが取り消され，ブロックチェーンから
取り除かれることさえあるかもしれませんが，長く待てば待つほど，その可能性は小さくなります．

.. note::
    Transactions are not guaranteed to be included in the next block or any specific future block,
    since it is not up to the submitter of a transaction, but up to the miners to determine in which block the transaction is included.

    If you want to schedule future calls of your contract, you can use
    the `alarm clock <http://www.ethereum-alarm-clock.com/>`_ or a similar oracle service.

.. note::
    トランザクションが，次のブロックや特定の未来のブロックに含まれることは，保証されません．
    なぜなら，トランザクションを投入した人ではなく，そのトランザクションがどのブロックに
    入るのかを決めるのは，マイナーにかかっているからです．

    もしコントラクトの将来の？？？callsをスケジュールしたいのなら，
     `alarm clock <http://www.ethereum-alarm-clock.com/>`_ や類似のオラクルサービスが利用できます．


.. _the-ethereum-virtual-machine:

.. index:: !evm, ! ethereum virtual machine

****************************
The Ethereum Virtual Machine イーサリアム仮想機械
****************************

Overview 概要
========

The Ethereum Virtual Machine or EVM is the runtime environment
for smart contracts in Ethereum. It is not only sandboxed but
actually completely isolated, which means that code running
inside the EVM has no access to network, filesystem or other processes.
Smart contracts even have limited access to other smart contracts.


イーサリアム仮想機械 EVMはイーサリアム上のスマートコントラクトのための
ランタイム環境です．これはサンドボックス化されているだけでなく
実際に完全に隔離されたものです．というのは，EVMの内で実行しているコードは，
ネットワークやファイルシステムや他のプロセスに，アクセスしません．
スマートコントラクトは，他のスマートコントラクトに対しても制限されたアクセス
しかできません．

.. index:: ! account, address, storage, balance

Accounts 口座
========

There are two kinds of accounts in Ethereum which share the same
address space: **External accounts** that are controlled by
public-private key pairs (i.e. humans) and **contract accounts** which are
controlled by the code stored together with the account.

イーサリアムでは2種類の口座があり，同じアドレス空間を共有しています．
**外部アカウント**は，公開鍵と秘密鍵の組み合わせによって（つまり人間によって）
制御され，**コントラクトアカウント**はその口座と一緒に保存されている
コードによって制御されます．

The address of an external account is determined from
the public key while the address of a contract is
determined at the time the contract is created
(it is derived from the creator address and the number
of transactions sent from that address, the so-called "nonce").

外部アカウントのアドレスは公開鍵によって決定されますが，コントラクトのアドレスは
コントラクトが生成された時点で決定されます．
（作成者のアドレスと，そのアドレスから送信されたトランザクションの数，いわゆる
「ナンス」から導出されます．）

Regardless of whether or not the account stores code, the two types are
treated equally by the EVM.

アカウントがコードを保存しているかどうかにかかわらず，これら二種類は
EVMによっては同等に取り扱われます．

Every account has a persistent key-value store mapping 256-bit words to 256-bit
words called **storage**.

すべてのアカウントは，永続的なキーバリューストアを有しており，これは256ビットのワードを
256ビットのワードに写像し，**ストレージ**と呼ばれます．

Furthermore, every account has a **balance** in
Ether (in "Wei" to be exact, `1 ether` is `10**18 wei`) which can be modified by sending transactions that
include Ether.

さらに，すべてのアカウントはイーサの単位で**残高**を有しており
（正確には「ウェイ」で，`1 イーサ`は`10**18 wei`），
イーサを含むトランザクションを送信することで変更することが可能です．

.. index:: ! transaction

Transactions トランザクション
============

A transaction is a message that is sent from one account to another
account (which might be the same or empty, see below).
It can include binary data (which is called "payload") and Ether.

トランザクションは，あるアカウントから別のアカウントへと送られるメッセージです．
（同じであたり空であったりするかもしれません，下を参照．）
トランザクションはバイナリデータ（「ペイロード」と呼ばれます）やイーサを
含むことができます．

If the target account contains code, that code is executed and
the payload is provided as input data.

対象となるアカウントがコードを含むのなら，そのコードが実行され
ペイロードは入力として渡されます．

If the target account is not set (the transaction does not have
a recipient or the recipient is set to ``null``), the transaction
creates a **new contract**.
As already mentioned, the address of that contract is not
the zero address but an address derived from the sender and
its number of transactions sent (the "nonce"). The payload
of such a contract creation transaction is taken to be
EVM bytecode and executed. The output data of this execution is
permanently stored as the code of the contract.
This means that in order to create a contract, you do not
send the actual code of the contract, but in fact code that
returns that code when executed.

対象の口座が指定されていない（トランザクションが受け取り人を
もたない，あるいは受け取り人が``null``に指定されている）場合，
トランサクションは**新しいコントラクト**を生成します．


.. note::
  While a contract is being created, its code is still empty.
  Because of that, you should not call back into the
  contract under construction until its constructor has
  finished executing.

.. note::
  コントラクトが生成されている途中では，そコードはまだ空のままです．
  このため，作業中のコントラクトには，
  コンストラクタが実行を終えるまでは，コールバック？するべきではありません．

.. index:: ! gas, ! gas price

Gas ガス
===

Upon creation, each transaction is charged with a certain amount of **gas**,
whose purpose is to limit the amount of work that is needed to execute
the transaction and to pay for this execution at the same time. While the EVM executes the
transaction, the gas is gradually depleted according to specific rules.

生成時に，各トランザクションは一定の量の**ガス**が課金されます．
この目的はトランザクションの実行に必要な作業量を限定し，その実行に対する支払い？
を同時に行うためです．EVMがトランザクションを実行している間，ガスは特定のルールに
したがって徐々に減って？行きます．

The **gas price** is a value set by the creator of the transaction, who
has to pay ``gas_price * gas`` up front from the sending account.
If some gas is left after the execution, it is refunded to the creator in the same way.

**ガス価格**を決めるのはトランザクションの作成者であり，
``gas_price * gas`` を送り先の口座から支払わなければなりません．
実行後にガスが残っていれば，同じやり方で作成者に払い戻されます．

If the gas is used up at any point (i.e. it would be negative),
an out-of-gas exception is triggered, which reverts all modifications
made to the state in the current call frame.

ガスがある時点で使い切られた（つまり，負の値になった）場合，
out-of-gas 例外が引き起こされ，現在の呼び出しの枠組みにおける状態への
変更はすべて取り消されます．

.. index:: ! storage, ! memory, ! stack

Storage, Memory and the Stack ストレージ，メモリ，スタック
=============================

The Ethereum Virtual Machine has three areas where it can store data-
storage, memory and the stack, which are explained in the following
paragraphs.

イーサリアム仮想機械はデータを保存することのできる三つの領域を有しています．
それらはストレージとメモリとスタックで，以降の段落で説明します．

Each account has a data area called **storage**, which is persistent between function calls
and transactions.
Storage is a key-value store that maps 256-bit words to 256-bit words.
It is not possible to enumerate storage from within a contract and it is
comparatively costly to read, and even more to modify storage.
A contract can neither read nor write to any storage apart from its own.

各アカウントは**ストレージ**と呼ばれるデータ領域を有しており，これは
関数呼び出しとトランザクションの間で永続的です．
ストレージはキーバリューストアで256ビットのワードを256ビットのワードにマップします．
コントラクトの内部からストレージを列挙することは不可能で，
ストレージを読むことは比較的コストがかかり，更新するのはもっとコストがかかります．

The second data area is called **memory**, of which a contract obtains
a freshly cleared instance for each message call. Memory is linear and can be
addressed at byte level, but reads are limited to a width of 256 bits, while writes
can be either 8 bits or 256 bits wide. Memory is expanded by a word (256-bit), when
accessing (either reading or writing) a previously untouched memory word (i.e. any offset
within a word). At the time of expansion, the cost in gas must be paid. Memory is more
costly the larger it grows (it scales quadratically).

第2のデータ領域は**メモリ**と呼ばれ，コントラクトはメッセージ毎に新たに初期化された
そのインスタンスを取得します．メモリはリニア？でバイトレベルでアクセスできますが，
読み出しは256ビット幅に限定されている一方で，書き込みは8ビットか256ビット幅のいずれかです．
メモリーはワード分（256ビット）拡張され，これは
それまで触れられていないメモリーワード（つまり，ワード以内の任意のオフセット？）に
アクセス（読み出しでも書き込みでも）したときに起ります．
拡張の時は，ガスでコストを払わなければなりません．
メモリは拡張されればされるほどコスト高になります（2次関数的に増えます）．

The EVM is not a register machine but a stack machine, so all
computations are performed on an data area called the **stack**. It has a maximum size of
1024 elements and contains words of 256 bits. Access to the stack is
limited to the top end in the following way:
It is possible to copy one of
the topmost 16 elements to the top of the stack or swap the
topmost element with one of the 16 elements below it.
All other operations take the topmost two (or one, or more, depending on
the operation) elements from the stack and push the result onto the stack.
Of course it is possible to move stack elements to storage or memory
in order to get deeper access to the stack,
but it is not possible to just access arbitrary elements deeper in the stack
without first removing the top of the stack.

EVMはレジスタ機械ではなくスタック機械ですので，すべての計算は**スタック**と呼ばれる
データ領域で実行されます．これは最大サイズが1024要素で，256ビットのワードを格納します．
スタックへのアクセスは以下のようにその上端のみに制限されています．
最上部の16要素の一つをスタックの上端にコピーするか，最上端の要素を
そこから16個下までの要素の一つと交換することが可能です．
他のすべての命令は最上部の２つ（もしくは一つ，もしくはそれ以上，命令に依存して）
の要素をスタックから取り，結果をスタックにプッシュします．
もちろん，スタックの要素をストレージやメモリに移してスタックのより深くに
アクセスを行うことは可能ですが，スタックの任意の深さの要素にだけアクセスすることは，
まずスタックの上部を取り除かないことには，不可能です．



.. index:: ! instruction

Instruction Set 命令セット
===============

The instruction set of the EVM is kept minimal in order to avoid
incorrect or inconsistent implementations which could cause consensus problems.
All instructions operate on the basic data type, 256-bit words or on slices of memory
(or other byte arrays).
The usual arithmetic, bit, logical and comparison operations are present.
Conditional and unconditional jumps are possible. Furthermore,
contracts can access relevant properties of the current block
like its number and timestamp.

EVMの命令セットは最小に保たれており，これは，
同意に関する問題を引き起こす可能性のある，誤った，あるいは不整合な実装を避けるためです．
すべての命令は，基本データ型，256ビットワード，もしくはメモリ（あるいは他のバイト配列）
の一部に対して実行されます．
通常の算術，ビット，論理，そして比較命令が存在します．
条件，および，無条件ジャンプが可能である．
さらに，コントラクトは現在のブロックの番号やタイムスタンプといった関連する特性にアクセス
することができます．

For a complete list, please see the :ref:`list of opcodes <opcodes>` as part of the inline
assembly documentation.

完全な一覧については，インラインアセンブリの文章の一部である
:ref:`list of opcodes <opcodes>`
を参照してください．

.. index:: ! message call, function;call

Message Calls メッセージコール
=============

Contracts can call other contracts or send Ether to non-contract
accounts by the means of message calls. Message calls are similar
to transactions, in that they have a source, a target, data payload,
Ether, gas and return data. In fact, every transaction consists of
a top-level message call which in turn can create further message calls.

コントラクトは他のコントラクトを呼び出したり，イーサをコントラクトでない
口座にメッセージコールをつかって送金することができます．
メッセージコールはトランザクションと似ており，送信元，送信先，データペイロード，
イーサ，ガス，戻り値となるデータを持ちます．実際，すべてのトランザクションは
最上位のメッセージコールから構成され，これが次に更なるメッセージコールを生成します．

A contract can decide how much of its remaining **gas** should be sent
with the inner message call and how much it wants to retain.
If an out-of-gas exception happens in the inner call (or any
other exception), this will be signaled by an error value put onto the stack.
In this case, only the gas sent together with the call is used up.
In Solidity, the calling contract causes a manual exception by default in
such situations, so that exceptions "bubble up" the call stack.

コントラクトは残っている**ガス**をどれだけ内部のメッセージコールにつかうべきか，
そして，どれだけ残したいのかを決めることができます．
内部のコールでout-of-gas exception（あるいは他の任意のexception）が生じた場合，
これはスタック上に置かれたエラー値によって通知されます．
Solidityでは，呼び出しを行ったコントラクトはそのような状況では
マニュアルexception?を引き起こすので，その結果，例外はコールスタックで「湧きあがり」ます．

As already said, the called contract (which can be the same as the caller)
will receive a freshly cleared instance of memory and has access to the
call payload - which will be provided in a separate area called the **calldata**.
After it has finished execution, it can return data which will be stored at
a location in the caller's memory preallocated by the caller.
All such calls are fully synchronous.

すでに述べたように，呼び出されたコントラクトは（呼び出したのと同じコントラクトの場合もあります）
新たに初期化されたメモリのインスタンスを受け取り，コールペイロード
- **コールデータ**とよばれる別の領域内に提供されます - へのアクセスを有します．
コントラクトが実行を完了したら，データを返すことができ，このデータは，
呼び出し側の，呼び出し側によって前もって割り当てられたメモリ中の場所に記録されます．

Calls are **limited** to a depth of 1024, which means that for more complex
operations, loops should be preferred over recursive calls. Furthermore,
only 63/64th of the gas can be forwarded in a message call, which causes a
depth limit of a little less than 1000 in practice.

呼び出しは深さ1024に**制限**されており，これはもっと複雑な命令については，
ループを再帰コールよりも優先すべきであることを意味しています．さらに，
64分の63のガスだけしかメッセージコールの中で転送できないので，実際には
1000よりも少し少ない深さに制限されます．

.. index:: delegatecall, callcode, library

Delegatecall / Callcode and Libraries デリゲートコール / コールコードとライブラリ
=====================================

There exists a special variant of a message call, named **delegatecall**
which is identical to a message call apart from the fact that
the code at the target address is executed in the context of the calling
contract and ``msg.sender`` and ``msg.value`` do not change their values.

メッセージコールには特殊な種類が存在しており，**デリゲートコール**と呼ばれます．
これはメッセージコールと同じものですが，対象のアドレスにおけるコードが呼び出している
コントラクトのコンテキストで実行され，
``msg.sender`` と ``msg.value`` が値を変えることがないという点が異なっています．

This means that a contract can dynamically load code from a different
address at runtime. Storage, current address and balance still
refer to the calling contract, only the code is taken from the called address.

これはコントラクトが実行時に別のアドレスから動的にコードをロードできることを意味しています．
ストレージ，現在のアドレス，残高は，呼び出しているコントラクトを参照しますが，
コードだけが呼ばれたアドレスから取得されます．

This makes it possible to implement the "library" feature in Solidity:
Reusable library code that can be applied to a contract's storage, e.g. in
order to implement a complex data structure.

このことによりSolidityにおいて"ライブラリ"の機能を実装することが可能になります．
つまり，例えば，複雑なデータ構造を実装するための，コントラクトのストレージに
適用可能な再利用可能ライブラリのコードです．

.. index:: log

Logs
====

It is possible to store data in a specially indexed data structure
that maps all the way up to the block level. This feature called **logs**
is used by Solidity in order to implement :ref:`events <events>`.
Contracts cannot access log data after it has been created, but they
can be efficiently accessed from outside the blockchain.
Since some part of the log data is stored in `bloom filters <https://en.wikipedia.org/wiki/Bloom_filter>`_, it is
possible to search for this data in an efficient and cryptographically
secure way, so network peers that do not download the whole blockchain
(so-called "light clients") can still find these logs.

ブロックのレベルに至るまでマッピングを行う，
特別にインデックス化されたデータ構造にデータを保存することが可能です．
この**ログ**と呼ばれる機能はSolidyにおいて
:ref:`events <events>`.
を実装するのに用いることができます．
コントラクトはログが生成された後にはログデータにアクセスすることはできませんが，
ブロックチェーンの外からは効率よくアクセスが可能です．
ログデータの一部は
`bloom filters <https://en.wikipedia.org/wiki/Bloom_filter>`_
の形で保存されているので，このデータを効率よく，暗号的に安全な方法で探索することが
可能ですので，ブロックチェーン全体のダウンロードをしないネットワークピア
（いわゆる"ライトクライアント"）でもこれらのログを探すことが可能です．

.. index:: contract creation

Create 生成
======

Contracts can even create other contracts using a special opcode (i.e.
they do not simply call the zero address as a transaction would). The only difference between
these **create calls** and normal message calls is that the payload data is
executed and the result stored as code and the caller / creator
receives the address of the new contract on the stack.

コントラクトは特別なオペコードを用いて（つまり，トランザクションが行うように
単にゼロアドレスを呼ぶのではなく）別のコントラクトを生成することさえ可能です．
これらの**クリエイトコール**と通常のメッセ―ジコールの唯一の違いは，
ペイロードデータが実行され結果がコードとして保存されて，呼び出し側 / 作成者側
がスタック上の新しいコントラクトのアドレスを受け取るということです．

.. index:: selfdestruct, self-destruct, deactivate

Deactivate and Self-destruct 非活性化と自己破壊
============================

The only way to remove code from the blockchain is when a contract at that address performs the ``selfdestruct`` operation. The remaining Ether stored at that address is sent to a designated target and then the storage and code is removed from the state. Removing the contract in theory sounds like a good idea, but it is potentially dangerous, as if someone sends Ether to removed contracts, the Ether is forever lost.

ブロックチェーンからコードを取り除く唯一の方法は，そのアドレスのコントラクトが``自己破壊``命令を実行することです．そのアドレスの残りのイーサは指定された対象に送金され，その上でストレージとコードは状態から削除されます．理論的にはコントラクトの削除は良い考えに見えますが，潜在的な危険を孕んでいて，もし誰かがイーサを削除されたアドレスに送金した場合，そのイーサは永久に失われてしまいます．

.. note::
    Even if a contract's code does not contain a call to ``selfdestruct``, it can still perform that operation using ``delegatecall`` or ``callcode``.

.. note::
    コントラクトのコードが仮に``自己破壊``への呼び出しを含んでいなくても，その命令を``デリゲートコール``や``コールコード``を用いることで実行することが可能です．

If you want to deactivate your contracts, you should instead **disable** them by changing some internal state which causes all functions to revert. This makes it impossible to use the contract, as it returns Ether immediately.

もしコントラクトを非活性化したいのでしたら，代わりに，すべての関数をrevertするように内部状態を変えることで，コントラクトを**disable**するべきです，これは，イーサを瞬時に返すことで，コントラクトを使うことを不可能にします．

.. warning::
    Even if a contract is removed by "selfdestruct", it is still part of the history of the blockchain and probably retained by most Ethereum nodes. So using "selfdestruct" is not the same as deleting data from a hard disk.

.. warning:
    仮にもしコントラクトが"自己破壊"によって取り除かれても，それは未だブロックチェーンのヒストリの一部であり，おそらく，ほとんどのイーサリアムのノードで保存されています．なので，"自己破壊"を用いることは，ハードディスクからデータを消去するのと同じではありません．
