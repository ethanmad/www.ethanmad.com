+++
date = "2015-05-01T21:04:58-08:00"
draft = true
title = "HyperTyper"

+++


![HyperTyper](https://raw.githubusercontent.com/ethanmad/HyperTyper/master/res/hypertyper%20128.png)

  HyperTyper is a tool to help those who can't type compete in typing competitions, like [TypeRacer][typeracer] and [10FastFingers][10ff].
  I aim to reduce discrimination against non- or transtypers from heterotypers by eliminating the social construct that is the keyboard. Switching layouts or not knowing how to type should not mean you have to repeatedly feel the pain of losing.
  The layout of a keyboard is a social construct created by the evil and discriminatory typewriter organization Remington and people who do not conform to the keyboard do not deserve to be punished for their noncomfornance.  
  Further, the privileged elitist typers like Sean Wrona who were unfairly born with the innate ability to type at ludicrous speeds do not deserve to win every competition in which they compete just because they practice typing for hours every day!

  With HyperTyper, this unjust discrimination has been put to an end; anyone can now be a HyperTyper and compete "fairly" even if they [lost a hand to a croc][hook] or suffer from [horrible hand spasms][spasms].
  Equal opportunnity for all!

  [typeracer]: http://typeracer.com/ "Real-time racing against real people."
  [10ff]: http://10fastfingers.com/ "Testing and competitions with daily leaderboards."
  [hook]: http://en.wikipedia.org/wiki/Captain_Hook/ "Do you like codfish?"
  [spasms]: https://gist.github.com/anonymous/bd15723d8e9118c1ea48 "gerangfelmHEALPAgerna!!"

## History
  In 11th grade, I had a second-semester project for my Design and Data Structures class in which I was allowed to make really whatever I wanted.
  Having never made a Chrome extension and being interested in breaking leaderboards, I decided to make a TypeRacer cheat extension after seeing some friends race on the site.  

  Over the course of a few weeks, I built HyperTyper in a few steps.
  I first learned how to edit the page DOM with JQuery and figured out how to input text into a text field.
  The next step was figuring out how to access the text field, given that TypeRacer assigns unique names (seemingly random strings) to their input boxes.
  Next, I had to make the actual Chrome extension, which involves a manifest file and a background script to figure out when the extension should display.
  Because the extension is only useful on a few sites, I didn't want the icon to always show up; instead, the extension only shows up and can only be activated on the supported sites. 
  At some point, I decided that I did not want HyperTyper injecting JQuery into every page.
  Being a JavaScript noob, transitioning the DOM manipulation code to pure JS was difficult and took time, but definitely worth it given the benefit of reduced resource use.

  Finally, to make the project more widely usable (net necessarily useful given it's function and purpose), I made it simple to add compatability with new (but functionally similar) websites by feeding in two arrays with the identifiers (either a name or class) of the text to type and the input field. Because the identifiers are either names or classes, HyperTyper knows which to expect based on the website it's being used on.

  After I added compatability for all the sites I could find, I uploaded the extension to the [Chrome Web Store][webstore] and also figured out how to make a [HyperTyper bookmarklet][bookmarklet] so as not to leave out users of other browsers.

  [webstore]: https://chrome.google.com/webstore/detail/hypertyper/emlnlmijjaghanenmpdjdckanpdinpgn?hl=en "Click install!"
  [bookmarklet]: javascript:(function()%7Bfor(var%20textOptions%3D%5B%22nonHideableWords%20unselectable%22%2C%22cw-QuotePanel-textToTypePanel%22%2C%22row1%22%2C%22practiceText%22%2C%22textData%22%5D%2CinputBoxOptions%3D%5B%22txtInput%22%2C%22cw-TypedinputBox%20race-go%22%2C%22form-control%22%2C%22tentry%22%2C%22userData%22%5D%2Cwebsite%3D-1%2CclassOrId%3D-1%2Cw%3D0%3Bw%3CtextOptions.length%3Bw%2B%2B)if(null!%3Ddocument.getElementsByClassName(textOptions%5Bw%5D)%5B0%5D)%7Bwebsite%3Dw%3BclassOrId%3D0%3Bbreak%7Delse%20if(null!%3Ddocument.getElementById(textOptions%5Bw%5D))%7Bwebsite%3Dw%3BclassOrId%3D1%3Bbreak%7Dif(-1%3Cwebsite%26%26-1%3CclassOrId)%7Bvar%20text%3B0%3D%3D%3DclassOrId%3Ftext%3Ddocument.getElementsByClassName(textOptions%5Bwebsite%5D)%5B0%5D.textContent%3A1%3D%3D%3DclassOrId%26%26(text%3Ddocument.getElementById(textOptions%5Bwebsite%5D).textContent)%3Bvar%20numWords%3Dtext.split(%22%20%22).length%2CinputBox%3Bnull!%3Ddocument.getElementsByClassName(inputBoxOptions%5Bwebsite%5D)%5B0%5D%3FinputBox%3Ddocument.getElementsByClassName(inputBoxOptions%5Bwebsite%5D)%5B0%5D%3Anull!%3Ddocument.getElementById(inputBoxOptions%5Bwebsite%5D)%26%26(inputBox%3Ddocument.getElementById(inputBoxOptions%5Bwebsite%5D))%3Bi%3D0%3Bvar%20addWord%3Dfunction()%7Bif(!(i%3E%3DnumWords))if(word%3Dtext.split(%22%20%22)%5Bi%2B%2B%5D%2C3%3D%3D%3Dwebsite)for(inputBox.value%3D%22%22%2Cl%3D0%3Bl%3Cword.length%3Bl%2B%2B)inputBox.value%2B%3Dword%5Bl%5D%3Belse%20inputBox.value%3Dword%7D%3BaddWord()%3Bwindow.onkeyup%3Dfunction(a)%7B32%3D%3D(a.keyCode%3Fa.keyCode%3Aa.which)%26%26addWord()%7D%7D%7D)() "Drag to your bookmarks bar!"

## Accomplishments
  Here's a list of what I would consider the project's successes:

  -  Blowing past opponents and high scores is a piece of cake
  -  Scores above 500 WPM are very possible even with just a single keyboard (much higher scores attainable with scripts and multiple keyboards)
  -  Winning is fun and there are a lot of different strategies (no mercy, wait until the last second, etc.)
  -  Smashing the spacebar is fun
  -  It's really easy to use
  -  Works on all platforms
  -  Doesn't require script to be inputted via console
  -  Works on multiple sites
  -  Doesn't need to be installed to the operating system


## Failures
  HyperTyper still has the following issues:

  -  It can't get around the CAPTCHA on TypeRacer
  -  Smashing the spacebar is tiring
  -  Simulating a user space bar press is impossible in a Chrome Extension
  -  Only works on sites that take input word-by-word rather than letter-by-letter
  -  Sometimes it works, sometimes it doesn't
  -  Vulnerable to breaking if a website changes

## Conclusion
  All in all, HyperTyper was fun to make and is still fun to use sometimes. I think my brother gets more use out of it than I do.
  Suprisingly, there are over 1650 users  of HyperTyper! I don't know how or why people find the extension, but maybe some people really do want to cheat in races.

  HyperTyper was a great learning experience involving Chrome extensions and JavaScript and it's hilarious to see people's reactions when the see my scores of over 500 WPM.
  Sometimes, just sometimes, people think it's actually humanely possible to achieve that kind of score.
  This project has also spawned a new competition: spacebar mashing (or writing a script to press space).
  On [Typing Speed Contest][tsc], the ranking is really determined by who can press the spacebar the fastest because all the top scores are the result of HyperTyper.

  Anyways, if you do end up using HyperTyper, please type responsibly. Don't make little kids cry when they're just trying to learn to type.

  [tsc]: http://typingspeedcontest.com "Yours truly on top."

## Links
  - [HyperTyper on Chrome Web Store][webstore]
  - [HyperTyper Website][website]
  - [HyperTyper on GitHub][github]

  [website]: http://www.ethanmad.com/HyperTyper
  [github]: https://github.com/ethanmad/HyperTyper

