# wheelie

dashを少しだけ便利にする車輪の再発明utility

## 一時ファイルを作りたくない人向けのコマンド

### qsub COMMAND \[ARG\]...

bashのプロセス置換(read)に対応するコマンド。

```
$ diff $(qsub -q 'echo a' 'echo b')
1c1
< a
---
> b
$ cat $(qsub eval 'echo a | sed p')
a
a
$ set 1 2 3
$ cat $(qsub fval 'echo $2' "$@")
2
$ cat $(echo a | qsub)
a
```
### qee QUOTED...

bashのプロセス置換(write)に対応するコマンド。
moreutilのpee。
```
$ echo a | qee tr a c | tr a b
b
c
$ echo a | qee -q 'tr a c' 'tr a b'
a
b
c
```

### pipe \[-r|-l\] QUOTED...

パイプの復帰値のポリシーを変更する。

```
$ pipe -r 'exit 1' 'exit 2' true; echo $?
2
$ pipe -l 'exit 1' 'exit 2' true; echo $?
1
$ pipe 'exit 1' 'exit 2' true; echo $?
0
```

### pipestatus VARIABLE QUOTED...

パイプの復帰値のリストを取得する。

```
$ pipestatus fuga 'exit 1' 'exit 2' true; echo $?; echo $fuga
0
1 2 0
```

### maybe COMMAND \[ARG\]...

標準出力にCOMMANDの成否の文脈を付加する。

本コマンドの右側にパイプでliftまたはnoopコマンドを
繋いだ場合、COMMANDが成功した場合にのみliftまたはnoopコマンドの
引数のコマンドが実行される。

```
$ maybe echo a | lift echo b | lift echo c | decxt
c
$ maybe echoa | lift echo b | lift echo c | decxt
./wheelie: 行 128: echoa: コマンドが見つかりません
$ maybe echo a | lift echob | lift echo c | decxt
./wheelie: 行 128: echob: コマンドが見つかりません
```

### either COMMAND \[ARG\]...

標準出力にCOMMANDの成否の文脈を付加する。
COMMANDが正常復帰する場合はCOMMANDの標準出力を、
そうでない場合はCOMMANDのエラー出力を
標準出力に出力する。

本コマンドの右側にパイプでliftまたはnoopコマンドを
繋いだ場合、COMMANDが成功した場合にのみliftまたはnoopコマンドの
引数のコマンドが実行される。

```
$ either echo a | lift echo b | lift echo c | decxt
c
$ either echoa | lift echo b | lift echo c | decxt 
./wheelie: 行 128: echoa: コマンドが見つかりません
$ msg=`either echo a | lift echob | lift echo c | decxt 2>&1`
$ echo "$msg"
./wheelie: 行 128: echob: コマンドが見つかりません
```

### lift COMMAND \[ARG\]...

eitherまたはmaybeコマンドの出力を受け取って
同じ文脈付きの標準出力を出力する。

本コマンドの右側にパイプでliftまたはnoopコマンドを
繋いだ場合、COMMANDが成功した場合にのみliftまたはnoopコマンドの
引数のコマンドが実行される。

### noop COMMAND \[ARG\]...

eitherまたはmaybeコマンドの出力を受け取って
同じ文脈付きの標準出力を出力する。

liftと違ってCOMMANDの成否にかかわらず
受け取った文脈の情報を変更しない。

本コマンドの右側にパイプでliftまたはnoopコマンドを繋いだ場合、
liftまたはnoopコマンドの引数のコマンドが実行されるかどうかは
COMMANDが実行されるかどうかと同じである。

```
$ either echo a | noop echob | lift echo c | decxt
./wheelie: 行 128: echob: コマンドが見つかりません
c
$ either echoa | noop echo b | lift echo c | decxt
./wheelie: 行 128: echoa: コマンドが見つかりません
```

### decxt

eitherまたはmaybeコマンドの出力を受け取って
付加された文脈を削除して元の標準出力を出力する。

パイプの左側のeitherまたはmaybeまたはliftコマンドがすべて成功しているは
正常復帰、それ以外の場合は失敗したコマンドの戻り値で復帰する。

### readcxt VARIABLE...

標準出力の文脈部分だけ読み取ってVARIABLEに代入する。
decxtの戻り値に相当する値が代入される。

```
$ either echoa | noop echo b | lift echo c | { readcxt x; case $x in 0) cat - ;; *) echo "ERROR: command failed, ecode=$x, msg=$(cat)";; esac; }
ERROR: command failed, ecode=127, msg=./wheelie: 行 128: echoa: コマンドが見つかりません
```

## リソース管理用のコマンド

### tmpf COMMAND \[ARG\]...

一時ファイルを作って引数に渡してコマンドを実行する。

```
$ tmpf fval 'echo a > $1; cat $1'
a
```
### trapend [QUOTED]

`trap QUOTED 0 1 2 3 15`と同義

```
$ (trapend 'echo a')
a
```

### dam FILE

一時ファイルに書き出してからFILEに変名します。

```
$ seq 1 10 > out
$ grep ^1 out | dam out
$ cat out
1
10
$ rm out
```

## 動的変数

### var NAME VALUE

変数名がNAMEである変数にVALUEを代入する。

```
$ for i in 1 2 3 4; do var hoge$i $i; done
$ for i in 1 2 3 4; do withvar hoge$i echo; done
1
2
3
4
```
### withvar VARIABLE COMMMAND \[ARG\]...

VARIABLEの値をCOMMMANDの最終引数に渡して実行する。

### readvar VARIABLE NAME \[VALUE\]

変数名がNAMEである変数の値をVARIABLEに代入する。

```
$ a1=hoge
$ n=1
$ readvar b a$n
$ echo $b
hoge
```

醜いが第3引数以降に位置パラメタを渡すことで
動的に位置パラメタを代入できる。

```
$ set a b c
$ readvar x 2 "$@"
$ echo $x
b
```

## 再帰的データ

### jsondir \[-s CHAR\] \[-n CHAR\] DIR \[JSON\]

json形式のデータを標準入力に受け取って
データに対応するディレクトリ構成を作成する。
sオプションにはキー内に"/"が存在した場合の
置換文字を指定する。デフォルトは"%"である。
nオプションにはキーが空文字だった場合の
置換文字を指定する。デフォルトはtabである。

```
$ echo '{"array": [1, 2], "hash": {"array": ["A", "B"]}, "num": 1}' > json
$ cat json | jsondir dir -s '\031'
$ find dir -type f
dir/num
dir/array/2
dir/array/1
dir/hash/array/2
dir/hash/array/1
$ read x <dir/hash/array/1
$ echo $x
A
$ rm -rf dir json
```

### yamldir \[-s CHAR\] \[-n CHAR\] DIR \[JSON\]

yaml形式のサブセットのデータを標準入力に受け取って
データに対応するディレクトリ構成を作成する。
使い方はjsondirを参照。

```
$ printf '#comment\narray : \n  - 1\n  - 2 \nhash:\n  array: [A, "B"] \n"num": 1' > yaml
$ yamldir -n '\031' dir yaml
$ grep -r . dir/
dir/num:1
dir/array/2:2
dir/array/1:1
dir/hash/array/2:B
dir/hash/array/1:A
$ rm -rf dir yaml
```

なお、以下の要素はサポートされていない。

- シングルクォート
- アンカーとエイリアス
- 複数行の文字列

### withread FILE COMMAND \[ARG\]...

FILEをreadした値をCOMMANDの最終引数に与えて実行する。

```
$ echo A >file
$ withread file echo
A
$ rm -f file
```

## 上記のコマンドと組み合わせるためのコマンド

### fval QUOTED \[ARG\]...

QUOTEDの中の位置パラメタでARGを参照できる。

```
$ fval 'echo $(($2-$1))' 1 2
1
```

### quote VARIABLE STRING...

printfの%qに対応するコマンド。

```
$ quote hoge " " "a b"
$ eval "printf '%s,\n' $hoge"
 ,
a b, 
```

### rot COMMAND \[ARG\]...

最後の引数を最初の引数に持ってくる。

```
$ rot echo 1 2 3
3 1 2
```

### rotn NUMBER COMMAND \[ARG\]...

NUMBER分末尾の引数を最初の引数に持ってくる。

```
$ rotn 2 echo 1 2 3
2 3 1
```

## 集合演算

ソート済重複なしのファイルを使って集合演算をする。

### toset FILE...

`sort | uniq`と同等。

```
$ printf '%s\n' 1 a " a b" 1 | toset
 a b
1
a
```

### union FILE...

集合の和を取る。

```
$ seq 1 3 | union - $(qsub seq 2 4)
1
2
3
4
```

### exc FILE...

集合の差を取る。

```
$ seq 1 3 | exc - $(qsub seq 2 4)
1
```

### meet FILE...

集合の積を取る。

```
$ seq 1 3 | meet - $(qsub seq 2 4)
2
3
```

### prd \[-s SEPARATOR\] FILE...

集合の直積を取る。-sで区切り文字を指定する。
ソート済でなくとも重複ありでも良い。

```
$ seq 1 3 | prd -s " " - $(qsub seq 2 4)
1 2
1 3
1 4
2 2
2 3
2 4
3 2
3 3
3 4
```

### sdiff FILE...

各集合にしか含まれない要素だけからなる集合を取り出す。

```
$ seq 1 3 | sdiff - $(qsub seq 2 4)
1
4
```

## その他

### ifany \[-n ERRNO\] COMMAND \[ARG\]...

標準入力がない場合はCOMMANDを実行しない。
moreutilのifneと同じ。
-nオプションで入力がない場合の戻り値を指定する。
デフォルトは0。

```
$ : | ifany -n 1 echo a
$ echo $?
1
```

### isany

標準入力がない場合は1で復帰する。

```
$ : | isany; echo $?
1
$ echo a | isany; echo $?
a
0
```

### null COMMAND \[ARG\]...

出力をすべて捨てる。

```
$ null echo a
$ null eco a
$
```

### enull COMMAND \[ARG\]...

エラー出力を捨てる。

```
$ enull echo a
a
$ enull eco a
$
```

### ig \[-e ECODE\[,ECODE\]...\] COMMAND \[ARG\]...

復帰値を前のコマンドから変更しない。
例外を-eオプションで指定できる。

```
$ :; ig false; echo $?
0
$ :; ig -e 2,3,4 eval '(exit 3)'; echo $?
3
```

### substr VARIABLE STRING OFFSET \[LENGTH\]

STRINGをOFFSETからLENGTH分切り出してVARIABLEに代入する。

```
$ substr x 12345 2 2
$ echo $x
23
```

### dp COMMAND ARG... POSTFIX

ARGの最後の1つを複製し、POSTFIXをつけて実行する。

```
$ dp echo file .back
file file.back
```

### getop OPTSTRING \[ARG\]...

getoptsの第二引数をOPT固定にし、静かなモード限定にしたもの。
shiftするときにOPTINDを-1しなくても良い。
`--option=...`といったオプションもパースできる。


```
$ while getop a:bc: -a A -bcC -x --long --opt=arg 1 2 3; do echo "OPT=$OPT"; echo "OPTARG=$OPTARG"; done
OPT=a
OPTARG=A
OPT=b
OPTARG=
OPT=c
OPTARG=C
OPT=?
OPTARG=x
OPT=long
OPTARG=
OPT=opt=
OPTARG=arg
$ echo $OPTIND
6
```
