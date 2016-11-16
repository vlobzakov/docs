# Events

Any action, available to be performed by means of API (including custom users’ scripts running), should be bound to some event, i.e. executed as a result of this event occurrence.
Each event refers to a particular entity. For example, the entry point for executing any action with application is the onInstall event.
<br>
<br>
Subscription to a particular application lifecycle event (e.g. topology changing) can be done via [Environment Level Events](#environment-level-events).
It’s also possible to bind extension execution to the onUninstall event - in such a way, you can implement custom logic of this extension removal from an environment.

## Application Level Events
```
{
  "jpsType": "update",
  "application": {    
    "onInstall": {},
    "onUninstall": {}
  }
}
```

### onInstall
### onUninstall

## Environment Level Events
```
{
  "jpsType": "update",
  "application": {
     "env" : {
        "onBeforeChangeTopology": {},
        "onAfterChangeTopology": {},
        "onBeforeRestartNode" : {},
        "onAfterRestartNode" : {},
        "onBeforeDelete" : {},
        "onAfterDelete" : {},
        "onBeforeAddNode" : {},
        "onAfterAddNode" : {},
        "onBeforeCloneNodes" : {},
        "onAfterCloneNodes" : {},
        "onBeforeLinkNode" : {},
        "onAfterLinkNode" : {},
        "onBeforeAttachExtIp" : {},
        "onAfterAttachExtIp" : {},
        "onBeforeDetachExtIp" : {},
        "onAfterDetachExtIp" : {},
        "onBeforeUpdateVcsProject" : {},
        "onAfterUpdateVcsProject" : {},
        "onBeforeSetCloudletCount" : {},
        "onAfterSetCloudletCount" : {},
        "onAfterChangeEngine" : {},
        "onBeforeChangeEngine" : {},
        "onBeforeStart" : {},
        "onAfterStart" : {},
        "onBeforeStop" : {},
        "onAfterStop" : {},
        "onBeforeClone" : {},
        "onAfterClone" : {},
        "onBeforeDeploy" : {},
        "onAfterDeploy" : {},
        "onBeforeResetNodePassword" : {},
        "onAfterResetNodePassword " : {},
        "onBeforeRemoveNode" : {},
        "onAfterRemoveNode" : {},
        "onBeforeRestartContainer" : {},
        "onAfterRestartContainer" : {},        
        "onBeforeMigrate" : {},        
        "onAfterMigrate" : {},
        "onBeforeRedeployContainer" : {},        
        "onAfterRedeployContainer" : {},  
        "onBeforeLinkDockerNodes" : {},        
        "onAfterLinkDockerNodes" : {},
        "onBeforeUnlinkDockerNodes" : {},        
        "onAfterUnlinkDockerNodes" : {},
        "onBeforeSetDockerEnvVars" : {},        
        "onAfterSetDockerEnvVars" : {},
        "onBeforeSetDockerEntryPoint" : {},        
        "onAfterSetDockerEntryPoint" : {},
        "onBeforeSetDockerRunCmd" : {},        
        "onAfterSetDockerRunCmd" : {},
        "onBeforeAddDockerVolume" : {},        
        "onAfterAddDockerVolume" : {},
        "onBeforeRemoveDockerVolume" : {},        
        "onAfterRemoveDockerVolume" : {}                                                                                                                                                                                                        
     }
  }
}
```                              

Events `onInstall`, `onUninstall`, `onBeforeDelete`, `onAfterDelete` can be executed once. Other events can be used more then one time.

### BeforeChangeTopology
### AfterChangeTopology
### BeforeRestartNode
### AfterRestartNode
### BeforeDelete
### AfterDelete
### BeforeAddNode
### AfterAddNode
### BeforeCloneNodes
### AfterCloneNodes
### BeforeLinkNode
### AfterLinkNode
### BeforeAttachExtIp
### AfterAttachExtIp
### BeforeDetachExtIp
### AfterDetachExtIp
### BeforeUpdateVcsProject
### AfterUpdateVcsProject
### BeforeSetCloudletCount
### AfterSetCloudletCount
### AfterChangeEngine
### BeforeChangeEngine
### BeforeStart
### AfterStart
### BeforeStop
### AfterStop
### BeforeClone
### AfterClone
### BeforeDeploy
### AfterDeploy
### BeforeResetNodePassword
### AfterResetNodePassword 
### BeforeRemoveNode
### AfterRemoveNode
### BeforeRestartContainer
### AfterRestartContainer
### BeforeMigrate
### AfterMigrate
### BeforeRedeployContainer
### AfterRedeployContainer
### BeforeLinkDockerNodes
### AfterLinkDockerNodes
### BeforeUnlinkDockerNodes
### AfterUnlinkDockerNodes
### BeforeSetDockerEnvVars
### AfterSetDockerEnvVars
### BeforeSetDockerEntryPoint
### AfterSetDockerEntryPoint
### BeforeSetDockerRunCmd
### AfterSetDockerRunCmd
### BeforeAddDockerVolume
### AfterAddDockerVolume
### BeforeRemoveDockerVolume
### AfterRemoveDockerVolume

