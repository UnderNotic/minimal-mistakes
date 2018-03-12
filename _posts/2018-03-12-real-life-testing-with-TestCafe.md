---
title: "Real life testing with TestCafe"
excerpt: "Check if your stuff runs on IE11 as expected"
header:
 teaser: "assets/images/real-life-testing-with-TestCafe.png"
tags:
  - testcafe
  - testing
  - e2e
  - functional testing
  - javascript
---

## TestCafe
TestCafe has lately become one of the most popular E2E testing tool in js/browser world.   
End to end testing gives a confidence booster, especially when we doing something tricky and still we want to support shenanigans like IE11. 

Let's jump into real-world example and make our hands dirty with TestCafe.

## Let's grab something that can be tested
Recently I created small library than can help with loading big files, by reading chunk by chunk.   
It's called `readery` and it's available on [github](https://github.com/UnderNotic/readery). Since testing file-reading in unit tests is not ideal scenario,
let's check how this piece of code works for real.

The plan is to:
- create html page that will make us of testing library
- write test file
- use TestCafe to run tests

Let's have html page setup out of the way:
```html

<!DOCTYPE html>
<html>
<head>
    <script src="readery-iife.min.js"></script>
</head>
<body>
    <input type="file" id="file-input" name="file" />
    <output id="file_output"></output>
    <script>
        function handleFileSelect(evt) {
            var file = evt.target.files[0];
            const fileOutput = document.getElementById('file_output');
            readery.readFromFile(file, d => {
                const div = document.createElement("div");
                div.appendChild(document.createTextNode(d));
                fileOutput.appendChild(div);
            });
        }
        document.getElementById('file-input').addEventListener('change', handleFileSelect, false);
    </script>
</body>
</html>
```
Should be self explanatory if not this is just a page with a button, which can be used to load up a file with values seperated by newline and append them as a div to dom.

## TestCafe test file
First step is to define name for test fixture and set url which will be tested. Test cafe is testing tool to run it, tested website must be already hosted, here it will be available locally on port 8080.

```js
fixture`Readery tests`.page`http://localhost:8080`;
```

Next we will finally start implementing test case.

```js
test("Should split correctly with default new line split", async t => {
})
```
It's worth to mention testcafe is in nature asynchronous. 
Every method on `t` is async and can be awaited. By the way test file can be ES2017 code.

Next TestCafe needs to be instructed to upload a [test file](https://github.com/UnderNotic/readery/blob/master/src/tests/test_file.txt) and then check if it was read correctly by checking content of `#file_output` element.


Uploading file is super easy: 
```js
  await t.setFilesToUpload("#file-input", "./test_file.txt");
```
Getting element content even easier:
```js
    var children = await Selector("#file_output").child();
```

Now we can cover assertions:
```js
    var count = await Selector("#file_output").childElementCount;

    await t.expect("1").eql(await children.nth(0).innerText);
    await t.expect("2").eql(await children.nth(1).innerText);
    await t.expect("3").eql(await children.nth(2).innerText);
    await t.expect("").eql(await children.nth(3).innerText);
    await t.expect("8").eql(await children.nth(4).innerText);
    await t.expect("a").eql(await children.nth(5).innerText);
    await t.expect("").eql(await children.nth(6).innerText);
    await t.expect("").eql(await children.nth(7).innerText);
    await t.expect("").eql(await children.nth(8).innerText);
    await t.expect("").eql(await children.nth(9).innerText);
    await t.expect("!").eql(await children.nth(10).innerText);
    await t.expect("9").eql(await children.nth(11).innerText);
    await t.expect("@").eql(await children.nth(12).innerText);
    await t.expect("123").eql(await children.nth(13).innerText);
    await t.expect("abc123zxy").eql(await children.nth(14).innerText);
    await t.expect("wwwwwwaaaaaaaazxczxcasddfgrtyyuiopoiklj[]kjhnbv!iklj[]kjhnbv!iklj[]kjhnbv!sdsdsd").eql(await children.nth(15).innerText);
    await t.expect(count).eql(16);
});
```

## Test run
To run tests TestCafe CLI can be used:
```js
 testcafe chrome **/*.test.js
```
Second parameter specifies browser, IE is also available, of course only on windows :)

However to actually run the test, website has to be hosted somewhere, most probably on localhost if you'r testing local code.
For that You can use good old [http-server](https://github.com/indexzero/http-server) npm package.

Now to run those two commands together, there is `|` operator but the problem with it is, that using http-server will cause process to never exit, which results in infinite continous integration test runs.
```js
"test": "http-server ./my/website/dir/ | testcafe chrome **/*.test.js"
```

To the rescue comes test cafe with `--app` parameter:
```js
"test": "testcafe chrome **/*.test.js --app 'http-server /my/website/dir/'"
```

## Technology is here
If You'r using travici for integration testing You can setup it easily to run testcafe with browser of your choice provided by saucelabs.
Pretty neat tutorial exaplaining it in details [here]( http://devexpress.github.io/testcafe/documentation/recipes/running-tests-using-travis-ci-and-sauce-labs.html
).


Cheers, for fully working example You can check mentioned readery [github repo](https://github.com/UnderNotic/readery).