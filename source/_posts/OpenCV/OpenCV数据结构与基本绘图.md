---
title: OpenCV数据结构与基本绘图
date: 2017-03-28 22:54:20
categories:
- 技术
tags:
- OpenCV基础
- c++
comments: true
---
1. IplImage与Mat
在最开始，OpenCV使用基于C语言接口构建，因此在较早版本的OpenCV中使用IplImage存储图像数据。这就造成了一定的问题，在调试了的时候，需要程序员手动release为IplImage分配的内存，不然可能造成内存泄漏。
在OpenCV2.0以来，就引入了C++接口，也就产生了基于C++的Mat数据结构，没错，它是可以自动内存管理的，因此我们现在用的最多的就是Mat数据结构了。Mat类很方便，不用手动分配内存、释放内存。
那么为什么IplImage还继续顽强存在呢，那就是因为大部分嵌入式系统只支持C语言.

2. Mat类 包含两个数据部分，一个部分是矩阵头（矩阵尺寸、存储方法、存储地址等），另一部分指向存储所有像素值得矩阵。

需要记住的是，大多时候复制Mat类，只是引用矩阵头，而不是复制整个像素矩阵，这将导致巨大的时间成本。因此，OpenCV采用引用计数机制，每个Mat对象都有独自的矩阵头，但是共享一个像素矩阵。常见的复制构造函数、赋值符号都是只复制矩阵头，而不是复制整个像素矩阵。

```
//以下三个Mat对象有不同的矩阵头，但是共享一个矩阵
Mat A;   //创建矩阵头部分
A = imread(const string & filename, 1);  // 为矩阵开辟空间
Mat B(A);  //采用复制构造函数
Mat C = A;
```

+ 引用计数机制在上面的例子中有很好的说明，矩阵的引用次数为3，当Mat C的矩阵头被释放后，引用计数就减1，当引用次数为0时，矩阵空间就会被释放。

+ 那么如果想复制整个矩阵，就需要使用clone()和copyTo()函数。

```
Mat D = A.clone();
Mat E;
A.copyTo(E);
```

+ 简要小结
++ OpenCV函数中C++接口，图像的内存都自动分配、释放；
++ 复制构造函数和赋值运算符只复制矩阵头；
++ clone(),copyTo()函数可以复制矩阵。


3. Mat对象的创建和显式
Mat类型有多种创建方法，
+ 采用默认构造函数
+ 使用构造函数初始化
+ 为IplImage对象创建Mat信息头
+ 使用Create（）成员函数
+ 采用Matlab方式初始化
+ 采用逗号分隔符方式初始化
+ 为已存在的Mat对象创建新信息头
具体初始化代码如下：
```
   //方法1：使用Mat（）构造函数
	Mat M(2, 2, CV_8UC3, Scalar(0,0,255));
	cout << "M = " << endl << " " << M << endl;

	//way2 没有太懂，由于这是多维的数组，使用cout出错
	int sz[3] = {2, 2, 2};
	Mat L(3, sz, CV_8UC3, Scalar::all(0));
	
	//way3 为IplImage创建信息头
	IplImage* img = cvLoadImage("D:\\C++程序联系文件夹（可选择性删除）\\OpenCV_Exercise\\learn.jpg",1);
	Mat mtx(img);

	//way4 使用Create()函数
	Mat K;
	K.create(4,4,CV_8UC(2));
	cout << "K = " << endl << " " << K << endl << endl;
	//way5 Matlab大法好
	Mat E = Mat::eye(4,4,CV_64F);
	cout << "E = " << endl << " " << E << endl << endl;

	Mat O = Mat::ones(2,2,CV_32F);
	cout << "O = " << endl << " " << O << endl << endl;

	Mat Z = Mat::zeros(3,3,CV_8UC1);
	cout << "Z = " << endl << " " << Z << endl << endl;

	//way6 
	Mat C = (Mat_<double>(3,3) << 0, -1, 2, -1, 5, -1, 0, -1, 0);
	cout << "C = " << endl << " " << C << endl << endl;

	//way7 为已存在的对象创建新的信息头
	Mat RowClone = C.row(0).clone();
	cout << "RowClone = " << endl << " " << RowClone << endl << endl;
```
Mat类型的格式化输出方法，
+ OpenCV默认方式
+ Python输出风格
+ 逗号分隔风格（CSV）
+ Numpy风格
+ C语言风格
例程如下：
```
	Mat r = Mat(3, 3, CV_8UC3);
	randu(r, Scalar::all(0), Scalar::all(255));
	//OpenCV格式化输出
	//way1 Opencv默认输出格式
	cout << "r 默认输出格式: " << endl << r << endl;

	//way2 python风格思密达
	cout << "r Python风格: " << endl << format(r, "python") << endl;

	//way3 CSV风格
	cout << "r CSV风格: "  << endl << format(r, "csv") << endl;

	//way4 Numpy风格
	cout << "r Numpy风格: " << endl << format(r, "numpy") << endl;

	//way5 C style
	cout << "r C style: " << endl << format(r, "C") << endl;
```
![输出格式](http://oapeb119y.bkt.clouddn.com/image/opencv/Mat%E8%BE%93%E5%87%BA%E9%A3%8E%E6%A0%BC.png)

4. OpenCV中常用基于图像的数据结构与函数介绍
+ 点的表示：Point 类
二维坐标系下的点
```
	Point2f p(6,2);
	cout << "[二维点] p = " << p << endl;

	Point3f l(1,2,3);
	cout << "[三维点] l = " << l << endl;

	vector<float> v;
	v.push_back(3);
	v.push_back(5);
	v.push_back(4);
	cout << "[基于Mat的Vector] v = " << Mat(v) << endl;

	vector<Point2f> points(20);
	for(size_t i = 0; i < points.size(); ++i)
	{
		points[i] = Point2f((float)(i*5),(float)(i%7));
	}
	cout << "[二维点向量] points = " << points << ";";
```
Point2f、Point3f等等其实都是一个类模板Point_<typename T>.具体定义core.hpp中。
```
template<typename _Tp> class Point_
{
public:
    typedef _Tp value_type;

    // various constructors
    Point_();
    Point_(_Tp _x, _Tp _y);
    Point_(const Point_& pt);
    Point_(const CvPoint& pt);
    Point_(const CvPoint2D32f& pt);
    Point_(const Size_<_Tp>& sz);
    Point_(const Vec<_Tp, 2>& v);

    Point_& operator = (const Point_& pt);
    //! conversion to another data type
    template<typename _Tp2> operator Point_<_Tp2>() const;

    //! conversion to the old-style C structures
    operator CvPoint() const;
    operator CvPoint2D32f() const;
    operator Vec<_Tp, 2>() const;

    //! dot product
    _Tp dot(const Point_& pt) const;
    //! dot product computed in double-precision arithmetics
    double ddot(const Point_& pt) const;
    //! cross-product
    double cross(const Point_& pt) const;
    //! checks whether the point is inside the specified rectangle
    bool inside(const Rect_<_Tp>& r) const;

    _Tp x, y; //< the point coordinates
}; 

typedef Point_<int> Point2i;
typedef Point2i Point;
typedef Point_<float> Point2f;

```
从这个二维点的类模板可以看出Point_<float> Point2f 都等价。

+ 颜色的表示： Scalar 类
这个也是OpenCV中实现的类模板，其实可以简单的看成包含4个元素的Vector,不过一般我们都是用RGB表示颜色，因此只用到三个元素。<b>需要注意的是，OpenCV默认的图片通道存储顺序不是我们熟悉的RGB，而是BGR。<\b>下面为Scalar模板类的源码，位于core.hpp中。
```
template<typename _Tp> class Scalar_ : public Vec<_Tp, 4>
{
public:
    //! various constructors
    Scalar_();
    Scalar_(_Tp v0, _Tp v1, _Tp v2=0, _Tp v3=0);
    Scalar_(const CvScalar& s);
    Scalar_(_Tp v0);

    //! returns a scalar with all elements set to v0
    static Scalar_<_Tp> all(_Tp v0);
    //! conversion to the old-style CvScalar
    operator CvScalar() const;

    //! conversion to another data type
    template<typename T2> operator Scalar_<T2>() const;

    //! per-element product
    Scalar_<_Tp> mul(const Scalar_<_Tp>& t, double scale=1 ) const;

    // returns (v0, -v1, -v2, -v3)
    Scalar_<_Tp> conj() const;

    // returns true iff v1 == v2 == v3 == 0
    bool isReal() const;
};

typedef Scalar_<double> Scalar;
```
+ 尺寸的表示： Size 类
与以上的基本都一样，也是一个类模板，此处不详细描述。常用：' Size(5, 6) // width 5, height 6 '.
+ 矩阵的表示： Rect 类
同上。
+ 颜色的空间转换： cvtColor() 函数
`void cvtColor(InputArray src, OutputArray dst, int code, int dstCn = 0);`
InputArray src：输入图像
OutputArray dst: 输出图像
int code: 颜色空间转换标识符，是一堆枚举值，具体见`.../imgproc/types_c.h`
int dstCn: 目标图像的通道数，默认值为0，取原图像的通道数



