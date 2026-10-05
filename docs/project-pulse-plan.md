# Project Pulse implementation plan

## Summary and goal

Project Pulse is a lightweight static dashboard for Mona's contributors. It should
make the state of the team’s projects scannable at a glance: which projects are
active, who owns them, their current status, recent activity, priority or risk,
and a short contributor-friendly summary. The result should be a polished,
accessible card-based frontend rather than a plain data page or a server
directory listing.

The implementation is intentionally dependency-light. The browser will load
`app/index.html`, `app/styles.css`, and `app/project-data.json` from a local
static server. The VS Code launch configuration will serve the `app` directory
with Python's standard library and open the dashboard's `index.html`.

## Ordered implementation phases

### Phase 1 — repository and contract review

**Owner:** Planner, with Orchestrator review.

1. Read the project brief and the custom agent guidance in
   `.github/agents/`.
2. Confirm the required output paths and the static-app constraint.
3. Establish the content and integration contract below before implementation:
   the page title is exactly `Project Pulse`; the page references
   `styles.css` and `project-data.json`; data has a top-level `projects` array;
   and every project has `name`, `owner`, `status`, `recentActivity`, and
   `priority`.
4. Record the dependency and parallel-work decisions so Designer and Coder do
   not edit the same file unexpectedly.

### Phase 2 — information architecture and visual direction

**Owner:** Designer. **File assignment:** `app/styles.css` (design direction
and styling implementation), with recommendations for `app/index.html`.

1. Define a clear hierarchy: page heading and short summary first, then the
   project overview/grid, then any supporting legend or contributor guidance.
2. Specify a responsive card layout that works on narrow and wide screens.
3. Define readable typography, spacing, color contrast, focus states, status
   badges, priority/risk treatment, borders, `border-radius`, and
   `box-shadow`.
4. Ensure the design supports accessible headings, landmarks, keyboard focus,
   semantic status text, and a non-color-only distinction between statuses or
   priorities.
5. Implement the visual system in `app/styles.css`, including `.dashboard` and
   `.project-card` selectors. Keep the stylesheet self-contained and avoid
   introducing an unrequested framework or build step.

### Phase 3 — content model and page implementation

**Owner:** Coder. **File assignments:** `app/project-data.json` and
`app/index.html`.

1. Create `app/project-data.json` with a top-level `projects` key containing
   multiple representative projects. Each object must include `name`, `owner`,
   `status`, `recentActivity`, and `priority`; add a concise summary if the
   interface needs one.
2. Build `app/index.html` with the exact title and a semantic, accessible
   dashboard shell. Link `styles.css` and reference/load `project-data.json`.
3. Render visible project cards from the data rather than leaving the page as a
   static empty shell. Every card must use the `project-card` class and show
   the project's status, `recentActivity`, and priority, along with its name and
   owner.
4. Keep the data-loading behavior deterministic. Handle a failed or malformed
   data request with a visible, contributor-friendly error state rather than an
   unhandled console failure. Avoid assuming that every optional field exists.
5. Integrate the Designer's hierarchy and class contract without changing the
   stylesheet's ownership.

### Phase 4 — local launch support

**Owner:** Coder. **File assignment:** `.vscode/launch.json`.

1. Create `.vscode/launch.json` as strict JSON with no comments.
2. Add a configuration named exactly **Run Project Pulse Dashboard**.
3. Configure it to serve from the `app` directory using exactly
   `python3 -m http.server 5500`.
4. Include `serverReadyAction` so the browser opens
   `http://localhost:%s/index.html`, not the directory root. The working
   directory and URL must therefore lead directly to the dashboard frontend.
5. Keep this configuration deterministic and compatible with the repository's
   existing VS Code/Codespace conventions. Do not modify `.vscode/tasks.json`.

### Phase 5 — integration, validation, and handoff

**Owner:** Orchestrator, with Designer and Coder verifying their areas.

1. Review all four assigned outputs together for matching filenames, selectors,
   data keys, and launch behavior.
2. Validate the JSON and launch configuration syntactically, inspect the HTML
   references and rendered card contract, and review responsive/accessibility
   behavior.
3. Start **Run Project Pulse Dashboard**, confirm the browser opens
   `app/index.html` and displays cards, then stop the preview server.
4. Report the agents' contributions, completed validation, limitations, and the
   final handoff. No implementation file should be committed as part of this
   planning phase.

## File ownership and responsibilities

| File | Primary owner | Assignment and acceptance contract |
| --- | --- | --- |
| `app/index.html` | Coder, informed by Designer | Exact `Project Pulse` title; semantic dashboard markup; link to `styles.css`; reference/load `project-data.json`; visible `.project-card` cards; name, owner, status, `recentActivity`, and priority displayed; accessible loading and error states. |
| `app/styles.css` | Designer | Polished responsive layout with `.dashboard` and `.project-card`; readable spacing and hierarchy; status/priority styling; `border-radius`; `box-shadow`; contrast, focus, and narrow-screen support. |
| `app/project-data.json` | Coder | Valid JSON with a top-level `projects` array and multiple realistic project records. Every record has `name`, `owner`, `status`, `recentActivity`, and `priority`; values are suitable for the visible card UI. |
| `.vscode/launch.json` | Coder | Strict JSON; launch name `Run Project Pulse Dashboard`; serves from `app`; runs `python3 -m http.server 5500`; uses `serverReadyAction` to open `http://localhost:%s/index.html`. |

### Designer responsibilities

The Designer owns the visual and usability decisions: information hierarchy,
responsive composition, spacing, typography, color and status semantics,
accessible contrast and focus states, card treatment, and the visual polish
that keeps the dashboard from becoming a plain page. The Designer implements
those decisions in `app/styles.css` and gives the Coder clear markup/class
expectations without editing `app/index.html` or the data/configuration files.

### Coder responsibilities

The Coder owns the data model, semantic page structure, rendering logic, and
launch support. The Coder creates `app/project-data.json`, implements
`app/index.html` to load and render the data safely, and creates
`.vscode/launch.json` with the specified server and browser URL. The Coder
integrates the Designer's CSS contract, avoids unrelated repository changes,
and performs syntax and browser smoke checks.

### Orchestrator responsibilities

The Orchestrator turns this plan into explicit assignments, prevents concurrent
edits to the same file, resolves integration issues, runs the validation pass,
and reports what Planner, Designer, and Coder delivered. The Orchestrator does
not replace the specialists by making an undifferentiated implementation edit.

## Dependencies and ordering

* Phase 1 must precede implementation because it fixes the data, selector, URL,
  and file contracts.
* Designer's information architecture should be agreed before final HTML
  markup, but the Designer can implement the independent stylesheet in parallel
  with Coder's data-model work.
* `app/index.html` depends on both the data schema from
  `app/project-data.json` and the class/layout contract from
  `app/styles.css`. The final integration review must follow both.
* `.vscode/launch.json` depends on the known app directory and target filename,
  but does not depend on the completed visual styling. It can be drafted in
  parallel with the app files and validated after all outputs exist.
* Browser validation is necessarily sequential after all files are present:
  syntax checks first, then launch the server, then inspect the rendered page,
  then stop the server and record results.

## Parallel versus sequential work decisions

### Safe parallel work

After Phase 1, these tasks may run concurrently because their file scopes do
not overlap:

* Designer implements `app/styles.css`.
* Coder creates `app/project-data.json`.
* Coder drafts `.vscode/launch.json`.

The Orchestrator may also review the brief and prepare the validation checklist
while those tasks run. Parallel work is safe only if the Designer and Coder
agree on the documented class/data contracts and do not edit one another's
files.

### Work that must remain sequential

* The contract review must precede the Designer/Coder assignments.
* Final `app/index.html` integration follows the stylesheet and data contracts;
  if the Coder starts it early, it must be reviewed again after both parallel
  tasks finish.
* Cross-file integration review follows all implementation tasks.
* Launch smoke testing follows JSON validation and file integration.
* Handoff reporting follows validation, so it does not claim that an untested
  launch or rendering path works.

## Validation expectations

### Static and structural checks

* Confirm all four assigned files exist and no implementation files outside the
  assignment were changed.
* Parse `app/project-data.json` and `.vscode/launch.json` as JSON. Confirm the
  data has a top-level `projects` array and that every project has all five
  required fields.
* Inspect `app/index.html` for the exact title, stylesheet link, data reference,
  semantic landmarks/headings, `.project-card` markup, and displayed
  `status`, `recentActivity`, and `priority` fields.
* Inspect `app/styles.css` for `.dashboard`, `.project-card`,
  `border-radius`, `box-shadow`, responsive layout rules, visible focus
  treatment, and readable contrast.
* Inspect launch configuration fields for the exact launch name, `app`
  working directory, `python3 -m http.server 5500`, and
  `http://localhost:%s/index.html`.

### Runtime and accessibility checks

* Use **Run Project Pulse Dashboard** and verify the browser opens the
  dashboard page rather than a directory listing.
* Confirm multiple project cards are visible and each shows the required
  project information.
* Confirm a missing/invalid data response produces a usable visible error
  state, while normal data produces no unhandled rendering error.
* Check the layout at narrow and wide viewport sizes, keyboard focus visibility,
  heading order, meaningful labels, and status/priority meaning without relying
  only on color.
* Stop the preview server after the smoke test and document any environment
  limitation (for example, browser automation not being available).

## Risks and edge cases

* **File URL restrictions:** Fetching JSON from `file://` may be blocked by
  browser CORS rules. Always validate through the configured local HTTP server,
  not by double-clicking the HTML file.
* **Directory listing regression:** A launch URL ending at the app root can show
  a listing. Keep both the `app` working directory and the explicit
  `index.html` URL aligned.
* **Malformed or unavailable data:** JSON syntax errors, a missing `projects`
  key, an empty array, or a failed fetch must not leave a blank page. Show a
  useful fallback and tolerate missing optional summary content.
* **Unexpected values:** Status and priority may contain unfamiliar values.
  Render readable text with a safe fallback class/text instead of assuming a
  fixed enum or injecting unsanitized HTML.
* **Accessibility and color:** Status badges must remain understandable in
  grayscale or for color-vision differences. Preserve contrast, keyboard
  focus, semantic headings, and readable text at small widths.
* **Responsive overflow:** Long project names, owner names, and activity text
  should wrap without forcing horizontal scrolling or breaking the card grid.
* **Port conflicts:** Port 5500 may already be occupied. Report the conflict
  clearly and stop any test server after validation; do not silently change the
  documented launch contract.
* **Scope drift and merge conflicts:** Designer and Coder must retain exclusive
  file ownership. The Orchestrator should resolve contract mismatches in a
  sequential integration pass rather than allowing overlapping edits.
* **Static-server assumptions:** `python3` must be available in the Codespace.
  If it is not, record the environment blocker rather than changing the
  required command without approval.

## Open questions

No framework, package manager, backend, or external API is required by the
brief. If the team wants filtering, sorting, persistence, or live project
updates later, those should be planned as a separate enhancement rather than
added to this deterministic static-dashboard scope.
