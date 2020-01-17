# CPP17-Fixes
New auto rules for direct-list-initialisation, static_assert With no Message, Different begin and end Types in Range-Based For Loop

New auto rules for direct-list-initialisation

Since C++11 there’s been a strange problem where:
auto x { 1 };
Is deduced as std::initializer_list<int>. Such behaviour is not intuitive as in most
cases you should expect it to work like int x { 1 };.
Brace initialisation is the preferred pattern in modern C++, but such exceptions make the
feature weaker.
With the new Standard, we can fix this so that it will deduce int.
To make this happen, we need to understand two ways of initialisation - copy and direct:
// foo() is a function that returns some Type by value
auto x = foo(); // copy-initialisation
auto x{foo()}; // direct-initialisation, initializes an
// initializer_list (until C++17)
int x = foo(); // copy-initialisation
int x{foo()}; // direct-initialisation
For the direct initialisation, C++17 introduces new rules:
• For a braced-init-list with a single element, auto deduction will deduce from that entry.
• For a braced-init-list with more than one element, auto deduction will be ill-formed.

For example:
auto x1 = { 1, 2 }; // decltype(x1) is std::initializer_list<int>
auto x2 = { 1, 2.0 }; // error: cannot deduce element type
auto x3{ 1, 2 }; // error: not a single element
auto x4 = { 3 }; // decltype(x4) is std::initializer_list<int>
auto x5{ 3 }; // decltype(x5) is int

static_assert With no Message

This feature adds a new overload for static_assert. It enables you to have the condition
inside static_assert without passing the message.
It will be compatible with other asserts like BOOST_STATIC_ASSERT. Programmers with
boost experience will now have no trouble switching to C++17 static_assert.
static_assert(std::is_arithmetic_v<T>, "T must be arithmetic");
static_assert(std::is_arithmetic_v<T>); // no message needed since C++17
In many cases, the condition you check is expressive enough and doesn’t need to be
mentioned in the message string.

Different begin and end Types in Range-Based For Loop

C++11 added range-based for loops:
for (for-range-declaration : for-range-initializer)
statement;
According to the C++14 standard that loop is equivalent to the following code:
auto && __range = for-range-initializer;
for ( auto __begin = begin-expr, __end = end-expr;
__begin != __end;
++__begin ) {
for-range-declaration = *__begin;
statement;
}
As you can see, __begin and __end have the same type. This works nicely but is not
scalable enough. For example, you might want to iterate until some sentinel value with a
different type than the start of the range.
In C++17 range-based for loops are defined as equivalent to the following code:
auto && __range = for-range-initializer;
auto __begin = begin-expr;
auto __end = end-expr;
for ( ; __begin != __end; ++__begin ) {
for-range-declaration = *__begin;
statement;
}
The types of __begin and __end might be different; only the comparison operator is
required. That change has no effect on existing for loops but it provides more options for
libraries. For example, this little change allows Range TS (and Ranges in C++20) to work
with the range-based for loop.

