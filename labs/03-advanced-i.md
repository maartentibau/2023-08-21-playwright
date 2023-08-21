- [1. Page Objects](#1-page-objects)
- [2. Test fixtures](#2-test-fixtures)
- [3. More Page Objects Functionality](#3-more-page-objects-functionality)
- [4. Nested `describe`](#4-nested-describe)

Checkout the branch _pw-03-starter_. The solution branch is _pw-03-solution_. If you managed to finish the former exercise successfully, you don't have to switches branches.

## 1. Page Objects

Create page objects `CustomersPage` and `CustomerPage`. `CustomersPage` should provide methods to add and edit a customer. `CustomerPage` should provide methods to fill in data to the form fields and a method that submits the form.

Use the newly created page objects in your test. You have to instantiate them every test manually.

Since Angular Material is optimised for accessibility, **use only user-facing selectors**.

<details>

<summary>Solution</summary>

**tests/page-objects/customers-page.ts**

```typescript
import { Page } from "@playwright/test";

export class CustomersPage {
  constructor(private page: Page) {}

  async add(): Promise<void> {
    await this.page.getByRole("link", { name: "Add Customer" }).click();
  }

  async edit(name: string): Promise<void> {
    await this.page
      .getByLabel(name)
      .getByRole("link", { name: "Edit" })
      .click();
  }
}
```

**tests/page-objects/customer-page.ts**

```typescript
import { Page } from "@playwright/test";

interface CustomerData {
  firstname?: string;
  lastname?: string;
  birthday?: Date;
  country?: string;
}

export class CustomerPage {
  constructor(private page: Page) {}

  async fillIn(customerData: CustomerData) {
    if (customerData.firstname !== undefined) {
      await this.page.getByLabel("Firstname").fill(customerData.firstname);
    }

    if (customerData.lastname !== undefined) {
      await this.page
        .getByLabel("Name", { exact: true })
        .fill(customerData.lastname);
    }

    if (customerData.birthday !== undefined) {
      const day = customerData.birthday.getDate();
      const month = customerData.birthday.getMonth() + 1;
      const year = customerData.birthday.getFullYear();
      await this.page.getByLabel("Birthdate").fill(`${day}.${month}.${year}`);
    }

    if (customerData.country !== undefined) {
      await this.page.getByLabel("Country").click();
      await this.page
        .getByRole("option", { name: customerData.country })
        .click();
    }
  }

  async submit() {
    await this.page.getByRole("button", { name: "Save" }).click();
  }
}
```

</details>

## 2. Test fixtures

Integrate test fixtures for the `CustomersPage` and `CustomerPage` into your `test`. This will instantiate the page objects automatically and on-demand and will make your code shorter.

Small hint:

```typescript
import { expect, test as base } from "@playwright/test";

const test = base.extend<>(); // ....
```

## 3. More Page Objects Functionality

Extend the test fixtures and page objects, so that they provide

- selectors for the customers list,
- be able to delete a customer,
- selector for row count, and
- a new page object along fixture for the sidemenu

The solution is available in the branch `_pw-03-solution_`.

## 4. Nested `describe`

You can nest the `describe` commmands. For example, you could have another `beforeEach` for the customer-related tests, where you click on the customers button.

The solution is available in the branch `_pw-03-solution_`.
