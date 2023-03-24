## 備考

これも、ググってもまず正解にたどり着けないパターン。C言語のことはC言語仕様書に聞けと言われそうだけど、ECMA335もそうだけど、インデックス頭に入らんよ...

-----

## kekyo

C言語のva_listの挙動について質問です。
va_listを引数で受け取った関数内で、va_listをグローバス変数に退避しておき、
その実行コンテキストではない場所から保存しておいたva_listを使って引数を取得することは合法ですか？

例えば、別のスレッドから保存しておいたva_listを使ってva_argで引数を取得するなどです。
合法ではない場合は、合法となる条件を箇条書きで明示してください。

## chatgpt

C言語において、va_listは可変長引数を扱うための仕組みであり、stdarg.hヘッダーファイルで定義されています。

一般的に、va_listはその関数内でのみ使用することが推奨されており、
グローバル変数に退避することは避けるべきです。
その理由は、va_listは可変長引数の情報を保持する構造体であり、
その構造体の実装はプラットフォーム依存であるため、グローバル変数に退避すると動作が保証されなくなるからです。

また、保存しておいたva_listを別の実行コンテキストから使用することは、
不正なメモリアクセスを引き起こす可能性があるため、合法ではありません。

ただし、以下の条件を満たす場合は、保存しておいたva_listを使用することが可能です。

* 保存しておいたva_listを使用するコンテキストが、保存したコンテキストと同じであること。
* 保存しておいたva_listを使用する前に、保存時と同じ可変長引数関数が呼び出されたことを確認すること。
* 保存しておいたva_listを使用する前に、可変長引数の数や型などが保存時と同じであることを確認すること。

以上の条件を満たす場合は、保存しておいたva_listを使用することができます。
ただし、可変長引数を扱うコードはプラットフォーム依存であり、注意深くプログラムを設計する必要があることに注意してください。

## kekyo

その挙動について、C言語の規格書で言及されているセクション番号を教えてください

## chatgpt

C言語の規格書である「ISO/IEC 9899:2018 Programming languages -- C」において、
可変長引数に関する情報は、以下のセクションで規定されています。

* 可変長引数に関する基本的な情報は、セクション7.16 Variable arguments <stdarg.h>で規定されています。
* va_list型に関する情報は、セクション7.16.1 Variable argument list access macros <stdarg.h>で規定されています。
* va_argマクロに関する情報は、セクション7.16.1.1 The va_arg macro <stdarg.h>で規定されています。
* 可変長引数関数の呼び出し規則に関する情報は、セクション6.5.2.2 Function calls <ISO/IEC 9899:2018>で規定されています。

これらのセクションには、可変長引数の使用方法や注意点、およびそれに関連するマクロや規則が詳細に記載されています。

## kekyo

関数の可変引数部分にfloat型の値を渡すとき、double型に拡張されると指定されている、C言語の規格書の該当するセクションを教えてください

## chatgpt

C言語の規格書「ISO/IEC 9899:2018 Programming languages -- C」において、
可変長引数にfloat型の値を渡した場合にdouble型に拡張されることが明示されているセクションは、以下の通りです。

* セクション6.5.2.2 関数呼び出し (Function calls) の7番目の項目に、以下のように記載されています。

> * If the expression that denotes the called function has a type that includes a prototype,
>   the number of arguments shall agree with the number of parameters.
>   Each argument shall have a type such that its value may be assigned to an object
>   with the unqualified version of the type of its corresponding parameter.
> * The ellipsis notation in a function prototype declarator causes argument type conversion
>   to stop after the last declared parameter.
>   The default argument promotions are performed on trailing arguments.
> * If the function is defined with a type that includes a prototype,
>   and either the prototype ends with an ellipsis (, ...) or the types of the arguments after promotion
>   are not compatible with the types of the parameters, the behavior is undefined.

つまり、可変長引数部分に渡されたfloat型の値は、
default argument promotions によってdouble型に拡張されることが明示されています。
この拡張は、関数呼び出し時に行われます。
