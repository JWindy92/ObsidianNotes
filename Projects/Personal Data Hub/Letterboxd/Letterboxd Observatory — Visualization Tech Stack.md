**Goal:** Build a bespoke, professional-quality data visualization dashboard inspired by editorial data journalism and cinematic interfaces. Priorities: dense but ergonomic layouts, custom graphics, meaningful animation, polished styling, and interactive exploration.

## Core Stack

### 1. D3.js — Custom Visualizations

**Role:** Core visualization engine.

- Custom charts, scales, axes, and data encodings
    
- SVG graphics, data-driven styling, and animated transitions
    
- Interactive timelines, network graphs, radial layouts, and rating plots
    
- Custom selections, hover effects, and filtering  
    **Use for:** Distinctive visualizations that go beyond standard chart templates.  
    Docs: [https://d3js.org/](https://d3js.org/)
    

### 2. React + TypeScript — Application Framework

**Role:** Dashboard structure and application state.

- Reusable components, navigation, filters, and detail panels
    
- Shared state across coordinated charts
    
- Type-safe data contracts  
    **Use for:** The application surrounding the visualizations.  
    Docs: [https://react.dev/](https://react.dev/) · [https://www.typescriptlang.org/](https://www.typescriptlang.org/)
    

### 3. SVG + CSS — Graphics and Styling

**Role:** Precise visual control and polish.

- Gradients, masks, clipping paths, custom shapes, and typography
    
- Responsive layouts, hover states, and transitions  
    **Use for:** Building a cohesive visual language and crisp, scalable graphics.  
    Docs: [https://developer.mozilla.org/en-US/docs/Web/SVG](https://developer.mozilla.org/en-US/docs/Web/SVG) · [https://developer.mozilla.org/en-US/docs/Web/CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)
    

### 4. Motion — Interface Animation

**Role:** React-oriented animation library.

- Animated entrances/exits and layout transitions
    
- Staggered animations, modal transitions, and coordinated movement  
    **Use for:** Interface animation; use D3 for data-driven chart transitions.  
    Docs: [https://motion.dev/](https://motion.dev/)
    

### 5. Three.js — Advanced Graphics (Optional)

**Role:** 3D graphics using WebGL.

- 3D film networks, particle systems, spatial exploration, and camera movement  
    **Use for:** Experiments that genuinely benefit from 3D. Defer until the 2D foundation is solid.  
    Docs: [https://threejs.org/](https://threejs.org/)
    

## Supporting Technologies

|Layer|Technology|Purpose|
|---|---|---|
|Frontend|React + TypeScript|Application and components|
|Visualization|D3.js|Custom charts and interactions|
|Graphics|SVG + CSS|Vector graphics and styling|
|Animation|Motion|Interface transitions|
|Data processing|Python + pandas|Transform and analyze Letterboxd data|
|Storage|SQLite|Local analytical data store|
|API|FastAPI (optional)|Serve data to the frontend|
|Build tooling|Vite|Frontend development|
|Deployment|Docker|Self-host on home server|

## Initial Project: Animated Film Timeline

- Chronological timeline of watched films
    
- Marker size or color based on personal rating
    
- Year and decade bands
    
- Hoverable film details and poster previews
    
- Filters for year, genre, and rating
    
- Smooth transitions when filtering
    
- Responsive layout and deliberate dark theme
    

## Learning Sequence

1. Establish layout, typography, spacing, and color system.
    
2. Build a static timeline using D3 and SVG.
    
3. Add scales, axes, labels, and rating-dependent markers.
    
4. Implement hover interactions and film detail panels.
    
5. Add filtering and animated transitions.
    
6. Optimize responsiveness and performance.
    
7. Reuse the design language for heatmaps, rating distributions, and taste networks.
    

## Design Principles

- Favor visual hierarchy over decoration.
    
- Use animation to explain changes and relationships.
    
- Keep dense interfaces readable through spacing, contrast, and progressive disclosure.
    
- Use standard chart libraries only when they meet the design requirements.
    
- Use D3 for distinctive custom graphics.
    
- Separate data ingestion and transformation from frontend presentation.
    

## Architecture

`Letterboxd export → Python ingestion/transformation → SQLite or JSON → React + TypeScript → D3/SVG visualizations`

**Guiding principle:** Build a personal data-journalism product, not just a dashboard filled with charts.