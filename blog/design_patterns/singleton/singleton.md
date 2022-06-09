---
date: '2022-06-09'
tags:
  - design_patterns
featured: true
authors:
  - quan
slug: singleton
title: Singleton
subTitle: Vạn Kiếm Quy Tông
disableSuggestEdit: true
bannerImage: ./singleton.png
---

# Definition

> [**Singleton**](https://en.wikipedia.org/wiki/Singleton_pattern) is a creational design pattern that lets you ensure that a class has only one instance and provide a global access point to this instance.

## Analogy
Giả sử ta có một chương trình với một nút bật tắt dark mode. Ta không thể tạo ra hai thể hiện, một chỉ để bật và một chỉ để tắt. Do đó chỉ nên có duy nhất một thể hiện của lớp đối tượng `DarkModeButton` được tạo và sử dụng xuyên suốt chương trình. 

## Solution
**Singleton Pattern** đảm bảo chỉ có một và chỉ một thể hiện được tạo ra trong chương trình.

# Implementation
Khi implement Singleton pattern, ta cần tuân thủ các quy tắc sau:
- **Private Constructor** để hạn chế truy cập từ bên ngoài.
- Tạo ra một biến **private static** để đảm bảo biến chỉ được khởi tạo trong class. 
- Có một phương thức **public static** để trả về instance được khởi tạo ở trên.

```ad-info
Tùy từng loại ngôn ngữ mà có nhiều cách implement khác nhau. Chẳng hạn JAVA có thể sử dụng khởi tạo Early/Eager, nhưng C++ thì không. 🤔 Bài viết này được viết dựa trên ngôn ngữ C/C++.
```

## Lazy Initialization
Cách xây dựng này tạo ra instance chỉ khi nào chúng ta cần dùng đến (gọi hàm `getInstance`). Nó trái ngược với Early/Eager khi mà cách khởi tạo đó khởi tạo instance bất chấp chúng ta có sử dụng đến nó hay không.

**Code**
```cpp
class DarkModeButton
{
private:
	// Static instance pointer
	static DarkModeButton* _instance;
	string _name;
	bool _state;

	// Private constructor
	DarkModeButton(string name)
	{
		_name = name;
		_state = false;
		cout << "Button created" << endl << endl;
	}

public:
	void setState(bool state) { _state = state; }
	string getName() { return _name; }

public:
	// Static method to get the instance
	static DarkModeButton* getInstance(string name)
	{
		if (_instance == nullptr)
		{
			_instance = new DarkModeButton(name);
		}
		return _instance;
	}
};

DarkModeButton* DarkModeButton::_instance = nullptr;
```

```ad-important
title: Lưu ý
Luôn phải khởi tạo giá trị cho instance pointer, vì instance chỉ được tạo ra khi có nhu cầu. Nếu không khởi tạo thì vùng nhớ mà instance trỏ đến sẽ là vùng nhớ rác, dẫn đến việc hao phí bộ nhớ không cần thiết.
```

Có thể dùng từ khóa `inline` (có ở C++17) để khởi tạo bên trong class: 

```cpp
inline static DarkModeButton* _instance = nullptr;
```

**Test**

```cpp
int main()
{
	DarkModeButton* button;
	button = DarkModeButton::getInstance("Dark mode");
	cout << button->getName() << endl << endl;
	cout << button << endl;
	
	DarkModeButton* button2;
	button2 = DarkModeButton::getInstance("Zoom out");
	cout << button2->getName() << endl << endl;
	cout << button << endl;
	return 0;
}
```

**Output**

```bat
Button created

Dark mode
000002B3A1507CA0

Dark mode
000002B3A1507CA0
```

Có thể thấy, dù cho hàm `getInstance` được gọi hai lần, nhưng vẫn chỉ có một constructor được tạo ra. Và đồng thời, nếu instance đã được tạo, hàm `getInstance` sẽ trả về đối tượng đã tạo. Địa chỉ của con trỏ `button` và `button2` đều là một, thể hiện cho việc chúng cùng trỏ vào một vùng nhớ.

```ad-bug
title: Pointer Problem!
Class `DarkModeButton` được tạo ra với con trỏ `_instance`, nhưng không có hàm hủy nào được gọi. Vì vẫn chưa có một object nào thực sự được tạo ra lúc run time (`_instance` sinh ra lúc compile time vì nằm trong khai báo class). 
```

Để tự động giải phóng vùng nhớ cho con trỏ `_instance`, ta cần sử dụng con trỏ thông minh ([[C-C++ Tips & Techniques#Smart Pointer]]) như sau:

```cpp
#include <memory>
using namespace std;

class DarkModeButton
{
private:
	inline static std::shared_ptr<DarkModeButton> _instance = nullptr;
	DarkModeButton();
	
public:
	static std::shared_ptr<DarkModeButton> getInstance(string name)
	{
		if (_instance == nullptr)
		{
			_instance = std::sharedd_ptr<DarkModeButton> (new DarkModeButton(name));
		}
		return _instance;
	}
};
```


## Thread Safe Initialization
```ad-bug
title: Trường hợp đa luồng
Cách khởi tạo trên chỉ hoạt động trong môi trường đơn luồng. Giả sử có hai luồng chạy song song và cùng gọi đến hàm `getInstance` trong cùng một thời điểm, lúc này hàm sẽ đồng thời tạo ra hai instance pointer, dẫn đến việc vi phạm nguyên tắc của Singleton.
```

Để khắc phục vấn đề trên, ta không tạo ra instance pointer trong phần khai báo thuộc tính nữa. Thay vào đó, ta tạo luôn một instance/instance pointer bên trong hàm `getInstance`. 

**Code**

```cpp
class DarkModeButton
{
private:
	string _name;
	bool _state;

	// Private constructor
	DarkModeButton(string name)
	{
		_name = name;
		_state = false;
		cout << "Button created" << endl;
	}

public:
	void setState(bool state) { _state = state; }
	string getName() { return _name; }

public:
	// Static method, include static instance pointer
	static DarkModeButton* getInstance(string name)
	{
		static DarkModeButton instance(name);
		return &instance;
	}
	// Use reference data type instead. This is also known as "Meyers Singleton"
	static DarkModeButton& getInstance(string name)
	{
		static DarkModeButton instance(name);
		return instance;
	}

	// Both initialization is only thread safe for C++11.
};
```

**Test**

Sử dụng instance pointer.
```cpp
int main()
{
	DarkModeButton* button;
	button = DarkModeButton::getInstance("Dark mode");
	cout << button->getName() << endl;
	cout << button << endl;

	button = DarkModeButton::getInstance("Zoom out");
	cout << button->getName() << endl;
	cout << button << endl;
	return 0;
}
```

Sử dụng reference instance.
```cpp
int main()
{
	DarkModeButton button = DarkModeButton::getInstance("Dark mode");
	cout << button.getName() << endl;
	cout << &button << endl;	
	
	button = DarkModeButton::getInstance("Zoom out");
	cout << button.getName() << endl;
	cout << &button << endl;

	return 0;
}
```

```ad-warning
Hai cách xây dựng hàm `getInstance` trên chỉ thread safe với C++11.
```

# Use Cases
Sử dụng khi ta muốn:
- Đảm bảo rằng chỉ có một instance của lớp.
- Quản lý việc truy cập tốt hơn vì chỉ có một instance.
- Có thể quản lý số lượng thể hiện của một lớp trong giới hạn nhất định.
