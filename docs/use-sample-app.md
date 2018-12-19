---
title: Use a Sample Java Client App with PCC
---
<a id="sample-java-app-connect"></a>

The sample Java client app at
[https://github.com/cf-gemfire-org/cloudcache-sample-app.git](https://github.com/cf-gemfire-org/cloudcache-sample-app.git)
demonstrates how to connect an app to a service instance.

These instructions assume:

-  A PCC service instance is running.
-  You have Cloud Foundry credentials for accessing the PCC service instance.
-  You have a service key for the PCC service instance.
-  You have a login on the Pivotal Commercial Maven Repository
at [https://commercial-repo.pivotal.io](https://commercial-repo.pivotal.io).
-  You have a `gfsh` client of the same version as is used
within your PCC service instance.

Follow these instructions to run the app.

1. Clone the sample Java app from [https://github.com/cf-gemfire-org/cloudcache-sample-app.git](https://github.com/cf-gemfire-org/cloudcache-sample-app.git).
1. Update your clone of the sample Java app to work with
your PCC service instance:
    - Modify the manifest in `manifest.yml` by replacing `service0` with 
    the name of your PCC service instance.
    - Replace the username and password in the <code>gradle.properties</code>
    file with your username and password for
    the Pivotal Commercial Maven Repository.
    - Update the GemFire version in the dependencies section of the
    `build.gradle` file to be the same as the version within your PCC service
    instance.
1. Build the app with

    ```
    $ ./gradlew clean build
    ```
1. In a second shell, run `gfsh`.
1. Use `gfsh` to connect to the PCC service instance as described in
[Connect with gfsh over HTTPS](accessing-instance.html#gfsh-connect-https).
1. Use `gfsh` to create a region named `test` as described in [Create Regions with gfsh](./using-pcc.html#create-regions).
This sample app places a single entry into the region,
so the region type is not important.
`PARTITION_REDUNDANT` is a good choice.
1. In the shell where the app was build, deploy and run the app with

    ```
    cf push -f manifest.yml
    ```
1. After the app starts, there will be an entry of ("1", "one")
in the <code>test</code> region. you can see that there is one
entry in the region with the `gfsh` command:

    ```
    gfsh>describe region --name=test
    ```
    For this very small region, you can print the contents of the entire 
    region with a `gfsh` query:

    ```
    gfsh>query --query='SELECT * FROM /test'
    ```
