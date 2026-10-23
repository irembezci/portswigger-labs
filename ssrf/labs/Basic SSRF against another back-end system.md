# Basic SSRF against another back-end system

## Objective

Use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`.

## Steps

1. Visit a product, click **Check stock**, intercept the request in Burp Suite, and send it to **Burp Intruder**.

<img width="2048" height="800" alt="lab2 1" src="https://github.com/user-attachments/assets/8336c689-2130-4d93-813b-260739ba434e" />


2. Change the `stockApi` parameter to:

```text
http://192.168.0.1:8080/admin
```

Highlight the final octet of the IP address and click **Add §**:

```text
http://192.168.0.§1§:8080/admin
```

<img width="766" height="196" alt="lab22" src="https://github.com/user-attachments/assets/abf1b6b5-7a77-4a2d-9031-6e2f8dd0e5e4" />

3. In the **Payloads** panel, select **Numbers** and configure the payload range:

```text
From: 1
To: 255
Step: 1
```
<img width="450" height="579" alt="lab23" src="https://github.com/user-attachments/assets/0c721deb-a13d-48a4-99d8-e567208d1db2" />

4. Click **Start attack** and wait for the requests to complete.

5. Sort the results by **Status code**. The request with payload `114` returned a `200` response, identifying the internal server:

```text
192.168.0.114:8080
```
<img width="2048" height="162" alt="lab24" src="https://github.com/user-attachments/assets/17012263-eacd-48e2-a9a5-299fdc154c90" />

6. Send the successful request to **Burp Repeater** and change the `stockApi` parameter to:

```text
http://192.168.0.114:8080/admin/delete?username=carlos
```

7. Send the request to delete the user `carlos`.

<img width="2048" height="756" alt="lab25" src="https://github.com/user-attachments/assets/245c3f54-4ab8-4669-aaba-48888791d8a1" />

8. Return to the lab page and verify that the lab is marked as **Solved**.
