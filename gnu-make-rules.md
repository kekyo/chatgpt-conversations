## 備考

Makefileのおきもちがわからない。全然うまく動かない...

あまり有用ではなかったけど、`SUFFIXES:`は良かった。
どうも組み込みのルールが悪さをするようで、これを先頭に書いておけば、思った通りになることがわかった。

ルール判定を調べようとして-dを指定したけど、出力が多すぎて、原因を探すのが困難で微妙だった。

なお、提示されたMakefileでは全然うまくいかない定期。

----

## kekyo

GNU makeのMakefileについての質問です。以下のようなMakefileを書きました。
この中で、(1) と (2)のルールを、ワイルドカードを使ったルールに変更してください。
ソースとなるcodegen.cやparse.cは、ほかにも同じ拡張子cを持ったファイルが複数存在するためです。

```makefile
stage2/libc-bootstrap.dll: $(LIBC)
	mkdir -p stage2
	cp $(LIBC) stage2

# (1)
stage2/codegen.s: codegen.c chibicc self.py
	mkdir -p stage2
	./self.py chibicc.h codegen.c > stage2/codegen.c
	./chibicc -o stage2/codegen.s stage2/codegen.c

# (2)
stage2/parse.s: parse.c chibicc self.py
	mkdir -p stage2
	./self.py chibicc.h parse.c > stage2/parse.c
	./chibicc -o stage2/parse.s stage2/parse.c

stage2/chibicc.exe: stage2/*.s stage2/libc-bootstrap.dll
	$(AS) $(ASFLAGS) -r stage2/libc-bootstrap.dll -o $@ stage2/*.s
```

## chatgpt

以下のように、ワイルドカードを使ったルールに変更できます。

```makefile
stage2/libc-bootstrap.dll: $(LIBC)
	mkdir -p stage2
	cp $(LIBC) stage2

# (1)
stage2/%.s: %c chibicc self.py
	mkdir -p stage2
	./self.py chibicc.h $< > stage2/$<
	./chibicc -o $@ stage2/$<

# (2)
stage2/%.s: %c chibicc self.py
	mkdir -p stage2
	./self.py chibicc.h $< > stage2/$<
	./chibicc -o $@ stage2/$<

stage2/chibicc.exe: stage2/*.s stage2/libc-bootstrap.dll
	$(AS) $(ASFLAGS) -r stage2/libc-bootstrap.dll -o $@ stage2/*.s
```

ここで、%はワイルドカードで、%.cというパターンは、任意の.cファイルに一致します。
したがって、stage2/%.sというルールは、stage2ディレクトリにある任意の.cファイルに対してマッチします。
$<は、ルールの最初の依存関係である.cファイルを表します。

## kekyo

GNU makeのルールをデバッグするのに役立つコマンドラインオプションはありますか？

## chatgpt

はい、makeには、ルールをデバッグするためのいくつかのコマンドラインオプションがあります。
以下に、よく使用されるものを紹介します。

* -nまたは--just-print: 実際にコマンドを実行せず、どのコマンドが実行されるかを表示します。
* -dまたは--debug: デバッグ出力を表示します。詳細な情報を提供し、どのルールがどのように評価されているかを表示します。
* -pまたは--print-data-base: Makefileデータベースの内容を表示します。
* -rまたは--no-builtin-rules: 組み込みのルールを無効にします。
* -Rまたは--no-builtin-variables: 組み込みの変数を無効にします。
* -jまたは--jobs: マルチプロセッシングを有効にし、同時に実行されるジョブ数を指定します。

これらのオプションを使用することで、Makefileのルールがどのように評価され、どのようにコマンドが実行されるかを理解することができます。

## kekyo

-rによる組み込みのルール無視を、Makefile中に指定することは出来ますか？

## chatgpt

はい、Makefile中で組み込みのルールを無効にすることができます。
それには、.SUFFIXESという特殊なマクロを使用します。
このマクロは、Makefileで使用する接尾辞を定義し、組み込みルールを無効にすることができます。

以下は、.SUFFIXESを使用して、Makefile中で組み込みのルールを無効にする例です。

```makefile
.SUFFIXES:
```

上記の例では、接尾辞を何も定義していませんが、.SUFFIXES:と定義することで、すべての組み込みルールを無効にすることができます。

また、特定の接尾辞を無効にする場合は、次のようにします。

```makefile
.SUFFIXES: .o .c
```

上記の例では、.oおよび.cの接尾辞を定義しています。
この場合、Makefile内で.oおよび.cの接尾辞を持つファイルを使用する場合、組み込みルールは無効になります。

-----

## 別のセッションで聞きなおした

が、イマイチ良く分からん... なんかもうkekyo makeを作りたくなってきた（違

## kekyo

GNU makeのMakefileで、ワイルドカードを指定した記述の方法がわかりません。
`%.exe: %.c` というルールを指定する場合に、追加の依存ファイルを指定したい場合、
`%.exe: %.c foo.txt` のように指定すると、`Circular dependency dropped`という警告が発生します。
これは何故ですか？正しく指定するにはどのように記述しますか？

## chatgpt

Circular dependency droppedという警告が発生するのは、Makefile内に循環依存関係が含まれていることを意味します。
これは、あるファイルが自分自身またはその依存関係に依存している場合に発生する可能性があります。
Makeは、循環依存関係を解決することができないため、警告を出してターゲットをスキップします。

%.exe: %.c foo.txtのように書くと、%.exeが%.cとfoo.txtに依存しているという意味になります。
しかし、%.cは%.exeに依存しているため、循環依存関係が発生します。

正しく指定するには、%.exe: %.cと%.exe: foo.txtという2つのルールを定義する必要があります。
次のように記述することができます。

```makefile
%.exe: %.c
    gcc -o $@ $<

%.exe: foo.txt
```

これにより、%.exeは%.cまたはfoo.txtのどちらかに依存していることになります。
2つのルールが循環依存関係を持たないため、Circular dependency droppedの警告が発生しなくなります。

## kekyo

上記のルールで、それぞれのファイルがサブディレクトリ `test`に存在する場合は、どのように記述すればよいですか？

## chatgpt

testサブディレクトリ内にあるファイルに対してルールを適用する場合は、
%.cやfoo.txtの前にtest/を付ける必要があります。
また、ターゲットの名前にもtest/を付ける必要があります。
例えば、testサブディレクトリ内のfoo.txtに依存する場合、次のように記述できます。

```makefile
test/%.exe: test/%.c
    gcc -o $@ $<

test/%.exe: test/foo.txt
```

$<は、最初の依存関係である%.cまたはfoo.txtを表します。
$@は、ターゲットの名前であるtest/%.exeを表します。
testサブディレクトリが存在しない場合は、Makefile内でtestサブディレクトリを作成するルールを追加する必要があります。

## kekyo

`test/%.exe: test/foo.txt` という依存関係を記述した場合、
`test/foo.txt` をtouchで更新すると、`test/%.exe: test/%.c`は再度実行されますか？

## chatgpt

test/%.exe: test/foo.txtという依存関係を記述した場合、test/foo.txtをtouchで更新した場合、
test/%.exe: test/%.cは再度実行されません。

Makeは、ターゲットとその依存関係のタイムスタンプを比較して、ターゲットを再構築する必要があるかどうかを決定します。
test/foo.txtをtouchで更新した場合、test/%.exeのタイムスタンプは変更されますが、
test/%.cのタイムスタンプは変更されないため、test/%.cは再度実行されません。

ただし、test/foo.txtに依存する別のターゲットがある場合、そのターゲットは再構築されます。
また、test/foo.txtに依存するターゲットを強制的に再構築するには、makeコマンドに-Bオプションを付けて実行することができます。

## kekyo

`test/foo.txt`が更新された場合に、`test/%.exe: test/%.c`が実行されるようにするには、
どのようにルールを記述すればよいですか？

## chatgpt

test/foo.txtが更新された場合に、test/%.exe: test/%.cが再度実行されるようにするには、
test/foo.txtを依存関係として直接指定するのではなく、中間的なターゲットを作成して、
test/%.cとtest/foo.txtの両方に依存するようにする必要があります。

例えば、test/%.exeの実行ファイルを作成する前に、test/%.cから中間ファイルtest/%.oを作成するようにします。
そして、test/%.oは、test/%.cとtest/foo.txtの両方に依存するようにします。
そのため、test/foo.txtが更新された場合、test/%.oも更新され、結果的にtest/%.exeも再度実行されることになります。

以下は、この方法で記述したMakefileの例です。

```makefile
# ターゲット名を `test/%.exe` にする
test/%.exe: test/%.o
    gcc -o $@ $<

# 中間ファイル `test/%.o` を作成する
test/%.o: test/%.c test/foo.txt
    gcc -c -o $@ $<

# `test/foo.txt`をターゲットとして直接指定しない
.PHONY: test/foo.txt
```

ここでは、.PHONYターゲットを使用して、test/foo.txtをターゲットとして直接指定しないようにしています。
.PHONYターゲットは、そのターゲット名をファイル名として持たない（存在しない）ターゲットに対して使用されます。
これにより、test/foo.txtがファイルであっても、ターゲットとして直接指定されることがなくなります。

## kekyo

以下のように記述した方がシンプルな気がしますが、そうしないのは、何故ですか？

```makefile
test/%.exe: test/%.c test/foo.txt
    gcc -o $@ $<

.PHONY: test/foo.txt
```

## chatgpt

ご提案のように、直接test/foo.txtを依存関係に指定することもできます。
しかし、この場合、test/foo.txtが更新された場合に、test/%.exeの再実行が正しく行われない可能性があります。

たとえば、test/%.exeとtest/foo.txtが同じディレクトリにある場合、以下のようにファイルを更新すると、
test/%.exeは再度実行されません。

```bash
touch test/foo.txt
```

これは、Makeが依存関係のタイムスタンプを見て、test/%.cよりもtest/foo.txtが新しい場合にのみ再実行するためです。
しかし、test/%.cよりも古くても、test/foo.txtが変更された場合には、test/%.exeも再実行する必要があるため、
この方法は信頼性に欠けます。

一方、中間ファイルを使用する方法は、依存関係の更新状況に関係なく、必要な再実行が行われることが保証されます。
また、Makeの慣習に従って中間ファイルを使用する方法は、より汎用的で明確な記述方法であると考えられます。

## kekyo

この問題を、提案のように、.PHONYを使用して解決するのは一般的な方法ですか？
又は目標を達成する別の一般的なルール記述方法はありますか？
つまり、生成ルールを一般化しつつ、その生成に依存するファイル群を指定可能で、
その依存ファイルが更新された場合はルールが実行される、と言うような条件です。

## chatgpt

.PHONYを使用する方法は、正確にはこの問題を解決するための一般的な方法ではありません。.PHONYは、ファイルが存在しないことを明示的に指定するための方法であり、Makeにそのファイルをターゲットとして処理しないように指示するために使用されます。一方、ここでの問題は、依存関係の更新状況に関係なく、必要な再実行を行うことができるようにすることです。

一般的な解決策としては、中間ファイルを使用する方法があります。中間ファイルは、ターゲットを生成するために一時的に作成されるファイルであり、ターゲットと同様にタイムスタンプが更新されます。中間ファイルを使用することで、依存関係の更新状況に関係なく、必要な再実行を保証することができます。

以下は、中間ファイルを使用してルールを記述する例です。

```makefile
SRCDIR := test
OBJDIR := obj

$(OBJDIR)/%.o: $(SRCDIR)/%.c
    gcc -c -o $@ $<

$(OBJDIR)/%.exe: $(OBJDIR)/%.o $(SRCDIR)/foo.txt
    gcc -o $@ $<

.PHONY: all
all: $(OBJDIR)/main.exe
```

この例では、中間ファイルとして、.oファイルを使用しています。
.oファイルは、.cファイルからコンパイルされ、ターゲットと同様にタイムスタンプが更新されます。
そして、.exeファイルの生成ルールでは、中間ファイルとして.oファイルを使用しています。
このように、中間ファイルを使用することで、依存関係の更新状況に関係なく、必要な再実行を保証することができます。
