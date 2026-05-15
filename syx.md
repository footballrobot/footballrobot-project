第八周：实现了机器人大概率踢进球门。实现了机器人和球出界的时候传送到地图其它位置（因为有8个场地并行运行，还遇到了用世界坐标，而不是各自场地坐标的问题）。实现了踢进球的时候出现一个奖杯的功能。


https://github.com/user-attachments/assets/d695b4c8-074b-4f87-b343-e22ea88e74f8



第七周：尝试把ppo换成sac，但是效果不好，无法中断后保留原来的训练效果，而且不适合改参数。发现原来的模型面对随机初始值没有泛化能力，hidden_unit从256改成512，num_layers从2改为3。

第五周：机器人有概率踢球进门。


https://github.com/user-attachments/assets/04da98e3-6971-42fd-9a86-06c22f225a62



第四周：现在机器人能踢到球了，不过是以一种很特别的方式，它把自己的一只脚停在原地，另一只脚紧贴着地面做圆周运动。
发现问题出在地面摩擦力过小，提高了摩擦系数。

第三周：训练速度较慢，1400万步用了一天时间训练，瓶颈在unity物理仿真，同时跑多个仿真实例，提速了5倍。

第二周：
[训练g1.docx](https://github.com/user-attachments/files/26437220/g1.docx)
[训练记录.docx](https://github.com/user-attachments/files/26437226/default.docx)

https://github.com/user-attachments/assets/50697cfd-4dd1-4363-bbef-b188ed9269cb



https://github.com/user-attachments/assets/f00060b9-b42a-4b1d-b430-aeac5ae6216b

第一周：
创建了github账号和仓库。
第一次完成机器人训练，50万次，能站稳了。
测试fork功能
[训练.docx](https://github.com/user-attachments/files/26437154/default.docx)








