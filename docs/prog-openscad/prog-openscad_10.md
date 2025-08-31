# 第十章：B

OpenSCAD 视觉参考

![](img/chapterart.png)

本附录是绘制、变换和组合本书中涵盖的 3D 和 2D 形状的快速视觉参考。每张截图旁都有一个示例 OpenSCAD 语句，可以用来生成该图像。在某些情况下，我们还包含了一个“阴影”对象，以说明操作之前形状的样子。示例代码语句不会生成这些阴影对象。

## 3D 基本图形

**长方体：**

```
`cube([30, 20, 10]);`
```

![fb03001](img/fb03001.png)

**居中长方体：**

```
`cube([30, 20, 10], center=true);` 
```

![fb03002](img/fb03002.png)

**球体：**

```
`sphere(10);` 
```

![fb03003](img/fb03003.png)

**平滑球体：**

```
`sphere(10, $fn=100);` 
```

![fb03004](img/fb03004.png)

**圆柱体：**

```
`cylinder(h=20, r=5);` 
```

![fb03005](img/fb03005.png)

**锥体：**

```
`cylinder(h=20, r1=5, r2=0);` 
```

![fb03006](img/fb03006.png)

**居中平滑截头锥：**

```
`cylinder(h=10, r1=3, r2=5, $fn=100, center=true);` 
```

![fb03007](img/fb03007.png)

**规则棱柱体：**

```
`cylinder(h=5, r=5, $fn=6);` 
```

![fb03008](img/fb03008.png)

## 2D 形状

**矩形：**

```
`square([30, 20]);` 
```

![fb03009](img/fb03009.png)

**居中矩形：**

```
`square([30, 20], center=true);` 
```

![fb03010](img/fb03010.png)

**圆形：**

```
`circle(10);` 
```

![fb03011](img/fb03011.png)

**规则多边形：**

```
`circle(10, $fn=5);`
```

![fb03012](img/fb03012.png)

**不规则多边形：**

```
`polygon([[0,0], [10,0], [10,10], [5,10]]);` 
```

![fb03013](img/fb03013.png)

**文本：**

```
`text("hello", font="Sans", size=20);`
```

![fb03014](img/fb03014.png)

## 组合形状

**从形状中减去：**

```
`difference() {`
 `sphere(10);`
 `translate([0,-15,0]) cube([15,30,15]);`
`}`
```

![fb03015](img/fb03015.png)

**从形状中进行多次减法：**

```
`difference() {`
 `sphere(10);`

 `cube([15, 15, 15]);`
 `cylinder(h=15, r=5);` 
`}`
```

![fb03016](img/fb03016.png)

**两个形状的交集：**

```
`intersection() {`
 `cube([10, 10, 10]);`
 `cylinder(h=15, r=5);` 
`}`
```

![fb03017](img/fb03017.png)

**从组合形状中减去：**

```
`difference() {`
 `union() {`
 `sphere(10);`
 `cylinder(h=30, r=5, center=true);` 
 `}`
 `cube([10, 30, 10], center=true);`
`}`
```

![fb03018](img/fb03018.png)

**凸包：**

```
`hull() {`
 `sphere(10);`
 `cylinder(h=20, r=5);` 
`}`
```

![fb03019](img/fb03019.png)

**闵可夫斯基和：**

```
`minkowski() {`
 `sphere(10, $fn=50);`
 `cylinder(h=20, r=5);` 
`}`
```

![fb03020](img/fb03020.png)

## 变换

**平移：**

```
`translate([5, 10, 0]) cube([5, 3, 1]);` 
```

![fb03021](img/fb03021.png)

**旋转：**

```
`rotate([0, 0, 60]) cube([30, 20, 10]);`
```

![fb03022](img/fb03022.png)

**反射：**

```
`mirror([1, 0, 0]) translate([5, 0, 0]) cylinder(h=1, r=5, $fn=5);` 
```

![fb03023](img/fb03023.png)

**调整尺寸：**

```
`resize([15, 20, 4]) sphere(r=5, $fn=32);` 
```

![fb03024](img/fb03024.png)

**挤出 2D 形状：**

```
`linear_extrude(height=10) {`
 `polygon([[0, 0], [10, 0],` 
 `[10, 10], [5, 10]]);` 
`}`
```

![fb03025](img/fb03025.png)

**旋转 2D 形状的挤出：**

```
`rotate_extrude(angle=180) translate([10, 0]) circle(5);`
```

![fb03026](img/fb03026.png)

## 循环

**重复形状：**

```
`for (x=[0:10:40]) {`
 `translate([x, 0, 0]) cube([5, 5, 10]);`
`}`
```

![fb03027](img/fb03027.png)

**改变重复形状的特征：**

```
`for (x=[0:1:4]) {`
 `h = x*5 + 5;`
 `translate([x*10, 0, 0]) cube([5, 5, h]);`
`}`
```

![fb03028](img/fb03028.png)

**重复形状的重复：**

```
`for (z=[0:15:45]) {`
 `for (x=[0:10:40]) {`
 `translate([x, 0, z]) cube([5, 5, 10]);`
 `}`
`}`
```

![fb03029](img/fb03029.png)
