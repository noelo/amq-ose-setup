Creating keystore
-----------
openssl genrsa -aes256 -out server.key 1024
openssl req -x509 -sha256 -new -key server.key -out server.csr
openssl x509 -sha256 -days 3652 -in server.csr -signkey server.key -out selfsigned.crt
openssl pkcs12 -export -name myservercert -in selfsigned.crt -inkey server.key -out keystore.p12
keytool -importkeystore -destkeystore mykeystore.jks -srckeystore keystore.p12 -srcstoretype pkcs12 -alias myservercert
keytool -list -v -keystore mykeystore.jks


Creating truststore
------------
keytool -import -file selfsigned.crt -alias firstCA -keystore mytruststore
keytool -list -v -keystore mytruststore
mv mytruststore mytruststore.jks


Base64 encoding for OSE
------------------------------
cat mykeystore.jks |base64 -w 0
cat mytruststore.jks |base64 -w 0

Copy and paste into the amq-secret.json 


Broker setup
-------------

oc create -f persistent-vol.json
oc create -f amq-secret.json
oc create -f amq-service-account.json
oc secrets add serviceaccount/amq-service-account secret/amq-app-secret --for=mount
oc process -f ./amq6-persistent.json  -v AMQ_ADMIN_USERNAME=admin,AMQ_ADMIN_PASSWORD=admin,AMQ_TRUSTSTORE=truststore.jks,AMQ_KEYSTORE=keystore.jks|oc create -f -


Removing everything
-------------------
oc delete all -l application=broker
oc delete serviceaccount amq-service-account
oc delete secret amq-app-secret
oc delete pvc broker-amq-claim
oc delete pv pv0001

