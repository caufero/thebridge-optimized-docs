# **Chapter 17 — UI & WebViewer Architecture**

## **17.1 Role of the UI Layer**

The UI is the ENTITÀ perspective made visible.

* FileMaker provides the **host layouts** and script triggers.
* The WebViewer provides a **dynamic rendering surface** driven by JSON.
* All UI behaviors are manifestations of CMP–ETY–LOG and the navigators.

The UI must never hard-code domain logic.
It only:

1. Receives JSON from the navigation layer.
2. Renders that JSON.
3. Sends JSON back to FileMaker when the user interacts.

Everything else is handled by the Instance Manager, Process Manager, and the Universal Navigators.

---

## **17.2 High-Level Data Flow**

The UI/WebViewer cycle follows this pattern:

1. FileMaker script prepares a JSON **VIEW_MODEL**.
2. WebViewer HTML/JS reads the VIEW_MODEL and renders the interface.
3. User interacts with the UI (click, filter, sort, edit).
4. JS builds a **COMMAND JSON** and sends it back to FileMaker.
5. FileMaker receives the command, calls **Navigate_X / Y / Z / Universal_Processor**.
6. The script computes the new state and rebuilds VIEW_MODEL.
7. WebViewer refreshes and re-renders.

This loop is universal for all screens.

---

## **17.3 The VIEW_MODEL JSON Contract**

The VIEW_MODEL is the single object that the UI reads.

Example: SuperTable view for entities of type `PHO`:

```json
{
  "view": "SuperTable",
  "entity_type": "PHO",
  "filters": {
    "lifecycle": "ALL",
    "priority": "ALL",
    "assigned_to": "ALL",
    "outcome": "ALL"
  },
  "columns": [
    {"id": "dna_id", "label": "DNA", "visible": true},
    {"id": "name", "label": "Name", "visible": true},
    {"id": "deadline", "label": "Deadline", "visible": true},
    {"id": "lifecycle", "label": "Lifecycle", "visible": true},
    {"id": "assigned_to", "label": "Assigned", "visible": true},
    {"id": "priority", "label": "Priority", "visible": true},
    {"id": "outcome", "label": "Outcome", "visible": true}
  ],
  "rows": [
    {
      "dna_id": "PHO25001",
      "name": "Chiamata Mario Rossi",
      "deadline": "2025-11-15",
      "lifecycle": "NEW",
      "assigned_to": "Sara Bianchi",
      "priority": "HIGH",
      "outcome": "NO_ANSWER",
      "is_selected": false
    },
    {
      "dna_id": "PHO25002",
      "name": "Chiamata Luigi Bianchi",
      "deadline": "2025-11-16",
      "lifecycle": "IN_PROGRESS",
      "assigned_to": "Marco Neri",
      "priority": "MEDIUM",
      "outcome": "INTERESTED",
      "is_selected": true
    }
  ],
  "meta": {
    "total_rows": 2,
    "visible_columns": 7
  }
}
```

The WebViewer UI does not know where this JSON came from.
It only knows how to render it.

---

## **17.4 FileMaker Side: Building VIEW_MODEL**

A dedicated script prepares the JSON for the WebViewer.

### **17.4.1 Script: `UI_Build_SuperTable_View`**

**Purpose:**
Build VIEW_MODEL based on current filters and selected entity_type.

```
// Script: UI_Build_SuperTable_View
// Context: SuperTable_View layout

Set Variable [ $entityType ; Value: $$filter_entity_type ]
Set Variable [ $lifecycle ; Value: $$filter_lifecycle ]
Set Variable [ $priority  ; Value: $$filter_priority ]
Set Variable [ $assigned  ; Value: $$filter_assigned ]
Set Variable [ $outcome   ; Value: $$filter_outcome ]

# 1. Get rows (simplified example using ExecuteSQL)
Set Variable [ $rowsJSON ;
  Value: UI_GetSuperTableRows_JSON ( $entityType ; $lifecycle ; $priority ; $assigned ; $outcome )
]

# 2. Define columns
Set Variable [ $columnsJSON ;
  Value:
  "[" &
    "{ \"id\": \"dna_id\", \"label\": \"DNA\", \"visible\": true }," &
    "{ \"id\": \"name\", \"label\": \"Name\", \"visible\": true }," &
    "{ \"id\": \"deadline\", \"label\": \"Deadline\", \"visible\": true }," &
    "{ \"id\": \"lifecycle\", \"label\": \"Lifecycle\", \"visible\": true }," &
    "{ \"id\": \"assigned_to\", \"label\": \"Assigned\", \"visible\": true }," &
    "{ \"id\": \"priority\", \"label\": \"Priority\", \"visible\": true }," &
    "{ \"id\": \"outcome\", \"label\": \"Outcome\", \"visible\": true }" &
  "]"
]

# 3. Build root VIEW_MODEL
Set Variable [ $viewModel ;
  Value:
  JSONSetElement ( "{}" ;
    [ "view" ; "SuperTable" ; JSONString ] ;
    [ "entity_type" ; $entityType ; JSONString ] ;
    [ "filters.lifecycle" ; $lifecycle ; JSONString ] ;
    [ "filters.priority" ; $priority ; JSONString ] ;
    [ "filters.assigned_to" ; $assigned ; JSONString ] ;
    [ "filters.outcome" ; $outcome ; JSONString ] ;
    [ "columns" ; JSONFormatElements ( $columnsJSON ) ; JSONRaw ] ;
    [ "rows" ; JSONFormatElements ( $rowsJSON ) ; JSONRaw ]
  )
]

Set Variable [ $viewModel ;
  Value:
  JSONSetElement ( $viewModel ;
    [ "meta.total_rows" ; Get ( FoundCount ) ; JSONNumber ] ;
    [ "meta.visible_columns" ; 7 ; JSONNumber ]
  )
]

# 4. Put into global field for WebViewer
Set Field [ UI::g_ViewModel_JSON ; $viewModel ]

Commit Records/Requests
Refresh Window [ Flush cached join results ; Flush cached external data ]
```

`UI_GetSuperTableRows_JSON` is a separate script or custom function that returns the `rows` array with the correct JSON structure.

---

## **17.5 WebViewer HTML/JS Shell**

The WebViewer uses an HTML shell that reads `UI::g_ViewModel_JSON` via the `GetLayoutObjectAttribute` technique or via `data:` URL injection.

### **17.5.1 WebViewer HTML Template**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>SuperTable View</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <style>
    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      font-size: 13px;
      background-color: #f5f6f8;
      color: #111;
    }

    .toolbar {
      padding: 8px 12px;
      background-color: #ffffff;
      border-bottom: 1px solid #dde0e5;
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .toolbar-title {
      font-weight: 600;
      margin-right: auto;
    }

    .badge {
      border-radius: 999px;
      padding: 2px 8px;
      font-size: 11px;
      border: 1px solid #ccd0d8;
      background-color: #f0f2f6;
    }

    .table-wrapper {
      padding: 8px 12px;
    }

    table {
      border-collapse: collapse;
      width: 100%;
      background-color: #ffffff;
      border-radius: 8px;
      overflow: hidden;
    }

    thead {
      background-color: #f0f2f6;
    }

    th, td {
      padding: 6px 8px;
      border-bottom: 1px solid #eceff3;
      text-align: left;
      font-size: 12px;
      white-space: nowrap;
    }

    tr:hover {
      background-color: #f7f9fc;
      cursor: pointer;
    }

    tr.selected {
      background-color: #e3f2ff;
    }

    .pill {
      border-radius: 999px;
      padding: 2px 8px;
      font-size: 11px;
      border: 1px solid rgba(0,0,0,0.06);
    }

    .pill.lifecycle-NEW {
      background-color: #e3f2fd;
    }

    .pill.lifecycle-IN_PROGRESS {
      background-color: #fff8e1;
    }

    .pill.lifecycle-DONE {
      background-color: #e8f5e9;
    }

    .pill.priority-HIGH {
      background-color: #ffebee;
    }

    .pill.priority-MEDIUM {
      background-color: #fff8e1;
    }

    .pill.priority-LOW {
      background-color: #e8f5e9;
    }
  </style>
</head>
<body>
  <div class="toolbar">
    <div class="toolbar-title" id="toolbar-title"></div>
    <div class="badge" id="badge-rows"></div>
    <div class="badge" id="badge-columns"></div>
  </div>

  <div class="table-wrapper">
    <table id="super-table">
      <thead id="super-table-head"></thead>
      <tbody id="super-table-body"></tbody>
    </table>
  </div>

  <script>
    /* Global view model */
    let viewModel = null;

    /* FileMaker bridge */
    function sendCommandToFileMaker(command) {
      var json = JSON.stringify(command);

      if (window.FileMaker) {
        /* Desktop / iOS */
        window.FileMaker.PerformScript("UI_Handle_Web_Command", json);
      } else if (window.webkit && window.webkit.messageHandlers && window.webkit.messageHandlers.FileMaker) {
        /* iOS WKWebView */
        window.webkit.messageHandlers.FileMaker.postMessage({
          scriptName: "UI_Handle_Web_Command",
          parameter: json
        });
      } else {
        console.log("Command to FileMaker:", json);
      }
    }

    /* Render toolbar */
    function renderToolbar(vm) {
      var title = document.getElementById("toolbar-title");
      var badgeRows = document.getElementById("badge-rows");
      var badgeCols = document.getElementById("badge-columns");

      title.textContent = "SuperTable • " + vm.entity_type;
      badgeRows.textContent = vm.meta.total_rows + " rows";
      badgeCols.textContent = vm.meta.visible_columns + " columns";
    }

    /* Render table header */
    function renderHeader(vm) {
      var head = document.getElementById("super-table-head");
      head.innerHTML = "";

      var tr = document.createElement("tr");
      vm.columns.forEach(function(col) {
        if (!col.visible) { return; }
        var th = document.createElement("th");
        th.textContent = col.label;
        tr.appendChild(th);
      });

      head.appendChild(tr);
    }

    /* Helper for cell content */
    function renderCell(colId, value, row) {
      if (colId === "lifecycle") {
        var span = document.createElement("span");
        span.className = "pill lifecycle-" + value;
        span.textContent = value;
        return span;
      }

      if (colId === "priority") {
        var span2 = document.createElement("span");
        span2.className = "pill priority-" + value;
        span2.textContent = value;
        return span2;
      }

      var span3 = document.createElement("span");
      span3.textContent = value == null ? "" : value;
      return span3;
    }

    /* Render table body */
    function renderBody(vm) {
      var body = document.getElementById("super-table-body");
      body.innerHTML = "";

      vm.rows.forEach(function(row) {
        var tr = document.createElement("tr");
        if (row.is_selected) {
          tr.classList.add("selected");
        }

        /* Row click: send selection command */
        tr.addEventListener("click", function() {
          sendCommandToFileMaker({
            type: "ROW_SELECT",
            entity_type: vm.entity_type,
            dna_id: row.dna_id
          });
        });

        vm.columns.forEach(function(col) {
          if (!col.visible) { return; }
          var td = document.createElement("td");
          var value = row[col.id];
          td.appendChild(renderCell(col.id, value, row));
          tr.appendChild(td);
        });

        body.appendChild(tr);
      });
    }

    /* Full render */
    function render(vm) {
      viewModel = vm;
      renderToolbar(vm);
      renderHeader(vm);
      renderBody(vm);
    }

    /* Entry point: FileMaker injects JSON here */
    function initFromFileMaker(jsonString) {
      try {
        var vm = JSON.parse(jsonString);
        render(vm);
      } catch (e) {
        console.error("Invalid VIEW_MODEL JSON", e);
      }
    }

    /* For local testing: optional stub */
    // document.addEventListener("DOMContentLoaded", function() {
    //   var sample = { ... };
    //   render(sample);
    // });
  </script>
</body>
</html>
```

`initFromFileMaker(jsonString)` is the entry function FileMaker calls by injecting the VIEW_MODEL into the WebViewer.

---

## **17.6 Injecting VIEW_MODEL into the WebViewer**

FileMaker can pass the VIEW_MODEL into the HTML in two main ways:

1. Using a `data:` URL calculation.
2. Using `GetLayoutObjectAttribute` with a global field.

### **17.6.1 WebViewer Calculation (data: URL)**

```
// WebViewer object calculation

"data:text/html," &
Substitute ( UI::g_HTML_SuperTable ;
  [ "{{VIEW_MODEL_JSON}}" ; GetAsURLEncoded ( UI::g_ViewModel_JSON ) ]
)
```

Where `UI::g_HTML_SuperTable` contains the HTML shell, including this line:

```html
<script>
  /* FileMaker will replace this placeholder */
  document.addEventListener("DOMContentLoaded", function() {
    var encoded = "{{VIEW_MODEL_JSON}}";
    var json = decodeURIComponent(encoded);
    initFromFileMaker(json);
  });
</script>
```

This approach keeps the WebViewer stateless and purely driven by the global JSON.

---

## **17.7 Handling UI Commands in FileMaker**

Whenever the user clicks a row or performs an action, the JS calls:

```js
sendCommandToFileMaker({
  type: "ROW_SELECT",
  entity_type: vm.entity_type,
  dna_id: row.dna_id
});
```

FileMaker must handle this in a single script.

### **17.7.1 Script: `UI_Handle_Web_Command`**

```
// Script: UI_Handle_Web_Command
// Parameter: JSON (command)

Set Variable [ $commandJSON ; Value: Get ( ScriptParameter ) ]
Set Variable [ $type        ; Value: JSONGetElement ( $commandJSON ; "type" ) ]

If [ $type = "ROW_SELECT" ]

  Set Variable [ $entityType ; Value: JSONGetElement ( $commandJSON ; "entity_type" ) ]
  Set Variable [ $dna        ; Value: JSONGetElement ( $commandJSON ; "dna_id" ) ]

  # Store selection in globals or context table
  Set Variable [ $$selected_entity_type ; Value: $entityType ]
  Set Variable [ $$selected_dna         ; Value: $dna ]

  # Call a navigation script if needed
  Perform Script [ "Navigate_Select_Entity" ;
    Parameter: $commandJSON
  ]

  # Rebuild and refresh view
  Perform Script [ "UI_Build_SuperTable_View" ]

End If
```

`Navigate_Select_Entity` can call `Navigate_Z` to open details, or simply reposition the context.

---

## **17.8 Example: Inline Attribute Edit**

To edit an attribute from the WebViewer, JS sends a mutation command.

### **17.8.1 JS side**

```js
function editName(row, newName) {
  sendCommandToFileMaker({
    type: "EDIT_ATTRIBUTE",
    entity_type: viewModel.entity_type,
    dna_id: row.dna_id,
    attribute: "name",
    value: newName
  });
}
```

### **17.8.2 FileMaker side**

```
// Script: UI_Handle_Web_Command (continuation)

Else If [ $type = "EDIT_ATTRIBUTE" ]

  Set Variable [ $entityType ; Value: JSONGetElement ( $commandJSON ; "entity_type" ) ]
  Set Variable [ $dna        ; Value: JSONGetElement ( $commandJSON ; "dna_id" ) ]
  Set Variable [ $attr       ; Value: JSONGetElement ( $commandJSON ; "attribute" ) ]
  Set Variable [ $value      ; Value: JSONGetElement ( $commandJSON ; "value" ) ]

  # Build Navigate_X payload
  Set Variable [ $payload ;
    Value:
    JSONSetElement ( "{}" ;
      [ "entity_type" ; $entityType ; JSONString ] ;
      [ "dna_id" ; $dna ; JSONString ] ;
      [ "attributes[0].name" ; $attr ; JSONString ] ;
      [ "attributes[0].value" ; $value ; JSONString ]
    )
  ]

  # Call Process Manager through Navigate_X
  Perform Script [ "Navigate_X" ; Parameter: $payload ]

  # Rebuild and refresh
  Perform Script [ "UI_Build_SuperTable_View" ]

End If
```

This closes the loop between UI and ontology:

* UI expresses an intent.
* Navigate_X translates it into OPE execution.
* ETY and LOG update.
* VIEW_MODEL regenerates.
* UI re-renders.

---

## **17.9 Summary**

The UI & WebViewer layer:

* is driven entirely by JSON VIEW_MODEL objects
* renders CMP–ETY–LOG outcomes without domain logic
* communicates with FileMaker through simple COMMAND JSON messages
* delegates all mutations to the navigators and Process Manager
* remains reusable for any entity type and any domain

Once this pattern is in place, every new domain view becomes a variation of the same architecture:
different VIEW_MODEL, same HTML shell, same communication pattern, same navigation engine.
