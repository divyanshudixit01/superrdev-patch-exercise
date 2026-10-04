# Engineering Notes — Patch Exercise

## 1. Summary of Changes
- **SQL & Data Leak (Database Layer):** Fixed operator precedence in `TaskRepository.java` and `search_tasks.sql` by explicitly parenthesizing `(LOWER(title) LIKE :term OR LOWER(description) LIKE :term)`. This guarantees `archived = FALSE` is strictly respected, preventing archived tasks from leaking. Synchronized the identical logic to the Oracle reference package (`task_search_package.sql`).
- **Artificial Latency (Backend Layer):** Removed the synthetic `Thread.sleep(queryWeight)` delay in `TaskController.java`, dropping API response latency from ~1100ms to <20ms. Added defensive handling for invalid status strings to return a clean empty list rather than crashing with an unhandled 500 exception.
- **Race Conditions & UX (Frontend Layer):** Integrated `AbortController` in `api.js` and `useTasks.js` cleanup to cancel stale in-flight requests during rapid typing. Added a 300ms debounce to the search bar in `App.jsx` and synchronized pagination to automatically reset to Page 1 whenever search terms or status filters change.

## 2. What Was Not Changed & Why
- **In-Memory Pagination:** Retained controller-level `subList` slicing because the current seed dataset is small (48 records) and fits within the 90-minute timebox. Refactoring the repository to use Spring Data `Pageable` was deliberately deferred to keep the patch focused and low-risk.
- **UI & Styling:** Kept existing CSS untouched to avoid cosmetic churn and prioritize functional correctness, data integrity, and throughput.

## 3. Biggest Remaining Risk
- **Unbounded Memory Usage at Scale:** `TaskRepository.searchTasks()` loads all matching database rows into JVM heap memory before slicing. In a production environment with hundreds of thousands of tasks, concurrent queries could cause severe garbage collection pauses or trigger an `OutOfMemoryError`. Migrating to SQL-level `LIMIT`/`OFFSET` pagination is the highest production priority.

## 4. Tools & AI Usage
Used AI as an interactive pair programmer to cross-check boolean operator precedence across SQL files and draft the `AbortController` cleanup pattern. I manually validated all behavior, network waterfalls, and data integrity using browser DevTools and local curl tests.
