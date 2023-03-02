### Background

Kerbrute is a tool used to enumerate valid Active directory user accounts that use Kerberos pre-authentication. Also, this tool can be used for password attacks such as password bruteforce, username enumeration, password spray etc. This tool is being used for many years by penetration testers during internal penetration testing engagements. This tool is originally written by Ronnie Flathers (ropnop) with contributor Alex Flores.

### Table of Content

-   Introduction to Kerberos authentication
-   Download Kerbrute
-   Kerbrute help – List available features
-   Find valid users / User enumeration
-   Kerbrute Password Spray
-   Password Bruteforce
-   Bruteforce username:password combos
-   Saving Output
-   Verbose mode
-   Mitigation
-   Conclusion

### Introduction to Kerberos authentication

The Kerberos service runs on its default port which is 88 in a domain controller system. This service comes in windows and the Linux system as well where it is used to implement authentication processes more securely in an Active Directory environment. For more information about Kerberos authentication process and service principal name (SPN) please consider visiting the below link:

**[https://www.hackingarticles.in/deep-dive-into-kerberoasting-attack/](https://www.hackingarticles.in/deep-dive-into-kerberoasting-attack/)**

### Download Kerbrute

Kerbrute can be downloaded from its official github repository release page. It was last modified in December 2019. The source code of the tool is also available, and it is also available for windows systems and other Linux architecture. For simplicity, we will download compiled **kerbrute\_linux\_amd64** for the kali Linux which will be going to be an attacking system for the demonstration. The tool can be downloaded from the link given below.

Download link:

**[https://github.com/ropnop/kerbrute/releases/tag/v1.0.3](https://github.com/ropnop/kerbrute/releases/tag/v1.0.3)**

### ![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiJoZSkfpQlra-lfqgS6RUnYLAhKZircXoSmKuNNcxI4K9aOo4TEbLjDLY5IJ69qSOLPFQVgZQuCgmFD0rDj0x7EsHXuv8QBRrSXcFcSXNqmO6lLTyfD_-XYSWL_86hnW0z_Sing_YI9k3rpCXqjCJVZZ0GJa8ofEAGKhaf_J87pOIaahWvzgDnpb_naQ/s16000/1.png?w=640&ssl=1)

### Kerbrute help – List available features

Once we download the tool in the kali machine, we can list the available options and features by executing the following command:

```
./kerbrute_linux_amd64
```

In the picture below, we can see that tools can perform various tasks such as bruteforce, bruteuser, password spray, userenum and version detection. Moreover, there are some flags available too which can be very handy during penetration testing.  During the internal assessment, many times we encounter security features and the password policy so increasing and decreasing threads can help us to make password attacks stealthier. We highly recommend using all available flags come with kerbrute to get practical experience and analyse the results.

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgor0nJy66lHYOaBBmIDHhssfVxQ9IJtdE5t78k0RDouL1UCtUIe5bJzAjrghuBqnr6EHl5hCwNnGvUdkrWRS1KcCLo6COnh9jtaq8_BkSX-6a4fKQPC5Uk_IMSyO1cHzJCD1BH3Xwf1b2OJmiFAUs3X9G-H15_1ZAZEPaalz08zGDMq8Zspg9H72pQTw/s16000/2.png?w=640&ssl=1)

### Find valid users / User enumeration

During the internal penetration testing engagements especially in the Active Directory environment, our initial goal is to find valid users. Once we find potential users from the company website or any other sort of misconfiguration then we can verify those users if they have valid accounts or not using kerbrute.  To do that, we will make a list of potential users that we obtained from OSINT or any other way. For the demonstration, we have created a user’s list and saved it as users.txt.

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh2xONhKJhsjlnKrrdpe9uSkgvaqDGpgqhSnqLxJr7JgIJsmm5RqqSmvA2QfOp4WSiaTmrlHEL-Ce56Acxhc-biMRjHjh4UWQTqMZtZBjJ6_Iidc53OhWWDJjTXC5KuqZCYDn-hZ1vlNKq97QB-7b27-h2Wzi3wyhamASP1ETBlVIHbZc0YLLqQ9hxbZA/s16000/3.png?w=640&ssl=1)

Then we provided users list and selected the userenum option. Next, we provided the domain controller IP address and domain name which is **ignite.local** in our case. The tool will test against each user account and verify if those users exist in the domain and using Kerberos pre-authentication.  In the picture below we can see that kapil, aarti, shreya, raj and pawan appeared as valid users using Kerberos authentication. Here we in the position where we can think about various Kerberos attack such as SPN and Kerberos bruteforce etc. To reproduce the proof of concept, feel free to use the below command.

./kerbrute\_linux\_amd64 userenum --dc 192.168.1.19 -d ignite.local users.txt

./kerbrute\_linux\_amd64 userenum --dc 192.168.1.19 -d ignite.local users.txt

```
./kerbrute_linux_amd64 userenum --dc 192.168.1.19 -d ignite.local users.txt
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiWIeARP6j9o46YNN1siF9A_lpyfcw1RTrNp2tiVBlxgPM68QpWa1wag2eQkAuUBZ7HjWwptspLbeMaRJ7vwkygwU4sAYRv8NPXovgQMOhuoXOcnByVrSUp85uM2k_a4YZSgE-xn_q8KtXOPRL7jGrZ6ZVSsGEzkobKFLYf3X9ey0cu43ceeHLAoRlMVg/s16000/4.png?w=640&ssl=1)

### Kerbrute Password Spray

Suppose we have obtained a password (**Password@1**) during the enumeration phase that can be anything such as OSNIT leaked password, service misconfiguration, smb share, ftp etc but we do not know the real owner of the obtained password. In the username enumeration phase, we found five valid users now we can test obtained passwords with their accounts. Password spray is like password bruteforce where we test each password against single users but in the password spray, we use a single password and test it against all valid accounts.  To do that, we created a new users list and saved it as users.txt. Then we used **passwordspray** option this time and provided the domain controller IP address and domain name along with a valid users list and obtained the password. In the picture below, we can see that three users’ accounts matched the obtained password. Now we can try log in via RDP, winrm and smb service. To reproduce the proof of concept, please consider following the below command.

./kerbrute\_linux\_amd64 passwordspray --dc 192.168.1.19 -d ignite.local users.txt Password@1

./kerbrute\_linux\_amd64 passwordspray --dc 192.168.1.19 -d ignite.local users.txt Password@1

```
./kerbrute_linux_amd64 passwordspray --dc 192.168.1.19 -d ignite.local users.txt Password@1
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgPZQAXXmzmgkQq6FqSe26ml7eo8uLVW-YexOzELbOVNg7N-_Zuuq10qVrTeSxyhHhDG_bTzZygOL8ghZpDuQfM8wl91QIRJhOK0xK2NOiRi35k7wUiyRnNPOjdD7oedzcFZpOYFYwdV_qlDTAGfJPyd4fKZ3foZxGd7XY6AIJwExBBAkGSL1k1pHwYLw/s16000/5.png?w=640&ssl=1)

### Password Bruteforce

Next, we will try password bruteforce using potential passwords against a single user. In password bruteforce, we test all potential passwords against a single user. Here we are using a common password list where you can try with different password lists to get the expected result. Password mutation or custom wordlist can be fruitful whenever we come across internal penetration testing. We highly recommend visiting our article to get familiar yourself with password mutation using crunch utility by visiting the below link.

**[https://www.hackingarticles.in/a-detailed-guide-on-crunch/](https://www.hackingarticles.in/a-detailed-guide-on-crunch/)**

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhENuw1gREpOUnPZlE3bdNQq4hn3J5rmQS47CYc1zuwijzEQtMpkGqH9Z99PvkTkLHtFkww-8mSGaqAwb-d3JOEvDEHGBx1_4xuzzj-A6-B9G1ZJQRa3-Z5icMiTNhM077rdHXA9-VF71ITU3sC4o3dfKnjheY9X3wJYO0zMTHnRdWsPIrMMrGs20U4hw/s16000/6.png?w=640&ssl=1)

Firstly, we will create a potential password to perform a bruteforce attack against the domain.

We created a password list and saved it as pass.txt Then we used the **bruteuser** option this time and provided the domain controller IP address, domain name and potential password list and username ( aarti).  The tool will show **+** sign when it triggers with the valid password. If you are in a real-world engagement, then be careful about the **account lockout policy** because it may affect our client’s business. It is very common to experience this problem during penetration testing and you might need to wait for 30 minutes to one hour to perform the attack again or sometime the system administrator needs to unlock it manually. Usually, it locks out the account after **5 attempts,** but a few companies set it at **3 attempts** as well. In the picture, we can see that user aarti’s password matched one password from the password list we provided. Now, we can use valid credentials to log in via RDP, psexec and evil-winrm. To reproduce the proof of concept then follow the below command.

./kerbrute\_linux\_amd64 bruteuser --dc 192.168.1.19 -d ignite.local pass.txt aarti

./kerbrute\_linux\_amd64 bruteuser --dc 192.168.1.19 -d ignite.local pass.txt aarti

```
./kerbrute_linux_amd64 bruteuser --dc 192.168.1.19 -d ignite.local pass.txt aarti
```

**![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEghKcmSpqf_NLiTPucBXNkJ-FV0vcswpS2UeNB0N5jc3WN-OAuTexD5NWuyIma1EAdNrLpD5aTTKBe4P6lHyBxEGqOuXdIlgu7pCnWh_zg8dka4GmhmR4BhxjLNHvMlN3ZkgBZFq-zvoAkpJ171DHOjI_JlfSyBGPy3Q2eckRjC1G80khnlNtHyXFk6HA/s16000/9.png?w=640&ssl=1)**

### Bruteforce username: password combos

In this example, we will create a combined username and password list and attempt to verify if they matched. To do that, we created a username and password list and saved it as userpass.txt and attempt to verify using pipe **(|)** along with the **( – )** flag. Here we have provided userpass list, domain controller IP address and the domain name as we did in the earlier attacks. Execution of the command verified two user accounts. To reproduce the proof of concept then feel free to repeat the process with the below command.

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj7XkOO4g3hBSMd9wu3IxHzHp5p0IOrM0UmqpqpChiMteIDVUn3Tc3UgQQVUVgHY_3r2lAvXbubRPtsqYrYXn54pTluwSQQaz3x8Q-QvWNxtlIiQ3K2dkjKNcVRYmOhIn70N5nzPWIU1NR969x7RiNhU7h3th2BhwVYCxMNdJnU3u-Ov-nuoLgsCBv2tg/s16000/10.png?w=640&ssl=1)

cat userpass.txt | ./kerbrute\_linux\_amd64 --dc 192.168.1.19 -d ignite.local bruteforce -

cat userpass.txt | ./kerbrute\_linux\_amd64 --dc 192.168.1.19 -d ignite.local bruteforce -

```
cat userpass.txt | ./kerbrute_linux_amd64 --dc 192.168.1.19 -d ignite.local bruteforce -
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjvMzUTPXUWyucloFIC4yStqV2ajzotDMlPhd_HQdxFkvKQEIoDdxhw5x1d7spCeyoXp0MiaFPwityy_WP4NsLUX7aM1Sx_RZU2UxBMo_APr4GZOEKj5Jzf5TFxgh1-W_ONRjGX6UeGkJI4scs-40RIuKNiFVcjw_4PnnA--H4lTgzaLq-9ut4p-qb18w/s16000/11.png?w=640&ssl=1)

### Saving Output

Saving output is always healthy whether we are solving CTF or in real-world engagements. If we save the output, then we do not have to run the command again and again to check the results. Also, it is beneficial, especially in a real-world project where we have to provide output to our clients in the penetration testing reports. We can save the output of our findings using the **\-o flag** providing the output file name. In this example, we have saved the output as result.txt. To reproduce the proof of concept, follow the below command where we append the -o flag in the previously used command.

./kerbrute\_linux\_amd64 userenum --dc 192.168.1.19 -d ignite.local users.txt -o result.txt

./kerbrute\_linux\_amd64 userenum --dc 192.168.1.19 -d ignite.local users.txt -o result.txt

```
 ./kerbrute_linux_amd64 userenum --dc 192.168.1.19 -d ignite.local users.txt -o result.txt
```

**![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj_zmGuTvFbs_v2n_w-_xHP3HgB-J9nDOdUvSjqALHm80CYNZF5wd69rsXvFDvuHhAB5z21WfsZziBugvq3tzAaiiUNrCa-P7TL7sEMc_pdT96fhOFbp9l1Hb83QUmd3Mf6gUdISJPuwtZa3qDK8PMcJAfq__D7KWXgC26voTUfYwIFD8boUFiInZC7gw/s16000/12.png?w=640&ssl=1)**

### Verbose mode

We can also use verbose mode using the **\-v flag** in our command. Verbose features give us insight into the tool with each user account. Here in the below example, we can see that when kerbrute is unable to verify the Kerberos account, it is showing user does not exist. In this example, we are attempting to perform username enumeration by using the same command we used during the username enumeration phase by appending **\-v** flag to get a verbose result. To reproduce the proof of concept, feel free to test the below command.

./kerbrute\_linux\_amd64 userenum --dc 192.168.1.19 -d ignite.local users.txt -v

./kerbrute\_linux\_amd64 userenum --dc 192.168.1.19 -d ignite.local users.txt -v

```
./kerbrute_linux_amd64 userenum --dc 192.168.1.19 -d ignite.local users.txt -v
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgkQA3rOhKWO7zX7DrSZPtW_3RUFZPGBkRna3CWuWWgB7RbSxZXbiCflytQKkfTdcCQp_UJmhzkUIV2vO8tnssF9XUcPNkgevcYOTi9Hj8V3D10fEKEXvcSc3KFfR8Fv65Z7qEsoVrhSOovqArE5PMJZ4dTj0cX7aX79b2Q9gKJIZdteVyLnxK-ZhFKXQ/s16000/13.png?w=640&ssl=1)

### Mitigation

There are multiple factors and ways which can help to harden the system.

1.  Hacking article recommends following a strong password policy and recommends avoiding using common passwords.
2.  Hacking article recommends applying an account lockout policy to mitigate brute-force attacks.
3.  Hacking article recommends using two-factor authentication: Two-factor authentication should be used for all user accounts.
4.  Hacking article also recommends to organisations educate employees about the potential threat and attacks by providing monthly awareness programs.
5.  Hacking article also recommends conducting penetration testing assessments twice a year.

### Conclusion

We have explored the kerbrute tool briefly and its special features which can allow an attacker to gain access to the internal network. We have explored multiple techniques to exploit the internal network using the kerbrute tool where we performed password spray, password bruteforce and userenum etc. Lastly, we also provided the steps to mitigate these attacks. I hope you have learned something new today. Happy hacking!

**Author:** Subhash Paudel is a Penetration Tester and a CTF player who has a keen interest in various technologies and loves to explore more and more. Additionally, he is a technical writer at Hacking articles. Contact here**: [Linkedin](https://au.linkedin.com/in/subhash-paudel-a021ab207)** and **[Twitter](https://twitter.com/subhashpaudel94)**