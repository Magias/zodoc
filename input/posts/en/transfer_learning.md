title: Transfer Learning With AlexNet
lead: An example of transfer learning with Alexnet neural network on a database of generated shapes.
Published: 2019-01-09
Tags: [matlab, imageset, shapes, transfer learning]
prerequisites: [Matlab]
Authors: [tesar-tech, magias]
---

Transfer learning is commonly used by deep learning applications. In practice,y ou can take a pretrained network and use it as a starting point to learn a new task. Fine-tuning a network with transfer learning is usually much faster and easier than training a network with randomly initialized weights from scratch. You can quickly transfer learned features to a new task using a smaller number of training images. For this example we will use Image dataset, that contains 4 labels: circle, rectangle , triangle and star, created with this[script](creating_an_image_set_with_various_shapes).

``` matlab
imds = imageDatastore('imgs_shapes', ... %Load image from folder
    'IncludeSubfolders',true, ... %Load also from subflder
    'LabelSource','foldernames'); %Use label source same as the file names 
%Split dataset on training and validation
[imdsTrain,imdsValidation] = splitEachLabel(imds,0.7,'randomized');

%Load and show random sample from image dataset
numTrainImages = numel(imdsTrain.Labels);
idx = randperm(numTrainImages,16);
figure
for i = 1:16
    subplot(4,4,i)
    I = readimage(imdsTrain,idx(i));
    imshow(I)
end

%Load pretrained neaural network
net = alexnet;

%Last 3 layers will be replaced 
inputSize = net.Layers(1).InputSize; %input size
layersTransfer = net.Layers(1:end-3);
numClasses = numel(categories(imdsTrain.Labels)); %Label count

%Final layers for our new neural network
layers = [
    layersTransfer
    fullyConnectedLayer(numClasses,'WeightLearnRateFactor',20,'BiasLearnRateFactor',20)
    softmaxLayer
    classificationLayer];

%Train Network
%Options for Training
options = trainingOptions('sgdm', ...
    'MiniBatchSize',10, ...
    'MaxEpochs',12, ... 
    'InitialLearnRate',1e-4, ...
    'Shuffle','every-epoch', ...
    'ValidationData',imdsValidation, ...
    'ValidationFrequency',3, ...
    'Verbose',false, ...
    'Plots','training-progress');

%Start training of our neural network with transfered and modified layers
netTransfer = trainNetwork(imdsTrain,layers,options);
```