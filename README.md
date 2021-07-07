# Wildfly Local HTTPS
How to setup local HTPPS on a Wildfly server

## References
- <https://web.dev/how-to-use-local-https>
- <https://github.com/FiloSottile/mkcert#installation>
- <https://medium.com/@hasnat.saeed/setup-ssl-https-on-jboss-wildfly-application-server-fde6288a0f40>

## Local certification
- following steps ate a summary of [this article](https://web.dev/how-to-use-local-https)
1. install [mkcert](https://github.com/FiloSottile/mkcert#installation)
2. execute the following command to generate a local certificate authority (CA)
```bash
mkcert -install # add mkcert to your local root CAs
```
3. restart your browser
4. execute the following command to create 2 .pem files: ```localhost.pem``` and ```localhost-key.pem```
```
mkcert localhost
```
> the command above also signs this certificate
5. keep both files generate in the previous step somewhere easy to find, as they'll be used [here](#configure-wildfly)

## Configure Wildfly
- following steps ate a summary of [this article](https://medium.com/@hasnat.saeed/setup-ssl-https-on-jboss-wildfly-application-server-fde6288a0f40)
1. navigate to ```<wildfly_folder>/standalone/configuration```
2. execute the following command with the two .pem files generated [here](#local-certification)
```bash
openssl pkcs12 -export -out wildfly-pkcs12.pfx -in localhost.pem -inkey localhost-key.pem
```
> this will generate the file ```wildfly-pkcs12.pfx```
3. type a password
4. make the following changes to your Wildfly's ```standalone.xml``` file
    1. add the above <security-realm> to the <security-realms> element; remenber to replace ```<your-password>``` with the password typed [here](#local-certification)
    ```bash
    <security-realm name="UndertowRealm">
      <server-identities>
         <ssl>
           <keystore path="wildfly-pkcs12.pfx" relative-to="jboss.server.config.dir" keystore-password="<your-password>" />
         </ssl>
       </server-identities>
     </security-realm>
    ```
    2. locate the configuration for the undertow subsystem and update the attribute "security-realm" of its child element ```<https-listener>``` to "UndertowRealm"
    ```bash
    <subsystem xmlns="urn:jboss:domain:undertow:8.0" default-server="default-server" default-virtual-host="default-host" default-servlet-container="default" default-security-domain="other" statistics-enabled="${wildfly.undertow.statistics-enabled:${wildfly.statistics-enabled:false}}">
      <buffer-cache name="default"/>
      <server name="default-server">
        <http-listener name="default" socket-binding="http" redirect-socket="https" enable-http2="true"/>
        <https-listener name="https" socket-binding="https" security-realm="UndertowRealm" enable-http2="true"/>
        <host name="default-host" alias="localhost">
          <location name="/" handler="welcome-content"/>
          <http-invoker security-realm="ApplicationRealm"/>
        </host>
      </server>
      <servlet-container name="default">
        <jsp-config/>
        <websockets/>
      </servlet-container>
      <handlers>
        <file name="welcome-content" path="${jboss.home.dir}/welcome-   content"/>
      </handlers>
    </subsystem>
    ```
6. start/restart your Wildfly server
7. you can access you site via HTTPS in <https://localhost:8443>
