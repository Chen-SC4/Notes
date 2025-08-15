# Qt 多线程

## QThread 常用方法

1. `QThread::currentThreadId()` 返回当前线程的 ID，它是一个静态方法。
2. `QThread::sleep` 静态方法，可以让当前线程休眠一段时间。
3. `QThread::idealThreadCount` 静态方法，返回当前系统的 CPU 核心数。
4. `QThread::yieldCurrentThread` 静态方法，让当前线程让出 CPU 时间片。

## QThread 使用示例

Qt 中的多线程主要是有两种使用方法，一种方法是让类继承 `QThread`，然后重写 `run` 方法。

```c++
class GenerateRandom : QThread {
	Q_OBJECT
protected:
	void run() override;
public slots:
	void ReceiveCount(int count) {
		this->RandomCount = count;
	}
private:
	int RandomCount;
}
```

由于 `run` 方法是一个 `void(void)` 类型的方法，所以如果需要给线程传递参数，需要首先通过信号与槽的方式，传递给线程类的成员变量。

这种方式的优点在于简单，如果线程只需要做一些简单的工作，使用这种方式可以很快地实现。

---
另一种方式是使用 `QThread.moveToThread` 方法，来完成多线程。

首先需要使用多线程的类需要继承自 `QObject`。

```cpp
class Generate : public QObject
{
    Q_OBJECT
public:
    explicit Generate(QObject *parent = nullptr);
public slots:
    void working(int count);
private:
signals:
    void SendArray(QVector<int>);
};
```

上面示例代码中的 `working` 就是我们想要放到线程中执行的函数。

然后在主线程中，需要创建出 `Generate` 对象以及一个 `QThread` 对象，然后把 `Generate` 对象**移动**到 `QThread` 对象中。

```cpp
MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent), ui(new Ui::MainWindow) {
    ui->setupUi(this);
    auto* generate = new Generate();

    auto* generateThread = new QThread(this);

    generate->moveToThread(generateThread);
    generateThread->start();

    connect(ui->start, &QPushButton::clicked, generate, [=]() {
        generate->working(10000);
    });

    connect(generate, &Generate::SendArray, this, [=](QVector<int> array) {
        for (auto i : array) {
            this->ui->randomList->addItem(QString::number(i));
        }
    });
}
```

使用 `moveToThread` 方法，将 `Generate` 对象移动到 `QThread` 对象中，这样 `Generate` 对象中的 `working` 函数就会在新的线程中执行。

有两点需要注意：
1. 在上面的 `connect` 函数中，接收者必须是 `generate` 对象，不能是 `this`。否则 `Lambda` 函数会放到 `this` 所在的线程的事件循环中。
2. 在执行之前，需要先调用 `generateThread->start()`，否则 `working` 函数不会执行。

对于上面的第一点，如果把 `generate` 写成 `this`，那么 `Lambda` 函数就会放到主线程的事件循环中，此时 `generate->working(10000);` 也会在主线程中执行。

如果希望在这种情况下，`generate->working(10000);` 在线程中执行，那么可以使用 `QMetaObject::invokeMethod` 方法。

```cpp
connect(ui->start, &QPushButton::clicked, this, [=]() {
		QMetaObject::invokeMethod(generate, "working", Qt::QueuedConnection, Q_ARG(int, 10000));
});
```
