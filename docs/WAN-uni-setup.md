---
title: Set Up a Unidirectional System
---

This sequence of steps sets up a unidirectional transfer,
such that all operations in cluster A are replicated in cluster B.
Two design patterns that use unidirectional replication are described in
[Blue-Green Disaster Recovery](design-patterns.html#WAN-pattern)
and
[CQRS Pattern Across a WAN](design-patterns.html#CQRS-WAN-pattern).

1. Create the cluster A service instance
using the cluster A Cloud Foundry credentials.
This example explicitly sets the `distributed_system_id` of cluster A
using a `-c` option with a command of the form:

    ```
    cf create-service p-cloudcache PLAN-NAME SERVICE-INSTANCE-NAME -c '{
    "distributed_system_id" : ID-VALUE }'
    ```

    Here is a cluster A example of the `create-service` command:

    <pre class='terminal'>
    $ cf create-service p-cloudcache wan-cluster wan1 -c '{
    "distributed_system_id" : 1 }'
    </pre>

    Verify the completion of service creation prior to continuing
    to the next step.
    Output from the `cf services` command will show the `last operation` as
    `create succeeded` when service creation completes.

1. Create a service key for cluster A.
The service key will contain generated credentials that this example
will use in the creation of the cluster B service instance:

    <pre class='terminal'>
    $ cf create-service-key wan1 k1
    </pre>

    Within the service key,
    each `username` is generated with a unique string
    appended so there will be unique user names for the different roles.
    The user names in this example have been modified
    to be easy to understand,
    and they are not representative of the user names that
    will be generated upon service key creation.
    Passwords generated for the service key are output in clear text.
    The passwords shown in this example have been
    modified to be easy to understand,
    and they are not representative of the passwords that
    will be generated upon service key creation.
    Here is sample output from `cf service-key wan1 k1`:

    <pre class='terminal'>
    Getting key k1 for service instance wan1 as admin...


    {
     "distributed_system_id": "1",
     "locators": [
      "10.0.16.21[55221]"
      "10.0.16.22[55221]"
      "10.0.16.23[55221]"
     ],
     "urls": {
      "gfsh": "https://cloudcache-1.example.com/gemfire/v1",
      "pulse": "https://cloudcache-1.example.com/pulse"
     },
     "users": [
      {
       "password": "cl-op-ABC-password",
       "roles": [
        "cluster_operator"
       ],
       "username": "cluster_operator_ABC"
      },
      {
       "password": "dev-DEF-password",
       "roles": [
        "developer"
       ],
       "username": "developer_DEF"
      }
     ],
     "wan": {
      "sender_credentials": {
       "active": {
        "password": "gws-GHI-password",
        "username": "gateway_sender_GHI"
       }
      }
     }
    }
    </pre>

1. Communicate the cluster A locators' IP and port addresses and
`sender_credentials` to the cluster B Cloud Foundry administrator. 

1.  Create the cluster B service instance
using the cluster B Cloud Foundry credentials.
This example explicitly sets the `distributed_system_id`.
Use a `-c` option with the command to specify the `distributed_system_id`,
the cluster A service instance's locators,
and the cluster A `sender_credentials`:

    <pre class='terminal'>
    $ cf create-service p-cloudcache wan-cluster wan2 -c '
    {
      "distributed_system_id":2,
      "remote_clusters":[
      {
        "remote_locators":[
          "10.0.16.21[55221]",
          "10.0.16.22[55221]",
          "10.0.16.23[55221]"],
        "trusted_sender_credentials":[
        {
          "username": "gateway_sender_GHI",
          "password":"gws-GHI-password"
        }]
      }]
    }'
    </pre>

    Verify the completion of service creation prior to continuing
    to the next step.
    Output from the `cf services` command will show the `last operation` as
    `create succeeded` when service creation is completed.

1. Create the service key of cluster B:

    <pre class='terminal'>
    $ cf create-service-key wan2 k2
    </pre>

    Note that the cluster B service key will contain unneeded
    (for the unidirectional setup)
    but automatically created `sender_credentials`.
    Here is sample output from `cf service-key wan2 k2`,
    which outputs details of the cluster B service key:

    <pre class='terminal'>
    Getting key k2 for service instance destination as admin...

    {
     "distributed_system_id": "2",
     "locators": [
      "10.0.24.21[55221]"
      "10.0.24.22[55221]"
      "10.0.24.23[55221]"
     ],
     "urls": {
      "gfsh": "https://cloudcache-2.example.com/gemfire/v1",
      "pulse": "https://cloudcache-2.example.com/pulse"
     },
     "users": [
      {
       "password": "cl-op-JKL-password",
       "roles": [
        "cluster_operator"
       ],
       "username": "cluster_operator_JKL"
      },
      {
       "password": "dev-MNO-password",
       "roles": [
        "developer"
       ],
       "username": "developer_MNO"
      }
     ],
     "wan": {
      "remote_clusters": [
      {
        "remote_locators": [
          "10.0.16.21[55221]",
          "10.0.16.21[55221]",
          "10.0.16.21[55221]"
        ],
        "trusted_sender_credentials": [
         "gateway_sender_GHI"
        ]
       }
      ],
      "sender_credentials": {
       "active": {
        "password": "gws-PQR-password",
        "username": "gateway_sender_PQR"
       }
      }
     }
    }
    </pre>

1. Communicate the cluster B locators' IP and port addresses 
to the cluster A Cloud Foundry administrator. 

1. Update the cluster A service instance using the cluster A
Cloud Foundry credentials to include the cluster B locators:

    <pre class='terminal'>
    $ cf update-service wan1 -c '
    {
      "remote_clusters":[
      {
        "remote_locators":[
          "10.0.24.21[55221]",
          "10.0.24.22[55221]",
          "10.0.24.23[55221]"]
      }]
    }'
    Updating service instance wan1 as admin
    </pre>

1. To observe and verify that the cluster A service instance has
been correctly updated,
it is necessary to delete and recreate the cluster A service key.
As designed, the recreated service key will have the same user identifiers
and passwords; new unique strings and passwords are not generated.
Use the cluster A Cloud Foundry credentials
in these commands:

    <pre class='terminal'>
    $ cf delete-service-key wan1 k1
    </pre>

    <pre class='terminal'>
    $ cf create-service-key wan1 k1
    </pre>

    The cluster A service key will now appear as:

    <pre class='terminal'>
    Getting key k1 for service instance wan1 as admin...

    {
     "distributed_system_id": "1",
     "locators": [
      "10.0.16.21[55221]",
      "10.0.16.22[55221]",
      "10.0.16.23[55221]"
     ],
     "urls": {
      "gfsh": "https://cloudcache-1.example.com/gemfire/v1",
      "pulse": "https://cloudcache-1.example.com/pulse"
     },
     "users": [
      {
       "password": "cl-op-ABC-password",
       "roles": [
        "cluster_operator"
       ],
       "username": "cluster_operator_ABC"
      },
      {
       "password": "dev-DEF-password",
       "roles": [
        "developer"
       ],
       "username": "developer_DEF"
      }
     ],
     "wan": {
      "remote_clusters": [
       {
        "remote_locators": [
         "10.0.24.21[55221]",
         "10.0.24.22[55221]",
         "10.0.24.23[55221]"
        ] ]
       }
      ],
      "sender_credentials": {
       "active": {
        "password": "gws-GHI-password",
        "username": "gateway_sender_GHI"
       }
      }
     }
    }
    </pre>


1. Use gfsh to create the cluster A gateway sender and the region.
Any region operations that occur after the region is created on
cluster A, but before the region is created on cluster B
will be lost.
    - Connect using gfsh and the cluster A `cluster_operator` credentials,
    which are needed to be authorized for the gateway sender
    creation operation:
        <pre class='terminal'>
        gfsh>connect --url=htt<span>ps</span>://cloudcache-1.example.com/gemfire/v1 --use-http --user=cluster\_operator\_ABC --password=cl-op-ABC-password
        </pre>
    - Create the cluster A gateway sender.
    The required `remote-distributed-system-id` option identifies the `distributed-system-id` of the destination cluster. It is 2 for this example:

        <pre class='terminal'>
        gfsh>create gateway-sender --id=send_to_2 --remote-distributed-system-id=2 --enable-persistence=true
        </pre>
    - Create the cluster A region.
    The `gateway-sender-id` associates region operations with a specific
    gateway sender.  The region must have an associated gateway sender in
    order to propagate region events across the WAN.

        <pre class='terminal'>
        gfsh>create region --name=regionX --gateway-sender-id=send_to_2 --type=PARTITION_REDUNDANT
        </pre>

1. Use gfsh to create the cluster B region.
    - Connect using gfsh and the cluster B `cluster_operator` credentials,
    which are needed to be authorized for the create operation:
        <pre class='terminal'>
        gfsh>connect --url=htt<span>ps</span>://cloudcache-2.example.com/gemfire/v1 --use-http --user=cluster\_operator\_JKL --password=cl-op-JKL-password
        </pre>
    - Create the cluster B region:

        <pre class='terminal'>
        gfsh>create region --name=regionX --type=PARTITION_REDUNDANT
        </pre>
