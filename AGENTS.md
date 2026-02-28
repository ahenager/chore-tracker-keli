# AI Agent Instructions: Chore Tracker App & Keli Platform

You are an expert developer assistant specialized in the proprietary **Keli** application development platform and the **Chore Tracker** application. Your primary directive is to assist with maintaining, improving, and supporting the Chore Tracker app by generating, debugging, and explaining Keli code.

## 1. Core Directives

* **Match Surrounding Code Style:** Always adapt your generated code to match the indentation, naming conventions, and structural patterns of the file or template you are editing. If the surrounding code uses specific parameter grouping or shorthand (e.g., `on`/`off` instead of `true`/`false`), mirror that exact style.
* **Architecture:** Chore Tracker heavily utilizes a `db off` **Dashboard** for UI, standard relational Keli templates for data storage (`User`, `Chore`, `Assignment`), and a dedicated REST API template. Keep UI logic on the client and complex data manipulation on the server.

## 2. Keli Language & Syntax Rules

Keli templates combine declarative object/parameter definitions with JavaScript logic. Keli templates are stored with a `.kjs` extension.

### Objects and Parameters
* **Structure:** Objects are declared on their own line using proper snake case (e.g., `Has_Day_Restriction`). Parameters follow on the same line or subsequent lines separated by commas.
* **Booleans:** Use Keli keywords `on` and `off` instead of `true` or `false` in parameter definitions (e.g., `vis off`, `mode on`, `insertable off`).
* **Calculated Fields (`c`):** Use the `c` parameter with Keli SQL to create database-level triggers that auto-update fields (e.g., `c SELECT sum(Value) FROM Assignment WHERE User = NEW.Id`). Note the use of `NEW` to reference the current record.
* **Event Listeners vs. Parameters:** Understand the difference between parameters (e.g., `bu`, `caption`, `icon`) and event listeners (e.g., `ac`, `bp`). Parameters evaluate to values; event handlers execute logic when an event is fired.
* **Rungs:** Use rungs for event execution order when overriding default behavior or chaining events (e.g., `ac.1 { ... stop(); }` prevents the default `ac.5` behavior from opening a record when clicking a table row).

### JavaScript Integration & Scopes
* **Tags:** JavaScript functions must be at the bottom of the template and use tags like `[Client]` and `[Server]`. Use `[Server, Access 'Chore_Tracker.Application', Rest]` to define REST endpoints.
* **Get/Set:** Keli implicitly translates variable calls to `get()` and `set()`. Standard variables without definitions will resolve to database fields or scope variables.
* **Scope Traversal:** Keli UI components launch in scopes. Use `_top` to access the parent panel's scope from a child table, and use the `->` operator to call functions across scopes (e.g., `_top->completeChore()`).

## 3. Keli SQL (Database Interactions)

Keli SQL is a custom dialect that translates to PostgreSQL. 

* **Automatic CDF:** Context and Deleted Date Filters (CDF) are applied automatically. `SELECT Name FROM User` translates to `SELECT Name FROM User WHERE _ctx = :_c AND Deleted_Date IS NULL`.
* **Variable Binding:** Use Keli parameter binding `:varName` (e.g., `WHERE Id = :userId`). Also supports inline dynamic parsing like `:dateStamp('c')`.
* **Query Functions:**
  * `q(query)` / `pq(query)`: Returns a single value, a single object (if multiple columns), or number of affected rows (if UPDATE/INSERT).
  * `query(query)` / `pquery(query)`: Returns an array of row objects.
  * `qk(query)`: Returns an object mapped by the first column (Query Key). Highly useful for mapping lookup IDs to specific values.
* **Saving Data:** Always use `save('Template_Name', arrayOfObjects)` to execute batch insertions/updates instead of looping `INSERT` queries.

## 4. Chore Tracker Domain Knowledge

When working on Chore Tracker business logic, understand these core entities and flows:

* **User (`chore_tracker.user.kjs`):** The individual performing the chores. Contains tracking for `Daily_Goal` and `Weekly_Goal`, with `c` triggers that auto-calculate `Daily_Total` and `Weekly_Total` from their assignments.
* **Chore (`chore_tracker.chore.kjs`):** The task definition. Chores hold `Value` (points/currency) and can have extensive limitations:
  * Restrictable by Date (`Start_Date`, `End_Date`).
  * Restrictable by Time (`Start_Time`, `End_Time`).
  * Restrictable by Day of Week (`Monday` through `Sunday` boolean flags).
  * Usage limits via `Per_Day` and `Per_Week`.
* **Assignment (`chore_tracker.assignment.kjs`):** The ledger of completed chores. Links a `User`, a `Chore`, and the `Date` it was completed. Contains an `Approved` boolean restricted to users with the `chore_tracker.supervisor` role.
* **Dashboard (`chore_tracker.dashboard.kjs`):** A `db off` interactive UI where users log their chores. It filters available chores using `getChoreFilter()` (evaluating all time/day restrictions dynamically) and uses `ac.1` events on the `Chores` table to prompt completion without opening the standard Keli browse.
* **REST API (`chore_tracker.api.kjs`):** Endpoints tagged with `[Rest]` that allow external clients to interact with the system. Endpoints include `getUserList`, `retrieveTasks`, `completeTask`, and `uncompleteTask`. Note that API functions require validating `request.method` manually and throwing a 405 error if incorrect.