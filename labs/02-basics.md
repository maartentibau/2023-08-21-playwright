- [1. Warm-Up](#1-warm-up)
- [2. Home greeting](#2-home-greeting)
- [3. Count customers](#3-count-customers)
- [4. Verify customer names](#4-verify-customer-names)
- [5. Add a new customer](#5-add-a-new-customer)
- [6. Rename a customer](#6-rename-a-customer)
- [7. Delete a customer](#7-delete-a-customer)
- [8. Spot the error](#8-spot-the-error)
- [9. User-Facing Selectors](#9-user-facing-selectors)

In this lab, we write the type of tests that should be sufficient to cover the majority of your needs.

Checkout the branch _pw-02-starter_. The solution branch is _pw-02-solution_. If you managed to finish the former exercise successfully, you don't have to switches branches.

Based on the slides, you should have all information available. Together with the official documentation on "https://playwright.dev/" try to come up with the solution on your own. Only, if you don't manage to do it, look up the solution.

Once you have a working test, **make sure it really works by letting it fail.**

## 1. Warm-Up

Verify that our application has an h1 with the text "Unforgettable Holidays." Name the test file **./tests/basics.spec.ts**, wrap the test in a test suite and use a `beforeEach` that opens the web application automatically.

<details>

<summary>Solution</summary>

**./tests/basics.spec.ts**

```typescript
import { test, expect } from "@playwright/test";

test.describe("Basics", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("");
  });

  test("header is Unforgettable Holidays", async ({ page }) => {
    await expect(page.locator("h1")).toHaveText("Unforgettable Holidays");
  });
});
```

</details>

## 2. Home greeting

Another check for a text. This time on the landing page with a `data-testid` selector.

Match against parts of the text, not the complete phrase "Eternal is an imaginary travel agency and is used as training application for Angular developers."

Add the test to the existing test file.

<details>

<summary>Solution</summary>

**./tests/basics.spec.ts**

```typescript
test("greeting on home", async ({ page }) => {
  await expect(page.getByTestId("txt-greeting-1")).toContainText(
    "imaginary travel agency"
  );
});
```

</details>

## 3. Count customers

Write a test that verifies that the customers list has exactly 10 rows.

<details>

<summary>Solution</summary>

**./tests/basics.spec.ts**

```typescript
test("customers list shows 10 rows", async ({ page }) => {
  await page.getByTestId("btn-customers").click();
  const locator = page.getByTestId("row-customer");
  await expect(locator).toHaveCount(10);
});
```

  </details>

## 4. Verify customer names

After you managed, to count the customer rows, the next should pick the 3rd and the 10th customer and verify that their names are "Hugo Brandt" and "Jan Janáček".

<details>

<summary>Solution</summary>

**./tests/basics.spec.ts**

```typescript
test("3rd customer is Brandt, Hugo; 10th is Janáček, Jan", async ({ page }) => {
  await page.getByTestId("btn-customers").click();
  const nameLocator = page.locator(
    "data-testid=row-customer >> data-testid=name"
  );

  await expect(nameLocator.nth(2)).toHaveText("Hugo Brandt");
  await expect(nameLocator.nth(9)).toHaveText("Jan Janáček");
});
```

</details>

## 5. Add a new customer

Add a new customer with firstname Nicholas and lastname Dimou. He should come from Greece and his birthday is 1.2.1978.

After the customer has been added, make sure that he shows up on the first page.

<details>

<summary>Solution</summary>

**./tests/basics.spec.ts**

```typescript
test("add Nicholas Dimou as new customer", async ({ page }) => {
  await page.getByTestId("btn-customers").click();
  await page.getByTestId("btn-customers-add").click();
  await page.getByTestId("inp-firstname").fill("Nicholas");
  await page.getByTestId("inp-name").fill("Dimou");
  await page.getByTestId("sel-country").click();
  await page.getByText("Greece").click();
  await page.getByTestId("inp-birthdate").fill("1.2.1978");
  await page.getByTestId("btn-submit").click();

  await expect(
    page.locator("data-testid=row-customer", {
      hasText: "Nicholas Dimou",
    })
  ).toBeVisible();
});
```

</details>

## 6. Rename a customer

Pick "Latitia, Bellitissa" and rename her to "Laetitia, Bellitissa-Wagner".

<details>

<summary>Solution</summary>

```typescript
test("rename Latitia to Laetitia", async ({ page }) => {
  await page.getByTestId("btn-customers").click();

  await page
    .locator("[data-testid=row-customer]", { hasText: "Latitia" })
    .getByTestId("btn-edit")
    .click();
  await page.getByTestId("inp-firstname").fill("Laetitia");
  await page.getByTestId("inp-name").fill("Bellitissa-Wagner");
  await page.getByTestId("sel-country").click();
  await page.getByText("Austria").click();
  await page.getByTestId("btn-submit").click();

  await expect(
    page.locator("data-testid=row-customer", { hasText: "Bellitissa-Wagner" })
  ).toBeVisible();
});
```

</details>

## 7. Delete a customer

Pick the user Knut Eggen and delete him.

The delete button opens the confirm dialog of the browser. Look up in the official documentation, how you can acknowledge that one.

<details>

<summary>Solution</summary>

**filename.ts**

```typescript
test("delete Knut Eggen", async ({ page }) => {
  test("delete Knut Eggen", async ({ page }) => {
    await page.getByTestId("btn-customers").click();

    await page
      .locator("[data-testid=row-customer]", { hasText: "Knut Eggen" })
      .getByTestId("btn-edit")
      .click();
    page.on("dialog", (dialog) => dialog.accept());
    await page.getByTestId("btn-delete").click();

    const locator = page.getByTestId("row-customer");
    await expect(locator).toHaveCount(10);

    await expect(
      page.locator("data-testid=row-customer", { hasText: "Knut Eggen" })
    ).not.toBeVisible();
  });
});
```

</details>

## 8. Spot the error

The following test fails. Find out why and fix it.

```typescript
test("select the same country again", async ({ page }) => {
  await page.getByTestId("btn-customers").click();

  await page
    .locator(
      "[data-testid=row-customer] p:text('Hugo Brandt') >> .. >> mat-icon"
    )
    .click();
  await page.getByTestId("sel-country").click();
  await page.click("Austria");

  await page.getByTestId("btn-submit").click();
});
```

<details>

<summary>Solution</summary>

```typescript
test("select the same country again", async ({ page }) => {
  await page.getByTestId("btn-customers").click();

  await page
    .locator(
      "[data-testid=row-customer] p:text('Hugo Brandt') >> .. >> mat-icon"
    )
    .click();
  await page.getByTestId("sel-country").click();
  await page.locator("data-testid=opt-country >> text=Austria").click();

  await page.getByTestId("btn-submit").click();
});
```

</details>

## 9. User-Facing Selectors

Write two tests with user-facing selectors (`page.getByRole` & `page.getByLabel`).

The first test should select the holiday Firenze and request a brochure. Fill in the address and make sure that the message "Brochure sent" shows up.

The second test should rename the customer Latitia to Laetitia and verify that she doesn't show up in the cutomers list anymore.

<details>

<summary>Solution</summary>

```typescript
test.describe("user-facing selectors", () => {
  test("should request brochure for Firenze", async ({ page }) => {
    await page.getByRole("link", { name: "Holidays", exact: true }).click();
    await page
      .getByLabel(/Firenze/i)
      .getByRole("link", { name: "Get a Brochure" })
      .click();
    await page.getByLabel("Address").fill("Domgasse 5");
    await page.getByRole("button", { name: "Send" }).click();
    await page.getByRole("status");
  });

  test("should rename Latitia to Laetitia", async ({ page }) => {
    await page.getByRole("link", { name: "Customers", exact: true }).click();
    await page
      .getByLabel(/Latitia/i)
      .getByRole("link", { name: "Edit Customer" })
      .click();
    await expect(page.getByLabel("Firstname")).toHaveValue("Latitia");
    await page.getByLabel("Firstname").fill("Laetitia");
    await page.getByRole("button", { name: "Save" }).click();
    await expect(page.getByRole("link", { name: "Edit Customer" })).toHaveCount(
      10
    );
    await expect(page.getByLabel(/Latitia/)).toHaveCount(0);
  });
});
```

</details>
