# lockable-resource

**Table of Contents** _generated with_ [_DocToc_](https://github.com/thlorenz/doctoc)

* [Lockable Resource Manager](lockable-resource.md#lockable-resource-manager)
  * [get info](lockable-resource.md#get-info)
    * [get all](lockable-resource.md#get-all)
    * [Get resource by label](lockable-resource.md#get-resource-by-label)
    * [if label validated](lockable-resource.md#if-label-validated)
    * [Get free number of label](lockable-resource.md#get-free-number-of-label)
    * [Get all resource](lockable-resource.md#get-all-resource)
  * [add or remove](lockable-resource.md#add-or-remove)
  * [management](lockable-resource.md#management)
    * [remove by label \(or name\)](lockable-resource.md#remove-by-label-or-name)
    * [create new item](lockable-resource.md#create-new-item)
  * [reserve & unlock](lockable-resource.md#reserve--unlock)
    * [by cli](lockable-resource.md#by-cli)
    * [by api](lockable-resource.md#by-api)
    * [examples](lockable-resource.md#examples)
  * [functions](lockable-resource.md#functions)
    * [`isLabelExists`](lockable-resource.md#islabelexists)
    * [`isResourceExists`](lockable-resource.md#isresourceexists)
    * [get label by name](lockable-resource.md#get-label-by-name)
    * [set Label](lockable-resource.md#set-label)
    * [with Closure](lockable-resource.md#with-closure)
  * [reference](lockable-resource.md#reference)

## Lockable Resource Manager

### get info

#### get all

**all items information**

```groovy
import org.jenkins.plugins.lockableresources.LockableResourcesManager

LockableResourcesManager manager = new org.jenkins.plugins.lockableresources.LockableResourcesManager()
// or LockableResourcesManager manager = org.jenkins.plugins.lockableresources.LockableResourcesManager.get()

manager.getResources().each{ r ->
  println """
    ${r.name}: ${r.getClass()}
              locked? : ${r.locked} : ${r.isLocked()}
            reserved? : ${r.reserved} : ${r.isReserved()}
                label : ${r.labels} : ${r.getLabels()}
          description : ${r.description} : ${r.getDescription()}
                queue : ${r.queueItemProject ?: ''}
          reserved by : ${r.reservedBy ?: ''}
                build : ${r.build ?: ''}
       queuingStarted : ${r.queuingStarted ?: ''}
       queuedContexts : ${r.queuedContexts ?: ''}

"""
}
```

**labels**

```groovy
import org.jenkins.plugins.lockableresources.LockableResourcesManager

stage('all label') {
  println '~~> all labels:'
  println new LockableResourcesManager().getAllLabels()
}
```

**resources**

```groovy
import org.jenkins.plugins.lockableresources.LockableResourcesManager
println new LockableResourcesManager().getResources()
```

* or

  ```groovy
  println org.jenkins.plugins.lockableresources.LockableResourcesManager.get().getResources()
  ```

#### Get resource by label

```groovy
import org.jenkins.plugins.lockableresources.LockableResourcesManager

stage('get label') {
  String l = 'my-label'
  println "~~> resources for ${l}:"
  println new LockableResourcesManager().getResourcesWithLabel( l, null )
}
```

* or

  ```groovy
  import org.jenkins.plugins.lockableresources.LockableResourcesManager
  println new LockableResourcesManager().getResourcesWithLabel( l, [:] )
  ```

#### if label validated

```groovy
import org.jenkins.plugins.lockableresources.LockableResourcesManager

stage('does label validated') {
  String l = 'my-label'
  println '~~> is ${l} valid:'
  println new LockableResourcesManager().isValidLabel(l)
}
```

#### Get free number of label

```groovy
import org.jenkins.plugins.lockableresources.LockableResourcesManager

stage('number of free') {
  String l = 'my-label'
  println new LockableResourcesManager().getFreeResourceAmount(l)
}
```

#### [Get all resource](https://issues.jenkins-ci.org/browse/JENKINS-46235?focusedCommentId=345401&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-345401)

```groovy
stage('get all resoruces') {
  def all_lockable_resources = GlobalConfiguration.all().get(org.jenkins.plugins.lockableresources.LockableResourcesManager.class).resources

  println "~~> free resource for ${l}"
  println all_lockable_resources
  // remove
  all_lockable_resources.removeAll { it.name.contains('somestr')}
}
```

### add or remove

### management

#### [remove by label \(or name\)](https://issues.jenkins-ci.org/browse/JENKINS-38906?focusedCommentId=353245&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-353245)

```groovy
stage('remove') {
    def manager = org.jenkins.plugins.lockableresources.LockableResourcesManager.get()
    def resources = manager.getResources().findAll {
        // println it.locked ? "${it} locked" : "${it.labels}"
        ( !it.locked ) && (
            it.name.equals('marslo') ||
            it.labels.contains('marslo') ||
            it.name.startsWith('marslo')
        )
    }
    currentBuild.description = "${resources.size()} locks"
    resources.each {
        println "Removing ${it.name} ~~> ${it.labels}"
        manager.getResources().remove(it)
    }
    manager.save()
}
```

* [remove all](https://issues.jenkins.io/browse/JENKINS-38906?focusedCommentId=358692&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-358692)

  ```groovy
  String lockName = 'lock name'
  def manager = org.jenkins.plugins.lockableresources.LockableResourcesManager.get()
  manager.getResources().removeAll { r -> lockNames.contains(r.name) && !r.locked && !r.reserved }
  manager.save()
  ```

#### create new item

```groovy
stage('create') {
    def manager = org.jenkins.plugins.lockableresources.LockableResourcesManager.get()
    def myr = manager.createResourceWithLabel('marslo', 'marslo-label')
}
```

### reserve & unlock

#### by cli

```bash
$ resource='marslo'
$ curl -XGET -uadmin:passwd https://my.jenkins.com/lockable-resources/reserve?resource=${resource}
$ curl -XGET -uadmin:passwd https://my.jenkins.com/lockable-resources/unreserve?resource=${resource}
```

#### by api

```groovy
stage('reserve & unlock') {
    def manager = org.jenkins.plugins.lockableresources.LockableResourcesManager.get()
    println manager.fromName('marslo')?.isReserved()

    println '~~> lock marslo:'
    manager.reserve([ manager.fromName('marslo') ], 'Marslo Jiao')
    println manager.fromName('marslo')?.isReserved()

    println '~~> unlock marslo:'
    manager.reset([ manager.fromName('marslo') ])
    println manager.fromName('marslo')?.isReserved()

}
```

#### [examples](https://config9.com/apps/jenkins/jenkins-lockable-resource-lock-without-unlocking/)

```groovy
print "START\n"
def all_lockable_resources = org.jenkins.plugins.lockableresources.LockableResourcesManager.get().resources
all_lockable_resources.each { r->
  if (r.isLocked() || r.isReserved()) {
  println "Lock " + r + " is locked or reserved by " + r.getBuild() + " B CARSE " + r.getLockCause()

  b = r.getBuild()

    if (b) {
      if (b.isBuilding()) { println "build:" + b + " is building" }
      if (b.getResult().equals(null)) { println "build:" + b + " result is not in yet" }

      if ( ! b.isBuilding() && ! b.getResult().equals(null)) {
        println "build:" + b + " is not building and result is " + b.getResult() + " yet the lock " + r + " is locked."
        println "ACTION RELEASE LOCK " + r

        println "getLockCause:" + r.getLockCause()
        println "getDescription:" + r.getDescription()
        println "getReservedBy:" + r.getReservedBy()
        println "isReserved:" + r.isReserved()
        println "isLocked:" + r.isLocked()
        println "isQueued:" + r.isQueued()

        //release the lock
        r.reset()

        println "getLockCause:" + r.getLockCause()
        println "getDescription:" + r.getDescription()
        println "getReservedBy:" + r.getReservedBy()
        println "isReserved:" + r.isReserved()
        println "isLocked:" + r.isLocked()
        println "isQueued:" + r.isQueued()
      }

    }
  }
}
```

### functions

#### `isLabelExists`

```groovy
def isLabelExists( String label ) {
  org.jenkins.plugins
     .lockableresources
     .LockableResourcesManager
     .get()
     .getResources()
     .findAll{ it.labels == label } != []
}
```

* or

  ```groovy
  withManager{ manager ->
    manager.getResources()
           .findAll{ it.labels == label } != []
  }
  ```

#### `isResourceExists`

```groovy
def isResourceExists( String name ) {
  org.jenkins.plugins
     .lockableresources
     .LockableResourcesManager
     .get()
     .getResources()
     .findAll{ it.name == name } != []
}
```

* or

  ```groovy
  withManager{ manager ->
    manager.getResources()
           .findAll{ it.name == name } != []
  }
  ```

#### get label by name

```groovy
def getLabelByName( String name ) {
  org.jenkins.plugins
     .lockableresources
     .LockableResourcesManager
     .get()
     .getResources()
     .findAll{ it.name == name }
     .collect{ it.labels }
     .join(' ')
}
```

* or

  ```groovy
  withManager{ manager ->
    manager.getResources()
           .findAll{ it.name == name }
           .collect{ it.labels }
           .join(' ')
  }
  ```

#### set Label

```groovy
def setLabel( String name, String label, String description = '' ) {
  LockableResourcesManager manager = LockableResourcesManager.get()
  description = ( description ? "${description} | " : '' ) + "created by Jenkins job ${env.BUILD_URL} automatically"
  if ( isResourceExists(name) ) {
    util.showError( "ERROR: resource ${name} has already exists in resource pool. Exit..." )
  } else {
    manager.createResourceWithLabel( name, label )
    manager.resources.findAll{ name == it.name }
                          .each{ it.setDescription(description) }
    manager.save()
    if ( ! isResourceExists(name) ) util.showError( "ERROR: resource ${name} failed to be added in resource pool. Exit..." )
  }
}
```

#### with Closure

```groovy
import org.jenkins.plugins.lockableresources.LockableResourcesManager

def withManager( Closure body ) {
  LockableResourcesManager manager = org.jenkins.plugins
                                                .lockableresources
                                                .LockableResourcesManager
                                                .get()
  body( manager )
}
```

### reference

* [javadoc](https://javadoc.jenkins.io/plugin/lockable-resources/org/jenkins/plugins/lockableresources/LockableResource.html)
* [configure-lockable-resources.groovy](https://gist.github.com/ansig/d7edafe38fbfc13c5b3cd15351849804)
* [collect-resources-data-for-graphite.groovy](https://gist.github.com/ansig/c9f2ac8e291d5dcb854d49f691f6c7e8)
* [LockableResourcesHelper.groovy](https://gist.github.com/glance-/aaa3c037757895798d4e1be5134bb843)
* [lockable\_resources\_from\_json.groovy](https://gist.github.com/evidex/520d7a096929bdda1779a51e380819be)
* [list\_lockable\_resources.groovy](https://gist.github.com/evidex/925fbc47a871141070b81e7dbbcf713f)
* [JenkinsJobDslCleanupLockableResources.groovy](https://gist.github.com/marcusphi/3380f83964249b1250a6d5230be741e5)
* [JenkinsScriptedPipelineCleanupLockableResources.groovy](https://gist.github.com/marcusphi/d5a5d5769b69627a5169dc440d52a223)
* [jenkins-remove-lockable-resources.groovy](https://gist.github.com/hrickmachado/86683781f700ea2b5a5012ad1d22b54)
