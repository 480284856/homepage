Let's talk about why neural networks encounter numerical explosions during training, especially when you are implementing a deep learning framework from scratch.

We'll use linear regression as an example, as shown in the figure:

![image](https://raw.githubusercontent.com/480284856/homepage/main/AIFramework/ZZImages/Why%20are%20you%20alwas%20meet%20a%20numerial%20overflow-1.png)

This example shows the first two linear layers of a neural network. The first layer has two input neurons and two output neurons; the second layer is the output layer, with two input neurons and one output neuron. Here, the squares represent logits, and the small circles represent activation values.

We assume a batch size of 4, and the derivatives of the loss function with respect to the output logits of the neural network are 121, 101, 74, and 47 — this is a 4×1 two-dimensional matrix. The parameters of the output layer form a 1×2 matrix with values -2 and 0.1. These numbers are real values taken from an actual project:

![immage](./ZZImages/Why%20are%20you%20alwas%20meet%20a%20numerial%20overflow-2.png)

Now we compute a delta (δ), i.e., the partial derivative of the loss function with respect to the input of the output layer:
1. It equals the partial derivative of the loss with respect to the network's output, multiplied by the corresponding parameter of the output layer's input.
2. For the first input neuron(the upper circle in the second layer), it equals the partial derivative multiplied by -2. This gives a 4×1 vector: -242, -202, -148, -94.
   
   From this, we can see that if the parameter is large, the delta essentially doubles — going from 121 to -242.
3. Similarly, for the second input of the output layer, the corresponding delta is the previous partial derivative multiplied by 0.1. This gives a 4×1 vector: 12.1, 10.1, 7.4, and 4.7.

At this point, the partial derivative of the loss with respect to the output layer's input is a 4×2 matrix.

Then we continue backpropagation further:
1. Assume the activation function is ReLU, and all neurons were in the active state during forward propagation. In this case, the delta corresponding to the activation function's input is also that 4×2 matrix.
2. We then compute the delta for the input of the first linear layer. Assume the four parameters are -2, -1, -0.1, and 2.
3. For the upper input of the first linear layer, the partial derivative of the loss equals:
   
   (a) All partial derivatives corresponding to the outputs of the previous layer, multiplied by the corresponding parameters from that neuron pointing to all outputs.

   (b) In other words, each column of the 4×2 matrix is multiplied by each row of the parameter matrix W, and then summed across columns.

At this point, we can see that the already-doubled -242 has doubled again. Comparing the numbers from the partial derivative at the output layer's input to the ones at the previous layer's input, we can observe that after just walking through one layer, the delta has doubled. If the network is deeper, it is obvious that the delta will grow to an astronomical number. This causes the absolute values of the parameters to become extremely large when updating the network's weights. 

Based on the analysis above, we can identify two main causes for numerical overflow:

1. The initial delta data is too large.
2. The initial parameters of the neural network are too large.

![image-3](./ZZImages/Why%20are%20you%20alwas%20meet%20a%20numerial%20overflow-3.png)

To address the first issue, in this specific example, the delta is large because the label values are very high. We can resolve this by applying normalization to those labels.

Regarding the second issue of excessive parameter size, we can address this during initialization by multiplying the parameters by a small factor, such as 0.01 or 0.1, to reduce their scale.

![image-4](./ZZImages/Why%20are%20you%20alwas%20meet%20a%20numerial%20overflow-4.png)