## TensorFlow初识神经网络

先走一波代码，先睹为快；

```python
import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt

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

### x_data is 300 rows
x_data = np.linspace(-1,1,300)[:,np.newaxis]
###print(x_data)
noise = np.random.normal(0,0.05,x_data.shape)
y_data = np.square(x_data) - 0.5 + noise
#print(y_data)

with tf.name_scope('inputs'):
    xs = tf.placeholder(tf.float32, [None,1],name='x_input')
    ys = tf.placeholder(tf.float32, [None,1],name='y_input')


## start to build neural network
## input layer to hidden layer
layer1 = add_layer(xs, 1, 10, activation_function=tf.nn.relu)
## hidden layer to output layer
prediction = add_layer(layer1, 10, 1, activation_function=None)

with tf.name_scope('loss'):
    loss = tf.reduce_mean(tf.reduce_sum(tf.square(ys - prediction), reduction_indices=[1]))
with tf.name_scope('train'):
    train_step = tf.train.GradientDescentOptimizer(0.1).minimize(loss)

### init all variables
init = tf.initialize_all_variables()
sess = tf.Session()

writer = tf.summary.FileWriter("E:/logs/", sess.graph)

sess.run(init)

#  draw one figure
fig = plt.figure()
ax = fig.add_subplot(1,1,1)
ax.scatter(x_data,y_data)
plt.ion()
plt.show()


for i in range(1000):
    ## training
    sess.run(train_step, feed_dict={xs:x_data, ys:y_data})
    if i % 50 == 0:
        ### to see the step improvement
        ###print(sess.run(loss,feed_dict={xs:x_data,ys:y_data}))
        try:
            ax.lines.remove(lines[0])
        except Exception:
           pass
        prediction_value = sess.run(prediction,feed_dict={xs:x_data})
        lines = ax.plot(x_data,prediction_value,'r',lw=5)
        plt.pause(0.5)
```

再看看效果是怎样的，看看下面：

![](图像结果\Figure_1.png)

![Figure_2](图像结果\Figure_2.png)

![Figure_3](图像结果\Figure_3.png)

![Figure_4](图像结果\Figure_4.png)

我们会看到红色的线会不断的逼近随机生成的点；从而对随机点的数据做正确的预测；后面我们会详细解析上面的代码。

