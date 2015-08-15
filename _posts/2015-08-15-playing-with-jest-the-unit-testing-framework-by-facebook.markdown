---
layout: post
title:  "Playing with Jest, the unit testing framework by Facebook"
date:   2015-08-15 19:13:54
description: Playing with Jest, the unit testing framework by Facebook
categories:
- blog
permalink: playing-with-jest-the-unit-testing-framework-by-facebook
---

If you already use React, you probably want to test your components (yes you want!). 
Good news, Facebook released not only React but also [Jest][jest] which is for *Painless JavaScript Unit Testing*.<br/><br/>
**...OR NOT**

##Windows not friendly
Currently Jest isn't realy painless on a Windows environment. You have to install Python on your machine, then Visual Studio Express follow by some environment variables to set. This post can help you a lot [Jest on Windows][jest-windows]. If you work in a team, repeat these operations on each machine and ... whait ! ... hey, where are you going ?

##Not compatible with a recent NodeJS version
The setup of Jest on Windows is already annoying but guess what, he's not compatible with the last version of NodeJS the `v0.12` (currently `v0.12.7` exactly). There is more informations available on this issue from their Github [Jest - Node 0.12][issues243]. The only way, is to downgrade NodeJS down to `v0.10` to run Jest. Or, a big workaround consist to swith for [io.js][iojs] which is compatible. That's what I did in order to start playing with Jest. In both case, it's not really acceptable for many companies to change their technical stack just for a dev dependency.

##Code Coverage not available
Having unit tests is a good practise but it's also recommended to know what proportion of your code is really covered by your tests. Istanbul is a famous reporter used for obtain a code coverage. All testing frameworks support Istanbul... except one. [Is there a way to have code coverage in the Javascript Jest testing framework?][issue101] or [Istanbul, Jest support][issue220]

#Hard debug with virtual DOM 
Sometimes, it's necessary to debug either a test or the associated production code to determine what is going on. With the duo Karma / Jasmine, tests are executed on real browsers like Firefox or Chrome. That means you can use the debugging tools embedded into these browsers (Chrome Dev Tools **<3** ). Like Jest use a virtual DOM, the only option available is to use the debbugger provided by NodeJS. He's entirely based on command line. I love command line, it's cool for lot of stuff but clearly you won't debug your code like that, trust me.

##Poor documentation
On all previous subjects, I did a lot of research in order to setup a correct testing environment with Jest. It was really hard to find accurate and usefull informations about it. Even when i was able to write my first test suite, i only find a small official post about testing React with Jest [Tutorial – React][jest-react]. Moreover, Jest throws cryptic stacks trace when he crash and you can't rely on it to figure it out.


##Conclusion
Even if Jest brings some features over Jasmine like automatic mocking all dependencies and promise a execution faster with the virtual DOM, it seems too unmature. Currently, it looks like a unit testing framework made by and for Facebook which certainly fit well to their own constraints but probably not yours... at least not mine. Luckily, you can test without pain your React components with Jasmine or Mocha and it works like a charm.

Happy testing !

Tags: [Jest, React, Unit testing framework, test]

[jest]: https://facebook.github.io/jest/
[jest-windows]: http://ryanlanciaux.github.io/blog/2014/08/02/using-jest-for-testing-react-components-on-windows/
[issues243]: https://github.com/facebook/jest/issues/243
[iojs]: https://iojs.org/en/index.html
[issue101]: https://github.com/facebook/jest/issues/101
[issue220]: https://github.com/gotwarlost/istanbul/issues/220#issuecomment-49512538
[jest-react]: https://facebook.github.io/jest/docs/tutorial-react.html#content  