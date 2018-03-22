---
id: 452
title: 'Seccon CTF 2014: Easy Cipher - Crypto 100pts'
date: 2014-12-21T15:35:19+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=452
permalink: /2014/12/easy-cipher-100pts-crypto-seccon-ctf-2014/
category: CTF
tags: [SecconCTF,Crypto]
---
## Easy Cipher

Easy Cipher 100pts Forensic Seccon Ctf 2014

<div id="table">
  <table>
    <tr>
      <th width="16%">
        Genre
      </th>
      
      <td colspan="2">
        Crypto
      </td>
    </tr>
    
    <tr>
      <th width="16%">
        Points
      </th>
      
      <td>
        100
      </td>
    </tr>
    
    <tr>
      <th width="16%">
        Question text
      </th>
      
      <td>
        <div id="detail">
          87 101 108 1100011 0157 6d 0145 040 116 0157 100000 0164 104 1100101 32 0123 69 67 0103 1001111 1001110 040 062 060 49 064 100000 0157 110 6c 0151 1101110 101 040 0103 1010100 70 101110 0124 1101000 101 100000 1010011 1000101 67 0103 4f 4e 100000 105 1110011 040 116 1101000 0145 040 1100010 0151 103 103 0145 1110011 0164 100000 1101000 0141 99 6b 1100101 0162 32 0143 111 1101110 1110100 101 0163 0164 040 0151 0156 040 74 0141 1110000 1100001 0156 056 4f 0157 0160 115 44 040 0171 1101111 117 100000 1110111 0141 0156 1110100 32 0164 6f 32 6b 1101110 1101111 1110111 100000 0164 1101000 0145 040 0146 6c 97 1100111 2c 100000 0144 111 110 100111 116 100000 1111001 6f 117 63 0110 1100101 0162 0145 100000 1111001 111 117 100000 97 114 0145 46 1010011 0105 0103 67 79 1001110 123 87 110011 110001 67 110000 1001101 32 55 060 100000 110111 0110 110011 32 53 51 0103 0103 060 0116 040 5a 0117 73 0101 7d 1001000 0141 1110110 1100101 100000 102 0165 0156 33
        </div>
      </td>
    </tr>
    
    <tr>
      <th>
        <a name="answer"></a>Answer
      </th>
      
      <td>
      </td>
    </tr>
  </table>
</div>

Crypto Kumpulan dari biner , oktal , decimal ,hexa decimal karena untuk decrypt tidak bisa langsung semua

tapi kalau kita coba satu per satu pasti akan lama dan yang paling gampang dengan menggunakan script kali ini di buat dengan ruby

<pre><code class="language-ruby">nums = %w(87 101 108 1100011 0157 6d 0145 040 116 0157 100000 0164 104 1100101
          32 0123 69 67 0103 1001111 1001110 040 062 060 49 064 100000 0157 110
          6c 0151 1101110 101 040 0103 1010100 70 101110 0124 1101000 101 100000
          1010011 1000101 67 0103 4f 4e 100000 105 1110011 040 116 1101000 0145
          040 1100010 0151 103 103 0145 1110011 0164 100000 1101000 0141 99 6b
          1100101 0162 32 0143 111 1101110 1110100 101 0163 0164 040 0151 0156
          040 74 0141 1110000 1100001 0156 056 4f 0157 0160 115 44 040 0171
          1101111 117 100000 1110111 0141 0156 1110100 32 0164 6f 32 6b 1101110
          1101111 1110111 100000 0164 1101000 0145 040 0146 6c 97 1100111 2c
          100000 0144 111 110 100111 116 100000 1111001 6f 117 63 0110 1100101
          0162 0145 100000 1111001 111 117 100000 97 114 0145 46 1010011 0105
          0103 67 79 1001110 123 87 110011 110001 67 110000 1001101 32 55 060
          100000 110111 0110 110011 32 53 51 0103 0103 060 0116 040 5a 0117 73
          0101 7d 1001000 0141 1110110 1100101 100000 102 0165 0156 33)
str = nums.map do |s|
  ord = if s.size &gt;= 5
          s.to_i(2)
        elsif s =~ /[a-f]/
          s.to_i(16)
        elsif s.start_with?('0')
          s.to_i(8)
        else
          s.to_i
        end
  ord.chr
end
puts str.join</code></pre>

Jalankan scriptnya dan ,

Welcome to the SECCON 2014 online CTF.The SECCON is the biggest hacker contest in Japan.Oops, you want to know the flag, don&#8217;t you?Here you are.SECCON{W31C0M 70 7H3 53CC0N ZOIA}Have fun!

Flag : SECCON{W31C0M 70 7H3 53CC0N ZOIA}

source : https://yous.be/2014/12/09/seccon-ctf-2014-easy-cipher-write-up/