
激活环境 conda activate yolov8
cd c:/Users/Administrator/Desktop/ultralytics-main  切换目录
from ultralytics import YOLO
从ultralytics里面导入YOLO这个类
yolo=YOLO("./yolov8n.pt",task="detect")
实例化优化yolo这个对象，里面第一个参数是要加载或使用的模型，task参数可省略
result=yolo(source="./ultralytics/assets/bus.jpg")
yolo task=detect mode=predict model=./yolov8n.pt source="./ultralytics/assets/bus.jpg"
yolo相当于启动命令  detect目标检测任务   mode这个参数就是您要它进行预测还是训练还是验证
model就是使用哪个模型来进行预测 source就是说我们对谁进行检测
conf这个参数的值越小，框出来的框框越多（iou参数结果与之相反）
（在vscode里面按CTRL+~ 可以打开终端
