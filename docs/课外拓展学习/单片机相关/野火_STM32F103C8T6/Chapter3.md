# 神说要有光，于是有了光

很高兴你能看到这里，本章我们要做的是，点亮我们STM32上面的板载LED（随便哪一个都可以）。这也是我们在学习STM32时的第一个项目。

拿野火的STM32F103C8T6举例，要点亮的就是上面的那三颗blingbling亮亮的东西(这个东西长这么大是因为我买了他们专门的扩展板，将STM32直接查到扩展板上使用了，正常人使用的时候一般都是用面包板加其他的配件的)。

![野火STM32F103C8T6](相关图片/3-1%20野火STM32F103C8T6.jpg)

ok，现在我们开始我们的教程。

在上一章一个打括号的内容中我有说到过，LED接的都是PA接口，且这个PA接口实际上就是我们的GPIOA（主要是自己要会去看PCB（就是电路图啦，只要看懂哪个接口控制就可以了，门槛比电路低多了），也自己要会去翻参考手册），而且，这个接口是接在LED的负极上的。于是就有了这个——“实际上在原理图中可以看出，在这块板子上，LED的正极是始终接3.3V的正电压，而PA1接的是LED的负极，也就是说LED不发光的原因实际上是因为LED两端的电压并没有达到正向导通电压的最低要求（1.2~4V），所以LED不发光。如果说想让他发光，用数电的理解就是给一个低电平信号就可以了。”（第二章的内容）。

但是实际我们在书写的时候我们会发现一个问题：按照我们C语言的思路，我们在写的时候应该是把端口看成是我们的变量，然后对这个对象进行编程操作。然而，在实际写的时候你会发现这样子是行不通的，因为根本没有GPIO这个变量（也别想着去新建一个GPIOA变量了，要不然就相当于重新写一个库了，而且还有一些寄存器的东西，很麻烦）。所以我们直接采取用正经的 库函数 的方法来书写这个功能。

（看到这里时，您可以先不往下拉进度条，先看着这个题目来自己尝试书写一下）
下面请听题：请使用STM32和其标准库来编写对应的程序（Template文件在之前的资料包中，按照上一章的内容来弄一下就可以了）。

程序的书写在 User->main.c 中进行书写。
要用到的函数有：RCC_APB2PeriphClockCmd、GPIO_SetBits、GPIO_Init、GPIO_ResetBits。
要用到的type类型有：GPIO_InitTypeDef。
（请学会善用 选中函数之后用鼠标右键=>前往定义(Go To Definition Of 'xx') 这一功能，详细见下图片）
（没有什么想法的话可以把这几个函数跟和类型复制到keil5中，然后去前往定义就可以了）

![前往定义](./相关图片/3-2%20前往定义.png#pic_center)

如果您能直接按照上面的提示来写出这个程序的话，那么您是真的很天才，已经不需要我这样的学习笔记辅佐您学习了。
如果您没有什么想法的话，不要气馁，这是正常的，毕竟我这份笔记在这个时候还没有写到对标准库函数的介绍。接下来我们简单的来介绍一下标准库函数。

在我所提供的Template文件中，主程序main.c（记得用Keil5打开，Template->Project中有一个 .uvproj 文件并且有Keil5的图标的，双击就可以了）中有一行这样的include：

![include](./相关图片/3-3%20include.png#pic_center)   

这个include导入的就是STM32的标准库，而所有的库函数则在左侧栏中的FWLIB中可以找到：

![Keil5左侧栏](./相关图片/3-4%20Keil5左侧栏.png#pic_center)

回到我们上一章说过的内容，PA1是在GPIOA底下的，GPIOA是在APB2中的，APB2是“AHB系统总线”的，而控制“AHB系统总线”的是“复位和时钟控制（RCC）”，然后我们可以发现RCC和GPIO在左侧栏中的FWLIB中都是有对应的文件的（一个是xxxx_rcc.c，还有一个是xxxx_gpio.c），根据上面的提示我们可以去找一下这几个函数有什么用。
（看到这里时，您可以先不往下拉进度条，先看着这个题目来自己尝试书写一下）

![系统结构图](./相关图片/2-2%20系统结构图.png#pic_center)

然后您就会发现，这些函数写的使用说明，全都是英文的……
请不要惊讶，因为这个标准库最早都是由外国人搭建的，就算是中国人搭建的，也是要给外国人使用的，可以理解是国际统一标准。
这时候您可以使用您的AI工具来帮您翻译这些内容，真挺好用的。
现在您要做的就是，翻译上面我所提供的提示函数，然后用这些函数去编写你的点亮LED的启动程序。
（注意，在使用前往定义这一功能的时候，不要拘泥于只查函数的，可以去看看函数后面的形参的定义是怎么样的）

如果您看到我这边的内容之后，还是对这个没有什么想法的话，不要气馁，因为我学到这里的时候也是没有什么想法的，说明您跟我的水平是大差不差的。而我是个很正常的人，这也变相说明了您也是个很正常的人，断然是没有比别人笨很多说法的。

所以，我现在想跟您分享的是，我们应该如何从零开始去编写我们的工程。
这里我采用vscode来进行工程项目的撰写，如果有个别不同的我会注明（因为vscode能加皮肤，还有让我很有编写欲望的高亮，咱就是说，咱很喜欢）。

我们看着上述的提示，先打开我们的Template，然后打开User中的main.c（连这一步都做不到的话，那您应该考虑放弃了），他长这样：

![main函数](./相关图片/3-6%20main函数.png)

这个main函数就是我们放代码的地方，上面的注释部分是给我们拿来写简介用的，这也是我的一个编程习惯，当然你不喜欢的话也可以放到README里面，甚至是不写。但是我还是建议你写一下，以免有人来问你或者你自己之后学习的过程中忘记了。

然后，我们把我们的相关函数放进去（RCC_APB2PeriphClockCmd、GPIO_SetBits、GPIO_Init、GPIO_ResetBits），用 右键->前往定义 的方式来去查看这个函数到底怎么用的（不要被庞大的英文和乱七八糟的形参给吓到，我们现在生活在一个充满着科技感的时代，好好利用你的AI工具）。

当你看完所有的函数之后，你会发现有一个地方要用到一个奇奇怪怪的自定义的结构体：GPIO_InitTypeDef，猜一下他是什么意思？
好叭，你应该也猜不到是什么意思，其实可以直接简单的去理解一下——GPIO是我们的接口，Init是初始化的意思，我们要Init（初始化）我们的GPIO（接口），学过面向对象程序设计的你应该能理解这一步是什么操作了，实际上就是构造一个对象，能把我们的GPIO接口从现实中带到我们的代码中进行操作。就我个人来理解，我这一步操作称呼为——实例化。
如果到这一步你还是不理解实例化到底有什么作用的话，就先将
```
GPIO_InitTypeDef gpio_init;
```
这串代码放在你的int main之后叭，你之后会知道的。

ok，现在你已经知道了init是初始化配置的操作，setbit跟resetbit很好理解，设置跟置零的区别，用数电的话来讲就是置一和置零的关系，还记得这章的开头说了什么吗？不记得的话再回去看看，你将会对这两个函数恍然大悟。而我们的rcc_clk是干嘛的呢？时钟信号不要忘了。什么？你说你不知道时钟信号是什么意思有什么用？我相信[这篇blog](https://blog.csdn.net/weixin_44395686/article/details/105318472)你会用上的。

所以我们现在只要给接口设置上对应的配置（回去瞅瞅你的GPIO_InitTypeDef的相关定义，就知道要配置什么了），记得模式是推挽输出模式（建议是去看一下使用手册中GPIO的那一大章，看完就应该知道推挽是什么意思了。但大多数人心急（我也是），所以也可以看[这篇介绍推挽输出模式的blog](https://blog.csdn.net/weixin_44788542/article/details/115303125)）

下面给出思考题跟完整代码：

```
#include "stm32f10x.h"

void LED_Config(void);

void LED_ON(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
void LED_OFF(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);


int main(void)
{
	LED_Config();

	LED_ON( GPIOA,GPIO_Pin_1);
	LED_ON( GPIOA,GPIO_Pin_2);
	LED_ON( GPIOA,GPIO_Pin_3);

    while(1);
}


void LED_Config(void)
{
	GPIO_InitTypeDef gpio_init ;	                      //初始化端口
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE); //开启端口时钟
	
	//关闭灯（初始化灯的状态）
	GPIO_SetBits( GPIOA, GPIO_Pin_1 );                    //让引脚端口1输出1，使得灯灭
	GPIO_SetBits( GPIOA, GPIO_Pin_2 );                    //让引脚端口2输出1，使得灯灭
	GPIO_SetBits( GPIOA, GPIO_Pin_3 );                    //让引脚端口3输出1，使得灯灭
	
	//配置io模式，推挽模式，50M
	gpio_init.GPIO_Pin    = GPIO_Pin_1;
	gpio_init.GPIO_Mode   = GPIO_Mode_Out_PP;
	gpio_init.GPIO_Speed  = GPIO_Speed_50MHz;			
	GPIO_Init(GPIOA, & gpio_init);                        //配置端口引脚的模式
	
	gpio_init.GPIO_Pin    = GPIO_Pin_2;
//	gpio_init.GPIO_Mode   = GPIO_Mode_Out_PP;
//	gpio_init.GPIO_Speed  = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, & gpio_init);                        //配置端口引脚的模式	
	
	gpio_init.GPIO_Pin    = GPIO_Pin_3;
//	gpio_init.GPIO_Mode   = GPIO_Mode_Out_PP;
//	gpio_init.GPIO_Speed  = GPIO_Speed_50MHz;	
	GPIO_Init(GPIOA, & gpio_init);                        //配置端口引脚的模式	
}

void LED_ON(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
{
	GPIO_ResetBits( GPIOx, GPIO_Pin );					//让引脚端口x输出0，使得灯亮
}

void LED_OFF(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
{
	GPIO_SetBits( GPIOx, GPIO_Pin );					//让引脚端口x输出1，使得灯灭
}

```

（这个代码看上去还是很冗杂的，有没有其他的办法可以让我们把这个代码给简化呢？或者，更精确一点——把这个 main 函数给简化呢？）