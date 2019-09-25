## Auto-Clustering
In Jelastic the following *nodeTypes* can be clusterized with help of built-in **Auto-Сlustering** feature:  
  * *mysql*, *mariadb*
  * *GlassFish*, *WildFly*  
  * *Couchbase*  

*Auto-Clustering* can be enabled either using *Auto-Clustering* switch at the dashboard:  
![autoclustering-switch](/img/autoclustering-switch.jpg)  
or via jps manifest with `cluster` parameter.  

### cluster parameter
To enable *Auto-Clustering* the **cluster** parameter is used as:  
  * *boolean* value - *true* invokes cluster creation with default configuration parameters  
   (default topology that will be created for *mysql*/*mariadb*: **master-slave** replication cluster with 2 nodes HA ProxySQL load balancer in front of)  
  * *object* - contains multiple options can be passed  
