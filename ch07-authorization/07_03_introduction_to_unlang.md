# unlang简介
FreeRADIUS中提供的unlang语言将授权的灵活性提升到了新的高度。 Unlang不是一种完整的编程语言，而是一种处理语言。

unlang的目的是实现策略而不是替换使用Perl或Python创建的复杂脚本。 Unlang坚持使用基本语法，包括条件语句和变量操作。 unlang代码不会被编译，但会被FreeRADIUS服务器解释。当服务器读取配置文件时会发生解释，这通常在启动期间发生。 unlang的使用仅限于配置文件中指定的部分，不能在模块内部使用。

unlang的一个关键特性是能够使用条件语句来控制处理请求的进程。
>FreeRADIUS安装unlang的手册页，您可以参考：
>
>$>man unlang

我们将演示各种条件语句的使用，以显示如何处理请求。

## 使用条件语句
条件语句很简单，但功能强大，它们仍然是任何软件的构建块。 Unlang提供了两种实现条件检查的方法：

>if语句。这包括else和elsif选项作为声明的一部分。
>
>switch语句。

本节将介绍if语句的各种用法。