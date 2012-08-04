sqUID
=====

> A unique ID generator for JavaScript. (Not UUID/GUID).

It seems like more and more these days I'm running into the problem of needing to generate safe, globally unique
identifiers in JavaScript.  Now, there are a few UUID/GUID solutions out there but I don't tend to like them.  Usually
they're long and complex and they always carry the caveat: "this depends on how random Math.random is".  Often times
there's an accompanying discussion about how different browsers handle things differently and it just ends up being
unappetizing.

This is my personal solution wrapped up in a tasty JS module.  I decided that most of the time when I need to
generate unique IDs, there's no real need for me to follow the UUID standard.  Instead, I've put a couple of techniques
together in such a way that I think I can "guarantee" uniqueness with pretty much the same amount of certainy you would
get by using a UUID generator.

> Note: That last sentence may or may not be complete BS.  I haven't done any actual probability calculations here.
> I'm just very confident in the technique and I think that if you read this README you will be too.

**One caveat before we go any farther:** sqUID is designed to be used within the context of an application.
Uniqueness would be far less likely if, say, multiple countries of the world decided to use sqUID independently and
all at the same time to try to generate "globally" unique human identifier tags or something.  You'd be ok if there
was a single application running and everyone all over the world had to access that application for their identifiers.
Just not so much if there were multiple apps running it independently and simultaneously and then afterward you needed
everything produced to be unique.

## Installation

Install it anywhere in whatever way you're used to installing scripts.

## Usage

As of right now, sqUID gives you a global object called `squid` with a single method: `gen`.  This method generates
a unique ID string looking something like this by default:

```javascript
squid.gen();
//=> '1344028640673-1000008-QNx25i5pvvp6DarDghIGkOXXd'
```

You have the option of passing in two arguments as well.  The first argument allows you to prefix your UID and the
second allows you to specify the length of the alpha-numeric portion of the UID at the end.

```javascript
squid.gen('myApp', 10);
//=> 'myApp-1344028778048-1000009-KGiMZxFfTu'
```

## How it works

A "sqUID" is designed to minimize how much we have to trust `Math.random`.  Mainly we rely on an
incrementor system and only have to count on the randomizer for cases in which concurrency or asynchronicity might cause our
incrementor system to glitch out on us.  Following are the steps for assembling a "sqUID".

1. If there is a prefix, tack that on to the beginning and add a dash to the end. Each piece of the sqUID
should be separated from the others by a dash.
2. Add a timestamp in milliseconds since Jan 1, 1970 (standard timestamp). Add a dash.
3. Add 1 to a global, seven-digit incrementor and tack it onto the end. Add a dash.
4. Randomly generate a 25 character alpha-numeric string and add it to the end.

This method of assembly works on the following principles: Because we begin with a timestamp, only sqUIDs generated
during the same millisecond of time ever run the risk of being the same.  

If, by chance, this does happen, each
sqUID should still come out unique due to the secondary incrementor.  Because the secondary
incrementor is 7 digits long, the only way you *should* ever risk getting the same incrementor twice in the same millisecond
would be if your application executed `squid.gen` more than 9,999,999 times in a single millisecond.  After getting that high,
the incrementor resets in order to keep all sqUIDs the same length.

So in the nearly impossible case that your application executes this command 9,999,999 times in a millisecond, or somehow
a bad JS implemtation allows concurrency to tack the same incrementor on to two sqUIDs that get generated at the exact
same moment, then (and only then) do we have to put any faith in the randomizer.  By default it will generate a 25 character
string composed of upper and lower case letters and all digits from 0 through 9.  Thus we only have to rely on the randomizer
in extremely rare and almost impossible cases and, even then, it would only have to create a difference between two strings.
