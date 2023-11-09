# Self-replicating code

A small journey through spoons, string escaping in Haskell and the theory of evolution. 
Be entertained.

## Making a spoon using only a spoon

I saw this thumbnail on Youtube and immediately felt **thumbnail envy**.
If I could come up with thumbnails like these, I would be a Youtube star.
No matter the contents of the video, the idea is genius.

![image](https://github.com/rubenmoor/haskell-quine/assets/6209273/07349fa7-15c8-4bce-a3ea-a73e84354565)

(Find the actual video [here](https://www.youtube.com/watch?v=OSfUUqNkrOQ) if you care regardlessly.)

The premise of the video is that of a fix point:
The input of the process is a spoon and the output is a spoon again.
If you focus solely on the spoon and treat everything else as *environment*, the spoon basically replicated itself.

Are there other such fix points? Of course:

* any self-hosted compiler: treat its source code and the programmer as *environment* and the compiler (the binary) produces itself
* DNA: from the perspective of e.g. human DNA, even the body of a human is *environment*. In biology, it's called the *vehicle*. DNA uses everything around it to replicate itself.
* Quines (if you don't know what that is, just read on)

## Better then spoons

If you actually watched the video, you realize an actual knive would have fit the idea of a fix point better.
Even a knive, though, doesn't work:
The resulting knive isn't made of the same material.

A friend of mine took up the idea of the fix point and pointed out that there exist programs that have their own source code as output.
Intruiging.
At this point, you probably should stop reading and instead pick your favourite programming language and implement such a program.
First spoiler: These self-replicating programs are called Quine.
They have a whole [Wikipedia-article](https://en.wikipedia.org/wiki/Quine_(computing)) and there is this [list of short quines in diffrent programming languages on Rosetta Code](https://rosettacode.org/wiki/Quine).

Do yourself a favour and don't open these links.
Programming a quine isn't easy.
In the following paragraphs I will show you how I approached the challenge.
Once you see the code you will think "Oh, of course!" and you will regret that you haven't tried it yourself.
This was the last spoiler warning.

## My first quine

My first attempts at quines were so bad, you should barely treat them as spoilers.
Also, there isn't a lot to comment. They are rather trivial:

```python
# barely-a-quine.py
with open(__file__, "r") as me:
    print(me.read(), end='')
```

```haskell
-- barely-a-quine.hs
import System.Environment (getProgName)
main = putStr =<< readFile =<< getProgName
```

Both examples just read a file, the actual source file, and print the contents.
The Wikipedia-article says, that's cheating, because reading a file is an *input* and quines can't have any input.
I disagree with Wikipedia, because:

* The challenge of writing a quine is fun. Any solution counts. The Wikipedia-article lists a couple of "cheating quines" and every single one of them made me giggle.
* The point is not that my quine has *input* but rather that it is not a *constructive quine*: The challenge is to write source code that constructs itself. By reading from the file system, my code just relies on the existence of a copy of itself ... to produce another copy.

So I tried again.
This time: no cheating.
This is where I basically gave up and looked up how quines are implemented on Rosetta Code.
(You can be better than me.)

## Most compact quines

Apparently Stephen Kleene put down a formal proof in 1938 that Quines are possible in any Turing-complete programming language.
So you don't even need modern computers for this kind of proof and, at the time, the notion of Turing Completeness probably wasn't common yet.
Either way, the existence of quines is old knowledge so what remains for the current-days programmer to do?

**Implement the shortest quine possible!**

This is what Rosetta Code presents as the most compact quine written in Haskell:

```haskell
ap(++)show"ap(++)show"
```

Two problems with that:

1. Even a Haskell programmer can barely read that.
2. To arrive at this compact form, trade-offs have to be accepted. This isn't a complete program but rather an expression that "evaluates to a string of itself" (quoting Rosetta Code). And to run the expression, more code is needed, destroying the quine.

Rosetta Code also presents a "full program", which is a bit easier to read and it's well worth stepping through the code, understanding each step:

```haskell
main = putStrLn $ (++) <*> show $ "main = putStrLn $ (++) <*> show $ "
```

1. `main =`: the entry point and we have to skip its type declaration `main :: IO ()` for the quine to work.
2. `putStrLn $`: write to stdout; eveything that follows
    * must in its entirety be of type `String`
    * will be the entire output of the program
3. `(++) <*> show`: the short story is that this code is equivalent to `\s -> s ++ show s` and thus is an anonymous function that takes a string argument `s` and evaluates to the same string, plus: it concatenates `show s` in the end. This should be thought of as a trick. Applying `show` to a `String` is a way of wrapping a `String` in quotation marks.
4. `$`: the function from the previous steps gets applied; note how the argument `s` of type `String` will be duplicated, it will appear first in its original form and then a second time wrapped in quotation marks.
5. "main = putStrLn $ (++) <*> show $ ": this is literally just a string, it is the source code **as string** and thus begins with "main = ". It ends right where it would begin to repeat itself.

### A remark on Step 3

While not terribly important for today's topic, I don't want to gloss over stuff.
`<*>` is part of the `Applicative` interface next to `pure`.
`Applicative` is short for *applicative functor* and `Applicative f` implies `Functor f`.
I mostly use `<*>` in the pattern `func <$> x <*> y` which gives rise to an easy intuition:

`func <$> x` is equivalent to `fmap func x` and typically means:
I have some function `func :: a -> b` and a value `x :: f a` and a result of type `f b`, where `f` is a Functor.
Instead of just applying `func` (`func x` or `func $ x` both don't compile), I am *fmapping* `func` into the Functor.
E.g. `sin <$> Just pi` evaluates to `Just 0`.

But what about a function that takes two arguments?
`<*>` got us covered:
E.g. `atan2 <$> Just Y <*> Just x` evaluates to, well, something like *Just 0.1234*, where "0.1234" is the result of *atan2(y, x)*.
The expression `atan2 <$> Just Y` is missing the second argument and has the type `Maybe (a -> b)`.

Unfortunately, this is not the pattern used here.
There is no `<$>`/no `fmap`.
But still, to the left of `<*>` we expect something of type `f (b -> c)`, where `f` is an *Applicative Functor*.
`(++)` is a function that takes two arguments, e.g. its type is `a -> b -> c`.
This helps identifying `f`: It is `a ->` or `(->) a`, such that `f (b -> c)` turns out equivalent to `a -> b -> c`.
The `Applicative` instance for `(->) a` says: `func2 <*> func1 = \x -> func2 x (func1 x)`.
My funny choice for the argument names, `func2` and `func1`, shall remind you that the first argument, `func2`, is a function that takes two arguments (`(++)`) and the second argument, `func1`, is a function that take one argument (`show`).

*In conclusion, someone probably wrote the code for Step 3 as `\s -> s ++ show s` and recognized the pattern that allows for the more compact, yet equivalent, `(++) <*> show`.*

You find the full introduction on the topic of applicative functors in [this chapter of learnyouahaskell.com](http://learnyouahaskell.com/functors-applicative-functors-and-monoids).

## Why "most compact" isn't actually that great

When you tried to come up with a quine yourself, you will have noticed how you are circling around a knot.
Every piece of code you add has to show up in the output of the program, which cries for more code to add.
By just trying around, it is not at all obvious that there is always a solution (luckily the Mathematicians from about 100 years ago have proof).
There certainly is a more systematic way to approach the problem!

But: Just looking at those very compact quines does not teach a whole lot on the nature of the problem.
When coding in real-life, there isn't usually a good reason to arrive at the most compact form possible,
rather there are best practices to follow (including top-level type signatures).

Let's move away from the oneliners and write a quine that is also human-readable, and - heaven forbid - extensible.
Yes, normally, good code can be edited without causing headaches.
Quines shouldn't be different.

## Turning oneliners into complete programs

A complete Haskell program sooner or later adds dependencies to libraries and consequently relies on additional files.
E.g. one of those declares build dependencies along with instructions on how to build the executable and so on.
Extending quines to multiple files isn't a straightforward step.
A program of one file can reproduce itself by just printing its source code.
As soon as more than one file is involved, we get into the business of naming and writing files.

Luckily we don't have to go that route and still can have programs that can be considered *complete* without any doubt.

E.g. my Python quine from above (a "Cheater") can be turned into an executable like this:

```python
#!/usr/bin/env python
# barely-a-quine.py
with open(__file__, "r") as me:
    print(me.read(), end='')
```

And it can be run just like this:

```bash
./barely-a-quine.py
```

This variation works on my system, NixOS, even without having Python installed beforehand:

```python
#! /usr/bin/env nix-shell
#! nix-shell -i python -p python312
with open(__file__, "r") as me:
    print(me.read(), end='')
```

To achieve something similar with Haskell, there is [stack's script interpreter](https://docs.haskellstack.org/en/stable/scripts/).
The usage example follows promptly.

When trying to extend an existing quine by just one line (e.g. the type signature `main :: IO ()`),
again headaches await.
We are appraoching a more systematic perspective on the inner workings of quines and thus resolve the headaches.
It is quite instructive, though, to go through the process on one's own.

And here I present my first quine that fulfills the criteria of a full Haskell program, extensible with dependencies even (and no cheating!):

```haskell
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package raw-strings-qq
-}
{-# LANGUAGE QuasiQuotes #-}
import Text.RawString.QQ
main = let f s = tail [r|
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package raw-strings-qq
-}
{-# LANGUAGE QuasiQuotes #-}
import Text.RawString.QQ
main = let f s = tail [r|
|] ++ s in putStrLn $ f $ f $ let g s = s ++ show s in g "|] ++ s in putStrLn $ f $ f $ let g s = s ++ show s in g "
```

Compare this code to the one-liner: does it have to be so long?
Maybe someone can come up with something more elegant.

But let's go through the code, step by step:

1. `#!/usr/bin/stack`: the shebang that allows proper execution
2. `{- stack script ...`: the magic of stack's script interpreter; even packages can be added
3. `{-# LANGUAGE QuasiQuotes #-}`: A haskell language extension that enables multi-line strings without a lot of noise; this is extremely helpful as we will see later.
4. `main =`: Woops! Still missing the type signature for `main`. But now things are different, we can just add `main :: IO ()` without headache; just be sure to add it twice: right above and a second time down below.
5. (still same line) `let f s = tail [r|`:
     * define the function `f` that takes `s` as argument; `f :: String -> String`
     * open a multiline string with `[r|`; it is closed at the beginning of the very last line by `|]`
     * `tail` removes the first character of the multiline string, which is a line break
6. `#!/usr/bin/stack`: we've seen that before: repeat the whole program, but now as a string; the process continues until the last line
7. `|] ++ s`: close the multi-line string and concatenate `s`; up to here we only have defined the function `f :: String -> String`
8. `in putStrLn $`: finally print something to the screen; what follows now must be the whole code as `String`
9. `f $ f $`: consider this a trick: applying `f` twice is just concatenation. We can do this because of the fact that our multi-line string **does not rely on quotes or escaping**; the first 8 lines of code are literally repeated in the program
10. `let g s = s ++ show s`: copying from the one-liner on Rosetta code: given that there is just one line left to complete the quine, we can apply the very same trick as explained above.

Douglas Hofstadter would appreciate the symmetry, I am sure:
The lines 1-8 are repeated identically as lines 9-16 and the line 17 repeats its first 57 columns horizontally, apart from quotation marks.

We are not done yet, however.
I dislike the fact that, again, I relied on tricks.
It becomes obvious that a quine requires *some* way to deal with strings and their specific syntax.
I.e. it seems as if, without the QuasiQuote syntax for multi-line strings, the quine would be impossible.
But can that be?
Haskell certainly is turing-complete without QuasiQuote syntax!

We get to that in the next and final pragraph.
First a remark on verifying quines.

### Testing quines

Using the one-file approach with the bash shebang, allows to write a very compact quine verifier in shell script.
E.g. save the above code in a file "quine.hs" and run

```sh
./quine.hs > tmp.txt && diff tmp.txt quine.hs
```

When there is an error in your Haskell syntax, you will see compiler errors.
When there is a difference between the output of the program and its source, you will get a diff view that helps to fix the problem.
When the program runs and produces its source as output, you get no output and a success return code.

## A quine without difficult tricks

When you read [Dijkstra: Notes on structured programming](https://dl.acm.org/doi/10.5555/1243380.1243381), you get a deeper understanding on what programming is.
Long story short: it's not, in general, about clever tricks.

Let's rewrite the quine to a representable form and in a way that reveals domain knowledge,
i.e. knowledge about the nature of the problem.
The [Wikipedia article](https://en.wikipedia.org/wiki/Quine_(computing)) helps out by providing "the basic structure of a quine" in Java.
Especially the Java 15 version is very readable.
I will provide an (equally elegant) Haskell version here:

(In the very end, I will present a version that doesn't rely on QuasiQuotes)

```haskell
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package raw-strings-qq
-}

{-# LANGUAGE QuasiQuotes #-}

import Text.RawString.QQ (r)

main :: IO ()
main = do
    let (pre, post) =
            ( tail [r|
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package raw-strings-qq
-}

{-# LANGUAGE QuasiQuotes #-}

import Text.RawString.QQ (r)

main :: IO ()
main = do
    let (pre, post) =
            ( tail [r||]
            , tail [r|
            )
    putStrLn pre
    putStrLn $ pre <> "|" <> "]"
    putStrLn $ "            , tail [r|\n" <> post <> "|" <> "]"
    putStrLn post|]
            )
    putStrLn pre
    putStrLn $ pre <> "|" <> "]"
    putStrLn $ "            , tail [r|\n" <> post <> "|" <> "]"
    putStrLn post
```

This quine follows a general recipe.

**Step 1**: Write the beginning of a program that defines two strings, `pre` and `post`.
This looks like this:

```haskell
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package raw-strings-qq
-}

{-# LANGUAGE QuasiQuotes #-}

import Text.RawString.QQ (r)

main :: IO ()
main = do
    let (pre, post) =
            ( ""
            , ""
            )
```

**Step 2**: Turn the incomplete program into a string and assign it to `pre`.

```haskell
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package raw-strings-qq
-}

{-# LANGUAGE QuasiQuotes #-}

import Text.RawString.QQ (r)

main :: IO ()
main = do
    let (pre, post) =
            ( tail [r|
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package raw-strings-qq
-}

{-# LANGUAGE QuasiQuotes #-}

import Text.RawString.QQ (r)

main :: IO ()
main = do
    let (pre, post) =
            ( tail [r||]
            , ""
            )
```

**Step 3**: Leave `post` be the empty string for now and write the end of the program,
i.e. the part where output happens:

```haskell
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package raw-strings-qq
-}

{-# LANGUAGE QuasiQuotes #-}

import Text.RawString.QQ (r)

main :: IO ()
main = do
    let (pre, post) =
            ( tail [r|
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package raw-strings-qq
-}

{-# LANGUAGE QuasiQuotes #-}

import Text.RawString.QQ (r)

main :: IO ()
main = do
    let (pre, post) =
            ( tail [r||]
            , ""
            )
    putStrLn pre
    putStrLn $ pre <> "|" <> "]"
```

The tricky part here: `pre` gets printed twice, once as is and the second time in its string representation.
Remember that the onliner relies on `(++) <*> show` for the same thing.
We have the freedom to spread things out in multiple lines.

Note that thanks to the way QuasiQuotes work, the opening bracket `[r|` of the `pre` string is part of the `pre` string.
It's its last line.
Also note, `tail [r|` allows to start the multi-line string with a newline and strip it immediately.
Finally note that I wrote `"|" <> "]"` which is equivalent to `"|]"`.
This way I avoid "|]" in the next step.
We will see that we don't actually depend on these "tricks".

**Step 4**: Assign the string representation of the new code at the end of the program to `post`.
This is an iterative step. You successively add lines at the end of the program to output `post`,
to print it just like `pre`.

(You have to scroll up to see the result.)

I can savely add 

```haskell
putStrLn $ pre <> "|" <> "]"
putStrLn $ "            , tail [r|\n" <> post <> "|" <> "]"
```
to the string representation.
If I had chosen `"|]"` that would be not the case.

And the program is complete.

### String escaping works, too!

The final version to present here works without the QuasiQuotes.
This makes the program look even less like a bag of tricks, at the expense of some additional work.
The basic structure is the same, though:

```haskell
#!/usr/bin/env stack
{- stack script
   --resolver lts-21.13
   --package text
-}

{-# LANGUAGE OverloadedStrings #-}

import qualified Data.Text as Text
import qualified Data.Text.IO as Text

main :: IO ()
main = do
    let (pre, post) =
            ( "#!/usr/bin/env stack\n\
              \{- stack script\n\
              \   --resolver lts-21.13\n\
              \   --package text\n\
              \-}\n\
              \\n\
              \{-# LANGUAGE OverloadedStrings #-}\n\
              \\n\
              \import qualified Data.Text as Text\n\
              \import qualified Data.Text.IO as Text\n\
              \\n\
              \main :: IO ()\n\
              \main = do\n\
              \    let (pre, post) ="
            , "            )\n\
              \    let mkStrRepr str =\n\
              \              Text.intercalate \"\\\\n\\\\\\n              \\\\\"\n\
              \            $ Text.lines\n\
              \            $ Text.replace \"\\\"\" \"\\\\\\\"\"\n\
              \            $ Text.replace \"\\\\\" \"\\\\\\\\\" str\n\
              \    Text.putStrLn pre\n\
              \    Text.putStrLn $ \"            ( \\\"\" <> mkStrRepr pre <> \"\\\"\"\n\
              \    Text.putStrLn $ \"            , \\\"\" <> mkStrRepr post <> \"\\\"\"\n\
              \    Text.putStrLn post"
            )
    let mkStrRepr str =
              Text.intercalate "\\n\\\n              \\"
            $ Text.lines
            $ Text.replace "\"" "\\\""
            $ Text.replace "\\" "\\\\" str
    Text.putStrLn pre
    Text.putStrLn $ "            ( \"" <> mkStrRepr pre <> "\""
    Text.putStrLn $ "            , \"" <> mkStrRepr post <> "\""
    Text.putStrLn post
```

Marvel at the backslashes and a defining property of the quine becomes painfully obvious.
The function `mkStrRepr` turns a string into the way the same string looks like in the original source code.
In the original source code, any backslash needs escaping, i.e. '\' is a reserved character and you need "\\"
for "\" to appear.
Given some string, we have to revert this and thus replace "\" with "\\".
Now the fun part: in the source code this looks like

```haskell
Text.replace "\\" "\\\\" str -- replacing \ with \\
```

This code, without QuasiQuotes, becomes much longer for a couple of reasons:

* QuasiQuotes makes the leading backslashes redundant, as well as the newline character and the trailing backslash
* QuasiQuotes does not use quotation marks, thus no need to ever escape a quotation mark
* Any added line shows up twice: we always maintain the code **and** its string representation

Of course even without QuasiQuotes there are ways to work around that.
E.g.

* replace the multiline string with a list of strings, to avoid the backslashes
* define a variable `q = chr(34)`, which is equal to '"', to avoid having to escape quotation marks ever

## Evolving quines

In biological evolution, replication started in the primordial soup.
It is hypothesized that small organic molecules (relatively speaking) grew cristalline structures
or just clumped together in a regular way next to mineral cristalline structures.

If the concentration of some primitive amino acid is high enough,
those can give rise to bigger molecules and eventually a molecule that - given certain environmental conditions - produces copies of itself.
Like with the spoon, the focus on this replicating molecule is arbitrary at first.
It's the environment that does the heavy lifting, not the molecule that is being copied.
In particular you need the energy of the sun next to the versatile building blocks (amino acids) for replication.

However, when choosing the time horizons of thousands (or millions, even billions) of years, the replicating molecule does become a salient feature.
And it's not a specific molecule that replicates itself physically (you can't duplicate atoms), but the structure it creates.

This raises a couple of questions:

* What would need to be done to have quines evolve?
* What is the primordial soup transferred to the world of computer programs?
* Is it possible to create an environment for evolving quines?
* Do we need error correcting quines? Or would they evolve by themselves?
* Would quine evolution favor Python (interpreted!) over GHC?
* Is there a way to avoid the combinatorical explosion? How did natural selection not get stuck?
* Can evolving quines create new knowledge? Can they outperform an intelligent programmer?
