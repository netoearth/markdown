I'll be honest, the first time I saw a regular expression, it was a scary experience. **It looks like a weird alien language!** I thought to myself: "I've spent months learning programming and now i gotta learn this seemly super complex language!?"

However, once I sat down to actually learn regex, I discovered it's not super hard, once you learn the syntax.

### [Permalink](https://indepthjavascript.dev/how-to-match-a-phone-number-with-regex-and-javascript#heading-why-should-i-even-bother-learning-regex "Permalink")Why should I even bother Learning Regex?

As you start coding more, and more, it really comes in handy in all types of situations, and not just to valid phone numbers, and email addresses. It's very helpful when extract data from logs, messy JSON data from API calls, and many other situations.

I'm going to teach you how to valid a phone number _with 1 line of code_, with 1 regular expression. **Validating a phone number WITHOUT regex becomes an obnoxious leetcode question.** 😧

### [Permalink](https://indepthjavascript.dev/how-to-match-a-phone-number-with-regex-and-javascript#heading-why-is-validating-a-phone-number-so-complex "Permalink")Why is validating a phone number so complex?

Let's say you have a form on your website to collect a phone number to spa-, I mean, SMS your subscribers, there are a bunch of different ways they could submit their phone numbers.

All of these are VALID US based numbers:

-   `202-515-5555`
-   `202 515 5555`
-   `(202)515 5555`
-   `1 202 515 5555`
-   `2025155555`
-   `1-202-515-5555`
-   `1202-515-5555`
-   `etc`

There are more valid combinations I didn't list, but you get the idea! Validating every combo because a nasty coding problem. _But NOT if you're using regex to validate it!_ 😉

### [Permalink](https://indepthjavascript.dev/how-to-match-a-phone-number-with-regex-and-javascript#heading-lets-start-with-the-simplest-case "Permalink")Let's start with the Simplest Case

Let's first validate `202-515-5555`. The basic idea of regex is to build a pattern to match a string. What is the pattern in `202-515-5555` ?

We start with 3 digits followed by a `-` followed by 3 more digits, followed by another `-`, then we end with 4 digits.

Here's what the regex pattern to match `202-515-5555` looks like:

Let's watch through this...

The `^` just indicates the beginning of the string. In the above regex, we're stating that the phone number must start with `\d{3}` since we have `^` in front of `\d{3}`.

Now **`\d` => stands for single digit**, and **`{3}` simple means `\d` repeats exactly 3 times.** **So `^\d{3}` means our phone number STARTS with 3 digits.**

Now let's just jump to the end: `$` indicates the end of the string match. **`\d{4}$` means our phone number must END with 4 digits**. _Does this make sense?_

**`-` just means the phone number must have a dash at that spot.**

So let's read the entire regex from left to right now:

1.  `^\d{3}` => start with 3 digits
2.  `-` => followed by a dash
3.  `\d{3}` => followed with 3 digits
4.  `-` => followed by a dash
5.  `\d{4}$` => ends with 4 digits

Please read the above section a few times, if necessary, to make sure you fully understand before we move on.

### [Permalink](https://indepthjavascript.dev/how-to-match-a-phone-number-with-regex-and-javascript#heading-how-do-i-match-the-phone-number-if-dashes-are-options "Permalink")How do I match the phone number if dashes are options?

Great question! How do we match BOTH: `202-515-5555` and `2025155555`

To make a character match optional, just add a `?` after it.

Here's what our new improved match looks like:

**`-?` simply means the `-` is optional**: it may, or may not, be there!

Let's read the entire regex again:

1.  `^\d{3}` => start with 3 digits
2.  `-?` => _optionally_ followed by a dash
3.  `\d{3}` => followed with 3 digits
4.  `-?` => _optionally_ followed by a dash
5.  `\d{4}$` => ends with 4 digits

### [Permalink](https://indepthjavascript.dev/how-to-match-a-phone-number-with-regex-and-javascript#heading-how-about-matching-the-phone-number-if-there-are-spaces-instead-of-dashes "Permalink")How about matching the phone number if there are spaces, instead of dashes?

Now let's work on matching: `202-515-5555`, `2025155555`, AND `202 515 5555`

Instead of optionally just having a `-` we can optionally have either `-` OR `` `. How do we represent this? Easy, put both ``\-`and` `inside of`\[...\]`like this:`\[ -\]\`.

Our new regex looks like this:

```
^\d{3}[ -]?\d{3}[ -]?\d{4}$
```

Now it's definitely starting to look alien! 😅

Let's break it down:

1.  `^\d{3}` => start with 3 digits
2.  `[ -]?` => _optionally_ followed by a _space OR dash_
3.  `\d{3}` => followed with 3 digits
4.  `[ -]?` => _optionally_ followed by a _space OR dash_
5.  `\d{4}$` => ends with 4 digits

### [Permalink](https://indepthjavascript.dev/how-to-match-a-phone-number-with-regex-and-javascript#heading-how-to-match-1-or-1-or-1-at-the-beginning-of-our-phone-number "Permalink")How to Match `1` or `1` or `1-` at the beginning of our phone number

Based on what we've learned, can you figure this out?

It's a bit tricky once you realize that the `1...` in the beginning is optional.

Let's take it step by step...

If you want the phone number to start with `1` _we add `^1` to the beginning of the string match_, correct? Now we want to optionally add a dash or space after the 1. Luckily, we already know how to do that: \`\[ -\]?'.

**Combining the 2 we get: `^1[ -]?`**

Adding this to our previous regex we get:

```
^1[ -]?\d{3}[ -]?\d{3}[ -]?\d{4}$
```

Can you sense there's something wrong here? **The above regex string match MUST start with `1`, it's not optional.** We need to make `1[ -]?` optional.

_How do we do that?_ Since we're talking about multiple elements here: `1` and `[ -]?` we need to place the whole thing in `(...)` creating a group. Then add a `?` after it to make the entire group optional!

I know it's a lot to take in! Here's what it looks like:

```
^(1[ -]?)?\d{3}[ -]?\d{3}[ -]?\d{4}$
```

Let's break it down again, with an extra step now:

1.  `^(1[ -]?)?` => _optionally_ start with 1, which is _optionally_ followed by a _space OR dash_
2.  `\d{3}` => start with 3 digits
3.  `[ -]?` => _optionally_ followed by a _space OR dash_
4.  `\d{3}` => followed with 3 digits
5.  `[ -]?` => _optionally_ followed by a _space OR dash_
6.  `\d{4}$` => ends with 4 digits

🤯

If you're still reading, congrats, you now know how to _think_ 'regex'!

![Picard Regex](https://i.imgflip.com/6mzwct.jpg)

**There's still 1 outstanding issue:** how to match a phone number like: `(202)515 5555`. I'm going to leave that one for the reader (_hint:_ use the [pipe operator: `(...|...)`)](https://regexone.com/lesson/conditionals).

### [Permalink](https://indepthjavascript.dev/how-to-match-a-phone-number-with-regex-and-javascript#heading-putting-it-all-together-to-test-an-actual-phone-number-string "Permalink")Putting it all Together to Test an Actual Phone Number String

Now let's take our regex and turn it into a regular expression in javaScript. To do that you just add `/.../` around it. Then use a method called `test`:

```
const regex = /^(1[ -]?)?\d{3}[ -]?\d{3}[ -]?\d{4}$/;
const phoneNumber = '1202-515-5555';

// test returns 'true' if there's a match and 'false' if there is not
const match = regex.test(phoneNumber);
```

If you want to learn regex properly, here's a EXCELLENT free tutorial: [RegexOne](https://regexone.com/). This is literally how I learned Regex. It's worth going through ALL the exercises.