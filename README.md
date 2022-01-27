# Approach

## Messages

I started by trying to create a clean way of dealing with sending
requests and parsing responses. I settled on creating dataclasses
as pythonic representations of the messages, which can be json-
serialized and deserialized. Marshmallow could have been used for
this purpose, but we're working only with the standard library 
here. The serialization step uses the dataclass `asdict()` method
to create a json-ifiable map, and that's pretty much it. The 
deserialization is funky cause I pull out the type field of the 
message and use it to determine dynamically which message class 
to construct. It's really dumb code, though, so in a larger 
project you'd just want a static mapping to avoid creating any
issues or constraints with naming.


## Connections

After that, I wrote the code that handles setting up and tearing
down connections. I don't like the idea of remembering to close 
sockets and do cleanup myself, so I tried to do it with a context
manager that would handle that for me and make it clear what the
scope of the connection was. As it turns out, the python socket 
API has a `create_connection()` method I wasn't aware of, which 
trivialized the problem for unencrypted sockets as the method
manages the context itself. I just wrapped over it with my own
name. The encrypted socket was basically the same thing, but you
needed to wrap the base socket in SSL, then yield that connection.
No cleanup or closing either way.

I then wrote some basic argparse stuff to handle the CLI aspect of
the program. Not really worth going into detail about

## Wordle strat

Given a pool of words, any guess, and a retry response to that
guess, you can always narrow down the pool of words to the set
of those which could still be the answer. You can then pick a 
random word for that set, retry the guess, and repeat the 
process. You can get pretty wild with this game and start 
looking at information theory to determine the best guesses and
stuff, but I chose to keep it simple. To check if any given word
in the current pool is still a possible solution, it's easier 
to define the conditions that would remove it from the pool:
- If the previous guess contained a correctly placed letter, and
  the test word does not have that letter in the same spot.
- If the previous guess had a letter that was in the solution
  but incorrectly placed, and the test word has the same 
  letter in the same spot.
- If the test word doesn't contain a letter that was marked as
  being present in the previous guess, but in the wrong position

The last case is where it gets funky. A mechanic of this game 
I wasn't really made aware of in the assignment is that a 0 doesn't necessarily 
mean the letter isn't present in the solution. It means that 
the instance of the letter isn't. This is in the documentation,
but consider the following response to the guess of "inion"
```
    Retry(id='57X3pZRyGbiJerRkWhmh', guesses=[{'word': 'pests',
    'marks': [0, 0, 0, 0, 0]}, {'word': 'dinky', 'marks':
    [0, 1, 1, 0, 0]}, {'word': 'inion', 'marks': [0, 0, 2, 2, 2]}]) 
```
So as it turns out, there is actually an "i" in the solution,
because the third character is marked as a 2. However, the 
first "i" gets a 0, because other than the correctly placed 
one, there are no more "i"'s in the solution.

The upshot of this is that you can't just see a 0 and instantly
filter out all words that contain the letter, only words that
contain excess instances of "i" in them. So you then get this
whole letter frequency problem, upon the discovery of which I 
realized I didn't care anymore and ignored this case entirely.

## Python versioning
Since I used 3.7 features and the login server only has 3.6, I 
had to create my own installation of python. I chose 3.7.11 
since it's what the grader runs on.


## Trying out new stuff
I'd never used the remote SSH programming feature of VSCode, and
decided to try it. It's a bit faster to edit in than vim, but it's
lacking a lot of features. This is probably because I tried to 
install the python plugin on the remote instance of vscode and it
didn't work, permissions, maybe? Oh well, problem for another time.


## Testing
I tested my code by running it. There was no debugging complex enough
to warrant a full debugger. Sometimes I printed out guesses and 
responses, and the pool of words left after, which is how I found 
the word frequency thing I mentioned above. After creating the 
connection management stuff, I stopped and checked that I could get
a response to the hello message sent by the client.
