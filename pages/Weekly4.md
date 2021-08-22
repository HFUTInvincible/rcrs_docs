## Abstract
	- 看了一遍学长们的总结，感觉对整个项目的结构更清楚了
	- 通讯相关的类全部看了一遍
## What is agent?
	- 之前一直以为 [[Agent]] 就是各种 [[Entity]]，后来才发现，其实在 [[Entity]] 层之上还有一层封装
	-
	  ```java
	  public abstract class Agent<E extends StandardEntity>
	  	extends AbstractAgent<StandardWorldModel, E> {
	  	...
	  }
	  ```
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
	- 传说中排智能体的名字也是从这里的 [[Platoon]] 来的
	- [[Agent]] 里面的 think(..) 方法指明了所有智能体在每个周期都要做的事情
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
	- 注意到，这个方法里面还调用了一个无参的 think() 方法，该 think() 方法在 [[Agent]] 中被定义为抽象方法，在子类 [[Office]] 和 [[Platoon]] 中得到实现
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
	- Tactics 中定义的每个智能体的 think() 方法也是在这里被调用的
## About Communication
	- 关于通信，学长的那个总结已经很全面了，我就再结合我自己看的代码写点东西
	- {{embed [[MessageManager]] }}
	- {{embed [[ChannelSubscriber]]}}
	- {{embed [[MessageCoordinator]] }}
	- {{embed [[HfutMessageTool]]}}