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

My spider sense is tingling. Treating value as bool is common in C++, but in most cases it is a technicality not modeling anything from real world.
And such technical ( read keystroke saving ) casts tend to backfire a lot. The decrement operation is even more wierd - I would expect some the following
identity to hold:
'''(a--)++ == a'''
and there is no way to define operator ++ to satify this property. So both operations are purely technical and do not model any special kind of vector.

Lets move on to *countdown_to_zero*. To *countdown_to_zero* from *t*, initialize vector *result* with *t* then while *t* is true push *t* into result
and decrement *t*. The value of *countdown_to_zero* will be the size of result.

My spider sense is tingling again. I do not know what type should *t* be. Ok, I konw that ints are implicitly convertible to bool, but this really is
a technicality. In mathematics there is really no reason that some number should be true or false. If the intention was to test if *t* is not zero,
I would prefer
'''for (; t !=0; t--)'''

Now even the small change from condition '''t''' to condition '''t != 0''' makes a difference. Now there is no need to have ocnversion from *Inheritance* to
bool. Not having implicit conversion from *Inheritance* to bool will break the chain of implicit conversions from *Inheritance* to int and the line
'''result.push_back(t)''' will not compile anymore.

This gives me the feeling that it is not template argument deduction that has bitten us. It is the fact that we abused C++ syntax and wrote purely technical
code that does not model any real interaction too closely. Would we choose more descriptive naming the problem would be apparent.
##The case for deriving from std::array
##Long term maintainability
If you want your code to stay long, you may want it to play nicer with the C++ standard to minimize the possibility that your code will break
 with new version of C++. There is a lecture by Titus Winters covering it in more detail.
<https://www.youtube.com/watch?v=tISy7EJQPzI>
##So?
-- Always read your code aloud.
-- Do not inherit from standard containers if you want your code to last.
-- If you are a library author, use uniform initialization.
-- Before introducing new rule into coding guidelines, check if your problem cannod be avoided using existing rules.
-- Introduce rules banning language features only as last resort.
