---
layout: post
title: Smart enum. Enum with built-in enum-to-string, string-to-enum converters.
---
&nbsp;&nbsp;&nbsp;&nbsp;If you frequently use enumerations in C++, you may come across with the task of converting an enum value to string and vice versa. Such languages like C# and Java have mechanism for string/enum conversion. Usually, to achieve this in C++, developers create switch/case constructions or fill enum/string associations into collections. I want to introduce you a macros, which helps you to create enumeration with built in enum to string and string to enum converters without an additional coding.
The next problems, which must be solved to write this kind of enumeration:

1. Suppress enum value initializer for parsing.
2. Convert arguments to string and parsing.
3. Search complexity.

&nbsp;&nbsp;&nbsp;&nbsp;As you know, you can assign initializer values for enumeration list. It causes some difficulties for parsing. Our task is to represent such enumeration list { val1 = 4, val2, val3, val4 = 66, val5 } to internal collections. We will use __VA_ARGS__ to read given values, parse theme as string, and fill internal collections. 
Let’s suppose that we want to make array of enumerator-list obtained from macros arguments. We can write something like that:

{% highlight C++%}
#define SMART_ENUM( EnumName, ...) \
enum EnumName{ __VA_ARGS__  }; \
int enumHolder[] = { __VA_ARGS__ };

SMART_ENUM(ExampleEnum, val1, val2, val3 )
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;But, there is the one problem: we can’t assign any value to enum literal ( for example val2 = 2 ) because __VA_ARGS__ will be expanded for array like:

{% highlight C++%}
int enumHolder[] = { val1, val2 = 2, val3 };
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;To avoid this problem I will use fitter-class for each macros argument:

{% highlight C++%}
struct Fitter
       {
             template <typename Any>
             EnumName& operator = (Any value)
             {
                    return m_value;
             }
             Fitter(EnumName value) :
                    m_value(value)
             { }
             operator EnumName() const
             {
                    return m_value;
             }
       private:
             EnumName  m_value;
       };
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;Each macros argument will be casted to the Fitter class, This class has an overloaded operator =, it helps as to “eat” the next value after enum literal. Constructor of this class coupes with enum literal without initializer as well.
The next array will be represented correctly inside the our macros:

{% highlight C++%}
#define GENERATE_ARRAY_3( p1, p2, p3 ) int arr[] = { (Fitter)p1, (Fitter)p2, (Fitter)p3 };
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;After solving previous problem, we can work with another. Having converted macros arguments to string, we can start looking for enum literals and add them to the separate collection.
 {% highlight C++%}
       std::string strs(#__VA_ARGS__);
       auto startSim = std::string::npos;
       do
       {
             startSim = strs.find_first_of('=');
             if (startSim != std::string::npos)
             {
                    auto endSim = strs.find_first_of(',', startSim);
                    if (endSim != std::string::npos)
                    {
                           strs.erase(startSim, endSim - startSim);
                    }
                    else
                    {
                           strs.erase(startSim, endSim - strs.size());
                    }
             }
       } while (startSim != std::string::npos);
       std::replace(strs.begin(), strs.end(), ',', ' ');
       std::vector<std::string> tokens;
       std::istringstream iss(strs);
       std::copy(std::istream_iterator<std::string>(iss),
             std::istream_iterator<std::string>(),
             std::back_inserter(tokens));
       m_data.m_implementation.init(tokens, arr);
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;The first part of the algorithm deletes all initializer values from the string, after that, we will have peeled string without numbers and '=' symbols. The next step is processing through the std::istream_iterator to get strings associated with enum literals, before that we have to get rid off comas in the our string using std::replace method.<br>
&nbsp;&nbsp;&nbsp;&nbsp;SMART_ENUM has mutable internal structure. Parsed data could be saved in the array or hash table. It helps to manipulate the search complexity for different enumerator-list length. The former gives us O(n) complexity, the latter O(1). This behavior became possible thanks to the type selector idiom.<br>
&nbsp;&nbsp;&nbsp;&nbsp;DATA_SIZE_FACTOR value in the smart enum implementation controls internal data storage type. If the count of the enumerator-list less than DATA_SIZE_FACTOR, the array will be used as internal data storage, otherwise the hash table.<br>

#### **How to use?** <br>
&nbsp;&nbsp;&nbsp;&nbsp;First of all, you have to define your enum, for example:

{% highlight C++%}
SMART_ENUM(ExampleEnum, val1, val2, val3 )
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;Where ExampleEnum – name of your enum; val1, val2, va3 – enumerator-list.
After definition ,you will get the helper class CExampleEnum (name of the helper class formed by next rule: C#enum name#) with 2 methods:

*CExampleEnum::ToEnum*  – converts a string to an enum value.<br>
*CExampleEnum::ToString*  – converts an enum value to string.<br>

If some conversation is not possible, the std::exception will be thrown.<br><br>
**Link to the project: [Smart enum](https://github.com/arturx64/smart_enum.git)**