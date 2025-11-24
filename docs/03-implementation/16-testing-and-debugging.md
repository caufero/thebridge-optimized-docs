# **Chapter 16 — Testing & Debugging**

## **16.1 Overview**

Testing and debugging in THE BRIDGE require a completely different mindset from traditional application QA.

Because THE BRIDGE is **ontology-driven**, the system’s correctness is based on verifying:

* the **integrity of the ontology** (CMP, MET, ATR, OPE)
* the **determinism of the navigators**
* the **consistency of ETY states**
* the **immutability and completeness of LOG**
* the **correct functioning of JSON payloads**
* the **correct communication between WebViewer and FileMaker scripts**

Testing focuses on **mechanics**, not business rules — the ontology defines all logic.

---

# **16.2 Categories of Tests**

Every deployment of THE BRIDGE requires tests across these six layers:

1. **Structural Tests (CMP integrity)**
2. **State Tests (ETY integrity)**
3. **History Tests (LOG integrity)**
4. **Operation Tests (OPE execution)**
5. **Navigation Tests (X, Y, Z behavior)**
6. **UI Integration Tests (WebViewer ⇄ FileMaker communication)**

Each layer has formal procedures and code tools.

---

# **16.3 CMP Structural Tests**

CMP_TABLE is the foundation of the ontology.
Structural defects must be detected early.

### **16.3.1 Test: CMP Schema Completeness**

This test ensures every CMP template contains:

* full `json_schema`
* MET references
* ATR references
* `default_values`
* UI config
* OPE permissions
* structure_id integrity

**Utility script: `CMP_Check`**

```
// CMP_Check
Set Variable [ $errors ; Value: "" ]

Go to Layout [ “CMP_TABLE” (CMP_TABLE) ]
Show All Records

Loop
  Set Variable [ $cmp ; Value: CMP_TABLE::json_schema ]

  If [ IsEmpty ( $cmp ) ]
    Set Variable [ $errors ;
      Value: $errors & "Missing schema for CMP ID " & CMP_TABLE::dna_id & "¶"
    ]
  End If

  # Ensure MET and ATR exist
  If [ IsEmpty ( JSONGetElement ( $cmp ; "attributes" ) ) ]
    Set Variable [ $errors ;
      Value: $errors & "No attributes for CMP ID " & CMP_TABLE::dna_id & "¶"
    ]
  End If

  Go to Record/Request/Page [ Next ; Exit after last ]
End Loop

If [ Length ( $errors ) > 0 ]
  Show Custom Dialog [ "CMP Structural Errors" ; $errors ]
Else
  Show Custom Dialog [ "CMP OK" ; "All structures valid." ]
End If
```

---

# **16.4 ETY Integrity Tests**

ETY_TABLE is the live state of the system.
Testing must ensure ETY entries match their template definitions.

### **16.4.1 Test: ETY Attribute Validity**

```
// ETY_Check
Go to Layout [ “ETY_TABLE” (ETY_TABLE) ]
Show All Records

Loop
  Set Variable [ $schema ;
    Value: CMP_GetSchemaFor ( ETY_TABLE::template_id )
  ]
  Set Variable [ $data ;
    Value: ETY_TABLE::json_data
  ]

  # Validate required attributes exist
  Set Variable [ $required ;
    Value: JSONListKeys ( $schema ; "attributes" )
  ]

  Set Variable [ $missing ; Value: "" ]

  Loop
    Set Variable [ $attr ;
      Value: GetValue ( $required ; $i )
    ]
    Exit Loop If [ $attr = "" ]

    If [ IsEmpty ( JSONGetElement ( $data ; $attr ) ) ]
      Set Variable [ $missing ;
        Value: $missing & $attr & " "
      ]
    End If

    Set Variable [ $i ; Value: $i + 1 ]
  End Loop

  If [ Length ( $missing ) > 0 ]
    Show Custom Dialog [
      "ETY Error for " & ETY_TABLE::dna_id ;
      "Missing attributes: " & $missing
    ]
  End If

  Go to Record/Request/Page [ Next ; Exit after last ]
End Loop
```

---

# **16.5 LOG Integrity Tests**

LOG_TABLE ensures the NATURA history is intact.

### **16.5.1 Test: Log Continuity**

Verify every ETY mutation has a corresponding log entry.

```
// LOG_Check
Go to Layout [ “ETY_TABLE” ]
Show All Records

Loop
  Set Variable [ $dna ; Value: ETY_TABLE::dna_id ]

  # Check log count
  Set Variable [ $logCount ;
    Value: ExecuteSQL (
      "SELECT COUNT(*) FROM LOG_TABLE WHERE dna_id = ?" ;
      "" ; "" ; $dna
    )
  ]

  If [ $logCount = 0 ]
    Show Custom Dialog [
      "LOG WARNING" ;
      "Entity " & $dna & " has no log history."
    ]
  End If

  Go to Record/Request/Page [ Next ; Exit after last ]
End Loop
```

---

# **16.6 Operation Tests (OPE Execution)**

Operations (OPE) define all behaviors.
Testing OPE correctness ensures mutations are consistent and deterministic.

### **16.6.1 Test: Apply OPE in Controlled Environment**

```
// Test_Operation
Set Variable [ $payload ;
  Value:
  JSONSetElement ( "{}" ;
    [ "dna_id" ; "PHO25001" ; JSONString ] ;
    [ "operation" ; "OPE_SET_OUTCOME" ; JSONString ] ;
    [ "value" ; "INTERESTED" ; JSONString ]
  )
]

Perform Script [ “Execute_Operation” ; Parameter: $payload ]
Show Custom Dialog [ “Result” ; Get ( ScriptResult ) ]
```

Expected:

* `scriptResult` contains updated ETY JSON
* LOG entry created
* mutation matches OPE definition
* no field-level corruption

---

# **16.7 Navigation Tests (X, Y, Z)**

### **16.7.1 Navigation_X (Attribute Mutation)**

```
// Test_Navigate_X
Set Variable [ $test ;
  Value:
  "{ 
     \"dna_id\": \"PHO25002\",
     \"entity_type\": \"PHO\",
     \"attributes\": [
        { \"name\": \"priority\", \"value\": \"HIGH\" }
     ]
   }"
]

Perform Script [ “Navigate_X” ; Parameter: $test ]
Show Custom Dialog [ "Navigate_X Output" ; Get ( ScriptResult ) ]
```

### **16.7.2 Navigation_Y (Creation + Filtering)**

```
// Test_Navigate_Y_Create
Set Variable [ $test ;
  Value:
  "{ 
     \"action\": \"CREATE\",
     \"template_id\": \"TPL_PHO_001\"
   }"
]

Perform Script [ “Navigate_Y” ; Parameter: $test ]
```

### **16.7.3 Navigation_Z (Depth Navigation)**

```
// Test_Navigate_Z_Children
Set Variable [ $test ;
  Value:
  "{ 
     \"action\": \"GET_CHILDREN\",
     \"dna_id\": \"ORD70002\"
   }"
]

Perform Script [ “Navigate_Z” ; Parameter: $test ]
```

---

# **16.8 UI / WebViewer Tests**

UI test strategy focuses on:

* verifying VIEW_MODEL correctness
* validating rendering
* ensuring command round-trips work

### **16.8.1 Test: VIEW_MODEL Injection**

```
// Script to test JSON injection
Set Field [ UI::g_ViewModel_JSON ;
"
{
  \"view\": \"SuperTable\",
  \"entity_type\": \"PHO\",
  \"filters\": {},
  \"columns\": [
    {\"id\": \"name\", \"label\": \"Name\", \"visible\": true}
  ],
  \"rows\": [
    {\"dna_id\": \"PHO1\", \"name\": \"Test A\", \"is_selected\": false},
    {\"dna_id\": \"PHO2\", \"name\": \"Test B\", \"is_selected\": false}
  ],
  \"meta\": {\"total_rows\": 2, \"visible_columns\": 1}
}
"
]
Refresh Window
```

Should render immediately.

---

# **16.9 Debugging Tools**

### **16.9.1 JSON Inspector (Recommended)**

A dedicated layout with:

* one global field for input JSON
* one global field showing formatted JSON
* a list of keys
* a recursive inspector routine

### **16.9.2 LOG Timeline Viewer**

Shows chronological NATURA history for a selected entity:

```sql
SELECT timestamp, operation_id, before_state, after_state
FROM LOG_TABLE
WHERE dna_id = ?
ORDER BY timestamp ASC
```

### **16.9.3 ETY Snapshot Comparison**

```
// Compare two ETY snapshots
Let ([
  a = JSONFormatElements ( $before ) ;
  b = JSONFormatElements ( $after )
];
  JSON_Diff ( a ; b )
)
```

---

# **16.10 Debugging Common Issues**

### **Issue 1 — VIEW_MODEL not rendering**

Check:

* malformed JSON
* missing keys (`columns`, `rows`)
* placeholders not substituted
* incorrect URL encoding

### **Issue 2 — WebViewer commands not reaching FileMaker**

Check:

* script name mismatch
* no `window.FileMaker.PerformScript` available
* using WKWebView but not using the iOS handler

### **Issue 3 — ETY attribute missing**

Check:

* CMP schema missing attribute
* template incorrectly defined
* OPE mutation removed it
* JSONDeleteElement executed unintentionally

### **Issue 4 — LOG not created**

Check:

* mutation ran outside Process Manager
* Navigate_X/Y/Z skipped Universal_Processor
* no write access to LOG_TABLE

---

# **16.11 Summary**

Testing THE BRIDGE involves:

* verifying CMP → ETY → LOG integrity
* verifying navigation commands
* verifying OPE execution
* verifying JSON pipelines
* ensuring UI/WebViewer communication works
* confirming entity histories are consistent

The QA process is highly mechanical and scriptable, because the ontology is deterministic.
Once all tests pass, the system is ready for deployment.
