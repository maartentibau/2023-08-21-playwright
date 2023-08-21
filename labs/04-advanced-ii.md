- [1. CRUD Test](#1-crud-test)
- [2. Sequential Tests](#2-sequential-tests)
- [3. Failing Test](#3-failing-test)
- [4. Network Stubbing I](#4-network-stubbing-i)
- [5. Sign In](#5-sign-in)
- [6. Cache Sessions](#6-cache-sessions)

Checkout the branch _pw-04-starter_. The solution branch is _pw-04-solution_. If you managed to finish the former exercise successfully, you don't have to switches branches.

## 1. CRUD Test

Create a new file **tests/crud.spec.ts**.

Add a test that runs in three different steps:

1. Add a new customer
2. Edit the customer
3. Delete the customer

Make use of the `page.step` to organise your test and use soft assertions.

Make sure that the test fails per step and verify that the execution still continues.

In addition, tag the test as 'slow'. Also apply `test.slow` on it, so that it gets a higher timeout.

Finally, make use of the fixtures.

<details>
<summary>Show Solution</summary>
<p>

**tests/crud.spec.ts**

```typescript
import { expect, test as base } from "@playwright/test";
import {
  CustomersFixtures,
  customersFixtures,
} from "./fixtures/customer.fixtures";
import { shellFixtures, ShellFixtures } from "./fixtures/shell.fixtures";

const test = base.extend<ShellFixtures & CustomersFixtures>({
  ...shellFixtures,
  ...customersFixtures,
});

test.describe("CRUD for Customers", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("");
  });

  test("add, edit, and delete customer", async ({
    page,
    sidemenuPage,
    customersPage,
    customerPage,
  }) => {
    await test.step("create", async () => {
      await sidemenuPage.select("Customers");
      await customersPage.add();
      await customerPage.fillIn({
        firstname: "Rudolf",
        lastname: "Huber",
        country: "Germany",
        birthday: new Date(1990, 5, 1),
      });
      await customerPage.submit();
      await expect.soft(customersPage.rowByName("Rudolf Huber")).toBeVisible();
    });

    await test.step("edit", async () => {
      await customersPage.edit("Rudolf Huber");
      await customerPage.fillIn({ firstname: "Rudi" });
      await customerPage.submit();
      await expect.soft(customersPage.rowByName("Rudi Huber")).toBeVisible();
    });

    await test.step("remove", async () => {
      await customersPage.edit("Rudi Huber");
      page.on("dialog", (dialog) => dialog.accept());
      await customerPage.remove();
      await expect.soft(customersPage.rowsLocator).toHaveCount(10);
      await expect
        .soft(customersPage.rowByName("Rudi Huber"))
        .not.toBeVisible();
    });
  });
});
```

</p>
</details>

## 2. Sequential Tests

Duplicate the test from above. This time though, make it a test group (suite) and ensure that the executions happens sequentially.

Since the sequential test will run on separate browser sessions, we have to enable the backend or the customers.

You can do that by disabling the switch "Mock Customers" on the landing page.

Make sure you come up with a randomised customer name. Otherwise you might collide with your co-worker's tests or a worker process.

Implement this test in **/tests/sequential-crud.spec.ts**

<details>
<summary>Show Solution</summary>
<p>

**tests/sequential-crud.spec.ts**

```typescript
import { expect, test as base } from "@playwright/test";
import { shellFixtures, ShellFixtures } from "./fixtures/shell.fixtures";
import {
  customersFixtures,
  CustomersFixtures,
} from "./fixtures/customer.fixtures";

const test = base.extend<ShellFixtures & CustomersFixtures>({
  ...shellFixtures,
  ...customersFixtures,
});

test.describe("Sequential CRUD", () => {
  test.describe.configure({ mode: "serial" });

  let firstname = "Rudolf";
  const lastname = `Huber ${Math.floor(Math.random() * 100000)}`;
  let name = `${firstname} ${lastname}`;

  test.beforeEach(async ({ page, sidemenuPage }) => {
    await page.goto("");
    await page.getByRole("switch", { name: "Mock Customers" }).click();
    await sidemenuPage.select("Customers");
  });

  test("add Rudolf Huber", async ({ customersPage, customerPage }) => {
    await customersPage.add();
    await customerPage.fillIn({
      firstname: firstname,
      lastname,
      country: "Germany",
      birthday: new Date(1990, 5, 1),
    });
    await customerPage.submit();
    await expect.soft(customersPage.rowByName(name)).toBeVisible();
  });

  test("rename Rudolf to Rudi", async ({ customersPage, customerPage }) => {
    firstname = "Rudi";
    await customersPage.edit(name);
    await customerPage.fillIn({ firstname });
    await customerPage.submit();
    name = `${firstname} ${lastname}`;
    await expect.soft(customersPage.rowByName(name)).toBeVisible();
  });

  test("remove Rudi", async ({ page, customersPage, customerPage }) => {
    await customersPage.edit(name);
    page.on("dialog", (dialog) => dialog.accept());
    await customerPage.remove();
    await expect.soft(customersPage.rowsLocator.first()).toBeVisible();
    await expect.soft(customersPage.rowByName(name)).not.toBeVisible();
  });
});
```

</p>
</details>

## 3. Failing Test

Change the test with Knut Eggen to a failed test, i.e. the test should fail and this is expected outcome.

You might want to lower the timeout so that the test doesn't take too long.

<details>
<summary>Show Solution</summary>
<p>

**tests/basics.spec.ts**

```typescript
test("delete Knut Eggen", async ({ page, customersPage, customerPage }) => {
  test.fail();
  await customersPage.edit("Knut Eggen");

  page.on("dialog", (dialog) => dialog.accept());
  await customerPage.remove();

  await expect(customersPage.rowsLocator).toHaveCount(10);
  await expect
    .configure({ timeout: 2000 })(customersPage.rowByName("Knut Eggen"))
    .toBeVisible();
});
```

</p>
</details>

## 4. Network Stubbing I

Let's overwrite the backend communication when it comes to loading customers.

Make sure, you disable the mocking of the customer (see Sequential Tests).

Write a test (in **tests/network.spec.ts**), that returns only one customer with following data:

```typescript
const customer = {
  firstname: "Isabell",
  lastname: "Sykora",
  birthdate: "1984-05-30",
  country: "AT",
};
```

Find out the request and the structure of the necessary response via Playwright's UI.

<details>

<summary>Solution</summary>

**/tests/network.spec.ts**

```typescript
import { expect, test as base } from "@playwright/test";
import { ShellFixtures, shellFixtures } from "./fixtures/shell.fixtures";
import {
  CustomersFixtures,
  customersFixtures,
} from "./fixtures/customer.fixtures";

const test = base.extend<ShellFixtures & CustomersFixtures>({
  ...shellFixtures,
  ...customersFixtures,
});

test.describe("network", () => {
  test("should mock customers request with Isabell Sykora", async ({
    page,
    customersPage,
    sidemenuPage,
  }) => {
    await page.goto("");
    await page.getByRole("switch", { name: "Mock Customers" }).click();
    page.route(
      "https://api.eternal-holidays.net/customers?page=0&pageSize=10",
      (req) =>
        req.fulfill({
          json: {
            content: [
              {
                firstname: "Isabell",
                name: "Sykora",
                birthdate: "1984-05-30",
                country: "AT",
              },
            ],
            total: 1,
          },
        })
    );

    await sidemenuPage.select("Customers");
    await expect(customersPage.rowsLocator).toHaveCount(1);
  });
});
```

</details>

## 5. Sign In

Write a test that signs in a user.

As for the credentials. You can use "john.list@host.com" as username as "John List" as password.

The test should also verify, that our applications sends a request to Auth0 on startup, if the user is signed in. To test that, you need to reload the page after the sign in process was successful.

The filename should be **tests/authentication.spec.ts**.

<details>

<summary>Solution</summary>

**./tests/authentication.ts**

```typescript
import test, { expect } from "@playwright/test";

test.describe("Authentication", () => {
  test("auth0 authenticates when already signed in", async ({ page }) => {
    await page.goto("");
    await page.getByRole("button", { name: "Sign In" }).click();
    await page.getByPlaceholder("yours@example.com").fill("john.list@host.com");
    await page.getByPlaceholder("your password").fill("John List");
    await page.getByRole("button", { name: "Log In" }).click();
    await page.getByText("Welcome John List").waitFor();

    let authorizeRequestSent = false;
    page.on("request", (req) => {
      if (req.url().startsWith("https://dev-xbu2-fid.eu.auth0.com/")) {
        authorizeRequestSent = true;
      }
    });
    await page.reload();

    await page.getByText("Welcome John List").waitFor();
    expect(authorizeRequestSent).toBe(true);
  });
});
```

</details>

## 6. Cache Sessions

Rename your **./tests/authentication.ts** to **./tests/authentication.setup.ts**

Create a new project type, which runs all files with an ending of "\*\*setup.ts". It should also run before all the projects.

Modify the **authentication.setup.ts**:

- It is not necessary to verify the request to Auth0
- At the end of thte test store the context into the file **/session.json**.

Create a new file, called **./tests/authenticated.spec.ts**. It should be the only test which uses the storage state of **/session.json**. The test should open the application and verify that the user is logged in.

<details>
<summary>Show Solution</summary>
<p>

First, define the path where we want to store the session data;

**tests/storage-path.ts**

```typescript
export const storagePath = "session.json";
```

---

Transform the former test to a setup test.

**tests/authentication.setup.ts**

```typescript
import test from "@playwright/test";
import { storagePath } from "./storage-path";

test.describe("Authentication", () => {
  test("auth0 authenticates when already signed in", async ({ page }) => {
    await page.goto("");
    await page.getByRole("button", { name: "Sign In" }).click();
    await page.getByPlaceholder("yours@example.com").fill("john.list@host.com");
    await page.getByPlaceholder("your password").fill("John List");
    await page.getByRole("button", { name: "Log In" }).click();
    await page.getByText("Welcome John List").waitFor();
    await page.context().storageState({ path: storagePath });
  });
});
```

---

Now create the "setup" project type and configure the dependencies.

**playwright.config.ts**

```typescript
export default defineConfig({

  // ...

  projects: [
    {
      name: 'setup',
      use: { browserName: 'chromium' },
      testMatch: '*.setup.ts',
    },
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'], storageState: 'session.json' },
      dependencies: ['setup'],
    },

    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
      dependencies: ['setup'],
    },

    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
      dependencies: ['setup'],
    },
});
```

---

Finally, write a test which loads the session data.

**tests/authenticated.spec.ts**

```typescript
import { expect, test } from "@playwright/test";
import { storagePath } from "./storage-path";

test.describe("Authenticated", () => {
  test.use({ storageState: storagePath });
  test("user should be authenticated", async ({ page }) => {
    await page.goto("");
    await expect(page.getByText("Welcome John List")).toBeVisible();
  });
});
```

</p>
</details>
