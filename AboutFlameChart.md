关于Flame Chart
----


第一篇： http://www.html5rocks.com/en/tutorials/developertools/revolutions2013/#toc-flame-chart 翻译

Flame chart visualization for JavaScript profiles
--------
Flame Chart视图提供了一种对Javascript处理过程的分时展现，类似Timeline和Network面板中的那样。
![flame chart](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xflame-chart-1.png.pagespeed.ic.MYq3YZ6WGt.webp)

水平坐标是时间，垂直坐标是调用栈。面板顶部是整个记录的展示，你可以用鼠标选中它，并且放大（或缩小）其中的某个区域（注：通过鼠标滚轮或者拖拽上面的游标），详细视图会相应的变化。

![Flame chart zoomed in](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xflame-chart-2.png.pagespeed.ic.A2Pjoa9vw7.webp)


在详细视图中，一个调用栈被展现成一些函数“块”堆叠成的栈。堆在另一个块上的函数被它下面的函数块调用。鼠标移上某个块显示它的函数名和一些时间数据：

  - Name — 函数名。
  - Self time — 函数完成当前调用花费的时间，仅包含函数自己的语句块，不包含其他被它调用的函数（花费的时间）。
  - Total time — 函数完成当前调用花费的时间， 包括任何被它调用的函数（花费的时间）。
  - Aggregated self time — 函数在（某个调用过程的）记录期间所有调用的时间总和，不包含被这个函数调用的其他函数（花费的时间）。
  - Aggregated total time — 函数在（某个调用过程的）记录期间所有调用的时间总和，包含被这个函数调用的其他函数（花费的时间）。

> 注：可以参照第二篇中例子的调用图，run100在run100d中被调用两次，那么当鼠标指向第一个run100时，上面的数据表示的分别是
   * self time：run100调用花费的时间，不包含now等被run100调用花费的时间
   * total time：run100调用花费的时间，包含now等被run100调用花费的时间
   * Aggregated self time： run100在run100d中调用两次，这两次调用花费的时间总和。不包含他们调用的now等函数花费的时间
   - Aggregated total time — run100在run100d中调用两次，这两次花费的时间总和。包含他们掉哟给你的now等函数花费的时间


![Flame chart showing timing data](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xflame-chart-3.png.pagespeed.ic.KPXonZwe5y.webp)

点击函数块会在Source面板中打开包含它的Javascript文件，并定位到定义这个函数的地方。

![Function definition in Sources panel](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xflame-chart-sources.png.pagespeed.ic.GO8QoRuZa0.webp)

要使用flame chart：
* 打开DevTools，点击Profiles面板。
* 选择Record Javascript CPU profile选项，点击Start。
* 要结束数据收集，点击Stop。
* 在Profile中，选择Flame Chart视图。

![use](http://1-ps.googleusercontent.com/x/s.html5rocks-hrd.appspot.com/www.html5rocks.com/en/tutorials/developertools/revolutions2013/xflame-chart-menu.png.pagespeed.ic._p2hfh6R_4.webp)




----------------------
第二篇：
https://github.com/jbdeboer/flame-chart-observations 翻译

Chrome's Flame Chart(火焰图？)
=========
Chrome在它的devTool的Profiler选项卡中新增了一种调试工具：flame chart。不像其他已有的Javascript profiler视图，flame chart能够告诉你某个函数在什么时候执行，而不仅仅是它花费了多长时间。

flame chart生成许多数字，他们代表什么含义？

![Sample flame chart](http://www.huronbox.com/~james/flame-chart/flame-chart-twitter.png)

flame chart 是一个基于样本的审查器：
  - 它通过webkit从V8 profiler中采集样本。
  - 在这些样本的基础上插入一组时间序列。（把这些采集的样本按照时间顺序展现出来）


样本大约每毫秒被采集一次。(These samples are taken approximately once every millisecond.  The flame chart interpolates these samples and assumes if the program is in the same place as the last sample then the program was in that place the whole time.) 上面这句不知道怎么翻译好，place指的是不是采样时间点？

小而快函数调用可能会有不完整的图；但总的来说整个栈图是相当准确的。

The Numbers（数字的意义）
-----------

鼠标移动到任意一个函数调用时会弹出一个带有一些关键数字的浮层。

![Flame chart showing function call numbers](http://www.huronbox.com/~james/flame-chart/flame-chart-with-numbers.png)

这是“run100d”调用的弹出层。在这个程序中，函数run100a,b,c,d被精确的设计成执行200ms。run100d调用run100两次，它（run100）重复调用`window.performance.now()`。
- *Name*：函数名。
- *Self time*：run100d在顶帧时的*number of samples* （样本数）。（因为每次采样大约1ms，因此这里的样本数number of samples应该就是耗费的毫秒数）
- *Total time*: run100d在栈中的*number of samples*（样本数）
- Aggregated self time: 调用栈中函数执行时以run100d作为顶帧（应该是指当前调用栈顶是run100d函数）的时候花费的时间。
- Aggregated total time: 调用栈中函数执行的时间。


上面的四个数字中，“Aggregated total time”是最有用的。在我的例子中，run100d只被调用一次，因此“Aggregated total time”就是run100d执行消耗的总时间。

run100d调用run100两次，run100的统计数字包含了所有run100d对run100的调用（两次）。而run100c对run100的调用的统计数据是不同的。

“self time”和“total time”是失实的。尽管它们都声称是毫秒级的，但它们事实上是通过flame chart观察的样本的数据。
   
Sampling
--------
flame chart大约每毫秒采集一个样本。实际上这个频率约为1.05ms。

为了测量它，我创建了10个循环函数，每个1/10毫秒。而实际上flame charts显示了每个采样点每毫秒大约产生了0.05ms的偏移。
见sample_rate.html的测试

![Sampling rate drifts](http://www.huronbox.com/~james/flame-chart/tenths.png)


--------------


总结
------
Flame Chart的原理应该是大约每毫秒获取一次profile中的内存堆数据，然后将相关的调用函数名提取出来，形成当前这一毫秒的调用栈，重复每一毫秒，形成按时间顺序的栈图。（还可以看到垃圾回收的执行哦～）

因为每毫秒获取一次样本，因此某个函数连续样本数基本上就约等于这个函数的调用时间，从而生成“self time”和“total time”（都是整数）这样的通过样本观察的数据，样本观察频率作者通过实验得出不是1ms而是大概1.05ms（估计采集需要时间吧）也正因为如此，作者才说这两个数据是不精确的。
