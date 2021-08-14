## K-Means
	- 步骤
		- 给定类的个数 n，聚类算法中，类也成为 “簇”
		- 随机选取 n 个点作为初始聚类中心
		- 把每个样本点划给离它最近的聚类中心
		- 聚类中心调整为重心
		- 重复上面两步，直到聚类中心不变
	- 优点
		- 简单、快速
	- 缺点
		- 簇数必须预先给定
		- 对初值敏感
		- 对孤立数据点敏感
## K-Means++
	- 核心思想
		- 初始聚类中心之间的相互距离越远越好
	- 步骤
		- 随机选取一个样本作为第一个聚类中心
		- 计算其余样本点到当前已有聚类中心的最短距离，并根据距离赋予每个点一个概率值，距离越远，这个值越大，最后使用轮盘法抽取一个点作为新的聚类中心。很明显，距离越远的点概率值越大，越有可能在轮盘法中被抽中作为新的聚类中心。
		- 重复上一步，直到选出 k 个初始聚类中心
		- 其余步骤同 K-Means 算法
	- 实现
		-
		  ```java
		  // K-Means++ 初始化
		  public static List<Point2D> init(int maxNum) {
		      List<Point2D> dataListCopy = new ArrayList<>(dataList);
		      List<Point2D> centers = new ArrayList<>();
		  
		      // 随机选取一个样本作为第一个聚类中心
		      centers.add(dataListCopy.remove(getRandomIndex(dataListCopy)));
		  
		      while (centers.size() < maxNum) {
		          // 保存点到当前已有聚类中心的最短距离
		          Map<Point2D, Double> minDistMap = new HashMap<>();
		          // 所有最短距离的和，用于归一化求概率
		          Double sum = 0.0;
		  
		          // 计算其余样本点到当前已有聚类中心的最短距离
		          for (Point2D point: dataListCopy) {
		              Double minDist = Double.MAX_VALUE;
		              for (Point2D center: centers) {
		                  Double dist = point.calcDistance(center);
		                  if (dist < minDist) minDist = dist;
		              }
		              minDistMap.put(point, minDist);
		              sum += minDist;
		          }
		  
		          // 归一化，转换为概率
		          for (Double value: minDistMap.values()) {
		              value = value / sum;
		          }
		  
		          // 轮盘算法
		          Double tryit = Math.random();
		          for (Map.Entry<Point2D, Double> entry: minDistMap.entrySet()) {
		              tryit -= entry.getValue();
		              if (tryit < 0) {
		                  // 选中
		                  centers.add(entry.getKey());
		                  break;
		              }
		          }
		      }
		      return centers;
		  }
		  ```
- ## Canopy + K-Means
	- ### Canopy 算法
		- Canopy 算法是一种粗聚类算法，其得到的聚类间可能有重复 (一个元素同时属于多个聚类)
		- 步骤
			- 给定两个阀值 $T_1$, $T_2$ 和样本集 $\boldsymbol{x}$
			- 随机取出一个样本作为新聚类中心 (这里也叫 canopy)，考虑剩余样本 $x_i$ 到该聚类中心的距离 $d_i$
				- 若 $d_i < T_1$ => $x_i$ 归入该 canopy，并从样本集中移除
				- 若 $T_1 \leqslant d_i < T_2$ => $x_i$ 归入该 canopy，但保留在样本集中
				- 若 $d_i > T_2$ => 继续
			- 重复上一步，直到样本集 $\boldsymbol{x}$ 为空
		- 实现
			-
			  ```java
			  // 非常简单!!
			  public class Canopy {
			  
			      private Double T1;
			      private Double T2;
			      private List<Point2D> dataList; // 数据集
			      private Map<Point2D, Set<Point2D>> clusters; // 用于保存粗聚类结果
			  
			      public Canopy(Double T1, Double T2, List<Point2D> dataList) {
			          this.dataList = new ArrayList<>();
			          this.dataList.addAll(dataList);
			          this.clusters = new HashMap<>();
			          this.T1 = T1;
			          this.T2 = T2;
			      }
			  
			      // 获得随机下标
			      private int getRandomIndex() {
			          return (int) (Math.random() * dataList.size());
			      }
			  
			      public Canopy calc() {
			          while (!dataList.isEmpty()) {
			              Point2D center = dataList.remove(getRandomIndex());
			              clusters.put(center, new HashSet<>());
			              // 不要使用基于迭代器的 for-each 循环
			              for (int i=0; i<dataList.size(); i++) {
			                  Point2D point = dataList.get(i);
			                  Double dist = center.calcDistance(point);
			                  if (dist < T2) clusters.get(center).add(point);
			                  if (dist < T1) dataList.remove(point);
			              }
			          }
			          return this;
			      }
			  
			      public Map<Point2D, Set<Point2D>> getResult() {
			          return clusters;
			      }
			  
			      public static void main(String[] args) {
			          List<Point2D> dataList = new ArrayList<>();
			          // 随机生成 x∈[0,10), y∈[0,10) 的 100 个点
			          for (int i=0; i<100; i++) {
			              dataList.add(Point2D.getRandomPoint(10.0, 10.0));
			          }
			          Canopy canopy = new Canopy(1.0, 3.0, dataList);
			          Map<Point2D, Set<Point2D>> result = canopy.calc().getResult();
			          System.out.println(result);
			      }
			  }
			  ```
		- 如何与 K-Means 算法结合
			- 把 Canopy 算法得到的聚类中心作为 K-Means 算法的初始聚类中心
			- 优点
				- 解决了 K-Means 必须指定簇数的缺点
				- 解决了 K-Means 对初值和孤立数据点敏感的问题
				- 可以通过舍弃点较少的簇，以解决噪声点的干扰
## Hierarchical Clustering
	- 即系统 (层次) 聚类，优点在于无需指定簇数
	- 样本间距离
	  collapsed:: true
		- 欧氏距离
		- 曼哈顿距离
	- 指标间距离
	- 类间距离
	  collapsed:: true
		- 最短距离
		- 最长距离
		- 重心法
		- 组间平均连接法
		- 组间 + 组内平均连接法
	- 聚类的谱系图
	- 步骤
	  collapsed:: true
		- 每个样本 (或指标，以下不再作区分) 看成一类
		- 计算所有类两两间距离，选距离最小的两个类合并为一个类
		- 重复上一步，直到所有类随后合为一类
		- 根据谱系图，选择合适的分类数，得到聚类结果
## DBSCAN
	- 以上几种聚类算法都是基于距离的聚类算法，而 DBSCAN 是一种基于密度的聚类算法
	- ![image_1628728316009_0.png](https://i.loli.net/2021/08/14/wKPuh5XxjgcB7mI.png){:height 252, :width 238}
	- ![image_1628728199458_0.png](https://i.loli.net/2021/08/14/WUtiEdYCkOJKS9q.png){:height 242, :width 250}
	- 可以看到，对于笑脸图，K-Means 算法完全无法处理 (上图)，而 DBSCAN 则可以很好地完成聚类 (下图)
	- ![image_1628728499048_0.png](https://i.loli.net/2021/08/14/j8k2Ia1Ll4iGoHK.png){:height 172, :width 239}
	- DBSCAN 算法也能很好地处理噪声，可以看到，上图中分散的噪声点对聚类结果没有影响