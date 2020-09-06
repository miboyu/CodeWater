# Fortran进阶篇01：数组高级

*上回说到数组之基础应用，未能详尽，实有不甘，故而一鼓作气将数组说完，以便利用。（咳咳，正经点……）*



我们知道数组的声明形式如下：

```fortran
INTEGER,DIMENSION(10)::iarray
```

但终究只是一维的，实际应用中更多的是二维数组，如Excel表格中见到的数据，又如线性代数中学到的矩阵。这些数据在Fortran中该怎么表示呢？

```fortran
INTEGER,DIMENSION(10,10)::iarray2
```

![img](Fortran进阶篇01：数组高级.assets/2_05.png)是不是太简单了点。。。那我要三维四维的呢？

```fortran
INTEGER,DIMENSION(10,10,3)::iarray3     !三维数组
INTEGER,DIMENSION(10,10,3,5)::iarray4   !四维数组
```

Fortran最多支持七维数组，一个正常人应该用不到这么多吧![img](Fortran进阶篇01：数组高级.assets/u1F602.png)一个好的程序员连四维数组都很少用到，三维数组都尽量拆成多个二维数组，为的是减少复杂度，减少出错机率。



插播一点小知识，内存中的数据都是一维的，每个存储位置都对应一个唯一的地址，那多维数组在内存中怎么存放呢？答案是把多维数组切成一个个的数据条，然后串成一个数据链，就变成一维的了。那怎么切呢？横着切还是竖着切？对二维数组而言是竖着切，即沿着内存序列走，第一个索引（行号）先变化，其次是第二个索引（列号），如果是多维数组，则依次为第三个索引、第四个索引……。知道这个有什么用呢？以最常见的二维数组为例，既然是列优先的存储顺序，下面示例中遍历方式二将比遍历方式一更高效，因为方式二的遍历顺序与内存顺序一致。再深入的原因就不展开了，如果你非觉得方式一看着舒服，不在乎这点性能损耗，那就随意吧![img](Fortran进阶篇01：数组高级.assets/2_05.png)

```fortran
REAL, DIMENSION(100,90)::array
REAL::sum=0INTEGER::i,j  ... !  array赋值
!遍历方式一
DO i=1,100
	DO j=1,90  
    	sum = sum + array(i,j)  
    END DO
END DO
!遍历方式二
DO j=1,90  
	DO i=1,100  
    	sum = sum + array(i,j)
    END DO
END DO
```



知道怎么定义多维数组，知道一维数组的索引方式，多维数组的索引也就很清楚了，不同维度的索引值用逗号隔开就行了。至于索引值的界限、下标三元组的索引方式、以及重定义索引上下界等知识点，聪明的你自行脑补就够了。



由于数组的操作经常会用到DO循环，有时还有IF判断，Fortran内置了两种专门针对数组的结构WHERE和FORALL，在遍历数组的同时兼具判别功能。由于这两种结构可以被DO和IF代替，作者建议不必花过多时间钻研WHERE和FORALL的用法，除非你想突显一下自己的语言水平或者想让代码更简洁一些（不代表速度会提升，至于并行优化目前还涉及不到）。



前面讲的所有数组知识，声明时都在DIMENSION关键字中指定了数组尺寸。如果事先不知道该定义多大的数组呢？一种办法是定义一个足够大的数组，以涵盖要保存的实际数组大小，许多老代码中就是这么解决的，这样带来的问题是空间浪费，也不具备扩展性。另一办法就是动态分配了，定义的时候不实际分配内存空间给这个数组，等到确定尺寸以后再来分配，看示例：

```fortran
REAL, ALLOCATABLE, DIMENSION(:,:)::array  !声明动态数组
INTEGER::NX,NY  !数组尺寸，待运行时决定...
ALLOCATE(array(NX,NY))                    !给动态数据分配空间
```

如果某个数组声明的时候不知道尺寸应该是多大，就用冒号代替，并添加ALLOCATABLE关键字，说明该数组是动态分配的，目前不需要分配具体的空间。等尺寸定下来以后，再用ALLOCATE()函数为其分配空间，此时分配的尺寸可以在运行的过程中根据实际计算动态决定。



既然能分配内存，自然也能回收，对于可分配数组，使用DEALLOCATE()函数进行回收，注意回收的数组变量必须是具有可分配（ALLOCATABLE）属性的。回收后的可分配数组可以再次使用ALLOCATE()函数进行分配。

```fortran
...
DEALLOCATE(array)        !回收内存
ALLOCATE(array(80,80))   !重新分配内存
```



判断一个可分配数组是否已分配内存，使用函数ALLOCATED()

```fortran
...
IF(ALLOCATED(array)) THEN  !如果已分配，先回收 
	DEALLOCATE(array)
END 
IFALLOCATE(array(20,20))
```



最后列举一些与数组有关的常用内置函数

```fortran
!数组信息查询函数
ALLOCATED(array)  !查询数组是否已分配，对可分配数组有效
LBOUND(array)     !查询数组各个维度的索引下界，默认情况下均为1
LBOUND(array,dim) !查询数组在指定维度上的索引下界，默认情况下为1
UBOUND(array)     !查询数组各个维度的索引上界，具体看定义
UBOUND(array,dim) !查询数组在指定维度上的索引上界，具体看定义
SIZE(array)       !查询数组元素的总个数
SIZE(array,dim)   !查询数组在指定维度上的宽度
SHAPE(array)      !查询数组的尺寸结构
```

```fortran
!数组元素统计函数
! mask是数组条件表达式，如对于数组array，mask可以是array>0
! 采用mask表示时不用具体索引到某个元素，实际函数中会对每个元素进行mask判断
ALL(mask)           !当数组所有元素的mask判断为真时返回真
ANY(mask)           !当数组任意元素的mask判断为真时返回真
COUNT(mask)         !统计数组中满足mask条件的元素个数
MINVAL(array)       !统计数组中所有元素的最小值
MINVAL(array, mask) !统计数组中满足mask条件的元素的最小值
MINLOC(array)       !统计数组中所有元素的最小值的位置索引
MINLOC(array, mask) !统计数组中满足mask条件的元素的最小值的位置索引
MAXVAL(array)       !统计数组中所有元素的最大值
MAXVAL(array, mask) !统计数组中满足mask条件的元素的最大值
MAXLOC(array)       !统计数组中所有元素的最大值的位置索引
MAXLOC(array, mask) !统计数组中满足mask条件的元素的最大值的位置索引
```

```fortran
!数组运算与变换函数
SUM(array)              !计算数组中所有元素之和
SUM(array, mask)        !计算数组中满足mask条件的元素之和
PRODUCT(array)          !计算数组中所有元素之积
PRODUCT(array, mask)    !计算数组中满足mask条件的元素之积
DOT_PRODUCT(vecA,vecB)  !计算两个同尺寸一维向量的点积
MATMUL(matA,matB)       !计算两个矩阵的乘积（尺寸要满足矩阵乘法要求）
TRANSPOSE(mat)          !计算矩阵的转置
RESHAPE(mat,shape)      !矩阵尺寸重构，如6*2矩阵转成3*4，不常用
```
