---
layout: post
title:  Why we migrated to Jest
date:   2017-04-24 10:11:25
tags: featured
image: /assets/article_images/2017-05-14-git-performance-benchmark/performance.jpg
image2: /assets/article_images/2017-05-14-git-performance-benchmark/performance_mobile.jpg
imageSrc: https://www.flickr.com/photos/amylovesyah/4528869007/
author: Omri Kochavi
---
Ever since I started working as a member of Amdocs&#39; NFV Front End team, the main flaw of our development environment was our unit testing ecosystem. While using [React](https://facebook.github.io/react/), [Redux](http://redux.js.org/docs/introduction/) and [Webpack](https://webpack.github.io/) provided a fast and reliable development experience (with such cool features such as [Hot Module Replacement](https://webpack.github.io/docs/hot-module-replacement.html), [react-devtools](https://github.com/facebook/react-devtools) and [redux-devtools](https://github.com/gaearon/redux-devtools) extensions for Chrome, and the super light [Webpack Dev Server](https://webpack.github.io/docs/webpack-dev-server.html)), running, fixing and writing tests was a frustrating, often agonizing experience.

A poorly performing unit testing system is not only bad for our gentle developers&#39; spirit, but also for the quality of our products. We are only humans, and humans tend to stop doing something that hurts them – and writing tests sure did hurt. The inevitable consequence of a cumbersome unit testing system is a lack of tests, or at least a lack of meaningful tests.

When we finally found the time to take care of this issue and I was assigned to the task, I decided the best way to approach it was to list all of our problems with the current system (which primarily used [karma](https://karma-runner.github.io/1.0/index.html), [mocha](https://mochajs.org/) and [expect](https://github.com/mjackson/expect)), and then find the appropriate tooling to solve them. I will now share these problems with you, and how [Jest](http://facebook.github.io/jest/) helped us to solve (almost) all of them.

# Problems

## Poor performance

This one is pretty straight forward – our tests were slow. Really slow. Karma launched a real Chrome window in order to run the tests, and then connected to it via a socket. We also had a separate webpack configuration for the unit tests themselves, so they had to be transpiled separately. This loading process alone – before even running the tests – could take up to 2-3 minutes. Running all of our tests, which by now includes about 330 tests, also took a considerable amount of time, about 40s to 1 minute. With the help of some kindergarten math, we can understand the whole process could take up to 4 minutes, which is simply too much – for both development and production.

## Too many dependencies

Prior to the migration, our unit test system was composed of a pretty large set of npm modules. Karma for test running, mocha for test orchestration, [chai](http://chaijs.com/) and expect for asserting and [istanbul](https://github.com/gotwarlost/istanbul) for test coverage. This multi-dependencies issue is problematic for a number of reasons:

- **Synchronization** – using multiple tools to fulfill a single task is generally not the best idea. Exactly like coordinating a team to do the task of one guy, stepping on each other&#39;s toes is not an unlikely incident. This coordination can be done, but can take some effort and often requires the use of plugins – which will just add more dependencies to our tree.
- **Flexibility** – as we all know, the JavaScript community is an ever-changing one. New tools and trends are around the corner every other week, and sometimes it&#39;s difficult to keep up. Maintaining many tools for unit testing means that each one of them has to support the exciting new tech you want to bring into your project before you can make the move. Maintaining one dependency that has a strong community behind it (and [Facebook](https://code.facebook.com/)), ensures that things will always be up to date, ready for use and well documented.
- **Hard to reason about** – learning and remembering how all these tools are connected and which one is responsible for each task is a pain, both for existing team members and for newcomers. Figuring out which one is causing a problem you are facing may cause considerable time loss and frustration.

## Unfriendly watch mode

Let&#39;s face the truth – no one likes to write unit tests. It has always been a leading candidate for the &quot;least exciting development task&quot; award. Thus, we always seek to make our life easier and this task a little bit less tedious. One of the things that can really help with this is a great watch mode. Whether you are refactoring an existing piece of code, adding tests to an existing suite or writing a completely new test, you want to see a quick response to your changes.

You also want to run only the tests that are affected by your changes. If you are working in the test area, that can be solved by marking only the test you are working on with _describe.only_ or _test.only_. But if you are working on the source code, you need to start figuring out which tests can be affected by your changes. If your project is more than small-medium in size, this will be very difficult, and usually you&#39;ll end up running all of them to be sure.

For us, karma&#39;s watch mode did not deliver for both of these parameters. From the moment the file was saved, it could take up to a minute or so for the runner to notice the change, transpile all of the files again and start running the tests, which will again take a considerable amount of time. It also has no mechanism of knowing which tests to run based on which file you changed, and the only way to run separate tests in watch mode was by marking the code.

## Cumbersome debugging and output

Another two important factors in a unit testing system is the ability to debug your tests, and the clarity of its output. While karma has a debug tab available in the browser, using it is heavy on performance, and often does not work well with watch mode, meaning you have to refresh your browser.

Karma&#39;s console output was also not a great fit for us. If we were running 300 tests and test #2 and #25 failed, there was no summary at the end and we had to scroll our way up to check which test failed. This is clearly not optimal.

# Jest to the rescue

After some research and reading time, we decided to give Jest a spin. Several voices in the community indicated that the folks at Facebook put in a lot of effort to vastly improve Jest&#39;s performance capabilities, which were way behind other solutions when our team started developing our project. Moreover, the motto at the bottom of Jest&#39;s homepage addressed one of the main philosophies we were searching for:

One of Jest&#39;s philosophies is to provide an integrated &quot;zero-configuration&quot; experience. We observed that when engineers are provided with ready-to-use tools, they end up writing more tests, which in turn results in more stable and healthy code bases.

## Migration process

I&#39;m not going to get into the technical guts of the migration process, as this is beyond the scope of this article. I&#39;m just going to note that it was a relatively painless and straightforward experience, especially considering the multiple libraries and frameworks Jest had to replace.

All of the migration details and options are very well documented in Jest&#39;s [docs](https://facebook.github.io/jest/docs/getting-started.html). Specific pages that proved to be very helpful for us are [Migrating to Jest](https://facebook.github.io/jest/docs/migration-guide.html#content) and [Usage with webpack](https://facebook.github.io/jest/docs/webpack.html#content). If you follow the instructions, you can integrate and start trying out Jest on your existing codebase and test suites in no time.

## Problem solving

After successfully integrating Jest into our codebase, we started examining how it can help us solve the problems I described earlier. The results were satisfying, and I&#39;m going to lay out how it assisted us in each problem.

### Pure performance

The first area of importance was performance – raw speed. The difference was apparent immediately. Jest was able to load, execute and report our test suites in under **20s** , which is a massive improvement over the 3-4 minutes we had with the previous system. Jest is able to do it this fast thanks to two major differences from karma:

- Jest runs in node utilizing a virtual browser, and does not require an actual browser window.
- Jest does not use webpack - it&#39;s redundant to package the files into a bundle because the execution is in a node environment.

As performance was the main problem we had with our previous unit testing system, we were really satisfied with this aspect of Jest right away.

### One dependency

Jest does it all by itself. You heard it right – Jest handles all of the aspects of unit testing that were conducted by numerous tools and plugins. Jest can do running, orchestration, assertion, and reporting on its own. This allowed us to remove 10+ module dependencies from our package.json, thus reducing our project&#39;s dependencies installation time. In addition, we now have just one testing framework, which has a strong community and knows how to coordinate all of the different aspects of unit testing by itself.

### Ultra friendly watch mode

One feature that took us completely by surprise is Jest&#39;s incredible watch mode. It is lightning fast and truly interactive – exactly like a watch mode should be. The second you hit the save button you can see the tests are running again, and there is almost no delay whatsoever.

The second exciting feature relates to the second issue we had with our previous watch mode. As said, karma had no mechanism to determine what tests to run based on your changes. Jest&#39;s watch mode (which actually utilizes the –o command line option) knows how to **look at your changes since the last commit to the VCS, and run only the tests that can be affected by these changes**. Jest does this by building a file dependency tree for each of your tests based on the import/require statements. This is a very powerful feature that comes out-of-the box with no configuration whatsoever. There are more useful features in the interactive mode worth noting – such as running tests based on a file/test name regex.

### Awesome debugging and output

Because Jest runs in a node environment, you can run Jest in a node process with a –debug-brk option (command taken from Jest docs).

_node --debug-brk ./node\_modules/.bin/jest --runInBand [any other arguments here]_

This way, you are free to connect any external debugger you usually use to debug your node applications by using [node-inspector](https://github.com/node-inspector/node-inspector), node&#39;s built-in debugger, or debug inside your favorite editor. It&#39;s worth noting that debugging did not seem to affect loading and execution time.

Jest&#39;s output is also satisfying. The colors are bright and informative, and after all tests run there is a summary of all the failing tests. You can also configure Jest to output coverage in numerous built-in ways.

## Caveats

Of course Jest is not perfect. We have two main issues that bother us:

- **Inability to use configurations from webpack** – our project uses a lot of aliases, which are handled through webpack. Because Jest does not use webpack, we had to configure them in our Jest configuration manually. It would be helpful if Jest could use some configurations from our webpack config file.
- **Confusion with**  **only** – while Jests support _test.only_ and _describe.only_, they are taken into consideration only within the same file. That means the only way to run a single test/file is by using command line options or the regex pattern of the interactive watch mode. We find this a bit uncomfortable, as in karma you can run a specific file by marking it with _describe.only_.

# Disclaimer

I realize that some/most of the issues we were having with karma could be solved by configuration, and I do not wish to claim that karma is not a great and capable testing framework. The point is that out-of-the-box capabilities were also a main focal point for us, and Jest comes with a lot of these. Of course Jest also needed to be configured (mostly webpack alias and resolve) – but it felt like we spent much less time configuring it.

# Conclusion

We found Jest suitable for our needs. Moreover, Jest brings along many useful features that you normally don&#39;t seek in a unit testing framework, but prove really helpful and developer-friendly. Another important lesson for us was that it&#39;s worth to put in the time to revisit some technologies, even if they did not seem like a good fit in the past – the JavaScript community is powerful, and technologies get (way) better thanks to that.

So if you haven&#39;t checked it out yet, or you&#39;re starting a new project – go give Jest a try if you seek a powerful, developer-oriented, well-backed testing framework with great out-of-the box capabilities.