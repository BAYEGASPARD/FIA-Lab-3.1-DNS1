# FIA Lab 3.1 – DNS1

## Infracstucture

>For this lab and DNSSEC that will follow, half of you will use BIND and half will use Unbound+NSD. If you have an even table number, you should use BIND, otherwise Unbound+NSD. We will students PC for installing DNS servers, we got os3.su domain, sub-domains with names < stdX >.os3.su will be delegated for your nameservers, ns.os3.su will point to your assigned public IP.

### Task 1 - Downloading and Installing a Caching Nameserver
>The sources for the latest version of BIND can be downloaded from the website http://www.isc.org. Latest Unbound can be downloaded from http://unbound.net. We will install NSD later.

### 1.1 - Validating the Download
>The http://www.isc.org website provides signature files in addition to the BIND tarball. These can be used to check if you have downloaded the version they intended to distribute. The Unbound website uses a different mechanism, in particular, a modification detection code, better known as a cryptographic hash.

#### Why is it wise to use a signature to check your download?
- We use signatures to verify that the file we downloaded is exactly the file that the website or company want us to use.
- This is done by first hashing the file(either using SHA-1 or MD5 checksums), to create a unique finger print.This hash is unique for this file and if 1 bit of data is altered from the content of the file, like for example a malicious person wanting to put a virus or malware in the file, the hashes will change and will not correspond to the original hash anymore.
- Hashing is mostly to provide integrity of the file.
- But unfortunately, hashing is vulnerable.How? The answer is the malacious person can hijack the website and change the hash to be exact to his hash with the virus in the file.Or a MITM can be done and the attacker adds his hash on the fly.
- To remedy this, we use signatures.The legetimate person will sign the hash from the original source with his private key.
- Then we can verify the signature with its public key.Thus providing integrity and validity or authenticity of the file.(see the diagram below for more details)
![](https://i.imgur.com/yVwucqT.png)
(source: https://proprivacy.com/guides/digital-signature)
#### Download the BIND tarball (also if you are doing the Unbound+NSD part) and check its validity using one of the signatures.

- We move to the download page of bind (https://www.isc.org/download/)and download the tarball with its signature.
![](https://i.imgur.com/t0hXSEK.png)
- Since isc.org is using the open source PGP keys as compared to the proprietry microsoft PKI, we need to use the GNU Privacy Guard(GPG) to verify the signatures.We need to install this package first.The download link : https://gnupg.org/.
or you use <code>apt-get install gnupg</code>
![](https://i.imgur.com/953C2tB.png)
- we import the key using the command <code>pgp --import keyfile.txt</code> the key provided from the official website as seen below.
![](https://i.imgur.com/sN0VB9h.png)
- After importing we will recieve this message.
![](https://i.imgur.com/XV5R8Ru.png)
- We perform verification using the command <code>gpg --verify sifile.asc tarballfile.tar.gz</code> as seen in the diagram below.
![](https://i.imgur.com/biswmNT.png)
- For unbound , validation is just by comparing the hashes.That's all, we generate the hash from the file and compare it with what is published on the unbound website.It is not the best option as from the explanations above, it is just verifying integrity and may be vulnerable if the website is compromised or MITM attack is performed.
- We use the command  <code>sha512sum tarbal_file</code>
![](https://i.imgur.com/JmX7I8D.png)

- We can also find the equivalent hash code from the website as seen in the picture below.
![](https://i.imgur.com/Ct1sFxX.png)
- The two are identical .This means we verified the integrity of the file.
(source: https://tutorials.ubuntu.com/tutorial/tutorial-how-to-verify-ubuntu#0)

### Which kind of signature is the best one to use? Why?
- Considering the different signatures below from the official site of isc.org, The highlighted one is the best(SHA-512).
- Reason is for xample is SHA1 has been said to be vulnerable to collision attacks which is why all browsers will be removing support for certificates signed with SHA1 by January 2017. SHA256 or SHA512 however, is currently much more resistant to collision attacks as it is able to generate a longer hash which is harder to break.
- A collision attack is when 2 different files produces the same hash.Thus the two files can substitude each other creating a security breach.
(https://www.keycdn.com/support/sha1-vs-sha256)
![](https://i.imgur.com/BJsgI9j.png)

### 1.2 - Documentation & Compiling
- I will be installing Unbound + NSD.
- Defining an unbound+NSD, Unbound is a DNS server mostly having features as validating, recursive, and caching. 
- It is developed as a modular system which has as feature to enhance DNSSEC validation, IPV6, and a client resolver library API as an integral part of the architecture.
(source: http://www.linuxfromscratch.org/blfs/view/8.1/server/unbound.html)
- Since I have never installed a DNS server on my computer before , I have no need worrying previous installations that can mess my new installations but if in case I am using a server initially maintained by a different person, I have to check.
- I can use the command <code>systemctl unbound status</code> which tells the service could not be found.
![](https://i.imgur.com/jxvBHxf.png)
- Also, we can also use the command <code>systemctl | grep unbound</code> which display no service related to unbound.Thus we have nothing to worry about related to previous installations.
![](https://i.imgur.com/nCaoy7B.png)
- There are two ways to install unbound dns resolver software.
- Either we install from source tarball or using package manager.
- If we are installing from package manager, the package manager will go to the ubuntu repository to download all required dependencies automatically , we use the command <code>apt-cache depends unbound</code> as seen in the picture below.
- We also have to install openssl for DNSSEC.
![](https://i.imgur.com/FHvXAIE.png)
- But if we are installing from the unbound source tarball, we have to add packages that will be indicated during the configuration using the ./donfig command.
(source: https://nlnetlabs.nl/documentation/unbound/howto-setup/)
- So, installing from the unbound source tarball, we first download the file , form the official website, check validity as seen above and decompress using the command <code>tar xvzf filename.tar.gz</code>
![](https://i.imgur.com/2iEEuzv.png)
- Before compiling, we first need to cresate a dedicated user and group to take control of the unbound after it is started.
- To do this, we use the following comands 
<code>
./configure \
</code>
<code>
--sysconfdir=/usr/local/etc/unbound \           
</code>
<code>
--disable-static  \           
</code>
<code>
--with-pidfile=/var/run/named.pid &&           
</code>
<code>
make
</code>
- Explaining the commands, <code>--sysconfdir=/usr/local/etc/unbound</code> places the config file in the specified directory while  <code>--disable-static</code>, is used to prevent installation of static libraries and <code>--with-pidfile=/var/run/named.pid</code> precise the location of pid file.
![](https://i.imgur.com/9Hx5QGM.png)
- After this , we compile with the command <code>make</code>
- Then <code>ma</code>




















<style>
code.blue {
  color: #337AB7 !important;
}
code.orange {
  color: #F7A004 !important;
}
</style>



### Thank you! :sheep: 

You can find me on

- GitHub
- Twitter
- or email me
