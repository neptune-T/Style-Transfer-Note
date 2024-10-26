


# IOB-NST（基于图像优化的神经风格迁移方法）

## 1. A Neural Algorithm of Artistic Style (2015)

   - **作者**：Leon A. Gatys等人  
   - **贡献**：这是风格迁移领域的奠基性论文，首次提出利用卷积神经网络（CNN）实现内容与风格分离。通过格拉姆矩阵捕捉风格特征，并通过优化生成与风格图像的匹配，生成图像既保留了内容的结构，又具有目标艺术风格。  
   - **核心思想**：通过CNN层次的特征提取，将风格信息与内容信息结合。  

最初是npr领域,

- **Stroke-Based Rendering (SBR) - 基于笔触的渲染**

 SBR在放置笔触时通常依赖于某种**优化算法**来逐步调整每个笔触的位置、角度、颜色和大小，以达到匹配原始图像的效果。一个常见的优化目标是最小化源图像与渲染图像之间的差异。

**目标函数**：SBR中的目标函数用于衡量渲染图像和源图像之间的相似度，可以使用**均方误差（MSE）**等来定义目标函数。假设 \( I_{\text{source}} \) 是源图像，\( I_{\text{rendered}} \) 是渲染的图像，则目标函数可以表示为：
     $$
     L = \sum_{i,j} \|I_{\text{source}}(i,j) - I_{\text{rendered}}(i,j)\|^2
    $$
     $$
     G_x = \frac{\partial I}{\partial x}, \quad G_y = \frac{\partial I}{\partial y}
     \]
     其中，\( G_x \) 和 \( G_y \) 分别表示水平方向和垂直方向的梯度。梯度大小：
     \[
     G = \sqrt{G_x^2 + G_y^2}
     $$
     
   - 通过提取这些高梯度的区域，SBR可以将笔触放置在图像中最需要细节表现的部分。


   - 为了产生多层次的笔触效果，SBR可能使用图像分层技术，通过分解图像的不同特征（如高频和低频成分）来放置不同的笔触类型。图像的高频部分用于描绘细节（如边缘），而低频部分则用于平滑的背景区域。
   - **高斯模糊**可以用于提取低频信息，而**拉普拉斯算子**可以用于提取高频细节：
     $$
     L(I) = \nabla^2 I
     $$
     



#### 2.1 **图像分割**
   - 基于区域的渲染需要将图像分割为多个区域，每个区域可以使用不同的渲染风格。图像分割的常用方法包括**阈值分割、区域增长、K均值聚类**等。通过这些方法，将图像分割为具有不同特征的区域。
   - **K均值聚类**是一种常用的分割方法，它将像素根据颜色或纹理分为多个聚类：
     $$
     J = \sum_{i=1}^{k} \sum_{x \in C_i} \|x - \mu_i\|^2
     $$
     其中，\( C_i \) 是第 \(i\) 个聚类，\( \mu_i \) 是该聚类的均值。通过最小化损失函数 \( J \)，找到最佳的区域划分。
   - 区域分割后，每个区域可以应用不同的渲染规则，产生风格化的效果。

#### 2.2 **区域匹配与几何变换**
   - 基于区域的渲染中，某些算法会通过替换区域中的形状或进行几何变换来实现风格化效果。例如，将图像中的物体替换为几何图形，如三角形、矩形等。这种替换通过形状匹配算法实现，常见方法包括**Procrustes分析**，它用于最小化形状之间的距离：
     $$
     d(X, Y) = \min_{R, s, t} \|sRX + t - Y\|
     $$
     其中，\( R \) 是旋转矩阵，\( s \) 是缩放因子，\( t \) 是平移向量，\( X \) 和 \( Y \) 是两个形状，目标是找到最佳的形变来匹配这些区域。

#### 2.3 **区域着色与纹理合成**
   - 对分割后的区域应用不同的渲染风格可能涉及纹理合成和着色。使用**傅里叶变换**或**小波变换**可以为不同区域生成合适的纹理效果：
     $$
     F(u,v) = \sum_{x=0}^{M-1} \sum_{y=0}^{N-1} I(x,y) e^{-j2\pi\left(\frac{ux}{M} + \frac{vy}{N}\right)}
     $$
     通过频域分析和重构生成不同的纹理风格。




1. **总损失函数**：
   - 目标是最小化总损失函数 \( L_{\text{total}} \)，从而找到最终生成的风格化图像 \( I^* \)：
     $$
     I^* = \arg \min_{I} L_{\text{total}}(I_c, I_s, I) = \arg \min_{I} \alpha L_{\text{C}}(I_c, I) + \beta L_{\text{S}}(I_s, I)
     $$
     这里，\( \alpha \) 和 \( \beta \) 是用于平衡内容和风格的重要性权重，\( I_c \) 是内容图像，\( I_s \) 是风格图像，\( I \) 是当前生成的图像。

2. **内容损失 \( L_{\text{C}} \)**：
   - 内容损失比较内容图像 \( I_c \) 和生成图像 \( I \) 在某些CNN层上的特征表示（通常使用预训练的VGG网络）。
   - 数学表达式为：
     $$
     L_{\text{C}} = \sum_{l \in \{l_c\}} \| F^l(I_c) - F^l(I) \|^2
     $$
     这里，\( F^l \) 是在第 \( l \) 层提取的图像特征，\( \{l_c\} \) 表示用于计算内容损失的VGG层集合。内容损失衡量的是在特定CNN层上，内容图像和生成图像之间的特征差异。

3. **风格损失 \( L_{\text{S}} \)**：
   - 风格损失则使用**Gram矩阵**来比较风格图像 \( I_s \) 和生成图像 \( I \) 的风格表示。Gram矩阵计算的是某层CNN输出特征之间的相互关联，能够捕捉图像的纹理和风格特征。
   - 数学表达式为：
     $$
     L_{\text{S}} = \sum_{l \in \{l_s\}} \| G(F^l(I_s)) - G(F^l(I)) \|^2
     $$
     其中，\( G(F^l(I)) \) 是图像在第 \( l \) 层的Gram矩阵表示，\( \{l_s\} \) 表示用于计算风格损失的层集合。通过最小化Gram矩阵的差异，生成图像会逐步学习并匹配风格图像的纹理。


- **内容损失 \( L_{\text{C}} \)** 保证生成图像保持内容图像的结构和形状。
- **风格损失 \( L_{\text{S}} \)** 则保证生成图像在纹理和颜色分布上匹配风格图像。
- 通过调节 \( \alpha \) 和 \( \beta \)，可以控制风格和内容在生成图像中的占比。



## 2. Neural Style Transfer: A Review (2017)
   - **作者**：综述论文  
   - **贡献**：总结了神经风格迁移（Neural Style Transfer, NST）的发展历程，讨论了不同的纹理建模方法以及如何通过CNN实现风格和内容的分离。文中回顾了早期的纹理建模方法，如基于马尔可夫随机场（MRF）等模型。  
   - **核心思想**：通过对NST技术的演变进行综述，为未来的研究提供了指导。  


### 3. **Artistic Style Transfer with Internal-external Learning (2021)**  
   - **作者**：多位研究者  
   - **贡献**：提出了一种基于对比学习的风格迁移方法，结合内部风格（从单一艺术品中学习）和外部风格（通过GAN学习人类感知的风格信息），生成更加逼真的艺术风格图像。  
   - **核心思想**：使用对比学习与GAN，结合内外风格信息进行风格迁移。  


### 4. **Style Injection in Diffusion: A Training-free Approach for Adapting Large-scale Diffusion Models for Style Transfer (2023)**  
   - **作者**：多位研究者  
   - **贡献**：提出了一种基于扩散模型的无训练风格迁移方法。该方法利用Stable Diffusion中的自注意力机制，在无需复杂优化的情况下，将风格和内容特征进行有效融合，从而生成更自然的风格图像。  
   - **核心思想**：通过扩散模型的自注意力机制实现内容与风格的无缝融合，减少训练复杂度。  


### 5. **Towards Highly Realistic Artistic Style Transfer via Stable Diffusion (2023)**  
   - **作者**：多位研究者  
   - **贡献**：基于Stable Diffusion的风格迁移，提出了一种分层和步骤感知的提示机制，能够生成高度真实的风格化图像。该方法有效解决了风格迁移过程中图像失真或不协调的问题。  
   - **核心思想**：利用Stable Diffusion和提示机制，提高风格迁移的真实性。  



# IOB-NST（基于图像优化的神经风格迁移方法）