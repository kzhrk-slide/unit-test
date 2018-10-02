Unit Testを書くべき理由
===

2018/xx/xx

Kobayashi Kazuhiro (kzhrk)

---

## Unit Testとは

単体テストとも呼ばれる。プログラムを構成するコードを分解し、小さい単位（Unit）の個々の機能が正しく動作しているかを検証するテストのこと。  
基本的には関数やメソッドが単体テストの対象になる。プログラム全体が正しく動作しているかを検証するテストは結合テストと呼ばれる。

### Pros

- 検証テストを実施する段階でバグや仕様漏れが潰せる
- テストを残すことで後々の、ユニット単位の機能拡張が用意になる

### Cons

- テストを書かなくてはならない
- 開発者のスキルセット（テストへの理解、テストが書けるかどうか）
- テストのコストが軽視され、運用時に負債になる可能性がある

### 関数、メソッドのテストとは

たとえば、下記のような関数があったとする。

```js
function add(a) {
  return a + 1;
}
```

`add`関数に`a`を渡したとき、`a+1`が返されるというのが正しい動作になる。  
「`add`関数に`1`を渡したときに`2`が返却されること」が確認できればテストが成功となる。

### テストから浮き出る仕様漏れ

先程の`add`関数に文字列`a`が渡された場合、このままのコードであれば、文字列`a1`が返却される。どのような挙動をするのが正しいだろうか。  
`null`を返却することだろうか。エラーをthrowすることだろうか。はたまた渡された文字列`a`をそのまま返却することだろうか。  
エラーがthrowされることが正しい仕様ということであれば、「`add`関数に文字列`a`を渡したとき、エラーがthrowされること」とテストを追加し、そのテストが通るようにコードを下記のように修正すればいい。

```js
function add(a) {
  if (typeof a !== 'number') {
    throw new Error('error: invalid argument');
  } else {
    return a + 1;
  }
}
```

このとき`Infinity`が`add`に渡された場合どうなるか。同じように仕様を固め、テストを追加し、コードの品質を上げていけばいい。

---

## JavaScritのテスト関連ツール

JavaScriptのUnit Testを行う場合、いくつかのテストツールを組み合わせる必要がある。  
JasmineとJestは依存関係無しで使えるテストフレームワーク（JestはJasmineをベースにしている）。

- [Mocha](https://mochajs.org/)
- [Chai](https://www.chaijs.com/)
- [QUnit](https://qunitjs.com/)
- [Jasmine](https://jasmine.github.io/)
- [Jest](https://jestjs.io/ja/)
- [Karma](https://karma-runner.github.io/)
- [AVA](https://github.com/avajs/ava)
- [Sinon](https://sinonjs.org/)
- [Casper](http://casperjs.org/)
- [Nightwatch](http://nightwatchjs.org/)
- etc...

### テスト環境を提供するツール

CLIでテストを実行するコマンドを提供する。  
ファイルのwatchをしたり、ブラウザを起動したり、コマンドラインで検証結果レポートを表示したり。

- Mocha
- Jasmine
- Jest
- Karma

### テスト構造を提供

BDD構造（○○のとき☓☓する）を提供するツール。

- Mocha
- Jasmine
- Jest

```js
describe('calculator', function() {
  // describes a module with nested "describe" functions
  describe('add', function() {
    // specify the expected behavior
    it('should add 2 numbers', function(done) {
      // Use assertion functions to test the expected behavior
      done();
    });
  });
});
```

### アサーション機能

テストが結果通りになっているか確認するためのツール。  
`require` or `import`するとグローバルに`except`や`assert`が定義される。

- Chai
- Jasmine
- Jest
- power-assert

```js
// Chai expect
expect(foo).to.be.a('string');
expect(foo).to.equal('bar');
// Jasmine expect
expect(foo).toBeString();
expect(foo).toEqual('bar');
// Chai assert
assert.typeOf(foo, 'string');
assert.equal(foo, 'bar');
```

### スナップショットの比較

以前の実行時からのコンポーネントやデータ構造の変更を確認するツール。  

- Jest
- AVA

### モック、スパイ、スタブ

#### モック

偽の関数やメソッドを作成する。  

```js
// jest
const myMock = jest.fn();
console.log(myMock()); // undefined

myMock
  .mockReturnValueOnce(10)
  .mockReturnValueOnce('x')
  .mockReturnValue(true);

console.log(myMock(), myMock(), myMock(), myMock()); // 10, 'x', true, true
```

APIへのアクセスを避けてテストする際に、APIからの返却を偽装するときなどに使う。

```js
// users.js
import axios from 'axios';

export default class Users {
  static getAll() {
    return axios.get('/users').then(res => {
      return res.data;
    });
  }
}
```

```js
// jest
import Users from './users';

jest.mock(Users.getAll);

test('get users', () => {
  const dummy = [{name: 'hoge'}];

  Users.getAll.mockResolvedValue(dummy);

  return Users.getAll().then(users => {
    expect(users).toEqual(dummy);
  })
});
```

#### スパイ

既存のメソッドの呼び出しのチェックを行う。  

```js
// jasmine
describe('spy', () => {
  let hoge = null;
  let fuga = null;

  beforeEach(()=>{
    hoge = {
      setFuga(val) {
        fuga = val;
      }
    };

    spyOn(hoge, 'setFuga');

    hoge.setFuga(123);
  });

  it('setFugaの呼び出しチェック', () => {
    expect(hgoe.setFuga).toHaveBeenCalled();
  });

  it('実態のhogeが呼び出されていない', () => {
    expect(fuga).toBeNull();
  });
});
```

#### スタブ

既存のメソッドを上書きする。

```js
// jasmine
describe('stub', () => {
  let hoge = null;
  let fuga = null;

  beforeEach(() => {
    hoge = {
      setFuga(val) {
        fuga = val;
      }
    };

    spyOn(hoge, 'setFuga').and.callThrough();
  });

  it('setFugaで値を入れてからsetFugaをstubにして動作を確認する', () => {
    hoge.setFuga(123);
    expect(fuga).toEqual(123);

    hoge.setFuga.and.stub();
    fuga = null;

    hoge.setFuga(123);
    expect(fuga).toBe(null);
  });
});
```

### コードカバレッジレポート

テスト結果のカバレッジレポートを出力する。

- Jest

### シナリオ実行

疑似ブラウザを使ってE2Eテストを行う。

- Nightwatch (+ Selenium)

### 簡単なテスト環境

- Mocha + Chai
- Jasmine
- Jest

---

## Jest - Zero configuration testing platform

Jestは設定ファイルほぼゼロで実行できるオールインワンなテストツール。  
プロジェクトルートに`./__test__/*.spec.js`か`./__test__/*.test.js`を置いて、コマンドラインで`jest`コマンドを実行するだけでテストが可能。

---

## Jestのサンプル

[https://github.com/kzhrk/ariaset](https://github.com/kzhrk/ariaset)

---

## なぜUnit Testを書くべきなのか

テストの実装工数がなくても、ユニットテストを書くつもりで関数、メソッドの粒度を決めていけば自ずと最適な粒度でJavaScriptの処理を分離することができる。  
たとえば、非同期でユーザデータを取得して、ユーザ表示を行うクラスがあったとする。  
下記はどちらも同じ処理だが、テストが書けるコードはどちらだろうか。

```js
[...]
  init() {
    this.createUsers();
  }
  createUsers(users) {
    const renderUsers = () => {
      [...]
    };

    axios.get('/user.json')
      .then(response => {
        renderUsers(response.data);
      });
  }
[...]
```

```js
[...]
  init() {
    this.getData().then(users => {
      this.createUsers(users);
    });
  }
  getData() {
    return axios.get('/user.json')
      .then(response => {
        return response.data;
      });
  }
  createUsers(users) {
  }
[...]
```

これは極端な例ではあるが、テストを意識することで関数や粒度のまずさに直感的に気づくことができる。  
まずはzero configurationのjestでテストをやっていきましょう。
