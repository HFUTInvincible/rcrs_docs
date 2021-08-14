## Intro
	- 这个类就是一个用来 reflectMessage 的工具类
	- 所谓 reflectMessage，就是根据一条 Information Message，还原出一个 entity 来
	- 可以用 Java 的反射类比
## Methods
	- 实现非常简单，首先对每种 Entity 都写一个 reflectMessage 函数，然后汇总到通用函数里
	-
	  ```java
	  // 给 MessageRoad 用的 reflectMessage
	  @Nonnull
	  public static Building reflectMessage(@Nonnull WorldInfo worldInfo, @Nonnull MessageBuilding message) {
	      Building building = (Building)worldInfo.getEntity(message.getBuildingID());
	      if (building != null) {
	          if (message.isFierynessDefined()) {
	              building.setFieryness(message.getFieryness());
	          }
	  
	          if (message.isBrokennessDefined()) {
	              building.setBrokenness(message.getBrokenness());
	          }
	  
	          if (message.isTemperatureDefined()) {
	              building.setTemperature(message.getTemperature());
	          }
	      } else {
	          building = new Building(message.getBuildingID());
	          if (message.isFierynessDefined()) {
	              building.setFieryness(message.getFieryness());
	          }
	  
	          if (message.isBrokennessDefined()) {
	              building.setBrokenness(message.getBrokenness());
	          }
	  
	          if (message.isTemperatureDefined()) {
	              building.setTemperature(message.getTemperature());
	          }
	  
	          worldInfo.addEntity(building);
	      }
	  
	      return building;
	  }
	  ```
	-
	  ```java
	  // 通用的 reflectMessage 函数
	  // 根据类型的不同调用各自的 reflectMessage 函数
	  @Nullable
	  public static StandardEntity reflectMessage(@Nonnull WorldInfo worldInfo, @Nonnull StandardMessage message) {
	      StandardEntity entity = null;
	      Set<EntityID> changedEntities = worldInfo.getChanged().getChangedEntities();
	      Class<? extends StandardMessage> messageClass = message.getClass();
	      if (messageClass == MessageCivilian.class) {
	          MessageCivilian mc = (MessageCivilian)message;
	          if (!changedEntities.contains(mc.getAgentID())) {
	              entity = reflectMessage(worldInfo, mc);
	          }
	      } else if (messageClass == MessageAmbulanceTeam.class) {
	          MessageAmbulanceTeam mat = (MessageAmbulanceTeam)message;
	          if (!changedEntities.contains(mat.getAgentID())) {
	              entity = reflectMessage(worldInfo, mat);
	          }
	      } else if (messageClass == MessageFireBrigade.class) {
	          MessageFireBrigade mfb = (MessageFireBrigade)message;
	          if (!changedEntities.contains(mfb.getAgentID())) {
	              entity = reflectMessage(worldInfo, mfb);
	          }
	      } else if (messageClass == MessagePoliceForce.class) {
	          MessagePoliceForce mpf = (MessagePoliceForce)message;
	          if (!changedEntities.contains(mpf.getAgentID())) {
	              entity = reflectMessage(worldInfo, mpf);
	          }
	      } else if (messageClass == MessageBuilding.class) {
	          MessageBuilding mb = (MessageBuilding)message;
	          if (!changedEntities.contains(mb.getBuildingID())) {
	              entity = reflectMessage(worldInfo, mb);
	          }
	      } else if (messageClass == MessageRoad.class) {
	          MessageRoad mr = (MessageRoad)message;
	          if (!changedEntities.contains(mr.getRoadID())) {
	              entity = reflectMessage(worldInfo, mr);
	          }
	      }
	  
	      return (StandardEntity)entity;
	  }
	  ```