Solidity
========

.. image:: logo.svg
    :width: 120px
    :alt: Solidity logo
    :align: center

Solidity is an object-oriented, high-level language for implementing smart
contracts. Smart contracts are programs which govern the behaviour of accounts
within the Ethereum state.
Solidityはスマートコントラクト実装のための，オブジェクト指向の高水準言語です．スマートコントラクトとはイーサリアムの状態におけるアカウントの動作を制御するプログラムです．

Solidity was influenced by C++, Python and JavaScript and is designed to target
the Ethereum Virtual Machine (EVM).
SolidityはC++, Python, およびJavaScriptの影響を受けており，イーサリアム仮想機械(EVM)を対象として設計されています．

Solidity is statically typed, supports inheritance, libraries and complex
user-defined types among other features.
Solidityはいくつかの特徴の中でも特に，静的に型付けられており，継承，ライブラリー，複雑なユーザー定義の型をサポートしています．

With Solidity you can create contracts for uses such as voting, crowdfunding, blind auctions,
and multi-signature wallets.
Solidityによって，投票，クラウドファンディング，ブラインドオークション，多重署名ウォレットなどの用途のためのコントラクトを生成することができます．

.. note::
    The best way to try out Solidity right now is using
    `Remix <https://remix.ethereum.org/>`_
    (it can take a while to load, please be patient). Remix is a web browser
    based IDE that allows you to write Solidity smart contracts, then deploy
    and run the smart contracts.

.. 注意::
    現状Solinityを試す最良の方法は
    `Remix <https://remix.ethereum.org/>`_を使うことです．
    (ロードするにはしばらく時間がかかりますので，辛抱してください).Remixはウェブブラウザー
    ベースのIDEで，Solidityのスマートコントラクトを書いた上で，そのスマートコントラクトをデプロイして実行することができます．

.. warning::
    Since software is written by humans, it can have bugs. Thus, also
    smart contracts should be created following well-known best-practices in
    software development. This includes code review, testing, audits and correctness proofs.
    Also note that users are sometimes more confident in code than its authors.
    Finally, blockchains have their own things to watch out for, so please take
    a look at the section :ref:`security_considerations`.

.. 警告::
    ソフトウェアは人間が書くものなので，バグがあることがあります．したがって，
    スマートコントラクトも，ソフトウェア工学において良く知られたベストプラクティスに従って作られるべきです．
    これには，コードレビュー，テスト，監査と正しさの証明が含まれます．
    また，ユーザはしばしばコードを，その作者よりも信頼していることに注意してください．
    最後に，ブロックチェーンはそれ自身に注意しておくべきことがあるので，次の節を是非参照してください :ref:`security_considerations`.

Translations　翻訳
------------

This documentation is translated into several languages by community volunteers
with varying degrees of completeness and up-to-dateness. The English version stands as a reference.
この文章はコニュニティーのボランティアによって，いくつかの言語に，完全性と最新性については様々ですが，翻訳されています．
英語版が基準となります．

* `Simplified Chinese <http://solidity-cn.readthedocs.io>`_ (in progress)
* `Spanish <https://solidity-es.readthedocs.io>`_
* `Russian <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (rather outdated)
* `Korean <http://solidity-kr.readthedocs.io>`_ (in progress)
* `French <http://solidity-fr.readthedocs.io>`_ (in progress)

Language Documentation言語仕様
----------------------

On the next pages, we will first see a :ref:`simple smart contract <simple-smart-contract>` written
in Solidity followed by the basics about :ref:`blockchains <blockchain-basics>`
and the :ref:`Ethereum Virtual Machine <the-ethereum-virtual-machine>`.

以降のページでは，まずSolidityで書かれた:ref:`simple smart contract <simple-smart-contract>`
を見て，次に:ref:`blockchains <blockchain-basics>`に関する基本，そして，
:ref:`Ethereum Virtual Machine <the-ethereum-virtual-machine>`と続きます．

The next section will explain several *features* of Solidity by giving
useful :ref:`example contracts <voting>`.
Remember that you can always try out the contracts
`in your browser <https://remix.ethereum.org>`_!

次の節はSolidityのいくつかの*機能*について，有用な:ref:`example contracts <voting>`
を示して説明します．
コントラクトを`自分のブラウザ<https://remix.ethereum.org>`_で
いつでも試してみることができることを忘れないで!

The fourth and most extensive section will cover all aspects of Solidity in depth.

4番目のそしてもっとも稠密な節では，Solidityのすべての面について深くカバーします．

If you still have questions, you can try searching or asking on the
`Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_
site, or come to our `gitter channel <https://gitter.im/ethereum/solidity/>`_.
Ideas for improving Solidity or this documentation are always welcome!

まだ疑問があれば，サーチしてみるか，`Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_に質問したり，
`gitter channel <https://gitter.im/ethereum/solidity/>`_を訪問することができます．
Solidityやこの文章を改良するアイデアはいつでも歓迎します!

Contents目次
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst
   solidity-in-depth.rst
   security-considerations.rst
   resources.rst
   using-the-compiler.rst
   metadata.rst
   abi-spec.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   frequently-asked-questions.rst
