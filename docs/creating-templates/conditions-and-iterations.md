#Control flows

##Iterations
<b>ForEach.</b>
Iterable object map:

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
- `settings` - fields values predefined at [user setting form](http://docs.cloudscripting.local/creating-templates/user-input-parameters/) (Optional).
- `license` - parameters from prepopulate action (Optional).
- `event` - object of parameters from [events](http://docs.cloudscripting.local/reference/events/). This parameters separate on before and after event parameters (Optional). So [actions](/reference/actions/) can be used before and after executing event.
- `this` - parameters object is sent with procedures name(Optional). Mode details about `this` parameter is [here](/reference/placeholders/#procedure-placeholders)

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

- `env.contexts` - deployed in environment contexts
- `env.extdomains` - binded external domains 

Full placeholders list [here](/reference/placeholders/)

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
In `execCmd` action compute nodes addresses are rewrited in balancer node and reload nginx service.
Events are `onAfterScaleIn` and `onAfterScaleOut` will executed after add or remove compute node.

###Iteration by all nodes in environment:

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

###Iteration by compute nodes with custom iterator name:
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

- `@cp` - custom iterator name (optional)

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
- `@` iterator number

In this case every environment node will have only one conjunction by nodeId.
