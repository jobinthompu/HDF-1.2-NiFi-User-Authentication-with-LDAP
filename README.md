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
2) My keystore is saved as ‘/root/scripts/cert.p12’ and a truststore is saved as ‘/root/scripts/cacert.jks’. and password is set as hadoop.

3) Below are the configuration updates you have to do in nifi.properties file in sandbox:

```
# vi /opt/nifi-1.1.0.0-10/conf/nifi.properties
```
4) Once opened in editor update below properties to given values [updating https port and certificate details]:

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

5) Now configure the authorized users in ‘authorized-users.xml’ file, configuration of user is based on certificate. Configure it exactly as you have in your certificate. Following is my configuration:

```
vi /opt/nifi-1.1.0.0-10/conf/authorized-users.xml
```
```
<user dn="CN=Demo, OU=Demo, O=Hortonworks, L=San Jose, ST=California, C=US"> 
<role name="ROLE_ADMIN"/>
</user>
```
![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/4.authorized-users.jpg)

6) Above configuration is to login as NiFi Administrator, every other users can be pulled from LDAP after this administrator assigns roles on request.

7) Configure ./conf/login-identity-providers.xml as below with reference to Knox Demo LDAP Server.

![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/5.login-identity-providers.jpg)

```
<provider> 
<identifier>ldap-provider</identifier> 
<class>org.apache.nifi.ldap.LdapProvider</class>  
<property name="Authentication Strategy">SIMPLE</property>  
<property name="Manager DN">uid=admin,ou=people,dc=hadoop,dc=apache,dc=org</property>  
<property name="Manager Password">admin-password</property>  
<property name="TLS - Keystore">/root/scripts/cert.p12</property>  
<property name="TLS - Keystore Password">hadoop</property>  
<property name="TLS - Keystore Type">PKCS12</property>  
<property name="TLS - Truststore">/root/scripts/cacert.jks</property>  
<property name="TLS - Truststore Password">hadoop</property>  
<property name="TLS - Truststore Type">JKS</property>  
<property name="TLS - Client Auth"></property>  
<property name="TLS - Protocol">TLS</property>  
<property name="TLS - Shutdown Gracefully"></property>  
<property name="Referral Strategy">FOLLOW</property>  
<property name="Connect Timeout">10 secs</property>  
<property name="Read Timeout">10 secs</property>  
<property name="Url">ldap://localhost:33389</property>  
<property name="User Search Base">ou=people,dc=hadoop,dc=apache,dc=org</property>  
<property name="User Search Filter">uid={0}</property>  
<property name="Authentication Expiration">12 hours</property>  
</provider> 
```

8) Once configured, restart NiFi server.
```
# /opt/nifi-1.1.0.0-10/bin/nifi.sh restart
```
9) Now say open ‘Chrome’ browser and load client certificate associated with ADMIN user and login to secure https url of NiFi running on sandbox:

[https://sandbox:9090/nifi](https://sandbox:9090/nifi)

10) When asked, confirm for security exception and proceed. Now you are securely logged in as Demo user with admin privileges. You can now grant access to any user requesting access who are part of LDAP Directory.

11) Open another browser say ‘Safari’ to establish another session

[https://sandbox:9090/nifi](https://sandbox:9090/nifi)

It will popup below screen for login,enter credentials for accounts part of LDAP. Below are credentials part of knox demo ldap we have configured.
```
tom/tom-password
admin/admin-password
sam/sam-password
guest/guest-password
```

![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/6.Login_page.jpg)

12) Enter the password and hit login, enter justification and it will show up below screen that request is pending with Admin who already have access using certificates.

![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/7.Login_page2.jpg)

13) Now go back to chrome browser where ‘Demo’ user is NiFi Administrator and assign role to Tom

![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/8.flow_users.jpg)

![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/Resources/images/9.User-roles.jpg)


14) Go back to the old session as tom in safari, refresh the browser and you will be logged in as tom with privileges assigned by NiFi administrator. You can test if for other users as well.

## Summary
Using Knox-Demo-LDAP, Implemented User Athentication with Ldap over NiFi secured with 2way-SSL

## References:

You can refer my HCC [Article](https://community.hortonworks.com/articles/7341/nifi-user-authentication-with-ldap.html) as well.

Thanks,

Jobin George
