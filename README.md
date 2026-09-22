# Enhanced Organization Chart & Simulator Tool

Single-page application for exploring and simulating an organization structure directly in the browser.

## Run / deploy

The public entry point is `index.html`. The repository can be deployed as a static site (for example on GitHub Pages or Vercel). The application uses SheetJS from CDN for Excel import/export.

## Excel input used by this version

This version is optimized for the workbook structure used in `EmployeeData_with_Teams_and_Levels.xlsx` and automatically prioritizes the worksheet **`Excel Output`**.

The parser reads columns by **header name**, not by Excel column letter, including:

- `Employee ID` (preferred unique identifier; row fallback when missing)
- `Employee Name`
- `Job Title`
- `Supervisor ID` (preferred reporting relationship)
- `Supervisor Name` (fallback reporting relationship)
- `Labor Type` (`Direct Labor` / `Indirect Labor`)
- `Level` (`L4`, `L5`, ...)
- `L4` ... `L8` (hierarchy-path fallback)
- `Location Name`
- `Regular/Temporary`
- `Employee Status`
- `Team`
- `Exit List` (e.g. `Yes` for identified potential exits)
- `Exit Priority` (priority level shown on Exit List cards)
- `IAS` (`Yes` / `No`)
- `BA` (`Yes` / `No`)
- `Grants` (`Yes` / `No`)

The source organizational level is preserved on the card. Graph layout depth is kept separately, so a top card can correctly be `Layer 4` without being relabeled `Layer 1`.

## Card actions

Every active card exposes six controls in the top-right corner:

1. **Add card below** — create a new position or search an existing employee and move them below the selected card.
2. **Edit card** — edit name, job title, location, regular/temporary, workforce type, and team.
3. **Assign / change supervisor** — search for another employee, then choose whether to move **only that person** or **the person + their entire reporting line**. Cycles are blocked.
4. **Center card** — focus and center the selected card.
5. **Status & Exit List** — manage the position status (`In Org`, `Natural Attrition`, `Open / Unstaffed`, `Closed Position`, `People Exited`) and, independently, add/remove any active person from the potential **Exit List** and set Exit Priority (`P1`–`P6`).
6. **Delete card** — enabled only for cards created manually in the simulator, and only when they have no direct reports. Imported Excel records are protected.

Existing drag-and-drop moves, hierarchy expansion, offboarding area, restore, Undo, Reset, search, Direct/Indirect filtering, and Excel export remain available. Drag-and-drop and **Add card below → Existing employee** use the same move-scope choice.

## Card information

Cards display:

- organizational Layer from the Excel `Level` field;
- position status;
- Manager / Individual Contributor indicator;
- New Position / Changed Supervisor badges when relevant;
- `Location Name`;
- `Regular/Temporary`;
- an **Exit List** badge, with priority when available;
- compact employee flags for **IAS** (blue), **BA** (purple), and **Grants** (light blue) when the corresponding Excel field is `Yes`;
- **Span of Control**, total headcount, and team.

Employees marked `Exit List = Yes` in the uploaded Excel are loaded as Exit List members and remain visible and fully interactive in the hierarchy. The simulator can then **add any other active person to the Exit List, remove an imported person from it, or change the Exit Priority** without changing the person's Position Status.

Exit List members are excluded from their direct manager's **Span of Control** count. Removing a person from the Exit List immediately adds them back to that count. Total Headcount and the reporting relationship itself are not removed. `People Exited` remains a separate status for people who have already left.

Exit List changes participate in **Undo** and **Reset**. Reset restores the Exit List and priorities exactly as they were in the uploaded Excel.

The previous Target Reduction logic is retained in the code but the Target Reduction block is currently hidden from the cards.
## Collapsible control panel

The left control panel can now be collapsed from the top navigation bar. On desktop, collapsing the panel gives the organization chart the full available width; the chart automatically refits to the enlarged canvas. The same control reopens the panel. Mobile behavior remains an overlay drawer.



## Move scope

Every existing-employee move supports two modes:

- **Move person + entire team** (default, preserving the previous simulator behavior): the selected employee and all direct/indirect reports move under the new supervisor.
- **Move only this person**: only the selected employee moves. Their direct reports stay in the original branch and are reassigned to the employee's previous manager; each report keeps its own downstream team. If the moved employee had no previous manager, those direct reports become organization roots.

Both modes update organizational layers, card metrics, Span of Control, Changed Supervisor indicators, and participate in **Undo** and **Reset**.

## Direct card drag & drop

Cards can be moved directly in the organization chart by pressing and dragging the card body onto the destination manager card. The destination card is highlighted while dragging. Releasing the card opens the existing move-scope choice so the user can select **Move only this person** or **Move person + entire team** before the change is applied.

The pointer-based implementation is designed to work consistently while the chart is zoomed or panned and does not interfere with card action buttons. Dragging an employee to the Offboarding Area continues to open the offboarding flow. Drag moves participate in **Undo** and **Reset** exactly like moves started from the card controls.

## Placeholder positions (0 HC)

The simulator supports unstaffed placeholder positions in addition to normal added cards.

- Create a placeholder from **Add card below → Placeholder position · 0 HC**.
- A placeholder is a structural organization node: it can have a supervisor, direct reports, be moved with drag and drop, and be exported.
- While unstaffed, the placeholder contributes **0** to Active Employees, Span of Control, Indirect Headcount, and Total Headcount calculations. Its descendants still contribute their own headcount normally.
- Fill a placeholder either from its assignment action or by dragging an existing employee directly onto the placeholder card. The placeholder is highlighted as a dedicated fill target; on drop, the selected employee is pre-populated in the confirmation dialog. The employee then fills the placeholder position and the placeholder node is removed, so total organization headcount does not increase.
- When filling a placeholder, the user can move only the selected employee or the employee together with their existing reporting line. Existing reports attached to the placeholder remain attached to the filled position.
- Placeholder creation and filling are included in Undo/Reset behavior.
- Excel export includes `Placeholder Position`, `Headcount Value`, and `Filled from Placeholder` fields, and the Summary separates active employees from active placeholder cards.
## Reordering peers within the same reporting line

Direct reports can now be reordered horizontally without changing reporting relationships or moving their teams. Drag an employee over a peer who has the same manager and release on the **left edge** of the peer card to place the employee before it, or on the **right edge** to place the employee after it. A blue insertion marker shows the intended position.

The center of the card keeps the existing drag-and-drop behavior for organizational moves, so dropping on another employee still supports moving the person/team under that employee. Dropping on an empty placeholder continues to fill the placeholder. Horizontal reordering changes only display order: supervisor, subtree, headcount, span of control, team membership and employee status are unchanged.

Peer order participates in Undo/Reset and is exported in the `Sibling Order` column. If that column is present on a later import, the simulator restores the same peer ordering.


## IAS / BA / Grants employee flags

The `Excel Output` sheet can include the boolean employee attributes `IAS`, `BA`, and `Grants`. Values such as `Yes`, `Y`, `True`, or `1` are interpreted as active flags.

Each active flag is shown directly on the employee card with a small colored square and label: **IAS = blue**, **BA = purple**, **Grants = light blue**. The flags are employee attributes only: they do not affect hierarchy, headcount, Span of Control, offboarding, placeholder logic, or peer ordering. Because they belong to the employee record, they remain attached to the person when that person is moved or used to fill a placeholder.

Excel export includes `IAS`, `BA`, and `Grants` columns for both active and offboarded employees.
