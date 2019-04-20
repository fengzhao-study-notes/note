##### 1��prefork�����̣߳�

������ʽ��
	��Apache������������mpm_preforkģ���Ԥ�ȴ�������ӽ���(Ĭ��Ϊ5��)��ÿ���ӽ���ֻ��һ���̣߳������յ��ͻ��˵������mpm_preforkģ���ٽ�����ת�����ӽ��̴���������ÿ���ӽ���ͬʱֻ�����ڴ����������������ǰ��������������Ԥ�ȴ������ӽ�����ʱ��mpm_preforkģ��ͻᴴ���µ��ӽ������������������Apache������ͼ����һЩ���õĻ����ǿ��е��ӽ�������ӭ�Ӽ������������������ͻ��˵�����Ͳ���Ҫ�ڽ��� ��Ⱥ��ӽ��̵Ĳ���������mpm_preforkģ���У�ÿ�������Ӧһ���ӽ��̣������ռ�õ�ϵͳ��Դ�����������ģ����Խ϶ࡣ����mpm_preforkģ����ŵ���������ÿ���ӽ��̶������������Ӧ�ĵ��������������������һ�������������Ͳ� ��Ӱ�쵽��������
�ɵ��ڲ�����

```shell
<IfModule mpm_prefork_module>
# ����ʱĬ�Ͽ������ӽ�����
StartServers 10
# ��С�����ӽ�����
MinSpareServers 8
# ��������ӽ�����
MaxSpareServers 10
# ����ͬʱ�������������
MaxRequestWorkers 250
# ÿ���ӽ��̿ɴ�����������
MaxConnectionsPerChild 500
</IfModule>
```
?	�������ڴ�����StartServers���ӽ��̺�,Ϊ������MinSpareServer����������,���ȴ���һ������,�ȴ�һ��,������������,�ٵȴ�һ��,�����ĸ�����....�Լ��������Ӵ����Ľ���,���ﵽÿ�봴��32��,ֱ������MinSpareServer������(���Կ������µ�MinSpareServerΪ5),�����Ԥ����(Prefork)������,�������صȵ���������ʱ�Ż�ʱ�䴴���µĽ���,�����ϵͳ��Ӧ�ٶ�����������.

##### 2��worker�����̶߳���̣�
?	ʹ���˶���̺Ͷ��̵߳Ļ��ģʽ��workerģʽҲͬ������Ԥ����һЩ�ӽ��̣�Ȼ�� ÿ���ӽ��̴���һЩ�̣߳�ͬʱ����һ�������̣߳�ÿ����������ᱻ���䵽һ���߳��������̱߳�����̻����������Ϊ�߳���ͨ�����������̵��ڴ�ռ䣬��ˣ��ڴ��ռ�û����һЩ���ڸ߲����ĳ����»��prefork�и�����õ��̣߳����ֻ������һЩ�����⣬���һ���̳߳���������Ҳ�ᵼ��ͬһ�����µ��̳߳������⣬����Ƕ���̳߳������⣬Ҳֻ��Ӱ��Apache��һ���֣�������ȫ���������õ�����̶��̣߳���Ҫ���ǵ��̵߳İ�ȫ�ˣ���ʹ��keep-alive�����ӵ�ʱ��ĳ���̻߳�һֱ��ռ�ã���ʹ�м�û��������Ҫ�ȴ�����ʱ�Żᱻ�ͷţ���������preforkģʽ��Ҳ�� �ڣ�
�ɵ��ڲ�����

```shell
# event MPM
<IfModule mpm_worker_module>
# ����������ʱ�������ӽ�������
StartServers 3 
# �����ӽ��̵���С����
MinSpareThreads 75 
# �����ӽ��̵���С����
MaxSpareThreads 250 
# ÿ���ӽ��̲������߳�����
ThreadsPerChild 25 
# �޶�������ͬһʱ���ڿͻ��������������������Ĭ����150���κγ����˸����Ƶ�����Ҫ����ȴ����У�һ��һ�������ӱ��ͷţ������е�����Ž��õ�����
MaxRequestWorkers 400 
# ÿ���ӽ�����������������������������������������������Ѿ��ﵽ�����ֵ���ӽ��̽���������������Ϊ0���ӽ��̽���Զ�������������ֵ����Ϊ��0ֵ�����Է�ֹ����PHP���µ��ڴ�й¶
MaxRequestsPerChild 0  
</IfModule>
```
##### 3��event
?	 IO��·���ã������¼�������ģ�͡�һ�����̴�������׽��֣�һ������ͬʱ�����������
��Apache���µĹ���ģʽ����workerģʽ�ı��֣����ѷ�����̴������з��������һworkerģʽ��ͬ���������������keep-alive�����ӵ�ʱ��ռ���߳���Դ���˷ѵ����⣬��event����ģʽ�У�����һЩר�ŵ��߳�����������Щkeep-alive���͵��̣߳�������ʵ���������ʱ�򣬽����󴫵ݸ����������̣߳�ִ����Ϻ����������ͷš�����ǿ���ڸ߲��������µ���������������һ���߳̾��ܴ������������ˣ�ʵ�����첽��������eventMPM������ĳЩ�����ݵ�ģ��ʱ����ʧЧ��������˵�workerģʽ��һ�������̴߳���һ�����󡣹ٷ��Դ���ģ�飬ȫ����֧��event MPM ��
�ɵ�����:

```shell
# event MPM
<IfModule mpm_worker_module>
# ����������ʱ�������ӽ�������
StartServers 3 
# �����ӽ��̵���С����
MinSpareThreads 75 
# �����ӽ��̵���С����
MaxSpareThreads 250 
# ÿ���ӽ��̲������߳�����
ThreadsPerChild 25 
# �޶�������ͬһʱ���ڿͻ��������������������Ĭ����150���κγ����˸����Ƶ�����Ҫ����ȴ����У�һ��һ�������ӱ��ͷţ������е�����Ž��õ�����
MaxRequestWorkers 400 
# ÿ���ӽ�����������������������������������������������Ѿ��ﵽ�����ֵ���ӽ��̽���������������Ϊ0���ӽ��̽���Զ�������������ֵ����Ϊ��0ֵ�����Է�ֹ����PHP���µ��ڴ�й¶ 
MaxRequestsPerChild 0 
</IfModule>
```