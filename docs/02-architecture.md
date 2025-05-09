# 2. High-Level Architecture

## Overview

The application uses Next.js App Router as its foundation, providing several key benefits:

- Server components for improved performance
- Built-in routing and API endpoints
- Simplified data fetching and state management

```
┌─────────────────────────────┐
│  App Router (Next.js)       │
└───────┬─────────────────────┘
        │
        │ route-level code-splitting (dynamic import)
        ▼
┌────────────┐      ┌───────────────┐
│ /layout.tsx│─┬──▶│Context Providers│
└────────────┘ │   └───────┬────────┘
               │           │
               ▼           ▼
       ┌─────────────┐  ┌─────────────────┐
       │Landing Page │  │   /[module]     │
       └─────────────┘  └─────────────────┘
                        Each module folder = feature slice
```

## Key Architecture Decisions

1. **Next.js App Router** - Leverages server components and modern routing

   - Uses React Server Components for improved performance
   - Simplified data fetching with built-in async/await support
   - Streamlined API routes within the same directory structure

2. **Dynamic imports** - Code-split at the module level for optimal loading

   - Each feature module is loaded only when needed
   - Reduces initial bundle size for faster startup
   - Improves performance on lower-bandwidth connections

3. **Context providers** - Shared state accessible to all modules

   - Centralised state management with React Context
   - Avoids prop drilling through component trees
   - Provides configuration settings and user preferences

4. **Feature slices** - Independent modules with their own state, components, and logic
   - Self-contained functionality reduces coupling
   - Enables parallel development by different teams
   - Simplifies testing and maintenance

## Data Flow Architecture

The application follows a clear data flow pattern to maintain predictable state updates:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Data Sources   │────▶│  State Context  │────▶│    UI Render    │
│  (API, Local)   │     │  (React Context)│     │  (Components)   │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        ▲                                              │
        │                                              │
        └──────────────────────────────────────────────┘
                       User Interactions
```

## Module Structure

Each feature module follows a consistent structure:

```
/[module]/
  ├── components/        # UI components specific to this module
  ├── hooks/             # Custom React hooks
  ├── api/               # API handlers and data fetching
  ├── utils/             # Helper functions
  ├── types.ts           # TypeScript type definitions
  ├── constants.ts       # Module-specific constants
  ├── page.tsx           # Main entry point (Next.js page)
  └── layout.tsx         # Optional module-specific layout
```

## State Management Strategy

The application uses a hybrid state management approach:

1. **Global state** - Managed through React Context

   - User preferences and authentication
   - Application-wide configuration
   - Cross-module shared data

2. **Module state** - Local to each feature module

   - Module-specific data and UI state
   - Internal form state and validations
   - Temporary calculations and processing

3. **Server state** - Managed through React Query / SWR
   - API data fetching and caching
   - Optimistic updates for better UX
   - Background refetching and synchronization

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Giraffe Platform                           │
│  ┌───────────────┐          ┌────────────────┐                  │
│  │  Giraffe Map  │◀────────▶│ Giraffe SDK    │                  │
│  └───────────────┘          └────────┬───────┘                  │
│                                      │                          │
└──────────────────────────────────────│──────────────────────────┘
                                       │
                                       │ Integration
                                       │
┌──────────────────────────────────────▼──────────────────────────┐
│                                                                 │
│                      LandiQ Plus Frontend                       │
│                     (Next.js on Vercel)                         │
│                                                                 │
└───────────┬─────────────────────────────┬───────────────────────┘
            │                             │
            │                             │
            ▼                             ▼
┌────────────────────┐        ┌────────────────────────────┐
│                    │        │                            │
│  Flat Files        │        │      Cloudflare Proxy      │
│  (Data & Templates)│        │                            │
│                    │        └──────────────┬─────────────┘
└────────────────────┘                       │
                                             │
                                             ▼
                             ┌─────────────────────────────────┐
                             │    External API Services        │
                             │                                 │
                             │ ┌─────────────┐ ┌─────────────┐ │
                             │ │ ArcGIS REST │ │ Other APIs  │ │
                             │ │ Servers     │ │ e.g. permis.│ │
                             │ └─────────────┘ └─────────────┘ │
                             │                                 │
                             └─────────────────────────────────┘
                                             │
                                             │
            ┌─────────────────────────┐      │
            │                         │      │
            │  Supabase               │◀─────┘
            │  (Configuration Storage)│
            │                         │
            └─────────────────────────┘
```

### Integration Architecture

LandiQ Plus is integrated into the Giraffe platform and works with external services:

1. **Giraffe Platform Integration**

   - Application embedded within Giraffe via their SDK
   - User authentication handled by Giraffe
   - Interacts with Giraffe's map for visualisation
   - Access to Giraffe shortlists and site selection in a user's project in Giraffe

2. **Frontend Hosting**

   - Next.js application hosted on Vercel
   - Server components for efficient rendering
   - Client-side components for interactive features

3. **Data Sources**

   - Flat files for templates and static data
   - External API services for dynamic data:
     - ArcGIS REST servers (mapservers and featureservers) for spatial data
     - Other APIs e.g. permissibility servers for detailed property information
   - All external API calls managed via Cloudflare proxy

4. **Configuration Storage**
   - Supabase used to store user preferences
   - e.g. PowerPoint slide configurations for the export module
   - Minimal data footprint with user-specific settings

### Data Flow

The application follows these primary data flows:

1. **User Interaction**

   - User accesses LandiQ Plus within Giraffe platform
   - Giraffe manages overall access to the LandiQ Plus app
   - Access to individual modules within LandiQ Plus managed by LandiQ Plus itself (need admin module in-app)
   - User selects sites, areas, or shortlists from the Giraffe map (Mapbox based) or via the Land iQ Plus app which interacts with Giraffe (e.g. to find available shortlists) via the Giraffe SDK

2. **Data Retrieval**

   - App requests data through Cloudflare proxy
   - Proxy forwards requests to ArcGIS and other services
   - Results processed and prepared for visualisation or analysis

3. **Export & Output**
   - User configurations retrieved from Supabase
   - Combined with data from external services
   - Processed into outputs (PowerPoint, reports, etc.)

### Configuration Storage

```
┌─────────────────────────────────┐
│        User Preferences         │
├─────────────────────────────────┤
│ - user_email (primary key)      │
│ - powerpoint_templates          │
│ - default_views                 │
│ - export_configurations         │
└─────────────────────────────────┘
```

Key characteristics:

- Email used as identifier (from Giraffe SDK)
- PowerPoint slide configurations for exports
- Default view preferences for different modules
- User-specific configurations for analysis preferences

## Performance Optimisation Strategy

1. **Server Components** - Render expensive components on the server
2. **Streaming** - Progressive rendering of page content
3. **Image Optimization** - Automatic image sizing and formats
4. **Route Prefetching** - Preload likely navigation paths
5. **Edge Caching** - CDN caching of static assets and pages
