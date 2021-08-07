- 为 [[Edge]] 的自定义比较器
	- < 0 => 给定点到 a 较近
	- = 0 => 给定点到 a, b 一样近
	- \> 0 => 给定点到 b 较近
-
  ``` java
  private class TheSortForEdge implements Comparator<Edge> {
      private Point2D reference;
  
      TheSortForEdge(Point2D reference) {
          this.reference = reference;
      }
  
      public int compare(Edge a, Edge b) {
          int d1 = (int) java.awt.geom.Line2D.ptLineDist(a.getStartX(), a.getStartY(), a.getEndX(), a.getEndY(), reference.getX(), reference.getY());
          int d2 = (int) java.awt.geom.Line2D.ptLineDist(b.getStartX(), b.getStartY(), b.getEndX(), b.getEndY(), reference.getX(), reference.getY());
          return d1 - d2;
      }
  }
  ```
- 用法：
-
  ```java
  allThePassableEdge.sort(new TheSortForEdge(new Point2D(police.getX(), police.getY())));
  ```
- 将所有边按到给顶点的距离升序排序