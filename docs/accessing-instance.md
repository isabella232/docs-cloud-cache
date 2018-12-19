---
title: Accessing a Service Instance
---

After you have created a service instance, you can start accessing it.
Usually, you set up cache regions before using your service instance from a deployed CF app.
You can do this with the gfsh command line tool.
To connect, you must set up a service key.

## <a id="create-service-key"></a> Create Service Keys

Service keys provide a way to access your service instance outside the scope of a deployed CF app. Run `cf create-service-key SERVICE-INSTANCE-NAME KEY-NAME` to create a service key. Replace `SERVICE-INSTANCE-NAME` with the name you chose for your service instance. Replace `KEY-NAME` with a name of your choice. You can use this name to refer to your service key with other commands.

<pre class='terminal'>
$ cf create-service-key my-cloudcache my-service-key
</pre>

Run `cf service-key SERVICE-INSTANCE-NAME KEY-NAME` to view the newly created service key.

<pre class='terminal'>
$ cf service-key my-cloudcache my-service-key
</pre>

The `cf service-key` returns output in the following format:

<pre class='terminal'>
{
 "distributed_system_id": "0",
 "locators": [
  "10.244.0.66[55221]",
  "10.244.0.4[55221]",
  "10.244.0.3[55221]"
 ],
 "urls": {
  "gfsh": "https://cloudcache-1.example.com/gemfire/v1",
  "pulse": "https://cloudcache-1.example.com/pulse"
 },
 "users": [
  {
   "password": "developer-password",
   "roles": [
    "developer"
   ],
   "username": "developer_XXX"
  },
  {
   "password": "cluster_operator-password",
   "roles": [
    "cluster_operator"
   ],
   "username": "cluster_operator_XXX"
   }
 ],
 "wan": {
  "sender_credentials": {
   "active": {
    "password": "gws-XXX-password",
    "username": "gateway_sender_XXX"
   }
  }
 }
}
</pre>

The service key specifies the user roles and URLs that are predefined
for interacting with and within the cluster:

- The cluster operator administers the pool,
performing operations such as creating and destroying regions,
and creating gateway senders.
The identifier assigned for this role is of the form
`cluster_operator_XXX`, where `XXX` is a unique string generated
upon service instance creation and incorporated in this user role's name.
- The developer does limited cluster administration such as region creation,
and the developer role is expected
to be used by applications that are interacting with region entries.
The developer does CRUD operations on regions.
The identifier assigned for this role is of the form `developer_XXX`,
where `XXX` is a unique string generated
upon service instance creation and incorporated in this user role's name.
- The gateway sender writes data that is sent to another cluster.
The identifier assigned for this role is of the form `gateway_sender_XXX`,
where `XXX` is a unique string generated
upon service instance creation and incorporated in this user role's name.
- A URL used to connect the gfsh client to the service instance
- A URL used to view the Pulse dashboard in a web browser,
which allows monitoring of the service instance status.
Use the developer credentials to authenticate.

## <a id="gfsh-connect-https"></a> Connect with gfsh over HTTPS

When connecting over HTTPS, you must use the same certificate you use to secure traffic into Pivotal Application Service (PAS)
or Elastic Runtime; that is, the certificate you use where your TLS termination occurs.
Before you can connect, you must create a truststore. 

### <a id="truststore"></a> Create a Truststore 

To create a truststore, use the same certificate you used to configure TLS termination. We suggest using the `keytool` command line utility to create a truststore file. 

1. Locate the certificate you use to configure TLS termination. 
1. Using your certificate, run the `keytool` command.

    For example:

    `$ keytool -import -alias ENV -file CERTIFICATE.CER -keystore TRUSTSTORE-FILE-PATH"`

    Where:<br>
    + `ENV` is your system environment.
    + `CERTIFICATE.CER` is your certificate file.
    + `TRUSTSTORE-FILE-PATH` is the path to the location where you want to create the truststore file, including the name you want to give the file.

3. When you run this command, you are prompted to enter a keystore password. Create a password and remember it!
4. When prompted for the certificate details, enter **yes** to trust the certificate.

The following example shows how to run `keytool` and what the output looks like:

<pre class='terminal'>
$ keytool -import -alias prod-ssl -file /tmp/loadbalancer.cer -keystore /tmp/truststore/prod.myTrustStore 
Enter keystore password:
Re-enter new password:
Owner: CN=*.url.example.com, OU=Cloud Foundry, O=Pivotal, L=New York, ST=New York, C=US
Issuer: CN=*.url.example.com, OU=Cloud Foundry, O=Pivotal, L=New York, ST=New York, C=US
Serial number: bd84912917b5b665
Valid from: Sat Jul 29 09:18:43 EDT 2017 until: Mon Apr 07 09:18:43 EDT 2031
Certificate fingerprints:
   MD5:  A9:17:B1:C9:6C:0A:F7:A3:56:51:6D:67:F8:3E:94:35
   SHA1: BA:DA:23:09:17:C0:DF:37:D9:6F:47:05:05:00:44:6B:24:A1:3D:77
   SHA256: A6:F3:4E:B8:FF:8F:72:92:0A:6D:55:6E:59:54:83:30:76:49:80:92:52:3D:91:4D:61:1C:A1:29:D3:BD:56:57
   Signature algorithm name: SHA256withRSA
   Version: 3

Extensions:

#1: ObjectId: 2.5.29.10 Criticality=true
BasicConstraints:[
  CA:true
  PathLen:0
]

#2: ObjectId: 2.5.29.11 Criticality=false
SubjectAlternativeName [
  DNSName: *.sys.url.example.com
  DNSName: *.apps.url.example.com
  DNSName: *.uaa.sys.url.example.com
  DNSName: *.login.sys.url.example.com
  DNSName: *.url.example.com
  DNSName: *.ws.url.example.com
]

Trust this certificate? [no]:  yes
Certificate was added to keystore
</pre>

### <a id="establish-https"></a> Establish the Connection with HTTPS 

After you have created the truststore, you can connect using HTTPS.

1. Download the Pivotal GemFire ZIP archive from [Pivotal Network](https://network.pivotal.io/products/pivotal-gemfire).
1. Unzip and locate the correct binary for your architecture. Use `gfsh` with Unix or `gfsh.bat` with Windows.
1. Add the `gfsh` binary to your path.
1. Set the `JAVA_ARGS` environment variable with the following command:

    `export JAVA_ARGS="-Djavax.net.ssl.trustStore=TRUSTSTORE-FILE-PATH"`

    Where: 
    `TRUSTSTORE-FILE-PATH` is the path to the TrustStore file you created in [Create a Truststore](#truststore).

    For example,
    <pre class='terminal'>
    $ export JAVA_ARGS="-Djavax.net.ssl.trustStore=/tmp/truststore/prod.myTrustStore"
    </pre>
5. Run the `gfsh` command-line interface,
and then issue a `connect` command that specifies an HTTPS URL of the form:

    ```
    connect --use-http=true --url=<HTTPS-gfsh-URL>
     --user=<cluster_operator_XXX>
     --password=<cluster_operator-password>
    ```

### <a id="dev-establish-https"></a> Establish the Connection with HTTPS in a Development Environment 

When working in a non-production, development environment,
a developer may choose to work in a less secure manner
by eliminating the truststore and SSL mutual authentication.

The steps to establish the `gfsh` connection become:

1. Download the Pivotal GemFire ZIP archive from [Pivotal Network](https://network.pivotal.io/products/pivotal-gemfire).
1. Unzip and locate the correct binary for your architecture. Use `gfsh` with Unix or `gfsh.bat` with Windows.
1. Add the `gfsh` binary to your path.
1. Run the `gfsh` command-line interface,
and then issue a `connect` command that specifies an HTTPS URL of the form:

    ```
    connect --use-http=true --use-ssl --skip-ssl-validation=true
     --url=<HTTPS-gfsh-URL> --user=<cluster_operator_XXX>
     --password=<cluster_operator-password>
    ```
