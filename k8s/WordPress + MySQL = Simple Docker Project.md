Practicing Docker can be hard, this simple project can help you practice a few Docker concepts. A very little understanding of Docker is needed, its fine even if you don't, all you need is just Docker installed and working.

## [Permalink](https://pavangudiwada.hashnode.dev/wordpress-mysql-simple-docker-project#heading-what-will-we-use "Permalink")What will we use?

-   Docker
-   WordPress
-   MySQL
-   Docker network

**Note**: This might seem complicated but it's not, just follow along and give it a try.

## [Permalink](https://pavangudiwada.hashnode.dev/wordpress-mysql-simple-docker-project#heading-steps "Permalink")Steps:

1.  We create a MySQL container and then exec into it to create a database.
2.  Then we create a WordPress container and expose internal port 80 to external 8080.
3.  WordPress cannot detect the MySQL database so we create a new network and add both the containers to the network.
4.  We add the database details to the WordPress site and done!

## [Permalink](https://pavangudiwada.hashnode.dev/wordpress-mysql-simple-docker-project#heading-step-1 "Permalink")Step 1:

First lets create the MySQL container, if you look at the MySQL docker hub page you will be told that you need a root password to create it. And thats how we know we should give it a environment variable and password.

Use `docker run --name wp-backend -e MYSQL_ROOT_PASSWORD=12345 -d mysql:latest`

-   \-e -> environment variables
-   \-d -> detached mode(runs in the background) ![DockerProject1_01_mysqlBackend.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647164819644/PDK3GQAlW.png?auto=compress,format&format=webp)

We have the container but we don't have a Database inside it. Wait for a few minutes for the container to be created, then run `docker exec -it wp-backend mysql -u root -p`

-   exec -> Execute a command inside the container
-   \-it mysql-> using interactive terminal of _MySQL_
-   \-u root -p -> The command that you want to run inside the MySQL container
    -   \-u -> username
    -   \-p -> ask for a password(MYSQL\_ROOT\_PASSWORD)

Once inside, run `create database wordpress;` to create a database for us to use.

Type`exit` to exit the container.

## [Permalink](https://pavangudiwada.hashnode.dev/wordpress-mysql-simple-docker-project#heading-step-2 "Permalink")Step 2:

Now that we have the **MySQL** backend lets create the **WordPress** frontend of our project.

We can't directly reach the WordPress container, so we open the port 80 inside the container and tell docker to map it to port 8080 of our machine.

![DockerProject1_02_wordpressFrontend.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647164839793/YeNclg1Is.png?auto=compress,format&format=webp)

Run `docker run -d --name wp-frontend -p 8080:80 wordpress`

-   \-p -> ports
-   8080:80 -> Attach 8080 of host machine to 80 of WordPress container.

If you go to localhost:8080 you should see the WordPress site. Make a few clicks and you will land on the page below.

![DockerProject1_03_wordpressUi.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647164850982/bCvBft4_E.png?auto=compress,format&format=webp)

## [Permalink](https://pavangudiwada.hashnode.dev/wordpress-mysql-simple-docker-project#heading-step-3 "Permalink")Step 3:

Our wp-frontend(WordPress container) cannot access the wp-backend(MySQL container) so we create a network and add both of them into it. It's like putting them in a same room so they can talk to each other.

Use `docker network create --attachable wp-network` to create a network

-   \--attachable -> Lets you add containers manually to this network.

Now we should addboth **wp-frontend** and **wp-backend** containers into the **wp-network** network.

```
docker network connect wp-network wp-frontend
docker network connect wp-network wp-backend
```

![DockerProject1_04_Network.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647164863199/5Wg7Izpgy.png?auto=compress,format&format=webp)

## [Permalink](https://pavangudiwada.hashnode.dev/wordpress-mysql-simple-docker-project#heading-step-4 "Permalink")Step 4:

Now go back to localhost:8080 and add the details

Database Name -> wordpress

Username -> root

Password -> the password you set(MYSQL\_ROOT\_PASSWORD)

Database Host -> wp-backend

![DockerProject1_05_DataBaseDetails.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647164885227/UZKh_Zl-v.png?auto=compress,format&format=webp)

Click **Submit**, then give in the user details and **Install**.

![DockerProject1_06_UserDetails.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647164902954/-BPzxnYUC.png?auto=compress,format&format=webp)

Then **login** with your details and you will be in your WordPress homepage.

![DockerProject1_07_WordpressHomepage.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647164915176/MLwKQBxL6.png?auto=compress,format&format=webp)

### [Permalink](https://pavangudiwada.hashnode.dev/wordpress-mysql-simple-docker-project#heading-tips "Permalink")Tips:

-   Use `docker stop containerID` to stop the container if you are unable to remove it.
    
-   Then`docker rm ContainerID` to remove the container.
    
-   You can use `docker rm ContainerID -f` to force stop and remove the container.
    

## [Permalink](https://pavangudiwada.hashnode.dev/wordpress-mysql-simple-docker-project#heading-resources-used "Permalink")Resources used:

[How to Easily Setup WordPress Using Docker](https://medium.com/@habibridho/how-to-easily-setup-wordpress-using-docker-a798081dc577)

[MySQL Docker hub](https://hub.docker.com/_/mysql)

[WordPress Docker hub](https://hub.docker.com/_/wordpress)

___

Thank you for reading, you can find me on [Twitter](https://twitter.com/pavangudiwada_), read my blogs on [Hashnode](https://pavangudiwada.hashnode.dev/) and [Medium](https://pavangudiwada.medium.com/) . Have a great time

[![Pavan Gudiwada](https://pavangudiwada.hashnode.dev/_next/image?url=https%3A%2F%2Fcdn.hashnode.com%2Fres%2Fhashnode%2Fimage%2Fupload%2Fv1644161808018%2Fy3Jyb2NWo.png%3Fw%3D256%26h%3D256%26fit%3Dcrop%26crop%3Dentropy%26auto%3Dcompress%2Cformat%26format%3Dwebp&w=640&q=75)](https://hashnode.com/@pavangudiwada)

Azure DevOps engineer always interested in learning DevOps, OpenSource and Technical Writing.

I also tweet simplified DevOps guides. Find me on Twitter and my content on Hashnode and Medium.