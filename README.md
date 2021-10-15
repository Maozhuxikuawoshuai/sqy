# sqy

#VS 问题记录

1.LINK2005和LNK1169 error，编码的时候先写的.cpp并且编译了，后把该.cpp改成.h后，编译出现该问题，重新编译还是报错。
问题原因：编译器生成了.cpp的.obj文件到debug目录下，改为.h后下次编译时会把该文件还识别成cpp，即使重编并不删除obj
解决办法：手动在debug目录下删除.obj描述文件
参考：https://www.jb51.net/article/208732.htm

2.接口stl跨编译器问题，解决方案：纯虚接口（.h头文件）替换，代码直接写到头文件里，让编译器直接运行而不用自己处理

原代码：
class ThreadPool
{
public:
	virtual void start(size_t number = 1) = 0;

	virtual void exec(std::function<void()> &&task) = 0;
}

跨编译器处理：
class ThreadPool
{
public:
	ThreadPool()
	{
		_tasks = new std::queue<std::function<void()>>();
		_mutex = new std::mutex();
	}

	virtual ~ThreadPool()
	{
		delete _tasks;
		delete _mutex;
	}

	virtual void start(size_t number = 1) = 0;

	virtual void exec(std::function<void()> &&task)
	{
		{
			std::lock_guard<std::mutex> lock(*_mutex);
			(*_tasks).emplace(task);
		}

		notifyOne();
	}
}
