## Intro
	- 对 [[MessageManager]] 里要发送的信息进行分类，投放到合适的信道
## Fields
## Methods
	- void coordinate([[AgentInfo]] agentInfo, [[WorldInfo]] worldInfo, [[ScenarioInfo]] scenarioInfo, [[MessageManager]] messageManager, ArrayList<[[CommunicationMessage]]> sendMessageList, List<List<[[CommunicationMessage]]>> channelSendMessageList)
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
		  
		  ```
	-