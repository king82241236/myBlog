# 如何自己创建一种编程语言
正好前段时间做了几个JS语言编译器的DEMO，我觉得我可以说说作为一个野路子是如何用LLVM实现语言的。这个还是比较初级的，大佬可以忽略，但或许可以帮助一些刚刚入门想要用LLVM实现语言的朋友。

# 照老虎画猫，实现第一个JS编译器
在制作JS二进制编译器的时候，我首先是参考了LLVM官方的的万花筒语言的编写教程（[教程地址](https://github.com/zy445566/llvm-guide-zh/blob/master/README.md)）实现了第一个DEMO（[DEMO的详细讲解地址](https://github.com/zy445566/myBlog/blob/master/20180825llvm/20180825jsvm_c/README.md)）,当然因为是DEMO所以功能不全也BUG多，但是制作套路还是能说说的。制作流程可以分为以下几步。

* 编写AST用于分析语言结构
* 将分析的语言绑定生成IR(中间语言)
* 生成二进制或汇编代码

第一步，简单来说就是写一个代码解析器，把代码解析成一个足够描述代码特征的对象，我们一般叫它AST

第二步，把你这个对象进行一个中间语言IR对象的绑定（LLVM自己实现了一套语言叫IR，这是一个中间语言。）

第三步，就是用LLVM把你绑定好的IR对象，生成IR语言的代码，或者直接生成编译器文件。

可能大家会有点蒙，不是很明白意思。那我一句话总结就是：把你的要实现的语言代码生成一个对象，再把这个对象绑定到IR语言的对象上，LLVM就能直接通过IR语言的对象生成IR代码，IR代码对应了机器码，就可以直接用LLVM生成机器码。其实这一套就是编译器的流程，LLVM最终生成的就是能实现上面功能的编译器了。

走到这里，由于很多人对LLVM还不是很明白，所以上面的话还是不是很懂，那我再白话一点好了。

先提一个问题，如果被关在没有网络的房间里只有一本CPU指令集和一台该CPU的电脑，你会怎么实现一个简单的语言？

其实就很简单了，你只要把你的语言的基础关键词对应到CPU指令集，再基于基础关键字上增加新关键字功能就好了。就比如指针移动粗暴一点直接对应到CPU的跳转地址的指令上就好了，然后再用指针移动实现跳转到地址执行，从而实现类似于IF的功能。

那么基于上面来说LLVM的实现就好讲多了。

那么上面一步就相相当于实现了IR到CPU指令集的转换。

但如果你要实现更好维护的语言，是否要实现IR的一个对象，这样的话只要修改IR对象，再将IR对象生成机器码，是不是就好维护多了？这就是绑定IR的原因。

但你后来发现写IR远远不够我高级语言的需求，同样为了维护性，是不是把我更高级的语言生成AST，再转成IR，这样我是不是就不用转机器码了？这就是我要生成AST树的原因。

# 第二个JS编译器的实现：我能否直接用现成的AST树工具呢?
由于要实现一个语言的AST树所花的时间非常多，个人经历又有限，写出来BUG又多，干嘛不用现成的JS的AST工具呢？

基于这个想法我想到了JS有名的babel来制作我第二个JS编译器（[第二个JS编译器的详细讲解地址](https://github.com/zy445566/myBlog/blob/master/20180825llvm/20180908jsvm_js/README.md)），有了AST树转换工具那对于我来说就是如虎添翼了(偷懒了)。

并且直接用node的llvm-node库来做IR绑定，剩下的就用node直接生成编译器了。步骤和上面差不多，我有点累了，就不细讲了，有空自己看吧。

目前来说LLVM出现让语言制造到达了前所未有的方便，否则你可能要考虑创建一个类似汇编的中间语言来实现在多平台运行，同时支持实现转化到不同CPU的机器码，这个很多搞软件编程的同学来说这是相当困难的。还是那句话“LLVM大法好啊”