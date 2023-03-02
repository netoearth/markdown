To follow along, you’ll need:

-   An Apple Mac with an Apple Silicon processor (M1, M2, etc – _not_ an Intel or AMD CPU)
-   [Azure Data Studio](https://azure.microsoft.com/en-us/products/data-studio)
-   [Docker Desktop](https://www.docker.com/) 4.16.1 or newer
-   An Internet connection

In Docker Desktop, go into Settings, Features in Development, and check the box for “Use Rosetta.” That’s the new 4.16 feature that allows you to run Intel-focused apps like Microsoft SQL Server on Apple Silicon.

[![](https://brentozarultd.wpenginepowered.com/wp-content/uploads/2023/01/use_rosetta-600x56.png)](https://brentozarultd.wpenginepowered.com/wp-content/uploads/2023/01/use_rosetta.png)

### 1\. Download & Run the SQL Server Container

We’ll follow the [instructions from Microsoft’s documentation](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker), but I’m going to abbreviate ’em here to keep ’em simple. Open Terminal and get the latest SQL Server 2022 container. You can run the below command in any folder – the file isn’t copied into your current folder.

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>sudo </span><span>docker </span><span>pull </span><span>mcr</span><span>.microsoft</span><span>.com</span><span>/</span><span>mssql</span><span>/</span><span>server</span><span>:</span><span>2022</span><span>-</span><span>latest</span></p></div></td></tr></tbody></table>

That’ll download the ~500MB container, which takes a minute or two depending on your Internet connection. Next, start the container:

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>sudo </span><span>docker </span><span>run</span><span> </span><span>-</span><span>e</span><span> </span><span>"ACCEPT_EULA=Y"</span><span> </span><span>-</span><span>e</span><span> </span><span>"MSSQL_SA_PASSWORD=&lt;YourStrong@Passw0rd&gt;"</span><span> </span><span>\</span></p><p><span>&nbsp;&nbsp; </span><span>-</span><span>p</span><span> </span><span>1433</span><span>:</span><span>1433</span><span> </span><span>--</span><span>name </span><span>sql1</span><span> </span><span>--</span><span>hostname </span><span>sql1</span><span> </span><span>\</span></p><p><span>&nbsp;&nbsp; </span><span>-</span><span>d</span><span> </span><span>\</span></p><p><span>&nbsp;&nbsp; </span><span>mcr</span><span>.microsoft</span><span>.com</span><span>/</span><span>mssql</span><span>/</span><span>server</span><span>:</span><span>2022</span><span>-</span><span>latest</span></p></div></td></tr></tbody></table>

Don’t use an exclamation point in your password – that can cause problems with the rest of the script. (Frankly, to keep things simple, I would just stick with upper & lower case letters plus numbers.)

In the above example, the container’s name will be sql1. If you decide to get fancy and change that, remember the name – you’ll need it later.

You’ll get an error about the platform – that’s okay, ignore it. Docker Desktop will show the container as running:

[![](https://brentozarultd.wpenginepowered.com/wp-content/uploads/2023/01/SQL-Server-2022-on-Macs-600x191.png)](https://brentozarultd.wpenginepowered.com/wp-content/uploads/2023/01/SQL-Server-2022-on-Macs.png)

And in less than a minute, you can connect to it from Azure Data Studio. Open ADS and start a new connection:

-   Server name: localhost
-   Port: 1433
-   Username: sa
-   Password: the one you picked above

With any luck, you’ll get a connection and be able to start running queries. To have fun, we’re going to want a sample database.

### 2\. Download & Restore the [Stack Overflow Database](https://www.brentozar.com/archive/2015/10/how-to-download-the-stack-overflow-database-via-bittorrent/)

Again, I’m going to abbreviate and change [Microsoft’s documentation](https://learn.microsoft.com/en-us/sql/linux/tutorial-restore-backup-in-sql-server-container?source=recommendations&view=sql-server-ver16) to keep things simple. Open Terminal and go into a folder where you’d like to keep the backup files. In my user folder, I have a folder called LocalOnly where I keep stuff that doesn’t need to be backed up, and I have Time Machine set to exclude that folder from my backups.

If you don’t have a folder like that, you can just go into your Downloads folder:

Download the Stack Overflow Mini database, a small ~1GB version stored on Github:

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>curl</span><span> </span><span>-</span><span>L</span><span> </span><span>-</span><span>o</span><span> </span><span>StackOverflowMini</span><span>.bak</span><span> </span><span>'https://github.com/BrentOzarULTD/Stack-Overflow-Database/releases/download/20230114/StackOverflowMini.bak'</span></p></div></td></tr></tbody></table>

Make a backups folder inside your Docker container – note that if you changed the container name from sql1 to something else in the earlier steps, you’ll need to change it here as well:

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>sudo </span><span>docker </span><span>exec</span><span> </span><span>-</span><span>it </span><span>sql1 </span><span>mkdir</span><span> </span><span>/</span><span>var</span><span>/</span><span>opt</span><span>/</span><span>mssql</span><span>/</span><span>backup</span></p></div></td></tr></tbody></table>

Copy the backup file into your container:

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>sudo </span><span>docker </span><span>cp</span><span> </span><span>StackOverflowMini</span><span>.bak</span><span> </span><span>sql1</span><span>:</span><span>/</span><span>var</span><span>/</span><span>opt</span><span>/</span><span>mssql</span><span>/</span><span>backup</span></p></div></td></tr></tbody></table>

In Azure Data Studio, restore the database:

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>RESTORE</span><span> </span><span>DATABASE</span><span> </span><span>StackOverflowMini</span><span> </span><span>FROM</span><span> </span><span>DISK</span><span>=</span><span>'/var/opt/mssql/backup/StackOverflowMini.bak'</span></p><p><span>WITH</span><span> </span><span>MOVE</span><span> </span><span>'StackOverflowMini'</span><span> </span><span>TO</span><span> </span><span>'/var/opt/mssql/data/StackOverflowMini.mdf'</span><span>,</span><span></span></p><p><span>MOVE</span><span> </span><span>'StackOverflowMini_log'</span><span> </span><span>TO</span><span> </span><span>'/var/opt/mssql/data/StackOverflowMini_log.ldf'</span><span>;</span></p></div></td></tr></tbody></table>

Presto – you now have the Stack Overflow database locally.

### 3\. Stop & Start the Docker Container

When you want to stop it, go to a Terminal prompt and type:

Because you’re exceedingly smart and almost sober, you can probably guess the matching command:

And yes, the database will still be there after you stop & start it.

### For Mac Users, This is a Godsend.

Because this just works, at least well enough to deal with development, blogging, demoing, presenting, etc. For those of us who’ve switched over to Apple Silicon processors, this is fantastic. I love that I can work on the First Responder Kit without having to fire up a Windows VM.

This isn’t for production use, obviously, and it’s not supported in any official way. In this post, I didn’t touch on security, firewalls, SQL Agent, other versions of SQL Server, performance tuning, memory management, or anything like that, nor do I intend to get involved with any of that in Docker anyway.

This particular combination of technologies (plain Docker running SQL Server for Linux on a Mac with Apple Silicon processors) is brand spankin’ new as of last week, so there isn’t anything out on the web in the way of troubleshooting. However, if you want to learn more about the two components that are probably the most new to you (Docker and SQL Server for Linux), subscribe to [Anthony Nocentino of Nocentino.com](https://www.nocentino.com/). He’s the go-to person for SQL Server on containers & Linux. He’s got [several Pluralsight courses](https://www.pluralsight.com/authors/anthony-nocentino) on these, too.