# Pytorch Library Loss Functions

Pytorch implementations for `Focal Loss`, `Center Loss`, `Island Loss`

## Focal Loss
Implementation from paper:  
[Focal Loss for Dense Object Detection](http://arxiv.org/abs/1708.02002) (TsungYi Lin, Priya Goyal, Ross B. Girshick, Kaiming He, Piotr Doll)

Focal Loss helps to reduce the problem of class imbalance by adding a focal term to the cross entropy loss. It is defined as:

$$L_{F} = -\frac{1}{N}\sum_{i=1}^{N} \alpha_j \cdot (1-p_{i,y_i})^{\gamma} \cdot \log(p_{i,y_i})$$

where $N$ is the number of samples in the minibatch, $p_{i, y_i}$ is the probability predicted for the sample $i$ to belong to ground truth class $y_i$, $\alpha_j$ is the weight of the class $j$, and $\gamma$ is the focusing parameter.  
Difficult samples will be associated by the network with a low $p_{i,y_i}$, and the loss will be higher for them because of focal term $(1- p_{i,y_i})^\gamma$. The higher $\gamma$ is, the more the loss will be focused on difficult samples. Common values for $\gamma$ are 1.5, 2. Common class weights $\alpha$ computation is in function `compute_class_weights`.

## Center Loss
Implementation from paper:  
[A Discriminative Feature Learning Approach for Deep Face Recognition](https://api.semanticscholar.org/CorpusID:4711865) (Yandong Wen, Kaipeng Zhang, Zhifeng Li, Yu Qiao)

Center Loss encourages the network to learn a compact representation of the data, which is helpful for datasets having high intra-class variability and high inter-class similarity, meaning that features for samples belonging to same class tend to be very spread in the feature space and features of samples belonging to different class tend to overlap. Intraclass-compactness is achieved by minimizing the distance between the output of the network and the center of the corresponding class. It is defined as:

$$L_{C} = \frac{1}{N} \sum_{i=1}^{N} \left\lVert x_i - c_{y_{i}} \right\rVert_{2}$$

Where $c_{y_{i}}$ is the class center of the correct class $y_i$ for sample ${x}_i$.

Ideally, the class centers should be learnt by computing the mean of the deep features produced at each step for all the samples of the same class in the training set. However, this would be inefficient and impractical. So, class centers are actually updated at each mini-batch iteration by averaging the deep features of the samples in the mini-batch. This may introduce large perturbations in the learning of the centers (for example, a mini-batch could contain only samples from a single class with a mean very different from the global mean). To avoid this, class centers are learnt using an SGD optimizer with fixed learning rate $\alpha \in [0,1]$.

So an actual center update is computed at each mini-batch through the following SGD update rule:

$$c_{j}^{t+1} = c_{j}^{t} - \alpha \cdot dc_{j}^{t}$$

Where $d{c}_{j}^{t}$ is the gradient of the center loss with respect to the class center $c_j$.

The CE loss encourages features separability, reducing the inter-class similarity, but doesn't act on the discriminative power of the features. Therefore, center loss is used along with the standard CE loss:

$$L = {L}_{CrossEntropy} + \lambda {L}_C$$

where $\lambda$ is a hyperparameter that balances the two loss functions. Intuitively, the CE loss forces the deep features of different classes staying apart while the center loss efficiently pulls the deep features of the same class to their centers.

## Island Loss
Implementation from paper:  
[Island Loss for Learning Discriminative Features in Facial Expression Recognition](http://arxiv.org/abs/1710.03144) (Jie Cai, Zibo Meng, Ahmed Shehab Khan, Zhiyuan Li, James O'Reilly, Yan Tong)

Island Loss improves the center loss to produce features that are not only compact (for samples in same class), but also separable. It is computed as:

$$L_I = \sum_{c_j}^{K} \sum_{c_k \neq c_j}^{K} \left(\frac{c_j \cdot c_k}{\left\lVert c_k \right\rVert_{2} - \left\lVert c_k \right\rVert_{2}} + 1\right)$$

Where ${c}_j$ and ${c}_k$ are the class centers of class $j$ and $k$ respectively. The +1 term is necessary to make the loss non-negative, since the cosine exists in $[-1,+1]$ range. Intuitively, we are minimizing the cosine similarities between the class centers which encourages the features of different classes to be more separable in the feature space.

The combined loss will be therefore the sum of Island Loss, center loss and CE loss:

$$L = L_{CrossEntropy} + \lambda_{global} (L_C + \lambda_{island} L_I)$$

Where $\lambda_{global}$ and $\lambda_{island}$ are hyperparameters that balance the three loss functions. 

The updating strategy is the SGD, exactly as for Center Loss.

## Example
Figure shows a training of 10 epochs of an `nn.EfficientNet_b0` over Cald3R&MenD3s dataset ([CalD3r and MenD3s: Spontaneous 3D facial expression databases](https://www.sciencedirect.com/science/article/pii/S1047320323002833)) for Facial Expression Recognition (FER).


Note how Center Loss helps in producing clustered features that are more discriminative than the ones produce by same network trained with Cross Entropy, especially for Neutral class which is very spread. Furthermore, Island Loss pushes clusters away from each other, producing even more discriminative features.

<p align="center">
  <img src="Images/cross.png" alt="CrossEntropy Loss" width="450"/>
  <br>
  <span>CrossEntropy Loss  - Accuracy:  58.83</span>
</p>
<p align="center">
  <img src="Images/center.png" alt="Center Loss" width="450"/>
  <br>
  <span>Center Loss  - Accuracy:  63.46</span>
</p>
<p align="center">
  <img src="Images/island.png" alt="Island Loss" width="450"/>
  <br>
  <span>Island Loss - Accuracy:  64.85</span>
</p>


## Acknowledgments

- [Pytorch](https://pytorch.org/)
- [Island Loss Paper](http://arxiv.org/abs/1710.03144)
- [Focal Loss Paper](http://arxiv.org/abs/1708.02002)
- [Center Loss Paper](https://api.semanticscholar.org/CorpusID:4711865)
- [Cald3R and MEnD3s](https://www.sciencedirect.com/science/article/pii/S1047320323002833)