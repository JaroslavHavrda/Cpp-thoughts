#Do not inherit from standard library?
##The case against inheriting from standard library clases
This post is inspired by the following blogpost:
<https://quuxplusone.github.io/blog/2018/12/11/dont-inherit-from-std-types/>
The important code is:
'''C++
struct Inheritance : std::vector<int>
{
   Inheritance(std::initializer_list<int> il):vector(il){}
   operator bool(() const {return !this->is_empty();}
   void operator--() {this->pop_back();}
}

template<class T> int countdown_to_zero(T t)
{
   std::vector results = {t};
   for (; t; --t)
      results.push_back(t);
   return results.size();
}

int main()
{
   Inheritance y = {1,2,3};
   std::cout << countdown_to_zero(y) << "\n";
}
'''
The code outputs 6, which may be surprising result.
##Is a / Has a rule
At this point I will like review the is a / has a rule.
<https://en.wikipedia.org/wiki/Has-a>
closely related to Liskov substitution principle
<https://en.wikipedia.org/wiki/Liskov_substitution_principle>.
You do not need any deep understanding of these principles. All I need to know for now is: inheritance reads aloud *is a*
 and composition reads aloud *has a*. Even this simple corolary will help us understand the code better.
##Reading the code aloud
Now we can read the code aloud.

*Inheritance* is a vector of integers. The value of *Inheritance* can be treated as bool in a way that empty vector is false and nonempty vector is true.
The value of *Inheritance* can be decreased by poping the last element.

my spider sense is tingling. This is the first time I have seen decrementing a vector. This is not the first time I have seen sam comprex object treated
as bool and every time I have seen it, it backfired spectacularly.  
##The case for deriving from std::array
##Long term maintainability
##So?
