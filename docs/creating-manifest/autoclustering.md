## Auto-Clustering
In Jelastic the following *nodeTypes* can be clusterized with help of built-in **Auto-Сlustering** feature:  
  * *mysql*, *mariadb*
  * *glassgish*, *wildgly*, *payara*  
  * *couchbase* 
  * *storage*

*Auto-Clustering* can be enabled either using *Auto-Clustering* switch at the dashboard:  
![autoclustering-switch](/img/autoclustering-switch.png)  
or via jps manifest with `cluster` parameter.  

### cluster parameter
To enable *Auto-Clustering* the `cluster` parameter is used as:  
  * *boolean* value - *true* invokes cluster creation with default configuration parameters  
   (default topology that will be created for *mysql*/*mariadb* is **master-slave** replication cluster with 2 nodes of HA ProxySQL load balancer in front of):  
   
@@@
```yaml
type: install
name: Auto-Cluster

nodes:
  - nodeType: payara
    cluster: true
    cloudlets: 6
    count: 4
```
```json
{
  "type": "install",
  "name": "Auto-Cluster",
  "nodes": [
    {
      "nodeType": "payara",
      "cluster": true,
      "cloudlets": 6,
      "count": 4
    }
  ]
}
```
@@!  
  
  
![autoclustering-switch](/img/autoclustering-cluster-default.png)  
 
  * *object* - this is applicable for *mysql*/*mariadb* `nodeType` only. Object contains multiple options can be passed as configuration paramters:   
    * `scheme` *[optional]* - configures database [replication scheme](https://jelastic.com/blog/mysql-mariadb-database-auto-clustering-cloud-hosting/) for:  
      * `mysql` - **slave**(Master-Slave), **master**(Master-Master), **single**(Single Primary Group Replication), **multi**(Multi Primary Group Replication)  
      * `mariadb` - **slave**(Master-Slave), **master**(Master-Master), **galera**(Galera Cluster)  
    * `is_proxysql` *[optional][boolean]* - *'true'* adds a **proxysql** load balancer layer to the topology and configures it as an entry point to the database cluster  
    * `db_user` *[optional]* - sets up a database username. If not defined the system will generate one by default  
    * `db_pass` *[optional]* - sets up a password for `db_user`. If not defined the system will generate one by default  
    * `jps` *[optional]* - overrides default cluster configuration steps with custom ones stated in the jps manifest  
      * `settings` - provides user's configuration parameters to the custom `jps` manifest
      

Auto-Clustering examples.  

*Master-Master* replication topology with ProxySQL node as the entry point:  
  
@@@
```yaml
type: install
name: Auto-Cluster

nodes:
  - nodeType: mysql
    cluster:
      scheme: master
      is_proxysql: true
      db_user: "mydbuser"
      db_pass: "mydbpasswd"
    cloudlets: 6
    count: 2
```
```json
{
  "type": "install",
  "name": "Auto-Cluster",
  "nodes": [
    {
      "nodeType": "mysql",
      "cluster": {
        "scheme": "master",
        "is_proxysql": true,
        "db_user": "mydbuser",
        "db_pass": "mydbpasswd"
      },
      "cloudlets": 6,
      "count": 2
    }
  ]
}
```
@@!  


      
      
 
    

