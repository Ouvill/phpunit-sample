# PHP Unit のサンプルアプリケーションを作成する

PHPUnit とは？

PHP の Unit テストフレームワーク。
コード品質を一定に保つことを目的にする

参考ドキュメント

[PHPUnit](https://phpunit.de/)

## PHPUnit のインストール

```
$ composer require --dev phpunit/phpunit ^9
$ ./vendor/bin/phpunit --version
```

## autoload の設定

autoload を設定すると自動でクラスをロードしてくれる。

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

autoload を追加したのでマップファイルを更新する。

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
