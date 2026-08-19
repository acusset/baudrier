# Reference: Worked Example

Before — one component mixing a stats header, inline filtering/sorting logic, a status-label lookup, and per-row rendering:

```tsx
'use client'
import { useMemo, useState } from 'react'

type Task = { id: string; title: string; status: 'todo' | 'active' | 'done'; priority: number }

export function TaskBoard({ tasks }: { tasks: Task[] }) {
  const [query, setQuery] = useState('')

  const statusLabels: Record<Task['status'], string> = {
    todo: 'To do',
    active: 'In progress',
    done: 'Done',
  }

  const visibleTasks = useMemo(() => {
    return tasks
      .filter((t) => t.title.toLowerCase().includes(query.toLowerCase()))
      .sort((a, b) => b.priority - a.priority)
  }, [tasks, query])

  const doneCount = tasks.filter((t) => t.status === 'done').length

  return (
    <div>
      <div className="board-stats">
        <span>{tasks.length} total</span>
        <span>{doneCount} done</span>
        <span>{Math.round((doneCount / tasks.length) * 100)}% complete</span>
      </div>

      <input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="Search tasks" />

      <ul>
        {visibleTasks.map((t) => (
          <li key={t.id} className={`task-row task-row--${t.status}`}>
            <span>{t.title}</span>
            <span className="task-status">{statusLabels[t.status]}</span>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

After — one JSX region → `StatsHeader`, one JSX region → `TaskRow`, derived filtering/sorting → `useFilteredTasks`, the lookup table → a module-level constant:

```tsx
'use client'
import { useMemo, useState } from 'react'

type Task = { id: string; title: string; status: 'todo' | 'active' | 'done'; priority: number }

const STATUS_LABELS: Record<Task['status'], string> = {
  todo: 'To do',
  active: 'In progress',
  done: 'Done',
}

function useFilteredTasks(tasks: Task[], query: string) {
  return useMemo(
    () =>
      tasks
        .filter((t) => t.title.toLowerCase().includes(query.toLowerCase()))
        .sort((a, b) => b.priority - a.priority),
    [tasks, query],
  )
}

function StatsHeader({ tasks }: { tasks: Task[] }) {
  const doneCount = tasks.filter((t) => t.status === 'done').length
  return (
    <div className="board-stats">
      <span>{tasks.length} total</span>
      <span>{doneCount} done</span>
      <span>{Math.round((doneCount / tasks.length) * 100)}% complete</span>
    </div>
  )
}

function TaskRow({ task }: { task: Task }) {
  return (
    <li className={`task-row task-row--${task.status}`}>
      <span>{task.title}</span>
      <span className="task-status">{STATUS_LABELS[task.status]}</span>
    </li>
  )
}

export function TaskBoard({ tasks }: { tasks: Task[] }) {
  const [query, setQuery] = useState('')
  const visibleTasks = useFilteredTasks(tasks, query)

  return (
    <div>
      <StatsHeader tasks={tasks} />
      <input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="Search tasks" />
      <ul>
        {visibleTasks.map((t) => (
          <TaskRow key={t.id} task={t} />
        ))}
      </ul>
    </div>
  )
}
```

Notes on the choices made:

- `StatsHeader` and `TaskRow` stayed in the same file as unexported functions — both are small and used only here, so a new file would be pure ceremony.
- `useFilteredTasks` is a plain function above the component, not a new file — it's a derived-value helper, not a piece meant to be imported elsewhere.
- `STATUS_LABELS` moved to module scope because it's static: it no longer needs to be recreated on every render, and lifting it out separates "data" from "logic" without changing behavior.
- Nothing about props, rendering, or interaction changed — this is purely a shape change for readability.

## When there's nothing worth slicing

If a "large" file is actually one cohesive block (e.g. a single big `switch` with no reusable sub-parts, or JSX that has no independently-nameable regions), don't force a split just to hit a line count — say so and leave it.
