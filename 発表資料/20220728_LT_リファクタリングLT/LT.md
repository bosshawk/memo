---
marp: true
theme: gaia
backgroundColor: floralwhite
style: |
  h1 {
    border-bottom: none;
    position: absolute;
    color: orange;
  }

  h2 {
    color: orange;
  }

  h1.title {
    font-size: 3.0em;
    text-align: left;
    top: 10%;
  }

  h1.sabtitle{
    font-size: 1.5em;
    text-align: left;
    top: 55%;
  }

  div.author {
    text-align: right;
    font-size: 30px;
    color: #7f7f7f;
    font-weight: bold;
    position: absolute;
    bottom: 10%;
    right: 3%;
  }

  span.inner {
    position: absolute;
    text-align: center;
    top: 50%;
    left: 50%;
    transform: translateY(-50%) translateX(-50%);
    margin: 0;
    padding: 10px 0;
    width: 1200px;
  }

  span.subtitle {
    font-size: 35px;
  }
  
  
---

# <h1 class="title">厳格になるPHPと<br>リファクタリング</h1>
# <h1 class="sabtitle">― 注意点とポイント ―</h1>

<!-- _paginate: false -->
<div class="author">
  2022年07月28日</br>
  リファクタリングLT</br>
  頓花
</div>


---
## 自己紹介

-  名前：<ruby>頓花<rp>（</rp><rt>どんが</rt><rp>）</rp></ruby> 
-  経歴：
　　2019年　株式会社ラクス入社　楽楽販売配属
- 技術：
  - 会社：PHP、JavaScript、Java、Postgres
  - 学生：C言語、Fortran
- 特徴：
  - 新しいもの好き（新デバイス、新技術 etc...）



<!-- paginate: true -->
---

## アジェンダ

- <span style="font-size: 50px">最近のPHPで注意するポイント</span>
<br>
- <span style="font-size: 50px">リファクタリングしなかったときの問題点</span>


---

## 最近のPHPの動向

<span class="inner" style="top: 37%;left: 25%; font-size:40px">"ゆるふわ"言語</span>

<span class="inner" style="font-size: 80px;">PHP、厳格になる</span>
 
<span class="inner" style="top: 66%;left: 60%"> だいたい、Nikitaの仕わz....おかげ</span>



<span style="position: absolute; bottom: 100px; right:50px;font-size:25px">※ Nikita：PHPに最も影響を与えたコントリビュータ(現在は離脱)<span>

---

## 最近のPHPの動向 - ピックアップ -

- PHP7.4：プロパティ型指定の導入
- PHP8.0：match式の導入／<span style="color: red;">緩やかな比較演算子の挙動変更</span>
- PHP8.1：列挙型・交差型・Readonlyアクセス修飾子の導入
- PHP8.2：null型・false型・true型の導入／<span style="color: red;">動的プロパティ禁止</span>
- ～PHP9：<span style="color: red;">未定義変数のアクセス禁止<span>

---

## PHPでのリファクタリング

<span class="inner" style="font-size: 50px">
ってことは・・・<br>
PHPでもたくさん型指定していいってこと？
</span>

<span style="position: absolute; bottom: 100px;right: 60px;">これから毎日、型宣言 しようぜ？</span>


---
## PHPで型指定 <span class="subtitle">-プロパティの型指定①-</span>

```php
class Base { // 元からあるクラス
  protected $a;
}
class Custom extends Base { // 追加するクラス
  protected int $a;
}
```
```
Fatal error: Type of Custom:: must not be defined
```
⇒ 継承前のクラスで指定してない型は指定できない

---
## PHPで型指定 <span class="subtitle">-プロパティの型指定②-</span>

```php
Class NewClass { // 新たに宣言するクラス
  protected int $number;
  public function getNumber(): int {
    return $this->number;
  }
}
echo (new NewClass())->getNumber();
```
```
Fatal error: Uncaught Error: must not be accessed before initialization...
```
⇒ 型指定した場合は、必ず初期化する必要がある

---

## PHPで型指定 <span class="subtitle">-引数の型指定-</span>

```php
Class NewClass {
  public function createName(string $name): string { // 新たに宣言する関数
    return "prefix_" . $name;
  }
}
echo (new NewClass())->createName($GET["name"]);
```
```
Fatal error: Uncaught TypeError: must be of type string, null given...
```
⇒ 型指定はPHPの型宣言はnullを許容しないのでエラーとなる

---

## PHPで型指定 <span class="subtitle">-特徴-</span>

- 型指定すると他の静的型付け言語に似た挙動になる
<br>
- 型を指定しないと特にエラーとならない
<br>
⇒ ゆるふわな挙動と厳格な挙動が混在する状態が作れる

---

## PHPで型指定 <span class="subtitle">-まとめ-</span>



- ゆるふわな頃のPHPやレガシーなコードに慣れていると
間違った使い方をしてしまう可能性がある
<br>


- 新しく導入された仕様／関数 などは注意して使おう



---

## 厳密な書き方しなくてもいいのでは？

型指定するのに注意がいるし・・・。
<br>
→ 問題がおきるまでそのまま使おう！


--- 

## 問題発生 <span class="subtitle">-PHP8バージョンアップ-</span>

1. <span style="color: red;">いくつかの下位互換性のない変更</span>
   - やかな比較演算子の挙動変更
   - 警告レベル変更
   (例：配列標準関数の引数型の厳格化)



---


## 緩やかな比較演算子の挙動変更

「==」などの演算子において文字列と数値の比較結果が一部変更
※厳密な比較演算子「===」がある

<table
style="
border-collapse: collapse;
  border: 1;
background: none;
      font-size: 25px;
      text-align: center;
      width: 50%;">
    <tr><th>条件式</th><th>7.x</th><th> 8.x</th></tr>
    <tr><td>0 == "0"</td><td >TRUE</td><td>TRUE</td></tr>
    <tr><td> 0 == "0.0"</td><td >TRUE</td><td>TRUE</td></tr>
    <tr><td> 0 == "foo"</td><td >TRUE</td><td style="color: orange;">FALSE</td></tr>
    <tr ><td style="
    border-left-color: red;
    border-top-color: red;
    border-bottom-color: red;
     border-width: 5px 1px 5px 5px;
     ">0 == ""</td>
     <td style="
    border-top-color: red;
    border-bottom-color: red;
     border-width: 5px 1px 5px 1px;
     "
     >TRUE</td><td style="
     color: orange;
         border-right-color: red;
    border-top-color: red;
    border-bottom-color: red;
     border-width: 5px 5px 5px 1px;
     ">FALSE</td></tr>
    <tr><td>42 == "42"</td><td>TRUE</td><td>FALSE</td></tr>
    <tr><td> 42 == "42foo"</td><td>TRUE</td><td style="color: orange;">FALSE</td></tr>
  </table>

--- 

## 問題発生 <span class="subtitle">-PHP8バージョンアップ-</span>

1. いくつかの下位互換性のない変更
   - やかな比較演算子の挙動変更
   - 警告レベル変更
   (例：配列標準関数の引数型の厳格化)


2. <span style="color: red;">10年以上のレガシーシステム</span>



---

## この二つから起きる問題は・・・？

<span class="inner" style="font-size: 70px; color: orange; ">
膨大な影響範囲を修正することに...
</span>

<span class="inner" style="top: 66%;left: 60%"> すべての比較演算子の利用箇所調査（約2万箇所以上に...）
</span>

<span style="position: absolute; bottom: 100px; right:50px;font-size:30px">
なんとか対応しリリース...（リリース後は特に大きな問題はなし）
</span>

---

## まったく対応しない場合のデメリット

- 修正範囲が膨大になり、修正コスト大
- EOLがあるため、時間的制限が発生


---

## チームで実施

- 差分ファイルの、静的解析で問題があったコードは必ず修正
(PhpStormの静的解析などを利用)

- ログから警告エラーを調査/監視し対象コードは逐次修正

## 個人的に実施

- 一度読んだところで処理が複雑な箇所には必ずコメントを追加



---

## まとめ

- 新しく導入された仕様／関数 などは注意して使おう
⇒ 導入前は調査が大切
<br>
- リファクタリングはできることがコツコツ実施しよう
⇒ 修正対象の選定は大切



---

<span class="inner" style="font-size: 70px; color: orange; ">
ご清聴ありがとうございました。
</span>


<!-- paginate: false -->

