# 如何在树莓派中运行使用MS-COCO训练的Tiny_YOLO_V2 

`注意：该方法仅适用于NCS V1。NCS V2 应当使用OpenVINO。`
## 1 树莓派的基础设置
### 步骤 1.1： 下载树莓派镜像
从树莓派的官方网站[下载raspbian的镜像](https://www.raspberrypi.org/downloads/raspbian/) Stretch。<br>

请注意，为了方便使用Tkinter制作的GUI 界面，所以应当选择使用Raspbian Stretch with desktop<br>
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/stretch.png) <br>
<br>测试运行版本：[2018-11-13-raspbian-stretch.img](https://downloads.raspberrypi.org/raspbian/images/raspbian-2018-11-15/2018-11-13-raspbian-stretch.zip.torrent)

### 步骤 1.2 ：烧录镜像文件

下载完成 镜像后，需要将镜像文件烧录进已经准备好的tf卡中， 本教程使用的软件为Win32DiskImager


### 步骤 1.3 ： 树莓派的基本设置


完成烧录后，将tf卡插入树莓派，借助Raspbian的简易入门教程，完成树莓派的基本设置。<br>
完成后，进行系统升级。
```shell
sudo aptitude update && sudo aptitude dist-upgrade  && sudo reboot
```
注意，在设置菜单中需要保证。打开VNC以及Serial Port

### 步骤 1.4： 将`Movidius NCS V1`插入树莓派
再次注意：该方法仅适用于`NCS V1`。NCS V2 不适用于本方法。

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

## 2 安装Movidius SDK
代码来自[Inter](https://github.com/movidius/ncsdk)
### 步骤 2.0： 更改RPi上交换文件大小
```shell

sudo nano /etc/dphys-swapfile

```
将`CONF_SWAPSIZE`的默认值100更改为1024或者更高。
```shell

sudo /etc/init.d/dphys-swapfile restart

```

### 步骤  2.1：克隆SDK仓库
```shell

cd 

git clone -b ncsdk2 https://github.com/movidius/ncsdk.git

```
### 步骤  2.2： 安装SDK

```shell

cd ncsdk/
sudo make install

```
输入密码后便开始SDK的安装过程，整个安装过程可能会持续十五分钟



### 步骤  2.3： 运行示例（包括 OpenCV安装）

当你看到如下的图片时候，表示你已经成功安装了SDK。<br>

![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/end-sdk.png) <br>
现在应当遵从指示，开启一个新的Terminal<br>
然后运行如下指令：

```shell
cd ncsdk/
sudo make examples

```
在确认安装OpenCV后，将开始的安装过程，整个安装过程可能会持续四十分钟。 <br>

### 步骤 2.4： 运行hello ncs程序，确认安装正确
```shell
cd /ncsdk/examples/apps/hello_ncs_py

make run

```
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/hello-ncs.png) <br>
如果你看到如上图所示的结果，那么恭喜你，完成了ncsdk的安装

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

## 安装NCAPPZOO
### 步骤 3.1： 克隆NACPPZOO
代码来自[Inter](https://github.com/movidius/ncappzoo)
```shell

git clone -b ncsdk2 https://github.com/movidius/ncappzoo.git

```


### 步骤 3.2： 开始tiny-yolo-v2（voc）运行环境
```shell
cd 
cd ncsdk/ncappzoo/tensorflow/tiny_yolo_v2/
sudo make all

```
在这个过程中，程序会自动下载darkflow，以及权重等需要的软件。<br>
运行过程中，应当会出现以下内容：

![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-1.png) <br>
下载权重以及cfg文件
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-2.png)<br>
生成对应网络的pb文件
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-3.png)<br>
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-4.png)<br>
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/make-all-5.png)<br>
生成graph文件，并将网络编译入Movidius
运行完成后，我们就已经成功

#### 步骤 3.2-5：错误提示
如果运行失败，并且terminal中没有出现如下语句：
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/2017.png)<br>
而是出现，<br>
`mvNCCompile, Copyright @ Intel Corporation 2016`<br>
那么，请运行`make uninstall`指令，卸载ncsdk。重新执行步骤2.0以后的步骤<br>


### 步骤3.3：开始首次运行
```shell
sudo make run

```
现在，运行上方的shell指令，就能够完成首次Tiny-yolo-v2的运行了。
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/end-make-run.png)<br>
最终获得的结果，可以看出，程序已经能够顺利识别椅子了。

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

## 运行MS-COCO为基础的Tiny-yolo-v2
### 步骤 4.1： 更改makefile
打开tiny-yolo-v2下的Makefile文件。
```java

DARKFLOW_FOLDER = darkflow
BUILD_FOLDER = built_graph
CFG_FILENAME = yolov2-tiny-voc.cfg
WEIGHTS_FILENAME = yolov2-tiny-voc.weights
TF_MODEL_FILENAME = yolov2-tiny-voc.pb
GRAPH_FILENAME = tiny_yolo_v2.graph
```
,将原文件中，上述六行代码更改为如下代码：
```java

DARKFLOW_FOLDER = darkflow
BUILD_FOLDER = built_graph
CFG_FILENAME = yolov2-tiny.cfg
WEIGHTS_FILENAME = yolov2-tiny.weights
TF_MODEL_FILENAME = yolov2-tiny.pb
GRAPH_FILENAME = tiny_yolo_v2.graph
```

### 步骤 4.2： 更改darkflow文件
```shell
sudo nano /usr/local/lib/python3.5/dist-packages/darkflow/utils/loader.py 

```
找到第121行
将`self.offset = 16` 更改为`self.offset = 20`


### 步骤4.3： 更改label.txt
将仓库中，label.txt文件覆盖至该目录<br>
原先使用的label是voc数据库的仅有二十个分类。<br>
现在使用的coco数据库，总共有80个分类。

### 步骤 4.4： 更改run.py
如果觉得下面步骤太过复杂，可以直接从仓库中下载修改好的run.py文件进行覆盖。
####  更改post_processing函数
##### 更改num_classes original_results reordered_results 变量
由于label的数量发生了变化，因此相应变量数目需要更改<br>
72行：
```python
# num_classes = 20
num_classes = 80
```

80行：
```python
# original_results = np.reshape(original_results, (13, 13, 125))
original_results = np.reshape(original_results, (13, 13, 425))
```
85行：
```python
# reordered_results = np.zeros((13 * 13, 5, 25))
reordered_results = np.zeros((13 * 13, 5, 85))
```
90行：
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

##### 需要更改锚点的设置，在py文件的97行：
```python
# shapes for the 5 Tiny Yolo v2 bounding boxes
# anchor_boxes = [1.08,1.19, 3.42,4.41, 6.63,11.38, 9.42,5.11, 16.62,10.52] # voc
anchor_boxes = [0.57273,0.677385,1.87446,2.06253,3.33843,5.47434,7.88282,3.52778,9.77052,9.16828] # coco
```
将原先voc使用的锚点替换为新的coco使用的锚点。具体数据同样可以在刚刚下载的cfg文件中查找到。<br>

#### 回到第六行，将原先的`labe_name`替换为现在的如下的label_name
![](https://github.com/asd2511/yolo-v2-with-ncs/blob/master/img/label-name.png)<br>

### 步骤 4.5 ：删除现有的Tiny-yolo-v2文件

```shell
sudo make clean
```
请不要担心，label.txt，与run.py不会被删除。

### 步骤 4.6
```shell
sudo make run
```
如果运行成功，那么恭喜你，完成了文件的转换。可以正常运行了~

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
