I’ve always been a big fan of the Python programming language. It’s great for prototyping, it has great libraries for data processing and machine learning and it’s overall just a joy to read and write with. But I also like types and if you’re at all familiar with the language then you know that variables are dynamically typed in Python and that type information is absent from the basic syntax.

Don’t get me wrong I love Python’s dynamically typed model, it’s what gives the language its distinct flexibility and fast prototyping capabilities. But it’s undeniable that for certain tasks and projects, types add great value. They give you type hinting and help in catching obvious and not so obvious bugs, some that may only manifest themselves under some specific corner cases.

## Typing as your new best friend

Now since Python 3.6 you can use a module called typing to static type your variables if you want to. The neat thing about typing is that you can choose to include type information at specific points in your code where you feel there is a need to do it. The syntax to declare a variable type goes like this:

In functions you can also declare the return type like this:

Typing provides you with container types like `Dict`, `List`, `Set` or `Tuple`.

I personnaly like this kind of syntax even if it might make the most hardcore python purist silently scream in cringe and agony. I feel it makes my code more readable and predictable but that might only be my Java part of the brain talking.

You can even define your own types by composing primitive types.

Or with classes.

If you want to declare a function that takes parameters of multiple types you can use `Any`.

To type check your newly written programs with typing your best choice is probably to use [mypy](https://github.com/python/mypy), a static type checker you can run before calling the python interpreter.

A cool thing you can do with this is build a new execution pipeline combining the static type checker and the Python interpreter in a single program executable. If there is a type error the program won’t run the interpreter and will output the type error messages, otherwise it will execute the python script.

Make this program executable and send in the python script you want to run as an argument.

And voila you now have a brand new python interpreter which runs a type checker prior to running the actual program.

That’s it! I hope you found this useful and might at least try it out of curiosity in some of your codebases. I think you’ll find there’s some merit to typing your variables in Python, even if it might slow you down in the short term, it might actually make you gain time in the longer run.

Bye for now!