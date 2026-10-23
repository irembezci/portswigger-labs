# SSRF with Blacklist-based Input Filter

## Objective

The objective of this lab is to exploit an SSRF vulnerability to access the internal admin interface and delete the user `carlos`. The application uses a blacklist-based filter, so the goal is to bypass these restrictions.

## Steps

First, I accessed a product, used the **Check stock** functionality and intercepted the request in Burp Suite. I then sent the request to **Burp Repeater** to modify the `stockApi` parameter.

I first changed the `stockApi` value to `http://127.0.0.1/`. The request was blocked, showing that the application was filtering the standard loopback address.

<img width="2048" height="727" alt="lab3" src="https://github.com/user-attachments/assets/75e681ac-8eb2-44a3-844c-375356bd0b37" />


I bypassed this restriction by using an alternative representation of the loopback address:

```text
http://127.1/
```

The request returned a successful response, confirming that `127.1` could be used to access the local system.

I then tried to access the admin interface with:

```text
http://127.1/admin
```

This request was blocked because the blacklist also detected the `/admin` path.

<img width="2047" height="841" alt="lab3-1" src="https://github.com/user-attachments/assets/f451ed9d-ced1-4eaf-a64b-9a2a12b05a49" />

To bypass this restriction, I used double URL encoding on the letter `a` in `admin`:

```text
http://127.1/%2561dmin
```

The server decoded the value and accessed the `/admin` endpoint successfully.

<img width="2048" height="735" alt="labf" src="https://github.com/user-attachments/assets/8d5e0b0e-46c7-4667-b0c9-13b5690a8c6a" />

Finally, I modified the URL to target the user deletion endpoint:

```text
http://127.1/%2561dmin/delete?username=carlos
```

After sending the request, the internal admin endpoint processed it and deleted the `carlos` user, completing the lab. The main techniques used were **SSRF, alternative IP representation and double URL encoding to bypass blacklist-based filtering**.
