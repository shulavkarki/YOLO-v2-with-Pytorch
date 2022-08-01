## YOLO-v1-with-Pytorch

### Overall Structure of YOLO v1 Network:  
![image](https://user-images.githubusercontent.com/40908371/182013335-922f1e30-b747-4851-960d-741a1ae64236.png)    
1. Use Convolution Network as feature extraction.  
2. Use fully connected layers to predict output probabilites and coordinates. 
  
  --------------------------------------------------------------------------------------------------
 ##### Let's break each one of em.  
 
#### 1. Convolution Netork
In the [yolo](https://arxiv.org/abs/1506.02640) paper, the author pretrain on 20 convolution layers on the ImageNet dataset.    
Here the convolution layers is used as feature extraction. 

Obviously, i'm not gonna pretain in the imagenet. coz, it gonna take up to lot of time to train(may be weeks).
Nonetheless, one can use any pretrained cnn network like resnet, or others mentioned in paper.

However.., here i've used custom architecture (Archi.config in repo) to train image classifier in custom dataset (pizza vs sandwich dataset.)  
If you've noticed the architecture, i've used the higher convolution followed by lower number of channels convolution, because it reduces the amount of computation and improves the non-linearity of the model. 
Result of Image Classifier:    
|  Optimizer | Epoch | Learning Rate| Training Accuracy | Testing Accuracy |
| --- | --- | --- | --- | --- |
|     SGD Gradient Descent       |  50  | 0.0001 | 93.50% | 91.41%  
  
The trained model is in ```./Saved Models/``` folder. You can pretrain CNN Network in your own dataset.  
For training and testing , you can find in  ```classifer.ipynb``` file within  the repo.  
Also, you can checkout this [repo](https://github.com/shulavkarki/Image-Classification-in-Custom-dataset).  
  
#### 3. FC Layers for prediction
Yolov1 frame object detection as a regression problem to spatially seperated bounding box and associated class probabilites.  
For the last convolution layer, it outputs a tensor shapeed (7, 7, 1024). Then the tensor expands using 2 FC layers as a form of linear regression. It outputs parameters and then reshapes into (7, 7, 30).  

## Now let's look at the workflow or how the model gets trained.

1. In the paper, the image is divided into S*S grid(virtually). The author has taken S = 7.
 ![image](https://user-images.githubusercontent.com/40908371/178311634-c970f0d6-1e09-486c-b2f8-4851961dae5e.png)
  
2.The output is S*S*(5B+C).  
Since, S=7. The image is alltogether divided into 7*7 grid.  
So, for each grid, the size of output is **5B+C**.  
Terms:  
B = Bounding Box  
C = Proababilies of each class  
If we consider B=1, and C = 'n' class., then for each grid 1 bouding box is predicted. It looks something like this.
![image](https://user-images.githubusercontent.com/40908371/178313510-cfe4ca18-4cdc-448b-ba64-c6053bb528f5.png)
If we consider B=2 then,
![image](https://user-images.githubusercontent.com/40908371/178315220-fb3e1b2a-e2cd-4b66-920e-f3528e3c5467.png)
This means, each grid is going to predict 2 bounding box which is defined by (x, y, w, h)which are center, width and height of bounding box.   
Therefore, the output is the flatten of size S**S*(5B+C).
The 30 in the fully connected layer is the (5B+C), where the author considers B=2 and C=20 classes(can predict upto 20 classes)

## Loss Function
[source](https://www.linkedin.com/feed/update/urn:li:activity:6929243398876909568/)
<!-- ![image](https://user-images.githubusercontent.com/40908371/178316932-40efa075-68d2-4027-b9df-f7848650bec5.png) -->

## Implementation 
However in this repo, i've ued yolov3.  
Unlike in the yolov1 where the final year consists of the regressor, here CNN is used in the final layer.  
### Consideration:  
- Classifier is used as Feature Selection/Feature Extraction.
- Extra layer is added to the classifier to get the output CNN.
- The output CNN should be in the dimension of S*S*C. where S: Grid size, and C= Channel.
- Here the S=13. Meaning the image is divided into 13 by 13 grid and the output consists of 13*13 height and weight and C channle.
- Here the C=7. Meaning the ouput will have 7 channel. [1st chnnel:Confidence Score, 2nd to 5th channel: x, y, w, h and 6th to 7th channel consists of probability score of given object falls in particular class.]  
- Here (x, y): Center of the bounding box. (w, h): Dimension of bouding box.  
- Since, here i've considered only two class. So, only two channel after x,y,w and h.  
- 

 

### Architecture:  
  
1. Classifier Netwwork    
![image](https://user-images.githubusercontent.com/40908371/181306245-654a3270-3e28-4441-8b9a-ce098ead4afe.png)
  
2. Object Detection Network  
![image](https://user-images.githubusercontent.com/40908371/181923530-5e035567-4e1e-4c3e-8478-3556240e7d57.png)

Actual Volume Interpretation:  
![image](https://user-images.githubusercontent.com/40908371/181935037-92292f52-13db-4055-a98e-7da5c779d5ec.png)

### Loss Function
The Yolov1 loss function is used in this implementation.  

#### Yolo Loss Function looks something like this:  
  
![image](https://user-images.githubusercontent.com/40908371/182013294-37031ca7-dea1-40f3-a864-6b7e6beb580c.png)  
  
Let's break each of em. 
  
1. Bounding Box Coordinate Loss/ Regression Loss.  
  
![image](https://user-images.githubusercontent.com/40908371/182013421-fdbb1f8e-fa17-4440-9368-e2027f92f6a3.png)  
  
  
<img src="https://user-images.githubusercontent.com/40908371/182176740-214aab10-0ada-4bb4-a440-72126873527a.png" alt="1obj" width=500/>
     
     
2. Confidence Loss  
  
![image](https://user-images.githubusercontent.com/40908371/182013462-7b5049fa-ba5f-4947-9a82-3d205c4bcdac.png)  
  



3. Classificaiton Loss  
  
![image](https://user-images.githubusercontent.com/40908371/182013451-351d3013-b9ea-4359-bd36-ce492a10a0d4.png)   
  
### Limitations of Yolo  
  
- Comparatively low recall and more localization error compared to Faster R_CNN.  
- Struggles to detect close objects because each grid can propose only 2 bounding boxes.  
- Struggles to detect small objects.  
