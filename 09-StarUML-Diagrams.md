# UML Diagrams in StarUML — Step by Step

Diagrams covered: **Class, Sequence, Use Case, Component** (this replaces the Activity diagram from `02-UML-Diagrams.md`; the theory and notation there still apply).

Running example: the **Schizophrenia Detection System** from your manual. If the examiner gives a different system, keep the same steps and swap in its actors, classes and components (mapping table at the end).

> Menu names can differ slightly between StarUML versions (v4/v5/v6). If something isn't where I say, right-click the element or check the **Properties** panel. Those two places cover almost everything.

---

## 0. StarUML basics (learn this layout once)

| Area | Where | Use |
|---|---|---|
| **Toolbox** | left side | pick shapes/relationships (changes with the diagram type) |
| **Diagram area** | centre | where you draw |
| **Model Explorer** | top right | tree of all elements + diagrams; right-click here to add diagrams |
| **Properties / Editors** | bottom right | rename, set visibility, type, multiplicity, stereotype |

**Golden rules**
1. Click a Toolbox item, then **click once on the canvas** to drop a shape.
2. For relationships: click the Toolbox item, then **click the source, then click the target**.
3. **Double-click** a shape (or a line) to edit its name.
4. Press **Esc** (or click the arrow at the top of the Toolbox) to go back to the selection tool.
5. Drag the shape's corner handles to resize.

**Start a project**
1. Open StarUML (if a license/evaluation popup appears, just close it or choose evaluation — it works for the exam).
2. **File → New** (Ctrl+N) → a blank project with an `Untitled` model is created.
3. **File → Save** (Ctrl+S) → save as `<rollno>_uml.mdj` right now, and press Ctrl+S after each diagram.

**Add a diagram**: **Model → Add Diagram →** *Class / Sequence / Use Case / Component Diagram* (or right-click the model in Model Explorer → **Add Diagram**).
Rename it: select it in Model Explorer, edit the name in Properties.

**Export for submission**: open the diagram → **File → Export Diagram As → PNG** (or PDF). **File → Export All Diagrams As** exports all of them at once.

---

## 1. Use Case Diagram

**What you'll draw:** 2 actors, a system boundary, 9 use cases, associations, `<<include>>`, `<<extend>>`.

### Steps
1. **Model → Add Diagram → Use Case Diagram**. Rename it `Use Case Diagram - Schizophrenia Detection`.
2. **System boundary:** Toolbox → **Subject** (the big rectangle) → drag a large rectangle in the centre. Double-click, name it `Schizophrenia Detection System`.
3. **Actors:** Toolbox → **Actor** → click left of the boundary → name `Doctor`. Click again on the right → name `Researcher`. (Actors stay **outside** the boundary.)
4. **Use cases:** Toolbox → **UseCase** → click **inside** the boundary → name it. Repeat for each:

| Use case name |
|---|
| Login |
| Upload 3D sMRI (.nii) |
| Enter Age and Sex |
| Pre-process MRI |
| Predict Schizophrenia |
| View Confidence Score |
| View Explanation |
| Download PDF Report |
| Analyze Model Results |

5. **Associations (actor ↔ use case):** Toolbox → **Association** → click the actor, then the use case.
   - Doctor → Login, Upload 3D sMRI, Predict Schizophrenia, View Confidence Score, Download PDF Report
   - Researcher → Login, Upload 3D sMRI, Analyze Model Results
6. **Include** (mandatory sub-step): Toolbox → **Include** → click the *base* use case, then the *included* one:
   - Predict Schizophrenia → Pre-process MRI
   - Upload 3D sMRI → Enter Age and Sex
   - Predict Schizophrenia → View Confidence Score
   
   StarUML labels the dashed arrow `«include»` automatically.
7. **Extend** (optional behaviour): Toolbox → **Extend** → click the *extension* use case, then the *base*:
   - View Explanation → Predict Schizophrenia
   - Download PDF Report → Predict Schizophrenia
8. Tidy up: drag shapes so no lines cross; resize the boundary to contain all use cases. Save.

### Checklist
- Actors outside the boundary, use cases inside
- Every functional requirement from your SRS appears as a use case
- Include arrows point **base → included**; extend arrows point **extension → base**

---

## 2. Class Diagram

**What you'll draw:** 11 classes, attributes + methods, inheritance, associations with multiplicity, composition.

### Steps
1. **Model → Add Diagram → Class Diagram**. Rename it.
2. **Create classes:** Toolbox → **Class** → click on canvas → type the name → Enter. Create: `User`, `Doctor`, `Researcher`, `Patient`, `MRIScan`, `Preprocessor`, `HybridModel`, `CNN`, `VisionTransformer`, `Prediction`, `Report`.
3. **Add attributes:** right-click the class → **Add → Attribute**, type it and press Enter. Use this format: `- name : Type`
   (If StarUML doesn't parse it, select the attribute and set **Visibility** and **Type** in the Properties panel instead.)
4. **Add operations:** right-click the class → **Add → Operation**, type e.g. `+ login() : boolean` and Enter.

Fill in each class:

| Class | Attributes | Operations |
|---|---|---|
| User | `- userId : int`, `- name : String`, `- email : String`, `- password : String` | `+ login() : boolean`, `+ logout() : void` |
| Doctor | *(inherits)* | `+ uploadScan() : void`, `+ viewResult() : Prediction`, `+ downloadReport() : File` |
| Researcher | *(inherits)* | `+ analyzeResults() : void` |
| Patient | `- patientId : int`, `- age : int`, `- sex : String` | — |
| MRIScan | `- scanId : int`, `- filePath : String`, `- format : String` | `+ validate() : boolean` |
| Preprocessor | — | `+ normalize(scan : MRIScan) : Tensor`, `+ resize(scan : MRIScan) : Tensor` |
| HybridModel | — | `+ predict(img : Tensor, age : int, sex : String) : Prediction` |
| CNN | — | `+ extractLocalFeatures() : Tensor` |
| VisionTransformer | — | `+ extractGlobalFeatures() : Tensor` |
| Prediction | `- label : String`, `- confidence : float` | `+ explain() : String` |
| Report | `- reportId : int`, `- generatedOn : Date` | `+ generatePDF() : File` |

5. **Inheritance:** Toolbox → **Generalization** → click the **child** (Doctor), then the **parent** (User). Repeat for Researcher → User.
6. **Associations:** Toolbox → **Association** → click class A, then class B. Then set multiplicity:
   - Select the line → in **Properties** open **End1 / End2** (or select the line's end in Model Explorer) → set **Multiplicity** (`1`, `*`, `0..1`, `1..*`) and optionally a **Name** on the line (e.g. `treats`).

| From | To | End 1 | End 2 | Label |
|---|---|---|---|---|
| Doctor | Patient | 1 | * | treats |
| Patient | MRIScan | 1 | * | has |
| MRIScan | Preprocessor | * | 1 | processed by |
| HybridModel | Prediction | 1 | * | produces |
| Prediction | Report | 1 | 1 | included in |

7. **Composition** (filled diamond): Toolbox → **Composition** → click the **whole** (HybridModel), then the **part** (CNN). Repeat for VisionTransformer.
   (Whole–part where parts can exist alone → **Aggregation**, hollow diamond.)
8. **Dependency** (dashed arrow): Toolbox → **Dependency** → Preprocessor → HybridModel (feeds).
9. Arrange the classes neatly: parents above children. Save.

### Checklist
- 3 compartments per class (name / attributes / operations)
- Visibility symbols `+ - #` present
- Hollow-triangle inheritance arrows point at the **parent**
- Multiplicities on all associations

---

## 3. Sequence Diagram

**Scenario to draw:** *Doctor uploads an MRI, gets prediction and downloads the report* (one scenario only; time flows top → bottom).

### Steps
1. **Model → Add Diagram → Sequence Diagram**. Rename it `Sequence Diagram - MRI Prediction`.
2. **Lifelines (left → right):** Toolbox → **Lifeline** → click near the top of the canvas → name it. Space them evenly in this order:
   `Doctor`, `Web UI`, `Server`, `Preprocessor`, `Hybrid Model`, `Report Generator`
   (If your Toolbox has an **Actor** lifeline use it for Doctor; otherwise a lifeline named `Doctor` is accepted.)
3. **Messages:** Toolbox → **Message** (solid arrow) → click the sender's lifeline, then the receiver's lifeline *at the height where the message should occur*. Double-click the arrow to type its label. Add them top to bottom:

| # | From → To | Label | Type |
|---|---|---|---|
| 1 | Doctor → Web UI | `upload(scan.nii, age, sex)` | Message |
| 2 | Web UI → Server | `submit(scan, age, sex)` | Message |
| 3 | Server → Server | `validate(scan)` | **Self Message** tool |
| 4 | Server → Preprocessor | `preprocess(scan)` | Message |
| 5 | Preprocessor → Server | `normalized tensor` | **Reply Message** (dashed) |
| 6 | Server → Hybrid Model | `predict(tensor, age, sex)` | Message |
| 7 | Hybrid Model → Server | `label, confidence, explanation` | Reply |
| 8 | Server → Web UI | `result` | Reply |
| 9 | Web UI → Doctor | `display prediction, confidence, explanation` | Reply |
| 10 | Doctor → Web UI | `downloadReport()` | Message |
| 11 | Web UI → Report Generator | `generatePDF(result)` | Message |
| 12 | Report Generator → Web UI | `report.pdf` | Reply |
| 13 | Web UI → Doctor | `PDF downloaded` | Reply |

4. **Invalid-file branch (alt fragment):** Toolbox → **Combined Fragment** → drag a box over messages 4–13 (and a space for the error path).
   - Select the fragment → Properties → **InteractionOperator = `alt`**.
   - Right-click the fragment → **Add → Operand** to get two sections; set each **Guard**: `[invalid file]` and `[valid file]`.
   - In the `[invalid file]` operand add: Server → Web UI `error("Invalid .nii file")` (Reply), Web UI → Doctor `show error`.
   - Keep messages 4–13 inside the `[valid file]` operand.
5. Check that arrows are horizontal, messages are in time order, and every arrow has a label. Save.

### Checklist
- Dashed vertical lifelines under each participant box
- Solid arrows = calls, dashed arrows = returns
- Starts and ends with the Doctor
- `alt` frame shows the error path

---

## 4. Component Diagram

**What you'll draw:** the software's main building blocks and how they connect through interfaces.

### Components for this system
| Component | Responsibility |
|---|---|
| Web UI | Upload form, results page, download button |
| Authentication Service | Login / session |
| Upload Controller (REST API) | Receives scan, age, sex; validates `.nii` |
| Preprocessing Service | Normalise + resize MRI |
| Hybrid Model Service | CNN + ViT + feature fusion → prediction |
| Explainability Module | Visual/textual explanation |
| Report Generator | Builds PDF |
| Storage (Database / File Store) | Scans, results, reports |

### Steps
1. **Model → Add Diagram → Component Diagram**. Rename it.
2. **Components:** Toolbox → **Component** → click canvas → name it. Create all 8 above. Lay them out in 3 layers: *Presentation* (Web UI) on the left → *Application* (Auth, Upload Controller, Preprocessing, Hybrid Model, Explainability, Report Generator) in the middle → *Data* (Storage) on the right.
3. **Interfaces (provided = lollipop):** Toolbox → **Interface** → click canvas → name it. Create:
   `IUpload`, `IPredict`, `IExplain`, `IReport`, `IStorage`, `IAuth`.
4. **Provided interface:** Toolbox → **Interface Realization** → click the *component*, then the *interface* (draws the lollipop). Examples:
   - Upload Controller → `IUpload`
   - Hybrid Model Service → `IPredict`
   - Explainability Module → `IExplain`
   - Report Generator → `IReport`
   - Storage → `IStorage`
   - Authentication Service → `IAuth`
5. **Required interface (socket):** Toolbox → **Dependency** → click the *component that needs it*, then the *interface*. Examples:
   - Web UI → `IAuth`, `IUpload`, `IReport`
   - Upload Controller → `IStorage`
   - Upload Controller → Preprocessing Service (direct dependency, dashed arrow)
   - Preprocessing Service → `IPredict`
   - Hybrid Model Service → `IExplain`
   - Report Generator → `IStorage`
6. (Optional) **Ports:** Toolbox → **Port** → click on a component's edge to add a port for an interface.
7. (Optional) Group layers with **Package**/**Frame**, e.g. `Presentation Layer`, `Application Layer`, `Data Layer`.
8. Add a stereotype if asked: select a component → Properties → **Stereotype** (e.g. `service`). Save.

### Checklist
- Each component has the component icon (two small rectangles)
- Every interface is provided by exactly one component and used by others
- Dashed arrows show dependencies; direction = "uses"

---

## 5. Different system? Same steps, new names

| Diagram | Take from the problem statement |
|---|---|
| Use Case | users → actors; features → use cases; system name → Subject |
| Class | nouns → classes; their data → attributes; verbs → operations |
| Sequence | pick one feature; participants in order of first involvement |
| Component | major modules/services/DB/UI → components; their APIs → interfaces |

---

## 6. Final exam routine (5 minutes at the end)

1. Rename every diagram (Properties) and give the project a title.
2. Align shapes: select several → right-click → **Align / Space evenly** if you need to tidy.
3. **File → Save** (Ctrl+S).
4. **File → Export Diagram As → PNG/PDF** for the diagram you were asked to draw (take a screenshot as backup).
5. Make sure nothing overlaps and all labels are readable.
