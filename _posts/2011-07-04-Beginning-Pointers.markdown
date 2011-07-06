---
layout: post
title: Beginning Pointers
---
#Beginning Pointers
I remember how frustrating it was when our CS 102 class incorporated *double pointers*. I didn't even really understand what I was doing with a single pointer (and, looking back now, I'm not so sure [UC](http://www.uc.edu) really primed us for that leap).  Most of the class was hopelessly lost, and worse, the assignments built on top of each other.  So the guy who tried to use the 'string' class was really screwed when we started using those double pointers to point at single 'char' pointers (you know, string constants).  And, at some point last year, I was reading through a Qt tutorial about how to create a spreadsheet application when it hit me:  **this** is the way to teach pointers!  

[**Note:** While these examples are in C, you can easily translate them to C++ and swap the 'printf' statements with equivalent 'cout' statements.] 

While I realize that I'm the fifty-millionth person to attempt an explanation of this, I do so anyway, if not to waste your time, but my own.  So, let's get to the business of dissolving this subject.  First of all, let's get down to what a pointer *is*.  Simply put, a pointer is just another variable, but instead of having an integer value like '5' or a character value like 'S' or a string value like 'Sorrell', it stores the value of a memory location.  So, to help our compiler along, we tell it which type of variable our memory location is pointing to by giving our pointer a type (like 'int *', or 'char *').  

One question that was never, ever, raised or addressed was this:  *Why would we ever want to use pointers?*

I'm sure that if you look on the interwebs, you can find a million reasons.  But here are a few useful ones.

  - **Modularization**.  The whole point of programming in a language other than Assembly is to abstract and break apart your program into nice, neat pieces and functions.  And in object-oriented programming, we have nice big objects to work with.  Suppose we're modeling a "real world" object:  a house.  So we have this huge object sitting in our memory, and when we want to visit that house, we have options.  We can either create another gigantic replica of that house (but that costs time and space), or we can just look up the address.  Now suppose we're having a party at this house.  We don't want to keep looking up that address (remember [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself)?).  So, we store that address in our convenient address holder:  the pointer!
  - **Speed**.  So, now that we're not passing around that huge house (which costs time and space), we gain speed and save space, right?  Right!  We will now pass that address as a parameter in our function calls, instead of sending the whole house. (Note:  there are times when you will want to pass the whole house, and this is called passing by value, and it creates a local copy of that house.  We will not be exploring that in this post.)
  - **Pointer arithmetic**.  Once you get a handle on pointers, there may be times when you need to manipulate or use consecutive memory addresses (for instance, [String Copying](http://sorrell.github.com/2011/06/16/Classic-C-String-Copy.html)).   
  
  
##Cut To The Code
So, remember hours ago when I mentioned that spreadsheet application?  And how it pertained to learning pointers?  Well, here's what I was talking about.  Suppose we have the code below:
{% highlight c %}
int main()
{
    int i = 5;
    int * iptr = &i;    // that says the pointer 'iptr' is equal to the address of 'i'
    return 0;
}
{% endhighlight %}

We have an int-type variable (i) with a value of 5.  And we have an int-type pointer (iptr) with a value of address-of-i.  Now, to visualize this, let's open up our spreadsheet.  And let's become the computer and do the memory management ourselves (I know, you can barely contain your excitement).  So, let's assign cell A1 to the variable 'i', and put a value in that cell.  
![A1](/images/0711-Pointers/1.png)

Now, see that 'fx' area that is highlighted?  That tells us the **actual value** of that cell.  And in this case, it's 5.  Let's now assign cell A2 to the variable 'iptr', and put a value in that cell.   
![A2](/images/0711-Pointers/2.png)

See the value that we've put into A2 (up by that little 'fx' area)?  We've given it the value '=A1'.  So cell A2 effectively points to A1.  Now, when we change A1, A2 *appears* to change with it!  Another way of saying this is that A2 *references* A1 - and when we *dereference* A2, we get the value of A1.  Our spreadsheet automatically dereferences for us and puts that value in the cell (even though 'fx' value never changes).

So, with all this talk of dereferencing, what does that look like in the code?
{% highlight c++ %}
int main()
{
    int i = 5;
    int * iptr = &i;
    *iptr = 42;     // this is the 'dereferencing' we're talking about
    printf("i's value is %i \n", i);       // this prints 'i's value is 42'
    i = 5;
    printf("i's value is %i \n", *iptr);       // this prints 'i's value is 5'
    return 0;
}
{% endhighlight %}

As you can see, the '*' character dereferences.  So let's recap:

- 'i' is stored in cell A1.  So when we call 'i', we are really just asking for the value at A1.  So even though 'i' isn't a pointer, per se, it points to a location in memory too (A1).
- 'iptr' is stored in cell A2.  So when we call 'iptr', we are really just asking for the value at A2, which happens to be a memory address.
- '\*iptr' is the dereferencing of cell A2. So when we call '\*iptr', we are really saying "Give me the value at the address iptr is pointing to," or in other words, "Give me the value of cell A1."
- At this point, you're probably thinking that calling '\*iptr' is equivalent to calling 'i'.  And you're right!  See in the above code how we set '\*iptr = 42' and then printed the value of 'i'?

See?  Pointers are that easy!  Here's a couple of parting tips before the next blog post.

- **Read from right to left for lvalues.**  lvalues are the stuff on the **left** side of the '=' sign.  So, when you see:
    - int * iptr = &i;
    - lvalue should be read from right to left... for instance "iptr **is a** pointer **to a** int type"
    - const int const * iptr = &i;
    - This should be read:  "iptr **is a** pointer (**which is** constant) **to a** int type (**which is** constant)"
- **Read from left to right for rvalues.** This should be obvious by now, but at any rate, take the following example.
    - int * iptr = &i;
    - rvalue should be read from left to right... for instance "the **address of** i"

In my next post, we'll talk about double pointers and arrays.