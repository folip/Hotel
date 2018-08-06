### 更好的方式：8-6

#### QML&C++交互编程：改为在C++中实例化后注册到QML中

==实现代码：==

**类声明**

```C++
#include <QObject>
class Hotel_list: public QObject
{
    Q_OBJECT
public:
    Q_INVOKABLE QString name(int idx);
private:
    QVector<hot> hotels;
    int size;
    QVector<int> search_results;
};
// 包含头文件；继承QOBJECT;将在QML中调用的函数声明为Q_INVOKABLE
```

**main.cpp**

```C++
Hotel_list hotel_list("test");
 
QQmlApplicationEngine engine;
QQmlContext* rootContex = engine.rootContext();//拿到engine的根上下文
rootContex->setContextProperty("hotel_list",  &hotel_list);//注册
//hotel_list为在QML中使用时的名称，另一个参数为 const *QOBJECT类型
engine.load(QUrl(QStringLiteral("qrc:/main.qml")));
```

**QML**

```
hotel_list.name(2);
//可直接调用
```

==遇到的问题：==

在

```
rootContex->setContextProperty("name", &a_name);
```

在这行中报错Qmlcontext不是完整类型，下拉发现另一条forward declaration of class qmlcontext

粗暴的解决方法：打开对应的头文件，删除`class qmlcontext;`这条前置声明，在开头加上`#include <qmlcontext.h>`....

==优势：==

加强前后端的分离，便于对象作为参数传递



#### 除customer_list等类外，添加customer等类并设置 Q_PROPERTY

```C++
void setphone_number(int idx, QString newphone_number);
//整体列表类的成员函数声明，不可在QML中直接调用
class customer: public QObject{
Q_PROPERTY(QString phone_numebr READ phoen_number WRITE set_phonenumber)
    public:
	Q_INVOKEABLE void setphone_number(QString newphone_number,const& customer_list cus_list = customer_list){
    cus_list.setphone_number(rank,newphone_number);
    }
    
    Q_INVOKABLE void create_order(const &order_list ol,);//省略了其他参数,
private:
 	rank;
}
```

==优势==

* 实现对最终数据的封装，保证界面前端不能直接改变数据，只能更改登录用户的数据
* 在开始设置customer.rank等信息后，可以将这些信息包含在对应的函数声明中，便利后续调用
* 在设置Q_PROPERTY后甚至进一步简化, 可以如此书写：

```
customer.phone_number = "110"
text = customer.phone_number
```

实质是调用对应的函数.
