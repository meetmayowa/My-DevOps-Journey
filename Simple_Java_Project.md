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