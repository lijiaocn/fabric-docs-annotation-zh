Hyperledger Fabric Functionalities(功能介绍)
==================================

Hyperledger Fabric is an implementation of distributed ledger technology
(DLT) that delivers enterprise-ready network security, scalability,
confidentiality and performance, in a modular blockchain architecture.
Hyperledger Fabric delivers the following blockchain network functionalities:

Identity management
-------------------

To enable permissioned networks, Hyperledger Fabric provides a membership
identity service that manages user IDs and authenticates all participants on
the network. Access control lists can be used to provide additional layers of
permission through authorization of specific network operations. For example, a
specific user ID could be permitted to invoke a chaincode application, but
blocked from deploying new chaincode.

.. note::

    Fabric提供了认证服务，用来管理用户ID、并核实参与者的身份。这里内容比较多，
    参考：http://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/04/27/hyperledger-fabric-ca-usage.html
    
    并且提供了ACL，限制参与者能够进行的操作。

Privacy and confidentiality
---------------------------

Hyperledger Fabric enables competing business interests, and any groups that
require private, confidential transactions, to coexist on the same permissioned
network. Private **channels** are restricted messaging paths that can be used
to provide transaction privacy and confidentiality for specific subsets of
network members. All data, including transaction, member and channel
information, on a channel are invisible and inaccessible to any network members
not explicitly granted access to that channel.

Efficient processing
--------------------

Hyperledger Fabric assigns network roles by node type. To provide concurrency
and parallelism to the network, transaction execution is separated from
transaction ordering and commitment. Executing transactions prior to
ordering them enables each peer node to process multiple transactions
simultaneously. This concurrent execution increases processing efficiency on
each peer and accelerates delivery of transactions to the ordering service.

In addition to enabling parallel processing, the division of labor unburdens
ordering nodes from the demands of transaction execution and ledger
maintenance, while peer nodes are freed from ordering (consensus) workloads.
This bifurcation of roles also limits the processing required for authorization
and authentication; all peer nodes do not have to trust all ordering nodes, and
vice versa, so processes on one can run independently of verification by the
other.

.. note:: 

    在Fabric中交易的入账和排序是分离的。
    Ordering节点只负责对交易进行排序，peer节点负责交易的入账。

Chaincode functionality
-----------------------

Chaincode applications encode logic that is
invoked by specific types of transactions on the channel. Chaincode that
defines parameters for a change of asset ownership, for example, ensures that
all transactions that transfer ownership are subject to the same rules and
requirements. **System chaincode** is distinguished as chaincode that defines
operating parameters for the entire channel. Lifecycle and configuration system
chaincode defines the rules for the channel; endorsement and validation system
chaincode defines the requirements for endorsing and validating transactions.

.. note::

    Chaincode中编码了交易的处理逻辑，保证所有的交易遵循同样的规则。
    这里提出了“System chaincode”以及其它chaincode，比较奇怪，需要核实
    下，是不是新增加的特性。（2018-06-22 17:07:06)

Modular design
--------------

Hyperledger Fabric implements a modular architecture to
provide functional choice to network designers. Specific algorithms for
identity, ordering (consensus) and encryption, for example, can be plugged in
to any Hyperledger Fabric network. The result is a universal blockchain
architecture that any industry or public domain can adopt, with the assurance
that its networks will be interoperable across market, regulatory and
geographic boundaries.

.. note::

    Fabric是模块化设计的，这个不多说了。在深入使用时，就会知道，很多地方都是
    可以配置的。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
