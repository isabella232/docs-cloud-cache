---
title: Set Up an Additional Unidirectional Interaction
---

Follow this sequence of steps to set up an additional unidirectional
transfer over WAN between two PCC service instances,
once an initial setup is in place for a first pair of PCC
service instances.

Call the first pair of PCC service instances A and B.
This set of directions sets up a unidirectional interaction from
service instance A to service instance C.
Service instance A is already created and has a service key. 

The GemFire cluster within each service instance uses an identifier
called a `distributed_system_id`.
This example assumes the assignment of `distributed_system_id = 1`
for cluster A,
`distributed_system_id = 2` for cluster B,
and `distributed_system_id = 3` for cluster C.

1. Communicate the cluster A locators' IP and port addresses and
`sender_credentials` to the cluster C Cloud Foundry administrator. 

1.  Create the cluster C service instance
using the cluster C Cloud Foundry credentials.
This example explicitly sets the `distributed_system_id`.
Use a `-c` option with the command to specify the `distributed_system_id`,
the cluster A service instance's locators,
and the cluster A `sender_credentials`:

    <pre class='terminal'>
    $ cf create-service p-cloudcache wan-cluster wan3 -c '
    {
      "distributed_system_id":3,
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
    `create succeeded` when service creation completes.

1. Create the service key of cluster C:

    <pre class='terminal'>
    $ cf create-service-key wan3 k3
    </pre>

    Note that the cluster C service key will contain unneeded
    (for the unidirectional setup)
    but automatically created `sender_credentials`.
    Here is sample output from `cf service-key wan3 k3`,
    which outputs details of the cluster C service key:

    <pre class='terminal'>
    Getting key k3 for service instance destination as admin...

    {
     "distributed_system_id": "3",
     "locators": [
      "10.0.32.21[55221]"
      "10.0.32.22[55221]"
      "10.0.32.23[55221]"
     ],
     "urls": {
      "gfsh": "https://cloudcache-3.example.com/gemfire/v1",
      "pulse": "https://cloudcache-3.example.com/pulse"
     },
     "users": [
      {
       "password": "cl-op-STU-password",
       "roles": [
        "cluster_operator"
       ],
       "username": "cluster_operator_STU"
      },
      {
       "password": "dev-VWX-password",
       "roles": [
        "developer"
       ],
       "username": "developer_VWX"
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
        "password": "gws-YZA-password",
        "username": "gateway_sender_YZA"
       }
      }
     }
    }
    </pre>

1. Communicate the cluster C locators' IP and port addresses 
to the cluster A Cloud Foundry administrator. 

1. Update the cluster A service instance using the cluster A
Cloud Foundry credentials to include the cluster C locators.
The cluster A service instance must specify as `remote_locators`
the details for all clusters it interacts with.
For this example, that is both clusters B and C:

    <pre class='terminal'>
    $ cf update-service wan1 -c '
    {
      "remote_clusters":[
      {
        "remote_locators":[
          "10.0.24.21[55221]",
          "10.0.24.22[55221]",
          "10.0.24.23[55221]",
          "10.0.32.21[55221]",
          "10.0.32.22[55221]",
          "10.0.32.23[55221]"]
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
         "10.0.24.23[55221]",
         "10.0.32.21[55221]",
         "10.0.32.22[55221]",
         "10.0.32.23[55221]"
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


1. Use gfsh to create the cluster A gateway sender and alter the
existing region.
    - Connect using gfsh and the cluster A `cluster_operator` credentials,
    which are needed to be authorized for the gateway sender
    creation operation:
        <pre class='terminal'>
        gfsh>connect --url=htt<span>ps</span>://cloudcache-1.example.com/gemfire/v1 --use-http --user=cluster\_operator\_ABC --password=cl-op-ABC-password
        </pre>
    - Create the cluster A gateway sender.
    The required `remote-distributed-system-id` option identifies the `distributed-system-id` of the destination cluster. It is 3 for this example:

        <pre class='terminal'>
        gfsh>create gateway-sender --id=send_to_3 --remote-distributed-system-id=3 --enable-persistence=true
        </pre>
    - Alter the existing cluster A region
    so that it specifies all gateway senders associated with the region.
    There are two gateway senders in this example,
    one that goes to cluster B and a second that goes to cluster C.

        <pre class='terminal'>
        gfsh>alter region --name=regionX --gateway-sender-id=send_to_2,send_to_3
        </pre>

1. Use gfsh to create the cluster C region.
    - Connect using gfsh and the cluster C `cluster_operator` credentials,
    which are needed to be authorized for the create operation:
        <pre class='terminal'>
        gfsh>connect --url=htt<span>ps</span>://cloudcache-3.example.com/gemfire/v1 --use-http --user=cluster\_operator\_STU --password=cl-op-STU-password
        </pre>
    - Create the cluster B region:

        <pre class='terminal'>
        gfsh>create region --name=regionX --type=PARTITION_REDUNDANT
        </pre>
