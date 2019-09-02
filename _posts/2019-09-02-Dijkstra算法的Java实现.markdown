---
layout: post
title:  "Dijkstra算法的Java实现"
date:   2019-09-02 18:23:00 +0800
category: java
---

Dijkstra可以在`O(n^2)`的时间复杂度内求从某一个起点出发到达任一一个目的地的最短路径。该算法维护一个`distance`数组，存放目前为止从起点到所有节点的已知最短路径。假设起点是`entrance`,当前访问节点是`current`，目标节点是j，如果`entrance`到`current`距离与`current`到`j`的距离之和小于`distance`里的存放的距离，则更新。当遍历完所有的节点，此时的`distace`数组就是起点到任一节点的最短路径。

算法的Java实现如下：

```java
class Dijkstra{
    int[] visited;
    int[] distance;
    int[][] matrix;

    public Dijkstra(int[][] matrix){
        visited = new int[matrix.length];
        distance = new int[matrix.length];
        this.matrix=matrix;
    }

    public int[] findAllShortestPathFrom(int entrance){
        //初始化visited
        Arrays.fill(visited,0);
        //初始化距离
        Arrays.fill(distance,Integer.MAX_VALUE);
        int current = 0;
        distance[entrance] = 0;  // 入口到自己的最短距离是0
        while(isIn(0,visited)){//当visited中还有0  即没访问过的节点时
            current=getUnvisitedMinIndex();
            for (int i = 0; i < visited.length; i++) {
                if(matrix[current][i]!= Integer.MAX_VALUE && matrix[current][i]+distance[current]<distance[i]){
                    distance[i]=matrix[current][i]+distance[current];
                }
            }
            visited[current]=1;
        }
        return distance;
    }

    public boolean isIn(int num,int[] list){
        for (int i = 0; i < list.length; i++) {
            if(num==list[i]){
                return true;
            }
        }
        return false;
    }

    //找到还未访问 但目前距离最近的节点的下标
    public int getUnvisitedMinIndex(){
        int min = Integer.MAX_VALUE;
        int index = 0;
        for (int i = 0; i < visited.length; i++) {
            if(visited[i]==0 && distance[i]<min){
                index=i;
                min = distance[i];
            }
        }
        return index;
    }
}
```