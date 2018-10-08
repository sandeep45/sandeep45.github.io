earlier course was about thinking in objects

today

we talk about how objects interact

balance upfront design

Strike = 2
Spare = 1
Open
Rolls

From Bob Martin

can i just make it the same
over and over
till you see the patern

rather than primitives -> extract class

Parse
Execute
Return

Generic Testing Terminology to make co-worker communication easy

Double -> Anything i make to make the tests easier
many forms
take a real object and stub something on it
verify double -> stops you from api drifting,
by not letting you double things which no longer exists

Stub -> set
setting state on the object object under test for a certain method

mock -> which watches and tests for changes. did it happen with expects call.
goal -> to decouple you from a farm away collabroator.

spy -> variant of mock. lets the real answer go away

fake -> real object, its a simplified object of the real thing

Time.now
requires you to stub now

if you inject Time
then you can send in a double
its an equally good player of Time
something which plays the roll of time

inheritence -> A gets it, if it doesnt understand, then parent gets it automatically
composition -> A is outisde and can call sub when it gets something it doesnt handle
decoration/declaration -> Sub sits outside, and calls A when needed

TDD is a deisgn tool, but not the design tool.
you can do a spike
sequence diagram
class diagram
write it out in english
etc.

- -- - - - -- - -

Design - Gang of 4

Head First Design Patterns

Composite & Facade
Facade -> meant to simplify, hide things i dont own, dont be glued to things you dont own
Composite -> put multiple objects in 1 object. then you just deal with this 1 object.
and this object pretends to be whoever you need it to be.
e.g. you have 2 databases. composite is the new database.
it deals with the multiple DB's. but for the outside, its the DB.

Adapter & Decorator
Decorator -> covers the object and augements with more things.
adds behavior
Adapter -> bending a single message to another thing.
e.g. rather than saying name, we can not say title
changes api

distinctions are subtle.

read the names of all the patterns and glance of them
so you can see them and then look them up




