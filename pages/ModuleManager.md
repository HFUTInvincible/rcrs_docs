- We use [[ModuleManager]] to manage instances of [[AbstractModule]] and [[ExtAction]].
-
  ``` java
  // get module
  this.pathPlanning = moduleManager.getModule("PoliceTargetAllocator.PathPlanning",
                                         "HFUTInvincibleRescue.algorithm.AStarPathPlanning");
  // get instance of ExtAction
  this.actionExtMove = moduleManager.getExtAction(
            "CommandExecutorFire.ActionExtMove",
            "adf.sample.extaction.ActionExtMove" );
  ```