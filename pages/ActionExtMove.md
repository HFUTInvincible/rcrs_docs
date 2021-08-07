## Fields
	- [[PathPlanning]] pathPlanning
	- [[EntityID]] target
	- int thresholdRest
		- Decide whether the agent needs to rest.
		- Initialized in constructor
		-
		  ``` java
		  this.thresholdRest = developData.getInteger( "ActionExtMove.rest", 100 );
		  ```
		- Used in ((cea5625f-888a-42fb-bcb3-24366c202ef7))
	- int kernelTime
		- Initialized in precompute, resume, preparate
		-
		  ``` java
		  try {
		      this.kernelTime = this.scenarioInfo.getKernelTimesteps();
		  } catch ( NoSuchConfigOptionException e ) {
		      this.kernelTime = -1;
		  }
		  ```
		- Used in ((cea5625f-888a-42fb-bcb3-24366c202ef7))
	- [[HfutMessageTool]] messageTool
-
## Methods
	- setTarget()
		-
		  ``` java
		  @Override
		  public ExtAction setTarget(EntityID target) {
		  	this.target = null;
		      StandardEntity entity = this.worldInfo.getEntity(target);
		      if (entity != null) {
		  		if (entity.getStandardURN().equals(StandardEntityURN.BLOCKADE)) {
		          	entity = this.worldInfo.getEntity(((Blockade) entity).getPosition());
		        	} else if (entity instanceof Human) {
		          	entity = this.worldInfo.getPosition((Human) entity);
		        	}
		        	if (entity != null && entity instanceof Area) {
		          	this.target = entity.getID();
		        	}
		  		// Why not
		          // else if (entity instanceof Area) {
		          //     this.target = entity.getID();
		        	// }
		      }
		      return this;
		  }
		  ```
		-
		  ``` java
		  // What's the difference
		  entity.getStandardURN().equals( StandardEntityURN.BLOCKADE )
		  entity instanceof Blokade
		  ```
	- calc()
		-
		  ``` java
		  @Override
		  public ExtAction calc() {
		      this.result = null;
		      Human agent = (Human) this.agentInfo.me();
		  			  
		      if (this.needRest(agent)) {
		          this.result = this.calcRest(agent, this.pathPlanning, this.target);
		          if (this.result != null) {
		              return this;
		          }
		      }
		      if (this.target == null) {
		          return this;
		      }
		      this.pathPlanning.setFrom(agent.getPosition());
		      this.pathPlanning.setDestination(this.target);
		      List<EntityID> path = this.pathPlanning.calc().getResult();
		      if (path != null && path.size() > 0) {
		          this.result = new ActionMove(path);
		      }
		      return this;
		  }
		  ```
	- needRest( [[Human]] agent)
	  id:: cea5625f-888a-42fb-bcb3-24366c202ef7
		- Determine whether the agent need to rest.
		-
		  ``` java
		  private boolean needRest(Human agent) {
		      int hp = agent.getHP();
		      int damage = agent.getDamage();
		      if (hp == 0 || damage == 0) {
		          return false;
		      }
		      int activeTime = (hp / damage) + ((hp % damage) != 0 ? 1 : 0);
		      if (this.kernelTime == -1) {
		          try {
		              this.kernelTime = this.scenarioInfo.getKernelTimesteps();
		          } catch (NoSuchConfigOptionException e) {
		              this.kernelTime = -1;
		          }
		      }
		      return damage >= this.thresholdRest
		              || (activeTime + this.agentInfo.getTime()) < this.kernelTime;
		  }
		  ```
	- [[Action]] calcRest( [[Human]] human, [[PathPlanning]] pathPlanning, [[EntityID]] target) #FIXME
		-
		  ``` java
		  private Action calcRest(Human human, PathPlanning pathPlanning, EntityID target) {
		        EntityID position = human.getPosition();
		        // Why not use BFS?
		        Collection<EntityID> refuges = this.worldInfo
		                .getEntityIDsOfType(StandardEntityURN.REFUGE);
		        int currentSize = refuges.size();
		        if (refuges.contains(position)) { // Already in refuge, just rest!
		            return new ActionRest();
		        }
		        // Try to find a path to nearest refuge
		        List<EntityID> firstResult = null;
		        while (refuges.size() > 0) {
		            pathPlanning.setFrom(position);
		            pathPlanning.setDestination(refuges);
		            List<EntityID> path = pathPlanning.calc().getResult();
		            if (path != null && path.size() > 0) {
		                if (firstResult == null) {
		                    firstResult = new ArrayList<>(path);
		                    if (target == null) {
		                        break;
		                    }
		                }
		                EntityID refugeID = path.get(path.size() - 1);
		                pathPlanning.setFrom(refugeID);
		                pathPlanning.setDestination(target);
		                List<EntityID> fromRefugeToTarget = pathPlanning.calc().getResult();
		                if (fromRefugeToTarget != null && fromRefugeToTarget.size() > 0) {
		                    return new ActionMove(path);
		                }
		                refuges.remove(refugeID);
		                // remove failed
		                if (currentSize == refuges.size()) {
		                    break;
		                }
		                currentSize = refuges.size();
		            } else { // Failed to find a path, break directly ???
		                break;
		            }
		        }
		        // Goto nearest refuge
		        // If cannot find a refuge, just throw it!
		        return firstResult != null ? new ActionMove(firstResult) : null;
		  }
		  ```
-