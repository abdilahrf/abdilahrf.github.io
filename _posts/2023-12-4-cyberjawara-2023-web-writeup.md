---
comments: true
title: Cyberjawara 2023 Web Writeup
date: 2023-12-04T00:00:00.000+07:00
category:
- ctf
tags:
- cj2023
private: false

---

### Static Web - 300pts - 63 solves

[Full Source](/uploads/cj2023/staticweb-index.js)

Given web application that serves static file by defining custom routes and using `fs.readFile` to read the content. 

```javascript

if (req.url.startsWith('/static/')) {
    const urlPath = req.url.replace(/\.\.\//g, '')
    const filePath = path.join(__dirname, urlPath);
    fs.readFile(filePath, (err, data) => {
        if (err) {
            res.writeHead(404);
            res.end("Error: File not found");
        } else {
            res.writeHead(200);
            res.end(data);
        }
    });
} 
```

Known that there could be possibility to path traversal there is a protection using `const urlPath = req.url.replace(/\.\.\//g, '')` but this regex is could be bypassed using `..././flag.txt`

```bash

curl --path-as-is "https://static-web.ctf.cyberjawara.id/static/..././config.js"
const secret = 'wWij1i23ejasdsdjvno2rnj123123';
const flag = 'CJ2023{1st_warmup_and_m1c_ch3ck}';

module.exports = {secret, flag}%    
```

![Static web flag](/uploads/static-web-flag.png)


### Magic 1 - 300pts - 28 solves

[Full Source](/uploads/cj2023/magic-1.zip)

The website allows users to upload images, subject to certain baseline restrictions. The application employs ImageMagick to process thumbnail images, and during my initial assessment, I tested for the CVE-2022-44268 vulnerability. However, it appears that the version of ImageMagick used is not vulnerable to this particular exploit.

To ensure the successful upload of an image, the system employs a function that checks various properties of the uploaded file. Here's the function:

```php

function canUploadImage($file) {
    $fileExtension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
    $finfo = new finfo(FILEINFO_MIME_TYPE);
    $fileMimeType = $finfo->file($file['tmp_name']);
    $maxFileSize = 500 * 1024;
    return (strpos($fileMimeType, 'image/') === 0 &&
        $file['size'] <= $maxFileSize &&
        strlen($file['name']) >= 30
    );
}
```
 
To pass the image upload check, the following criteria must be met:

- The filename must be 30 characters or longer.
- The file size should not exceed 500 KB.
- The file MIME type must be detected as 'image/'.

there is variable that is hold file extension, but no validation is implemented to check wheter the uploaded file is using valid image extension or not so we can use `.php` to execute php code, but since there is image validation using finfo we need to use valid image then inject the php code into exif comment.

![](/uploads/exif-comment-php.png)

the filename we can use `payloadpayloadpayloadpayload.php` then after uploading the file will be accessible at `domain/results/original-payloadpayloadpayloadpayload.php`

![](/uploads/magic1-uname.png)
![](/uploads/magic1-flag.png)



### Magic 2 - 880pts - 4 solves

[Full Source](/uploads/cj2023/magic-2.zip)

This challenge is continuing the first one, but there is modification to the code `magic.php`, now the `move_uploaded_file()` to stored the original file that we used in the first attack is removed and now there is extension validation to match `.php`.

```php
return (strpos($fileMimeType, 'image/') === 0 &&
    $fileExtension !== 'php' &&
    $file['size'] <= $maxFileSize &&
    strlen($file['name']) >= 30
);
```

Suprisingly the default extension supported in php-fpm to execute php code is `.php` and `.phar`, in this case we should use `.php.phar` because the nginx config will only send the request to php-fpm only if the url have match `\.php` regex.

[payload.php.phar](/uploads/cj2023/payloadpayloadpayloadpayloadpayload.php.phar)

so we can use this `payloadpayloadpayloadpayloadpayload.php.phar` as the filename, but at this step we cannot use exif comment to store the payload as it will be removed when the image resized, as explained in this blog https://www.synacktiv.com/publications/persistent-php-payloads-in-pngs-how-to-inject-php-code-in-an-image-and-keep-it-there.html we can use tEXt chunks to defeat the imagick resizing.


![](/uploads/magic2-flag.png)


### Wonder Drive - 850 pts - 5 solves

[Full Source](/uploads/cj2023/wonder-drive.zip)

Given website thats have functionality to upload file, create folder, share file, login, register.

Analyzing the source code giving one possible attack path that we can exploit, because the way the application stored access to file.

```python
@app.route('/accept_share/<token>', methods=['GET', 'POST'])
def accept_share(token):
    if 'username' not in session:
        return redirect(url_for('login'))

    username = session['username']

    s = URLSafeSerializer(app.secret_key)
    try:
        data = s.loads(token)
    except:
        return 'Invalid or expired share link', 404

    if request.method == 'POST':
        access_file = f"accounts/{username}/access"
        with open(access_file, "a", encoding="ascii") as f:
            f.write(f"{data['filepath']}\n")
        return redirect(url_for('user_repository_root', username=username))

    file_info = {'filepath': data['filepath'], 'user': data['user']}
    return render_template('accept_share.html', file_info=file_info, token=token)
```

this will write the repository path that we are having access to, and the challenge is to have access to `repository/wonderadmin/flag.txt`.

because of the `create_directory` function did not sanitize the directory name, we can use %0a or newline to smuggle when our repository path written to `access_file`, the payload would be like `test%0arepository/wonderadmin/flag.txt`.

The steps is to create directory `test%0arepository` then inside of that create `wonderadmin` then upload anything as `flag.txt` file, click to share access and accept it.

```
repository/test
repository/wonderadmin/flag.txt
```

and we now have access to read flag from wonderadmin.

![](/uploads/wonderadmin-flag.png)
