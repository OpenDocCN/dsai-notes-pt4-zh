# 人工智能—机器学习中的数学（七月在线出品） - P1：Taylor展式与拟牛顿 - 七月在线-julyedu - BV1Vo4y1o7t1

这次我们探讨它的展示与它的相关应用，如米牛顿。我们首先给出塔的展示的本身的，它的定义，它的展示的公式的本身。然后我们利用它来计算某一些函数的近似值，解释一下经济系数，它到底内部原因是什么。

然后我们看一下平方公式又是为什么是可以如此计算的。我们下面呢重点来探讨一下牛顿法，它的相关的内容。比如说我们先探讨一下它的梯度下降算法。在此基础之上，我们给出牛顿法以及拟牛顿它相应的一些内容。首先。

如果给定一个函数FX它如果性质足够的好，意味着如果说FX在某一点X0处，它具备可以计算它的N阶导数。那么说FX就可以在X0处做N阶的台脑展开，得到这样一个式子。

它的意思指的是在X零处的函数值加上X与X零的差值乘上一阶导。XX零的差值的平方乘上二阶导，当然有一个二的阶乘的系数。如果是N阶导的话，就一个N的阶乘的一个系数。这就是塔勒展示。如果我们令X0等于0。

我们在0X等于零在原点处做塔的展开，这样的式子就是麦克ling公式。所以麦克lning公式和它的展示，它的区别仅仅是展开的值本身不一样而已。本质它们是一个内容。那么我们利用这样一个公式。

非常方便的就能够来去计算一些初等函数的值。事实上，我们在原点对sine X做展开，sine导数是负的cosine X负cosine X的导数。换句化讲，sine X二阶导就是sine X本身。

那这样子就能够得到它的它的展示是这样的一个一个值。我们如果是取它的前边的若干项的和，就可以作为3X某一个弧度数，它的正弦值的近似。比如说做一的X方等于这样一个值，而这样一个过程，如果令X等于一。

那就意味着这是一。所以我们可以把后面的值写成。I的阶乘分之1，让这个I从零到无穷大，这样的一个加和最终就等于E的一次幂。这其实可以看作是E的一个定义。就是。自然呃自然数的接乘的倒数的和就是一。

当然我这个自然数是从零开始数的。事实上我们如果说让去计算，比如说E的100次方呢。计算一个非常大的一个数的若干次幂。那么说我们如果还是在零点处做材了展开的时候，往往这样一个加和项。

它的误差是比较大的那这样子我们在实践中可以做一定程度的一个小的变化。比如说我们给定一个正实数X，如果让大家计算一下100次方等于几呢？当然不能够利用系统函数，我们自己去做一个这样的过程。

一种可行的思路我们可以这么来处理，就是我们总是可以把这个X写作是login2的若干倍，加上一个余项的形式。这个K是一个整数，R是一个小数。实际上这个式子我们可以把它看作比方说大家要把123。

可不可以写成十的若干倍加上余项啊，当然可以可以写成12的10次幂加上3，这个十就是这个式子里面这个login2。我们要求的就是这个A和这个R。呃，这个K和这个R其实就是这个意思。所以任意给另一个X。

我们总是可以对浪2去。做除法，然后得到它的。我们要做的K和这个R的。那么说如果我们做到这一步了，一的X方就可以写成这个式子，这是两个数的加和取指数，就是两个数各自取指数的乘积。

而这个式子E的Lin2乘以K，可以写作一的Lin2的K次幂。一的Lin2，这是二的指数在对数再取指数，所以它本身就等于2，这是K次幂。所以这个数就是二的K41，这个一的2次方。

那如果在K是一个整数的时候，二的K4幂是非常方便计算它的值的。而R是一个从负0。5的L文2到正的0。5倍的login2，一个在原点处附近的一个值。所以我们就可以把E的R次方利用刚才的塔的展示把它给展开。

那么说E的R次方就能够做近似计算了。如此一来一乘就可以计算得到E的X方。事实上，在实践当中，有些语言真的就是利用这样一个机制来去计算的逆次运算。我们再来看它的它展示的第二个应用。

就是我们在决策数随机森明中会提到一个非常重要的概念。基尼系数，这是一个在实践中也是很重要的。比如说看一下社会上的贫富差距，我们也是用的基尼系数。而基尼系数呢，它的定义是这么来做的。

就是在某一个类别它发生的概率，乘以这个类别不发生的概率。然后所有的类别把它都加起来。这样一个做法就是我们的基尼系数本身。而这个东西为什么是这样一个奇怪的定义呢？

事实上我个人觉得我们可以利用塔的展示做一个解释。因为。FX如果等于负的login X这么一个函数，我们总是可以让它在X等于一处做弹到展开，我们只去展开到一阶，高阶给忽略掉。

那么这样的FX在等于一处近似等于一减X。我们知道商的定义是这个。信息商越大就表示这个越它的不确定性越大，这是信息商的定义。我们要把负的login X在这，负的loginP的K次幂换成一减P的K。

也就变成了下面这个式子。所以说。基尼系数这样一个定义，它和商是非常近似的，只是可以认为商这么一个值在X等于一处做材的展开，忽略掉高阶项的一个近似而已。我们如果把这么一个近似的情况画的一个图上的话。

基尼系数是这么一个曲线，商是这样一个值。而这样的一个值，事实上我们给的这个是商的2分之1。因为这里面有一个log可以与一为底P的case幂，也可以以二为底，或者是以1为底，这只是一个系数而已。

事实上我们不去管它的系数本身，所以有一个系数为了保持它是一样的。这样子的话。不管是商还是基尼系数，他们都可以对分类的误差率有一个非常好的近似。所以说我们在实践当中就可以利用基尼系数做决策数的特征的选择。

做特征，它的分分类点的分割点的计算。这个我们在讲到决策数随机C运营那一部分的时候，再详细探讨它的展示它的这一方面的基济细数的应用。事实上我们可以这么来做一个事情，就是假定给大家任意的一个正数A。

比如说A等于。123，那么说求123的平方根等于多少呢？任给一个A，我们想去求A的平方根等于几？怎么做呢？我们可以试着这么来去考虑假定要算的值等于X。

那就意味着X的平方是等于A的那就意味着X平方减A是等于零的。好，我就定义FX是等于X平方减A。那我要做的事情就是求FX等于零的时候，X应该等于几呢？我们现在就把FX做一阶的它的展开，高阶放在里面去。

因此，高阶忽略掉等号变成了约等于符号。我们要做的是FX等于0，把这个零带入到方程的左边。那么说右边是这个式子，为了计算，为了表达方便，把这个约用符号换成了等于符号，这是一个近似相等的。

如果说在X0处它的导数是不等于零的时候，我们就可以把它变成这样一个形式，你们都去除以XX0的导数。把X0放到右边去得到这样一个值。这样的一个值，我们FX0知道的是X0的平方减A。FX等于X平方减A。

所以FX的导数是等于2X在X0处的导数呢就是两倍的X0，把这两个值带入到这个式子里面去，就得到了它。这个式子稍加整理就得到了这样一个迭代公式。就是我们可以在任意一个点X0，我们带入右边这个式子里面去。

就能够得到1个X1。我们把X1再带到这个里面去，就能得到1个X2，把X2带进去就得到X3。我们不停的做这个事情。在某一次，当XI的值和XI减一的值，它的差距足够小的时候。

我们就将XI输出作为根号A的平方根的一个近似节。这样子我们就可以利用这个公式将。A任意一个正数的平方根计算出来了，这个其实就是牛顿迭代公式。我们利用刚才这个公式能够非常方便的写出这样一个代码。

计算A的平方根。而这个式子核心代码就是这一句。事实上，这样一个代码大概只需要5到6次的迭代，就能够得到一个A非常好的一个近似值。而多说一句，我们题目中一般而言，这个是可以在任意一点做。做初值的。事实上。

我在程序里面假定用A的。一半作为。平方根的一个初值。比如说要算100的平方根，目标肯定是十了。但其实X或初值是等于50的。事实上，我们如果是给一个更加好一点的初值的话，能够减少迭代次数。

甚至于减少到一次或者是两次，有兴趣大家可以看一下相关的内容，相关的论文。好了，我们现在呢再来看另外一个大的话题，就是。牛顿法你牛顿。如此一来，我们就先要介绍一下稀度下降算法。

假定我们通过某一种技术手段想去估算某一个参数西塔。并且已经能够对这个西塔建立了一个目标函数。J西塔。比如说我们利用最小二乘法建立好的目标函数是这样子的这就是一个关于西塔的目标函数，或者是一个损失函数。

或者是一个。效育函数增加函数，比如说用最大自然估计，用等等等等手段。都能够建立好一个关于s塔的目标函数。那么说我们现在要做的事情是。那么说我们想去算一下某一个西塔星，当等于西星的时候。

这样一个损失函数可以取得最小值。那么说这个西塔星应该在哪儿呢？我们可以这么来计算，就是先出随机的或者是先验性的给定一个西塔的初值。我们假定记得西塔。然后我们沿着这个西塔的负的梯度方向。

让这个西塔G去做一个迭代，得到新的西塔G，然后不停的做这样一个事情。那么说就给定一个西塔初值，不停的迭代再下降，再下降，不停做这个事情，最终就能够把这个西塔得到一个局部的极小值，这就是梯度下降算法。

这个下应算法是我们在积济学习中非常重要，也是在时间当中用的非常多的一个内容。当然，梯度下降算法它的相对的概念就是梯度上升算法，它的本质是一样的，就取决于我们是取它的局部的最大值还是局部的极小值。所以。

负的就是梯度下降算法，正的就是梯度上升算法。比如说阿尔法 go在做棋盘的这个全值的计算的时候，早期是直接使用监督的学习方式去来计算一下，沿着梯度上升去做的。当然后期他做了一个增强学习，那是领说了。因此。

我们这里边要想做季度下降或者度上升，只需要算好七度，然后给定某一个合适的学习率，或者叫步长就可以完成这个内容了。那么说我们可不可以利用咱今天说的材到展示做一点点的变化呢？我们来看一下它的内容。

如果目标函数FX，它的二阶导是连续的。我们就可以FX在某一个点处XK处做它的展开，得到这样一个值。这是在SK处做塔的展开，然后忽略掉它的二阶以及更高阶的二阶以上的鱼子式放在无云小里面去。然后呢。

这样一个式子，XK是一个常数。而这个X是我们的某一个变量。因此，这样一个式子可以看作是关于X的一个函数。我们不妨记做sX，这是关于X的一个函数。因此，这一个函数忽略掉高阶的无穷小。

这个等号就是一个约等于符号。然后我们对这样一个函数，关于X求导数。对，这个求导数，这是一的。给他求导数。然后第一项就。因为我们对X求导数，所以它是等于零的。我们第二项。我们做第二项求导数。只有X。

所以这个是它的导数，这是一个E的，这写错了，这是E的。然后这是它的FFKX的一阶导，然后对这个求导数是两倍的这么一个值，这个二和这个二消掉了，所以是二阶导乘以这个值，也就是它。

所以这是一个约等于这样一个值。然后我们对这样一个式子，如果想去求FX的极值。那么说这个极值它至少需要是注点，换句话讲，它需要的导数是要等于零的。既然导数等于0，那么说我们让这样一个式子直接等于0。

如果这个时候FK到二阶导是不等于零的，我们就可以把它除过去，移过移过来就得到这样的一个公式。这个公式就是我们利用了二阶导的信息做的一个近似计算。而这个式子里面用到了二阶导数。事实上。

这样的一个迭代公式就是牛顿法。大家如果把我们刚才给定的。FX等于X平方减A，看作是一个一阶导的话，带入这个式子，马上就能够得到刚才我们说的A的平方根的牛顿迭代公式。和本质是一样的。

而我们刚才给定的这样一个计算过程，是假定FX是1元的。如果是多元的，它的过程是完全一样的。只不过如果多元，它的一阶导就是一个向量。ただ。二阶岛就是一个矩阵。非赠举证。事实上。

这样一个过程就是我们牛顿的迭代方迭代的一个方式。我们假定在红色的是我们的目标函数，我们想替它下降。当前假定找到的值是XK这一点在这儿。那么说我们如果在SK处做一个切线。在给定某一个学习率阿尔法K的时候。

那么说从SK到了下一个点，到了这个值，这其实本质上就是梯度下降算法。而我们用的牛顿法，其实就是给定这一点的函数值，这点的导数值和这点的二阶导数值做了一个抛物线。这个抛物线是虚的。蓝色的这条线。

然后求这个抛物线的梯度为0，也就是它的最小值这一点。那么说SK就变到SK加上D这么一个值。所以本质上牛顿法是用一个二次函数去做近似梯度下降方法是用一个一次的函数做的一个近似，如此而已。所以一般而言。

牛顿法它因为具有一个二阶的收敛性，在某些部分函数中，它的收敛速度是比T的下降要快的。比如说线性回归那这次回归，我们在继续学习的课程中，会跟大家来详细阐述它的相关的问题。虽然如此。

但是呢虽然它具有一个二次的收敛性，但是我们往往要要求初始点尽量的是比较靠近于极值点的。为什么呢？因为如果我们发现它的二阶。导数的矩阵黑色矩阵是其一的这个牛顿方向就是不存在的。

也就是这里边的这个啊二阶导数值等于零的情况。如果不是这样子。如果发生KC矩阵，它不是正定的，那么它的牛顿方向甚至于都可能是相反的方向。等会儿我们看一个小例子。

另外呢我们在计算过程之中需要算一个黑en矩阵的逆矩阵。二阶偏导数的逆实实上计算的时间复杂度也是存在的。因此，牛顿法虽然具有二阶收敛性，在实践当中，某些目问题中也是发挥了重要作用。

但是往往我们需要对牛顿法在稍微的加一点点的改进。我们先来看第一个问题，就是黑en矩阵。如果不是正定的时候，它的搜索方向或许成了反方向了。左边这个图是我们刚才看到的那个正常的牛顿法的迭代方式方式。

但是如果我们的目标函数是红色这条线，假定当前值XK在这一点处，显然我们想找到这个值是我们的目标的一个局部的极小值。但是如果在这一点就给定了它的函数值一阶导和二阶导的时候。那么说我们给出的这个。

牛顿的近似的曲线，这个抛物线就是这样的一个曲线。这个蓝色的曲线它的极值点是在这一点处，这个是SK加上DK的值。而这个值显然它是反方向了，我们应该向左走的，它向右走了。

这就是它的二阶导是不是正定的情况所发生的事事情。事实上这个因为是1元的，所以它的二阶岛是一个负数，负数是牛顿方向完全失效了。那如此一来，我们往往就需要把牛顿方向变成一个。不是那么的牛顿方向。

比如说近似的一个牛顿方向，我们把它叫做拟牛顿半牛顿差不多的牛顿。事实上他的。原因是有这么几条。第一个就是我们要去计算这个二阶导的黑色矩阵，它的逆是影响到我们上网效率的。

另外呢我们发现搜索方向可以沿着负梯度方向，也可以沿着牛顿方向。那么我们可不可以沿着既不是梯度方向，也不是牛顿方向的某一个其他方向呢？事实上也是可以的，只要保证这么一个做的方向。

它所确定的那个矩阵是正定的，我们容易计算的就可以了。那么说我们有各种各样的拟牛顿的方案，我们来看一个它的相关的推导。假定我们的目标函数是FF西塔，它的梯度假定是G西塔，它的二阶导，我们记住H西塔。

如果西塔是一个N元的情况，则G西塔是一个向量。而二阶导矩阵这个H是一个N乘N的一个方阵。是黑粉矩阵。我们现在让F西塔在西塔I处做塔刀展开，得到这样一个式子。我们忽略到它更高级的无穷角放在里面去。

那这样子这个等号就变成了一个约等。我们现在仍然用刚才的思路把这么一个式子，方等式的约等式的两边。同时关于西塔求导数L西塔求导数，我们刚才给出定义就是G西塔。这个对西塔求导数是0。

这个对最求导数就是G西塔I。这个对C的求导数是两倍的。HI乘以西塔减西塔I这个二和这个二消掉了，因此就是这个。我们现在既然是任意的一个值。好，我们现在假定西塔是等于西塔A减一的，那么说就得到这样一个等。

这个等式我们把G西塔放在方程左边得到这样一个符号。我们用GI减1减G做G西塔A减1。这样一个值的时候，这有1个HI，我们两边同时对HI求逆就得到这样一个值。我们如果令HI的逆是一个。

N乘N的一个方阵记作CI，然后用德尔塔GI等于GI减去GI减1，这是7度的差值。德尔塔西塔I等于西塔I减去西塔I减1，这是我们参数的差值。那么说我们就得到这样一个非常重要的式子。

CI乘以德塔GI等于德尔塔西塔。然后我们再做一个非常有创造性的一个做法。假定我们当前这一步得到的黑森矩阵逆矩阵，也就是CI是经过加上一个低阶的两个。事实上，这个是一个一阶的。

这是一个一阶的两个小矩阵就能够。近似得到下一个时刻，他的。黑色举证逆举证。而这两个第一阶的逆矩阵，假定是CI是呃VI是某一个列向量，则VI的转制是一个行向量。

所以VI乘以VI的转质就是一个N乘N的一个低解这个阶，甚至于它的质VI乘以VI的转制，它的质只有一这么一个低质的一个矩阵。这个东西我们假定它加起来的时候存在一个系数AI，所以AI是一个标量，是一个数。

然后呢，或许它还需要对另外的一个向量，UI也是一个N行一列的一个。响料。UI也是1个UI的展制，就是一个航向量，所以这仍然是一个N乘N的一个矩阵，乘以某一个系数BI它还是个系数，这样子得到这个事情。

这是一个非常有创造性的想法。他的提出者是这三位数学家。然后呢，我们如果做了这样一个事情之后。🎼CI等于这个式子。好，我们利用刚才这个值就把CI加一这个值带到刚才这个上面这个结论里面去，就得到这样一个值。

让你把德尔塔GI加一都乘过来。然后得到这样一个等式。这个等式里边大家会发现。比如说我们观看这个。这个值其实是一个向一个矩阵乘以一个向量得到的。而我们想去求的是VIUIA和B这样一些系数。

VI既然是一个向量，我们就观察这个等式里边哪些是已经的一个向量呢？比如说da尔塔西塔I加一是参数的变化值，后一步和前一步，它的参数的差值。所以我们不妨就把这个参数的差值给认为是我们要做的那个VI。

如此一来，剩下的是AI这个系数。然后剩下的是VI的转制戴尔塔GI加一，这个值让它等于一。和这一项和最后这个d尔塔斯塔A加一，它们两个是完全相等的。同样道理。我们让UI这个列向量等于这样一个列向量。

如果UI的列向量等，因为CI是1个N乘N的矩阵，然后德尔塔GI加一是一个后一个梯度值减去前一个梯度值，它的一个差值，这是一个德尔塔GI加一是一个列向量，乘完也是一个列向量嘛。

所以我们如果令UI等于这个值的时候，剩下的项。剩下的是BI乘以UI的转制乘以德尔塔GI加一，让这个剩下的值等于-1。那这一个值和这个值也是完全能够消掉的，这样子这个等式就能够完全成立了。

而我们进一步考察，刚才给定这个式子里面VI的转制是个航向量。第尔塔GA加一是一个列向量，它乘完是一个数。所以AI的值可以写成这个数分之1。这样一个道理，同样这个值是个数。

把它写它方程的右边BI得到这样一个数。好了，我们有了这样一些过程之后，大家会发现VIUIAI和BI都能够通过我们已知的值能够把它给表示出来。因此回带到我们的这个式子里面去。

就得到了通过CI去计算CI加一的一个迭代的一个式子。这个就是DFP算法，他去近似计算黑森矩阵的。一举阵的一个方案。事实上，我们最早的CIC0可以用I来去做近似。因为如果C0是一个单位阵的时候。

牛顿迭代这种拟牛顿的迭代方案其实就退化成为了。梯度下降算法，因为一个。AI乘以。GI的时候，CI等于单位阵，它就是一个梯度嘛。所以我们用单位阵做近似是完全可以的。这是利用刚才这样一个式子。

就能够把它写成这么一个python的一个代码，然后放到我们的基术下情算目面去就可以了。另外呢我们如果。仍然是利用这样一个式子。假定CI是乘以这么一个值和它的一个转制。

再加上某1个VI和VI的转制和AI的一个乘积的形式，得到下一个值的。我们还是利用刚才那种计算方式就能得到这样的一个迭代公式。这样解代公式，它的名气更加大，叫BFGS。我们利用它也能够写出相关的一个代码。

这个是BFGS的代码。而不管是刚才我们用的一种方案去做的这个代码。BLP的一个代码，或者是我们给定的BFGS这个代码都能够把它去加入到我们的。梯度下降算法里面去，这个事实上是我利用。

老师的回归现梯度下降，加了一一句话。然后加在这里面就能够把梯度下降算法改造成为你牛顿的方法。然后这是我们的迭代的一个方案。事实上这样一个过程。

右上的这张图是直接使用沿着我们自然函数的正气度方向去做上升去得到的一个分界面。如果我们是用刚才的BFTS的方案得到的分界，它是这样子的。事实上。它的结果并不是完全一样。大家看细节，其实是有一点点的。

不一样的。但是呢用BFGS方案，它的确是收敛速度比。直接使用倾向降要越快一些。我这个数据里边。大概如果是使用我们的稀度下降算法，大约需要10的4次方，也就是1万次迭代完成收敛。

而BFGS方案我给它打出来了，大概只需要呃大概需要。呃，810次左右。到了811次，后面的值就完全收敛了。换句话讲，800多次或者说1000次左右的一个收敛。而。随机而这个老师的回归梯度下降方法。

梯度上升用的是十的4次方。当然，一个数据不能说明什么问题，只是说从感性上大家可以看一下，我们利用了材料展示。深入下去就能够得到我们想要的BFGS。当然了。

我们刚才给定的BFGS是1个N乘N的一个方阵ZK的迭代方式。在实践当中，如果N是比较大的。比如说我们用做自然元处理做图像处理在。CNN在。卷积神经网络出现之前的那么一个阶段，我们这个N是比较大的时候。

那么说我们往往是可以。取前若干次的C，让它直接做题。比如说前10次，当然实间中取2次足以了。那这样一个。本来是一个N乘N的一个矩阵，就能够只用N乘K做近似。比如说我们记做N乘M吧。

这个M如果我们认为是一个线是一个常数的话，那OM乘N就是一个线性的把M看到这常数，这个关于N是一个线性的这样子LBFGSlimit memory在有限内存的这种方案之下。

它就适合于特征比较巨大的寻优问题。好了，这就是我们今天跟大家来探讨的它的展示及其相关的应用。😊，我们发现它的展示它本身是数学分析里面的一个非常重要的工具，在做计计资计算。

在做迭代公式里面有很多很多重要的应用。另外呢T6下降算法，它其实还涉及到了下降方向的一个修正。比如说还涉及到了自适应学习率，如何去设置问题。经济系数呢，刚才我们给定的它是用的商的近似来去做的一个解释。

在实践当中，我们往往用与均匀分布的距离，也就是A的面积除以A的面积加上B的面积，作为经济系数的一个计算方案。这部内容呢，我们在经济学习的课程之中，再给大家来做进一步的探讨和讨论。好了。

如果大家有什么更多的问题呢，非常希望大家在我们的主来EU点com的社区上。😊。

![](img/d232838529150aa119ca23764be133e4_1.png)

发言，然后我们共同探讨。

![](img/d232838529150aa119ca23764be133e4_3.png)