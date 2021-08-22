## Abstract
	-
	  1. 把上一周没怎么看懂的 [[ActionExtClear]] 再看了一遍 (还是没怎么看懂的说)
	-
	  2. 学习了各种聚类算法，并用 java 实现了其中三个
	-
	  3. 研究了一下通讯系统，就 [[MessageUtil]], [[HfutMessageTool]] 等几个类
## About [[ActionExtClear]]
	- 首先这个类的作用是动作提取：也就是调用方指定一个需要清理的位置，然后用这个类计算出要完成这一目标，下一个动作应该是什么。
	- 暂且不看它的实现，想一下如果是自己是这个类，应该怎么完成动作提取。
	- 首先，警察的动作只有三个，Rest、Move 和 Clear。我们的想法也很简单：先算出一条到目的地的路，然后沿路走就好了，走不动就清障再走，要休息了就休息，到了目的地就救人。还有路上如果有被卡的人，一定先救人。
	- 把这个想法往代码上套，就会发现大致是符合的：
	- 首先计算休息，需要休息且可以休息，就休息
	- 不用休息，就往目的地走，这可以分为三种情况：
		- 已经到了目的地：那就清理障碍救人
		- 已经到了目的地相邻的块：先看看有没有要救的人，要是有我先把这个块给清理一下，把这个块里的人给救了；没有的话我就准备去下一个块了
		- 和目的地之间还隔着一些块：可以转化为第二种
	- 当然上面是理想情况，实际上会有很多异常情况，比如明明别人说这里有障碍要清理，结果到了发现根本就没障碍；又或者明明到了和目的地相邻的块，结果发现和目的地没有相邻的边。这些都要特殊处理，所以导致代码看上去很晦涩。
	- 至于 getContinueAction，在官方的 sample 里是没有这个函数的。我认为这个函数应该最初是为了处理救人的行为而设计，也就是处理 “一定要把当前块里所有能救的人全部救了才去下一个块” 这一逻辑。不管是那种情况，都会用 getRescueAction 先看看能不能救人，然后选最优边，并转到 getContinueAction 上。
## About [[Clustering Algorithms]]
	- {{embed [[Clustering Algorithms]] }}
## About Communication System
	- {{embed [[MessageUtil]] }}
	- {{embed [[HfutMessageTool]] }}