So, as part of the RFC I proposed that was declined back in January, I gave an alternative syntax for operator overload magic methods. This had a mixed reception with voters in general. Nikic and a few other voters voted no or abstained specifically because they didn't like that syntax/didn't like adding new syntax.

In this post, I want to give a brief overview of how magic methods work inside the engine, and then ask about the wider community's opinions

## (How) Do Magic Methods Work?

The PHP engine is given a file to interpret and it encounters a class definition. Internally in the engine, these are stored as what are call "class entries" or "ce" in several of the functions/variables. These are structs (sort of like a defined type in C that has an array of values with pre-defined keys) that contain basically all the information about the class definition. Need to know its parent class? It's in the ce. Need to check if it's abstract? It's in the ce. Need to see if it has a certain method defined? It's in the ce.

Now, as part of interpreting the class definition it has to do a *lot* of things, as you can imagine. Everything that a class *could* define within itself has at least one step within this process. One of the earliest thing it does is collect all the functions that are defined on the class.

*After* it has collected all the functions defined on the class, it goes through a special process that checks for magic methods. It does this by iterating over all the functions defined on the class, and checking if the name of the function matches any of the magic methods available in PHP. 

If it finds a match, it grabs that function's body and creates a special place on the ce to store it so that it's more easily accessible in the rest of the engine. It also checks if the function passes any special rules that magic method has, such as type restrictions, parameters, etc.

Once the magic method is stored in its special location on the ce, it can be directly called within the engine by other parts of the language if the need arises. Importantly, this way of organizing the magic methods makes them:

1. Cheap to check for on any class.
2. Easy to check for without using multiple macros/calls to drill deep into the ce.
3. Better to work with when it comes to inheritance structures.
4. A bit more friendly to call directly from the engine instead of handling the way you would a user function call (which has some additional overhead).

## But What *Are* Magic Methods?

This is something that isn't really covered in a collective, comprehensive way in the documentation, in internals, in RFCs, or anywhere really. Magic methods are... magic methods. Methods that perform some kind of magic.

I would argue that this naming and lack of overall explanation really hurts people's understanding and usage of the contained features however, so let me tell you my own interpretation of what magic methods *are* having worked within PHP for nearly 20 years, and having spent the last year or so tinkering and working with the ce in the engine:

"Magic methods" are "engine hooks". They are an API to allow PHP developers to *modify* behavior within the engine at runtime for a defined set of features. 

If you want to access a property on a class, the property *must* be defined... *or* you can "hook" into the property part of the engine using the `__get()` method and interrupt the normal process with custom behavior. If you want to call a method on a class, the function *must* be defined with the name and signature that you are calling it with... *or* you can "hook" into the method call part of the engine using the `__call()` method.

## Magic Is Bad!

I really hate the term "magic methods". It implies something mysterious that just isn't the case. If these were instead called "object engine API hooks" or something similar, I think that people would feel a lot less trepidation about using them correctly, and would also be in the correct mindset when using them.

Magic is bad, because magic is inscrutable. You're never quite sure how magic works. But "object engine API hooks"? Well, *those* are specific APIs to provide custom behavior for a defined set of classes at extremely specific points within the engine.

## Syntax Matters

To me, and I might be alone in this (it's certainly not a view that was shared by all voters I found out), a big problem with "magic methods" is that they look like any other method definition. You use the function keyword within the class, you provide parameters to the function, and you provide a function body that details the behavior.

But these *aren't* used like functions most of the time. *Functions* happen when they are called, but *magic methods* happen *in the middle of* an internal process within the engine. Developers, I feel, get a false sense of understanding simply by using the same syntax as normal methods. It primes developer's minds to think in the mode of "writing a method".

Because of this, I pretty strongly believe that magic methods deserve their own, separate syntax to visually and mentally separate these features for developers. I also think there should be some overall guiding philosophies of what magic methods are for within the documentation to guide both PHP developers and people developing RFCs.

If I see `function __get($var) {}`, my mind is operating in the mode of writing a function. But if I see something like `behavior get($var) {}`, I am instead mentally prepared to tell my class how it should *behave* within the engine, instead of what it should do when the `__get` method is called.

## But Syntax is Hard

There are three problems however:

1. There is a lot of inertia and backward-compatibility concerns with changing the syntax for magic methods, and the benefits that I've described are more abstract and less immediately apparent.
2. Using a syntax everyone is familiar with (such as `function`) is simple, while settling on a new syntax is difficult.
3. *If* you wanted the syntax to more correctly described the magic methods, you might want more than one syntax, and that could be potentially even more confusing for some developers.

## My Question for Everyone

So this is my question of the PHP community. How would you improve magic methods? Do you think that the syntax could be improved, and if so how? Do you think that their behavior and usage could be improved, and if so how? Do you think there is a better way to accomplish these features than something like magic methods? Do you have a viable replacement?

I would love to hear a wide variety of thoughts and opinions on this subject. Any RFC that tries to change something like what I'm describing here is doomed to failure without a *LOT* of research and justification. A lot of PHP developers that I know "don't like" magic methods, but the features they provide are incredibly useful! I want to move the language in a direction where the developers actually like a feature that is so useful as magic methods.
