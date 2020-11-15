### Java SPI

API是站在应用的角度定义了功能如何实现，SPI是系统为第三方专门开发的扩展规范以及动态加载扩展点的机制。下图反映了API和SPI之间的不同：
![API和SPI](../images/apiandspi.png "API和SPI")

当作为服务提供方利用SPI机制时，需要遵循SPI的约定：
  * 先编写好服务接口的实现类，即服务提供类;
  * 在classpath的META-INF/services目录下创建一个以接口全限定名命名的UTF-8文本文件，并在该文件中希尔实现类的全限定名(多个实现类以换行符分隔);
  * 调用JDK中的java.util.ServiceLoader组件中的load()方法，根据上述文件发现并加载具体的服务实现;
  
