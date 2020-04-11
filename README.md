## cocos-bezier贝塞尔曲线

#### 简介
**贝塞尔曲线**（Bézier curve）又被称为**贝兹曲线**或**贝济埃曲线**，是应用于二维图形应用程序的数学曲线，它的数学基础是伯恩斯坦多项式（Bernstein polynomial，since 1912），1959年法国数学家Paul de Casteljau提出了数值稳定的de Casteljau算法，开始贝塞尔曲线的图形化应用研究，而贝塞尔曲线的名称来源于一位就职于雷诺的法国工程师Pierre Bézier，他在1962年开始对贝塞尔曲线做了广泛的宣传，他使用这种只需要很少的控制点就能生成复杂平滑曲线的方法来进行汽车车体的工业设计。
贝塞尔曲线因为它控制简便却具有极强的描述能力，迅速在工业设计和计算机图形学等相关领域得到了广泛应用。比如在矢量绘图中，贝塞尔曲线用来给需要无限制地缩放的平滑曲线定模，许多绘图软件都提供了绘制贝塞尔曲线的功能。贝塞尔曲线还用于动画时间控制以实现美观逼真的缓动效果，还用于机器人转动手臂等方面的设计。

对于贝塞尔曲线来说，最重要的点是，数据点和控制点。
**数据点**： 指一条路径的起始点和终止点。
**控制点**：控制点决定了一条路径的弯曲轨迹，根据控制点的个数，贝塞尔曲线被分为一阶贝塞尔曲线（0个控制点）、二阶贝塞尔曲线（1个控制点）、三阶贝塞尔曲线（2个控制点）等等。从1-n阶的连续函数，都可以计算得到一条光滑曲线。

#### 贝塞尔曲线展示
以下公式中：**B(t)**为**t**时间下点的坐标，**P0**为起点，**Pn**为终点，**Pi**为控制点


##### 一阶贝塞尔曲线
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040913303664.jpg)

![一阶贝塞尔曲线](https://img-blog.csdnimg.cn/20200409113341580.gif#pic_center)

##### 二阶贝塞尔曲线
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409133202610.jpg#pic_center)

![二阶贝塞尔曲线](https://img-blog.csdnimg.cn/20200409113453811.gif#pic_center)

##### 三阶贝塞尔曲线
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409133318252.jpg#pic_center)

![三阶贝塞尔曲线](https://img-blog.csdnimg.cn/20200409113515506.gif#pic_center)

##### 四阶贝塞尔曲线

![四阶贝塞尔曲线](https://img-blog.csdnimg.cn/20200409113538469.gif#pic_center)

##### 五阶贝塞尔曲线

![五阶贝塞尔曲线](https://img-blog.csdnimg.cn/20200409123516293.gif#pic_center)

通用公式：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409133359547.jpg#pic_center)

#### cocos引擎中的Bezier
cocos-2dx引擎中自带的是**三阶**贝塞尔曲线，需要两个控制点，可以通过让**两个控制点坐标相同**，转化成二阶，即抛物线。
推荐一个网站，可以模拟和生成多种样式的曲线[在线生成贝塞尔曲线(英文)](https://cubic-bezier.com/)或[在线生成贝塞尔曲线(中文)](http://yisibl.github.io/cubic-bezier/)
#####  C++函数原型
```cpp
//结构体
typedef struct _ccBezierConfig {
    Vec2 endPosition; // end position of the bezier
    Vec2 controlPoint_1; // Bezier control point 1
    Vec2 controlPoint_2; // Bezier control point 2
} ccBezierConfig; 

// BezierTo创建函数
static BezierTo* create(float t, const ccBezierConfig& c);
```

#####  参数解读
endPosition：曲线的终点
controlPoint_1： 曲线起点的控制点(控制第一个波峰或波谷)
controlPoint_2：曲线终点的控制点(控制第二个波峰或波谷)
当 controlPoint_1 == controlPoint_2 时，曲线只有一个波峰或波谷
**注意：lua中调用的时候，参数的顺序发生了变化，下面截取了部分代码**
```javascript
ccBezierConfig config;
config.controlPoint_1 = arr[0];
config.controlPoint_2 = arr[1];
config.endPosition = arr[2];
CC_SAFE_DELETE_ARRAY(arr);

BezierTo* tolua_ret = BezierTo::create(t, config);
if (NULL != tolua_ret)
{
    int nID = (tolua_ret) ? (int)tolua_ret->_ID : -1;
    int* pLuaID = (tolua_ret) ? &tolua_ret->_luaID : NULL;
    toluafix_pushusertype_ccobject(tolua_S, nID, pLuaID, (void*)tolua_ret,"cc.BezierTo");
    return 1;
}
```
所以这里的顺序是 controlPoint_1，controlPoint_2，endPosition

##### 实例
**要求**：模拟一个射箭的过程，路径是抛物线，运动过程中箭头的方向要与曲线运动方向一致。
**思路**：利用贝塞尔曲线让箭头运动起来，运动的同时每一帧计算曲线斜率，斜率转换成角度，并设置箭头的旋转角度即可。
###### lua调用

```javascript
-- 创建箭头sprite
local target = display.newSprite("ui_jiantou.png")
target:addTo(self)

-- 一些坐标
local originP = cc.p(100, 100) -- 起点
local controlP1 = cc.p(550, 500) -- 控制点1
local controlP2 = cc.p(550, 500) -- 控制点2
local endP = cc.p(1000, 100) -- 终点
local bezierPos = {controlP1, controlP2, endP}

-- 初始化
target:setPosition(originP)
target:setRotation(-30) -- 大概给一个起始角度，注意：setRotation()参数是角度值，正值表示顺时针旋转，与我们斜率的正负值相反

local angle -- 箭头角度值
local lastP = originP -- 上一帧点坐标
local scheduler = require("cocos.framework.scheduler") -- 全局计时器
local action1 = cc.CallFunc:create(function() 
    self.handler = scheduler.scheduleGlobal(function() -- 注册全局计时器
        local curP = cc.p(target:getPosition()) -- 当前帧坐标
        local radian = cc.pToAngleSelf((cc.pSub(curP, lastP))) --向量夹角弧度
        angle = radian * 180 / math.pi -- 弧度值转换为角度 公式：弧度＝度*π/180
        target:setRotation(-angle)
        lastP = curP
    end, 0) 
end)
local action2 = cc.BezierTo:create(5, bezierPos) -- 自带的为三阶贝塞尔，可以让control1等于control2变为二阶，实现抛物线
local action3 = cc.CallFunc:create(function() 
    if self.handler then 
        scheduler.unscheduleGlobal(self.handler) -- 注销全局计时器
        target:stopAllActions()
    end 
end)
-- 动作1动作2同时运行结束后，再执行动作3
local action = transition.sequence({cc.Spawn:create({action1, action2}), action3})
target:runAction(action)
```
###### 辅助画线
为了更直观准确的观测运动轨迹，我们可以把轨迹画出来
引擎中DrawNode类，可以画出各种形状
C++源码
```cpp
/** Draw a cubic bezier curve with color and number of segments
*
* @param origin The origin of the bezier path.
* @param control1 The first control of the bezier path.
* @param control2 The second control of the bezier path.
* @param destination The destination of the bezier path.
* @param segments The number of segments.
* @param color Set the cubic bezier color.
*/
void drawCubicBezier(const Vec2 &origin, const Vec2 &control1, const Vec2 &control2, const Vec2 &destination, unsigned int segments, const Color4F &color);
```
lua调用
```javascript
-- 画线 贝塞尔曲线
local myDrawNode = cc.DrawNode:create() 
myDrawNode:drawCubicBezier(originP, controlP1, controlP2, endP, 100, cc.c4f(0,0,1.0,1))
myDrawNode:addTo(self, 100)

-- 画点
local myPoint = cc.DrawNode:create() 
myPoint:drawPoints({originP, controlP1, controlP2, endP}, 4, 10, cc.c4f(1,0,0,1)) 
myPoint:addTo(self, 100)
```

##### 运行展示
图中，红点依次是 起点、控制点1（与控制点2重合）、终点

![demo](https://img-blog.csdnimg.cn/20200409125724188.gif#pic_center)
