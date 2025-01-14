> 原文：https://rtklibexplorer.wordpress.com/2016/11/13/demo5-code-features/

[TOC]

## 一、定位结果解算 (RTKNAVI, RTKPOST, RNX2RTKP)

### 1、明显的功能差异

#### 1. 重写Kalman滤波时间更新

卡尔曼滤波器的状态更新已被重新编写，以删除所有的零元素乘法。这大大减少了启用接收器动态时的计算时间。

#### 2. GLONASS、SBAS频间偏差

增加了代码，以校正和管理GLONASS和SBAS卫星的频间偏差，其依据是固定和保持 fix-and-hold  的扩展（从模糊度解算结果直接反馈到卡尔曼滤波器状态）。 这样就可以使用 GLONASS 和 SBAS 卫星与 M8N 接收机一起解算模糊度，或者在基准站和流动站接收机不一致时使用。

#### 3. 整周模糊度解算

整周模糊度解算已得到加强，包括每个历元最多三次尝试，使用不同的卫星组合解决歧义。额外的尝试可能会移除 GLONASS 和 SBAS 卫星和/或移除新获得的卫星，取决于各种条件。

#### 4. 整周模糊度约束

增加了用于整周模糊度解算的可调整约束条件，以帮助尽量减少错误修复的机会。

#### 5. 相位偏差

相位偏差的公共部分已被分散一个单独的变量中，而不是在所有卫星中分散开来。这使得相位偏差状态在调试过程中更容易解释，也消除了一些来自不同载波波长的卫星的周期的不当添加问题。

#### 6. 核心算法加注释

大多数核心定位算法代码都增加了注释。

#### 7. Trace

Trace 输出得到了加强和修改，为调试不良解决方案提供了更多的相关信息。

### 2、增加输入参数和选项

#### 1. pos1-posmode = static-start

以 "静态 "模式开始求解，但在第一次定点后切换到 "运动学 "模式。如果漫游器是静止的，这种模式通常会比运动模式更快地获得第一个定点，但如果漫游器在第一个定点之前开始移动，则会失败。

#### 2. pos2-gloarmode = fix-and-hold

扩展了 fix-and-hold 方法，以校准 GLONASS 卫星的信道间偏差和 SBAS 卫星的类似误差。请注意，固定和保持反馈只发生在第一次固定之后。

#### 3. pos2-arfilter = on,off

如果新的或恢复的卫星因其加入而大大降低了 AR-Ratio 值，则拒绝将其用于模糊性解算。这是对使用 "arlockcnt "的盲目延迟的一种替代或增强，因为卫星将只在必要时被排除在解之外，而不是在一个固定的时间长度。

#### 4. pos2-arthres1 = x

整周模糊度的解算被推迟到位置状态的方差达到这个阈值。它的目的是避免在卡尔曼滤波器收敛之前进行错误的固定。"arthres1 "选项存在于发布代码的配置文件中，但没有任何用途。

#### 5. pos2-minfixsats = n

获得固定解所需的最小卫星数。用来避免从极少的卫星中获得错误的定位，特别是在频繁的周跳期间。

#### 6.  pos2-minholdsats = n

保持一个整周模糊度解所需的最小卫星数。用于避免来自极少数卫星的错误保持，特别是在频繁的周跳期间。

## 二、原始数据到 RINEX 转换 (RTKCONV, CONVBIN, RTKNAVI)

### 1、明显的功能差异

#### 1. 周跳

M8T原始输出到周跳的转换算法已被修改，以更接近于 M8N 的算法。这保证了每次失锁后都会标记周跳，直到载波相位测量质量足以重新设置相位偏差估计时才会标记。

#### 2. 半周期无效位

半周期无效位（half-cycle invalid bit ）是根据锁定时间为 SBAS 卫星设置的，因为接收器的半周期无效位对这些卫星不起作用。在 Demo5 代码中，对 M8N 和 M8T 都做了这一修改，但对 M8N 接收机只移植回了 2.4.3 代码。

#### 3. 观测质量

每个采样的接收机观测质量指标都记录在 RINEX 观测文件中。伪距和载波相位测量中的单字符 SNR 字段被用于此目的。M8T 报告伪距和载波相位的质量指标，M8N 只报告载波相位的质量指标。这仅仅是为了提供信息，RTKLIB 并不使用这些字段。

### 2、接收机特殊选项

#### -STD_SLIP = x

如果载波相位测量的标准差（由接收器报告）大于或等于 x（比例=0.04米/计数），则载波相位测量被标记为周跳。这个功能现在在 2.4.3 和 demo5 代码中都存在，但在 2.4.3 的文档中没有列出。它只用于 M8T 接收机。如果不设置这个选项，那么周跳将使用与有效载波相位测量相同的固定阈值，在发布的代码中，这可能导致周跳在相位偏移估计有效后不能被标记出来。