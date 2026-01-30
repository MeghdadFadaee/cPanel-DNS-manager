# cPanel API Documentation (UAPI and Basic Auth)

This is a complete documentation for using the cPanel API to manage DNS and other account features.

---

## 1. Overview

cPanel provides a **REST-like API** called **UAPI**, which allows users to manage services and resources of their account.

* **Base URL:** `https://SERVER:2083/execute/`
* **Authentication:** Basic Auth (`username:password`) or API Token
* **Content Type:** JSON

> Note: For DNS management, the `DNS` module is used.

---

## 2. Authentication

### 2.1 Basic Auth

Header:

```
-u username:password
```

example with curl:

```bash
curl -u "myuser:mypassword" https://SERVER:2083/execute/DNS/parse_zone?zone=example.com
```

### 2.2 API Token

* Go to cPanel → Security → Manage API Tokens
* Create a token and use it in the header:

```bash
-H "Authorization: cpanel USERNAME:TOKEN"
```

> Recommendation: API Token is more secure than using the password.

---

## 3. DNS Management (UAPI)

### 3.1 List Records (`parse_zone`)

**URL:**

```
https://SERVER:2083/execute/DNS/parse_zone
```

**Parameters:**

| Parameter | Description        |
|-----------|--------------------|
| zone      | Zone name to parse |

**Example:**

```bash
curl -k -u "myuser:mypassword" \
"https://SERVER:2083/execute/DNS/parse_zone?zone=example.com"
```

**Output:**

* JSON object
* `dname_b64` (Base64-encoded record name)
* `data_b64` (Base64-encoded value)
* `record_type` (A, CNAME, TXT, SRV, etc.)
* `line_index` (used for edit/delete)

> All names ended with `*_b64` must be Base64 decoded.

Output Example:

```json
{
  "warnings": null,
  "errors": null,
  "data": [
    {
      "ttl": 14400,
      "type": "record",
      "record_type": "A",
      "data_b64": [
        "MC4wLjAuMAo="
      ],
      "dname_b64": "ZXhhbXBsZS5jb20K",
      "line_index": 1
    }
  ],
  "metadata": {
    "transformed": 1
  },
  "messages": null,
  "status": 1
}
```

### 3.2 Add Record (`add_zone_record`)

**URL:**

```
https://SERVER:2083/execute/DNS/add_zone_record
```

**Parameters:**

| Parameter | Description                      |
|-----------|----------------------------------|
| zone      | Zone name                        |
| name      | Record name (use @ for root)     |
| type      | Record type (A, CNAME, TXT, SRV) |
| address   | IPv4 for A record                |
| cname     | Target for CNAME                 |
| txtdata   | TXT record value                 |
| priority  | SRV priority                     |
| weight    | SRV weight                       |
| port      | SRV port                         |
| target    | SRV target                       |
| ttl       | TTL in seconds (default 14400)   |

**Example (A Record):**

```bash
curl -k -u "myuser:mypassword" \
"https://SERVER:2083/execute/DNS/add_zone_record?zone=example.com&name=www.example.com.&type=A&address=1.2.3.4&ttl=300"
```

### 3.3 Edit Record (`edit_zone_record`)

**URL:**

```
https://SERVER:2083/execute/DNS/edit_zone_record
```

**Parameters:**

| Parameter  | Description                |
|------------|----------------------------|
| zone       | Zone name                  |
| line_index | Line index from parse_zone |
| address    | New IPv4 (for A)           |
| cname      | New CNAME target           |
| txtdata    | New TXT value              |
| priority   | SRV priority               |
| weight     | SRV weight                 |
| port       | SRV port                   |
| target     | SRV target                 |
| ttl        | TTL in seconds             |

**Example (Edit A Record):**

```bash
curl -k -u "myuser:mypassword" \
"https://SERVER:2083/execute/DNS/edit_zone_record?zone=example.com&line_index=82&address=5.6.7.8"
```

### 3.4 Remove Record (`remove_zone_record`)

**URL:**

```
https://SERVER:2083/execute/DNS/remove_zone_record
```

**Parameters:**

| Parameter  | Description                |
|------------|----------------------------|
| zone       | Zone name                  |
| line_index | Line index from parse_zone |

**Example:**

```bash
curl -k -u "myuser:mypassword" \
"https://SERVER:2083/execute/DNS/remove_zone_record?zone=example.com&line_index=82"
```

### 3.5 Notes on Records

* **A Record:** IP address
* **CNAME:** Fully-qualified domain name (must end with a dot)
* **TXT:** Text string (SPF, verification, etc.)
* **SRV:** Fields: `priority weight port target` (order matters)

**Decoding Base64:**

```bash
echo MC4wLjAuMAo= | base64 -d
```

---

## 4. Error Handling

* `status: 1` → success
* `status: 0` → error, `errors` contains messages

**Common Errors:**

| Error            | Reason                                 |
|------------------|----------------------------------------|
| 401 Unauthorized | Username/password incorrect            |
| Access denied    | DNS API disabled or token invalid      |
| Zone not found   | Domain does not belong to this account |
| Invalid name     | Record name missing trailing dot       |
| Invalid value    | Value incompatible with record type    |

---

## 5. Tips

* Always use HTTPS
* SRV and TXT values must be Base64 decoded before display or use
* Use `@` or `example.com.` for root domain records
* Default TTL is 14400 unless specified otherwise
* Use `line_index` for editing and deleting records in automation

---

## 6. References

* cPanel UAPI: [https://documentation.cpanel.net/display/SDK/UAPI+Functions+-+DNS](https://documentation.cpanel.net/display/SDK/UAPI+Functions+-+DNS)
* cPanel API Tokens: [https://documentation.cpanel.net/display/SDK/API+Tokens](https://documentation.cpanel.net/display/SDK/API+Tokens)
