## Intro
## Methods
	- void subscribe([[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, [[ScenarioInfo]] scenarioInfo, [[MessageManager]] messageManager)
		-
		  ```java
		  @Override
		  public void subscribe(AgentInfo agentInfo, WorldInfo worldInfo, ScenarioInfo scenarioInfo,
		                        MessageManager messageManager) {
		    
		      // System.out.println("time:"+agentInfo.getTime());
		      // System.out.println("This is SampleChannelSubscriber");
		    
		      // 这个函数每个智能体每周期都会执行一次，但只有在第三周期会分配信道，其他周期什么也不干
		      if (agentInfo.getTime() == 3) {
		          int numChannels = scenarioInfo.getCommsChannelsCount()-1;
		  
		          // 计算当前智能体最多可用多少条信道
		          int maxChannelCount = 0;
		          boolean isPlatoon = isPlatoonAgent(agentInfo, worldInfo);
		          if (isPlatoon) {
		              maxChannelCount = scenarioInfo.getCommsChannelsMaxPlatoon();
		          } else {
		              maxChannelCount = scenarioInfo.getCommsChannelsMaxOffice();
		          }
		  
		          System.out.println("我:"+agentInfo.me().toString());
		  
		  		// 调用 getChannelNumber 方法，计算当前智能体能使用的第 i 条信道应该是所有信道中的那一条
		          StandardEntityURN agentType = getAgentType(agentInfo, worldInfo);
		          int[] channels = new int[maxChannelCount];
		          for (int i = 0; i < maxChannelCount; i++) {
		              channels[i] = getChannelNumber(agentType, i, numChannels);
		              System.out.println("i:"+i+", channel:"+channels[i]);
		          }
		  
		          messageManager.subscribeToChannels(channels);
		  
		      }
		  //        if (agentInfo.getTime() == 50) {
		  //            int numChannels = scenarioInfo.getCommsChannelsCount()-1; // 0th channel is the voice channel
		  //
		  //            // 最大订阅信道数，即一个智能体最多订阅多少个信道。
		  //            int maxChannelCount = 0;
		  //            boolean isPlatoon = isPlatoonAgent(agentInfo, worldInfo);
		  //            if (isPlatoon) {
		  //                maxChannelCount = scenarioInfo.getCommsChannelsMaxPlatoon();
		  //            } else {
		  //                maxChannelCount = scenarioInfo.getCommsChannelsMaxOffice();
		  //            }
		  //
		  //            System.out.println("我:"+agentInfo.me().toString()+"  这是第二次订阅");
		  //
		  //
		  //            StandardEntityURN agentType = getAgentType(agentInfo, worldInfo);
		  //            int[] channels = new int[maxChannelCount];
		  //            for (int i = 0; i < maxChannelCount; i++) {
		  //                channels[i] = getChannelNumber2(agentType, i, numChannels);
		  //                System.out.println("i:"+i+", channel:"+channels[i]);
		  //            }
		  //
		  //            messageManager.subscribeToChannels(channels);
		  //
		  //        }
		  }
		  ```
	- int getChannelNumber([[StandardEntityURN]] agentType, int channelIndex, int numChannels)
		- 计算当前智能体的第 channelIndex 条信道应该被分配哪条信道
		- 分配策略
			- (消防，警察，救护)，(消防，警察，救护)，... 三个一轮分配即可
		- [[FIXME]] 这是个静态方法！
		- [[FIXME]] 如果没有消防，会不会有信道闲置？
		- [[FIXME]] 当信道无法平均分配时，是否应该优先将信道分配给更需要的智能体？
		-
		  ```java
		  public static int getChannelNumber(StandardEntityURN agentType,
		                                     int channelIndex, int numChannels) {
		      // System.out.println("this is getChannelNumber()");
		      int agentIndex = 0;
		      if (agentType == StandardEntityURN.FIRE_BRIGADE ||
		              agentType == StandardEntityURN.FIRE_STATION) {
		          agentIndex = 1;
		      } else if (agentType == StandardEntityURN.POLICE_FORCE ||
		                     agentType == StandardEntityURN.POLICE_OFFICE) {
		          agentIndex = 2;
		      } else if (agentType == StandardEntityURN.AMBULANCE_TEAM ||
		                     agentType == StandardEntityURN.AMBULANCE_CENTRE) {
		          agentIndex = 3;
		      }
		  
		      int index = (3*channelIndex)+agentIndex;
		      // 防止循环
		      if(numChannels == 3){
		          index = (2*channelIndex)+agentIndex;
		      }
		      if ((index%numChannels) == 0) {
		          index = numChannels;
		      } else {
		          index = index % numChannels;
		      }
		      return index;
		  }
		  ```