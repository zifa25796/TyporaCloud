# Class

> 在该文档中写的所有变量以及函数都是这些 Class 必备的
>
> 后面极有可能会加上其他的 Helper Method

## Finger（树莓派）

- 每一根手指落在屏幕上都会生成一个 Finger Object
- 变量：
	- ID（手指从 int：0 开始记录）
	- 手指落在屏幕上的位置
	- 手指抬起的位置
	- 手指落下的时间（绝对时间戳）
	- 手指抬起的时间
- 函数：
	- OnFingerUp
		- 记录手指抬起的时间
	- CheckClick
		- 负责检测当前手势是否符合一个点击
		- 去获得一下现在的时间戳
		- 人机交互优化：
			- 决定多长时间（从落下到抬起）算作一个点击（目前200ms）
	- CheckGesture（Flag）
		- 负责检查当前的手指移动距离是否符合一个手势
		- 去获得手指抬起的位置（Up：y_final - y_init …..）
		- 人机交互优化：
			- 决定多长时间（从落下到抬起）算作一个点击（目前20px）
		- 在写这个函数的时候，需要考虑写4份，分为上下左右
	- BestPointer（可选）
		- 辅助 CheckGesture
		- 用来判断手指朝着哪个方向移动

## Event（PC端/残缺版树莓派）

- 每一个手指在屏幕（陀螺仪）上进行相应的操作以后，会生成一个 TouchEvent Object
	- ACTION_DOWN
	- ACTION_UP
	- ACTION_MOVE
	- ACTION_LEFT_CLICK
	- ACTION_RIGHT_CLICK
	- ACTION_SCROLL
	- ACTION_THREE_UP
	- ACTION_THREE_DOWN
	- ACTION_THREE_LEFT
	- ACTION_THREE_RIGHT
	- ACTION_GYRO_MOVE
- 每一个EVENT都是一个动作当中的一段动作
	
	- DOWN，UP是比较独特的，因为这两个EVENT不会产生相应的Trigger Event
	
	- 当 DOWN，UP 组合起来以后，会产生一个LEFT_CLICK
	
	- 这个时候，进行Trigger Event，相当于说发送一个指令给PC端口，包含一段数据
	
		- ```json
			{
			    EVENT: ACTION_LEFT_CLICK,
			    POS: {
			        x: 0,
			        y: 0
			    }
			}
			```
	
	- MOVE EVENT数据包
	
		- ```json
			{
			    EVENT: ACTION_MOVE
			    POS: {
			        x: 0,
			        y: 0
			    }
			}
			```
	
		- 虽然这个数据包看上去和上一个LEFT CLICK看上去一样，但是在PC端口因为识别到了不同的EVENT
	
		- 会进行不同的处理
	
		- 例：
	
			- Click只是单纯的执行了一个点击效果
	
			- Move会根据上一次记录鼠标的位置，以及这一次新发过去的位置来进行一个距离差判断
	
			- 虽然说这部分可以放在PC端口处理，但是为了EVENT统一性，我们将跟新MOVE EVENT的数据包为
	
			- ```json
				{
				    EVENT: ACTION_MOVE
				    DISTANCE: {
				        x: 0,
				        y: 0
				    }
				}
				```
	
			- 即，意味着，PC端将移动鼠标x距离，y距离
	
			- 因为MOVE EVENT可以非常平凡的出发，只要计算判定为一个手势而不是点击，就会生成一个MOVE
	
			- 所以每一个move数据包很有可能只是移动了1px
	
- 给每一个EVENT匹配一个相应的ID
  - 要么，全局变量（建议：另外开一个文件来专用定义ID，.h）

  	- ```c++
  		#define ID
  		
  		#define ACTION_DOWN 10
  		```
  	- ```c++
  		#include id.h
  		
  		int a = ACTION_DOWN
  		```
  - 
  - 要么，局部Class变量，但是唯一实例
  - 最好有一定规律

- 变量：

  - EVENT ID（这个EVENT到底是哪个）

- 函数

  - TriggerEvent（具体实现形式不一定要是一个函数，可以是class，可以是体外函数）
  	- 我们执行该EVENT
  	- 每个EVENT都有一个自己的函数，例如使用 switch(event_id)
  	
  		- ```c++
  			switch(event_id) {
  			    case 1:
  			        continue;
  			    case 2:
  			        continue;
  			    default:
  			        continue;
  			}
  			```
  		- 
  	- （其他做法，为每一个event写一个class，但是event太多了不太合适）
  	
  - Trigger建议：
  	- 在写event函数的时候最好进行一下分类
  	
  	- 例：
  		- DOWN，UP 因为是单根手指动作，可以放在一个函数内判断
  		- MOVE 因为涉及到移动鼠标应该为自己独立的函数
  		- SCROLL 因为涉及到滚轮应该为自己独立的函数
  		- LEFT，RIGHT CLICK因为是点击动作，可以放在一个函数内，但是由于一个是单手指动作，一个是双手指动作，也可也考虑分开存放
  		- THREE 所有三手指函数应该放在一个函数内
  		
  	- ```c++
  		Event mEvent = Event(构造函数，应该有相应的数据传入);
  		mEvent.Trigger()
  		```

## GestureRecognition（树莓派）

- 手势识别

- 当相应的手指数据从安卓端传送至树莓派，该Class应该对其进行判断，然后创建Event的创建
	- 传进来的是Finger Object
	
- 基础逻辑：
	- 首先要进行对现在有多少手指在屏幕上的判断
		- 涉及到，现在在屏幕上的手指数量是不是最终手指数量
		
		- 也就是说，要有个阈值去决定，从第一根手指落下开始，多少秒内落下后续手指才算一个动作
		
		- 以及，是否支持，后续添加手指进行event的改变
		
			- ```c++
				Event.changeID(newID)
				```
	
- 变量：

	- 手指数量
	
- 函数：

  - AddOneFInger
    - 为当前动作多加一个手指进去
    - 重新判断这个是什么手势
    - 手指数量++
    - 触发一个手势的判断
  - RemoveOneFinger
  	- 减去一个手指
  	- 手指数量 - -
  	- 当所有的手指数量为0的时候，我们可以判断，前面这一段时间进行的手势操作已经结束了
  		- 这里针对的是单次手势操作，例如单击，三根手指向上（显示所有窗口）
  			- 牵涉到一个阈值，例如三根手指的手势操作，我们不一定要求整个时间段所有手指都在屏幕上面，可以假设，经过一段时间的三根手指移动，哪怕后面只有两根手指在屏幕上进行前面的操作，也人作为前面的手势还再继续
  		- 另外一部分持续发送的手势操作不受此影响，例如鼠标移动

## QR（可选）

- 通过生成和扫描QC code 去连接端口

## Client（PC）

- 在PC端口上实现TCP数据接收
- 去理解从树莓派上发过来的数据，以及进行相应的自动化操作

## Server（树莓派）

- 在树莓派端口上实现TCP数据发送/接收
	- 对PC的发送
	- 接收安卓的数据
	
- 发送

    - 从树莓派发送一个自动化操作指令给PC端
    - ```json
        {
          EVENT: MOUSE_MOVE,
          DISTANCE: {
            x: 1,
            y: 1
          },
          SPEED: 2.00
        }
        ```
    - 

- 接收

    - 从安卓端口发送一个数据包给树莓派（接收），来进行手势理解

    - ```json
        {
          ID: 1,
          LANDING_POS: {
            x: 1,
            y: 1
          },
          ....RISING...,
          RISING_TIME: ,
          ....TIME..,
          
        }
        ```

    - 请求动作（先不用实现）：

        - 树莓派和安卓上的数据互通应该是请求形式的
        - 因为TCP是一个双向交互（阻塞）的沟通协议
            - A发送一个数据给B，必须在B回应后，A才继续执行后续代码
            - 树莓派发送一个数据请求给安卓，安卓再发送相应的数据给树莓派。
    
- 变量

    - Socket
    - Port

- 函数

    - Connet
        - 连接相关
        
    - SendMessage(string msg)
        - 负责发送数据
        
    - GetMessage
    
        - 负责收取数据
    
        - 需要拆分数据，以建立相关的 Object 以及触发相关的函数
    
            - 因为收到的数据是 string
    
            - 需要类似 split，适当的 cast，最后组成一个 Object
    
                - 例如：
    
                - ```json
                    {
                      ID: 1,
                      LANDING_POS: {
                        x: 1,
                        y: 1
                      },
                      ....RISING...,
                      RISING_TIME: ,
                      ....TIME..,
                      
                    }
                    
                    String: "1,1,1,122020344,...."
                    ```
    
                - ```c++
                    msg.split(",");
                    finger_id = msg[0]
                      ...
                    ```





#### 流程图

1.   手指落下，传输数据，建立 Object
2.   手指进行相关操作，传输数据，更改 Object
3.   手指抬起，传输数据，更改 Object
     1.   因为手指抬起了，我们知道这个手指相关的 手势 结束了
     2.   去触发这个识别到的手势
     3.   在手势执行完成之后，进行一个 Object的删除



#### Utility.c 改成 Interface

