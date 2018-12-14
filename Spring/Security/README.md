# Spring Security

## 核心组件

### SecurityContextHolder

用于存储安全上下文(security context)的信息。当前的操作的用户是谁，该用户是否已经被认证，他拥有哪些角色权限...这些都被保存在 **SecurityContextHolder** 中。**SecurityContextHolder** 默认使用 **ThreadLocal** 策略来存储认证信息(这是一种与线程绑定的策略)。Spring Security 在用户登录的时候自动绑定认证信息到当前线程，在用户退出时，自动清除当前线程的认证信息。但这一切的前提是，在web场景下使用Spring Security，而如果是Swing界面，Spring页提供了支持，**SecurityContextHolder** 的策略则需要被替换。

#### 获取当前用户的信息

因为身份信息是和线程绑定的，所以可以在程序的任何地方使用静态方法获取用户信息。一个典型的获取当前登录用户的姓名的例子如下所示
