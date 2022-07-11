# Xiangqi



![UI](https://github.com/splendor-kill/xiangqi/raw/main/env.render.png)



## How to use

```python
from xiangqi import Env

env = Env(opening)  # 不传开局状态表示从头下起

ob = env.reset()  # 初始观察
while True:
    env.render()  # 显示棋局
    player = # which agent are responsible for ob['cur_player']?
    action = player.make_decision(**ob)  # agent 的决策
    ob, reward, done, info = env.step(action)
    if done:
        env.render()
        break
```



## Design decisions

### observation

以当前玩家的视角观察，包括坐标系、行、列，以及当前的合法走法，上一步被对手吃掉的子(用于reward shaping)



### action

* ~~方案1~~

  以当前玩家的视角行动，action space: `MultiDiscrete(9, 10, 9, 10)`

  前面2维表示源位置`(col, row)`，后面2维表示目标位置

  空间大小8100

* 方案2
  任一位置的行，列，四方2x2范围的位置，空间大小2550

* 方案3

  对每一类子，从右到左，从上到下编号，然后针对棋子类别编码最大可能的走法

  车炮：17，马：8，象：4，士：4，兵：3，将：4

  空间大小119

**注意**：

* 策略应该包含对对手求和请求的处理，回答是否同意和





## Details

默认执红者先下

观察到的信息包括：

* `board_state`：棋盘状态，[FEN串](https://www.xqbase.com/protocol/cchess_fen.htm)，它包含了局面的所有信息
* `sue_draw`: 是否对手在求和，**如果是，需要对求和做出反应**，即是否同意和局



操作环境的输入(即`env.step(action)`中的`action`)为一个字符串：

```python
"RESIGN": 认输，认输会引起本局结束
"SUE_DRAW": 求和，主动求和只要发这个字符串，对手需要回复"yes"或"no"表示同意与否
UCCI走法串: 如"b7b0"，即源位置和目标位置的编码

```

操作环境的输出(即`env.step(action)`的返回值)分为四个部分：

* `ob` (`dict`) 观察，切换当前玩家后(即下一个玩家)的观察，具体描述参看上面，`done`为`True`时为`None`
* `reward` (`int`) 回报，当前玩家(即做本次action的玩家)的回报，`done`为`False`时为`None`
* `done` (`bool`)，这一局是否结束
* `info` (`str`)，一些附加信息



显示棋局`env.render()`：

字符界面下显示的棋局。棋盘不以玩家视角变化的，而是固定红色在下，黑色在上。

棋子在颜色上做了区分，在字形上也做了区分，红色的感觉更带人性一些，黑方有点像原始文明

另外"将军"，吃子等信息也会显示



常量

```python
N_ROWS = 10  # term: rank
N_COLS = 9  # term: file

REWARD_WIN = 1
REWARD_LOSE = -1
REWARD_DRAW = 0
REWARD_ILLEGAL = -5  # illegal action

class Action(IntEnum):
    ADVANCE = 1  # 进
    RETREAT = 2  # 退
    TRAVERSE = 3 # 平
    SUE_DRAW = 4 # 求和
    RESIGN = 5   # 认输

class Camp(IntEnum):
    BLACK = 1
    RED = 2

class Force(IntEnum):
    SHUAI = 1
    SHI = 2
    XIANG = 3
    MA = 4
    JU = 5
    PAO = 6
    BING = 7
```

帮助函数

```python
def decode(board_state)  # 解码棋盘状态，整数->(Camp, Force)列表(从左到右，从上到下)
def chinese_to_ucci(action, camp, board)  # 中文纵线格式 -> ucci格式，如炮二平五->h7e7
def ucci_to_chinese(action)  # 上面函数的逆
```



## reference

* [Xiangqi](https://en.wikipedia.org/wiki/Xiangqi)

* [中式记法](https://zh.wikipedia.org/wiki/%E8%B1%A1%E6%A3%8B) [西式记法](http://wxf.ca/xq/computer/wxf_notation.html) [着法表示](https://www.xqbase.com/protocol/cchess_move.htm)

* [术语](http://wxf.ca/xq/computer/XIANGQI_TERMS_IN_ENGLISH.pdf)

* [UCCI](https://www.xqbase.com/protocol/cchess_ucci.htm) [UCI](https://gist.github.com/aliostad/f4470274f39d29b788c1b09519e67372)

* [Elo计算](https://www.xqbase.com/protocol/elostat.htm)

* [源码](https://github.com/Zeta36/chess-alpha-zero.git)

  

