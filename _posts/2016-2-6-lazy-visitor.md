---
layout: post
title: Lazy visitor. Modification of the visitor design pattern and downcasting without RTTI.
---

**Dear reader.**

&nbsp;&nbsp;&nbsp;&nbsp;I hope this finds you well. I want to introduce your modification for the classic Visitor pattern to make usage of this pattern more agile and easier. I've called this modification - lazy visitor. Culmination of my topic will be an example for making downcasting without RTTI, based on the lazy visitor.<br/>

&nbsp;&nbsp;&nbsp;&nbsp;First of all, we will make a revision of original visitor pattern.
What features does this pattern give us?

* You can assign some operations, for any class in the class hierarchy, without changing signature of classes from the hierarchy.
* You can restore lost information from pointer to the base class.
* You can determinate special behavior for certain derived classes, having only pointer to the base class.
* Provides double dispatch mechanism.

&nbsp;&nbsp;&nbsp;&nbsp;We have a class hierarchy for different shapes. Each derived class has a unique internal data to describe position, such as: width, height, radius etc. Let's imagine that we have unexpectedly decided, that we need found the way how to represent information about this classes to the console, window, or save to the file. If we want to avoid ruining classic class hierarchy and open a way for adding new functionality, we can use Visitor pattern to solve this problem.

{% highlight C++%}

#include <iostream>
#include <vector>

// asdad
class Visitor
{
public:
	virtual void visit(class Circle *shape) = 0;
	virtual void visit(class Square *shape) = 0;
	virtual void visit(class Triangle *shape) = 0;
};

class Shape
{
public:
	Shape(int x, int y)
		: m_x(x)
		, m_y(y)
	{}

	int getX()
	{
		return m_x;
	}

	int getY()
	{
		return m_y;
	}

	virtual ~Shape() {};
	virtual void draw() = 0;
	virtual void accept(class Visitor &v) = 0;

protected:
	int m_x;
	int m_y;
};

class Circle : public Shape
{
public:
	Circle(int x, int y, int radius)
		: Shape(x, y)
		, m_radius(radius)
	{}

	virtual void draw() override
	{
		std::cout << "Circle   X=" << m_x << " Y=" << m_y << " Radius=" << m_radius << std::endl;
	}

	virtual void accept(class Visitor &visitor) override
	{
		visitor.visit(this);
	}

	int getRadius()
	{
		return m_radius;
	}
private:

	int m_radius;
};

class Square : public Shape
{
public:
	Square(int x, int y, int width, int height)
		: Shape(x, y)
		, m_width(width)
		, m_height(height)
	{}

	virtual void draw() override
	{
		std::cout << "Square   X=" << m_x << " Y=" << m_y << " Width=" << m_width << " Height=" << m_height << std::endl;
	}

	virtual void accept(class Visitor &visitor) override
	{
		visitor.visit(this);
	}

	int getWidth()
	{
		return m_width;
	}

	int getHeight()
	{
		return m_height;
	}
private:

	int m_width;
	int m_height;
};

class Triangle : public Shape
{
public:

	Triangle(int x, int y, int z)
		: Shape(x, y)
		, m_z(z)
	{}
	virtual void draw() override
	{
		std::cout << "Triangle X=" << m_x << " Y=" << m_y << " Z=" << m_z << std::endl;
	}

	virtual void accept(class Visitor &visitor) override
	{
		visitor.visit(this);
	}

	int getZ()
	{
		return m_z;
	}
private:

	int m_z;
};

class FileVisitor : public Visitor
{
public:
	virtual void visit(Circle *shape) override
	{
		std::cout << "Save to file Circle";
	}
	virtual void visit(Square *shape)  override
	{
		std::cout << "Save to file Square";
	}
	virtual void visit(Triangle *shape) override
	{
		std::cout << "Save to file Triangle";
	}
};

class ConsoleVisitor : public Visitor
{
public:
	virtual void visit(Circle *shape) override
	{
		std::cout << "Show Circle";
	}
	virtual void visit(Square *shape)  override
	{
		std::cout << "Show Square";
	}
	virtual void visit(Triangle *shape) override
	{
		std::cout << "Show Triangle";
	}
};

void consoleOutput(Shape* shape)
{
	ConsoleVisitor visitor;
	shape->accept(visitor);
}

void show(Shape* shape)
{
	shape->draw();
}

void main()
{
	std::vector<Shape*> shapes = { new Circle(4, 6, 80), new Square(5, 8, 20, 25), new Triangle(5, 7, 8) };

	for (Shape* shape : shapes)
	{
		show(shape);
		consoleOutput(shape);
	}

	for (Shape* shape : shapes)
	{
		delete shape;
	}
}
{% endhighlight %}

&nbsp;&nbsp;&nbsp;&nbsp;This an example shows how to use the visitor patter. Class ConsoleVisitor appends new operation to the whole Shape class hierarchy. Each method “visit” in the visitor class describes certain behavior for certain derived class.<br/>
&nbsp;&nbsp;&nbsp;&nbsp;Now, if you want to define personal behavior for each derived class without changing class hierarchy, you just need to write new visitor and pass it to the accept method.<br/>
But, visitor has well-known drawbacks:

* If you add to the target class hierarchy new class, you have to change all visitors as well ( new class = additional “visit” method).
* If I want to add special behavior only for one derived class, I have to write new visitor with the implementation only for the one “visit” method. That is redundantly.
* Result extraction from the visitor after work with several derived classes to combine results is not convenient. For instance: when you want to combine coordinates for square and triangle by using the visitor pattern.

&nbsp;&nbsp;&nbsp;&nbsp;How to avoid all these drawbacks and, as a bonus, have ability to make easy downcasting?<br/>
To make visitor agile, we can replace simple method calling to callback, calling using lambda expressions, functors, or direct function calling:
{% highlight C++%}
class LazyVisitor: public Visitor
{
public:
	virtual bool tryVisit(Circle *shape) override
	{
		if (m_circleCallback)
		{
			m_circleCallback(shape);
			return true;
		}
		return false;
	}
	virtual bool tryVisit(Square *shape) override
	{
		if (m_squareCallback)
		{
			m_squareCallback(shape);
			return true;
		}
		return false;
	}
	virtual bool tryVisit(Triangle *shape) override
	{
		if (m_triangleCallback)
		{
			m_triangleCallback(shape);
			return true;
		}
		return false;
	}

	void setCircleCallback(const std::function<void(Circle*)>& circleCallback)
	{
		m_circleCallback = circleCallback;
	}
	void setSquareallback(const std::function<void(Square*)>& squareCallback)
	{
		m_squareCallback = squareCallback;
	}
	void setTriangleCallback(const std::function<void(Triangle*)>& triangleCallback)
	{
		m_triangleCallback = triangleCallback;
	}

private:
	std::function<void(Circle*)> m_circleCallback;
	std::function<void(Square*)> m_squareCallback;
	std::function<void(Triangle*)> m_triangleCallback;
};
{% endhighlight %}

&nbsp;&nbsp;&nbsp;&nbsp;As you can see we can set any behavior for any derived class using lambda expression, and it is really suitable.<br/>
The next example shows how to get individual information for every derived class.
{% highlight C++%}
void dataGetter(Shape* shape)
{
	int radius = 0;
	int width = 0;
	int height = 0;
	int z = 0;
	std::string additionalInfo;
	LazyVisitor lazyVisitor;
	lazyVisitor.setCircleCallback([&](Circle* shape)
	{
		radius = shape->getRadius();
		additionalInfo = "Circle";
	});
	lazyVisitor.setSquareallback([&](Square* shape)
	{
		width = shape->getWidth();
		height = shape->getHeight();
		additionalInfo = "Square";
	});
	lazyVisitor.setTriangleCallback([&](Triangle* shape)
	{
		z = shape->getZ();
		additionalInfo = "Triangle";
	});

	if (shape->tryAccept(lazyVisitor))
	{
		std::cout << "Obtained from: " << additionalInfo.c_str() << " Radius:" << radius << " Width:" << width << " Height:" << height << " Z:" << z << std::endl;
	}
}
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;We set lambda expression for each interested type. If actual type of shape is Circle, lambda expression for Circle type will be called etc.
We also can create new visitor only to manage only the one derived class with any behavior.

{% highlight C++%}
void dataGetterSquareOnly(Shape* shape)
{
	int width = 0;
	int height = 0;
	LazyVisitor lazyVisitor;
	lazyVisitor.setSquareallback([&](Square* shape)
	{
		width = shape->getWidth();
		height = shape->getHeight();
	});

	if (shape->tryAccept(lazyVisitor))
	{
		std::cout << "Shape is Square." << " Width:" << width << " Height:" << height << std::endl;
	}
	else
	{
		std::cout << "Shape is not Square." << std::endl;
	}
}
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;We can create numerous visitors with different behaviors emplace, without visitor class declarations. 
But, we still have many of duplicated codes in the lazy visitor. We can solve this problem by using template version of the lazy visitor with std::tuple.<br/>
&nbsp;&nbsp;&nbsp;&nbsp;In this version, we will have separate components for each derived class. Each component can set callback function and call the “visit” method for certain derived class.
Using this approach, the role of the lazy visitor class is shrink to the collection of visitor components, declared on the compile time.
{% highlight C++%}
template<typename T>
class VizitorComponent
{
public:

	typedef T Type;
	bool tryVisit(T *shape)
	{
		if (m_callback)
		{
			m_callback(shape);
			return true;
		}
		return false;
	}

	void setCallback(const std::function<void(T*)>& callback)
	{
		m_callback = callback;
	}
private:
	std::function<void(T*)> m_callback;
};


template<typename Ttuple>
class LazyVisitor
{
public:
	template<typename T>
	bool tryVisit(T *shape)
	{
		VizitorComponent<T>& component = std::get<VizitorComponent<T>>(m_tuple);
		return component.tryVisit(shape);
	}

	template<typename T>
	void setCallback(const std::function<void(T*)>& callback)
	{
		VizitorComponent<T>& component = std::get<VizitorComponent<T>>(m_tuple);
		return component.setCallback(callback);
	}

private:

	Ttuple m_tuple;
};

 And icing on the cake is lazy visitor parametrized  declaration.

typedef LazyVisitor<std::tuple<
	VizitorComponent<class Circle>,
	VizitorComponent<class Square>,
	VizitorComponent<class Triangle>
> > TVisitor;

{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;Now you can use TVisitor instead of visitor class.
In conclusion, as I have promised, we can write downcast function for any class hierarchy which is using lazy visitor. 
{% highlight C++%}

/////////////////// Downcast implementation ///////////////////
template<typename TDerived, typename TBase>
TDerived* Downcast(TBase* shape)
{
	TDerived* retObject = nullptr;
	TVisitor lazyVisitor;
	lazyVisitor.setCallback<TDerived>([&](TDerived* shapeDerived)
	{
		retObject = shapeDerived;
	});

	shape->tryAccept(lazyVisitor);

	return retObject;
}
{% endhighlight %}
**Link to the project: [Lazy visitor](https://github.com/arturx64/lazy_visitor)**
