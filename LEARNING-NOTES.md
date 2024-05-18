# LEARNING NOTES:

## Initial/Basic setup.



- For the first initial project openning, generate a keystore via the command below. Keystore
  password should be `changeit`

```shell
keytool -genkey -keyalg RSA -alias thekeystore -keystore thekeystore -storepass changeit -validity 360 -keysize 2048
```

- put the generated keystore to `etc/cas/thekeystore`.

- Add new file named `cas.properties` under `etc/cas/config`

```properties
todo:add basic cas.properties content.
```

- Run the gradle wrapper command below to start basic cas server:

```shell
./gradlew clean copyCasConfiguration build run
```

- Open https://localhost:8443/cas/ and enter the credentials below:

```text
Username: casuser 
Password: Mellon
```

- Check that you have successfully logged in.

- Congrats! You have successfully entered the **CAS** World!
