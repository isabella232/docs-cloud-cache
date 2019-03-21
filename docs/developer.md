
# Pivotal Cloud Cache Developer Guide



This document describes how a Pivotal Cloud Foundry (PCF) app developer can choose a service plan, create and delete Pivotal Cloud Cache (PCC) service instances, and bind an app.

You must install the [Cloud Foundry Command Line Interface](http://docs.pivotal.io/pivotalcf/cf-cli/install-go-cli.html) (cf CLI) to run the commands in this topic.

In this topic:

- [Viewing All Plans Available for Pivotal Cloud Cache](view-plans.html)
- [Creating a Pivotal Cloud Cache Service Instance](create-instance.html)
    - [Provide Optional Parameters](create-instance.html#params)
    - [Enable Session State Caching with the Java Buildpack](create-instance.html#ssc)
    - [Enable Session State Caching Using Spring Session](create-instance.html#ssc-spring-session)
    - [Dev Plans](create-instance.html#service-instance-dev-plan)
- [Set Up WAN-Separated Service Instances](WAN-setup.html)
    - [Set Up a Bidirectional System](WAN-setup.html#WAN-bidirectional-setup)
    - [Set Up a Unidirectional System](WAN-setup.html#WAN-unidirectional-setup)
- [Deleting a Service Instance](delete-instance.html)
- [Updating a Pivotal Cloud Cache Service Instance](update-instance.html)
    - [Rebalancing a Cluster](update-instance.html#cluster-rebalancing)
    - [Restarting a Cluster](update-instance.html#cluster-restart)
    - [About Changes to the Service Plan](update-instance.html#plan-updates)
- [Accessing a Service Instance](accessing-instance.html)
    - [Create Service Keys](accessing-instance.html#create-service-key)
    - [Connect with gfsh over HTTPS](accessing-instance.html#gfsh-connect-https)
        - [Create a Truststore](accessing-instance.html#truststore)
        - [Establish the Connection with HTTPS](accessing-instance.html#establish-https)
        - [Establish the Connection with HTTPS  in a Development Environment](accessing-instance.html#dev-establish-https)
- [Using Pivotal Cloud Cache](using-pcc.html)
    - [Create Regions with gfsh](using-pcc.html#create-regions)
    - [Java Build Pack Requirements](using-pcc.html#java-build-pack-requirement)
    - [Bind an App to a Service Instance](using-pcc.html#bind-service)
    - [Use the Pulse Dashboard](using-pcc.html#pulse)
    - [Access Service Metrics](using-pcc.html#nozzle)
    - [Access Service Broker Metrics](using-pcc.html#nozzle-service-metrics)
    - [Export gfsh logs](using-pcc.html#exporting_logs)
    - [Deploy an App JAR File to the Servers](using-pcc.html#deploy-app-jars)
- [Connecting a Spring Boot App to Pivotal Cloud Cache with Session State Caching](Spring-SessionState.html)
    - [Use the Tomcat App](Spring-SessionState.html#tomcat)
    - [Use a Spring Session Data GemFire App](Spring-SessionState.html#spring-session)
- [Creating Continuous Queries Using Spring Data GemFire](Spring-CQs.html)
