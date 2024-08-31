# openCV 入门学习记录

## 数据采集

```python
import cv2  
import numpy as np  
import os  

person = 'username'	#准备识别人的名称
total_collection=200	#准备收集识别数量

# 打开摄像头  
cap = cv2.VideoCapture(0)
#这里使用 cv2.data 下的haarcascade_frontalface_alt2.xml 进行人脸识别
face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_alt2.xml')  

# 创建 dataset 目录  
if not os.path.exists("dataset/" + person):  
    os.makedirs("dataset/" + person)  

#如果摄像头没有打开
if not cap.isOpened():  
    print("Error: Could not open camera.")  
    exit()
    
count = 0	#记录当前收集到人脸的数量
while True:  
    ret, frame = cap.read()  
    if ret:  
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)  #转成灰度图
        faces = face_cascade.detectMultiScale(gray, scaleFactor=1.5, minNeighbors=5)  

        for (x, y, w, h) in faces:  
            cv2.rectangle(frame, (x, y), (x+w, y+h), (255, 0, 0), 2)   #画框框

            # 只拍摄第一个检测到的面孔  
            if count < total_collection:  
                count += 1  
                cv2.imwrite(f"dataset/{person}/{count}.jpg", gray[y:y+h, x:x+w])  # 保存面孔图像  
                print(f"Saved image {count}: dataset/{person}/{count}.jpg")  

        cv2.imshow("result", frame)  

        # 按 'q' 键退出  
        if cv2.waitKey(1) & 0xFF == ord('q'):  
            break  

        # 当达到了200张图片后时停止  
        if count >= total_collection:
            print("Collected 200 face images.")  
            break  

# 释放资源  
cap.release()  
cv2.destroyAllWindows()
```

文件结构：

- dataset
  - person1
    - pic1
    - pic2
  - person2
    - pic1
    - pic2
    - ...





## 训练并保存自己的识别器

```python
#----------------------训练并保存自己的识别器---------------------------
import os  
import cv2  
import numpy as np  
import pickle  

BASE_DIR = os.path.dirname(os.path.abspath(__file__))  
image_dir = os.path.join(BASE_DIR, 'dataset')  

current_id = 0  
label_ids = {}  
x_train = []  
y_labels = []  

# 创建一个 LBPH 面部识别器  
recognizer = cv2.face.LBPHFaceRecognizer_create()  # 注意构造函数的使用  
classifier = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')  

for root, dirs, files in os.walk(image_dir):  
    for file in files:  
        path = os.path.join(root, file)  # 每张图片的绝对路径  
        image = cv2.imread(path)  

        if image is None:  # 检查图像是否成功加载  
            print(f"Warning: Could not read image {path}")  
            continue  

        gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)  
        image_array = np.array(gray, 'uint8')  
        label = os.path.basename(root)  

        # 打标签  
        if label not in label_ids:  
            label_ids[label] = current_id  
            current_id += 1  

        id_ = label_ids[label]  

        # 识别np  
        faces = classifier.detectMultiScale(image_array, scaleFactor=1.1, minNeighbors=3)  

        if len(faces) == 0:  # 如果没有检测到面孔  
            print(f"Warning: No faces detected in image {path}")  
            continue  

        for (x, y, w, h) in faces:  
            roi = image_array[y:y + h, x:x + w]  
            x_train.append(roi)  
            y_labels.append(id_)  

        # 可选：显示图像以进一步调试  
        # cv2.imshow("Image", image)  
        # cv2.waitKey(1000)  # 等待1秒查看  
        # cv2.destroyAllWindows()  

# 在训练之前打印 x_train 和 y_labels 的长度  
print(f"Number of training samples: {len(x_train)}")  
print(f"Number of labels: {len(y_labels)}")  

if len(x_train) == 0 or len(y_labels) == 0:  
    print("Error: No training samples found. Exiting.")  
    exit()  

# 保存标签文件  
with open('label.pickle', 'wb') as f:  
    pickle.dump(label_ids, f)  

# 训练并保存模型  
recognizer.train(x_train, np.array(y_labels))  
recognizer.save('mytrainer.xml')  

print("Training complete and model saved.")
```





## 运行

```python
#-----------------------------运行-----------------------------
import cv2  
import numpy as np  
import pickle  
import os  

cap = cv2.VideoCapture(0)  # 第一个摄像头  
face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_alt2.xml')  
recognizer = cv2.face.LBPHFaceRecognizer_create()  

# 检查并加载模型文件  
model_path = 'mytrainer.xml'  
if not os.path.exists(model_path):  
    print(f"Model file '{model_path}' not found.")  
else:  
    recognizer.read(model_path)  

# 加载标签  
labels = {}  
with open('label.pickle', 'rb') as f:  
    origin_labels = pickle.load(f)  # {'yoyo':5,...}  
    labels = {v: k for k, v in origin_labels.items()}  

while True:  
    ret, frame = cap.read()  
    if ret:  
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)  
        faces = face_cascade.detectMultiScale(gray, scaleFactor=1.5, minNeighbors=5)  

        if len(faces) > 0:  
            for (x, y, w, h) in faces:  
                gray_roi = gray[y:y + h, x:x + w]  
                id_, conf = recognizer.predict(gray_roi)  
                if conf > 60:  # 修改为置信度低于60时绘制框和标签  
                    # 打印调试信息，只有在检测到人脸时  
                    print(f"Detected {len(faces)} faces")  
                    print(f"Confidence: {conf}, ID: {id_}")  
                    cv2.rectangle(frame, (x, y), (x + w, y + h), (255, 0, 0), 2)  
                    cv2.putText(frame, str(labels[id_]), (x, y - 15), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 0, 0), 2)  

        cv2.imshow("Result", frame)  

        # 等待用户按 'q' 键退出  
        if cv2.waitKey(1) & 0xFF == ord('q'):  
            break  

cap.release()  
cv2.destroyAllWindows()
```