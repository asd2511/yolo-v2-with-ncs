# How to run Tiny_YOLO_V2 using MS-COCO training in the Raspberry Pi

`Note: This method is only available for NCS V1. NCS V2 should use OpenVINO。`
## 1 Raspberry Pi basic settings
### Step 1.1: Download the Raspberry Pi image
Download the raspbian Stretch image from the [official website of the Raspberry Pi]
https://www.raspberrypi.org/downloads/raspbian/) 。<br>

Please note that you should choose to use Raspbian Stretch with desktop.<br>
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/stretch.png) <br>
<br>runging test version：[2018-11-13-raspbian-stretch.img](https://downloads.raspberrypi.org/raspbian/images/raspbian-2018-11-15/2018-11-13-raspbian-stretch.zip.torrent)

### Step 1.2: Burn the image file

After the image is downloaded, you need to burn the image file into the prepared tf card. The software used in this tutorial is Win32DiskImager.


### Step 1.3: Basic settings for the Raspberry Pi


After completing the burning, insert the tf card into the Raspberry Pi and complete the basic settings of the Raspberry Pi with Raspbian's easy-to-follow tutorial.<br>
After the completion, perform a system upgrade.
```shell
sudo aptitude update && sudo aptitude dist-upgrade  && sudo reboot
```
Note that it needs to be guaranteed in the setup menu. Open VNC and Serial Port

### Step 1.4: Insert `Movidius NCS V1` into the Raspberry Pi
Note again: this method is only available for `NCS V1`. NCS V2 is not available for this method.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

## 2 Install the Movidius SDK
Code from[Inter](https://github.com/movidius/ncsdk)
### Step 2.0: Change the size of the swap file on RPI
```shell

sudo nano /etc/dphys-swapfile

```
Change the default value of `CONF_SWAPSIZE` from 100 to 1024 or higher.
```shell

sudo /etc/init.d/dphys-swapfile restart

```

### Step 2.1: Cloning the SDK Repository
```shell

cd 

git clone -b ncsdk2 https://github.com/movidius/ncsdk.git

```
### Step 2.2: Install the SDK

```shell

cd ncsdk/
sudo make install

```
After entering the password, the SDK installation process begins, and the entire installation process may last for fifteen minutes.



### Step 2.3: Run the example (including OpenCV installation)

When you see the picture below, it means that you have successfully installed the SDK.<br>

![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/end-sdk.png) <br>
You should now follow the instructions or open a new Terminal<br>
Then run the following command:

```shell
sudo make examples

```
在确认安装OpenCV后，将开始的安装过程，整个安装过程可能会持续四十分钟。 <br>

### Step 2.4: Run the hello ncs program to confirm that the installation is correct.
```shell
cd /ncsdk/examples/apps/hello_ncs_py

make run

```
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/hello-ncs.png) <br>
If you see the results shown above, congratulations, complete the installation of ncsdk

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

## Install NCAPPZOO
### Step 3.1: Clone NACPPZOO
Code from[Inter](https://github.com/movidius/ncappzoo)
```shell

git clone -b ncsdk2 https://github.com/movidius/ncappzoo.git

```


### Step 3.2: Start set the tiny-yolo-v2 (voc) runtime environment
```shell
cd 
cd ncsdk/ncappzoo/tensorflow/tiny_yolo_v2/
sudo make all

```
In the process, the program will automatically download darkflow, as well as the required software such as weights.<br>
During the operation, the following should appear:

![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-1.png) <br>
Download weight and cfg file
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-2.png)<br>
Generate the corresponding network pb file
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-3.png)<br>
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-4.png)<br>
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-5.png)<br>
Generate a graph file and compile the network into Movidius
After the operation is complete, we have succeeded

#### Step 3.2-5: Error message
If the run fails, and the following statement does not appear in the terminal:
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/2017.png)<br>
But appear,<br>
`mvNCCompile, Copyright @ Intel Corporation 2016`<br>
Then, run the `make uninstall` command to uninstall ncsdk. Re-execute the steps after step 2.0<br>


### Step 3.3: Start the first run
```shell
sudo make run

```
Now, running the shell command above, you can complete the first Tiny-yolo-v2 run.
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/end-make-run.png)<br>
As a result of the final results, it can be seen that the program has been able to recognize the chair smoothly.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

## Running MS-COCO-based Tiny-yolo-v2
### Step 4.1: Change the makefile
Open the Makefile under tiny-yolo-v2.
```java

DARKFLOW_FOLDER = darkflow
BUILD_FOLDER = built_graph
CFG_FILENAME = yolov2-tiny-voc.cfg
WEIGHTS_FILENAME = yolov2-tiny-voc.weights
TF_MODEL_FILENAME = yolov2-tiny-voc.pb
GRAPH_FILENAME = tiny_yolo_v2.graph
```
in the original file, the above six lines of code are changed to the following code:
```java

DARKFLOW_FOLDER = darkflow
BUILD_FOLDER = built_graph
CFG_FILENAME = yolov2-tiny.cfg
WEIGHTS_FILENAME = yolov2-tiny.weights
TF_MODEL_FILENAME = yolov2-tiny.pb
GRAPH_FILENAME = tiny_yolo_v2.graph
```

### Step 4.2: Change the darkflow file
```shell
sudo nano /usr/local/lib/python3.5/dist-packages/darkflow/utils/loader.py 

```
Find line 121
Change `self.offset = 16` to `self.offset = 20`


### Step 4.3: Change label.txt
Overwrite the `label.txt` file in the repository to this directory<br>
The original label used was only twenty categories of the voc database.<br>
There are a total of 80 categories in the coco database now in use.

### Step 4.4: Change run.py
If you feel that the following steps are too complicated, you can download the modified `run.py` file directly from the repository for overwriting.
####  Change the post_processing function
##### Change` num_classes`` original_results`` reordered_results` variable
Since the number of labels has changed, the number of corresponding variables needs to be changed.<br>
Line 72:
```python
# num_classes = 20
num_classes = 80
```

Line 80:
```python
# original_results = np.reshape(original_results, (13, 13, 125))
original_results = np.reshape(original_results, (13, 13, 425))
```
Line 85:
```python
# reordered_results = np.zeros((13 * 13, 5, 25))
reordered_results = np.zeros((13 * 13, 5, 85))
```
Line 90:
```python
'''
for b_box_voltron in range(125):
                b_box = row * num_grids + col
                b_box_num = int(b_box_voltron / 25)
                b_box_info = b_box_voltron % 25'''
                
for b_box_voltron in range(425):
                b_box = row * num_grids + col
                b_box_num = int(b_box_voltron / 85)
                b_box_info = b_box_voltron % 85
```

##### Need to change the anchor settings in line 97 of the py file:
```python
# shapes for the 5 Tiny Yolo v2 bounding boxes
# anchor_boxes = [1.08,1.19, 3.42,4.41, 6.63,11.38, 9.42,5.11, 16.62,10.52] # voc
anchor_boxes = [0.57273,0.677385,1.87446,2.06253,3.33843,5.47434,7.88282,3.52778,9.77052,9.16828] # coco
```
Replace the anchor used by the original voc with the anchor used by the new coco. The specific data can also be found in the cfg file just downloaded.<br>

####Go back to the sixth line and replace the original `labe_name` with the current label_name
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/label-name.png)<br>
You can find it in the repository, `label_name.txt`
### Step 4.5: Delete the existing Tiny-yolo-v2 file

```shell
sudo make clean
```
Please don't worry, `label.tx`t, and `run.py` won't be deleted.

### Step 4.6
```shell
sudo make run
```
If the operation is successful, then congratulations, complete the conversion of the file. Can run normally~<br>
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/apple.png)<br>
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
