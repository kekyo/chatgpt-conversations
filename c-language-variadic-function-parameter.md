## 背景

chibicc-cil移植時の疑問から。もとのchibiccはネイティブスタックを直接扱うので、
関数宣言と異なる引数群をスタックに積んでも支障ない（cdeclを前提とした場合）が、
chibicc-cilでは、`System.ArgIterator`に強い制約がある関係で、`ArgIterator`の使用を放棄しようとしていて、
可変引数だった場合は独自の可変引数群情報を保持するクラス(`__va_arglist`)のインスタンスを渡すようにした。

そうすると、引数が完全に省略されている場合でも、`__va_arglist`を渡さなければならず、
これはデバッグ時に何度も問題の特定に苦労している（一見正しい引数を渡しているように見えるが間違っているような場合）ので、
問題を軽減する方法がないかどうかを探していた。

最終的な仕様書の言及から、chibicc (x64) は引数があろうがなかろうが、スタックに積まれる引数のレイアウトが変わることはないため、
引数がない == 引数がvoidになっている ==> variadic functionとする、というようにしたのではないかと推測しました。

-----

## kekyo

C言語のコンパイラ実装を解析していて、関数宣言のパラメータ部が省略された場合の対応に疑問があるため質問です。
関数宣言のパラメータ部が完全に省略されている場合、例えば `int foo()` のように、引数を一つも取らない場合、
コンパイラの実装はこれをvariadic functionと認識していました。
その処理は同時に、関数宣言のパラメータ部を明示的に `int foo(void)` のように、voidと指定した場合も同様の処理を行います。

省略された場合はともかく、voidと明示した場合でもvariadic functionのみなすのは何故なのか、推測してください。

## chatgpt

C言語において、引数が完全に省略された場合の関数宣言は、引数の型が不定である可変個引数関数 (variadic function) として扱われます。
一方、引数が存在する場合は、引数の型が明示的に指定されなければならず、void と指定することで引数を取らない関数を宣言できます。

これはC言語の言語仕様によるもので、C言語では可変個引数関数をサポートしています。
可変個引数関数は、引数の個数が固定でない関数であり、stdarg.h ヘッダファイルに定義されたマクロを使用して引数の処理を行います。
可変個引数関数を宣言する際には、引数の最後に ... を指定します。
しかし、引数が完全に省略された場合でも、C言語の仕様では可変個引数関数として扱うことになっているため、コンパイラの実装もそれに従っているのだと思われます。

また、引数が存在する場合には、void と指定することで引数を取らない関数を宣言できますが、
これは明示的に引数の個数が0であることを示すものであり、可変個引数関数ではありません。
したがって、コンパイラの実装においても、明示的に void を指定した場合は可変個引数関数ではなく、引数を取らない関数として扱う必要があります。

## kekyo

この使用を担保する、C言語仕様書の該当するセクションを示して下さい

## chatgpt

C言語の仕様書であるC11規格書の6.7.6.3項 "Function declarators"に、関数宣言の形式について記述があります。

この項の4番目の節には、次のように記述されています。

> An empty list in a function declarator that is part of a definition of that function specifies that the function has no parameters.
> The empty list in a function declarator that is not part of a definition of that function specifies that no information about
> the number or types of the parameters is supplied.

つまり、関数宣言のパラメータ部が完全に省略された場合、関数定義の一部である場合はパラメータがないことを示し、
関数定義ではない場合はパラメータの数や型に関する情報が提供されていないことを示します。

また、次の5番目の節には、次のように記述されています。

> If the declarator includes a parameter type list, the list specifies the types of the parameters;
> the parameter names are optional.

つまり、パラメータ部が明示的に指定されている場合には、パラメータの型が指定され、パラメータ名は省略可能であることが規定されています。

さらに、6番目の節には、次のように記述されています。

> If the declarator includes an empty parameter type list, the function has no parameters.

つまり、void が明示的に指定された場合は、引数のない関数であることが規定されています。
これは、可変個引数関数ではなく、明示的に引数の数が0であることを示すものです。
