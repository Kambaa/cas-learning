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

## Configs on Db:

Default behaviour is setup on the projects etc/ dir and use the gradle script to copy to /etc or C:
\etc before running the app.

First add the dependency on `build.gradle` file under `dependencies`:

```text
    implementation "org.apereo.cas:cas-server-support-configuration-cloud-jdbc"
```

And then, generate The Tables On Db(below is postgres):

```sql
create table if not exists CAS_SETTINGS_TABLE
(
    ID          serial
        constraint cas_settings_pk primary key,
    NAME        TEXT UNIQUE                       NOT NULL,
    VALUE       TEXT                              NOT NULL,
    DESCRIPTION TEXT DEFAULT ('açıklama giriniz') NOT NULL
);

```

```sql
INSERT INTO public.cas_settings_table (id, name, value, description)
VALUES (1, 'cas.tgc.crypto.encryption.key', '<SOMEKEY>',
        'Generated encryption key [] of size [256] for [Ticket-granting Cookie]. The generated key MUST be added to CAS settings');
INSERT INTO public.cas_settings_table (id, name, value, description)
VALUES (2, 'cas.tgc.crypto.signing.key', '<SOMEKEY>', e'Generated signing key [] of size [512] for [Ticket-granting Cookie]. The generated key MUST be added to CAS settings:
');
INSERT INTO public.cas_settings_table (id, name, value, description)
VALUES (3, 'cas.webflow.crypto.signing.key', '<SOMEKEY>',
        'Generated signing key [] of size [512]. The generated key MUST be added to CAS settings:');
INSERT INTO public.cas_settings_table (id, name, value, description)
VALUES (4, 'cas.webflow.crypto.encryption.key', '<SOMEKEY>',
        'Generated encryption key [] of size [16]. The generated key MUST be added to CAS settings:');
INSERT INTO public.cas_settings_table (id, name, value, description)
VALUES (5, 'cas.authn.accept.enabled', 'true',
        'CAS is configured to accept a static list of credentials for authentication. While this is generally useful for demo purposes, it is STRONGLY recommended that you DISABLE this authentication method by setting ''cas.authn.accept.enabled=false'' and switch to a mode that is more suitable for production.');
INSERT INTO public.cas_settings_table (id, name, value, description)
VALUES (6, 'spring.security.user.name', 'demo', 'Spring security secured username');
INSERT INTO public.cas_settings_table (id, name, value, description)
VALUES (7, 'spring.security.user.password', 'demo', 'Spring security secured password');
```

After successful insertion, open the file `<PROJECT_DIR>src/main/resources/application.yml` and add
these db connection information:

```yml
# Cas ayarlarının veritabanında alınmasını sağlamak adına aşağıdaki ayarları yapıyoruz.
cas:
  spring:
    cloud:
      jdbc:
        driver-class: org.postgresql.Driver
        url: jdbc:postgresql://localhost:5432/<DBNAME>
        user: <USERNAME>
        password: <PASSWORD>
```

With the inserted rows, some warnings about key generation will not come again.

## Setting up logging configuration:

Project's default behaviour is to use the `etc/cas/config/log4j2.xml` file at the project
root, if you copy this to `/etc/cas/config` or `C:/etc/cas/config`, it will override the config.

To use a config file at the project's classpath(`src/main/resources`), simply add a
configuration to
the `application.yml`:

```yaml
logging:
  config: 'classpath:config/logging/log4j2.xml'
```

you can modify default configurations according to your needs. For a basic console logging derived
from the original check out the example below:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--
All loggers are asynchronous because of log42.component.properties in cas-server-core-logging-api.
Set -Dlog4j2.contextSelector=org.apache.logging.log4j.core.selector.BasicContextSelector or override log42.component.properties to turn off async
-->
<!-- Specify the refresh internal in seconds. -->
<Configuration monitorInterval="5">
    <Appenders>
        <Console name="casConsole" target="SYSTEM_OUT">
            <PatternLayout pattern="%highlight{%d %p [%c] - &lt;%m&gt;%n}"/>
        </Console>
        <RollingFile name="casFile" fileName="casconfigserver.log" append="false"
                     filePattern="cas-%d{yyyy-MM-dd-HH}-%i.log">
            <PatternLayout pattern="%highlight{%d %p [%c] - &lt;%m&gt;%n}"/>
            <Policies>
                <OnStartupTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="10 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
        </RollingFile>
    </Appenders>

    <Loggers>
        <Logger name="tr.com.epias.cas" level="info" includeLocation="true" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.apereo.cas" level="info" includeLocation="true" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.apereo.cas.authentication.DefaultAuthenticationManager" level="error"
                includeLocation="true" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.apereo.cas.web.support.AbstractThrottledSubmissionHandlerInterceptorAdapter"
                level="error" includeLocation="true" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.apereo.cas.DefaultCentralAuthenticationService" level="error"
                includeLocation="true" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.apereo.cas.logout.DefaultLogoutManager" level="error"
                includeLocation="true" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.apache" level="info" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.springframework.cloud.server" level="info" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.springframework.cloud.config" level="info" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.springframework.cloud.bus" level="info" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.springframework.cloud.vault" level="warn" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.springframework.security" level="info" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.springframework.boot" level="info" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.springframework.boot.autoconfigure.security" level="info"
                additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.springframework.amqp" level="off" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="org.springframework.web" level="info" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Logger name="com.hazelcast" level="info" additivity="false">

            <AppenderRef ref="casConsole"/>
        </Logger>
        <Root level="error">
            <AppenderRef ref="casConsole"/>
        </Root>
    </Loggers>
</Configuration>
```
