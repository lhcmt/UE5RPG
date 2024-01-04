# UE5RPG 一个类似刺客信条的Demo
学点3C
课程链接https://www.youtube.com/watch?v=FNTyIWkv5k8&list=PLiSlOaRBfgkcPAhYpGps16PT_9f28amXi&index=2
更新日志：

#### 第1、2章
功能走跑蹲、转向倾斜叠加，非常简单的动画蓝图，从角色蓝图读取变量，比较简单
复杂的建议直接看ALS

#### 第3章 翻越功能
- 我这边用了F键，翻墙键按下会从角色腰部开始，向角色正前方间隔一定高度(30cm)尝试打3次1.5米长的球形探测，以寻找前方是否存在可以翻越的墙体。
- 如果命中了墙体，则从命中点开始，抬高1米，间隔0.5米从上向下打球形探测，直到无法探测到平台停止，以判断翻越墙体的厚度（Demo这里最大探测5次，可以游戏视情况而定）
- 探测出墙体厚度后，从最后一个垂直探测位置（上一步没有碰撞的球形探测）再打一根很深的射线，以检测落地的位置
- 最终得到3个坐标
  - 翻墙起点（第一个碰撞点）
  - 翻墙厚度（第二个碰撞点）
  - 落地位置（射线探测点）
- 最后将3个坐标传给角色的MotionWarping组件，AddOrUpdateWarpTarget
- 翻越动画是一个完整的动画序列，创建成montage以后，在蒙太奇上用MotionWarp的动画通知打3个位置（翻墙起跳，跨越，落地）

感觉这里处理的比较粗糙，比如多块板子堆叠的时候肯会穿模。过于靠近可翻越墙体的时候起跳动画也会略有穿模，需要调整下。另外可以把跳跃和翻越做到一个按键（space）上

#### 第四章 背刺动画
- 同样用到了MotionWarp，这次是背刺攻击的前半段，用MotionWarp吸附到背刺对象的背后指定位置
- 新增两个动画，小刀背刺攻击、被背刺，制作成蒙太奇，EnableRootMotion。
- 指定位置通过背刺对象蓝图上的一个固定场景组将来获取
- 背刺范围判定：非常简便在背刺对象背后挂个Overlap的碰撞体，进入就可以背刺（按钮的时候判断）
- 背刺单位倒地后开启SimulatePhysics，可以变成布娃娃，没什么好说的
- 这里背刺的时候角色并没有停止接收移动输入，可能后续会改

#### 第五、六章 血量体力属性、UI、冲刺、体力衰减与恢复
- 比较简单的Gameplay逻辑，自定义的ActorComponent组件BP_PlayerStats，管理HP和体力，经验值等
- 视频中在角色中创建UI并缓存，做法比较临时，并没有完全按照视频中的做，不过问题不大


#### 第三十、三十一章 攀爬（先做移动相关
- 攀爬控制和攀爬动画，略微做的和视频不一样，在速度计算和下爬动画上更好一点，视频没做手部IK所以比较粗糙，在斜坡切换的时候也没有处理好和墙壁的贴合问题，后续修复


#### 第三十二章 攀爬到顶部的动画
- 依然是打射线探测的方式，然后使用MotionWarp来处理攀爬到顶部的动作，需要注意的是，引擎升级到了5.3，因为之前5.0版本中的MotionWarp组件，并不能支持不带根运动的蒙太奇使用MotionWarp。


#### 新增状态管理组件BP_StateComponent，2023.12.26 
- 使用状态关系表，来进行角色状态的管理，之前老项目中就采用了这种方式，能比较好的处理第三人称角色状态比较复杂的情况。
- 状态之间关系分为，共存、打断、静止
- 组件存贮一个状态数组，表示角色拥有的所有共存状态，当新状态想添加的时候，会查询表格，判断是否可以进入此新状态,以及如果进入新状态，会结束掉哪些旧状态。
- 举个例子，跳跃会取消下蹲、但是在翻越过程中，跳跃则不能进入


#### 视频内容第二十五章 FootIK和下蹲胶囊体处理
- FootIK直接使用了UE提供的CR_Mannequin_BasicFootIK ControlRig，具体没细看
- 下蹲胶囊体缩减半高，调整Mesh相对位置。增加蹲起限高判定。以及跳跃解除下蹲时候，也会进行起跳判定。同时修改了StateMap的中跳跃打断下蹲的关系，由角色蓝图中判断是否需要打断下蹲
- 调整了跳跃下蹲动画融合
- 遇到了一个蓝图离奇BUG，同一个函数会执行2次，可能是CopyPase导致的，删除节点重新创建就好了，很诡异