# Closure Basics
// function is able to access the variables outside of the scope.
//Example 1
function greeting(msg)
{
    return function who(name)
    {
        console.log(`${msg} , ${name} !`);
    };
}

var hello = greeting("Hello");
var howdy = greeting("Howdy")

hello("Rohit")
howdy("Rahul");

// Example 2

function homework(name)
{
    return function topic()
    {
        console.log(`${name} says to study ${this.topic}`);
    };
}

var assignment = homework("Kyle");
// assignment(); to resolve this.topic

var homework = {
    topic : "JS",
    assignment : assignment
}
homework.assignment();

# Adding Up Closure
function adder(num1) {
    return function addTo(num2){
        return num1 + num2;
    };
}

var add10To = adder(10);
var add42To = adder(42);

add10To(15);    // 25
add42To(9);     // 51

Each instance of the inner addTo(..) function is closing over its own num1 variable (with values 10 and 42,
respectively), so those num1's don't go away just because adder(..) finishes. When we later invoke one of 
those inner addTo(..) instances, such as the add10To(15) call, its closed-over num1 variable still exists 
and still holds the original 10 value. The operation is thus able to perform 10 + 15 and return the answer 25.

An important detail might have been too easy to gloss over in that previous paragraph, so let's reinforce it: 
closure is associated with an instance of a function, rather than its single lexical definition. In the 
preceding snippet, there's just one inner addTo(..) function defined inside adder(..), so it might seem like 
that would imply a single closure.

But actually, every time the outer adder(..) function runs, a new inner addTo(..) function instance is 
created, and for each new instance, a new closure. So each inner function instance (labeled add10To(..) 
and add42To(..) in our program) has its own closure over its own instance of the scope environment from 
that execution of adder(..).

Even though closure is based on lexical scope, which is handled at compile time, closure is observed as a 
runtime characteristic of function instances.

# Hoisting 
studentName = "Suzy";

greeting();

function greeting()
{
    console.log(`Hello ${studentName} !`);
}
var studentName;  // runs successfully because of variable hoisting

//function hoisting
greet();
//attches itself to the nearest function
function greet()
{
    console.log(`Hello kyle`);
}

//we might think this may throw undefined but no

var studentName = "Frank";

console.log(studentName);

var studentName;       // we might think that since it is redefined that this may output undefined but 
console.log(studentName);    // how it looks after hoisting 
/* var studentName;
var studentName;
studentName = "Frank";
console.log(studentName); //Frank
console.log(studentName); //Frank
*/
var studentName = "Frank";
console.log(studentName);   // Frank

var studentName;
console.log(studentName);   // Frank <--- still!

// let's add the initialization explicitly
var studentName = undefined;
console.log(studentName);   // undefined <--- see!?


# Diving Deep into Closure
function lookupStudent(studentID) {
    // function scope: BLUE(2)

    var students = [
        { id: 14, name: "Kyle" },
        { id: 73, name: "Suzy" },
        { id: 112, name: "Frank" },
        { id: 6, name: "Sarah" }
    ];

    return function greetStudent(greeting){
        // function scope: GREEN(3)

        var student = students.find(
            student => student.id == studentID
        );

        return `${ greeting }, ${ student.name }!`;
    };
}

var chosenStudents = [
    lookupStudent(6),
    lookupStudent(112)
];

// accessing the function's name:
chosenStudents[0].name;
// greetStudent

chosenStudents[0]("Hello");
// Hello, Sarah!

chosenStudents[1]("Howdy");
// Howdy, Frank!

The first thing to notice about this code is that the lookupStudent(..) outer function creates 
and returns an inner function called greetStudent(..). lookupStudent(..) is called twice, 
producing two separate instances of its inner greetStudent(..) function, both of which are saved 
into the chosenStudents array.

We verify that's the case by checking the .name property of the returned function saved in 
chosenStudents[0], and it's indeed an instance of the inner greetStudent(..).

After each call to lookupStudent(..) finishes, it would seem like all its inner variables would be 
discarded and GC'd (garbage collected). The inner function is the only thing that seems to be returned 
and preserved. But here's where the behavior differs in ways we can start to observe.

While greetStudent(..) does receive a single argument as the parameter named greeting, 
it also makes reference to both students and studentID, identifiers which come from the enclosing
scope of lookupStudent(..). Each of those references from the inner function to the variable in an 
outer scope is called a closure. In academic terms, each instance of greetStudent(..) closes over the 
outer variables students and studentID.

Closure allows greetStudent(..) to continue to access those outer variables even after the outer scope 
is finished (when each call to lookupStudent(..) completes). Instead of the instances of students and 
studentID being GC'd, they stay around in memory. At a later time when either instance of the greetStudent(..)
function is invoked, those variables are still there, holding their current values.

If JS functions did not have closure, the completion of each lookupStudent(..) call would immediately tear
down its scope and GC the students and studentID variables. When we later called one of the greetStudent(..)
functions, what would then happen?

If greetStudent(..) tried to access what it thought was a BLUE(2) marble, but that marble did not actually 
exist (anymore), the reasonable assumption is we should get a ReferenceError, right?

But we don't get an error. The fact that the execution of chosenStudents[0]("Hello") works and returns 
us the message "Hello, Sarah!", means it was still able to access the students and studentID variables. 
This is a direct observation of closure!



var keeps = [];

for (var i = 0; i < 3; i++) {
    keeps[i] = function keepI(){
        // closure over `i`
        return i;
    };
}

keeps[0]();   // 3 -- WHY!?
keeps[1]();   // 3
keeps[2]();   // 3


You might have expected the keeps[0]() invocation to return 0, since that function was created during the 
first iteration of the loop when i was 0. But again, that assumption stems from thinking of closure as 
value-oriented rather than variable-oriented.

Something about the structure of a for-loop can trick us into thinking that each iteration gets its own 
new i variable; in fact, this program only has one i since it was declared with var.

Each saved function returns 3, because by the end of the loop, the single i variable in the program has 
been assigned 3. Each of the three functions in the keeps array do have individual closures, but they're 
all closed over that same shared i variable.

Of course, a single variable can only ever hold one value at any given moment. So if you want to preserve 
multiple values, you need a different variable for each. 


var keeps = [];

for (var i = 0; i < 3; i++) {
    // new `j` created each iteration, which gets
    // a copy of the value of `i` at this moment
    let j = i;

    // the `i` here isn't being closed over, so
    // it's fine to immediately use its current
    // value in each loop iteration
    keeps[i] = function keepEachJ(){
        // close over `j`, not `i`!
        return j;
    };
}
keeps[0]();   // 0
keeps[1]();   // 1
keeps[2]();   // 2


Each function is now closed over a separate (new) variable from each iteration, even though all of them 
are named j. And each j gets a copy of the value of i at that point in the loop iteration; that j never 
gets re-assigned. So all three functions now return their expected values: 0, 1, and 2!

Again remember, even if we were using asynchrony in this program, such as passing each inner keepEachJ() 
function into setTimeout(..) or some event handler subscription, the same kind of closure behavior would 
still be observed.

Recall the "Loops" section in Chapter 5, which illustrates how a let declaration in a for loop actually 
creates not just one variable for the loop, but actually creates a new variable for each iteration of the loop.
That trick/quirk is exactly what we need for our loop closures: 


var keeps = [];

for (let i = 0; i < 3; i++) {
    // the `let i` gives us a new `i` for
    // each iteration, automatically!
    keeps[i] = function keepEachI(){
        return i;
    };
}
keeps[0]();   // 0
keeps[1]();   // 1
keeps[2]();   // 2
//Since we're using let, three i's are created, one for each loop, so each of the three closures just 
//work as expected.

