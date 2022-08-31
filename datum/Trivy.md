This article talks about Trivy, which is a simple and comprehensive vulnerability scanner for containers and other artifacts, suitable for Continuous Integration and Testing.

### Table of Contents

-   Introduction
-   Installation
-   Scanning Git Repository
-   Scanning Container Image
-   Scanning Filesystem
-   Scanning the running Containers
-   Embed Trivy in Dockerfile

### Introduction

Trivy is an open-source tool by **[aqua security](https://aquasecurity.github.io/trivy/dev/)** to scan for vulnerabilities and misconfiguration errors. This tool works at various levels: it can evaluate Infrastructure as Code, inspect container images, deliver configuration file assistance, analyze Kubernetes implementations, and review the code in a Git repository. With the ease of usage, trivy can be simply be integrated in CI/CD pipeline (DevSecOps) by installing and adding binary to the project. Trivy offers complete visibility across programming language and operating system packages and has a wide database of vulnerabilities which allows quick scans of critical CVEs. With various new advancements in the tool, it has helped pen-testers and cybersecurity researchers to ensure continuous scans making the process of DevSecOps faster and more efficient.

### Installation

The installation is quite simple. Follow the below-given commands to install Trivy from the official repository on your ubuntu machine.

sudo apt-get install wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb\_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt-get install wget apt-transport-https gnupg lsb-release wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add - echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb\_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list

```
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjJJkmu8GHT6Sc3frJXUPfeVBigSPD-zukZSLUYdIXtmsp-zS_qRN9PUYwLS5xhKbtbM6QGrdfoFcbL8wNEx0_qdMmy3JSpIa88KRSUUXWJxJaF7wDPx_p6BmjZ26_FZEP4ngq3BaTxd-NNEjOXb2AqcaykIWro_LJUMAwYnJsXcIPOc-UZ2bA4e4d8wA/s16000/1.png?w=640&ssl=1)

sudo apt-get install trivy

sudo apt-get update sudo apt-get install trivy

```
sudo apt-get update
sudo apt-get install trivy
```

Once the tool has been installed and updated, you are ready to scan files.

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjzrw0eCdhpJKAmzs63C5DUzL1uSrbgI-G9UF-b-4b_SbzsXXsPEPxQNWvY_QDc_AVUqfkz6W8RWjnTtEiUXY-dAF_jR_eRrAn8-NMdDaxRpEpRyxT8b0wz7PGbt1kaxEa_-0U1uyIsBJiz_jpNxvwrb5uBWQe_cqCshH0chJofNEXnqzhp03oMRrRz9g/s16000/2.png?w=640&ssl=1)

### Scanning Git Repository

 As I have described above, we can use trivy for scanning security loopholes among multiple platforms.

 If you are using Git Repository and you can scan git file directly without downloading the entire package.

sudo trivy repo https://github.com/appsecco/dvna

sudo trivy repo https://github.com/appsecco/dvna

```
sudo trivy repo https://github.com/appsecco/dvna
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEibXzPYjbgZM0yzwwlDT2gbU1y46lklKuVSoSJ-OQPNhYKg20n0xa31g5eHFZ6oKmINtWHMyMOCmDxpfkUixrOtKv-X3byU-Q92Ec-zY1XFZ_zyoyGoyw6NP2ct4Xwm4GNfoIMXz6DwTL-6MsxIerV2wWib0QVskBWbmfKuXuSXzrHTSorHh1CEVfxhRg/s16000/3.png?w=640&ssl=1)

### Scanning Container Image

With the ever-growing threats to docker security, Trivy is one of the best tools available in the market for scanning Container Images. 

You can easily run a quick scan on the docker images to report any vulnerabilities by following the below-given steps.

Step1: Check the Image ID of the Container image you want to scan.

```
sudo docker images
```

Step2: Use the below-given command to scan the container image.

sudo trivy image 4621d4fe2959

sudo trivy image 4621d4fe2959

```
sudo trivy image 4621d4fe2959
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiCRH0VJ3sf6qBDs6EadNv0tZe_SfLEtDU0rdYlZXh-rnrO1KNGqSIDLO9LWWTcmvFSFUasmf4TOmb0tqdGFwYW4Z0sXQmm_sEzgqn-XpNJVB3FG478rjsa6iRtHqAiTmI17UxbNkEMPJuJvCxcB4OGJd3Gi3BYYbKTsDBjW6OcPnsJYxBwNA0-qwRNQw/s16000/4.png?w=640&ssl=1)

You can also scan the images for a particular severity of vulnerabilities and save the report in text format using the below-given command.

sudo trivy image --severity HIGH 4621d4fe2959 \> result.txt

sudo trivy image --severity HIGH 4621d4fe2959 > result.txt tail result.txt

```
sudo trivy image --severity HIGH 4621d4fe2959 > result.txt
tail result.txt
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjL1lofqNvke1zc6tUuf6RG6RY6HSMFEz3FeeZSWJ9lsXzeQwFu9AxiSD7ZZpEq-OJ6X2Nu417L_iBkMhqP_jJmY5vIRGJVR13_cEokB6ErVe1eAJ4HfGpQDeTDDGGHvOexSo8LmBtOJHJdW8yFVC9U5_HHTbMttApIgwUr00PfHz9OKOzV9qhEgWJKVw/s16000/5.png?w=640&ssl=1)

### Scanning Filesystem

Trivy can be used to scan a filesystem (such as a host machine, a virtual machine image, or an unpacked container image filesystem).

(Note: We are using [vulnerable-node](https://github.com/cr0hn/vulnerable-node) from Filesystem for this practical.)

Use the below-given command to scan any filesystem for vulnerabilities.

```
trivy conf services/
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEglKpRjy3nKSAKVj4RmiyB4IP459DTh09_DMu6rFao3kOpTGQzsMOT2cxt5AXa5PsxbveSnxuo4GuT48SSHWemARoHET8xl28XWiSdpQDrQgFOvHYsgy4k6K0aOKkng9w42elgWHQDhFbDkt6gd7RNJ5ckr3NRSaMEInOdCVWi5P_bChIx9JdnY56lmMQ/s16000/6.png?w=640&ssl=1)

### Scanning the running Containers

You can quickly scan the running container from inside. Follow the below-given steps to scan a docker file.

Step1: Run the docker file that you want to scan.

sudo docker run -it alpine

sudo docker run -it alpine

```
sudo docker run -it alpine
```

Step2: Add Trivy scanner to the file and run it.

&& curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin \\ && trivy filesystem --exit-code 1 --no-progress /

apk add curl \\ && curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin \\ && trivy filesystem --exit-code 1 --no-progress /

```
apk add curl \
&& curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin \    && trivy filesystem --exit-code 1 --no-progress /
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh7ZlI5RltN-TQxOoQs_cDHRUjwGNf7RRiU7ZTARriPW1Ssv_d0jTCl7sc7xRprm_9HV2_ddsDzratEfm9PCTnNeThs3DkRodaxocewZBWls-iR8wxvmsgiumuokSZgXIN_5mcsZ9P7n1kbK1uJC5a_v4QxCJJ8HaGLS9zG3MC7GkIL8iMdStVrCmxeVQ/s16000/7.png?w=640&ssl=1)

### Embed Trivy in Dockerfile

You can also scan the image as part of the build process by embedding Trivy in the Dockerfile. This approach can be used to update Dockerfiles currently using Aqua’s Micro scanner. Follow the below-given steps to scan the docker file while building it.

Step1: Add trivy to the docker file.

&& curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/master/contrib/install.sh | sh -s -- -b /usr/local/bin \\

&& trivy filesystem --exit-code 1 --no-progress /

FROM alpine:3.7 RUN apk add curl \\ && curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/master/contrib/install.sh | sh -s -- -b /usr/local/bin \\ && trivy filesystem --exit-code 1 --no-progress /

```
FROM alpine:3.7
 RUN apk add curl \
    && curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/master/contrib/install.sh | sh -s -- -b /usr/local/bin \
    && trivy filesystem --exit-code 1 --no-progress /
```

 Step2 : Build the image.

sudo docker build -t vulnerable image .

sudo docker build -t vulnerable image .

```
sudo docker build -t vulnerable image .
```

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhBWSgJKezqdBEDhXlPlfzX_MNnPvm17-IR1nZ7iv1ywcU9Ut73bIr3vyESN_3zqaBzmPFrC6786oDjyEZeOQdnczNZ2Z1meptDBkFf2TiCSaTvO9EEbupcg6I3NoNmBBI628VHmmWDAtFLfR2u5X3vI6r7XJwopIZryOH3I1B8WLCVik1MIhECwgKYMw/s16000/8.png?w=640&ssl=1)

It will scan the docker file while the image is being built and give the report as shown below.

![](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjSe8EsDwrbg88ex0nty9RfdKaZkemAcg7zRMIcSMUkls8ITjwqeugATPz3Gix-AeDw8lpqMI9yDUOo3L8HIc2o2HjH3mpyhnw2ge_MlgIMCQ_tyvSY-Hz3mG-4DgpMOArxBngY60S0d8oP76lNzo3uPVny3xyvA9WhuX8qGUyrjHjuD2WzQ6I7-bAMrw/s16000/9.png?w=640&ssl=1)

Thanks for reading the article.

**Author**: Mukund Mehrotra is a cybersecurity researcher, technical writer and an enthusiastic pen-tester at Hacking Articles. Contact **[here](https://www.linkedin.com/in/mukund-mehrotra-513aa9165/)**. 

## Post navigation