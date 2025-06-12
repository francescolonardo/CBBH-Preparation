# [File Upload Attacks](https://academy.hackthebox.com/module/details/136)

## Skills Assessment

You are contracted to perform a penetration test for a company's e-commerce web application. The web application is in its early stages, so you will only be testing any file upload forms you can find.

Try to utilize what you learned in this module to understand how the upload form works and how to bypass various validations in place (if any) to gain remote code execution on the back-end server.

**Extra Exercise**

Try to note down the main security issues found with the web application and the necessary security measures to mitigate these issues and prevent further exploitation.

### Questions

#### Question #01

**Question**

Try to exploit the upload form to read the flag found at the root directory `/`.

![Firefox - Homepage](./assets/screenshots/file_upload_attacks_firefox_homepage.png)

![Firefox - File Upload Page](./assets/screenshots/file_upload_attacks_firefox_file_upload_page.png)

![Burp Suite - File Upload Attempt 1](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_1.png)

```
┌─[eu-academy-1]─[10.10.15.8]─[htb-ac-1461567@htb-w2arhmpril]─[~]
└──╼ [★]$ python3 -c "from PIL import Image; Image.new('RGBA', (1, 1), (0, 0, 0, 0)).save('test.png')"

┌─[eu-academy-1]─[10.10.15.8]─[htb-ac-1461567@htb-w2arhmpril]─[~]
└──╼ [★]$ file test.png 

test.png: PNG image data, 1 x 1, 8-bit/color RGBA, non-interlaced
```

![Burp Suite - File Upload Attempt 2](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_2.png)

![Burp Suite - File Upload Attempt 3](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_3.png)

![Burp Suite - File Upload Attempt 4](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_4.png)

![Burp Suite - File Upload Attempt 5](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_5.png)

![Burp Suite - File Upload Attempt 6](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_6.png)

![Burp Suite - File Upload Attempt 7](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_7.png)

![Burp Suite - File Upload Attempt 8](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_8.png)

![Burp Suite - File Upload Attempt 9](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_9.png)

![Burp Suite - File Upload Attempt 10](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_10.png)

```php
<?php
require_once('./common-functions.php');

// uploaded files directory
$target_dir = "./user_feedback_submissions/";

// rename before storing
$fileName = date('ymd') . '_' . basename($_FILES["uploadFile"]["name"]);
$target_file = $target_dir . $fileName;

// get content headers
$contentType = $_FILES['uploadFile']['type'];
$MIMEtype = mime_content_type($_FILES['uploadFile']['tmp_name']);

// blacklist test
if (preg_match('/.+\.ph(p|ps|tml)/', $fileName)) {
    echo "Extension not allowed";
    die();
}

// whitelist test
if (!preg_match('/^.+\.[a-z]{2,3}g$/', $fileName)) {
    echo "Only images are allowed";
    die();
}

// type test
foreach (array($contentType, $MIMEtype) as $type) {
    if (!preg_match('/image\/[a-z]{2,3}g/', $type)) {
        echo "Only images are allowed";
        die();
    }
}

// size test
if ($_FILES["uploadFile"]["size"] > 500000) {
    echo "File too large";
    die();
}

if (move_uploaded_file($_FILES["uploadFile"]["tmp_name"], $target_file)) {
    displayHTMLImage($target_file);
} else {
    echo "File failed to upload";
}
```

![Burp Suite - File Upload Attempt 11](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_11.png)

![Burp Suite - File Upload Attempt 12](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_12.png)

![Burp Suite - File Upload Attempt 13](./assets/screenshots/file_upload_attacks_burpsuite_file_upload_attempt_13.png)

**Answer**

```
HTB{m4573r1ng_upl04d_3xpl0174710n}
```

---
---
