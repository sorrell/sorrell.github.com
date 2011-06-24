---
layout: post
title: String Copy (in the key of C)
---
#String Copy (yes, in the key of C)
I'm not ashamed to admit that I'm a bit of a [Joel on Software](http://www.joelonsoftware.com/) fan (though his advice has now stopped blooming).  So, as an aging CS student, I harken back every-so-often to his [Advice for CS College Students](http://www.joelonsoftware.com/articles/CollegeAdvice.html).  One of the nuggets of advice he gives is to learn C.  

*(A brief aside:  Joel also says that 'C is becoming increasingly rare' - this may have been true in 2005, but [not now](http://redmonk.com/sogrady/2011/04/04/changes-programming-languages/))*

Within his C-advice, he writes 
>...if you can't explain why *while (\*s++ = \*t++);* copies a string, or if that isn't the most natural thing in the world to you, well, you're programming based on superstition...

I understood that the loop would terminate once the null character (\0) was hit.  But that understanding felt like *reading* the math book instead of doing the problems.  So, I did what any burgeoning CS student would do:  I tried to put the code in action.

My first hack at this looked a little (lot) like the following.

    int main()
    {
    	char * word = "Sorrell";
    	char newWord[sizeof(word)];
    	while (*newWord++ = *word++ ) {};
    	printf("word:%s and newWord:%s\n", word, newWord);
    	return 0;
    }
    
My thinking was that if I just created 'newWord' as an uninitialized char *, and then tried to copy to that memory, I could be overwriting memory that might be in use for something else.  Creating an array seemed logical, because that string 'word' is just a character array, right?  Well, yes and no. I got this error from the compiler:

    stringcopy.c:9: error: lvalue required as increment operand
    
Huh?  So much for that simple test, now I've just opened a can of worms.  Maybe I should just abstract it a little more, I thought.

    void copy_string(char *s, char *t)
    {
    	while (*s++ = *t++) {};
    }

    int main()
    {
    	char * word = "Sorrell";
    	char newWord[sizeof(word)];
    	copy_string(newWord, word);
    	printf("word:%s and newWord:%s\n", word, newWord);
    	return 0;
    }
    
This worked! And, it left me scratching my head... why did this work?  Well, take a look at that 'copy_string' function.  Even though 's' and 't' are taking the address of 'newWord' and 'word' respectively, they are, in fact, new **variables**.  This is an important point.  With our two new char * variables(s and t), both pointing to the same memory locations as 'newWord' and 'word' respectively, we can successfully iterate over (and copy the contents between) memory locations.  But, that still doesn't answer the question as to **why** we can't use that first implementation and iterate over 'newWord'.  Luckily, [K&R](http://en.wikipedia.org/wiki/The_C_Programming_Language) have the answer for us in 5.3 - Pointers and Arrays:

>There is one difference between an array name and a pointer that must be kept in mind. A pointer is a variable, so pa=a and pa++ are legal. But an array name is not a variable; constructions like a=pa and a++ are illegal.

*(And, really, that is a chapter worth reading in its entirety.)* 

You can see the iterations [in this file](https://github.com/sorrell/Miscellaneous/blob/master/C/stringcopy.c).  One more thing I learned from this little project is to **reset the pointer** before you try to print out your strings.  Iterating with that pointer has changed where it's pointing (duh), so if you don't reset the pointer, printing will be futile.  Here's what the final implementation looked like:

    int main()
    {
    	char * word = "Sorrell";
    	char * newWord = (char*) malloc (sizeof(word));
    	while (*newWord++ = *word++) {};
    	word = &word[-sizeof(word)];		//reset the pointer back to the beginning mem location
    	newWord = &newWord[-sizeof(newWord)];	// same as above
    	printf("word:%s and newWord:%s\n", word, newWord);
    }
    
Voila.  I now understand much more about C, pointers, and arrays, and it all started with one, seemingly innocuous, line of code.  As always.
 
 