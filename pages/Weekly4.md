- [[AbstractComponent]] (rescuecore2.components)
	- [[AbstractAgent]] (rescuecore2.components)
		- [[Agent]] (adf.agent)
			- [[Office]] (adf.agent.office)
				- [[OfficeAmbulance]] (adf.agent.office)
				- [[OfficeFire]] (adf.agent.office)
				- [[OfficePolice]] (adf.agent.office)
			- [[Platoon]] (adf.agent.platoon)
				- [[PlatoonAmbulance]] (adf.agent.platoon)
				- [[PlatoonFire]] (adf.agent.platoon)
				- [[PlatoonPolice]] (adf.agent.platoon)
- [[Agent]] 层是对 [[Entity]] 层的又一次封装
-
  ```java
  public abstract class Agent<E extends StandardEntity>
  	extends AbstractAgent<StandardWorldModel, E> {
  	...
  }
  ```
- 其非抽象的 think(...) 方法指明了所有智能体在每个周期都要做的事情
-
  ```java
      protected void think(int time, ChangeSet changed, Collection<Command> heard) {
          this.agentInfo.recordThinkStartTime();
          this.agentInfo.setTime(time);
          if (1 == time) {
              if (this.communicationModule != null) {
                  ConsoleOutput.out(State.ERROR, "[ERROR ] Loader is not found.");
                  ConsoleOutput.out(State.NOTICE, "CommunicationModule is modified - " + this);
              } else {
                  this.communicationModule = new StandardCommunicationModule();
              }
  
              this.messageManager.registerMessageBundle(new StandardMessageBundle());
          }
  
          if (time >= this.ignoreTime) {
              this.messageManager.subscribe(this.agentInfo, this.worldInfo, this.scenarioInfo);
              if (!this.messageManager.getIsSubscribed()) {
                  int[] channelsToSubscribe = this.messageManager.getChannels();
                  if (channelsToSubscribe != null) {
                      super.send(new AKSubscribe(this.getID(), time, channelsToSubscribe));
                      this.messageManager.setIsSubscribed(true);
                  }
              }
          }
  
          this.agentInfo.setHeard(heard);
          this.agentInfo.setChanged(changed);
          this.worldInfo.setChanged(changed);
          this.messageManager.refresh();
          this.communicationModule.receive(this, this.messageManager);
  
          try {
              this.think();
          } catch (Exception var5) {
              var5.printStackTrace();
          }
  
          this.messageManager.coordinateMessages(this.agentInfo, this.worldInfo, this.scenarioInfo);
          this.communicationModule.send(this, this.messageManager);
      }
  ```
- 注意到还调用了一个无参的 think() 方法，该 think() 方法在 [[Agent]] 中被定义为抽象方法，在子类 [[Office]] 和 [[Platoon]] 中得到实现
-
  ```java
  // Abstract, In Agent
  protected abstract void think();
  
  // In Office
  protected void think() {
      this.rootTacticsCenter.think(this.agentInfo, this.worldInfo,
                                   this.scenarioInfo, this.moduleManager,
                                   this.messageManager, this.developData);
  }
  
  // In Platoon
  protected void think() {
      Action action = this.rootTactics.think(this.agentInfo, this.worldInfo,
                                             this.scenarioInfo, this.moduleManager,
                                             this.messageManager, this.developData);
      if (action != null) {
          this.agentInfo.setExecutedAction(this.agentInfo.getTime(), action);
          this.send(action.getCommand(this.getID(), this.agentInfo.getTime()));
      }
  }
  ```
- 注意到，Tactics 中定义的每个智能体的 think() 方法在这里被调用
- 信道
	- 0 信道是声音信道 (isRadio = False)
	- 其他信道是无线电信道 (isRadio = True)
-
  ```java
  protected int[] getChannelsByAgentType(StandardEntityURN agentType, AgentInfo agentInfo,
                                         WorldInfo worldInfo, ScenarioInfo scenarioInfo,
                                         int channelIndex) {
      int numChannels = scenarioInfo.getCommsChannelsCount()-1;
      int maxChannelCount = 0;
      boolean isPlatoon = isPlatoonAgent(agentInfo, worldInfo);
      if (isPlatoon) {
          maxChannelCount = scenarioInfo.getCommsChannelsMaxPlatoon();
      } else {
          maxChannelCount = scenarioInfo.getCommsChannelsMaxOffice();
      }
      int[] channels = new int[maxChannelCount];
  
      for (int i = 0; i < maxChannelCount; i++) {
          channels[i] = SampleChannelSubscriber.getChannelNumber(agentType, i, numChannels);
      }
      return channels;
  }
  ```
- 我们写的 MessageCoordinator 中的 subscribe(...) 方法在这里被调用
-
  ```java
  // In MessageManager, Called in think(...) in Agent
  public void subscribe(AgentInfo agentInfo, WorldInfo worldInfo, ScenarioInfo scenarioInfo) {
      if (this.channelSubscriber != null) {
          this.channelSubscriber.subscribe(agentInfo, worldInfo, scenarioInfo, this);
      }
  
  }
  ```
- 我们写的 ChannelSubscriber 中的 coordinate(...) 方法在这里被调用
-
  ```java
  // In MessageManager, Called in think(...) in Agent
  public void coordinateMessages(AgentInfo agentInfo, WorldInfo worldInfo, ScenarioInfo scenarioInfo) {
      this.channelSendMessageList = new ArrayList(scenarioInfo.getCommsChannelsCount());
  
      for(int i = 0; i < scenarioInfo.getCommsChannelsCount(); ++i) {
          this.channelSendMessageList.add(new ArrayList());
      }
  
      if (this.messageCoordinator != null) {
          this.messageCoordinator.coordinate(agentInfo, worldInfo, scenarioInfo,
                                             this, this.sendMessageList,
                                             this.channelSendMessageList);
      }
  
  }
  ```