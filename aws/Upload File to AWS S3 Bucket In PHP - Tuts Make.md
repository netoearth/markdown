PHP upload file to amazon aws s3 bucket; This tutorial will show you how to upload file/image aws s3 bucket in PHP using aws-s3 PHP SDK.

Aws PHP SDK upload file to s3 bucket tutorial will provide you complete guide on how to create aws s3 bucket account and how to upload file/image to amazon aws s3 bucket using PHP aws s3 SDK.

## How to Upload Files to Amazon AWS s3 Bucket using PHP

-   Step 1 – Create AWS S3 Bucket Account
-   Step 2 – Create PHP App
-   Step 3 – Install AWS S3 PHP SDK
-   Step 4 – Create File Upload Form
-   Step 5 – Create upload-to-s3.php file

### Step 1 – Create AWS S3 Bucket Account

First of all, you need to create aws s3 bucket account by following the below given steps:

Setup amazon aws s3 bucket account; so you need to create account on **amazon s3** to store our images/files. First, you need to sign up for Amazon.

You should follow this link to [signup.](https://aws.amazon.com/)  After successfully signing you can create your bucket. You can see the below image for better understanding.

![create bucket account on amazon s3 cloud storage](https://www.tutsmake.com/wp-content/uploads/2019/11/laravel-file-upload-amazon-s3-bucket-1024x879.png)

![create bucket account on amazon s3 cloud storage](https://www.tutsmake.com/wp-content/uploads/2019/11/laravel-file-upload-amazon-s3-bucket-1-1024x885.png)

Now You need to create a bucket policy, so you need to go to [this link. And the page looks](https://awspolicygen.s3.amazonaws.com/policygen.html) like this.

You can see the page looks like this.

![](https://www.tutsmake.com/wp-content/uploads/2019/11/Laravel-amazon-s3-file-2-1024x699.png)

You have created a bucket policy, Then copy-paste into a bucket policy. You can see the below image.

![amazon s3 bucket policy](https://www.tutsmake.com/wp-content/uploads/2019/11/laravel-amazon-s3-bucket-policy-for-laravel-3-1024x756.png)

Now you will go [here](https://console.aws.amazon.com/iam/home?region=us-east-2#/security_credential) to get our Access Key Id and Secret Access Key.

### Step 2 – Create PHP App

Visit your server directory; if you use xampp; So visit xampp/htdocs directory. And create a new directory which name file-upload-app.

### Step 3 – Install AWS S3 PHP SDK

Execute the following command on the terminal to install aws s3 PHP SDK:

```
composer require aws/aws-sdk-php
```

### Step 4 – Create File Upload Form

Create file/image upload HTML form; so visit your app root directory and create index.php file and add the following code into it:

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p></td><td><div><p><code>&lt;!</code><code>DOCTYPE</code> <code>html&gt;</code></p><p><code>&lt;</code><code>html</code> <code>lang</code><code>=</code><code>"en"</code><code>&gt;</code></p><p><code>&lt;</code><code>head</code><code>&gt;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;</code><code>meta</code> <code>charset</code><code>=</code><code>"UTF-8"</code><code>&gt;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;</code><code>title</code><code>&gt;PHP File Upload Example - Tutsmake.com&lt;/</code><code>title</code><code>&gt;</code></p><p><code>&lt;/</code><code>head</code><code>&gt;</code></p><p><code>&lt;</code><code>body</code><code>&gt;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;</code><code>form</code> <code>action</code><code>=</code><code>"upload-to-s3.php"</code> <code>method</code><code>=</code><code>"post"</code> <code>enctype</code><code>=</code><code>"multipart/form-data"</code><code>&gt;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;</code><code>h2</code><code>&gt;PHP Upload File&lt;/</code><code>h2</code><code>&gt;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;</code><code>label</code> <code>for</code><code>=</code><code>"file_name"</code><code>&gt;Filename:&lt;/</code><code>label</code><code>&gt;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;</code><code>input</code> <code>type</code><code>=</code><code>"file"</code> <code>name</code><code>=</code><code>"anyfile"</code> <code>id</code><code>=</code><code>"anyfile"</code><code>&gt;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;</code><code>input</code> <code>type</code><code>=</code><code>"submit"</code> <code>name</code><code>=</code><code>"submit"</code> <code>value</code><code>=</code><code>"Upload"</code><code>&gt;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;</code><code>p</code><code>&gt;&lt;</code><code>strong</code><code>&gt;Note:&lt;/</code><code>strong</code><code>&gt; Only .jpg, .jpeg, .gif, .png formats allowed to a max size of 5 MB.&lt;/</code><code>p</code><code>&gt;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>&lt;/</code><code>form</code><code>&gt;</code></p><p><code>&lt;/</code><code>body</code><code>&gt;</code></p><p><code>&lt;/</code><code>html</code><code>&gt;</code></p></div></td></tr></tbody></table>

### Step 5 – Create upload-to-s3.php file

Create one file name upload-to-s3.php file and add the below code into your upload-to-s3.php .php file. The following PHP code uploads the files from the webserver with validation:

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p><p>33</p><p>34</p><p>35</p><p>36</p><p>37</p><p>38</p><p>39</p><p>40</p><p>41</p><p>42</p><p>43</p><p>44</p><p>45</p><p>46</p><p>47</p><p>48</p><p>49</p><p>50</p><p>51</p><p>52</p><p>53</p><p>54</p><p>55</p><p>56</p><p>57</p><p>58</p><p>59</p><p>60</p><p>61</p></td><td><div><p><code>&lt;?php</code></p><p><code>require</code> <code>'vendor/autoload.php'</code><code>;</code></p><p><code>use</code> <code>Aws\S3\S3Client;</code></p><p><code>$s3Client</code> <code>= </code><code>new</code> <code>S3Client([</code></p><p><code>'version'</code> <code>=&gt; </code><code>'latest'</code><code>,</code></p><p><code>'region'</code>&nbsp; <code>=&gt; </code><code>'YOUR_AWS_REGION'</code><code>,</code></p><p><code>'credentials'</code> <code>=&gt; [</code></p><p><code>'key'</code>&nbsp;&nbsp;&nbsp; <code>=&gt; </code><code>'ACCESS_KEY_ID'</code><code>,</code></p><p><code>'secret'</code> <code>=&gt; </code><code>'SECRET_ACCESS_KEY'</code></p><p><code>]</code></p><p><code>]);</code></p><p><code>if</code><code>(</code><code>$_SERVER</code><code>[</code><code>"REQUEST_METHOD"</code><code>] == </code><code>"POST"</code><code>){</code></p><p><code>if</code><code>(isset(</code><code>$_FILES</code><code>[</code><code>"anyfile"</code><code>]) &amp;&amp; </code><code>$_FILES</code><code>[</code><code>"anyfile"</code><code>][</code><code>"error"</code><code>] == 0){</code></p><p><code>$allowed</code> <code>= </code><code>array</code><code>(</code><code>"jpg"</code> <code>=&gt; </code><code>"image/jpg"</code><code>, </code><code>"jpeg"</code> <code>=&gt; </code><code>"image/jpeg"</code><code>, </code><code>"gif"</code> <code>=&gt; </code><code>"image/gif"</code><code>, </code><code>"png"</code> <code>=&gt; </code><code>"image/png"</code><code>);</code></p><p><code>$filename</code> <code>= </code><code>$_FILES</code><code>[</code><code>"anyfile"</code><code>][</code><code>"name"</code><code>];</code></p><p><code>$filetype</code> <code>= </code><code>$_FILES</code><code>[</code><code>"anyfile"</code><code>][</code><code>"type"</code><code>];</code></p><p><code>$filesize</code> <code>= </code><code>$_FILES</code><code>[</code><code>"anyfile"</code><code>][</code><code>"size"</code><code>];</code></p><p><code>$ext</code> <code>= </code><code>pathinfo</code><code>(</code><code>$filename</code><code>, PATHINFO_EXTENSION);</code></p><p><code>if</code><code>(!</code><code>array_key_exists</code><code>(</code><code>$ext</code><code>, </code><code>$allowed</code><code>)) </code><code>die</code><code>(</code><code>"Error: Please select a valid file format."</code><code>);</code></p><p><code>$maxsize</code> <code>= 10 * 1024 * 1024;</code></p><p><code>if</code><code>(</code><code>$filesize</code> <code>&gt; </code><code>$maxsize</code><code>) </code><code>die</code><code>(</code><code>"Error: File size is larger than the allowed limit."</code><code>);</code></p><p><code>if</code><code>(in_array(</code><code>$filetype</code><code>, </code><code>$allowed</code><code>)){</code></p><p><code>if</code><code>(</code><code>file_exists</code><code>(</code><code>"upload/"</code> <code>. </code><code>$filename</code><code>)){</code></p><p><code>echo</code> <code>$filename</code> <code>. </code><code>" is already exists."</code><code>;</code></p><p><code>} </code><code>else</code><code>{</code></p><p><code>if</code><code>(move_uploaded_file(</code><code>$_FILES</code><code>[</code><code>"anyfile"</code><code>][</code><code>"tmp_name"</code><code>], </code><code>"upload/"</code> <code>. </code><code>$filename</code><code>)){</code></p><p><code>$bucket</code> <code>= </code><code>'YOUR_BUCKET_NAME'</code><code>;</code></p><p><code>$file_Path</code> <code>= __DIR__ . </code><code>'/upload/'</code><code>. </code><code>$filename</code><code>;</code></p><p><code>$key</code> <code>= </code><code>basename</code><code>(</code><code>$file_Path</code><code>);</code></p><p><code>try</code> <code>{</code></p><p><code>$result</code> <code>= </code><code>$s3Client</code><code>-&gt;putObject([</code></p><p><code>'Bucket'</code> <code>=&gt; </code><code>$bucket</code><code>,</code></p><p><code>'Key'</code>&nbsp;&nbsp;&nbsp; <code>=&gt; </code><code>$key</code><code>,</code></p><p><code>'Body'</code>&nbsp;&nbsp; <code>=&gt; </code><code>fopen</code><code>(</code><code>$file_Path</code><code>, </code><code>'r'</code><code>),</code></p><p><code>'ACL'</code>&nbsp;&nbsp;&nbsp; <code>=&gt; </code><code>'public-read'</code><code>,</code></p><p><code>]);</code></p><p><code>echo</code> <code>"Image uploaded successfully. Image path is: "</code><code>. </code><code>$result</code><code>-&gt;get(</code><code>'ObjectURL'</code><code>);</code></p><p><code>} </code><code>catch</code> <code>(Aws\S3\Exception\S3Exception </code><code>$e</code><code>) {</code></p><p><code>echo</code> <code>"There was an error uploading the file.\n"</code><code>;</code></p><p><code>echo</code> <code>$e</code><code>-&gt;getMessage();</code></p><p><code>}</code></p><p><code>echo</code> <code>"Your file was uploaded successfully."</code><code>;</code></p><p><code>}</code><code>else</code><code>{</code></p><p><code>echo</code> <code>"File is not uploaded"</code><code>;</code></p><p><code>}</code></p><p><code>}</code></p><p><code>} </code><code>else</code><code>{</code></p><p><code>echo</code> <code>"Error: There was a problem uploading your file. Please try again."</code><code>;</code></p><p><code>}</code></p><p><code>} </code><code>else</code><code>{</code></p><p><code>echo</code> <code>"Error: "</code> <code>. </code><code>$_FILES</code><code>[</code><code>"anyfile"</code><code>][</code><code>"error"</code><code>];</code></p><p><code>}</code></p><p><code>}</code></p><p><code>?&gt;</code></p></div></td></tr></tbody></table>

Note that:- Please add key, secret and bucket name in the above given code as shown below:

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p></td><td><div><p><code>$s3Client</code> <code>= </code><code>new</code> <code>S3Client([</code></p><p><code>'version'</code> <code>=&gt; </code><code>'latest'</code><code>,</code></p><p><code>'region'</code>&nbsp; <code>=&gt; </code><code>'YOUR_AWS_REGION'</code><code>,</code></p><p><code>'credentials'</code> <code>=&gt; [</code></p><p><code>'key'</code>&nbsp;&nbsp;&nbsp; <code>=&gt; </code><code>'ACCESS_KEY_ID'</code><code>,</code></p><p><code>'secret'</code> <code>=&gt; </code><code>'SECRET_ACCESS_KEY'</code></p><p><code>]</code></p><p><code>]);</code></p><p><code>$bucket</code> <code>= </code><code>'YOUR_BUCKET_NAME'</code><code>;</code></p></div></td></tr></tbody></table>

## Conclusion

PHP upload file to amazon aws s3 bucket; Through this tutorial, you have learned how to upload file aws s3 bucket in PHP using aws-s3 PHP SDK.

## Recommended PHP Tutorials

![](https://secure.gravatar.com/avatar/646742c2bc0bba213cdca19a447a8cc3?s=120&r=g)

My name is Devendra Dode. I am a full-stack developer, entrepreneur, and owner of Tutsmake.com. I like writing tutorials and tips that can help other developers. I share tutorials of PHP, Python, Javascript, JQuery, Laravel, Livewire, Codeigniter, Node JS, Express JS, Vue JS, Angular JS, React Js, MySQL, MongoDB, REST APIs, Windows, Xampp, Linux, Ubuntu, Amazon AWS, Composer, SEO, WordPress, SSL and Bootstrap from a starting stage. As well as demo example.

[View all posts by Admin](https://www.tutsmake.com/author/devtutsmake-com/)

## Post navigation