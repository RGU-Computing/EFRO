# 🎓 EFRO: Education Funding Regulations Ontology

EFRO is a semantic framework developed to support education providers, auditors, and knowledge engineers in modelling, interpreting, and validating **UK education funding regulations**. Think of EFRO as an operating system for **funding compliance** in post-compulsory education. It provides a machine-interpretable structure of policies, monitoring activities, evidencing requirements, and institutional obligations.

---

## 📘 Description

Educational institutions face increasing complexity in complying with funding rules issued by government bodies like the UK Department for Education. While LLMs can assist with text understanding, their hallucination risks make them unsuitable for compliance-critical scenarios. EFRO mitigates this by modelling funding rules as ontological constructs that support SPARQL-based compliance checking, policy reasoning, and evidence tracking.

---

## ⚙️ Features

### 🔹 Main Classes

EFRO provides comprehensive coverage of key funding and compliance elements:

- **Student**: A learner enrolled in a programme.
- **StudyProgramme**: A funded course or learning aim.
- **Traineeship**: A specialised study programme eligible for early progression rules.
- **Activity**: Events such as withdrawal, re-enrolment, or progression.
- **ProgressionActivity**: Movement into employment, training, or apprenticeship.
- **FundingCondition**: Regulatory constraints tied to funding.
- **Status, EvidenceDocument, ComplianceCriterion**: For monitoring and auditing.

### 🔸 Object Properties

- `enrolledIn`: Connects students to programmes.
- `hasQualifyingPeriod`: Defines duration requirements.
- `hasProgressionActivity`: Links a student to what they progressed into.
- `recordedIn`: Tracks where evidence is logged.
- `hasFundingImpact`: Relates actions to funding outcomes.

### 🔸 Data Properties

- `hasPlannedHours`, `hasQualifyingPeriodDuration`
- `daysAttendedBeforeWithdrawal`
- `plannedHoursRetained`, `progressedInWeek`

---

## 📂 Quick Access

| Resource                      | Link |
|------------------------------|------|
| 🔗 RDF/OWL Ontology           | [EFRO.owl](./EFRO.owl) |
| 📖 HTML Documentation         | [View Docs](https://rgu-computing.github.io/EFRO/) |
| 🧪 Competency Questions       | [Competency_Questions.md](./Competency_Questions.md) |
| 📄 License (CC BY 4.0)        | [View License](https://creativecommons.org/licenses/by/4.0/) |

---

## 🧾 Citation

> Umair Arshad et al., “**EFRO: An Ontology for Education Funding Compliance**,” submitted to ISWC 2025.  
> GitHub Repo: [https://github.com/RGU-Computing/EFRO](https://github.com/RGU-Computing/EFRO)

---

## 🤝 Get Involved

We welcome feedback, use cases, and contributions.  
Open an issue or fork this repo to contribute.

### Installation & Setup

```bash
git clone https://github.com/RGU-Computing/EFRO.git

