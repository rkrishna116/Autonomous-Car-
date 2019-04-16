We trained a convolutional neural network (CNN) to map raw pixels from a single
front-facing camera directly to steering commands. This end-to-end approach
proved surprisingly powerful. With minimum training data from humans the system
learns to drive in traffic on local roads with or without lane markings and on
highways. It also operates in areas with unclear visual guidance such as in parking
lots and on unpaved roads.
The system automatically learns internal representations of the necessary processing
steps such as detecting useful road features with only the human steering angle
as the training signal. We never explicitly trained it to detect, for example, the outline
of roads.
Compared to explicit decomposition of the problem, such as lane marking detection,
path planning, and control, our end-to-end system optimizes all processing
steps simultaneously. We argue that this will eventually lead to better performance
and smaller systems. Better performance will result because the internal
components self-optimize to maximize overall system performance, instead of optimizing
human-selected intermediate criteria, e. g., lane detection. Such criteria
understandably are selected for ease of human interpretation which doesn’t automatically
guarantee maximum system performance. Smaller networks are possible
because the system learns to solve the problem with the minimal number of
processing steps.
We used an NVIDIA DevBox and Torch 7 for training and an NVIDIA
DRIVETM PX self-driving car computer also running Torch 7 for determining
where to drive. The system operates at 30 frames per second (FPS).

CNNs have revolutionized pattern recognition [2]. Prior to the widespread adoption of CNNs,
most pattern recognition tasks were performed using an initial stage of hand-crafted feature extraction
followed by a classifier. The breakthrough of CNNs is that features are learned automatically
from training examples. The CNN approach is especially powerful in image recognition tasks because
the convolution operation captures the 2D nature of images. Also, by using the convolution
kernels to scan an entire image, relatively few parameters need to be learned compared to the total
number of operations.
While CNNs with learned features have been in commercial use for over twenty years [3], their
adoption has exploded in the last few years because of two recent developments. First, large, labeled
data sets such as the Large Scale Visual Recognition Challenge (ILSVRC) [4] have become available
for training and validation. Second, CNN learning algorithms have been implemented on the
massively parallel graphics processing units (GPUs) which tremendously accelerate learning and
inference.

OVERVIEW OF DAVE-2 SYSTEM

Three cameras are mounted behind the windshield of the data-acquisition car. Time-stamped video
from the cameras is captured simultaneously with the steering angle applied by the human driver.
This steering command is obtained by tapping into the vehicle’s Controller Area Network (CAN)
bus. In order to make our system independent of the car geometry, we represent the steering command
as 1=r where r is the turning radius in meters. We use 1=r instead of r to prevent a singularity
when driving straight (the turning radius for driving straight is infinity). 1=r smoothly transitions
through zero from left turns (negative values) to right turns (positive values).
Training data contains single images sampled from the video, paired with the corresponding steering
command (1=r). Training with data from only the human driver is not sufficient. The network must
learn how to recover from mistakes. Otherwise the car will slowly drift off the road. The training

DATA COLLECTION

Training data was collected by driving on a wide variety of roads and in a diverse set of lighting
and weather conditions. Most road data was collected in central New Jersey, although highway data
was also collected from Illinois, Michigan, Pennsylvania, and New York. Other road types include
two-lane roads (with and without lane markings), residential roads with parked cars, tunnels, and
unpaved roads. Data was collected in clear, cloudy, foggy, snowy, and rainy weather, both day and
night. In some instances, the sun was low in the sky, resulting in glare reflecting from the road
surface and scattering from the windshield.
Data was acquired using either our drive-by-wire test vehicle, which is a 2016 Lincoln MKZ, or
using a 2013 Ford Focus with cameras placed in similar positions to those in the Lincoln. The
system has no dependencies on any particular vehicle make or model. Drivers were encouraged to
maintain full attentiveness, but otherwise drive as they usually do. As of March 28, 2016, about 72
hours of driving data was collected.
data is therefore augmented with additional images that show the car in different shifts from the
center of the lane and rotations from the direction of the road.

NETWORK ARCHITECTURE

We train the weights of our network to minimize the mean squared error between the steering command
output by the network and the command of either the human driver, or the adjusted steering
command for off-center and rotated images (see Section 5.2). Our network architecture is shown in
Figure 4. The network consists of 9 layers, including a normalization layer, 5 convolutional layers
and 3 fully connected layers. The input image is split into YUV planes and passed to the network.
The first layer of the network performs image normalization. The normalizer is hard-coded and is not
adjusted in the learning process. Performing normalization in the network allows the normalization
scheme to be altered with the network architecture and to be accelerated via GPU processing.
The convolutional layers were designed to perform feature extraction and were chosen empirically
through a series of experiments that varied layer configurations. We use strided convolutions in the
first three convolutional layers with a 22 stride and a 55 kernel and a non-strided convolution
with a 33 kernel size in the last two convolutional layers.
We follow the five convolutional layers with three fully connected layers leading to an output control
value which is the inverse turning radius. The fully connected layers are designed to function as a
controller for steering, but we note that by training the system end-to-end, it is not possible to make
a clean break between which parts of the network function primarily as feature extractor and which
serve as controller.
