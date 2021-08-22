## Intro
	- 通讯的基础类，每个类都有一个 MessageManager 实例，收发信息都通过它完成
## Fields
	- ArrayList<[[CommunicationMessage]]> sendMessageList
		- 要发送的消息
	- List<List<[[CommunicationMessage]]>> channelSendMessageList
		- 分类后的消息
	- List<[[CommunicationMessage]]> receivedMessageList
		- 这个周期收到的消息
	- [[MessageCoordinator]] messageCoordinator
	- [[ChannelSubscriber]] channelSubscriber
	- int[] subscribedChannels
		- 这个智能体订阅的信道
## Methods
	- void addMessage(@Nonnull [[CommunicationMessage]] message)
	  id:: 6120c43f-36a3-4837-bd10-084468f9496d
		- 添加要发送的消息
		-
		  ```java
		  public void addMessage(@Nonnull CommunicationMessage message) {
		      this.addMessage(message, true);
		  }
		  ```
	- void addMessage(@Nonnull [[CommunicationMessage]] message, boolean checkDuplication)
		-
		  ```java
		  CommunicationMessagepublic void addMessage(@Nonnull CommunicationMessage message, boolean checkDuplication){
		      if (message != null) {
		          String checkKey = message.getCheckKey();
		          if (checkDuplication && !this.checkDuplicationCache.contains(checkKey)) {
		              this.sendMessageList.add(message);
		              this.checkDuplicationCache.add(checkKey);
		          } else {
		              this.sendMessageList.add(message);
		              this.checkDuplicationCache.add(checkKey);
		          }
		  
		      }
		  }
		  ```
	- void subscribe([[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, [[ScenarioInfo]] scenarioInfo)
		- 调用 [[ChannelSubscriber]] 订阅信道
		-
		  ```java
		  public void subscribe(AgentInfo agentInfo, WorldInfo worldInfo,
		                        ScenarioInfo scenarioInfo) {
		      // Use subscribe we implemented in ChannelSubscriber
		      if (this.channelSubscriber != null) {
		          this.channelSubscriber.subscribe(agentInfo, worldInfo, scenarioInfo, this);
		      }
		  }
		  ```
	- List<[[CommunicationMessage]]> getReceivedMessageList()
	  id:: 6120c6b5-3327-4bf3-8cf5-f74e211c25c3
		- 获得这一周期收到的所有消息
		-
		  ```java
		      @Nonnull
		      public List<CommunicationMessage> getReceivedMessageList() {
		          return this.receivedMessageList;
		      }
		  ```
	- void coordinateMessages([[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, [[ScenarioInfo]] scenarioInfo)
	  id:: 6120c5ac-26b8-4854-ae03-fc3a6d9a6547
		- 调用 [[MessageCoordinator]] 对消息进行分类投放
		-
		  ```java
		  public void coordinateMessages(AgentInfo agentInfo, WorldInfo worldInfo,
		                                 ScenarioInfo scenarioInfo) {
		      // Initialize channelSendMessageList
		      this.channelSendMessageList =
		        	new ArrayList(scenarioInfo.getCommsChannelsCount());
		  
		      for(int i = 0; i < scenarioInfo.getCommsChannelsCount(); ++i) {
		          this.channelSendMessageList.add(new ArrayList());
		      }
		  
		      // Use coordinate we implemented in MessageCoordinator
		      if (this.messageCoordinator != null) {
		          this.messageCoordinator.coordinate(agentInfo, worldInfo, scenarioInfo, this, this.sendMessageList, this.channelSendMessageList);
		      }
		  }
		  ```