# Q1 — Writing an SRS Document

The exam gives you a **problem statement** and asks for specific parts (abstract, functional requirements, non-functional requirements, etc.). Follow the public **IEEE 830 / "SRS template"** structure and write only the parts asked, in the template's order and numbering.

---

## 1. Method: problem statement → SRS in 6 steps

1. **Underline actors** (who uses it) → these become *Users / User classes*.
2. **Underline verbs** ("upload", "display", "generate", "download") → each becomes one *Functional Requirement* (FR).
3. **Underline qualities** ("within 3–5 seconds", "multiple users", "secure", "user-friendly") → each becomes a *Non-Functional Requirement* (NFR).
4. **Underline nouns/inputs/outputs** (MRI scan, age, sex, PDF report) → *Inputs/Outputs, Interfaces, Definitions*.
5. **Write the abstract**: *problem → existing limitation → proposed solution → inputs → processing → outputs*.
6. **Number everything** (FR-1, FR-2… NFR-1…) and use "The system shall …" wording.

---

## 2. Blank template (copy this and fill in)

```markdown
# Software Requirements Specification (SRS)
## for <Project Title>

Version 1.0 | Prepared by: <Name, Roll No> | Date: <dd/mm/yyyy>

## Abstract / Problem Statement
<Domain + problem in 2 lines>. <Limitation of existing system>. This project proposes
<your system> that accepts <inputs>, performs <processing>, and produces <outputs>
to <benefit>.

## 1. Introduction
### 1.1 Purpose
This document specifies the software requirements of <system name>. It is intended for
<developers, testers, doctors/clients, examiners>.
### 1.2 Scope
The system <does what>. It <does NOT do what>. Benefits: <list>.
### 1.3 Definitions, Acronyms and Abbreviations
| Term | Meaning |
|---|---|
| SRS | Software Requirements Specification |
| <term> | <meaning> |
### 1.4 References
- IEEE Std 830-1998, Recommended Practice for Software Requirements Specifications
- <any dataset / paper / website>
### 1.5 Overview
Section 2 gives the overall description; Section 3 lists specific requirements.

## 2. Overall Description
### 2.1 Product Perspective
<Standalone / web-based / part of larger system. Mention client–server, DB, ML model.>
### 2.2 Product Functions (summary)
- <function 1>
- <function 2>
### 2.3 User Classes and Characteristics
| User | Description | Technical level |
|---|---|---|
| <User 1> | <role> | <level> |
### 2.4 Operating Environment
Browser (Chrome/Edge), OS (Windows/Linux), server (Tomcat / Flask), Java/Python version, DB.
### 2.5 Design and Implementation Constraints
<language, framework, input format, hardware, regulations>
### 2.6 Assumptions and Dependencies
<internet available, valid input format, trained model available …>

## 3. Specific Requirements
### 3.1 External Interface Requirements
- **User Interface:** <screens, upload form, result page>
- **Hardware Interface:** <PC/server, GPU if needed>
- **Software Interface:** <OS, DB, libraries, browser>
- **Communication Interface:** HTTP/HTTPS
### 3.2 Functional Requirements
| ID | Requirement |
|---|---|
| FR-1 | The system shall <verb> … |
| FR-2 | The system shall … |
### 3.3 Non-Functional Requirements
| ID | Category | Requirement |
|---|---|---|
| NFR-1 | Performance | The system shall … within <n> seconds |
| NFR-2 | Scalability | The system shall support multiple concurrent users |
| NFR-3 | Reliability | … |
| NFR-4 | Usability | … |
| NFR-5 | Security | … |
| NFR-6 | Maintainability | … |
### 3.4 Other Requirements (optional)
<Legal/ethical, database requirements, logging>

## 4. Appendix (optional)
<Diagrams: use case, class, sequence, activity>
```

---

## 3. Worked example (from your manual: Schizophrenia Detection)

```markdown
# Software Requirements Specification (SRS)
## for Deep Learning Model for Schizophrenia Detection Using 3D Structural MRI

Version 1.0

## Abstract / Problem Statement
Schizophrenia is a severe, chronic mental disorder affecting thinking, emotions and
behaviour. Early and accurate diagnosis improves treatment outcomes, but conventional
diagnosis relies on clinical interviews and behavioural assessments, which are
time-consuming and subjective. The existing SE-VGG-11-BN model uses only structural MRI,
excludes clinical information (age, sex), and lacks interpretability, so users cannot
trust its predictions. This project proposes a hybrid deep-learning diagnostic system that
accepts a 3D structural MRI scan (.nii), the patient's age and biological sex. The MRI is
normalised and resized, then passed to a hybrid CNN + Vision Transformer model: the CNN
extracts local anatomical features and the ViT captures global context. Age and sex are
processed in parallel and fused with the image features to improve prediction. The system
outputs a prediction, a confidence score, an explanation, and a downloadable PDF report.

## 1. Introduction
### 1.1 Purpose
This SRS specifies the requirements of the Schizophrenia Detection System, intended for
developers, testers, clinicians and researchers.
### 1.2 Scope
A web-based decision-support tool that classifies a patient's sMRI as schizophrenia /
healthy control, shows confidence and explanation, and generates a report. It is a
decision-support aid and does not replace a clinician's diagnosis.
### 1.3 Definitions, Acronyms and Abbreviations
| Term | Meaning |
|---|---|
| sMRI | Structural Magnetic Resonance Imaging |
| NIfTI (.nii) | Neuroimaging Informatics Technology Initiative file format |
| CNN | Convolutional Neural Network |
| ViT | Vision Transformer |
| SRS | Software Requirements Specification |
### 1.4 References
- IEEE Std 830-1998
- SE-VGG-11-BN schizophrenia detection paper (baseline model)
### 1.5 Overview
Section 2 describes the product; Section 3 lists detailed requirements.

## 2. Overall Description
### 2.1 Product Perspective
A standalone web application with a browser front end, a server-side back end, and a
trained hybrid CNN-ViT model for inference.
### 2.2 Product Functions
- Upload 3D sMRI (.nii) with age and sex
- Pre-process the scan (normalise, resize)
- Predict using the hybrid model and show confidence
- Explain the prediction visually/textually
- Generate and download a PDF report
### 2.3 User Classes and Characteristics
| User | Description |
|---|---|
| Doctor | Uses the system to assist clinical diagnosis and analysis |
| Researcher | Studies schizophrenia using MRI-based deep-learning models |
### 2.4 Operating Environment
Modern web browser; server with Python, PyTorch/TensorFlow; Windows/Linux; GPU optional.
### 2.5 Design and Implementation Constraints
Input must be .nii; model accepts fixed-size volumes after resizing; patient data must be
handled securely.
### 2.6 Assumptions and Dependencies
Trained model weights are available; user provides a valid, correctly oriented scan.

## 3. Specific Requirements
### 3.1 External Interface Requirements
- **User Interface:** upload form (file + age + sex), result page, download button
- **Hardware Interface:** server (CPU/GPU)
- **Software Interface:** Python ML libraries, web framework, PDF library
- **Communication Interface:** HTTPS
### 3.2 Functional Requirements
| ID | Requirement |
|---|---|
| FR-1 | The system shall provide a user interface to upload a 3D sMRI scan in .nii format. |
| FR-2 | The system shall accept the patient's age and biological sex as inputs. |
| FR-3 | The system shall pre-process the scan (normalisation and resizing). |
| FR-4 | The system shall display the prediction and its confidence score. |
| FR-5 | The system shall provide a visual or textual explanation of the prediction. |
| FR-6 | The system shall generate and allow download of a PDF report. |
### 3.3 Non-Functional Requirements
| ID | Category | Requirement |
|---|---|---|
| NFR-1 | Performance | Results shall be generated within 3–5 seconds. |
| NFR-2 | Scalability | The system shall support multiple simultaneous users. |
| NFR-3 | Reliability | Predictions shall be consistent and accurate for the same input. |
| NFR-4 | Usability | The interface shall be user-friendly. |
| NFR-5 | Security | Uploaded scans and patient information shall be handled securely. |
```

---

## 4. Phrase bank (reuse in any problem)

**Functional (use "The system shall …")**
- allow users to register / log in / log out
- accept <file/input> and validate its format and size
- display <result> to the user
- store/retrieve/update/delete <records>
- search/filter <items> by <criteria>
- generate and download a <report/receipt/PDF>
- send notification/e-mail on <event>
- allow admin to manage users and roles

**Non-functional (category → sentence)**
- **Performance:** respond to user requests within <n> seconds.
- **Scalability:** support at least <n> concurrent users.
- **Reliability/Availability:** be available 99% of the time; produce consistent results.
- **Security:** authenticate users, encrypt data in transit (HTTPS) and at rest, restrict access by role.
- **Usability:** be learnable by a new user within <n> minutes; provide clear error messages.
- **Maintainability:** use modular design with documented code.
- **Portability:** run on Windows/Linux and major browsers.

**Tips**
- Each requirement must be **testable** — use numbers where the statement gives them.
- Don't mix FR and NFR: *what it does* vs *how well it does it*.
- Never invent requirements that contradict the problem statement; adding 1–2 sensible extras (login, error handling) is fine.
- Keep the section numbers exactly as the template — examiners look for them.
