# Lab: SQL injection attack, querying the database type and version on Oracle

Bu labda SQL Injection açığı ürünlerin **category filter** kısmında bulunuyor. Bu kez amacımız SQL Injection kullanarak veritabanından **Oracle database version bilgisini elde etmek.**

İlk olarak ürün kategorisini değiştiren request'i Burp Suite üzerinden inceledik. Kategori değerinin SQL sorgusuna dahil edildiğini bildiğimiz için önce sorgunun kaç sütun döndürdüğünü bulmamız gerekiyor.

Bunun için `ORDER BY` kullanarak sütun sayısını kontrol ettik.

```text
Accessories' ORDER BY 1--
````
<img width="1254" height="450" alt="Screenshot 2026-09-27 at 9 47 59 PM" src="https://github.com/user-attachments/assets/62bf4157-c9e4-4cf4-9eb6-c5a2281ae86e" />

Bu istek çalıştı.

Daha sonra

```text
Accessories' ORDER BY 2--
```

gönderdik ve bu da çalıştı.

<img width="1258" height="601" alt="Screenshot 2026-09-27 at 9 50 54 PM" src="https://github.com/user-attachments/assets/f4518d43-e48e-4c8b-9a2a-0d5438fe15c3" />

Son olarak

```text
Accessories' ORDER BY 3--
```

gönderdiğimizde `500 Internal Server Error` aldık. Buradan sorgunun **2 sütun döndürdüğünü** anladık. Çünkü `ORDER BY 1` ve `ORDER BY 2` çalışırken `ORDER BY 3` çalışmadı.

Artık `UNION SELECT` kullanırken bizim eklediğimiz sorgunun da iki sütun döndürmesi gerekiyor. Öncelikle UNION sorgusunun çalışıp çalışmadığını görmek için iki sütuna basit metin değerleri yerleştirdik:

```text
Accessories' UNION SELECT 'test1','test2' FROM dual--
```

Burada `test1` ve `test2` veritabanından aldığımız gerçek değerler değil. Sadece iki sütuna da metin gönderebildiğimizi kontrol etmek için kullandığımız test değerleri.

Oracle veritabanlarında `SELECT` sorgularında `FROM` kullanılması gerektiği için `dual` tablosunu ekledik:

```sql
UNION SELECT 'test1','test2' FROM dual
```

Bu aşamadan sonra artık asıl istediğimiz veriyi sorgulayabiliriz.

<img width="1255" height="419" alt="Screenshot 2026-09-27 at 10 09 44 PM" src="https://github.com/user-attachments/assets/f52e03b3-2330-4d1d-98bf-4249ca15066d" />

Labın amacı Oracle'ın **database version** bilgisini almak olduğu için `test1` ve `test2` yerine Oracle'ın versiyon bilgisini içeren `BANNER` sütununu kullandık.

Kullandığımız payload:

```text
Accessories' UNION SELECT BANNER,NULL FROM v$version--
```
<img width="1257" height="373" alt="Screenshot 2026-09-27 at 10 18 49 PM" src="https://github.com/user-attachments/assets/c55ed51a-0fff-4ed0-b82c-90d02431add0" />

Buradaki parçaları incelediğimizde

```text
BANNER
```

Oracle'ın versiyon bilgisini içeren sütundur.

```text
NULL
```

ise ikinci sütunu doldurmak için kullanılıyor. Çünkü daha önce yaptığımız testlerde sorgunun iki sütun döndürdüğünü bulmuştuk.

```text
v$version
```

Oracle'ın veritabanı ve bileşenlerinin versiyon bilgilerini içeren sistem görünümüdür. Son olarak `--` ile uygulamanın bizim girdimizden sonra eklediği SQL'in geri kalanını yorum haline getiriyoruz.

<img width="601" height="145" alt="Screenshot 2026-09-27 at 10 33 06 PM" src="https://github.com/user-attachments/assets/61bdbaf9-8653-4cf0-8368-2440d9a85417" />

Sonuç olarak

```sql
UNION SELECT BANNER,NULL FROM v$version--
```

ile mevcut sorguya Oracle'ın version bilgisini döndüren kendi sorgumuzu eklemiş olduk. Response içerisinde Oracle'ın version string'i görüntülendi ve lab tamamlandı.
