# kubist

01 Mongo on Kubernetes
=====

This runs AWS, but nothing is specific to it.  Should run anywhere.

Setup
===

1. Spin up kubernetes cluster (http://kubernetes.io/docs/getting-started-guides/aws/)
1. Label the minions
	
	a. Minion 1: node=mongo-db1, arbiter=true
	a. Minion 2: node=mongo-db1, arbiter=true
````	
	add example
````
1. Create /data/db directories on Minion 1 an 2
1. Create pods, replication controllers and services from 01-mongo-arbiter-demo dir
````
kubectl create -f ./create
````
1. Check to make sure everything is running
````
kubectl get pods
kubectl get rc
kubectl get svc
````
1. Configure mongo replication set
	a. ssh  to  Minion 1
	a. open mongo on the MongoDB container
	a. initiate the replica set
````
rs.initiate(
{
        "_id" : "demoRS",
        "version" : 1,
        "protocolVersion" : NumberLong(1),
        "members" : [
                {
                        "_id" : 0,
                        "host" : "mongo1-svc.default.svc.cluster.local:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ]})
````
	a. add the second server
````
rs.add("mongo2-svc.default.svc.cluster.local:27017")		
````
	a. add the arbiter
````
rs.addArb("arbiter-svc.default.svc.cluster.local:27017")
````
	a. check status (you should see there healthy entries (primary, secondary and arbiter)
````
rs.status()
````




Experiments
====
1.  We are going to simulate a node going down.  We are going to chose the node running both mongo and the arbiter.  We expect the following:
	a. Kubernetes detects that one of the mongo pods is gone and that the arbiter is pod is down as well
	a. MongoDB changes remaining healthy node to secondary since it lost the majority
	a. Kubernetes replications controller spins up a new arbiter pod on the healthy node
	a. MongoDB detect the arbiter and the healthy node is elected primary.  
*** Note: detections can take quite a long time.  It's possible to speed that up by hitting the services, for example with curl.
 
To remove thie node stop kubelet and docker services:
````
for SERVICES in  kubelet docker; do
   sudo systemctl stop $SERVICES
   sudo systemctl disable $SERVICES
   sudo systemctl status $SERVICES
done
````
To restore the node, restart kubelet and docker services:
````
for SERVICES in docker kubelet; do
   sudo systemctl start $SERVICES
   sudo systemctl enable $SERVICES
   sudo systemctl status $SERVICES
done
````
To check replica set, connect to the health node with mongo and run rs.status()

