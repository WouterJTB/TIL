# Spark on k8s

## Resource management

- Drivers can not span multiple nodes
- Ideally, the Spark Executor resources (including overhead) are a divisor of the allocatable k8s node resources, so that you don't leave unused resources on a node (if this holds true for all jobs in the nodepool, resources can still be allocated somewhat efficiently). AWS has a [useful blog on this](https://aws.amazon.com/blogs/containers/optimizing-spark-performance-on-kubernetes/).
- Drivers and executors [can be assigned to a different nodepool](https://github.com/kubeflow/spark-operator/issues/1471). I.e. you can run executors on a spot instance nodepool while running drivers on on-demand instances.
- For (now only on) AWS, consider [Karpenter](https://karpenter.sh/)
- Few/fat nodes/executors vs. many/small nodes/executors:
  - Many/small nodes/executors still need to communicate, communication between driver/executors on the same pod will be fast (near instant). Communication between nodes will have a (likely very minor) latency cost.
  - Many/small nodes/executors should be more fault tolerant (less work lost if it crashes).
  - Take into account parellelization of the job vs number of shuffles.
