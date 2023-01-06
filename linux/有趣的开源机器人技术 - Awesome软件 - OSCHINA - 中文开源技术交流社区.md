许可证：BSD 官网：https://www.ros.org/ ROS (Robot Operating System, 机器人操作系统) 提供一系列程序库和工具以帮助软件开发者创建机器人应用软件。它提供了硬件抽象、设备驱动、库函数、可视化、消息传递和软件包管理等诸多功能。 ROS 目前只能在基于 Unix 的平台上运行。ROS 的软件主要在 Ubuntu 和 Mac OS X 系统上测试，同时 ROS 社区仍持续支持 Fedora，Gentoo，Arch Linux 和其它 Linux 平台。 与此同时，Microsoft Windows 端口的 ROS 已经实现，但并未完全开发完成。

![ROS 机器人操作系统](https://static.oschina.net/uploads/logo/ros_Uxcfc.png)

许可证：MIT 开发语言：Python 官网：https://pyrobot.org/ PyRobot 是由 Facebook 和卡内基梅隆大学共同开发的机器人框架，可以运行由 PyTorch 训练的深度学习模型。在设计上，PyRobot 框架旨在使 AI 研究者和学生在不具备设备驱动程序、控制或规划专业知识的情况下，在几小时内操控机器人工作。 PyRobot 是基于机器人操作系统 ROS 上的轻量级平台，提供了一组无关硬件的 API，供开发人员控制各种的机器人。PyRobot 抽象了底层控制器与程序之间沟通的细节，因此对于 AI 研究人员来说，可以不再需要理解机器人的底层操作，能够专注地开发上层 AI 机器人应用程序。

![PyRobot AI 机器人框架](https://static.oschina.net/uploads/logo/pyrobot_45xm2.png)

开发语言：C/C++ 官网：https://gazebosim.org/home Gazebo 是一个三维多机器人动力学仿真。它能够模拟复杂和现实的环境中关节型机器人。 Gazebo 为仿真带来了一种全新的方法，它具有完整的开发库和云服务工具箱，使仿真变得容易。在具有高保真传感器流的真实环境中快速迭代你的新物理设计。在安全方面测试控制策略，并在持续集成测试中利用模拟。

![Gazebo 三维多机器人动力学仿真](https://static.oschina.net/uploads/logo/gazebo_lH0tW.png)

许可证：Apache-2.0 开发语言：C/C++ 官网：https://pyrobot.org/ RobWork 是用于模拟和控制机器人系统的 C++ 库的集合，用于研究和教育以及实际的机器人应用。 特点包括： 各种类型的工业机械手、串行、树形和并行结构的运动学建模。 通用路径规划、路径优化、碰撞检测和逆运动学算法。 机械手、控制器和传感器的运动学和动态仿真。 用于模拟吸盘、平行和灵巧夹具的插件。 用于集成用户算法和可视化的简单且可扩展的 GUI 和插件系统。 基于 SWIG 的脚本接口，可将 RobWork 扩展到多种脚本语言，例如 Python、Lua 和 Java。

![RobWork C++ 机器人库](https://static.oschina.net/uploads/logo/robwork_D3iuk.png)

开发语言：C/C++、Python 官网：https://www.eng.yale.edu/grablab/openhand/ Yale OpenHand Project 是一个机械臂的开源设计方案，通过快速原型技术，以鼓励更多的变化和创新机械硬件。商业的机械臂一般价格非常昂贵，专为某些平台所设计，难以修改。 该项目的目的是提供一系列开源设计，并通过社区的贡献来完善设计，并提供大量基于该平台的修改和变种。

![OpenHand 开源机械臂](https://static.oschina.net/uploads/logo/openhand_kTc8w.png)

许可证：Apache-2.0、LGPL-3.0 开发语言：C/C++、Python、JavaScript 官网：http://openrave.org/ Webots 是用于模拟机器人的开放源代码和多平台桌面应用程序。它提供了一个完整的开发环境来对机器人进行建模，编程和仿真。 它被设计用于专业用途，并且广泛用于工业，教育和研究。自 1998 年以来，Cyberbotics Ltd. 一直将 Webots 作为其主要产品。 Webots 提供了一个完整的开发环境来对机器人、车辆和机械系统进行建模、编程和仿真。

开发语言：C/C++ Dummy-Robot 是开源的机械臂机器人项目，可以用来做一些人手无法做到的操作，比如软件去抖、运动范围的重映射、力矩强增强等等。

许可证：GPL-3.0 开发语言：C/C++、C# ElectronBot 是一个桌面级小机器工具人，外观设计的灵感来源是 WALL-E 里面的 EVE。 机器人具备 USB 通信显示画面功能，具备 6 个自由度（手部 roll、pitch，颈部，腰部各一个），使用自己修改的特制舵机支持关节角度回传。 按照作者的原话，EVE 是一个能动的电脑配件，通过主机进行供电，可做各种有意思的事情：自定义动作展示（可编舞）、手势响应、“量子纠缠” 两人视频对话、电脑控制器等等。

许可证：Apache-2.0、LGPL-3.0 开发语言：C/C++、Python 官网：http://openrave.org/ OpenRAVE 为在现实世界的机器人应用程序中测试、开发和部署运动规划算法提供了一个环境。主要关注与运动规划相关的运动学和几何信息的模拟和分析。OpenRAVE 的独立性质允许轻松集成到现有的机器人系统中。 它提供了许多命令行工具来与机器人和规划器一起工作，并且运行时核心足够小，可以在控制器和更大的框架内使用。一个重要的目标应用是工业机器人自动化。

![OpenRAVE 机器人系统环境](https://static.oschina.net/uploads/logo/openrave_ipNSI.png)

开发语言：C/C++、JavaScript OpenROV（open-source remotely operated vehicle）是一个开源的可远程操作的低成本远程机器人潜水项目，用于水下勘探和教育。基于 BeagleBone 搭建。该开源的项目包括基于 Arduino 的代码和基于 Node.js 代码，以及整个硬件设计的方案。

![OpenROV 机器人潜水项目](https://static.oschina.net/uploads/logo/openrov_BOg8A.png)

许可证：GPL 开发语言：Java、Python 官网：https://www.arduino.cc/ Arduino 系统虽不是为机器人而生，但却是目前机器人爱好者最热衷的机器人硬件设备。Arduino 的诞生是为了能有一个可以实现快速原型的电子和软件平台。对机器人来说，Arduino 也是一块非常好用的控制系统。不单提供了非常直接和丰富的硬件接口，更重要的是一套非常浅显易懂而且内容丰富的软件库。许多现成的硬件设备都有直接的代码样例，只需要复制和粘贴就可以使用。比如红外传感器，各种类型的超声波，加速度，陀螺仪，编码器。这大大节省了机器人爱好者们的原型开发速度。（百度百科）

![Arduino 开源电子原型平台](https://static.oschina.net/img/logo/arduino.gif)