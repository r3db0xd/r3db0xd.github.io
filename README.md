# Rishab Dhar — Cloud Architecture Portfolio

A high-performance, zero-dependency interactive portfolio and digital resume. 

As a Senior Cloud Systems Engineer, I built this site to reflect my core engineering principles: **lightweight, fast, and highly resilient.** By entirely avoiding heavy JavaScript frameworks (React, Vue, etc.) in favor of pure, native web technologies, this project achieves instant load times and fluid 60fps animations while handling complex 3D mathematics in the DOM.

👉 **[Live Deployment](https://r3db0xd.github.io)**

---

## 🏗️ Technical Architecture & Deep Dive

This project relies on strict separation of concerns, utilizing modern CSS rendering engines and vanilla JavaScript for state management.

### 1. The Rendering Engine (CSS 3D + 2D Sync)
One of the core engineering challenges was bypassing WebKit/Chromium rendering bugs that distort text when placed inside 3D transforms.
* **The Problem:** Hardware-accelerated `rotateX` and `rotateZ` transforms physically squash child text nodes, making them unreadable.
* **The Solution (Decoupled Overlays):** The 3D isometric glass leaves (APP, K8S, GCP) exist in a purely aesthetic `transform-style: preserve-3d` stack. The interactive pop-ups and text elements live in a completely isolated, purely 2D `z-index: 1000` overlay. 
* **The Math:** When a user hovers, JavaScript triggers a synchronized CSS transition that applies the exact mathematical inverse (`rotateZ(45deg) rotateX(-60deg)`) to the 2D elements, creating the illusion of a sticker peeling perfectly flat off a 3D surface with zero text distortion.

### 2. Autonomous State Management
* **Fluid Carousel:** A custom JavaScript array-shifting algorithm rotates the 3D stack (`cycleIsoStack()`) autonomously every 4 seconds, updating class states to manage depth (`translateZ`) and opacity without blocking the main thread.
* **Dynamic Canvas Engines:** The background features two distinct HTML5 `<canvas>` engines tied to the theme state:
  * **Dark Mode:** A localized Matrix-rain physics engine.
  * **Light Mode:** An ambient, auto-typing code snippet generator using `requestAnimationFrame`.

### 3. Responsive DOM Pipeline
* **Viewport Lockdown:** Strict `overflow-x: hidden` and dynamic `vw` padding ensures the layout never breaks the X-axis across mobile devices.
* **Tactile Fallbacks:** 3D hover states gracefully degrade to native tap-to-toggle (`classList.toggle('active')`) logic on mobile viewports (<600px).
* **Intersection Observers:** Scroll-based animations (like the staggered experience bullet points and the terminal auto-typing effect) are lazy-loaded via the `IntersectionObserver` API to conserve CPU cycles until the elements actually enter the viewport.

---

## ⚙️ Systems Flow Diagram

The following diagram illustrates the event-driven architecture and rendering pipeline:

```mermaid
graph TD
    %% User Inputs
    User((User Event)) --> |Hover / Tap| UI[UI Interaction Layer]
    User --> |Scroll| Obs[Intersection Observer]
    User --> |Click| Theme[Theme Toggle]

    %% UI Interaction Flow
    UI --> |Trigger Hover State| CSS3D[CSS 3D Engine]
    CSS3D --> |Lift Glass Layer| Z[TranslateZ Transformation]
    UI --> |Sync 2D Overlay| CSS2D[CSS 2D Inverse Math]
    CSS2D --> |Peel Effect| Flat[rotateX/rotateZ Inverse]

    %% Observer Flow
    Obs --> |Intersect: Terminal| Term[Terminal JS Engine]
    Term --> |Character Print Loop| DOM1[DOM Text Injection]
    Obs --> |Intersect: Lists| Stag[Staggered Fade Logic]
    Stag --> DOM2[CSS Opacity Transition]

    %% Theme Flow
    Theme --> State{State Manager}
    State --> |Dark Mode| CanvasD[Load Matrix Canvas]
    State --> |Light Mode| CanvasL[Load Code Canvas]
    State --> CSSVar[Update CSS Custom Properties]

    %% Render
    Z --> Render((Browser Paint))
    Flat --> Render((Browser Paint))
    DOM1 --> Render((Browser Paint))
    DOM2 --> Render((Browser Paint))
    CanvasD --> Render((Browser Paint))
    CanvasL --> Render((Browser Paint))
    CSSVar --> Render((Browser Paint))

    classDef core fill:#020402,stroke:#00FF41,stroke-width:2px,color:#fff;
    classDef action fill:#1E40AF,stroke:#2563EB,stroke-width:2px,color:#fff;
    class Theme,Obs,UI action;
    class Render core;