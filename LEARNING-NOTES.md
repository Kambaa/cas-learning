# LEARNING NOTES:

## Initial/Basic setup.

- Go to https://getcas.apereo.org/ui and download a overlay template project. You can use the
  [example settings applied url](https://getcas.apereo.org/ui?artifactId=cas-learning&casVersion=6.6.15&commandlineShellSupported=true&dependencies=webapp-tomcat&deploymentType=executable&description=WAR%20overlay%20to%20use%20as%20a%20starting%20template%20for%20Apereo%20CAS%20deployments.&dockerSupported=true&githubActionsSupported=true&groupId=tr.com.example&helmSupported=false&herokuSupported=false&javaVersion=11&language=java&name=cas-learning&nativeImageSupported=false&openRewriteSupported=true&packageName=org.apereo&packaging=war&puppeteerSupported=true&type=cas-overlay&version=1.0.0)

- Generate a keystore and put it to `/etc/cas/keystore`(on Linux) or `C:\etc\cas\keystore`(on
  Windows):

  You can do this two ways:

  **1. Automatic generation using the Gradle task defined in the project:**

  In the project, there's a gradle task that does this automatically, but the problem is, developer
  machines are protected so you need to run the command below as an _"administrator"_
  ```shell
  sudo ./gradlew createKeystore
  ```
  **2. Manually:**
  You can generate a keystore manually via the command below. Keystore password should be `changeit`
  ```shell
  keytool -genkey -keyalg RSA -alias thekeystore -keystore thekeystore -storepass changeit -validity 360 -keysize 2048
  ```
  Enter the asked fields as you see fit and put the generated keystore to `etc/cas/thekeystore`.


- Prepare configurations for project run
  to `/etc/cas/`(on Linux) or `C:\etc\cas\`(onWindows)

  Again you can do this automatically or manually. Automatic gradle operation task can be called(as
  an _"administrator"_) like:
  ```shell
  sudo ./gradlew copyCasConfiguration
  ```
  Or you can just copy the `etc/cas/config` on the project to your system's `/etc/cas/config`(On
  Linux) or `C:\etc\cas\config`(On Windows)

- Run the gradle wrapper command below to start basic cas server:

```shell
./gradlew run
```

- Open https://localhost:8443/cas/ and enter the credentials below:

```text
Username: casuser 
Password: Mellon
```

- Check that you have successfully logged in.

- Congrats! You have successfully entered the **CAS** World!
