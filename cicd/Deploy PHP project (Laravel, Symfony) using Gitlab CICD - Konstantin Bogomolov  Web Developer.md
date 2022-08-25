I wanted to do an automatic deployment every time I make a push to the repository. Here is a simple configuration for the GitLab CI/CD pipeline I used at first to build and deploy to the remote server. It builds backend and frontend. Use it for any PHP application, for example, based on Laravel or Symfony frameworks.

Using Gitlab CD you will not need to install Composer and NodeJS on your production machine, because everything will be prepared before deployment.

### [](https://bogomolov.tech/gitlab-ci-cd-symfony/#Prepare-webserver "Prepare webserver")Prepare webserver

You can use an existing user or create a new one. To create a new user, run the commands below.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>sudo adduser deployer</span><br></pre></td></tr></tbody></table>

Add web server user (e.g. www-data) to the deployer group, so you don’t have to change file permissions every time.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>sudo usermod -a -G deployer www-data</span><br></pre></td></tr></tbody></table>

Create a new directory for the project.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>sudo mkdir /var/www/your_project</span><br></pre></td></tr></tbody></table>

Then change the owner of the folder to the `deployer` user.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>sudo chown -R deployer:deployer /var/www/your_project</span><br></pre></td></tr></tbody></table>

If you use a config file, you can put it in the new directory. If not, delete `cp .env.local $CI_COMMIT_SHA/` command in the configuration below.

Create a new SSH key, if you don’t have it. To do that, log in as a `deployer` user.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>su deployer</span><br></pre></td></tr></tbody></table>

Then run command to create SSH key. Change email in the example below.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>ssh-keygen -t ed25519 -C <span>"email@example.com"</span></span><br></pre></td></tr></tbody></table>

Copy content of the public key to the `authorized_keys` file.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>cat ~/.ssh/id_rsa.pub &gt;&gt; ~/.ssh/authorized_keys</span><br></pre></td></tr></tbody></table>

Copy the private key content to use it on GitLab configuration.

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>cat ~/.ssh/id_rsa</span><br></pre></td></tr></tbody></table>

### [](https://bogomolov.tech/gitlab-ci-cd-symfony/#Setup-GitLab-repository "Setup GitLab repository")Setup GitLab repository

To enable or disable GitLab CI/CD Pipelines in your project, go to [GitLab](https://gitlab.com/), then navigate to **Settings** > **General** > **Visibility, project features, permissions** page, then expand the **Repository** section and enable the **Pipelines** toggle.

Add environment variables. Go to your project’s **Settings** > **CI/CD** and expand the **Variables** section. Click the **Add Variable** button.

Add these variables:

-   **SSH\_PRIVATE\_KEY** - copied id\_rsa from the step above
-   **SSH\_USER** - deployer
-   **SSH\_HOST** - your server IP address
-   **PROJECT\_FOLDER** - any folder name you wanted

### [](https://bogomolov.tech/gitlab-ci-cd-symfony/#Prepare-CI-CD-project-configuration "Prepare CI/CD project configuration")Prepare CI/CD project configuration

Create a .gitlab-ci.yml file in the root of the project with the content below. Modify any step if necessary.

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br></pre></td><td><pre><span><span>image:</span> <span>bogkonstantin/php-7.4-node-12-debug:latest</span></span><br><span></span><br><span><span>stages:</span></span><br><span>    <span>-</span> <span>build</span></span><br><span>    <span>-</span> <span>deploy</span></span><br><span></span><br><span><span>build:</span></span><br><span>    <span>stage:</span> <span>build</span></span><br><span>    <span>only:</span></span><br><span>        <span>-</span> <span>master</span></span><br><span>    <span>artifacts:</span></span><br><span>        <span>paths:</span></span><br><span>            <span>-</span> <span>./</span></span><br><span>    <span>script:</span></span><br><span>        <span>-</span> <span>export</span> <span>APP_ENV=prod</span></span><br><span>        <span>-</span> <span>composer</span> <span>install</span> <span>--no-dev</span> <span>--optimize-autoloader</span></span><br><span>        <span>-</span> <span>yarn</span> <span>install</span></span><br><span>        <span>-</span> <span>yarn</span> <span>encore</span> <span>production</span></span><br><span>        <span>-</span> <span>rm</span> <span>-r</span> <span>node_modules</span></span><br><span>        <span>-</span> <span>mkdir</span> <span>-p</span> <span>var</span> <span>&amp;&amp;</span> <span>chmod</span> <span>-R</span> <span>777</span> <span>var</span></span><br><span></span><br><span><span>deploy:</span></span><br><span>    <span>stage:</span> <span>deploy</span></span><br><span>    <span>only:</span></span><br><span>        <span>-</span> <span>master</span></span><br><span>    <span>before_script:</span></span><br><span>        <span>-</span> <span>'which ssh-agent || ( apt-get update -y &amp;&amp; apt-get install openssh-client -y )'</span></span><br><span>        <span>-</span> <span>eval</span> <span>$(ssh-agent</span> <span>-s)</span></span><br><span>        <span>-</span> <span>echo</span> <span>"$SSH_PRIVATE_KEY"</span> <span>|</span> <span>tr</span> <span>-d</span> <span>'\r'</span> <span>|</span> <span>ssh-add</span> <span>-</span></span><br><span>        <span>-</span> <span>mkdir</span> <span>-p</span> <span>~/.ssh</span></span><br><span>        <span>-</span> <span>chmod</span> <span>700</span> <span>~/.ssh</span></span><br><span>        <span>-</span> <span>'[[ -f /.dockerenv ]] &amp;&amp; echo -e "Host *\n\tStrictHostKeyChecking no\n\n" &gt;&gt; ~/.ssh/config'</span></span><br><span>    <span>script:</span></span><br><span>        <span>-</span> <span>zip</span> <span>-r</span> <span>$CI_COMMIT_SHA.zip</span> <span>.</span></span><br><span>        <span>-</span> <span>scp</span> <span>-p</span> <span>$CI_COMMIT_SHA.zip</span> <span>$SSH_USER@$SSH_HOST:/var/www/$PROJECT_FOLDER/</span></span><br><span>        <span>-</span> <span>ssh</span> <span>$SSH_USER@$SSH_HOST</span> <span>"cd  /var/www/$PROJECT_FOLDER/ &amp;&amp; unzip -q $CI_COMMIT_SHA.zip -d $CI_COMMIT_SHA &amp;&amp; rm $CI_COMMIT_SHA.zip &amp;&amp; cp .env.local $CI_COMMIT_SHA/ &amp;&amp; ln -sfn $CI_COMMIT_SHA current &amp;&amp; exit"</span></span><br></pre></td></tr></tbody></table>

Some explanations

-   `image` - is a Docker image, created by me. It is a Debian-based image with PHP 7.4, NodeJS 12, Composer 2, npm, and Yarn installed. See Dockerfile [here](https://hub.docker.com/r/bogkonstantin/php-7.4-node-12-debug) for details
-   `stages` - only build and deploy here. It is a good idea to add tests as a new stage
-   `build` - in the build stage we will install composer packages (without dev packages), build frontend and remove node\_modules folder to reduce the package size, since we will not use it in production. Change yarn command to npm if you use npm
-   `deploy` - add the SSH key to the running container, archive built application, upload it to the remote server, unpack it and change symlink to the new build.

That’s it. If you want to test it locally - use Gitlab Runner. Read about it [here](https://docs.gitlab.com/runner/).  
Make a commit and push it to the remote repository.  
Don’t forget to change the folder in the Nginx or Apache configuration.