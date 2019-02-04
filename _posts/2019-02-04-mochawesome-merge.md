---
layout: post
title: "Configuring Mochawesome reporter to Cypress"
date: 2019-02-04 12:55:00 +0300
categories: testing
---

[Cypress](https://www.cypress.io) has made a revolution in software end-to-end testing.
Having a set of killer features, it delivers such a smooth experience to developers and testing engineers, especially compared to older tools like Selenium, that makes it win any competition.
Since the first public release of Cypress, I started to use and promote it on every project I support.

If you want to use Cypress on your project you might be interested in generating test reports.
Since Cypress is built on top of [Mocha](https://mochajs.org/) testing framework, it supports the same test reporters as Mocha does. One of the most popular reporters for Mocha is [Mochawesome](https://adamgruber.github.io/mochawesome/). It provides an interactive HTML page together with JSON file with all the details about your test suites. This is the most user-friendly report I've ever seen.

## The Problem

Since the version [3.0.0](https://github.com/cypress-io/cypress/releases/tag/v3.0.0) Cypress executes each spec in isolation. This leads to Mocha generating a separate test report for each spec.
Thus, there's no out-of-the-box solution to generate one mochawesome report for all your specs.

Fortunately, this problem is solvable and below I will show you how.

## A Solution

To be able to generage one big mochawesome report for all Cypress specs I've released a special npm package - [mochawesome-merge](https://www.npmjs.com/package/mochawesome-merge).
From the name you can already guess that it's responsibility is to merge multiple mochawesome reports together.
And that's correct.

### Mochawesome Configuration

I'm going to explain how to integrate mochawesome-merge into an existing Cypress configuration.
The idea is to configure Cypress to generate many test reports and then run a special command to merge them.

Let's start with installing the mochawesome reporter.

~~~
$ npm install mochawesome -D
~~~

Once installed, let's configure Cypress to use the reporter.
Open `cypress.json` file and add `"reporter": "mochawesome"` to it.

~~~ json
{
  "reporter": "mochawesome"
}
~~~

This will generate test reports within the `mochawesome-report` directory under your project root every time you run your cypress tests. But if you run tests now you will be able to see a report only for one of your specs — the one that was run the latest. Reports for all the other tests are being overridden and lost. That's not what we want to achieve, so let's disable the mochawesome's `overwrite` property.

~~~ json
{
  "reporter": "mochawesome",
  "reporterOptions": {
    "overwrite": false
  }
}
~~~

Now if you run your tests you will be able to see many reports being generated under the `mochawesome-report` directory. For each spec two files `mochawesomeXXX.json` and `mochawesomeXXX.html` are being created, where `XXX` is the numeric count. Having 3 digits after the name puts a limitation to have only less than 1000 spec files. But that's enough for most of the projects.

One more limitation is that those numbers are just numbers and they are not related to the spec name, so there's no way to have a mnemonic link between the spec file and the test report name. It means that if you wish to have valid test reports, it's impossible to run only specific Cypress specs — you will need to run them all together. And don't forget to clean the `mochawesome-report` directory before running the tests — to prevent data corruption.

There's one optimization we can do right now. Since Mochawesome generates a lot of html reports and we will consume only one report for all the specs, these intermediate reports are redundant and can be disabled. Let's be more explicit and also mark `json` reports as enabled.


~~~ json
{
  "reporter": "mochawesome",
  "reporterOptions": {
    "overwrite": false,
    "html": false,
    "json": true
  }
}
~~~

### Merging Reports

Ok, we've configured Mochawesome to generate one json report per spec file.
Now let's merge these reports into one big report.
For that we will use [mochawesome-merge](https://www.npmjs.com/package/mochawesome-merge).

First, install the package.

~~~
$ npm install mochawesome-merge --save-dev
~~~

`mochawesome-merge` comes with JavaScript SDK as well as CLI.
It's up to you, which of the interface you prefer.
Here we will be using JavaScript SDK.

Basically, it provides a function that accepts some optional config and returns a promise that resolves to a merged Mochawesome JSON report.


~~~ javascript
const { merge } = require('mochawesome-merge')

merge().then(report => {
  console.log(report)
})
~~~

We need to execute this function after Cypress finishes running all tests.
Fortunately, Cypress does not throw when a test fails. This makes our code simpler.
We don't need to handle errors to be sure that `mochawesome-merge` is executed whatever the test result is.

~~~ javascript
const cypress = require('cypress')
const { merge } = require('mochawesome-merge')

async function runTests() {
  await cypress.run()
  const jsonReport = await merge()
}

runTests()
~~~

To run this code, let's put it in the `scripts/cypress.js` file and configure an npm script in `package.json`.

~~~ json
{
  "scripts": {
    "cy:run": "node scripts/cypress.js"
  }
}
~~~

Now if you run `npm run cy:run` this code will work, but JSON reports will be accumulated within the `mochawesome-report` directory.
Let's modify the function to clean the directory before running tests.
For this I used [`fs-extra`](https://www.npmjs.com/package/fs-extra) npm package.
You can choose other solutions, like running this in an [npm pre- hook](https://docs.npmjs.com/misc/scripts) or just using another library.

Let's make another improvement and exit with non-zero status code when tests fail.

~~~ javascript
const cypress = require('cypress')
const fse = require('fs-extra')
const { merge } = require('mochawesome-merge')

async function runTests() {
  await fse.remove('mochawesome-report') // remove the report folder
  const { totalFailed } = await cypress.run() // get the number of failed tests
  const jsonReport = await merge() // generate JSON report
  process.exit(totalFailed) // exit with the number of failed tests
}

runTests()
~~~

### Generating HTML Report

Until now we did not use the JSON report generated by `mochawesome-merge`.
We just throw it away each time the tests run.

Let's use it to generate a beautiful Mochawesome HTML report.
For this we will use [`mochawesome-report-generator`](https://www.npmjs.com/package/mochawesome-report-generator) npm package.
This package is used internally by Mochawesome (both packages are maintained by the same author).
We will use it here as well.

Let's first install it.

~~~
$ npm install mochawesome-report-generator --save-dev
~~~

And here's what we add to our code.

~~~ diff
  const cypress = require('cypress')
  const fse = require('fs-extra')
  const { merge } = require('mochawesome-merge')
+ const generator = require('mochawesome-report-generator')

  async function runTests() {
    await fse.remove('mochawesome-report')
    const { totalFailed } = await cypress.run()
    const jsonReport = await merge()
+   await generator.create(jsonReport)
    process.exit(totalFailed)
  }

  runTests()
~~~

Now, if you run the command you will get a nice-looking html report under `mochawesome-report/mochawesome.html` path.

### Future Improvements

`mochawesome-report-generator` accepts multiple [configuration options](https://www.npmjs.com/package/mochawesome-report-generator#options) that allow you to control how your report is being generated.

For example, there's an `inline` option that bakes all static assets within the html report file. This is useful if you want to share this report with someone and don't want to deal with its dependencies.

Also, you may find it useful to log a link to the generated report.
Depending on your terminal you will have an ability to open the report by clicking on a link rather than opening a folder by yourself.

----

I hope this guide helped you to integrate Mochawesome into your Cypress setup.
Here I have described the way I did it on my project.
In case of any questions I will be happy to help.

