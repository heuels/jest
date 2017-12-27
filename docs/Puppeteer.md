---
id: puppeteer
title: Using with puppeteer
---

[Puppeteer](https://github.com/GoogleChrome/puppeteer) is a library which
provides a high-level API to control headless Chrome or Chromium over the
DevTools Protocol. With the help of
[custom async test environments](Configuration.md#testenvironment-string) it is
possible to set up Jest to run both end-to-end and visual regression tests in a
real browser environment.

### A puppeteer example

First, import
[jest-image-snapshot](https://github.com/americanexpress/jest-image-snapshot)
package and extend `expect` with a custom matcher.

```js
// setupTestFrameworkScriptFile.js
import {toMatchImageSnapshot} from 'jest-image-snapshot';

expect.extend({toMatchImageSnapshot});
```

Then set up a custom test environment. Note the use of `this.global` object to
pass the puppeteer instance to the test suites.

```js
// puppeteerEnvironment.js
const NodeEnvironment = require('jest-environment-node');
const puppeteer = require('puppeteer');

module.exports = class PuppeteerEnvironment extends NodeEnvironment {
  constructor(config) {
    super(config);
  }

  async setup() {
    await super.setup();
    this.global.browser = await puppeteer.launch();
  }

  async teardown() {
    await this.global.browser.close();
    await super.teardown();
  }

  runScript(script) {
    return super.runScript(script);
  }
};
```

```js
// test.js
describe('/ (Home Page)', () => {
  let page;

  beforeAll(async () => {
    page = await global.browser.newPage();
    await page.goto('https://www.example.com');
  });

  it('should load without error', async () => {
    const text = await page.evaluate(() => document.body.textContent);
    expect(text).toContain('Example Domain');
  });

  it('should not have visual regressions', async () => {
    const screenshot = await page.screenshot();
    expect(screenshot).toMatchImageSnapshot();
  });
});
```

For advanced setup that features instantiating a single puppeteer instance and
connecting to it via a websocket see example in
[this repository](https://github.com/xfumihiro/jest-puppeteer-example).
