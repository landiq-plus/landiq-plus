# PowerPoint Template System

This document outlines the design and implementation of the PowerPoint template system in LandiQPlus.

## Overview

The PowerPoint generator allows users to create standardised PowerPoint presentations with property analysis content. The system provides predefined slide layouts that users can configure with specific data and map visualisations.

## Slide Layout System

### Predefined Layouts

The system offers a set of predefined slide layouts:

- **Cover Slide**: Title, subtitle, main image, logo
- **Property Overview**: Key stats, location map, aerial view
- **Zoning Analysis**: Zoning map, permitted uses, development controls
- **Flood Analysis**: Flood extent map, flood categorisation, risk assessment
- **Development Potential**: Developable area analysis, yield calculations
- **Site Constraints**: Composite constraints map, summary table
- **Valuation**: Valuation metrics, comparable sales
- **Executive Summary**: Key findings and recommendations

### Layout Configuration

Each layout has:

- Fixed structural elements (headers, positioning, branding)
- Configurable content areas (map frames, data tables, text boxes)
- Consistent styling across slides

## Map Screenshot Configuration

Users can configure screenshots for map frames with the following options:

### Layer Selection

- Base layers (aerial imagery, street map, topographic)
- Analysis layers (zoning, flooding, bushfire, etc.)
- User-drawn features (property boundaries, developable areas)

### Rendering Options

- Layer ordering (z-index for overlays)
- Transparency per layer (0-100%)
- Visibility toggles per layer
- Boundary styling (colour, width, style)
- Extent control (padding, fixed scale)

### Labelling

- Feature labels (on/off)
- Label styling (font, size, colour)
- Label positioning
- Legend inclusion

## Data Configuration

Users can configure data displayed in text areas:

- Property metadata (address, lot/DP, area)
- Planning controls (FSR, height, minimum lot size)
- Calculated metrics (developable area, yield)
- Custom text fields

## Scoring Logic

The system includes scoring logic for different aspects:

- Site suitability scores (0-100)
- Constraint severity ratings
- Development potential assessments
- Market opportunity evaluation

Scores can be visualised as:

- Numerical values
- Bar/radar charts
- Colour-coded indicators

## Implementation Approach

The template system will be implemented using:

- **PptxGenJS** for PowerPoint generation
- **React components** for the template configuration UI
- **JSON schema** for template definitions
- **Canvas API** for generating map screenshots with GIS layers
- Stored template configurations in Supabase

## Future Considerations

While custom user-uploaded templates are not supported initially, the architecture will allow for potential extension in the future.
