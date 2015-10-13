# 使用到的linux指令汇总

* 修改密码  
  passwd username  

* ssh远程登陆配置  
  在 /etc/ssh/sshd_config中进行[配置](http://www.jinbuguo.com/openssh/sshd_config.html)

* linux添加环境变量env

  1、在/etc/profile文件中添加变量【对所有用户生效（永久的）】

  2、在用户目录下的.bash_profile文件中增加变量【对单一用户生效（永久的）】

  3、直接运行export命令定义变量【只对当前shell（BASH）有效（临时的）】

* mac 配置jdk环境变量

  修改 ~/.bash_profile
  
  export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home #jdk安装路径   
  
  export PATH=$JAVA_HOME/bin:$PATH 
  
  export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
source ~/.bash_profile

* 查看当前时间信息date命令
  
  向date命令传递参数适用‘+‘（加号），在传递的参数中（%Y表示年 %m表示月 %d表示天 %H表示小时（表示的时间是00-23）%M表示分钟 %S表示秒 %s（表示unix时间戳的秒数）

  如：date +'%Y-%m-%d %H:%M:%S' date +%s

* 查看端口占用进程号

  lsof -i:8080
