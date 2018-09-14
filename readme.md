# go入門

参考

- [はじめてのGo](http://gihyo.jp/dev/feature/01/go_4beginners)
- [Go Web プログラミング](https://astaxie.gitbooks.io/build-web-application-with-golang/content/ja/01.0.html)

```
$ go version
go version go1.11 darwin/amd64
```

-----


## 概要

Goは、2009年にGoogleにより発表されたオープンソースのプログラミング言語です。

- シンプルな言語仕様
- Windows、OS X、Linuxなどの環境に合わせた実行ファイルを生成できるクロスコンパイル
- 並行処理のサポート
- 型がある
- 標準のコーディング規約がある
- 巨大なコードでも高速にコンパイルできて大規模開発にも適してる
- 自由で楽しい `ruby` と堅牢で安心な `go`
- オブジェクト指向ではなく、関数型プログラミング

### シンプル

繰り返し構文はfor文しかなく、while文やdo/while文などもない。

ifの波括弧は省略できず、三項演算子もありません。

開発者による表現のばらつきを抑えてて、誰が書いてもだいたい同じような書き方になる。

Arrayに `.count` 、 `.length` 、 `.size` とエイリアスの多い `ruby` とは逆。

### 危険の回避

意図せぬエラーの原因になる暗黙の型変換などは排除。

使われていない変数、使われていないインポートはコンパイルエラー。

コードを追加することよりもコードを削除するほうが、影響範囲が読みにくく難しい。

### 例外の排除

`try/catch` がなく、発生した例外を戻り値として呼び出し側に返す。

例外を返せないエラー時にはパニックとリカバという例外を制御する方法もある。

### 他値を返す

`python` も `ruby` もできないわけではない。タプルとか

### 平行処理

ゴルーチンという軽量スレッドを用いて並行処理ができる。 `Promise` のようなもの。

同時に実行されているゴルーチンの間ではチャネルという機能でデータをやりとりできる。

### 標準パッケージ

https://golang.org/pkg/

### コマンド

https://golang.org/doc/cmd

|コマンド|説明|
|:-:|:-:|
|go build|プログラムのビルド|
|go fmt|Goの規約に合わせてプログラムを整形|
|go get|外部パッケージの取得|
|go install|プログラムのビルドとインストール|
|go run|プログラムのビルドと実行|
|go test|テストやベンチマークの実行|
|go tool yacc|パーサをGoで出力するGo実装のyacc（パーサジェネレータ）|
|godoc|ソースからドキュメントの生成|


-----


## とりあえず動かしてみる

### インストール

```
$ brew install go

各々shellにpath通す
$ echo 'export PATH=$PATH:/usr/local/opt/go/libexec/bin' >> ~/.zshrc
$ source ~/.zshrc

$ go version
go version go1.11 darwin/amd64
```

もしくは[ここ](https://golang.org/doc/install)からインストール

### Hello world

```hello.go
package main

import "fmt"

func main(){
    fmt.Printf("Hello, world\n")
}
```

```
$ go run hello.go
hello world
```

### ビルドしてバイナリを実行してみる

```
$ go build hello.go
$ ls
hello hello.go
$ ./hello
hello world
```

64ビットのMac OS Xなら64ビットのMac OS Xで動くバイナリができる

他の環境でbuildしたいならオプションで指定する

この `バイナリができる` ってのがとても良くて、環境作らなくて良いし、どこでも動くのでかなり強力！

### コードフォーマット

標準のコーディング規約があって、開発者同士の嗜好を巡って無駄な言い争いが起こることを防いでる

```
$ go fmt hello.go
```

ソフトタブ(スペース2,4つ)がハードタブ(タブ)に変換された

ソフトタブが主流なのでキモく感じるけど、速さを追求して(スペース数分byteを無駄にしない)って事だと思う。

フォーマットしてくれるし規約として一貫性出るからむしろありがたい。

### ドキュメント

パッケージの説明をみたり、公式サイトと同様のインタフェースでlocalにサーバー立てれる(オフライン以外必要ないけど)

```
$ godoc fmt
$ godoc -http=":3000"
http://localhost:3000/
```


-----


## プロジェクト構成

ファイルが増えたときは、パッケージを分割する場合があります。

その場合は、プロジェクトをきちんと構成し環境変数を指定する必要があります。

myprojectというプロジェクトの中でgosampleというパッケージを作成し、そのパッケージをmainパッケージから呼び出すよう構成してみます。

```
$ tree myproject
myproject
├── bin # go install時の格納先
├── pkg # 依存パッケージのオブジェクトファイル
└── src # プログラムのソースコード
```

```
$ cd myproject
$ export GOPATH=`pwd`

$ echo $GOPATH
/path/to/myproject
```

GOPATHがプロジェクトルートになり配下3つのディレクトリの命名規則を用いて依存関係を解決する

プロジェクトルートを `.envrc` 指定の方が便利かも

```.envrc
export GOPATH=`pwd`
```

### パッケージの作成

Goでは，1つのパッケージは1つのディレクトリに格納します。

Message変数を持ったgosampleとそれをprintするmain.goを作ります。

```src/gosample/gosample.go
package gosample

var Message string = "hello world"
```

```src/main/main.go
package main

import (
    "fmt"
    "gosample"
)

func main() {
    fmt.Println(gosample.Message)
}
```

### 実行

```
$ cd $GOPATH
$ go run src/main/main.go
hello world
```

### ビルド

```
$ cd $GOPATH/src/main

$ go install # $GOPATH/binに入る
$ go build # その場にバイナリ作る
```


-----


## パッケージを公開してみよう

gitでリポジトリが公開されていればパッケージとして使用可能

rubygemsに公開する時くらいの心理障壁はない

```diff
├── bin
│   └── main
├── pkg
└── src
-    ├── gosample
-    │   └── gosample.go
+    ├── github.com
+    │   └── arakawamoriyuki
+    │       └── gosample
+    │           └── gosample.go
    └── main
        └── main.go
```

```diff
package main

import (
    "fmt"
-    "gosample"
+    "github.com/arakawamoriyuki/gosample"
)

func main() {
    fmt.Println(gosample.Message) // hello world
}
```

あとはgithubでリポジトリ作って公開するだけ

```
$ cd $GOPATH/src/github.com/arakawamoriyuki/gosample
$ git init
$ git add .
$ git commit -m 'my first golang package'
$ git remote add origin https://github.com/arakawamoriyuki/gosample
$ git push origin master
```

### 公開したパッケージを使う

`go get` コマンドでプロジェクト内に展開します

```
$ cd $GOPATH/src
$ rm -rf github.com # 一旦けして

$ go get github.com/arakawamoriyuki/gosample # 取得

$ go run main/main.go # 実行してみる
```

### 依存パッケージ管理

[依存パッケージ管理（Go）](https://gist.github.com/nakaji-s/7969970)

> Revision指定できないの？？と思った方。鋭いです。お察しの通り__できません__（少なくとも現行バージョンの1.2では不可能です）。

> パブリックなパッケージは常に後方互換性を持たせるべきで、異なる機能性を持たせたいときは、パッケージ自体の名前を変えて、import pathも変更するべき。

とはいえ依存パッケージ管理には需要があり、以下のようなツールがある。

- vgo
- dep
- Glide
- gom

[Google Trend](https://trends.google.co.jp/trends/explore?geo=JP&q=golang%20vgo,golang%20dep,golang%20glide,golang%20gom)

#### Modules (vgo)

[Go & Versioning(vgo)を読んで大きな変更が入ったなと思った](https://qiita.com/lufia/items/67701e2f927c77a75d6e)

> Go 1.11で試験的な導入、Go 1.12で正式サポート

[8月28日にリリースされたGo v1.11](https://github.com/golang/go/releases)から試験的に利用可能な公式で作っている依存パッケージ管理ツールです。

`v1.12` になった際には `dep` から `Modules (vgo)` に移行すると思われる。

- $GOPATHが不要になる
- `dep` では必要なvendorディレクトリが必要なくなる
- パッケージ側はセマンティックバージョニングでタグ付けする
- go.modファイルに依存関係が書かれる

```
Modulesを利用する
$ export GO111MODULE=on

buildするとgo.modが作られて依存関係が書かれる
$ cd $GOPATH
$ go build src/main/main.go
```

```go.mod
module github.com/arakawamoriyuki/go-handson

require github.com/arakawamoriyuki/gosample v1.0.0 // indirect
```

[go1.11のModulesを使って依存パッケージを管理する](https://qiita.com/fuku2014/items/ae24c2504097a4dcad28)

[Go 1.11 の modules・vgo を試す - 実際に使っていく上で考えないといけないこと #golang](https://www.wantedly.com/companies/wantedly/post_articles/132270)

[vgo (Versioned Go) に関する覚え書き](http://text.baldanders.info/golang/go-and-versioning/)

#### dep

[github golang/dep](https://github.com/golang/dep)

[Go 1.11 の modules・vgo を試す - 実際に使っていく上で考えないといけないこと #golang](https://www.wantedly.com/companies/wantedly/post_articles/132270)

> dep は Go が公式に "実験的に" 作っている依存管理ツールです。glide など以前から存在していたツールからの migration の仕組みも持っており、dep への移行を促すプロジェクトも多いです。

-----


## 言語仕様

### パッケージ宣言

```sample.go
package main
```

`go run` ではmainパッケージのmain関数が実行される。

それ以外の、例えば `gosample.go` にmain関数を置いて実行しても `package main` ではないのでエラーになる (`go run: cannot run non-main package`)

`gosample.go` で `package main` (と偽って)宣言してmain関数を用意したら実行できる模様。


### インポート

区切りにカンマはいらない

GOPATH環境変数からパスを解決する

```sample.go
import (
    "fmt"
    "github.com/arakawamoriyuki/gosample"
)
```

インポートにはいくつかのオプションがある。以下の例では

- `文字列` はエイリアスになる( `fmt.Println()` が `f.Println()` )
- `.` は中の関数が展開される ( `strings.ToUpper()` が `ToUpper()` )
- `_` は使用していないパッケージだと明示する (使用してなくてもコンパイルエラーにならない)

```sample.go
import (
    f "fmt"
    _ "github.com/wdpress/gosample"
    . "strings"
)
```

### 型

|型|説明|
|:-:|:-:|
|uint8|8ビット符号なし整数|
|uint16|16ビット符号なし整数|
|uint32|32ビット符号なし整数|
|uint64|64ビット符号なし整数|
|int8|8ビット符号あり整数|
|int16|16ビット符号あり整数|
|int32|32ビット符号あり整数|
|int64|64ビット符号あり整数|
|float32|32ビット浮動小数|
|float64|64ビット浮動小数|
|complex64|64ビット複素数|
|complex128|128ビット複素数|
|byte|uint8のエイリアス|
|rune|Unicodeのコードポイント|
|uint|32か64ビットの符号なし整数|
|int|32か64ビットの符号あり整数|
|uintptr|ポインタ値用符号なし整数|
|error|エラーを表わすインタフェース|

文字列はダブルクォート

```sample.go
var Message string = "hello world"
```

ヒアドキュメントはバッククォート

```sample.go
var Message string = `first line
second line
third line`
```

型推論 (明らかにわかる型はコンパイラが推論して型宣言を省略できる)

```sample.go
message := "hello world"
```

### 変数

```sample.go
//var 変数名 型 = 値
var message string = "hello world"

// 同じ型なら省略可能
var foo, bar, buz string = "foo", "bar", "buz"
var (
    a string = "aaa"
    b = "bbb"
    c = "ccc"
    d = "ddd"
    e = "eee"
)
```

### 定数

定数はconst、再代入不可になる

```sample.go
const Hello string = "hello"
Hello = "bye" // cannot assign to Hello
```

### ゼロ値

代入しない場合ゼロ値になる

```sample.go
var i int // 整数型のゼロ値 0 になる
```

|型|ゼロ値|
|:-:|:-:|
|整数型|0|
|浮動小数点型|0.0|
|bool|false|
|string|""|
|配列|各要素がゼロ値の配列|
|構造体|各フィールドがゼロ値の構造体|
|そのほかの型|nil|

### if

条件に丸括弧はいらない

三項演算子はない

```sample.go
a, b := 10, 100
if a > b {
    fmt.Println("a is larger than b")
} else if a < b {
    fmt.Println("a is smaller than b")
} else {
    fmt.Println("a equals b")
}
```

### for

基本的なfor

```sample.go
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

whileっぽいfor

```sample.go
n := 0
for n < 10 {
    fmt.Printf("n = %d\n", n)
    n++
}
```

無限ループ

```sample.go
for {
    doSomething()
}
```

ループを終了する `break` 、スキップする `continue` などは利用可能。

### switch

カンマで区切った複数の値も指定可能。

```sample.go
n := 10
switch n {
case 15:
    fmt.Println("FizzBuzz")
case 5, 10:
    fmt.Println("Buzz")
case 3, 6, 9:
    fmt.Println("Fizz")
default:
    fmt.Println(n)
}
```

caseが1つ実行されると次のcaseにうつらない。 `fallthrough` キーワードで次にうつる事もできる。

```sample.go
n := 3
switch n {
case 3:
    n = n - 1
    fallthrough
case 2:
    n = n - 1
    fallthrough
case 1:
    n = n - 1
    fmt.Println(n) // 0
}
```

### 関数

引数には型を指定、複数同じ型なら一つにまとめられる。

```sample.go
func sum(i, j int) {
    fmt.Println(i + j)
}
```

戻り値は関数定義のあとに型指定。

```sample.go
func sum(i, j int) int {
    return i + j
}
```

複数値を返せるので複数の型指定。

```sample.go
func swap(i, j int) (int, int) {
    return j, i
}
```

名前付き戻り値で `return` の後の値を省略できる。(代入された値を返す。代入されていなければ結果的にゼロ値を返す)

```sample.go
func div(i, j int) (result int, err error) {
    if j == 0 {
        err = errors.New("divied by zero")
        return // return 0, errと同じ
    }
    result = i / j
    return // return result, nilと同じ
}
```

無名関数

```sample.go
func(i, j int) {
    fmt.Println(i + j)
}(2, 4)
```

変数に入れる事もできます。

```sample.go
var sum func(i, j int) = func(i, j int) {
    fmt.Println(i + j)
}
```

可変長引数も利用できます。

```sample.go
func sum(nums ...int) (result int) {
    // numsは[]int型
    for _, n := range nums {
        result += n
    }
    return
}

func main() {
    fmt.Println(sum(1, 2, 3, 4))  // 10
}
```


-----


### エラー

goはエラーを戻り値で表現するため、`try/catch` や `throw` がありません。

自作のエラーは，errorsパッケージを使って表現します。

```sample.go
package main

import (
    "errors"
    "fmt"
    "log"
)

func div(i, j int) (int, error) {
    if j == 0 {
        // 自作のエラーを返す
        return 0, errors.New("divied by zero")
    }
    return i / j, nil
}

func main() {
    n, err := div(10, 0)
    if err != nil {
        // エラーを出力しプログラムを終了する
        log.Fatal(err)
    }
    fmt.Println(n)
}
```

複数の値を返す場合もエラーを最後にする慣習があるのでそれに従いましょう。

## 配列

配列は固定長で、長さを指定する。

```sample.go
var arr1 [4]string
```

`[...]` で暗黙的に長さの指定。
暗黙の型変換と組み合わせた例。

```sample.go
arr := [...]string{"a", "b", "c", "d"}
```

もちろん引数にも型と長さの指定をする必要があります。

```sample.go
func fn(arr [4]string) {
    fmt.Println(arr)
}

func main() {
    var arr1 [4]string
    var arr2 [5]string

    fn(arr1) // ok
    fn(arr2) // コンパイルエラー
}
```

スライスという可変長配列も定義できる。

```sample.go
var s []string
s := []string{"a", "b", "c", "d"}
```

pythonのように値を部分的に切り出す事ができます。

```sample.go
s := []int{0, 1, 2, 3, 4, 5}
fmt.Println(s[2:4])      // [2 3]
fmt.Println(s[0:len(s)]) // [0 1 2 3 4 5]
fmt.Println(s[:3])       // [0 1 2]
fmt.Println(s[3:])       // [3 4 5]
fmt.Println(s[:])        // [0 1 2 3 4 5]
```


### append

`append` はスライスの末尾に値を追加し、その結果を返す組込み関数です。

```sample.go
s1 := []string{"a", "b"}
s2 := []string{"c", "d"}
s1 = append(s1, s2...) // s1にs2を追加
fmt.Println(s1)        // [a b c d]
```

### range

添字によるアクセスの代わりに `range` を使用できます。

```sample.go
var arr [4]string

arr[0] = "a"
arr[1] = "b"
arr[2] = "c"
arr[3] = "d"

for i, s := range arr {
    // i = 添字, s = 値
    fmt.Println(i, s)
}
```

```
$ go run range.go
0 a
1 b
2 c
3 d
```

## マップ

`string` のキーに `int` の値を格納するマップ

```sample.go
var month map[string]int = map[string]int{}

month := map[string]int{
    "January": 1,
    "February": 2,
}
```

キーの存在確認

```sample.go
_, ok := month["January"]
if ok {
    // データがあった場合
}
```

マップからデータを消す

```sample.go
delete(month, "January")
```

[Go にジェネリクスがなくても構わない人たちに対する批判について](https://methane.hatenablog.jp/entry/2017/09/19/Go_%E3%81%AB%E3%82%B8%E3%82%A7%E3%83%8D%E3%83%AA%E3%82%AF%E3%82%B9%E3%81%8C%E3%81%AA%E3%81%8F%E3%81%A6%E3%82%82%E6%A7%8B%E3%82%8F%E3%81%AA%E3%81%84%E4%BA%BA%E3%81%9F%E3%81%A1%E3%81%AB%E5%AF%BE%E3%81%99)

[Go言語 - 空インターフェースと型アサーション](https://blog.y-yuki.net/entry/2017/05/08/000000)

JAVAのように配列の中やマップの中まで型定義できるジェネリクスは現時点でない模様。

`interfacce{}` 型を利用するらしい

## ポインタ

Goはポインタを扱うことができます。

PHPでいう引数に&つけて明示的に参照渡すやつ。 `function increment(&$var) {...}`

int型も参照渡しできるし配列を値渡しもできる。

```sample.go
func callByValue(i int) {
    i = 20 // 値を上書きする
}

func callByRef(i *int) {
    *i = 20 // 参照先を上書きする
}

func main() {
    var i int = 10
    callByValue(i) // 値を渡す
    fmt.Println(i) // 10
    callByRef(&i) // アドレスを渡す
    fmt.Println(i) // 20
}
```

明示的にポインタだと宣言できない言語(rubyもpythonもjavascriptも)が多い中、中で更新されてるか毎度疑わなくて良くなるので嬉しい。

引数受ける側も関数利用側もポインタ渡す/受けるなら `*` もしくは `&` つけないとコンパイルエラーになるので、副作用理解した上でプログラミングできる。

明示的なのはとても嬉しい。

## defer

ファイル開いたり、データベースにコネクション貼ったり、エラーが起きても起きなくても実行して欲しいものに書く。

`defer` は延期という意味。

`finaly` みたいなイメージで良いと思う。

```sample.go
func main() {
    file, err := os.Open("./error.go")
    if err != nil {
        // エラー処理
    }
    // 関数を抜ける前に必ず実行される
    defer file.Close()
    // 正常処理
}
```

## パニック

エラーは戻り値によって表現するのが基本だが、

配列やスライスの範囲外にアクセスした場合や、ゼロ除算をしてしまった場合などはエラーを返せない。

この状態をパニックという。

パニックで発生したエラーは `recover` で拾えるので、deferで処理する事でエラー処理ができる。

```sample.go
func main() {
    defer func() {
        err := recover()
        if err != nil {
            // runtime error: index out of range
            log.Fatal(err)
        }
    }()

    a := []int{1, 2, 3}
    fmt.Println(a[10]) // パニックが発生
}
```

パニックは自分で起こす事もできる。けど、基本的にエラーは関数の戻り値として呼び出し側に返しましょう。

```sample.go
a := []int{1, 2, 3}
for i := 0; i < 10; i++ {
    if i >= len(a) {
        panic(errors.New("index out of range"))
    }
    fmt.Println(a[i])
}
```


-----


## type

以下の関数はint型の `id` と `priority` を受け取る。

```sample.go
func ProcessTask(id, priority int) {
}
```

同じ `int` 型なので

引数の順番間違えてもコンパイル通っちゃう間違えやすいインターフェースになってる。

```sample.go
var id int = 3
var priority int = 5
ProcessTask(id, priority)
ProcessTask(priority, id) // 順番間違えてもコンパイル通る
```

場合に応じて独自の型を定義すると安全になる。

```sample.go
type ID int
type Priority int

func ProcessTask(id ID, priority Priority) {
}

var id ID = 3
var priority Priority = 5
ProcessTask(priority, id) // コンパイルエラー
```

## 構造体（struct）

構造体も独自の型として宣言可能。

構造体のプロパティはドットでアクセス可能

```sample.go
type Task struct {
    ID int
    Detail string
    done bool
}

var task Task = Task{
    ID: 1,
    Detail: "buy the milk",
    done: true,
}
fmt.Println(task.ID) // 1
fmt.Println(task.Detail) // "buy the milk"
fmt.Println(task.done) // true
```

## ポインタ型

```sample.go
var task Task = Task{} // Task型
var task *Task = &Task{} // Taskのポインタ型
```

ポインタ型ではない型は値渡しされる。

```sample.go
type Task struct {
    ID int
    Detail string
    done bool
}

func Finish(task Task) {
    task.done = true
}

func main() {
    task := Task{done: false}
    Finish(task)
    fmt.Println(task.done) // falseのまま
}
```

ポインタ型は参照渡しされる。

```sample.go
func Finish(task *Task) {
    task.done = true
}

func main() {
    task := &Task{done: false}
    Finish(task)
    fmt.Println(task.done) // true
}
```

逆にポインタ型でなければ値渡しされる。

```sample.go
func Finish(task Task) {
    task.done = true
}

func main() {
    task := Task{done: false}
    Finish(task)
    fmt.Println(task.done) // false
}
```

## new

構造体は組み込み関数 `new` でゼロ値で初期化

```sample.go
type Task struct {
    ID int
    Detail string
    done bool
}

var task Task = new(Task)
```

## コンストラクタ

Goには構造体のコンストラクタにあたる構文がありません。

代わりにNewで始まる関数を定義し、その内部で構造体を生成するのが通例です。

```sample.go
func NewTask(id int, detail string) *Task {
    task := &Task{
        ID: id,
        Detail: detail,
        done: false,
    }
    return task
}

func main() {
    task := NewTask(1, "buy the milk")
    // &{ID:1 Detail:buy the milk done:false}
    fmt.Printf("%+v", task)
}
```

## メソッド

とりあえずメソッドと関数の違い

関数 = 処理をまとめたもの

メソッド = 自身に対する操作をする関数。 `this.name` とかプロパティを利用する関数。

メソッドはメソッド名の前にメソッドを定義したい型を指定する。

変数名は `this` にあたいするものだとと読み替えてもいいかも。

```sample.go
func (変数名 メソッドを定義したい型) メソッド名() 戻り値の型 {
}
```

```sample.go
type Task struct {
    ID int
    Detail string
    done bool
}

func NewTask(id int, detail string) *Task {
    task := &Task{
        ID: id,
        Detail: detail,
        done: false,
    }
    return task
}

// Task.Stringメソッドの宣言、taskの文字列表現を返す
func (task Task) String() string {
    str := fmt.Sprintf("%d) %s %t", task.ID, task.Detail, task.done)
    return str
}

// Task.Finishメソッドの宣言、taskを参照渡しして完了する
func (task *Task) Finish() {
    task.done = true
}

func main() {
    task := NewTask(1, "buy the milk")
    fmt.Printf(task.String())
    task.Finish()
    fmt.Printf(task.String())
}
```

## インターフェース

Javaの `implements` のようなもので、

Taskに実装したString()というメソッドが定義されている事を強制する、などの用途に利用する。

実装すべき関数名が単純な場合は，その関数名にerを加えた名前を付ける慣習がある。

```sample.go
// .String()メソッドを実装している事をコンパイラに知らせる
type Stringer interface {
    String() string
}

// この関数には.String()メソッドを実装しているオブジェクトを渡せる。
func Print(stringer Stringer) {
    fmt.Println(stringer.String())
}

Print(task)
```

例えば、 `fmt.Print` の引数は `String()` を実装している事を `interface` で強制しているので先ほど作った `task` を渡せる。

```sample.go
type Stringer interface {
    String() string
}

fmt.Print(task)
```

インターフェースではないけど `ruby` の `DateTime` とかもそうで、文字列じゃないのに `puts` できるのは基底クラス `Object` に `to_s` が実装されているから。

### interface{}

すべての引数を受け付けるAny型が作れる

```sample.go
type Any interface {
}

func Do(any Any) {
  // do something
}
```

書き方自体はこれと同じ

```sample.go
func Do(any interface{}) {
  // do something
}
```

## 型の埋め込み

Goでは，継承はサポートされていません。

代わりにほかの型を「埋め込む」(Embed) という方式で構造体やインタフェースの振る舞いを拡張できます。

```sample.go
// 苗字と名前を持ったUser構造体
type User struct {
    FirstName string
    LastName string
}

// User.FullName()メソッドの定義
func (u *User) FullName() string {
    fullname := fmt.Sprintf("%s %s", u.FirstName, u.LastName)
    return fullname
}

// Userのコンストラクタ
func NewUser(firstName, lastName string) *User {
    return &User{
        FirstName: firstName,
        LastName: lastName,
    }
}

// Userを埋め込んだTask構造体を定義
type Task struct {
    ID int
    Detail string
    done bool
    *User // Userを埋め込む(Embed)
}

// Taskのコンストラクタ
func NewTask(id int, detail, firstName, lastName string) *Task {
    task := &Task{
        ID: id,
        Detail: detail,
        done: false,
        User: NewUser(firstName, lastName),
    }
    return task
}

func main() {
    task := NewTask(1, "buy the milk", "Jxck", "Daniel")

    // TaskにUserのフィールドが埋め込まれている
    fmt.Println(task.FirstName)
    fmt.Println(task.LastName)

    // TaskにUserのメソッドが埋め込まれている
    fmt.Println(task.FullName())

    // Taskから埋め込まれたUser自体にもアクセス可能
    fmt.Println(task.User)
}
```

## インタフェースの埋め込み

構造体( `struct` )だけではなくインターフェース( `interface` )も「埋め込む」(Embed) 事ができます

```sample.go
// 読み込みメソッドがある事をインターフェースで宣言
type Reader interface {
    Read(p []byte) (n int, err error)
}

// 書き込みメソッドがある事をインターフェースで宣言
type Writer interface {
    Write(p []byte) (n int, err error)
}

// 読み込みメソッドも書き込みメソッドもある事をインターフェースで宣言
type ReadWriter interface {
    Reader
    Writer
}
```

## 型変換 (キャスト)

暗黙の型変換はできないけど明示的に型変換(キャスト)はできます。

```sample.go
var s string = "abc"
var b []byte = []byte(s) // string -> []byte
fmt.Println(b)           // [97 98 99]
```

キャストに失敗した場合はパニックが発生します。

```sample.go
// cannot convert "a" (type string) to type int
a := int("a")
```

## 型の検査

Type Assertionで型を調べる事ができます。

第一戻り値が元の値、第二戻り値が調べた結果です。

```sample.go
s, ok := value.(string) // Type Assertion
if ok {
    fmt.Printf("value is string: %s\n", s)
} else {
    fmt.Printf("value is not string\n")
}
```

Type Switchで型で分岐処理ができます。

```sample.go
switch v := value.(type) {
case string:
    fmt.Printf("value is string: %s\n", v)
case int:
    fmt.Printf("value is int: %d\n", v)
case Stringer:
    fmt.Printf("value is Stringer: %s\n", v)
}
```


-----


## ゴルーチン

軽量スレッド、ゴルーチンを利用して非同期処理できる

以下は `https://google.com` にリクエストを3回送る迷惑なコード。

同期処理の場合は、1度目のリクエスト完了後に2度目,3度目...と直列で続く。

```sample.go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func main() {
    urls := []string{
        "https://google.com",
        "https://google.com",
        "https://google.com",
    }
    for _, url := range urls {
        res, err := http.Get(url)
        if err != nil {
            log.Fatal(err)
        }
        defer res.Body.Close()
        fmt.Println(url, res.Status)
    }
}
```

`go` キーワードで非同期処理の場合は、並行してリクエストを行うのでその分早い。

```sample.go
package main

import (
    "fmt"
    "log"
    "net/http"
    "sync"
)

func main() {
    wait := new(sync.WaitGroup)
    urls := []string{
        "https://google.com",
        "https://google.com",
        "https://google.com",
    }
    for _, url := range urls {
        // waitGroupに追加
        wait.Add(1)
        // 取得処理をゴルーチンで実行する
        go func(url string) {
            res, err := http.Get(url)
            if err != nil {
                log.Fatal(err)
            }
            defer res.Body.Close()
            fmt.Println(url, res.Status)
            // waitGroupから削除
            wait.Done()
        }(url)
    }
    // 待ち合わせ
    wait.Wait()
}
```

## チャネル

複数のゴルーチンのデータをやりとりしたい場合チャネルを利用する。

まずは組み込み関数 `make` でチャネルの作り方と書き込み、読み込み方法について。

```sample.go
// stringを扱うチャネルを生成
ch := make(chan string)

// チャネルにstringを書き込む
ch <- "a"

// チャネルからstringを読み出す
message := <- ch
```

チャネルを利用して

HTTPリクエストを並行して発行し、早く取得されたステータスから順に受け取って処理しておく事ができる

```sample.go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func main() {
    urls := []string{
        "https://google.com",
        "https://google.com",
        "https://google.com",
    }
    statusChan := make(chan string)
    for _, url := range urls {
        // 取得処理をゴルーチンで実行する
        go func(url string) {
            res, err := http.Get(url)
            if err != nil {
                log.Fatal(err)
            }
            defer res.Body.Close()
            statusChan <- res.Status
        }(url)
    }
    for i := 0; i < len(urls); i++ {
        // waitする必要がなく、先に受け取ったチャネルから処理できる
        fmt.Println(<-statusChan)
    }
}
```

## select文を用いたイベント制御

読み出しや書き込みイベント制御のためのselect文もあります。

主な用途はfor/select文とbreakを用いて実装するタイムアウト処理などに利用されます。

```sample.go
ch1 := make(chan string)
ch2 := make(chan string)
for {
    select {
    case c1 := <-ch1:
        // ch1からデータを読み出したときに実行される
    case c2 := <-ch2:
        // ch2からデータを読み出したときに実行される
    case ch2 <- "c":
        // ch2にデータを書き込んだときに実行される
    }
}
```

また、`make` 関数には同時に3つまで読み出されないかぎりチャネルに書き込まない制御をするバッファ付きチャネルや、

同時起動数制御などメッセージキューのような動作をしてくれるバッファ指定引数があります。

## 便利なパッケージ

- `encoding/json` JSONから構造体、構造体からJSONに変換
- `os`, `io` ファイルの作成、書き込み、読み込みなど
- `io/ioutil` ファイルの操作のラッパー
- `net/http` HTTPサーバ作成API
- `text/template` webに必要なhtmlのテンプレートエンジン

## フレームワーク

`Gin` がデファクト?

- Gin
- Echo
- iris
- Goji
- beego
- Revel
- Martini
- Gorilla


-----

## API作ってみよう

`httprouter` を使ってAPIサーバーを作って見ましょう。

```
├── bin
├── pkg
└── src
    ├── github.com
    │   └── julienschmidt
    │       └── httprouter
    │           ├── LICENSE
    │           ├── README.md
    │           ├── params_go17.go
    │           ├── params_legacy.go
    │           ├── path.go
    │           ├── path_test.go
    │           ├── router.go
    │           ├── router_test.go
    │           ├── tree.go
    │           └── tree_test.go
    └── main
        └── main.go
```

`httprouter` をgo getします。

```
$ cd $GOPATH/src
$ go get github.com/julienschmidt/httprouter
```

`main.go` を作成します。

```main.go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "log"
    "net/http"

    "github.com/julienschmidt/httprouter"
)

// /Hello/:langにハンドルされているHello関数
func Hello(w http.ResponseWriter, r *http.Request, p httprouter.Params) {
    lang := p.ByName("lang") // langパラメーターを取得する
    fmt.Fprintf(w, lang)     // レスポンスに値を書き込む
}

// /ExampleにハンドルされているExample関数
func Example(w http.ResponseWriter, r *http.Request, p httprouter.Params) {
    defer r.Body.Close() // Example関数が終了する時に実行されるdeferステートメント

    // リクエストボディを読み取る
    bodyBytes, err := ioutil.ReadAll(r.Body)
    if err != nil {
        // リクエストボディの読み取りに失敗した => 400 Bad Requestエラー
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // JSONパラメーターを構造体にする為の定義
    type ExampleParameter struct {
        ID   int    `json:"id"`
        Name string `json:"name"`
    }
    var param ExampleParameter

    // ExampleParameter構造体に変換
    err = json.Unmarshal(bodyBytes, &param)
    if err != nil {
        // JSONパラメーターを構造体への変換に失敗した => 400 Bad Requestエラー
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // 構造体に変換したExampleParameterを文字列にしてレスポンスに書き込む
    fmt.Fprintf(w, fmt.Sprintf("%+v\n", param))
}

func main() {
    router := httprouter.New() // HTTPルーターを初期化

    // /HelloにGETリクエストがあったらHello関数にハンドルする
    // :langはパラメーターとして扱われる
    router.GET("/Hello/:lang", Hello)

    // /ExampleにPOSTリクエストがあったらExample関数にハンドルする
    router.POST("/Example", Example)

    // Webサーバーを8080ポートで立ち上げる
    err := http.ListenAndServe(":8080", router)
    if err != nil {
        log.Fatal(err)
    }
}
```

APIサーバーを実行します。

```
$ cd $GOPATH/src/main
$ go run main.go
```

コードを確認しながら試して見ましょう。

```
$ curl http://localhost:8080/Hello/golang
Hello golang

$ curl -XPOST -d "{\"id\": 1, \"name\": \"arakawa\"}" http://localhost:8080/Example
{ID:1 Name:arakawa}
```

`/FizzBuzz/:num` で FizzBuzz APIを作って見ましょう。

```
$ curl http://localhost:8080/FizzBuzz/1
1
$ curl http://localhost:8080/FizzBuzz/2
2
$ curl http://localhost:8080/FizzBuzz/3
Fizz!
$ curl http://localhost:8080/FizzBuzz/4
4
$ curl http://localhost:8080/FizzBuzz/5
5
$ curl http://localhost:8080/FizzBuzz/6
Buzz!
$ curl http://localhost:8080/FizzBuzz/7
7
$ curl http://localhost:8080/FizzBuzz/15
FizzBuzz!
```
