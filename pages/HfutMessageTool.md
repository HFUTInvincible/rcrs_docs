## Intro
## Methods
	- void reflectMessage( [[AgentInfo]] agentInfo, 
	  \                   [[WorldInfo]] worldInfo,
	  \                   [[ScenarioInfo]] scenarioInfo,
	  \                   [[MessageManager]] messageManager)
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
	- public void sendInformationMessages( [[AgentInfo]] agentInfo,
	  \                                   [[WorldInfo]] worldInfo,
	  \                                   [[ScenarioInfo]] scenarioInfo,
	  \                                   [[MessageManager]] messageManager)
		- 报告智能体的最新情况，也就是发送 Information Message
	- public void sendRequestMessages( [[AgentInfo]] agentInfo,
	  \                               [[WorldInfo]] worldInfo, 
	  \                               [[ScenarioInfo]] scenarioInfo,
	  \                               [[MessageManager]] messageManager)
		- 发送求救信息，比如被卡住了，就请警察来清障；被火困住了，就请消防来灭火
		-
	- Set< [[EntityID]] > getEntrancesOfBuilding( [[Building]] building)
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