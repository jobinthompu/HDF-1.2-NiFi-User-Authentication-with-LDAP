# NiFi  User Authentication with LDAP in HWX sandbox

## Short Description

Setting up **NiFi User Aunthentication with LDAP** in HWX Sandbox with Knox-demo-ldap on HDF-1.2 Version

## Prerequisite

1) Assuming you already have latest version of NiFi-0.6.0/HDF-1.2.0 downloaded on your HW Sandbox, else execute below after ssh connectivity to sandbox is established:

```
# cd /opt/
# wget http://public-repo-1.hortonworks.com/HDF/centos6/1.x/updates/1.2.0.0/HDF-1.2.0.0-91.tar.gz
# tar -xvf HDF-1.2.0.0-91.tar.gz
```
2) Make sure Knox is installed on your sandbox and demo LDAP is started via Ambari
![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/1.Ambari-knox.jpg)

## Steps
1) I already created Certification Authorities and client certificates at www.tinycert.org
![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/2.TinyCert.jpg)

How to create certificates as well configurations can be found in below article:

https://community.hortonworks.com/content/kbentry/886/securing-nifi-step-by-step.html

If you are too lazy to create them, try with mine :) Attached here [certificates.zip](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/cert/certificates.zip) 

- Use cert-browser.pfx to load into browser to be a NiFi administrator 'DEMO'
- Upload other two certificates to Sandbox under '/root/scripts/' and execute below commands, while executing last command enter 'hadoop' as password and 'yes' when asked if it can be trusted.

```
# cd /root/scripts/
# mv cert.pfx cert.p12
# openssl x509 -outform der -in cacert.pem -out cacert.der
# keytool -import -keystore cacert.jks -file cacert.der
```
2. My keystore is saved as ‘/root/scripts/cert.p12’ and a truststore is saved as ‘/root/scripts/cacert.jks’. and password is set as hadoop.

3. Below are the configuration updates you have to do in nifi.properties file in sandbox:

```
# vi /opt/nifi-1.1.0.0-10/conf/nifi.properties
```
4. Once opened in editor update below properties to given values [updating https port and certificate details]:

```
nifi.web.http.host=
nifi.web.http.port=
nifi.web.https.host=sandbox
nifi.web.https.port=9090
nifi.security.keystore=/root/scripts/cert.p12
nifi.security.keystoreType=PKCS12
nifi.security.keystorePasswd=hadoop
nifi.security.keyPasswd=hadoop
nifi.security.truststore=/root/scripts/cacert.jks
nifi.security.truststoreType=JKS
nifi.security.truststorePasswd=hadoop
nifi.login.identity.provider.configuration.file=./conf/login-identity-providers.xml
nifi.security.user.login.identity.provider=ldap-provider
```
![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/3.Nifi-properties.jpg)

5. Now configure the authorized users in ‘authorized-users.xml’ file, configuration of user is based on certificate. Configure it exactly as you have in your certificate. Following is my configuration:

```
vi /opt/nifi-1.1.0.0-10/conf/authorized-users.xml
```
```
<user dn="CN=Demo, OU=Demo, O=Hortonworks, L=San Jose, ST=California, C=US"> 
<role name="ROLE_ADMIN"/>
</user>
```
![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/4.authorized-users.jpg)

