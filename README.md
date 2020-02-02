# dashutil

dashを少しだけ便利にするutility

## 一時ファイルを作りたくない人向けのコマンド

### rdcmd COMMAND...

bashのプロセス置換(read)に対応するコマンド。

```
$ diff $(rdcmd echo a) $(rdcmd echo b)
> a
< b
```
### qee COMMAND...

bashのプロセス置換(write)に対応するコマンド。
moreutilのpeeの1コマンド版。

```
$ echo a | qee grep a | tr a b
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
$ pipe -a fuga 'exit 1' 'exit 2' ture; echo $?; echo $fuga
0
1 2 0
$ pipe 'exit 1' 'exit 2' true; echo $?
2
```
### either

## リソース管理用のコマンド

### defer COMMAND...

シグナルトラップを追加する。

```
$ (touch file; defer rm file)
$ ls file
ls: cannot access 'file': No such file or directory
```

### tmpf COMMAND...

一時ファイルを作ってコマンドを実行する。

```
$ tmpf fval 'echo a > $1; cat $1'
a
```

## 疑似データ構造

### data

- data VARIABLE \[= \[KEY VALUE\]...\]
- data DATA set KEY VALUE
- data DATA get KEY COMMAND...
- data DATA get-default KEY VALUE COMMAND...
- data DATA exists KEY
- data DATA values COMMAND...
- data DATA keys COMMAND...
- data DATA pairs COMMAND...
- data DATA rm KEY...

```
$ data hoge = \
>   apple  1
>   orange 2
$ data $hoge get orange echo
2
$ data $hoge get cake echo

$ data $hoge get-default cake 0 echo
0
$ data $hoge values echo
1
3
$ data $hoge pairs echo
apple 1
orange 2
```

## 上記のコマンドと組み合わせるためのコマンド

### fval QUOTED \[ARG\]...

### quote \[-v VARIABLE\] STRING...

printfの%qに対応するコマンド。

```
$ quote 'a b' "' '"
'a b' \'' '\'
$ quote -v foge 'echo "|"'
$ eval $foge
|
```

### rot COMMAND...

### rotn NUMBER COMMAND...

### cutin NUMBER COMMAND...

## その他

### null COMMAND...

### exists VARIABLE




