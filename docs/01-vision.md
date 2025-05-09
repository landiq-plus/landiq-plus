# 1. Vision

Build a **lean, highly-configurable React TypeScript web application** that re-uses the existing domain logic (Local Area, Development, Land-Use, Imagery, Feasibility, Valuation, Shortlist, Open Space, GIS utilities, Merge, PPTX generation) but exposes every module as an independent plug-and-play feature.

## Key Objectives

• Streamline the codebase – abolish 3 000-line "God" files.  
• Adopt a **feature-slice architecture** to localise concerns and promote lazy loading.  
• Centralise data-shape definitions and external endpoint references.  
• Expose a simple **configuration object** so non-developers can decide which modules show up and how they are grouped in the UI.  
• Ship an SPA with a landing page (card grid) and a persistent top-right dropdown for quick navigation.  
• Write everything in **TypeScript**, React 18, Next 14 (App Router), Shadcn, Tailwind CSS.
