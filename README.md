# PHP で kintone 上の画像を取得、表示

## "Cybozu HTTP client for PHP"インストール

[ドキュメント](https://github.com/ochi51/cybozu-http)
```bash
composer require ochi51/cybozu-http
```


## クラスを定義

```php
require 'vendor/autoload.php';

use CybozuHttp\Client;
use CybozuHttp\Api\KintoneApi;
$api = new KintoneApi(new Client([
  'domain'    => 'cybozu.com',
  'subdomain' => 'your-subdomain',

  // ベーシック認証を使用する場合
  'login'     => 'your-login-name',
  'password'  => 'your-password',

  // APIトークンを使用する場合
  'use_api_token' => true,
  'token'         => 'zhU6...RPYaor',
]));
```

## レコードを取得

```php
$appId  = '32';  // アプリID
$id     = '165'; // レコード番号
$record = $api->record()->get($appId, $id);
```

## "FileKey"でkintone上の画像を取得

```php
$fileField = $record['画像']['value']

// 【取得結果(例)】
// array(1) {
//   [0]=> array(4) {
//     ["fileKey"]=> string(49) "20230312160318A7...BEE5407302061"
//     ["name"]=> string(16) "ファイル名.jpg"
//     ["contentType"]=> string(10) "image/jpeg"
//     ["size"]=> string(5) "30023"
//   }
// }

$base64Urls = array();
foreach($fileField as $file) {
  // バイナリ文字列が返ってくる
  $bin = $api->file()->get($file['fileKey']);

  // 画像じゃない場合はスキップ
  if (!strstr($file['contentType'], 'image/')) continue;

  // base64 URL を格納
  $base64Urls[] = 'data:'.$file['contentType'].';base64,'.base64_encode($bin);
}
```

## 画像を表示

```php
<?php foreach($base64Urls as $url): ?>
  <img src="<?= $url; ?>" />
<?php endforeach; ?>
```