# CSS Reflow Tracer (a.k.a. jaia)

This is a little script that will show a curated list of mutation events on your web page.
You can use this event log to trace down CSS reflows.
It will tell you when class changes, style changes and DOM insertion / removal happens.

Use it in combination with the reflow logger in Firefox (Console > CSS > Info).

## Usage

Copy and paste the script in your browsers Console, or drag this <a href="javascript:(var jaia=function(){function d(a){var
b=[].slice.call(a.target.classList),c=a.oldValue?a.oldValue.split("
"):[],d=b.filter(function(a){return-1===c.indexOf(a)}),e=c.filter(function(a){return-1===b.indexOf(a)});d.length&&console.log("Class
'"+d.join(", ")+"' added",a.target),e.length&&console.log("Class '"+e.join(",
")+"' removed",a.target)}function e(a){var b=function(a){return
a.split(";").filter(function(a){return""!==a.trim()}).reduce(function(a,b){return
b=b.split(":").map(function(a){return
a.trim()}),a[b[0]]=b[1],a},{})},c=b(a.target.getAttribute("style")),d=b(a.oldValue||""),e=Object.keys(c).filter(function(a){return!d[a]}),f=Object.keys(d).filter(function(a){return!c[a]}),g=Object.keys(c).filter(function(a){return
c[a]!==d[a]});e.forEach(function(b){console.log("Style added '"+b+":
"+c[b]+"'",a.target)}),f.forEach(function(b){console.log("Style removed '"+b+":
"+d[b]+"'",a.target)}),g.forEach(function(b){void 0!==d[b]&&console.log("Style
change '"+b+"', newValue '"+c[b]+"', was '"+d[b]+"'",a.target)})}function
f(a){[].forEach.call(a.addedNodes,function(b){console.log("Node added",b,"as
child of",a.target)})}function
g(a){[].forEach.call(a.removedNodes,function(b){console.log("Node
removed",b,"as child of",a.target)})}var
a=document.body,b=window.MutationObserver||window.WebKitMutationObserver||window.MozMutationObserver,c=new
b(function(a){a.forEach(function(a){"attributes"===a.type&&"class"===a.attributeName?d(a):"attributes"===a.type&&"style"===a.attributeName&&e(a),"childList"===a.type&&a.addedNodes.length>0&&f(a),"childList"===a.type&&a.removedNodes.length>0&&g(a)})});return{start:function(){c.observe(a,{attributes:!0,childList:!0,characterData:!0,subtree:!0,attributeOldValue:!0})},stop:function(){c.disconnect()}}}())">bookmarklet</a>&nbsp;
to the bookmarks toolbar of your browser in order to inject it to the website
you are currently visiting.

Then to start logging, run:

```javascript
jaia.start()
// do an action that will cause reflows
jaia.stop()
```

## Demo

* Open about:blank, and then open the developer tools
* Paste the jaia script in the Console
* Enable reflows under CSS > Info (Firefox only)
* Now prepare the page by adding two divs and a function

```javascript
document.body.appendChild(document.createElement('div'))
document.body.appendChild(document.createElement('div'))
document.body.querySelectorAll('div')[0].textContent = 'First div'
document.body.querySelectorAll('div')[1].textContent = 'Second div'

function magic() {
  var d = document.querySelectorAll('div')[0]
  d.style.display = d.style.display === 'none' ? 'block' : 'none'
}
```

Now call the `magic()` function and see that a reflow happens.

```javascript
magic()
// undefined
// reflow: 0.1ms
```

To track down the reflow we enable jaia, and execute the same function again.

```javascript
jaia.start()
magic()
```

Now jaia will log information about probable causes of the reflow in the browsers console, right *after* the reflow.

```
reflow: 0.19ms
"Style change 'display', newValue 'block', was 'none'" <div style="display: block;">
```

The last argument of the log is always the element involved.
You can click it to jump to the node.

## Advanced stuff

It's not always as easy as one single line.
It can happen that multiple actions are logged, you'll have to investigate on your own for now.
A lot of times a reflow will also be caused by a class change,
you'll need to walk through all the rules in the class to track down the reflow.

If you don't get any log information, the reflow probably comes from: `:hover` or `:active` pseudo selectors in CSS.
