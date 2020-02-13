# dashutil

dashを少しだけ便利にする車輪の再発明utility

## 一時ファイルを作りたくない人向けのコマンド

### psub

fishのpsubと同じ。
bashのプロセス置換(read)に対応するコマンド。

```
$ diff $(echo a | psub) $(echo b | psub)
> a
< b
```
### qee COMMAND \[ARG\]...

bashのプロセス置換(write)に対応するコマンド。
moreutilのpee。
 \[ARG\]
```
$ echo a | qee 'tr a c' | tr a b
c
b
```

### pipe \[-r|-l|-a VARIABLE\] QUOTED...

パイプの復帰値のポリシーを変更する。

```
$ pipe -r 'exit 1' 'exit 2' true; echo $?
2
$ pipe -l 'exit 1' 'exit 2' ture; echo $?
1
$ pipe -a fuga 'exit 1' 'exit 2' ture; echo $?; echo $fuga
0
1 2 0
$ pipe 'exit 1' 'exit 2' true; echo $?
0

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
echoa: コマンドが見つかりません
$ maybe echo a | lift echob | lift echo c | decxt
echob: コマンドが見つかりません
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
echoa: コマンドが見つかりません
$ msg=`either echo a | lift echob | lift echo c | decxt 2>&1`
$ echo "$msg"
echob: コマンドが見つかりません
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
echob: コマンドが見つかりません
c
$ either echoa | noop echo b | lift echo c | decxt
echoa: コマンドが見つかりません
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
$ either echoa | noop echo b | lift echo c | {
> readcxt x
> case $x
> in 0) cat -
> ;; *) echo "ERROR: command failed, ecode=$x, msg=$(cat)"
> esac; }
ERROR: command failed, ecode=127, msg=echoa: コマンドが見つかりません
```

## リソース管理用のコマンド

### tmpf COMMAND \[ARG\]...

一時ファイルを作って引数に渡してコマンドを実行する。

```
$ tmpf fval 'echo a > $1; cat $1'
a
```
### trapend [QUOTED]

以下と同義

```
$ trap QUOTED 0 1 2 3 15
```

## 動的変数

### var NAME VALUE

変数名がNAMEである変数にVALUEを代入する。

```
$ for i in 1 2 3 4; do
>   var hoge$i $i
> done
$ for i in 1 2 3 4; do
>   withvar hoge$i echo
> done
1
2
3
4
```
### withvar VARIABLE COMMMAND \[ARG\]...

VARIABLEの値をCOMMMANDの最終引数に渡して実行する。

### readvar VARIABLE NAME

変数名がNAMEである変数の値をVARIABLEに代入する。

```
$ a1=hoge
$ n=1
$ read b a$n
$ echo $b
hoge
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
$ jsondir dir -s '\031' << EOF
{ "array": [ 1, 2 ],
  "hash": { "array": [ "A", "B"]},
  "num": 1 }
EOF
$ find dir -type f
dir/num
dir/array/2
dir/array/1
dir/hash/array/2
dir/hash/array/1
$ read x dir/hash/array/1
$ echo $x
A
```

### withread FILE COMMAND \[ARG\]...

FILEをreadした値をCOMMANDの最終引数に与えて実行する。

```
$ cat file
A
$ withread file echo
A
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
$ quote foge '" " "a b"'
$ eval "printf '%s, ' $hoge"
 , a b
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

## その他

### ifany \[-n ERRNO\] COMMAND \[ARG\]...

標準入力がない場合はCOMMANDを実行しない。
moreutilのifneと同じ。
-nオプションで入力がない場合の戻り値を指定する。
デフォルトは0。

### null COMMAND \[ARG\]...

出力をすべて捨てる。

```
$ null echo a
$ null eco a
$
```

### enull COMMAND \[ARG\]...

エラー出力を捨てる。

### ig COMMAND \[ARG\]...

復帰値を前のコマンドから変更しない。

### substr VARIABLE STRING OFFSET \[LENGTH\]

STRINGをOFFSETからLENGTH分切り出してVARIABLEに代入する。

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
OPT=x
OPTARG=
OPT=long
OPTARG=
OPT=opt=
OPTARG=arg
$ echo $OPTIND
6
```
