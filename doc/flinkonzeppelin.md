### Flink on Zeppelin

首先需要安装zeppelin。下载zeppelin，解压缩，将conf文件夹下的zeppelin-env.sh.template重命名为zeppelin-env.sh，并增加配置
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home。
然后即可启动zeppelin。启动命令为bin/zeppelin-daemon.sh start，zeppelin默认启动地址是127.0.0.1，也就是本机，这样就只能在启动
zeppelin的机器上访问，若想在所有机器上都能访问，可将zeppelin.server.addr设置为0.0.0.0，默认端口是8080，因此在zeppelin启动后即
可在浏览器输入127.0.0.1:8080访问zeppelin。
