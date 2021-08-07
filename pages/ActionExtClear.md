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
	  id:: 610d8541-3fe9-44b7-bfd8-1f93076ae25b
	  collapsed:: true
		-
		  ``` java
		  @Override
		  public ExtAction calc() {
		    
		      getTheInfomationOfTheWorld();
		  
		      this.result = null;
		      PoliceForce policeForce = (PoliceForce) this.agentInfo.me();
		  
		      // 需要休息则试着休息
		      if (this.needRest(policeForce)) {
		          List<EntityID> list = new ArrayList<>();
		          if (this.target != null) {
		              list.add(this.target);
		          }
		          this.result = this.calcRest(policeForce, this.pathPlanning, list);
		          if (this.result != null) {
		              this.lastLocationX = policeForce.getX();
		              this.lastLocationY = policeForce.getY();
		              return this;
		          }
		      }
		  
		      if (this.target == null) {
		          return this;
		      }
		  
		      // EntityID agentPosition = policeForce.getPosition();
		      StandardEntity targetEntity = this.worldInfo.getEntity(this.target);
		      StandardEntity positionEntity =
		              Objects.requireNonNull(this.worldInfo.getEntity(policeForce.getPosition()));
		  
		      if (targetEntity == null || !(targetEntity instanceof Area)) {
		          return this;
		      }
		  
		    	// 如果上次目标没完成，则继续
		      if (this.JudgeWhetherINeedContinue) {
		          this.result = this.getTheContinueAction(policeForce);
		          if (this.result != null) {
		              this.lastLocationX = policeForce.getX();
		              this.lastLocationY = policeForce.getY();
		              return this;
		          }
		      }
		    
		    	// 按警察和目标的相对位置，分以下三种情况
		      // 1. 警察所在位置就是目标位置
		      if ((policeForce.getPosition()).equals(this.target)) {
		          this.result = this.getAreaClearAction(policeForce);
		      }
		      // 2. 警察所在 area 和目标 area 相邻 (有公共边)
		      else if (((Area) targetEntity).getEdgeTo(policeForce.getPosition()) != null) {
		          this.result = this.getNeighbourAction(policeForce, (Area) targetEntity);
		      }
		    	// 3. 警察和目标隔开
		      else {
		          List<EntityID> path = this.pathPlanning.getResult(policeForce.getPosition(),
		                                                            this.target);
		          if (path != null && path.size() > 0) {
		              int index = path.indexOf(policeForce.getPosition());
		              if (index == -1) { // 警察的位置不在找到的 path 里
		                  Area area = (Area) positionEntity;
		                  for (int i = 0; i < path.size(); i++) {
		                      if (area.getEdgeTo(path.get(i)) != null) { //
		                          // 找到第一个和起点所在区域相邻的位置
		                          index = i; // 找到即记录其索引
		                          break;
		                      }
		                  }
		              }
		              else if (index >= 0) { // 警察的位置在找到的 path 里
		                  index++; // TODO
		              }
		              if (index >= 0 && index < (path.size())) {
		                  StandardEntity entity = this.worldInfo.getEntity(path.get(index));
		                  this.result = this.getNeighbourAction(policeForce, (Area) entity);
		                  // 如果得到的是一个 ActionMove 未指定目的地的实例，则丢弃
		                  if (this.result != null && this.result.getClass() == ActionMove.class) {
		                      if (!((ActionMove) this.result).getUsePosition()) {
		                          this.result = null;
		                      }
		                  }
		              }
		              if (this.result == null) { // 否则沿路径移动
		                  this.result = new ActionMove(path);
		                  this.count = 0;
		              }
		          }
		      }
		      // 更新位置信息
		      this.lastLocationX = policeForce.getX();
		      this.lastLocationY = policeForce.getY();
		      return this;
		  }
		  ```
	- [[Action]] getAction()
	- boolean isCurve([[Area]] previous, [[Area]] current, [[Area]] next, [[Vector2D]] edgeline)
	- boolean isClearable([[PoliceForce]] policeForce, [[Blockade]] blockade, [[Vector2D]] clearline)
	- double getAcuteAngle([[Vector2D]] first, [[Vector2D]] second)
	- [[Action]] tryAreaClear([[PoliceForce]] police, [[EntityID]] target)
	- [[Vector2D]] scaleClear([[Vector2D]] vector)
	- boolean intersectTwo([[Blockade]] blockade, [[Blockade]] another)
	- [[Action]] getRescueAction(PoliceForce police, Edge nextEgde)
	  collapsed:: true
		-
		  ``` java
		  private Action getRescueAction(PoliceForce police, Edge nextEgde) {
		  
		      getTheInfomationOfTheWorld();
		  
		      EntityID position = police.getPosition();
		      StandardEntity entity = this.worldInfo.getEntity(position);
		    
		      if (!(entity instanceof Road)) {
		          return null;
		      }
		    
		      Road road = (Road)entity;
		    
		      if (!road.isBlockadesDefined() || road.getBlockades().isEmpty()) {
		          return null;
		      }
		      
		      // get all other kinds of human in this area
		      Set<Human> allTheHumanOnRoad = new HashSet<>();
		  
		      Collection<Blockade> blockades = this.worldInfo.getBlockades(road);
		  
		      for (StandardEntity tempEntity :
		             this.worldInfo.getEntitiesOfType(
		               StandardEntityURN.AMBULANCE_TEAM,
		               StandardEntityURN.FIRE_BRIGADE,
		               StandardEntityURN.CIVILIAN)) {
		              Human human = (Human)tempEntity;
		              if (human.isPositionDefined() &&
		                  human.getPosition().equals(position) &&
		                  human.isXDefined() &&
		                  human.isYDefined()) {
		                  allTheHumanOnRoad.add(human);
		              }
		      }
		  
		      Set<Human> theHumanBeStucked = new HashSet<>();
		  
		      // find those who were stucked by blockades
		      for (Human human : allTheHumanOnRoad) {
		          int humanPositionX = human.getX();
		          int humanPositionY = human.getY();
		          for (Blockade blockade : blockades) {
		              if (this.isInside(humanPositionX, humanPositionY, blockade) ||
		                  this.JudgeWhetherNearBlockade(humanPositionX, humanPositionY,
		                                                blockade, 500)) {
		                  theHumanBeStucked.add(human);
		                  break;
		              }
		          }
		  
		      }
		    
		      allTheHumanOnRoad.clear();
		  
		      if (theHumanBeStucked.isEmpty()) {
		          return null;
		      }
		      int policeX = police.getX();
		      int policeY = police.getY();
		  
		      // find those who can be helped when clearing toward the next edge
		      Set<Human> byTheWay = new HashSet<>();
		  
		      if (nextEgde != null) {
		          Point2D midPosition = this.getMidPoint(nextEgde.getLine());
		  
		          int midPositionX = (int)midPosition.getX();
		          int midPositionY = (int)midPosition.getY();
		  
		          for (Human human : theHumanBeStucked) {
		              if (java.awt.geom.Line2D.ptLineDist(policeX, policeY,
		                                                  midPositionX, midPositionY,
		                                                  human.getX(), human.getY())
		                      < this.myClearRadius *4/5) {
		                  byTheWay.add(human);
		              }
		          }
		          theHumanBeStucked.removeAll(byTheWay);
		      }
		      if (!theHumanBeStucked.isEmpty()) { //help the rest of human first
		  
		          byTheWay.clear();
		  
		          int bestTargetX = 0;
		          int bestTargetY = 0;
		        
		          int count = -1;
		        
		          // find a direction that can help most human by one action
		          for (Human human : theHumanBeStucked) {
		              int humanX = human.getX();
		              int humanY = human.getY();
		  
		              Vector2D vector = new Vector2D(humanX - policeX, humanY - policeY);
		              if (vector.getLength() < this.theClearDistance) {
		                  vector.normalised().scale(this.theClearDistance + 500);
		              }
		              humanX = policeX + (int)vector.getX();
		              humanY = policeY + (int)vector.getY();
		  
		              int anotherCount = 0;
		  
		              for (Human other : theHumanBeStucked) {
		                  if (other.equals(human)) {
		                      continue;
		                  }
		                  if (java.awt.geom.Line2D.ptLineDist(humanX, humanY,
		                                                      policeX, policeY,
		                                                      other.getX(), other.getY())
		                          < this.myClearRadius * 4 / 5) {
		                      ++anotherCount;
		                  }
		              }
		  
		              if (anotherCount > count) {
		                  count = anotherCount;
		                  bestTargetX = humanX;
		                  bestTargetY = humanY;
		              }
		              else if (anotherCount == count) {
		                  if (this.worldInfo.getDistance(police,human) >
		                          this.getDistance(policeX, policeY,bestTargetX, bestTargetY)) {
		                      bestTargetX = humanX;
		                      bestTargetY = humanY;
		                  }
		              }
		          }
		          this.lastDestinationX = bestTargetX;
		          this.lastDestinationY = bestTargetY;
		          this.JudgeWhetherINeedContinue = true;
		          return this.getTheContinueAction(police);
		      }
		  
		      if (!byTheWay.isEmpty()) { // then clear and move to the next edge
		          Point2D point = this.getMidPoint(nextEgde.getLine());
		          this.lastDestinationX = (int)point.getX();
		          this.lastDestinationY = (int)point.getY();
		          this.JudgeWhetherINeedContinue = true;
		          return this.getTheContinueAction(police);
		      }
		      return null; // impossible branch
		  }
		  ```
	- [[Action]] getNeighbourAction(PoliceForce police, Area target)
	- [[Action]] calcRest([[Human]] human, [[PathPlanning]] pathPlanning, Collection<[[EntityID]]> targets)
	- boolean equalsPoint(double p1X, double p1Y, double p2X, double p2Y, double range) #TODO
	  collapsed:: true
		- 把点看成半径为 range 的圆
		- 该函数返回 true 当且仅当 p1 内含于 p2
		- 字面上看该函数的作用是判断两点是否为同一点
		-
		  ``` java
		  private boolean equalsPoint(double p1X, double p1Y,
		                              double p2X, double p2Y, double range) {
		      return (p2X - range < p1X && p1X < p2X + range)
		          && (p2Y - range < p1Y && p1Y < p2Y + range);
		  }
		  ```
	- [[Action]] getAreaClearAction([[PoliceForce]] police)
		- 只在 ((610d8541-3fe9-44b7-bfd8-1f93076ae25b)) 里，当警察在目标上时使用
		-
		  ``` java
		  private Action getAreaClearAction(PoliceForce police) {
		  
		      getTheInfomationOfTheWorld();
		  
		      EntityID policePosition = police.getPosition();
		      StandardEntity entity = this.worldInfo.getEntity(policePosition);
		      if (!(entity instanceof Road)) { // 不在路上，没法接触清除
		          return null;
		      }
		      Road road = (Road)entity;
		    
		      // 在路上，但路没堵住，不用清除
		      if (!road.isBlockadesDefined() || road.getBlockades().isEmpty()) {
		          return null;
		      }
		    
		      // 能救人，先救人
		      Action action = this.getRescueAction(police, null); //help human first
		      if (action != null) {
		          return action;
		      }
		  
		      // 获得警察所在 area (Road) 所有可通过的边
		      List<Edge> allThePassableEdge = new ArrayList<>(this.getAllThePassableEdge(road));
		  
		      // 并将按到警察的距离升序排序
		      allThePassableEdge.sort(new TheSortForEdge(new Point2D(police.getX(), police.getY())));
		    
		      // 由远及近考虑所有边
		      for (int i = allThePassableEdge.size() - 1; i >= 0; --i) {
		          // TODO: 为什么是中点？
		          Point2D point = this.getMidPoint(allThePassableEdge.get(i).getLine());
		  		// 
		          this.lastDestinationX = (int)point.getX();
		          this.lastDestinationY = (int)point.getY();
		          // 
		          this.JudgeWhetherINeedContinue = true;
		          action = this.getTheContinueAction(police);
		          // 
		          if (action != null) {
		              return action;
		          }
		      }
		      
		      // 对所有边的尝试均失败
		      // 猜测：也就是警察自己被卡住了
		      int roadX = road.getX();
		      int roadY = road.getY();
		  
		      Collection<Blockade> blockades = this.worldInfo.getBlockades(road);
		      for (Edge edge : allThePassableEdge) {
		          Point2D midPoint = this.getMidPoint(edge.getLine());
		          double midPointX = midPoint.getX();
		          double midPointY = midPoint.getY();
		  
		          Vector2D vector = (new Vector2D(roadX - midPointX, roadY - midPointY))
		              .normalised().scale(250).getNormal();
		          for (Blockade blockade : blockades) {
		              if (this.intersect(roadX, roadY, midPointX, midPointY, blockade) ||
		                      this.intersect(roadX + vector.getX(), roadY + vector.getY(),
		                              midPointX + vector.getX(), midPointY + vector.getY(), blockade) ||
		                      this.intersect(roadX - vector.getX(), roadY - vector.getY(),
		                              midPointX - vector.getX(), midPointY - vector.getY(), blockade)) {
		                  return new ActionClear(blockade); // 不能过人，则使用接触清除
		              }
		          }
		      }
		      return null;
		  }
		  ```
	- [[Action]] getTheContinueAction([[PoliceForce]] police)
		-
		  ``` java
		  private Action getTheContinueAction(PoliceForce police) { // basic function
		  
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
		      // 如果警察自己被困住，用接触清除方法先清理自己身上的障碍物
		      if (!covers.isEmpty()) {
		          return new ActionClear((Blockade) this.getTheClosestEntity(covers,
		                                                                     this.agentInfo.me()));
		      }
		      covers.clear();
		  
		      Vector2D vector = (new Vector2D(this.lastDestinationX - policeX,
		                                      this.lastDestinationY - policeY))
		        .normalised().scale(this.theClearDistance);
		  
		      int clearX = policeX + (int) vector.getX();
		      int clearY = policeY + (int) vector.getY();
		  
		      // 下面的 300 和 200 有什么特别意义吗？范围清理那个长方形框的大小？为什么还能往后呢？
		      vector = vector.normalised().scale(-300);
		  
		      int startX = policeX + (int) vector.getX();
		      int startY = policeY + (int) vector.getY();
		  
		      vector = vector.normalised().scale(250).getNormal();
		      List<Blockade> removeBlockadesList = new ArrayList<>();
		  
		      // 根据距离确定清除范围是否足够
		      // farPosition 记录的是最远点的坐标
		      // 若目标超出清除范围，则目标点 (lastDestinationX, lastDestinationY) 即为最远点
		      // 否则为能清除到的最远点 (clearX, clearY)
		      int farPositionX = this.getDistance(policeX,
		                                          policeY,
		                                          this.lastDestinationX,
		                                          this.lastDestinationY) >
		              this.getDistance(policeX, policeY, clearX, clearY)
		        ? this.lastDestinationX : clearX;
		    
		      int farPositionY = farPositionX == clearX ? clearY : this.lastDestinationY;
		  
		      // 根据语义猜测，下面循环的作用是找出所有不在清除范围内的障碍物
		      // 并将之从 blokades 中删除，只留下能清除到的障碍物
		      for (Blockade blockade : blockades) {
		          // 以下 if(!(...)) 中，只有所有条件均为假，整个条件表达式的值才为真
		          // 不是只能像一个方向清除吗？怎么还有往后比较的呢？
		          // 用五条线是否和障碍物相交，来判断障碍物是否再清理区域内
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
		  
		      // 如果所有障碍物都没法清除，返回
		      if (blockades.isEmpty()) {
		          this.lastDestinationX = 0;
		          this.lastDestinationY = 0;
		          this.JudgeWhetherINeedContinue = false;
		          return null;
		      }
		  
		      // 否则优先清除最近的
		      Blockade closest = (Blockade)this.getTheClosestEntity(blockades, police);
		  
		      // 再判断一次？？ 
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
		          // 防止反复清理一个地方？
		          if (this.equalsPoint(clearX, clearY, this.oldClearX, this.oldClearY, 1000)) {
		              ++this.count;
		              if (this.count >= this.forcedMove) {
		                  this.oldClearX = 0;
		                  this.oldClearY = 0;
		                  this.count = 0;
		                  return new ActionMove(Lists.newArrayList(position), clearX, clearY);
		              }
		          }
		          // 记录
		          this.oldClearX = clearX;
		          this.oldClearY = clearY;
		          // 返回范围清理
		          return new ActionClear(clearX, clearY);
		      }
		  ```
	- boolean isInside(double pX, double pY, [[Blockade]] blockade)
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
	  collapsed:: true
		- 判断给定直线 (agentX, agentY) - (pointX, pointY) 是否与障碍物相交
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
## Subclass
	- [[TheSortForEdge]]