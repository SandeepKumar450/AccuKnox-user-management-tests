# AccuKnox-user-management-tests
import { test, expect } from '@playwright/test';

test.describe('OrangeHRM User Management E2E', () => {
  const adminUsername = 'Admin';
  const adminPassword = 'admin123';
  const newUsername = 'TestUser123';
  const editedUsername = 'UpdatedUser123';
  const employeeName = 'Sandeepsreeram';

  test.beforeEach(async ({ page }) => {
    await page.goto('/');
    await page.getByPlaceholder('Username').fill(adminUsername);
    await page.getByPlaceholder('Password').fill(adminPassword);
    await page.getByRole('button', { name: 'Login' }).click();
    await expect(page).toHaveURL(/dashboard/);
  });

  test('Add, Search, Edit, Validate, and Delete a User', async ({ page }) => {
    // Navigate to Admin > User Management
    await page.getByRole('menuitem', { name: 'Admin' }).click();
    await expect(page.locator('h6')).toHaveText('System Users');

    // Add User
    await page.getByRole('button', { name: 'Add' }).click();
    await page.getByLabel('User Role').click();
    await page.getByRole('option', { name: 'ESS' }).click();

    await page.getByPlaceholder('Type for hints...').fill(employeeName);
    await page.waitForTimeout(1000); // Wait for suggestions
    await page.keyboard.press('ArrowDown');
    await page.keyboard.press('Enter');

    await page.getByLabel('Status').click();
    await page.getByRole('option', { name: 'Enabled' }).click();

    await page.getByLabel('Username').fill(newUsername);
    await page.getByLabel('Password').fill('Password123!');
    await page.getByLabel('Confirm Password').fill('Password123!');

    await page.getByRole('button', { name: 'Save' }).click();
    await expect(page.locator('div.oxd-toast')).toHaveText(/Success/);

    // Search User
    await page.getByPlaceholder('Username').fill(newUsername);
    await page.getByRole('button', { name: 'Search' }).click();
    await expect(page.locator('div.oxd-table-cell').first()).toContainText(newUsername);

    // Edit User
    await page.getByRole('button', { name: 'Edit' }).click();
    const usernameInput = page.getByLabel('Username');
    await usernameInput.fill('');
    await usernameInput.fill(editedUsername);
    await page.getByRole('button', { name: 'Save' }).click();
    await expect(page.locator('div.oxd-toast')).toHaveText(/Success/);

    // Validate Edited User
    await page.getByPlaceholder('Username').fill(editedUsername);
    await page.getByRole('button', { name: 'Search' }).click();
    await expect(page.locator('div.oxd-table-cell').first()).toContainText(editedUsername);

    // Delete User
    await page.getByRole('button', { name: 'Delete' }).click();
    await page.getByRole('button', { name: 'Yes, Delete' }).click();
    await expect(page.locator('div.oxd-toast')).toHaveText(/Successfully Deleted/);
  });

  test.afterEach(async ({ page }) => {
    // Logout
    await page.getByRole('img', { name: 'profile picture' }).click();
    await page.getByText('Logout').click();
    await expect(page).toHaveURL(/auth\/login/);
  });
});
