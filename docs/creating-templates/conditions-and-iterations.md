#Control flows

##Iterations
<b>ForEach.</b>
The main iterable object is *ForEach*. Its map:

```
{
  "env": {
    "nodes": [],
    "contexts": [],
    "extdomains": []
  },
  "nodes": {},
  "settings": {},
  "license": {},
  "event": {
    "params": {},
    "response": {}
  },
  "this": {}
}
```
where    
- `settings [optional]` - fields values predefined within a [user setting form](http://docs.cloudscripting.com/creating-templates/user-input-parameters/)   
- `license [optional]` - link to fetch parameters specified within [prepopulate](http://docs.cloudscripting.com/creating-templates/user-input-parameters/) custom script. It enables to customize default field values and can be further initialized through [placeholders](http://docs.cloudscripting.com/reference/placeholders/) `$(license.{any_name}` within a manifest.   
- `event [optional]` - object entity with [event](http://docs.cloudscripting.com/reference/events/) parameters.  Can be of two types that allows initiation of a particular [action](http://docs.cloudscripting.com/reference/actions/) before and after event execution   
- `this [optional]` - parameters object to be transmitted within the procedure body. [See the full list of available placeholders](http://docs.cloudscripting.com/reference/placeholders/#procedure-placeholders) on this parameter.   

Iteration can be executed by `env.nodes`, `nodes`, `env.contexts` and `env.extdomains` objects:

```
{
  "forEach(env.extdomains)": [
    {
      "writeFile": {
        "nodeGroup": "cp",
        "path": "/var/lib/jelastic/keys/${@i}.txt",
        "body": "hello"
      }
    }
  ]
}
```
where    
- `@i` - default iterator name

```
{
  "forEach(env.contexts)": [
    {
      "writeFile": {
        "nodeGroup": "cp",
        "path": "/var/lib/jelastic/keys/${@i.context}.txt",
        "body": "1"
      }
    }
  ]
}
```
where  
- `env.contexts` -  list of contexts (applications) deployed to environment    
- `env.extdomains` - bound external domains 

[See the full placeholders list](/reference/placeholders/)

Scaling nodes example:
```
{
	"jpsType": "install",
	"application": {
		"name": "Scaling Example",
		"env": {
			"onAfterScaleIn[nodeGroup:cp]": {
				"call": "ScaleNodes"
			},
			"onAfterScaleOut[nodeGroup:cp]": {
				"call": "ScaleNodes"
			}
		},
		"procedures": [{
			"id": "ScaleNodes",
			"onCall": [{
				"forEach(nodes.cp)": {
					"execCmd": {
						"commands": [
							"{commands to rewrite all Compute nodes internal IP addresses in balancer configs. Here balancer node is NGINX}",
							"/etc/init.d/nginx reload"
						],
						"nodeGroup": "bl"
					}
				}
			}]
		}]
	}
}
```
As a result of execCmd, compute nodes internal IP addresses are rewritten within balancer configs and *NGINX* balancer node is reloaded. `onAfterScaleIn` and `onAfterScaleOut` events are executed immediately after adding or removing a compute node.

Nested conditions:   
  
###Iteration by all nodes in environment

```
{
  "forEach(env.nodes)": [{
    "execCmd": {
	  "nodeId": "${@i.id}",
		"commands": [
	      "echo ${@i.address} > /tmp/example.txt"
		]
	}
  }]
}
```

###Iteration by compute nodes with custom iterator name
```
{
  "forEach(cp:nodes.cp)": {
    "execCmd" : {
      "nodeId": "${@cp.id}",
      "nodeGroup": "${@cp.nodeGroup}",
      "nodeType": "${@cp.nodeType}",
      "commands": [
        "echo ${@cp.address} > /tmp/example.txt"
      ]
    }
  }
}
```
where 
- `@cp [optional]` - custom iterator name

Checking balancer stack type:
Custom iterator name can be used for nesting cycles one into another:
```
{
  "forEach(item:env.nodes)": [
    {
      "forEach(env.nodes)": [
        {
          "execCmd": {
            "nodeId": "${@i.id}",
            "commands": "[[ '${@i.id}' -eq '${@item.id}' ]] && touch /tmp/${@}.txt || touch /tmp/${@}${@}.txt"
          }
        }
      ]
    }
  ]
}
```
where
- `@` iterator number

In this case every environment node will have only one conjunction by node ID.