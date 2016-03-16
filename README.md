# RDM (iRODS account) authentication token cache service

This web application mainly consists of two Servlets that are provided in the [Apache JCS core](https://commons.apache.org/proper/commons-jcs/).

* `org.apache.commons.jcs.auxiliary.remote.server.RemoteCacheStartupServlet`: starting up the remote cache server
* `org.apache.commons.jcs.admin.servlet.JCSAdminServlet`: cache management interface

The missing piece of configuring `JCSAdminServlet` in terms of [Velocity Templates]() is also integrated into this application. The missing template `.vm` files are extracted from the JCS source code and modified torwards the needs of RDM.  The two template files are located in the folder `src/main/webapp/WEB-INF/vm_templates`.

## Building the package

Clone the source code from GitHub.

```bash
$ git clone https://github.com/donders-research-data-management/rdm-authN-cache.git
$ cd rdm-authN-cache
```

Configure the JCS cache behavior in the file `src/main/resources/cache.ccf`, please refer to [JCS documentation](https://commons.apache.org/proper/commons-jcs/getting_started/intro.html) for the detail of the configuration parameters. 

Build the `war` file requires Maven.

```bash
$ mvn package -e -Dmaven.test.skip=true
```

Upon success, the application `war` file ready for deployment can be found as `target/rdm-authN-cache.war`.

## Testing the package

```bash
$ mvn  -Drdm.irods.username=U505173-ru.nl -Drdm.irods.userhotp=123456 test
```

where the value of `rdm.irods.userhotp` should be replaced by a proper one-time password.

## Deployment (to Tomcat)

Simply copy the application `war` into the `webapps` directory of the Tomcat.  For example,

```bash
$ sudo cp target/rdm-authN-cache.war /opt/tomcat/webapps
```

## Test JCS remote cache server

Upon the deployment, a JCS remote cache server will be started to listen on `localhost` at port `1101` (default port configured in `cache.ccf`). One could test the connnection using the `telnet` program:

```bash
$ telnet localhost 1101
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
```

If you see the same output, it implies the JCS cache server is available. Type `CTRL` + `]` to quit the telnet connection.

## Management portal

The webapp provides a simple interface to manage cached objects (i.e. the authenticated IRODSAccount) on the JCS server.  Assuming you are running the webapp on `localhost`, the URL to the interface would be: [http://localhost/rdm-authN-cache](http://localhost/rdm-authN-cache)

## Known issues

1. It seems that the JCS remote server prevents Tomcat to be shutted down properly.  After running the `shutdown.sh` script of Tomcat, the JVM process of Tomcat is not killed.  For the moment, one should kill the JVM process manually before restarting the Tomcat.

2. For some reason, the JAVA RMI library tends to resolve server host into IPV6 address. This can cause `java.net.UnknownHostException` when starting up the JCS webapp. When it happens, add to the environment variable `JAVA_OPTS` of tomcat with `-Djava.net.preferIPv4Stack=true`.

