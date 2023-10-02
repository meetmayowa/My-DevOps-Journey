# How To Install Apache Tomcat 10 on Ubuntu 20.04
**By Mayowa Ipinyomi**

Date: 02/10/2023

### Introduction

[Apache Tomcat](https://tomcat.apache.org/) is a web server and servlet container that is used to serve [Java](https://www.oracle.com/java/) applications. It’s an open source implementation of the [Jakarta Servlet](https://jakarta.ee/specifications/servlet/), [Jakarta Server Pages](https://jakarta.ee/specifications/pages/), and other technologies of the [Jakarta EE](https://jakarta.ee/) platform.

In this tutorial, you’ll deploy Apache Tomcat 10 on Ubuntu 20.04. You will install Tomcat 10, set up users and roles, and navigate the admin user interface.

## Prerequisites

- One Ubuntu 20.04 server with a sudo non-root user and a firewall, which you can set up by following the [Ubuntu 20.04 Initial Server Setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04).



## Step 1 — Installing Tomcat

In this section, you will set up Tomcat 10 on your server. To begin, you will download its latest version and set up a separate user and appropriate permissions for it. You will also install the Java Development Kit (JDK).

For security purposes, Tomcat should run under a separate, unprivileged user. Run the following command to create a user called `tomcat`:

```
sudo useradd -m -d /opt/tomcat -U -s /bin/false tomcat
```

By supplying `/bin/false` as the user’s default shell, you ensure that it’s not possible to log in as `tomcat`.

You’ll now install the JDK. First, update the package manager cache by running:

```
sudo apt update
```

Then, install the JDK by running the following command:

```
sudo apt install default-jdk
```

Answer `y` when prompted to continue with the installation.

When the installation finishes, check the version of the available Java installation:


`java -version`

The output should be similar to this:

```
Output
openjdk version "11.0.14" 2022-01-18
OpenJDK Runtime Environment (build 11.0.14+9-Ubuntu-0ubuntu2.20.04)
OpenJDK 64-Bit Server VM (build 11.0.14+9-Ubuntu-0ubuntu2.20.04, mixed mode, sharing)

```

To install Tomcat, you’ll need the latest Core Linux build for Tomcat 10, which you can get from the [downloads page](https://tomcat.apache.org/download-10.cgi). Select the latest Core Linux build, ending in `.tar.gz`. At the time of writing, the latest version was `10.1.7`.

First, navigate to the `/tmp` directory:

```
cd /tmp
```

Download the archive using `wget` by running the following command:

```
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.13/bin/apache-tomcat-10.1.13.tar.gz
```

The `wget` command downloads resources from the Internet.

Then, extract the archive you downloaded by running:

Since you have already created a user, you can now grant `tomcat` ownership over the extracted installation by running:

```
sudo chown -R tomcat:tomcat /opt/tomcat/
sudo chmod -R u+x /opt/tomcat/bin
```

Both commands update the settings of your `tomcat` installation. To learn more about these commands and what they do, visit [Linux Permissions Basics and How to Use Umask on a VPS](https://www.digitalocean.com/community/tutorials/linux-permissions-basics-and-how-to-use-umask-on-a-vps#chmod).

In this step, you installed the JDK and Tomcat. You also created a separate user for it and set up permissions over Tomcat binaries. You will now configure credentials for accessing your Tomcat instance.


## Step 2 — Configuring Admin Users

To gain access to the **Manager** and **Host Manager** pages, you’ll define privileged users in Tomcat’s configuration. You will need to remove the IP address restrictions, which disallows all external IP addresses from accessing those pages.

Tomcat users are defined in `/opt/tomcat/conf/tomcat-users.xml`. Open the file for editing with the following command:

```
sudo nano /opt/tomcat/conf/tomcat-users.xml
```

Add the following lines before the ending tag:

/opt/tomcat/conf/tomcat-users.xml

```
<role rolename="manager-gui" />
<user username="manager" password="manager_password" roles="manager-gui" />

<role rolename="admin-gui" />
<user username="admin" password="admin_password" roles="manager-gui,admin-gui" />
```

Replace highlighted passwords with your own. When you’re done, save and close the file.

Here you define two user roles, `manager-gui` and `admin-gui`, which allow access to **Manager** and **Host Manager** pages, respectively. You also define two users, `manager` and `admin`, with relevant roles.

By default, Tomcat is configured to restrict access to the admin pages, unless the connection comes from the server itself. To access those pages with the users you just defined, you will need to edit config files for those pages.

To remove the restriction for the **Manager** page, open its config file for editing:

```
sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
```

Comment out the `Valve` definition, as shown:

opt/tomcat/webapps/manager/META-INF/context.xml

```
...
<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.Csr>
</Context>
```

Save and close the file, then repeat for **Host Manager**:

```
sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
```

You have now defined two users, `manager` and `admin`, which you will later use to access restricted parts of the management interface. You’ll now create a `systemd` service for Tomcat.

## Step 3 — Creating a `systemd` service

The `systemd` service that you will now create will keep Tomcat quietly running in the background. The `systemd` service will also restart Tomcat automatically in case of an error or failure.

Tomcat, being a Java application itself, requires the Java runtime to be present, which you installed with the JDK in step 1. Before you create the service, you need to know where Java is located. You can look that up by running the following command:

```
sudo update-java-alternatives -l
```

The output will be similar to this:

```
Output
java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
```

Note the path where Java resides, listed in the last column. You’ll need the path momentarily to define the service.

You’ll store the `tomcat` service in a file named `tomcat.service`, under `/etc/systemd/system`. Create the file for editing by running:

```
sudo nano /etc/systemd/system/tomcat.service
```

Add the following lines:

/etc/systemd/system/tomcat.service

Modify the highlighted value of `JAVA_HOME` if it differs from the one you noted previously.

Here, you define a service that will run Tomcat by executing the startup and shutdown scripts it provides. You also set a few environment variables to define its home directory (which is `/opt/tomcat` as before) and limit the amount of memory that the Java VM can allocate (in `CATALINA_OPTS`). Upon failure, the Tomcat service will restart automatically.

When you’re done, save and close the file.

Reload the `systemd` daemon so that it becomes aware of the new service:

```
sudo systemctl daemon-reload
```

You can then start the Tomcat service by typing:

```
sudo systemctl start tomcat
```

Then, look at its status to confirm that it started successfully:

```
sudo systemctl status tomcat
```

The output will look like this:
Press `q` to exit the command.

To enable Tomcat starting up with the system, run the following command:

```
sudo systemctl enable tomcat
```

In this step, you identified where Java resides and enabled `systemd` to run Tomcat in the background. You’ll now access Tomcat through your web browser.

## Step 4 — Accessing the Web Interface

Now that the Tomcat service is running, you can configure the firewall to allow connections to Tomcat. Then, you will be able to access its web interface.

Tomcat uses port `8080` to accept HTTP requests. Run the following command to allow traffic to that port:

```
sudo ufw allow 8080
```

In your browser, you can now access Tomcat by navigating to the IP address of your server:

```
http://your_server_ip:8080

```

You’ll see the default Tomcat welcome page: