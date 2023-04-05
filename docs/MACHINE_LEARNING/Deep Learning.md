# Deep Learning

### Different types of n

1. Regression: the func outputs a scalar
2. Classfication: given options(classes), the func outputs the correct one.
3. ...
4. Structure Learning: create something with structure (images, document...) 让机器创造某种东西，这是一件不简单的事情。

### How to find the function we need

1. Function with unknow parameters.
    + y = f(  )
    + We can suppose a func based on domain data.  e.g. `y = x * w + b`
    + `x` is known data, called **Feature**
    + In ML, we call this `func y` **Model**
    + `w`, which multiplies with x is called **weight**
    + b, which adds `x` is called **bias**
2. Define Loss from training data.
    	+ **Loss**: is a function of parameters L(w, b). It means how good a set of values is.
    	+ **Label**: the true value. (Compare to predicted values)
     + 两种常见的计算误差的方法：
        + **L is mean absolute error (MAE)**:  $e=|y-\hat{y}| $
        + **L is mean square error (MSE)**：$e=(y-\hat{y})^{2}$
        + 在有一些例子中，y 和 $\hat{y}$ 都是概率，会选择Cross-entropy。（挖坑）
     + **Error Surface**: 根据穷举参数得到Loss，描绘出等高线图。(Loss 越小越好)
3. Optimization
    + 最佳化的一种方法是：Gradient Descent
        1. (Randomly)Pick an initial value $w^0$ 
        2. Compute $\left.\frac{\partial L}{\partial w}\right|_{w=w^{0}}$ 
            + ![截屏2022-09-19 上午10.53.44](/Users/viobert/myBlog/self_impro/docs/MACHINE_LEARNING/images/Gradient Descent.png)
            + 计算$w_0$的斜率，为正 positive 就说明此处的L的向着w减小的地方减小；为负Negative相反。然后我们总是希望L减小，所以需要向使得L变小的地方“跨一步”
            + 决定跨一步的因素有两个：
                1. $\left.\frac{\partial L}{\partial w}\right|_{w=w^{0}}$ 斜率
                2. $\eta$ : **learning rate** 由自己来设定。在ML中自己设定的参数，叫做**hyperparameters**。
            + $w^{1} \leftarrow w^{0}-\left.\eta \frac{\partial L}{\partial w}\right|_{w=w^{0}}$
        3. Update `w` iteratively.
4. 

