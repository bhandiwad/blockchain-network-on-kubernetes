
1. Deploy Hyperledger Fabric Network into Kubernetes Cluster

#### Understand the network topology

This pattern provides a script which automatically provisions a sample Hyperledger Fabric network consisting of four organizations, each maintaining one peer node, and a 'solo' ordering service. Also, the script creates a channel named as `channel1`, joins all peers to the channel `channel1`, install chaincode on all peers and instantiate chaincode on channel. The pattern also helps to drive execution of transactions against the deployed chaincode.

#### Copy Kubernetes configuration scripts

Clone or download the Kubernetes configuration scripts to your user home directory.
  ```
  $ git clone https://github.com/bhandiwad/blockchain-network-on-kubernetes
  ```

Navigate to the source directory
  ```
  $ cd blockchain-network-on-kubernetes
  $ ls
  ```
In the source directory,
  * `configFiles` contains Kubernetes configuration files
  * `artifacts` contains the network configuration files
  * `*.sh` scripts to deploy and delete the network

#### Modify the Kubernetes configuration scripts

If there is any change in network topology, need to modify the configuration files (.yaml files) appropriately. The configuration files are located in `artifacts` and `configFiles` directory. For example, if you decide to increase/decrease the capacity of persistent volume then you need to modify `createVolume.yaml`.

The Kubernetes Server version v1.11.x or above uses `containerd` as its container runtime therefore using `docker.sock` of the worker node is not possible. You need to deploy and use a Docker daemon in a container. In case, your Kubernetes server version is smaller than 1.11.x then you need to modify the `configFiles/peersDeployment.yaml` file to point to a Docker service. Change instances of `tcp://docker:2375` to `unix:///host/var/run/docker.sock` with a text editor.

#### Run the script to deploy your Hyperledger Fabric Network

Once you have completed the changes (if any) in configuration files, you are ready to deploy your network. 

Check your kubectl CLI version as:

```
$ kubectl version --short
```

This command will give you `Client Version` and `Server Version`. 
If the `Client version > v1.11.x` i.e. 1.12.x or more then use `setup_blockchainNetwork_v2.sh` to set up the network. Run the following command:

```
cp setup_blockchainNetwork_v2.sh setup_blockchainNetwork.sh
```

If the `Client version <= v1.11.x` then use `setup_blockchainNetwork_v1.sh` to setup the network. Copy the script as shown.
```
cp setup_blockchainNetwork_v1.sh setup_blockchainNetwork.sh
```

Now execute the script to deploy your hyperledger fabric network.

  ```
  $ chmod +x setup_blockchainNetwork.sh
  $ ./setup_blockchainNetwork.sh
  ```

  > If you are using a Standard IKS cluster with multiple workers nodes, do `./setup_blockchainNetwork.sh --paid` so that the shared volume of the blockchain containers would work properly.

Note: Before running the script, please check your environment. You should able to run `kubectl commands` properly with your cluster as explained in step 3.

#### Delete the network

If required, you can bring your hyperledger fabric network down using the script `deleteNetwork.sh`. This script will delete all your pods, jobs, deployments etc. from your Kubernetes cluster.

  ```
  $ chmod +x deleteNetwork.sh
  $ ./deleteNetwork.sh
  ```

### 2. Test the deployed network

After successful execution of the script `setup_blockchainNetwork.sh`, check the status of pods.

  ```
  $ kubectl get pods
  NAME                                    READY     STATUS    RESTARTS   AGE
  blockchain-ca-7848c48d64-2cxr5          1/1       Running   0          4m
  blockchain-orderer-596ccc458f-thdgn     1/1       Running   0          4m
  blockchain-org1peer1-747d6bdff4-4kzts   1/1       Running   0          4m
  blockchain-org2peer1-7794d9b8c5-sn2qf   1/1       Running   0          4m
  blockchain-org3peer1-59b6d99c45-dhtbp   1/1       Running   0          4m
  blockchain-org4peer1-6b6c99c45-wz9wm    1/1       Running   0          4m
  ```

As mentioned above, the script joins all peers on one channel `channel1`, install chaincode on all peers and instantiate chaincode on channel. It means we can execute an invoke/query command on any peer and the response should be same on all peers. Please note that in this pattern tls certs are disabled to avoid complexity. In this pattern, the CLI commands are used to test the network. For running a query against any peer, need to get into a bash shell of a peer, run the query and exit from the peer container.

Use the following command to get into a bash shell of a peer:

  ```
  $ kubectl exec -it <blockchain-org1peer1 pod name> bash
  ```

And the command to be used to exit from the peer container is:

  ```
  # exit
  ```

**Note:** Stay logged into your peer to complete these commands.

**Query**

Chaincode was instantiated with the values as `{ a: 100, b: 200 }`. Let’s query to `org1peer1` for the value of `a` to make sure the chaincode was properly instantiated.
  ```
  peer chaincode query -C channel1 -n cc -c '{"Args":["query","a"]}'
  ```

  ![](images/first-query.png)

**Invoke**

Now let’s submit a request to `org2peer1` to move 20 from `a` to `b`. A new transaction will be generated and upon successful completion of transaction, state will get updated.
  ```
  peer chaincode invoke -o blockchain-orderer:31010 -C channel1 -n cc -c '{"Args":["invoke","a","b","20"]}'
  ```

  ![](images/invoke.png)

**Query**

Let’s confirm that our previous invocation executed properly. We initialized the key `a` with a value of 100 and just removed 20 with our previous invocation. Therefore, a query against `a` should show 80 and a query against `b` should show 220. Now issue the query request to `org3peer1` and `org4peer1` as shown.
  ```
  peer chaincode query -C channel1 -n cc -c '{"Args":["query","a"]}'
  peer chaincode query -C channel1 -n cc -c '{"Args":["query","b"]}'
  ```

  ![](images/second-query.png)

  ![](images/third-query.png)


## Troubleshooting

[See DEBUGGING.md.](DEBUGGING.md)

## Reference Links

* [Hyperledger Fabric](https://hyperledger-fabric.readthedocs.io/en/release-1.1/)
* [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)

## License

This code pattern is licensed under the Apache Software License, Version 2.  Separate third party code objects invoked within this code pattern are licensed by their respective providers pursuant to their own separate licenses. Contributions are subject to the [Developer Certificate of Origin, Version 1.1 (DCO)](https://developercertificate.org/) and the [Apache Software License, Version 2](https://www.apache.org/licenses/LICENSE-2.0.txt).

[Apache Software License (ASL) FAQ](https://www.apache.org/foundation/license-faq.html#WhatDoesItMEAN)
