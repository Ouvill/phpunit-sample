# PHP Unit のサンプルアプリケーションを作成する

PHPUnit とは？

PHP の Unit テストフレームワーク。
コード品質を一定に保つことを目的にする

参考ドキュメント

[PHPUnit](https://phpunit.de/)


## PHP Project の作成

`composer` を利用して PHP プロジェクトを作成します。

```
$ composer init
```

上記コマンドを実行すると対話形式で PHP のプロジェクトが作成できます。

以下のようなプロジェクトを作成しました。

```json
{
    "name": "ouvill/phpunit-practice",
    "description": "phpunit practice",
    "license": "MIT",
    "require": {}
}
```

## PHPUnit のインストール

```
$ composer require --dev phpunit/phpunit ^9
$ ./vendor/bin/phpunit --version
```

## autoload の設定

autoload を設定すると自動でクラスをロードしてくれます。

`composer.json`

```json
{
  "require-dev": {
    "phpunit/phpunit": "^9"
  },
  "autoload": {
    "classmap": [
      "src/"
    ]
  }
}
```

autoload を追加したのでマップファイルを更新します。

```
$ composer update
```

## コード作成

`src/Email.php`

```php
<?php
declare(strict_types = 1);

final class Email
{
    private $email;

    private function __construct(string $email)
    {
        $this->ensureIsValidEmail($email);
        $this->email = $email;
    }

    public static function fromString(string $email): self
    {
        return new self($email);
    }

    public function __toString(): string
    {
        return $this->email;
    }

    private function ensureIsValidEmail()
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException(
                sprintf(
                    '"%s" is not a valid email address',
                    $email
                )
            );
        }
    }
}
```

## テストコード作成

`tests/EmailTest.php`

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

final class EmailTest extends TestCase
{
    public function testCanBeCreatedFromValidEmailAddress(): void
    {
        $this->assertInstanceOf(
            Email::class,
            Email::fromString('user@example.com')
        );
    }

    public function testCannotBeCreatedFromInvalidEmailAddress(): void
    {
        $this->expectException(InvalidArgumentException::class);

        Email::fromString('invalid');
    }

    public function testCanBeUsedAsString(): void
    {
        $this->assertEquals(
            'user@example.com',
            Email::fromString('user@example.com')
        );
    }
}

```

## テスト実行

コードとテストが記述できたので、テストを実行します。

```bash
$ ./vendor/bin/phpunit --bootstrap vendor/autoload.php tests/EmailTest.php
```

結果

```
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.
...                                                                 3 / 3 (100%)

Time: 11 ms, Memory: 4.00 MB

OK (3 tests, 3 assertions)
```

無事テストが通ったら完了です。


通ったテスト一覧を表示することもできます。

```
./vendor/bin/phpunit --bootstrap vendor/autoload.php --testdox tests
```

結果

```
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.

Email
 ✔ Can be created from valid email address
 ✔ Cannot be created from invalid email address
 ✔ Can be used as string

Time: 12 ms, Memory: 4.00 MB

OK (3 tests, 3 assertions)
```

## Composer Scripts を設定する

テスト実行コマンドが長いので、composer scripts を記載します。

参考:　[Scripts](https://getcomposer.org/doc/articles/scripts.md)

`composer.json`

```json
{
  "require-dev": {
    "phpunit/phpunit": "^9"
  },
  "autoload": {
    "classmap": [
      "src/"
    ]
  },
  "scripts": {
    "test": [
      "phpunit --bootstrap vendor/autoload.php --testdox tests"
    ]
  }
}
```

以下のコマンドでテストが実行できるようになります。

```
$ composer run-script test
```

## GitHub Actions を追加して Test の実行を自動化する

Git でコードを管理していると思います。

GitHub に Push したとき、自動でテストしてくれるように設定しましょう。

最近追加された GitHub Actions を利用します。

GitHub のリポジトリから Actions を開きます。

PHP の`Set up this workflow` をクリック。

以下のように `- name: Run test suite` のコメントを外します。

`.github/workflows/php.yml`

```php
name: PHP Composer

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Validate composer.json and composer.lock
      run: composer validate

    - name: Install dependencies
      run: composer install --prefer-dist --no-progress --no-suggest

    # Add a test script to composer.json, for instance: "test": "vendor/bin/phpunit"
    # Docs: https://getcomposer.org/doc/articles/scripts.md

    - name: Run test suite
      run: composer run-script test
```

コードが編集できたら、`start commit` -> `Commit new file` でコードを追加します。

以上で GitHub にコードが’ Push されるたびにテストが実行されます。

## まとめ

しっかりとテストを書いて、安心してコード変更できるようになりましょう。
