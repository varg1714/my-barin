---
source: https://blog.csdn.net/weixin_51545953/article/details/125143025
create: 2025-11-29 22:45
read: false
knowledge: false
---
#### 文章目录

*   [一、TSP 问题简介](#TSP_5)
*   [二、数学建模](#_11)
*   [三、实现细节](#_47)
*   [四、案例实战](#_73)
*   *   [4.1 测试案例说明](#41__77)
    *   [4.2 Java 完整代码](#42_Java__142)
    *   *   [4.2.1 TSP_Instance 实例类](#421_TSP_Instance__144)
        *   [4.2.2 TSP_Solution 结果类](#422_TSP_Solution__173)
        *   [4.2.3 TSP_Util 工具类](#423_TSP_Util__214)
        *   [4.2.4 TSP_Solver_ILS 算法类](#424_TSP_Solver_ILS__278)
        *   [4.2.5 RunAndPlot 运行类](#425_RunAndPlot__473)
    *   [4.3 运行结果展示](#43__640)

## 一、TSP 问题简介

旅行推销员问题 (TSP) 提出以下问题：“给定 n n n 个城市的列表，其中有一个起始城市，以及每对城市之间的距离，访问每个城市一次并返回起始城市的最短可能路线是什么？”。

这又是一个重要的 [NP-hard](https://so.csdn.net/so/search?q=NP-hard&spm=1001.2101.3001.7020) 组合优化，特别是在运筹学和理论计算机科学领域。这个问题最早是在 1930 年提出的，是离散最优化中研究最深入的问题之一。

## 二、数学建模

让我们考虑一组 n n n 个城市，其中每个城市 i i i 具有坐标 ( x i , y i ) , i = 1 , 2 , . . . , n (x_i,y_i),i = 1,2,...,n (xi​,yi​),i=1,2,...,n。

在这种情况下，状态空间的每个点 X X X 必须代表我们访问 n n n 个城市的顺序中的一个可能的排列。

为简单起见，我们考虑使用字典顺序的构造初始解决方案：

![](https://i-blog.csdnimg.cn/blog_migrate/629c4b2665f8a6fb9130c0c50fe3062f.png#pic_center)

  
目标函数评估在于计算对应于任意向量 X X X 的旅程的长度 f f f：

f (X) = ∑ i = 1 n − 1 d ( X i , X i + 1 ) + d ( X N , X 1 ) f(X)=\sum_{i=1}^{n-1} d\left(X_i, X_{i+1}\right)+d\left(X_N, X_1\right) f(X)=i=1∑n−1​d(Xi​,Xi+1​)+d(XN​,X1​)

其中， X i X_i Xi​ 是 X X X 的第 i i i 个元素，如果 X i = k ， X i + 1 = l X_i = k，X_{i+1} = l Xi​=k，Xi+1​=l ，则城市 k k k 城市 l l l 的欧式距离为：

d 1 ( X i , X i + 1 ) = ( x l − x k ) 2 + ( y l − y k ) 2 d_1\left(X_i, X_{i+1}\right)=\sqrt{\left(x_l-x_k\right)^2+\left(y_l-y_k\right)^2} d1​(Xi​,Xi+1​)=(xl​−xk​)2+(yl​−yk​)2 ​

城市 k k k 城市 l l l 的伪欧式距离为：

d 2 ( X i , X i + 1 ) = ( x l − x k ) 2 + ( y l − y k ) 2 10 d_2\left(X_i, X_{i+1}\right)=\sqrt{\frac{\left(x_l-x_k\right)^2+\left(y_l-y_k\right)^2}{10}} d2​(Xi​,Xi+1​)=10(xl​−xk​)2+(yl​−yk​)2​ ​

注意，在 f f f 的上述定义中，最后一项 d ( X N , X 1 ) d(X_N,X_1) d(XN​,X1​) 表示返回起始城市的旅程的最后一段。

众所周知，与旅行推销员问题相关联的复杂性远高于背包问题。对于一个有 n n n 个城市的问题，要考虑的潜在旅行次数是 n ! n! n! ，它随 n n n 的增长速度比 2 n 2^n 2n 快得多：

![](https://i-blog.csdnimg.cn/blog_migrate/1d9282c1c706d6585015eb71c758eb94.png#pic_center)

  
为了说明问题的复杂性，如果目标函数的一次评估需要 109 109 109 秒，那么评估每个可能解决方案的简单枚举算法将需要以下 CPU 时间：

![](https://i-blog.csdnimg.cn/blog_migrate/9f7854cecd971877b78c61ac45854f98.png#pic_center)

## 三、实现细节

**有关迭代局部搜索算法的详细介绍请看：**

TSP 问题的一个最简单的邻域算子是在当前解向量 X X X 中随机交换两个位置 (见**图 1.8**)。

![](https://i-blog.csdnimg.cn/blog_migrate/ffb6e677465429f5f8f11415e1fdac40.png#pic_center)

这种操纵状态空间的点的方式确保了所产生的邻居保持排列，即 n n n 个城市的旅行。

在 SA 算法中实现这样一个操作符会产生可接受的结果，但是 SA 的性能可以通过使用另一个邻域操作符来提高，该邻域操作符交换两个随机选择的索引 ( m , n ) (m,n) (m,n) 之间的所有位置，如**图 1.9** 所示：

![](https://i-blog.csdnimg.cn/blog_migrate/55868d9c3e51f7dc112d6d4e2214be36.png#pic_center)

交换两个元素可以使用位运算，例如想交换 a 和 b，可以采用下面的代码：

```
b = a ^ b;
a = a ^ b;
b = a ^ b;

```

**注意**：此方法只是减少了中间变量的使用，降低了内存消耗，并不能提升性能

## 四、案例实战

**本项目的所有数据和代码均上传至 GitHub 仓库：[https://github.com/WSKH0929/MetaHeuristics（如果对你有帮助的话可以点个 Star❤️哟~）](https://github.com/WSKH0929/MetaHeuristics)**

### 4.1 测试案例说明

使用的是 tsplib 上的数据 att48，这是一个对称 TSP 问题，城市规模为 48，其最优值为 10628（取整的伪欧氏距离）

**tsplib 地址：[http://comopt.ifi.uni-heidelberg.de/software/TSPLIB95/tsp/](http://comopt.ifi.uni-heidelberg.de/software/TSPLIB95/tsp/)**

下面是 att48 的文件内容：

```
NAME : att48
COMMENT : 48 capitals of the US (Padberg/Rinaldi)
TYPE : TSP
DIMENSION : 48
EDGE_WEIGHT_TYPE : ATT
NODE_COORD_SECTION
1 6734 1453
2 2233 10
3 5530 1424
4 401 841
5 3082 1644
6 7608 4458
7 7573 3716
8 7265 1268
9 6898 1885
10 1112 2049
11 5468 2606
12 5989 2873
13 4706 2674
14 4612 2035
15 6347 2683
16 6107 669
17 7611 5184
18 7462 3590
19 7732 4723
20 5900 3561
21 4483 3369
22 6101 1110
23 5199 2182
24 1633 2809
25 4307 2322
26 675 1006
27 7555 4819
28 7541 3981
29 3177 756
30 7352 4506
31 7545 2801
32 3245 3305
33 6426 3173
34 4608 1198
35 23 2216
36 7248 3779
37 7762 4595
38 7392 2244
39 3484 2829
40 6271 2135
41 4985 140
42 1916 1569
43 7280 4899
44 7509 3239
45 10 2676
46 6807 2993
47 5185 3258
48 3023 1942
EOF

```

### 4.2 Java 完整代码

#### 4.2.1 TSP_Instance 实例类

```
package com.wskh.classes.tsp;

import lombok.Data;
import lombok.ToString;

/**
 * @Author：WSKH
 * @ClassName：TSP_Instance
 * @Description：
 * @Time：2023/5/13/22:20
 * @Email：1187560563@qq.com
 * @Blog：wskh0929.blog.csdn.net
 */
@Data
@ToString
public class TSP_Instance {
    // 实例名
    String name;
    // 城市数量
    int n;
    // 每个城市的坐标
    double[][] locations;
}


```

#### 4.2.2 TSP_Solution 结果类

```
package com.wskh.classes.tsp;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

import java.util.Arrays;

/**
 * @Author：WSKH
 * @ClassName：TSP_Solution
 * @Description：
 * @Time：2023/5/13/22:52
 * @Email：1187560563@qq.com
 * @Blog：wskh0929.blog.csdn.net
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TSP_Solution {
    // 路程长度
    double pathLen;
    // 路径
    int[] path;

    public TSP_Solution copy() {
        return new TSP_Solution(pathLen, path.clone());
    }

    @Override
    public String toString() {
        return "pathLen = " + pathLen + " , path = " + Arrays.toString(path);
    }
}


```

#### 4.2.3 TSP_Util 工具类

```
package com.wskh.utils;

import com.wskh.classes.tsp.TSP_Instance;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

/**
 * @Author：WSKH
 * @ClassName：TSP_Util
 * @Description：
 * @Time：2023/5/13/22:15
 * @Email：1187560563@qq.com
 * @Blog：wskh0929.blog.csdn.net
 */
public class TSP_Util {
    // 读取tsp数据
    public static TSP_Instance readTSP_Instance(String path) throws IOException {
        TSP_Instance tspInstance = new TSP_Instance();
        BufferedReader bufferedReader = new BufferedReader(new FileReader(path));
        String line = null;
        int row = 1;
        while ((line = bufferedReader.readLine()) != null) {
            if (line.contains("NAME")) {
                tspInstance.setName(line.split(" : ")[1]);
            } else if (line.contains("DIMENSION")) {
                tspInstance.setN(Integer.parseInt(line.split(" : ")[1]));
                tspInstance.setLocations(new double[tspInstance.getN()][2]);
            } else if (row >= 7 && row < 7 + tspInstance.getN()) {
                String[] split = line.split(" ");
                int index = Integer.parseInt(split[0]) - 1;
                int x = Integer.parseInt(split[1]);
                int y = Integer.parseInt(split[2]);
                tspInstance.getLocations()[index][0] = x;
                tspInstance.getLocations()[index][1] = y;
            }
            row++;
        }
        bufferedReader.close();
        return tspInstance;
    }

    // 计算两点之间的取整的伪欧式距离
    public static double calcDistance(double[] p1, double[] p2) {
        return Math.ceil(Math.sqrt((Math.pow(p1[0] - p2[0], 2) + Math.pow(p1[1] - p2[1], 2)) / 10d));
    }

    // 评价函数，传入一个路径和距离矩阵，返回该路径的长度
    public static double calcPathLen(int[] path, double[][] distances) {
        double pathLen = 0d;
        for (int i = 0; i < path.length - 1; i++) {
            pathLen += distances[path[i]][path[i + 1]];
        }
        // 还要算上从最后一个城市回到起始城市的距离
        pathLen += distances[path[path.length - 1]][path[0]];
        return pathLen;
    }

}

```

#### 4.2.4 TSP_Solver_ILS 算法类

```
package com.wskh.meta_heuristics.ILS.tsp;

import com.wskh.classes.tsp.TSP_Instance;
import com.wskh.classes.tsp.TSP_Solution;
import com.wskh.utils.TSP_Util;
import lombok.Data;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Random;

/**
 * @Author：WSKH
 * @ClassName：TSP_Solver_ILS
 * @Description：
 * @Time：2023/5/17/9:38
 * @Email：1187560563@qq.com
 * @Blog：wskh0929.blog.csdn.net
 */
@Data
public class TSP_Solver_ILS {
    // 随机数种子
    Long seed;
    // 迭代次数
    int epochs = 20000;
    // 局部搜索次数
    int localSearchCnt = 50;
    // 扰动时对序列进行的分组数(这个数要在3-cityNum之间)
    public int groupNum = 5;

    // 构造函数
    public TSP_Solver_ILS(Long seed, int epochs, int localSearchCnt, int groupNum) {
        this.seed = seed;
        this.epochs = epochs;
        this.localSearchCnt = localSearchCnt;
        this.groupNum = groupNum;
    }

    // 城市数量
    int n;
    // 城市坐标
    double[][] locations;
    // 距离矩阵
    double[][] distances;
    // 随机数生成对象
    Random random;
    // 当前解
    TSP_Solution curSolution;
    // 最优解
    TSP_Solution bestSolution;

    // 求解函数
    public TSP_Solution solve(TSP_Instance tspInstance) {
        long startTime = System.currentTimeMillis();
        // 初始化操作
        init(tspInstance);
        System.out.println("城市数量为: " + n);
        System.out.println("初始解为: " + bestSolution);
        // 迭代局部搜索过程
        curSolution = localSearch(curSolution.copy());
        for (int epoch = 0; epoch < epochs; epoch++) {
            TSP_Solution solution = perturbation(curSolution.copy());
            solution = localSearch(solution);
            if (solution.getPathLen() < curSolution.getPathLen()) {
                curSolution = solution;
            }
        }
        // 输出结果
        System.out.println("最终找到的最优解为: " + bestSolution);
        System.out.println("求解用时: " + (System.currentTimeMillis() - startTime) / 1000d + " s");
        return bestSolution;
    }

    // 扰动
    private TSP_Solution perturbation(TSP_Solution solution) {
        int[] newX = new int[n];
        // 获取groupNum个index，这groupNum个index将bestSequence划分为(groupNum-1)组(并非均分)
        // n - 2的原因是：indexArr[0]和indexArr[groupNum-1]都已经确定
        int[] indexArr = new int[groupNum];
        int elementNumInOneGroup = (n - 2) / (groupNum - 2);
        indexArr[0] = 0;
        for (int i = 1; i < indexArr.length - 1; i++) {
            indexArr[i] = random.nextInt(elementNumInOneGroup) + (i - 1) * elementNumInOneGroup + 1;
        }
        indexArr[groupNum - 1] = n;
        // 将组序打乱
        List<Integer> groupCodeList = new ArrayList<>();
        for (int i = 0; i <= groupNum - 2; i++) {
            groupCodeList.add(i);
        }
        Collections.shuffle(groupCodeList, random);
        // 赋值
        int index = 0;
        for (Integer integer : groupCodeList) {
            for (int j = indexArr[integer]; j < indexArr[integer + 1]; j++) {
                newX[index] = solution.getPath()[j];
                index++;
            }
        }
        TSP_Solution newSolution = new TSP_Solution(TSP_Util.calcPathLen(newX, distances), newX);
        if (newSolution.getPathLen() < bestSolution.getPathLen()) {
            bestSolution = newSolution.copy();
        }
        return newSolution;
    }

    // 局部搜索过程
    private TSP_Solution localSearch(TSP_Solution solution) {
        for (int i = 0; i < localSearchCnt; i++) {
            // 随机使用两个邻域算子构造新解
            int[] newPath = random.nextInt(2) == 0 ? neighborhoodOperator1(solution.getPath()) : neighborhoodOperator2(solution.getPath());
            double newPathLen = TSP_Util.calcPathLen(newPath, distances);
            if (newPathLen < solution.getPathLen()) {
                solution = new TSP_Solution(newPathLen, newPath);
                if (newPathLen < bestSolution.getPathLen()) {
                    bestSolution = solution.copy();
                }
            }
        }
        return solution;
    }

    // 邻域算子1：在当前解向量 X 中随机交换两个位置
    private int[] neighborhoodOperator1(int[] X) {
        int[] newX = X.clone();
        int i = random.nextInt(n);
        int j = random.nextInt(n);
        while (i == j) {
            j = random.nextInt(n);
        }
        // 采用位运算交换 i 和 j 处的两个元素
        newX[j] = newX[i] ^ newX[j];
        newX[i] = newX[i] ^ newX[j];
        newX[j] = newX[i] ^ newX[j];
        return newX;
    }

    // 邻域算子2：交换两个随机选择的索引 (i,j) 之间的所有位置
    private int[] neighborhoodOperator2(int[] X) {
        int[] newX = X.clone();
        int i = random.nextInt(n);
        int j = random.nextInt(n);
        while (i == j) {
            j = random.nextInt(n);
        }
        // 确保 i < j
        if (i > j) {
            j = i ^ j;
            i = i ^ j;
            j = i ^ j;
        }
        // 交换 (i,j) 之间的所有位置
        int sum = i + j;
        int maxI = sum / 2;
        if (sum % 2 == 0) {
            maxI--;
        }
        for (; i <= maxI; i++) {
            newX[sum - i] = newX[i] ^ newX[sum - i];
            newX[i] = newX[i] ^ newX[sum - i];
            newX[sum - i] = newX[i] ^ newX[sum - i];
        }
        return newX;
    }

    // 初始化操作
    private void init(TSP_Instance tspInstance) {
        n = tspInstance.getN();
        locations = tspInstance.getLocations();
        random = seed == null ? new Random() : new Random(seed);
        // 计算距离矩阵
        distances = new double[n][n];
        for (int i = 0; i < distances.length; i++) {
            for (int j = i + 1; j < distances.length; j++) {
                // i到j等于j到i
                distances[i][j] = TSP_Util.calcDistance(locations[i], locations[j]);
                distances[j][i] = distances[i][j];
            }
        }
        // 生成初始解，为简单起见，我们考虑使用字典顺序的生成初始解决方案
        curSolution = new TSP_Solution();
        curSolution.setPath(new int[n]);
        for (int i = 0; i < n; i++) {
            curSolution.getPath()[i] = i;
        }
        curSolution.setPathLen(TSP_Util.calcPathLen(curSolution.getPath(), distances));
        bestSolution = curSolution.copy();
    }

}

```

#### 4.2.5 RunAndPlot 运行类

```
package com.wskh.meta_heuristics.ILS.tsp;

import com.wskh.classes.tsp.TSP_Instance;
import com.wskh.classes.tsp.TSP_Solution;
import com.wskh.utils.TSP_Util;
import javafx.animation.KeyFrame;
import javafx.animation.Timeline;
import javafx.event.ActionEvent;
import javafx.event.EventHandler;
import javafx.scene.Scene;
import javafx.scene.canvas.Canvas;
import javafx.scene.canvas.GraphicsContext;
import javafx.scene.control.Button;
import javafx.scene.layout.AnchorPane;
import javafx.scene.paint.Color;
import javafx.scene.shape.Circle;
import javafx.scene.shape.LineTo;
import javafx.scene.shape.MoveTo;
import javafx.scene.shape.Path;
import javafx.stage.Stage;
import javafx.util.Duration;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

/**
 * @Author：WSKH
 * @ClassName：Run
 * @Description：
 * @Time：2023/5/14/11:52
 * @Email：1187560563@qq.com
 * @Blog：wskh0929.blog.csdn.net
 */
public class RunAndPlot extends javafx.application.Application {

    // 可视化区域的宽度
    int stageW = 1000;
    // 可视化区域的高度
    int stageH = 1000;
    // 坐标偏移
    int offsetX = 100;
    int offsetY = 100;

    // 主要函数
    @Override
    public void start(Stage primaryStage) throws Exception {

        // 初始化画布和面板
        AnchorPane pane = new AnchorPane();
        Canvas canvas = initCanvas(stageW - offsetX * 2, stageH - offsetY * 2);
        canvas.relocate(offsetX, offsetY);
        pane.getChildren().add(canvas);

        // 读取tsp数据
        TSP_Instance tspInstance = TSP_Util.readTSP_Instance("data/tsp/att48.tsp");
        // 固定使用随机数种子
        Long seed = 2023L;
        System.out.println("------------------------- 迭代局部搜索算法求解TSP问题 -----------------------------");
        TSP_Solution solution = new TSP_Solver_ILS(seed, 40000, 100, 6).solve(tspInstance);

        // 获取最佳路径
        int[] bestPath = solution.getPath().clone();

        // 按照屏幕尺寸等比例调整城市坐标
        List<double[]> fitLocationList = fitLocation(tspInstance.getLocations());

        // 绘制城市
        HashMap<Integer, Circle> map = new HashMap<>();
        List<Circle> circleList = new ArrayList<>();
        for (double[] position : fitLocationList) {
            Circle circle = new Circle(position[0], position[1], 5);
            pane.getChildren().add(circle);
            circleList.add(circle);
        }

        // 添加播放按钮
        Button button = new Button("播放");
        final int[] pCnt = {0};
        button.setOnAction(new EventHandler<ActionEvent>() {
            // 按钮点击事件
            @Override
            public void handle(ActionEvent event) {
                // 路径 动画
                Timeline animation = new Timeline(new KeyFrame(Duration.millis(50), new EventHandler<ActionEvent>() {
                    @Override
                    public void handle(ActionEvent event) {
                        if (pCnt[0] < bestPath.length - 1) {
                            int cur = bestPath[pCnt[0]];
                            int next = bestPath[pCnt[0] + 1];
                            double[] p1 = fitLocationList.get(cur);
                            double[] p2 = fitLocationList.get(next);
                            Path path = new Path();
                            path.getElements().add(new MoveTo(p1[0], p1[1]));
                            path.getElements().add(new LineTo(p2[0], p2[1]));
                            pane.getChildren().add(path);
                            pCnt[0]++;
                        } else if (pCnt[0] == bestPath.length - 1) {
                            int cur = bestPath[pCnt[0]];
                            int next = bestPath[0];
                            double[] p1 = fitLocationList.get(cur);
                            double[] p2 = fitLocationList.get(next);
                            Path path = new Path();
                            path.getElements().add(new MoveTo(p1[0], p1[1]));
                            path.getElements().add(new LineTo(p2[0], p2[1]));
                            pane.getChildren().add(path);
                            pCnt[0]++;
                        }
                    }
                }));
                animation.setCycleCount(Timeline.INDEFINITE);
                animation.play();
            }
        });
        pane.getChildren().add(button);

        // 一切就绪，准备展示
        primaryStage.setTitle("TSP路径可视化");
        primaryStage.setScene(new Scene(pane, stageW, stageH));
        primaryStage.show();
    }

    private List<double[]> fitLocation(double[][] locations) {
        List<double[]> locationList = new ArrayList<>();
        for (double[] location : locations) {
            locationList.add(location.clone());
        }
        // 获取宽度和高度方向上的最大值
        double maxX = locationList.get(0)[0];
        double maxY = locationList.get(0)[1];
        for (double[] position : locationList) {
            maxX = Math.max(maxX, position[0]);
            maxY = Math.max(maxY, position[1]);
        }
        // 计算缩放比例
        double rateX = (stageW - 2 * offsetX) / maxX;
        double rateY = (stageH - 2 * offsetY) / maxY;
        // 按照比例自适应调整
        List<double[]> fitLocationList = new ArrayList<>();
        for (double[] position : locationList) {
            fitLocationList.add(new double[]{position[0] * rateX + offsetX, position[1] * rateY + offsetY});
        }
        return fitLocationList;
    }

    // 初始化画布对象
    private Canvas initCanvas(double l, double w) {
        Canvas canvas = new Canvas(l, w);
        GraphicsContext gc = canvas.getGraphicsContext2D();
        // 边框
        gc.setStroke(Color.BLACK);
        gc.setLineWidth(2);
        gc.strokeRect(0, 0, l, w);
        // 填充
        gc.setFill(new Color(127 / 255d, 255 / 255d, 170 / 255d, 1d));
        gc.fillRect(0, 0, l, w);
        return canvas;
    }

    public static void main(String[] args) {
        launch(args);
    }
}

```

### 4.3 运行结果展示

**控制台输出：**

```
------------------------- 迭代局部搜索算法求解TSP问题 -----------------------------
城市数量为: 48
初始解为: pathLen = 49840.0 , path = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47]
最终找到的最优解为: pathLen = 10888.0 , path = [17, 6, 27, 5, 36, 18, 26, 16, 42, 29, 35, 45, 32, 19, 46, 20, 38, 31, 23, 44, 34, 9, 41, 25, 3, 1, 28, 4, 47, 24, 13, 12, 22, 10, 11, 14, 39, 2, 33, 40, 15, 21, 0, 7, 8, 37, 30, 43]
求解用时: 0.365 s

```

**可视化展示：**

![](https://i-blog.csdnimg.cn/blog_migrate/58c8726b04b5b52d7505cb076ec9a7f7.gif#pic_center)