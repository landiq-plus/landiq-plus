# LandiQ Plus Modules

This directory contains documentation for each of the feature modules in the LandiQ Plus application. Each module is designed as a self-contained feature slice with its own components, state management, and business logic.

## Available Modules

| Module                          | Description                          | Primary Use Cases                       |
| ------------------------------- | ------------------------------------ | --------------------------------------- |
| [Local Area](./localArea.md)    | Area-based analysis and reporting    | Context analysis, planning studies      |
| [Development](./development.md) | Development potential assessment     | Feasibility studies, yield testing      |
| [Land Use](./landUse.md)        | Land use analysis and permissibility | Zoning review, highest and best use     |
| [Imagery](./imagery.md)         | Aerial imagery viewing and analysis  | Site context, historical changes        |
| [Feasibility](./feasibility.md) | Financial feasibility calculations   | Development viability, optimization     |
| [Valuation](./valuation.md)     | Property valuation estimation        | Market assessment, acquisition pricing  |
| [Shortlist](./shortlist.md)     | Multi-property batch analysis        | Portfolio assessment, site selection    |
| [Open Space](./openSpace.md)    | Open space accessibility analysis    | Planning studies, gap identification    |
| [Merge](./merge.md)             | Parcel amalgamation analysis         | Site assembly, development optimization |
| [PPTX](./pptx.md)               | PowerPoint report generation         | Reporting, presentation creation        |

## Module Architecture

Each module follows the feature slice architecture described in the main architecture document. This means that each module:

1. Is **lazy-loaded** via dynamic imports (only loaded when needed)
2. Is **self-contained** with its own components, hooks, and logic
3. Has **clear boundaries** with defined inputs and outputs
4. Is **independently testable** with its own test suite
5. Manages its own **internal state** while using shared global state as needed

## Module Integration

Modules integrate with each other and with the Giraffe platform through:

- **GiraffeContext** - For access to Giraffe SDK functionality
- **ConfigContext** - For accessing user preferences and configurations
- **Shared services** - For common functionality like API access
- **Supabase storage** - For persisting user preferences and configurations

## Adding New Modules

To add a new module to the system:

1. Create a new directory under `src/modules/`
2. Implement the standard module structure (see architecture docs)
3. Add the module to the `MODULES` array in `src/config/modules.ts`
4. Create a documentation file in `docs/modules/`
5. Add appropriate tests in the module's `__tests__` directory
