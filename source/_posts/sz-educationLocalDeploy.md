---
title: Instruction for deploying sz-education locally 
date: 2021-03-10 14:36:12
tags: 
	- SpringBoot
	- SpringCloud
	- tutorial
categories: Spring Cloud
id: sz-educationLocalDeploy
password: taiyanggegege
---

轼辙云 (sz-education)  is a new generation of microservices teaching management platform based on Spring Cloud, providing multi-tenancy, permission management, exams and exercises. 

The exams support single choice, multiple choice, indefinite choice, judgement and short answer questions.

<!--more-->

This article mainly describes how to run the project locally, including downloading, importing, modifying the configuration, and running the project. 

# 1. Environmental Preparation

First install the development environment:

> - JDK: 1.8.0_202
> - MySQL: 8.0+
> - Redis
> - RabbitMQ
> - node.js
> - consul
> - zipkin
> - git
> - maven

Development tool: `IntelliJ IDEA 2020.3.1 x64`

#### 1. IntelliJ IDEA installation plugin `lombok`

Open the IntelliJ IDEA welcome page, select "Configure->Plugins" in the bottom right corner and choose Marketplace.
Or select "File > Settings > Plugins" in the IDEA project, then enter `lombok` and install.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/sz-education@master/docs/images/MyDeployNotes/lombokPlugin.png" width="100%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">lombok plugin</div>
</center>

#### 2. IntelliJ IDEA configuration maven setting

Open the IntelliJ IDEA welcome page, select "Configure->Preferences" in the bottom right corner, or select "File > Settings" in the main IDEA interface, then search for `maven` keyword. Then we override the `settings.xml` in the following page and adjust the local repository, maven home path and other related parameters as needed.

Here the `setting.xml` is saved in `docs/deploy` directory.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/sz-education@master/docs/images/MyDeployNotes/mavenSetting.png" width="100%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Maven setting</div>
</center>


#### 3. Installing `QRCode` dependencies

The project's QR code generation function is dependent on the `QRCode.jar` jar package, the jar package in the source code of the `docs/deploy` directory, you need to manually execute the command to install to the local maven repository, the specific command is as follows, note that the file directory `(-Dfile)` needs to be replaced with your actual local directory.

```shell
$  mvn install:install-file -Dfile=QRCode.jar -DgroupId=QRCode -DartifactId=QRCode -Dversion=3.0 -Dpackaging=jar
```

#### 4. Configuring the SDK

Open IntelliJ IDEA, select "Configure->Structure for New Projects" in the bottom right corner of the welcome screen, or select "File > Project Structure" on the main IDEA project page, and then select "Structure for New Projects" in the Select the correct JDK version in the SDK option of the project. As shown below.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/sz-education@master/docs/images/MyDeployNotes/JDKconfig.png" width="100%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">SDK configuring</div>
</center>


# 2. Import the project

You can go back to the welcome page, select Get from Version Control, and fill in the URL: `https://github.com/shimmerjordan/sz-education.git` in the pop-up screen, or you can choose to git locally first and then import the local project in IDEA. Either way is fine, as long as the project is successfully imported.

# 3. Modify configuration

## 1. Modify the configuration of each service as needed

The relevant configuration for each service is modified in the `config-service\src\main\resources\config` folder, such as ip, port, database name, etc.

All configurations have default values, such as

> MySQL：`localhost:3306`,  username: `root`,  password: `password`
> Redis：`localhost:6379`
> RabbitMQ：`localhost:5672`

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/sz-education@master/docs/images/MyDeployNotes/serviceConfig.png" width="100%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">service configuring</div>
</center>

## 2. Run the database initialization script

Create four new databases: `microservice-user`, `microservice-exam`,` microservice-auth`, and `microservice-gateway`. 

Execute the SQL scripts under the source directory `deploy/mysql/` respectively:

> - microservice-user.sql
> - microservice-exam.sql
> - microservice-auth.sql
> - microservice-gateway.sql

# 4. Startup environment

- Start consul: `consul agent -dev`
- Start MySQL
- Start Redis
- Start RabbitMQ
- Start zipkin: `java -jar zipkin-server.jar`

# 5. Start back-end project

## 1. Configuring JVM parameters

The purpose is to limit the memory of each service, configuration method: in IntelliJ IDEA menu, select "Run->Edit Configurations", select Spring Boot, then select the specific service, in the VM options input box, enter: `-Xms128m -Xmx128m`, as the following figure shows:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/sz-education@master/docs/images/MyDeployNotes/JVMConfig.png" width="100%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">JVM Config</div>
</center>

## 2. Start following services in sequence

- config-service
- auth-service
- user-service
- exam-service
- gateway-service

If you need the monitoring functiond then start the following services:

- monitor-service
- msc-service

## 3. Verify that the service was started successfully

Visit the consul console ([http://localhost:8500](http://localhost:8500)) and check if the health check of each service is normal.

# 6. Start front-end project

## 1. Installing front-end dependencies

Run the following command in cmd respectively In the `frontend/sz-education-ui` and `frontend/sz-education-web` directories

```shell
$ npm install
```

If the installation has taken too long, consider using a mirroring service.

It is also possible that the installation of `node-saas`or `sass-loader` fails with the following error.

> ```shell
> npm ERR! code ELIFECYCLE
> npm ERR! errno 1
> npm ERR! node-sass@3.13.1 postinstall: `node scripts/build.js`
> npm ERR! Exit status 1
> npm ERR!
> npm ERR! Failed at the node-sass@3.13.1 postinstall script.
> npm ERR! This is probably not a problem with npm. There is likely additional logging output above.
> ```

Possible solution:

- Update `sass-loader` and `node-sass` to the latest version in `package.json`
- Set the `node-sass` data source in the current directory (the project that requires `npm`): `npm config set sass_binary_site=https://npm.taobao.org/mirrors/node-sass`, then run `npm i`

## 2. `IntelliJ IDEA` Configuring 

You can refer to the back-end project to configure the startup environment with the parameters shown as following:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/sz-education@master/docs/images/MyDeployNotes/frontConfig.png" width="100%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Frontend Config</div>
</center>

If the vue-runtime-helpers error is reported as shown below, you can refer to these possible solutions:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/sz-education@master/docs/images/MyDeployNotes/vue-runtime-helpers-error.png" width="100%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">vue-runtime-helpers-error</div>
</center>

Possible solutions:

- Find a folder elsewhere and put it into the path: `sz-education\frontend\sz-education-ui\node_modules` or `sz-education\frontend\sz-education-web\node_modules` 

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/sz-education@master/docs/images/MyDeployNotes/vue-runtime-helpers-folder.png" width="30%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">vue-runtime-helpers-folder</div>
</center>

- or run `npm install --save vue-runtime-helpers` separately under the corresponding project folder.

Then run the front-end project respectively. You can visit following links after launching successfully:

|  Project  |               URL                |
| :-------: | :------------------------------: |
| Front-end | [localhost:8080](localhost:8080) |
| Back-end  | [localhost:9527](localhost:9527) |

Default account/password:

|  Type   | Account | Password |
| :-----: | :-----: | :------: |
|  Admin  |  admin  |  123456  |
| Student | student |  123456  |
| Teacher | teacher |  123456  |

# 7. Related monitoring addresses

|                 Name                 |                URL                 |
| :----------------------------------: | :--------------------------------: |
|         RabbitMQ Monitoring          | [localhost:15672](localhost:15672) |
| Spring Boot Admin Service Monitoring |  [localhost:9186](localhost:9186)  |
|         zipkin link tracking         |  [localhost:9411](localhost:9411)  |
|                consul                |  [localhost:8500](localhost:8500)  |

# 8. Appendix (Can be ignored for local deployment)

## 1. Secret Key Library

Use `keytool` to generate a `jwt token` keystore:

```shell
$ keytool -genkeypair -alias jwt -keyalg RSA -dname "CN=jwt, L=Berlin, S=Berlin, C=DE" -keypass abc123 -keystore jwt.jks -storepass abc123
```

Execute the following command, enter the key, and copy the output public key

```shell
$ keytool -list -rfc --keystore jwt.jks | openssl x509 -inform pem -pubkey
```

## 2. GC parameters

Examples of production parameters.

```shell
-Xmx512m -Xms256m -XX:+UnlockDiagnosticVMOptions -XX:+UseConcMarkSweepGC -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 -XX:AutoBoxCacheMax=20000 -XX:-UseBiasedLocking -XX:-UseCounterDecay -XX:-OmitStackTraceInFastThrow -Djava.security.egd=file:/dev/urandom -XX:+CMSParallelInitialMarkEnabled -XX:+ParallelRefProcEnabled -XX:+ExplicitGCInvokesConcurrent -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintTenuringDistribution -XX:+HeapDumpOnOutOfMemoryError -XX:+PerfDisableSharedMem -XX:+PrintCompressedOopsMode -XX:-PrintGCApplicationStoppedTime
```

## 3. Intranet penetration issues

> Invalid Host header

After the intranet penetration agent are configured, the page appears this `Invalid Host header error`. The Intranet penetration tool I am using is `frps` , and I finally found the solution on google:

In the case of `vue-cli` version **2.x**, we need to modify the `webpack.dev.conf.js` in the `devServer` object to add `disableHostCheck: true`

```javascript
devServer: {
  disableHostCheck: true,
}
```

In the case of `vue-cli` version **3.0**, we need to modify the configuration of `vue.config.js` 

```javascript
module.exports = {
  devServer: {
    disableHostCheck: true
  }
}
```

Once this is set, the page will be successfully forwarded.