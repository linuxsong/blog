---
title: A*算法的JavaScript实现
key: A*算法的JavaScript实现
tags: 算法
author: LinuxSong
---

A* (A-Star)算法是一种路径搜索和图形遍历算法，它有较好的性能和准确度。

A*的算法公式：f(n) = g(n) + h(n)

f(n)是节点n的综合优先级。当我们选择下一个要遍历的节点时，我们总会选取综合优先级最高（值最小）的节点。g(n) 是节点n距离起点的代价。h(n)是节点n距离终点的预计代价，这也就是A*算法的启发函数。

A*算法在运算过程中，每次从优先队列中选取f(n)值最小（优先级最高）的节点作为下一个待遍历的节点。

另外，A*算法使用两个集合来表示待遍历的节点，与已经遍历过的节点，这通常称之为open_set和close_set。

完整的算法描述如下：
``` 
* 初始化open_set和close_set；
* 将起点加入open_set中，并设置优先级为0（优先级最高）；
* 如果open_set不为空，则从open_set中选取优先级最高的节点n：
    * 如果节点n为终点，则：
        * 从终点开始逐步追踪parent节点，一直达到起点；
        * 返回找到的结果路径，算法结束；
    * 如果节点n不是终点，则：
        * 将节点n从open_set中删除，并加入close_set中；
        * 遍历节点n所有的邻近节点：
            * 如果邻近节点m在close_set中，则：
                * 跳过，选取下一个邻近节点
            * 如果邻近节点m也不在open_set中，则：
                * 设置节点m的parent为节点n
                * 计算节点m的优先级
                * 将节点m加入open_set中
```
根据上面的算法描述，用js实现了一个算法演示。

<!--more-->

先看一下效果

<div id="canvas"></div>

**鼠标点击画布可以重新生成地图**

红色方块为起点，绿色方块为终点，白色方块为允许的路径，灰色方块为障碍物，蓝色方块为候选路径，黄色方块为最终路径


为简化逻辑，我们假定地图只允许上下左右4个方向移动。地图的起点设置为最左侧的中间位置，终点设置为最右侧的中间位置。

下面是实现代码：

{% highlight javascript linenos %}
// 每行的方块数
const lineBlockCount = 20
const canvasSize = 400
// 每个方块的大小
let blockSize;
// 方块的颜色
let mapColor;
let aStar;

function setup() {
    mapColor = {
        // 各种方块的颜色 
        'blank': color('white'), // 空白块
        'barrier': color('gray'), // 障碍物
        'enter': color('red'), // 起点
        'exit': color('green'), // 终点
        'candidate': color('cyan'), // 候选路径
        'path': color('yellow') // 最终路径
    }
    aStar = new AStar(lineBlockCount);
    blockSize = canvasSize / aStar.mapSize;
    let canvas = createCanvas(canvasSize, canvasSize);
    canvas.parent('canvas');
    background(255);
    // 设置帧率（为了方便观看，设置每秒10帧）
    frameRate(10);
}

function draw() {
    noFill();
    stroke(0)
    for (let i = 0; i < lineBlockCount; i++) {
        for (let j = 0; j < lineBlockCount; j++) {
            if (aStar.mapData[i][j].isEnter()) {
                fill(mapColor['enter']);
            } else if (aStar.mapData[i][j].isCandidate) {
                fill(mapColor['candidate']);
            } else {
                fill(mapColor[aStar.mapData[i][j].type]);
            }
            rect(j * blockSize, i * blockSize, blockSize, blockSize)
        }
    }
    if (!aStar.isFinish) {
        aStar.nextStep();
    } else {
        if (aStar.isFindPath) {
            fill(mapColor['path']);
            aStar.path.forEach(function (node) {
                rect(node.y * blockSize, node.x * blockSize, blockSize, blockSize)
            });
            noLoop();
        } else {
            // 若找不到可行的路径，则重新生成地图
            reset();
        }
    }
}

// 鼠标点击时重新生成随机地图
function mousePressed() {
    reset();
    loop();
}

// 重新生成对象
function reset() {
    aStar = new AStar(lineBlockCount);
}

// A-Star算法实现
class AStar {
    constructor(mapSize) {
        this.openSet = {};
        this.closeSet = {};
        this.mapSize = mapSize;
        this.isFinish = false;
        this.isFindPath = false;
        this.path = [];
        this.initMap();
    }

    initMap() {
        this.mapData = new Array(this.mapSize);
        for (let i = 0; i < this.mapSize; i++) {
            this.mapData[i] = new Array(this.mapSize);
            for (let j = 0; j < this.mapSize; j++) {
                this.mapData[i][j] = new Node(i, j, 'blank');
            }
        }

        this.randomBarrier();
        this.setEnterAndExit();
        this.addNodeToOpenSet(this.enterNode);
    }

    // 随机生成障碍物
    randomBarrier() {
        // 障碍物的数量按总方块数的1/5
        let barrierCount = Math.round(this.mapSize * this.mapSize / 5);
        for (let i = 0; i < barrierCount; i++) {
            let randX = this.getRandomInt(0, this.mapSize);
            let randY = this.getRandomInt(0, this.mapSize);
            this.mapData[randX][randY].type = 'barrier';
        }
    }

    // 生成随机数
    getRandomInt(min, max) {
        min = Math.ceil(min);
        max = Math.floor(max);
        return Math.floor(Math.random() * (max - min)) + min; //The maximum is exclusive and the minimum is inclusive
    }

    // 设置起点和终点，其中起点设置为左侧中间位置，终点为右侧中间位置
    setEnterAndExit() {
        let enterX = Math.floor(this.mapSize / 2);
        let enterY = 0;
        let exitX = enterX;
        let exitY = this.mapSize - 1;
        this.mapData[enterX][enterY].type = 'enter';
        this.mapData[enterX][enterY].totalCost = 0;
        this.mapData[exitX][exitY].type = 'exit';
        this.enterNode = this.mapData[enterX][enterY];
        this.exitNode = this.mapData[exitX][exitY];
    }

    // 获取 openSet中代价最小的节点
    getMinCostNode() {
        let minCost = Number.MAX_VALUE
        let node = null;
        for (let key in this.openSet) {
            if (this.openSet[key].totalCost < minCost) {
                minCost = this.openSet[key].totalCost
                node = this.openSet[key];
            }
        }
        return node;
    }

    // 规划下一步的路径
    nextStep() {
        if (this.isFinish) {
            return;
        } else if (Object.keys(this.openSet).length === 0) {
            this.isFinish = true;
            return;
        }
        let node = this.getMinCostNode();
        if (node.isExit()) {
            this.isFinish = true;
            this.isFindPath = true;
            this.updatePath(node);
        } else {
            this.deleteNodeFromOpenSet(node);
            this.addNodeToCloseSet(node);
            this.addNeighborNodeToOpenSet(node);
        }
    }

    updatePath(node) {
        while (node != null) {
            this.path.push(node);
            node = node.prev;
        }

        this.path.reverse();
    }

    // 处理临近节点，为简化逻辑，假设只允许上下左右4个方向移动
    addNeighborNodeToOpenSet(node) {
        this.processNodeCoordinate(node, node.x - 1, node.y)
        this.processNodeCoordinate(node, node.x, node.y - 1)
        this.processNodeCoordinate(node, node.x + 1, node.y)
        this.processNodeCoordinate(node, node.x, node.y + 1)
        //this.processNodeCoordinate(node, node.x - 1, node.y - 1)
        //this.processNodeCoordinate(node, node.x + 1, node.y + 1)
        //this.processNodeCoordinate(node, node.x - 1, node.y + 1)
        //this.processNodeCoordinate(node, node.x + 1, node.y - 1)
    }

    // 处理节点，若合法有效则加入到openSet中
    processNodeCoordinate(prev, x, y) {
        if (!this.isValidCoordinate(x, y)) {
            return;
        }
        let key = `${x},${y}`;
        if (!this.mapData[x][y].isBarrier() &&
            typeof (this.closeSet[key]) === 'undefined' &&
            typeof (this.openSet[key]) === 'undefined') {

            this.calcNodeTotalCost(this.mapData[x][y]);
            this.mapData[x][y].prev = prev
            this.addNodeToOpenSet(this.mapData[x][y]);
        }
    }

    // 添加节点到openSet中
    addNodeToOpenSet(node) {
        let key = `${node.x},${node.y}`;
        node.isCandidate = true;
        this.openSet[key] = node;
    }

    // 添加节点到closeSet中
    addNodeToCloseSet(node) {
        let key = `${node.x},${node.y}`;
        this.closeSet[key] = node;
    }

    // 检测坐标是否有效
    isValidCoordinate(x, y) {
        return x >= 0 && x < this.mapSize &&
            y >= 0 &&
            y < this.mapSize;
    }

    // 从openSet中删除某个节点
    deleteNodeFromOpenSet(node) {
        let key = `${node.x},${node.y}`;
        delete this.openSet[key];
    }

    // 计算节点的总代价（越小优先级越高)
    calcNodeTotalCost(node) {
        node.totalCost = this.gCost(node) + this.hCost(node);
    }

    // 计算节点到起点的代价
    gCost(node) {
        return node.distance(this.enterNode);
    }

    // 计算节点到终点的代价
    hCost(node) {
        return node.distance(this.exitNode);
    }
}

// 节点类
class Node {
    constructor(x, y, type) {
        this.x = x;
        this.y = y;
        this.prev = null;
        this.totalCost = null;
        this.type = type;
        // 候选路径
        this.isCandidate = false;
    }

    isEnter() {
        return this.type === 'enter';
    }

    isBlank() {
        return this.type === 'blank';
    }

    isBarrier() {
        return this.type === 'barrier';
    }

    isExit() {
        return this.type === 'exit';
    }

    // 计算离另一节点的距离
    distance(node) {
        let dx = Math.abs(node.x - this.x);
        let dy = Math.abs(node.y - this.y);
        return dx + dy;
    }
}
{% endhighlight %}

<script src='/assets/js/p5.min.js'></script>
<script>
// 每行的方块数
const lineBlockCount = 20
const canvasSize = 400
// 每个方块的大小
let blockSize;
// 方块的颜色
let mapColor;
let aStar;

function setup() {
    mapColor = {
        // 各种方块的颜色 
        'blank': color('white'), // 空白块
        'barrier': color('gray'), // 障碍物
        'enter': color('red'), // 起点
        'exit': color('green'), // 终点
        'candidate': color('cyan'), // 候选路径
        'path': color('yellow') // 最终路径
    }
    aStar = new AStar(lineBlockCount);
    blockSize = canvasSize / aStar.mapSize;
    let canvas = createCanvas(canvasSize, canvasSize);
    canvas.parent('canvas');
    background(255);
    frameRate(10);
}

function draw() {
    noFill();
    stroke(0)
    for (let i = 0; i < lineBlockCount; i++) {
        for (let j = 0; j < lineBlockCount; j++) {
            if (aStar.mapData[i][j].isEnter()) {
                fill(mapColor['enter']);
            } else if (aStar.mapData[i][j].isCandidate) {
                fill(mapColor['candidate']);
            } else {
                fill(mapColor[aStar.mapData[i][j].type]);
            }
            rect(j * blockSize, i * blockSize, blockSize, blockSize)
        }
    }
    if (!aStar.isFinish) {
        aStar.nextStep();
    } else {
        if (aStar.isFindPath) {
            fill(mapColor['path']);
            aStar.path.forEach(function (node) {
                rect(node.y * blockSize, node.x * blockSize, blockSize, blockSize)
            });
            noLoop();
        } else {
            // 若找不到可行的路径，则重新生成地图
            reset();
        }
    }
}

// 鼠标点击时重新生成随机地图
function mousePressed() {
    reset();
    loop();
}

// 重新生成对象
function reset() {
    aStar = new AStar(lineBlockCount);
}

// A-Star算法实现
class AStar {
    constructor(mapSize) {
        this.openSet = {};
        this.closeSet = {};
        this.mapSize = mapSize;
        this.isFinish = false;
        this.isFindPath = false;
        this.path = [];
        this.initMap();
    }

    initMap() {
        this.mapData = new Array(this.mapSize);
        for (let i = 0; i < this.mapSize; i++) {
            this.mapData[i] = new Array(this.mapSize);
            for (let j = 0; j < this.mapSize; j++) {
                this.mapData[i][j] = new Node(i, j, 'blank');
            }
        }

        this.randomBarrier();
        this.setEnterAndExit();
        this.addNodeToOpenSet(this.enterNode);
    }

    // 随机生成障碍物
    randomBarrier() {
        // 障碍物的数量按总方块数的1/5
        let barrierCount = Math.round(this.mapSize * this.mapSize / 5);
        for (let i = 0; i < barrierCount; i++) {
            let randX = this.getRandomInt(0, this.mapSize);
            let randY = this.getRandomInt(0, this.mapSize);
            this.mapData[randX][randY].type = 'barrier';
        }
    }

    getRandomInt(min, max) {
        min = Math.ceil(min);
        max = Math.floor(max);
        return Math.floor(Math.random() * (max - min)) + min; //The maximum is exclusive and the minimum is inclusive
    }

    // 设置起点和终点，其中起点设置为左侧中间位置，终点为右侧中间位置
    setEnterAndExit() {
        let enterX = Math.floor(this.mapSize / 2);
        let enterY = 0;
        let exitX = enterX;
        let exitY = this.mapSize - 1;
        this.mapData[enterX][enterY].type = 'enter';
        this.mapData[enterX][enterY].totalCost = 0;
        this.mapData[exitX][exitY].type = 'exit';
        this.enterNode = this.mapData[enterX][enterY];
        this.exitNode = this.mapData[exitX][exitY];
    }

    // 获取 openSet中代价最小的节点
    getMinCostNode() {
        let minCost = Number.MAX_VALUE
        let node = null;
        for (let key in this.openSet) {
            if (this.openSet[key].totalCost < minCost) {
                minCost = this.openSet[key].totalCost
                node = this.openSet[key];
            }
        }
        return node;
    }

    // 规划下一步的路径
    nextStep() {
        if (this.isFinish) {
            return;
        } else if (Object.keys(this.openSet).length === 0) {
            this.isFinish = true;
            return;
        }
        let node = this.getMinCostNode();
        if (node.isExit()) {
            this.isFinish = true;
            this.isFindPath = true;
            this.updatePath(node);
        } else {
            this.deleteNodeFromOpenSet(node);
            this.addNodeToCloseSet(node);
            this.addNeighborNodeToOpenSet(node);
        }
    }

    updatePath(node) {
        while (node != null) {
            this.path.push(node);
            node = node.prev;
        }

        this.path.reverse();
    }

    // 处理临近节点，为简化逻辑，假设只允许上下左右4个方向移动
    addNeighborNodeToOpenSet(node) {
        this.processNodeCoordinate(node, node.x - 1, node.y)
        this.processNodeCoordinate(node, node.x, node.y - 1)
        this.processNodeCoordinate(node, node.x + 1, node.y)
        this.processNodeCoordinate(node, node.x, node.y + 1)
        //this.processNodeCoordinate(node, node.x - 1, node.y - 1)
        //this.processNodeCoordinate(node, node.x + 1, node.y + 1)
        //this.processNodeCoordinate(node, node.x - 1, node.y + 1)
        //this.processNodeCoordinate(node, node.x + 1, node.y - 1)
    }

    // 处理节点，若合法有效则加入到openSet中
    processNodeCoordinate(prev, x, y) {
        if (!this.isValidCoordinate(x, y)) {
            return;
        }
        let key = `${x},${y}`;
        if (!this.mapData[x][y].isBarrier() &&
            typeof (this.closeSet[key]) === 'undefined' &&
            typeof (this.openSet[key]) === 'undefined') {

            this.calcNodeTotalCost(this.mapData[x][y]);
            this.mapData[x][y].prev = prev
            this.addNodeToOpenSet(this.mapData[x][y]);
        }
    }

    // 添加节点到openSet中
    addNodeToOpenSet(node) {
        let key = `${node.x},${node.y}`;
        node.isCandidate = true;
        this.openSet[key] = node;
    }

    // 添加节点到closeSet中
    addNodeToCloseSet(node) {
        let key = `${node.x},${node.y}`;
        this.closeSet[key] = node;
    }

    // 检测坐标是否有效
    isValidCoordinate(x, y) {
        return x >= 0 && x < this.mapSize &&
            y >= 0 &&
            y < this.mapSize;
    }

    // 从openSet中删除某个节点
    deleteNodeFromOpenSet(node) {
        let key = `${node.x},${node.y}`;
        delete this.openSet[key];
    }

    // 计算节点的总代价（越小优先级越高)
    calcNodeTotalCost(node) {
        node.totalCost = this.gCost(node) + this.hCost(node);
    }

    // 计算节点到起点的代价
    gCost(node) {
        return node.distance(this.enterNode);
    }

    // 计算节点到终点的代价
    hCost(node) {
        return node.distance(this.exitNode);
    }
}

// 节点类
class Node {
    constructor(x, y, type) {
        this.x = x;
        this.y = y;
        this.prev = null;
        this.totalCost = null;
        this.type = type;
        // 候选路径
        this.isCandidate = false;
    }

    isEnter() {
        return this.type === 'enter';
    }

    isBlank() {
        return this.type === 'blank';
    }

    isBarrier() {
        return this.type === 'barrier';
    }

    isExit() {
        return this.type === 'exit';
    }

    // 计算离另一节点的距离
    distance(node) {
        let dx = Math.abs(node.x - this.x);
        let dy = Math.abs(node.y - this.y);
        return dx + dy;
    }
}
</script>

上面的算法实现中，获取openSet中代价最小的节点采用了遍历查询的方式，从运行效率上来讲比较低，更高效的方法是采用优先队列来实现。


其中Canvas的绘制部分，使用了[p5.js](https://p5js.org/){:target="_blank"}的js库

算法的描述与说明部分参考了[https://paul.pub/a-star-algorithm/](https://paul.pub/a-star-algorithm/){:target="_blank"}
