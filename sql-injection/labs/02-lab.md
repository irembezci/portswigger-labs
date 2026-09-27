# Lab: SQL injection vulnerability allowing login bypass

Bu labda SQL Injection açığı **login function** içerisinde bulunuyor ve amacımız SQL Injection kullanarak uygulamaya **`administrator` kullanıcısı olarak giriş yapmak.**

İlk olarak login request'ini Burp Suite üzerinden inceledik. Request içerisinde `username` ve `password` olmak üzere iki kullanıcı kontrollü parametre olduğunu gördük:

```text
username=test&password=test123
````

<img width="630" height="460" alt="Screenshot 2026-09-27 at 9 04 15 PM" src="https://github.com/user-attachments/assets/5e65c294-85be-460b-9b49-d98051e81f42" />

Uygulamanın arka planda bu bilgileri kullanarak kabaca şöyle bir SQL sorgusu oluşturduğunu düşünebiliriz:

```sql
SELECT * FROM users
WHERE username = 'test'
AND password = 'test123';
```

Burada hem kullanıcı adının hem de parolanın doğru olması gerekiyor. Ancak bizim administrator kullanıcısının gerçek parolasını bilmemize gerek yok. Amacımız, `username` alanını kullanarak parola kontrolünü sorgunun dışına çıkarmak.

<img width="798" height="421" alt="Screenshot 2026-09-27 at 9 11 24 PM" src="https://github.com/user-attachments/assets/9fbb7ef1-faf2-4e3e-b7b4-274d8fa70185" />

Bunun için `username` değerini:

```text
administrator'--
```

şeklinde değiştirdik. Parola kısmını ise `test123` olarak bıraktık:

```text
username=administrator'--&password=test123
```

Uygulama bu değeri SQL sorgusuna yerleştirdiğinde sorgu kabaca şöyle oluyor:

```sql
SELECT * FROM users
WHERE username = 'administrator'--'
AND password = 'test123';
```

Burada eklediğimiz `'` karakteri `administrator` değerini çevreleyen string'i kapatıyor. Ardından gelen `--` ise sorgunun geri kalanını yorum haline getiriyor.

Bu nedenle

```sql
AND password = 'test123';
```

kısmı artık çalışmıyor. Sorgunun aktif kısmı:

```sql
SELECT * FROM users
WHERE username = 'administrator'
```

haline geliyor.

<img width="1242" height="661" alt="Screenshot 2026-09-27 at 9 11 33 PM" src="https://github.com/user-attachments/assets/8ab00280-5839-4108-a6b4-8450910f90a5" />

Böylece uygulama administrator kullanıcısını buluyor ancak parolasını kontrol etmiyor. Sonuç olarak gerçek parolayı bilmeden **`administrator` kullanıcısı olarak giriş yapmış olduk.**
