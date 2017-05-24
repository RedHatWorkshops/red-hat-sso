# SSL Keystore
```
keytool -genkeypair -alias keystore -storetype jks -keyalg RSA -keysize 2048 -keypass password1 -keystore keystore.jks -storepass password1 -dname "CN=SSL-Keystore,OU=Sales,O=Systems Inc,L=Raleigh,ST=NC,C=US" -validity 730 -v
```

# JGroups Keystore
```
keytool -genkeypair -alias jgroups -storetype jceks -keyalg RSA -keysize 2048 -keypass password1 -keystore jgroups.jceks -storepass password1 -dname "CN=JGROUPS,OU=Sales,O=Systems Inc,L=Raleigh,ST=NC,C=US" -validity 730 -v
```

# Make sure the project name "myproject"
```
oc create secret generic sso-app-secret --from-file=keystore.jks --from-file=jgroups.jceks
oc create serviceaccount sso-service-account
oc policy add-role-to-user view system:serviceaccount:myproject:sso-service-account -n myproject
oc secret add sa/sso-service-account secret/sso-app-secret
```

# Login as admin to install image streams
```
sudo oc login -u system:admin
sudo oc create -n openshift -f jboss-image-streams.json
sudo oc process -f sso70-mysql.json -v HTTPS_NAME=keystore -v HTTPS_PASSWORD=password1 | sudo oc create -n myproject -f -
```
