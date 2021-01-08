# kube_vertx_demo
## A Kubernetes, Vert.x/Hazelcast/vxms demo
This is an event driven microservice demo using three Vertx instances, each running in a separate kubernetes pod (rc & service). The instances are clustered using the hazlecast kubernetes plugin (currently the REST endpoint). To run the demo, a running kubernetes (with DNS enabled) is assumed (http://kubernetes.io/docs/hellonode/).
The demo application is a small user-administration in AngularJs. The frontend-verticle contains the UI and a vxms REST Endpoint (the aggregator) (https://github.com/amoAHCP/vxms). The read requests will be passed to the read-verticle using the Vert.x event bus (clustered Hazelcast), and the write request to the write-verticle. All instances are using the same mongodb deployment running in the same kubernetes cluster.

## run the demo
1. run the "create_mongoDB.sh" script to create a mongodb replication controller and the corresponding service. The other container will acces the mongodb by DNS name.
2. run "build_frontend_image.sh -v $VERSION -p $PROJECTNAME -d true", this will create the docker image, build the maven project and upload it to your gcloud docker repository. When finished, do the same with build_read_image.sh & build_write_image.sh
3. create the services by executing "create_services.sh"
4. create the replication controllers by executing "create_controllers.sh"

When you type "kubectl get services" you should see something like that:

| NAME                | CLUSTER_IP     | EXTERNAL_IP     | PORT(S)     | SELECTOR                 | AGE |
|---------------------|:--------------:|:---------------:|:-----------:|:------------------------:|----:|
|frontend-verticle   |10.3.250.98    |146.148.3.148   |80/TCP      |name=frontend-verticle   |10s|
|kubernetes          |10.3.240.1     |<none>          |443/TCP     |<none>                   |12d|
|mongo               |10.3.245.198   |<none>          |27017/TCP   |name=mongo               |40s|
|read-verticle       |10.3.243.101   |<none>          |5701/TCP    |name=read-verticle       |10s|
|write-verticle      |10.3.255.80    |<none>          |5701/TCP    |name=write-verticle      |10s|

The frontend-verticle gets an external ip address (it may take some minutes until the external ip appears) and should be now accessible in the browser (the demo uses port 80).

- To see all running pods, type: "kubectl get pods"

| NAME                                 | READY     | STATUS    | RESTARTS |
|---------------------|:--------------:|:---------------:|:-----------:|----:|
|frontend-verticle-controller-aqe2n   |1/1       |Running   |0|         
|mongo-controller-heyhc               |1/1       |Running   |0|         
|read-verticle-controller-8rmfl       |1/1       |Running   |0|         
|write-verticle-controller-3pj73      |1/1       |Running   |0|          

- To get the logs of a pod type: "kubectl logs write-verticle-controller-3pj73".
- To see all running replication controller, type: "kubectl get rc"
- To scale the frontend-container type: "kubectl scale rc frontend-verticle-controller --replicas=2"
