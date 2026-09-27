# Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

Bu labda SQL Injection açığı ürünlerin **category filter** kısmında bulunuyor. Amacımız SQL Injection kullanarak veritabanının **version bilgisini elde etmek.**

İlk olarak Burp Suite üzerinden ürün kategorisini değiştiren request'i inceledik. Daha sonra `category` parametresinin SQL sorgusuna dahil edildiğini test etmek için SQL Injection payload'ları gönderdik.

Öncelikle sorgunun kaç sütun döndürdüğünü bulmamız gerekiyordu. Bunun için `ORDER BY` kullanarak sütun sayısını kontrol ettik.

```text
Accessories' ORDER BY 1#
````

Daha sonra

```text
Accessories' ORDER BY 2#
```

denedik.

Her iki sorgu da çalışırken

```text
Accessories' ORDER BY 3#
```

gönderdiğimizde uygulama hata verdi. Böylece sorgunun **2 sütun döndürdüğünü** belirledik.

Bir sonraki adımda bu sütunların metin değerlerini kabul edip etmediğini kontrol ettik. Bunun için gerçek bir veritabanı bilgisi istemek yerine, sadece test amacıyla iki metin değeri kullandık:

```text
Accessories' UNION SELECT 'abc','def'#
```
<img width="1237" height="297" alt="Screenshot 2026-09-27 at 11 56 16 PM" src="https://github.com/user-attachments/assets/4a7febd1-174a-4f17-9f85-a7a93b27f84d" />

Buradaki `abc` ve `def` gerçek veriler değildir. `UNION SELECT` ile eklediğimiz sorgunun iki sütunla uyumlu olduğunu ve bu sütunların metin değerlerini gösterebildiğini kontrol etmek için kullandığımız test değerleridir. Sütunların uygun olduğunu gördükten sonra artık veritabanının version bilgisini sorgulayabiliriz.

MySQL'de veritabanı sürümünü öğrenmek için `@@version` değişkenini kullanabiliriz:

```text
Accessories' UNION SELECT @@version,NULL#
```

Burada iki sütun olması gerektiği için ikinci sütuna `NULL` verdik. Oluşan SQL sorgusunu basitleştirirsek:

```sql
SELECT ...
FROM products
WHERE category = 'Accessories'
UNION
SELECT @@version, NULL#
```

`@@version` veritabanının sürüm bilgisini döndürür. Böylece uygulamanın normalde göstermediği **database version** bilgisini SQL Injection üzerinden elde etmiş olduk.
