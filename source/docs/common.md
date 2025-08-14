  # Common.py API 接口说明文档

  ## 📋 目录

- [概述](#概述)
- [依赖库](#依赖库)
- [快速开始](#快速开始)
- [API 接口详细说明](#api-接口详细说明)
- [颜色定义常量](#1-颜色定义常量)
- [日志输出函数](#2-日志输出函数)
- [图像转换函数](#3-图像转换函数)
- [配置文件处理函数](#4-配置文件处理函数)
- [图像处理函数](#5-图像处理函数)
- [颜色管理类](#6-颜色管理类)
- [图像标注函数](#7-图像标注函数)
- [几何计算函数](#8-几何计算函数)
- [姿态变换函数](#9-姿态变换函数)
- [矩阵变换函数](#10-矩阵变换函数)
- [使用示例](#使用示例)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [注意事项](#注意事项)
- [版本信息](#版本信息)
- [更新日志](#更新日志)

## 📖 概述

`common.py` 是一个专为ROS机器人系统设计的通用工具函数库，提供了图像处理、坐标变换、数学计算等常用功能的API接口。该模块主要用于ROS机器人系统中的数据处理和转换操作，具有高效、稳定、易用的特点。

### ✨ 主要特性

- 🖼️ **图像处理**: 支持OpenCV与ROS图像格式转换
- 🔄 **坐标变换**: 提供四元数、欧拉角、变换矩阵间的相互转换
- 📐 **几何计算**: 包含距离计算、角度计算、坐标映射等功能
- ⚙️ **配置管理**: 支持YAML配置文件的读写操作
- 🎨 **可视化**: 提供图像标注和颜色管理功能

## 📦 依赖库

```python
import cv2          # OpenCV图像处理库 (>=4.0.0)
import math         # Python数学库
import yaml         # YAML配置文件处理 (>=5.0)
import rospy        # ROS Python客户端库
import numpy as np  # 数值计算库 (>=1.19.0)
import transforms3d # 3D变换库 (>=0.3.0)
import random       # 随机数生成库
```

### 安装依赖
```bash
pip install opencv-python numpy pyyaml transforms3d
```

## 🚀 快速开始

### 基本导入
```python
from sdk.common import cv2_image2ros, qua2rpy, distance, loginfo

# 使用日志功能
loginfo("系统初始化完成")

# 计算两点距离
dist = distance((0, 0), (3, 4))  # 返回 5.0
```

### 图像处理示例
```python
import cv2
from sdk.common import cv2_image2ros, get_area_max_contour

# 读取图像并转换为ROS格式
image = cv2.imread("test.jpg")
ros_image = cv2_image2ros(image, "camera_frame")

# 查找最大轮廓
contours, _ = cv2.findContours(gray, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
max_contour, area = get_area_max_contour(contours, threshold=100)
```

## 📚 API 接口详细说明

### 1. 颜色定义常量

#### `range_rgb` 字典
定义了常用颜色的BGR值范围，用于图像处理中的颜色识别和过滤。

```python
range_rgb = {
    'red': (0, 50, 255),      # 红色范围
    'blue': (255, 50, 0),     # 蓝色范围
    'green': (50, 255, 0),    # 绿色范围
    'black': (0, 0, 0),       # 黑色
    'white': (255, 255, 255), # 白色
}
```

**使用示例：**

```python
from sdk.common import range_rgb

# 获取红色范围
red_range = range_rgb['red']  # (0, 50, 255)

# 在图像处理中使用
lower_red = np.array([0, 50, 50])
upper_red = np.array([10, 255, 255])
```

### 2. 日志输出函数

#### `loginfo(msg)`
输出带颜色的ROS日志信息，便于调试和监控。

**参数：**
- `msg` (str): 要输出的消息内容

**返回值：** 无

**示例：**
```python
from sdk.common import loginfo

loginfo("机器人启动成功")
loginfo("检测到目标物体")
loginfo("坐标转换完成")
```

### 3. 图像转换函数

#### `cv2_image2ros(image, frame_id='')`
将OpenCV图像转换为ROS Image消息格式。

**参数：**
- `image` (numpy.ndarray): OpenCV格式的图像数据
- `frame_id` (str): 坐标系ID，默认为空字符串

**返回值：** ROS Image消息对象

**示例：**

```python
import cv2
from sdk.common import cv2_image2ros

# 读取图像
image = cv2.imread("camera_image.jpg")

# 转换为ROS格式
ros_img = cv2_image2ros(image, "camera_frame")

# 发布到ROS话题
pub = rospy.Publisher('/camera/image_raw', Image, queue_size=10)
pub.publish(ros_img)
```

#### `bgr8_to_jpeg(value, quality=75)`
将BGR8格式的图像数据转换为JPEG格式。

**参数：**

- `value` (numpy.ndarray): 原始BGR8格式图像数据
- `quality` (int): JPEG质量，范围1-100，默认75

**返回值：** JPEG格式的字节数据

**示例：**

```python
from sdk.common import bgr8_to_jpeg

# 转换图像格式
jpeg_data = bgr8_to_jpeg(image, quality=90)

# 保存为文件
with open("output.jpg", "wb") as f:
    f.write(jpeg_data)
```

### 4. 配置文件处理函数

#### `get_yaml_data(yaml_file)`
读取YAML配置文件并返回解析后的数据。

**参数：**

- `yaml_file` (str): YAML文件路径

**返回值：** 解析后的配置数据

**示例：**

```python
from sdk.common import get_yaml_data

# 读取配置文件
config = get_yaml_data("robot_config.yaml")

# 使用配置数据
robot_name = config['robot_name']
max_speed = config['max_speed']
```

#### `save_yaml_data(data, yaml_file)`

将数据保存为YAML格式文件。

**参数：**

- `data` (dict): 要保存的数据
- `yaml_file` (str): 目标YAML文件路径

**返回值：** 无

**示例：**

```python
from sdk.common import save_yaml_data

# 准备配置数据
config_data = {
    'robot_name': 'my_robot',
    'max_speed': 1.5,
    'sensors': ['camera', 'lidar', 'imu']
}

# 保存配置文件
save_yaml_data(config_data, "new_config.yaml")
```

### 5. 图像处理函数

#### `get_area_max_contour(contours, threshold=50)`

获取轮廓列表中面积最大的轮廓，并过滤掉面积过小的轮廓。

**参数：**

- `contours` (list): 轮廓列表
- `threshold` (int): 面积阈值，默认50

**返回值：** 元组 (最大轮廓, 最大面积)

**示例：**

```python
import cv2
from sdk.common import get_area_max_contour

# 查找轮廓
contours, _ = cv2.findContours(binary_image, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

# 获取最大轮廓
max_contour, max_area = get_area_max_contour(contours, threshold=100)

if max_contour is not None:
    # 绘制最大轮廓
    cv2.drawContours(image, [max_contour], -1, (0, 255, 0), 2)
    print(f"最大轮廓面积: {max_area}")
```

#### `warp_affine(image, points, scale=1.0)`

对图像进行仿射变换，使两点连线水平对齐。

**参数：**
- `image` (numpy.ndarray): 输入图像
- `points` (tuple): 两个点的坐标 ((x1, y1), (x2, y2))
- `scale` (float): 缩放比例，默认1.0

**返回值：** 变换后的图像

**示例：**

```python
from sdk.common import warp_affine

# 人脸对齐示例
face_image = cv2.imread("face.jpg")
eye_points = ((100, 120), (150, 125))  # 两眼坐标

# 对齐图像
aligned_face = warp_affine(face_image, eye_points, scale=1.0)
cv2.imwrite("aligned_face.jpg", aligned_face)
```

### 6. 颜色管理类

#### `Colors` 类

提供Ultralytics颜色调色板，用于图像标注和可视化。

**方法：**

- `__call__(i, bgr=False)`: 获取指定索引的颜色
- `hex2rgb(h)`: 将十六进制颜色转换为RGB元组

**示例：**

```python
from sdk.common import Colors

# 创建颜色实例
colors = Colors()

# 获取颜色
color1 = colors(0)      # 获取第一个颜色 (RGB格式)
color2 = colors(1, bgr=True)  # 获取第二个颜色 (BGR格式)

# 绘制不同颜色的边界框
for i, box in enumerate(boxes):
    color = colors(i)
    cv2.rectangle(image, box[0], box[1], color, 2)
```

### 7. 图像标注函数

#### `plot_one_box(x, img, color=None, label=None, line_thickness=None)`

在图像上绘制边界框和标签。

**参数：**

- `x` (list): 边界框坐标 [x1, y1, x2, y2]
- `img` (numpy.ndarray): 目标图像
- `color` (tuple): 绘制颜色，默认随机
- `label` (str): 标签文本
- `line_thickness` (int): 线条粗细

**返回值：** 无

**示例：**

```python
from sdk.common import plot_one_box, Colors

# 创建颜色实例
colors = Colors()

# 绘制边界框
box = [100, 100, 200, 200]
plot_one_box(box, image, color=colors(0), label="Person", line_thickness=2)

# 批量绘制
for i, detection in enumerate(detections):
    box = detection['bbox']
    label = f"{detection['class']}: {detection['confidence']:.2f}"
    plot_one_box(box, image, color=colors(i), label=label)
```

### 8. 几何计算函数

#### `distance(point_1, point_2)`

计算两点间的欧几里得距离。

**参数：**

- `point_1` (tuple): 第一个点坐标 (x1, y1)
- `point_2` (tuple): 第二个点坐标 (x2, y2)

**返回值：** 两点间距离 (float)

**示例：**

```python
from sdk.common import distance

# 计算距离
dist = distance((0, 0), (3, 4))  # 返回 5.0
dist = distance((10, 20), (15, 25))  # 返回 7.07...

# 在机器人导航中使用
robot_pos = (x, y)
target_pos = (target_x, target_y)
remaining_distance = distance(robot_pos, target_pos)
```

#### `box_center(box)`

计算边界框的中心点。

**参数：**

- `box` (list): 边界框坐标 (x1, y1, x2, y2)

**返回值：** 中心点坐标 (tuple)

**示例：**

```python
from sdk.common import box_center

# 计算边界框中心
box = [100, 100, 200, 200]
center = box_center(box)  # 返回 (150.0, 150.0)

# 在目标跟踪中使用
for detection in detections:
    bbox = detection['bbox']
    center_point = box_center(bbox)
    print(f"目标中心: {center_point}")
```

#### `point_remapped(point, now, new, data_type=float)`

将点坐标从一个图像尺寸映射到另一个图像尺寸。

**参数：**
- `point` (tuple): 原始点坐标 (x, y)
- `now` (tuple): 当前图像尺寸 (width, height)
- `new` (tuple): 新图像尺寸 (width, height)
- `data_type` (type): 返回数据类型，默认float

**返回值：** 映射后的点坐标

**示例：**

```python
from sdk.common import point_remapped

# 坐标映射
original_point = (100, 100)
original_size = (640, 480)
new_size = (1280, 960)

new_point = point_remapped(original_point, original_size, new_size)
# 返回 (200.0, 200.0)

# 在图像缩放中使用
resized_image = cv2.resize(image, new_size)
keypoints = [point_remapped(p, original_size, new_size) for p in original_keypoints]
```

#### `vector_2d_angle(v1, v2)`

计算两个2D向量间的夹角。

**参数：**
- `v1` (list/numpy.ndarray): 第一个向量
- `v2` (list/numpy.ndarray): 第二个向量

**返回值：** 夹角（度），范围 -180 到 180

**示例：**

```python
import numpy as np
from sdk.common import vector_2d_angle

# 计算向量夹角
v1 = np.array([1, 0])
v2 = np.array([0, 1])
angle = vector_2d_angle(v1, v2)  # 返回 90.0

# 在机器人转向中使用
current_direction = np.array([robot_vx, robot_vy])
target_direction = np.array([target_vx, target_vy])
turn_angle = vector_2d_angle(current_direction, target_direction)
```

### 9. 姿态变换函数

#### `qua2rpy(*qua)`

将四元数转换为欧拉角（Roll-Pitch-Yaw）。

**参数：**

- `qua`: 四元数，可以是ROS Quaternion对象或四元数数组 [x, y, z, w]

**返回值：** 欧拉角元组 (roll, pitch, yaw)

**示例：**

```python
import math
from sdk.common import qua2rpy

# 从ROS Quaternion转换
from geometry_msgs.msg import Quaternion
quat = Quaternion()
quat.x, quat.y, quat.z, quat.w = 0, 0, 0, 1
roll, pitch, yaw = qua2rpy(quat)

# 从数组转换
quat_array = [0, 0, 0, 1]  # 单位四元数
roll, pitch, yaw = qua2rpy(*quat_array)

# 转换为度数
roll_deg = math.degrees(roll)
pitch_deg = math.degrees(pitch)
yaw_deg = math.degrees(yaw)
```

#### `rpy2qua(roll, pitch, yaw)`

将欧拉角转换为四元数。

**参数：**

- `roll` (float): 滚转角（弧度）
- `pitch` (float): 俯仰角（弧度）
- `yaw` (float): 偏航角（弧度）

**返回值：** ROS Quaternion对象

**示例：**

```python
import math
from sdk.common import rpy2qua

# 欧拉角转四元数
roll = 0
pitch = 0
yaw = math.pi/2  # 90度偏航角

quat = rpy2qua(roll, pitch, yaw)

# 在ROS消息中使用
from geometry_msgs.msg import PoseStamped
pose_msg = PoseStamped()
pose_msg.pose.orientation = quat
```

### 10. 矩阵变换函数

#### `xyz_quat_to_mat(xyz, quat)`

将位置和四元数转换为4x4变换矩阵。

**参数：**

- `xyz` (list): 位置向量 [x, y, z]
- `quat` (list): 四元数 [x, y, z, w]

**返回值：** 4x4变换矩阵 (numpy.ndarray)

**示例：**

```python
import numpy as np
from sdk.common import xyz_quat_to_mat

# 创建变换矩阵
position = [1, 2, 3]
quaternion = [0, 0, 0, 1]  # 单位四元数

transform_matrix = xyz_quat_to_mat(position, quaternion)
print(f"变换矩阵:\n{transform_matrix}")
```

#### `xyz_rot_to_mat(xyz, rot)`
将位置和旋转矩阵转换为4x4变换矩阵。

**参数：**
- `xyz` (list): 位置向量 [x, y, z]
- `rot` (numpy.ndarray): 3x3旋转矩阵

**返回值：** 4x4变换矩阵

**示例：**

```python
import numpy as np
from sdk.common import xyz_rot_to_mat

# 创建旋转矩阵（绕Z轴旋转90度）
rotation_matrix = np.array([
    [0, -1, 0],
    [1, 0, 0],
    [0, 0, 1]
])

position = [1, 2, 3]
transform_matrix = xyz_rot_to_mat(position, rotation_matrix)
```

#### `xyz_euler_to_mat(xyz, euler, degrees=True)`

将位置和欧拉角转换为4x4变换矩阵。

**参数：**

- `xyz` (list): 位置向量 [x, y, z]
- `euler` (list): 欧拉角 [roll, pitch, yaw]
- `degrees` (bool): 欧拉角是否为度数，默认True

**返回值：** 4x4变换矩阵

**示例：**

```python
from sdk.common import xyz_euler_to_mat

# 使用度数
position = [1, 2, 3]
euler_angles = [0, 0, 90]  # 90度偏航角
mat1 = xyz_euler_to_mat(position, euler_angles, degrees=True)

# 使用弧度
import math
euler_radians = [0, 0, math.pi/2]
mat2 = xyz_euler_to_mat(position, euler_radians, degrees=False)
```

#### `mat_to_xyz_euler(mat, degrees=True)`

从4x4变换矩阵提取位置和欧拉角。

**参数：**

- `mat` (numpy.ndarray): 4x4变换矩阵
- `degrees` (bool): 返回的欧拉角是否为度数，默认True

**返回值：** 元组 (位置向量, 欧拉角)

**示例：**

```python
from sdk.common import mat_to_xyz_euler

# 从变换矩阵提取信息
xyz, euler = mat_to_xyz_euler(transform_matrix, degrees=True)
print(f"位置: {xyz}")
print(f"欧拉角: {euler}")

# 在机器人位姿估计中使用
robot_pose_matrix = get_robot_pose_matrix()
position, orientation = mat_to_xyz_euler(robot_pose_matrix)
```

## 💡 使用示例

### 完整的图像处理流程

```python
import cv2
import numpy as np
from sdk.common import (
    cv2_image2ros, get_area_max_contour, plot_one_box,
    Colors, distance, box_center
)

def process_image(image):
    """完整的图像处理示例"""
    # 1. 图像预处理
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    
    # 2. 边缘检测
    edges = cv2.Canny(blurred, 50, 150)
    
    # 3. 查找轮廓
    contours, _ = cv2.findContours(edges, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    # 4. 获取最大轮廓
    max_contour, area = get_area_max_contour(contours, threshold=100)
    
    if max_contour is not None:
        # 5. 获取边界框
        x, y, w, h = cv2.boundingRect(max_contour)
        bbox = [x, y, x+w, y+h]
        
        # 6. 计算中心点
        center = box_center(bbox)
        
        # 7. 绘制结果
        colors = Colors()
        plot_one_box(bbox, image, color=colors(0), label=f"Area: {area:.0f}")
        cv2.circle(image, (int(center[0]), int(center[1])), 5, (0, 255, 0), -1)
    
    # 8. 转换为ROS格式
    ros_image = cv2_image2ros(image, "camera_frame")
    
    return image, ros_image
```

### 机器人姿态控制示例

```python
import math
import numpy as np
from sdk.common import (
    qua2rpy, rpy2qua, xyz_euler_to_mat, 
    mat_to_xyz_euler, distance
)

class RobotPoseController:
    def __init__(self):
        self.current_pose = None
    
    def set_target_pose(self, x, y, z, roll, pitch, yaw):
        """设置目标位姿"""
        # 创建目标变换矩阵
        target_matrix = xyz_euler_to_mat([x, y, z], [roll, pitch, yaw])
        return target_matrix
    
    def get_pose_error(self, current_matrix, target_matrix):
        """计算位姿误差"""
        # 提取位置和姿态
        current_pos, current_euler = mat_to_xyz_euler(current_matrix)
        target_pos, target_euler = mat_to_xyz_euler(target_matrix)
        
        # 计算位置误差
        pos_error = distance(current_pos, target_pos)
        
        # 计算角度误差
        angle_error = abs(current_euler[2] - target_euler[2])  # 偏航角误差
        
        return pos_error, angle_error
    
    def control_loop(self, target_pose):
        """控制循环"""
        while True:
            # 获取当前位姿
            current_matrix = self.get_current_pose_matrix()
            
            # 计算误差
            pos_error, angle_error = self.get_pose_error(current_matrix, target_pose)
            
            # 检查是否到达目标
            if pos_error < 0.01 and angle_error < 0.01:
                print("到达目标位姿")
                break
            
            # 执行控制命令
            self.execute_control_command(pos_error, angle_error)
```

## 🎯 最佳实践

### 1. 性能优化

```python
# 避免重复计算
colors = Colors()  # 创建一次，重复使用

# 使用numpy向量化操作
points = np.array([[x1, y1], [x2, y2], [x3, y3]])
distances = np.linalg.norm(points, axis=1)
```

### 2. 错误处理

```python
def safe_image_conversi
