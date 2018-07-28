# Brotkeys.js

Brotkeys builds upon [jaywcjaylove's hotkeys](https://github.com/jaywcjlove/hotkeys) and depends on [their release](https://github.com/jaywcjlove/hotkeys/blob/master/dist/hotkeys.common.min.js) to offer the additional functionality of reacting not only to a given key press, but also to words, by specifying a javascript function that shall be executed.

## Features

* listen for specific words
* set a character that has to preceed any input
* be informed on start and end of user input, or at every character

## Usage

### Basic Example

Executes the given javascript function when the word is typed. There is no way to start a new word while typing the old word. So `hello` will show a popup saying "general kenobi!", but if you start typing `about` and then decide to switch to `hello`, there won't be any reaction to `abhello`.

```html
<script src="./libs/jaywcjlove_hotkeys/hotkeys.min.js"></script>
<script src="./libs/lucidbrot_brotkeys/brotkeys.js"></script>
<script type="text/javascript">
    // The HotkeyManager class is provided by Brotkeys.js
    var manager;
    var wordMap = new Map([
        ["hello", function(){alert("general kenobi!");}],
        ["about", function(){window.open("https://www.example.com/about", "_self");}],
        ["disable", function(){manager.disable();
                               console.log("No longer listening for words");}]
    ]);
    manager = new HotkeyManager(wordMap, new Map([]));
    manager.enable_f_mode(false); // explanation in next subsection
</script>
```

### F_Mode Example

Brotkeys.js was developed with the idea of [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb) in mind. In short: press F to see what to type in order to open a link without using the mouse. See below for an example of how vimium works (the yellow things) and how a simple website is able to offer the same functionality with Brotkeys.js.

![./ex1.gif](./ex1.gif)

In order for this to work, there's a special option that is set to enabled by default: `manager.enable_f_mode(true)` causes Brotkeys.js to only listen to words that are entered after they press `F`. You can change that to some other character using `manager.set_f_mode_character('g')` for example.

#### Interrupting Words

Here's what F_Mode is good for: Your user can abort typing one word and start typing another. For example  by pressing `F` instantly again after having pressed `F` for the first time. Or by pressing `Esc` anytime during F_Mode.

```html
<script src="./libs/jaywcjlove_hotkeys/hotkeys.min.js"></script>
<script src="./libs/lucidbrot_brotkeys/brotkeys.js"></script>
<script type="text/javascript">
	var manager;
// words of the form [f]abcdefg unless enable_f_mode is set to false
var wordMap = new Map([
	// default settings
	["f", function(){manager.abort_f_mode();}],
	["d", function(){console.log("user disabled shortcuts"); manager.disable();}],
	// my defined words
	["asd", function(){alert("you typed asd");}],
	["asdf", function(){alert("you typed asdf");}],
	["qwer", function(){alert("you typed qwer");}],
]);
// single characters that can interrupt at any time during the word-typing mode
var interruptMap = new Map([
	["escape", function(){manager.abort_f_mode();}],
]);
manager = new HotkeyManager(wordMap, interruptMap);
</script>
```

(Of course, `Esc` is not the smartest choice if your users are already using Vimium, because it leaves the insert mode of Vimium.)

![ex2.giv](./ex2.gif)

### Get Notified

Register a listener function to be called whenever F_Mode is entered or left.

```javascript
// please notify me on entering and leaving fmode
var notifyFModeFunc = function(entering){
	if(entering){
		StyleSwapper.showKeys(true, "LB-SS-swap1"); //important: this class must be defined in an _external_ css file.
	} else {
		StyleSwapper.showKeys(false, "LB-SS-swap1");
	}
};
manager.setNotifyFModeFunction(notifyFModeFunc);
// To disable the notifyMe function:
// manager.setNotifyMeFunction(undefined);
```

Similarly, you can set a function to be called whenever the user is typing _anything_ while in F_Mode. If you have turned F_Mode off, replace this sentence with "... whenever the user is typing _anything_. period.".

```javascript
// please notify me on every keystroke instead of only at the end
var notifyFunc = function(current_word, remaining_words_possible){
var index_in_word = current_word.length;
console.log("---notify---\n["+index_in_word+"]Remaining words: "+remaining_words_possible+"\nCurrent word: "+current_word);
};
manager.setNotifyMeFunction(notifyFunc);
/*
Example result:

keys.html:331 ---notify---
[1]Remaining words: asd,asdf
Current word: a
keys.html:331 ---notify---
[2]Remaining words: asd,asdf
Current word: as
keys.html:331 ---notify---
[3]Remaining words: 
Current word: asf
f not found in available word options. Leaving f_mode.
keys.html:331 ---notify---
[1]Remaining words: qwer
Current word: q
keys.html:331 ---notify---
[2]Remaining words: qwer
Current word: qw
keys.html:331 ---notify---
[3]Remaining words: qwer
Current word: qwe
keys.html:331 ---notify---
[4]Remaining words: qwer
Current word: qwer
*/
// To disable the notifyMe function:
// manager.setNotifyMeFunction(undefined);

// To disable f_mode functionality (fmode means that you have to first press f - or whatever character you designed - before I start accepting words) :
// manager.enable_f_mode(false);
```

### Multiple HotkeyManagers

I don't know why you would do this, but that works as well. Just be aware that using `hotkeys.unbind('*')` would break the behaviour of the Managers (but that's always the case, not just when using multiple managers).  

```javascript
// a manager
var man1 = new HotkeyManager(new Map([["free", function(){console.log("man1: you typed free");}]]), new Map([]));
// another manager
var man2 = new HotkeyManager(new Map([["tree", function(){console.log("man2: you typed tree");}]]), new Map([]));
```

To distinguish between them for debugging, it can help to set a prefix the internal log messages:

```javascript
man2.log_prefix = "[M2]";
man1.log_prefix = "[M1]";
```

### Case Sensitivity

The `HotkeyManager` has three settings regarding case sensitivity, which are all by default set to true (insensitive):

```javascript
// config for key case insensitivity
		this.fmode_caseInsensitivity = true;
		this.interrupt_caseInsensitivity = true;
		this.word_caseInsensitivity = true;
```

You should be able to set these values using direct qualified access.

The first line is whether `F` and `f` both trigger F_Mode.
The second line is whether characters in interruptMap are case insensitive.
The third line is whether capitalization in words matters. True if it does not matter.

## Implementation Details

If you want to know what's going on, without reading the actual code.

### Construction

`constructor(wordMap, interruptMap)`

wordMap maps user-typed words to javascript functions. InterruptMap maps single characters that are always interrupting word entering to execute their own javascript function to those functions.

### current_link_word

Do not modify this, it's used internally. However, reading is fine.
It tells you what characters from the current word the user has already entered.

### mode

Again, if you overwrite this, maybe you break something. But you can read this and get one of the values as specified here:

```javascript
this.ModeEnum = Object.freeze({"f_mode":1, "pre_f_mode":2, "all_disabled":3});
```

### log

Use `log_verbose`, `log_error`, `log_happy` as you please. No side effects apart from formatted writing to `console.log()`.

### enter_f_mode, leave_f_mode, abort_f_mode

Do what they say. Abort simply logs and then calls leave. These are used internally but should be ok to use from outside.

### enable_f_mode

Overwrites `leave_f_mode`, so if for some reason, you're overwriting that as well, consider this.

### disable

Does not unregister the listener to hotkeys.js - it simply does not act on the events until activated again using `enable_f_mode` with either true or false as parameter. (True if you want F_Mode).

### hotkeys_init

There's no harm in calling this, I think. But there's also no reason to. It's called from the constructor.

### Anything Else

Don't touch unless you've read the code.

## Licencing

```javascript
log_happy(text){
		/*
			https://stackoverflow.com/a/21457293/2550406
			(c) bartburkhardt cc-by-sa 3.0
		*/
		[code skipped]
	}
```

`log_happy` was originally provided under CC-BY-SA. However, if this is of concern to you, just remove the function and replace it with a simple `console.log()`. Also, I don't think they care - after all, everybody steals from stackoverflow.

My own code is provided as-is without guarantees, under MIT Licence as provided in the LICENCE file.

I'd be happy to hear about it when you use my code though :)
eric@mink.li

## Note on the Beauty of the Code

It's probably ugly. I've tried a little, but not too hard, and javascript does not offer the options I'm used to. This is not a "sorry", it's just a "I knew that it was ugly when I wrote it, I'm not incompetent".