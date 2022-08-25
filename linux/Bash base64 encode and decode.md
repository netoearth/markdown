To encode or decode standard input/output or any file content, Linux uses base64 encoding and decoding system. Data are encoded and decoded to make the data transmission and storing process easier. Encoding and decoding are not similar to encryption and decryption. Encoded data can be easily revealed by decoding. So, this command line utility tool can’t be used for data security. Alphabet, number and ‘=’ symbol are used to encode any data.

## **Syntax:**

## **base64 \[OPTION\] \[INFILE\] \[OUTFILE\]**

You can use different types of options with base64 command. Data can be taken from any file or standard input while encoding or decoding. After encode or decode, you can send the output in a file or print the output in the terminal.

### **Options:**

**\-e or –encode**

This option is used to encode any data from standard input or from any file. It is the default option.

**\-d or –decode**

This option is used to decode any encoded data from standard input or from any file.

**\-n or –noerrcheck**

By default, base64 checks error while decoding any data. You can use –n or –noerrcheck option to ignore checking at the time of decoding.

**\-u or –help**

This option is used to get information about the usage of this command.

**\-i, –ignore-garbage**

This option is used to ignore non-alphabet character while decoding.

**–copyright**

It is used to get copyright information.

**–version**

It is used to get the version information.

How you use the base64 command in Linux is shown in this tutorial by using some examples.

### **Example#1: Encoding text data**

You can encode any text data by using base64 in the command line. When you want to encode any data using base64 then using -e or –encode option is optional. So, if you don’t mention any option with base64 then it will work for encoding. The following command will encode the data, ‘**linuxhint.com’** and print the encoded data as output.

$ echo  'linuxhint.com' | base64

**Output:**

![](https://linuxhint.com/wp-content/uploads/2018/12/1-31.png)

### **Example#2: Decoding text data**

The following command will decode the encoded text, **‘bGludXhoaW50LmNvbQ==**‘ and print the original text as output.

$ echo 'bGludXhoaW50LmNvbQo=' | base64 \--decode

**Output:**

![](https://linuxhint.com/wp-content/uploads/2018/12/2-28.png)

### **Example#3: Encoding text file**

Create a text file named, ‘**sample.txt**’ with the following text that will be encoded by using base64.

You can print the encoded text in the command line or store the encoded text into another file. The following command will encode the content of the s**ample.txt** file and print the encoded text in the terminal.

**Output:**

![](https://linuxhint.com/wp-content/uploads/2018/12/3-26.png)

The following commands will encode the content of the s**ample.txt** file and save the encoded text into the **encodedData.txt** file.

$ base64 sample.txt > encodedData.txt  
$ cat encodedData.txt

**Output:**

![](https://linuxhint.com/wp-content/uploads/2018/12/4-25.png)

### **Example#4: Decoding text file**

The following command will decode the content of the **encodedData.txt** file and print the output in the terminal

$ base64 -d encodedData.txt

**Output:**

![](https://linuxhint.com/wp-content/uploads/2018/12/5-27.png)

The following commands will decode the content of the **encodedData.txt** file and store the decoded content into the file, **originalData.txt**.

$ base64 --decode encodedData.txt > originalData.txt  
$ cat originalData.txt

**Output:**

![](https://linuxhint.com/wp-content/uploads/2018/12/6-25.png)

### **Example#5: Encoding any user-defined text**

Create a bash file named **encode\_user\_data.sh** with the following code. The following script will take any text data as input, encode the text by using base64 and print the encoded text as output.

#!/bin/bash  
echo "Enter Some text to encode"  
read text  
etext\=\`echo \-n $text | base64\`  
echo "Encoded text is : $etext"

Run the script.

$ base  encode\_user\_data.sh

**Output:**

![](https://linuxhint.com/wp-content/uploads/2018/12/7-26.png)

### **Example#6: Checking user validity by decoding text**

Create a bash file named **checkValidity.sh** and add the following code. In this example, a secret text is taken from the user.  A predefined encoded text is decoded by base64  and compared with the user input. If both values are equal then the output will be ‘**You are authenticated**’ otherwise the output will be ‘**You are not authenticated**’. Using this simple decoding code, normal validation can be done very easily.

#!/bin/bash  
echo "Type your secret code"  
read secret  
otext\=\`echo 'Nzc3Nzk5Cg==' | base64 --decode\`  
if \[ $secret == $otext \]; then  
echo "You are authenticated"  
else  
echo "You are not authenticated"  
fi

Run the script.

**Output:**

![](https://linuxhint.com/wp-content/uploads/2018/12/8-25.png)

#### **Conclusion:**

For any sensitive data like password or any confidential data, encoding and decoding system is not suitable at all. You must use encryption and decryption system for securing these type of data.

#### **References:**

-   [\[RFC\] The Base16, Base32, and Base64 Data Encodings](https://datatracker.ietf.org/doc/html/rfc3548)
-   [base64 manpage](https://linux.die.net/man/1/base64)

### About the author

![](https://linuxhint.com/wp-content/uploads/2018/11/channel-logo-150x150.jpg)

I am a trainer of web programming courses. I like to write article or tutorial on various IT topics. I have a YouTube channel where many types of tutorials based on Ubuntu, Windows, Word, Excel, WordPress, Magento, Laravel etc. are published: [Tutorials4u Help](https://www.youtube.com/channel/UCIa6d3QPq7Ulqz8Ha2WRf3A).