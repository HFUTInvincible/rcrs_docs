## Fields
	- [[PathPlanning]] pathPlanning
	- int thresholdRest
	- int kernelTime
	- [[EntityID]] target
	- [[Logger]] logger
	- [[Vector2D]] detour_Left, detour_Right
	- [[Action]] lastAction
	- int lastLocationX, lastLocationY
	- [[HfutMessageTool]] messageTool
## Methods
	- [[ExtAction]] calc()
	- [[Action]] calcLoad([[AmbulanceTeam]] agent, [[PathPlanning]] pathPlanning, [[EntityID]] targetID)
	- [[Action]] calcUnload([[AmbulanceTeam]] agent, [[PathPlanning]] pathPlanning, [[Human]] transportHuman, [[EntityID]] targetID)
	- boolean needRest([[Human]] agent)
	- [[EntityID]] convertArea([[EntityID]] targetID)
	  id:: 610c6725-3d43-44a8-8929-a9bfbef92156
	- [[Action]] calcRefugeAction([[Human]] human, [[PathPlanning]] pathPlanning, Collection<[[EntityID]]> targets, boolean isUnload)
	- boolean isOnOrNearTheEdge(int pX, int pY, [[Blockade]] blockade)
	- boolean isInBlockades(double pX, double pY)
	- double getAngle([[Vector2D]] v1, [[Vector2D]] v2)
	- boolean isPassable(List<[[EntityID]]> path)