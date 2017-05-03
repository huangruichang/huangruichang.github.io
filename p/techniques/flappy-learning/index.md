# FlappyLearning 源码笔记

## 意图
这个项目是通过神经网络算法和遗传算法去训练出一只可以持续飞行的 flappy bird，以展示机器学习的强大和魅力。项目地址:[https://github.com/xviniette/FlappyLearning][1]

## 深度学习科普
关于深度学习的科普[《神经网络浅讲：从神经元到深度学习》][2]和关于遗传算法的科普[《非常好的理解遗传算法的例子》][3]，个人认为都比较清晰易懂。

## 与 BP 神经网络比较
在这里项目里，作者没有使用 BP 神经网络进行训练，个人认为，可能主要是 flappy bird 这个游戏最求的是越高分越好，无法产生有限数据集供以训练，所以作者在训练的时候引入遗传算法，通过突变和择优代替 BP 神经网络里的反向传递以调整神经网络的参数。

## 项目结构

[FlappyLearning][4] 的目录结构如下:

![在这里输入图片描述][5]

项目看上去非常简单，基本包括图片资源、证书和代码。index.html 是项目的入口。game.js 是 flappy-bird 的实现。Neuroevolution.js 是提供深度学习工具的 js。

## Neuroevolution.js 在 game.js 里的调用
game.js 里面实现 flappy bird 的游戏主体非常清晰并且容易理解，所以就不赘述了。在 game.js 里面，调用 Neuroevolution.js 里的工具的地方有三处：

初始化：
````
var start = function(){
	Neuvol = new Neuroevolution({
		population:10,
		network:[2, [2], 1],
	}); // 这句
	game = new Game();
	game.start();
	game.update();
	game.display();
}
````

生成小鸟后代：
````
Game.prototype.start = function(){
	this.interval = 0;
	this.score = 0;
	this.pipes = [];
	this.birds = [];

	this.gen = Neuvol.nextGeneration(); //这句
	for(var i in this.gen){
		var b = new Bird();
		this.birds.push(b)
	}
	this.generation++;
	this.alives = this.birds.length;
}
````

通过神经网络计算行为和记录结束得分：
````
for(var i in this.birds){
	if(this.birds[i].alive){

		var inputs = [
		this.birds[i].y / this.height,
		nextHoll
		];

		var res = this.gen[i].compute(inputs); // 这句
		if(res > 0.5){
			this.birds[i].flap();
		}

		this.birds[i].update();
		if(this.birds[i].isDead(this.height, this.pipes)){
			this.birds[i].alive = false;
			this.alives--;
			Neuvol.networkScore(this.gen[i], this.score); // 这句
			if(this.isItEnd()){
				this.start();
			}
		}
	}
}
````


## Neuroevolution.js 解读

### 基础结构
让我们看看 Neuroevolution.js 内部是如何工作的。这个文件整体是一个类，叫 Neuroevolution。里面有7个子类，分别是：Generations、Generation、Genome、Network、Layer、Neuron。

Generations: 生代集，用于管理 Generation 的类。
Generation: 生代，用于存储和管理 Genome 的类，拥有使 Genome 杂交的能力。
Genome: 基因组，用于存储和管理得分和神经网络的类。
剩下的 Network，Layer 和 Neuron 就是神经网络的基本元素，参考上文的科普文章。


### 结构图
![在这里输入图片描述][6]

### 流程图
![在这里输入图片描述][7]
通过不断的择优、突变和杂交，最终产生出接近最优的神经网络，这个项目的最终结果就是训练出一只一直飞的小鸟，amazing。

## 总结
深度学习就像上一年的 VR，虽然火热得像是炒作，但是不影响它对社会、对技术的价值，我认为程序员都应该去学习，或者去了解深度学习。它本身也很酷不是吗？
作为一个机器学习的门外汉，通过两个星期的学习，对深度学习产生了感性的认识。但是对于正统的数理统计公式还是一头雾水，连 tensorflow 的入门 demo，mnist 都不太看得懂。虽然能大致看懂这个项目的源码，但是距离使用深度学习解决问题还有很长的一段距离。

## 最后
本人才疏学浅，如有纰漏，还望指正。


  [1]: https://github.com/xviniette/FlappyLearning
  [2]: http://www.cnblogs.com/subconscious/p/5058741.html
  [3]: http://blog.csdn.net/b2b160/article/details/4680853/
  [4]: https://github.com/xviniette/FlappyLearning
  [5]: https://dn-coding-net-production-pp.qbox.me/b6c1fbd8-6814-4fcd-b4b1-f1f75bc3a1a5.png
  [6]: https://dn-coding-net-production-pp.qbox.me/f565fd05-4a7a-4007-8552-e553a1416462.png
  [7]: https://dn-coding-net-production-pp.qbox.me/d9685794-51e5-45a7-bf24-b0463e4245c5.png