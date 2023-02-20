### 10.3.2 lambda 表达式
```C++
[capture list](parameter list) -> return type{ function body}
```
其中capture list(捕获列表)是一个lambda所在函数中定义的局部变量的列表(通常为空);  
return type,paremeter list和function body 与其他普通函数一元,返回类型,参数列表和函数体  
但与普通函数不同,lambda必须使用尾置返回来指定返回类型  
例子:定义如下lambda表达式f  
```c++
auto f = []{return 42;};
```
该例子定义了一个可调用对象f,不接受参数,返回42.  
lambda的调用方式与普通函数的调用方式相同,都是使用调用运算符:  
```C++
cout << f() << endl; //打印42
```
如果忽略返回类型,lambda根据函数体中的代码推断出返回类型.如果函数体只是一个return语句,则返回类型从返回的表达式的类型推断而来,否则返回类型为void.  
__向lambda传递参数__  
与普通函数不同的是,lambda不能有默认参数.
带参例子:
```C++
[](const string &a, const string &b){return a.size()<b.size()};
```
如下所示,可以使用此lambda调用stable_sort:  
```C++
stable_sort(words.begin(), words.end(), [](const string &a, const string &b){return a.size < b.size();});
```

__使用捕获列表(略)__  
__for_each算法__  
此算法接受一个可调用对象,并对输入序列中每个元素调用此对象:  
```C++
for_each(wc, words.end(), [](const string &s){cout << s <<" ";});
cout << endl;
```
### 10.3.3 lambda捕获和返回  
当定义一个lambda时,编译器生成一个与lambda对应的新的(未命名的)类类型.  

<table>
	<tr>
	    <th colspan="2">表10.1: lambda捕获列表</th>
	</tr >
	<tr>
	    <td>[]</td>
	    <td>空捕获列表.lambda不能使用所在函数中的变量.一个lambda只有捕获变量后才能使用它们</td>
	</tr>
	<tr>
	    <td>[names]</td>
	    <td>names是一个逗号分隔的名字列表,这些名字都是lambda所在函数的局部变量.默认情况下,捕获列表中的变量都被拷贝.名字前如果使用了&,则采用引用捕获方式</td>
	</tr>
	<tr>
	    <td>[&}</td>
	    <td>隐式捕获列表,采用引用捕获方式.lambda体中所使用的来自所在函数的实体都采用引用方式使用</td>
	</tr>
	<tr>
	    <td>[=]</td>
	    <td>隐式捕获列表,采用值捕获方式.lambda体将拷贝所使用的来自所在函数的实体的值</td>
	</tr>
	<tr>
	    <td>[&, identifier_list]</td>
	    <td>identifier_list是一个逗号分隔的列表,包含0个活多个来自所在函数的变量.这些变量采用值捕获的方式,而任何隐式捕获的变量都采用引用方式捕获. identifier_list中的名字前面不能使用&</td>
	</tr>
	<tr>
	    <td>[=, identifier_list]</td>
	    <td>identifier_list中的变量都采用引用方式捕获,而任何隐式捕获的变量都采用值方式捕获.identifier_list的名字不能包括this,且这些名字之前必须使用&</td>
	</tr>
</table>


__值捕获__  
类似参数传递,变量的捕获方式也可以是值或引用.  
与传值参数类似,采用值捕获的前提是变量可以拷贝.与参数不同,被捕获的变量的值是在lambda创建时拷贝,而不是调用时拷贝:也就是下面j是42,而不是0.  
```C++
void fcn1()
{
	size_t v1 = 42; //局部变量
	// 将v1拷贝到名为f的可调用对象  
	auto f = [v1]{ return v1; };
	v1 = 0;
	auto j = f();  //j为42,f保存了创建它时的v1的拷贝
}
```

__引用捕获__  
定义lambda时可以采用引用方式捕获变量.
```C++
void fcn1()
{
	size_t v1 = 42; //局部变量
	// 对象f包含v1的引用 
	auto f = [&v1]{ return v1; };
	v1 = 0;
	auto j = f();  //j为0,f保存了创建它时的v1的引用而非拷贝
}
```
当使用lambda函数体内使用此变量时,实际上使用的是引用所绑定的对象.在本例子中,当lambda返回v1时,它返回的是v1指向的对象的值.  
引用捕获与返回引用有相同的限制和问题: 采用引用方式捕获变量,就必须确保被引用的对象在lambda执行时是存在的.  
lambda捕获的都是局部变量,这些变量在函数结束后就不复存在,如果lambda可能在函数结束后执行,捕获的引用指向的局部变量是消失的.  

**引用捕获有时是必要的.例如希望biggies函数接受一个ostream的引用,用来输出数据,并接受一个字符作为分隔符:**
```C++
void biggies(vector<string> &words, vvector<string>::size_type sz, ostream &os = cout, char c = ' ')
{
	// 打印count的语句改为打印到os
	for_each(words.begin(), words.end(),[&os, c](const string &s){os<<s<<c;});
}
```
因为不能拷贝ostream对象,因此捕获os的唯一方法就是捕获其引用(或指向os的指针)  

**隐式捕获**  
上面是显示列出希望使用的来自所在函数的变量, 还可以让编译器根据lambda中的代码来推断要使用那些变量.(隐式捕获)  
为了指示编译器推断捕获列表,应在捕获列表中写一个&或=. &告诉编译器采用捕获引用方式, =则表示采用值捕获方式.  
例如
```C++
// sz为隐式捕获,值捕获方式
wc = find_if(words.begin(), words.end(),
	[=](const string &s){return s.size()>=sz;});
```

如果希望对一部分变量采用值捕获,对其他变量采用引用捕获,可以混合使用隐式捕获和显示捕获:
```C++
void biggies(vector<string> &words,vector<string>::size_type sz,ostream &os = cout, char c = ' ')
{
	// 其他处理与前例一样
	// os隐式捕获,引用捕获,c显式捕获,值捕获
	for_each(words.begin(), words.end(),
	[&, c](const string &s){os<<s<<c;});
	// os显式捕获,引用捕获,c隐式捕获,值捕获
	for_each(words.begin(), words.end(),
	[=, &os](const string &s){os<<s<<c;});
}
```
**可变lambda**  
默认情况下，对于一个值被拷贝的变量，lambda不会改变其值，如果希望改变一个被捕获的变量的值，就必须在参数列表首加上关键字<font color=red>***mutable***</font>。因此可变lambda能省略参数列表：  
```C++
void fcn3()
{
	size_t v1 = 42;  //局部变量  
	// f可以改变它所捕获的变量的值
	auto f = [v1]()mutable{return ++v1;};
	v1 = 0;
	auto j = f();  //j为43
}
```
但其实不如直接引用捕获也能改

**指定lambda返回类型**  
```C++
transform(v1.begin(), v1.end(), v1.begin(),[](int i) -> int {if(i<0> return -i; else return i;)});
```
### 10.3.4 参数绑定  
当需要很多地方使用相同的lambda操作时,通常希望定义一个函数,而不是编写多次相同的lambda表达式  

对于捕获局部变量的lambda,用函数来替换它就不容易了.   
**标准库的bind函数**  
bind定义在头文件functional中,可以将bind函数看作一个通用的函数适配器,它接受一个可调用对象,生成一个新的可调用对象来适应原对象的参数列表.  
调用bind的一般形式为:
```C++
auto newCallable = bind(callable, arg_list);
```
当调用newCallable时,newCallable会调用callable,并传递给callabld,arg_list中的参数  
(略,有点没弄清楚)

### 10.4 再探迭代器  
标准库在头文件iterator中还定义了额外几种迭代器.  
- 插入迭代器(insert iterator): 这些迭代器被绑定到一个容器上,可用来向容器插入元素  
- 流迭代器(stream iterator): 这些迭代器被绑定到输入或输出流上,可用来遍历所关联的IO流  
- 反向迭代器(reverse iterator): 这些迭代器向后而不是向前移动.除了forwar_list之外的标准库容器都有反向迭代器.  
- 移动迭代器(move iterator): 这些专用的迭代器不是拷贝其中的元素,而是移动它们.  

10.4.1 插入迭代器  
插入器是一种迭代器适配器,它接受一个容器,生成一个迭代器,能实现向给定容器添加元素.  
当通过一个插入迭代器进行赋值时,该迭代器调用容器操作来向给定容器的指定位置插入一个元素.  
<table>
	<tr>
		<th colspan="2">表10.2:插入迭代器操作</th>
	</tr>
	<tr>
		<td>it = t</td>
		<td>在it指定的当前位置插入值t. 假定c是it绑定的容器,依赖于插入迭代器的不同种类,此赋值会分别调用c.push_back(t),c.push_front(t)或c.insert(t,p),其中p为传递给inserter的迭代器位置</td>
	</tr>
	<tr>
		<td>*it,++it,it++</td>
		<td>这些操作虽然存在,但不会对it做任何事情,每个操作都返回it</td>
	</tr>
</table>

插入器有三种类型,差异在于元素插入的位置:  
- back_inserter创建一个使用push_back的迭代器  
- front_inserter创建一个使用push_front的迭代器  
- inserter创建一个使用insert的迭代器. 此函数接受第二个参数,这个参数必须是一个指向给定容器的迭代器.元素将被插入到给定迭代器所表示的元素之前.

**10.4.2 iostream迭代器**  
标准库定义了可以用于IO类型对象的迭代器.  
istream_iterator读取输入流,ostream_iterator向一个输出流写数据.  
这些迭代器将它们对应的流当作一个特定类型的元素序列来处理.通过使用流迭代器,可以用泛型算法从流对象读取数据以及向其写入数据  
**istream_iterator操作**  
感觉用不到略到10.4.3反向迭代器  

10.4.3 反向迭代器  
反向迭代器就是在容器中从尾元素向首元素反向移动的迭代器.(学过了,略)  

### 10.5 泛型算法结构  
任何算法的最基本的特性是它要求其迭代器提供哪些操作.  
算法要求的迭代器操作可以分为5个迭代器类型(iterator category). 每个算法都会对它的每个迭代器参数指明需提供哪些迭代器  
<table>
	<tr>
		<th colspan ="2">表10.5::迭代器类别</th>
	</tr>
	<tr>
		<td>输入迭代器</td>
		<td>只读,不写:单遍扫描,只能递增</td>
	</tr>
	<tr>
		<td>输出迭代器</td>
		<td>只写,不读:单遍扫描,只能递增</td>
	</tr>
	<tr>
		<td>前向迭代器</td>
		<td>可读写:多遍扫描,只能递增</td>
	</tr>
	<tr>
		<td>双向迭代器</td>
		<td>可读写:多遍扫描,可递增递减</td>
	</tr>
	<tr>
		<td>随机访问迭代器</td>
		<td>可读写:多遍扫描,支持全部迭代器运算</td>
	</tr>
</table>

第二种算法分类的方式是按照是否读,写或是重排序列中的元素来分类.  
**10.5.1 5类迭代器**  
类似容器,迭代器也定义了一组公告操作. 一些操作所有迭代器都支持,另外一些只有特定类别的迭代器猜支持.  
例如ostream_iterator只支持递增,解引用和赋值.vector,string和deque的迭代器除了这些操作外,还支持递增,关系和算术运算.  
**迭代器类别**  
输入迭代器(input iterator):可以读取序列中的元素.一个输入迭代器必须支持:  
- 用于比较两个迭代器的相等和不相等运算  
- 用于推进迭代器的前置和后置递增运算(++)
- 用于读取元素的解引用运算符(*):解引用只会出现在赋值运算符的右侧
- 箭头运算符(->),等价于(*it).member,即解引用迭代器,并提取对象的成员  

输入迭代器只用于顺序访问.算法find和accumulate要求输入迭代器,而istream_iterator是一种输入迭代器  

输出迭代器(output iterator):可以看作输入迭代器功能上的补集--只写而不读元素. 输出迭代器必须支持:  
- 用于推进迭代器的前置和后置递增运算(++)  
- 解引用运算符(*),只出现在赋值运算符的左侧(向一个已经解引用的输出迭代器赋值,就是将值写入它所指向的元素)  
ostream_iterator是一种输出迭代器  

前向迭代器(forward iterator): 可以读写元素.这类迭代器只能在序列中沿一个方向移动.前向迭代器支持所有输入和输出迭代器的操作,而且可以多次读写同一个元素.算法replace要求前向迭代器,forward_list上的迭代器是前向迭代器.  

双向迭代器(bidirectional iterator): 可以正向/反向读写序列中的元素.除了支持所有前向迭代器的操作外,还支持前置和后置递减运算符(--).算法reverse要求双向迭代器,除了forward_list之外,其他标准库都提供符合双向迭代器要求的迭代器.  

随机访问迭代器(random-access iterator): 提供在常量时间内访问序列中任意元素的能力.此类迭代器支持双向迭代器的所有功能.还支持表3.7(就是vector和string迭代器支持的运算,加减n移动,迭代器间加减,关系运算符)
- 用于比较两个迭代器相对位置的关系运算符  
- 迭代器和一个整数值的加减运算(表3.7)
- 用于两个迭代器上的减法运算符(-),得到距离(表3.7)
- 下标运算符(iter[n]),与*(iter[n])等价  
**算法sort要求随机访问运算符,array,deque,string和vector的迭代器都是随机访问迭代器,用于访问内置数组元素的指针也是**

10.5.2 算法形参模式  
在任何算法分类之上,还有一组参数规范.  
大多数算法具有如下4种形式之一:  
```C++
alg(beg, end, other args);
alg(beg, end, dest, other args);
alg(beg, end, beg2, other args);
alg(beg, end, beg2, end2, other args);
```
dest指的是目的位置.除了这些迭代器参数, 一些算法还接受额外的,非迭代器的特定参数.  

10.5.3 算法命名规范  
除了参数规范,算法还遵循一套命名和重载规范.例如: 如何提供一个操作代替默认的<或==运算符以及算法是将输出数据写入输入序列还是一个分离的目的位置

**区分拷贝元素的版本和不拷贝的版本**  
算法都在名字后面附加一个_copy  
```C++
reverse(beg, end);  //反转输入范围中元素的顺序
reverse_copy(beg, end, dest);  //将元素逆序拷贝到dest
```

一些算法同时提供_copy和if版本. 这些版本接受一个目的位置迭代器和一个谓词:  
```C++
// 从v1中删除奇数元素
remove_if(v1.begin(), v1.end(), [](int i){return i%2;});
remove_copy_if(v1.begin(), v1.end(), back_inserter(v2), [int i]{return i % 2;});
```
**10.6 特定容器算法**  
与其他容器不同,链表类型list和forward_list定义了几个成员函数形式的算法.  
特别是,它们定义了独有的sort, merge, remove, reverse和unique.通用版本的sort要求随机访问迭代器,因此不能用于list和forward_list,因为这两个类型版本分别提供双向迭代器和前向迭代器.  
对于list和forward_list,应该优先使用成员函数版本的算法而不是通用算法.  
<table>
	<tr>
		<th colspan="2">表10.6: list和forward_list成员函数版本的算法(这些操作都返回void</th>
	</tr>
	<tr>
		<td>lst.merge(lst2)</td>
		<td>将来自lst2的元素合并如lst.lst和lst2必须是有序的</td>
	</tr>
	<tr>
		<td>lst.merge(lst2, comp)</td>
		<td>元素将从lst2中删除. 在合并之后lst2变成空.第一个版本使用< 运算符第二个版本使用给定的比较操作</td>
	</tr>
	<tr>
		<td>lst.remove(val)</td>
		<td>调用erase删除掉与给定值相等(==)或令一元谓词为真的每个元素</td>
	</tr>
	<tr>
		<td>lst.remove_if(pred)</td>
		<td>调用erase删除掉与给定值相等(==)或令一元谓词为真的每个元素</td>
	</tr>
	<tr>
		<td>lst.reverse()</td>
		<td>反转lst中元素的顺序</td>
	</tr>
	<tr>
		<td>lst.sort()</td>
		<td>使用< 或给定比较操作排序元素</td>
	</tr>
	<tr>
		<td>lst.sort(comp)</td>
		<td>使用< 或给定比较操作排序元素</td>
	</tr>
	<tr>
		<td>lst.unique()</td>
		<td>调用erase删除同一个值的连续拷贝.第一个版本使用==,第二个版本使用给定的二元谓词</td>
	</tr>
	<tr>
		<td>lst.unique(pred)</td>
		<td>调用erase删除同一个值的连续拷贝.第一个版本使用==,第二个版本使用给定的二元谓词</td>
	</tr>
</table>

<table>
	<tr>
		<th colspan="2">表10.7: list和forward_list的splice成员函数版本的算法</th>
	</tr>
	<tr>
		<th colspan="2">lst.splice(args)||或flst.splice_after(args)</th>
	</tr>
	<tr>
		<td>(p,lst2)</td>
		<td>p是一个指向lst中元素的迭代器,或一个指向flst首前位置的迭代器.函数将lst2的所有元素移动到lst中p之前的位置||或是flst中p之后的位置.将元素从lst2中删除.lst2的类型必须与lst或flst相同,且不能是同一个链表</td>
	</tr>
	<tr>
		<td>(p,lst2,p2)</td>
		<td>p2是一个指向lst2位置的有效的迭代器.将p2指向的元素移动到lst中,||或将p2之后的元素移动到flst中.lst2可以是与lst或flst相同的链表</td>
	</tr>
	<tr>
		<td>(p,lst2,b,e)</td>
		<td>b和e必须表示lst2中的合法范围.将给定范围的元素从lst2移动到lst或flst. lst2与lst(或flst)可以是相同的链表,但p不能指向给定范围中的元素</td>
	</tr>
</table>

**链表特有的操作会改变容器**  
多数链表特有的算法都与其通用版本和相似,但不完全相同.链表特有版本与通用版本间的一个至关重要的区别是链表版本会改变底层的容器.例如,remove的链表版本会删除指定的元素.unique的链表版本会删除第二个和后继的重复元素.等等

**小结:ref标准库函数,从一个指向不能拷贝的类型的对象的引用生成一个可拷贝的对象.**

### chapter11 关联容器  
- 11.1 使用关联容器
- 11.2 关联容器概述
- 11.3 关联容器操作
- 11.4 无序容器

关联容器和顺序容器有着根本的不同：关联容器中的元素是按关键字来保存和访问的。顺序容器中的元素是按它们在容器中的位置来顺序保存和访问的。  
虽然关联容器的很多行为与顺序容器相同，但其不同之处反映了关键字的作用。

关联容器支持高效的关键字查找和访问。两个主要的关联容器(associative-container)类型是map和set。  
map中的元素是一些关键字-值(key-value)对:关键字起到索引的作用,值则表示与索引相关联的数据.  
set中每个元素只包含一个关键字:set支持高效的关键字查询操作--检查一个给定关键字是否在set中.例如,在某些文本处理过程中,可以用一个set来保存想要忽略的单词.字典则是一个很好的使用map的例子:可以将单词作为关键字,将单词释义作为值.

标准库提供8个关联容器,如表11.1所示.
<table>
	<tr>
		<th colspan="2">表11.1:关联容器类型</th>
	</tr>
	<tr>
		<td>按关键字有序保存元素</td>
		<td> </td>
	</tr>
	<tr>
		<td>map</td>
		<td>关联数组:保存关键字-值对</td>
	</tr>
	<tr>
		<td>set</td>
		<td>关联字即值:即只保存关键字的容器</td>
	</tr>
	<tr>
		<td>multimap</td>
		<td>关键字可重复出现的map</td>
	</tr>
	<tr>
		<td>multiset</td>
		<td>关键字可重复出现的set</td>
	</tr>
	<tr>
		<td>无序集合</td>
		<td> </td>
	</tr>
	<tr>
		<td>unordered_map</td>
		<td>用哈希函数组织的map</td>
	</tr>
	<tr>
		<td>unordered_set</td>
		<td>用哈希函数组织的set</td>
	</tr>
	<tr>
		<td>unordered_multimap</td>
		<td>关键字可重复出现的哈希组织的map</td>
	</tr>
	<tr>
		<td>unordered_multiset</td>
		<td>关键字可重复出现的哈希组织的set</td>
	</tr>
</table>

11.1 使用关联容器  
map是关键字-值对的集合.map类型通常被称为关联数组(associative array).关联数组与"正常数组"类似,不同在于其下标不不必是整数.通过一个关键字而不是位置来查找值.  

与之相对,set是关键字的简单集合.当只想知道一个值是否存在时,set就发挥用处了.

**使用map**  
一个经典的使用关联数组的例子是单词计数程序：  
```C++
// 统计每个单词在输入中出现的次数
map<string, size_t> word_count;  //string到size_t的空map 
string word;
while (cin >> word)
	++word_count[word];
for (const auto &w: word_count)
	cout << w.first << " occurs " << w.second << ((w.second > 1) ? " times " : " time ") << endl;
```
类似顺序容器,关联容器也是模板. 为定义一个map,必须指定关键字和值的类型.  
当对word_count进行下标操作时，使用一个string作为下标，获得与此string相关联的size_t类型的计数器.  
**在用下标访问元素时，vector使用vector::size_type作为下标类型，而数组下标的正确类型则是size_t。vector使用的下标实际也是size_t，源码是typedef size_t size_type。**

范围for语句遍历map,打印每个单词和对应的计数器.当从map中提取一个元素时,得到一个pair类型的对象.pair是一个模板类型,保存两个名为first和second的(公有)数据成员.map所使用的pair用first成员保存关键字,用second成员保存对应的值.  

使用set
可以使用set保存想忽略的单词,只对不在集合中的单词统计出现次数:
```C++
// 统计输入中每个单词出现的次数
map<string, size_t> word_count;  //string
到size_t的空map
set<string> exclude = {"The","But","And","Or","An","A","the","but","and","or","an","a"};
string word;
while(cin >> word)
	// 只统计不在exclude中的单词
	if (exclude.find(word)==exclude.end())
		++word_cound[word];  //获取并递增word的计数器
```
与其他容器类似,set也是模板.find调用返回一个迭代器.如果给定关键字在set中,迭代器指向该关键字.否则,find返回尾后迭代器.  

11.2 关联容器概述  
关联容器(有序和无序)都支持9.2中介绍的普通容器操作.
<table>
	<tr>
		<th colspan="2">表9.2:容器操作</th>
	</tr>
	<tr>
		<td>类型别名</td>
		<td> </td>
	</tr>
	<tr>
		<td>iterator</td>
		<td>此容器类型的迭代器类型</td>
	</tr>
	<tr>
		<td>const_iterator</td>
		<td>可以读取元素,但不能修改元素的迭代器类型</td>
	</tr>
	<tr>
		<td>size_type</td>
		<td>无符号整数类型,足够保存此种容器类型最大可能容器的大小</td>
	</tr>
	<tr>
		<td>different_type</td>
		<td>带符号整数类型,足够保存两个迭代器之间的距离元素类型</td>
	</tr>
	<tr>
		<td>value_type</td>
		<td>元素类型</td>
	</tr>
	<tr>
		<td>reference</td>
		<td>元素的左值类型,与value_type&含义相同</td>
	</tr>
	<tr>
		<th colspan="2">构造函数</th>
	</tr>
	<tr>
		<td>C c;</td>
		<td>默认构造函数,构造空容器(array</td>
	</tr>
	<tr>
		<td>C c(c2);</td>
		<td>构造c2的拷贝c1</td>
	</tr>
	<tr>
		<td>C c(b,e);</td>
		<td>构造c,将迭代器b和e指定的范围内的元素拷贝到c(array不支持)</td>
	</tr>
	<tr>
		<th colspan="2">赋值与swap</th>
	</tr>
	<tr>
		<td>c1 = c2;</td>
		<td>将c1中的元素替换为c2中元素</td>
	</tr>
	<tr>
		<td>c1={a,b,c...};</td>
		<td>将c1中的元素替换为列表中元素(不适合array)</td>
	</tr>
	<tr>
		<td>a.swap(b);</td>
		<td>交换a和b的元素</td>
	</tr>
	<tr>
		<td>swap(a,b)</td>
		<td>与a.swap(b)等价</td>
	</tr>
	<tr>
		<th colspan="2">大小</th>
	</tr>
	<tr>
		<td>c.size();</td>
		<td>c中元素的数目(不支持forward_list)</td>
	</tr>
	<tr>
		<td>c.max_size();</td>
		<td>c中可保存的最大元素数目</td>
	</tr>
	<tr>
		<td>c.empty();</td>
		<td>c中有元素则返回false,否则返回true</td>
	</tr>
	<tr>
		<th colspan="2">添加/删除元素(不适合array)在不同容器中,这些操作的接口都不同</th>
	</tr>
	<tr>
		<td>c.insert(args);</td>
		<td>将args中的元素拷贝进c</td>
	</tr>
	<tr>
		<td>c.emplace(inits);</td>
		<td>使用inits构造c中的一个元素</td>
	</tr>
	<tr>
		<td>c.erase(args);</td>
		<td>删除args指定的元素</td>
	</tr>
	<tr>
		<td>c.clear()</td>
		<td>删除c中的所有元素,返回void</td>
	</tr>
	<tr>
		<th colspan="2">关系运算符</th>
	</tr>
	<tr>
		<td>==,!=</td>
		<td>所有容器都支持相等(不等)运算符</td>
	</tr>
	<tr>
		<td><,<=,>,>=;</td>
		<td>关系运算符(无序关联容器不支持)</td>
	</tr>
	<tr>
		<th colspan="2">获取迭代器</th>
	</tr>
	<tr>
		<td>c.begin(),c.end()</td>
		<td>返回指向c的首元素和尾元素之后位置的迭代器</td>
	</tr>
	<tr>
		<td>c.cbegin(),c.cend();</td>
		<td>返回const_iterator</td>
	</tr>
	<tr>
		<th colspan="2">反向容器的额外成员(不支持forward_list)</th>
	</tr>
	<tr>
		<td>reverse_iterator</td>
		<td>按逆序寻址元素的迭代器</td>
	</tr>
	<tr>
		<td>const_reverse_iterator</td>
		<td>不能修改元素的逆序迭代器</td>
	</tr>
	<tr>
		<td>c.rbegin(),c.rend()</td>
		<td>返回指向c的尾元素和首元素之前位置的迭代器</td>
	</tr>
	<tr>
		<td>c.crbegin(),c.crend()</td>
		<td>返回const_reverse_iterator</td>
	</tr>
</table>

关联容器不支持顺序容器的位置相关的操作,例如push_front或push_back.原因是关联容器中元素是根据关键字存储的,这些操作对关联容器没有意义.而且关联容器也不支持构造函数或插入操作这些接受一个元素值和一个数量值的操作.  

关联容器还支持一些顺序容器不支持的操作和类型别名.无序容器还提供一些用来调整哈希性能的操作.  
**关联容器的迭代器都是双向的**  

11.2.1 定义关联容器  
初始化multimap或multiset  
```c++
// 定义一个有20元素的vector,保存0-9每个整数的两个拷贝
vector<int> ivec;
for (vector<int>::size_type i=0;i!=10;++i)
{
	ivec.push_back(i);
	ivec.push_back(i);
}
// iset包含来自ivec的不重复元素,miset包含所有20个元素
set<int> iset(ivec.cbegin(),ivec.cend());
multiset<int> miset(ivec.cbegin(),ivec.cend());
```

11.2.2 关键字类型的要求(略)  
11.2.3 pair类型  
在介绍关联容器操作之前,需要了解名为pair的标准库类型,它定义在头文件utility中  
一个pair保存两个数据成员.类似容器,pair是一个用来生成特定类型的模板.  
当创建一个pair时,必须提供两个类型名,pair的数据成员将具有对应的类型.
两个类型不要求一样:
```c++
pair<string, string> anon; //保存两个string
pair<string, size_t> word_count;
pair<string, vector<int>> line;
```
<table>
	<tr>
		<th colspan="2">表11.2:pair上的操作</th>
	</tr>
	<tr>
		<td>pair<T1,T2> p;</td>
		<td>p是一个pair,两个类型分别是T1和T2的成员都进行了值初始化</td>
	</tr>
	<tr>
		<td>pair<T1,T2> p(v1,v2)</td>
		<td>p是一个成员类型为T1,T2的pair;first和second成员分别用v1和v2初始化</td>
	</tr>
	<tr>
		<td>pair<T1,T2> p={v1,v2}</td>
		<td>等价于p(v1,v2)</td>
	</tr>
	<tr>
		<td>make_pair(v1,v2)</td>
		<td>返回一个用v1和v2初始化的pair.pair的类型从v1和v2的类型推断而来</td>
	</tr>
	<tr>
		<td>p.first;</td>
		<td>返回p的名为first的(公有)数据成员</td>
	</tr>
	<tr>
		<td>p.second;</td>
		<td>返回p的名为second的(公有)数据成员</td>
	</tr>
	<tr>
		<td>p1 relop p2</td>
		<td>关系运算符(<,>,<=,>=)按字典序定义:例如p1.first< p2.second或!(p2.first< p1.first) && p1.second< p2.second成立,p1< p2为true,关系运算利用元素的< 运算符来实现 </td>
	</tr>
	<tr>
		<td>p1 == p2</td>
		<td>当first和second成员分别相等时,两个pair相等.相等性判断利用元素的==运算符实现</td>
	</tr>
	<tr>
		<td>p1 != p2</td>
		<td> </td>
	</tr>
</table>

创建pair对象的函数  
```C++
pair<string, int>
process(vector<string> &v)
{
	// 处理v
	if(!v.empty())
		return {v.back(),v.back().size()}; //列表初始化
	// 或
		return pair<string, int>(v.back(), v.back().size());
	// 或
		return make_pair(v.back(), v.back().size());
	else
		return pair<string, int>();  //隐式构造返回值
}
```

11.3 关联容器操作  
除了9.2中列出的类型,关联容器还定义了表11.3中列出的类型.  
这些类型表示容器关键字和值的类型  
<table>
	<tr>
		<th colspan="2">表11.3:关联容器额外的类型别名</th>
	</tr>
	<tr>
		<td>key_type</td>
		<td>此容器类型的关键字类型</td>
	</tr>
	<tr>
		<td>mapped_type</td>
		<td>每个关键字关联的类型:只适用于map</td>
	</tr>
	<tr>
		<td>value_type</td>
		<td>对于set,与key_type相同,对于map,为pair<const key_type, mapped_type></td>
	</tr>
</table>

map的每个元素是一个pair对象，包括一个关键字和一个关联的值。由于不能改变一个元素的关键字，因此pair的关键字部分是const的。  
与顺序容器一样，使用作用域运算符来提取一个类型的成员：例如，map<string, int>::key_type  
只有map类型(unordered_map, unordered_multimap,multimap和map)才定义了mapped_type

11.3.1 关联容器迭代器  
当解引用一个关联容器迭代器时,会得到一个类型为容器的value_type的值的引用.对map而言,value_type是一个pair类型,其first成员保存const的关键字,second成员保存值.  
```c++
// 获得指向word_count中一个元素的迭代器
auto map_it = word_count.begin();
// *map_it是指向一个pair<const string, size_t>对象的引用
cout << map_it->first; //打印此元素的关键字
cout << " " << map_it->second; //元素的值
map_it->first = "new key";//错误,因为关键字是const无法修改
++map_it->second;//可以通过迭代器改变元素.
```

**set的迭代器是const的**  
虽然set类型同时定义了iterator和const_iterator类型,但这两种类型都只允许只读访问set中的元素.  
```c++
set<int> iset = {0,1,2,3,4,5,6,7,8,9};
set<int>:: iterator set_it = iset.begin();
if (set_it != iset.end())
{
	*set_it = 42; //wrong,only read
	cout << *set_it << endl;
}
```

**遍历关联容器**  
map和set类型都支持表9.2的bigin和end操作.与往常一样,可以用这些函数获取迭代器,然后用迭代器来遍历容器.
```c++
// 获得一个指向首元素的迭代器
auto map_it = word_count.cbegin();
// 比较当前迭代器和尾后迭代器
while(map_it != word_count.cend())
{
	// 解引用迭代器,打印关键字-值对
	cout << map_it->first << " occurs " << map_it->second << " times" << endl;
	++map_it; //递增迭代器,移动到下一个元素
}
```

关联容器和算法  
通常不对关联容器使用泛型算法. 关键字是const这一特性意味着不能将关联容器传递给修改或重排容器元素的算法, 因为这类算法需要向元素写入值.  

关联容器可用只读取元素的算法.但很多这类算法都要搜索序列.但由于关联容器的元素不能通过它们的关键字进行查找,所以有时也不是一个好主意.

在实际编程中,如果真要对关联容器使用算法,可以将其当作一个源序列,要么当作一个目的位置.  
例如可以用泛型copy算法将元素从一个关联容器拷贝到另一个序列.  
类似的,可以调用inserter将一个插入器绑定(10.4.1,page358)到一个关联容器.通过使用inserter,将关联容器当作一个目的位置来调用另一个算法.

**11.3.2添加元素**
关联容器的inset向容器添加一个元素或一个元素范围.  
由于map和set(以及对应的无序类型)包含不重复的关键字,因此插入一个已存在的元素对容器没用任何影响.  
```c++
vector<int> ivec = {2,4,6,8,2,4,6,8}; //ivec有8个元素
set<int> set2; //空集合
set2.insert(ivec.cbegin(), ivec.cend()); //4个元素
set2.insert({1,3,5,7,1,3,5,7}) ; //8个元素
```

insert有两个版本,分别接受一对迭代器,或是一个初始化器列表,这两个版本的行为类似对应的构造函数.--对于一个给定的关键字,只有第一个带此关键字的元素才被插入到容器中.  

向map添加元素  
对一个map进行insert操作时,注意元素类型是pair.通常对于想要插入的数据,没有现成的pair对象.可以在insert的参数列表中创建一个pair:  
```c++
// 向word_count插入word的4种方法
word_count.insert({word,1});
word_count.insert(make_pair(word,1));
word_count.insert(pair<string,size_t>(word,1));
word_count.insert(map<string,size_t>::value_type(word,1));
```
<table>
	<tr>
		<th colspan="2">表11.4: 关联容器insert操作</th>
	</tr>
	<tr>
		<td>c.insert(v)</td>
		<td>v是value_type类型的对象,args用来构造一个元素</td>
	</tr>
	<tr>
		<td>c.emplace(args)</td>
		<td>对于map和set,只有当元素的关键字不在c中时才插入(或构造)元素.函数返回一个pair,包含一个迭代器,指向具有指定关键字的元素,以及一个指示插入是否成功的bool值</td>
	</tr>
	<tr>
		<td>c.insert(b,e)</td>
		<td>b和e是迭代器,表示一c::value_type类型值的范围,i1是这种值的花括号列表,函数返回void</td>
	</tr>
	<tr>
		<td>c.insert(i1)</td>
		<td>对于map和set,直插入关键字不在c中的元素,对于multimap和multiset,则会插入范围中的每个元素</td>
	</tr>
	<tr>
		<td>c.insert(p,v)</td>
		<td>类似insert(v)(或emplace(args)),但将迭代器p作为一个提示,指出从哪里开始搜索新元素应该存储的位置.返回一个迭代器,指向具有关键字的元素</td>
	</tr>
	<tr>
		<td>c.emplace(p,args)</td>
		<td>类似insert(v)(或emplace(args)),但将迭代器p作为一个提示,指出从哪里开始搜索新元素应该存储的位置.返回一个迭代器,指向具有关键字的元素</td>
	</tr>
</table>

检测insert的返回值  
展开递增语句
```c++
++((ret.first)->second);
```
向multiset multimap添加元素  
```c++
multimap<string,string> authors;
authors.insert({"Barth,John","Sot-Weed Factor"});
authors.insert({"Barth,John","Lost in the Funhouse"});  //插入第二个元素
```

11.3.3 删除元素  
关联容器定义了三个版本的erase,如表11.5.  
与顺序容器一样,可以通过传递erase一个迭代器或一个迭代器对来删除一个元素或者一个元素范围.这两个版本的erase与对应的顺序容器的操作非常类似:指定的元素被删除,函数返回void.  

关联容器提供一个额外的erase操作,它接受一个key_type参数.删除所有匹配给定关键字的元素(如果存在),返回实际删除的元素的数量.  
```c++
// 删除一个关键字,返回删除的元素数量
if (word_count.erase(removal_word))
	cout << "ok: " << removal_word << " removed\n";
else cout << "oops: " << removal_word << " not found!\n";
```
<table>
	<tr>
		<th colspan="2">表11.5:从关联容器删除元素</th>
	</tr>
	<tr>
		<td>c.erase(k)</td>
		<td>从c中删除每个关键字为k的元素.返回一个size_type值,指出删除的元素的数量</td>
	</tr>
	<tr>
		<td>c.erase(p)</td>
		<td>从c中删除迭代器p指定的元素.p必须指向c中一个真实元素,不能等于c.end().返回一个指向p之后元素的迭代器,若p指向c的尾元素,则返回c.end()</td>
	</tr>
		<tr>
		<td>c.erase(b,e)</td>
		<td>删除迭代器对b和e所表示的范围中的元素,返回e</td>
	</tr>
</table>

**11.3.4** map的下标操作  
map和unordered_map容器都提供了下标运算符和一个对应的at函数  
```c++
map<string, size_t> word_count; //empty map
// 插入一个关键字为"Anna"的元素,关联值进行初始化,然后将1赋予它
word_count["Anna"] = 1;
```
- 在word_count中搜索关键字为Anna的元素,未找到
- 将一个新的关键字-值插入到word_count中.关键字是const string,保存Anna,值进行初始化.
- 提取出新插入的元素,将值1赋予它.  

<table>
	<tr>
		<th colspan="2">表11.6:map和unordered_map的下标操作</th>
	</tr>
	<tr>
		<td>c[k]</td>
		<td>返回关键字为k的元素,如果k不在c中,添加一个关键字为k的元素,对其进行值初始化</td>
	</tr>
	<tr>
		<td>c.at(k)</td>
		<td>访问关键字为k的元素,带参数检查,若k不在c中,抛出out_of_range异常</td>
	</tr>
</table>

**使用下标操作的返回值**  
map的下标运算符与其他下标运算符的另一个不同之处是其返回类型.通常情况下,解引用一个迭代器所返回的类型与下标运算符返回的类型是一样的.  
但对map则不然,当对一个map进行下标操作时,会获得一个mapped_type对象,但当解引用一个map迭代器时,会得到一个value_type对象.  

**11.3.5 访问元素** 
<table>
	<tr>
		<th colspan="2">表11.7:在一个关联容器中查找元素的操作</th>
	</tr>
	<tr>
		<th colspan="2">lower_bound和upper_bound不适用于无序容器</th>
	</tr>
	<tr>
		<th colspan="2">下标和at操作只适用于非const的map和unordered_map</th>
	</tr>
	<tr>
		<td>c.find(k)</td>
		<td>返回一个迭代器,指向第一个关键字为k的元素,若k不在容器中则返回尾后迭代器</td>
	</tr>
	<tr>
		<td>c.count(k)</td>
		<td>访问关键字为k的元素的数量.</td>
	</tr>
	<tr>
		<td>c.lower_bound(k)</td>
		<td>返回一个迭代器,指向第一个关键字不小于k的元素</td>
	</tr>
	<tr>
		<td>c.upper_bound(k)</td>
		<td>返回一个迭代器,指向第一个关键字大于k的元素</td>
	</tr>
	<tr>
		<td>c.equal_range(k)</td>
		<td>返回一个迭代器pair,表示关键字等于k的元素的范围.若k不存在,pair的两个成员均等于c.end()</td>
	</tr>
</table>

11.3.6 一个单词转换的map  
一个程序展示map的创建,搜索以及遍历.  
这个程序的功能是:给定一个string,将它转换为另一个string.程序的输入是两个文件,第一个文件保存的是一些规则,用来转换第二个文件中的文本.每条规则由两部分组成:一个可能出现在输入文件中的单词和一个用来替换它的短语.  
即,每当第一个单词出现在输入中,就替换为对应的词语.第二个输入文件包含要转换的文本  

单词转换程序使用三个函数.函数word_transform管理整个过程, 它接受两个ifstream参数:第一个参数应绑定到单词转换文件,第二个参数应绑定到要转换的文本文件.函数buildMap会读取转换规则文件,并创建一个map,用于保存每个单词到其转换内容的映射.函数transform接受一个string,如果存在转换规则,返回转换后的内容.  
```c++
void word_transform(ifstream &map_file, ifstream &input)
{
	auto trans_map = buildMap(map_file);  //保存转换规则
	string text;
	while (getline(input, text))
	{
		istringstream stream(text);  //读取每个单词
		string word;
		bool firstword = true;  //控制是否打印空格
		while(stream >> word)
		{
			if(firstword)
				firstword = false;
			else
				cout << " ";
			// transform返回它的第一个参数或其转换之后的形式
			cout << transform(word, trans_map); //打印输出
		}
		cout << endl;  //完成一行的转换
	}
}

map<string, string> buildMap(ifstream &map_file)
{
	map<string, string> trans_map;  //保存转换规则
	string key;  //要转换的单词
	string value; //替换后的内容
	// 读取第一个单词存入key,行内剩余存入value
	while (map_file >> key && getline(map_file, value))
		if (value.size>1)  //检查是否由转换规则
			trans_map[key] = value.substr(1);  //跳过前导空格
		else 
			throw runtime_error("no rule for " +key);
	return trans_map;
}

const string & transform(const string &s, const map<string, string> &m)
{
	// 实际的转换工作
	auto map_it = m.find(s);
	// 如果单词在转换规则map
	if (map_it != m.cend())
		return map_it->second;
	else
		return s;
}

```
11.4 无序容器  
除了哈希管理操作外,无序容器还提供了与有序容器相同的操作(find, insert等).类似的,无序容器也有允许重复关键字的版本  

管理桶  
无序容器在存储组织上为一组桶,每个桶保存零个或多个元素.  
无序容器使用一个哈希函数将元素映射到桶.为了访问一个元素,容器首先计算元素的哈希值,它指出应该搜索哪个桶.  
容器将具有一个特定哈希值的所有元素都保存在相同的桶中.如果容器允许重复关键字,所有具有相同关键字的元素也都会在同一个桶中.  
因此无序容器的性能依赖于哈希函数的质量和桶的数量和大小.  

对于相同的参数,哈希函数必须总是产生相同的结果.  
理想情况下,哈希函数还能将每个特定的值映射到唯一的桶.但是,将不同关键字的元素映射到相同的桶也是允许的.  

无序容器提供了一组管理桶的函数,如表11.8,这些成员函数允许我们查询容器的状态以及在必要时强制容器进行重组.  

<table>
	<tr>
		<th colspan="2">表11.8:无序容器管理操作</th>
	</tr>
	<tr>
		<th colspan="2">桶接口</th>
	</tr>
	<tr>
		<td>c.bucket_count()</td>
		<td>正在使用的桶的数目</td>
	</tr>
	<tr>
		<td>c.max_bucket_count()</td>
		<td>容器能容纳的最多的桶的数量</td>
	</tr>
	<tr>
		<td>c.bucket_size(n)</td>
		<td>第n个桶中有多少个元素</td>
	</tr>
	<tr>
		<td>c.bucket(k)</td>
		<td>关键字为k的元素在哪个桶中</td>
	</tr>
	<tr>
		<th colspan="2">桶迭代</th>
	</tr>
	<tr>
		<td>local_iterator</td>
		<td>可以用来访问桶中元素的迭代器类型</td>
	</tr>
	<tr>
		<td>const_local_iterator</td>
		<td>桶迭代器的const</td>
	</tr>
	<tr>
		<td>c.begin(n),c.end(n)</td>
		<td>桶n的首元素迭代器和尾后迭代器</td>
	</tr>	
	<tr>
		<td>c.cbegin(n),c.cend(n)</td>
		<td>const_local_iterator桶n的首元素迭代器和尾后迭代器</td>
	</tr>
	<tr>
		<th colspan="2">哈希策略</th>
	</tr>
	<tr>
		<td>c.load_factor()</td>
		<td>每个桶的平均元素数量,返回float值</td>
	</tr>
	<tr>
		<td>c.max_load_factor()</td>
		<td>c试图维护的平均桶大小,返回float值.c会在需要时添加新的桶,以使得load_factor<=max_load_factor</td>
	</tr>
	<tr>
		<td>c.rehash(n)</td>
		<td>重组存储,使得bucket_count>=n,且bucket_count>size/max_loan_factor</td>
	</tr>	
	<tr>
		<td>c.reserve(n)</td>
		<td>重新存储,使得c可以保存n个元素且不必rehash</td>
	</tr>
</table>

无序容器对关键字类型的要求  
默认情况下,无序容器使用关键字类型的==运算符来比较元素,还使用一个hash<key_type>类型的对象来生成每个元素的哈希值. 标准库为内置类型(包括指针)提供了hash模板.还为一些标准库类型,包括string和将在12章介绍的智能指针类型定义了hash.  
因此,可以直接定义关键字是内置类型(包括指针类型),string还是智能指针类型的无序容器.  

但不能直接定义关键字类型为自定义类类型的无序容器.与容器不同,不能直接使用哈希模板,而必须提供自己的hash模板版本(16.5介绍).  
不使用默认的hash,而是使用另一种方法.类似于为有序容器重载关键字类型的默认比较操作.  
为了将Sale_data用作关键字,需要提供函数来替代==运算符和哈希值计算函数.  
从这些重载函数开始:  

size_t hasher(const Sales_data &sd)
{
	return hash<string>()(sd.ibsn());
}
bool eq0p(const Sales_data &lhs, const Sales_data &rhs)
{
	return lhs.isbn() == rhs.isbn();
}
hasher函数使用一个标准库hash类型对象来计算ISBN成员的哈希值,该hash类型建立在string类型指示.

### 第12章动态内存   
- 12.1 动态内存与智能指针  
- 12.2 动态数组
- 12.3 使用标准库:文本查询程序

至此，编写的程序中所使用的对象都有着严格定义的生存期。全局对象在程序启动时分配，在程序结束时销毁.对于局部自动对象,当进入其定义所在块时被创建,离开块时销毁.局部static对象在第一次使用前分配,在程序结束时销毁.  

除了自动和static对象外,C++还支持动态分配对象.动态分配的对象的生存期与在哪里创建无关,只有当显式地被释放时,这些对象才会销毁.  

动态对象的正确释放:为了更安全的使用动态对象,标准库定义了两个智能指针类型来管理动态分配的对象.当一个对象应该被释放时,指向它的智能指针可以确保自动地释放它.  

至此,只使用过静态内存或栈内存.静态内存用来保存局部static对象,类static数据成员以及定义在任何函数之外的变量.  
栈内存用来保存定义在函数内的非static对象.   
分配在静态或栈内存中的对象由编译器自动创建或销毁.  
对于栈对象,仅在其定义的程序块运行时才存在:static对象在使用前分配,程序结束时销毁.  

除了静态内存和栈内存,每个程序还拥有一个内存池.这部分内存被称为自由空间(free store)或堆(heap).  
程序用来存储动态分配(dynamically allocate)的对象的内存是堆.  
动态对象的生存期由程序来控制,也就是说动态对象需要显式的释放内存.  

12.1 动态内存与智能指针  
C++中,动态内存的管理是通过一对运算符来完成的:new, 在动态内存中为对象分配空间并返回一个指向该对象的指针,可以选择对对象进行初始化;  
delete,接受一个动态对象的指针,销毁该对象,并释放与之关联的内存.  

如果忘记释放内存,就会造成内存泄漏.如果还有指针引用内存的情况下就释放了内存,这是指针再用时就会产生引用非法内存的指针.  

为了更容易安全的使用动态内存,新的标准库提供了两种指针指针(smart pointer)类型来管理动态对象.  
智能指针的行为类似常规指针,重要的区别是**它负责自动释放所指向的对象.**  
新标准库提供的这两种智能指针的区别在于管理底层指针的方式:shared_ptr允许多个指针指向同一个对象; unique_ptr则"独占"所指向的对象.  
标准库还定义了一个名为weak_ptr的伴随类,它是一种弱引用,指向shared_ptr所管理的对象.这三种类型都定义在memory头文件中.  

12.1.1 shared_ptr类   
类似vector,指针指针也是模板.因此,创建一个智能指针时,必须提供额外的信息--指针指向的类型.  
```c++
shared_ptr<string> p1;   // shared_ptr,可以指向string
shared_ptr<list<int>> p2;  //shared_ptr,可以指向int的list
```
默认初始化的智能指针保存着一个空指针.  
初始化智能指针的其他方法:  
```c++
// 1. 使用new返回的指针来初始化智能指针
shared_ptr<int> p2(new int(42));  //p2指向一个值为42的int
// 接受指针参数的智能构造函数是explicit的.  
// 2.因此,不能将内置指针隐式转换为一个智能指针,必须使用直接初始化形式来初始化一个指针指针:  
shared_ptr<int> p1 = new int(1024); //错误,必须使用直接初始化
shared_ptr<int> p2 (new int(1024)); //正确,使用了直接初始化形式
// p1的初始化隐式地管理编译器用一个new返回的int * 来创建一个shared_pt, 这是一个内置指针,但不能用内置指针到智能指针间的隐式转换,因此第一条初始化语句是错误的.  
// 由于相同的原因,一个返回shared_ptr的函数不能在其返回语句中隐式转换一个普通指针.  
shared_ptr<int> clone(int p){
	return new int(p);  //错误,隐式转换为shared_ptr<int>}
// 必须将shared_ptr显式绑定到一个想要返回的指针上:  
shared_ptr<int> clone(int p){
	return shared_ptr<int>(new int(p));//显式的用int*创建shared_ptr<int>}
// 3. make_shared标准库函数
shared_ptr<int> p3 = make_shared<int>(42)
// 推荐使用make_shared,而不是new
```

<table>
	<tr>
		<th colspan="2">表12.1:shared_ptr和unique_ptr都支持的操作</th>
	</tr>
	<tr>
		<td>shared_ptr<T> sp</td>
		<td>空智能指针,指向类型T的对象</td>
	</tr>
	<tr>
		<td>unique_ptr<T> up</td>
		<td>空智能指针,指向类型T的对象</td>
	</tr>
	<tr>
		<td>p(就是将share_ptr/unique_ptr放在条件里)</td>
		<td>将p用作一个条件判断,若p指向一个对象,则为true</td>
	</tr>
	<tr>
		<td>*p</td>
		<td>解引用p,获得它指向的对象</td>
	</tr>
	<tr>
		<td>p->mem</td>
		<td>等价于(*p).mem,类似于this指针使用方法/td>
	</tr>
	<tr>
		<td>p.get()</td>
		<td>返回p中保存的指针,要小心使用,若智能指针释放了其对象,返回的指针所指向的对象也会消失</td>
	</tr>
	<tr>
		<td>swap(p,q)</td>
		<td>交换p和q的指针</td>
	</tr>	
	<tr>
		<td>p.swap(q)</td>
		<td>交换p和q的指针</td>
	</tr>
</table>
<table>
	<tr>
		<th colspan="2">表12.2:shared_ptr独有的操作</th>
	</tr>
	<tr>
		<td>shared_ptr<T>(args)</td>
		<td>返回一个shared_ptr,指向一个动态分配的类型T的对象,使用args初始化此对象</td>
	</tr>
	<tr>
		<td>shared_ptr<T>p(q)</td>
		<td>p是shared_ptr q的拷贝;此操作会递增q中的计数器,q中的指针必须能转换为T*</td>
	</tr>
	<tr>
		<td>p=q</td>
		<td>p和q都是shared_ptr,所保存的指针必须能相互转换.此操作会递减p的引用计数,递增q的引用计数;若p的引用计数变为0,则将其管理的原内存释放</td>
	</tr>
	<tr>
		<td>p.unique()</td>
		<td>若p.use_count()为1,返回true,否则false</td>
	</tr>
	<tr>
		<td>p.use_count()m</td>
		<td>返回与p共享对象的智能指针数量;可能很慢,主要用于调试</td>
	</tr>
</table>

make_shared函数  
最安全的分配和使用动态内存的方法是调用一个名为make_shared的标准库函数.  
该函数在动态内存中分配一个对象并初始化它,返回指向此对象的shared_ptr.与智能指针一样,make_shared也定义在头文件memory中.  

当要使用make_shared时,必须指定想要创建的对象的类型.  
```c++
// 指向一个值为42的int的shared_ptr
shared_ptr<int> p3 = make_shared<int>(42)
// p4指向一个值为"99999999"的string
shared_ptr<string> p4 = make_shared<string>(10,'9');
// p5指向一个值初始化的int,即值为0
shared_ptr<int> p 5 = make_shared<int>();
// 当然可以使用auto来定义对象保存make_shared的结果
auto p6 = make_shared<vector<string>>();
```
类似于顺序容器的emplace成员,make_shared用其参数来构造给定类型的对象.例如,**调用make_shared<string>时传递的参数必须与string的某个构造函数相匹配.**即使用模板T的现下类型的构造函数来初始化shared_ptr(看起来很实用,不用记shared_ptr自己的初始化方法,而只用记T的当下类型的构造函数接口即可).  

小知识插入: 经常使用STL的同学肯定对push_front，push_back和insert不会陌生。我们对容器进行插入操作经常要用到这三个函数，来将元素放置在容器头部、指定位置或容器尾部。

C++11新标准引入了三个新函数——emplace_front、emplace和emplace_back来代替上述三个函数。

这两种操作的主要区别在于：emplace使用**构造函数**而不是拷贝函数。  
调用push或insert函数后，会先创建一个局部临时对象，并使用拷贝函数给该临时对象赋值，并将其压入容器中。而调用emplace函数后，则会在容器管理的内存空间中直接调用构造函数创建对象。
由于emplace避免了对象的复制和移动，因此其插入的效率会更高。
对容器内插入大对象的时候，应当优先考虑使用emplace方法  

shared_ptr的拷贝和赋值  
当进行拷贝或赋值操作时,每个shared_ptr都会记录由多少个其他shared_ptr指向相同的对象.  
```c++
auto p = make_shared<int>(42);  //p指向的对象只有p一个引用者
auto q(p);  //p和q指向相同的对象,此对象有两个引用者
```
可以认为,每个shared_ptr都有一个关联的计数器,通常称其为**引用计数**(reference count).无论何时拷贝一个shared_ptr,计数器都会递增.  
例如,用一个shared_ptr初始化另一个shared_ptr,或将它作为参数传递给一个函数以及作为函数的返回值时,它所关联的计数器都会递增.  
当给shared_ptr赋予一个新值或shared_ptr被销毁(例如一个局部的shared_ptr离开其作用域)时,计数器就会递减.  

**shared_ptr自动销毁所管理的对象......**  
当指向一个对象的最后一个shared_ptr被销毁时,shared_ptr类会自动销毁此对象.  
它是通过另一个特殊的成员函数--**析构函数(destructor)**完成销毁工作的.  

shared_ptr的析构函数会递减它所指向的对象的引用计数.如果引用计数变为0,shared_ptr的析构函数就会自动销毁对象,并释放它占用的内存.  

**......shared_ptr还会自动释放相关联的内存**  
当动态对象不再被使用时,shared_ptr类会自动释放动态对象.这一特性使得动态内存的使用变得非常容易.  
```c++
// factory返回一个shared_ptr,指向一个动态分配的对象
shared_ptr<Foo> factory(T arg)
{
	// 处理arg
	// shared_ptr负责释放内存
	return make_shared<Foo>(arg);
}
// 这里返回了一个shared_ptr,所以其分配的对象还在.如果只是使用,而没有返回的话,离开函数shared_ptr就会消失,其指向的对象也会被销毁
```

由于在最后一个shared_ptr销毁前内存都不会释放,我们希望shared_ptr在无用之后不再保留.如果没有销毁程序不再需要的shared_ptr,就会浪费内存.所以要记得主动释放(但怎么释放呢?)  
share_ptr在无用之后仍然保留的一种可能情况是,将shared_ptr存放在一个容器中,随后重排容器,从而不再需要某些元素.者情况下,应该确保用erase删除那些不再需要的shared_ptr元素

使用动态生存期的资源的类  
程序使用动态内存出于以下三种原因之一:  
1. 程序不知道自己需要使用多少对象 //容器就是个例子
2. 程序不知道所需对象的准确类型   //15章中会介绍
3. 程序需要在多个对象间共享数据  //本节定义的一个类,使用动态内存是为了让多个对象共享相同的底层数据

目前,使用的类,分配的资源都与对应对象生存期一致.例如拷贝一个vector时,原vector和副本vector中的元素是相互分离的  
```c++
vector<string> v1;
{//新作用域
	vector<string> v2={"a","an","the"};
	v1 = v2;
}//v2被销毁
```
但还有某些类分配的自由具有与原对象相独立的生存期.  
假定希望定义一个Blob的类,保存一组元素.与容器不同,希望Blob对象的不同拷贝之间共享相同的元素.  
一般而言,如果两个对象共享底层的数据,当某个对象被销毁时,不能单方面销毁底层数据:
```c++
Blob<string> b1;
{
	Blob<string> b2 = {"a","an","the"};
	b1 = b2;
} //b2被销毁,但b2中的元素不能销毁.b1指向最初由b2创建的元素.
```
使用动态内存的一个常见原因是允许多个对象共享相同的状态(即,我们希望拷贝后的对象和原对象共享底层元素,且其底层元素的生存期是独立,只要有对象还在用,就能保存(shared_ptr的特性))  

12.1.2 直接管理内存  
C++定义了两个运算符来分配和释放动态内存.new分配内存,delete释放new分配的内存.  

使用new动态分配和初始化对象  
在自由空间分配的内存是无名的,因此new无法为其分配的对象命名,而是返回一个指向该对象的指针:
```c++
int *pi = new int; //pi指向一个动态分配的,未初始化的无名对象
// 此new表达式,在自由空间构造一个int对象,并返回指向该对象的指针.
// 右边的int可以直接使用int的构造函数进行初始化,例如
string *ps = new string(10,'9');
// 初始化器
auto p1 = new auto(obj);  //p指向一个与obj类型相同的对象,该对象用obj进行初始化  
auto p2 = new auto{a,b,c}; //错误,括号中只能有单个初始化器
```

动态分配的const对象  
```c++
// 用new分配const对象是合法的:  
const int *pci = new const int(1024);
// 分配并默认初始化一个const的空string
const string *pcs = new const string;
```
对于一个定义了默认构造函数的类类型,其const动态对象可以隐式初始化,而其他类型的对象必须显示初始化.由于其分配的对象是const的,new返回的指针是一个指向const的指针.  

内存耗尽时,new就分配不了自由空间/堆了  
此时就会抛出一个类型为bad_alloc的异常,**可以改变使用new的方式来阻止它抛出异常**  
```c++
// 如果分配失败,new返回一个空指针  
int *p1 = new int;  //如果分配失败,new返回一个空指针  
int *p2 = new (nothrow) int; //如果分配失败,new返回一个空指针;
```
这种形式的new为**定位new(placement new). 定位new表达式允许向new传递额外的参数.  
如果将nothrow传递给new,告诉程序不能抛出异常,若没内存,则返回空指针.  
bad_alloc和nothrow都定义在头文件new中.  

释放动态内存(delete)delte接受一个指针,指向释放的对象.  
delete与new类型类似,delete表达式也执行两个动作:销毁给定的指针指向的对象; 释放对应的内存.  

指针值和delete  
动态对象的生存期直到释放时为止  
shared_ptr管理的内存在最后一个shared_ptr销毁时被自动释放.\[智能指针\]  
但对于一个由内置指针管理的动态对象,直到被显示释放之前都是存在的.\[new生成的内置指针管理的动态对象\]  

```c++
Foo * factory(T arg)
{
	// 视情况处理arg
	return new Foo(arg);  //之后要记得释放此内存
}

void use_factory(T arg)
{
	Foo *p = factory(arg);
	// 使用p但不delete它
} //p离开了它的作用域,它是一个内置指针而不是智能指针,它所指向的内存没有被释放
// 在本例中,p是指向factory分配的内存的唯一指针,一旦p被销毁,这块内存就无法被释放了,所以应该在use_factory中需要释放内存.
```
与类类型不同,内置类型的对象被销毁时什么也不会发生.特别是,当一个指针离开其作用域时,它所指向的对象什么也不会发生.如果这个指针指向的是动态内存,那么内存不会被释放.  

由内置指针(而不是智能指针)管理的动态内存被显示释放前都一直存在.  

动态内存管理出错:
1. 忘记delete->内存泄漏,这块内存永远不会被归还给自由空间
2. 使用已经释放掉的对象.
3. 同一块内存释放两次.如果delete两次,自由空间就可能被破坏.  

delete之后重置指针值  
delete一个指针后,指针值就变成无效,但指针扔保存着(已经释放的)动态内存的地址.变成空悬指针(dangling pointer),即指向一块曾经保存数据对象但现在无效的内存的指针.  

避免空悬指针的一种方法:在指针即将离开其作用域之前释放掉它所关联的内存. 这样就不会保存指针,如果需要保存之后用,可以在delete之后将nullptr赋予指针.  
```c++
int *p(new int(42)); //p指向动态内存
auto q=p;   //pq指向相同的内存
delete p;   //pq都变成无效
p=nullptr;  //指出p不再绑定任何对象
```

12.1.3 shared_ptr和new结合使用  
这部分12.1.1已经有

不要混合使用普通指针和智能指针  

shared_ptr可以协调对象的析构,但这仅限于其自身的拷贝(也是shared_ptr)之间.这也是推荐make_shared而不是new的原因  
这样就能在分配对象的同时就将shared_ptr与之绑定,从而避免将同一块内存绑定到多个独立创建的shared_ptr  

**下面是一个重要的例子: 将智能指针作为参数传入函数时,当函数的参数是以值传递时,实参会被拷贝到函数中,这就导致智能指针被复制, 引用计数值+1,所以传入的智能指针的内存不会被释放,但不能传入普通指针,普通指针会导致类型不符,如果将普通指针硬转换为智能指针,智能指针引用数在函数中第一次创建,所以在函数中时引用计数值为1,内存会被释放.**  
```C++
// 函数被调用时,ptr被创建并初始化,导致智能指针被复制
void process(shared_ptr<int> ptr)
{
	// 使用ptr
} // ptr离开作用域,被销毁,但ptr指向的内存不会被释放.
// 使用process的正确方法是传递给它一个shared_ptr
shared_ptr<int> p(new int(42));//引用计数为1
process(p); //拷贝p会递增它的引用计数,在process中引用计数值为2
int i = *p; //对其解引用,引用计数值为1

// 错误用法将普通指针传入  
int *x(new int(1024));  
process(x);  //错误,不能将普通指针直接转换为智能指针  
process(shared_ptr<int>(x)); //合法,但因为引用计数值为1,函数结束后内存将被释放,x将编程空悬指针  
int j = *x;  //未定义的,x是一个空悬指针
```

不要使用get初始化另一个智能指针或为智能指针赋值  
get函数,返回一个内置指针,指向智能指针管理的对象.设计情况:需要向不能使用智能指针的代码传递一个内置指针.  
使用get返回的代码不能delete此指针  

就是如果在一个新的代码块使用了get返回的指针进行初始化智能指针的话,该智能指针在该代码块结束时将被释放空间,这会导致get返回的指针指向的内存因为智能指针将内存释放而没了.所以导致系列的问题.  

**其他shared_ptr操作**  
可以使用reset来讲新的指针赋予一个shared_ptr;
```c++
p = new int(1024);   //错误,不能将一个指针赋予shared_ptr
p.rest(new int(1024));  //正确,p指向一个新对象
```
与赋值相似,reset会更新引用计数+1  
reset与unique一起使用来控制多个shared_ptr共享的对象:在改变底层对象之前,检查是否是当前对象仅有的用户,如果不少,改变前要做依次新的拷贝.  
```c++
if (!p.unique())
	p.reset(new string(*p));  //不是唯一的用户,分配新的拷贝
*p += newVal;   //唯一的用户,可以改变对象的值.
```
12.1.4 智能指针和异常  
如果使用智能指针,即使程序异常导致结束过早,智能指针也能确保内存不需要时会智能释放.  
```c++
void f()
{
	shared_ptr<int> sp(new int(42)); 
	// 这段代码抛出一个异常,且在f中未被捕获
} //函数结束时,shared_ptr自动释放
```
智能指针陷阱
- 不使用相同的内置指针值初始化(或reset)多个智能指针
- 不delete get()返回的指针  
- 不使用get()初始化或reset另一个智能指针
- 如果使用get()返回的指针,最后一个对应的智能指针销毁后,指针就变成无效了
- 如果使用智能指针管理的资源不是new分配的内存,记得传递给它一个删除器(deleter)

12.1.5 unique_ptr  
定义一个unique_ptr时,需要将其绑定到一个new返回的指针上.  
初始化unique_ptr必须采用直接初始化形式:  
```c++
unique_ptr<double> p1;  
unique_ptr<int> p2(new int(42)); 
// unique_ptr不支持普通的拷贝或赋值操作
```
```c++
// unique_ptr不支持普通的拷贝或赋值操作
unique_ptr<string> p1(new string("Stegosaurus"));
unique_ptr<string> p2(p1); //wrong,不支持拷贝
unique_ptr<string> p3;
p3 = p2;  //错误,不支持赋值
```
<table>
	<tr>
		<th colspan="2">表12.4:unique_ptr操作<th>
	</tr>
	<tr>
		<td>unique_ptr<T> u1</td>
		<td>空unique_ptr,可以指向类型为T的对象.u1会使用delete来释放它的指针.
	</tr>
	<tr>
		<td>unique_ptr<T, D> u2</td>
		<td>u2会使用一个类型为D的可调用对象来释放它的指针.
	</tr>
	<tr>
		<td>unique_ptr<T,D> u(d)</td>
		<td>空unique_ptr,指向类型为T的对象,用类型为D的对象d代替delete</td>
	</tr>
	<tr>
		<td>u=nullptr</td>
		<td>释放u指向的对象,将u置空</td>
	</tr>
	<tr>
		<td>u.release()</td>
		<td>u放弃堆指针的控制权,返回指针,将u置空</td>
	</tr>
	<tr>
		<td>u.reset()</td>
		<td>释放u指向的对象</td>
	</tr>
	<tr>
		<td>u.reset(q)</td>
		<td>如果提供了内置指针q,令u指向这个对象,否则置空</td>
	</tr>
	<tr>
		<td>u.reset(nullptr)</td>
		<td>如果提供了内置指针q,令u指向这个对象,否则置空</td>
	</tr>
</table>
虽然不能赋值或拷贝,但可以用release或reset将指针的所有权从一个unique_ptr转移到另一个unique  
(因为release是将原指针置空,然后将其指针返回,所以可以用
p2(p1.release())将p1置空,并返回p1的指针,这里给p2的初始化,从而完成转移)
(reset,若p2是空,则将给与的p指针给p2,若p2不为空,p2原来的指针将被释放.)两者的差别是用p.release()返回的指针初始化时,若p2为空,可以直接用release返回的指针初始化,若p2不为空,需要p2.reset将p2置空后再赋值.

```c++
// 将所有权从p1(指向string Stegosaurus)转移给p2
unique_ptr<string> p2(p1.release());  //release使得p1置空
unique_ptr<string> p3(new string("Trex"));
// 将所有权从p3转移给p2
p2.reset(p3.release());  //reset释放p2原来指向的内存
```

**不能拷贝unique_ptr规则有一个例外:可以拷贝或赋值一个将要被销毁的unique_ptr.**  
最常见的例子是函数返回一个unique_ptr  

```c++
unique_ptr<int> clone(int p)
{
	return unique_ptr<int>(new int(p));  //赋值
	// 或返回一个局部对象的拷贝
	unique_ptr<int> ret(new int(p));
	return ret;
}
```

向unique_ptr传递删除器  
unique_ptr和shared_ptr一样,默认情况下用delete释放它指向的对象.  
与shared_ptr一样,可以重载一个unique_ptr中默认的删除器.但两者管理删除器的方式与shared_ptr不同.  

类似,在<>中unique_ptr指向类型后提供删除器类型.在创建或reset一个这种unique_ptr类型的对象时,必须提供一个指定类型的可调用对象(删除器)

```c++
// p指向一个类型为objT的对象,并使用一个类型为delT的对象释放objT对象
// 它会调用一个名为fcn的delT类型对象
unique_ptr<objT, delT> p (new objT, fcn);  //不是很懂
```

12.1.6 weak_ptr  
weak_ptr是一种不控制所指向对象生存期的智能指针,它指向由一个shared_ptr管理的对象.将一个weak_ptr绑定到一个shared_ptr不会改变shared_ptr的引用计数.  
当最后一个shared_ptr被销毁,对象被释放,weak_ptr的对象也会被释放.  
<table>
	<tr>
		<th colspan="表12.5:weak_ptr">
	<tr>
	<tr>
		<td>weak_ptr<T> w</td>
		<td>空weak_ptr指向类型为T的对象</td>
	</tr>
	<tr>
		<td>weak_ptr<T> w(sp)</td>
		<td>sp为shared_ptr,weak_ptr指向与sp相同的对象</td>
	</tr>
	<tr>
		<td>w=p</td>
		<td>p是shared_ptr或weak_ptr,赋值后w与p共享对象</td>
	</tr>
	<tr>
		<td>w.reset()</td>
		<td>将w置空</td>
	</tr>
	<tr>
		<td>w.use_count()</td>
		<td>与w共享对象的shared_ptr的数量</td>
	</tr>
	<tr>
		<td>w.expired()</td>
		<td>若w.use_count()为0,返回true,否则false</td>
	</tr>
	<tr>
		<td>w.lock()</td>
		<td>expired是true,返回空shared_ptr,否则返回w的对象的shared_ptr</td>
	</tr>
</table>

创建weak_ptr,用一个shared_ptr来初始化  
```c++
auto p = make_shared<int>(42);
weak_ptr<int> wp(p);  //wp弱共享p,p的引用计数未改变(弱共享这个词真准确)
```

展示weak_ptr用途
```c++
// 对于访问一个不存在的元素的尝试,StrBlobPtr抛出一个异常
class StrBlobPtr
{
public:
	StrBlobPtr(): curr(0){}
	StrBlobPtr(StrBlob &a, size_t sz = 0): wptr(a.data), curr(sz) {}
	std::string & deref() const;
	StrBlobPtr & incr();  //前缀递增
private:
// 若检查成功,check返回一个指向vector的shared_ptr
	std::shared_ptr<std::vector<std::string>> check(std::size_t, const std::string&) const;
	// 保存一个weak_ptr,意味着底层vector可能会被销毁  
	std::weak_ptr<std::vector<std::string>> wptr;
	std::size_t curr;  //在数组的当前位置
};

std::shared_ptr<std::vector<std::string>>
StrBlobPtr::check(std::size_t i, const std::string &msg) const
{
	auto ret = wptr.lock();  //vector是否存在?
	if(!ret)
		throw std::runtime_error("unbound StrBlobPtr");
	if(i>=ret->size())
		throw std::out_of_range(msg);
	return ret;  //否则,返回指向vector的shared_ptr
}

// 14章学习如何定义运算符,这里定义deref和incr的函数,用来解引用和递增StrBlobPtr
// deref成员调用check,检查使用vector是否安全以及curr是否在合法范围内:  
std::string & StrBlobPtr::deref() const
{
	auto p = check(curr, "dereference past end");
	return (*p)[curr];  //(*p)是对象所指向的vector,解引用,下标返回元素
}
// 如果check成功,p就是一个shared_ptr,指向StrBlobPtr所指向的vector.

// 前缀递增:返回递增后的对象的引用
StrBlobPtr& StrBlobPtr::incr()
{
	// 如果curr已经指向容器的尾后位置,就不能递增它
	check(curr, "increment past end of StrBlobPtr");
	++curr;  //推进当前位置
	return *this;
}
// 为访问data成员,指针类型必须声明为StrBlob的friend,还要为StrBlob类定义begin和end操作,返回一个指向它自身的StrBlobPtr:
// 对于StrBlob中的友元声明来说,此前置声明是必要的
class StrBlobPtr;
class StrBlob
{
	friend class StrBlobPtr;
	// 返回指向首元素和尾后元素的StrBlobPtr
	StrBlobPtr begin() {return StrBlobPtr(*this);}
	StrBlobPtr end() 
	{
		auto ret = StrBlobPtr(*this, data->size());
		return ret;
	}
}
```

12.2 动态数组  
new和delete运算符一次分配/释放一个对象.  
但需要一次为很多对象分配内存.C++和标准库提供了两种一次分配一个对象数组的方法:new和数组,allocator类  
C++定义了另一种new表达式语法,可以分配并初始化一个对象数组.标准库中包含一个名为allocator的类,允许将分配和初始化分离.使用allocator通常会提供更好的性能和内存管理能力.  

使用容器的类可以使用默认版本的拷贝,赋值和析构操作.  
分配动态数组的类必须定义自己版本的操作,在拷贝,赋值以及销毁对象时管理所管理的内存.

12.2.1 new和数组  
将new T[]分配的内存为"动态数组".new返回的是一个元素类型的指针.  
由于分配的内存不是一个数组类型,因此不能对动态数组调用begin()或end),这些数组使用,数组维度来指向首尾元素的指针.

虽然通常称new T\[\]分配的内存为"动态数组",但其实new分配一个数组时,并未得到一个数组类型的对象,而是得到一个数组元素类型的指针.即指向数组第一个元素的指针.(应该是吧)  
由于分配的内存并不是一个数组类型,因此不能对动态数组调用begin或end.这些函数使用数组维度,来返回指向首元素的和尾后元素的指针.同样也不能使用范围for语句处理动态数组中的元素.  

生成一个长度为0的动态数组不会报错,返回尾后指针.(但没用)  

释放动态数组: ```c++ delete p; delete [] pa; ```

**智能指针和动态数组**  
标准库提供了一个可以管理new分配的数组的unique_ptr版本.  
为了用一个unique_ptr管理动态数组,必须在对象类型后跟一对空方括号:  
```c++
// up指向一个包含10个未初始化int的数组
unique_ptr<int[]> up(new int[10]);
up.release();  //自动用delete[]销毁其指针.
// <int[]>指出up指向的是一个int数组而不是一个int.
// 由于up指向一个数组,当up销毁它管理的指针时,会自动使用delete[].
```
shared_ptr不支持直接管理动态数组,如果想要用shared_ptr管理动态数组,必须提供自己定义的删除器.  
```c++ 
shared_ptr<int> sp(new int[10], [](int *p){delete [] p;});
sp.reset();   //使用提供的lambda释放数组,它使用delete[]
```

12.2.2 allocator类  
new将内存分配和对象构造合在一起,delete将对象析构和内存释放合在一起.   
但有时因为内存分配是大块的,对象是慢慢构建或数量是未知的,此时不希望一次性构造很多个不需要的对象(构造对象也需要成本),此时希望将内存分配和构造对象分开来,就有了allocator类.  

**allocator类**  
标准库allocator类定义在头文件memory中,将内存分配和对象构造分离开.  
allocator提供一种类型感知的内存分配方法,它分配的内存是原始的,未构造的.  
(什么叫做原始的,未构造的内存?)
类似,vector,allocator是一个模板.需要给allocator说明对象类型  
allocator对象分配内存时,会根据给定的对象类型来确定恰当的内存大小和对齐位置:  
```c++
allocator<string> alloc;  //可以分配string的allocator对象
auto const p = alloc.allocate(n);   //分配n个未初始化的string
```
<table>
	<tr>
		<th colspan="2">表12.7:标准库allocator类及其算法</th>
	</tr>
	<tr>
		<td>allocator<T> a</td>
		<td>定义了一个名为a的allocator对象,它可以为类型为T的对象分配内存</td>
	</tr>
	<tr>
		<td>a.allocate(n)</td>
		<td>分配一段原始的,未构造的内存,保存n个类型为T的对象</td>
	</tr>
	<tr>
		<td>a.construct(p,args)</td>
		<td>p必须是一个类型为T*的指针,指向一块原始内存;arg被传递给类型为T的构造函数,用来在p指向的内存中构造一个对象.</td>
	</tr>
	<tr>
		<td>a.destroy(p)</td>
		<td>p为T*类型的指针,此算法对p指向的对象执行析构函数</td>
	</tr>
	<tr>
		<td>a.deallocate(p,n)</td>
		<td>释放从T*指针p中地址开始的内存,这块内存保存了n个类型未T的对象,p必须是一个先前由allocate返回的指针,且n必须是p创建时所要求的大小.在调用deallocate之前,用户必须对每个在这块内存创建的对象调用destroy</td>
	</tr>
</table>

allocator分配的内存是未构造的(unconstructed),按需在此内存中构造对象.在新标准库中,construct成员函数接受一个指针和零个或多个额外参数args,在给定的位置构造一个元素.额外参数args是用来初始化构造的对象,需要符合构造函数的,即这些额外参数是与构造对象的类型相匹配的初始化器:  
```c++
auto q = p;   //q指向最后构造的元素之后的位置
alloc.construct(q++);  //*q为空字符串
alloc.construct(q++,10,'c');  //*q为cccccccccc
alloc.construct(q++,"hi");  //*q为hi
// 使用完对象后,对每个构造的元素调用destroy来销毁它们.  
// destroy接受一个指针,对指向的对象执行析构函数  
while(q!=p)
	alloc.destroy(--q); //释放真正构造的string
// --q是因为q一开始指向的是最后构造的元素之后的位置,所以需要向前删除元素.
```

拷贝和填充初始化内存的算法  
标准库为allocator类定义了两个伴随算法,可以在未初始化内存中创建对象.都定义在头文件memory中.  
<table>
	<tr>
		<th colspan="2">表12.8:allocator算法</th>	
	</tr>
	<tr>
		<th colspan="2">这些函数是在划分的内存块创建元素,而不是重新系统分配内存给它们</th>	
	</tr>
	<tr>
		<td>uninitialized_copy(b,e,b2)</td>
		<td>从迭代器b和e指出的输入范围中拷贝元素到迭代器b2指定的未构造的原始内存中.</td>
	</tr>
	<tr>
		<td>uninitialized_copy_n(b,e,b2)</td>
		<td>从迭代器b指向的元素开始,拷贝n个到b2开始的内存</td>
	</tr>
	<tr>
		<td>uninitialized_fill(b,e,t)</td>
		<td>从迭代器b和e指定的原始内存范围创建对象,对象的值均为t的拷贝.</td>
	</tr>	
	<tr>
		<td>uninitialized_fill_n(b,n,t)</td>
		<td>从迭代器b指向的元素开始创建n个对象,均为t的拷贝.</td>
	</tr>
</table>

12.3 使用标准库:文本查询程序  
实现一个简单的文本插叙程序,允许用户在一个给定文件中查询单词,查询结果是单词在文件中出现的次数以及其所在行的列表,如果一行出现多次,只列出一次.  

程序设计过程:  
1. 实现思路:
- 读取输入文件时,记住单词出现的每一行.因此,程序逐行读取输入文件,并将每一行分解为独立的单词
- 当程序生成输出时,能提取每个单词所关联的行号,行号必须按升序且无重复
- 打印给定行号的文本
2. 对应功能实现所需的标准库
- vector<string>来保存整个输入文件的一份拷贝.输入文件中的每一行保存为vector中的一个元素.当需要打印一行时,可以用行号作为下标来提取行文本.
- 使用一个istringstream来将每行分解为单词.
- 使用一个set来保存每个单词在输入文本出现的行号,且保证了每行只出现一次且按升序
- 使用一个map来将每个单词与它出现的行号set关联起来.就能提取任意单词的set  
- 还使用了shared_ptr
3. 数据结构  
- 虽然直接用vector,set和map可以，但可以定义一个类，TextQuery，包含一个vector和一个map，vector用来保存输入文件的文本，map用来关联每个单词和它出现的行号的set。这个类将会有一个直接用来读取给定输入文件的构造函数和一个执行查询的操作.
- 查询操作:查找map成员,检查给定单词是否出现.设定这个函数的难点在于要决定返回什么内容?需要知道它出现了多少次,出现的行号以及每行的文本.
- 返回这些内容最简单的方法是定义另一个类,命名为QueryResult,来保存查询结果,这个类由一个print函数,完成结果打印工作.  
- 这需要在两个类间共享数据.  

**在类间共享数据**: 由于QueryResult所需要的数据都保存在一个TextQuery对象中,可以选择拷贝,但拷贝浪费时间和内存.所以可以考虑通过返回指向TextQuery对象内部的迭代器(或指针)来避免拷贝操作.(这种方法需要避免TextQuery对象被提前销毁,不然QueryResult就将指向不存在的对象的数据.)

使用TextQuery类  
```c++
void runQueries(ifstream &infile)
{
	// infile 是一个ifstream,指向要处理的文件
	TextQuery tq(infile);  //保存文件并建立查询map
	// 与用户交互:提示用户输入要查询的单词,完成查询并打印结果
	while(true)
	{
		cout << "enter word to look for, or q to quit: ";
		string s;
		// 若遇到文件尾或用户输入'q'时循环停止
		if (!(cin >> s) || s == 'q') break;
		// 指向查询并打印结果
		print(cout, tq.query(s)) << endl;
	}
}
```
12.3.2 文本查询类的定义  
创建TextQuery类,用户创建此类对象提供一个istream,用来读取输入文件.这个类还提供一个query操作,接受一个string,返回一个QueryResult表示string出现的那些行.  

设计类的数据成员时,需要考虑与QueryResult对象共享数据的需求.QueryResult类需要共享保存输入文件的vector和保存单词关联的行号的set,因此这个类应该有两个数据成员:一个指向动态分配的vector(保存输入文件)的shared_ptr和一个string到shared_ptr<set>的map. map将文件中每个单词关联到一个动态分配的set上,而此set保存了该单词出现的行号.  
```c++
class QueryResult;  
class TextQuery
{
public:
	using line_no = std::vector<std::string>::size_type;
	TextQuery(std::ifstream&);
	QueryResult query(const std::string&) const;
private:
	std::shared_ptr<std::vector<std::string>> file; //输入文件
	// 每个单词到它所在的行号的集合的映射
	std::map<std::string, std::shared_ptr<std::set<line_no>> > wm;
};
```

TextQuery构造函数接受一个ifstream,逐行读取输入文件  
```c++
TextQuery::TextQuery(ifstream &is): file(new vector<string>)
{
	string text;
	while(getline(is, text))    //对文本的每一行
	{
		file->push_back(text);  //保存此行文本
		int n = file->size()-1;
		istringstream line(text);  //将此行文本分解为单词
		string word;
		while(line >> word)      //对行中每个单词
		{
			// 如果单词不在wm中,以之为下标在wm中添加一项
			auto &lines = wm[word];  //lines是一个引用
			if (!lines)  
				lines.reset(new set<line_no>);
			lines->insert(n);   //将此行号插入set
		}
	}
}

// QueryResult类
// 三个数据成员,string保存查询单词,shared_ptr指向保存输入文件的vector,一个shared_ptr,指向保存单词出现行号的set.唯一的成员函数是一个构造函数,初始化这三个数据成员:
class QueryResult
{
	friend std::ostream & print(std::ostream&, const QueryResult&);
public:
	QueryResult(std::string s,std::shared_ptr<std::set<line_no>> p, std::shared_ptr<std::vector<std::string>> f): sought(s), lines(p), file(f) {}
private:
	std::string sought;  //查询单词
	std::shared_ptr<std::set<line_no>> lines;      //出现的行号
	std::shared_ptr<std::vector<std::string>> file;  //输入文件
};

// query函数
TextQuery::query(const string &sought) const
{
	// 如果未找到sought,将返回一个指向该set的指针 
	static shared_ptr<set<line_no>> nodata(new set<line_no>);
	auto loc = wm.find(sought);
	if (loc == wm.end())
		return QueryResult(sought, nodata, file);
	else
		return QueryResult(sought, loc->second, file);
}
// print函数
ostream &print(ostream &os, const QueryResult &qr)
{
	// 如果找到了,打印出现次数和出现的位置
	os << qr.sought << "occurs" << qr.lines->size() << " "<< make_plural(qr.lines->size(),"time","s") << endl;}
	for (auto num: *qr.lines)
		os << "\t(line " <<num+1 << ") " << *(qr.file->begin() +num) << endl;
	return os;
```

### 第三部分 类设计者的工具
- 13章 拷贝控制
- 14章 操作重载与类型转换
- 15章 面向对象程序设计
- 16章 模板与泛型编程  

插入: 类的成员函数后面加 const，表明这个函数不会对这个类对象的数据成员（准确地说是非静态数据成员）作任何改变。对数据成员只读.不加const的成员函数是可读可写的.

第七章涵盖了类的所有基本知识:类作用域,数据隐藏以及构造函数,还介绍了类的一些重要特性:成员函数,隐式this指针,友元以及const,static和mutable成员.第三部分将延伸,介绍拷贝控制,重载运算符,继承和模板.  

13章将介绍通过定义构造函数来控制在类类型的对象初始化时做什么,类还可以控制在对象拷贝,赋值,移动和销毁时做什么.还会介绍新标准引入的两个重要概念:**右值引用和移动操作**.  

14章介绍运算符重载. 类可以重载的运算符中有一种特殊的运算符--**函数调用运算符**.还介绍了一种特殊类型的类成员函数--转换运算符.这些运算符定义了类类型的隐式转换机制.  

15章介绍继承和动态绑定.继承和动态绑定与数据抽象一起构成了面对对象编程的继承.继承令关联类型的定义更简单,而动态绑定可以帮助编写类型无关的代码,忽略具有继承关系的类型之间的差异.  

16章介绍函数模板和类模板.模板可以让我们写出与类型无关的通用类和函数.新标准引入了一些模板相关的新特性:可变参数模板,模板类型别名以及控制实例化的新方法.  

### 13章拷贝控制  
- 13.1 拷贝,赋值与销毁
- 13.2 拷贝控制和资源管理
- 13.3 交换操作
- 13.4 拷贝控制示例
- 13.5 动态内存管理类
- 13.6 对象移动

本章学习类如何控制该类型对象拷贝,赋值,移动或销毁时做什么.类通过拷贝控制操作copy control:拷贝构造函数copy constructor,拷贝赋值运算符copy-assignment operator,移动构造函数move constructor,移动赋值运算符move-assignment operator以及析构函数destructor来实现.  

13.1 拷贝,赋值与销毁  
13.1.1 拷贝构造函数  
拷贝构造函数的第一个参数必须是一个引用类型
```c++ 
class Foo()
{
public:
	Foo();  //默认构造函数
	Foo(const Foo &);  //拷贝构造函数
};  //拷贝构造函数通常是隐式的,而不是显示的构造.
```
如果没有主动为一个类定义拷贝构造函数,编译器会自动给一个合成拷贝构造函数(synthesized copy constructor),其大致就是将每个属性都对应进行复制,每个成员的类型决定其如何被拷贝,即使用该类型的拷贝方法进行拷贝,如类成员将用其拷贝构造函数进行拷贝  

```c++
// 拷贝初始化和直接初始化的差异
string dots(10, '.');  //直接初始化
string s(dots);        //直接初始化
string s2 = dots;      //拷贝初始化
string null_book = "9999-999"; //拷贝初始化
string nines = string(100, '9');  //拷贝初始化
```
**拷贝初始化不仅在使用=定义变量时发生**  
- 将一个对象作为实参传递给一个非引用类型的形参(即以非引用传入函数时,将进行拷贝操作)
- 从一个返回类型为非引用类型的函数返回一个对象(即,函数返回一个非引用类型时,将进行拷贝然后返回出去)
- 用花括号列表初始化一个数组中的元素或一个聚合类中的成员  
**某些类类型会对它们所分配的对象使用拷贝初始化:例如使用初始化标准库容器或是调用其insert或push成员时,容器会对其元素进行拷贝初始化. 而用emplace成员创建的元素是进行直接初始化.**  
拷贝/移动复制函数一般都是存在且可访问的(例如,不能是private的)  

13.1.2 拷贝赋值运算符  
类也可以控制对象如何赋值:  

```c++
Sales_data trans, accum;
trans = accum;  //使用Sales_data的拷贝赋值运算符
// 与拷贝构造函数一样,如果类未定义自己的拷贝赋值运算符,编译器会为它合成一个.
```

重载赋值运算符  
重载运算符本质上是函数,其名字由operator关键字接符号组成.  
```c++class Foo
{
public:
	Foo & operator=(const Foo&);  //赋值运算符
}; //赋值运算符通常返回一个指向其左侧运算对象的引用
```

同样的有个合成拷贝运算符(synthesized copy-assignment operator)  

13.1.3 析构函数  
无论何时一个对象被销毁,就会自动调用其析构函数:  
- 变量在离开其作用域时被销毁
- 当一个对象被销毁时,其成员被销毁
- 容器(无论是标准库容器还是数组)被销毁时,其元素被销毁
- 对于动态分配的对象,当对指向它的指针应用delete运算符时被销毁
- 对于临时对象,当创建它的完整表达式结束时被销毁

note:当指向一个对象的引用或指针离开作用域时,析构函数不会执行  \
同样的,合成析构函数(synthesized destructor)  

13.1.4 三/五法则  
需要析构函数的类也需要拷贝和赋值操作  
(例子还行,就是说合成拷贝函数可能只是简单的拷贝指针,而析构函数是delete,当对象A拷贝至B,析构A可能顺便delete了B的属性,这导致析构B时,会导致delete不存在的内存块,导致出错)

13.1.5 使用=default  
可以通过将拷贝控制成员定义为=default,来显示地要求编译器生成合成的版本  
```c++
class Sales_data
{
public:
	Sales_data() = default;
	Sales_data(const Sales_data	&) = default;
	Sales_data & operator=(const Sales_data &);
	~Sales_data()= default;
};
Sales_data& Sales_data::operator=(const Sales_data&) = default;
```
13.1.6 阻止拷贝  
在新标准下,可以通过将拷贝构造函数和拷贝赋值运算符定义为**删除的函数(deleted function)** 来阻止拷贝.  
删除函数是:声明了,但不能使用.  
```c++
struct NoCopy()
{
	NoCopy() = default;
	NoCopy(const NoCopy&) = delete;  //阻止拷贝
	NoCopy &operator=(const NoCopy&) = delete; //阻止赋值
	~NoCopy() = default; //析构函数不能是删除的成员
}; 
```

private拷贝控制  
新标准发布前,类是通过将其拷贝构造函数和拷贝赋值运算符声明为private来阻止拷贝  

13.2 拷贝控制和资源管理  
管理类外资源的类必须定义拷贝控制成员,这种类一般需要拷贝构造函数和一个拷贝赋值运算符,一般由两种拷贝语义:定义拷贝操作,使得类的行为看起来像一个值或指针.  

13.2.1 行为像值的类  
类值的行为，对于类管理的资源，每个对象都应该有一份自己的拷贝。对于ps指向的string，每个HasPtr对象必须有自己的拷贝，  
- 定义的拷贝构造函数，应该完成拷贝对象string，而不是拷贝指针
- 定义析构函数来释放string
- 定义一个拷贝赋值运算符来释放对象当前的string，并从右侧运算对象拷贝string  
```c++
class HasPtr
{
public:
	HasPtr(const std::string &s = std::string()): ps(new std::string(s)), i(0){} //对ps指向的string,每个HasPtr对象都有自己的拷贝.
	HasPtr(cosnt HasPtr &p): ps(new std::string(*p.ps)), i(p.i){}
	HasPtr & operator=(const HasPtr&);
	~HasPtr(){delete ps;}  //定义析构函数来释放string
private:
	std::string *ps;
	int i;
};
```
第一个构造函数接受一个(可选的)string参数.这个构造函数动态分配它自己的string副本,并将指向string的指针保存在ps中.拷贝构造函数也分配它自己的string副本.析构函数对指针成员ps执行delete,释放构造函数中分配的内存.

类值拷贝赋值运算符  
```c++
HasPtr& HasPtr::operator=(const HasPtr &rhs)
{
	auto newp = new string(*rhs.ps);  //释放底层string
	delete ps;  //释放旧内存
	ps = newp;
	i = rhs.i;
	return *this;  //返回本对象
}
```

13.2.2 定义行为像指针的类  
对于行为类似指针的类,需要定义其拷贝构造函数和拷贝赋值运算符,来拷贝指针成员本身而不是它指向的string.  
令一个类类似指针的行为,最好的方法是使用shared_ptr来管理类中的资源.有时希望直接管理资源,使用引用计数就很有用.  

**定义一个使用引用计数的类**  
```c++
class HasPtr
{
public:
	// 构造函数分配新的string和新的计数器,将计数器置为1
	HasPtr(const std::string &s = std::string()): ps(new std::string(s)), i(0), use(new std::size_t(1)) {}
	// 拷贝构造函数拷贝所有三个数据成员,并递增计数器
	HasPtr(const HasPtr &p): ps(p.ps), i(p.i), use(p.use){++*use;}
	HasPtr & operator=(const HasPtr&);
	~HasPtr();
private:
	std::string *ps;
	int i;
	std::size_t *use;  //记录多少个对象共享*ps的成员
};

HasPtr::~HasPtr   //析构函数在计数器为0时释放内存
{
	if (--*use == 0)
	{
		delete ps;
		delete use;   //释放计数器内存
	}
}
HasPtr & HasPtr:: operator=(const HasPtr &rhs)
{
	++*rhs.use;   //递增右侧运算对象的引用计数
	if(--*use == 0)   //减少左侧运算对象的计数器,并当如果左侧对象仅有一个对象的话释放该对象的内存.
	{
		delete ps;
		delete use;
	}
	ps = rhs.ps;
	i = rhs.i;
	use = rhs.use;
	return *this;
}
```

13.3 交换操作  
swap经常被定义为
```c++
HasPtr temp = v1;
v1 = v2;
v2 = temp;
```
但交换操作,我们希望swap函数是进行交换指针
```c++
string *temp = v1.ps;
v1.ps = v2.ps;
v2.ps = temp;
```
**编写自己的swap函数**  
```c++
class HasPtr
{
	friend void swap(HasPtr&, HasPtr&);
};
inline   //声明为内置函数
void swap(HasPtr &lhs, HasPtr &rhs)
{
	using std::swap;
	swap(lhs.ps,rhs.ps);  //交换指针而不是string数据
	swap(lhs.i, rhs.i);
// 注意这里使用的是swap而不是std::swap,因为指针和int是内置类型,内置类型没用自己的swap,所以本例中的swap会调用标准库std::swap. 但如果一个类的成员有自己类型特定的swap时,再写std::swap就会有问题.
}
```

13.4 拷贝控制示例  
设计两个类的设计,Message和Folder,分布表示电子邮件信息和消息目录.  
为了记录Message位于哪些Folder中,每个Message都会保存一个它所在Folder的指针的set,同样,每个Folder都保存一个它包含的Message的职责的set  
Message类会提供save和remove操作,来给一个给定Folder添加一条Message或是从中删除一条Message.  

```c++
class Message
{
	friend class Folder;
public:
	explicit Message(const std::string &str = ""):contents(str){}
	Message(const Message&);
	Message& operator=(const Message&);
	~Message();
	void save(Folder&);
	void remove(Folder&);
private:
	std::string contents;
	std::set<Folder*> folders;
	// 拷贝构造函数,拷贝赋值运算符和析构函数所使用的工具函数
	// 将本Message添加到指向参数的Folder中
	void add_to_Folder(const Message&);
	// 从folders中的每个Folder中删除本Message
	void remove_from_Folders();
};
void Message::save(Folder &f)
{
	folders.insert(&f);
	f.addMsg(this);
}
void Message::remove(Folder &f)
{
	folders.erase(&f);
	f.remMsg(this);
}
// 将本Message添加到指向m的Folder中
void Message::add_to_Folders(const Message &m)
{
	for(auto f : m.folders)
		f->addMsg(this);
}
Message::Message(const Message &m):contents(m.contents), folders(m.folders)
{
	add_to_Folders(m);
}
// 从对应的Folder中删除本Message
void Message::remove_from_Folders()
{
	for (auto f : folders)
		f->remMsg(this);
}
Message::~Message()
{
	remove_from_Folders();
}
void swap(Message &lhs, Message& rhs)
{
	using std::swap;
	for (aito f: lhs.folders)
		f->remMsg(&lhs);
	for (auto f: rhs.folders)
		f->remMsg(&rhs);
	swap(lhs.folders, rhs.folders);
	swap(lhs.contents,rhs.contents);
	for (auto f:lhs.folders)
		f->addMsg(&lhs);
	for (auto f:rhs.folders)
		f->addMsg(&rhs);
}
```

13.5 动态内存管理类  
某些类需要在运行时分配可变大小的内存空间。这种类通常可以用标准容器来保存它们的数据。  
某些类需要自己进行分配内存，这些类一般来说必须定义自己的拷贝控制成员来管理所分配的内存。这类就是动态内存管理类？  

实现标准库vector类的一个简化版本，只用于string--StrVec  
StrVec类的设计  
vector类将元素保存在连续内存中，预先分配足够的内存，接下来若没有足够内存，vector将重新分配新空间，将旧元素移动到新空间，释放旧空间，添加新元素。  
StrVec类将使用类似的策略，将使用allocator来获得原始内存,由于allocator分配的内存是未构造的,将在添加新元素时用allocator的construct成员在原始内存中创建对象.删除时用destroy成员来销毁元素.  
每个StrVec有三个指针成员指向其元素所使用的内存:  
- elements,指向分配的内存的首元素  
- first_free,指向最后一个实际元素之后的位置  
- cap,指向分配的内存末尾之后的位置  
![](2022-11-13-09-42-40.png)

除了这些指针外,StrVec还有一个名为alloc的静态成员,其类型为allocator<string>,alloc成员会分配StrVec使用的内存,该类还有4个工具函数:  
- alloc_n_copy会分配内存,并拷贝一个给定范围中的元素  
- free会销毁构造的元素并释放内存  
- chk_n_alloc保证StrVec至少容纳一个新的元素的空间,如果没有空间添加新元素,chk_n_alloc会调用reallocate来分配更多内存.  
- reallocate在内存用完时为StrVec分配新内存  

```c++
class StrVec
{
public:
	StrVec(): elements(nullptr), first_free(nullptr),cap(nullptr){}
	StrVec(const StrVec&);  //拷贝构造函数
	StrVec &operator=(const StrVec&);  //拷贝赋值运算符
	~StrVec();
	void push_back(const std::string&);  
	size_t size() const {return first_free - elements;}
	size_t capacity() const {return cap - elements;}
	std::string *begin() const {return elements;}
	std::string *end() const {return first_free;}

private:
	Static std::allocator<std::string> alloc;  //分配元素
	// 被添加元素的函数所使用
	void chk_n_alloc(){if(size()==capacity()) reallocate();}
	// 工具函数,被拷贝构造函数,赋值运算符和析构函数所使用
	std::pair<std::string*, std::string*> alloc_n_copy(const std::string*,const std::string*);
	void free();  //销毁元素并释放内存
	void reallocate();  //获得更多内存并拷贝已有元素
	std::string *elements;
	std::string *first_free;
	std::string * cap;
};
```

使用construct  
```c++
void StrVec::push_back(const string& s)
{
	chk_n_alloc();  //确保有空间容纳新元素
	alloc.construct(first_free++, s);
}
// allocator分配内存时,记住内存是未构造的,为使用此原始内存,必须调用construct,在此内存中构造一个对象.传递给construct的第一个参数是一个指针,指向调用allocate所分配的未构造的内存空间.
```

alloc_n_copy成员  
```c++
pair<string*, string*> StrVec::alloc_n_copy(const string *b, const string *e)
{
	// 分配空间保存给定范围中的元素
	auto data = alloc.allocate(e - b);
	// 初始化并返回一个pair,该pair由data和uninitialized_copy的返回值构成
	return {data, uninitialized_copy(b,e,data)};
}
```
alloc_n_copy用尾后指针减去首元素指针来计算需要多少空间,分配内存后,在此空间构造给定元素的副本,  
在返回语句中完成拷贝,返回语句中对返回值进行了列表初始化.返回pair的first成员指向分配的内存的开始位置;second成员则是uninitialized_copy的返回值,此值是一个指向最后一个构造元素之后位置的指针.  

free成员  
两个责任:首先destroy元素,然后调用deallocate来释放StrVec自己分配的内存空间.  
```c++
void StrVec::free()
{
	// 不能传递给deallocate一个空指针,如果elements为0,函数什么也不做
	if(elements)
	{
		for (auto p = first_free; p! = elements; /**/)
			alloc.destroy(--p);
		alloc.deallocate(elements, cap - elements);
	}
}
```

拷贝控制成员  
```c++
StrVec::StrVec(const StrVec &s)
{
	auto newdata = alloc_n_copy(s.begin(),s.end());
	elements = newdata.first;
	first_free = cap = newdata.second;
}
```
alloc_n_copy返回是一个指针的pair,first成员指向第一个构造的元素,second指向最后一个构造的元素之后的位置  

析构函数
```c++
StrVec::~StrVec(){free();}
```

拷贝赋值运算符  
```c++
StrVec &StrVec::operator=(const StrVec &rhs)
{
	auto data = alloc_n_copy(rhs.begin(),rhs.end());
	free();
	elements = data.first;
	first_free = cap = data.second;
	return *this;
}
```

在重新分配内存的过程中移动而不是拷贝元素  
reallocate成员函数:   
- 为一个新的更大的string数组分配内存
- 在内存空间的前一部分构造对象,保存现有元素
- 销毁原内存空间中的元素,并释放这块内存  

移动构造函数和std::move  
通过新标准库引入的两种机制,就可以避免string的拷贝.  
首先,有一些标准库类,包括string,都定义了所谓的"移动构造函数".  
移动构造函数是将资源从给定对象"移动"而不是拷贝正在创建的对象.  

第二个机制是一个名为move的标准库函数,它定义在utility头文件中.  
关于move需要了解两个关键点:1.reallocate在新内存中构造string时,必须调用move来表示使用string的移动构造函数,而不是使用拷贝构造函数.  
其次,通常不为move提供一个using声明,即直接调用std::move  

reallocate成员  
```c++
void StrVec::reallocate()
{
	// 分配当前两倍大小的内存空间
	auto newcapacity = size() ? 2*size() : 1;
	auto newdata = alloc.allocate(newcapacity);
	// 将数据从旧内存移动到新内存  
	auto dest = newdata;  //指向新数组中下一个空闲位置
	auto elem = elements;  //指向旧数组中下一个元素
	for (size_t i = 0; i != size(); ++i)
		alloc.construct(dest++, std::move(*elem++));
	free();  //移动完释放旧内存空间
	// 更新数据结构
	elements = newdata;
	first_free = dest;
	cap = elements + newcapacity;
}
```
construct的第二个参数(确定使用哪个构造函数的参数)是move返回的值.调用move返回的结果会令construct使用string的移动构造函数.由于使用移动构造函数,string管理的内存将不会被拷贝,相反构造的每个string都会从elem指向的string那里接管内存的所有权.  

13.6 对象移动  
使用移动而不是拷贝的一个原因是能大幅提示性能,另一个原因源于IO类或unique_ptr类,这些类都包含不能被共享的资源(如指针或IO缓冲).因此这些类型的对象不能拷贝但可以移动.  

13.6.1 右值引用  
为支持移动操作,新标准引入了一种新的引用类型--右值引用(rvalue reference), 所谓右值引用就是必须绑定到右值的引用,通过使用&&而不是&来获得右值引用.  
右值引用一个重要的性质--只能绑定到一个将要销毁的对象.因此,可以自由地将一个右值引用的自由"移动"到另一个对象中.  
常规引用(可称为左值引用lvalue reference),将其绑定到左值  
右值引用需要绑定到要求转换的表达式,字面常量或是返回右值的表达式  
```c++
int i = 42;
int &r = i;  //r左值引用
int &&rr = i;  //错误,不能将右值引用绑定到左值上
int &r2 = i*42;   //错误,i*42是一个右值  
int &&rr2 = i*42;  //正确,rr2绑定到乘法结果
```

左值持久:右值短暂  
左值由持久的状态,右值要么是字面常量,要么是在表达式求值中创建的临时对象.  
右值引用只能绑定到临时对象,可知
- 所引用的对象将要被销毁
- 该对象没有其他用户  
这两个特性意味着:使用右值引用的代码可以自由地接管所引用的对象的资源  

变量是左值,不能将一个右值引用直接绑定到一个变量上,即使这个变量是右值引用类型也不行.  

标准库move函数  
虽然不能将一个右值引用直接绑定到一个左值上,但可以显式地将一个左值转换为对应的右值引用类型. 还可以通过调用move的新标准库函数来获得绑定到左值上的右值引用,此函数定义在头文件utility中.move函数返回给定对象的右值引用.  
```c++
int &&rr1 = 42;  //字面常量是右值
int &&rr2 = rr1;  //错误
int &&rr3 = std::move(rr1);  //ok
```
move调用告诉编译器:我们有一个左值,但我们希望像一个右值一样处理它.但同时,调用move旧意味着,除了对rr1赋值或销毁外,将不再使用它.  

13.6.2 移动构造函数和移动赋值运算符  
类似拷贝构造函数，移动构造函数的第一个参数是该类类型的一个引用。不同于拷贝构造函数的是，这个引用参数在移动构造函数中是一个右值引用。与拷贝构造函数一样，任何额外的参数都必须有默认实参。  

```c++
StrVec::StrVec(StrVec &&s) noexcept :elements(s.elements), first_free(s.first_free), cap(s.cap)  //移动操作不应该抛出任何异常
{
	// 令s进入这一的状态:对其运行析构函数是安全的
	s.elements = s.first_free = s.cap = nullptr;
}
// noexcept(通知标准库我们的构造函数不跑出任何异常)
```
与拷贝构造函数不同,移动构造函数不分配任何新内存;它接管给定的StrVec中的内存.在接管内存后,将给定对象的指针都置为nullptr.

移动操作,标准库容器和异常.  
当编写一个不抛出异常的移动操作时,需要将此事通知标准库.  
一种通知标准库的方法是在我们的构造函数中指明noexcept.在一个构造函数中,noexcept出现在参数列表和初始化列表开始的冒号之间:  
```c++
class StrVec
{
public:
	StrVec(StrVec&&) noexcept;  //移动构造函数
};
StrVec::StrVec(StrVec &&s) noexcept : /* 成员初始化器*/
{/*构造函数体*/}
// 必须在类头文件的声明和定义中(如果定义在类外的话)都指定noexcept
// 不抛出异常的移动构造函数和移动赋值运算符必须标记为noexcept
```

移动赋值运算符  
```c++
StrVec &StrVec::operator=(StrVec &&rhs) noexcept
{
	// 直接检测自赋值
	if (this != &rhs)
	{
		free();  //释放已有元素
		elements = rhs.elements;  //从rhs接管资源
		first_free = rhs.first_free;
		cap = rhs.cap;
		// 将rhs置于可析构状态(很重要的一步)
		rhs.elements = rhs.first_free = rhs.cap = nullptr;
	}
	return *this;
}
```

合成的移动操作  
与拷贝操作不同的是,对于某些类,当没有自定义移动构造函数时,编译器不会生成合成移动操作,将用拷贝操作替代.  

略至移动迭代器  

移动迭代器  
新标准库中定义了一种移动迭代器(move iterator)适配器.一个移动迭代器通过改变给定迭代器的解引用运算符的行为来适配此迭代器.  
一般来说,一个迭代器的解引用运算符返回的是一个指向元素的左值.而移动迭代器的解引用运算符将生成一个右值引用.  

通过调用标准库的make_move_iterator函数将一个普通迭代器转换为一个移动迭代器.此函数接受一个迭代器参数,返回一个移动迭代器(移动迭代器支持正常的迭代器操作)  
可以将一对移动迭代器传递给算法,特别是,可以将移动迭代器传递给uninitialized_copy:  
```c++
void StrVec::reallocate()
{
	// 分配大小两倍于当前规模的内存空间
	auto newcapacity = size() ? 2*size() :1;
	auto first = alloc.allocate(newcapacity);
	//移动元素
	auto last = uninitialized_copy(make_move_iterator(begin()),make_move_iterator(end()),first);
	free();  
	elements = first;
	first_first = last;
	cap = elements + newcapacity;
}
```

13.6.3 右值引用和成员函数  
因为移动构造运算符和拷贝构造运算符一样,成员函数就得看输入的参数是右值还是左值从而判断是移动构造函数还是拷贝构造函数.右值引用版本调用move来将其参数传递给construct.  

因为目前是允许向右值赋值,但这个很奇怪(如下例子),有时需要阻止这种使用方式,希望强制左侧运算对象(即,this指向的对象)是一个左值.
```c++
string s1= "asdf", s2 = "asdfa";
s1+s2 = "wow;  //s1+s2是一个临时对象,右值,却可以被赋值
```
即希望强制左侧运算对象s1+s2为左值.在参数列表后放置一个**引用限定符**(reference qualifier):
```c++
class Foo
{
public:
	Foo &operator = (const Foo&) &; //引用限定符,只能向可修改的左值赋值
};
Foo &Foo::operator=(const Foo &rhs) &
{   //执行将rhs赋予本对象所需的操作
	return *this;
}
```
引用限定符可以是&或&&,分别指出this可以指向一个左值还是右值.类似const限定符,引用限定符只能用于(非static)成员函数,且必须同时出现在函数的声明和定义中.
```c++
Foo &retFoo();  //返回一个引用,retFoo调用是一个左值
Foo retVal();  //返回一个值,retVal调用是一个右值
Foo i, j;  //i,j是左值
i = j;  //正确,i是左值
retFoo() = j;  //正确,retFoo()返回一个左值
retVal() = j; //错误,retVal()返回一个右值
i = retVal();  //正确,可以将一个右值作为赋值操作的右侧运算对象
```
一个函数可以同时使用const和引用限定.引用限定符必须跟随在const限定符之后:  
```c++
class Foo
{
public:
Foo someMen() & const;  //错误
Foo anotherMen() const &; //正确
}
```

重载和引用函数  
就像一个成员函数可以根据是否有const来区分其重载版本一样.引用限定符也可以区分重载版本.因此可以使用const和引用限定符来定制各个情况的重载版本.  
```c++
class Foo()
{
public:
	Foo sorted() &&;  //用于可改变的右值
	Foo sorted() const &;  //可用于任何类型的Foo
private:
	vector<int> data;
};
Foo Foo::sorted() &&
{
	sort(data.begin(),data.end());
	return *this;
} //本对象为右值,因此可以原址排序
Foo Foo::sorted() const &
{
	Foo ret(*this);  // 拷贝一个副本
	sort(ret.data.begin(), ret.data.end());  //排序副本
	return ret;   //返回副本
}

retVal().sorted();  //retVal()是一个右值,调用Foo::sorted() &&
retFoo().sorted();  //retFoo()是一个左值,调用Foo::sorted() const &
```

如果定义两个或两个以上具有相同名字和相同参数列表的成员函数,就必须对所有函数加上引用限定符,或所有都不加:  
```c++
class Foo
{
public: 
	Foo sorted() &&;
	Foo sorted() const;  //错误,必须加上引用限定符
}
```

**如果一个成员函数有引用限定符,则具有相同参数列表的所有版本都必须有引用限定符**

第14章 重载运算和类型转换  
- 14.2 输入和输出运算符
- 14.3 算术和关系运算符
- 14.4 赋值运算符
- 14.5 下标运算符
- 14.6 递增和递减运算符
- 14.7 成员访问运算符
- 14.8 函数调用运算符
- 14.9 重载,类型转换与运算符

在第4章中，C++定义了大量运算符以及内置类型的自动转换规则。和内置类型的转换一样，类类型转换隐式地将一种类型的对象转换成另一种我们所需类型的对象。  

只能重载已有的运算符，无权发明新的运算符号，要发明新的运算符号不如定义新的成员函数来的直接。  
对于一个重载的运算符，其优先级和结合律与对应的内置运算符保持一致。  

**如果一个运算符函数是成员函数,则它的第一个(左侧)运算对象绑定到隐式的this指针上.因此,成员运算符函数的(显式)参数数量比运算符的运算对象总数少一个.**
```c++
// 一个非成员运算符函数的等价调用  
data1 + data2;  //普通的表达式
operator+(data1,data2);  //等价的函数调用
// 都调用了非成员函数operator+,传入data1作为第一个实参,data2作为第二个实参.
// 一个成员函数运算符函数的等价调用
data1 += data2;  //基于"调用"的表达式
data1.operator+=(data2);  //对成员运算符函数的等价调用
// 上面两条语句都调用了成员函数operator+=,||将this绑定到data1的地址||,将data2作为实参传入函数.
```

选择作为成员或者非成员  
- 赋值(=),下标([]),调用(())和成员访问箭头(->)运算符必须是成员  
- 复合赋值运算符一般来说应该是成员,但不是必须,这一点与赋值运算符不同
- 改变对象状态的运算符或者与给定类型密切相关的运算符,如递增,递减和解引用运算符,通常应该是成员
- 具有对称性的运算可能转换任意一端的运算对象,例如算术,相等性,关系和位运算符等,通常应该是普通的非成员函数. 

当把运算符定义为成员函数时,它的左侧运算对象必须是运算符所属类的一个对象
```c++
string s = "world";
string t = s + "!";  //正确,将一个const char*加到一个string中
string u = "hi" + s;  //当+运算符是string的成员时就会出错.因为是左侧"hi"调用+,而"hi"不是string类的
```

14.2 输入和输出运算符  
IO标准库分别使用>>和<<执行出入和输出操作.这两个运算符,IO库定义了用其读写内置类型的版本,而类则需要自定义其对象的新版本以支持IO操作  
14.2.1 重载输出运算符<<  
```c++
ostream &operator<<(ostream &os, const Sales_data &item)
{
	os << item.isbn() << " " << item.units_sold << " " << item.revenue << " " << item.avg_prive();
	return os;
}
```
如上例子,通常情况下,返回类型是ostream &(ostream的引用); 输入的第一个参数是非常量ostream对象的引用.ostream是非常量是因为向流写入内容会改变其状态; 该形参是引用是因为无法直接复制一个ostream对象.  
第二个形参一般来说是一个常量的引用,该常量是想要打印的类类型.其是引用是为了避免复制实参.  

输出运算符尽量减少格式化操作: 尤其是不会打印换行符.如果打印了换行符,则用户就无法在用户的同一行内接着打印东西了.  

输入输出运算符必须是非成员函数(即上面例子是ostream& operator<< 而不是Sales_data:: ostream& operator<<)  
不然的话,左侧运算对象就必须是类的一个对象.  
```c++
Sales_data data;
data << cout;  //如果operatro<< 是Sales_data的成员
// 因此如果希望类自定义IO运算符,就必须将其定义为非成员函数,当然,此时一般需要读写类的非公有数据成员,所以一般IO运算符被声明为友元
```  

14.2.2 重载输入运算符>>
```c++
istream &operator>>(istream &is, Sales_data &item)
{
	double price;   //不需要初始化，因为将先读入数据到price，之后才使用它
	is >> item.bookNo >> item.units_sold >> price;
	if (is) 
		item.revenue = item.units_sold * price;
	else 
		item = Sales_data();  //输入失败的话就赋值默认值
	return is;
}
```
同样的, 返回类型是istream &,输入一个istream &,此时需要注意输入失败的情况.

14.3 算术和关系运算符  
通常情况下,算术和关系运算符定义为非成员函数以允许对左侧或右侧的运算对象进行转换.  

14.3.1 相等运算符  
14.3.2 关系运算符  
14.4.4 赋值运算符  
除了之前的拷贝赋值和移动赋值运算符外,类还可以定义其他赋值运算符以使用别的类型作为右侧运算对象.  
例如,vector类还定义了第三种赋值运算符  
```c++
vector<string> v;
v = {"a", "an", "the"};
//即新的赋值方式,右边为花括号
```
复合赋值运算符(比如+=,-=等)  
复合赋值运算符不非得是类的成员,不过还是倾向于把包括复合赋值在内的所有赋值运算都定义在类的内部. 为了与内置类型的复合赋值保持一致,类中的复合赋值运算符也返回其左侧运算对象的引用.
```c++
Sales_data & Sales_data::operator+=(const Sales_data &rhs)
// 类的成员函数
{
	units_sold += rhs.units_sold;
	revenue += rhs.revenue;
	return *this;
}
// 赋值运算符必须定义成类的成员,复合赋值运算符通常也应该是,但不是必须.这两类运算符都返回左侧运算对象的引用.
```
14.5 下标运算符  
**下标运算符必须是成员函数**  
表示容器的类通常可以通过元素在容器中的位置访问元素,这些类一般会定义下标运算符operator\[\].  

**如果一个类包含下标运算符,则一般定义两个版本: 一个返回普通引用,另一个是类的常量成员并返回常量引用.  
```c++
class StrVec
{
public:
	std::string& operator[](std::size_t n){return elements[n];}  //返回普通引用
	const std::string & operator[](std::size_t n) const {return elements[n]}  //返回常量引用
private:
	std::string * elements;  //指向数组首元素的指针
};
```
当StrVec非常量时,可以给元素赋值,当对常量对象取下标时,不能为其赋值  

14.6 递增和递减运算符  
在迭代器类中通常会实现递增运算符(++)和递减运算符(--),这两种运算符使得可以在元素的序列中前后移动.  
C++不要求递增递减运算符必须是类的成员,但因为改变的是所操作对象的状态,所以建议将其设置为成员函数  

定义前置递增/递减运算符  
```c++
class StrBlobPtr
{
public:
	StrBlobPtr& operator++();
	StrBlobPtr& operator--();
};
```
递增和递减运算符的工作原理很相似:首先**调用check函数**检验StrBlobPtr是否有效,如果是,检查给定的索引是否有效.如果check函数没有抛出异常,则运算符返回对象的引用.  
```c++
StrBlobPtr& StrBlobPtr::operator++()
{
	// 如果curr已经指向容器的尾后位置,则无法递增它
	check(curr, "increment past end of StrBlobPtr");
	++curr;  //将curr在当前状态向前移动一个元素.
	return *this;
}
// StrBlobPtr& StrBlobPtr::operator--()类似(--curr略)
```

**区分前置和后置运算符**  
因为这类运算符都一样,普通的重载形式无法区分这两种情况.  
为解决这个问题,后置版本接受一个额外的(不被使用)int类型的形参.当使用后置运算符时,编译器为这个形参提供一个值为0的实参.这个形参的唯一作用就是区分前置版本和后置版本的函数,而不是真的要实现后置版本时参与运算.  
```c++
class StrBlobPtr
{
public:
	StrBlobPtr& operator++(int);  //后置运算符
	StrBlobPtr& operator--(int);  //后置运算符
};

```c++
StrBlobPtr& StrBlobPtr::operator++(int)
{
	// 此处无须检查有效性,调用前置递增运算符时才需要检查
	StrBlobPtr ret = *this;   //记录当前的值,然后返回该复制的值
	++*this;  //向前移动一个元素,前置++需要检查递增的有效性
	return ret;   
}
// StrBlobPtr& StrBlobPtr::operator--()类似(--*this略)
```
**显式地调用后置运算符**  
```c++
StrBlobPtr p(al);  //p指向a1中的vector
p.operator++(0);  //调用后置版本的operator++
p.operator++();  //调用前置版本的operator++
```

14.7 成员访问运算符  
在迭代器类及智能指针类中常常用到解引用运算符(*)和箭头运算符(->)
```c++
class StrBlobPtr
{
public:
	std::string& operator*() const
	{
		auto p = check(curr, "derefernce past end");  //解引用运算符首先检查curr是否仍在作用范围内.是,返回curr所指元素的一个引用.
		return (*p)[curr];  //(*p)是对象所指的vector
	}
	std::string* operator->() const
	{ //箭头运算符不执行自己的操作,而是调用解引用运算符并返回解引用结果元素的地址.
		return & this->operator*();
	}
}
//箭头运算符必须是类的成员. 解引用通常也是,但不是必须 
```
注意:这两个运算符定义为const成员,这是因为获取一个元素并不会改变StrBlobPtr对象的状态,同时,返回值分别是非常量string的引用或指针,因为一个StrBlobPtr只能绑定到非常量的StrBlobPtr对象.  

```c++
(*point).mem;  //point是一个内置的指针类型
point.operator()->mem;  //point是类的一个对象
```

14.8 函数调用运算符  
```c++
class PrintString
{
public:
	PrintString(ostream &o = cout, char c = ' '): os(o), sep(c){}
	void operator()(const string &s) const {os<<s<<sep;}
private:
	ostream &os;  //用于写入的目的流
	char sep;	  //用于将不同输出隔开的字符
};
PrintString printer;
printer(s);  //在cout中打印s,后面跟一个空格
PrintString errors(cerr, '\n');
errors(s);  //在cerr中打印s,后面跟一个换行符
// 函数对象通常作为泛型算法的实参.例如,可以使用标准库for_each算法和PrintString类来打印容器的内容
for_each(vs.begin(), vs.end(), PrintString(cerr,'\n'));
// for_each的第三个参数是类型PrintString的一个临时对象.
```

14.8.1 lambda是函数对象  
```c++
// 根据单词的长度对其进行排序,对于长度相同的按字母表顺序排序
stable_sort(words.begin(), words.end(), [](const string &a, const string &b){return a.size() < b.size();});
// 该行为类似下面这个类的一个未命名对象
class ShorterString
{
public:
	bool operator() (const string &s1, const string &s2) const {return s1.size()<s2.size();}
};
// 用这个类代替lambda表达式,重写调用stable_sort
stable_sort(words.begin(), words.end(),ShorterString());
// 第三个实参是新构建的ShorterString对象,当stable_sort内部的代码每次毕竟string时就会调用该对象,此时对象将调用运算符的函数体.
```

表示lambda及相应捕获行为的类  
当一个lambda表达式通过引用捕获变量时,编译器可以直接使用该引用而无须在lambda产生的类中将其存储为数据成员  
当一个lambda表达式通过值捕获变量,将值捕获的变量拷贝到lambda中.因此这种lambda产生的类必须为每个值捕获的变量建立对应的数据成员,同时创建构造函数,令其使用捕获的变量的值来初始化数据成员.  
```c++
// 获得第一个指向满足条件元素的迭代器,该元素满足size() is >= sz
auto wc = find_if(words.begin(), words.end(), [sz](const string &a){return a.size() >= sz;});

// 该lambda表达式产生的类将如下:
class SizeComp
{
	SizeComp(size_t n): sz(n) {}  //该形参对应捕获的变量
	//该调用运算符的返回类型,形参和函数体都与lambda一致
	bool operator()(const string &s) const
	{return s.size()>=sz;}
private:
	size_t sz;  //该数据成员对应通过值捕获的变量
};

auto wc = find_if(words.begin(), words.end(), SizeComp(sz));
```
lambda表达式产生的类,不含默认构造函数,赋值运算符及默认析构函数;它是否含有默认的拷贝/移动构造函数则要看捕获的数据类型而定

14.8.2 标准库定义的函数对象  
![](2022-11-15-15-19-43.png)

**在算法中使用标准库函数对象**  
```c++
// 传入一个临时的函数对象用于执行两个string对象的>比较运算
sort(svec.begin(), svec.end(), greater<string>());
// 当使用sort比较元素时,不再是使用默认的<运算符,而是调用给定的greater函数对象.该对象负责在string元素之间执行>比较运算

// 需要注意的是,标准库规定其函数对象对于指针同样适用.  
vector<string*> nameTable;
sort(nameTable.begin(),nameTable.end(), [](string *a, string *b){return a < b});  //错误,因为string未定义<行为
sort(nameTable.begin(),nameTable.end(),less<string*>());//正确,使用标准库规定指针的less.
```
14.8.3 可调用对象与function  
C++中有几种可调用的对象:函数,函数指针,lambda表达式,bind创建的对象以及重载了函数调用运算符的类(例子:上面lambda产生的类)

不同类型可能具有相同的调用形式  
```c++
int add(int i, int j){return i+j;} //普通函数
auto mod = [](int i, int j){return i%j;};  //lambda,其产生一个未命名的函数对象类
struct divide
{
	int operator()(int denominator, int divisor){return denominator / divisor;}
};  //函数对象类
```
上面这些可调用对象分别对参数执行了不同的算术运算,但共享同一种调用形式:  
```c++
int (int, int)
```
我们希望使用这些可调用对象构建一个简单的桌面计算器.为了实现这一目的,需要定义一个函数表(function table)用于存储指向这些可调用对象的"指针",当程序需要执行某个特定的操作时,从表中查找该调用的函数.  
C++中,函数表使用map来实现.对于此例子,用运算符符号的string作为关键字,使用实现运算符的函数作为值.  
```c++
// 构建从运算符到函数指针的映射关系,其中函数接受两个int,返回一个int
map<string, int(*)(int, int)> binops;
binops.insert({"+",add});  //{"+",add}是一个pair  
// 但不能将mod或divide存入,因为不是函数指针  
binops.insert({"%", mod});  //mod不是一个函数指针,mod是一个lambda表达式,每个lambda有自己的类类型.  
```
标准库function类型  
可以使用一个名为**function**的新的标准库类型解决上述问题.function定义在function头文件中.  
<table>
	<tr>
		<th colspan="2">表14.3: function的操作</th>
	</tr>
	<tr>
		<td>function<T> f;</td>
		<td>f是一个用来存储可调用对象的空function,这些可调用对象的调用形式应该与函数类型T相同</td>
	</tr>
	<tr>
		<td>function<T> f(nullptr)</td>
		<td>显式地构造一个空的function</td>
	</tr>
	<tr>
		<td>function<T> f(obj)</td>
		<td>在f中存储可调用对象obj的副本</td>
	</tr>
	<tr>
		<td>f</td>
		<td>将f作为条件:当f含有一个可调用对象时为真,否则为假</td>
	</tr>
	<tr>
		<td>f(args)</td>
		<td>调用f中的对象,参数是args</td>
	</tr>
	<tr>
		<th colspan="2">定义为function<T>的成员的类型</th>
	</tr>
	<tr>
		<td>result_type</td>
		<td>该function类型的可调用对象返回的类型</td>
	</tr>
	<tr>
		<td>argument_type</td>
		<td>当T有一个或两个实参时定义的类型.如果T只有一个实参,则argument_type是该类型的同义词;</td>
	</tr>
	<tr>
		<td>first_argument_type</td>
		<td>如果T有两个实参,则first_argument_type,second_argument_type分别代表两个实参的类型</td>
	</tr>
	<tr>
		<td>second_argument_type</td>
		<td> </td>
	</tr>
</table>

```c++
function<int(int,int)> f1 = add;  //函数指针
function<int(int,int)> f2 = divide();  // 函数对象类的对象
function<int(int,int)> f3 = [](int i, int j){return i*j;}; //lambda
cout << f1(4,2) << endl;
cout << f2(4,2) << endl;
cout << f3(4,2) << endl;
```

使用这个function类型可以重新定义map:  
- 列举可调用对象与二元运算符对应关系的表格
- 所有可调用对象都必须接受两个int,返回一个int
- 其中的元素可以是函数指针,函数对象或lambda
```c++ map<string, function<int(int, int)>> binops ```
能把所有可调用对象,包括函数指针,lambda或者函数对象在内,都添加到这个map中  
```c++ map<string, function<int(int,int)>> binops = 
{
	{"+",add},  //函数指针
	{"-",std::minus<int>()},  //标准库函数对象
	{"/",divide()},  //用户定义的函数对象
	{"*",[](int i, int j){return i*j;}},  //未命名的lambda
	{"%",mod}  // 命名了的lambda对象
}
```
索引binops,将得到function对象的引用.function类型重载了调用运算符,该运算符接受它自己的实参,然后将其传递给存好的可调用对象:  
```c++
binops["+"](10,5); //调用add(10,5),其他四个类似,略
```

重载的函数与function  
当相同的名字的函数有重载版本时,就很难分清是哪个  
```c++
int add(int i, int j){return i+j;}
Sales_data add(const Sales_data&, const Sales_data&);
map<string, function<int(int,int)>> binops;
binops.insert("+",add);  //无法分清是哪个add,错误
// 解决二义性的方法一:存储函数指针
int (*fp)(int, int) = add;  //将指针后面的类型来分清
binops.insert({"+",fp});  //正确,fp指向一个正确的add版本
// 方法二:使用lambda(即用其输入的参数来分辨)
binops.insert({"+", [](int a, int b){return add(a,b);}});
```

14.9 重载, 类型转换与运算符  
7.5.4一个实参调用的非显式构造函数定义了一种隐式的类型转换,这种构造函数将实参类型的对象转换为类类型.  

对于类类型的类型转换,通过定义 类型转换运算符实现.  
转换构造函数和类型转换运算符共同定义了**类类型转换**(class-type conversions),有时也被称为用户定义的类型转换(user-defined conversions)

14.9.1 类型转换运算符  
类型转换运算符(conversion operator)是类的一种特殊成员函数,负责将一个类类型的值转换成其他类型.类型转换函数的一般形式如下:  
```c++ operator type() const; ```
type()表示某种类型.类型转换运算符可以面向任意类型(除void)进行定义.  
一个类型转换函数必须是类的成员函数; 不能声明返回类型,形参列表也必须为空.类型转换函数通常是const  

```c++
class SmallInt
{
private:
	std::size_t val;
public:
	SmallInt(int i = 0):val(i)  //构造函数将算术类型值转换为SmallInt对象.
	{
		if(i<0||i>255)\
			throw std::out_of_range("Bad SmallInt value");
	}
	operator int() const{return val;}  //类型转换函数/将SmallInt对象转换为int
};
SmallInt si;
si = 4; //将4隐式转换为SmallInt,然后调用SmallInt::operator=
si + 3; //将si隐式转换为int,然后执行整数的加法
SmallInt si = 3.14;  //调用SmallInt(int)构造函数
// SmallInt的类型转换运算符将si转换成int
si +3.14;  //内置类型转换将所得的int继续转换成double
```

**类型转换运算符是隐式执行的,所以无法传递实参参数列表为空,没有返回类型, 返回一个对于类型的值.  
```c++
class SmallInt;
operator int(SmallInt&);  //错误,不是成员函数.
class SmallInt
{
public:
	int operator int() const;  //错误,应该无返回类型
	operator int(int = 0) const;  //错误,参数列表应为空
	operator int*() const{return 42;}  //int*为int指针,而返回的是42不是int指针.
}
```

显式的类型转换运算符(为避免编译器自动执行类型转换而造成异常)  
```c++
public:
	explicit operator int() const {return val;}  //编译器不会自动执行类型转换
SmallInt si = 3;  //正确,SmallInt的构造函数不是显式
si + 3;  //错误,这里需要隐式的类型转换,但类的运算符是显式的
static_cast<int>(si) +3;  //正确,显式地请求类型转换
```
但上述存在例外,就是当表达式用作条件时,编译器会将显式的类型转换自动使用.  
当表达式出现在以下位置时,显式的类型转换将被隐式地执行:  
- if, while及do语句的条件部分
- for语句头的条件表达式
- 逻辑非运算符(!), 逻辑或运算符(||),逻辑与运算符(&&)的运算对象
- 条件运算符(?:)的条件表达式

14.9.2 避免有二义性的类型转换  
14.9.3 函数匹配与重载运算符略  

如果类重载了函数调用运算符operator()则该类的对象被称为"函数对象".这样的对象常用于标准函数中.lambda表达式是一种简便的定义函数对象类的方式.  

在类中可以定义转换源或转换目的是该类型本身的类型转换,这一的类型转换将自动执行.  
只接受单独一个实参的非显式构造函数定义了从实参类型到类类型的类型转换  
而非显式的类型转换运算符则定义了从类类型到其他类型的转换

第15章 面向对象程序设计  
- 15.1 OOP：概述
- 15.2 定义基类和派生类
- 15.3 虚函数
- 15.4 抽象基类
- 15.5 访问控制与继承
- 15.6 继承中的类作用域
- 15.7 构造函数与拷贝控制
- 15.8 容器与继承
- 15.9 文本查询程序再探  

OOP基于三个基本概念：数据抽象，继承和动态绑定。第7章已经介绍了数据抽象，本章介绍继承和动态绑定。  

15.1 OOP：概述  
OOP的核心就是数据抽象，继承和动态绑定。通过数据抽象，可以将类的接口和实现分离；使用继承，可以定义相似的类型并对其相似关系建模；使用动态绑定，可以在一定程度上忽略相似类型的区别，而统一的使用它们的对象。  

继承(inheritance)  
通常在层次关系的根部有一个基类(base class),其他类直接或间接的从基类继承而来,这些继承来的类称为派生类(derived class).  

派生类必须通过使用**类派生列表(class derivation list)**,明确指出从哪个(些)基类继承而来.类派生列表: 冒号,然后是以逗号分隔的基类列表, 其中每个基类前面可以有访问说明符:  
```c++
class Quote{};
class Bulk_quote : public Quote{};
```

动态绑定(dynamic binding),能用同一段代码分别处理Quote和Bulk_quote的对象.  
```c++
double print_total(ostream &os, const Quote &item, size_t n){}
```
第二个参数const Quote &item,既可以用基类Quote的对象调用,也能用派生类Bulk_quote的对象调用  
在运行时选择函数的版本,所以动态绑定又被称为运行时绑定(run-time binding)  
在C++中,当使用基类的引用(或指针)调用一个虚函数时将发生动态绑定.  

15.2 定义基类和派生类  
首先完成Quote类的定义:  
**类Base中加了Virtual关键字的函数就是虚拟函数,在Base的派生Derived中就可以通过重写虚拟函数来实现对基类虚拟函数的<font color = red>覆盖</font>**  

```c++
class Quote
{
public:
	Quote() = default;  //使用合成构造函数
	Quote(const std::string &book, double sales_price): bookNo(book), price(sales_price){}
	std::string isbn() const {return bookNo;}
	// 返回给定数量的书籍的销售总额
	// 派生类负责改写并使用不同的折扣计算算法
	virtual double net_price(std::size_t n) const {return n*price;}
	virtual ~Quote() = default;  //对析构函数进行动态绑定.
	// 当一个类作为其他类的基类时，它的析构函数应该加上virtual
private: 
	std::string bookNo;
protected:
	double price = 0.0;  //代表普通状态下不打折.
};
```
重载与覆盖的不同  
- 重载的几个函数必须在同一个类中；  
- 覆盖的函数必须在有继承关系的不同的类中  
- 覆盖的几个函数必须要求函数名、参数、返回值都相同；
- 重载的函数必须函数名相同，参数不同。参数不同的目的就是为了在函数调用的时候编译器能够通过参数来判断程序是在调用的哪个函数。这也就很自然地解释了为什么函数不能通过返回值不同来重载，因为程序在调用函数时很有可能不关心返回值，编译器就无法从代码中看出程序在调用的是哪个函数了。
- 覆盖的函数前必须加关键字Virtual；
- 重载和Virtual没有任何瓜葛，加不加都不影响重载的运作。

**成员函数与继承**  
派生类可以继承其基类的成员,派生类有时需要对某些操作提供自己的新定义来**<font color=red>覆盖override</font>**从基类继承的旧定义.  

在C++中,基类必须将它的两种成员函数区分:一种是基类希望其派生类进行覆盖的函数;另一种是希望派生类不改变的函数.  
对于前者,基类将其定义为**虚函数(virtual)**.当使用指针或引用虚函数时,该调用将被动态绑定.根据引用或指针所绑定的对象不同,该调用可能执行基类的版本,也可能执行派生类的版本.  

基类通过在其成员函数的声明前加上关键字virtual使得该函数执行动态绑定.任何构造函数之外的非静态函数(有this指针的函数)都可以是虚函数.  

访问控制与继承(protected:)  
派生类可以继承基类定义的成员,但派生类的成员函数不一定有权访问从基类继承而来的成员: 派生类能访问公有成员,而不能访问私有成员.  
当希望派生类能访问该成员,同时禁止其他用户访问,就是受保护的protected访问运算符  

15.2.2 定义派生类  
派生类构造函数:  
```c++
Bulk_quote(const std::string & book, double p, std::size_t qty,double disc) : Quote(book,p), min_qty(qty), discount(disc){};
// 基类的成员用基类的构造函数初始化,派生类自己的自己初始化.
```

继承与静态成员  
如果基类定义了一个静态成员,则在整个继承体系中只存在该成员的唯一定义.不论从基类中派生出多少个派生类,对于每个静态成员来说都只存在唯一的实例:  
```c++
class Base
{
public:
	static void statmem();
};
class Derived:public Base
{
	void f (const Derived&);
};
// 静态成员遵循通用的访问控制规则,如果基类中的成员是private,派生类无权访问它.假设静态成员是可访问的,则可以通过基类使用它,也能通过派生类使用它.  
void Derived::f(const Derived& derived_obj)
{
	Base::statmem();  //Base定义了statmem
	Derived::statmem(); //正确,Derived继承statmem
	// 派生类的对象能访问基类的公开静态成员
	derived_obj.statmem();  //通过Derived对象访问
	statmem();  //通过this对象访问.
}
```
派生类的声明: 
```c++
class Bulk_quote:public Quote;  //错误,派生列表不能出现在这里
class Bulk_quote;  //正确:声明派生类的正确方式
```

被用作基类的类:必须是已经定义而非仅仅声明.  

一个类是基类,同时也能是一个派生类
```c++
class Base{/*...*/};
class D1:public Base {/**/};
class D2:public D1 {/**/};
// Base是D1的直接基类,同时是D2的间接基类.
```

防止继承的发生: 希望某一类不被其他类继承,可以在类后跟一个关键字final:  
```c++ class NoDerived final {/**/}; //NoDerived不能作为基类. ```

15.2.3 类型转换与继承  
**理解基类与派生类之间的类型转换是理解C++语言面向对象编程的关键所在.**  
(派生类向基类的自动类型转换只对指针或引用类型有效)  
(不存在从基类向派生类的隐式类型转换)  
```c++
Quote base;
Bulk_quote* bulkp = &base;  //指针//错误,不能将基类转换成派生类
Bulk_quote& bulkp = base;  //引用//错误,不能将基类转换成派生类
```

通常情况下,如果想把引用或指针绑定到一个对象上,则引用或指针的类型应与对象的类型一样,或对象的类型含有一个可接受的const类型转换规则.  

存在继承关系的类是一个重要的例外:可以将基类的指针或引用绑定到派生类对象上.例如:可以用Quote&指向一个Bulk_quote对象,也可以把一个Bulk_quote对象的地址赋给一个Quote*.  
(智能指针也支持派生类向基类的类型转换,这意味着可以将一个派生类对象的指针存储到一个基类的智能指针内.)

15.3 虚函数  
在C++中,当使用基类的引用或指针调用一个虚成员函数时会执行动态绑定.  
动态绑定只有当通过指针或引用调用虚函数时才会发生.  

virtual函数在派生类中覆盖时,需要在后面加上override  
```c++ void f1(int) const override;```
需要注意派生类覆盖的函数其参数等都是相同的,与重载不一样.  

回避虚函数的机制: 某些情况下,希望对虚函数的调用不进行动态绑定,而是强迫执行虚函数的某个特定版本.--使用作用域运算符可以实现:  \
```c++
double undiscounted = baseP->Quote::net_price(42);  //::作用域运算符强制执行.
```

**如果一个派生类虚函数需要调用它的基类版本,但没有::,则在运行时该调用将被解析为对派生类版本自身的调用,从而导致无限递归.**

15.4 抽象基类(含有纯虚函数的类)  
纯虚函数(pure virtual):通过在声明后面加上=0来说明一个虚函数为纯虚函数,且纯虚函数不需要定义,只有声明.  
```c++ double net_price(std::size_t) const = 0; ```

含有(或未经覆盖直接继承)纯虚函数的类是抽象基类(abstract base class).  
抽象基类**负责定义接口,而后续的其他类可以覆盖该接口**.但不能直接创建一个抽象基类的对象.只能定义其派生类的对象.  

关键概念:重构refactoring  
重构负责重新设计类的体系以便将操作和/或数据从一个类移动到另一个类中.对于OOP的应用程序来说,重构是一种很普遍的现象.  

15.5 访问控制和继承  
每个类分别控制自己的成员初始化过程,每个类还分别控制其成员对于派生类来说是否可访问(accessible)  

受保护的成员(protected:关键字):  
一个类使用protected关键字来声明与派生类分享但不被其他公共访问使用的成员.  
- 和私有成员类似,受保护的成员对于类的用户来说是不可访问的.
- 和公有成员类似,受保护的成员对派生类的成员和友元来说是可访问的
- (a)派生类的成员或友元只能通过派生类对象来访问基类的受保护成员.  
- (b)派生类对于一个基类对象中的受保护成员没有任何访问特权.

```c++ 理解a,b两句
class Base
{
protected:
	int prot_mem;  
};
class Sneakey: public Base
{
	friend void clobber(Sneaky&);  //(a通过派生类Sneaky来访问基类成员prot_mem)能访问Sneaky::prot_mem
	friend void clobber(Base&);  //(b派生类不能直接通过基类来访问基类的prot_mem)不能访问Base::prot_mem
	int j;  //j默认是private
};
void clobber(Sneaky &s) {s.j = s.prot_mem = 0;}  //a
void clobber(Base &b){b.prot_mem=0;}     //b
```

公有,私有和受保护继承  
某个类对其继承而来的成员的访问权限受到两个因素影响:1是在基类中该成员的访问说明符(public,private,protected),2是派生类的派生列表中的访问说明符(继承的那个public,private,protected).  
```c++
class Base
{
public: void pub_mem;
protected: int prot_mem;
private: priv_mem;
};
// 下面是一个派生列表中的访问说明符为public的派生类(或者叫结构?)
struct Pub_Derv : public Base //(自己命名:公有继承)
{
	int f(){return prot_mem;}   //正确,派生类能访问protected成员
	char g(){return priv_mem;}  //private成员不能被派生类访问
};
// 派生列表的访问说明符为private
struct Priv_Derv : private Base //(自己命名:私有继承)
{
	int f1() const {return prot_mem;}  //private不影响派生类的访问权限
};
// Pub_Derv和Priv_Derv都能访问prot_mem,同时都不能访问priv_mem
```
从上可知,因素2派生类的派生列表中的访问说明符对于派生类的成员(及友元)能否访问基类的成员没有影响.  
对基类成员的访问权限只与因素1基类的访问说明符有关.  
Pub_Derv和Priv_Derv只能访问prot_mem,不能访问priv_mem  

因素2(派生访问说明符)的目的是控制派生类从基类获得从public成员和protected成员作为自己的public成员还是private成员:  
```c++
Pub_Derv d1;  //继承自Base的成员是public的
Priv_Derv d1;  //继承自Base的成员是private的
Prot_Derv d3;  ///(自己命名:保护继承)
d1.pub_mem(); //正确  
d2.pub_mem(); //错误,pub在派生类中是private的
d3.pub_mem();  //错误,不能直接访问,pub在派生类中是protected
// pub在派生类中是protected,所以可以在Prot_Derv的成员函数中可以使用pub_mem,或在派生类中通过派生类的成员访问
```

**派生类向基类转换的可访问性**  

**派生类向基类转换:因为在派生类对象中含有基类,所以能把派生类的修行当成基类对象来使用,而且也能将基类的指针或引用绑定到派生类对象的基类部分上**  
```c++
// Quote是Bulk_quote的基类
Quote item;  //item是基类对象
Bulk_quote bulk;  //bulk是派生类对象
Quote *p = &item;  //基类指针p指向基类Quote的对象item
p = &bulk;  //p指向bulk的Quote部分
Quote &r = bulk;  //基类引用r绑定到bulk的Quote部分
```
这种基类的指针或引用绑定到派生类对象的基类部分上的转换行为,称为**派生类到基类derived-to-base类型转换**.和其他类型转换一样,编译器会隐式执行派生类到基类的转换.
\[之后重新编写15.2.2与15.2.3\]

派生类向基类的转换是否可访问由使用该转换的代码决定,同时派生类的派生访问说明符(公有继承public,私有/保护继承)也有影响:假定D继承自B  
- 只有D是公有继承B时,<font color=red><用户代码></font>才能派生类向基类的转换(因为此时基类成员在D中是公有的,用户可以访问)
- 无论D是什么继承,D的<font color=red><成员函数和友元></font>都能使用派生类向基类的转换:即派生类向其直接基类的类型转换对于派生类的成员和友元来说是可访问的.(因为此时,基类B的所有成员都由D继承而来,算作D的成员,而D自己的成员不论是公有私有还是保护,D的成员函数和友元函数都能访问)
- 如果D继承B的方式是公有继承或保护继承,则D的<font color=red>\<派生类的成员和友元></font>可以使用D向B的类型转换,如果D是私有继承,则不能使用(因为当D是私有时,从B继承来成员在D中是私有的,而D的派生类成员和友元本来就不能访问D的私有部分,公有保护也如此推理)

友元与继承(友元关系不能传递也不能继承)  
基类的友元在访问派生类成员时不具有特殊性,类似的,派生类的友元也不能随意访问基类的成员.  
```c++
class Base
{
	friend class Pal;  //Pal在访问Base的派生类时不具有特殊性
};
class Pal
{
public:
	int f(Base b){return b.prot_mem;}  //正确,Pal是Base的友元
	int f2(Sneaky s) {return s.j;}  //错误,Pal不是Sneaky的友元,所以无法访问s自己的成员j
	// 
	int f3(Sneaky s) {return s.prot_mem;} //正确,Pal是Base的友元,而prot_mem是Sneaky从Base中公有继承而来的,因此prot_mem可以被Sneaky的对象s访问,且这部分是基类Base的,所以在友元Pal可以访问基类Base的部分.综上两部分,可以在Pal中通过Sneaky的对象s访问基类Base的成员prot_mem.[可以弄个逻辑图(保护派生类,基类,公有继承,私有继承,用户,成员,成员函数可以访问的权限图)]
};
```
Pal是Base的友元,所以Pal能访问Base对象的成员,这种可访问性包括Base对象内嵌在其派生类对象的情况.  

友元关系不能继承,即不能通过Pal的友元来访问Base的部分.

改变个别成员的可访问性  
有时需要改变派生类继承的某个名字的访问级别,通过使用using声明可以达到该目的:
```c++
class Base
{
public:
	std::size_t size() const {return n;}
protected:
	std::size_t n;
};
class Derived: private Base
{
public:
	using Base::size;  //保持对象尺寸相关的成员的访问级别
// 因为是私有继承,size和n成Derived的私有成员,但使用using声明语句,将size变成公有成员的访问属性,下面同理
protected:
	using Base::n;
};
```

默认的继承保护级别  
默认情况下,使用class关键字定义的派生类是私有继承,而struct关键字定义的派生类是公有继承.(struct不叫结构也叫类?!)  
```c++
class Base{/**/};
struct D1: Base{/**/};  //默认public继承
class D2: Base{/**/};  //默认private继承
```
**stuct和class唯一的差别是默认成员访问符及默认派生访问说明符之间**

15.6 继承中的类作用域  
每个类定义自己的作用域,在这个作用域内定义类的成员.  
当存在继承关系时,**派生类的作用域嵌套在其基类的作用域之内**.  
如果一个名字在派生类的作用域内无法正确解析,则编译器将继续在外层的基类作用域中寻找该名字的定义.  

名字冲突与继承：派生类的成员将隐藏同名的基类成员.  
通过作用域运算符来使用隐藏的成员:  
```c++
struct Derived: Base
{
	int get_base_mem(){return Base::mem;}
};
```
作用域运算符将覆盖掉原有的查找规则,并指示编译器从Base类的作用域开始查找mem. 除了覆盖继承而来的虚函数,派生类最好不要重用其他定义在基类中的名字.

定义派生类中的函数不会重载其基类中的成员.和其他作用域一样,如果内层作用域(派生类)的成员与基类(外层作用域)的某个成员同名,则外层作用域的将暂时被隐藏.  
```c++
struct Base
{
	int memfcn();
};
struct Derived: Base
{
	int memfcn(int);  //隐藏基类的memfcn
};
Derived d; Base b;
b.memfcn();   //调用Base::memfcn
d.memfcn(10); //调用Derived::memfcn
d.memfcn();   //错误,参数列表为空的memfcn被隐藏了
d.Base::memfcn();  //正确,调用Base::memfcn
```

虚函数与作用域  
因此,基类和派生类的虚函数接受的实参要相同,不然就无法通过基类的引用或指针来调用派生类的虚函数:  
```c++
class Base
{
public:
	virtual int fcn();
};
class D1:public Base
{
public: 
	// 隐藏基类的fcn,这个fcn不是虚函数
	// D1继承了Base::fcn()的定义
	int fcn(int);        //形参列表与Base中的fcn不一样
	virtual void f2();   //是一个新的虚函数,Base中不存在
};
class D2:public D1
{
public:
	int fcn(int);  //非虚函数,隐藏D1::fcn(int)
	int fcn();     //覆盖了Base的虚函数fcn
	void f2();     //覆盖D1的虚函数f2();
};
```

通过基类调用隐藏的虚函数(即上面内外作用域嵌套的延伸例子)  
```c++
Base bobj;D1 d1obj; D2 d2obj;
Base *bp1 = &bobj; *bp2 = &d1obj, *bp3 = &d2obj;
bp1->fcn();   //虚调用,将在运行时调用Base::fcn
bp2->fcn();   //虚调用,将在运行时调用Base::fcn
bp3->fcn();   //虚调用,将在运行时调用D2::fcn

D1 *d1p = &d1obj; D2 *d2p = &d2obj;
bp2->f2();  //错误,Base没有f2成员
d1p->f2();  //虚调用,将在运行时调用D1::f2()
d2p->f2();  //虚调用,将在运行时调用D2::f2()

Base *p1 = &d2obj; D1 *p2 = &d2obj; D2 *p3 = &d2obj; //三个不同类型指针指向同一个派生类对象
p1->fcn(42);  //错误,Base没有接受int参数的fcn
p2->fcn(42);  //静态绑定,调用D1::fcn(int)
p3->fcn(42);  //静态绑定,调用D2::fcn(int)
// 需要注意的是,由于调用的是非虚函数,所以不会发生动态绑定,而是静态绑定(后两行)
```

15.7 构造函数与拷贝控制   
虚函数的作用:实现多态性,多态性是将接口与实现进行分离,如sum,对不同派生类的定义不同的sum,但都是调用sum成员函数来实现多态.  
15.7.1 虚析构函数(总结将基类的析构函数定义为虚析构函数即可)  
继承关系对基类拷贝控制最直接的影响是基类通常应该定义一个虚析构函数,这样就能动态分配继承体系中的对象.  

一个Quote*类型的指针,但指向派生类Bulk_quote类型的对象.当需要delete这个指针的时候,编译器需要清楚的是执行Bulk_quote的析构函数.和其他函数一样,通过在基类中将析构函数定义为虚函数,来实现多态,从而以确保执行正确的析构函数版本:  
```c++
class Quote
{
public:
	// 如果删除的是一个指向派生类对象的基类指针,则需要虚析构函数
	virtual ~Quote() = default;  //动态绑定析构函数
};
```
和其他虚函数一样,析构函数的虚属性也会被继承.因此,无论Quote的派生类使用合成的析构函数还是自己定义的析构函数,都将是虚析构函数. 只要基类的析构函数是虚函数,就能确保当我们delete基类指针时将运行正确的析构函数版本  
```c++
Quote *itemP = new Quote;  //静态类型与动态类型一致
delete itemP;   //调用Quote的析构函数
itemP = new Bulk_quote;  //静态类型与动态类型不一致
delte itemP;  //调用Bulk_quote的析构函数.
```

虚析构函数将阻止合成移动操作  
如果一个类定义了析构函数,编译器不会为这个类合成移动操作(13.6.2节)  

15.7.2 合成拷贝控制与继承  
派生类的合成拷贝控制成员的行为(即合成构造,合成拷贝函数,合成析构函数等),这些合成的成员还负责使用基类中对应的操作对基类部分进行初始化,拷贝赋值或销毁.(即派生类的合成拷贝控制用自己的成员函数控制自己的成员,对于从基类继承来的成员,用基类的成员函数进行控制)  

派生类中删除的拷贝控制与基类的关系  
- 如果基类中的默认构造函数,拷贝构造函数,拷贝赋值运算符或析构函数是被删除的,那么派生类中对应的函数也是被删除的.(原因是编译器不能使用基类成员来执行派生类对象基类部分的构造,赋值或销毁操作)  
- 如果基类中的析构函数被删除,则派生类中合成的默认和拷贝构造函数也是被删除的
- 编译器不会合成一个删除掉的移动操作.  

```c++
class B
{
public:
	B();  //可访问的默认构造函数
	B(const B&) = delete; //显式删除的拷贝构造函数
	// 其他成员,不含有移动构造函数
}; 
class D:public B
{
	// 没有声明构造函数
};
D d;   //正确,D的合成默认构造函数使用B的默认构造函数
D d2(d); //错误,D的合成构造函数是被删除的
D d3(std::move(d));  //错误,隐式地使用D的被删除的拷贝构造函数
```

移动操作与继承  
大多数基类都会定义一个虚析构函数,因此默认情况下,基类通常不含有合成的移动操作,而且在它的派生类中也没有合成的移动操作  
因为基类如果没有移动操作的话,会阻止派生类拥有自己的合成移动操作,所以当需要移动操作时,应首先在基类中定义.  
```c++
class Quote
{
public: 
	Quote() = default;  //对成员依次进行默认初始化
	Quote(const Quote&) = default;  //对成员依次进行拷贝
	Quote(Quote &&) = default;  //对成员依次拷贝
	Quote& operator=(const Quote&) = default;  //拷贝赋值
	Quote& operator=(Quote&&) = default; //移动赋值
	virtual ~Quote() = default;
}; //Quote的派生类也将自动合成移动操作
```

15.7.3 派生类的拷贝控制成员  
在15.2.2中说过,派生类构造函数在其初始化阶段不仅要初始化派生类自己的成员,也要初始化派生类对象的基类部分.派生类的拷贝和移动构造函数在拷贝和移动自有成员时,也要拷贝和移动派生类对象的基类部分的成员.(当派生类定义了拷贝或移动操作时,该操作负责拷贝或移动基类部分成员在内的整个对象)  

和构造函数及赋值运算符不同的是,析构函数只负责销毁派生类自己分配的资源. 如前所示,对象的成员是被隐式销毁的,派生类对象的基类部分也是自动销毁的.  

定义派生类的拷贝或移动构造函数  
在默认情况下,基类默认构造函数初始化派生类对象的基类部分.如果想拷贝或移动基类部分,则必须在派生类的构造函数初始值列表中显式地使用基类的拷贝(或移动)构造函数.  
```c++
class Base {/**/};
class D:public Base
{
public:
	D(const D&d): Base(d)  //显式地拷贝基类成员
	D(D&& d): Base(std::move(d))  //显式地移动基类成员
};
```

派生类赋值运算符  
与拷贝和移动构造函数一样,派生类的赋值运算符也必须显式地位其基类部分赋值:  
```c++
// Base::operator=(const Base&) 不会被自动调用
D &D::operator=(const D &rhs)
{
	Base::operator = (rhs);  //为基类部分赋值(显式的调用基类的赋值运算符)
	return *this;
}
```

派生类析构函数(只负责销毁由派生类自己分配的资源)  
```c++
class D: public Base
{
public:
	//Base::~Base被自动调用执行
	~D(){/**/}
}; //对象销毁的顺序:派生类析构函数先执行,然后是基类的析构函数
```

在构造函数和析构函数中调用虚函数(略)  

15.7.4 继承的构造函数  
在C++11新标准中,派生类能够重用其直接基类定义的构造函数.  
一个类只初始化它的直接基类, 一个类也只继承其直接基类的构造函数. 类不能继承默认,拷贝和移动构造函数,如果派生类没有定义,则不会继承,而是编译器为派生类合成.  

如果派生类要继承基类构造函数的话,则使用基类名的using声明语句  
```c++
class Bulk_quote: public Disc_quote
{
public:
	using Disc_quote::Disc_quote;  //继承Disc_quote的构造函数
	double net_price(std::size_t) const;
};
```

继承的构造函数的特点(略)  

15.8 容器与继承  
当使用容器存放继承体系中的对象时,通常必须采取间接存储的方式.因为不允许容器中保存不同类型的元素,所以不能把具有继承关系的多种类型的对象直接存放在容器中.  

当派生类对象被赋值给基类对象时,其中的派生类部分将被"切掉",因此容器和存在继承关系的类型无法兼容.  

在容器中放置(智能指针)而非对象  
当希望在容器中存放具有继承关系的对象时，实际上存放的通常是基类的指针(智能指针),这些指针所指对象的动态类型可能是基类类型,也可能是派生类类型: 
```c++
vector<shared_ptr<Quote>> basket;
basket.pushu_back(make_shared<Quote>("0-201-82470-1",50));
basket.push_back(make_shared<Bulk_quote>("0-201-54848-8",50,10,.25));

cout << basket.back()->net_price(15) << endl;
```
因为basket存放着shared_ptr,所以必须解引用basket.back()的返回值以获得net_price的对象.  
我们将basket定义为shared_ptr<Quote>,但是在第二个push_back传入的是一个Bulk_quote对象的shared_ptr.  
正如我们可以将一个派生类的普通指针转换成基类指针一样,也能把派生类的智能指针转换成基类的智能指针.  

15.8.1 编写Basket类(略)  
```c++
class Basket
{
public:
	// Basket使用合成的默认构造函数和拷贝控制成员
	void add_item(const std::shared_ptr<Quote> &sale){items.insert(sale);}
	// 打印每本书的总价和购物篮中所有书的总价
	double total_reveipt(std::ostream &) const;
private:
	// 该函数用于比较shared_ptr,multiset成员会用到它
	static bool compare(const std::shared_ptr<Quote> &lhs, const std::shared_ptr<Quote> &rhs){return lhs->isbn() < rhs->isbn();}
}
```

15.9 文本查询程序再探(实战例子,之后再看吧,好长阿)  

小结:  
重构(refactoring) 重新设计程序 以便将一些相关的部分搜集到一个单独的抽象中,然后使用新的抽象代替原理的代码.  
通常情况下,重构类的方法是将数据成员和函数成员移动到继承体系的高级别节点当中,从而避免代码冗余.  

第16章 模板与泛型编程  
- 16.1 定义模板  
- 16.2 模板实参推断  
- 16.3 重载与模板
- 16.4 可变参数模板
- 16.5 模板特例化

OOP和泛型编程都能处理在编写程序时不知道类型的情况,不同之处在于:OOP能处理类型在程序之前都未知的情况,而在泛型编程中,在编译时就能获知类型.  
本书第二部分介绍的容器,迭代器和算法都是泛型编程的例子.当编写一个泛型程序时,是独立于任何特定类型赖编写代码的.当使用一个泛型程序时,提供类型或值,程序实例可在其上运行.  

之前第三章和第二部分学习了如何使用模板,这里将学习如何定义模板  

16.1 定义模板  
16.1.1 函数模板(function template)  
```c++
template <typename T>  //模板定义以关键字template开始,后跟模板参数列表(template parameter list), 这是一个逗号分隔的一个或多个模板参数的列表,用<>包围起来
int compare(const T &v1, const T &v2)
{
	if(v1<v2>) return -1;
	if(v2<v1>) return 1;
	return 0;
}
```
当传入具体的实参时,编译器会生成对应类型的模板实例.  

