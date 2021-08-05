## Fields
	- [[PathPlanning]] pathPlanning
	- thresholdRest
	- kernelTime
	- [[EntityID]] target
	- [[Logger]] logger
	- [[Vector2D]] detour_Left, detour_Right
	- [[Action]] lastAction
	- lastLocationX, lastLocationY
	- [[HfutMessageTool]] messageTool
## Methods
	- [[ExtAction]] calc()
		-
		  ```java
		    @Override
		    public ExtAction calc() {
		        logger.info("ActionTransport is calculating...");
		        this.result = null;
		        AmbulanceTeam agent = (AmbulanceTeam) this.agentInfo.me();
		        Human transportHuman = this.agentInfo.someoneOnBoard();
		  
		        logger.info("lastAction:" + lastAction + ",lastLactionX:" + lastLocationX + ",lastLocationY:" + lastLocationY + "\nX:" + agent.getX() + ",Y:" + agent.getY());
		  
		        /**
		         *
		         */
		        if (lastAction != null) {
		            if (lastAction.getClass() == ActionMove.class && lastLocationX == agent.getX() && lastLocationY == agent.getY()) {
		                for (Blockade blockade : this.worldInfo.getBlockades(this.agentInfo.getPositionArea())) {
		                    if (isOnOrNearTheEdge(agent.getX(), agent.getY(), blockade)) {
		                        Vector2D detour1 = detour_Right.normalised().scale(detour_Right.getLength() * 15);
		                        Vector2D detour2 = detour_Left.normalised().scale(detour_Left.getLength() * 15);
		                        double movePointX1 = this.agentInfo.getX() + detour1.getX();
		                        double movePointY1 = this.agentInfo.getY() + detour1.getY();
		                        double movePointX2 = this.agentInfo.getX() + detour2.getX();
		                        double movePointY2 = this.agentInfo.getY() + detour2.getY();
		                        if (!isInBlockades(movePointX1, movePointY1)) {
		                            this.result = new ActionMove(this.pathPlanning.calc().getResult(), (int) movePointX1, (int) movePointY1);
		                            lastLocationX = agent.getX();
		                            lastLocationY = agent.getY();
		                            lastAction = this.result;
		                            logger.info("Detouring...");
		                            return this;
		                        } else if (!isInBlockades(movePointX2, movePointY2)) {
		                            this.result = new ActionMove(this.pathPlanning.calc().getResult(), (int) movePointX2, (int) movePointY2);
		                            lastLocationX = agent.getX();
		                            lastLocationY = agent.getY();
		                            lastAction = this.result;
		                            logger.info("Detouring...");
		                            return this;
		                        }
		                    }
		                }
		            }
		        }
		        /**
		         * 此处结束
		         */
		        if (transportHuman != null) {
		            logger.info(transportHuman + " is on bored, calculating unload.");
		            this.result = this.calcUnload(agent, this.pathPlanning, transportHuman,
		                    this.target);
		            if (this.result != null) {
		                lastLocationX = agent.getX();
		                lastLocationY = agent.getY();
		                lastAction = this.result;
		                return this;
		            }
		        }
		        if (this.needRest(agent)) {
		            EntityID areaID = this.convertArea(this.target);
		            ArrayList<EntityID> targets = new ArrayList<>();
		            if (areaID != null) {
		                targets.add(areaID);
		            }
		            this.result = this.calcRefugeAction(agent, this.pathPlanning, targets,
		                    false);
		            if (this.result != null) {
		                lastLocationX = agent.getX();
		                lastLocationY = agent.getY();
		                lastAction = this.result;
		                return this;
		            }
		        }
		        if (this.target != null) {
		            this.result = this.calcLoad(agent, this.pathPlanning, this.target);
		            lastLocationX = agent.getX();
		            lastLocationY = agent.getY();
		            lastAction = this.result;
		        }
		        return this;
		    }
		  ```