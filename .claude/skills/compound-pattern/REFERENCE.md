# Compound Pattern — Reference

## JavaScript template

```jsx
import { createContext, useContext, useState } from 'react';

const RootContext = createContext(null);

function useRootContext() {
  const ctx = useContext(RootContext);
  if (!ctx) throw new Error('<Root.X> must be used inside <Root>');
  return ctx;
}

function Root({ children }) {
  const [open, setOpen] = useState(false);
  return (
    <RootContext.Provider value={{ open, setOpen }}>
      {children}
    </RootContext.Provider>
  );
}

function Toggle() {
  const { open, setOpen } = useRootContext();
  return <button onClick={() => setOpen(o => !o)}>{open ? 'Close' : 'Open'}</button>;
}
Toggle.displayName = 'Root.Toggle';

function Panel({ children }) {
  const { open } = useRootContext();
  return open ? <div>{children}</div> : null;
}
Panel.displayName = 'Root.Panel';

const Compound = Object.assign(Root, { Toggle, Panel });
export default Compound;
```

## TypeScript template

```tsx
import { createContext, useContext, useState } from 'react';

interface RootContextValue {
  open: boolean;
  setOpen: React.Dispatch<React.SetStateAction<boolean>>;
}

const RootContext = createContext<RootContextValue | null>(null);

function useRootContext(): RootContextValue {
  const ctx = useContext(RootContext);
  if (!ctx) throw new Error('<Root.X> must be used inside <Root>');
  return ctx;
}

interface RootProps { children: React.ReactNode }

function Root({ children }: RootProps) {
  const [open, setOpen] = useState(false);
  return (
    <RootContext.Provider value={{ open, setOpen }}>
      {children}
    </RootContext.Provider>
  );
}

function Toggle() {
  const { open, setOpen } = useRootContext();
  return <button onClick={() => setOpen(o => !o)}>{open ? 'Close' : 'Open'}</button>;
}
Toggle.displayName = 'Root.Toggle';

interface PanelProps { children: React.ReactNode }

function Panel({ children }: PanelProps) {
  const { open } = useRootContext();
  return open ? <div>{children}</div> : null;
}
Panel.displayName = 'Root.Panel';

const Compound = Object.assign(Root, { Toggle, Panel });
export default Compound;
```

> Use `Object.assign` rather than assigning to `Root.Toggle` — it avoids TypeScript errors about properties not existing on the function type.

## Before / after example

### Before — monolithic `Dropdown`

```tsx
<Dropdown
  label="Options"
  items={items}
  onSelect={handleSelect}
  isOpen={open}
  onToggle={setOpen}
  renderItem={(item) => <span>{item.label}</span>}
/>
```

```tsx
function Dropdown({ label, items, onSelect, isOpen, onToggle, renderItem }) {
  return (
    <div>
      <button onClick={onToggle}>{label}</button>
      {isOpen && (
        <ul>
          {items.map(item => (
            <li key={item.id} onClick={() => onSelect(item)}>
              {renderItem(item)}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### After — compound `Dropdown`

```tsx
<Dropdown>
  <Dropdown.Toggle>Options</Dropdown.Toggle>
  <Dropdown.List>
    {items.map(item => (
      <Dropdown.Item key={item.id} onSelect={handleSelect} item={item}>
        {item.label}
      </Dropdown.Item>
    ))}
  </Dropdown.List>
</Dropdown>
```

## Common pitfalls

| Pitfall | Fix |
|---|---|
| Context consumer wrapped in a stray `<div>` breaks access | Remind the user: context works across any depth, but `React.Children.map` only sees direct children — stick to Context API |
| Sub-component used outside root silently returns `undefined` | Guard with the `useRootContext` null-check throw |
| TS error: property does not exist on function type | Use `Object.assign` instead of direct property assignment |
| Multiple compound roots nested (e.g. nested dropdowns) | Each root creates its own context instance — this works correctly with the Context API approach |
