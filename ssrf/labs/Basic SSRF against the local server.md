## Basic SSRF Against the Local Server

### Objective

The goal was to exploit an SSRF vulnerability in the stock check functionality to access the internal admin interface and delete the user `carlos`.

### Steps

First, I accessed a product and clicked **Check stock**. I intercepted the resulting `POST /product/stock` request in Burp Suite and sent it to **Repeater**.

The request contained a `stockApi` parameter:

```text
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```
<img width="488" height="458" alt="ssrf3" src="https://github.com/user-attachments/assets/edaf3fb9-69f8-4418-be23-fa9c5bf94af2" />

The application uses this parameter to make a server-side request to the specified URL.

I replaced the `stockApi` value with:

```text
http://localhost/admin
```

The server returned the internal admin interface in the response, confirming that the application was vulnerable to **SSRF**.

The admin interface revealed the deletion endpoint for the target user:

```text
/admin/delete?username=carlos
```
<img width="648" height="490" alt="ssrf2" src="https://github.com/user-attachments/assets/38a7fbf6-e8f2-4297-af9a-ee92d27d53c5" />

I then changed the `stockApi` parameter to:

```text
http://localhost/admin/delete?username=carlos
```

and sent the request again through Burp Repeater.

The server processed the request through its own internal access to `localhost`, causing the `carlos` account to be deleted.

<img width="2048" height="783" alt="ssrf1" src="https://github.com/user-attachments/assets/329bdad0-b154-4bde-9441-9c231598ce7f" />


### Key Takeaway

The vulnerability occurred because the application trusted the user-controlled `stockApi` parameter and allowed the server to make requests to internal resources.

**SSRF flow:**

```text
Attacker
   ↓
/product/stock
   ↓
stockApi=http://localhost/admin
   ↓
Internal Admin Interface
```

The same mechanism was then used to access:

```text
http://localhost/admin/delete?username=carlos
```

This demonstrated how SSRF can be used to access internal-only functionality that is not directly accessible to an external user.
