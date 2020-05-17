---

layout: post

title: "ADL - why we can use '<<' operator without calling proper namespace?"

categories: [c++]
tags: [short]

---

During daily c++ skills improvment I met some interesting abbrevation - ADL - which stand from "Argument-dependent lookup", or is better known as [Koenig lookup](https://en.wikipedia.org/wiki/Andrew_Koenig_%28programmer%29). I have never wondered on this implicit mechanism which is exposed i.e. when we try to pass data to _std::cout_ stream.
```c++
std::cout << "Some string";
```
But we will back to this example later.

How we can define ADL? In simple, when we try to pass to a function some argument from the same namespace, then we don't need explicit call namespace of the function. For better understanding some code snippet below:
```c++
#include <iostream>
namespace first {
	struct data{};
	void func(data item) {
		std::cout << "In first namespace" << std::endl;
	};
}

namespace second {
	struct data{};
}

void func(second::data item) {
	std::cout << "Function from global scope" << std::endl;
}

int main() {
	func(second::data());
	func(first::data());

	return 0;
}
```
Output:
>Function from global scope

>In first namespace

As we can see, there is unnecessary to explicit call namespace name.

Now we can return to our question about << operator. ``std::cout << "Some string";`` this code can be transformed into function call ``operator<<(std::cout, "Some string");``. Now we can see, that the left argument goes from _std_ namespace, that is why compiler is searching in this namespace first. But someone can ask now, why we must call explicit namespace in endl invocation? That's because ``std::cout << std::endl;`` is not a function call and ADL dose note apply in this case. The only way to do this is to call ``endl(std::cout);`` explicit. As we can see, we call function endl with an argument from std, so again compiler is searching in std namespace.

My conclusion is that ADL can speed up writing code, but also can lead to some misunderstood when we have some functions names same as in other namespaces. We do not need to call proper namespece for function every time. But we can be hurt with this ADL. What will be the behavior of this code?
```c++
#include <iostream>
#include <algorithm>

namespace first {
	struct data{};
	void swap(data& item, data& secondItem) {
		std::cout << "In first namespace" << std::endl;
	};
}

int main() {
	auto firstArg = first::data();
	auto secondArg = first::data();
    
	using std::swap;
	swap(firstArg, secondArg);

	return 0;
}
```
Probably, user wants to use _std::swap_ but ADL will search first in the namespace of an arguments and find other implementation.

# Resources:
[Argument-dependent lookup cppreference.com](https://en.cppreference.com/w/cpp/language/adl)

