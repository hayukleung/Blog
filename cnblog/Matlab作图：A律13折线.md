---
title: Matlab作图：A律13折线
date: 2012-05-03 22:23:31
tags: communication
---
[原文地址](http://www.cnblogs.com/liangxiaxu/archive/2012/05/03/2481490.html)

[TOC]

打开matlab，新建m文件，敲入以下代码后运行。
```matlab
x1=-1:0.001:-0.5;
y1=0.25*x1-0.75;
axis([-1.1,1.1,-1.1,1.1]);
plot(x1,y1);
hold on
x2=-0.5:0.0001:-0.25;
y2=0.5*x2-0.625;
plot(x2,y2);
x3=-0.25:0.00001:-0.125;
y3=x3-0.5;
plot(x3,y3);
x4=-0.125:0.000001:-0.0625;
y4=2*x4-0.375;
plot(x4,y4);
x5=-0.0625:0.0000001:-0.03125;
y5=4*x5-0.25;
plot(x5,y5);
x6=-0.03125:0.0000001:-0.015625;
y6=8*x6-0.125;
plot(x6,y6);
x7=-0.015625:0.0000001:0.015625;
y7=16*x7;
plot(x7,y7);
x8=0.015625:0.0000001:0.03125;
y8=8*x8+0.125;
plot(x8,y8);
x9=0.03125:0.0000001:0.0625;
y9=4*x9+0.25;
plot(x9,y9);
x10=0.0625:0.000001:0.125;
y10=2*x10+0.375;
plot(x10,y10);
x11=0.125:0.00001:0.25;
y11=x11+0.5;
plot(x11,y11);
x12=0.25:0.0001:0.5;
y12=0.5*x12+0.625;
plot(x12,y12);
x13=0.5:0.001:1;
y13=0.25*x13+0.75;
plot(x13,y13);
grid on
title('A律13折线');
text(0,0,'原点');
set(gca,'xtick',[-1:0.1:1]);
set(gca,'ytick',[-1:0.125:1]);
```