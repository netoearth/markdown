一、将需要持续集成的项目放到GitLab上

(1)先创建一个空白项目

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzkzL2JlZThkNDAyYWZlMzAyY2VmYzRkNjI1ZGVlNjBlODNkLnBuZw== "GitLab CI/CD For PHP")

(2)项目建完后，画面会提供如下命令，

在本地需要上传的文件夹内，执行完红框内的命令即可把本地文件上传到该项目

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzczNC9hZDE1ZTI3NTBmYzk0ZTQ4N2Y1Y2JjNzU4MzVhODRkNi5wbmc= "GitLab CI/CD For PHP")

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzcyNi85YTVhMDRhNTRjZmJhYjg5NjkzN2UwYWZmMGY4ZTc5Ni5wbmc= "GitLab CI/CD For PHP")

二、在项目中新建.gitlab-ci.yml文件，修改模板即可

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzE3NC83ODRhM2NlMDE5N2IzYzVjYjU4YTEyZjM2MWQ4Y2RjNi5wbmc= "GitLab CI/CD For PHP")

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzEwMy81NDAwZTkxMGIyYzZiZDY2NWJhYzM4MzkzZGM3ZTc2Zi5wbmc= "GitLab CI/CD For PHP")

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzE2MC9kZWUyYzUzOTRlNTAyNWE4NWQzM2FlY2UzOThjZjUxOC5wbmc= "GitLab CI/CD For PHP")

三、开启共享的runner（比如公司内部提供的共享runner）

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzg2MC8zYTczMzQ5YzQ4Mzc1Nzc4MjZjNjg1NTg1NGJjN2E3Yy5wbmc= "GitLab CI/CD For PHP")

四、运行pipeline

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzM1MC9lMDBlYmVhNmMyZDBmOTFkZTdmYzU1Nzc2MTFjOTdjNi5wbmc= "GitLab CI/CD For PHP")

五、查看执行结果

第一个Job>build执行成功：

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzYyMS8zOTAxMTM2NGNmMjM2M2VmZTk0MjIwMTg2NDU0MGI5NS5wbmc= "GitLab CI/CD For PHP")

整个pipelines执行结果：

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzEzMS9hNDY5ZTk3ZWRhNmJhZTFiNGUyYWMyNGVhNDJhOGYwYi5wbmc= "GitLab CI/CD For PHP")

![GitLab CI/CD For PHP](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzMzMi8xYTRhYzExMGY5Y2FiNWJlYzg1NTllMTNlZjA4NGFlNC5wbmc= "GitLab CI/CD For PHP")

六、参考文档

(1)[Gitlab CI yaml官方配置文件翻译](https://segmentfault.com/a/1190000010442764#articleHeader5)

(2)[Getting started with GitLab CI/CD](https://docs.gitlab.com/ee/ci/quick_start/README.html)

原文链接：来源网络，如有侵犯到您的权益请联系zengyin969@gmail.com进行下架处理