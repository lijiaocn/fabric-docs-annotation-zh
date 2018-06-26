Capability Requirements(特性约束)
---------------------------------

Because Fabric is a distributed system that will usually involve multiple
organizations (sometimes in different countries or even continents), it is
possible (and typical) that many different versions of Fabric code will exist in
the network. Nevertheless, it’s vital that networks process transactions in the
same way so that everyone has the same view of the current network state.

.. note:: Fabric是由很多组织参与形成的分布式网络，不同的组织可能使用不同版本的
          软件，导致网络中会有多个版本的软件存在。
          需要有一个协调的方法， 防止因版本不同引发混乱， “特性约束（capability
          requirments）”就是被用来
          做协调的。

This means that every network -- and every channel within that network – must
define a set of what we call “capabilities” to be able to participate in
processing transactions. For example, Fabric v1.1 introduces new MSP role types
of “Peer” and “Client”. However, if a v1.0 peer does not understand these new
role types, it will not be able to appropriately evaluate an endorsement policy
that references them. This means that before the new role types may be used, the
network must agree to enable the v1.1 ``channel`` capability requirement,
ensuring that all peers come to the same decision.

Only binaries which support the required capabilities will be able to participate in the
channel, and newer binary versions will not enable new validation logic until the
corresponding capability is enabled.  In this way, capability requirements ensure that
even with disparate builds and versions, it is not possible for the network to suffer a
state fork.

.. note:: 具体做法就是定义一个“特性集(capabilities)”，只有支持所要求的特性的程序能够
          加入Channel，并且版本较新的程序不能使用“特性集”之外的新特性，必须与其它Peer
          保持一致。

Defining Capability Requirements
================================

Capability requirements are defined per channel in the channel configuration (found
in the channel’s most recent configuration block). The channel configuration contains
three locations, each of which defines a capability of a different type.

.. note:: “特性约束”在Channel的配置中设置，分为Channel、Orderer、Application三类,
          作用于不同的组件，也可以理解为三个级别，作用范围不同。

+------------------+-----------------------------------+----------------------------------------------------+
| Capability Type  | Canonical Path                    | JSON Path                                          |
+==================+===================================+====================================================+
| Channel          | /Channel/Capabilities             | .channel_group.values.Capabilities                 |
+------------------+-----------------------------------+----------------------------------------------------+
| Orderer          | /Channel/Orderer/Capabilities     | .channel_group.groups.Orderer.values.Capabilities  |
+------------------+-----------------------------------+----------------------------------------------------+
| Application      | /Channel/Application/Capabilities | .channel_group.groups.Application.values.          |
|                  |                                   | Capabilities                                       |
+------------------+-----------------------------------+----------------------------------------------------+

* **Channel:** these capabilities apply to both peer and orderers and are located in
  the root ``Channel`` group.

* **Orderer:** apply to orderers only and are located in the ``Orderer`` group.

* **Application:** apply to peers only and are located in the ``Application`` group.

The capabilities are broken into these groups in order to align with the existing
administrative structure. Updating orderer capabilities is something the ordering orgs
would manage independent of the application orgs. Similarly, updating application
capabilities is something only the application admins would manage. By splitting the
capabilities between "Orderer" and "Application", a hypothetical network could run a
v1.6 ordering service while supporting a v1.3 peer application network.

.. note:: Orderer和Application的特性约束是分开管理、独立设置的。

However, some capabilities cross both the ‘Application’ and ‘Orderer’ groups. As we
saw earlier, adding a new MSP role type is something both the orderer and application
admins agree to and need to recognize. The orderer must understand the meaning
of MSP roles in order to allow the transactions to pass through ordering, while
the peers must understand the roles in order to validate the transaction. These
kinds of capabilities -- which span both the application and orderer components
-- are defined in the top level "Channel" group.

.. note:: Orderer和Application都包含的特性，在Channel中管理。

.. note:: It is possible that the channel capabilities are defined to be at version
          v1.3 while the orderer and application capabilities are defined to be at
          version 1.1 and v1.4, respectively. Enabling a capability at the "Channel"
          group level does not imply that this same capability is available at the
          more specific "Orderer" and "Application" group levels.

Setting Capabilities
====================

Capabilities are set as part of the channel configuration (either as part of the
initial configuration -- which we'll talk about in a moment -- or as part of a
reconfiguration).

.. note:: We have a two documents that talk through different aspects of channel
          reconfigurations. First, we have a tutorial that will take you through
          the process of :doc:`channel_update_tutorial`. And we also have a document that
          talks through :doc:`config_update` which gives an overview of the
          different kinds of updates that are possible as well as a fuller look
          at the signature process.

Because new channels copy the configuration of the Orderer System Channel by
default, new channels will automatically be configured to work with the orderer
and channel capabilities of the Orderer System Channel and the application
capabilities specified by the channel creation transaction. Channels that already
exist, however, must be reconfigured.

The schema for the Capabilities value is defined in the protobuf as:

.. code:: bash

  message Capabilities {
        map<string, Capability> capabilities = 1;
  }

  message Capability { }

As an example, rendered in JSON:

.. code:: bash

  {
      "capabilities": {
          "V1_1": {}
      }
  }

Capabilities in an Initial Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the ``configtx.yaml`` file distributed in the ``config`` directory of the release
artifacts, there is a ``Capabilities`` section which enumerates the possible capabilities
for each capability type (Channel, Orderer, and Application).

The simplest way to enable capabilities is to pick a v1.1 sample profile and customize
it for your network. For example:

.. code:: bash

    SampleSingleMSPSoloV1_1:
        Capabilities:
            <<: *GlobalCapabilities
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *SampleOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *SampleOrg

Note that there is a ``Capabilities`` section defined at the root level (for the channel
capabilities), and at the Orderer level (for orderer capabilities). The sample above uses
a YAML reference to include the capabilities as defined at the bottom of the YAML.

.. note:: 上面的例子中，有两个Capabilities，第一个是Channel级别的，是Channel的特性约束，
          第二个是Orderer的特性约束，是“Orderer”的子项。

When defining the orderer system channel there is no Application section, as those
capabilities are defined during the creation of an application channel. To define a new
channel's application capabilities at channel creation time, the application admins should
model their channel creation transaction after the ``SampleSingleMSPChannelV1_1`` profile.

.. code:: bash

   SampleSingleMSPChannelV1_1:
        Consortium: SampleConsortium
        Application:
            Organizations:
                - *SampleOrg
            Capabilities:
                <<: *ApplicationCapabilities

.. note:: 应用级别的特性约束。

Here, the Application section has a new element ``Capabilities`` which references the
``ApplicationCapabilities`` section defined at the end of the YAML.

.. note:: The capabilities for the Channel and Orderer sections are inherited from
          the definition in the ordering system channel and are automatically included
          by the orderer during the process of channel creation.

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
