**Ros2 初次使用**

```cpp
#include "rclcpp/rclcpp.hpp"

int mian(int argc, char **argv){
    //初始化客户端
    rclcpp::init(argc,argv);
    //创建节点指针
    auto node = rclcpp::Node::make_shared("helloworld_node_cpp")  //返回Node节点的指针
    //输出日志
    RCLCPP_INFO(node->get_logger(),"helloworld!");
    rclcpp::shutdown();
    
}



```

