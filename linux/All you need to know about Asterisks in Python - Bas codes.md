  ![An Architectural Asterisk](https://bas.codes/static/10560a4ccc3731e089ac2826c5cb0542/6a068/1440-victor-martin-G_LwkPWkYcg-unsplash.jpg "An Architectural Asterisk")

Most developers know the asterisk (`*`) character as multiplication operator in Python:

However, the asterisk has some special meaning for list or dictionary data structures.

## [](https://bas.codes/posts/python-asterisks#args-and-kwargs)`*args` and `**kwargs`

### [](https://bas.codes/posts/python-asterisks#args-in-the-function-definition)`*args` in the function definition

The most widely known usage of the asterisk operator, also known as destructuring operator, in Python is its usage in functions.

Let’s say you have a function that can add values and returns the sum of these values:

```
def add(number_1, number_2):
    return number_1 + number_2

print(add(1,2)) # 3
```

What if you want to sum up an arbitrary number of values? You can add an asterisk in front of the argument’s name like so:

```
def add(*numbers):
    sum = 0
    for number in numbers:
        sum += number
    return sum
```

As you might have already noticed from the function’s body, we expect `numbers` to be an iterable. Indeed, if you check the type, `numbers` is a `tuple`. And, indeed, we can now call the function with an arbitrary number of arguments:

### [](https://bas.codes/posts/python-asterisks#-in-the-function-call)`*` in the function call

We have seen so far that we can define a function so that it takes a list of parameters. But what if we have a function with a fixed set of arguments and we want to pass a list of values to it. Consider this function:

```
def add(number_1, number_2, number_3):
    return number_1 + number_2 + number_3
```

This function takes _exactly_ 3 parameters. Let’s assume we have a list with exactly three elements. Of course, we could call our function like this:

```
my_list = [1, 2, 3]

add(my_list[0], my_list[1], my_list[2])
```

Luckily the destructuring operator (`*`) works in both ways. We have already seen the usage in a function’s definition, but we can use it also to call a function:

```
my_list = [1, 2, 3]

add(*my_list)
```

### [](https://bas.codes/posts/python-asterisks#kwargs-in-function-definition)`**kwargs` in function definition

In the same way we can destructure lists with the asterisk (`*`) operator, we can use the double-asterisk (`**`) operator to destructure dictionaries in Python functions.

Consider a function like this:

```
def change_user_details(username, email, phone, date_of_birth, street_address):
    user = get_user(username)
    user.email = email
    user.phone = phone
    ...
```

If we call it with keyword arguments (`kwargs`), a function call might look like this:

```
change_user_details('bascodes', email='blog@bascodes.example.com', phone='...', ...)
```

In that case we could rewrite our function like this to accept an arbitrary number of keyword arguments that are then represented as a dictionary called `kwargs`:

```
def change_user_details(username, **kwargs):
    user = get_user(username)
    user.email = kwargs['email']
    user.phone = kwargs['phone']
    ...
```

Of course, we can use the `kwargs` dictionary like any other dictionary in Python, so that the function might become a bit cleaner if we actually utilize the dictionary data structure like so:

```
def change_user_details(username, **kwargs):
    user = get_user(username)
    for attribute, value in kwargs.items():
        setattr(user, attribute, value)
    ...
```

### [](https://bas.codes/posts/python-asterisks#-in-the-function-call-1)`**` in the function call

Of course, the `**` operator works for calling a function as well:

```
details = {
    'email': 'blog@bascodes.example.com',
    ...
}
change_user_detail('bascodes', **details)
```

## [](https://bas.codes/posts/python-asterisks#restricting-how-functions-are-called)Restricting how functions are called

### [](https://bas.codes/posts/python-asterisks#keyword-arguments-only)Keyword arguments only

One of the most surprising features of the asterisk in function definitions is that it can be used standalone, i.e. without a variable (parameter) name. That being said, this is a perfectly valid function definition in Python:

```
def my_function(*, keyword_arg_1):
    ...
```

But what does the standalone asterisk do in that case? The asterisk catches all (non-keyword) arguments in a list as we have seen above. There is no variable name in our case that would make the list. After the `*` we have a variable called `keyword_arg_1`. Since the `*` has already matched all positional arguments, we’re left with `keyword_arg_1`, which must be used as a keyword argument.

Calling the above function with `my_function(1)` will raise an error:

```
TypeError: my_function() takes 0 positional arguments but 1 was given
```

### [](https://bas.codes/posts/python-asterisks#positional-arguments-only)Positional arguments only

What if we want to force the users of our function to use _positional_ arguments only – as opposed to _keyword_ arguments only in the last example?

Well, there is a very Pythonic way. We interpret the `/` sign (the opposite of multiplication) to do the trick:

```
def only_positional_arguments(arg1, arg2, /):
    ...
```

Surprisingly few Python developers know about this trick which has been introduced to Python 3.8 through [PEP 570](https://peps.python.org/pep-0570/).

If you call the last function with `only_positional_arguments(arg1=1, arg2=2)`, this will raise a TypeError:

```
TypeError: only_positional_arguments() got some positional-only arguments passed as keyword arguments: 'arg1, arg2'
```

## [](https://bas.codes/posts/python-asterisks#usage-of--and--in-literals)Usage of `*` and `**` in literals

The asterisk (`*`) and double asterisk (`**`) operators do not only work for function definitions and calls, but they can be used to construct lists and dictionaries.

### [](https://bas.codes/posts/python-asterisks#constructing-lists)Constructing lists

Let’s say, we have two lists and want to merge them.

```
my_list_1 = [1, 2, 3]
my_list_2 = [10, 20, 30]
```

Of course, we could merge these lists with the `+` operator:

```
merged_list = my_list_1 + my_list_2
```

However, the `*` operator gives us a bit more flexibility. Let’s say, we want to include a scalar value in the middle, we could use:

```
some_value = 42
merged_list = [*my_list_1, some_value, *my_list_2]

# [1, 2, 3, 42, 10, 20, 30]
```

### [](https://bas.codes/posts/python-asterisks#constructing-dictionaries)Constructing dictionaries

Again, what is true for lists and the asterisk (`*`) operator is true for dictionaries and the double-asterisk operator (`**`).

```
social_media_details = {
    'twitter': 'bascodes'
}

contact_details = {
    'email': 'blog@bascodes.example.com'
}

user_dict = {'username': 'bas', **social_media_details, **contact_details}
# {'email': 'blog@bascodes.example.com', 'twitter': 'bascodes', 'username': 'bas'}
```

## [](https://bas.codes/posts/python-asterisks#destructuring-lists)Destructuring lists

You might already know, that you can split elements of a list to multiple variables like so:

```
my_list = [1, 2, 3]
a, b, c = my_list

# a -> 1
# b -> 2
# c -> 3
```

But did you know that you can use the asterisks (`*`) to assign to variables when you have a list of arbitrary length?

Say, you want the first element and the last one of a list in a specific variable.

You can do this by using the asterisk like so:

```
my_list = [1, 2, 3]
a, *b, c = my_list
```

In this example `a` now contains the first element, and `c` contains the last element of `my_list`. What is in `b`, however?

In `b`, there is the entire list, excluding the first and last element, i.e. `[2]`. Note that `b` is a list now.

This idea might get a bit clearer, when we look at larger lists:

```
my_list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
a, *b, c = my_list

# a -> 1
# b -> [2, 3, 4, 5, 6, 7, 8, 9]
# c -> 10
```