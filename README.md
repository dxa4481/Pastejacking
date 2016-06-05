# Pastejacking

Browsers [now allow](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand) developers to automatically add content to a user's clipboard, following certain conditions. Namely, this can only be triggered on browser events. This post details how you can exploit this to trick a user into running commands they didn't want to get ran, and gain code execution.

It should also be noted, for some time similar attacks have been possible via [html/css](https://thejh.net/misc/website-terminal-copy-paste). What's different about this is the text can be copied after an event, it can be copied on a short timer following an event, and it's easier to copy in hex characters into the clipboard, which can be used to exploit VIM, all shown below.

## Demo

Here is a demo of a website that entices a user to copy an innocent looking command https://security.love/Pastejacking

This demo uses JavaScript to hook into the copy event, which will fire via ctrl+c or right-click copy. Right now this demo does works in Chrome, Firefox, and Safari but not with Internet Explorer, however there is a demo below which is IE compatible.

```bash
echo "not evil"
```

Will be replaced with

```bash
echo "evil"\r\n
```

Note the newline character gets appended to the end of the line. When a user goes to paste the echo command into their terminal, "evil" will automatically get echoed to the screen without giving the user a chance to review the command before it executes. 

[This demo](https://security.love/Pastejacking/index1.html) hooks into the keydown event, so if a user uses keyboard shortcuts, i.e. ctrl+c or command+c, an 800ms timer gets set that will override the user's clipboard with malicious code. This demo works in Chrome, Firefox, and Internet Explorer, but is not compatable with Safari.


More sophisticated payloads that hide themselves can also be used, such as something [demoed here](https://security.love/Pastejacking/index3.html) and seen below

```bash
touch ~/.evil
clear
echo "not evil"
```

This command will create an evil file in your home directory and clear the terminal out. The victim appears to have the command they intended to copy, nicely pasted into the terminal.


## Impact
This method can be combined with a phishing attack to entice users into running seemingly innocent commands. The malicious code will override the innocent code, and the attacker can gain remote code execution on the user's host if the user pastes the contents into the terminal.

## How do you protect yourself?
This is not so straight forward. One solution may be to verify the contents of your clipboard before pasting into a terminal, but be careful where you verify these commands. For example if you paste into vim, vim macros may be used to exploit you. An example of this can be seen [in this demo](https://security.love/Pastejacking/index2.html) and below

```javascript
copyTextToClipboard('echo "evil"\n \x1b:!cat /etc/passwd\n');
```

This demo **echo evil** when pasted in terminal, and it will cat the user's /etc/passwd file when pasted into vim.

One solution around this can be seen below

```bash
"+p       -- within vim to paste clipboard without interpreting as vim command
```

If you're running iTerm, you will actually get warned if the command ends with a newline as seen here:

![iTerm](http://i.imgur.com/W8pweF1.png) 

Of course it goes without saying, take note of the source you're pasting from, and exercise additional caution if pasting from questionable sources.
