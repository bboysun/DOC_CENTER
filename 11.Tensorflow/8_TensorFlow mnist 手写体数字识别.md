## TensorFlow mnist 手写体数字识别（session2中mnist.py）

#### 一、简介

MNIST是一个入门级的计算机视觉数据集，它包含各种手写数据图片：

![1561888539993](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561888539993.png)

我们这里从一个很简单的数学模型开始，它叫做Softmax Regression。

#### 二、数据集

```Python
from tensorflow.examples.tutorials.mnist import input_data

# reload data for train and test
mnist = input_data.read_data_sets('MNIST_data',one_hot=True)
```

根据此代码可以下载两部分数据集，mnist.train中有60000行的训练数据集和mnist.test中有10000行的测试数据集。每个mnist数据单元有两部分组成，一张包含手写数据的图片和一个对应的标签。我们把这些图片设为xs，把这些标签设为ys。训练数据集合测试数据集都包含xs和ys，比如训练数据集的图片是mnist.train.images，训练数据集的标签是mnist.train.labels。

每一张图片包含28像素*28像素。我们可以用一个数字数组来表示这张图片：

![1561889002382](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561889002382.png)

我们把这个数组展开成一个向量，长度是28*28=784。从这个角度来看，mnist数据集中的图片就是784维向量空间里面的点，并且拥有复杂的结构。

在mnist训练数据集中，mnist.train.images是一个形状为[60000,784]的张量，第一个维度数据用来索引图片，第二个维度数据用来索引每张图片中的像素点。在此张量里的每一个元素，都表示某张图片里的某个像素的强度值，值介于0和1之间。

![1561889331589](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561889331589.png)

相应的mnist数据集的标签介于0到9的数字，用来描述给定的图片里表示的数字。我们使用标签数据是one-hot vectors。一个one-hot向量除了某一位的数字是1以外其余各维度数字都是0。数字n将表示成一个只有在第n维度（从0开始）数字为1到10维向量。比如，标签0将表示成([1,0,0,0,0,0,0,0,0,0,0])。因此，mnist.train.labels是一个[60000,10]的数字矩阵。

![1561889631193](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561889631193.png)

此时，数据集已经准备完成了。

代码中定义如下：

```Python
# define placeholder for inputs to network
xs = tf.placeholder(tf.float32,[None,784])  # 28*28
ys = tf.placeholder(tf.float32,[None,10])
```

#### 三、Softmax回归介绍

从0到9，我们希望得到给定图片代表每个数字的概率。Softmax模型可以用来给不同的对象分配概率。即使在之后，我们训练更精细的模型时，最后一步也需要用Softmax来分配概率。

Softmax回归分两步：

第一步：给定图片某个特定数字类的证据，可以对图片像素值进行加权求和。如果像素具有很强的证据说明这张图片不属于该类，那么相应的权值为负数，相反如果这个像素拥有有利的证据支持这张图片属于这个类，那么权值是正数。例如下面的，红色代表负数权值，蓝色代表正数权值。

![1561980664684](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561980664684.png)

也需要加入一个额外的偏置量（bias），因此对于给定的输入图片x它代表的是数字i的证据可以表示为：

![1561980818968](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561980818968.png)

其中，Wi代表权重，bi代表数字i类的偏置量，j代表给定图片x的像素索引，用于像素求和。然后用Softmax函数可以把这些证据转换成概率y:

![1561980918223](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561980918223.png)

这里的Softmax可以看成是一个激励函数，把我们定义的线性函数的输出转换成我们想要的格式，也就是关于10个数字类的概率分布。因此，给定一张图片，它对于每一个数字的吻合度可以被Softmax函数转换成一个概率值。

![1561981484533](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561981484533.png)

展开等式右边的子式，可以得到：

![1561981833055](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561981833055.png)

更多的时候把Softmax模型函数定义为前一种，把输入值当成幂指数求值，再正则化这些结果值。这个幂运算表示，更大的证据对应更大的假设模型里面的乘数权重值。反之，拥有更少的证据意味着在假设模型里面拥有更小的乘数系数。假设迷行里的权值不可以是0或者负值。Softmax然后会正则化这些权重值，使他们的总和等于1，以此构造一个有效的概率分布。对于Softmax回归模型可以用下面的图解释，对于输入的xs加权求和，再分别加上一个偏置量，最后再输入都Softmax函数中：

![1561982220184](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561982220184.png)

转换成矩阵可以看到：

![1561982263787](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561982263787.png)

也可以用向量表示这个计算过程，矩阵乘法和向量相加。

![1561982320648](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561982320648.png)

再进一步就演变成：

![1561982354662](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1561982354662.png)

代码中：

```Python
def add_layer(inputs, in_size, out_size, activation_function=None):
    # in_size row，out_size column matrix
    with tf.name_scope('layer'):
        with tf.name_scope('weights'):
            Weights = tf.Variable(tf.random_normal([in_size,out_size]), name='W')
        with tf.name_scope('biases'):
            biases = tf.Variable(tf.zeros([1, out_size]) + 0.1, name='b')
        with tf.name_scope('Wx_plus_b'):
            Wx_plus_b = tf.matmul(inputs,Weights) + biases

        if activation_function is None:
            outputs = Wx_plus_b
        else:
            outputs = activation_function(Wx_plus_b)
        return outputs
```

#### 四、训练模型

为了训练好我们的模型，我们需要一个成本的标准，这个成本就用cross_entropy交叉熵来处理。

然后，对cross_entropy通过梯度下降最终得到一个最优的回归模型。代码如下：

```Python
# the error between prediction and real data
cross_entropy = tf.reduce_mean(-tf.reduce_sum(ys * tf.log(prediction),reduction_indices=[1])) # loss
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
```

通过TensorFlow的session来训练出一个合理的回归模型。

```python
sess = tf.Session();
sess.run(tf.initialize_all_variables())
```

#### 五、评估模型

以上，我们都是用mnist训练数据集进行建模，接下来我们要用mnist测试数据集对我们已经建好的模型进行评估，来最终确定我们创建的手写体数字回归模型对手写体数字的识别率到底有多少。

```python
for i in range(1000):
    batch_xs,batch_ys = mnist.train.next_batch(100)
    sess.run(train_step, feed_dict={xs:batch_xs,ys:batch_ys})
    if i%50 == 0:
        print(compute_accuracy(mnist.test.images,mnist.test.labels))
```

#### 六、最终结果

我们这个模型的最终识别准确率达到0.8779，这个准确率其实不算高，后面我们会对这个分类模型做进一步的优化处理。

结果如下图：

![1562157023164](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1562157023164.png)