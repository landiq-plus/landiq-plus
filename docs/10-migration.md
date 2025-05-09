# 10. Incremental Migration Strategy

Rather than a full rewrite, we'll migrate the existing app incrementally, minimizing risk and allowing for continuous delivery of value.

## Migration Phases

### Phase 1: Extract Common Utilities

1. **Extract PPTX helpers** (`src/utils/pptx/*`) – stateless.
   - Move to new folder structure
   - Convert to TypeScript
   - Add unit tests
   - Create Zod schemas for input/output validation
   - **Add configuration layer** for user-defined templates and formats

### Phase 2: First Feature Slice

2. **Carve out Imagery module** first
   - Smallest surface area
   - Relatively self-contained
   - Create module folder structure
   - Extract business logic into hooks
   - Write Zod schemas & tests

### Phase 3: Core Navigation

3. **Port Local Area module** next
   - Follow feature-slice blueprint
   - Create pure components
   - Extract domain logic to hooks

### Phase 4: Configuration System

4. **Build user configuration system**
   - Create configuration data models and storage
   - Implement configuration UI components
   - Design user-friendly configuration interfaces for:
     - PPTX report generator (slide templates, scoring logic)
     - Local Area modal (benchmark metrics e.g. required primary schools per X people)
     - Land Use modal (land use parameters - area, width, distance to public transport, distance to open space, bushfire, flood, max slope etc)
     - Site reviewer (assessment fields, evaluation criteria, dashboard configs)
     - Feasibility calculator (development metrics, financial assumptions, dwelling parameters)
   - Secure configuration with role-based permissions
   - Add validation and safe defaults

### Phase 5: Landing Page & Navigation

5. **Build landing page & nav** once configuration and slices exist
   - Implement module configuration
   - Create navigation components
   - Test navigation between modules
   - Add configuration access points in UI

### Phase 6: Complete Migration

6. **Decommission old monolith folders** once parity reached
   - Migrate remaining modules
   - Redirect old routes
   - Remove legacy code
   - Ensure all modules support user configuration

## Configuration Architecture

The new architecture will support user configuration in key areas:

1. **PPTX Generator**

   - User-defined slide templates
   - User-added custom logos
   - Customisable scoring logic and thresholds
   - Template libraries with sharing capabilities

2. **Land Use Modal**

   - Configurable parameters for each land use type
   - Minimum area and other constraint settings
   - Custom evaluation criteria

3. **Site Reviewer**

   - User-defined assessment fields
   - Customizable evaluation methodologies
   - Configurable display options and reporting

4. **Feasibility Calculator**
   - Development metric presets (building footprint, floor heights, efficiency ratios)
   - Construction cost templates by development type and LGA
   - Financial assumption profiles (interest rates, fees, profit margins)
   - Dwelling size and price templates
   - Social/affordable housing scenario settings
   - Custom control parameter sets
   - LMR (Low Mid Rise) housing configuration options
   - Exportable calculation templates

## Compatibility Strategy

During migration, the new app will run alongside the old one, allowing for:

- A/B testing of new modules
- Gradual user migration
- Fallback to old implementation if issues arise
- Testing of configuration capabilities with select users
