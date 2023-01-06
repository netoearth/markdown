[Skip to content](https://it.digitaino.com/use-touchid-to-authenticate-sudo-on-macos/#primary)

Your TouchID equipped Mac can easily be configured to use your fingerprint to approve `sudo` commands.

![](https://it.digitaino.com/wp-content/uploads/2022/07/sudo_touchid.png)

Use your favorite text editor and open the file

`/etc/pam.d/sudo`

and add the following line

`auth sufficient pam_tid.so`

below the `pam_smartcard.so` line as shown below and then save (Ctrl+O for `pico`) the file.

```
$ sudo pico /etc/pam.d/sudo

  UW PICO 5.09                     File: /etc/pam.d/sudo                        

# sudo: auth account password session
auth       sufficient     pam_smartcard.so
auth       sufficient     pam_tid.so
auth       required       pam_opendirectory.so
account    required       pam_permit.so
password   required       pam_deny.so
session    required       pam_permit.so



^G Get Help  ^O WriteOut  ^R Read File ^Y Prev Pg   ^K Cut Text  ^C Cur Pos   
^X Exit      ^J Justify   ^W Where is  ^V Next Pg   ^U UnCut Text^T To Spell

File Name to write : /etc/pam.d/sudo                                            
^G Get Help  ^T  To Files                                                     
^C Cancel    TAB Complete                                                     

                               [ Wrote 7 lines ]                                
^G Get Help  ^O WriteOut  ^R Read File ^Y Prev Pg   ^K Cut Text  ^C Cur Pos   
^X Exit      ^J Justify   ^W Where is  ^V Next Pg   ^U UnCut Text^T To Spell  
  
```

That’s it. Now when you open a new Terminal window you can use TouchID to approve sudo commands. If you also have your Apple Watch set to [unlock your Mac](https://support.apple.com/en-us/HT206995 "unlock your Mac"), you will also be able to approve sudo commands by double-clicking the side button on the watch.

Keep in mind that this file is somewhat protected by macOS so after each OS update you will need to add the line to the file. Other than that, it works perfectly!

## Post navigation

We use cookies on our website to give you the most relevant experience by remembering your preferences and repeat visits. By clicking “Accept All”, you consent to the use of ALL the cookies. However, you may visit "Cookie Settings" to provide a controlled consent.