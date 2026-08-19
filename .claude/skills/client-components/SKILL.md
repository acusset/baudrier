---
name: client-components
description: Use this skill when you get the error message saying the component should mention "use client" in order to be rendered
---

## Rules

- The goal is to use Server Components as much as possible
- Client components should be atomic (like Buttons, Link etc.), not full Pages or Layouts.
- Client components should be deep inside the rendering tree. We should avoid adding it at a page level.
- Adding "use client" is not the magic solution to fix rendering issues

## Actions

Find the component in the page or layout or other large component that requires interactivity (onClick, onChange etc.) and to isolate it on it's own component.
