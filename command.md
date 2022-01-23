# Tryton Docker Commands
Commands to run barebone tryton system

1. Start PostgreSQL instance
```shell
$ docker run --name tryton-postgres -e POSTGRES_PASSWORD=mysecretpassword -e POSTGRES_DB=tryton -d postgres
```

2. Setup the database
```shell
$ docker run --link tryton-postgres:postgres -e DB_PASSWORD=mysecretpassword -it tryton/tryton trytond-admin -d tryton --all
```

3. Start a tryton instance
```shell
$ docker run --name tryton -p 8000:8000 --link tryton-postgres:postgres -e DB_PASSWORD=mysecretpassword -d tryton/tryton
```

4. Start tryton cron instance
```shell
$ docker run --name tryton-cron --link tryton-postgres:postgres -e DB_PASSWORD=mysecretpassword -d tryton/tryton trytond-cron -d tryton
```

5. Get help
```shell
$ docker run --rm -ti tryton/tryton --help
```



## Steps to run tryton with docker and inside tryton network
Following commands will setup a tryton bridge network, setup the conatiner in the particular bridge network, add volume for database and tryton data

1. Create a docker tryton bridge network
```shell
$ docker network create tryton
```

2 Running tryton database server
A database server may contain multiple different databases. People often confuse the term database to both mean:

- the related collection of structured data that is stored together, and
- the program that manipulates and manages access to this data.

We will going to start a database server in the below steps

   2.a. Create a docker volume 
```shell
docker volume create tryton-database
```

   2.b. Set up a secure password for your postgres instance.

    #### On Linux and MacOS
    
    ```shell
    export  POSTGRES_PASSWORD='your_top_secret_password'
    ```

    #### on Windows
    ```poweshell
    $POSTGRES_PASSWORD='your_top_secret_password'
    ```

    #### Variable interpolating in poweshell
    While the above command works well for linux machine, the POSTGRES_PASSWORD we defined in step 3, doens't get picked up properly in step 4. In powershell the call to variable needs to be changed from 

    CORRECT: Use {...} to delineate the variable name:
    ```powershell
    PS> echo "my passowd is ${POSTGRES_PASSWORD}"

    my passowd is your_top_secret_password
    ```

   2.c. Set up postgresql container
```shell
$ docker run --name tryton-postgres --env PGDATA=/var/lib/postgresql/data/pgdata --env POSTGRES_DB=tryton --env POSTGRES_PASSWORD=${POSTGRES_PASSWORD}  --network tryton --detach postgres
```

The command does the following 
* docker run: starts a new Docker container  based on the latest postgres Docker image (end of the command line)
* --name: names the container tryton-postgres, and
* --env PGDATA=/var/lib/postgresql/data/pgdata: tells PostgreSQL to store the database in /var/lib/postgresql/data/pgdata, so its data is stored in the Docker volume, and finally
* --env POSTGRES_DB: tells PostgreSQL to create a database called tryton
* --env POSTGRES_PASSWORD: it sets the username and password needed to connect to postgres (default) and your_top_secret_password
* --mount source=tryton-database,target=/var/lib/postgresql/data: inside the container it mounts the tryton-database volume at /var/lib/postgresql/data, and
* --network: attaches the container to the tryton network
* --detach: detaches your terminal’s standard input, output and error from the container so you get the command prompt back.

**Note:** To use a particular version of postgresql use following tag postgres:12 instead of simple postgres at the end of the command.

   2.d. Initializat the Database
```shell
$ docker run --env DB_HOSTNAME=tryton-postgres --env DB_PASSWORD=${POSTGRES_PASSWORD} --network tryton --interactive --tty --rm tryton/tryton trytond-admin -d tryton --all
```

* docker run: starts a new docker container (line 1) based on the latest tryton Docker image (end of the command line), and
* --env DB_HOSTNAME & --env DB_PASSWORD: Picks up the  correct database server to use with the correct login username (default - postgres) and password
* --network : attaches the container to the tryton network, and
* in the container runs the trytond-admin command against the tryton database and initializes and updates everything using the --all option 
* --tty: allocates a pseudo-TTY  and keeps the terminal’s standard input connected to the container, so you can enter in the admin user’s email address and password, and
* --rm: ensure to remove the container because it isn’t needed anymore

3. Running tryton server

Tryton server is the one which provides the business logic. Tryton’s advanced modularity allows you to select the correct business logic for your requirements by activating only the modules that you intend to use.

3.b Creating docker volume for data
The tryton server is required for storing attachments or copies of sales invoice. To ensure the data do not get deleted
 





**TODO**
- [ ] Identify how can we collect account information from email boxes.
- [ ] Use gmail contancts to build a customer database 
- [ ] Use csv upload for customer database