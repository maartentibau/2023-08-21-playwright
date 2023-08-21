- [1. Setup](#1-setup)
- [2. First test](#2-first-test)
- [3. Run natively](#3-run-natively)
- [4. Run in UI mode](#4-run-in-ui-mode)
- [5. Setup automatic application start](#5-setup-automatic-application-start)
- [6. Run via VSCode](#6-run-via-vscode)
- [7. Debug via VSCode](#7-debug-via-vscode)
- [8. Native Debugging](#8-native-debugging)
- [9. Configuration Tweaking](#9-configuration-tweaking)
- [10. Assertion](#10-assertion)
- [11. Tracing](#11-tracing)

Checkout the branch _pw-01-starter_. The solution branch is _pw-01-solution_.

For some exercises, VSCode is required. Install the official Playwright extension
from https://marketplace.visualstudio.com/items?itemName=ms-playwright.playwright.

## 1. Setup

Install Playwright via `npm init playwright`. When the scripts asks for various options, use the default values.

Verify that the "package.json" has a dev dependency to `@playwright/test`, and that there is a **playwright.config.ts**.

Remove the folder **text-examples** and also the file **example.spec.ts** in **tests**.

## 2. First test

We don't write our first test manually (for the first steps), but use the code generator.

Start the application manually via `npm run start`. Keep the terminal running and in a separate, run `npx playwright codegen`.

A browser and a small window with the name "Playwright inspector" should open. In its menu, the "Record" item should be
in red color.

In your browser, type in "http://localhost:4200". Click on the "Customers" menu item. A table with
customer entries should appear. Click on the header which says "Customers".

Go the "Playwright inspector". Stop the recording and copy the content into a new file in our repository which you
call **tests/init.spec.ts**.

It should look (kind of) like this:

_tests/init.spec.ts_

```typescript
import { test, expect } from "@playwright/test";

test("test", async ({ page }) => {
  await page.goto("http://localhost:4200/");
  await page.getByTestId("btn-customers").click();
  await page.getByRole("heading", { name: "Customers" }).click();
});
```

## 3. Run natively

Execute

```bash
npx playwright test ./tests/init.spec.ts
```

You will see that the test runs and ends successfully.

If you want to see the browsers in action, run

```bash
npx playwright test ./tests/init.spec.ts --headed
```

To speed them up, you can run the tests only in chromium. That would be

```bash
npx playwright test ./tests/init.spec.ts --headed --project=chromium
```

Replace the "test" script in the **package.json**.

```jsonc
"test": "playwright test"
```

## 4. Run in UI mode

Running Playwright via CLI is useful, but most of the time, you'll use the UI mode. Add a new entry
to the `scripts` property in the **package.json**:

```jsonc
"test:dev": "playwright test --ui"
```

Make yourself acquainted with it:

- Verify that the test automatically runs as soon as you change something in the sourcecode
- Find the "Pick Locator" feature and use it to find out the selector for the "Settings" header in home
- Find where the UI shows the console.logs of the application. It should print "Angular is running in development mode".
- Find out which network requests happen, when you click on the Customers menu button.
- Run the test in webkit

Now run it via `npm run test:dev`. You should that the Playwright UI opens.

## 5. Setup automatic application start

Stop your application.

At the end of the configuration in **playwright.config.ts**, uncomment the `webServer` property and set it to

```ts
{
  webServer: {
    command: 'npm run start',
    url: 'http://localhost:4200',
    reuseExistingServer: !process.env.CI,
  }
}
```

Rerun the tests and you will say that Playwright automatically starts and stops the application.

## 6. Run via VSCode

Open your VSCode, and run this test. If your Playwright extension works, you should say a play button left to the `test`
command. Click on it.

## 7. Debug via VSCode

1. Set a breakpoint to the last line of your test, where it selects for the header "Customers".
2. Do a right mouse-click on the play button. A menu should open. Click on debug.
3. The browser starts and stops at the breakpoint.
4. In VSCode, replace `page.getByRole('heading', { name: 'Customers' }).click();` with `page.locator("text='Customers'")` You should see that two elements inside the DOM are highlighted.
5. Change the selector to `"text=Holidays"`. The browser should highlight the holidays button and the header. Now change it to `"text='Holidays'"` and only the button should be selected.
6. Change your locator page to the original one.

## 8. Native Debugging

Add another script `test:debug` to the **package.json**. It should execute `playwright test --debug --project=chromium`. Playwright starts in debugging mode which offers more features than VSCode but lacks the IDE integration.

Execute the test, play with the embedded element locator, and take a look at the Playwright inspector's bottom area
where you see a detailed history of the commands that have been executed so far.

## 9. Configuration Tweaking

Let's extend our configuration with additional values:

- `config.expect`: By default, an assertion has a timeout of five seconds. Reduce it to four seconds. Add the property `expect: {timeout: 4000}` to the `config`
  property.
- `config.use.baseURL`: The URL of our application is http://localhost:4200. Add a `baseURL` property to `config.use`
  and set its value to `http://localhost:4200`.
- `config.use.screenshot`: We want to see what went wrong, when a test failed. Set the `screenshot` property
  to `only-on-failure`.
- `config.use.trace`: For more detailed analysis, we want to have the tracing data as well. Set the property's value
  to `retain-on-failure`.
- `fullyParallel: false`: Our application doesn't run on optimal hdardware. Therefore we need to disable fully parallel execution. This means that the tests within a file run in sequential order.

## 10. Assertion

Let's simplify our test a little bit and add an assertion at the end.

**1. Use `baseURL`**

You configured already the `baseUrl` in _playwright.config.ts_, so no need to use the value http://localhost:4200 in the
test anymore.

Replace the command

`await page.goto('http://localhost:4200')`

with

`await page.goto('')`.

Re-run the test and make sure it is still working.

**4. Asserting**

The success criteria for our test is that the header "Customers" is shown. At the moment we just click on it. Although
the test will fail if the header is not present, this is not a "real assertion".

Replace

`await page.getByRole('heading', { name: 'Customers' }).click();`

with

```typescript
await expect(page.getByRole("heading", { name: "Customers" })).toBeVisible();
```

The final test should look like this:

```typescript
import { expect, test } from "@playwright/test";

test("test", async ({ page }) => {
  await page.goto("http://localhost:4200/");
  await page.getByTestId("btn-customers").click();
  await expect(page.getByRole("heading", { name: "Customers" })).toBeVisible();
});
```

By now, you know to re-run the test, don't you 👌😉.

## 11. Tracing

If you remember, you set the `config.use.trace` property to `retain-on-failure`. Since your test always succeeded, there
is no trace yet. Time to change that.

Change the assertion to

```typescript
await expect(
  page.getByRole("heading", { name: "Customer", exact: true })
).toBeVisible();
```

Re-run the test and make sure it fails.

Verify that you have a directory _test-results/tests-init-test-Chromium_ and that it contains a _trace.zip_ and a _
test-failed-1.png_.

The png file is the screenshot. We want to see the trace. In order to do that,
run `npx playwright show-trace ./test-results/init-test-chromium/trace.zip`.

A new window should open. Take your time and make sure you understand its elements. Hover over the timeline on the top,
click on the commands, etc.
