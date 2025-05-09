# 9. Testing & Quality Gates

A comprehensive testing strategy ensures code quality and prevents regressions.

## Quality Gates by Stage

| Stage    | Tool                                                         | Purpose                                      |
| -------- | ------------------------------------------------------------ | -------------------------------------------- |
| Lint     | ESLint + `@typescript-eslint` + Prettier (Husky pre-commit). | Enforce code style and catch common errors   |
| Unit     | Jest + RTL; 80% coverage per slice.                          | Test component and utility function behavior |
| e2e      | Playwright; basic smoke per module route.                    | Test user flows and integration              |
| Type     | `tsc --noEmit` in CI.                                        | Verify type safety                           |
| Security | `npm audit` + `eslint-plugin-security`.                      | Detect security vulnerabilities              |

## Testing Strategy

### Unit Tests

Each feature slice has its own test directory:

```
modules/localArea/
└─ __tests__/
   ├─ LocalArea.test.tsx     # Test main component
   ├─ controller.test.ts     # Test API controller
   └─ useLocalArea.test.ts   # Test custom hook
```

Unit tests focus on:

- Component rendering and user interactions
- Hook behavior and state management
- Utility function correctness
- API controller behavior with mocked responses

### End-to-End Tests

Playwright tests verify key user flows:

```ts
// tests/e2e/localArea.spec.ts
test("Local Area module loads and displays map", async ({ page }) => {
  await page.goto("/local-area");

  // Test navigation works
  await expect(
    page.getByRole("heading", { name: "Local Area Analysis" })
  ).toBeVisible();

  // Test map loads
  await expect(page.locator(".mapboxgl-map")).toBeVisible();

  // Test basic interaction
  await page.getByRole("button", { name: "Select Area" }).click();
  await expect(page.getByText("Draw an area on the map")).toBeVisible();
});
```

### CI Pipeline

The CI pipeline enforces quality gates:

1. Linting (eslint + prettier)
2. Type checking (tsc)
3. Unit tests (jest)
4. E2E tests (playwright)
5. Security audit (npm audit)

Only when all gates pass can code be merged to the main branch.
