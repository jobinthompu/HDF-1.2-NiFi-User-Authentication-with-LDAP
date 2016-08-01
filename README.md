# NiFi  User Authentication with LDAP in HWX sandbox

## Short Description

Setting up **NiFi User Aunthentication with LDAP** in HWX Sandbox with Knox-demo-ldap

## Prerequisite

1) Assuming you already have latest version of NiFi-0.6.0/HDF-1.2.0 downloaded on your HW Sandbox, else execute below after ssh connectivity to sandbox is established:

```
# cd /opt/
# wget http://public-repo-1.hortonworks.com/HDF/centos6/1.x/updates/1.2.0.0/HDF-1.2.0.0-91.tar.gz
# tar -xvf HDF-1.2.0.0-91.tar.gz
```
2) Make sure Knox is installed on your sandbox and demo LDAP is started via Ambari

![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/images/1.Ambari-knox.png)

## Steps
1) I already created Certification Authorities and client certificates at www.tinycert.org

![alt tag](https://github.com/jobinthompu/NiFi-User-Authentication-with-LDAP-/blob/master/images/1.Ambari-knox.png)

How to create certificates as well configurations can be found in below article:

https://community.hortonworks.com/content/kbentry/886/securing-nifi-step-by-step.html

If you are too lazy to create them, try with mine :) [Attached as certificates.zip]

- Use cert-browser.pfx to load into browser to be a NiFi administrator 'DEMO'

- Upload other two certificates to Sandbox under '/root/scripts/' and execute below commands, while executing last command enter 'hadoop' as password and 'yes' when asked if it can be trusted.

```
# cd /root/scripts/
# mv cert.pfx cert.p12
# openssl x509 -outform der -in cacert.pem -out cacert.der
# keytool -import -keystore cacert.jks -file cacert.der
```
