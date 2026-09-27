# Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

Bu labda SQL Injection açığı ürünlerin **category filter** kısmında bulunuyor. Kullanıcı bir kategori seçtiğinde uygulama bu değeri kullanarak veritabanında bir SQL sorgusu çalıştırıyor. PortSwigger'ın verdiği örnek sorgu şu şekilde:

```sql
SELECT * FROM products
WHERE category = 'Gifts'
AND released = 1;
````

Burada `category` kullanıcının seçtiği kategoriye göre değişiyor. `released = 1` ise sadece yayınlanmış ürünlerin gösterilmesini sağlıyor. Labdaki amacımız ise **yayınlanmamış ürünleri de görüntülemek.**

<img width="928" height="484" alt="Screenshot 2026-09-27 at 7 55 44 PM" src="https://github.com/user-attachments/assets/08fdf886-5752-4c96-854d-3296d0170d4a" />

İlk olarak request'i Burp Suite üzerinden inceleyelim. Bizim labımızda kategori `Accessories` olarak gönderiliyor:

```text
category=Accessories
```
<img width="1262" height="472" alt="Screenshot 2026-09-27 at 7 38 58 PM" src="https://github.com/user-attachments/assets/928371ae-5838-4839-9eea-49058539eef0" />

Uygulamanın bunu SQL sorgusuna dahil ettiğini düşündüğümüzde sorgu kabaca şöyle oluyor:

```sql
SELECT * FROM products
WHERE category = 'Accessories'
AND released = 1;
```

Buraya kadar her şey normal. Kullanıcı bir kategori seçiyor ve uygulama da bu kategoriye göre ürünleri getiriyor.

<img width="1242" height="799" alt="Screenshot 2026-09-27 at 7 37 45 PM" src="https://github.com/user-attachments/assets/95142fd0-a914-4d6d-b096-e6d41ea512f7" />

Burada bizim için önemli olan şey, **kullanıcının girdiği `Accessories` değerinin SQL sorgusunun içine giriyor olması.** Bunu daha iyi anlamak için kategori değerinin sonuna tek tırnak ekleyerek:

```text
Accessories'
```

gönderdik.

<img width="1243" height="385" alt="Screenshot 2026-09-27 at 7 41 22 PM" src="https://github.com/user-attachments/assets/fa0deb3e-d9d1-4d1a-adf8-3c9b91afd604" />

Bu sefer uygulama `500 Internal Server Error` döndürdü. Tek başına bu sonuç SQL Injection olduğunu kanıtlamıyor ancak sadece bir `'` eklediğimizde uygulamanın davranışının değişmesi, girdimizin SQL sorgusunu etkiliyor olabileceğini gösterdi.

Bunun üzerine, uygulamanın kullanıcıdan aldığımız `Accessories` değerini SQL sorgusuna nasıl eklediğini düşünelim.

Uygulamanın mantığını basitleştirirsek:

```text
WHERE category = ' + kullanıcı girdisi + ' AND released = 1
````

Normalde kullanıcı:

```text
Accessories
```

girdiğinde uygulamanın oluşturduğu sorgu:

```sql
WHERE category = 'Accessories'
AND released = 1
```

şeklinde oluyor.

Yani uygulama bizim girdiğimiz `Accessories` değerini kendi eklediği tırnakların arasına yerleştiriyor. Bu yüzden bizim gönderdiğimiz değer, sadece uygulamanın göstereceği kategoriyi belirlemekle kalmıyor. Eğer girdinin içine SQL ifadeleri ekleyebilirsek, bunlar da uygulamanın oluşturduğu sorgunun içine dahil olabilir. Bunu kullanarak sorgunun nasıl çalıştığını değiştirmeyi deneyebiliriz.

Öncelikle `Accessories` değerinin sonuna bir `'` ekliyoruz:

```text
Accessories'
```

Buradaki amacımız `Accessories` değerini çevreleyen string'i kapatmak. Çünkü uygulama zaten girdimizin sonuna kendi tırnağını ekliyor. Biz kendi tırnağımızı girdinin içine koyduğumuzda, SQL sorgusundaki string daha erken kapanmış oluyor. Böylece artık string'in içinde yalnızca bir kategori adı bulunması gerekmiyor. String'i kapattıktan sonra sorgunun devamına kendi SQL ifademizi ekleyebiliriz.

Bu nedenle girdiyi:

```text
Accessories' OR 1=1--
```

şeklinde değiştiriyoruz.

Buradaki parçaları sırayla inceleyelim.

İlk olarak:

```text
'
```

`Accessories` değerini kapatıyor.

Ardından:

```sql
OR 1=1
```

ekliyoruz. `1=1` her zaman doğru olan bir koşul olduğu için `OR` ile birlikte sorgunun sonucunu değiştiriyor. Artık sorgunun yalnızca `category = 'Accessories'` koşuluna uyması gerekmiyor.

Son olarak:

```text
--
```

ekliyoruz. Bunun amacı, bizim girdimizden sonra uygulamanın otomatik olarak ekleyeceği SQL kısmını yorum haline getirmek.

Uygulama bizim gönderdiğimiz değeri kendi sorgusuna yerleştirdiğinde sorgu şu hale geliyor:

```sql
SELECT * FROM products
WHERE category = 'Accessories' OR 1=1--'
AND released = 1;
```

Burada bizim eklediğimiz `'` karakteri `Accessories` string'ini kapatmış oldu. Ardından `OR 1=1` sorguya yeni bir koşul ekledi.

`1=1` her zaman doğru olduğu için:

```sql
category = 'Accessories' OR 1=1
```

ifadesi yalnızca `Accessories` kategorisindeki ürünleri seçmekle sınırlı kalmıyor.

Sonrasında gelen `--` ise uygulamanın eklediği:

```sql
AND released = 1
```

kısmının yorum olarak değerlendirilmesini sağlıyor. Böylece ürünlerin yayınlanmış olup olmadığına bakılmıyor. Sonuç olarak uygulama, normalde yalnızca `released = 1` olan ürünleri gösterirken **yayınlanmamış ürünleri de göstermeye başlıyor** ve lab tamamlanıyor.
