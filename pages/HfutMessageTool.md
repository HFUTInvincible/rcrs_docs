## Intro
	- 在 [[MessageManager]] 的基础上，封装了一系列策略，提供了发送信息消息、发送请求等几个统一的接口供智能体调用
## Attention
	- 官方也实现了一个不太好的 MessageTool，并由于比赛规则，我们不能修改这个类
	- 所以考虑自己实现一个相同功能的 [[HFUTMessageTools]] 类，将其加入一个每个周期都会调用的函数，并屏蔽掉官方实现的 MessageTool
	- 官方实现的 MessageTool 发送的消息有明显的特征 (如下)，我们可以在分类投放时将所有这类消息过滤掉，实现对官方 MessageTool 的规避
	- 当然，我们自己的 [[HfutMessageTool]] 里就不能发这种消息了，否则也会被过滤掉
	-
	  ```java
	  if(msg instanceof StandardMessage){ //把所有的官方 MessageTool 发来的信息过滤掉
	      StandardMessage smsg = (StandardMessage) msg;
	      if(smsg.getSendingPriority() == StandardMessagePriority.LOW){
	          continue;
	      }
	      if((smsg.getSendingPriority() == StandardMessagePriority.NORMAL) &&
	              (smsg instanceof CommandPolice || smsg instanceof CommandFire || smsg instanceof CommandAmbulance)){
	          continue;
	      }
	  }
	  ```
## Fields
	- HashMap<[[EntityID]], [[EntityID]]> entrance2building
	- [[EntityID]] dominanceAgentID
	- Set<[[EntityID]]> receivedPassableRoads
		- 在中心智能体的认知里，所有可以通过的路构成的集合
		- 初始化和后续更新都在 ((611765ab-25bf-4508-abe6-f4ec760c1f75)) 里。
		- 注意：更新时如果发现有建筑可能倒塌，堵塞原来可以通行的路，则要全部清除重新计算
	- [[EntityID]] dominanceAgentID
		- 初始化和更新都在 ((611765ab-25bf-4508-abe6-f4ec760c1f75)) 里
		  later:: 1628925781983
## Methods
	- void reflectMessage( [[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, [[ScenarioInfo]] scenarioInfo, [[MessageManager]] messageManager)
	  done:: 1628922188563
	  now:: 1628922187963
	  later:: 1628922008605
		- 为这个周期发送过信息智能体打上时间戳
		-
		  ``` java
		  public void reflectMessage(AgentInfo agentInfo,
		                             WorldInfo worldInfo,
		                             ScenarioInfo scenarioInfo,
		                             MessageManager messageManager) {
		      /******************************************************************/
		      // 下面两句应该是多余的
		      Set<EntityID> changedEntities =
		              worldInfo.getChanged().getChangedEntities();
		      changedEntities.add(agentInfo.getID());
		      /******************************************************************/
		      int time = agentInfo.getTime();
		      for (CommunicationMessage message :
		              messageManager.getReceivedMessageList(StandardMessage.class)) {
		          StandardEntity entity = null;
		          entity = MessageUtil.reflectMessage(worldInfo,
		                  (StandardMessage) message);
		          if (entity != null) {
		              this.receivedTimeMap.put(entity.getID(), time);
		          }
		      }
		  }
		  ```
	- void updateInfo([[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, [[ScenarioInfo]] scenarioInfo, [[MessageManager]] messageManager)
	  id:: 611765ab-25bf-4508-abe6-f4ec760c1f75
		-
		  ```java
		  private void updateInfo(AgentInfo agentInfo, WorldInfo worldInfo,
		                          ScenarioInfo scenarioInfo,
		                          MessageManager messageManager) {
		      Collection<StandardEntity> Roads = worldInfo.getEntitiesOfType(ROAD);
		      for (StandardEntity o : Roads) {
		          Road road = (Road) o;
		          if (isGetCross(road) && road.isBlockadesDefined() && road.getBlockades().size() > 0) {
		              logger.info("(" + road.getX() + "," + road.getY() + ")");
		          }
		      }
		      logger.info(agentInfo.getTime());
		  
		      if (this.maxTimeStep == Integer.MAX_VALUE) {
		          try {
		              this.maxTimeStep = scenarioInfo.getKernelTimesteps();
		          } catch (NoSuchConfigOptionException e) {
		          }
		      }
		  
		      this.agentsPotition.clear();
		      this.dominanceAgentID = agentInfo.getID();
		  
		      for (StandardEntity entity :
		              worldInfo.getEntitiesOfType(AMBULANCE_TEAM, FIRE_BRIGADE,
		                      POLICE_FORCE)) {
		          Human human = (Human) entity;
		          this.agentsPotition.add(human.getPosition());
		          // 同一位置的中心智能体还能是 2 个？？？
		          if (agentInfo.getPosition().equals(human.getPosition())
		                  && dominanceAgentID.getValue() < entity.getID().getValue()) {
		              // 默认只使用编号最小的中心智能体？
		              this.dominanceAgentID = entity.getID();
		          }
		      }
		  
		      boolean aftershock = false; // 余震
		      for (EntityID id : agentInfo.getChanged().getChangedEntities()) {
		          if (this.prevBrokenessMap.containsKey(id) && worldInfo.getEntity(id).getStandardURN().equals(BUILDING)) {
		              Building building = (Building) worldInfo.getEntity(id);
		              int brokenness = this.prevBrokenessMap.get(id);
		              this.prevBrokenessMap.get(id);
		              if (building.isBrokennessDefined()) {
		                  // 损坏得更严重了，则有可能引发余震
		                  if (building.getBrokenness() > brokenness) {
		                      aftershock = true;
		                  }
		              }
		          }
		      }
		  
		      // 清空，重新记录 brokenness
		      this.prevBrokenessMap.clear();
		      for (EntityID id : agentInfo.getChanged().getChangedEntities()) {
		          if (!worldInfo.getEntity(id).getStandardURN().equals(BUILDING)) {
		              continue;
		          }
		  
		          Building building = (Building) worldInfo.getEntity(id);
		          // 如果建筑损坏，有可能产生余震？
		          if (building.isBrokennessDefined()) {
		              // 记录之
		              this.prevBrokenessMap.put(id, building.getBrokenness());
		          }
		      }
		  
		      // 如果可能发生余震，则原来的可以通过的路也可能变得无法通过，因此要清空重新计算
		      if (aftershock) {
		          this.receivedPassableRoads.clear();
		      }
		  
		      // 根据 MessageRoad 重新计算可以通过的路
		      for (CommunicationMessage message :
		              messageManager.getReceivedMessageList(MessageRoad.class)) {
		          MessageRoad messageRoad = (MessageRoad) message;
		          Boolean passable = messageRoad.isPassable();
		          if (passable != null && passable) {
		              this.receivedPassableRoads.add(messageRoad.getRoadID());
		          }
		      }
		  
		      // 为什么不使用后面写的函数呢？
		      if (agentInfo.getTime() == 2) {
		          //entrance
		          for (StandardEntity entity :
		                  this.worldInfo.getEntitiesOfType(StandardEntityURN.BUILDING)) {
		              for (EntityID id : ((Building) entity).getNeighbours()) {
		                  if (this.worldInfo.getEntity(id) instanceof Road) {
		                      this.entrance2building.put(id, entity.getID());
		                  }
		              }
		          }
		          System.out.println("entrance2building:" + entrance2building);
		      }
		  }
		  ```
	- void sendInformationMessages( [[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, [[ScenarioInfo]] scenarioInfo, [[MessageManager]] messageManager)
	  later:: 1628922096530
		- 报告智能体的最新情况，也就是发送 Information Message
		-
		  ```java
		  public void sendInformationMessages(AgentInfo agentInfo,
		                                      WorldInfo worldInfo,
		                                      ScenarioInfo scenarioInfo,
		                                      MessageManager messageManager) {
		      this.logger = TestLogger.getLogger("HuftMessageTool");
		    
		      // 通过 worldInfo 获得这一周期发送改变的 Entity
		      Set<EntityID> changedEntities =
		              worldInfo.getChanged().getChangedEntities();
		  
		      this.updateInfo(agentInfo, worldInfo, scenarioInfo, messageManager);
		  
		      if (isPositionMoved(agentInfo) && isDominance(agentInfo)) {
		          for (EntityID entityID : changedEntities) {
		              if (!(isRecentlyReceived(agentInfo, entityID)))//
		                  // 5个周期没有收到关于这个实体的信息，那么我就发送它的最新信息。
		              {
		                  StandardEntity entity = worldInfo.getEntity(entityID);
		                  CommunicationMessage message = null;
		                  switch (entity.getStandardURN()) {
		                      case ROAD:
		                          Road road = (Road) entity;
		                          // 如果道路可以通过 但是我记录的可通过道路list里面没有它
		                          if (isNonBlockadeAndNotReceived(road)) {
		                              message = new MessageRoad(true,
		                                      StandardMessagePriority.NORMAL,
		                                      road, null,
		                                      true, false);
		                              messageManager.addMessage(
		                                      new MessageRoad(false,
		                                              StandardMessagePriority.HIGH,
		                                              road, null,
		                                              true, false)
		                              );
		                          } else {
		                              message = new MessageRoad(false,
		                                      StandardMessagePriority.NORMAL,
		                                      road, null,
		                                      true, false);
		                          }
		  
		                          break;
		                      case BUILDING:
		                          Building building = (Building) entity;
		  
		                          if (isOnFire(building))// 如果建筑正在着火 //或者建筑被淹了
		                          {
		                              message = new MessageBuilding(true,
		                                      StandardMessagePriority.HIGH, building);
		                              messageManager.addMessage(
		                                      new MessageBuilding(false,
		                                              StandardMessagePriority.HIGH,
		                                              building)
		                              );
		                          } else {
		                              message = new MessageBuilding(false,
		                                      StandardMessagePriority.NORMAL,
		                                      building);
		                          }
		                          break;
		                      case CIVILIAN:
		                          Civilian civilian = (Civilian) entity;
		  //                            //如果这个人被卡住
		  //                            StandardEntity positionEntity = worldInfo
		  //                            .getPosition(civilian);
		  //                            if (positionEntity instanceof Building) {
		  //
		  //                                Building building1 = (Building)positionEntity;
		  //                                if (isOnFire(building1))// 如果建筑正在着火或者建筑被淹了
		  //                                {
		  //                                    messageManager.addMessage(
		  //                                            new CommandFire(true,
		  //                                            StandardMessagePriority.HIGH,
		  //                                            null,building1.getID(),
		  //                                            CommandFire.ACTION_EXTINGUISH)
		  //                                    );
		  //                                }
		  //                                EntityID entrance2building = this
		  //                                .getClosestEntityID(this
		  //                                .getEntrancesOfBuilding(building1),
		  //                                civilian.getID());
		  //                                if (entrance2building != null && !this
		  //                                .isGetCross((Area)this.worldInfo.getEntity
		  //                                (entrance2building))) {
		  //                                    messageManager.addMessage(
		  //                                            new CommandPolice(true,
		  //                                            StandardMessagePriority.HIGH,
		  //                                            null, entrance2building,
		  //                                            CommandPolice.ACTION_CLEAR)
		  //                                    );
		  //                                }
		  //                                if (isUnmovalCivilian(civilian))
		  //                                {
		  //                                    messageManager.addMessage(
		  //                                            new CommandAmbulance(true,
		  //                                            StandardMessagePriority.HIGH,
		  //                                            null, civilian.getID(),
		  //                                            CommandAmbulance.ACTION_RESCUE)
		  //                                    );
		  //                                }
		  //                            }
		                          // 如果这个人受了伤或者被掩埋
		                          if (isUnmovalCivilian(civilian)) {
		                              message = new MessageCivilian(true,
		                                      StandardMessagePriority.HIGH, civilian);
		                              //liu
		                              //message = new CommandPolice(true,
		                              // StandardMessagePriority.HIGH,null,
		                              // agentInfo.getPosition(),CommandPolice
		                              // .ACTION_CLEAR);
		  
		                              messageManager.addMessage(
		                                      new MessageCivilian(false,
		                                              StandardMessagePriority.HIGH,
		                                              civilian)
		                                      //new CommandPolice(false,
		                                      // StandardMessagePriority.HIGH,null,
		                                      // agentInfo.getPosition(),
		                                      // CommandPolice.ACTION_CLEAR)
		                              );
		                              //logger.info("CIV:" + agentInfo.getID());
		                              //liu
		                          } else {
		                              message = new MessageCivilian(false,
		                                      StandardMessagePriority.HIGH, civilian);
		                          }
		                          break;
		                  }
		  
		                  messageManager.addMessage(message);
		              }
		          }
		      }
		  
		      recordLastPosition(agentInfo);
		  }
		  ```
	- void sendRequestMessages( [[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, [[ScenarioInfo]] scenarioInfo, [[MessageManager]] messageManager)
		- 发送求救信息，比如被卡住了，就请警察来清障；被火困住了，就请消防来灭火
		-
		  ```java
		  public void sendRequestMessages(AgentInfo agentInfo, WorldInfo worldInfo,
		                                  ScenarioInfo scenarioInfo,
		                                  MessageManager messageManager) {
		      logger.info("sendRequst");
		      if (agentInfo.me().getStandardURN() == AMBULANCE_TEAM) {
		          logger.info("AA");
		          int currentTime = agentInfo.getTime();
		          Human me = (Human) agentInfo.me();
		          int agentX = me.getX();
		          int agentY = me.getY();
		          StandardEntity positionEntity = worldInfo.getPosition(me);
		          if (positionEntity instanceof Road)//以下处理救护车被障碍卡住的情况
		          {
		              boolean isSendRequest = false;
		  
		              Road road = (Road) positionEntity;
		              // 如果我被障碍卡住就求救
		              if (road.isBlockadesDefined() && road.getBlockades().size() > 0) {
		                  for (Blockade blockade : worldInfo.getBlockades(road)) {
		                      if (blockade == null || !blockade.isApexesDefined()) {
		                          continue;
		                      }
		  
		                      if (this.isInside(agentX, agentY,
		                              blockade.getApexes())) {
		                          isSendRequest = true;
		                      }
		                  }
		              }
		              // 如果我连续几个周期都在同一条路上，也说明我被卡住了
		              if (this.lastPosition != null && this.lastPosition.getValue()
		                  == road.getID().getValue()) {
		                  this.stayCount++;
		                  if (this.stayCount > this.getMaxTravelTime(road)) {
		                      isSendRequest = true;
		                  }
		              } else {
		                  this.lastPosition = road.getID();
		                  this.stayCount = 0;
		              }
		  
		              if (isSendRequest && ((currentTime - this.lastSentTimePolice)
		                                    >= this.sendingAvoidTimeClearRequest)) {
		                  this.lastSentTimePolice = currentTime;
		                  // 救护车在进入 refuge 的入口出被卡，就让警察直接去清理建筑？
		                  if (this.entrance2building.keySet().contains(me.getPosition())) {
		                      EntityID building =
		                              this.entrance2building.get(me.getPosition());
		                      messageManager.addMessage(
		                              new CommandPolice(true,
		                                      StandardMessagePriority.HIGH, null,
		                                      building, CommandPolice.ACTION_CLEAR)
		                      );
		                      messageManager.addMessage(
		                              new CommandPolice(false,
		                                      StandardMessagePriority.HIGH, null,
		                                      building, CommandPolice.ACTION_CLEAR)
		                      );
		                  } else {
		                      messageManager.addMessage(
		                              new CommandPolice(true,
		                                      StandardMessagePriority.HIGH, null,
		                                      me.getPosition(),
		                                      CommandPolice.ACTION_CLEAR)
		                      );
		                      messageManager.addMessage(
		                              new CommandPolice(false,
		                                      StandardMessagePriority.HIGH, null,
		                                      me.getPosition(),
		                                      CommandPolice.ACTION_CLEAR)
		                      );
		                  }
		  
		              }
		          }
		          //以下处理救护车被困在建筑物中的情况
		          if (positionEntity instanceof Building) {
		              Building building = (Building) positionEntity;
		  
		              boolean entranceBlock = false;
		              EntityID entrance =
		                  this.getClosestEntityID(this.getEntrancesOfBuilding(building),
		                                          agentInfo.getID());
		  
		              if (entrance != null &&
		                  !this.isGetCross((Area) this.worldInfo.getEntity(entrance))) {
		                  entranceBlock = true;
		              }
		  
		              if (entranceBlock && ((currentTime - this.lastSentTimePolice)
		                                    >= this.sendingAvoidTimeClearRequest)) {
		                  this.lastSentTimePolice = currentTime;
		                  messageManager.addMessage(
		                          new CommandPolice(true,
		                                  StandardMessagePriority.HIGH, null,
		                                  building.getID(),
		                                  CommandPolice.ACTION_CLEAR)
		                  );
		                  messageManager.addMessage(
		                          new CommandPolice(false,
		                                  StandardMessagePriority.HIGH, null,
		                                  building.getID(),
		                                  CommandPolice.ACTION_CLEAR)
		                  );
		              }
		  //                if (entranceBlock && isOnFire(building) && ((currentTime -
		  //                this.lastSentTimeFire) >= this
		  //                .sendingAvoidTimeExtinguishRequest)){
		  //                    this.lastSentTimeFire = currentTime;//在着火的建筑里被困
		  //                    messageManager.addMessage(
		  //                            new CommandFire(true, StandardMessagePriority
		  //                            .HIGH, null, building.getID(), CommandFire
		  //                            .ACTION_EXTINGUISH)
		  //                    );
		  //                    messageManager.addMessage(
		  //                            new CommandFire(false, StandardMessagePriority
		  //                            .HIGH, null, building.getID(), CommandFire
		  //                            .ACTION_EXTINGUISH)
		  //                    );
		  //                }
		          }
		  
		  //            if (me.isBuriednessDefined() && me.getBuriedness() > 0) {//
		  //            以下处理救护车被掩埋的情况，
		  //                if((currentTime - this.lastSentTimeAmbulance) >= this
		  //                .sendingAvoidTimeRescueRequest){
		  //                    messageManager.addMessage(
		  //                            new CommandAmbulance(true,
		  //                            StandardMessagePriority.HIGH, null, agentInfo
		  //                            .getPosition(), CommandAmbulance.ACTION_RESCUE));
		  //                    this.lastSentTimeAmbulance = currentTime;
		  //                }
		  //            }
		      } else if (agentInfo.me().getStandardURN() == POLICE_FORCE) {
		          logger.info("BB");
		          int currentTime = agentInfo.getTime();
		          Human me = (Human) agentInfo.me();
		  
		          //以下处理警察被掩埋的情况
		          if (me.isBuriednessDefined() && me.getBuriedness() > 0) {
		              if ((currentTime - this.lastSentTimeAmbulance)
		                  >= this.sendingAvoidTimeRescueRequest) {
		  //                    messageManager.addMessage(
		  //                            new CommandAmbulance(true,
		  //                            StandardMessagePriority.HIGH, null, me.getID(),
		  //                            CommandAmbulance.ACTION_RESCUE)
		  //                    );
		  //                    messageManager.addMessage(
		  //                            new CommandAmbulance(false,
		  //                            StandardMessagePriority.HIGH, null, me.getID(),
		  //                            CommandAmbulance.ACTION_RESCUE)
		  //                    );
		                  // 这里是漏删了一行吗？都没有发送求援请求了！
		                  this.lastSentTimeAmbulance = currentTime;
		              }
		              StandardEntity positionEntity = worldInfo.getPosition(me);
		              if (positionEntity instanceof Building) {
		                  //以下处理警察被掩埋在出口被堵着的建筑里
		                  Building building = (Building) positionEntity;
		                  boolean entranceBlock = false;
		                  EntityID entrance =
		                      this.getClosestEntityID(this.getEntrancesOfBuilding(building),
		                                              agentInfo.getID());
		                  if (entrance != null &&
		                      !this.isGetCross((Area) this.worldInfo.getEntity(entrance))) {
		                      entranceBlock = true;
		                  }
		                  if (entranceBlock &&
		                      ((currentTime - this.lastSentTimePolice)
		                       >= this.sendingAvoidTimeClearRequest)) {
		                      this.lastSentTimePolice = currentTime;
		                      messageManager.addMessage(
		                              new CommandPolice(true,
		                                      StandardMessagePriority.HIGH, null,
		                                      building.getID(),
		                                      CommandPolice.ACTION_CLEAR)
		                      );
		                      messageManager.addMessage(
		                              new CommandPolice(false,
		                                      StandardMessagePriority.HIGH, null,
		                                      building.getID(),
		                                      CommandPolice.ACTION_CLEAR)
		                      );
		                  }
		                  //以下处理警察被掩埋在了着火的建筑里
		                  if (isOnFire(building)
		                      && ((currentTime - this.lastSentTimeFire)
		                           >= this.sendingAvoidTimeExtinguishRequest)) {
		                      this.lastSentTimeFire = currentTime;
		                      messageManager.addMessage(
		                              new CommandFire(true,
		                                      StandardMessagePriority.HIGH, null,
		                                      building.getID(),
		                                      CommandFire.ACTION_EXTINGUISH)
		                      );
		                      messageManager.addMessage(
		                              new CommandFire(false,
		                                      StandardMessagePriority.HIGH, null,
		                                      building.getID(),
		                                      CommandFire.ACTION_EXTINGUISH)
		                      );
		                  }
		  
		              }
		          }
		      } else if (agentInfo.me().getStandardURN() == FIRE_BRIGADE) {
		          logger.info("aaa");
		          int currentTime = agentInfo.getTime();
		          Human me = (Human) agentInfo.me();
		          int agentX = me.getX();
		          int agentY = me.getY();
		          StandardEntity positionEntity = worldInfo.getPosition(me);
		          if (positionEntity instanceof Road)//以下处理消防员被障碍卡住的情况
		          {
		              boolean isSendRequest = false;
		              logger.info("bbb");
		              Road road = (Road) positionEntity;
		              // 如果我被障碍卡住就求救
		              if (road.isBlockadesDefined() && road.getBlockades().size() > 0) {
		                  for (Blockade blockade : worldInfo.getBlockades(road)) {
		                      if (blockade == null || !blockade.isApexesDefined()) {
		                          continue;
		                      }
		  
		                      if (this.isInside(agentX, agentY,
		                              blockade.getApexes())) {
		                          isSendRequest = true;
		                          logger.info("ccc");
		                      }
		                  }
		                  if (isSendRequest) {
		                      logger.info(me.getID() + "------被卡住了-_---_---_---");
		                  }
		              }
		              // 如果我连续几个周期都在同一条路上，也说明我被卡住了
		              if (this.lastPosition != null && this.lastPosition.getValue()
		                  == road.getID().getValue()) {
		                  this.stayCount++;
		                  if (this.stayCount > this.getMaxTravelTime(road)) {
		                      isSendRequest = true;
		                      logger.info("ddd");
		                  }
		              } else {
		                  logger.info("eee");
		                  this.lastPosition = road.getID();
		                  this.stayCount = 0;
		              }
		  
		              // 如果我被障碍卡住了，我就给消防中心智能体发消息
		              if (isSendRequest) {
		                  logger.info(agentInfo.getID() + "卡住了1");
		                  messageManager.addMessage(
		                          new CommandFire(true,
		                                  StandardMessagePriority.HIGH, null, null, 5)
		                  );
		              }
		              if (isSendRequest && ((currentTime - this.lastSentTimePolice)
		                                    >= this.sendingAvoidTimeClearRequest)) {
		                  this.lastSentTimePolice = currentTime;
		                  logger.info(agentInfo.getID() + "卡住了2");
		                  if (this.entrance2building.keySet().contains(me.getPosition())) {
		                      EntityID building =
		                              this.entrance2building.get(me.getPosition());
		                      messageManager.addMessage(
		                              new CommandPolice(true,
		                                      StandardMessagePriority.HIGH, null,
		                                      building, CommandPolice.ACTION_CLEAR)
		                      );
		                      messageManager.addMessage(
		                              new CommandPolice(false,
		                                      StandardMessagePriority.HIGH, null,
		                                      building, CommandPolice.ACTION_CLEAR)
		                      );
		                  } else {
		                      messageManager.addMessage(
		                              new CommandPolice(true,
		                                      StandardMessagePriority.HIGH, null,
		                                      me.getPosition(),
		                                      CommandPolice.ACTION_CLEAR)
		                      );
		                      messageManager.addMessage(
		                              new CommandPolice(false,
		                                      StandardMessagePriority.HIGH, null,
		                                      me.getPosition(),
		                                      CommandPolice.ACTION_CLEAR)
		                      );
		                  }
		              }
		          }
		          if (me.isBuriednessDefined() && me.getBuriedness() > 0) { //
		              // 以下处理消防员被掩埋的情况
		              messageManager.addMessage(
		                      new CommandFire(true, StandardMessagePriority.HIGH,
		                              null, null, 5)
		              );
		              if ((currentTime - this.lastSentTimeAmbulance)
		                  >= this.sendingAvoidTimeRescueRequest) {
		  //                    messageManager.addMessage(
		  //                            new CommandAmbulance(true,
		  //                            StandardMessagePriority.HIGH, null, me.getID(),
		  //                            CommandAmbulance.ACTION_RESCUE)
		  //                    );
		  //                    messageManager.addMessage(
		  //                            new CommandAmbulance(false,
		  //                            StandardMessagePriority.HIGH, null, me.getID(),
		  //                            CommandAmbulance.ACTION_RESCUE)
		  //                    );
		                  this.lastSentTimeAmbulance = currentTime;
		              }
		          }
		          //以下处理消防员被困在建筑物中的情况
		          if (positionEntity instanceof Building) {
		              Building building = (Building) positionEntity;
		              boolean entranceBlock = false;
		              EntityID entrance =
		                  this.getClosestEntityID(this.getEntrancesOfBuilding(building),
		                                          agentInfo.getID());
		              if (entrance != null
		                  && !this.isGetCross((Area) this.worldInfo.getEntity(entrance))) {
		                  entranceBlock = true;
		              }
		              // 被困在建筑，给消防中心智能体发消息
		              if (entranceBlock) {
		                  messageManager.addMessage(
		                          new CommandFire(true,
		                                  StandardMessagePriority.HIGH, null, null, 5)
		                  );
		              }
		              if (entranceBlock && ((currentTime - this.lastSentTimePolice)
		                                    >= this.sendingAvoidTimeClearRequest)) {
		                  this.lastSentTimePolice = currentTime;
		                  messageManager.addMessage(
		                          new CommandPolice(true,
		                                  StandardMessagePriority.HIGH, null,
		                                  building.getID(),
		                                  CommandPolice.ACTION_CLEAR)
		                  );
		                  messageManager.addMessage(
		                          new CommandPolice(false,
		                                  StandardMessagePriority.HIGH, null,
		                                  building.getID(),
		                                  CommandPolice.ACTION_CLEAR)
		                  );
		              }
		              // 消防员被掩埋在着火且出不去的建筑的情况
		              if (entranceBlock
		                  && isOnFire(building)
		                  && me.isBuriednessDefined()
		                  && me.getBuriedness() > 0) {
		                  if ((currentTime - this.lastSentTimeFire)
		                      >= this.sendingAvoidTimeExtinguishRequest) {
		                      messageManager.addMessage(
		                              new CommandFire(true,
		                                      StandardMessagePriority.HIGH, null,
		                                      building.getID(),
		                                      CommandFire.ACTION_EXTINGUISH)
		                      );
		                      messageManager.addMessage(
		                              new CommandFire(false,
		                                      StandardMessagePriority.HIGH, null,
		                                      building.getID(),
		                                      CommandFire.ACTION_EXTINGUISH)
		                      );
		                      this.lastSentTimeFire = currentTime;
		                  }
		              }
		          }
		      }
		  }
		  ```
	- Set< [[EntityID]] > getEntrancesOfBuilding([[Building]] building)
		- 使用 BFS，以建筑为中心逐层向外搜索，寻找建筑物的入口
		-
		  ``` java
		  private Set<EntityID> getEntrancesOfBuilding(Building building) {
		          if (building == null) {
		              return new HashSet<>();
		          }
		          Set<EntityID> entrances = new HashSet<>();
		          Set<EntityID> visitedID = new HashSet<>();
		          Stack<EntityID> stack = new Stack<>();
		          stack.push(building.getID());
		          visitedID.add(building.getID());
		          while (!stack.isEmpty()) {
		              EntityID entityID = stack.pop();
		              visitedID.add(entityID);
		  		  
		              StandardEntity standardEntity = this.worldInfo.getEntity(entityID);
		              if (standardEntity == null || !(standardEntity instanceof Area)) {
		                  return null;
		              }
		              Area area = (Area) standardEntity;
		              if (area instanceof Road) {
		                  entrances.add(entityID);
		              } else {
		                  for (EntityID neighborID : area.getNeighbours()) {
		                      if (visitedID.contains(neighborID)) {
		                          continue;
		                      }
		                      stack.push(neighborID);
		                  }
		              }
		          }
		          return entrances;
		      }
		  ```