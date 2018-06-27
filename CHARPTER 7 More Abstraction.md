**CHAPTER 7 More abstraction**

In the previous chapters,you looked at Python's main built-in object types(numbers,strings,lists,tuples,and dictionries);you peeked at the wealth of built-in functions and standard libraries;and you even created your own functions.Now,only one thing seems to be missing making your own objects.And that's what you do in this chapter.

You may wonder how useful this is.It might be cool to make your kinds of objects,but what would you use them for?Which all the dictionaries and sequences and numbers and strings available,can't you just use them and make the functions do the job?Ctertainly,but making your own objects(and especially types or classes of objects)is a certral concept in Python-so central,in fact,that Python is called an object-oriented language(along with Smalltalk,C++,Java,and many others).In this chapter,you learn how to make objects.You learn about polymorphism and encapsulation,methods and attributes,superclasses,and inheritance-you learn a lot.So let's get started.

> If you're already familiar with the concepts of object-oriented programming,you probably know about constructors.Constructors will not be dealt with in this chapter;for a full discussion,see Chapter 9.

**The Magic of Objects**

In object-oriented programming,the term object loosely means a collection of data(attributes) with a set of methods for accessing and manipulating those data.There are several reasons for using objects instead of sticking with global variables and functions.Some of the most important benefits of objects include the following:

* Polymorphism:You can use the same operations on objects of different classes,and they will work as if "by magic."
* Encapsulation:You hid unimportant details of how objects work from the outside world.
* Inheritance:You can create specialized classes of objects from general ones.

In many presentations of object-oriented programming.the order of these concepts is different.Encapsulation and inheritance are presented first,and then they are used to model real-world objects.That's all fine and dandy,but in my opinion,the most interesting feature of object-oriented programming is polymorphism.It is also the feature that confuses most people(in my experience).Therefore I'll start with polymorphism,and try to show that this concept alone should be enough to make you like object-oriented programming.

**Polymorphism**

The term polymorphism is derived from a Greek word meaning"having multiple forms."Basically,that means that even if you don't know what kind of object a variable refers to,you may still be able to perform operations on it that will work differently depending on the type(or class)of the objects.For example,assume that you are creating an online payment system for a commercial web site that sells food.Your program receives a "shopling cart"of goods from another part of the system(or other similar systems that may be designed in the future)-all you need to worry about is summing up to the total and billing some credit card.

Your first thought may be to specify exactly how the goods must be represented when your program receives them.For example,you may want to receive them as tuples,like this:

```
('SPAM', 2.50)
```

If all you need is a descriptive tag and a price,this is fine.But it's now very flexible,Let's say that some clever person starts and auctioning service as part of the web site-where the price of an item is gradually reduced until someone buys it.It would be nice if the user could put the object in her shopping cart,procced to the checkout(your part of the system),and just wait until the price was right before clicking the Pay button.

But that wouldn't work with the simple tuple scheme.For that to work,the object would need to check its current price(through some network magic)each time your code asked for the price-it couldn't be frozen like in a tuple.You can solve that by making a function:

```
# Don't do it like this...
def getPrice(object):
	if isinstance(object, tuple):
		return object[1]
	else:
		return magic_network_method(object)
```

> The type/class checking and use of isinstance here is meant to illustrate a point-namely that type checking isn't generally a satisfactory solution.Avoid type checking if you possibly can.The function isinstance is described in the section"Investigating Inheritance,"later in this chapter. 

In the preceding code,I use the function isinstance to find out whether the object is a tuple.If it is,its second element is returned;otherwise,some"magic"network method is called.

Assuming that the network stuff already exists, you're solved the problem-for now.But this still isn't very flexible.What if some clever programmer decides that she'll represent the price as a string with a hex value,stored in dictionary under the key'price'?No problem-you just update your function:

```
# Don't do it like this...
def getPrice(object):
	if isinstance(object, tuple):
		return object[1]
	elif isinstance(object, dict):
		return int(object['price'])
	else:
		return magic_network_method(object)
```

Now, surely you must have covered every possibility?But let's say someone decides to add a new type of dictionary with the price stored under a different key.What do you do now?You could certainly update getPrice again,but for how long could you continue doing that?Every time someone wanted to implement some priced object differently,you would need to reimplement your module.But what if you already sold your module and moved on to other,cooler projects-what would the client do then?Clearly,this is an inflexible and impractical way of coding the different behaviors.

So what do you do instead?You let the objects handle the operating themselves.It sounds really obvious,but think about how much easier things will get.Every new object type can retrieve or calculate its own price and return it to you-all you have to do is ask for it.And this is where polymorphism(and, to some extend,encapsulation)enters the scene.

**Polymorphism and Methods**

You receive an object and have no idea of how it is implemented-it may have any one of many "shapes."All you know is that you can ask for its price,and that's enough for you.The way you do that should be familiar:

```
>>>object.getPrice()
2.5
```

Functions that are bound to object attributes like this are called methods. You've already encountered them in the form of string,list,and dictionary methods.There,too,you saw some polymophism:

```
>>>'abc'.count('a')
1
>>>[1, 2, 'a'].count('a')
1
```

If you had a variable x,you wouldn't need to know whether it was a string or a list to call the count method-it wouldn't work regardless(as long as you supplied a single character as the argument).

Let's do an experiment.The standard library random contains a function called choice that selects a random element from a sequence.Let's use that to give your variable a value:

```
>>>from random import choice
>>>x = choice(['Hello, world!', [1, 2, 'e', 'e', 4]])
```

After performing this,x can either contain the string 'Hello, world!' or the list[1, 2, 'e', 'e', 4]-you don't know, and you don't have to worry about it.All you care about is how many times you find 'e' in x,and you can find that out regardless of whether x is a list or a string.By calling the count method as before,you find out just that:

```
>>>x.count('e')
2
```

In this case, it seems that the list won out.But the point is that you didn't need to check.Your only requirement was that x has a method called count that takes a single character as an argument and returned an integer.If someone else had made his own class of objects that had this method,it wouldn't matter to you-you could use his objects just as well as the strings and lists.

**Polymorphism Comes in Many Forms**

Polymorphism is at work every time you can "do something"to an object without having to know exactly what kind of object it is.This doesn't apply only to methods-we've already used polymorphism a lot in the form of built-in operators and functions.Consider the following:

```
>>>1 + 2
3
>>>'Fish' + 'license'
'Fishlicense'
```

Here,the plus operator(+) works fine for both numbers(integers in this case)and strings(as well as other types of sequences).To illustrate the point,let's say you wanted to make a function called add that added two things together.You could simply define it like this(equivalent to,but less efficient than,the add function from the operator module):

```
def add(x, y):
	return x+y
```

This would also work with many kinds of arguments:

```
>>>add(1, 2)
3
>>>add('Fish', 'license')
'Fishlicense'
```

This might seem silly,but the point is that the arguments can be anything that supports addition.If you want to write a function that prints a message about the length of an object,all that's required is that it has a length(that the len function will work on it).

```
def length_message(x):
	print("The length of", repr(x), "is", len(x))
```

As you can see,the function also uses repr,but repr is one of the grand masters of polymorphism-it works with anything.Let's see how:

```
>>>length_message('Fnord')
The length of 'Fnord' is 5
>>>length_message([1, 2, 3])
The length of [1, 2, 3] is 3
```

Many functions and operators are polymorphism-probably most of yours will be,too,even if you don't intend them to be.Just by using polymorphic functions and operators,the polymorphism "rubs off."In fact,virtually the only thing you can do to destroy this polymorphism is to do explicit type checking with functions such as type,`,and issubclass.If you can,you really should avoid destroying polymorphism this way.What matters should be that an object acts the way you want,not whether it is of the right type(or class).

> The form of polymorphism discussed here,which is so central to the Python way of programming,is sometimes called"duck typeing",The term derives from the phrase,"If it quacks like a duke..."For more information,see http://en.wikipedia.org/wiki/Duck_typing.