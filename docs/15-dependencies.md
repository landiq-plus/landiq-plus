# 15. Dependencies & Packages

This is a comprehensive list of all packages we'll use, organized by category. These dependencies have been carefully selected for their maintainability, performance, and compatibility.

## Core & Framework

| Package    | Version | Purpose                              |
| ---------- | ------- | ------------------------------------ |
| next       | ^14.0.0 | React framework with App Router      |
| react      | ^18.2.0 | UI library                           |
| react-dom  | ^18.2.0 | DOM bindings for React               |
| typescript | ^5.3.0  | Type checking & developer experience |
| eslint     | ^8.56.0 | Code quality & consistency           |
| prettier   | ^3.1.0  | Code formatting                      |

## UI & Presentation

| Package           | Version  | Purpose                     |
| ----------------- | -------- | --------------------------- |
| tailwindcss       | ^3.4.0   | Utility-first CSS framework |
| postcss           | ^8.4.0   | CSS processing              |
| framer-motion     | ^11.0.0  | Animation library for React |
| lucide-react      | ^0.312.0 | Icon library                |
| react-hook-form   | ^7.50.0  | Form state management       |
| date-fns          | ^3.2.0   | Date manipulation           |
| @headlessui/react | ^1.7.18  | Accessible UI components    |

## GIS & Mapping

| Package      | Version | Purpose                            |
| ------------ | ------- | ---------------------------------- |
| @turf/turf   | ^6.5.0  | Advanced geospatial analysis in JS |
| @arcgis/core | ^4.28.0 | ESRI ArcGIS JavaScript SDK         |
| proj4        | ^2.9.0  | Coordinate reference system tools  |
| leaflet      | ^1.9.4  | Interactive maps for local area    |

## State Management & Data

| Package               | Version | Purpose                                       |
| --------------------- | ------- | --------------------------------------------- |
| zod                   | ^3.22.0 | Schema validation with TypeScript integration |
| @tanstack/react-query | ^5.17.0 | Data fetching & server state                  |
| @supabase/supabase-js | ^2.39.0 | Supabase client for auth/data                 |
| zustand               | ^4.4.0  | Simple state management                       |
| immer                 | ^10.0.3 | Immutable state updates                       |

## Document Generation

| Package         | Version  | Purpose                           |
| --------------- | -------- | --------------------------------- |
| pptxgenjs       | ^3.12.0  | Create PowerPoint presentations   |
| jspdf           | ^2.5.1   | PDF generation                    |
| html-to-image   | ^1.11.11 | Convert HTML to images (for docs) |
| chart.js        | ^4.4.1   | Chart generation for reports      |
| react-chartjs-2 | ^5.2.0   | React wrapper for Chart.js        |
| exceljs         | ^4.4.0   | Excel file manipulation           |

## Testing

| Package                     | Version | Purpose                      |
| --------------------------- | ------- | ---------------------------- |
| jest                        | ^29.7.0 | JavaScript testing framework |
| @testing-library/react      | ^14.1.2 | React component testing      |
| @testing-library/user-event | ^14.5.2 | User event simulation        |
| @playwright/test            | ^1.40.0 | End-to-end testing           |
| msw                         | ^2.0.12 | API mocking for tests        |

## Utilities

| Package                   | Version  | Purpose                           |
| ------------------------- | -------- | --------------------------------- |
| uuid                      | ^9.0.1   | Generate unique IDs               |
| lodash-es                 | ^4.17.21 | Utility functions (tree-shakable) |
| nanoid                    | ^5.0.4   | Small, secure ID generator        |
| client-zip                | ^2.4.4   | Create ZIP files in the browser   |
| deepmerge                 | ^4.3.1   | Deep merge objects                |
| browser-image-compression | ^2.0.2   | Compress images client-side       |

## Development

| Package           | Version  | Purpose                             |
| ----------------- | -------- | ----------------------------------- |
| husky             | ^8.0.3   | Git hooks                           |
| lint-staged       | ^15.2.0  | Run linters on staged files         |
| turbo             | ^1.12.0  | High-performance build system       |
| @vercel/analytics | ^1.1.1   | Vercel usage analytics              |
| sentry            | ^7.94.1  | Error tracking                      |
| @swc/core         | ^1.3.102 | Fast JavaScript/TypeScript compiler |

This dependency list is a starting point and may evolve as development progresses. All packages will be evaluated for bundle size impact, maintenance status, and security posture before final inclusion.

## Dependency Management

To maintain a clean dependency tree:

1. Use `npm-check` regularly to identify unused or outdated dependencies
2. Prefer ESM-compatible packages for better tree-shaking
3. Avoid packages with large transitive dependency graphs
4. Lock versions using a lockfile (package-lock.json)
5. Run security audits (`npm audit`) regularly
