## TensorFlow综述

#### 一、简介

TensorFlow是一个编程系统，使用图来表示计算任务。图中的节点被称之为op（operation的缩写）。一个op获得0个或者多个Tensor，执行计算，产生0个或者多个Tensor。每个Tensor是一个类型化的多维数组。例如，你可以将一小组图像集表示为一个四维浮点数数组，这四个维度分别是[batch, height, width, channels]。一个TensorFlow图描述了计算的过程。为了进行计算，图必须在会话里面被启动。会话将图的op分发到诸如CPU或者GPU之类的设备上，同时提供执行op的方法。这些方法执行后，将产生的Tensor返回。在Python语言中，返回的Tensor是numpy ndarray对象。

#### 二、计算图

TensorFlow程序通常被组织成一个构建阶段和一个执行阶段。在构建阶段，op的执行步骤被描述成一个图。在执行阶段，使用会话执行图中的op。

举个栗子，通常在构建阶段创建一个图来表示需要训练的神经网络，然后在执行阶段反复执行图中的训练节点op。

#### 三、构建图

创建源op，源op不需要任何输入，例如，常量（constant），源op的输出被传递给其他op做运算。op构造器的返回值作为被构造出的op的输出，该返回值可以作为输入传递给其他op的构造器。TensorFlow Python库有默认图，op构造器可以为其增加节点。这个默认图对许多程序来说已经足够用了。

`import tensorflow as tf`

这句话就是导入TensorFlow Python库中的默认图。

详见下面的代码及注释

```python
import tensorflow as tf

# 创建一个常量 op, 产生一个 1x2 矩阵. 这个 op 被作为一个节点
# 加到默认图中.
#
# 构造器的返回值代表该常量 op 的返回值.
matrix1 = tf.constant([[3., 3.]])

# 创建另外一个常量 op, 产生一个 2x1 矩阵.
matrix2 = tf.constant([[2.],[2.]])

# 创建一个矩阵乘法 matmul op , 把 'matrix1' 和 'matrix2' 作为输入.
# 返回值 'product' 代表矩阵乘法的结果.
product = tf.matmul(matrix1, matrix2)
```

上面的代码表示，默认图中现在添加了三个op，两个constant常量op，一个matmul矩阵乘法操作op。

为了真正进行矩阵相乘运算，并得到矩阵乘法的结果，就接着看下面的了~~~

#### 四、在会话中启动图

以上构造阶段完成，才能启动图。启动图的第一步就是创建一个session对象。如果无任何创建参数，会话构造器将启动默认图。

```python
# 启动默认图.
sess = tf.Session()

# 调用 sess 的 'run()' 方法来执行矩阵乘法 op, 传入 'product' 作为该方法的参数. 
# 上面提到, 'product' 代表了矩阵乘法 op 的输出, 传入它是向方法表明, 我们希望取回
# 矩阵乘法 op 的输出.
#
# 整个执行过程是自动化的, 会话负责传递 op 所需的全部输入. op 通常是并发执行的.
# 
# 函数调用 'run(product)' 触发了图中三个 op (两个常量 op 和一个矩阵乘法 op) 的执行.
#
# 返回值 'result' 是一个 numpy `ndarray` 对象.
result = sess.run(product)
print result
# ==> [[ 12.]]

# 任务完成, 关闭会话.
sess.close()
```

Session对象在使用完后需要关闭以释放资源。除了显示调用close外，也可以用with代码块来自动完成关闭操作。

```python
with tf.Session() as sess:
  result = sess.run([product])
  print result
```

#### 五、Tensor

TensorFlow程序使用Tensor数据结构来代表所有的数据，计算图中，操作间传递的数据都是Tensor。可以把Tensor看做是一个n维的数组或者列表。后续都会详细介绍。

#### 六、变量

变量维护图执行过程中的状态信息，variables for more details。详见下面的栗子，用变量实现一个简单的计数器。

```python
# 创建一个变量, 初始化为标量 0.
state = tf.Variable(0, name="counter")

# 创建一个 op, 其作用是使 state 增加 1

one = tf.constant(1)
new_value = tf.add(state, one)
update = tf.assign(state, new_value)

# 启动图后, 变量必须先经过`初始化` (init) op 初始化,
# 首先必须增加一个`初始化` op 到图中.
init_op = tf.initialize_all_variables()

# 启动图, 运行 op
with tf.Session() as sess:
  # 运行 'init' op
  sess.run(init_op)
  # 打印 'state' 的初始值
  print sess.run(state)
  # 运行 op, 更新 'state', 并打印 'state'
  for _ in range(3):
    sess.run(update)
    print sess.run(state)

# 输出:

# 0
# 1
# 2
# 3
```

代码中assign op是图所描绘的表达式的一部分，正如，add op一样。所以调用run执行表达式之前，他并不会执行赋值操作。

通常，将一个统计模型中的参数表示为一组变量。例如，可以将一个神经网络的权重作为某个变量存储在一个Tensor中。在训练过程中，通过重复运行训练图，更新这个Tensor。

#### 七、Fetch

为了取回操作的输出内容，可以在使用session对象的run方法执行图时，传入一些Tensor，这些Tensor会帮你取回结果。在上面的栗子，我们只需取回了单个节点的state，同时我们也可以取回多个组成一个Tensor。详见代码如下

```python
input1 = tf.constant(3.0)
input2 = tf.constant(2.0)
input3 = tf.constant(5.0)
intermed = tf.add(input2, input3)
mul = tf.mul(input1, intermed)

with tf.Session():
  result = sess.run([mul, intermed])
  print result

# 输出:
# [array([ 21.], dtype=float32), array([ 7.], dtype=float32)]
```

#### 八、Feed

上述示例在计算图中引入了Tensor，以常量或者变量的形式存储。TensorFlow还提供了feed机制，该机制可以临时替代图中的任意操作中的Tensor，可以对图中任何操作提交补丁，直接插入一个Tensor。

Feed使用一个Tensor值临时替换一个操作的输出结果。可以提供Feed数据作为run方法的参数。Feed的作用域只在调用它的方法内有效，方法结束，Feed就会消失。最常见的栗子是将某些特殊的操作指定为Feed操作，标记的方法是使用`tf.placeholder()`为这些操作创建占位符。

```python
input1 = tf.placeholder(tf.types.float32)
input2 = tf.placeholder(tf.types.float32)
output = tf.mul(input1, input2)

with tf.Session() as sess:
  print sess.run([output], feed_dict={input1:[7.], input2:[2.]})

# 输出:
# [array([ 14.], dtype=float32)]
```



以上就是TensorFlow的简单入门介绍。

参考：http://www.tensorfly.cn/tfdoc/get_started/basic_usage.html