# Othello

TypeScript と HTML5 Canvas を使用してオセロゲームを作ることを目指します。

## 環境構築

開発にはTypeScriptというプログラミング言語を使用します。TypeScriptはJavaScriptに静的型付けを追加したプログラミング言語であり、静的型付けによりVSCodeなどの高機能エディタの助けを受けやすくなり、バグの作り込みを防いだり補完機能を十分に利用することができ、開発効率の向上が期待できます。

JavaScriptはブラウザ上で実行できるスクリプト言語としては事実上のスタンダードとなっています。我々が今回使用するTypeScriptをブラウザ上で実行するためには、これをJavaScriptに翻訳してやる必要があります。

以下でTypeScriptを用いてブラウザ上で動作するアプリケーションを作るための環境構築をしていきましょう。

以降の内容はNode.jsとnpmがインストールされている環境を前提としています。
```
$ node -v
v14.15.1

$ npm -v
6.14.8
```
上の2つのコマンドをターミナルで実行してなんらかのバージョン番号が表示されない場合は、各OSのパッケージ管理システムや[https://nodejs.org/en/](https://nodejs.org/en/)公式ベージからインストールしてください。

Node.jsはJavaScriptのランタイムであり、あなたが書いたJavaScriptのコードを実行することができます。上述のとおりJavaScriptはブラウザ上で実行することもできますが、Node.jsを使用するとあなたのターミナル上でも実行することができるようになります。

npmはNode.jsのためのパッケージ管理システムです。インターネット上に公開されているパッケージをインストールしてあなたの環境で使用できるようにします。

プロジェクトのトップディレクトリで以下のコマンドを実行して`package.json`を作成してください。このファイルにはプロジェクトの様々な設定が記述されています。
```
$ npm init -y
```

npmでインストールしたパッケージはnode_modulesというディレクトリに保存されるので、それらがgitのコミットに含まれないように`.gitignore`ファイルを作成して、gitの管理から無視されるようにしておきましょう。
```
$ echo /node_modules/ >> .gitignore
```

### TypeScript

まずはターミナル上でTypeScriptを実行することができる環境を構築します。
```
$ npm install --save-dev typescript ts-node @types/node
```

npmをつかって`typescript`, `ts-node`, `@types/node`というパッケージをインストールします。

- `typescript`: TypeScriptで書いたコードをJavaScriptに変換(トランスパイル)する。
- `ts-node`: ターミナルで直接TypeScriptのコードを実行できるようにする。
- `@typs/node`: Node.jsのデフォルトモジュールで型情報を利用した補完を使えるようにする。

上のコマンドを実行すると、node_modulesディレクトリにパッケージがインストールされます。
同時にpackage.jsonのdevDependenciesフィールドにインストールされたパッケージが追記されます。

node_modulesディレクトリを見ると、指定した3つのパッケージ以外にも`arg`, `diff`, `make-error`といったパッケージがインストールされていることがわかると思います。
`ts-node`パッケージがこれらのパッケージへの依存をもつため、`ts-node`をインストールしようとするとnpmが自動的に依存関係を解決し、これらのパッケージをインストールしてくれます。
これらのすべての依存関係を含めたバージョン情報はpackage-lock.jsonというファイルに記述されています。

package.jsonとpackage-lock.jsonを参照することで、新しい開発者がこのプロジェクトに加わったときに、同一の環境を簡単に構築することができるのです。

新たに加わった開発者は以下のコマンドを実行することでpackage-lock.jsonを参照して、すべての依存パッケージをインストールすることができます。
```
$ npm ci
```

それではパッケージをインストールしたことで何ができるようになったのかを見ていきましょう。

まずは比較のためにJavaScriptのファイルを作成します。srcというディレクトリを作成して`src/main.js`というファイルを作成してください。その中身はこんな感じです。

```javascript
const main = () => {
  const a = "Helllo World!!";
  console.log(a);
};

main();
```

`main`という関数をつくって実行しているだけです。main関数の中では`a`という変数に`"Hello World!!"`という文字列を代入して、`console.log`メソッドで変数aをプリントしています。
このファイルを実行してみましょう。

```
$ node src/main.js
Hello World!!
```

このファイルをTypeScriptのコードに変更していきます。まずはファイル名を`src/main.ts`に変更しましょう。TypeScriptはJavaScriptのスーパーセットなので、このファイルは完全に合法なTypeScriptのファイルです。
とはいえ、せっかくなので変数aに型をつけてみましょう。以下のようになります。
```typescript
const main = () => {
  const a: string = "Helllo World!!";
  console.log(a);
};

main();
```

これを先ほどと同様に実行しようとしてもエラーになってしまうでしょう。Node.jsではTypeScriptを直接実行することができないためです。
```
$ node src/main.ts
src/main.ts:2
  const a: string = "Helllo World!!";
        ^

SyntaxError: Missing initializer in const declaration
    at wrapSafe (internal/modules/cjs/loader.js:979:16)
    at Module._compile (internal/modules/cjs/loader.js:1027:27)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1092:10)
    at Module.load (internal/modules/cjs/loader.js:928:32)
    at Function.Module._load (internal/modules/cjs/loader.js:769:14)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:72:12)
    at internal/main/run_main_module.js:17:47
```

TypeScriptのファイルをブラウザやNode.jsで実行できるようにするために、先程インストールした`typescript`パッケージを使用します。このパッケージをインストールすることで`tsc`コマンドを使用できるようになります。以下のコマンドで`src/main.ts`から`main.js`を生成します。そして生成された`main.js`がNode.jsによって正しく実行できることを確認してください。
```
$ npx tsc --outFile main.js src/main.ts 
$ node main.js
Helllo World!!
```

確認できたら`main.js`を削除しておきます。
```
$ rm main.js
```

とはいえTypeScriptのコードを実行するたびに、変換→実行のステップを踏むのは面倒なので直接実行したくなります。そのために先程インストールした`ts-node`を使用します。以下のコマンドでTypeScriptのファイルが直接実行できることを確認してください。
```
$ npx ts-node src/main.ts
Helllo World!!
```

最後に今後のためにTypeScriptのための設定ファイルを追加しておきましょう。`tsconfig.json`というファイルを作ってください。
```json
{
  "include": ["src/**/*.ts"],
  "compilerOptions": {
    "target": "ES2019",
    "module": "commonjs",
    "sourceMap": true,
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "moduleResolution": "node",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
  }
}
```

### Formatter

コードのスタイルを統一することは、可読性・保守性を向上させるためにとても重要です。[Googleもそう言っています。](https://google.github.io/styleguide/cppguide.html)
我々の開発でも自動フォーマッタを導入して、コードのスタイルを統一しましょう。

```
$ npm install --save-dev prettier
```

`.prettierrc.json`という名前でフォーマッタの設定ファイルを作成しておきます。
```json
{
  "singleQuote": true
}
```

以下のコマンドを実行すると`src/`以下のファイルがフォーマットされます。現時点では`src/main.ts`のみが対象です。
```
$ npx prettier --write ./src
```

少しわかりにくいですが、`"Hello World!!"`のダブルクオートがシングルクオートに変わっているのを確認できると思います。

毎回このようにフォーマットを実行しても良いですが、新たにプロジェクトに加わったチームメンバーや、少し時間を開けて開発に戻ってきた自分自身が、フォーマットのために実行すべきコマンドがわからなくて困る場面が簡単に想像できます。npmのタスクランナー機能を使用して将来の自分が困らないようにしておきましょう。

`package.json`のscriptフィールドに以下のように追記してください。
```json
  "scripts": {
    "format": "prettier --write ./src",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```

以下のコマンドでフォーマッタを走らせられるようになりました。エディタの機能をつかって保存時に自動的にフォーマットされる設定をするのもよいでしょう。
```
$ npm run format
```

### Test

テスト駆動開発というプログラム開発手法があります。プログラムに必要な各機能について実装する前にテストを書き、テストを通すための最低限の実装を行い、テストが通る状態を維持しながらコードをリファクタリングする。このような開発サイクルを短いスパンで繰り返していくことで、頻繁に開発に対するフィードバックが得られ開発効率が向上することが期待できます。

単体テストのフレームワークである`jest`を導入します。
```
$ npm install --save-dev jest ts-jest @types/jest
```

続けて、jestの設定ファイルを作成します。package.jsonの`scripts`フィールドに`"test": "jest"`という行が追加されていることに注意してください。
```
$ npx jest --init
```

自動生成された`jest.config.js`を以下のように書き換えてください。
```javascript
module.exports = {
  // Indicates which provider should be used to instrument code for coverage
  coverageProvider: "v8",
  // A list of paths to directories that Jest should use to search for files in
  roots: [
    "<rootDir>/src"
  ],
  // The test environment that will be used for testing
  testEnvironment: "jsdom",
  // A map from regular expressions to paths to transformers
  transform: {
    "^.+\\.ts$": "ts-jest"
  },
};
```

これでテストフレームワークの導入ができました。簡単なテスト駆動開発を体験してみましょう。
`src/add.test.ts`というファイルを作成してください。`.test.ts`という拡張子のファイルはjestによってテスト対象のファイルと認識されます。

```typescript
import { add } from './add'

test('add_function_returns_sum_of_two_number', () => {
  const result = add(1, 1);
  expect(result).toBe(2);
})
```
1行目で`src/add.ts`内で定義された`add`関数をインポートしています。これにより、このファイル内で`add`関数が使用できるようになります。
3行目の`test`関数はjestにより提供される関数です。1つ目の引数はテストケースの名前です。そのテストケースが何をテストしているのかが理解できる説明的な名前をつけるようにしましょう。あなたのチームで合意ができているのであれば日本語で名前をつけても構いません。2つ目の引数はテストを走らせたときに実行される関数です。ここではアロー関数式で与えています。
4行目で1と1を引数として`add`関数を実行し、5行目でその結果が2であることを確かめています。

それでは、ここで一度テストを走らせてちゃんと失敗することを確認しましょう。`src/add.ts`ファイルは存在すらしていないので当然失敗するはずです。
実際にエラーメッセージを読んでみると`./add`というモジュールが見つからないと言っているようです。
```
$ npm run test
...
 FAIL  src/add.test.ts
  ● Test suite failed to run

    src/add.test.ts:1:21 - error TS2307: Cannot find module './add' or its corresponding type declarations.

    1 import { add } from './add'
                          ~~~~~~~

Test Suites: 1 failed, 1 total
Tests:       0 total
Snapshots:   0 total
Time:        2.814 s
Ran all test suites.
...
```

エラーを解消するために`src/add.ts`を作成して、`add`関数を用意しましょう。これで`add.test.ts`内で`add`関数が使用できるようになります。
```
export const add = (l: number, r: number): number => {
  return 0;
}
```

テストを走らせてみましょう。`src/add.test.ts`の5行目で`result`には2が期待されているのに0が入っていましたよと報告されています。先程実装した`add`関数はいつでも0を返すので当然の結果です。
```
$ npm run test
...
 FAIL  src/add.test.ts
  ✕ add_function_returns_sum_of_two_number (4 ms)

  ● add_function_returns_sum_of_two_number

    expect(received).toBe(expected) // Object.is equality

    Expected: 2
    Received: 0

      3 | test('add_function_returns_sum_of_two_number', () => {
      4 |   const result = add(1, 1);
    > 5 |   expect(result).toBe(2);
        |                  ^
      6 | })

      at Object.<anonymous> (src/add.test.ts:5:18)
      at TestScheduler.scheduleTests (node_modules/@jest/core/build/TestScheduler.js:333:13)
      at runJest (node_modules/@jest/core/build/runJest.js:387:19)
      at _run10000 (node_modules/@jest/core/build/cli/index.js:408:7)
      at runCLI (node_modules/@jest/core/build/cli/index.js:261:3)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 1 total
Snapshots:   0 total
Time:        3.782 s, estimated 4 s
Ran all test suites.
...
```

テストを通過できるように`add`関数の実装を進めましょう。とりあえず2を返すようにしたらテストをパスすることができそうです。
```typescript
export const add = (l: number, r: number): number => {
  return 2;
}
```
```
$ npm run test
...
 PASS  src/add.test.ts
  ✓ add_function_returns_sum_of_two_number (2 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        3.862 s, estimated 4 s
Ran all test suites.
```

無事テストをパスすることができました！

とはいえ明らかにこの`add`関数の実装には問題があります。我々が求める`add`関数の振る舞いを十分に表現できるようにテストケースを増やしてみましょう。
```typescript
import { add } from './add'

test('add_function_returns_sum_of_two_number', () => {
  let result = add(1, 1);
  expect(result).toBe(2);
  result = add(-1, 1);
  expect(result).toBe(0);
  result = add(-1, -1);
  expect(result).toBe(-2);
  result = add(100, 222);
  expect(result).toBe(322);
})
```

そして、テストを実行します。ここでは結果を表示することはしませんが、このテストは失敗するはずです。テストをパスするように`add`関数を書き換えてみましょう。

このように、テストを書く→テストに失敗する→テストをパスするように実装する→リファクタリングをする、というサイクルを細かい粒度で回していくのがテスト駆動開発です。
今回は単純な例だったのでリファクタリングは行いませんでしたが、テストが通る状態を維持することによってリファクタリングのしやすさは格段に向上します。

### webpack

我々の最終目標はブラウザ上でオセロゲームを実装することでした。[webpack](https://webpack.js.org/)は依存関係をもつモジュール群を静的なアセットとしてバンドルするツールです。適切に設定することで内部的に`typescript`パッケージを呼び出してTypeScriptのトランスパイルを同時にすることができます。

```
$ npm install --save-dev webpack webpack-cli webpack-dev-server ts-loader
```

つづけてwebpack用の設定ファイルを作成します。ファイル名は`webpack.config.js`です。
```javascript
const path = require("path");

module.exports = {
  entry: {
    index: path.join(__dirname, 'src/index.ts'),
  },
  output: {
    path: path.join(__dirname, 'dist'),
  },
  module: {
    rules: [
      { test: /\.ts$/, use: 'ts-loader' },
    ],
  },
  resolve: {
    extensions: ['.ts', '.js', '.json'],
  },
  target: ["web"],
  mode: process.env.NODE_ENV || "development",
  devtool: 'inline-source-map',
  devServer: {
    contentBase: path.join(__dirname, 'dist')
  }
};
```

5行目で`src/index.ts`をエントリーポイントとして指定しています。このファイルから再帰的に依存モジュールをたどり、`index.js`というファイル名ですべてのモジュールが含まれたファイルを出力します。
8行目で出力先ディレクトリを指定しており、結果的に`dist/index.js`が出力されることになります。
12行目では`.ts`という拡張子のファイルをwebpackが処理するときに`ts-loader`というものを使うように設定しています。`ts-loader`は`typescript`パッケージを使用してTypeScriptファイルをトランスパイルする役割があります。

`src/index.ts`を用意します。アラートウィンドウで"Hello World!"と表示をして、以前作った`add`関数を呼び出すプログラムです。
```typescript
import { add } from './add';

alert("Hello World!\n 10+15=" + add(10,15));
```

それではwebpackを走らせてみましょう。
```
$ npx webpack
```

`dist/index.js`というファイルができていることが確認できると思います。

続けて`dist/index.html`を作成しておきます。先程生成された`dist/index.js`を読み込むだけのページです。
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Othello</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script defer src="index.js"></script>
  </head>
  <body>
  </body>
</html>
```

以下のコマンドを実行してください。`./dist/`ディレクトリ以下のファイルをサーブする開発用のWebサーバーを実行することができます。これによって、ブラウザから`dist/index.html`や`dist/index.js`にアクセスできるようになります。このサーバーを実行している間はソースコードの変更が監視され、変更が検知されると自動的にビルドが実行されます。

```
$ npx webpack serve
```

[http://localhost:8080](http://localhost:8080)をブラウザで開いてください。以下のように表示されたら成功です。
```
Hello World!
10+15=25
```

最後に、開発中に何度も使うことになるコマンドをnpmのタスクランナーに登録しておきましょう。
package.jsonに以下のように追記してください。
```json
  "scripts": {
    "build": "webpack",
    "serve": "webpack serve",
    "format": "prettier --write ./src",
    "test": "jest"
  },
```

## CUIで遊べるオセロを作る
我々の最終目標はブラウザ上で遊べるオセロゲームを作ることでしたが、はじめはもう少しシンプルなものから始めましょう。簡単なものから段階を踏んで複雑なものを作っていくのはとても良いプラクティスです。

ブラウザで動作するオセロゲームの前段階として、CUI(キャラクタインターフェース)であるターミナル上で動作するオセロゲームを作っていきましょう。

以下に動作イメージを提示します。この通りに作る必要は全くありません。あくまで理解を助けるためのサンプルです。

```
$ npm run game-start
ゲームを開始します。
  a b c d e f g h
  - - - - - - - -
1| | | | | | | | |
2| | | | | | | | |
3| | | |-| | | | |
4| | |-|o|x| | | |
5| | | |x|o|-| | |
6| | | | |-| | | |
7| | | | | | | | |
8| | | | | | | | |
  - - - - - - - -
  黒(x): 2
  白(o): 2

黒(x)の手番です。座標を入力してください。例: c4
> c4

  a b c d e f g h
  - - - - - - - -
1| | | | | | | | |
2| | | | | | | | |
3| | |-| |-| | | |
4| | |x|x|x| | | |
5| | |-|x|o| | | |
6| | | | | | | | |
7| | | | | | | | |
8| | | | | | | | |
  - - - - - - - -
  黒(x): 4
  白(o): 1

白(o)の手番です。座標を入力してください。例: c3
> c3
.
.
.
```

### 標準入力・標準出力を扱う
`src/main.ts`を編集しましょう。

```typescript
import { createInterface } from 'readline';

const main = () => {
  console.log('好きな数字を入力してください');
  process.stdout.write('> ');

  const reader = createInterface({
    input: process.stdin,
    output: process.stdout,
  });

  reader.on('line', function (line) {
    console.log('あなたが入力したのは"' + line + '"です');
    console.log('');
    console.log('好きな数字を入力してください');
    process.stdout.write('> ');
  });
};

main();
```

以下のように実行してください。終了するときは`Ctrl-C`です。
```
$ npx ts-node src/main.ts
```

`console.log`と`process.stdout.write`はどちらも標準出力に出力する関数ですが、前者は出力後に自動で改行がはいるという違いがあります。

`reader.on('line', function)`で与えた関数はターミナルでエンターキーを押す度に実行されます。そのとき引数には入力された文字列が与えられます。

この単純なゲームを少しずつ改善していきます。
ゲームを開始するコマンドをタスクランナーに登録しておきましょう。`package.json`に追記します。
```
  "scripts": {
    "start-game": "ts-node src/main.ts",
```

### 盤面の表示
`src/othello.ts`というファイルを作成しましょう。これ以降オセロのシステムはこのファイルに記述することにします。こうすることで、コードの再利用性が高まります。これ以降も適宜ファイルを分けながら開発していきましょう。

まずはオセロの盤面を表現する型`Board`を定義します。
どのような構造体でも構いません。以下は一つの例です。
```
export type Row = [
  boolean,
  boolean,
  boolean,
  boolean,
  boolean,
  boolean,
  boolean,
  boolean
];

export type BoardArray = [Row, Row, Row, Row, Row, Row, Row, Row];

export type Board = {
  black: BoardArray;
  white: BoardArray;
  black_turn: boolean;
};
```

初期盤面を生成する関数を作成します。以下のような関数を実装していきます。
```
export function generate_initial_board(): Board {
  ...
}
```

まずはテストを書きましょう。`src/othello.test.ts`というファイルを作成してください。
```typescript
import { Board, generate_initial_board } from './othello';

test('generate_initial_board', () => {
  const board = generate_initial_board();
  expect(board.black.length).toBe(8);
  expect(board.black[0].length).toBe(8);
  expect(board.black[0][0]).toBe(false);
  expect(board.black[4][3]).toBe(true);
  expect(board.black[3][3]).toBe(false);
  expect(board.white.length).toBe(8);
  expect(board.white[0].length).toBe(8);
  expect(board.white[0][0]).toBe(false);
  expect(board.white[4][3]).toBe(false);
  expect(board.white[3][3]).toBe(true);
  expect(board.black_turn).toBe(true);
});
```

テストが失敗することを確認して、テストが通るように実装してください。

main関数の実装を進めていきます。main関数の最初に盤面を初期化する処理をいれて盤面を表示しましょう。
このとき盤面を文字列化する関数があると便利です。今回はテストも自分で書いてみてください。テストを書いてその後実装してください。`src/main.ts`内で`src/othello.ts`で定義した関数や型を使用するためには、`src/othello.test.ts`の先頭に書いたようなimport文を書く必要があります。
```typescript
export function stringify_board(board: Board): string {
  ...
}
```

改行を含む文字列を扱うときにはテンプレートリテラル(バッククォート)が便利です。
```typescript
const raw_string = `apple
banana
suika`;
```

つぎに盤面の下に黒白それぞれのスコアを表示します。
```
  黒(x): 2
  白(o): 2
```

スコアを計算する関数を作りましょう。しつこいようですが**テスト**を書いてください！！
```typescript
// [黒の石数, 白の石数]を返す
export function calc_score(board: Board): [number, number] {
  ...
}
```

最後に手番を表示するようにして、この節はおしまいです。
```
$ npm run game-start
ゲームを開始します。
  a b c d e f g h
  - - - - - - - -
1| | | | | | | | |
2| | | | | | | | |
3| | | | | | | | |
4| | | |o|x| | | |
5| | | |x|o| | | |
6| | | | | | | | |
7| | | | | | | | |
8| | | | | | | | |
  - - - - - - - -
  黒(x): 2
  白(o): 2

黒(x)の手番です。座標を入力してください。例: c4
```

### 石を置く
与えられた座標に石をおいて盤面を変化させてみましょう。いまのところ与えられた座標が合法手かどうかは考えなくて良いことにします。

`put_stone()`という関数をつくることにします。以下のような関数になるはずです。テストを書いてから実装しましょう。
```typescript
export function put_stone(point: [number, number], black_turn: boolean, board: Board) {
  ...
}
```

石をひっくり返す関数も作成します。
```typescript
export function flip_stone(point: [number, number], board: Board) {
  ...
}
```

手番を進める関数も作成します。
```typescript
export function move_turn(board: Board) {
  ...
}
```

これらの関数を`src/main.ts`内で使用するためには"c4"といった座標の文字列を`[2, 3]`といった`[number, number]`の型に変換する関数があると便利そうです。
```typescript
export function parse_coord(coord_str: string): [number, number] {
  ...
}
```

この節で作成した関数を`src/main.ts`に組み込んでいきましょう。
ここまでで我々のゲームは黒と白の石を交互に好きな位置におけるゲームになっているとおもいます。

### 合法手以外を打てないようにする
好き勝手な位置に石を置かれてはゲームが成立しないので、ユーザーからの入力が合法手であるかをする必要があります。
```typescript
export function is_valid_move(p: [number, number], board: Board): boolean {
  ...
}
```

ある手が合法であるかを確認するためには、縦横斜め方向にグリッドを探索する必要があります。
このような探索では以下のような定数を用意しておいてベクトル同士の演算として扱うと簡潔に書けることが多いです。
```typescript
const DIRECTIONS = {
  up:    [-1,  0],
  down:  [ 1,  0],
  left:  [ 0, -1],
  right: [ 0,  1],
  ul:    [-1, -1],
  ur:    [-1,  1],
  dl:    [ 1, -1],
  dr:    [ 1,  1],
} as const;

function add_vec(p: readonly [number, number], q: readonly [number, number]): [number, number] {
  ...
}

const new_p = add_vec(p, DIRECTIONS.up);
```

探索中に盤面の配列をオーバーした添字でアクセスするとエラーが起きてしまうことに注意してください。
そのようなケースを扱うテストを用意しておくと良いですね。

`is_valid_move`関数ができたら、ユーザーが非合法手を入力したときに怒って、もう一度入力を待ってくれるように`src/main.ts`を変更しておいてください。
いまのところ、挟まれた石がひっくり返るというルールに関して考えなくても良いことにします。

### 合法手を表示する
もう少し我々のゲームを洗練させるために、せっかくなので着手可能位置のヒントを表示してあげることにしましょう。
この機能を実現するためにはすべての合法手を列挙する必要があります。この機能をもつ関数`all_valid_state`を作成しましょう。

前節で実装した`is_valid_move`をうまく使うと簡単に実装できると思います。

以前実装した`stringify_board`も変更して着手可能位置を表示するように変更してください。着手可能位置を表示するオプションを引数として与えたり、もしくは新たに別の関数を作ってもよいでしょう。このときもはじめにテストを書くことを推奨します。

### 挟まれた石をひっくり返す
いよいよオセロの核心である、挟まれた石をひっくり返す処理を実装していきましょう。
盤面と新たに置かれた石が入力として与えられたときにある一方向についてひっくり返る石の配列を返してくれる関数があればよさそうです。
この関数をすべての方向について呼び出すことでひっくり返すべきすべての石が得られます。
関数内部の処理は前前節で実装したものと共通点が多いのでうまく共通化できると実装量が節約できそうです。

最後に、現在の盤面と次の着手が与えられて次の盤面を返すような関数`next_state`を実装してこれを`src/main.ts`から呼ぶことにしましょう。
`src/main.ts`の中身は少ないほどブラウザへの移植が楽になります。

これでオセロゲームはほとんど遊べるようになっていると思います。
最後の節では細々した機能を追加してゲームの体裁を整えます。

### パス、終了判定、勝敗判定
CUIオセロゲームの最後の節です。
以下の機能を実装してください。

- 着手可能位置がない場合には手番を飛ばす
- ゲームの終了を判定する
- ゲーム終了時に勝敗に関するメッセージを出力する

## ブラウザで遊べるオセロを作る
webpackの節では`dist/index.html`と`src/index.ts`を編集しました。
`dist`ディレクトリは自動生成したファイルだけが入っていてほしいような気がするので、少し設定を変更しましょう。

Webpackのプラグインを追加します。
```
npm install --save-dev html-webpack-plugin
```

`src/index.html`というファイルを作成します。
```html
<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8">
  <title><%= htmlWebpackPlugin.options.title %></title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <canvas id="canvas"></canvas>
</body>
</html>
```

`webpack.config.js`を編集します。
```javascript
const path = require("path");
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  ...
  plugins: [
    new HtmlWebpackPlugin({
      chunks: ['index'],
      filename: 'index.html',
      title: 'Othello',
      template: 'src/index.html'
    }),
  ],
  ...
};
```

### Canvas
オセロの盤面をページ上に表示するのにCanvasというHTMLの仕様を使います。Canvasを使用することで線分・長方形・円といった図形や画像を表示することができます。

`src/index.ts`を編集してCanvasを試してみましょう。
```typescript
const main = () => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  canvas.height = 800;
  canvas.width = 800;
  const ctx = canvas.getContext('2d');

  if (ctx != undefined) {
    // 長方形に塗りつぶす 左上(100, 100) 幅: 400, 高さ: 400
    ctx.fillStyle = 'green';
    ctx.fillRect(100, 100, 400, 400);

    // 線をひく (100, 0) から (500, 600)
    ctx.strokeStyle = 'gray';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(100, 0);
    ctx.lineTo(500, 600);
    ctx.stroke();

    // 円 中心(200, 300) 半径10
    ctx.strokeStyle = 'gray';
    ctx.lineWidth = 2;
    ctx.fillStyle = 'white';
    ctx.beginPath();
    ctx.arc(200, 300, 10, 0, Math.PI * 2, false);
    ctx.fill();
    ctx.stroke();
  }
};

main();
```

実行してブラウザで結果を見てみましょう。
```
$ npm run serve
```

これ以降ブラウザ上で実行するJavaScriptに変換するので`tsconfig.json`を編集します。
`module`フィールドを以下のように変換してください。
```json
    "module": "esnext",
```

`commonjs`というのはTypeScriptをNode.jsで実行できるように変換する設定で、`esnext`はブラウザ用の設定です。この変更に伴って`src/main.ts`が実行できなくなっているので、引き続き`game-start`タスクが正常に実行できるように`package.json`を編集しておきましょう。
```json
  "scripts": {
    "start-game": "ts-node -O '{\"module\": \"commonjs\"}' src/main.ts",
```

### 盤面を表示する
ゲームの盤面を表現するには盤と駒をそれぞれ表示する必要がありそうです。まずは盤を表示する関数から作成していきましょう。

`src/drawer.ts`というファイルを作成して描画関係の処理はここに記述することにします。
盤を表示する`draw_grid`関数を作成することにします。この関数のように演算の結果が戻り値以外にも影響を与える関数のことを"副作用を持つ"といいます。このような副作用を持つ関数はテストをするのが難しくバグに繋がる可能性が高いので、一般的にはできる限り避けるべきですが避けるのが難しいこともあります。今回のケースでは`CanvasRenderingContext2D`オブジェクトをモックすることでテストを書くこともできますが省略しておきます。興味があれば調べて実装してみてください。
```typescript
export function draw_grid(ctx: CanvasRenderingContext2D): void {
  ...
}
```

実装できたら`src/index.ts`からこの関数を呼び出してみます。前の節で書いた`main`関数内の描画処理は削除して構わないです。

今のところ盤面のサイズは800×800で固定していますが、これはユーザーの画面サイズに合わせて変化させた方が良さそうです。
`draw_grid`関数を`height`, `width`を追加で受け取るように書き換えてみましょう。
```typescript
export function draw_grid(ctx: CanvasRenderingContext2D, height: number, width: number): void {
  ...
}
```

今後描画に関するあらゆる関数は`ctx, height, width`のセットを引数にとることが予想できるので、代わりに`canvas: HTMLCanvasElement`を与えるようにした方がよさそうです。
```typescript
export function draw_grid(canvas: HTMLCanvasElement): void {
  ...
}
```

とはいえ、いつも`height`, `width`を気にしながらコードを書くのは面倒に感じます。ちょっと便利な関数を導入することにします。
私達の座標系では常に盤面のサイズを100×100に固定しておいて、実際に描画メソッドを呼び出す前に座標系を変換してあげる関数を作成します。
これらの関数は副作用がないのでテストを書いてみましょう。
```typescript
export function convert_vec(x: number, y: number, canvas: HTMLCanvasElement): [number, number] {
  ...
}

export function convert_scal(a: number, canvas: HTMLCanvasElement): number {
  ...
}
```

`src/drawer.test.ts`
```typescript
test('convert_vec', () => {
  document.body.innerHTML = '<canvas id="canvas"></canvas>';
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  canvas.height = 800;
  canvas.width = 800;

  let [orig_x, orig_y] = [0, 0];
  let [new_x, new_y] = convert_vec(orig_x, orig_y, canvas);
  expect(new_x).toBe(0);
  expect(new_y).toBe(0);
  ...
});
```

これらの関数を用いて`draw_grid`関数を書き直しておきましょう。

続けて駒の描画を実装していきます。
まずは一つのコマを置く関数から始めましょう。引数の`i, j`は座標ではなくi行j列を表現していることに注意してください。
```typescript
export function draw_piece(i: number, j: number, canvas: HTMLCanvasElement): void {
  ...
};
```

この関数を用いて全ての駒を描画する関数も作ります。`draw_pieces`は`Board`を引数にとって全ての駒を表示する関数です。

最後に、ここまで定義してきた関数を組み合わせて盤面を表示する関数`draw_board`を作成します。
処理の初めに前回の描画を消去する`context.clearRect(x, y, w, h)`メソッドを呼ぶようにしておいてください。

`src/index.ts`から`draw_board`を呼んで盤面を表示しましょう。

### ゲームループ
一般的なコンピュータゲームでは一定の時間間隔ごとにゲームの内部状態や画面描画を更新するようなゲームループの仕組みがあります。CUIのオセロゲームで行ったようにユーザーからの入力があるまで状態の更新を行わないようなゲームループも考えられます。
今回は一定の時間間隔で更新するゲームループを採用しましょう。

`src/game.ts`ファイルを作成します。
```typescript
import { Board, generate_initial_board } from "./othello";

export type Game = {
  last: number;      // 最後に盤面の更新をした時刻 (ms)
  interval: number;  // (interval)ms 毎に盤面の更新を行う
  board: Board;
  canvas: HTMLCanvasElement;
};

export function create_game(canvas: HTMLCanvasElement): Game {
  return {
    last: performance.now(),
    interval: 1000, // ms
    board: generate_initial_board(),
    canvas: canvas
  }
}

function update_game(game: Game): void {
  console.log(game.last);
}

export function start_loop(game: Game): void {
  const run = (now: number) => {
    let delta = now - game.last;
    while (delta >= game.interval) {
      delta -= game.interval;
      game.last = now - delta;
      // ここで盤面の更新・描画処理を行う
      update_game(game);
    }
    requestAnimationFrame(run);
  };
  requestAnimationFrame(run);
}
```

`src/index.ts`
```typescript
const main = () => {
  const game = create_game(canvas);
  start_loop(game);
};
```

どのような動作をするかブラウザで確認してみましょう。`console.log`の出力をブラウザで確認するには、chromeであればページ上で右クリックをして"Inspect"を選択して開発者ツールを表示します。開発者ツールの"Console"タブを開いてください。1秒に一回数値が表示されているのが確認できると思います。

厳密には環境によって異なりますが、基本的にはブラウザの画面更新は1/60秒に一度行われます。`requestAnimationFrame`関数に渡された関数(`run`)は次回の画面更新時に呼び出されることになります。`run`関数の中では再度`requestAnimationFrame(run)`が呼ばれている再帰構造になっているので、基本的には1/60秒毎に`run`関数が呼び出され続けるようになっているわけです。

`run`関数内部では、最後の画面更新時刻`last`と現在時刻`now`を比較して、その差が`interval`よりも大きいときに画面更新を行うようにしています。これによって1秒毎に出力する仕組みです。

この節では、1秒毎にランダムな手でオセロゲームを更新する処理を実装してみましょう。
前節で実装した`draw_board`関数や前章で実装した`next_state`関数、`all_valid_move`関数を使用するとよいでしょう。

### マウス入力を扱う
この節ではユーザーからのマウス入力を受けて、駒を置く処理を作っていきましょう。
ユーザーは黒番固定ですすめていくことにします。

ブラウザでユーザーの入力を扱うときにはイベント駆動と呼ばれるプログラミングモデルが使用されます。
ページ上のcanvas要素に対するマウスクリックのようなイベントに対してイベントリスナーという関数を登録します。
プログラマーはこの登録をするだけでブラウザ側がユーザーのクリックを検知して登録したイベントリスナーを呼び出してくれます。

```typescript
...
const main = () => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  canvas.height = 800;
  canvas.width = 800;
  ...
  canvas.addEventListener('click', (e: MouseEvent) => {
    const rect = canvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    alert(x + ", " + y);
  })
};
...
```

canvas上をクリックするたびにアラートウィンドウが表示されることが確認できると思います。

さて、それではこのマウス入力機能を私たちのゲームに導入していきましょう。
全体の方針は以下のような形になります。
`Game`構造体に`user_input`のような新たなフィールドを追加します。イベントリスナーではマウスクリックを検知したときに`user_input`にユーザーの入力を保存しておきます。
次の盤面更新時にユーザーの手番であれば`user_input`の値に応じて盤面を進行させます。

まずは、`Game`に`user_input`フィールドを追加します。このフィールドの型はユーザー入力がある場合は座標でない場合は`null`になります。このような型は`[number, number] | null`のように表現できます。`create_game`関数を同時に更新することを忘れないでください。

イベントリスナーを登録する関数`register_mouse_input_listner`を作成します。
この中ではイベントリスナーを作成し、それをcanvas要素のイベントリスナーとして追加します。
イベントリスナーではマウスの座標を読み取り、それを`game`変数の`user_input`フィールドに保存するようにしましょう。
```typescript
function register_mouse_input_listner(game: Game): void {
  ...
}
```
この関数はゲーム初期化時に一度だけ呼ばれれば良いので`create_game`内で呼び出すことにしておきます。

`update_game`関数を編集して、`user_input`に応じて盤面を更新する処理を実装しましょう。ユーザーの入力を処理した後には`user_input`の値を消去しておいて、入力を重複して処理することを防いでください。
canvas上の座標をオセロ盤面の行数・列数に変換する関数があると便利そうです。この関数は描画処理を担当する`src/drawer.ts`ファイルに記述するのがよいと思います。
`update_game`関数の中身は"盤面の状態を更新"して、更新があれば"盤面を描画する"処理ですから、以下のように整理できそうです。`update_state`関数は盤面を更新して更新があれば`true`を返すようにします。`update_state`関数は描画に関係しないのでテストを書くことも簡単そうです。
```typescript
function update_game(game: Game): void {
  if (update_state(game)) {
    draw_board(game.board)
  }
}
```

最後に1秒に一回の更新ではプレイ感が非常に悪いので、1/60秒に一度の更新にしておきましょう。

ここまででランダムプレイヤーと対戦するオセロゲームができているはずです。かなり完成が近づいてきました。

### 盤面以外の表示など
最後にゲームとしての体裁を整えていきましょう。

#### 手番・スコアの表示
手番やスコアといった盤面以外の情報を表示します。
`src/index.html'に情報を表示するためのタグを追加します。
```html
...
<body>
  <canvas id="canvas"></canvas>
  <div>
    <span id="message"></span>
  </div>
</body>
...
```

`Game`構造体に新たなフィールドを追加しましょう。
```
export type Game = {
  last: number;      // 最後に盤面の更新をした時刻 (ms)
  interval: number;  // (interval)ms 毎に盤面の更新を行う
  board: Board;
  canvas: HTMLCanvasElement;
  user_input: [number, number] | null;
  message_holder: HTMLSpanElement;
};
```

しかるべきタイミングで以下のように`innerText`に値をセットすることでメッセージを出力できます。
```typescript
game.message_holder.innerText = "メッセージ";
```

`create_message(board: Board): string`のような関数を作成すればテストしやすくなるでしょう。

#### 開始前ページ
現状ではページを開くと勝手に対局が始まってしまいます。これでは少しぶっきらぼうな感じがしますね。
`Game`構造体に新たなフラグを導入して、ゲームが進行中かどうかを管理できるようにしてみましょう。

`create_game`内で以下の処理を行います。
1. 開始前のメッセージを表示する。例: 'ゲームを開始するのに"開始"ボタンを押してください'
2. "開始"ボタンを表示する
3. "開始"ボタンのイベントリスナーを登録する。イベントリスナーの処理は以下の通り
    1. 進行中フラグをオンにする
    2. ボタンを表示にする

`update_game`ではゲーム進行フラグがオンでないときには状態の更新を行わないようにしておきます。

ボタンの表示・非表示を切り替えるには以下のようにするとよいです。
```html
<button id="start_button">開始</button>
```

```typescript
const start_button = document.getElementById('start_button') as HTMLButtonElement;
start_button.style.display = 'none';   // 非表示
start_button.style.display = 'inline';  // 表示
```

余裕があれば、"開始"ボタンの代わりに先手・後手を選べるようにしてみてください。
他にも人間同士の対局ができるようにしてもよいですね。

#### 終了時の処理
盤面更新時に終了の判定をして終了した旨のメッセージを表示するようにしてください。同時に進行中フラグをオフにして開始ボタンを再表示しましょう。
再度ゲームを開始できるように、開始時のイベントリスナーで盤面の初期化処理が走るようにします。

#### 盤面サイズの自動調整
canvasのサイズは固定になっていますが、さまざまなデバイスでプレイされることを考えるとウィンドウの幅に合わせて自動調節したほうが良さそうに思われます。
ブラウザの横幅は`document.body.clientWidth`、高さは`document.body.clientHight`で取得できます。800pxと横幅・高さの中で、最も小さいものを盤面のサイズに採用すれば良さそうです。

`draw_board`関数内でこれらの値をチェックして、`canvas.hight`、`canvas.width`を設定すると、ウィンドウサイズが変わったときにも対応できます。

#### 待った
これは少し発展的な課題になりますが、もし時間があれば"待った"機能を実装してもよいと思います。
`Game`構造体で`board`フィールドの代わりに`board_history`のように盤面の履歴を持っておくことで実現できます。

## AIをつくる
今のところランダムな合法手を打つAIと対局できる退屈なオセロゲームですが、この章ではもう少しやりごたえのあるゲームにするため、より賢いAIを作成していきます。
とはいえもうしばらくランダムなAIにお付き合いください。まずはリファクタリングから始めましょう。

### AIのインターフェース
ボードゲームのAIというのは単純化すると盤面を入力として受け取って次の手を出力する関数のようなものです。
この節では私達の作ったゲームにおいてAIが持つべきインターフェースを定義してしまいます。これによって、ランダムな手を打つAIでもより賢いAIでも、このインターフェースを持つ限りは簡単に入れ替えることができるようになります。

`src/ai.ts`ファイルを作成します。AIが与えられた`Board`に対して変更を加えないように`Readonly`とされていることに注意してください。
```typescript
export type AIAgent = {
  next_move(board: Readonly<Board>): [number, number]
};
```

このインターフェースにマッチするようにランダムプレイヤーを作成してみましょう。
```typescript
export function new_random_player(): AIAgent {
  return {
    next_move: (board: Readonly<Board>) => {
      // ここをランダムプレーヤーにする
      return [0, 0];
    }
  };
};
```

この`new_random_player`を使うようにオセロゲームを書き換えてください。
`Game`構造体を以下のように変更して、開始処理と共にこれを初期化し、`update_state`関数内で'user'かどうかによって条件分岐をするのがよいかと思います。
```typescript
export type Game = {
  last: number;
  interval: number;  // (interval)ms 毎に盤面の更新を行う
  board: Board;
  canvas: HTMLCanvasElement;
  user_input: [number, number] | null;
  message_holder: HTMLSpanElement;
  black_player: AIAgent | 'user';
  white_player: AIAgent | 'user';
};
```

リファクタリングを完了して、今までと同様に動作することを確認してください。

### ディープコピー
まずはこの先開発で頻繁に使うことになる`deep_copy`を実装します。どのような振る舞いをするのか説明するために、テストを書いてみます。
```typescript
test("deep_copy_board_array", () => {
  let board = generate_initial_board();
  const copied_board_black = deep_copy_board_array(board.black);
  board = put_stone([0,0], true, board)
  expect(!board.black[0][0]).toBe(copied_board_black[0][0])
});

test("deep_copy_board", () => {
  let board = generate_initial_board();
  const copied_board = deep_copy_board(board);
  board = put_stone([0,0], true, board)
  board = move_turn(board);
  expect(!board.black_turn).toBe(copied_board.black_turn)
  expect(!board.black[0][0]).toBe(copied_board.black[0][0])
});
```

`deep_copy`ではコピー元の値を変更したときにコピー先は変更されません。盤面を探索するAIを作るときには重要な機能です。
以下のような実装ではテストをパスすることができません。そもそもタイプエラーで実行すらできないので、一時的に`Readonly`を外して実行します。変数名は`copied`となっていますが、実際には全く同一のオブジェクトを参照しています。そのためコピー元に対するあらゆる変更はコピー先に影響します。逆もまた然りです。
```typescript
export function deep_copy_board_array(board_array: Readonly<BoardArray>): BoardArray {
  const copied = board_array;
  return copied;
}

export function deep_copy_board(board: Readonly<Board>): Board {
  const copied = board;
  return copied;
}
```

```
 FAIL  src/othello.test.ts (5.115 s)
  ● deep_copy_board_array

    expect(received).toBe(expected) // Object.is equality

    Expected: true
    Received: false

      105 |   const copied_board_black = deep_copy_board_array(board.black);
      106 |   board = put_stone([0,0], true, board)
    > 107 |   expect(!board.black[0][0]).toBe(copied_board_black[0][0])
          |                              ^
      108 | });
      109 |
      110 | test("deep_copy_board", () => {

      at Object.<anonymous> (src/othello.test.ts:107:30)

  ● deep_copy_board

    expect(received).toBe(expected) // Object.is equality

    Expected: false
    Received: true

      113 |   board = put_stone([0,0], true, board)
      114 |   board = move_turn(board);
    > 115 |   expect(!board.black_turn).toBe(copied_board.black_turn)
          |                             ^
      116 |   expect(!board.black[0][0]).toBe(copied_board.black[0][0])
      117 | });

      at Object.<anonymous> (src/othello.test.ts:115:29)
```

次にオブジェクトや配列のコピーとしてよく紹介されている`Object.asign()`やスプレッド構文はどうでしょう？
先ほどと違って`deep_copy_board`の一つ目のケースは通っていることに注目してください。これらのコピーは"浅いコピー"と呼ばれていて、オブジェクトや配列の1階層目については想定通りのコピーをしてくれます。一方入れ子になっているオブジェクトや配列に関しては、同じ参照を持つことになるので注意が必要です。
```typescript
export function deep_copy_board_array(board_array: Readonly<BoardArray>): BoardArray {
  const copied = [...board_array] as BoardArray; // Object.asign([], board_array) as BoardArray;
  return copied;
}

export function deep_copy_board(board: Readonly<Board>): Board {
  const copied = {...board};  // Object.asign({}, board);
  return copied;
}
```
```
 FAIL  src/othello.test.ts
  ● deep_copy_board_array

    expect(received).toBe(expected) // Object.is equality

    Expected: true
    Received: false

      105 |   const copied_board_black = deep_copy_board_array(board.black);
      106 |   board = put_stone([0,0], true, board)
    > 107 |   expect(!board.black[0][0]).toBe(copied_board_black[0][0])
          |                              ^
      108 | });
      109 |
      110 | test("deep_copy_board", () => {

      at Object.<anonymous> (src/othello.test.ts:107:30)

  ● deep_copy_board

    expect(received).toBe(expected) // Object.is equality

    Expected: true
    Received: false

      114 |   board = move_turn(board);
      115 |   expect(!board.black_turn).toBe(copied_board.black_turn)
    > 116 |   expect(!board.black[0][0]).toBe(copied_board.black[0][0])
          |                              ^
      117 | });

      at Object.<anonymous> (src/othello.test.ts:116:30)
```

"深いコピー"を実現する方法はいくつかあって、`lodash`というパッケージを使ったり、一度JSON文字列に変換するといった方法が一般的のようですが、今回は深さが決まっているので自分で簡単に実装してみましょう。以下のようになります。テストが通ることを確認してください。
```typescript
export function deep_copy_board_array(board_array: Readonly<BoardArray>): BoardArray {
  return board_array.map(r=>[...r]) as BoardArray;
}

export function deep_copy_board(board: Readonly<Board>): Board {
  return {
    ...board,
    black: deep_copy_board_array(board.black),
    white: deep_copy_board_array(board.white),
  }
}
```

### 弱いAI
この節では本格的にAIを作る前にほんの少しだけ賢いAIを作っていきます。このAIはとても弱いと思いますが、ボードゲームにおけるAIの仕組みを理解する上で役に立つはずです。

このAIの戦略はこうです。自分の手番が来たら現在の盤面の全ての合法手を打ってみます。その結果自分の駒が一番多くなるような手を選んで打ちます。これだけです。

それでは実装していきましょう。このAIを`weak_agent`と名付けました。`src/weak_agent.ts`ファイルに実装していきます。
```typescript
import { AIAgent } from "./ai";
import { Board } from "./othello";

export function new_weak_agent(): AIAgent {
  return {
    next_move: weak_agent_move
  };
};

function weak_agent_move(board: Readonly<Board>): [number, number] {
  // 全ての合法種を列挙する

  // for 一つの合法手 of 全ての合法手
    // 盤面をコピーする
    // 盤面を進める
    // 盤面を評価する

  // 最も自分の駒が多かった合法手を返す
}
```

弱いAIが実装できたら、ランダムAIと差し替えて遊んでみましょう。先ほどのリファクタリングのおかげで簡単に差し替えることができたはずです。

さて、このAIのアルゴリズムは2つの部分からなっています。
1つ目は現在の盤面から手を読んで将来の盤面を生成する"探索"部分。2つ目は生成された盤面の良さを測る"盤面評価"部分です。

ほとんどのボードゲームAIはこの二つの部分から成り立っています。

今回の"探索"は盤面を一手しか進めませんでしたが、より深く何手も探索することもできますし、選択的にある局面を深く探索するといった工夫も考えられます。
"盤面評価"部分は今回のようにオセロの知識を使った人手による設計以外にも、機械学習を用いたものも一般的です。

以降ではこの二つの機能に関して解説します。

### 盤面評価

### 探索

#### min-max法

#### α-β法