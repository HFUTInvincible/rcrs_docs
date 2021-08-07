- 警察有以下两种清障方式：
- 移动到障碍物旁边使其缩小
	-
	  ``` java
	  public ActionClear(@Nonnull EntityID targetID) {
	      this.target = targetID;
	      this.useOldFunction = true;
	  }
	  
	  public ActionClear(@Nonnull Blockade blockade) {
	      this(blockade.getID());
	  }
	  ```
- 范围清除。清除矩形框内约 1/3 的障碍物
	-
	  ``` java
	  public ActionClear(@Nonnull AgentInfo agent, @Nonnull Vector2D vector) {
	      this((int)(agent.getX() + vector.getX()), (int)(agent.getY() + vector.getY()));
	  }
	  
	  public ActionClear(int destX, int destY) {
	      this.useOldFunction = false;
	      this.posX = destX;
	      this.posY = destY;
	  }
	  
	  public ActionClear(int destX, int destY, @Nonnull Blockade blockade) {
	      this(destX, destY);
	      this.target = blockade.getID();
	  }
	  ```
	- ![](https://img-blog.csdnimg.cn/20190510195156471.png)