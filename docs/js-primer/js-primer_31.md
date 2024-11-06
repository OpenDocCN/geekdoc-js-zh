# 日期

> 原文：[`jsprimer.net/basic/date/`](https://jsprimer.net/basic/date/)

本章将学习如何在 JavaScript 中处理日期和时间的[Date](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Date)。

## [](#date-object)*日期对象*

*`Date`对象与`String`、`Array`等一样，是在 ECMAScript 中定义的内置对象。

通过实例化`Date`对象，可以获得表示特定时间的对象。 在`Date`中，“时间”以相对毫秒形式保留，以 UTC（协调世界时）的 1970 年 1 月 1 日 0 时 0 分 0 秒为基准。 本章将这个毫秒值称为“时间值”。 `Date`对象的实例各自具有一个时间值，并提供用于处理日期、小时和分钟等的方法。

### [](#create-instance)*创建实例*

*`Date`对象的实例始终使用 new 运算符创建。 创建`Date`对象的实例有两种主要方法。 一种是实例化当前时间，另一种是实例化任意时间。

#### [](#instance-current-time)*实例化当前时间*

*如果在 new 时不传递构造函数参数，则创建的实例将表示当前时间。 如果只需要当前时间的时间值而不是`Date`对象的实例，则可以使用`Date.now`方法的返回值。 创建的实例的时间值可以通过`getTime`方法获取。 此外，使用`toISOString`方法可以将时间转换为 UTC 的[ISO 8601](https://ja.wikipedia.org/wiki/ISO_8601)格式的字符串�� ISO 8601 是一种国际标准的字符串格式，表示时间和时区信息，例如`2006-01-02T15:04:05.999+09:00`。 由于这种格式易于理解，因此被广泛使用。

```
// 現在の時刻を表すインスタンスを作成する
const now = new Date();
// 時刻値だけが欲しい場合にはDate.nowメソッドを使う
console.log(Date.now());

// 時刻値を取得する
console.log(now.getTime());
// 時刻をISO 8601 形式の文字列で表示する
console.log(now.toISOString()); 
```

#### [](#instance-any-time)*实例化任意时间*

*通过传递构造函数参数，可以创建表示任意时间的实例。 `Date`的构造函数根据传递的数据类型和参数来确定时间的指定方法。 `Date`支持以下 3 种参数。

+   传递时间值

+   传递表示时间的字符串

+   通过数字分别传递时间部分（年、月、日等）

第一种方法是在构造函数中传递表示毫秒的数字类型参数时应用。 将传递的数字视为相对于 UTC 的 1970 年 1 月 1 日 0 时 0 分 0 秒的时间值。 由于此方法的基准时间和时区是固定的，因此不会受执行环境时区的影响，因此很安全。 因此，与其他两种方法不同，不需要考虑时区。

```
// 時刻のミリ秒値を直接指定する形式
// 1136214245999はUTCにおける"2006 年 1 月 2 日 15 時 04 分 05 秒 999"を表す
const date = new Date(1136214245999);
// 末尾の'Z'はUTCであることを表す
console.log(date.toISOString()); // => "2006-01-02T15:04:05.999Z" 
```

第二种方法是在传递字符串类型的参数时应用。 当传递符合[RFC2822](https://www.rfc-editor.org/rfc/rfc2822#section-3.3)或[ISO 8601](https://ja.wikipedia.org/wiki/ISO_8601)格式的字符串时，将解析该字符串并使用得到的时间值创建`Date`的实例。

下面的代码将传递 ISO 8601 格式的字符串以创建`Date`的实例。 如果字符串包含时区，则将计算该时区的时间值。 如果无法从字符串中读取时区，则需要注意根据执行环境的时区计算时间值。 此外，由于不同的浏览器可能返回不同的结果，因此要注意 ISO 8601 格式以外的字符串解析。

```
// UTCにおける"2006 年 1 月 2 日 15 時 04 分 05 秒 999"を表すISO 8601 形式の文字列
const inUTC = new Date("2006-01-02T15:04:05.999Z");
console.log(inUTC.toISOString()); // => "2006-01-02T15:04:05.999Z"

// 上記の例とは異なり、UTCであることを表す'Z'がついていないことに注意
// Asia/Tokyo(+09:00)で実行すると、UTCにおける表記は9 時間前の06 時 04 分 05 秒になる
const inLocal = new Date("2006-01-02T15:04:05.999");
console.log(inLocal.toISOString()); // "2006-01-02T06:04:05.999Z" (Asia/Tokyoの場合) 
```

第三种方法是逐个指定时间，例如年、月、日等的部分数字。

```
new Date(year, month, day, hour, minutes, seconds, milliseconds); 
```

当向构造函数传递两个以上的参数时，将使用此方法创建`Date`实例。 表示月内日期的第三个参数（`day`）及其后的参数是可选的。 如果省略这些参数，每个参数的默认值为`0`，但是表示日期（`day`）的第三个参数的默认值为`1`。 此外，要注意月份（`month`）的第二个参数对应于 1 月的`0`，并且月份是以`0`到`11`的数字指定的。

与前述的两种方法不同，此方法无法指定时区。 传递的数字始终被视为本地时区的时间。

由于结果取决于执行环境，基本上不应该使用这种方法。

如果要逐个指定时间，则最好使用[Date.UTC](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Date/UTC)方法。 参数的格式相同，但`Date.UTC`方法将传递的数字视为 UTC 时间，并返回该时间值。

```
// 実行環境における"2006 年 1 月 2 日 15 時 04 分 05 秒 999"を表す
// タイムゾーンを指定することはできない
const date1 = new Date(2006, 0, 2, 15, 4, 5, 999);
console.log(date1.toISOString()); // "2006-01-02T06:04:05.999Z" (Asia/Tokyoの場合)

// Date.UTCメソッドを使うとUTCに固定できる
const ms = Date.UTC(2006, 0, 2, 15, 4, 5, 999);
// 時刻値を渡すコンストラクタと併用する
const date2 = new Date(ms);
console.log(date2.toISOString()); // => "2006-01-02T15:04:05.999Z" 
```

请注意，如果传递的参数不符合任何重载，或者传递的字符串无法解析为时间，则将创建`Date`的实例。 但是，由于该实例的时间是无效的，因此`getTime`方法将返回`NaN`，`toString`方法将返回`Invalid Date`字符串。

```
// 不正なDateインスタンスを作成する
const invalid = new Date("");
console.log(invalid.getTime()); // => NaN
console.log(invalid.toString()); // => "Invalid Date"
// 不正なDateインスタンスかを判定
console.log(Number.isNaN(invalid.getTime())); // => true 
```

### [](#instance-method)*Date 实例方法*

*`Date`对象的实例具有许多方法，但大多数是用于获取和更新时间各部分的方法，如`getHours`和`setHours`。

下面的代码将日期转换为特定格式的字符串。 要注意像`getMonth`和`setMonth`这样以数字表示月份的方法，需要注意月份是从 0 到 11 的数字。 要显示`Date`实例的时间是哪个月，需要将`getMonth`方法的返回值加 1。

```
// YYYY/MM/DD 形式の文字列に変換する関数
function formatDate(date) {
    const yyyy = String(date.getFullYear());
    // Stringの`padStart`メソッド（ES2017）で2 桁になるように0 埋めする
    const mm = String(date.getMonth() + 1).padStart(2, "0");
    const dd = String(date.getDate()).padStart(2, "0");
    return `${yyyy}/${mm}/${dd}`;
}

const date = new Date("2006-01-02T15:04:05.999");
console.log(formatDate(date)); // => "2006/01/02" 
```

`getTimezoneOffset`メソッドは、実行環境のタイムゾーンのUTC**からの**オフセット値を**分**単位の数値で返します。 たとえばAsia/TokyoタイムゾーンはUTC+9 時間なのでオフセット値は-9 時間となり、`getTimezoneOffset`メソッドの返り値は`-540`です。

```
// getTimezoneOffsetはインスタンスメソッドなので、インスタンスが必要
const now = new Date();
// 時間単位にしたタイムゾーンオフセット
const timezoneOffsetInHours = now.getTimezoneOffset() / 60;
// UTCの現在の時間を計算できる
console.log(`Hours in UTC: ${now.getHours() + timezoneOffsetInHours}`); 
```

## [](#usecase)*現実のユースケースとDate*

*ここまで`Date`オブジェクトとインスタンスメソッドについて述べましたが、 多くのユースケースにおいては機能が不十分です。 たとえば次のような場合に、`Date`では直感的に記述できません。

+   任意の書式の文字列から時刻に変換するメソッドがない

+   「時刻を1 時間進める」のように時刻を前後にずらす操作を提供するメソッドがない

+   任意のタイムゾーンにおける時刻を計算するメソッドがない

+   `YYYY/MM/DD HH:mm`のよ��なフォーマットに��づいた文字列への変換を提供するメソッドがない

そのため、JavaScriptにおける日付・時刻の処理は、標準のDateではなくライブラリを使うことが一般的になっています。 代表的なライブラリとしては、[Day.js](https://day.js.org/)、[date-fns](https://date-fns.org/)、[js-joda](https://github.com/js-joda/js-joda)、[moment.js](https://momentjs.com/)の後継である[Luxon](https://github.com/moment/luxon/)などがあります。

```
// Day.jsで現在時刻のDay.jsオブジェクトを作る
const now = dayjs();
// addメソッドで10 分進める
const future = now.add(10, "minute");
// formatメソッドで任意の書式の文字列に変換する
console.log(future.format("YYYY/MM/DD HH:mm")); 
```

## [](#conclusion)*まとめ*

*この章では、Dateオブジェクトについて学びました。

+   `Date`オブジェクトのインスタンスはある特定の時刻を表すビルトインオブジェクト

+   `Date`における「時刻」は、UTC（協定世界時）の1970 年 1 月 1 日 0 時 0 分 0 秒を基準とした相対的なミリ秒として保持されている

+   `Date`コンストラクタで任意の時間を表す`Date`インスタンスを作成できる

+   `Date`インスタンスメソッドにはさまざまなものがあるが、現実のユースケースでは機能が不十分になりやすい

+   ビルトインオブジェクトの`Date`のみではなく、ライブラリも合わせて利用するのが一般的*******
