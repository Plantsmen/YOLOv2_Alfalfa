# 训练YOLOv2用于检测苜蓿花

注：①图像不需要统一大小②只可以检测单目标物体

## 1.1 导入数据

Load the training data for vehicle detection into the workspace.

```matlab
clear;clc;
% 第一次使用需要自己进行标注数据，并把它保存为gTruth，否则会出错。
load('gTruth.mat');                               %标记的gTruth
trainingfiles = gTruth.DataSource.Source;         %图像位置
traininglabel = gTruth.LabelData;                 %图像标签
```

Create an imageDatastore using the files from the table.

```matlab
imagesize = [227 277 3] ;
imgfiles = imageDatastore(trainingfiles);
% trainfiles = augmentedImageDatastore(imagesize,imgfiles,"ColorPreprocessing","gray2rgb");%灰度图像专用
```

Create a boxLabelDatastore using the label columns from the table.

```matlab
imglabel = boxLabelDatastore(traininglabel);
% trainlabels = augmentedImageDatastore(imagesize,imgfiles,"ColorPreprocessing","gray2rgb");%灰度图像专用
```

Combine the datastores.

```matlab
ds = combine(imgfiles, imglabel);
```

## 1.2 加载Yolov2模型

```matlab
Load a preinitialized YOLO v2 object detection network.
net = load('yolov2VehicleDetector.mat');
```

## 1.3 设置训练参数

```matlab
options = trainingOptions('sgdm',...
          'InitialLearnRate',0.001,...
          'Verbose',true,...
          'MiniBatchSize',32,...
          'MaxEpochs',100,...
          'Shuffle','never',...
          'VerboseFrequency',10,...
          'CheckpointPath',tempdir);
```

## 1.4 训练Yolov2模型

```matlab
Train the YOLO v2 network.
[detector,info] = trainYOLOv2ObjectDetector(ds,lgraph,options)
```

## 1.5 绘制训练模型损失曲线

```matlab
You can verify the training accuracy by inspecting the training loss for each iteration.
figure
plot(info.TrainingLoss)
grid on
xlabel('Number of Iterations')
ylabel('Training Loss for Each Iteration')
```

## 1.6 保存模型

```matlab
save("Yolo2Model.mat","detector");
save("TrainedInfo.mat","info");
```

## 二、测试模型

## 2.1 加载Yolov2模型

```matlab
clear;clc;
load Yolo2Model.mat
```

## 2.2 读取图像

```matlab
[file,path] = uigetfile("测试集\");
filepath = fullfile(path,file);
```

## 2.3 目标检测

```matlab
if file ~= 0
    img = imread(filepath);
    [bboxes,scores] = detect(detector,img); %利用训练好Yolov2模型进行检查
end
scores = strcat(string(round(scores,2)*100),"%");
number = string(numel(scores));
```

## 2.4 输出识别结果

```matlab
if(~isempty(bboxes))
    img = insertObjectAnnotation(img,'rectangle',bboxes,scores);
end
imshow(img),title(strcat("检测个数：",number));
```

