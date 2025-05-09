# 7. Navigation

The application uses a consistent navigation system that derives its structure from the centralized module configuration.

## Primary Navigation Components

### Navbar (Fixed Top-Right)

- **components/navigation/Navbar.tsx** - Main navigation container
- **components/navigation/ModuleDropdown.tsx** - Dropdown listing enabled modules grouped by `group`
- Modules are displayed in their respective groups (Analysis, Reporting, Utilities)
- Only enabled modules appear in the dropdown

### Landing Cards

- **components/navigation/LandingCards.tsx** - Grid of cards on the home page
- Each card represents an enabled module from the `MODULES` configuration
- Clicking a card navigates to that module
- Cards show the module icon, label, and a brief description

## Implementation Details

Both navigation components derive their content from the same configuration source (`config/modules.ts`), ensuring a single source of truth:

```tsx
// Example of landing page implementation
import { MODULES } from "@/config/modules";

export default function LandingPage() {
  const enabledModules = MODULES.filter((module) => module.enabled);

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {enabledModules.map((module) => (
        <LandingCard
          key={module.key}
          icon={module.icon}
          label={module.label}
          href={module.route}
          group={module.group}
        />
      ))}
    </div>
  );
}
```

## Accessibility

Navigation components implement full keyboard navigation support and appropriate aria attributes for screen readers.
