---
layout: post
title: Qt based multithreading requester
---

&nbsp;&nbsp;&nbsp;&nbsp;As a Qt developer I really like signal/slot system based on event loop. 
Furthermore Qt provides very convenient way of [multithreading communication](http://doc.qt.io/archives/qt-4.8/threads-qobject.html)
based on signals and slots.
Let`s imagine that we have 2 different Qt objects living in different threads.
The first object emits signal to connected slot in the object 2.
The second object calls request slot in the second thread context, than emits signal to 
connected slot in the object 1. The first object calls response slot in the first thread context, 
correspondingly. As result we have something like request/response construction for different threads:
![image-title-here](/programming-world/assets/images/sequence-diagram-qt-multithread.png)

&nbsp;&nbsp;&nbsp;&nbsp;It is very useful especially if you are working with GUI based application. 
You can start request operation in one thread and than handle result of this request in the another thread.
But If you have lots of such operations it is not convenient to make different connections, declare slots and 
emit signals for simple request/response construction. 

&nbsp;&nbsp;&nbsp;&nbsp;The next part of the topic will be devoted to multithreading requester based on QT with very simple interface.

&nbsp;&nbsp;&nbsp;&nbsp;When we want  to make any request/response operation we do not want to spend much time for creating different
signal and slots. It is much easier to have request and response operations in one place with hidden multithreading 
functionality. Using functional objects and asynchronous approach we can make a request to the background thread and 
declare lambda expression, which will be called when request is finished, in the caller thread as response. 
{% highlight C++%}
	Requester::CRequester::
		Request<Requester::CResponse<int>>( &CWorker::RequestInt, Worker, "Hello" ). // Worker thread
		Response( this, [] ( int val ) // Sender thread
	{
		Log( QString("Response. Value: %1").arg( val ) );
	} );
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;As you can see from the example above, the "Request" function put the "CWorker::RequestInt" to the event loop 
of the "Worker" thread with some parameters. when the "CWorker::RequestInt" is finished" the lambda expression in 
the "Response" method will be called in the sender thread context ( the first parameter of the "Response" 
method is sender thread. It must be QObject class with corresponding thread affinity ).
All request methods must have the obligatory first parameter. This parameter is response proxy object.
The "Requester::CResponse" template parameters are related to the "Response" method signature and can be
variadic or event empty if there is no return value.
 
&nbsp;&nbsp;&nbsp;&nbsp;The first template argument of the “Response” method is very important. 
It is response proxy objec. If you want to pass some arguments from the request method and initiate 
response procedure, you should call this object using the overloaded operator ():
{% highlight C++%}
	void CWorker::RequestInt( const Requester::CResponse<int>& res, const QString& str )
	{
		int send_val = 33;
		res( send_val );
	}
{% endhighlight %}

&nbsp;&nbsp;&nbsp;&nbsp;There is one more way of convenient usage of the requester.
The "Request" function can take a functional object that will be called in any given thread.
{% highlight C++%}
	Requester::CRequester::
		Request<Requester::CResponse<float, float>>( Worker, [] ( const Requester::CResponse<float, float>& res )
	{
	        // Worker thread
		float f1 = 3.14f;
		float f2 = 32.64f;
		res( f1, f2 );
	} ).
		Response( this, [] ( float ret_f1, float ret_f2 )
	{
	        // Sender thread
		Log( QString("Response Lambda. Sent 1: %1, Sent: %2").arg( QString::number( ret_f1 ) ).arg( QString::number( ret_f2 ) ) );
	} );
{% endhighlight %}

&nbsp;&nbsp;&nbsp;&nbsp;In summary, we get very simple tool for multithreading communication without any explicit requirements of
making signals and suitable slots keeping in mind all thread routine.

**Link to the project: [Qt-Multithreaded-Requester](https://github.com/arturx64/Qt-Multithreaded-Requester)**