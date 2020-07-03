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
  * set up a *[scalilngMode](https://docs.cloudscripting.com/creating-manifest/basic-configs/#nodes-definition)* parameter for the layer
    
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

The `validation` field is not tied to a clustering feature and can be used for any other templates.
  
  
## Auto-Clustering Configuration

In this chapter, we consider as an example of how auto-clustering is implemented in order to provide you with all necessary information to create your own auto-clustering solution of the software stack you want.

 The steps to create own auto-clustering solution are listed below. Actually the process is not different of how to build and publish any software stack template. The complete process is explained in detail in our [tutorial](https://jelastic.com/blog/build-software-stack-container-image-private-paas/) and can be taken as an example.
   1. Create cluster configuration file in JSON format e.g. https://github.com/jelastic-jps/mysql-cluster/blob/master/addons/mysql-cluster.json.
   
   ```
   {
  "jps": "https://raw.githubusercontent.com/jelastic-jps/mysql-cluster/master/addons/auto-clustering/auto-cluster.jps",
  "defaultState": true,
  "nodeGroupData": {
    "scalingMode": "NEW"
  },
  "compatibleAddons": ["mysql-auto-cluster"],
  "settings": {
    "data": { "scheme": "mm" },
    "fields": [
      {
        "type": "list",
        "caption": "Scheme",
        "name": "scheme",
        "values": {
          "mm": "Master - Master",
          "ms": "Master - Slave"
        }
      }
    ]
  },
  "validation": {
    "minCount": 2,
    "scalingMode": "NEW",
    "rules": {
      "scheme": {
        "mm": {
          "maxCount": 2
        }
      }
    }
  },
  "description": "<p>Ready-to-work scalable MySQL Cluster with master-slave asynchronous replication and ProxySQL load balancer in front of it. Is supplied with embedded Orchestrator GUI for convenient cluster management and provides even load distribution, slaves healthcheck and autodiscovery of newly added DB nodes</p>"
}
```
   
   2. Create a Docker file to build docker image. e.g.: 
     
     FROM DOCKER_BASE:mysqldblayer  
     LABEL name="MySQL CE"
     LABEL nodeType=mysql
     LABEL nodeTypeAlias=mysqlAPP_MAJOR_VERSION 
     LABEL cluster="https://raw.githubusercontent.com/jelastic-jps/mysql-cluster/master/addons/mysql-cluster.json"  
     
   where the `cluster` label is a mandatory parameter that should be added to the Dockerfile and must contain a URL to the cluster configuration file.  
   3. Push image to the Docker Hub.  
   4. Publish image as template in JCA.  
   5. The published template with auto-cluster functionality should be available and may be used at Jelastic Dashboard.  
 
 Let's take a closer look at the config file to realize what parameters can be changed if necessary.
So, a new label `cluster` has been added to dockerized templates, which keeps a link to the auto-clustering config file in *json* format. The config is reloaded every time a template is updated or when re-imported manually.
Possible values:
  * *false* (default) - clustering is disabled
  * relative path to the label *sourceUrl* (example of sourceUrl: https://raw.githubusercontent.com/jelastic/icons/master/mysql/)
  * *true* - substitutes the relative path “jelastic/cluster.json” to the label “sourceUrl”
  * full URL to the config file.

Configuration file example:

```
{
  "convertible": false,
  "jps": "https://raw.githubusercontent.com/jelastic-jps/...",
  "defaultState": true,
  "required": true,
  "nodeGroupData": {
    "scalingMode": "NEW",
    "skipNodeEmails": true
  },
  "compatibleAddons": [
    "mysql-auto-cluster"
  ],
  "settings": {
    "data": {
      "scheme": "mm"
    },
    "fields": [
      {
        "type": "list",
        "caption": "Scheme",
        "name": "scheme",
        "default": "mm",
        "values": {
          "mm": "Master - Master",
          "ms": "Master - Slave"
        }
      }
    ]
  },
  "validation": {
    "minCount": 2,
    "scalingMode": "NEW",
    "extraNodesCount": 1,
    "rules": {
      "scheme": {
        "mm": {
          "maxCount": 2,
          "extraNodesCount": 3
        }
      }
    }
  },
  "description": "<p>Ready-to-work scalable MySQL...</p>",
  "skipOnEnvInstall": true,
  "skipOnEnvInstall": "http://...",
  "skipOnEnvInstall": ["http://...", "http://..."],
  "targetRegions": {
    "type": "vz7"  
  },
  "extraNodes": {
    "...": "..."  
  },
  "recommended": {
    "...": "..."  
  },
  "requires": ["..."]
}
```
Where: 
* `convertible`: false - this parameter determines whether the cluster can be enabled for the created layer (for example, GlassFish);
  * `jps` - link to the manifest;
  * `defaultState` - default state;
  * `required` - forced inclusion of the cluster (only on the wizard (!));
  * `nodeGroupData` - group node parameters;
    * `scalingMode` - if defined, it cannot be changed (only on the wizard);
    * `skipNodeEmails`: true - prevents sending letters about adding nodes;
  * `compatibleAddon`s - an array of “id” add-ons with *ON_ENV_INSTALL*: helps identify layers for which *ON_ENV_INSTALL* has already worked and indicate the corresponding cluster status at the UI. A similar check is on cloudscripting to avoid re-enabling the cluster;
  * `settings.data` - default settings for installing the cluster;
  * `settings.fields` - a list of additional fields/settings in cloudscripting format;
  * `validation` - validation settings (number of nodes in a layer, type of scaling, number of nodes in extra layers);
  * `description` - text for the tooltip that appears next to the Auto-Clustering field. It can be customized by writing a key in the localization: LT_EnvWizard_Tip_Cluster _% (nodeType). You can also set default text for the tooltip: LT_EnvWizard_Tip_Cluster_default;
  * `skipOnEnvInstall` - prevents the installation of a package with the ON_ENV_INSTALL environment variable. Possible values:
    * package URL
    * array of package URLs
    * true
  * `targetRegions` - [filter of regions](https://docs.cloudscripting.com/creating-manifest/basic-configs/#regions-filtering) in which autocluster can be used;
  * `extraNodes` - allows  displaying and edit extra cluster layers on the wizard;
  * `recommended` - displays additional notifications of recommended resources for the correct cluster operation;
  * `requires` - a list of unprotected *nodeType* templates used in the *extraNodes* field;

The cluster settings can be passed(set up or overridden) to Create/Change topology actions e.g.:

```
{
  "nodeGroup": "mysql",
  "cluster": {
    "jps": "http://clusters.com/mysql/manifest.jps",
    "settings": {
      "data": {
        "scheme": "mm"
      },
      "fields": {
        "name": "scheme",
        "default": "mm",
        "values": {
          "mm": "Master - Master",
          "ms": "Master - Slave"
        }
      }
    }
  }
}
```

By default, to enable auto-clustering, a config from a template (stack) is used using custom settings.
The default cluster config is replaced by the cluster object if the *jps* field is present in it (it was done for testing).
If the cluster option is not specified, and the options `defaultState` or `required` are *true* in the cluster config,  the auto-clustering will be enabled automatically.
Installing a package with [ON_ENV_INSTALL](https://docs.cloudscripting.com/creating-manifest/basic-configs/#environment-variables) is not performed if its value matches the link in the cluster config.

The new fields are added to the *nodeGroup* settings right after the layer is created and the cluster is installed:
  * `cluster` - cluster status and settings
  * `clusterTemplate` - the configuration from which the installation was performed (required for the user interface).
  
```
{
  "cluster": {
    "settings": {
      "scheme": "ms"
    },
    "jps": "c11e802f-a35f-4fa6-b666-24859df9f985",
    "enabled": true
  },
  "clusterTemplate": {
    "settings": {
      "fields": [
        {
          "default": "mm",
          "values": {
            "mm": "Master - Master",
            "ms": "Master - Slave"
          },
          "name": "scheme",
          "caption": "Scheme",
          "type": "list"
        }
      ]
    },
    "compatibleAddons": [
      "mysql-auto-cluster"
    ],
    "description": "<p>Ready-to-work scalable MySQL Cluster with master-slave asynchronous replication and ProxySQL load balancer in front of it. Is supplied with embedded Orchestrator GUI for convenient cluster management and provides even load distribution, slaves healthcheck and autodiscovery of newly added DB nodes</p>",
    "nodeGroupData": {
      "scalingMode": "NEW"
    }
  }
}
```

  !!!note
   
   All fields in the *nodeGroup* settings (*cluster*, *clusterTemplate*, *scalingMode*, *skipNodeEmails*) can be easily redefined/deleted by the user.
   
If auto-clustering was disabled when the layer was being created the switch is not present in the topology wizard, and the cluster parameter will be written as: *"cluster": {"enabled": false}* in the *nodeGroup* settings.

![cluster-disabled-by-creation](/img/cluster-disabled-by-creation.png)
  
### validation 
      
With respect to auto-clustering configuration the *validation* section allows to:
  * disable scaling (for example, for DAS nodes)
  * specify max/min number of nodes
  * set min number of cloudlets
  * specify min number of IPv4 addresses to be attached
  * specify value for *scalingMode*: NEW/CLONE
  * specify the number of nodes in additional layers (only when the cluster is turned on)

E.g. for MySQL:

```
{
  "validation": {
    "minCount": 2,
    "minCloudlets": 2,
    "minExtip": 1,
    "scalingMode": "NEW",
    "extraNodesCount": 2,
    "tag": "...",
    "rules": {
      "scheme": {
        "mm": {
          "maxCount": 2
        }
      }
    }
  }
}
```

In the *validation* main block the default values can be specified for  `minCount`, `maxCount`, `minCloudlets`, `minExtip`, `scalingMode`, `extraNodesCount`. If the `scalingMode` is present in `validation` section the value can't be changed in the wizard.  
The field `extraNodesCount` can be utilized in auto-clustering only. The all specified fields can be overidden in `rules` section depending on cluster settings.
