## Fields
	- double dx
	- double dy
	- double length
## Methods
	- double dot([[Vector2D]] v)
		- 计算两个向量的数量积 (点积)
		-
		  ```java
		  public double dot(Vector2D v) {
		      return this.dx * v.dx + this.dy * v.dy;
		  }
		  ```
	- [[Vector2D]] scale(double amount)
		-
		  ``` java
		  public Vector2D scale(double amount) {
		      return new Vector2D(this.dx * amount, this.dy * amount);
		  }
		  ```
	- [[Vector2D]] getNormal()
		- 获得当前向量的法向量 (长度相同，但与当前向量垂直)
		-
		  ```java
		  public Vector2D getNormal() {
		      return new Vector2D(-this.dy, this.dx);
		  }
		  ```