# Q2 — UML Diagrams (draw any one of the 4)

> Your manual doesn't list which 4 diagrams, so this covers the standard four for an SE lab: **Use Case, Class, Sequence, Activity**. If your syllabus says otherwise (e.g. State or Component), check with your faculty. The steps below are generic, and each has a worked example on the schizophrenia system, so they apply to whatever system the examiner gives you.

**Exam tips:** draw with ruler/neat boxes, give the diagram a title, label every arrow, and add a 2-line explanation of the notation used. Draw the actors/classes first, then the connections.

Free tools to practise and check: https://www.plantuml.com/plantuml (paste the code blocks below).

---

## 1. Use Case Diagram

**Purpose:** shows *who* uses the system and *what* they can do (functional view).

**Notation**
- Actor = stick figure (outside the system boundary)
- Use case = oval, named verb + noun ("Upload MRI")
- System boundary = rectangle around all use cases
- Association = plain line actor ↔ use case
- `<<include>>` = base *always* uses the other (dashed arrow base → included)
- `<<extend>>` = optional behaviour (dashed arrow extension → base)
- Generalization = hollow-triangle arrow (child → parent)

**Steps**
1. Identify actors (from the "Users" section of the SRS): Doctor, Researcher.
2. List use cases (from functional requirements): one oval per FR.
3. Draw a rectangle titled with the system name; put the ovals inside.
4. Put actors outside, left/right; connect each actor to the use cases they perform.
5. Add `<<include>>` for mandatory sub-steps (e.g. Predict *includes* Pre-process MRI).
6. Add `<<extend>>` for optional steps (e.g. View Explanation *extends* Predict).
7. Check: every FR appears as a use case; every use case has an actor.

**PlantUML**
```
@startuml
left to right direction
title Use Case Diagram - Schizophrenia Detection System
actor Doctor
actor Researcher

rectangle "Schizophrenia Detection System" {
  usecase "Login" as UC1
  usecase "Upload 3D sMRI (.nii)" as UC2
  usecase "Enter Age and Sex" as UC3
  usecase "Pre-process MRI" as UC4
  usecase "Predict Schizophrenia" as UC5
  usecase "View Confidence Score" as UC6
  usecase "View Explanation" as UC7
  usecase "Download PDF Report" as UC8
  usecase "Analyze Model Results" as UC9
}

Doctor --> UC1
Doctor --> UC2
Doctor --> UC3
Doctor --> UC5
Doctor --> UC6
Doctor --> UC8
Researcher --> UC1
Researcher --> UC2
Researcher --> UC9

UC5 ..> UC4 : <<include>>
UC2 ..> UC3 : <<include>>
UC7 ..> UC5 : <<extend>>
UC8 ..> UC5 : <<extend>>
UC6 ..> UC5 : <<include>>
@enduml
```

---

## 2. Class Diagram

**Purpose:** static structure — classes, attributes, operations, relationships.

**Notation:** a box with 3 compartments: `ClassName` / `attributes` / `methods()`.
Visibility: `+` public, `-` private, `#` protected.
Relationships:
- Association: plain line (with multiplicity 1, 0..*, 1..*)
- Aggregation: hollow diamond (whole–part, part can exist alone)
- Composition: filled diamond (part dies with whole)
- Inheritance/Generalization: hollow triangle arrow to parent
- Dependency: dashed arrow

**Steps**
1. List **nouns** from the problem statement → candidate classes (User, Doctor, Patient, MRIScan, Model, Report…).
2. Remove duplicates/irrelevant ones; keep 5–8 classes.
3. For each class write attributes (data) with types.
4. Write methods (verbs from the FRs): `upload()`, `predict()`, `generateReport()`.
5. Find inheritance (Doctor and Researcher are Users) → draw triangle arrows.
6. Add associations with multiplicity (Doctor 1 — * Patient; Patient 1 — * MRIScan).
7. Mark composition/aggregation (HybridModel *has* CNN and ViT).
8. Recheck: every attribute belongs to the right class; no class without a relationship.

**PlantUML**
```
@startuml
title Class Diagram - Schizophrenia Detection System
class User {
  - userId : int
  - name : String
  - email : String
  - password : String
  + login() : boolean
  + logout() : void
}
class Doctor {
  + uploadScan() : void
  + viewResult() : Prediction
  + downloadReport() : File
}
class Researcher {
  + analyzeResults() : void
}
class Patient {
  - patientId : int
  - age : int
  - sex : String
}
class MRIScan {
  - scanId : int
  - filePath : String
  - format : String = ".nii"
  + validate() : boolean
}
class Preprocessor {
  + normalize(scan : MRIScan) : Tensor
  + resize(scan : MRIScan) : Tensor
}
class HybridModel {
  + predict(img : Tensor, age : int, sex : String) : Prediction
}
class CNN {
  + extractLocalFeatures() : Tensor
}
class VisionTransformer {
  + extractGlobalFeatures() : Tensor
}
class Prediction {
  - label : String
  - confidence : float
  + explain() : String
}
class Report {
  - reportId : int
  - generatedOn : Date
  + generatePDF() : File
}

User <|-- Doctor
User <|-- Researcher
Doctor "1" -- "*" Patient : treats
Patient "1" -- "*" MRIScan : has
MRIScan "1" --> "1" Preprocessor : processed by
Preprocessor ..> HybridModel : feeds
HybridModel *-- CNN
HybridModel *-- VisionTransformer
HybridModel "1" --> "*" Prediction : produces
Prediction "1" -- "1" Report : included in
@enduml
```

---

## 3. Sequence Diagram

**Purpose:** *order of messages* between objects for ONE scenario (time flows top → bottom).

**Notation**
- Participant box at top; dashed vertical **lifeline** below it
- Solid arrow with filled head = synchronous message `name()`
- Dashed arrow = return/response
- Thin rectangle on lifeline = activation (object is working)
- `alt`/`opt`/`loop` frames for conditions/loops

**Steps**
1. Pick **one scenario**: "Doctor uploads MRI and gets prediction + report".
2. List participants left → right in order of first involvement: Doctor, UI, Server, Preprocessor, Model, ReportGenerator.
3. Draw lifelines downwards.
4. Add messages in time order: `1: upload(scan, age, sex)` → `2: validate()` → `3: preprocess()` → `4: predict()` → returns.
5. Add return arrows (dashed) with data: `prediction, confidence`.
6. Add an `alt` frame for invalid file: [invalid] show error / [valid] continue.
7. Number all messages.
8. Check: every arrow has a verb; the scenario starts and ends with the actor.

**PlantUML**
```
@startuml
title Sequence Diagram - MRI Prediction and Report
actor Doctor
participant "Web UI" as UI
participant "Server" as S
participant "Preprocessor" as P
participant "Hybrid Model" as M
participant "Report Generator" as R

Doctor -> UI : 1: upload(scan.nii, age, sex)
UI -> S : 2: submit(scan, age, sex)
S -> S : 3: validate(scan)
alt invalid file
  S --> UI : 4: error("Invalid .nii file")
  UI --> Doctor : show error
else valid file
  S -> P : 5: preprocess(scan)
  P --> S : 6: normalized & resized tensor
  S -> M : 7: predict(tensor, age, sex)
  M --> S : 8: label, confidence, explanation
  S --> UI : 9: result
  UI --> Doctor : 10: display prediction, confidence, explanation
  Doctor -> UI : 11: downloadReport()
  UI -> R : 12: generatePDF(result)
  R --> UI : 13: report.pdf
  UI --> Doctor : 14: PDF downloaded
end
@enduml
```

---

## 4. Activity Diagram

**Purpose:** workflow / flowchart of activities with decisions and parallel paths.

**Notation**
- Filled circle = start; circle with ring = end
- Rounded rectangle = activity/action
- Diamond = decision (with `[guard]` conditions on arrows) / merge
- Thick bar = fork (split into parallel) / join (synchronise)
- Swimlanes (optional) = who performs which activity

**Steps**
1. Draw the **start node**.
2. List the activities in order from the user's actions: Login → Upload MRI → Enter age & sex.
3. Add a **decision** diamond after validation: [valid] / [invalid → error → back to upload].
4. Add a **fork** where things run in parallel: (a) MRI → pre-process → CNN + ViT features; (b) age & sex → encode.
5. **Join** the paths → fuse features → classify.
6. Continue: show confidence → show explanation → [download report?] decision → generate PDF.
7. Draw the **end node**.
8. Check: every decision has all guards; every fork has a matching join; exactly one start.

**PlantUML**
```
@startuml
title Activity Diagram - Schizophrenia Detection
start
:Login;
:Upload 3D sMRI (.nii);
:Enter age and sex;
if (File valid?) then (yes)
  fork
    :Normalize and resize MRI;
    :Extract features with CNN + ViT;
  fork again
    :Encode age and sex;
  end fork
  :Fuse image and clinical features;
  :Classify (prediction + confidence);
  :Display confidence score;
  :Display explanation;
  if (Download report?) then (yes)
    :Generate PDF report;
    :Download report;
  else (no)
  endif
else (no)
  :Show error message;
  stop
endif
stop
@enduml
```

---

## 5. If the examiner gives a *different* system

Re-use the same recipe:

| Diagram | Take from the problem statement |
|---|---|
| Use Case | users → actors, features → use cases |
| Class | nouns → classes, their data → attributes, verbs → methods |
| Sequence | pick ONE feature, list who talks to whom in order |
| Activity | the same feature as a flowchart with decisions |

Examples: *Library system* → actors Librarian/Member; classes Book, Member, Loan, Fine; sequence "issue book"; activity "return book with fine check".
*Online shopping* → Customer, Admin; Product, Cart, Order, Payment; sequence "place order"; activity "checkout".
