# 13. Open Questions / Clarifications Needed

During the planning phase, several questions arose that require stakeholder input before implementation. This document tracks these questions and will be updated as decisions are made.

## Multi-tenant Theming

**Question**: Should modules support **multi-tenant theming** (per-organisation colours & logo)?

**Considerations**:

- Would allow different clients to have branded versions of the application
- Requires additional configuration and storage
- Could be implemented via CSS variables and theme context
- May need a theme editor UI

**Status**: ✅ Decided: Do not support multi-tenant theming for now

## Database Technology

**Question**: Do we keep Supabase or migrate to PlanetScale for non-spatial data?

**Considerations**:

- Supabase offers auth, storage, and database in one platform
- PlanetScale may offer better performance for relational data
- Migration would require data export/import and API changes
- Need to evaluate cost implications

**Status**: ✅ Decided: Use Supabase for database

## Offline Capability

**Question**: Required offline-capability (PWA) for field use?

**Considerations**:

- Would require service worker implementation
- Need to determine what data needs to be available offline
- Sync strategy for offline changes
- Increases complexity significantly

**Status**: ✅ Decided: Do not require offline capability

## PowerPoint Templates

**Question**: Authoring PPTX templates – will users upload their own `.pptx` sample?

**Considerations**:

- User-uploaded templates add flexibility
- Requires template parsing and validation
- May need a template editor or preview
- Could start with predefined templates and add user uploads later

**Status**: ✅ Decided: System will define available slide layouts and options

**Implementation Details**:

- Predefined slide layouts for users to select from
- Users can configure which map layers appear in screenshots:
  - Selection of layer types (aerial imagery, zoning, etc.)
  - Control of rendering order
  - Transparency settings per layer
  - Padding and boundary options
  - Label configuration
- Text data fields configurable for text boxes on slides
- Scoring logic defined by system to score different aspects of slides

## Resolution Process

These questions have been resolved through stakeholder input. Implementation details will be reflected in the relevant technical documentation sections.
