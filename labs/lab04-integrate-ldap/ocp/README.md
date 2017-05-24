# Integrate LDAP

You can Federate backend authentication systems into SSO to manage users centrally

## Install SSO

Follow the guide [here](https://github.com/RedHatWorkshops/red-hat-sso/blob/master/labs/lab01-setup-sso/ocp/README.md)

## Install FreeIPA  Template

Import the community version of FreeIPA template into openshift

```
oc create -f https://raw.githubusercontent.com/freeipa/freeipa-container/master/freeipa-server-openshift.json -n openshift
```


## Deploy FreeIPA

NOTE: This works with `oc cluster up` or environments with dymanic storage.

Create Service Account

```
oc project hackathon
oc create serviceaccount useroot 
oc adm policy add-scc-to-user anyuid -z useroot
```

Next, create your ipa server with the following parameters

![freeipa-parameters](images/freeipa-parameters.png)

In order for the deploymnet to kick off...edit the deployment config; and change the image to `adelton/freeipa-server:centos-7`

![freeipa-image](images/freeipa-image.png)

OPTIONAL: If you look at the deployment logs and see the following

```
Configuring Kerberos KDC (krb5kdc). Estimated time: 30 seconds
  [1/9]: adding kerberos container to the directory
  [2/9]: configuring KDC
  [3/9]: initialize kerberos container
WARNING: Your system is running out of entropy, you may experience long delays
```

It means that your system isn't creating enough random data; run this to speed it along (run ^c after a minute or two)
```
while true; do find /; done 
```

Add the router's IP address in your `/etc/hosts` file in order to access the fake domain you created

```
172.16.1.222	ipa.example.test
```

Login to the pod to find out the admin password

```
[root@ocp-aio]# oc get pods 
NAME                     READY     STATUS    RESTARTS   AGE
freeipa-server-1-dp1sv   1/1       Running   0          15m
sso-1-sp5ws              1/1       Running   0          1h
sso-mysql-1-3tbj7        1/1       Running   0          1h

[root@ocp-aio]# oc exec freeipa-server-1-dp1sv -- env | grep PASSWORD
PASSWORD=5YqaAHLmgXHjWvUlXarmFN7yunhXOIRS
```

Login with `username: admin` and `password: <the displayed password>`

```
firefox https://ipa.example.test
```

## Import LDAP users into SSO

Navigate and login into your `<ssourl>/admin/auth`; for example mine was `https://secure-sso-hackathon.apps.172.16.1.222.nip.io/admin/auth`. The login page should look like this

![sso-login](images/sso-login.png)

The default login is:

```
username: admin
password: admin
```

Navigate to `User Federation ~> Add Provider` and select `ldap`

Configure with the following Parameters

* `Edit Mode: WRITABLE`
* `Vendor: Red Hat Directory Server`
* `Username LDAP attribute: uid`
* `RDN LDAP attribute: uid`
* `UUID LDAP attribute: uid`
* `User Object Classes: person`
* `Connection URL: ldap://freeipa-server` - Change this to your servicename or service IP Address
* `Users DN: cn=users,cn=accounts,dc=example,dc=test` - If you want a specific OU, specify that OU (this gets ALL users)
* `Authentication Type: none` - Uses anonymous bind
* `Search Scope: One Level`

Consult the below picture if you get stuck

![freeipa-ssoconfig](images/freeipa-ssoconfig.png)

Now you can create a user and click on `Synchronize changed users` and you'll be able to see users on the `Users` section (left hand navigation side)

![sso-sync-users](images/sso-sync-users.png)

