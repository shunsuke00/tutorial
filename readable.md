# リーダブルコード めも
ここではエンジニアの入門書として有名な  
「リーダブルコード ~より良いコードを書くためのシンプルで実践的なテクニック~ 」  
を読んで重要そうなところをメモしておく。  

書籍等のリンクは
[公式の出版元？](https://www.oreilly.co.jp/books/9784873115658/)
や
[他人の要点まとめ](https://qiita.com/KNR109/items/3b14e2e8f89a33c0f959)
を参照のこと。  
ちなみに原著（英語版）だと無料PDFがあるらしい。

以下の構成とページ内リンクは
- [気持ち（思想）](#0-気持ち思想)
- [表面上の改善](#1-表面上の改善)
- [ループとロジックの単純化](#2-ループとロジックの単純化)
- [コードの再構成](#3-コードの再構成)


# 0. 気持ち（思想）
「良い」コードを書くのに重要な考えは

**<span style="color: orange; font-size: 18px">コードは他の人が最短時間で理解（変更を加えたりバグを見つけたり）できるように書かなければならない。</span>**  

ただし必ずしもコードが短いとは限らない。例えば
```
return exponent >= 0 ? mantissa * (1 << exponent) : mantissa / (1 << -exponent);
```
よりも
```
if (exponent >= 0) {
    return mantissa * (1 << exponent);
} else {
    return mantissa / (1 << -exponent);
}
```
の方が行は多いが理解しやすい。

<br>

# 1. 表面上の改善
ここで説明する表面上の改善は  
開発のどの段階でも行うことができ  
かつコードの前部分を知っている必要がないので  
お手軽に取り組むことができる。

## 1-1. 命名規則

## 1-2. 改行と整列
改行位置や空行の挿入位置などで基準となる考えを挙げる。

- [よくあるレイアウトで一貫性を持たせる](#よくあるレイアウトで一貫性を持たせる)
- [列を整列させる](#列を整列させる)
- [意味あるまとまりでブロック分割する](#意味あるまとまりでブロック分割する)

<br>

### よくあるレイアウトで一貫性を持たせる

「よくあるレイアウト」とは、例えばインデントや```{}```で
```Javascript
if (... 
) {
const variable = ...;}
```
このような慣れていないレイアウトよりも
```Javascript
if (...) {
    const variable = ...;
}
```
この方が見やすい、  
というように一般的に用いられるレイアウトを使用せよということ。
※レイアウトだけでなく命名規則等も一般的なものを使用するべき。  
また、一般的な様式は複数ある可能性もある。

「一貫性を持たせる」というのは、  
並び順や改行位置をファイル内（あるいはコード全体を通して）で統一すべき  
ということである。
```Javascript
me      = 20;
father  = 50;
mother  = 50;
brother = 17;
```
例えば上記のような順番で変数を宣言したのであれば  
その後にそれらを呼び出すことがあれば、順番を揃えるのが良い。
```Javascript
addOneYear(me);
addOneYear(father);
addOneYear(mother);
addOneYear(brother);
```
<br>



### 列を整列させる
以下はダメな例
```Javascript
me = func("Doug Adams", "Mr. Douglas Adams", "");
father = func(" Jake  Brown", "Mr. Jake Brown III", "");
brother = func("No Such Guy", "", "no match found");
```
このコードの改善点は「=」を整列させることと  
引数の列をそろえること（後者は人によって好き嫌い分かれる）
```Javascript
me      = func("Doug Adams"  , "Mr. Douglas Adams" , "");
father  = func(" Jake  Brown", "Mr. Jake Brown III", "");
brother = func("No Such Guy" , ""                  , "no match found");
```
<br>

### 意味あるまとまりでブロック分割する
ポイントは2つで
- 宣言をグループごとに分けること
- 処理を段落ごとに分けること

宣言を乱雑に行うと
```C++
// 多分C++
class FrontnedServer {
    public:
        FrontendServer();
        void ViewProfile();
        void OpenDatabase();
        void SaveProfile();
        string ExtractQueryParam();
        void ReplyOK();
        void FindFriends();
        void ReplyNotFound();
        void CloseDatabase();
        ~FrontendServer();
}
```
このように読みたくなくなるので
```C++
class FrontnedServer {
    public:
        FrontendServer();
        ~FrontendServer();

        // ハンドラ
        void ViewProfile();
        void SaveProfile();
        void FindFriends();

        // リクエストとリプライのユーティリティ
        string ExtractQueryParam();
        void ReplyOK();
        void ReplyNotFound();

        // データベースのヘルパー
        void OpenDatabase();
        void CloseDatabase();
}
```
上記のようにグループ分けすべき。  

処理を段落に分けるとは
```Python
# 多分Python

# ユーザーのメール長をインポートして、システムのユーザーと照合する。
# そして、まだ友達になっていないユーザーの一覧を表示する。
def suggest_new_friends(user, email_password):
    friends = user.friends()
    friend_emails = set(f.email for f in friends)
    contacts = import_contants(user.email, email_password)
    contact_emails = set(c.email for c in contacts)
    non_friend_emails = contact_emails - friend_emails
    suggested_friends = User.objects.select(email__in=non_friend_emails)
    display['user'] = user
    display['friends'] = friends
    display['suggested_friends'] = suggested_friends
    return render("suggested_friends.html", display)
```
このように多くの処理を分割し
```Python
def suggest_new_friends(user, email_password):
    # ユーザーの友達のメールアドレスを取得する。
    friends = user.friends()
    friend_emails = set(f.email for f in friends)

    # ユーザーのメールアカウントから全てのメールアドレスをインポート
    contacts = import_contants(user.email, email_password)
    contact_emails = set(c.email for c in contacts)

    # まだ友達になっていないユーザーを探す
    non_friend_emails = contact_emails - friend_emails
    suggested_friends = User.objects.select(email__in=non_friend_emails)

    # それをページに表示する
    display['user'] = user
    display['friends'] = friends
    display['suggested_friends'] = suggested_friends

    return render("suggested_friends.html", display)
```
読み手にとってわかりやすくすること。

<br>


## 1-3. コメント
コメントは画面（あるいは行数）を占領する代わりに、  
コードを理解する時間を短くするという価値を持たせる必要がある。  
重要なことは

- [コードからすぐにわかることや補足的なコメントをしない](#コードからすぐにわかることや補足的なコメントをしない)
- [コードの欠陥に「TODO:」や「XXX:」などのコメントをつける](#コードの欠陥にtodoやxxxなどのコメントをつける)
- [コードを見た人が疑問・驚きを抱きそうなところにコメントをする](#コードを見た人が疑問・驚きを抱きそうなところにコメントをする)
- [全体像についてのコメントをする](#全体像についてのコメントをする)
- [曖昧な代名詞を避ける](#曖昧な代名詞を避ける)
- [関数の動作を正確に記述する](#関数の動作を正確に記述する)
- [入出力の実例（コーナーケースを含む）をつける](#入出力の実例コーナーケースを含むをつける)
- [わかりにくい引数にインラインコメントをつける](#わかりにくい引数にインラインコメントをつける)
- [情報密度を高くする](#情報密度を高くする)

<br>

### コードからすぐにわかることや補足的なコメントをしない
以下はダメな例。
```Javascript
// profitに新たな値を設定する
function Substitute(input) {
    profit = input;
}
```
（命名規則など）コードを改善することで「優れたコード > ひどいコード+コメント」を実現すべき↓
```Javascript
function SetProfit(profit_) {
    profit = profit_;
}
```
<br>

### コードの欠陥に「TODO:」や「XXX:」などのコメントをつける
未完成な部分や欠陥が存在する部分はコメントを残しておくべき。
よく用いられる記法は

| 記法          | 意味 |
| ----         | ---- |
| ```TODO:```  | あとで手をつける |
| ```FIXME:``` | 既知の欠陥があるコード |
| ```HACK:``` | あまり綺麗じゃない解決策 |
| ```XXX:``` | 大きな問題がある |

他にも大文字と小文字で問題の重要度を表現したりするのも効果的。

<br>

### コードを見た人が疑問・驚きを抱きそうなところにコメントをする
例えばわざと遅い処理を採用していたり、  
コードを見た人が？？？となりそうな部分には、  
その疑問に対する回答をコメントとして残しておくべき。

また関数の実行に長い時間が必要になるなど、  
その仕様を知らないまま使って後から！？！？となりそうな部分には、  
最初にその仕様をコメントで知らせておくべき。

<br>

### 全体像についてのコメントをする
新しくチームに参加した人にあなたのコードを渡すとして、  
その人がコード（あるいは特定のファイル）を読み始める時にあなたは何か口頭で伝えるか？  
伝えるならその内容をコメントで書くべき。  
この要約コメントを書くべきなのは

- ファイルについて
- 関数について
- 関数内部の大きな塊について

など。ファイル構造や読む順序を伝えたければ READEME も有用。

<br>

### 曖昧な代名詞を避ける
```Javascript
// 引数を配列に追加し、そのサイズを調べる
```
このコメントでは「その」が引数と配列のどちらを指しているのか曖昧（ちなみに「サイズ」も曖昧）で、以下のように代名詞を避けて明確にするべき
```Javascript
// 引数を配列に追加し、追加後の配列の要素数を調べる
```
※この程度の簡単な処理であれば、本来コメントは不要

<br>

### 関数の動作を正確に記述する
```Javascript
// このファイルに含まれる行数を返す
function CountLines(filename) {...}
```
このコメントでは「行数」が複数の意味を持つため、処理を正確に記述するべき↓
```Javascript
// このファイルに含まれる改行文字('\n')を数える
function CountLines(filename) {...}
```

<br>

### 入出力の実例（コーナーケースを含む）をつける
コーナーケースを含む関数の処理を可視化できる実例を添えると良い。  
こうすることで細かい仕様を短く伝えることが可能になるかも。
```Javascript
// 'src'の先頭や末尾にある'charsを除去する
// 実例：Strip("abba/a/ba", "ab")は"/a/"を返す
function Strip(src, chars) {...}
```

<br>

### わかりにくい引数にインラインコメントをつける
関数呼び出しの際に、渡している値がなんの値かわかりにくい  
（あるいは引数名を理解しておくと便利な）場合はインラインコメントが有用。
```Javascript
Connect(/* timeout_ms = */ 10, /* use_encryption = */ false);
```
※Pythonなどでは名前付き引数が使用できるので、コメントアウトする必要がない

<br>

### 情報密度を高くする
ITの業界用語を用いて情報密度を高くすればコメントを短くすることができる。  
ただし、コードを書く人と読む人のIT的な教養が必要となり、辞書的なものを用意しておいた方がいいかも。

情報密度の高い言葉
- キャッシュ
- 正規化

<br>


# 2. ループとロジックの単純化


# 3. コードの再構成




