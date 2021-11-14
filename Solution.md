# Solution:

### Glassfish:
 - Install Glassfish server and change HTTP port to 8088.

    Installing GlassFish 6.2.2. First we need to install JDK. It 6.2.2 supports JDK 11 to 17.
    
      ```console
      apt install openjdk-11-jdk
      ```

      Now we can download GlassFish from Github.

      ```console
      # Download
      wget https://github.com/eclipse-ee4j/glassfish/releases/download/6.2.2/glassfish-6.2.2.zip -O glassfish.zip

      # Extract
      unzip glassfish.zip

      # Move
      mv glassfish6 /opt/
      ```
      
      ![image](https://user-images.githubusercontent.com/23631617/141667457-a2a6693b-41f5-42be-ad7e-65fe294396fc.png)

      To change port to 8088, edit `domain.xml` at `/opt/glassfish6/glassfish/domains/domain1/config`, find `<network-listeners>` tag then edit the value of
      http-listener-1 port from `8080` to `8088`. Since we are changing HTTP port we chose http-listener-1, and to change HTTPS port we need to edit http-listener-2.
      
      ![image](https://user-images.githubusercontent.com/23631617/141667640-0288875b-49a2-4f44-80fa-d514ef3ce2cf.png)


 - Create a demo Java (11) servlet application with maven.
   
   ```console
   # To install Maven
   apt install maven
   
   # To create demo applicaion
   mvn archetype:generate -DgroupId=np.com.pranavp.app -DartifactId=assignment-maven \
   -DarchetypeArtifactId=maven-archetype-webapp -DarchetypeVersion=1.4 -DinteractiveMode=false
   ```
   
   Since we are not using any tests, we don't need to enable test plugin and we can also remove test folder.
   
   To compile the project with Java 11, we need to edit `pom.xml` and remove all the children of `<properties>` tag and replace them with
   
   ```xml
   <properties>
    <maven.compiler.release>11</maven.compiler.release>
   </properties>
   ```

 - Generate war package.

  The project that we created already has war plugin enabled. So, we can issue the following command to generate war package.
  
  ```console
  cd ~/assignment-maven
  mvn package
  ```
  
  Now the war file is generated at `~/assignment-maven/target/assignment-maven.war`.
 
 - Deploy the war using glassfish app server.

 First we need to start GlassFish server.
 
 ```console
 cd /opt/glassfish6/bin/asadmin
 
 ./asadmin start-domain
 ./asadmin deploy ~/assignment-maven/target/assignment-maven.war
 ```

 ![image](https://user-images.githubusercontent.com/23631617/141670169-556012c0-e0ab-4eaf-9657-5a3afa19b9fe.png)
 
 ![image](https://user-images.githubusercontent.com/23631617/141670205-9592bf38-6558-46d7-a325-38ad7228c098.png)

---

### Gunicorn
 - Create Django starter project in a separate virtual environment.

 ```console
 # Create virtual environment
 virtualenv ~/assignment-django -p 3
 
 # Activate virtual environment
 source ~/assignment-django/bin/activate
 
 # Install Django and gunicorn
 pip install django gunicorn
 
 # Make source directory - src
 cd ~/assignment-django/
 mkdir src
 
 # Initialize Django project under src directory
 django-admin startproject assignment src/
 ```
 
 ![Initialized Django Inside Virtualenv](https://user-images.githubusercontent.com/23631617/141670629-f8ba5e7d-df76-4251-af38-5ee43777c104.png)
 
 - Deploy the 3 instance of application using Gunicorn in 8089 port.

 During deployment, we disable `debug` mode and add our hostname to `ALLOWED_HOSTS` inside `settings.py`. Since this is just a demo project, it is not changed.
 
 ```console
 # Navigate to project directory
 cd ~/assignment-django/src
 
 # Run Gunicorn with 3 workers as a deamon
 gunicorn assignment.wsgi --name assignment-django --workers 3 --daemon --bind :8089
 ```
 
 ![image](https://user-images.githubusercontent.com/23631617/141671811-e7b0c301-1490-45dd-a95e-45603e297f57.png)
 ![image](https://user-images.githubusercontent.com/23631617/141671828-8fa88cf7-9fd4-4ed1-b3b0-bdb314a5e514.png) 


 - Dump access log in a file in non-default pattern.

 Because its not a good idea to run gunicorn as root, we need to create and change permissions of `/var/log/gunicorn.access.log`. Otherwise, while not running
 as root, gunicorn daemon crashes silently.
 
 ```console
 # Create access log
 sudo touch /var/log/gunicorn.access.log
 
 # Change owner to current user
 sudo chown gunicorn /var/log/gunicorn.access.log
 sudo chgrp gunicorn /var/log/gunicorn.access.log
 
 # Enforce hardened permission
 sudo chmod 660 /var/log/gunicorn.access.log
 
 # Start Gunicorn as daemon with our custom log format
 gunicorn assignment.wsgi --name assignment-django --workers 3 --bind :8089 \
 --access-logfile /var/log/gunicorn.access.log \
 --access-logformat '"%(h)s ---  %(t)s "%(r)s" "%(a)s"' --daemon
 ```
 
 ![image](https://user-images.githubusercontent.com/23631617/141682305-99c12700-ffc1-4c7f-bb77-336ff05088fc.png)

 
 - Dump error log in a file.

Following the same steps to secure error log file because it can potentially contain sensitive information.

```console
 # Create access log
 sudo touch /var/log/gunicorn.error.log
 
 # Change owner to current user
 sudo chown gunicorn /var/log/gunicorn.error.log
 sudo chgrp gunicorn /var/log/gunicorn.error.log
 
 # Enforce hardened permission
 sudo chmod 660 /var/log/gunicorn.error.log 
 ```
 
 ![Error log](https://user-images.githubusercontent.com/23631617/141682686-bf72bb8c-66bb-474c-b378-75c28bc00434.png)
