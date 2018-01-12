# Self Healing Kubernetes Cluster
A proposal for a self healing Kubernetes cluster that makes use of community driven node problem detector 
to repair broken down VMs in a Kubernetes cluster

## Abstract
Kubernetes cluster nodes are often found to be in an unhealthy state for different reasons. There has been an initial attempt by the community to make various node problems visible to the upstream layers in Kubernetes cluster management stack using the Node Problem Detector [1], commonly known as NPD. Having an NPD daemon surely helps in detecting issues, but there haven't been any direct efforts to develop a rectifier to fix the detected issues to the best of our knowledge. 

Keeping this in mind, we propose a self-healing Kubernetes cluster which makes use of the NPD daemon to detect issues at every node running in the cluster. The NPD daemon would then try to initially fix the issue at the node itself. This fix would be done with the help of another daemon running at every node which tries to repair broken nodes by restarting daemons, or any other kinds of in-node fixes. If it fails to fix the node after this, the NPD would pass events to a daemon running in the control plane of the Kubernetes cluster (master node) to replace the existing unhealthy node with a healthy one. Thus, by providing this two level healing mechanism - both from within and outside the node, we come up with this idea of a self-healing Kubernetes cluster that tries to repair/replace any unhealthy nodes.

## Proposed Architecture
![Architecture Diagram Goes Here](/architecture.png?raw=true "Self healing Kubernetes cluster")

The following is an explanation of the illustration of the repair mechanism depicted above
1. The problem is first detected at the node by the **Node Problem Detector (NPD)**  daemon. The NPD does this by scanning through various important log files (like syslog, docker.log, kubelet.log etc) to detect when a system goes to an unhealthy state.
2. The NPD then passes the detected problem to the **Daemon Rectifier**.
3. The daemon rectifier tries to fix the from from inside the node by restarting relavent daemons, or fix up system issues.
4. If the daemon rectifier fails to fix the issue, the NPD signals to the in-cluster rectifier called the **Node Rectifier** about the presence of an unhealthy node.
5. The node rectifier then marks this node as unschedulable to avoid scheduling of pods on this unhealthy node.
6. Node rectifier then requests the cloud provider through its APIs to create a similar new node to replace the unhealthy node.
7. Once the new node is ready, all executing resources from the unhealthy node is drained and this load is tried to be pushed on to the newly created node.
8. Finally, the node rectifier deletes the unhealthy node.

The above described steps are a continuous process and in this way, the NPD daemon running on each node in the cluster tries to keep all nodes in the cluster in an healthy state.

## References
1. [Node Problem Detector](https://github.com/kubernetes/node-problem-detector)
