# Self-replicating code

A small journey through spoons, string escaping in Haskell and the theory of evolution. 
Be entertained.

## Making a spoon using only a spoon

I saw this thumbnail on Youtube and immediately felt **thumbnail envy**.
No matter the contents of the video, the idea is genius.

![image](https://github.com/rubenmoor/haskell-quine/assets/6209273/07349fa7-15c8-4bce-a3ea-a73e84354565)

(Find the actual video [here](https://www.youtube.com/watch?v=OSfUUqNkrOQ) if you care regardlessly.)

The premise of the video is that of a fix point:
The input of the process is a spoon and the output is a spoon again.
If you focus solely on the spoon and treat everything else as *environment*, the spoon basically replicated itself.

Are there other such fix points? Of course:

* any self-hosted compiler: treat its source code and the programmer as *environment* and the compiler (the binary) produces itself
* DNA: from the perspective of e.g. human DNA, even the body of a human is *environment*. It's called the *vehicle*. DNA uses everything around it to replicate itself.
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
In the ollowing paragraphs I will show you how I approached it.
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
# barely-a-quine.hs
import System.Environment (getProgName)
main = putStr =<< readFile =<< getProgName
```

Both example just read a file, the actual source file, and print the contents.
The Wikipedia-article says that's cheating, because reading a file is an *input* and quines can't have any input.
I disagree with Wikipedia:

* The challenge of writing a quine is fun. Any solution counts. The Wikipedia-article lists a couple of "cheating quines" and every single one of them made me giggle.
* The point is not that my quine has *input* but rather that it is not a *constructive quine*: The challenge is to write source code that constructs itself. By reading from the filesystem, my code just relies on the existence of a copy of itself ... to produce another copy.

This is where I basically gave up and looked up how quines are implemented on Rosetta Code.
You can be better than me.

## Most compact quines

Apparently Stephen Kleene put down a formal proof in 1938 that Quines are possible in any Turing-complete programming language.
You don't need modern computers for that and the notion of Turing Completeness probably wasn't common at the time.
Either way, the existence of quine is old knowledge so what remains for the current-days programmer to do?

Implement the shortest quine possible!

This is what Rosetta Code presents as the most compact quine written in Haskell:

(From here on, all code will be Haskell)

```haskell
ap(++)show"ap(++)show"
```

Two problems with that:

1. Even a Haskell programmer can barely read that.
2. To arrive at this compact form, compromises were made. This isn't a complete program but rather an expression that "evaluates to a string of itself" (quoting Rosetta Code). And to run the expression, more code is needed, destroying the essential quine property.

