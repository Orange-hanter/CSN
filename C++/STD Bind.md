*What is std::bind?*
https://en.cppreference.com/w/cpp/utility/functional/bind

**Example 1:**
Task: without reduplication of code change order polatity of arguments throwing at function 
```c++
[[include]] <iostream>
[[include]] <functional>

template<class T>
void pp(T x1, T x2 ) {

	 std::cout << x1 << ' ' << x2 << '\n';
}

  

int main() {

	 using namespace std::placeholders;

	 auto v1 = std::bind(pp<int>, _2, _1);
	 auto v2 = std::bind(pp<int>, _1, _2);

	 v1(1,2);
	 v2(1,2);
}

```

Result of runing:
```text
2 1
1 2
```


**Example 2:**
Task: Bind at one function two different calc function and twist order of throwing arguments in forwart order (seckond argument is for internal function)
```c++
[[include]] <iostream>
[[include]] <functional>

template<typename T>
int external(T x, T y) {
 return x*100 + y;
}

template<typename T>
int internal(T x) {
	 return x + 1;
}

int main() {
	 using namespace std::placeholders;
	 auto foobar = std::bind(external<int>, std::bind(&internal<int>, _2), _1);
	 
	 std::cout << foobar(1,2) 
			 << '\n'
			 << foobar(2,2);
}

```

Result of runing:
```text
301
202
```


TODO: write about ref/cref