One of the most confused mechanisms in JavaScript is the this keyword. It's a special identifier keyword that's automatically 
defined in the scope of every function, but what exactly it refers to bedevils even seasoned JavaScript developers.

# Why this?

function identify() {
	return this.name.toUpperCase();
}

function speak() {
	var greeting = "Hello, I'm " + identify.call( this );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};

var you = {
	name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER

This code snippet allows the identify() and speak() functions to be re-used against multiple 
context (me and you) objects, rather than needing a separate version of the function for each object.

Instead what you could have done was 
function identify(context) {
	return context.name.toUpperCase();
}

function speak(context) {
	var greeting = "Hello, I'm " + identify( context );
	console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE

However, the this mechanism provides a more elegant way of implicitly "passing along" an object reference, 
leading to cleaner API design and easier re-use.


