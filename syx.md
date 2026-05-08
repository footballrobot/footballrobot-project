第七周：尝试把ppo换成sac，但是效果不好，中断后继续训练效果都消失了，而且不适合改参数。发现原来的模型面对随机初始值没有泛化能力，hidden_unit从256改成512，num_layers从2改为3。

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








