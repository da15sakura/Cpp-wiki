# doxygen<!-- omit in toc -->
- [コメントスタイル](#コメントスタイル)
  - [JavaDocスタイル](#javadocスタイル)
  - [Qtスタイル](#qtスタイル)
  - [C++スタイル（JavaDocスタイル派生）](#cスタイルjavadocスタイル派生)
- [ドキュメント付け](#ドキュメント付け)
  - [注意点](#注意点)
  - [共通](#共通)
  - [ファイル](#ファイル)
  - [class, struct, enum, union, namespace, define, typedef](#class-struct-enum-union-namespace-define-typedef)
  - [function](#function)
- [出力設定オプション](#出力設定オプション)
  - [基本設定0](#基本設定0)
  - [基本設定1](#基本設定1)
  - [プロジェクト設定1](#プロジェクト設定1)
  - [プロジェクト設定2](#プロジェクト設定2)

## コメントスタイル

### JavaDocスタイル
`/**`と`*/`で囲む。  
その間の行頭の`*`は内容とは無関係である。 
```cpp
/**
contents
*/
```
文中で目立たせたい場合には例えば次のようにする。 
```cpp
/*****************************************//**
* contents ここまでが単に"/*"で囲っている↑↑ここからがDoxygenコメント
*********************************************/
```

### Qtスタイル
`/*!`と`*/`で囲む。  
その間の行頭の`*`は内容とは無関係である。 
```cpp
/*!
contents
*/
```
文中で目立たせたい場合には例えば次のようにする。 
```cpp
/*****************************************//*!
* contents ここまでが単に"/*"で囲っている↑↑ここからがDoxygenコメント
*********************************************/
```

### C++スタイル（JavaDocスタイル派生）
内部的にはJavaDocスタイルと認識されているようだ。コメント開始位置を`///`または`//!`とする 
```cpp
/// contents
//! contents
```
文中で目立たせたい場合には例えば次のようにする。 
```cpp
////////////////////////////////////////////// この行は何の意味もない
/// contents
//! contents
////////////////////////////////////////////// この行は何の意味もない
```

## ドキュメント付け

### 注意点
- 同一ファイル内に異なるスタイルが混在していても良い。
- クラスのメンバーやメソッドなどをドキュメント付けするには，それらを内包するオブジェクト（この場合クラス）に対しドキュメント付けしなければならない。<br>
これと同様に，グルーバル変数や関数などをドキュメント付けするには，それらを内包するオブジェクト（この場合ファイル）に対しドキュメント付けしなければならない。
- 特殊コマンドの前区切りには`\`か`@`が必要であり，その後区切りにはスペースかタブ文字が必要である。
- 記述構造対象の直前に記述することにより，構造指示子（`\class`や`\struct`など）そのものを省略できる。さらには要約説明の特殊コマンド指示子（`\brief`）を省略できる。**冗長性を避けるために直前に記述することを推奨する。**
- 記述対象の直後に記述する際には，各スタイルに合わせて`/**<` `*/`や`/*!<` `*/`や`///<`や`//!<`の記述が適用出来る。
- クラス名などをドキュメントの内容に含めると，自動的にリンクが付けられるが，これを防ぐためにはその名称の前に%をつけることで回避できる。

以下ではJavaDoc形式で例示する。  
各指示子の詳しい説明は，http://www.doxygen.jp/commands.html にある。 

### 共通
主なセクション指示子を次に挙げる。
```cpp
/**
...
\brief 要約説明
\details 詳細説明
\author 著者
\date yyyy/mm/dd
\version 4.0
\bug バグ
\todo TODO
\deprecated 非推奨
\test テスト
\warning 警告
\note 覚え書き
\remarks 意見
\sa 参照
\since から
*/
```
"JAVADOC_AUTOBRIEF"や"QT_AUTOBRIEF"をOFFのときには，要約説明の後に空白行を入れることで，空白行より後ろを詳細説明とすることができる。   
```cpp
/**
...
\brief 要約説明

詳細説明
...
*/
```
"JAVADOC_AUTOBRIEF"や"QT_AUTOBRIEF"をONにすることと，要約説明の最後に'.'を入れることで，'.'より後ろを詳細説明とすることができる。 
```cpp
/**
...
\brief 要約説明.
詳細説明
...
*/
```

### ファイル
```cpp
/**\file（\fileはこのファイル内のグルーバル関数などをドキュメント化するためには必須）
\brief 要約説明
*/
```
#ドキュメント付けの注意点で書いたようにファイルに対しドキュメント付けをしなければグルーバル変数や関数などへのドキュメント付けが有効にはならないので，最低限以下の記述が必要である。 
```cpp
/**\file*/
```

### class, struct, enum, union, namespace, define, typedef
```cpp
class CHoge {...};
struct SHoge {...};
enum EHoge {...};
union UHoge {...};
namespace hoge {...};
#define HOGE_DEF (100)
typedef int HOGE;

/**\class CHoge
\brief 要約説明
*/
/**\struct SHoge
\brief 要約説明
*/
/**\enum EHoge
\brief 要約説明
*/
/**\union UHoge
\brief 要約説明
*/
/**\namespace hoge
\brief 要約説明
*/
/**\def HOGE_DEF
\brief 要約説明
*/
/**\typedef HOGE
\brief 要約説明
*/
```
冗長性を避けるために直前に書くことにより，次のように書ける。 
```cpp
/**要約説明*/
class CHoge {...};

/**要約説明*/
struct SHoge {...};

/**要約説明*/
enum EHoge {...};

/**要約説明*/
union UHoge {...};

/**要約説明*/
namespace hoge {...};

/**要約説明*/
#define HOGE_DEF (100)

/**要約説明*/
typedef int HOGE;
```

### function
```cpp
bool funcHoge(int hoge1, _TCHAR* hoge2, double& hoge3);

/**\fn bool funcHoge(int hoge1, _TCHAR* hoge2, double& hoge3)
\brief hoge関数である
\param[in] hoge1 引数1説明
\param[in,out] hoge2 引数2説明
\param[out] hoge3 引数3説明
\retval true 正常終了
\retval false 異常終了
*/
```
冗長性を避けるために直前に書くことにより，次のように書ける。 
```cpp
/**hoge関数である
\param[in] hoge1 引数1説明
\param[in,out] hoge2 引数2説明
\param[out] hoge3 引数3説明
\retval true 正常終了
\retval false 異常終了
*/
bool funcHoge(int hoge1, _TCHAR* hoge2, double& hoge3);
```
- 戻り値が１つの意味しかない場合（個数など）には，`\retval`の代わりに`\return`の一行のみが使える。
- 戻り値がない場合(`void`)には`\retval`,`\return`は省略してよい。

## 出力設定オプション

### 基本設定0
Doxygenのデフォルト値から変更を推奨する値を記述する。 
|タブ|パラメータ|意味|デフォルト値|推奨設定値|対応するWizard場所|
|---|---|---|---|---|---|
|Project|OUTPUT_LANGUAGE|言語|English|Japanese||
||ALWAYS_DETAILED_SEC|briefだけのメンバも詳細表示するか|NO|YES|
||FULL_PATH_NAMES|フルパスとするか|YES|NO||
||JAVADOC_AUTOBRIEF|JavaDocスタイルコメントで最初の'.'までを要約説明と解釈するか|NO|YES||
||QT_AUTOBRIEF|Qtスタイルコメントで最初の'.'までを要約説明と解釈するか|NO|YES||
||BUILTIN_STL_SUPPORT|STLソースを入力としてインクルードしたくないか|NO|YES||
|Build|SORT_MEMBER_DOCS|メンバーをアルファベット順びソートするか|YES|NO||
|Source Browser|VERBATIM_HEADERS|ヘッダーファイルが埋め込まれる|YES|NO||
|HTML|HTML_DYNAMIC_SECTIONS|表示非表示を制御できるセクションが生成される|NO|YES||
||GENERATE_HTMLHELP|CHMファイルを生成する|NO|YES||
||CHM_INDEX_ENCODING|CHMファイル用文字エンコード|CP932|||
||ENUM_VALUES_PER_LINE|列挙値を一行にいくつ表示させるか|4|1||
|LaTex|GENERATE_LATEX|LaTexを生成するか|YES|NO||
|Dot|HAVE_DOT|Dotの起動を許可するか|NO|YES||

### 基本設定1
個人の環境に合わせて変更を推奨する値を記述する。 
|タブ|パラメータ|意味|デフォルト値|推奨設定値|対応するWizard場所|
|---|---|---|---|---|---|
|HTML|HHC_LOCATION|hhc.exeのパス||hhc.exeのパス||
|Dot|DOT_PATH|graphviz実行環境フォルダーのパス||graphviz実行環境フォルダーのパス||

### プロジェクト設定1
プロジェクトの適性に合わせて変更を推奨する値を記述する。 
|タブ|パラメータ|意味|デフォルト値|推奨設定値|対応するWizard場所|
|---|---|---|---|---|---|
|Build|EXTRACT_ALL|Doxygenコメントが無いものも抽出|NO|||
||EXTRACT_PRIVATE|privateも抽出|NO|||
||EXTRACT_STATIC|staticも抽出|NO|||	
|Input|INPUT_ENCODING|インプットファイルの文字エンコード|UTF-8|CP932||
||RECURSIVE|サブディレクトリーを再帰的に探すか|NO|||
|Dot|HIDE_UNDOC_RELATIONS|ドキュメント付けされていないクラスなどで，継承・コラボレーション図を作成しないか|YES|NO||

### プロジェクト設定2
プロジェクトごとに変更を推奨する値を記述する。 
|タブ|パラメータ|意味|デフォルト値|推奨設定値|対応するWizard場所|
|---|---|---|---|---|---|
|Project|PROJECT_NAME|プロジェクト名||||
||PROJECT_NUMBER|プロジェクト番号||||
||OUTPUT_DIRECTRY|出力フォルダパス||||	
|Input|INPUT|入力ファイルパス||||
|HTML|HTML_OUTPUT|HTML Outputフォルダパス||||
||CHM_FILE|出力CHMファイル名||||