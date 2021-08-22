## Intro
	- 对 [[MessageManager]] 里要发送的信息进行分类，投放到合适的信道
## Fields
## Methods
	- void coordinate([[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, [[ScenarioInfo]] scenarioInfo, [[MessageManager]] messageManager, ArrayList<[[CommunicationMessage]]> sendMessageList, List<List<[[CommunicationMessage]]>> channelSendMessageList)
		- 滤去原有的 HfutMessageTool 发送的消息，并将有效消息分类
		-
		  ```java
		  @Override
		      public void coordinate(AgentInfo agentInfo, WorldInfo worldInfo, ScenarioInfo scenarioInfo, MessageManager messageManager,
		                         ArrayList<CommunicationMessage> sendMessageList, List<List<CommunicationMessage>> channelSendMessageList) {
		  
		      logger =TestLogger.getLogger("Coordinator/"+agentInfo.me());
		  
		      logger.info("----------------------------time:"+agentInfo.getTime()+"----------------------------");
		  
		      // have different lists for every agent
		      ArrayList<CommunicationMessage> policeMessages = new ArrayList<>();
		      ArrayList<CommunicationMessage> ambulanceMessages = new ArrayList<>();
		      ArrayList<CommunicationMessage> fireBrigadeMessages = new ArrayList<>();
		  
		      ArrayList<CommunicationMessage> voiceMessages = new ArrayList<>();//声音通讯
		  
		      StandardEntityURN agentType = getAgentType(agentInfo, worldInfo);
		      logger.info("我想发送的所有消息：");
		      //从信息列表里面给信息分类，分别添加到相应的集合里
		      logger.info(sendMessageList.size());
		      for (CommunicationMessage msg : sendMessageList) {
		          logger.info(msg);
		          if (msg instanceof StandardMessage && !msg.isRadio()) {
		              voiceMessages.add(msg);//声音通讯消息集合
		          } else {
		              if(useOffice){
		                  if(msg instanceof StandardMessage){ //把所有的原始MessageTool的信息过滤掉
		                      StandardMessage smsg = (StandardMessage) msg;
		                      if(smsg.getSendingPriority() == StandardMessagePriority.LOW){
		                          continue;
		                      }
		                      if((smsg.getSendingPriority() == StandardMessagePriority.NORMAL) &&
		                              (smsg instanceof CommandPolice || smsg instanceof CommandFire || smsg instanceof CommandAmbulance)){
		                          continue;
		                      }
		                  }
		                  //世界模型中的信息消息正常发送
		                  if (msg instanceof MessageBuilding) {
		                      fireBrigadeMessages.add(msg);
		                  } else if (msg instanceof MessageCivilian) {
		                      ambulanceMessages.add(msg);
		                  } else if (msg instanceof MessageRoad) {
		                      fireBrigadeMessages.add(msg);
		                      ambulanceMessages.add(msg);
		                      policeMessages.add(msg);
		                  }
		                  // 正常发送的智能体命令消息(其中命令消息CommandAmbulance/CommandFire/CommandPolice是High等级的)
		                  /*******************
		                  命令类消息分四类:CommandAmbulance/CommandFire/CommandPolice/CommandScout
		                   其中CommandScout一定是中心智能体发送的;而其他三类可能是中心智能体发送给排智能体的,也有可能是其他排智能体发送排智能体的
		                  *********************/
		                  else if (msg instanceof CommandAmbulance) {
		                      ambulanceMessages.add(msg);
		                  } else if (msg instanceof CommandFire) {
		                      fireBrigadeMessages.add(msg);
		                  } else if (msg instanceof CommandPolice) {
		                      policeMessages.add(msg);
		                  } else if (msg instanceof CommandScout) {
		                      if (agentType == StandardEntityURN.FIRE_STATION) {
		                          fireBrigadeMessages.add(msg);
		                      } else if (agentType == StandardEntityURN.POLICE_OFFICE) {
		                          policeMessages.add(msg);
		                      } else if (agentType == StandardEntityURN.AMBULANCE_CENTRE) {
		                          ambulanceMessages.add(msg);
		                      }
		                  }
		                  //正常发送的排智能体报告消息
		                  else if (msg instanceof MessageReport) {
		                      if (agentType == StandardEntityURN.FIRE_BRIGADE) {
		                          fireBrigadeMessages.add(msg);
		                      } else if (agentType == StandardEntityURN.POLICE_FORCE) {
		                          policeMessages.add(msg);
		                      } else if (agentType == StandardEntityURN.AMBULANCE_TEAM) {
		                          ambulanceMessages.add(msg);
		                      }
		                  }
		                  //排智能体的信息消息正常发送
		                  else if (msg instanceof MessageFireBrigade) {
		                      fireBrigadeMessages.add(msg);
		      //                        ambulanceMessages.add(msg);
		      //                        policeMessages.add(msg);
		                  } else if (msg instanceof MessagePoliceForce) {
		      //                        ambulanceMessages.add(msg);
		                      policeMessages.add(msg);
		                  } else if (msg instanceof MessageAmbulanceTeam) {
		                      ambulanceMessages.add(msg);
		      //                        policeMessages.add(msg);
		                  }
		              }
		  
		              else {//如果不使用中心智能体，则拒绝发送一部分类型的信息以节省信道空间
		                  if(msg instanceof StandardMessage){ //把所有的原始MessageTool的信息过滤掉
		                      StandardMessage smsg = (StandardMessage) msg;
		                      if(smsg.getSendingPriority() == StandardMessagePriority.LOW){
		                          continue;
		                      }
		                      if(smsg.getSendingPriority() == StandardMessagePriority.NORMAL &&
		                              (smsg instanceof CommandPolice || smsg instanceof CommandFire || smsg instanceof CommandAmbulance)){
		                          continue;
		                      }
		                  }
		                  // 世界模型中的信息消息正常发送
		                  if (msg instanceof MessageBuilding) {
		                      fireBrigadeMessages.add(msg);
		                  } else if (msg instanceof MessageCivilian) {
		                      ambulanceMessages.add(msg);
		                  } else if (msg instanceof MessageRoad) {
		                      fireBrigadeMessages.add(msg);
		                      ambulanceMessages.add(msg);
		                      policeMessages.add(msg);
		                  }
		                  // 正常发送的智能体命令消息(其中命令消息CommandAmbulance/CommandFire/CommandPolice是High等级的)
		                  else if (msg instanceof CommandAmbulance) {
		                      ambulanceMessages.add(msg);
		                  } else if (msg instanceof CommandFire) {
		                      fireBrigadeMessages.add(msg);
		                  } else if (msg instanceof CommandPolice) {
		                      policeMessages.add(msg);
		                  } else if (msg instanceof CommandScout) {
		                      if (agentType == StandardEntityURN.FIRE_STATION) {
		                          fireBrigadeMessages.add(msg);
		                      } else if (agentType == StandardEntityURN.POLICE_OFFICE) {
		                          policeMessages.add(msg);
		                      } else if (agentType == StandardEntityURN.AMBULANCE_CENTRE) {
		                          ambulanceMessages.add(msg);
		                      }
		                  }
		                  // 上报消息和工作状态消息都弃用
		                  else if (msg instanceof MessageReport) {
		      //                        if (agentType == StandardEntityURN.FIRE_BRIGADE) {
		      //                            fireBrigadeMessages.add(msg);
		      //                        } else if (agentType == StandardEntityURN.POLICE_FORCE) {
		      //                            policeMessages.add(msg);
		      //                        } else if (agentType == StandardEntityURN.AMBULANCE_TEAM) {
		      //                            ambulanceMessages.add(msg);
		      //                        }
		                  } else if (msg instanceof MessageFireBrigade) {
		      //                    fireBrigadeMessages.add(msg);
		      //                    ambulanceMessages.add(msg);
		      //                    policeMessages.add(msg);
		                  } else if (msg instanceof MessagePoliceForce) {
		      //                    ambulanceMessages.add(msg);
		      //                    policeMessages.add(msg);
		                  } else if (msg instanceof MessageAmbulanceTeam) {
		      //                    ambulanceMessages.add(msg);
		      //                    policeMessages.add(msg);
		                  }
		              }
		  
		          }
		      }
		  
		      //如果信道数大于一，就根据智能体类型发送相应信道信息
		      if (scenarioInfo.getCommsChannelsCount() > 1) {
		          // send radio messages if there are more than one communication channel
		          int[] channelSize = new int[scenarioInfo.getCommsChannelsCount() - 1];
		          logger.info("下面处理 POLICE_FORCE 的信息");
		          setSendMessages(scenarioInfo, StandardEntityURN.POLICE_FORCE, agentInfo, worldInfo, policeMessages,
		                  channelSendMessageList, channelSize);
		          logger.info("下面处理 AMBULANCE_TEAM 的信息");
		          setSendMessages(scenarioInfo, StandardEntityURN.AMBULANCE_TEAM, agentInfo, worldInfo, ambulanceMessages,
		                  channelSendMessageList, channelSize);
		          logger.info("下面处理 FIRE_BRIGADE 的信息");
		          setSendMessages(scenarioInfo, StandardEntityURN.FIRE_BRIGADE, agentInfo, worldInfo, fireBrigadeMessages,
		                  channelSendMessageList, channelSize);
		  
		      }
		      logger.info("\n");
		  
		      //下面是处理喊的信息，即非信道信息
		      ArrayList<StandardMessage> voiceMessageLowList = new ArrayList<>();
		      ArrayList<StandardMessage> voiceMessageNormalList = new ArrayList<>();
		      ArrayList<StandardMessage> voiceMessageHighList = new ArrayList<>();
		  
		      for (CommunicationMessage msg : voiceMessages) {
		          if (msg instanceof StandardMessage) {
		              StandardMessage m = (StandardMessage) msg;
		              switch (m.getSendingPriority()) {
		                  case LOW:
		                      voiceMessageLowList.add(m);
		                      break;
		                  case NORMAL:
		                      voiceMessageNormalList.add(m);
		                      break;
		                  case HIGH:
		                      voiceMessageHighList.add(m);
		                      break;
		              }
		          }
		      }
		  
		      // 声音信道带宽无限，所有消息全部塞进去即可
		      channelSendMessageList.get(0).addAll(voiceMessageHighList);
		      channelSendMessageList.get(0).addAll(voiceMessageNormalList);
		      channelSendMessageList.get(0).addAll(voiceMessageLowList);
		  }
		  ```
	- void setSendMessages([[ScenarioInfo]] scenarioInfo, [[StandardEntityURN]] agentType, [[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, List<[[CommunicationMessage]]> messages, List<List<[[CommunicationMessage]]>> channelSendMessageList, int[] channelSize)
		- 根据智能体类型、消息类型及信道情况，将消息投放到智能体的信道中
		-
		  ```java
		  protected void setSendMessages(ScenarioInfo scenarioInfo, StandardEntityURN agentType, AgentInfo agentInfo,
		                                 WorldInfo worldInfo, List<CommunicationMessage> messages,
		                                 List<List<CommunicationMessage>> channelSendMessageList,
		                                 int[] channelSize) {
		  //        logger.info("this is setSendMessages");
		      int channelIndex = 0;
		      // 计算该agentType的消息需要发送到哪一个信道
		      int[] channels = getChannelsByAgentType(agentType, agentInfo, worldInfo, scenarioInfo, channelIndex);
		      int channel = channels[channelIndex];
		      int channelCapacity = scenarioInfo.getCommsChannelBandwidth(channel);
		      // start from HIGH, NORMAL, to LOW
		      for (int i = StandardMessagePriority.values().length-1; i >= 0; i--) {
		          logger.info("当前优先级************************************* "+(new Integer(i)).toString());
		          //先处理高优先级的，再处理低优先级的消息
		          for (CommunicationMessage msg : messages) {
		              StandardMessage smsg = (StandardMessage) msg;
		              if (smsg.getSendingPriority() == StandardMessagePriority.values()[i]) {
		                  //往指定信道里面加一些字节的信息
		                  channelSize[channel-1] += smsg.getByteArraySize();
		                  logger.info("当前消息:"+toPrintStandardMessage(smsg));
		                  logger.info("当前信道为："+channel+",信道已经使用了："+channelSize[channel-1]);
		                  //如果已经超过了信道容量
		                  if (channelSize[channel-1] > channelCapacity) {
		                      channelSize[channel-1] -= smsg.getByteArraySize();
		                      logger.info("信道已经满了，我已经发送："+channelSize[channel-1]+"字节消息");
		                      //当前信道已经满了，换下一个信道
		                      channelIndex++;
		                      //如果还有下一个已经订阅的信道（排智能体大部分只能订阅一个信道）
		                      if (channelIndex < channels.length) {
		                          channel = channels[channelIndex];
		                          channelCapacity = scenarioInfo.getCommsChannelBandwidth(channel);
		                          channelSize[channel-1] += smsg.getByteArraySize();
		                      } else {
		                          // 如果没有下一个信道了，就不发送该条信息了。
		                          // if there is no new channel for that message types, just break
		                          break;
		                      }
		                  }
		                  channelSendMessageList.get(channel).add(smsg);
		              }
		          }
		      }
		  }
		  ```