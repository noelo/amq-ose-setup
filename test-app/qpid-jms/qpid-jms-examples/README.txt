===========================
Running the client examples
===========================

Use maven to build the module, and additionally copy the dependencies
alongside their output:

  mvn clean package dependency:copy-dependencies -DincludeScope=runtime -DskipTests

Now you can run the examples using commands of the format:

  Linux:   java -cp "target/classes/:target/dependency/*" org.apache.qpid.jms.example.HelloWorld

  Windows: java -cp "target\classes\;target\dependency\*" org.apache.qpid.jms.example.HelloWorld

NOTE: The examples expect to use a Queue named "queue". You may need to create
this before running the examples, depending on the broker/peer you are using.

NOTE: You can configure the connection and queue details used by updating the
JNDI configuration file before building. It can be found at:
src/main/resources/jndi.properties

NOTE: The earlier build command will cause Maven to resolve the client artifact
dependencies against its local and remote repositories. If you wish to use a
locally-built client, ensure to "mvn install" it in your local repo first.

java  -cp "target/classes/:target/dependency/*" -Djavax.net.ssl.trustStore=client.jks -Djavax.net.ssl.trustStorePassword=password org.apache.qpid.jms.example.Sender
java  -cp "target/classes/:target/dependency/*" -Djavax.net.ssl.trustStore=client.jks -Djavax.net.ssl.trustStorePassword=password org.apache.qpid.jms.example.Receiver

