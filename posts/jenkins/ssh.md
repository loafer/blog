### Execute shell script on remote host using ssh  
在配置`Execute shell script on remote host using ssh`执行远程脚本时，如果出现以下异常信息：
```shell
cd /root/pcs15
channel stopped
[SSH] Exception:Algorithm negotiation fail
com.jcraft.jsch.JSchException: Algorithm negotiation fail
	at com.jcraft.jsch.Session.receive_kexinit(Session.java:520)
	at com.jcraft.jsch.Session.connect(Session.java:286)
	at com.jcraft.jsch.Session.connect(Session.java:150)
	at org.jvnet.hudson.plugins.SSHSite.createSession(SSHSite.java:141)
	at org.jvnet.hudson.plugins.SSHSite.executeCommand(SSHSite.java:151)
	at org.jvnet.hudson.plugins.SSHBuilder.perform(SSHBuilder.java:60)
	at hudson.tasks.BuildStepMonitor$1.perform(BuildStepMonitor.java:20)
	at hudson.model.AbstractBuild$AbstractBuildExecution.perform(AbstractBuild.java:779)
	at hudson.maven.MavenModuleSetBuild$MavenModuleSetBuildExecution.build(MavenModuleSetBuild.java:939)
	at hudson.maven.MavenModuleSetBuild$MavenModuleSetBuildExecution.doRun(MavenModuleSetBuild.java:890)
	at hudson.model.AbstractBuild$AbstractBuildExecution.run(AbstractBuild.java:534)
	at hudson.model.Run.execute(Run.java:1728)
	at hudson.maven.MavenModuleSetBuild.run(MavenModuleSetBuild.java:544)
	at hudson.model.ResourceController.execute(ResourceController.java:98)
	at hudson.model.Executor.run(Executor.java:405)
Build step 'Execute shell script on remote host using ssh' marked build as failure
```
这是由于目标服务器没有jsch使用的算法。请在目标服务器的`/etc/ssh/sshd_config`最后一行增加如下内容:
```shell
KexAlgorithms diffie-hellman-group1-sha1,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1
```
然后重启 sshd服务 
```shell
service restart sshd
```
