---
layout: post
title: How to change a permission of pem file to connect ssh in Windows 10
tag: know-how
subtitle: easy way
---





# How to change a permission of pem file to connect ssh in Windows 10







I faced with a problem. 

![image-20201028192743249](D:\git\heojoon.github.io\img\2020-08-24_sftp_chroot 설정.md)



1. You open properties of the pem file and move security Tab, then Click the "고급" button

<img src="D:\git\heojoon.github.io\img\image-20201028193213213.png" alt="image-20201028193213213" style="zoom:67%;" />



2. You have to change ownership of the file.



<img src="D:\git\heojoon.github.io\img\image-20201028193407962.png" alt="image-20201028193407962" style="zoom: 67%;" />

3. You select a username in your computer to decide the file's ownership. (must a user of person, EX MYCOM)

<img src="D:\git\heojoon.github.io\img\image-20201028193523881.png" alt="image-20201028193523881" style="zoom: 67%;" />

4. You select remove all Inheritance.

<img src="D:\git\heojoon.github.io\img\image-20201028193804585.png" alt="image-20201028193804585" style="zoom:67%;" />

5. You add new permission to use. and Select security principal. (ex. MYCOM) , **Only Check the Read**

![image-20201028194040729](D:\git\heojoon.github.io\img\image-20201028194040729.png)

6. The result screen should be as follows.

![image-20201028194202231](D:\git\heojoon.github.io\img\image-20201028194202231.png)



![image-20201028194329201](D:\git\heojoon.github.io\img\image-20201028194329201.png)

7. Finally, Try reconnecting it. Then you'll be able to success.



![image-20201028194408420](D:\git\heojoon.github.io\img\image-20201028194408420.png)