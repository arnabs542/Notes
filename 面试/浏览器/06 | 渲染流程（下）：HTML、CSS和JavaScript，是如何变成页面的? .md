## 06 | 渲染流程（下）：HTML、CSS和JavaScript，是如何变成页面的？
分层、绘制、分块、光栅化、合成。
### 分层
渲染尽情还需要为特定的节点生成专用的图层，并生成一颗对应的图层树(LayerTree)。
![20200901055631](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901055631.png)
最终合成的页面
![20200901055707](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901055707.png)
通常情况下，并不是布局树的每个节点都包含一个图层，如果一个节点没有对应的层，那么这个节点就从属于父节点的涂层。途中的span标签就是。
![20200901055746](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901055746.png)
**可被提升为涂层的元素需要满足两个条件**
1. 拥有层叠上下文属性的元素会被提升为单独的一层。
![20200901060023](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901060023.png)
从图中可以看出，明确定位属性的元素、定义透明属性的元素、使用 CSS 滤镜的元素等，都拥有层叠上下文属性。
2. **需要剪裁（clip）的地方也会被创建为图层。**
剪裁就是，比如：我们设置一个div大小为200*200，但是div中p标签的文字比较多，文字所显示的区域会超出200*200的面积，这个时候会发生剪裁。
``` HTML
<style>
      div {
            width: 200;
            height: 200;
            overflow:auto;
            background: gray;
        } 
</style>
<body>
    <div >
        <p>所以元素有了层叠上下文的属性或者需要被剪裁，那么就会被提升成为单独一层，你可以参看下图：</p>
        <p>从上图我们可以看到，document层上有A和B层，而B层之上又有两个图层。这些图层组织在一起也是一颗树状结构。</p>
        <p>图层树是基于布局树来创建的，为了找出哪些元素需要在哪些层中，渲染引擎会遍历布局树来创建层树（Update LayerTree）。</p> 
    </div>
</body>
```
![20200901060445](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901060445.png)
![20200901060508](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901060508.png)

### 图层绘制
渲染引擎会将一个图层的绘制拆分成很多小的绘制指令。将指令排序成待绘制列表。
![20200901060750](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901060750.png)

### 栅格化（raster）操作
绘制只是用来记录绘制顺序和绘制指令的列表，而实际上绘制操作渲染引擎中的**合成线程**来完成的。
![20200901061350](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901061350.png)
**视口**
什么是视口？
![20200901061513](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901061513.png)
基于网页主页面很大，合成线程会将图层划分为图块（tile）。
合成线程会按照视口附近的图块来优先生成位图，实际生成位图的操作是由栅格化来执行的。
所谓的栅格化，是指将图块转换为位图。
![20200901061633](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901061633.png)
图块是栅格化的最小单位，渲染进程维护了一个栅格化的线程池，所有图块栅格化都是在线程池执行的。
下图可与看出，渲染进程将生成图块的指令发送给GPU，然后再**GPU**中执行生成图块的位图，并保存在GPU的内存中。
![20200901061802](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901061802.png)

### 合成和显示
- 一旦所有图块被光栅化，合成线程会生成一个绘制图块的命令-“DrawQuad”，然后提交给浏览器进程。
- 浏览器用viz组件来接受DrawQuad命令
- 最后DrwaQuad命令，将界面内容绘制到内存中，最后显示在屏幕上。

### 渲染流水线大总结
整个渲染流程，从HTML到DOM、样式计算、布局、图层、绘制、光栅化、合成和显示。
![20200901062613](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901062613.png)
1. 渲染进程将 HTML 内容转换为能够读懂的 DOM 树结构。
2. 渲染引擎将 CSS 样式表转化为浏览器可以理解的 styleSheets，计算出 DOM 节点的样式。
3. 创建布局树，并计算元素的布局信息。
4. 对布局树进行分层，并生成分层树。
5. 为每个图层生成绘制列表，并将其提交到合成线程。
6. 合成线程将图层分成图块，并在光栅化线程池中将图块转换成位图。
7. 合成线程发送绘制图块命令 DrawQuad 给浏览器进程。
8. 浏览器进程根据 DrawQuad 消息生成页面，并显示到显示器上。


## 重排 重绘 合成 （重排大于重绘）
1. **更新了元素的几何属性（重排）**
![20200901062839](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901062839.png)
当你用JavaScript或者CSS修改元素的几何位置属性，比如改变元素的宽度、高度等
浏览器会触发信重新布局，这个过程叫**重排**。**重排需要更新完整的渲染流水线，所以开销也最大**。
2. **更新元素的绘制属性（重绘）**
![20200901063727](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901063727.png)
改了元素的背景颜色，布局阶段将不会被执行，直接进入了绘制阶段。**重绘省去了布局和分层阶段，所以执行效率会比重排操作要高一些。**
3. **直接合成阶段**
![20200901063844](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901063844.png)
直接合成，避开了布局和渲染的子阶段，**相对于重绘和重排，合成打打提升绘制效率**。


## 网页生成过程
网页生成过程：

1. HTML被HTML解析器解析成DOM 树
2. css则被css解析器解析成CSSOM 树
3. 结合DOM树和CSSOM树，生成一棵渲染树(Render Tree)
4. 生成布局（flow），即将所有渲染树的所有节点进行平面合成
5. 将布局绘制（paint）在屏幕上
6. 通过虚拟dom层计算出操作总得差异，一起提交给浏览器
![20200901065433](https://hzy-1301560453.cos.ap-shanghai.myqcloud.com/2020/pictures/20200901065433.png)

**渲染**
网页生成的时候，至少会渲染一次。在用户访问的过程中，还会不断重新渲染。

**重排比重绘大：**
重绘：某些元素的外观被改变，例如：元素的填充颜色
重排：重新生成布局，重新排列元素。