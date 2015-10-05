Creating keystore
-----------------

NOTE: Ensure CN in step 2 is set to the hostname of the AMQ route which is exposed below
NOTE: When prompted for passwords use password

keytool -genkey -alias broker -keyalg RSA -keystore keystore.jks
keytool -export -alias broker -keystore keystore.jks -file broker_cert
keytool -genkey -alias client -keyalg RSA -keystore client.jks
keytool -import -alias broker -keystore client.jks -file broker_cert

cp client.jks truststore.jks

oc secrets new amq-app-secret keystore.jks truststore.jks


Add keystore and truststore to secret
-------------------------------------

oc secrets new amq-app-secret keystore.jks truststore.jks

oc get secret amq-app-secret -o json >> amq-secret.json


Broker setup
-------------

oc create -f persistent-vol.json

oc create -f amq-secret.json

oc create -f amq-service-account.json

oc secrets add serviceaccount/amq-service-account secret/amq-app-secret --for=mount

oc process -f ./amq6-persistent.json  -v AMQ_ADMIN_USERNAME=admin,AMQ_ADMIN_PASSWORD=admin,AMQ_TRUSTSTORE=truststore.jks,AMQ_KEYSTORE=keystore.jks|oc create -f -



Expose the route
----------------
oc expose service broker-amq-tcp-ssl

Edit the route to add ssl termination

oc edit route broker-amq-tcp-ssl

Add under spec after host
  tls:
    termination: passthrough

Removing everything
-------------------
oc delete all -l application=broker

oc delete serviceaccount amq-service-account

oc delete secret amq-app-secret

oc delete pvc broker-amq-claim

oc delete pv pv0001
