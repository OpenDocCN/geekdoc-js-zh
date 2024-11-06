# ループと反復処理

> 原文：[`jsprimer.net/basic/loop/`](https://jsprimer.net/basic/loop/)

この章では、while 文やfor 文などの基本的な反復処理と制御文について学んでいきます。

プログラミングにおいて、同じ処理を繰り返すために同じコードを繰り返し書く必要はありません。 ループやイテレータなどを使い、反復処理として同じ処理を繰り返し実行できます。

また、for 文などのような構文だけではなく、配列のメソッドを利用して反復処理を行う方法もあります。 配列のメソッドを使った反復処理もよく利用されるため、合わせて見ていきます。

## [](#while-statement)*while 文*

*while 文は`条件式`が`true`であるならば、反復処理を行います。

```
while (条件式) {
    実行する文;
} 
```

while 文の実行フローは次のようになります。 最初から`条件式`が`false`である場合は、何も実行せずwhile 文は終了します。

1.  `条件式` の評価結果が`true`なら次のステップへ、`false`なら終了

1.  `実行する文`を実行

1.  ステップ1へ戻る

次のコードでは`x`の値が10 未満であるなら、コンソールへ繰り返しログが出力されます。 また、`実行する文`にて、`x`の値を増やし`条件式`が`false`となるようにしています。

```
let x = 0;
console.log(`ループ開始前のxの値: ${x}`);
while (x < 10) {
    console.log(x);
    x += 1;
}
console.log(`ループ終了後のxの値: ${x}`); 
```

つまり、`実行する文`の中で`条件式`が`false`となるような処理を書かないと無限ループします。 JavaScriptにはより安全な反復処理の書き方があるため、while 文は使う場面が限られています。

安易にwhile 文を使うよりも、ほかの書き方で解決できないかを考えてからでも遅くはないでしょう。

## [](#infinite-loop)*[コラム] 無限ループ*

*反復処理を扱う際に、コードの書き間違いや条件式のミスなどから無限ループを引き起こしてしまう場合があります。 たとえば、次のコードは条件式の評価結果が常に`true`となってしまうため、無限ループが発生してしまいます。

```
let i = 1;
// 条件式が常にtrueになるため、無限ループする
while (i > 0) {
    console.log(`${i}回目のループ`);
    i += 1;
} 
```

無限ループが発生してしまったときは、あわてずにスクリプトを停止してからコードを修正しましょう。

ほとんどのブラウザは無限ループが発生した際に、自動的にスクリプトの実行を停止する機能が含まれています。 また、ブラウザで該当のスクリプトを実行しているページ（タブ）またはブラウザそのものを閉じることで強制的に停止できます。 Node.jsで実行している場合は`Ctrl + C`を入力し、終了シグナルを送ることで強制的に停止できます。

無限ループが発生する原因のほとんどは条件式に関連する実装ミスです。 まずは条件式の確認をしてみることで問題を解決できるはずです。

## [](#do-while-statement)*do-while 文*

*do-while 文はwhile 文とほとんど同じですが実行順序が異なります。

```
do {
    実行する文;
} while (条件式); 
```

do-while 文の実行フローは次のようになります。

1.  `実行する文`を実行

1.  `条件式` の評価結果が`true`なら次のステップへ、`false`なら終了

1.  ステップ1へ戻る

while 文とは異なり、必ず最初に`実行する文`を処理します。

そのため、次のコードのように最初から`条件式`を満たさない場合でも、 初回の`実行する文`が処理され、コンソールへ`1000`と出力されます。

```
const x = 1000;
do {
    console.log(x); // => 1000
} while (x < 10); 
```

この仕組みをうまく利用し、ループの開始前とループ中の処理をまとめて書くことができます。 しかし、while 文と同じくほかの書き方で解決できないかを考えてからでも遅くはないでしょう。

## [](#for-statement)*for 文*

*for 文は繰り返す範囲を指定した反復処理を書くことができます。

```
for (初期化式; 条件式; 増分式) {
    実行する文;
} 
```

for 文の実行フローは次のようになります。

1.  `初期化式` で変数の宣言

1.  `条件式` の評価結果が`true`なら次のステップへ、`false`なら終了

1.  `実行する文` を実行

1.  `増分式` で変数を更新

1.  ステップ2へ戻る

次のコードでは、for 文で1から10までの値を合計して、その結果をコンソールへ出力しています。

```
let total = 0; // totalの初期値は0
// for 文の実行フロー
// iを0で初期化
// iが10 未満（条件式を満たす）ならfor 文の処理を実行
// iに1を足し、再び条件式の判定へ
for (let i = 0; i < 10; i++) {
    total += i + 1; // 1から10の値をtotalに加算している
}
console.log(total); // => 55 
```

このコードは1から10までの合計を電卓で計算すればいいので、普通は必要ありませんね。 もう少し実用的なものを考えると、任意の数値の入った配列を受け取り、その合計を計算して返すという関数を実装すると良さそうです。

次のコードでは、任意の数値が入った配列を受け取り、その合計値を返す `sum` 関数を実装しています。 `numbers`配列に含まれている要素を先頭から順番に変数`total`へ加算することで合計値を計算しています。

```
function sum(numbers) {
    let total = 0;
    for (let i = 0; i < numbers.length; i++) {
        total += numbers[i];
    }
    return total;
}

console.log(sum([1, 2, 3, 4, 5])); // => 15 
```

JavaScriptの配列である`Array`オブジェクトには、反復処理のためのメソッドが備わっています。 そのため、配列のメソッドを使った反復処理も合わせて見ていきます。

## [](#array-foreach)*配列の`forEach`メソッド*

*配列には`forEach`メソッドというfor 文と同じように反復処理を行うメソッドがあります。

`forEach`メソッドでの反復処理は、次のように書けます。

```
const array = [1, 2, 3];
array.forEach(currentValue => {
    // 配列の要素ごとに呼び出される処理
}); 
```

JavaScriptでは、関数がファーストクラスであるため、その場で作った無名関数（名前のない関数）を引数として渡せます。

引数として渡される関数のことを**コールバック関数**と呼びます。 また、コールバック関数を引数として受け取る関数やメソッドのことを**高階関数**と呼びます。

```
const array = [1, 2, 3];
// forEachは"コールバック関数"を受け取る高階関数
array.forEach(コールバック関数); 
```

`forEach`メソッドのコールバック関数には、配列の要素が先頭から順���に渡されて実行されます。 つまり、コールバック関数の仮引数であるcurrentValueには、1から3の値が順番に渡されます。

```
const array = [1, 2, 3];
array.forEach(currentValue => {
    console.log(currentValue);
});
// 1
// 2
// 3
// と順番に出力される 
```

先ほどのfor 文の例と同じ数値の合計を返す`sum` 関数を`forEach`メソッドで実装してみます。

```
function sum(numbers) {
    let total = 0;
    numbers.forEach(num => {
        total += num;
    });
    return total;
}

sum([1, 2, 3, 4, 5]); // => 15 
```

`forEach`はfor 文の`条件式`に相当するものはなく、必ず配列のすべての要素を反復処理します。 変数`i`といった一時的な値を定義する必要がないため、シンプルに反復処理を書けます。

## [](#break-statement)*break 文*

*break 文は処理中の文から抜けて次の文へ移行する制御文です。 while、do-while、forの中で使い、処理中のループを抜けて次の文へ制御を移します。

```
while (true) {
    break; // *1 へ
}
// *1 次の文 
```

switch 文で出てきたものと同様で、処理中のループ文を終了できます。

次のコードでは配列の要素に1つでも偶数を含んでいるかを判定しています。

```
const numbers = [1, 5, 10, 15, 20];
// 偶数があるかどうか
let isEvenIncluded = false;
for (let i = 0; i < numbers.length; i++) {
    const num = numbers[i];
    // numが2で割り切れるなら偶数
    if (num % 2 === 0) {
        isEvenIncluded = true;
        break;
    }
}
console.log(isEvenIncluded); // => true 
```

1つでも偶数があるかがわかればいいため、配列内から最初の偶数を見つけたらfor 文での反復処理を終了します。 このような処理は、使い回せるように関数として実装するのが一般的です。

同様の処理をする `isEvenIncluded` 関数を実装してみます。 次のコードでは、break 文が実行され、ループを抜けた後にreturn 文で結果を返しています。

```
// 引数の`num`が偶数ならtrueを返す
function isEven(num) {
    return num % 2 === 0;
}
// 引数の`numbers`に偶数が含まれているならtrueを返す
function isEvenIncluded(numbers) {
    let isEvenIncluded = false;
    for (let i = 0; i < numbers.length; i++) {
        const num = numbers[i];
        if (isEven(num)) {
            isEvenIncluded = true;
            break;
        }
    }
    return isEvenIncluded;
}
const array = [1, 5, 10, 15, 20];
console.log(isEvenIncluded(array)); // => true 
```

return 文は現在の関数を終了させることができるため、次のように書くこともできます。 `numbers`に1つでも偶数が含まれていれば結果は`true`となるため、偶数の値が見つかった時点で`true`を返しています。

```
function isEven(num) {
    return num % 2 === 0;
}
function isEvenIncluded(numbers) {
    for (let i = 0; i < numbers.length; i++) {
        const num = numbers[i];
        if (isEven(num)) {
            return true;
        }
    }
    return false;
}
const numbers = [1, 5, 10, 15, 20];
console.log(isEvenIncluded(numbers)); // => true 
```

偶数を見つけたらすぐにreturnすることで一時的な変数が不要となり、より簡潔に書けました。

### [](#array-some)*配列の`some`メソッド*

*先ほどの`isEvenIncluded`関数は、偶数を見つけたら `true` を返す関数でした。 配列では`some`メソッドで同様のことが行えます。

`some`メソッドは、配列の各要素をテストする処理をコールバック関数として受け取ります。 コールバック関数が、一度でも`true`を返した時点で反復処理を終了し、`some`メソッドは`true`を返します。

```
const array = [1, 2, 3, 4, 5];
const isPassed = array.some(currentValue => {
    // テストをパスするとtrue、そうでないならfalseを返す
}); 
```

`some`メソッドを使うことで、配列に偶数が含まれているかは次のように書くことができます。 受け取った値が偶数であるかをテストするコールバック関数として`isEven`関数を渡します。

```
function isEven(num) {
    return num % 2 === 0;
}
const numbers = [1, 5, 10, 15, 20];
console.log(numbers.some(isEven)); // => true 
```

## [](#continue-statement)*continue 文*

*continue 文は現在の反復処理を終了して、次の反復処理を行います。 continue 文は、while、do-while、forの中で使うことができます。

たとえば、while 文の処理中で`continue`文が実行されると、現在の反復処理はその時点で終了されます。 そして、次の反復処理で`条件式`を評価するところからループが再開されます。

```
while (条件式) {
    // 実行される処理
    continue; // `条件式` へ
    // これ以降の行は実行されません
} 
```

次のコードでは、配列の中から偶数を集め、新しい配列を作り返しています。 偶数ではない場合、処理中のfor 文をスキップしています。

```
// `number`が偶数ならtrueを返す
function isEven(num) {
    return num % 2 === 0;
}
// `numbers`に含まれている偶数だけを取り出す
function filterEven(numbers) {
    const results = [];
    for (let i = 0; i < numbers.length; i++) {
        const num = numbers[i];
        // 偶数ではないなら、次のループへ
        if (!isEven(num)) {
            continue;
        }
        // 偶数を`results`に追加
        results.push(num);
    }
    return results;
}
const array = [1, 5, 10, 15, 20];
console.log(filterEven(array)); // => [10, 20] 
```

もちろん、次のように`continue`文を使わずに「偶数なら`results`へ追加する」という書き方も可能です。

```
if (isEven(number)) {
    results.push(number);
} 
```

この場合、条件が複雑になってきた場合にネストが深くなってコードが読みにくくなります。 そのため、ネストしたif 文のうるう年の例でも紹介したように、 できるだけ早い段階でそれ以上処理を続けない宣��をすることで、複雑なコードになることを避けています。

### [](#array-filter)*配列の`filter`メソッド*

*配列から特定の値だけを集めた新しい配列を作るには`filter`メソッドを利用できます。

`filter`メソッドには、配列の各要素をテストする処理をコールバック関数として渡します。 コールバック関数が`true`を返した要素のみを集めた新しい配列を返します。

```
const array = [1, 2, 3, 4, 5];
// テストをパスしたものを集めた配列
const filteredArray = array.filter((currentValue, index, array) => {
    // テストをパスするならtrue、そうでないならfalseを返す
}); 
```

先ほどのcontinue 文を使った値の絞り込みは`filter`メソッドを使うとより簡潔に書けます。 次のコードでは、`filter`メソッドを使って偶数だけに絞り込んでいます。

```
function isEven(num) {
    return num % 2 === 0;
}

const array = [1, 5, 10, 15, 20];
console.log(array.filter(isEven)); // => [10, 20] 
```

## [](#for-in-statement)*for...in 文*

*for...in 文はオブジェクトのプロパティに対して、反復処理を行います。^(1)

```
for (プロパティ in オブジェクト) {
    実行する文;
} 
```

次のコードでは`obj`のプロパティ名を`key`変数に代入して反復処理をしています。 `obj`には、3つのプロパティ名があるため３回繰り返されます （ループのたびに毎回新しいブロックを作成しているため、ループごとに定義する変数`key`は再定義エラーになりません。詳細は「関数とスコープ」の章の「ブロックスコープ」で解説します）。

```
const obj = {
    "a": 1,
    "b": 2,
    "c": 3
};
// 注記: ループのたびに毎回新しいブロックに変数 keyが定義されるため、再定義エラーが発生しない
for (const key in obj) {
    const value = obj[key];
    console.log(`key:${key}, value:${value}`);
}
// "key:a, value:1"
// "key:b, value:2"
// "key:c, value:3" 
```

オブジェクトに対する反復処理のためにfor...in 文は有用に見えますが、多くの問題を持っています。

JavaScriptでは、オブジェクトは何らかのオブジェクトを継承しています。 for...in 文は、対象となるオブジ��クトのプロパティを列挙する場合に、親オブジェクトまで列挙可能なものがあるかを探索して列挙します。 そのため、オブジェクト自身が持っていないプロパティも列挙されてしまい、意図しない結果になる場合があります。

安全にオブジェクトのプロパティを列挙するには、`Object.keys`メソッド、`Object.values`メソッド、`Object.entries`メソッドなどが利用できます。

先ほどの例である、オブジェクトのキーと値を列挙するコードはfor...in 文を使わずに書けます。 `Object.keys`メソッドは引数のオブジェクト自身が持つ列挙可能なプロパティ名の配列を返します。 そのためfor...in 文とは違い、親オブジェクトのプロパティは列挙されません。

```
const obj = {
    "a": 1,
    "b": 2,
    "c": 3
};
Object.keys(obj).forEach(key => {
    const value = obj[key];
    console.log(`key:${key}, value:${value}`);
});
// "key:a, value:1"
// "key:b, value:2"
// "key:c, value:3" 
```

また、for...in 文は配列に対しても利用できますが、こちらも期待した結果にはなりません。

次のコードでは、配列の要素が列挙されそうですが、実際には配列のプロパティ名が列挙されます。 for...in 文が列挙する配列オブジェクトのプロパティ名は、要素のインデックスを文字列化した"0"、"1"となるため、その文字列が`num`へと順番に代入されます。 そのため、数値と文字列の加算が行われ、意図した結果にはなりません。

```
const numbers = [5, 10];
let total = 0;
for (const num in numbers) {
    // 0 + "0" + "1" という文字列結合が行われる
    total += num;
}
console.log(total); // => "001" 
```

配列の内容に対して反復処理を行う場合は、for 文や`forEach`メソッド、後述するfor...of 文を使うべきでしょう。

このようにfor...in 文は正しく扱うのが難しいですが、代わりとなる手段が豊富にあります。 そのため、for...in 文の利用は避け、`Object.keys`メソッドなどを使って配列として反復処理するなど別の方法を考えたほうがよいでしょう。

## [](#for-of-statement)*[ES2015] for...of 文*

*最後にfor...of 文についてです。

JavaScriptでは、`Symbol.iterator`という特別な名前のメソッドを実装したオブジェクトを**iterable**と呼びます。 iterableオブジェクトは、for...of 文で反復処理できます。

iterableオブジェクトは反復処理時の動作が定義されたオブジェクトの総称で、反復処理時に次の返す値を定義しています。 for...of 文では、iterableオブジェクトから次の返す値を1つ取り出し、variableに代入して反復処理を行います。

```
for (variable of iterable) {
    実行する文;
} 
```

実はすでにiterableオブジェクトは登場していて、Arrayはiterableオブジェクトです。

次のようにfor...of 文で、配列から値を取り出して反復処理を行えます。 for...in 文とは異なり、インデックス値ではなく配列の値を列挙します。

```
const array = [1, 2, 3];
for (const value of array) {
    console.log(value);
}
// 1
// 2
// 3 
```

JavaScriptではStringオブジェクトもiterableです。 そのため、文字列を1 文字ずつ列挙できます。

```
const str = "𠮷野家";
for (const value of str) {
    console.log(value);
}
// "𠮷"
// "野"
// "家" 
```

そのほかにも、`TypedArray`、`Map`、`Set`、DOM NodeListなど、`Symbol.iterator`が実装されているオブジェクトは多いです。 for...of 文は、それらのiterableオブジェクトで反復処理に利用できます。

## [](#conclusion)*まとめ*

*在这一章中，我们比较了使用循环结构（如 for 循环）和使用数组方法进行迭代处理的方式。在循环结构（如 for 循环）中，可以使用 continue 语句和 break 语句，但在数组方法中无法使用它们。另一方面，数组方法的不同之处在于不需要管理临时变量，并且可以将处理写成回调函数。

这两种方法在迭代处理中都经常被使用。由于没有哪一种方法是绝对优越的，因此掌握这两种方法都是重要的。此外，在“数组”章节中我们也会对数组方法进行详细解释。

> ¹. `for...in`语句列举属性的顺序在 ES2019 之前是实现相关的，但在 ES2020 中已经确定了列举的顺序。 ↩************
