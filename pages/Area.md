## Methods
	- [[Edge]] getEdgeTo([[EntityID]] neighbour)
	-
	  ```java
	  id:: 610db51e-a7e1-46f3-9ab5-27d5327d37db
	  public Edge getEdgeTo(EntityID neighbour) {
	      Iterator var2 = this.getEdges().iterator();
	  
	      Edge next;
	      do {
	          if (!var2.hasNext()) {
	              return null;
	          }
	  
	          next = (Edge)var2.next();
	      } while(!neighbour.equals(next.getNeighbour()));
	  
	      return next;
	  }
	  ```