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

Create Service Account

```
oc project hackathon
oc create serviceaccount useroot 
oc adm policy add-scc-to-user anyuid -z useroot
```
