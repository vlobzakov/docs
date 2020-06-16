## Auto-Clustering
In Jelastic the following *nodeTypes* can be clusterized with help of built-in **Auto-Сlustering** feature:  
  * *mysql*, *mariadb-dockerized*, *postgresql*
  * *glassfish*, *wildfly*, *payara*  
  * *couchbase*, *mongodb-dockerized* 

*Auto-Clustering* can be enabled either using *Auto-Clustering* switch at the dashboard:  
![autoclustering-switch](/img/autoclustering-switch.png)  
or via jps manifest with `cluster` parameter.  

### cluster parameter
To enable *Auto-Clustering* the `cluster` parameter is used as:  
  * *boolean* value - *true* invokes cluster creation with default configuration parameters  
  
  !!!note  
  
    * Default topology that will be created for the nodeTypes *mysql* and *mariadb-dockerized* is **[master-slave](https://jelastic.com/blog/mysql-mariadb-database-auto-clustering-cloud-hosting/)** replication cluster with 2 nodes of HA ProxySQL load balancer in front of.  
    * In case of nodeType *postgresql* there is only one topology available - **[master-slave](https://jelastic.com/blog/postgresql-auto-clustering-master-slave-replication/).  
    * The *wildfly* cluster is created in [Managed Domain Mode](https://jelastic.com/blog/wildfly-managed-domain-in-containers-auto-micro-clustering-and-scaling/) with topology that comrises one Domain Controller node and Worker nodes. Number of Worker nodes is defined by *[count](basic-configs/#nodes-definition)* parameter.  
    * The *payara*/*glassfish* cluster is created with topology that comrises one [DAS node and Worker nodes](https://jelastic.com/blog/glassfish-payara-auto-clustering-cloud-hosting/). Number of Worker nodes is defined by *[count](basic-configs/#nodes-definition)* parameter.  
    * The *mongodb* cluster is created as **[replica-set](https://jelastic.com/blog/mongodb-replica-set-master-slave-failover/)** with topology that comrises tree nodes one *Primary* and two *Secondary* nodes.      
   
For example:  

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

### validation 
      
The validation parameter allows to:  
  * specify maximum number of nodes in the cluster
  * specify minimum number of nodes in the cluster  
    
Once the validation parameters were aplied to respective environment parameters via jps manifest, you won't be able to change them in the wizard.  
 
Following example shows how to restrict a scaling limit of worker nodes between 3 and 5 for Payara Cluster:  

@@@
```yaml
type: install
name: Validation
nodes:
 nodeType: payara
 cloudlets: 6
 count: 3
 validation:
   minCount: 3
   maxCount: 5
 cluster: true
```
```json
{
  "type": "install",
  "name": "Validation",
  "nodes": {
    "nodeType": "payara",
    "cloudlets": 6,
    "count": 3,
    "validation": {
      "minCount": 3,
      "maxCount": 5
    },
    "cluster": true
  }
}
```
@@!  

Respectively trying to decrease below 3 the number of worker nodes in the wizard the corresponding warning will be displayed:  

![validation-min](/img/validation-min.png)
  
  
## Auto-Clustering Configuration

In this chapter, we consider as an example of how auto-clustering is implemented for popular software stack MySQL in order to provide you with all necessary information to create your own auto-clustering solution of the software stack you want.

 The steps to create own auto-clustering solution are listed below. Actually the process is not different of how to build and publish any software stack template. The complete process is explained in detail in our [tutorial](https://jelastic.com/blog/build-software-stack-container-image-private-paas/) and can be taken as an example.:
   1. Create cluster configuration file in JSON format e.g. https://github.com/jelastic-jps/mysql-cluster/blob/master/addons/mysql-cluster.json.
   2. Create a Docker file to build your custom template. e.g.: 
     
     FROM DOCKER_BASE:mysqldblayer  
     LABEL name="MySQL CE"
     LABEL nodeType=mysql
     LABEL nodeTypeAlias=mysqlAPP_MAJOR_VERSION 
     LABEL cluster="https://raw.githubusercontent.com/jelastic-jps/mysql-cluster/master/addons/mysql-cluster.json"  
     
   where the `cluster` label is a mandatory parameter that should be added to the Dockerfile and must contain a URL to the cluster configuration file.
   3. Push image to the Docker Hub.
   4. Publish image as template in JCA.
   5. The published template with auto-cluster functionality should be available at Jelastic Dashboard.
 
 Let's take a closer look at the config file to realize what parameters can be changed if necessary.
 
   

