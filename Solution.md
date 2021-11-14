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
      wget https://github.com/eclipse-ee4j/glassfish/releases/download/6.2.2/glassfish-6.2.2.zip -o glassfish.zip

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
 - Generate war package.
 - Deploy the war using glassfish app server.

### Gunicorn
 - Create Django starter project in a separate virtual environment.
 - Deploy the 3 instance of application using Gunicorn in 8089 port.
 - Dump access log in a file in non-default pattern.
 - Dump error log in a file.