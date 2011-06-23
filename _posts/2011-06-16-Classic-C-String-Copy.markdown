---
layout: post
title: String Copy (in the key of C)
---
#String Copy (yes, in the key of C)
I'm not ashamed to admit that I'm a bit of a [Joel on Software](http://www.joelonsoftware.com/) fan (though his advice has now stopped blooming).  So, as an aging CS student, I harken back every-so-often to his [Advice for CS College Students](http://www.joelonsoftware.com/articles/CollegeAdvice.html).  One of the nuggets of advice he gives is to learn C.  

(A brief aside:  Joel also says that 'C is becoming increasingly rare' - this may have been true in 2005, but [not now](http://redmonk.com/sogrady/2011/04/04/changes-programming-languages/))

Within his C-advice, he writes 
>...if you can't explain why *while (\*s++ = \*t++);* copies a string, or if that isn't the most natural thing in the world to you, well, you're programming based on superstition...

So, I did what any burgeoning CS student would do:  I tried to put the code in action.

My first hack at this looked a little like the following.

    int main()
    {
    	char * word = "Sorrell";
    	char newWord[sizeof(word)];
    	while (*newWord++ = *word++ ) {};
    	printf("word:%s and newWord:%s\n", word, newWord);
    	return 1;
    }
    
My thinking was that if I just created 'newWord' as an uninitialized char *, and then tried to copy to that memory, I could be overwriting memory that might be in use for something else.  Creating an array seemed logical, because that string 'word' is just a character array, right?  Well, yes and no. I got this error from the compiler:

    stringcopy.c:9: error: lvalue required as increment operand
    
Huh?  