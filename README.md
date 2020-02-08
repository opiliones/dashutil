# dashutil

dashを少しだけ便利にするutility

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
$ echo a | qee 'grep a' | tr a b
a
b
```

### pipe \[-r|-l|-a VARIABLE\] QUOTED...

パイプの復帰値のポリシーを変更する。

```
$ pipe -r 'exit 1' 'exit 2' true; echo $?
2
$ pipe -l 'exit 1' 'exit 2' ture; echo $?
1
$ pipe 'exit 1' 'exit 2' true; echo $?
0
```
### pipestatus VARIABLE QUOTED...

bashのPIPESTATUS相当。VARIABLEに各コマンドの復帰値を代入する。

```
$ pipestatus -a fuga 'exit 1' 'exit 2' ture; echo $?; echo $fuga
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
> ;; *) echo "ERROR: command failed, ecode=$?, msg=$(cat)"
> esac; }
ERROR: command failed, ecode=127, msg=echoa: コマンドが見つかりません
```

## リソース管理用のコマンド

### defer COMMAND \[ARG\]...

シグナルトラップを追加する。

```
$ (touch file; defer rm file; touch file2; defer rm file2)
$ ls file file2
ls: cannot access 'file': No such file or directory
ls: cannot access 'file2': No such file or directory
```

### tmpf COMMAND \[ARG\]...

一時ファイルを作って引数に渡してコマンドを実行する。

```
$ tmpf fval 'echo a > $1; cat $1'
a
```

## 動的変数

### var NAME VALUE

変数名がNAMEである変数にVALUEを代入する。

### withvar VARIABLE COMMMAND \[ARG\]...

VARIABLEの値をCOMMMANDの最終引数に渡して実行する。

### readvar VARIABLE NAME

変数名がNAMEである変数の値をVARIABLEに代入する。

## 再帰的データ

### jsondir DIR \[-s CHAR\]

json形式のデータを標準入力に受け取って
データに対応するディレクトリ構成を作成する。
sオプションにはキー内に"/"が存在した場合の
置換文字を指定する。デフォルトは"%"である。

```
$ jsondir json -s '\031' << EOF
{ "array": [ 1, 2 ],
  "hash": { "array": [ "A", "B"]},
  "num": 1 }
EOF
$ find json -type f
array/1
array/2
hash/array/1
hash/array/2
num
$ read x hash/array/1
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

### quote \[-v VARIABLE\] STRING...

printfの%qに対応するコマンド。

```
$ quote 'a b' "' '"
'a b' \'' '\'
$ quote -v foge '" " "a b"'
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

### cutin NUMBER COMMAND \[ARG\]...

最後の引数をNUMBERの引数に持ってくる。

```
$ cutin 2 echo 1 2 3
1 3 2
```

## その他

### ifany \[-n ERRNO\] COMMAND \[ARG\]...

標準入力がない場合はCOMMANDを実行しない。
moreutilのifneと同じ。

### null COMMAND \[ARG\]...

出力をすべて捨てる。

```
$ null echo a
$ null eco a
$
```

### enull COMMAND \[ARG\]...

エラー出力を捨てる。

### exists VARIABLE...

変数が存在するかどうかを復帰値で返す。

```
$ exists a && echo $a
$ a=yes
$ exists a && echo $a
yes
```

