# Wireshark Challenge

> Unfortunately, DNS Corporation still uses a number of very old routers throughout there network. On the bright side though, even though these devices don't automatically use SSL for their management portals, your admins know to make sure to always use https. As part of your pentest, you managed to capture some traffic during when an admin changed the wifi password. Since it's SSL encrypted, you shouldn't be able to decrypt it, right? It's not like you'd be able to find the private key for someone's router on the internet. RIGHT? Download snoop.pcap, and let's find out. Your flag is the new Wifi password.

> [Download the PCAP here](https://range.metaproblems.com/739c7a4b6b9d8d9281bb3a4c964e68ca/snoop.pcap) or grab it with wget below

> *Tip: for this one, see if you can find a repository that stores hardcoded SSL keys.*


A database of private SSL/SSH keys can be found at this GitHub repo:
[devttys0/littleblackbox](https://github.com/devttys0/littleblackbox)


First, we need to get the VM prepared by installing a few libraries, most of which are referenced on the github link above.
<br>
`sudo apt-get update`

`sudo apt-get install libssl-dev libpcap-dev libsqlite3-dev`

We *also* need to install an older version of OpenSSL because the `sudo make install` command we'll need to run shortly will fail without it:

`sudo apt install libssl1.0-dev`

Download the PCAP file with wget if you didn't download it manually already:
<br>
`wget https://range.metaproblems.com/739c7a4b6b9d8d9281bb3a4c964e68ca/snoop.pcap`

Use git clone to download the git file for littleblackbox:
<br>
`git clone https://github.com/devttys0/littleblackbox.git`

`cd littleblackbox/src`

`./configure`

`make`

`sudo make install`

This command threw an error for me:
>adhd@DESKTOP-I1T2G01:/mnt/c/Users/adhd/littleblackbox/src$ sudo make install
>cp ../docs/littleblackbox.1.gz /usr/local/share/man/man1/littleblackbox.1.gz
>cp: cannot create regular file '/usr/local/share/man/man1/littleblackbox.1.gz': No such file or directory

I went ahead and created the directory manually:
<br>
`sudo mkdir /usr/local/share/man/man1`

Continuing on:
<br>
`sudo make install`

From the littleblackbox github page, we can see we need to run the following:
<br>
`littleblackbox --pcap=/mnt/c/Users/adhd/snoop.pcap`

The output gives us an RSA Private Key as output:
>-----BEGIN RSA PRIVATE KEY-----
MIICWwIBAAKBgQDOPa+w/2o5IuWs3eV2MVXEpyqLYfZScbyPpr2mY8zkbdKC6DFq
zG6cBY7S06qobVjXmOgQMkoVoO8ihbD1NB6V/4xyDgMwJJ8uSfpaB/JyzefeoNz9
Gcg+s+wpKoG84PTHyfVy6xMTCwZ+qC26JLGPquu/ucwEljHy0WVYPmb9VQIDAQAB
AoGAYrG+W9M+f+0lP95IKpFdW+grQdw1RirLc2r1oqRrrnynmqGG1HbUD7HRMS69
ojABrdqsYuPN9B+5kCmuDwlMANwIwV3ZwxE7A7Hy1tpi9PgckTjZW8rCl3ciEZkx
Y+Xw9j9QGlSI6Hxthocb/4eCwwMenLrSZDj6oKuZ7DgJUJkCQQDl88c7RJsTS6HN
ztAjFxpKobIgzy9u1AH15WDqqd2rawtJk2FTFcz0GrAy/gawKU42wFqZOKv28iMq
96fGcPN3AkEA5ZpSL+vQD1WAEd7Vv56zqmTOTpEOGoDD5zxfch4gvr8rCgU6hDmz
0Y3UQ7MRJrTNvVwYXpIUoazBBUZUfbpQkwJAagxTBXJOUke/BzspogU1itWnYJos
NeBwRwbR+2b7Y+KqAfSGHdsf+jOUru+YBgYGnBl5rtAD/o8MyPQN2+abYQJABhbD
mzW7vMxdqxunu38v8JLfzcGXCCjmCRnWxiX6ZFSZhZiB5sPI+wOx32G+ULJ2ylDI
7KkfFvKH4+Xrk7H/NQJAJWQusAs1tHhDDddvcvqe4J5q0qvNdOSTs0Cu2CimWPxe
tfcz64o64XWgmCAaFq2pfaN4oC1kaGnIbUEdtIqNXw==
-----END RSA PRIVATE KEY-----

Save the key information to a file (there may be an easier way but I don't know Linux very well yet, so...)
<br>
`vim /mnt/c/Users/adhd/snoop.key`

Paste the RSA key in
Press ESC and then `:wq`

We now need to add the RSA key into Wireshark so it can help us decode the traffic:

*References:*
<br>
*<https://support.pushtechnology.com/s/article/Decrypting-Secure-PCAPs> and*
<br>
*<https://superuser.com/questions/1430350/ssl-protocol-seems-to-be-missing-in-wireshark>*

*Note that the SSL option from the first link is gone because it's been deprecated, so instead of that we need to use the TLS option in Wireshark as shown in the second link*

Open the snoop.pcap in Wireshark

In Wireshark, add the new key to the keys list:

- Edit > Preferences > Protocols > TLS
- RSA keys list > Edit...
- Add a new key with +
- IP address 0.0.0.0
- Key File C:/Users/adhd/snoop.key
- Click OK


Now all of the encrpyted traffic is highlighted in green. I chose to look for POST entries in the Info column since we knew we were looking for someone setting a password.

On line 1511, click HTML Form URL Encoded: application/x-www-form-urlencoded.
Line "wpa_sta_auth_shared_key" has the unencrypted password.

**Danc3LikeNoOnesW@tch1ngEncryp7LikeEveryoneIs**

:grin: