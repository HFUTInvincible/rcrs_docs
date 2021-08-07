## Class
	- [[ExtAction]] => [[ActionExtClear]]
## Fields
	- [[PathPlanning]] pathPlanning
	- [[EntityID]] target
	- int oldClearX
	- int oldClearY
	- int count
	- boolean JudgeWhetherINeedContinue
	- int lastTime
	- int forcedMove
	- int thresholdRest
	- int kernelTime
	- int lastLocationX
	- int lastLocationY
	- int lastDestinationX
	- int lastDestinationY
	- int myClearRadius
	- int theClearDistance
## Methods
	- [[ExtAction]] updateInfo([[MessageManager]] messageManager)
	- [[ExtAction]] setTarget([[EntityID]] target)
	- [[ExtAction]] calc()
	- [[Action]] getAction()
	- boolean isCurve([[Area]] previous, [[Area]] current, [[Area]] next, [[Vector2D]] edgeline)
	- boolean isClearable([[PoliceForce]] policeForce, [[Blockade]] blockade, [[Vector2D]] clearline)
	- double getAcuteAngle([[Vector2D]] first, [[Vector2D]] second)
	- [[Action]] tryAreaClear([[PoliceForce]] police, [[EntityID]] target)
	- [[Vector2D]] scaleClear([[Vector2D]] vector)
	- boolean intersectTwo([[Blockade]] blockade, [[Blockade]] another)
	- [[Action]] getRescueAction(PoliceForce police, Edge nextEgde)
	- [[Action]] getNeighbourAction(PoliceForce police, Area target)
	- [[Action]] calcRest([[Human]] human, [[PathPlanning]] pathPlanning, Collection<[[EntityID]]> targets)
	- boolean equalsPoint(double p1X, double p1Y, double p2X, double p2Y, double range)
	- [[Action]] getAreaClearAction([[PoliceForce]] police)
	- [[Action]] getTheContinueAction([[PoliceForce]] police)
	  collapsed:: true
		-
		  ``` java
		  private Action getTheContinueAction(PoliceForce police) { //basic function
		  
		      getTheInfomationOfTheWorld();
		  
		      if (!JudgeWhetherINeedContinue) {
		          this.lastDestinationX = 0;
		          this.lastDestinationY = 0;
		          return null;
		      }
		  
		      this.JudgeWhetherINeedContinue = false;
		      EntityID position = police.getPosition();
		  
		      StandardEntity entity = this.worldInfo.getEntity(position);
		  
		      if (!(entity instanceof Road)) {
		          this.lastDestinationX = 0;
		          this.lastDestinationY = 0;
		          return null;
		      }
		  
		      Road road = (Road)entity;
		  
		      if (!road.isBlockadesDefined() || road.getBlockades().isEmpty()) {
		          this.lastDestinationX = 0;
		          this.lastDestinationY = 0;
		          return null;
		      }
		  
		      this.JudgeWhetherINeedContinue = true;
		      Collection<Blockade> blockades = this.worldInfo.getBlockades(road);
		      int policeX = police.getX();
		      int policeY = police.getY();
		      Set<Blockade> covers = new HashSet<>();
		      for (Blockade blockade : blockades) {
		          if (this.isInside(policeX, policeY, blockade)) {
		              covers.add(blockade);
		          }
		      }
		      if (!covers.isEmpty()) { //if police is covered by blockade, clear it first
		          return new ActionClear((Blockade) this.getTheClosestEntity(covers,
		                                                                     this.agentInfo.me()));
		      }
		      covers.clear();
		  
		      Vector2D vector = (new Vector2D(this.lastDestinationX - policeX,
		                                      this.lastDestinationY - policeY))
		        .normalised().scale(this.theClearDistance);
		  
		      int clearX = policeX + (int) vector.getX();
		      int clearY = policeY + (int) vector.getY();
		  
		      vector = vector.normalised().scale(-300);
		  
		      int startX = policeX + (int) vector.getX();
		      int startY = policeY + (int) vector.getY();
		  
		      vector = vector.normalised().scale(250).getNormal();
		      List<Blockade> removeBlockadesList = new ArrayList<>();
		  
		      int farPositionX = this.getDistance(policeX,
		                                          policeY,
		                                          this.lastDestinationX,
		                                          this.lastDestinationY) >
		              this.getDistance(policeX, policeY, clearX, clearY)
		        ? this.lastDestinationX : clearX;
		    
		      int farPositionY = farPositionX == clearX ? clearY : this.lastDestinationY;
		  
		      //functions in java.awt.geom.Area are not fast enough
		      for (Blockade blockade : blockades) {
		          if (!(this.intersect(startX, startY, farPositionX, farPositionY, blockade) ||
		                  this.intersect(policeX, policeY,
		                                 farPositionX + vector.getX(),
		                                 farPositionY + vector.getY(), blockade) ||
		                  this.intersect(policeX, policeY,
		                                 farPositionX - vector.getX(),
		                                 farPositionY - vector.getY(), blockade) ||
		                  this.intersect(policeX, policeY,
		                                 startX + vector.getX(),
		                                 startY + vector.getY(), blockade) ||
		                  this.intersect(policeX, policeY,
		                                 startX - vector.getX(),
		                                 startY - vector.getY(), blockade)))
		          {
		              removeBlockadesList.add(blockade);
		          }
		      }
		  
		      blockades.removeAll(removeBlockadesList);
		      removeBlockadesList.clear();
		  
		      if (blockades.isEmpty()) {
		          this.lastDestinationX = 0;
		          this.lastDestinationY = 0;
		          this.JudgeWhetherINeedContinue = false;
		          return null;
		      }
		  
		      Blockade closest = (Blockade)this.getTheClosestEntity(blockades, police);
		  
		      if (!(this.intersect(startX, startY, clearX, clearY, closest) ||
		              this.intersect(startX - vector.getX(), startY - vector.getY(),
		                      clearX + vector.getX(), clearY + vector.getY(), closest) ||
		              this.intersect(startX + vector.getX(), startY + vector.getY(),
		                      clearX - vector.getX(), clearY - vector.getY(), closest))) {
		          this.oldClearX = 0;
		          this.oldClearY = 0;
		          this.count = 0;
		          return new ActionMove(Lists.newArrayList(position),
		                                lastDestinationX,
		                                lastDestinationY);
		      }
		      else {
		          if (this.equalsPoint(clearX, clearY, this.oldClearX, this.oldClearY, 1000)) {
		              ++this.count;
		              if (this.count >= this.forcedMove) {
		                  this.oldClearX = 0;
		                  this.oldClearY = 0;
		                  this.count = 0;
		                  return new ActionMove(Lists.newArrayList(position), clearX, clearY);
		              }
		          }
		          this.oldClearX = clearX;
		          this.oldClearY = clearY;
		          return new ActionClear(clearX, clearY);
		      }
		  ```
	- boolean isInside(double pX, double pY, [[Blockade]] blockade)
	  collapsed:: true
		- 判断点是否在障碍物内部
		-
		  ``` java
		  private boolean isInside(double pX, double pY, Blockade blockade) {
		  
		      Point2D p = new Point2D(pX, pY);
		      int apex[] = blockade.getApexes();
		      Vector2D v1 = (new Point2D(apex[apex.length - 2], apex[apex.length - 1])).minus(p);
		      Vector2D v2 = (new Point2D(apex[0], apex[1])).minus(p);
		      double theta = this.getAngle(v1, v2);
		  
		      for(int i = 0; i < apex.length - 2; i += 2) {
		          v1 = (new Point2D(apex[i], apex[i + 1])).minus(p);
		          v2 = (new Point2D(apex[i + 2], apex[i + 3])).minus(p);
		          theta += this.getAngle(v1, v2);
		      }
		      return Math.round(Math.abs((theta / 2) / Math.PI)) >= 1;
		  }
		  ```
	- double getAngle([[Vector2D]] v1, [[Vector2D]] v2)
	- boolean JudgeWhetherNearBlockade(double pX, double pY, [[Blockade]] blockade, double range)
	- boolean intersect([[Blockade]] blockade, [[Blockade]] another)
		-
		  ``` java
		  private boolean intersect(double agentX, double agentY,
		                            double pointX, double pointY, Blockade blockade) {
		      List<Line2D> lines = GeometryTools2D.pointsToLines(
		          GeometryTools2D.vertexArrayToPoints(blockade.getApexes()), true);
		      for(Line2D line : lines) {
		          Point2D start = line.getOrigin();
		          Point2D end = line.getEndPoint();
		          double startX = start.getX();
		          double startY = start.getY();
		          double endX = end.getX();
		          double endY = end.getY();
		          if(java.awt.geom.Line2D.linesIntersect(
		                  agentX, agentY, pointX, pointY,
		                  startX, startY, endX, endY
		          )) {
		              return true;
		          }
		      }
		      return false;
		  }
		  ```
	- boolean intersect(double agentX, double agentY, double pointX, double pointY, [[Blockade]] blockade)
	- double getDistance(double fromX, double fromY, double toX, double toY)
	- [[Point2D]] getMidPoint([[Line2D]] line)
	- Set<[[Edge]]> getAllThePassableEdge([[Area]] area)
	- boolean needRest([[Human]] agent)
	- [[StandardEntity]] getTheClosestEntity(Collection<? extends [[StandardEntity]]> entities, [[StandardEntity]] reference)
	- void getTheInfomationOfTheWorld()