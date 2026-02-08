# DICOM Developer Guide

## Table of Contents

1. [Introduction to DICOM](#introduction-to-dicom)
2. [DICOM Standard Parts](#dicom-standard-parts)
3. [DICOM Information Model](#dicom-information-model)
4. [DICOM Fundamentals](#dicom-fundamentals)
5. [DICOM File Format](#dicom-file-format)
6. [DICOM Data Elements and Tags](#dicom-data-elements-and-tags)
7. [Sequences and Nested Data Structures](#sequences-and-nested-data-structures)
8. [Value Representations (VR)](#value-representations-vr)
9. [Transfer Syntaxes](#transfer-syntaxes)
10. [UID Structure and Organization](#uid-structure-and-organization)
11. [Character Set Support](#character-set-support)
12. [DICOM Networking](#dicom-networking)
13. [DICOM Services (DIMSE)](#dicom-services-dimse)
14. [Information Object Definitions (IODs)](#information-object-definitions-iods)
15. [Service-Object Pair (SOP) Classes](#service-object-pair-sop-classes)
16. [Conformance Statements](#conformance-statements)
17. [DICOMweb](#dicomweb)
18. [DICOM Structured Reporting (SR)](#dicom-structured-reporting-sr)
19. [Modality Worklist and MPPS](#modality-worklist-and-mpps)
20. [Storage Commitment](#storage-commitment)
21. [DICOM Printing](#dicom-printing)
22. [Presentation States](#presentation-states)
23. [Key Object Selection Documents](#key-object-selection-documents)
24. [Enhanced Multi-frame IODs](#enhanced-multi-frame-iods)
25. [Encapsulated Documents](#encapsulated-documents)
26. [Waveform and Signal Data](#waveform-and-signal-data)
27. [Overlay Data](#overlay-data)
28. [Radiotherapy (RT) Objects](#radiotherapy-rt-objects)
29. [IHE Integration Profiles](#ihe-integration-profiles)
30. [DICOMDIR and Media Storage](#dicomdir-and-media-storage)
31. [Working with DICOM Files](#working-with-dicom-files)
32. [DICOM Libraries and Tools](#dicom-libraries-and-tools)
33. [Code Examples](#code-examples)
34. [Best Practices](#best-practices)
35. [Common Pitfalls](#common-pitfalls)
36. [Security Considerations](#security-considerations)
37. [Resources and References](#resources-and-references)

---

## Introduction to DICOM

### What is DICOM?

**DICOM** (Digital Imaging and Communications in Medicine) is the international standard for medical images and related information. It defines the formats for medical images that can be exchanged with the data and quality necessary for clinical use.

### History and Purpose

- **Established**: 1993 by NEMA (National Electrical Manufacturers Association)
- **Purpose**: To standardize the communication of medical imaging information
- **Scope**: Covers image formats, network protocols, and information models
- **Current Version**: DICOM is continuously updated with supplements and corrections

### Why DICOM Matters for Developers

DICOM is crucial for:
- Medical imaging system integration
- PACS (Picture Archiving and Communication Systems)
- Medical image viewers
- Radiology Information Systems (RIS)
- Electronic Health Records (EHR) integration
- AI/ML applications in medical imaging
- Telemedicine applications

---

## DICOM Standard Parts

The DICOM Standard is organized into multiple parts, each addressing a specific aspect of the standard. Understanding the structure helps navigate the full specification.

| Part | Title | Description |
|------|-------|-------------|
| **PS3.1** | Introduction and Overview | General overview, history, scope, and outline of the entire standard |
| **PS3.2** | Conformance | How to write and interpret Conformance Statements (see [Conformance Statements](#conformance-statements)) |
| **PS3.3** | Information Object Definitions (IODs) | Defines the structure and attributes of all DICOM data objects (images, reports, waveforms, etc.) |
| **PS3.4** | Service Class Specifications | Defines Service Classes such as Storage, Query/Retrieve, Worklist, Print, etc., and how SCU/SCP interact |
| **PS3.5** | Data Structures and Encoding | Rules for encoding DICOM data elements, value representations, transfer syntaxes, and byte ordering |
| **PS3.6** | Data Dictionary | Complete registry of all standardized DICOM data elements (tags), their VRs, and descriptions |
| **PS3.7** | Message Exchange | Defines DIMSE protocol messages (C-STORE, C-FIND, C-MOVE, N-CREATE, etc.) and association negotiation |
| **PS3.8** | Network Communication Support | DICOM Upper Layer Protocol for TCP/IP, including connection establishment and PDU formats |
| **PS3.10** | Media Storage and File Format | How DICOM objects are stored as files, including the preamble, prefix, file meta information, and DICOMDIR |
| **PS3.11** | Media Storage Application Profiles | Profiles for storing DICOM on removable media (CD, DVD, USB) for specific clinical scenarios |
| **PS3.12** | Media Formats and Physical Media | Physical media specifications (file systems, disc formats) |
| **PS3.14** | Grayscale Standard Display Function | Defines the Grayscale Standard Display Function (GSDF) for consistent image display across monitors |
| **PS3.15** | Security and System Management Profiles | Security profiles including TLS, digital signatures, attribute confidentiality, and audit logging |
| **PS3.16** | Content Mapping Resource | Code sets, templates, and context groups used for coded entries (e.g., anatomy codes, procedure codes) |
| **PS3.17** | Explanatory Information | Non-normative explanatory annexes, use cases, and implementation guidance |
| **PS3.18** | Web Services (DICOMweb) | RESTful web services: WADO-RS, STOW-RS, QIDO-RS, and UPS-RS (see [DICOMweb](#dicomweb)) |
| **PS3.19** | Application Hosting | Framework for hosting image processing algorithms as plug-ins within DICOM applications |
| **PS3.20** | Imaging Reports using HL7 Clinical Document Architecture | How to encode radiology reports using HL7 CDA within DICOM |
| **PS3.21** | Transformations between DICOM and Other Representations | Mappings between DICOM and other formats (e.g., HL7 FHIR ImagingStudy resources) |
| **PS3.22** | Real-Time Communication | DICOM Real-Time Video and related services for surgical/procedural workflows |

### How to Read the DICOM Standard

1. **Start with PS3.1** for orientation
2. **Use PS3.6** (Data Dictionary) as a constant reference for tag lookups
3. **Refer to PS3.3** (IODs) to understand what attributes belong to each object type
4. **Check PS3.5** for encoding and parsing rules
5. **See PS3.4** for service class behavior (how C-STORE, C-FIND, etc., work)
6. **Consult PS3.17** for practical implementation guidance and use cases

The standard is freely available at [https://www.dicomstandard.org/current](https://www.dicomstandard.org/current).

---

## DICOM Information Model

### Patient-Study-Series-Instance Hierarchy

DICOM organizes medical imaging data in a strict four-level hierarchy:

```
┌──────────────────────────────────────────────────────────┐
│  PATIENT                                                  │
│  └── Identified by Patient ID (0010,0020)                │
│      ┌──────────────────────────────────────────────┐    │
│      │  STUDY                                        │    │
│      │  └── One imaging session / visit              │    │
│      │      Identified by Study Instance UID         │    │
│      │      (0020,000D)                              │    │
│      │      ┌──────────────────────────────────┐     │    │
│      │      │  SERIES                           │     │    │
│      │      │  └── One acquisition or group     │     │    │
│      │      │      Identified by Series Instance│     │    │
│      │      │      UID (0020,000E)              │     │    │
│      │      │      ┌────────────────────┐       │     │    │
│      │      │      │  INSTANCE (Image)   │       │     │    │
│      │      │      │  └── Single image   │       │     │    │
│      │      │      │      or object      │       │     │    │
│      │      │      │      SOP Instance   │       │     │    │
│      │      │      │      UID (0008,0018)│       │     │    │
│      │      │      └────────────────────┘       │     │    │
│      │      └──────────────────────────────────┘     │    │
│      └──────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

### Entity-Relationship Model

The DICOM Information Model uses an Entity-Relationship approach:

```
Patient ──(1:N)──> Study ──(1:N)──> Series ──(1:N)──> Instance
```

**Patient**: A real-world individual. Attributes include name, ID, date of birth, sex.

**Study**: Represents a single imaging encounter/visit. A patient can have multiple studies over time. Attributes include study date/time, referring physician, accession number, study description.

**Series**: A grouping within a study, typically one acquisition or one set of related images. Attributes include modality, series number, series description, body part examined.

**Instance**: A single DICOM object (image, structured report, presentation state, etc.). Each instance has a globally unique SOP Instance UID.

### Unique Identifiers in the Hierarchy

| Level | UID Tag | Tag Number | Scope |
|-------|---------|------------|-------|
| Patient | Patient ID | (0010,0020) | Local to institution |
| Study | Study Instance UID | (0020,000D) | Globally unique |
| Series | Series Instance UID | (0020,000E) | Globally unique |
| Instance | SOP Instance UID | (0008,0018) | Globally unique |

### Real-World Example

```
Patient: John Smith (ID: PAT001)
  └── Study: 2024-01-15 CT Chest (Study UID: 1.2.840.113619.2.55...)
      ├── Series 1: Axial 5mm (Series UID: 1.2.840.113619.2.55.1...)
      │   ├── Instance 1 (Slice 1)
      │   ├── Instance 2 (Slice 2)
      │   └── ... (120 slices)
      ├── Series 2: Coronal Reformat (Series UID: 1.2.840.113619.2.55.2...)
      │   └── ... (60 slices)
      └── Series 3: Dose Report (Series UID: 1.2.840.113619.2.55.3...)
          └── Instance 1 (SR Dose Report)
```

### Composite and Normalized Information Models

DICOM defines two types of information models:

**Composite IODs**: Contain attributes from multiple real-world entities in a single object. For example, a CT Image IOD contains patient information, study information, series information, equipment information, and image data all in one object. This redundancy makes each file self-describing.

**Normalized IODs**: Contain attributes from only a single real-world entity. Used in services like Print Management and Modality Worklist, where objects represent single entities such as a Print Job or a Scheduled Procedure Step.

---

## DICOM Fundamentals

### Core Concepts

1. **Service-Object Pair (SOP)**: Combination of a DICOM service and an information object
2. **Application Entity (AE)**: A software component that communicates using DICOM
3. **Application Entity Title (AET)**: Unique identifier for an Application Entity
4. **Association**: Network connection between two Application Entities
5. **Presentation Context**: Agreement on what data and encoding to use during an association
6. **Service Class**: Defines the roles (SCU/SCP) and behavior for a group of operations

### DICOM Architecture Layers

```
┌─────────────────────────────────────────┐
│     DICOM Applications                   │
│     (Viewers, PACS, Modalities)          │
├─────────────────────────────────────────┤
│     Service Classes                      │
│     (Storage, Q/R, Worklist, Print)     │
├─────────────────────────────────────────┤
│     DICOM Message Services (DIMSE)      │
│     (C-STORE, C-FIND, N-CREATE, etc.)  │
├─────────────────────────────────────────┤
│     DICOM Upper Layer Protocol          │
│     (Association negotiation, PDUs)     │
├─────────────────────────────────────────┤
│     TCP/IP Transport                     │
└─────────────────────────────────────────┘
```

### Service Class Roles

Every DICOM communication involves two roles:

- **Service Class User (SCU)**: The entity initiating a request (client role)
- **Service Class Provider (SCP)**: The entity fulfilling the request (server role)

A single application can be both SCU and SCP simultaneously for different services. For example, a modality acts as an SCU when sending images (C-STORE SCU) but acts as an SCP when receiving worklist items (MWL SCP).

### DICOM Standard Parts Overview

- **Part 3**: Information Object Definitions
- **Part 4**: Service Class Specifications
- **Part 5**: Data Structures and Encoding
- **Part 6**: Data Dictionary
- **Part 7**: Message Exchange
- **Part 8**: Network Communication Support
- **Part 10**: Media Storage and File Format
- **Part 15**: Security Profiles
- **Part 16**: Content Mapping Resource
- **Part 18**: Web Services (DICOMweb)

See [DICOM Standard Parts](#dicom-standard-parts) for the complete list with descriptions.

---

## DICOM File Format

### File Structure

A DICOM file consists of:

1. **File Preamble** (128 bytes): Usually zeros, reserved for application use
2. **DICOM Prefix** (4 bytes): "DICM" in ASCII
3. **File Meta Information**: Contains metadata about the file
4. **Data Set**: The actual DICOM data elements

### File Meta Information

The File Meta Information includes:

- **Transfer Syntax UID** (0002,0010): Encoding rules
- **Media Storage SOP Class UID** (0002,0002): Type of DICOM object
- **Media Storage SOP Instance UID** (0002,0003): Unique identifier
- **Implementation Class UID** (0002,0012): Software that created the file
- **Implementation Version Name** (0002,0013): Version of the software

### Basic File Structure

```
┌────────────────────────────┐
│  File Preamble (128 bytes) │
├────────────────────────────┤
│  DICM Prefix (4 bytes)     │
├────────────────────────────┤
│  File Meta Information     │
│  - Transfer Syntax         │
│  - SOP Class UID           │
│  - SOP Instance UID        │
│  - etc.                    │
├────────────────────────────┤
│  Data Set                  │
│  - Patient Information     │
│  - Study Information       │
│  - Series Information      │
│  - Image Data              │
│  - etc.                    │
└────────────────────────────┘
```

---

## DICOM Data Elements and Tags

### Data Element Structure

Each DICOM data element consists of:

1. **Tag**: (Group, Element) - 4 bytes
2. **Value Representation (VR)**: 2 bytes (optional, depends on transfer syntax)
3. **Value Length**: 2 or 4 bytes
4. **Value Field**: Variable length

### Tag Format

Tags are written as `(GGGG,EEEE)` where:
- `GGGG` = Group number (hexadecimal)
- `EEEE` = Element number (hexadecimal)

Example: `(0010,0010)` = Patient's Name

### Common DICOM Tags

| Tag | Name | Description |
|-----|------|-------------|
| (0008,0018) | SOP Instance UID | Unique identifier for the instance |
| (0008,0020) | Study Date | Date of the study |
| (0008,0060) | Modality | Type of equipment (CT, MR, US, etc.) |
| (0010,0010) | Patient Name | Patient's full name |
| (0010,0020) | Patient ID | Primary hospital ID |
| (0010,0030) | Patient Birth Date | Patient's date of birth |
| (0010,0040) | Patient Sex | M, F, or O |
| (0020,000D) | Study Instance UID | Unique identifier for the study |
| (0020,000E) | Series Instance UID | Unique identifier for the series |
| (0020,0013) | Instance Number | Image number in a series |
| (0028,0010) | Rows | Number of rows in the image |
| (0028,0011) | Columns | Number of columns in the image |
| (0028,0100) | Bits Allocated | Number of bits allocated per pixel |
| (7FE0,0010) | Pixel Data | The actual image pixel data |

### Group Numbers

- **0000-0001**: Command and File Meta Information
- **0002-0003**: File Meta Information
- **0004-0007**: Directory Information
- **0008-0009**: Identifying Information
- **0010-0019**: Patient Information
- **0020-0029**: Study/Series/Image Relationship
- **0028-002F**: Image Presentation
- **0032-0038**: Study/Visit/Patient Relationship
- **0040-004F**: Procedure/Service Information
- **0050-0059**: Nuclear Medicine/PET
- **0060-0069**: Histogram/Segmentation/Registration
- **0070-0079**: Presentation State
- **7FE0-7FFF**: Pixel Data

---

## Sequences and Nested Data Structures

### What are Sequences?

Sequences (VR = SQ) are DICOM's mechanism for representing nested, hierarchical data. A sequence contains zero or more **Items**, and each item contains a set of data elements — essentially a dataset within a dataset. Sequences are essential for representing complex clinical information.

### Sequence Structure

```
Sequence Tag (e.g., Referenced Study Sequence)
├── Item 1
│   ├── Data Element A
│   ├── Data Element B
│   └── Nested Sequence
│       ├── Item 1.1
│       │   ├── Data Element C
│       │   └── Data Element D
│       └── Item 1.2
│           └── Data Element E
├── Item 2
│   ├── Data Element A
│   └── Data Element B
└── (Sequence Delimitation Item, if undefined length)
```

### Item Encoding

Each item in a sequence is encoded as:

| Component | Tag | Length |
|-----------|-----|--------|
| **Item** | (FFFE,E000) | Defined or undefined (FFFFFFFF) |
| **Item Delimitation Item** | (FFFE,E00D) | 0 (only if item has undefined length) |
| **Sequence Delimitation Item** | (FFFE,E0DD) | 0 (only if sequence has undefined length) |

### Common Sequences

| Tag | Name | Purpose |
|-----|------|---------|
| (0008,1115) | Referenced Series Sequence | Links to other series |
| (0008,1140) | Referenced Image Sequence | Links to other images |
| (0008,1110) | Referenced Study Sequence | Links to other studies |
| (0040,0260) | Performed Protocol Code Sequence | Coded protocol information |
| (0040,0275) | Request Attributes Sequence | Requested procedure info |
| (0008,0082) | Institution Code Sequence | Coded institution info |
| (0008,1084) | Admitting Diagnoses Code Sequence | Coded diagnoses |
| (0054,0016) | Radiopharmaceutical Information Sequence | Nuclear medicine info |
| (0070,0001) | Graphic Annotation Sequence | Annotations on images |
| (0040,A730) | Content Sequence | Structured Report content |

### Defined vs Undefined Length Sequences

**Defined Length**: Sequence length is specified in bytes. The parser reads exactly that many bytes.

**Undefined Length** (0xFFFFFFFF): The parser must read until it encounters the Sequence Delimitation Item tag (FFFE,E0DD). Similarly, items can have undefined length and use the Item Delimitation Item tag (FFFE,E00D).

### Coded Entries (Code Sequences)

Many sequences use a standardized pattern for coded values:

```
Code Sequence
└── Item
    ├── (0008,0100) Code Value          → e.g., "T-28000"
    ├── (0008,0102) Coding Scheme Designator → e.g., "SRT" (SNOMED-RT)
    ├── (0008,0103) Coding Scheme Version    → e.g., "1.0"
    └── (0008,0104) Code Meaning         → e.g., "Lung"
```

This pattern is used extensively in:
- Anatomy coded terms (Body Part)
- Procedure codes
- Finding codes in Structured Reports
- Laterality
- Device identifiers

---

## Value Representations (VR)

### What is VR?

Value Representation defines the data type and format of a DICOM element's value.

### Common VRs

| VR | Name | Description | Example |
|----|------|-------------|---------|
| AE | Application Entity | Application Entity Title | "PACS_SERVER" |
| AS | Age String | Age in format nnnD, nnnW, nnnM, nnnY | "025Y" |
| CS | Code String | Uppercase alphanumeric | "MR", "CT" |
| DA | Date | YYYYMMDD format | "20240101" |
| DS | Decimal String | Decimal number as string | "1.5" |
| DT | Date Time | YYYYMMDDHHMMSS.FFFFFF | "20240101120000.000000" |
| FD | Floating Point Double | 64-bit floating point | 3.14159 |
| FL | Floating Point Single | 32-bit floating point | 3.14 |
| IS | Integer String | Integer as string | "256" |
| LO | Long String | Character string (max 64 chars) | "Description text" |
| LT | Long Text | Text (max 10240 chars) | Long description |
| OB | Other Byte | Byte stream | Binary data |
| OD | Other Double | Double stream | Array of doubles |
| OF | Other Float | Float stream | Array of floats |
| OW | Other Word | Word stream (16-bit) | Pixel data |
| PN | Person Name | Person name format | "Doe^John" |
| SH | Short String | Short string (max 16 chars) | "HEAD" |
| SL | Signed Long | 32-bit signed integer | -123456 |
| SQ | Sequence | Sequence of items | Nested structures |
| SS | Signed Short | 16-bit signed integer | -1234 |
| ST | Short Text | Short text (max 1024 chars) | Text description |
| TM | Time | HHMMSS.FFFFFF | "120000.000000" |
| UI | Unique Identifier | UID format | "1.2.840.10008.5.1.4.1.1.2" |
| UL | Unsigned Long | 32-bit unsigned integer | 123456 |
| UN | Unknown | Unknown VR | Raw data |
| US | Unsigned Short | 16-bit unsigned integer | 1234 |
| UT | Unlimited Text | Very long text | Extended text |

### Person Name (PN) Format

Person names follow a specific format with components separated by "^":

```
FamilyName^GivenName^MiddleName^Prefix^Suffix
```

Example: `"Smith^John^Robert^Dr^Jr"`

---

## Transfer Syntaxes

### What is Transfer Syntax?

Transfer Syntax defines how DICOM data is encoded and compressed. It specifies:
- Byte ordering (little-endian or big-endian)
- Explicit or Implicit VR
- Compression method (if any)

### Common Transfer Syntaxes

| UID | Name | Description |
|-----|------|-------------|
| 1.2.840.10008.1.2 | Implicit VR Little Endian | Default, no compression |
| 1.2.840.10008.1.2.1 | Explicit VR Little Endian | Most common, no compression |
| 1.2.840.10008.1.2.2 | Explicit VR Big Endian | Big-endian byte ordering |
| 1.2.840.10008.1.2.4.50 | JPEG Baseline (Process 1) | Lossy JPEG compression |
| 1.2.840.10008.1.2.4.51 | JPEG Extended (Process 2 & 4) | Lossy JPEG compression |
| 1.2.840.10008.1.2.4.57 | JPEG Lossless, Non-Hierarchical (Process 14) | Lossless JPEG |
| 1.2.840.10008.1.2.4.70 | JPEG Lossless, Non-Hierarchical, First-Order Prediction | Lossless JPEG |
| 1.2.840.10008.1.2.4.80 | JPEG-LS Lossless | Lossless JPEG-LS |
| 1.2.840.10008.1.2.4.81 | JPEG-LS Lossy (Near-Lossless) | Near-lossless JPEG-LS |
| 1.2.840.10008.1.2.4.90 | JPEG 2000 Lossless | Lossless JPEG 2000 |
| 1.2.840.10008.1.2.4.91 | JPEG 2000 | Lossy JPEG 2000 |
| 1.2.840.10008.1.2.5 | RLE Lossless | Run-Length Encoding |

### Implicit vs Explicit VR

**Implicit VR**:
- VR is not explicitly specified in the data
- Must be looked up in the DICOM data dictionary
- More compact but requires dictionary knowledge

**Explicit VR**:
- VR is explicitly included in each data element
- Self-describing format
- Slightly larger file size but more robust

---

## UID Structure and Organization

### What are UIDs?

Unique Identifiers (UIDs) are globally unique strings used throughout DICOM to identify objects, services, and transfer syntaxes. They ensure that every entity can be unambiguously identified worldwide.

### UID Format

UIDs follow a dot-separated numeric format defined by ISO 8824:

```
<org-root>.<suffix>

Example: 1.2.840.10008.5.1.4.1.1.2
         │ │ │   │     └── DICOM-defined suffix
         │ │ │   └── Organization (NEMA/Medical)
         │ │ └── Country (840 = USA)
         │ └── Member body type (2 = branch)
         └── ISO (1)
```

### UID Rules

1. Maximum length: **64 characters**
2. Characters allowed: **digits (0-9) and periods (.)** only
3. Each component must not have leading zeros (except "0" itself)
4. Must be globally unique — once assigned, never reused
5. UIDs must not end with a period

### UID Categories

| Root | Purpose | Example |
|------|---------|---------|
| 1.2.840.10008.* | DICOM Standard UIDs | SOP Classes, Transfer Syntaxes |
| 1.2.840.113619.* | GE Healthcare | Vendor-specific private UIDs |
| 1.2.840.113681.* | Philips Healthcare | Vendor-specific private UIDs |
| 1.3.6.1.4.1.* | IANA Private Enterprise | Organization-specific UIDs |
| 2.25.* | UUID-derived UIDs | Random/generated UIDs |

### Well-Known DICOM UIDs

**SOP Class UIDs** identify the type of DICOM object or service:
- `1.2.840.10008.1.1` — Verification SOP Class (C-ECHO)
- `1.2.840.10008.5.1.4.1.1.2` — CT Image Storage
- `1.2.840.10008.5.1.4.1.1.4` — MR Image Storage
- `1.2.840.10008.5.1.4.1.2.2.1` — Study Root Q/R Information Model - FIND

**Transfer Syntax UIDs** identify data encoding:
- `1.2.840.10008.1.2` — Implicit VR Little Endian
- `1.2.840.10008.1.2.1` — Explicit VR Little Endian
- `1.2.840.10008.1.2.4.50` — JPEG Baseline

### UID Generation Methods

1. **Organization root + sequential suffix**: Use your registered OID root and append incrementing numbers
2. **UUID-derived (2.25.*)**: Convert a UUID (128-bit) to a decimal number and prefix with "2.25."
3. **Hash-based**: Derive from object content for reproducibility (non-standard but common)
4. **Timestamp-based**: Use date/time components as part of the suffix

### Instance UID Relationships

```
Study Instance UID: 1.2.840.113619.2.55.3.123456
  └── Series Instance UID: 1.2.840.113619.2.55.3.123456.1
      ├── SOP Instance UID: 1.2.840.113619.2.55.3.123456.1.1
      ├── SOP Instance UID: 1.2.840.113619.2.55.3.123456.1.2
      └── SOP Instance UID: 1.2.840.113619.2.55.3.123456.1.3
```

Note: While UID components may suggest hierarchy, the standard does not require this — each UID must simply be globally unique.

---

## Character Set Support

### Default Character Repertoire

By default, DICOM uses the **ISO 646** character repertoire (essentially ASCII). This covers basic Latin characters, digits, and common punctuation.

### Specific Character Set (0008,0005)

To support international characters, DICOM uses the **Specific Character Set** attribute to declare the character encoding used in string VR values.

### Supported Character Sets

| Value | Character Set | Description |
|-------|--------------|-------------|
| (empty/absent) | ISO IR 6 | Default ASCII repertoire |
| ISO_IR 100 | ISO 8859-1 | Latin-1 (Western European) |
| ISO_IR 101 | ISO 8859-2 | Latin-2 (Central European) |
| ISO_IR 109 | ISO 8859-3 | Latin-3 (South European) |
| ISO_IR 110 | ISO 8859-4 | Latin-4 (North European) |
| ISO_IR 144 | ISO 8859-5 | Cyrillic |
| ISO_IR 127 | ISO 8859-6 | Arabic |
| ISO_IR 126 | ISO 8859-7 | Greek |
| ISO_IR 138 | ISO 8859-8 | Hebrew |
| ISO_IR 148 | ISO 8859-9 | Latin-5 (Turkish) |
| ISO_IR 13 | JIS X 0201 | Japanese (Katakana) |
| ISO_IR 166 | TIS 620-2533 | Thai |
| ISO_IR 192 | Unicode (UTF-8) | Universal character set |
| GB18030 | GB 18030 | Chinese (PRC) |
| GBK | GBK | Chinese (simplified) |

### Multi-Valued Character Sets

When multiple character sets are needed (e.g., for Japanese which requires Kanji + Romaji), the Specific Character Set can be multi-valued:

```
Specific Character Set: ISO 2022 IR 6\ISO 2022 IR 87
```

This uses ISO 2022 escape sequences to switch between character sets within a single value.

### Character Set Handling Rules

1. **Specific Character Set applies only to string VRs**: SH, LO, PN, ST, LT, UT, UC, UR
2. **Does not affect binary VRs**: UI, DA, TM, AS, CS, DS, IS always use the default repertoire
3. **Each data element is decoded independently** based on the declared character set
4. **Person Name (PN) VR has special rules**: Three component groups (alphabetic, ideographic, phonetic) separated by "=" can each use different character sets
5. **UTF-8 (ISO_IR 192) is recommended** for new implementations to avoid complexity of ISO 2022 escape sequences

### Person Name with Multiple Character Sets

```
Alphabetic=Ideographic=Phonetic

Example (Japanese):
Yamada^Tarou=山田^太郎=やまだ^たろう
```

---

## DICOM Networking

### DICOM Network Architecture

DICOM networking uses a client-server model with:
- **Service Class User (SCU)**: Client that requests services
- **Service Class Provider (SCP)**: Server that provides services

### Network Components

1. **Application Entity (AE)**: Software component
2. **AE Title (AET)**: Unique identifier (max 16 characters)
3. **IP Address**: Network location
4. **Port**: TCP port (default: 104, commonly 11112)

### Association Establishment

The process of establishing a DICOM association:

```
SCU                                    SCP
 │                                      │
 ├──────── A-ASSOCIATE Request ────────>│
 │         (AE Title, Presentation     │
 │          Contexts, Parameters)      │
 │                                      │
 │<─────── A-ASSOCIATE Accept ─────────┤
 │         (Accepted Contexts)         │
 │                                      │
 ├──────── DIMSE Messages ─────────────>│
 │<─────── DIMSE Responses ────────────┤
 │                                      │
 ├──────── A-RELEASE Request ──────────>│
 │<─────── A-RELEASE Response ──────────┤
 │                                      │
```

### Presentation Context

A Presentation Context defines:
- **Abstract Syntax**: What (SOP Class UID)
- **Transfer Syntax**: How (encoding/compression)

Example:
```
Abstract Syntax: CT Image Storage (1.2.840.10008.5.1.4.1.1.2)
Transfer Syntax: Explicit VR Little Endian (1.2.840.10008.1.2.1)
```

---

## DICOM Services (DIMSE)

### DIMSE Overview

DIMSE (DICOM Message Service Element) defines the commands used in DICOM network communication.

### DIMSE-C (Composite)

Operations on composite objects (images):

1. **C-STORE**: Store an instance
   - SCU sends DICOM object to SCP
   - Used for sending images to PACS

2. **C-FIND**: Query for instances
   - Search for patients, studies, series, or images
   - Returns matching attributes

3. **C-GET**: Retrieve instances
   - Request SCP to send instances to SCU
   - SCP initiates C-STORE operations

4. **C-MOVE**: Move instances
   - Request SCP to send instances to a third-party destination
   - Destination specified by AE Title

5. **C-ECHO**: Verify connectivity
   - DICOM "ping"
   - Tests if association can be established

### DIMSE-N (Normalized)

Operations on normalized objects:

1. **N-EVENT-REPORT**: Report an event
2. **N-GET**: Get attribute values
3. **N-SET**: Set attribute values
4. **N-ACTION**: Perform an action
5. **N-CREATE**: Create an instance
6. **N-DELETE**: Delete an instance

### Query/Retrieve Levels

DICOM queries can be performed at different levels:

1. **Patient Level**: Find patients
2. **Study Level**: Find studies for a patient
3. **Series Level**: Find series in a study
4. **Image Level**: Find images in a series

### C-FIND Example Flow

```
SCU                                    SCP
 │                                      │
 ├──────── C-FIND Request ─────────────>│
 │         Query Keys:                 │
 │         PatientName = "Smith*"      │
 │         StudyDate = "20240101"      │
 │                                      │
 │<─────── C-FIND Response ────────────┤
 │         (Pending - First Match)     │
 │<─────── C-FIND Response ────────────┤
 │         (Pending - Second Match)    │
 │<─────── C-FIND Response ────────────┤
 │         (Success - No More Matches) │
 │                                      │
```

---

## Information Object Definitions (IODs)

### What are IODs?

Information Object Definitions specify the structure and content of DICOM objects. Each IOD defines which attributes are required, conditional, or optional.

### IOD Modules

IODs are composed of modules, each containing related attributes:

- **Patient Module**: Patient demographic information
- **Study Module**: Study-level information
- **Series Module**: Series-level information
- **Image Module**: Image-specific information
- **Equipment Module**: Equipment information

### Common IODs

1. **CR Image Storage**: Computed Radiography images
2. **CT Image Storage**: Computed Tomography images
3. **MR Image Storage**: Magnetic Resonance images
4. **US Image Storage**: Ultrasound images
5. **Secondary Capture Image Storage**: Screen captures, derived images
6. **RT Structure Set Storage**: Radiotherapy structure sets
7. **RT Plan Storage**: Radiotherapy treatment plans
8. **RT Dose Storage**: Radiotherapy dose information
9. **Grayscale Softcopy Presentation State**: Display settings
10. **Structured Report**: Structured medical reports

### Image Pixel Module

Critical attributes for image data:

- **Samples per Pixel** (0028,0002): 1 for grayscale, 3 for RGB
- **Photometric Interpretation** (0028,0004): MONOCHROME1, MONOCHROME2, RGB, etc.
- **Rows** (0028,0010): Image height in pixels
- **Columns** (0028,0011): Image width in pixels
- **Bits Allocated** (0028,0100): Bits per pixel (8, 16, etc.)
- **Bits Stored** (0028,0101): Actual bits used
- **High Bit** (0028,0102): Position of high bit
- **Pixel Representation** (0028,0103): 0=unsigned, 1=signed
- **Pixel Data** (7FE0,0010): The actual pixel values

---

## Service-Object Pair (SOP) Classes

### What are SOP Classes?

A SOP (Service-Object Pair) Class combines:
- **Service**: What operations can be performed (C-STORE, C-FIND, etc.)
- **Object**: What type of data (CT Image, MR Image, etc.)

### SOP Class UID Structure

SOP Class UIDs follow the pattern: `1.2.840.10008.5.1.4.1.1.X`

### Storage SOP Classes

| SOP Class UID | Name | Modality |
|---------------|------|----------|
| 1.2.840.10008.5.1.4.1.1.1 | CR Image Storage | CR |
| 1.2.840.10008.5.1.4.1.1.2 | CT Image Storage | CT |
| 1.2.840.10008.5.1.4.1.1.4 | MR Image Storage | MR |
| 1.2.840.10008.5.1.4.1.1.6.1 | US Image Storage | US |
| 1.2.840.10008.5.1.4.1.1.7 | Secondary Capture Image Storage | OT |
| 1.2.840.10008.5.1.4.1.1.20 | Nuclear Medicine Image Storage | NM |
| 1.2.840.10008.5.1.4.1.1.128 | PET Image Storage | PT |

### Query/Retrieve SOP Classes

- **Patient Root Q/R**: 1.2.840.10008.5.1.4.1.2.1.1
- **Study Root Q/R**: 1.2.840.10008.5.1.4.1.2.2.1
- **Patient/Study Only Q/R**: 1.2.840.10008.5.1.4.1.2.3.1

### Other Important SOP Classes

- **Verification SOP Class** (C-ECHO): 1.2.840.10008.1.1
- **Modality Worklist**: 1.2.840.10008.5.1.4.31
- **Storage Commitment**: 1.2.840.10008.1.20.1

---

## Conformance Statements

### What is a Conformance Statement?

A Conformance Statement is a formal document that describes the DICOM capabilities of a product. It is required by the DICOM standard (PS3.2) and specifies exactly what a device or software can do with DICOM.

### Purpose

Conformance Statements enable:
1. **Interoperability assessment**: Determine if two systems can communicate before deployment
2. **Integration planning**: Understand supported services, transfer syntaxes, and SOP classes
3. **Troubleshooting**: Reference when connection or data exchange problems occur
4. **Procurement**: Include in RFPs/RFIs to specify DICOM requirements

### Structure of a Conformance Statement

| Section | Content |
|---------|---------|
| **Introduction** | Product overview, intended use, references |
| **Networking** | Supported AE titles, ports, real-world activities |
| **Implementation Model** | Application entities, real-world activities mapped to DICOM services |
| **AE Specification** | For each AE: supported SOP classes, roles (SCU/SCP), transfer syntaxes |
| **Communication Profiles** | Network protocols (TCP/IP), security profiles (TLS) |
| **Extensions** | Private SOP classes, private transfer syntaxes, private tags |
| **Configuration** | Configurable parameters (timeouts, AE titles, ports) |
| **Media Interchange** | Supported media formats and application profiles |
| **Character Sets** | Supported character encodings |

### Key Elements to Check

When reviewing a Conformance Statement for integration:

1. **SOP Classes**: Does the device support the required storage/query SOP classes?
2. **Transfer Syntaxes**: Are compatible transfer syntaxes supported (especially compression)?
3. **Role**: Is the device SCU, SCP, or both for each service?
4. **Association Negotiation**: Maximum PDU size, number of simultaneous associations
5. **Character Set Support**: Does it handle international characters?
6. **Security**: Does it support TLS? What cipher suites?
7. **Extended Negotiation**: Are any extensions required?

### Real-World Activity Mapping

Conformance Statements map real-world clinical activities to DICOM services:

```
Real-World Activity          → DICOM Service
─────────────────────────────────────────────
Acquire Image                → C-STORE SCU (send to PACS)
Query PACS                   → C-FIND SCU
Retrieve Study               → C-MOVE SCU or C-GET SCU
Receive Images               → C-STORE SCP
Check Worklist               → MWL C-FIND SCU
Verify Connection            → C-ECHO SCU/SCP
Print Film                   → N-CREATE, N-SET, N-ACTION SCU
Report Completed Procedure   → MPPS N-CREATE, N-SET SCU
```

---

## DICOMweb

### Overview

DICOMweb (PS3.18) defines RESTful web services for accessing DICOM data over HTTP/HTTPS. It provides a modern, web-friendly alternative to traditional DICOM networking.

### DICOMweb Services

| Service | Full Name | HTTP Method | Purpose |
|---------|-----------|-------------|---------|
| **WADO-RS** | Web Access to DICOM Objects - RESTful | GET | Retrieve DICOM objects (studies, series, instances, metadata, bulk data, rendered images) |
| **STOW-RS** | Store Over the Web - RESTful | POST | Store DICOM objects via HTTP multipart upload |
| **QIDO-RS** | Query based on ID for DICOM Objects - RESTful | GET | Search for studies, series, and instances using query parameters |
| **UPS-RS** | Unified Procedure Step - RESTful | GET/POST/PUT/DELETE | Manage worklist items and procedure steps |
| **WADO-URI** | Web Access to DICOM Objects - URI | GET | Legacy single-instance retrieval using query string parameters |

### WADO-RS (Retrieve)

Retrieve DICOM objects using hierarchical URLs:

```
# Retrieve entire study
GET /studies/{StudyInstanceUID}

# Retrieve a series within a study
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}

# Retrieve a single instance
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}

# Retrieve metadata only (JSON or XML)
GET /studies/{StudyInstanceUID}/metadata
Accept: application/dicom+json

# Retrieve rendered image (JPEG/PNG)
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/rendered
Accept: image/jpeg

# Retrieve specific frames
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/frames/1,2,3

# Retrieve bulk data (pixel data)
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/bulkdata/{tag}
```

### STOW-RS (Store)

Store DICOM objects via HTTP POST:

```
# Store instances to a study
POST /studies/{StudyInstanceUID}
Content-Type: multipart/related; type="application/dicom"

--boundary
Content-Type: application/dicom

<DICOM file bytes>
--boundary--
```

The response indicates which instances were successfully stored and which failed.

### QIDO-RS (Query)

Search for DICOM objects using URL query parameters:

```
# Search for studies
GET /studies?PatientName=Smith*&StudyDate=20240101-20240131&ModalitiesInStudy=CT
    &limit=10&offset=0&includefield=00081030,00080061

# Search for series within a study
GET /studies/{StudyInstanceUID}/series?Modality=CT

# Search for instances within a series
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances

# Common query parameters:
#   PatientName, PatientID, StudyDate, Modality, StudyDescription
#   AccessionNumber, StudyInstanceUID, SeriesInstanceUID
#   limit, offset, includefield, fuzzymatching
```

### DICOMweb vs Traditional DICOM

| Aspect | Traditional DICOM | DICOMweb |
|--------|------------------|----------|
| **Protocol** | Custom binary over TCP | HTTP/HTTPS (REST) |
| **Data Format** | Binary DICOM | DICOM, JSON, XML, JPEG, PNG |
| **Firewall Friendly** | Requires open ports | Uses standard HTTP/S ports |
| **Authentication** | AE Title based | OAuth, JWT, API keys |
| **Infrastructure** | Dedicated DICOM network | Standard web infrastructure |
| **Client Complexity** | DICOM library required | Any HTTP client works |
| **Performance** | Efficient for bulk transfer | Higher overhead per request |
| **Streaming** | Built-in | Requires chunked encoding |

### DICOM JSON Model

DICOMweb uses a JSON representation of DICOM data:

```json
{
  "00100010": {
    "vr": "PN",
    "Value": [
      {
        "Alphabetic": "Smith^John"
      }
    ]
  },
  "00080060": {
    "vr": "CS",
    "Value": ["CT"]
  },
  "00080020": {
    "vr": "DA",
    "Value": ["20240115"]
  },
  "00280010": {
    "vr": "US",
    "Value": [512]
  }
}
```

---

## DICOM Structured Reporting (SR)

### Overview

DICOM Structured Reporting (SR) provides a standard way to encode clinical reports, measurements, and findings as DICOM objects. Unlike free-text reports, SR documents use a tree structure of coded entries that are machine-readable.

### SR Document Types (SOP Classes)

| SOP Class | Purpose |
|-----------|---------|
| Basic Text SR | Simple text-based reports |
| Enhanced SR | Rich reports with references to images and spatial coordinates |
| Comprehensive SR | Full-featured reports with all content types |
| Comprehensive 3D SR | 3D spatial coordinates and measurements |
| Key Object Selection Document | Mark significant images |
| Mammography CAD SR | Computer-aided detection results for mammography |
| Chest CAD SR | Computer-aided detection for chest imaging |
| X-Ray Radiation Dose SR | Radiation dose information (RDSR) |
| Acquisition Context SR | Describes acquisition conditions |
| Procedure Log | Procedure event logging |

### SR Document Tree Structure

An SR document is organized as a tree of **Content Items**:

```
SR Document Root (Container)
├── Concept Name: "Imaging Report"
├── Content Item: CONTAINER "Findings"
│   ├── Content Item: TEXT "Finding" → "Mass in right upper lobe"
│   ├── Content Item: CODE "Finding Site" → (T-28020, SRT, "Right Upper Lobe")
│   ├── Content Item: NUM "Size" → 2.5 cm
│   │   └── Content Item: CODE "Measurement Method" → (125203, DCM, "Bidimensional")
│   └── Content Item: IMAGE "Referenced Image"
│       └── Reference to: CT Image (SOP Instance UID)
│           └── Content Item: SCOORD "Region of Interest"
│               └── Graphic Data: CIRCLE (x,y,r)
├── Content Item: CONTAINER "Impression"
│   └── Content Item: TEXT → "Suspicious mass, recommend biopsy"
└── Content Item: CODE "Procedure Reported"
    └── (25045-6, LN, "CT Chest")
```

### Content Item Types

| Type | Code | Description |
|------|------|-------------|
| **TEXT** | TEXT | Free-text content |
| **CODE** | CODE | Coded value from a coding scheme |
| **NUM** | NUM | Numeric measurement with units |
| **DATE** | DATE | Date value |
| **TIME** | TIME | Time value |
| **DATETIME** | DATETIME | Combined date/time value |
| **PNAME** | PNAME | Person name |
| **UIDREF** | UIDREF | Reference to a UID |
| **IMAGE** | IMAGE | Reference to a DICOM image |
| **WAVEFORM** | WAVEFORM | Reference to a waveform |
| **SCOORD** | SCOORD | 2D spatial coordinates on an image |
| **SCOORD3D** | SCOORD3D | 3D spatial coordinates in a frame of reference |
| **TCOORD** | TCOORD | Temporal coordinates |
| **COMPOSITE** | COMPOSITE | Reference to a composite DICOM object |
| **CONTAINER** | CONTAINER | Groups child content items |

### Relationship Types

Content items are linked to their parent using relationship types:

| Relationship | Meaning |
|-------------|---------|
| **CONTAINS** | Parent contains child |
| **HAS OBS CONTEXT** | Observation context (observer, equipment) |
| **HAS ACQ CONTEXT** | Acquisition context information |
| **HAS CONCEPT MOD** | Concept modifier (qualifier) |
| **HAS PROPERTIES** | Properties of the parent concept |
| **INFERRED FROM** | Child evidence supports parent conclusion |
| **SELECTED FROM** | Spatial/temporal coordinate selected from referenced image |

### SR Templates

The DICOM standard defines templates (PS3.16) that specify the structure and content requirements for specific types of reports:

- **TID 1500**: Measurement Report — standardized measurements and qualitative evaluations
- **TID 1501**: Measurement Group — a group of related measurements
- **TID 2000**: Basic Diagnostic Imaging Report — basic structure for radiology reports
- **TID 10001**: Projection X-Ray Radiation Dose — dose report for projection radiography
- **TID 10011**: CT Radiation Dose — CT dose report (used in RDSR)
- **TID 1004**: Language of Content — specifies report language

### Radiation Dose Structured Reports (RDSR)

RDSR (Radiation Dose Structured Reports) are a key use of SR for recording patient radiation exposure:

```
RDSR Document
├── CT Acquisition
│   ├── CT Acquisition Type: "Spiral Acquisition"
│   ├── CTDIvol: 12.5 mGy
│   ├── DLP: 450.2 mGy·cm
│   ├── Scanning Length: 360 mm
│   └── CT Acquisition Parameters
│       ├── Tube Voltage: 120 kV
│       ├── Tube Current: 250 mA
│       └── Exposure Time: 0.5 s
└── CT Dose Length Product Total: 450.2 mGy·cm
```

---

## Modality Worklist and MPPS

### Modality Worklist (MWL)

Modality Worklist allows imaging equipment to query a worklist server (typically connected to the RIS/HIS) for scheduled procedures. This eliminates manual patient data entry at the modality.

### MWL Information Flow

```
    RIS / HIS                    Worklist SCP              Modality (SCU)
        │                            │                          │
        ├── Schedule Procedure ──>   │                          │
        │                            │                          │
        │                            │  <── C-FIND Request ────┤
        │                            │      (Query for today's │
        │                            │       scheduled exams)   │
        │                            │                          │
        │                            ├── C-FIND Response ──────>│
        │                            │   (Matching worklist     │
        │                            │    items returned)       │
        │                            │                          │
        │                            │  Technologist selects    │
        │                            │  patient from worklist   │
        │                            │  on modality console     │
        │                            │                          │
```

### MWL Key Attributes

| Tag | Name | Description |
|-----|------|-------------|
| (0008,0050) | Accession Number | Order number from RIS |
| (0010,0010) | Patient Name | Patient demographic |
| (0010,0020) | Patient ID | Patient identifier |
| (0020,000D) | Study Instance UID | Pre-assigned study UID |
| (0032,1060) | Requested Procedure Description | What procedure to perform |
| (0040,0001) | Scheduled Station AE Title | Which modality should perform |
| (0040,0002) | Scheduled Procedure Step Start Date | When to perform |
| (0040,0007) | Scheduled Procedure Step Description | Step details |
| (0040,0009) | Scheduled Procedure Step ID | Unique step identifier |
| (0040,0100) | Scheduled Procedure Step Sequence | Contains step details |

### Modality Performed Procedure Step (MPPS)

MPPS allows modalities to report back the status of procedures as they are performed. It provides a "closing the loop" mechanism for procedure tracking.

### MPPS Workflow

```
Modality                              MPPS SCP (RIS)
    │                                      │
    ├── N-CREATE (MPPS In Progress) ──────>│
    │   Status: "IN PROGRESS"              │
    │   Start Date/Time                    │
    │   Patient/Study Info                 │
    │                                      │
    │   ... Acquisition in progress ...    │
    │                                      │
    ├── N-SET (MPPS Completed) ───────────>│
    │   Status: "COMPLETED"               │
    │   End Date/Time                      │
    │   Performed Series                   │
    │   Number of Instances                │
    │   Dose Information                   │
    │                                      │
    │   OR                                 │
    │                                      │
    ├── N-SET (MPPS Discontinued) ────────>│
    │   Status: "DISCONTINUED"            │
    │   Reason for Discontinuation        │
    │                                      │
```

### MPPS Key Attributes

| Tag | Name | Description |
|-----|------|-------------|
| (0008,0060) | Modality | Type of equipment |
| (0040,0241) | Performed Station AE Title | Which modality performed |
| (0040,0242) | Performed Station Name | Station name |
| (0040,0244) | Performed Procedure Step Start Date | Actual start date |
| (0040,0250) | Performed Procedure Step End Date | Actual end date |
| (0040,0252) | Performed Procedure Step Status | IN PROGRESS, COMPLETED, DISCONTINUED |
| (0040,0253) | Performed Procedure Step ID | Unique identifier |
| (0040,0340) | Performed Series Sequence | Series created during procedure |

### Complete RIS-Modality-PACS Workflow

```
1. Order placed in RIS → Worklist item created
2. Modality queries MWL → Gets scheduled patient/procedure
3. Technologist selects worklist item → Patient data auto-populated
4. Modality sends MPPS "In Progress" → RIS knows exam started
5. Acquisition performed → Images created
6. Modality sends images via C-STORE → PACS stores images
7. Modality sends MPPS "Completed" → RIS knows exam finished
8. Radiologist reads study on PACS → Report created
```

---

## Storage Commitment

### Overview

Storage Commitment (N-ACTION/N-EVENT-REPORT) provides a mechanism for confirming that DICOM objects have been safely stored. It allows the sender to verify that the receiving system has taken responsibility for the data before the sender deletes its local copy.

### Why Storage Commitment Matters

- **Modalities have limited storage**: Need confirmation before deleting local images
- **Legal/regulatory requirements**: Ensure images are properly archived
- **Data integrity**: Verify that stored objects match what was sent
- **Workflow automation**: Trigger downstream processes after confirmed storage

### Storage Commitment Workflow

```
SCU (Modality)                          SCP (PACS/Archive)
    │                                        │
    ├── C-STORE images ────────────────────>│
    │   (Standard image transfer)           │
    │                                        │
    ├── N-ACTION (Storage Commit Request) ─>│
    │   Transaction UID                     │
    │   Referenced SOP Sequence:            │
    │   - SOP Class UID + SOP Instance UID  │
    │   - SOP Class UID + SOP Instance UID  │
    │   (List of instances to commit)       │
    │                                        │
    │   ... SCP verifies storage ...        │
    │                                        │
    │<── N-EVENT-REPORT (Commit Result) ────┤
    │   Success:                            │
    │   - Referenced SOP Sequence (stored)  │
    │   Failures:                           │
    │   - Failed SOP Sequence (not stored)  │
    │                                        │
    │   SCU can now safely delete local     │
    │   copies of successfully committed    │
    │   instances                           │
    │                                        │
```

### Storage Commitment Status

| Status | Meaning |
|--------|---------|
| **Success** | All requested instances are committed |
| **Partial Success** | Some instances committed, some failed |
| **Failure** | No instances committed |

### Referenced SOP Sequence

The request and response use sequences to list the instances:

```
Referenced SOP Sequence (0008,1199)
├── Item 1
│   ├── Referenced SOP Class UID: 1.2.840.10008.5.1.4.1.1.2 (CT)
│   └── Referenced SOP Instance UID: 1.2.3.4.5.6.7.8.9.1
├── Item 2
│   ├── Referenced SOP Class UID: 1.2.840.10008.5.1.4.1.1.2 (CT)
│   └── Referenced SOP Instance UID: 1.2.3.4.5.6.7.8.9.2
└── ...
```

---

## DICOM Printing

### Overview

DICOM Printing allows medical images to be printed on film printers. The Print Management service class defines how to create print jobs, compose film layouts, and manage print queues.

### Print Management SOP Classes

| SOP Class | Purpose |
|-----------|---------|
| Basic Film Session | Controls overall print session (number of copies, priority) |
| Basic Film Box | Represents a single sheet of film (format, orientation, layout) |
| Basic Grayscale Image Box | A single image position on the film (grayscale) |
| Basic Color Image Box | A single image position on the film (color) |
| Basic Annotation Box | Text annotation on the film |
| Print Job | Represents the print job in the queue |
| Printer | Represents the physical printer (status, configuration) |
| Presentation LUT | Lookup table for display transformation |

### Print Workflow

```
SCU (Workstation)                       SCP (Printer)
    │                                        │
    ├── N-CREATE Film Session ─────────────>│
    │   (Copies, Priority, Medium Type)     │
    │                                        │
    ├── N-CREATE Film Box ─────────────────>│
    │   (Layout: STANDARD\2,2 = 2x2 grid) │
    │   Response includes Image Box UIDs    │
    │                                        │
    ├── N-SET Image Box 1 ─────────────────>│
    │   (Pixel data, polarity, magnification)│
    │                                        │
    ├── N-SET Image Box 2 ─────────────────>│
    │   (Pixel data, polarity, magnification)│
    │                                        │
    ├── N-ACTION Print ────────────────────>│
    │   (Execute the print job)             │
    │                                        │
    ├── N-DELETE Film Box ─────────────────>│
    ├── N-DELETE Film Session ─────────────>│
    │                                        │
```

### Film Layout Formats

The Image Display Format tag specifies the layout:
- `STANDARD\1,1` — Single image per film
- `STANDARD\2,2` — 2×2 grid (4 images)
- `STANDARD\2,3` — 2×3 grid (6 images)
- `STANDARD\3,4` — 3×4 grid (12 images)
- `ROW\1,2` — 1 image in row 1, 2 images in row 2
- `COL\1,2` — 1 image in column 1, 2 images in column 2

---

## Presentation States

### Overview

Presentation States are DICOM objects that store display settings separately from image pixel data. They allow consistent image presentation without modifying the original images.

### Types of Presentation States

| SOP Class | Purpose |
|-----------|---------|
| Grayscale Softcopy Presentation State (GSPS) | Window/level, annotations, shutters, graphic overlays for grayscale images |
| Color Softcopy Presentation State | Similar to GSPS but for color images |
| Pseudo-Color Softcopy Presentation State | Color mapping for grayscale images (e.g., heat maps) |
| Blending Softcopy Presentation State | Blend multiple images (e.g., PET/CT fusion) |
| XA/XRF Grayscale Softcopy Presentation State | For X-Ray Angiography/Fluoroscopy |
| Advanced Blending Presentation State | Advanced multi-image blending |

### GSPS Content

A Grayscale Softcopy Presentation State can specify:

```
Presentation State
├── Referenced Series/Images (which images this applies to)
├── Display Area
│   ├── Displayed Area Selection (crop/zoom region)
│   └── Presentation Size Mode (SCALE TO FIT, TRUE SIZE, MAGNIFY)
├── Windowing
│   ├── Window Center/Width
│   └── VOI LUT Sequence (custom lookup tables)
├── Modality LUT
│   └── Rescale Slope/Intercept or LUT data
├── Softcopy VOI LUT
├── Graphic Annotations
│   ├── Graphic Objects (lines, circles, polygons, rulers)
│   │   ├── Graphic Type: POINT, POLYLINE, CIRCLE, ELLIPSE
│   │   ├── Graphic Data: coordinate pairs
│   │   └── Graphic Filled: YES/NO
│   └── Text Objects
│       ├── Unformatted Text Value
│       ├── Anchor Point
│       └── Bounding Box
├── Display Shutters
│   ├── Shutter Shape: RECTANGULAR, CIRCULAR, POLYGONAL
│   └── Shutter coordinates
├── Overlay Activation
│   └── Which overlays to display
├── Spatial Transformation
│   ├── Image Rotation (0, 90, 180, 270)
│   └── Image Horizontal Flip (YES/NO)
└── Presentation LUT
    └── Presentation LUT Shape (IDENTITY, INVERSE, LIN OD)
```

### Key Use Cases

1. **Radiologist annotations**: Mark findings with arrows, measurements, text labels
2. **Hanging protocols**: Define standard layouts and window settings per exam type
3. **Key images**: Apply annotations to highlight significant findings
4. **Image fusion**: Blend PET and CT images for oncology review
5. **Consistent display**: Ensure images appear the same across different workstations

---

## Key Object Selection Documents

### Overview

Key Object Selection (KOS) documents are a lightweight DICOM object used to flag significant images or instances within a study. They allow radiologists and other clinicians to mark important images without modifying the originals.

### KOS Structure

```
Key Object Selection Document
├── Patient/Study/Series Information
├── Content Date/Time (when selection was made)
├── Content Sequence (SR-like structure)
│   ├── Concept Name: "Key Object Selection"
│   ├── Content Item: CODE "Key Object Flag"
│   │   └── Value: (113001, DCM, "Rejected for Quality Reasons")
│   │         or (113010, DCM, "Best in Set")
│   │         or (113013, DCM, "For Referring Provider")
│   │         or (113018, DCM, "For Surgery")
│   └── Content Item: IMAGE references
│       ├── Referenced SOP Class UID
│       ├── Referenced SOP Instance UID
│       └── Referenced Frame Number (for multi-frame)
└── Current Requested Procedure Evidence Sequence
    └── Referenced Series Sequence → Referenced SOP Sequence
```

### Common KOS Document Titles

| Code | Meaning |
|------|---------|
| (113000, DCM) | Of Interest |
| (113001, DCM) | Rejected for Quality Reasons |
| (113010, DCM) | Best in Set |
| (113013, DCM) | For Referring Provider |
| (113018, DCM) | For Surgery |
| (113020, DCM) | For Printing |
| (113030, DCM) | Manifest |
| (113035, DCM) | Signed Manifest |
| (113038, DCM) | Correct (marks previously rejected instances as valid) |

### Use Cases

1. **Teaching files**: Mark interesting cases for education
2. **Quality control**: Flag images with artifacts for review or rejection
3. **Clinical communication**: Share significant findings with referring physicians
4. **Surgical planning**: Select key images needed for surgical consultation
5. **Prefetch/routing**: Use KOS to trigger intelligent prefetch of relevant priors

---

## Enhanced Multi-frame IODs

### Overview

Enhanced Multi-frame IODs are newer DICOM object types that store an entire series of images in a single DICOM file instead of separate files per slice. They were introduced to address limitations of the classic single-frame approach.

### Classic vs Enhanced IODs

| Aspect | Classic (e.g., CT Image Storage) | Enhanced (e.g., Enhanced CT Image Storage) |
|--------|------|------|
| **File per slice** | Yes — one file per image | No — entire series in one file |
| **Redundant data** | Patient/study info repeated in every file | Shared header, per-frame metadata |
| **Frame access** | Read entire file | Access individual frames by index |
| **SOP Class** | 1.2.840.10008.5.1.4.1.1.2 | 1.2.840.10008.5.1.4.1.1.2.1 |
| **Metadata model** | Flat attributes | Shared + Per-Frame Functional Groups |

### Functional Group Sequences

Enhanced IODs organize metadata into Functional Groups:

```
Enhanced CT Image
├── Shared Functional Groups Sequence (5200,9229)
│   └── Item (attributes common to ALL frames)
│       ├── CT Image Frame Type Sequence
│       ├── Pixel Measures Sequence (pixel spacing, slice thickness)
│       ├── Frame Content Sequence
│       └── CT Acquisition Type Sequence
│
├── Per-Frame Functional Groups Sequence (5200,9230)
│   ├── Item 1 (Frame 1)
│   │   ├── Frame Content Sequence (frame number, acquisition time)
│   │   ├── Plane Position Sequence (image position)
│   │   ├── Plane Orientation Sequence (image orientation)
│   │   └── Frame VOI LUT Sequence (window center/width)
│   ├── Item 2 (Frame 2)
│   │   └── ...
│   └── Item N (Frame N)
│       └── ...
│
└── Pixel Data (7FE0,0010) — all frames concatenated
```

### Common Enhanced IODs

| Enhanced SOP Class | Modality | SOP Class UID |
|-------------------|----------|---------------|
| Enhanced CT Image | CT | 1.2.840.10008.5.1.4.1.1.2.1 |
| Enhanced MR Image | MR | 1.2.840.10008.5.1.4.1.1.4.1 |
| Enhanced XA Image | XA | 1.2.840.10008.5.1.4.1.1.12.1.1 |
| Enhanced US Volume | US | 1.2.840.10008.5.1.4.1.1.6.2 |
| Enhanced PET Image | PT | 1.2.840.10008.5.1.4.1.1.130 |

### Benefits of Enhanced Multi-frame

1. **Reduced I/O overhead**: One file to open/close instead of hundreds
2. **Shared metadata**: Eliminates redundant patient/study info across slices
3. **Efficient network transfer**: Single C-STORE for entire acquisition
4. **Frame-level metadata**: Precise per-frame parameters (e.g., diffusion b-values in MR)
5. **Better suited for 4D data**: Time-varying datasets, cardiac cine, dynamic contrast

### Concatenation

For very large datasets that exceed practical single-file limits, the Concatenation mechanism splits an Enhanced Multi-frame object across multiple files while maintaining logical unity:

- **Concatenation UID** (0020,9161): Links files in the same concatenation
- **In-Concatenation Number** (0020,9162): Position within the concatenation
- **Concatenation Frame Offset Number** (0020,9228): Starting frame number

---

## Encapsulated Documents

### Overview

DICOM can encapsulate non-image documents such as PDFs, CDA (Clinical Document Architecture) reports, and other file types within DICOM objects. This allows all clinical documents to be managed alongside images in PACS.

### Encapsulated Document SOP Classes

| SOP Class | UID | Content Type |
|-----------|-----|-------------|
| Encapsulated PDF Storage | 1.2.840.10008.5.1.4.1.1.104.1 | PDF documents |
| Encapsulated CDA Storage | 1.2.840.10008.5.1.4.1.1.104.2 | HL7 CDA XML documents |
| Encapsulated STL Storage | 1.2.840.10008.5.1.4.1.1.104.3 | 3D printing models |
| Encapsulated OBJ Storage | 1.2.840.10008.5.1.4.1.1.104.4 | 3D OBJ models |
| Encapsulated MTL Storage | 1.2.840.10008.5.1.4.1.1.104.5 | 3D material definitions |

### Encapsulated PDF Structure

```
DICOM Encapsulated PDF
├── Patient Module (Patient Name, ID, etc.)
├── Study Module (Study Date, Description, etc.)
├── Series Module (Modality = "DOC" or "OT")
├── SOP Common Module (SOP Class/Instance UIDs)
├── Encapsulated Document Module
│   ├── MIME Type of Encapsulated Document (0042,0012) = "application/pdf"
│   ├── Document Title (0042,0010) = "Radiology Report"
│   ├── Concept Name Code Sequence (report type)
│   └── Encapsulated Document (0042,0011) = <PDF binary data>
└── Acquisition Context Module
```

### Use Cases

1. **Radiology reports**: Store finalized PDF reports alongside images in PACS
2. **Surgical planning**: Embed 3D-printed model files (STL) for surgical guides
3. **Regulatory documents**: Store consent forms and procedural documentation
4. **Cross-enterprise exchange**: Package clinical documents for IHE XDS workflows
5. **Research**: Attach analysis reports to imaging studies

---

## Waveform and Signal Data

### Overview

DICOM supports storage and exchange of waveform data such as ECG, EEG, hemodynamic signals, and audio recordings. This is defined in the Waveform IODs.

### Waveform SOP Classes

| SOP Class | Purpose |
|-----------|---------|
| 12-Lead ECG | Standard 12-lead electrocardiogram |
| General ECG | Other ECG configurations |
| Ambulatory ECG | Holter monitor and ambulatory ECG |
| Hemodynamic Waveform | Pressure and flow measurements |
| Basic Cardiac Electrophysiology | Intracardiac signals |
| Arterial Pulse Waveform | Arterial pulse measurements |
| Respiratory Waveform | Respiratory signals |
| General Audio Waveform | Audio recordings |
| Real-Time Audio Waveform | Streaming audio during procedures |

### Waveform Data Structure

```
Waveform DICOM Object
├── Patient/Study/Series/SOP Modules
├── Waveform Identification Module
│   └── Content Date/Time, Acquisition DateTime
├── Waveform Module
│   ├── Number of Waveform Channels (003A,0005)
│   ├── Number of Waveform Samples (003A,0010)
│   ├── Sampling Frequency (003A,001A) — samples per second
│   ├── Waveform Data (5400,1010) — encoded signal values
│   └── Channel Definition Sequence (003A,0200)
│       ├── Channel Source Sequence → (Lead I, Lead II, etc.)
│       ├── Channel Sensitivity (003A,0210)
│       ├── Channel Sensitivity Units (003A,0211)
│       ├── Channel Baseline (003A,0213)
│       └── Waveform Bits Allocated (003A,021A)
└── Waveform Annotation Module
    └── Annotations (measurements, intervals, etc.)
```

### ECG-Specific Concepts

- **Leads**: Standard lead designations (I, II, III, aVR, aVL, aVF, V1-V6)
- **Sampling Rate**: Typically 500 Hz for diagnostic ECG
- **Amplitude Resolution**: Usually 10 μV per bit
- **Annotations**: P-wave, QRS complex, T-wave intervals and amplitudes

---

## Overlay Data

### Overview

Overlays are binary bitmap graphics stored within DICOM images that can be displayed on top of the pixel data. They are used for annotations, region of interest markers, graphics, and text burned into the image.

### Overlay Attributes

Overlays use group numbers 60xx (where xx ranges from 00 to 1E in steps of 2):

| Tag | Name | Description |
|-----|------|-------------|
| (60xx,0010) | Overlay Rows | Height of overlay bitmap |
| (60xx,0011) | Overlay Columns | Width of overlay bitmap |
| (60xx,0040) | Overlay Type | "G" (Graphics) or "R" (Region of Interest) |
| (60xx,0050) | Overlay Origin | Position of overlay relative to image |
| (60xx,0100) | Overlay Bits Allocated | Always 1 (binary bitmap) |
| (60xx,3000) | Overlay Data | The bitmap data |
| (60xx,0015) | Number of Frames in Overlay | For multi-frame overlays |
| (60xx,0022) | Overlay Description | Text description of the overlay |
| (60xx,0045) | Overlay Subtype | Further classification of overlay purpose |
| (60xx,1500) | Overlay Label | Short label for the overlay |

### Overlay Types

- **Graphics (G)**: Annotations, arrows, measurement lines, text graphics burned into the image by the modality
- **ROI (R)**: Region of Interest markers highlighting specific anatomical areas or lesions

### Historical Note

In older DICOM versions, overlay data was sometimes embedded in unused high bits of the pixel data (e.g., bits 12-15 of 16-bit pixels). Modern DICOM stores overlays in separate (60xx,3000) data elements. Implementations must handle both legacy embedded overlays and modern separate overlays.

---

## Radiotherapy (RT) Objects

### Overview

DICOM defines specialized IODs for radiotherapy (RT), covering the entire treatment planning and delivery workflow. These objects allow interoperability between treatment planning systems, linear accelerators, and record-and-verify systems.

### RT IODs

| SOP Class | Purpose |
|-----------|---------|
| RT Structure Set | Contoured structures (organs, tumors) on images |
| RT Plan | Treatment plan parameters (beams, angles, MLC) |
| RT Dose | Calculated or measured dose distributions |
| RT Image | Portal images and DRRs (Digitally Reconstructed Radiographs) |
| RT Beams Treatment Record | Delivered treatment record |
| RT Brachy Treatment Record | Brachytherapy delivery record |
| RT Ion Plan | Ion beam (proton/carbon) treatment plans |
| RT Ion Beams Treatment Record | Ion beam delivery record |

### RT Structure Set

Contains contoured regions of interest drawn on CT/MR images:

```
RT Structure Set
├── Structure Set Module
│   ├── Structure Set Label
│   └── Structure Set Date/Time
├── ROI Contour Module
│   └── ROI Contour Sequence
│       ├── ROI 1: "GTV" (Gross Tumor Volume)
│       │   ├── ROI Display Color: [255, 0, 0] (red)
│       │   └── Contour Sequence
│       │       ├── Contour 1 (Slice z=100mm)
│       │       │   ├── Contour Geometric Type: CLOSED_PLANAR
│       │       │   ├── Number of Contour Points: 45
│       │       │   └── Contour Data: [x1,y1,z1, x2,y2,z2, ...]
│       │       └── Contour 2 (Slice z=103mm)
│       │           └── ...
│       ├── ROI 2: "PTV" (Planning Target Volume)
│       │   └── ...
│       └── ROI 3: "Spinal Cord" (organ at risk)
│           └── ...
├── RT ROI Observations Module
│   └── RT ROI Interpreted Type: "GTV", "PTV", "ORGAN", "AVOIDANCE"
└── Referenced Frame of Reference
    └── Links to CT image series used for contouring
```

### RT Plan

Contains all treatment plan parameters:

```
RT Plan
├── RT General Plan Module
│   ├── RT Plan Label, Date, Description
│   └── RT Plan Geometry: "PATIENT"
├── Dose Reference Module
│   └── Target Prescription Dose: 6000 cGy
├── Fraction Group Module
│   ├── Number of Fractions Planned: 30
│   └── Referenced Beam Sequence
├── Beam Module
│   └── Beam Sequence
│       ├── Beam 1: "RAO"
│       │   ├── Beam Type: STATIC or DYNAMIC
│       │   ├── Radiation Type: PHOTON
│       │   ├── Beam Energy: 6 MV
│       │   ├── Gantry Angle: 45°
│       │   ├── Collimator Angle: 0°
│       │   ├── Couch Angle: 0°
│       │   └── Control Point Sequence
│       │       ├── MLC positions
│       │       ├── Jaw positions
│       │       └── Cumulative Meterset Weight
│       └── Beam 2: "LAO"
│           └── ...
└── Patient Setup Module
    └── Patient Position: HFS (Head First Supine)
```

### RT Dose

Contains calculated dose distributions:

```
RT Dose
├── Dose Grid Scaling (3004,000E): Factor to convert stored values to dose in Gy
├── Dose Units (3004,0002): "GY" or "RELATIVE"
├── Dose Type (3004,0004): "PHYSICAL", "EFFECTIVE", "ERROR"
├── Dose Summation Type (3004,000A): "PLAN", "BEAM", "FRACTION"
├── Image Pixel Module (rows, columns, bits)
├── Pixel Data → 3D dose grid values
└── DVH Module (optional)
    └── DVH Sequence → dose-volume histogram data
```

---

## IHE Integration Profiles

### What is IHE?

IHE (Integrating the Healthcare Enterprise) is an initiative that defines **Integration Profiles** — standardized workflows using DICOM, HL7, and other standards. While IHE is not part of DICOM itself, it specifies how DICOM services should be used together in real clinical workflows.

### Key IHE Radiology Profiles

| Profile | Description | DICOM Services Used |
|---------|-------------|---------------------|
| **SWF** (Scheduled Workflow) | End-to-end radiology workflow from order to report | MWL, MPPS, C-STORE, Storage Commitment |
| **PIR** (Patient Information Reconciliation) | Correct patient demographics after acquisition | Update patient info, resend corrected objects |
| **CPI** (Consistent Presentation of Images) | Ensure images display consistently across viewers | GSPS, Hanging Protocols |
| **KIN** (Key Image Note) | Mark significant images with annotations | Key Object Selection Documents |
| **RID** (Retrieve Information for Display) | Web-based access to reports and images | HTTP/HTTPS, WADO |
| **PDI** (Portable Data for Imaging) | Export imaging data to portable media (CD/DVD) | DICOMDIR, media profiles |
| **XDS-I** (Cross-enterprise Document Sharing for Imaging) | Share imaging across enterprises | WADO, KOS manifests, XDS registry |
| **IOCM** (Imaging Object Change Management) | Reject, correct, or delete imaging objects | KOS rejection notes, C-STORE |

### SWF (Scheduled Workflow) — The Foundation Profile

SWF is the most fundamental IHE profile. It orchestrates the standard radiology workflow:

```
Step 1: Order Entry (HL7 ORM)
  └── RIS creates order, generates worklist

Step 2: Patient Arrives
  └── ADT registration (HL7 ADT)

Step 3: Modality Queries Worklist
  └── MWL C-FIND (modality → worklist server)

Step 4: Acquisition Begins
  └── MPPS N-CREATE "In Progress" (modality → RIS)

Step 5: Images Acquired
  └── C-STORE images (modality → PACS)

Step 6: Acquisition Ends
  └── MPPS N-SET "Completed" (modality → RIS)

Step 7: Storage Commitment
  └── N-ACTION request (modality → PACS)
  └── N-EVENT-REPORT response (PACS → modality)

Step 8: Images Available
  └── Study appears on radiologist worklist

Step 9: Report Created
  └── Dictation/structured report

Step 10: Report Distributed
  └── Available in RIS/EHR
```

### IHE Connectathon

IHE runs annual **Connectathon** testing events where vendors test their implementations against each other using the defined integration profiles. Successful testing results are published and can be referenced during product evaluation.

---

## DICOMDIR and Media Storage

### What is DICOMDIR?

DICOMDIR is a special DICOM file that serves as a directory/index of all DICOM files stored on a media (CD, DVD, USB drive). It provides a hierarchical listing that enables navigation without scanning every file.

### DICOMDIR Structure

```
DICOMDIR File
├── File Meta Information
│   └── Media Storage SOP Class UID: 1.2.840.10008.1.3.10 (Media Storage Directory)
├── File-set ID
└── Directory Record Sequence (0004,1220)
    ├── PATIENT Record
    │   ├── Patient Name, ID, Birth Date
    │   └── Offset to first child STUDY record
    │       ├── STUDY Record
    │       │   ├── Study Date, Description, Instance UID
    │       │   └── Offset to first child SERIES record
    │       │       ├── SERIES Record
    │       │       │   ├── Modality, Series Instance UID
    │       │       │   └── Offset to first child IMAGE record
    │       │       │       ├── IMAGE Record
    │       │       │       │   ├── SOP Instance UID
    │       │       │       │   └── Referenced File ID: "DICOM\IMG001"
    │       │       │       └── IMAGE Record
    │       │       │           └── Referenced File ID: "DICOM\IMG002"
    │       │       └── SERIES Record
    │       │           └── ...
    │       └── STUDY Record
    │           └── ...
    └── PATIENT Record
        └── ...
```

### Media Application Profiles

DICOM defines application profiles for specific media use cases:

| Profile | Media | Content |
|---------|-------|---------|
| STD-GEN-CD | CD-R | General purpose DICOM on CD |
| STD-GEN-DVD | DVD | General purpose DICOM on DVD |
| STD-GEN-USB | USB | General purpose DICOM on USB |
| STD-GEN-EMAIL | Email | DICOM objects via email |

### File Naming on Media

DICOM media files follow the **ISO 9660** file naming convention:
- 8-character filename, no extension (or .dcm)
- Path format: `DICOM/IMAGE001` or `DICOM/000001/000001`
- Case insensitive
- Subdirectories allowed up to 8 levels deep

---

## Working with DICOM Files

### Reading DICOM Files

Basic workflow:
1. Open file and verify DICOM prefix
2. Read File Meta Information
3. Determine Transfer Syntax
4. Parse Data Set elements
5. Extract pixel data if present

### Writing DICOM Files

Basic workflow:
1. Create File Meta Information
2. Set Transfer Syntax
3. Add required data elements
4. Add image pixel data
5. Write preamble, prefix, meta info, and dataset
6. Calculate and store file size

### Anonymization

Common approach to anonymize DICOM files:

**Tags to Remove/Modify**:
- Patient Name (0010,0010)
- Patient ID (0010,0020)
- Patient Birth Date (0010,0030)
- Patient Address (0010,1040)
- Institution Name (0008,0080)
- Referring Physician Name (0008,0090)
- Operator Name (0008,1070)

**Tags to Keep**:
- Study/Series/Instance UIDs (generate new ones)
- Image-related attributes (Rows, Columns, Pixel Data)
- Study Date (may need to shift consistently)

### Pixel Data Extraction

Steps to extract and display pixel data:

1. Read pixel data tag (7FE0,0010)
2. Check Photometric Interpretation
3. Apply Window/Level settings if present
4. Handle byte ordering and bit depth
5. Convert to display format (RGB, grayscale)

### Window/Level (Brightness/Contrast)

Formula for applying Window/Level:

```
if pixel_value <= (level - width/2):
    display_value = 0
elif pixel_value >= (level + width/2):
    display_value = max_value
else:
    display_value = ((pixel_value - level) / width + 0.5) * max_value
```

Where:
- **Window Center (Level)** (0028,1050): Middle of intensity range
- **Window Width** (0028,1051): Range of intensities

---

## DICOM Libraries and Tools

### Python Libraries

#### pydicom
```python
# Most popular Python DICOM library
pip install pydicom

# Features:
- Read/write DICOM files
- Modify DICOM attributes
- Pixel data access
- No network support (use pynetdicom)
```

#### pynetdicom
```python
# Python DICOM networking
pip install pynetdicom

# Features:
- Implement DICOM SCU/SCP
- C-ECHO, C-STORE, C-FIND, C-MOVE, C-GET
- Association management
- Works with pydicom for data handling
```

#### highdicom
```python
# High-level DICOM abstraction
pip install highdicom

# Features:
- Simplified DICOM creation
- Support for advanced IODs
- Structured reports
- Segmentation objects
```

### JavaScript/TypeScript Libraries

#### dcmjs
```javascript
// JavaScript DICOM library
npm install dcmjs

// Features:
- Browser and Node.js support
- Read/write DICOM files
- JSON representation
- Works with Cornerstone viewers
```

#### dicom-parser
```javascript
// Lightweight DICOM parser
npm install dicom-parser

// Features:
- Fast parsing
- Low-level access
- Browser compatible
- Used by Cornerstone
```

### Java Libraries

#### dcm4che
```java
// Comprehensive Java DICOM toolkit
// Maven: org.dcm4che:dcm4che-core:5.31.1
// Modules: dcm4che-net, dcm4che-imageio, dcm4che-json, dcm4che-hl7

// Features:
// - Complete DICOM standard implementation
// - Full DIMSE service support (C-ECHO, C-STORE, C-FIND, C-MOVE, C-GET)
// - Transfer syntax negotiation and transcoding
// - HL7 v2 integration and DICOMweb support
// - Foundation of the dcm4chee PACS server
```

#### PixelMed
```java
// Java DICOM toolkit for education and research
// http://www.pixelmed.com/

// Features:
// - DICOM file I/O and network protocols
// - Validation, Structured Reports, anonymization
// - Education-focused with extensive Javadoc
```

### C++ Libraries

#### DCMTK (DICOM Toolkit)
```cpp
// Reference C++ implementation
https://dicom.offis.de/dcmtk

// Features:
- Complete DICOM standard
- Command-line tools
- Network support
- Industry standard
```

#### GDCM (Grassroots DICOM)
```cpp
// Modern C++ DICOM library
https://gdcm.sourceforge.net/

// Features:
- Modern C++
- Python bindings
- Image codecs
- Performance optimized
```

### .NET Libraries

#### fo-dicom (Fellow Oak DICOM)
```csharp
// .NET DICOM library
NuGet: fo-dicom

// Features:
- Cross-platform (.NET Core)
- Network services
- Image transcoding
- Modern async/await
```

#### ClearCanvas
```csharp
// Enterprise DICOM framework
https://www.clearcanvas.ca/

// Features:
- Full-featured PACS
- Viewer framework
- Enterprise ready
- Commercial support
```

### Command-Line Tools

#### dcmtk Tools
```bash
# Dump DICOM file contents
dcmdump file.dcm

# Send DICOM file (C-STORE)
storescu -aec PACS_AET -aet MY_AET localhost 11112 file.dcm

# Query PACS (C-FIND)
findscu -aec PACS_AET -aet MY_AET -k PatientName="Smith*" localhost 11112

# Test connectivity (C-ECHO)
echoscu -aec PACS_AET -aet MY_AET localhost 11112

# Convert transfer syntax
dcmconv -te +te inputfile.dcm outputfile.dcm
```

#### GDCM Tools
```bash
# Dump DICOM information
gdcmdump file.dcm

# Anonymize DICOM file
gdcmanon -i input.dcm -o output.dcm

# Convert to different transfer syntax
gdcmconv -i input.dcm -o output.dcm -w
```

---

## Code Examples

### Python: Reading DICOM Files

```python
import pydicom
import numpy as np
import matplotlib.pyplot as plt

# Read DICOM file
ds = pydicom.dcmread('image.dcm')

# Access attributes
print(f"Patient Name: {ds.PatientName}")
print(f"Study Date: {ds.StudyDate}")
print(f"Modality: {ds.Modality}")
print(f"Image Size: {ds.Rows} x {ds.Columns}")

# Access pixel data
pixel_array = ds.pixel_array

# Apply window/level if present
if hasattr(ds, 'WindowCenter') and hasattr(ds, 'WindowWidth'):
    center = ds.WindowCenter
    width = ds.WindowWidth
    
    # Handle multiple window settings
    if isinstance(center, pydicom.multival.MultiValue):
        center = center[0]
        width = width[0]
    
    # Apply windowing
    img_min = center - width / 2
    img_max = center + width / 2
    pixel_array = np.clip(pixel_array, img_min, img_max)
    pixel_array = ((pixel_array - img_min) / (img_max - img_min) * 255).astype(np.uint8)

# Display image
plt.imshow(pixel_array, cmap='gray')
plt.title(f"{ds.PatientName} - {ds.Modality}")
plt.axis('off')
plt.show()
```

### Python: Creating DICOM Files

```python
import pydicom
from pydicom.dataset import Dataset, FileDataset
from datetime import datetime
import numpy as np

# Create file meta information
file_meta = Dataset()
file_meta.MediaStorageSOPClassUID = '1.2.840.10008.5.1.4.1.1.2'  # CT Image Storage
file_meta.MediaStorageSOPInstanceUID = pydicom.uid.generate_uid()
file_meta.TransferSyntaxUID = '1.2.840.10008.1.2.1'  # Explicit VR Little Endian
file_meta.ImplementationClassUID = pydicom.uid.generate_uid()

# Create dataset
ds = FileDataset('output.dcm', {}, file_meta=file_meta, preamble=b"\0" * 128)

# Add patient information
ds.PatientName = "Doe^John"
ds.PatientID = "12345"
ds.PatientBirthDate = '19800101'
ds.PatientSex = 'M'

# Add study information
ds.StudyInstanceUID = pydicom.uid.generate_uid()
ds.StudyDate = datetime.now().strftime('%Y%m%d')
ds.StudyTime = datetime.now().strftime('%H%M%S')
ds.StudyDescription = 'Test Study'
ds.StudyID = '1'

# Add series information
ds.SeriesInstanceUID = pydicom.uid.generate_uid()
ds.SeriesNumber = 1
ds.Modality = 'CT'

# Add image information
ds.SOPClassUID = file_meta.MediaStorageSOPClassUID
ds.SOPInstanceUID = file_meta.MediaStorageSOPInstanceUID
ds.InstanceNumber = 1

# Create sample pixel data (512x512 image)
pixel_data = np.random.randint(0, 4096, size=(512, 512), dtype=np.uint16)

# Set image attributes
ds.SamplesPerPixel = 1
ds.PhotometricInterpretation = 'MONOCHROME2'
ds.Rows = 512
ds.Columns = 512
ds.BitsAllocated = 16
ds.BitsStored = 12
ds.HighBit = 11
ds.PixelRepresentation = 0
ds.PixelData = pixel_data.tobytes()

# Save file
ds.save_as('output.dcm', write_like_original=False)
print("DICOM file created successfully!")
```

### Python: DICOM Network SCU (C-STORE)

```python
from pynetdicom import AE, StoragePresentationContexts
from pynetdicom.sop_class import CTImageStorage
import pydicom

# Create Application Entity
ae = AE(ae_title='MY_SCU')

# Add presentation contexts
ae.add_requested_context(CTImageStorage)
# Or add all storage contexts:
# ae.requested_contexts = StoragePresentationContexts

# Read DICOM file
ds = pydicom.dcmread('image.dcm')

# Associate with peer
assoc = ae.associate('192.168.1.100', 11112, ae_title='PACS_SCP')

if assoc.is_established:
    # Send C-STORE request
    status = assoc.send_c_store(ds)
    
    # Check the status
    if status and status.Status == 0x0000:
        print('C-STORE successful')
    else:
        print('C-STORE failed')
    
    # Release association
    assoc.release()
else:
    print('Association rejected, aborted or never connected')
```

### Python: DICOM Network SCP (Storage Server)

```python
from pynetdicom import AE, StoragePresentationContexts, evt
from pynetdicom.sop_class import VerificationSOPClass
import os

def handle_store(event):
    """Handle C-STORE request"""
    ds = event.dataset
    ds.file_meta = event.file_meta
    
    # Save to disk
    filename = f"{ds.SOPInstanceUID}.dcm"
    ds.save_as(filename, write_like_original=False)
    
    print(f"Stored: {filename}")
    
    # Return success status
    return 0x0000

# Create Application Entity
ae = AE(ae_title='MY_SCP')

# Add supported presentation contexts
ae.supported_contexts = StoragePresentationContexts
ae.add_supported_context(VerificationSOPClass)

# Set event handlers
handlers = [(evt.EVT_C_STORE, handle_store)]

# Start SCP
ae.start_server(('', 11112), block=True, evt_handlers=handlers)
```

### Python: DICOM Query (C-FIND)

```python
from pynetdicom import AE
from pynetdicom.sop_class import PatientRootQueryRetrieveInformationModelFind
from pydicom.dataset import Dataset

# Create Application Entity
ae = AE(ae_title='MY_FIND_SCU')
ae.add_requested_context(PatientRootQueryRetrieveInformationModelFind)

# Create query dataset
ds = Dataset()
ds.PatientName = 'Smith*'
ds.PatientID = ''
ds.StudyDate = '20240101-20240131'
ds.StudyDescription = ''
ds.QueryRetrieveLevel = 'STUDY'

# Required return keys
ds.StudyInstanceUID = ''
ds.AccessionNumber = ''

# Associate and send C-FIND
assoc = ae.associate('192.168.1.100', 11112, ae_title='PACS_SCP')

if assoc.is_established:
    responses = assoc.send_c_find(ds, PatientRootQueryRetrieveInformationModelFind)
    
    for (status, identifier) in responses:
        if status and status.Status in (0xFF00, 0xFF01):
            # Pending response
            print(f"\nPatient: {identifier.PatientName}")
            print(f"Study UID: {identifier.StudyInstanceUID}")
            print(f"Study Date: {identifier.StudyDate}")
            print(f"Description: {identifier.StudyDescription}")
    
    assoc.release()
```

### JavaScript: Reading DICOM with dcmjs

```javascript
const dcmjs = require('dcmjs');
const fs = require('fs');

// Read DICOM file
const dicomData = fs.readFileSync('image.dcm');
const arrayBuffer = dicomData.buffer.slice(
    dicomData.byteOffset,
    dicomData.byteOffset + dicomData.byteLength
);

// Parse DICOM
const dataset = dcmjs.data.DicomMessage.readFile(arrayBuffer);
const naturalDataset = dcmjs.data.DicomMetaDictionary.naturalizeDataset(dataset.dict);

// Access attributes
console.log('Patient Name:', naturalDataset.PatientName);
console.log('Study Date:', naturalDataset.StudyDate);
console.log('Modality:', naturalDataset.Modality);
console.log('Image Size:', naturalDataset.Rows, 'x', naturalDataset.Columns);

// Access pixel data
const pixelData = naturalDataset.PixelData;
console.log('Pixel data length:', pixelData.length);
```

### C++: Reading DICOM with DCMTK

```cpp
#include <dcmtk/dcmdata/dctk.h>
#include <iostream>

int main(int argc, char *argv[]) {
    if (argc < 2) {
        std::cerr << "Usage: " << argv[0] << " <dicom-file>" << std::endl;
        return 1;
    }

    // Load DICOM file
    DcmFileFormat fileFormat;
    OFCondition status = fileFormat.loadFile(argv[1]);
    
    if (status.bad()) {
        std::cerr << "Error: cannot read file: " << status.text() << std::endl;
        return 1;
    }

    // Get dataset
    DcmDataset *dataset = fileFormat.getDataset();

    // Read string values
    OFString patientName, studyDate, modality;
    dataset->findAndGetOFString(DCM_PatientName, patientName);
    dataset->findAndGetOFString(DCM_StudyDate, studyDate);
    dataset->findAndGetOFString(DCM_Modality, modality);

    // Read numeric values
    Uint16 rows, columns;
    dataset->findAndGetUint16(DCM_Rows, rows);
    dataset->findAndGetUint16(DCM_Columns, columns);

    // Print information
    std::cout << "Patient Name: " << patientName << std::endl;
    std::cout << "Study Date: " << studyDate << std::endl;
    std::cout << "Modality: " << modality << std::endl;
    std::cout << "Image Size: " << rows << " x " << columns << std::endl;

    return 0;
}
```

### C#: Reading DICOM with fo-dicom

```csharp
using FellowOakDicom;
using FellowOakDicom.Imaging;
using System;

class Program
{
    static void Main(string[] args)
    {
        if (args.Length < 1)
        {
            Console.WriteLine("Usage: program <dicom-file>");
            return;
        }

        // Load DICOM file
        var dicomFile = DicomFile.Open(args[0]);
        var dataset = dicomFile.Dataset;

        // Read attributes
        var patientName = dataset.GetSingleValue<string>(DicomTag.PatientName);
        var studyDate = dataset.GetSingleValue<string>(DicomTag.StudyDate);
        var modality = dataset.GetSingleValue<string>(DicomTag.Modality);
        var rows = dataset.GetSingleValue<ushort>(DicomTag.Rows);
        var columns = dataset.GetSingleValue<ushort>(DicomTag.Columns);

        // Print information
        Console.WriteLine($"Patient Name: {patientName}");
        Console.WriteLine($"Study Date: {studyDate}");
        Console.WriteLine($"Modality: {modality}");
        Console.WriteLine($"Image Size: {rows} x {columns}");

        // Get pixel data
        var image = new DicomImage(dataset);
        var bitmap = image.RenderImage().AsClonedBitmap();
        
        Console.WriteLine($"Pixel data dimensions: {bitmap.Width} x {bitmap.Height}");
    }
}
```

### Java: Reading DICOM Files with dcm4che

```java
import org.dcm4che3.data.Attributes;
import org.dcm4che3.data.Tag;
import org.dcm4che3.data.VR;
import org.dcm4che3.io.DicomInputStream;

import java.io.File;
import java.io.IOException;

public class ReadDicom {
    public static void main(String[] args) {
        if (args.length < 1) {
            System.err.println("Usage: ReadDicom <dicom-file>");
            return;
        }

        File dicomFile = new File(args[0]);

        try (DicomInputStream dis = new DicomInputStream(dicomFile)) {
            // Read all attributes including pixel data
            Attributes attrs = dis.readDataset();
            Attributes fmi = dis.readFileMetaInformation();

            // Access common attributes
            String patientName = attrs.getString(Tag.PatientName);
            String studyDate = attrs.getString(Tag.StudyDate);
            String modality = attrs.getString(Tag.Modality);
            int rows = attrs.getInt(Tag.Rows, 0);
            int columns = attrs.getInt(Tag.Columns, 0);

            // Print information
            System.out.println("Patient Name: " + patientName);
            System.out.println("Study Date: " + studyDate);
            System.out.println("Modality: " + modality);
            System.out.println("Image Size: " + rows + " x " + columns);

            // Access transfer syntax from file meta information
            if (fmi != null) {
                String transferSyntax = fmi.getString(Tag.TransferSyntaxUID);
                System.out.println("Transfer Syntax: " + transferSyntax);
            }

            // Access pixel data attributes
            int bitsAllocated = attrs.getInt(Tag.BitsAllocated, 0);
            int bitsStored = attrs.getInt(Tag.BitsStored, 0);
            String photometric = attrs.getString(Tag.PhotometricInterpretation);
            System.out.println("Bits Allocated: " + bitsAllocated);
            System.out.println("Bits Stored: " + bitsStored);
            System.out.println("Photometric Interpretation: " + photometric);

            // Access all tags in the dataset
            System.out.println("\n--- All Tags ---");
            attrs.accept((attrs1, tag, vr, value) -> {
                System.out.printf("(%04X,%04X) %s: %s%n",
                    tag >>> 16, tag & 0xFFFF, vr, attrs1.getString(tag));
                return true;
            }, false);

        } catch (IOException e) {
            System.err.println("Error reading DICOM file: " + e.getMessage());
        }
    }
}
```

---

## Best Practices

### File Handling

1. **Always validate DICOM prefix**: Check for "DICM" at byte 128
2. **Handle transfer syntaxes properly**: Support common syntaxes at minimum
3. **Preserve UIDs**: Don't modify existing UIDs unless anonymizing
4. **Use appropriate VRs**: Follow DICOM standard for each tag
5. **Handle large files carefully**: Stream data for large datasets

### Networking

1. **Configure timeouts**: Set appropriate connection and operation timeouts
2. **Handle association rejections**: Implement retry logic with backoff
3. **Support multiple transfer syntaxes**: Negotiate the best available
4. **Use appropriate AE Titles**: Follow naming conventions (max 16 chars)
5. **Implement C-ECHO**: Always support verification for testing

### Data Quality

1. **Validate required tags**: Check for mandatory elements
2. **Use proper date/time formats**: Follow DICOM standards (YYYYMMDD, HHMMSS)
3. **Handle character sets**: Support international characters
4. **Verify UIDs**: Ensure UIDs are properly formatted
5. **Check pixel data integrity**: Validate dimensions match declared values

### Performance

1. **Use streaming for large files**: Don't load entire files into memory
2. **Cache parsed data**: Avoid re-parsing the same files
3. **Parallel processing**: Process multiple files concurrently when possible
4. **Optimize pixel data handling**: Use efficient data structures
5. **Connection pooling**: Reuse network connections when appropriate

### Security

1. **Sanitize file paths**: Prevent directory traversal attacks
2. **Validate input data**: Check file sizes and formats
3. **Use TLS for networking**: Implement DICOM TLS when possible
4. **Implement access controls**: Verify permissions before operations
5. **Audit operations**: Log all file and network operations

### Anonymization

1. **Remove all identifying information**: Use comprehensive tag lists
2. **Generate new UIDs**: Create new identifiers for anonymized data
3. **Shift dates consistently**: Maintain temporal relationships
4. **Remove private tags**: Check for vendor-specific identifiers
5. **Validate anonymization**: Verify no PHI remains

---

## Common Pitfalls

### 1. Byte Order Confusion

**Problem**: Incorrectly handling little-endian vs big-endian encoding

**Solution**: Check Transfer Syntax UID and handle accordingly
```python
import struct

# Little-endian (most common)
value = struct.unpack('<H', bytes_data)[0]

# Big-endian
value = struct.unpack('>H', bytes_data)[0]
```

### 2. VR Length Fields

**Problem**: Incorrect VR length calculation (2 vs 4 bytes)

**Solution**: Some VRs use 4-byte length fields (OB, OD, OF, OL, OW, SQ, UC, UN, UR, UT)
```python
# VRs with 2-byte reserved field and 4-byte length
long_vrs = ['OB', 'OD', 'OF', 'OL', 'OW', 'SQ', 'UC', 'UN', 'UR', 'UT']

if vr in long_vrs:
    # Skip 2 reserved bytes, read 4-byte length
    length = struct.unpack('<I', file.read(4))[0]
else:
    # Read 2-byte length directly
    length = struct.unpack('<H', file.read(2))[0]
```

### 3. Undefined Length Sequences

**Problem**: Not handling sequences with undefined length (0xFFFFFFFF)

**Solution**: Parse until Sequence Delimitation Item (FFFE,E0DD)
```python
if length == 0xFFFFFFFF:
    # Undefined length - read until delimiter
    while True:
        tag = read_tag(file)
        if tag == (0xFFFE, 0xE0DD):
            break
        # Process item
```

### 4. Implicit VR Pitfalls

**Problem**: Assuming VR is present when using Implicit VR Transfer Syntax

**Solution**: Look up VR in data dictionary for Implicit VR
```python
from pydicom.datadict import dictionary_VR

def get_vr(tag, transfer_syntax):
    if is_explicit_vr(transfer_syntax):
        return read_vr_from_file()
    else:
        return dictionary_VR(tag)
```

### 5. Missing File Meta Information

**Problem**: Forgetting to include File Meta Information when creating files

**Solution**: Always create proper File Meta Information group
```python
file_meta = Dataset()
file_meta.FileMetaInformationGroupLength = calculate_length()
file_meta.FileMetaInformationVersion = b'\x00\x01'
file_meta.MediaStorageSOPClassUID = sop_class_uid
file_meta.MediaStorageSOPInstanceUID = instance_uid
file_meta.TransferSyntaxUID = transfer_syntax_uid
file_meta.ImplementationClassUID = impl_class_uid
```

### 6. Character Encoding Issues

**Problem**: Not handling special characters correctly

**Solution**: Check Specific Character Set (0008,0005) and decode properly
```python
if hasattr(ds, 'SpecificCharacterSet'):
    charset = ds.SpecificCharacterSet
    # Handle encoding: ISO_IR 100, ISO_IR 192, etc.
else:
    charset = 'ascii'  # Default
```

### 7. Private Tags Confusion

**Problem**: Mishandling private tags (odd group numbers)

**Solution**: Private tags need private creator element
```python
# Group numbers 0x0009-0xFFFF with odd values are private
is_private = (tag.group % 2 == 1) and (tag.group >= 0x0009)

if is_private:
    # Need to find private creator
    creator_element = (tag.group, tag.element >> 8)
```

### 8. Pixel Data Interpretation

**Problem**: Incorrect pixel value interpretation

**Solution**: Check Photometric Interpretation, Rescale Slope/Intercept
```python
# Get pixel array
pixels = ds.pixel_array

# Apply modality LUT
if hasattr(ds, 'RescaleSlope') and hasattr(ds, 'RescaleIntercept'):
    pixels = pixels * ds.RescaleSlope + ds.RescaleIntercept

# Handle photometric interpretation
if ds.PhotometricInterpretation == 'MONOCHROME1':
    # Invert: lower values = brighter
    pixels = np.max(pixels) - pixels
```

### 9. Multi-frame Images

**Problem**: Treating multi-frame images as single frame

**Solution**: Check Number of Frames attribute
```python
num_frames = getattr(ds, 'NumberOfFrames', 1)

if num_frames > 1:
    # Reshape pixel data
    frames = ds.pixel_array.reshape(num_frames, ds.Rows, ds.Columns)
    # Process each frame
    for i, frame in enumerate(frames):
        process_frame(frame)
```

### 10. Network Timeout Issues

**Problem**: Not setting appropriate timeouts for network operations

**Solution**: Configure reasonable timeouts based on operation type
```python
# Set timeouts
ae.network_timeout = 30  # Connection timeout
ae.acse_timeout = 30     # Association timeout
ae.dimse_timeout = 60    # DIMSE operation timeout

# For large files, increase timeout
if file_size > 100_000_000:  # 100 MB
    ae.dimse_timeout = 300  # 5 minutes
```

---

## Security Considerations

### Patient Privacy (HIPAA/GDPR)

1. **Encrypt data at rest**: Use file system or database encryption
2. **Encrypt data in transit**: Use TLS for DICOM networking
3. **Implement access controls**: Role-based access to DICOM data
4. **Audit all access**: Log who accessed what data and when
5. **Proper anonymization**: Remove all PHI when sharing data

### Secure DICOM Networking

1. **Use DICOM TLS**: Implement secure DICOM protocol
2. **Mutual authentication**: Both parties verify identity
3. **Certificate management**: Proper PKI infrastructure
4. **Network segmentation**: Isolate medical networks

### Input Validation

1. **Validate file size**: Prevent resource exhaustion
2. **Check file format**: Verify DICOM structure
3. **Sanitize paths**: Prevent directory traversal
4. **Limit recursion**: Prevent stack overflow with sequences
5. **Validate UIDs**: Ensure proper format

### Common Vulnerabilities

1. **Buffer overflows**: Check lengths before copying data
2. **Integer overflows**: Validate arithmetic operations
3. **Path traversal**: Sanitize file paths
4. **XML attacks**: Be careful with Structured Reports
5. **Denial of Service**: Implement rate limiting

### Security Checklist

- [ ] All network traffic encrypted (TLS)
- [ ] Authentication required for all operations
- [ ] Authorization checks before data access
- [ ] Audit logging enabled
- [ ] PHI properly protected
- [ ] Input validation implemented
- [ ] Error messages don't leak sensitive info
- [ ] Secure defaults configured
- [ ] Regular security updates applied
- [ ] Vulnerability scanning performed

---

## Resources and References

### Official Standards

- **DICOM Standard**: [https://www.dicomstandard.org/](https://www.dicomstandard.org/)
- **NEMA Publications**: [https://www.nema.org/](https://www.nema.org/)
- **DICOM Conformance Statements**: Check vendor websites

### Online Resources

- **DICOM Library**: [https://www.dicomlibrary.com/](https://www.dicomlibrary.com/)
- **Innolitics DICOM Browser**: [https://dicom.innolitics.com/](https://dicom.innolitics.com/)
- **DICOM is Easy**: [https://dicomiseasy.blogspot.com/](https://dicomiseasy.blogspot.com/)

### Books

1. **"Digital Imaging and Communications in Medicine (DICOM)"** - Oleg S. Pianykh
2. **"PACS and Imaging Informatics"** - Huang, Zhou, Zhang
3. **"Practical Imaging Informatics"** - Branstetter

### Sample Data

- **Medical Image Net**: Public DICOM datasets
- **The Cancer Imaging Archive (TCIA)**: [https://www.cancerimagingarchive.net/](https://www.cancerimagingarchive.net/)
- **DICOM Sample Images**: Various vendors provide test images

### Testing Tools

- **DVTk**: DICOM Validation Toolkit
- **DICOM Web Viewer**: Online DICOM viewer for testing
- **Orthanc**: Lightweight DICOM server for development

### Communities

- **DICOM Google Group**: Active community discussions
- **Stack Overflow**: Tag [dicom]
- **GitHub**: Many open-source DICOM projects

### Training and Certification

- **IHE Connectathons**: Annual testing events
- **SIIM (Society for Imaging Informatics in Medicine)**: Professional organization
- **PACS Administrator Training**: Various vendors and organizations

---

## Appendix A: Quick Reference

### Common DICOM Tags

```
(0008,0018) SOP Instance UID
(0008,0020) Study Date
(0008,0030) Study Time
(0008,0060) Modality
(0008,0070) Manufacturer
(0010,0010) Patient Name
(0010,0020) Patient ID
(0010,0030) Patient Birth Date
(0010,0040) Patient Sex
(0020,000D) Study Instance UID
(0020,000E) Series Instance UID
(0020,0011) Series Number
(0020,0013) Instance Number
(0028,0010) Rows
(0028,0011) Columns
(0028,0100) Bits Allocated
(0028,0103) Pixel Representation
(7FE0,0010) Pixel Data
```

### DIMSE Commands Quick Reference

| Command | Purpose | Example Use Case |
|---------|---------|------------------|
| C-ECHO | Verify connection | Test PACS connectivity |
| C-STORE | Store instance | Send image to PACS |
| C-FIND | Query | Search for studies |
| C-GET | Retrieve to SCU | Pull images from PACS |
| C-MOVE | Send to third party | Send images to workstation |

### Transfer Syntax Quick Reference

| UID | Name | Compression |
|-----|------|-------------|
| 1.2.840.10008.1.2 | Implicit VR LE | None |
| 1.2.840.10008.1.2.1 | Explicit VR LE | None |
| 1.2.840.10008.1.2.4.50 | JPEG Baseline | Lossy |
| 1.2.840.10008.1.2.4.70 | JPEG Lossless | Lossless |
| 1.2.840.10008.1.2.4.90 | JPEG 2000 Lossless | Lossless |
| 1.2.840.10008.1.2.5 | RLE Lossless | Lossless |

---

## Appendix B: Troubleshooting Guide

### Problem: Cannot read DICOM file

**Possible causes:**
1. Not a valid DICOM file (check for "DICM" prefix)
2. Corrupted file
3. Unsupported transfer syntax
4. Incorrect file permissions

**Solutions:**
- Use `dcmdump` or similar tool to validate
- Check file integrity
- Convert transfer syntax with `dcmconv`
- Verify file permissions

### Problem: Network association fails

**Possible causes:**
1. Incorrect IP address or port
2. Wrong AE Title
3. Firewall blocking connection
4. SCP not running
5. Unsupported presentation contexts

**Solutions:**
- Test with C-ECHO using `echoscu`
- Verify AE Title configuration
- Check firewall rules
- Confirm SCP is listening
- Review presentation context negotiation

### Problem: Pixel data displays incorrectly

**Possible causes:**
1. Wrong photometric interpretation
2. Missing window/level application
3. Incorrect byte order
4. Wrong bits allocated/stored

**Solutions:**
- Check Photometric Interpretation tag
- Apply Window Center/Width
- Verify transfer syntax byte order
- Review pixel representation settings

### Problem: Query returns no results

**Possible causes:**
1. Incorrect query keys
2. Wrong query level
3. Wildcards not supported
4. Date format incorrect

**Solutions:**
- Use proper wildcards (*)
- Set correct QueryRetrieveLevel
- Check date format (YYYYMMDD)
- Test with known existing data

---

## Conclusion

This guide provides a comprehensive overview of DICOM for developers. Remember:

1. **Start Simple**: Begin with basic file I/O before networking
2. **Use Libraries**: Leverage existing DICOM libraries
3. **Test Thoroughly**: Use sample data and validation tools
4. **Follow Standards**: Adhere to DICOM specifications
5. **Consider Security**: Always protect patient privacy
6. **Stay Updated**: DICOM standard evolves continuously

DICOM is complex but powerful. With proper understanding and tools, you can build robust medical imaging applications that integrate seamlessly with healthcare systems worldwide.

For questions and updates, refer to the [DICOM Standard](https://www.dicomstandard.org/) and community resources.

---

**Document Version**: 2.0  
**Last Updated**: 2026  
**License**: This document is provided for educational purposes.
