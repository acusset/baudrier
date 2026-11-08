# Reference: Worked Examples

## Example A — route-level `layout.tsx` (siblings exist)

`app/dashboard/page.tsx` mixes dashboard chrome with the overview content, and `app/dashboard/settings/page.tsx` already exists as a sibling that would want the same chrome:

```tsx
// Before: app/dashboard/page.tsx
export default function Page() {
  return (
    <div className="dashboard-shell">
      <header className="dashboard-header">
        <span>Acme</span>
        <nav>
          <a href="/dashboard">Overview</a>
          <a href="/dashboard/settings">Settings</a>
        </nav>
      </header>
      <main>
        <h1>Overview</h1>
        <p>Welcome back.</p>
      </main>
    </div>
  );
}
```

Everything from the outer `<div>` through `</header>` and the `<main>` opening tag is generic dashboard chrome — it belongs to the section, not to the overview content. Siblings exist, so it becomes a real `layout.tsx`:

```tsx
// After: app/dashboard/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard-shell">
      <header className="dashboard-header">
        <span>Acme</span>
        <nav>
          <a href="/dashboard">Overview</a>
          <a href="/dashboard/settings">Settings</a>
        </nav>
      </header>
      <main>{children}</main>
    </div>
  );
}
```

```tsx
// After: app/dashboard/page.tsx
export default function Page() {
  return (
    <>
      <h1>Overview</h1>
      <p>Welcome back.</p>
    </>
  );
}
```

Next.js wires the wrapping automatically — `page.tsx` needs no import, and `app/dashboard/settings/page.tsx` now gets the same header/nav for free.

## Example B — co-located component (no siblings)

`app/page.tsx` is a one-off landing page. It has real structural chrome (a full-bleed background wrapper and a centered card), but nothing else lives under `app/`, so a `layout.tsx` here would just be a same-effect detour through the special-file mechanism for a single consumer:

```tsx
// Before: app/page.tsx
export default function Home() {
  return (
    <div className="landing-shell">
      <main className="landing-card">
        <h1>Welcome</h1>
        <p>Get started by editing this page.</p>
      </main>
    </div>
  );
}
```

The outer `<div>` and the `<main>`'s shell classes (width/centering/padding/background) are layout; the heading and copy are content:

```tsx
// After: app/home-layout.tsx
export default function HomeLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="landing-shell">
      <main className="landing-card">{children}</main>
    </div>
  );
}
```

```tsx
// After: app/page.tsx
import HomeLayout from "./home-layout";

export default function Home() {
  return (
    <HomeLayout>
      <h1>Welcome</h1>
      <p>Get started by editing this page.</p>
    </HomeLayout>
  );
}
```

Here `page.tsx` does the wrapping explicitly, because there's no `layout.tsx` file convention doing it automatically.

## Don't extract when there's nothing structural

If the "wrapper" is a single unstyled `<div>` with no layout-bearing classes, splitting it out is churn, not clarity — leave it and say so.
