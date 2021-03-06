# AlgoMorphism

<img src="pictures/am_logo.png" width="250" height="100%">

## Description & Back Propagation Experiment

AlgoMorphism (AM) is a general driven framework for Object-Oriented Programing (OOP) on Back-Propagation Optimization 
algorithm. Defining an optimization experiment using objects. Object examples could be Dataset Object (DO) or input of 
experiment and Model Object (MO) which is optimized.

Firstly at an experiment call or develop a custom Dataset Object (DO), every DO has 1 to 3 elements:
- rain element: train batched examples,
- val element (optional): validation batched examples,
- test element (optional): test batched examples.

Secondly, call or develop custom Cost Objects, Score Object:
- Cost-Loss Object: this object applies forward propagation and keeps in memory the computation chain of cost function.
- Cost-Metric Object: this object applies forward propagation and return the cost.
- Score Object (optional): this object is computed at most cases in classification experiments. Same examples is:
  - Accuracy Score
  - F1 Score

Thirdly, call or develop custom Model Object (MO), every has one required element:
- status list: this list contains what the MO reads as input/output by DO.

Also MO has 2 to 3 elements:
- a list of Cost-Loss Objects,
- a list of Cost-Metric Objects,
- a list of Score Objects (optional)

An MO Inherits the Base neural network Object (BO) with the status and DO as required elements. With BO, MO is becomes a trainable.

Finished the training, the experiment has completed. Plot the results per iteration (aka history), calling the `multiple_models_history_figure` function to plot Cost-Metric, Score per iteration (aka epoch).


## Learning Graphs Example
In this learning experiment we try to classify using Graph Constitutional Networks, a random generated types of graphs with different number of nodes. We call the DO by framework, secondly we illustrative the MO graph classification learning and train the model. After the training we plot the leaning results.

### Configure and call the generated \`Simple Graph Dataset\`

```python
import algomorphism as am
import tensorflow as tf

examples = 500
n_nodes_min = 10
n_nodes_max = 20
graph_types = ['cycle', 'star', 'wheel', 'complete', 'lollipop',
                'hypercube', 'circular_ladder', 'grid']

n_c = len(graph_types)
g_dataset = am.dataset.datasets.generate.SimpleGraphsDataset(examples, n_nodes_min, n_nodes_max, graph_types)
a_train = g_dataset.get_train_data()
a_dim = a_train.shape[1]
```

### Illustrate a custom \`Graph Convolutional Classifier\` MO
```python
class GraphConvolutionalClassifier(tf.Module, am.model.base.BaseNeuralNetwork):
  def __init__(self):
    tf.Module.__init__(self, name='gcn_classifer')
    status = [
      [0],
      [1, 2]
    ]
    self.score_mtr = am.model.base.MetricBase(self,
      [tf.keras.metrics.CategoricalAccuracy()],
      status=status,
      mtr_select=[0]
    )

    self.cost_mtr = am.model.base.MetricBase(self,
      [tf.keras.metrics.CategoricalCrossentropy()],
      status=status,
      mtr_select=[0],
      status_out_type=1
    )
    
    self.cost_loss = am.model.base.LossBase(self,
      [tf.keras.losses.CategoricalCrossentropy()],
      status=status,
      loss_select=[0]
    )
    
    am.model.base.BaseNeuralNetwork(status, g_dataset)
    
    self.gcn1 = am.model.layers.GCN(a_dim, 32)
    self.gcn2 = am.model.layers.GCN(32, 64)
    
    self.flatten = tf.keras.layers.Flatten()
    self.out = am.model.FC(a_dim*64, n_c, 'softmax')

  def __call__(self, inputs):
    x = self.gcn1(inputs[0], inputs[1])
    x = self.gcn2(x, inputs[1])
    x = self.flatten(x)
    
    y = self.out(x)
    y = tuple([y])
    return y
```

### Train MO
```python
gcn = GraphConvolutionalClassifier()
gcn.train(g_dataset, epochs=150, print_types=['train', 'val'])
```

### Plot History
```python
am.figure.opt.multiple_models_history_figure([gcn])
```