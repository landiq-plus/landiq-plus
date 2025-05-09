# 11. Short Test after Each Stage

A key part of our incremental approach is to verify each migration step before proceeding to the next. This ensures we catch issues early and maintain quality throughout the process.

## Testing Checkpoints

### After Each Feature Migration

After migrating each feature or component:

1. **Write a Playwright test**
   - Open the module route
   - Run the primary user action
   - Assert that the DOM contains expected elements
   - Test basic interactivity

```typescript
// Example test for the Imagery module
test("Imagery module allows viewing historical images", async ({ page }) => {
  // Navigate to the imagery module
  await page.goto("/imagery");

  // Select a location (pre-configured test location)
  await page.getByRole("button", { name: "Select Test Location" }).click();

  // Verify imagery loads
  await expect(page.locator(".imagery-viewer")).toBeVisible();

  // Test timeline navigation
  await page.getByRole("button", { name: "Next Image" }).click();

  // Verify year display updates
  await expect(page.getByTestId("year-display")).not.toHaveText("2023");
});
```

2. **Before continuing to the next feature**
   - Measure bundle size with `next build-analyze`
   - Ensure it hasn't ballooned compared to the previous build
   - Check for unnecessary duplicate dependencies

```bash
# Check bundle size
npm run build-analyze

# Output will show size changes since last build
# Look for unexpected large increases
```

### Post-Migration Integration Tests

After migrating multiple features:

1. **Test cross-module functionality**

   - Navigating between modules
   - Shared state persistence
   - Data passing between features

2. **Performance testing**
   - Initial load time
   - Time-to-interactive
   - Memory usage over time

## User Acceptance Testing

For critical features, conduct UAT with actual users before finalizing the migration:

1. Set up a staging environment with both old and new implementations
2. Allow users to try both and provide feedback
3. Address issues before proceeding with the full migration
