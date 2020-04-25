# What are iterables?
which can be iterated over
Examples:
var arr = [10,20,30];

var arrCopy = [...arr];

var greeting = "Hello world !";
var chars = [...greeting];

chars; //["H","e","l","l","o",......]

Map(object,value)

var buttonNames = new Map();
buttonName.set(btn1, "Button1");
buttonNames.set(btn2, "Button2");

for(let[btn,btnName] of buttonNames) {
  btn.addEventListener("click", function onClick(){
        console.log(`clicked ${btnName}`);
  });
}

# More specific

for(let btnName of buttonNames.values()){
console.log(btnName);
}
//Button1
//Button2
