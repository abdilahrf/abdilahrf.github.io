---
id: 1813
title: Writeup ICE CTF
date: 2015-09-04T16:06:58+00:00
author: abdilahrf
layout: post
guid: https://www.hasnydes.us/?p=1813
permalink: /2015/09/writeup-ice-ctf/
categories: Misc
tags: [IceCTF]
---
#### Warm Up &#8211; 10 

> &#8216;`_ the Planet!`&#8216; Fill in the blank
> 
> Hint : You should probably watch the movie <a href="http://www.imdb.com/title/tt0113243/" target="_blank">Hackers</a>, go on&#8230; we&#8217;ll wait
> 
> Flag : **hack**

&nbsp;

#### Facebook &#8211; 15 {.panel-title}

> Could there be something hidden on our <a href="https://facebook.com/icectf" target="_blank">facebook page</a>?
> 
> Hint : If I were a flag, where would I be?
> 
> [<img class="aligncenter size-full wp-image-1817" src="http://abdilahrf.github.io/images/2015/09/flag-facebook.png" alt="flag facebook" width="1366" height="768" />](http://abdilahrf.github.io/images/2015/09/flag-facebook.png)
> 
> **flag** : flag\_why\_hide\_stuff\_on_facebook

####  {.panel-title}

#### Not Found &#8211; 15 {.panel-title}

> I lost this problem!!! Can you help me find it? There&#8217;s a flag in it for you if you do 😉
> 
> Hint : Perhaps it&#8217;s on the CDIV(194) page?
> 
> ini flag yang di temuin secara ga sengaja , kalau di liat dari hintnya CDIV(194) , dari google ngak ngebantu apapun ya , tapi pas gw coba buka <a href="https://icec.tf/play/logout" target="_blank">https://icec.tf/play/logout</a> dan ternyata not found , flagnya ada di 404 page :/
> 
> [<img class="aligncenter size-full wp-image-1816" src="http://abdilahrf.github.io/images/2015/09/404-flag.png" alt="404 flag" width="1366" height="768" />](http://abdilahrf.github.io/images/2015/09/404-flag.png)
> 
> **Flag** : flag\_there\_you\_are\_you\_silly\_flag

&nbsp;

#### Numeric &#8211; 20 {.panel-title}

> bG9zdF9pbl90aGVfbnVtYmVycwo=
> 
> ini keliatannya mengarah ke base64 , setelah di decode ternyata benar
> 
> **Flag :** lost\_in\_the_numbers

#### Oh No! &#8211; 20 {.panel-title}

> We seem to have misplaced the flag! Sorry about that, but we swear it was left on this <a href="https://icec.tf/api/autogen/serve/flag.html?static=false&pid=6f256632fcf2f334335d0391eb53ad91" target="_blank">site</a>. Perhaps you can find it?
> 
> Hint : Is it possible that it&#8217;s there, but you just can&#8217;t see it?
> 
> flagnya ternyata ada di source
> 
> **Flag :**flag_cbaeed98896e32e0fc0ff62b7e260318b9c99a46

&nbsp;

#### ROT13 &#8211; 25 {.panel-title}

> Can you decipher <a href="https://icec.tf/problem-static/stage1/crypto/rot13/code.txt" target="_blank">this secret message</a>?
> 
> Hint : This cipher better be crackable&#8230;
> 
> decode dengan rot13
> 
> **Flag :**rot\_13\_isnt_secure

&nbsp;

#### Document Troubles &#8211; 30 {.panel-title}

> We found this <a href="https://icec.tf/problem-static/stage2/forensics/doc_troubles/file.docx" target="_blank">document</a>, and we think it contains the flag. Can you find it?
> 
> Hint : What exactly is a .docx file?
> 
> download .docx filenya dan extract untuk mendapatkan flag.txt
> 
> [<img class="aligncenter size-full wp-image-1818" src="http://abdilahrf.github.io/images/2015/09/flag-docx.png" alt="flag docx" width="1366" height="768" />](http://abdilahrf.github.io/images/2015/09/flag-docx.png)
> 
> **Flag :**this\_would\_be\_the\_flag\_you\_are\_looking\_for

#### Simple &#8211; 30 {.panel-title}

> Simple, right? `nc vuln2015.icec.tf 10000`.
> 
> Hint : What is at the other end?

&nbsp;

    abdilahrf@hasnydes:~$ nc vuln2015.icec.tf 10000
    My character is j(0x6a)
    Can you add some (positive) number larger than 100 to turn it into the letter i(0x69)? (i like it)
    255
    Great job! The flag is: very_simple_right
    
    
    

#### Liar &#8211; 30 {.panel-title}

> We found this <a href="https://icec.tf/problem-static/stage1/web/liar/liar.html" target="_blank">site</a> which seems to just give you the flag. You wouldn&#8217;t mind entering it for us, would you?
> 
> Hint : It&#8217;s not as easy as it seems. Maybe the flag is hidden somewhere? Is it executable?

pertama gw kira flagnya **<span id="f">not_this_time  </span>**<span id="f">, haha ternyata bukan nga semudah itu setelah gw coba view-source ternyata ada javascript function yang di *obcusfate gw mencoba untuk decode namun hasilnya tambah susah untuk di translate arti dari script itu</span> <span id="f"></span>

ternyata

    
    window.s=[0x32, 0x66, 0x32, 0x65, 0x65, 0x35, 0x30, 0x63, 0x38, 0x39, 0x65, 0x36, 0x38, 0x35, 0x32, 0x33, 0x30, 0x64, 0x33, 0x61, 0x35, 0x61, 0x36, 0x64, 0x36, 0x37, 0x35, 0x63, 0x36, 0x35, 0x63, 0x31, 0x70, 0x72, 0x69, 0x6e, 0x74, 0x46, 0x6c, 0x61, 0x67, 0x32, 0x66, 0x32, 0x65, 0x65, 0x35, 0x30, 0x63, 0x38, 0x39, 0x65, 0x36, 0x38, 0x35, 0x32, 0x33, 0x30, 0x64, 0x33, 0x61, 0x35, 0x61, 0x36, 0x64, 0x36, 0x37, 0x35, 0x63, 0x36, 0x35, 0x63, 0x31];
    

setelah di decode itu adalah &#8220;printFlagprintFlagprintFlag&#8221;
  
dan ternyata kalau

    
    String.fromCharCode.apply(null,window.s.slice(0x20,0x29))
    
    

di jalankan di console dia mengambil kata printFlag sebagai nama fungsi , dan tinggal kita jalankan fungsi printFlag() untuk memunculkan flagnya

[<img class="aligncenter size-full wp-image-1822" src="http://abdilahrf.github.io/images/2015/09/printFlag.png" alt="printFlag" width="1366" height="768" />](http://abdilahrf.github.io/images/2015/09/printFlag.png)

**flag :** hidden\_in\_the_code

&nbsp;

#### Cryptic Crypto &#8211; 35 {.panel-title}

> Can you take a look at this non-sense <a href="https://icec.tf/problem-static/stage1/crypto/cryptic_crypto/text.txt" target="_blank">wall of text</a> we stumbled upon?
> 
> Hint : We&#8217;re fairly sure this contains english text

&nbsp;

sepertinya itu adalah subtitution cipher

    watvkljanvct vamla kl kcr ilhrae njr fnz ruurwkmxrbt zteletiloz fmkc rewatvkmle, kcr wlexrazmle lu meulainkmle uali n arnhnsbr zknkr kl nvvnarek elezrezr. kcr lamjmenkla lu ne rewatvkrh irzznjr zcnarh kcr hrwlhmej krwcemyor errhrh kl arwlxra kcr lamjmenb meulainkmle lebt fmkc mekrehrh arwmvmrekz, kcrarst varwbohmej oefnekrh vrazlez uali hlmej kcr znir. zmewr flabh fna m neh kcr nhxrek lu kcr wlivokra, kcr irkclhz ozrh kl wnaat lok watvklbljt cnxr srwlir mewarnzmejbt wlivbrp neh mkz nvvbmwnkmle ilar fmhrzvarnh.
    
    ilhrae watvkljanvct mz crnxmbt snzrh le inkcrinkmwnb kcrlat neh wlivokra zwmrewr vanwkmwr; watvkljanvcmw nbjlamkciz nar hrzmjerh naloeh wlivoknkmlenb cnaherzz nzzoivkmlez, inqmej zowc nbjlamkciz cnah kl sarnq me vanwkmwr st net nhxraznat. mk mz kcrlarkmwnbbt vlzzmsbr kl sarnq zowc n ztzkri, sok mk mz meurnzmsbr kl hl zl st net qelfe vanwkmwnb irnez. kcrzr zwcrirz nar kcrarular krairh wlivoknkmlenbbt zrwoar; kcrlarkmwnb nhxnewrz, r.j., mivalxrirekz me mekrjra unwklamgnkmle nbjlamkciz, neh unzkra wlivokmej krwcelbljt aryomar kcrzr zlbokmlez kl sr wlekmeonbbt nhnvkrh. kcrar rpmzk meulainkmle-kcrlarkmwnbbt zrwoar zwcrirz kcnk valxnsbt wneelk sr salqre rxre fmkc oebmimkrh wlivokmej vlfraâ€”ne rpnivbr mz kcr ler-kmir vnhâ€”sok kcrzr zwcrirz nar ilar hmuumwobk kl mivbrirek kcne kcr srzk kcrlarkmwnbbt sarnqnsbr sok wlivoknkmlenbbt zrwoar irwcnemziz.
    
    kcr jalfkc lu watvkljanvcmw krwcelbljt cnz anmzrh n eoisra lu brjnb mzzorz me kcr meulainkmle njr. watvkljanvct'z vlkrekmnb ula ozr nz n kllb ula rzvmlenjr neh zrhmkmle cnz brh inet jlxraeirekz kl wbnzzmut mk nz n frnvle neh kl bmimk la rxre valcmsmk mkz ozr neh rpvlak. me zlir doamzhmwkmlez fcrar kcr ozr lu watvkljanvct mz brjnb, bnfz vraimk mexrzkmjnklaz kl wlivrb kcr hmzwblzoar lu rewatvkmle qrtz ula hlwoirekz arbrxnek kl ne mexrzkmjnkmle. watvkljanvct nbzl vbntz n indla albr me hmjmknb amjckz inenjrirek neh vmanwt lu hmjmknb irhmn.
    
    ubnj_zoszkmkokmle_wmvcraz_nar_snh
    
    

decode menggunakan tools
  
http://rumkin.com/tools/cipher/cryptogram-solver.php

    
    CRYPTOGRAPHY PRIOR TO THE MODERN AGE WAS EFFECTIVELY SYNONYMOUS WITH ENCRYPTION THE CONVERSION OF INFORMATION FROM A READABLE STATE TO APPARENT NONSENSE THE ORIGINATOR OF AN ENCRYPTED MESSAGE SHARED THE DECODING TECHNIQUE NEEDED TO RECOVER THE ORIGINAL INFORMATION ONLY WITH INTENDED RECIPIENTS THEREBY PRECLUDING UNWANTED PERSONS FROM DOING THE SAME SINCE WORLD WAR I AND THE ADVENT OF THE COMPUTER THE METHODS USED TO CARRY OUT CRYPTOLOGY HAVE BECOME INCREASINGLY COMPLEX AND ITS APPLICATION MORE WIDESPREAD MODERN CRYPTOGRAPHY IS HEAVILY BASED ON MATHEMATICAL THEORY AND COMPUTER SCIENCE PRACTICE CRYPTOGRAPHIC ALGORITHMS ARE DESIGNED AROUND COMPUTATIONAL HARDNESS ASSUMPTIONS MAKING SUCH ALGORITHMS HARD TO BREAK IN PRACTICE BY ANY ADVERSARY IT IS THEORETICALLY POSSIBLE TO BREAK SUCH A SYSTEM BUT IT IS INFEASIBLE TO DO SO BY ANY KNOWN PRACTICAL MEANS THESE SCHEMES ARE THEREFORE TERMED COMPUTATIONALLY SECURE THEORETICAL ADVANCES E G IMPROVEMENTS IN INTEGER FACTORIZATION ALGORITHMS AND FASTER COMPUTING TECHNOLOGY REQUIRE THESE SOLUTIONS TO BE CONTINUALLY ADAPTED THERE EXIST INFORMATION THEORETICALLY SECURE SCHEMES THAT PROVABLY CANNOT BE BROKEN EVEN WITH UNLIMITED COMPUTING POWER AN EXAMPLE IS THE ONE TIME PAD BUT THESE SCHEMES ARE MORE DIFFICULT TO IMPLEMENT THAN THE BEST THEORETICALLY BREAKABLE BUT COMPUTATIONALLY SECURE MECHANISMS THE GROWTH OF CRYPTOGRAPHIC TECHNOLOGY HAS RAISED A NUMBER OF LEGAL ISSUES IN THE INFORMATION AGE CRYPTOGRAPHY'S POTENTIAL FOR USE AS A TOOL FOR ESPIONAGE AND SEDITION HAS LED MANY GOVERNMENTS TO CLASSIFY IT AS A WEAPON AND TO LIMIT OR EVEN PROHIBIT ITS USE AND EXPORT IN SOME JURISDICTIONS WHERE THE USE OF CRYPTOGRAPHY IS LEGAL LAWS PERMIT INVESTIGATORS TO COMPEL THE DISCLOSURE OF ENCRYPTION KEYS FOR DOCUMENTS RELEVANT TO AN INVESTIGATION CRYPTOGRAPHY ALSO PLAYS A MAJOR ROLE IN DIGITAL RIGHTS MANAGEMENT AND PIRACY OF DIGITAL MEDIA FLAG SUBSTITUTION CIPHERS ARE BAD
    
    

**Flag :** FLAG\_SUBSTITUTION\_CIPHERS\_ARE\_BAD

&nbsp;

#### Logoventures &#8211; 35 {.panel-title}

> <p class="problem-description">
>   I believe I misplaced something in this <a href="https://icec.tf/problem-static/stage1/forensics/logoventures/logo.gif" target="_blank">image</a>, could you see if you can find it for me?
> </p>
> 
> <div class="row">
>    Hint : Only if you could slow down time itself
> </div>

<div class="row">
</div>

<div class="row">
  <a href="http://abdilahrf.github.io/images/2015/09/flag.gif"><img class="aligncenter size-full wp-image-1832" src="http://abdilahrf.github.io/images/2015/09/flag.gif" alt="flag" width="300" height="400" /></a>file flagnya diselipkan di 1 layer yang tampil mungkin kurang dari 0.01s frame rate , jadi kita bisa buka via &#8220;Gimp/Photoshop&#8221; untuk melihat frame yang di cari , di case ini gw pake ini <a href="http://regex.info/exif.cgi?imgurl=https%3A%2F%2Ficec.tf%2Fproblem-static%2Fstage1%2Fforensics%2Flogoventures%2Flogo.gif" target="_blank">Exif</a>
</div>

<div class="row">
</div>

<div class="row">
  <strong>Flag : </strong>boy_this_goes_by_so_fast
</div>