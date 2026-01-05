# JWT (JSON Web Token) Kurulumu
WAMP sunucusunda **JWT (JSON Web Token)** kullanarak kimlik doğrulama sistemini nasıl kurduğumu anlatıyorum.---

## 1. **JWT Kütüphanesini yükleme**

İlk olarak kütüphanesini yükledim. Node.js ile kurdum çünkü bende kuruluydu. Bunun için terminalden şunu yazdım:

```bash
npm install jsonwebtoken
```

JWT işlemleri için gerekli olan kütüphaneyi projeye dahil ettim.

---

## 2. **Login İşlemi ve JWT Token Oluşturma**

Bir **login.php** dosyası oluşturdum ve kullanıcı girişi yaptıktan sonra bir JWT token'ı oluşturacak şekilde yazdım. Bu token, kullanıcının kimliğini doğrulamak için kullanılacak.

```php
<?php
require_once 'vendor/autoload.php';
use \Firebase\JWT\JWT;

$key = "my_secret_key";  // Gizli anahtar, güvenli bir şekilde saklanmalı!

$username = "test_user";
$password = "test_password";

// Şifre kontrolü
if ($username == "test_user" && $password == "test_password") {
    $issuedAt = time();
    $expirationTime = $issuedAt + 3600;  // Token 1 saat geçerli
    $payload = array(
        "iat" => $issuedAt,
        "exp" => $expirationTime,
        "username" => $username
    );

    // JWT token oluştur
    $jwt = JWT::encode($payload, $key);

    echo json_encode(
        array(
            "message" => "Başarıyla giriş yapıldı",
            "jwt" => $jwt
        )
    );
} else {
    echo json_encode(array("message" => "Geçersiz giriş"));
}
?>
```

Kullanıcı adı ve şifreyi kontrol ediyor. Eğer doğruysa, bir JWT token’ı oluşturup kullanıcıya gönderiyor.

---

## 3. **JWT Token Doğrulama**

Oluşturduğum token'ı doğrulamak için başka bir dosya oluşturdum. Bu dosya, gelen token'ı doğrulayıp geçerli olup olmadığını kontrol ediyor.

```php
<?php
require_once 'vendor/autoload.php';
use \Firebase\JWT\JWT;

$key = "my_secret_key";  // Aynı gizli anahtar

// Authorization header'dan token'ı al
$authHeader = $_SERVER['HTTP_AUTHORIZATION'];
if ($authHeader) {
    $arr = explode(" ", $authHeader);
    $jwt = $arr[1];  // Bearer token'ı al

    try {
        // Token'ı doğrula
        $decoded = JWT::decode($jwt, $key, array('HS256'));

        echo json_encode(array("message" => "Hoş geldin, " . $decoded->username));
    } catch (Exception $e) {
        echo json_encode(array("message" => "Geçersiz token"));
    }
} else {
    echo json_encode(array("message" => "Token gerekli"));
}
?>
```

Gelen token'ı alıyor ve doğruluyor. Eğer token geçerliyse, kullanıcının bilgilerini döndürür. Geçersizse hata mesajı gönderir.
