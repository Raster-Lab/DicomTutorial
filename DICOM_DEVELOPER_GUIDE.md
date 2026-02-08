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

## Advanced DICOM Networking & PDU Protocol

### Protocol Data Units (PDUs)

DICOM networking uses Protocol Data Units (PDUs) for message exchange over TCP/IP. Understanding PDU structure is essential for implementing DICOM network services.

#### PDU Types

| PDU Type | Value | Purpose |
|----------|-------|---------|
| A-ASSOCIATE-RQ | 0x01 | Association request from SCU |
| A-ASSOCIATE-AC | 0x02 | Association acceptance from SCP |
| A-ASSOCIATE-RJ | 0x03 | Association rejection from SCP |
| P-DATA-TF | 0x04 | Transfer of DIMSE messages |
| A-RELEASE-RQ | 0x05 | Release request |
| A-RELEASE-RP | 0x06 | Release response |
| A-ABORT | 0x07 | Abort association |

### PDU Structure Details

#### A-ASSOCIATE-RQ PDU Structure

```
+-------------------+
| PDU-type (0x01)   | 1 byte
+-------------------+
| Reserved          | 1 byte
+-------------------+
| PDU-length        | 4 bytes (big-endian)
+-------------------+
| Protocol-version  | 2 bytes
+-------------------+
| Reserved          | 2 bytes
+-------------------+
| Called-AE-title   | 16 bytes (space-padded)
+-------------------+
| Calling-AE-title  | 16 bytes (space-padded)
+-------------------+
| Reserved          | 32 bytes
+-------------------+
| Variable Items:   |
|  - Application    |
|    Context        |
|  - Presentation   |
|    Contexts       |
|  - User Info      |
+-------------------+
```

**Application Context Item** (0x10):
- Identifies DICOM application layer protocol
- Standard value: `1.2.840.10008.3.1.1.1`

**Presentation Context Item** (0x20):
- Contains:
  - Presentation Context ID (odd number, 1-255)
  - Abstract Syntax (SOP Class UID)
  - Transfer Syntax list (one or more)

**User Information Item** (0x50):
- Maximum Length Sub-item (0x51): Maximum PDU size
- Implementation Class UID (0x52): Implementation identifier
- Implementation Version Name (0x55): Software version
- SCP/SCU Role Selection (0x54): Negotiates roles
- Extended Negotiation (0x56): Extended features

#### P-DATA-TF PDU Structure

```
+-------------------+
| PDU-type (0x04)   | 1 byte
+-------------------+
| Reserved          | 1 byte
+-------------------+
| PDU-length        | 4 bytes
+-------------------+
| Presentation-data-|
| value Items       | Variable length
+-------------------+
```

Each Presentation-data-value item contains:
- Presentation Context ID (1 byte)
- DIMSE message fragments with:
  - Message Control Header (bit 0: last fragment, bit 1: command/data)
  - DIMSE command or data fragment

### Association Negotiation Deep Dive

#### Step-by-Step Association Process

**1. SCU Prepares Association Request**
```
A-ASSOCIATE-RQ includes:
- Called AE Title: "PACS_SERVER"
- Calling AE Title: "WORKSTATION1"
- Application Context: 1.2.840.10008.3.1.1.1
- Presentation Contexts (example):
  
  PC ID 1:
    Abstract Syntax: CT Image Storage (1.2.840.10008.5.1.4.1.1.2)
    Transfer Syntaxes:
      - Explicit VR Little Endian (1.2.840.10008.1.2.1)
      - Implicit VR Little Endian (1.2.840.10008.1.2)
      - JPEG Lossless (1.2.840.10008.1.2.4.70)
  
  PC ID 3:
    Abstract Syntax: MR Image Storage (1.2.840.10008.5.1.4.1.1.4)
    Transfer Syntaxes:
      - Explicit VR Little Endian (1.2.840.10008.1.2.1)
  
- User Information:
  Max PDU Length: 16384 bytes
  Implementation Class UID: 1.2.840.xxxxx
  Implementation Version: MyDICOM_v1.0
```

**2. SCP Evaluates Request**
- Validates Called AE Title matches configured AE
- Checks each Presentation Context:
  - Accepts or rejects Abstract Syntax
  - Selects ONE Transfer Syntax from offered list
  - Assigns acceptance/rejection code

**3. SCP Sends Association Response**
```
A-ASSOCIATE-AC includes:
- Presentation Context Results:
  
  PC ID 1: Accepted (0x00)
    Selected Transfer Syntax: JPEG Lossless (1.2.840.10008.1.2.4.70)
  
  PC ID 3: Accepted (0x00)
    Selected Transfer Syntax: Explicit VR Little Endian (1.2.840.10008.1.2.1)
  
- User Information:
  Max PDU Length: 16384 bytes (confirmed or smaller)
  Implementation Class UID: 1.2.840.yyyyy
  Implementation Version: PACSServer_v2.5
```

#### Presentation Context Negotiation Result Codes

| Code | Meaning |
|------|---------|
| 0x00 | Acceptance |
| 0x01 | User rejection |
| 0x02 | No reason given (provider rejection) |
| 0x03 | Abstract syntax not supported |
| 0x04 | Transfer syntaxes not supported |

### Maximum PDU Size Negotiation

**Purpose**: Limits memory usage and prevents buffer overflows

**Negotiation**:
1. SCU proposes maximum PDU size (e.g., 16384, 32768, 65536 bytes)
2. SCP accepts or proposes smaller value
3. Both sides use the MINIMUM of proposed values

**Common Values**:
- **16384** (16 KB): Conservative default
- **32768** (32 KB): Common for moderate loads
- **65536** (64 KB): High performance
- **131072** (128 KB): Maximum practical limit

**Impact on Performance**:
- Larger PDU = fewer network round-trips
- Larger PDU = more memory usage
- Optimal size depends on network latency and bandwidth

**Calculation Example**:
```
Image size: 512 x 512 x 2 bytes = 524,288 bytes
PDU size: 16,384 bytes
Number of P-DATA PDUs needed: 524,288 / 16,384 = 32 PDUs
```

### DICOM Network State Machine

```
        +----------+
        |  Idle    |<--------+
        +----------+         |
             |               |
             | (Open TCP)    |
             v               |
        +----------+         |
        | Awaiting |         |
        | A-ASSOC  |         |
        +----------+         |
             |               |
   +---------+---------+     |
   |                   |     |
   v (RQ)           v (AC)   |
+----------+    +----------+ |
| Negotia- |    |  Assoc   | |
| tion     |    |  Estab-  | |
+----------+    | lished   | |
   |            +----------+ |
   | (AC/RJ)         |       |
   |                 |       |
   +-------+---------+       |
           |                 |
           v                 |
      +----------+           |
      | Data     |           |
      | Transfer |           |
      +----------+           |
           |                 |
           | (A-RELEASE)     |
           v                 |
      +----------+           |
      | Release  |-----------+
      | Pending  |
      +----------+

States:
1. Idle: No connection
2. Awaiting A-ASSOCIATE: TCP connected, waiting for association
3. Negotiation: Processing association request
4. Association Established: Ready for DIMSE operations
5. Data Transfer: Active DIMSE commands
6. Release Pending: Graceful shutdown in progress
```

### Handling Network Errors

#### Timeout Strategies

**Connection Timeout**:
```python
# Example timeout configuration
CONNECT_TIMEOUT = 30  # seconds to establish TCP connection
ASSOC_TIMEOUT = 30    # seconds to complete association negotiation
DIMSE_TIMEOUT = 60    # seconds for DIMSE command response
IDLE_TIMEOUT = 300    # seconds of inactivity before auto-release
```

**Timeout Handling Pattern**:
```python
import socket
import time

def connect_with_timeout(host, port, timeout=30):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(timeout)
    try:
        sock.connect((host, port))
        return sock
    except socket.timeout:
        raise ConnectionError(f"Connection timeout after {timeout}s")
    except Exception as e:
        raise ConnectionError(f"Connection failed: {e}")

def send_with_retry(sock, data, max_retries=3):
    for attempt in range(max_retries):
        try:
            sock.sendall(data)
            return True
        except socket.error as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)  # Exponential backoff
    return False
```

#### Association Rejection Handling

**Rejection Reasons**:
1. **Called AE Title Not Recognized**: Wrong destination AE
2. **Calling AE Title Not Recognized**: Source not authorized
3. **No Presentation Contexts Accepted**: No compatible SOP Classes
4. **Temporary Congestion**: SCP too busy

**Handling Strategy**:
```python
def handle_association_rejection(rejection_pdu):
    result = rejection_pdu.result
    source = rejection_pdu.source
    reason = rejection_pdu.reason
    
    if source == 1:  # DICOM UL service-user
        if reason == 1:
            raise ValueError("Called AE Title not recognized")
        elif reason == 2:
            raise ValueError("Calling AE Title not recognized")
        elif reason == 3:
            raise PermissionError("Calling AE Title not authorized")
    elif source == 2:  # DICOM UL service-provider (ACSE)
        if reason == 1:
            raise ConnectionError("No reason given")
        elif reason == 2:
            raise NotImplementedError("Application context name not supported")
    elif source == 3:  # DICOM UL service-provider (Presentation)
        if reason == 1:
            raise ConnectionError("Temporary congestion")
        elif reason == 2:
            raise ConnectionError("Local limit exceeded")
```

#### Abort Handling

**Abort Sources**:
- **DICOM UL service-user**: Application layer abort
- **DICOM UL service-provider**: Protocol error

**Common Abort Reasons**:
1. **Invalid PDU**: Malformed PDU received
2. **Unexpected PDU**: PDU received in wrong state
3. **Unrecognized PDU**: Unknown PDU type
4. **Application Timeout**: Application took too long
5. **Protocol Error**: Violation of DICOM Upper Layer Protocol

**Abort Recovery**:
```python
def handle_abort(abort_pdu):
    source = abort_pdu.source
    reason = abort_pdu.reason
    
    # Log the abort
    logger.error(f"Association aborted - Source: {source}, Reason: {reason}")
    
    # Clean up resources
    close_tcp_connection()
    
    # Decide on retry strategy
    if reason in [0x00, 0x02]:  # Timeout or application error
        # May be transient, retry with backoff
        return "RETRY"
    else:  # Protocol errors
        # Likely configuration issue, don't retry immediately
        return "FAIL"
```

### Asynchronous Operations Window

**Purpose**: Allow multiple outstanding DIMSE operations on single association

**Negotiation**:
```
User Information Item includes:
  Asynchronous Operations Window Sub-item:
    - Max operations invoked (by SCU)
    - Max operations performed (by SCP)
```

**Use Cases**:
- Parallel image retrieval (multiple C-STORE operations)
- Concurrent queries
- Improved throughput for high-latency connections

**Example**:
```python
# Request async window during association
assoc_rq.user_info.async_ops_window = (5, 1)  # 5 invoked, 1 performed

# After association accepted, check negotiated values
if assoc_ac.user_info.async_ops_window:
    max_invoked, max_performed = assoc_ac.user_info.async_ops_window
    # Can now send up to max_invoked operations before waiting
```

### PDU Fragmentation

**When P-DATA PDUs are Fragmented**:
- DIMSE message exceeds maximum PDU size
- Large pixel data needs multiple PDUs

**Fragment Indicators**:
- **Message Control Header byte**:
  - Bit 0: `0` = more fragments, `1` = last fragment
  - Bit 1: `0` = command set, `1` = data set

**Reassembly Algorithm**:
```python
def reassemble_pdus(pdu_list):
    """Reassemble fragmented P-DATA PDUs into complete DIMSE message"""
    command_fragments = []
    data_fragments = []
    
    for pdu in pdu_list:
        for pv_item in pdu.presentation_data_values:
            control_header = pv_item.message_control_header
            is_last = (control_header & 0x02) != 0
            is_data = (control_header & 0x01) != 0
            
            if is_data:
                data_fragments.append(pv_item.data)
            else:
                command_fragments.append(pv_item.data)
            
            if is_last:
                # Complete message received
                break
    
    command = b''.join(command_fragments)
    data = b''.join(data_fragments)
    
    return decode_dimse_command(command), data
```

### Network Performance Optimization

#### Connection Pooling

**Strategy**: Maintain pool of persistent associations

```python
class AssociationPool:
    def __init__(self, host, port, ae_title, max_connections=5):
        self.host = host
        self.port = port
        self.ae_title = ae_title
        self.max_connections = max_connections
        self.pool = []
        self.in_use = set()
    
    def acquire(self):
        """Get association from pool or create new one"""
        # Try to reuse idle association
        for assoc in self.pool:
            if assoc not in self.in_use:
                self.in_use.add(assoc)
                return assoc
        
        # Create new if under limit
        if len(self.pool) < self.max_connections:
            assoc = self.create_association()
            self.pool.append(assoc)
            self.in_use.add(assoc)
            return assoc
        
        # Wait for available association
        return self.wait_for_available()
    
    def release(self, assoc):
        """Return association to pool"""
        self.in_use.remove(assoc)
```

#### TCP Options for DICOM

```python
import socket

def optimize_socket(sock):
    """Optimize TCP socket for DICOM traffic"""
    # Disable Nagle's algorithm for lower latency
    sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)
    
    # Increase buffer sizes for large images
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_SNDBUF, 262144)  # 256 KB
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 262144)  # 256 KB
    
    # Enable keep-alive to detect dead connections
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)
    
    # Set keep-alive parameters (Linux-specific)
    if hasattr(socket, 'TCP_KEEPIDLE'):
        sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPIDLE, 60)   # 60s before first keep-alive
        sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPINTVL, 10)  # 10s between keep-alives
        sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPCNT, 5)     # 5 failed keep-alives
```

### Troubleshooting Network Issues

#### Diagnostic Steps

**1. Test Basic Connectivity**
```bash
# Test TCP connectivity
telnet pacs-server.hospital.com 11112

# Or with netcat
nc -zv pacs-server.hospital.com 11112

# Test DICOM echo
echoscu -v pacs-server.hospital.com 11112 -aet MYSCU -aec PACSSCP
```

**2. Capture Network Traffic**
```bash
# Capture DICOM traffic with tcpdump
tcpdump -i eth0 -w dicom_capture.pcap 'port 11112'

# Analyze with Wireshark (DICOM dissector)
wireshark dicom_capture.pcap
```

**3. Enable Verbose Logging**
```python
# Enable detailed logging in pynetdicom
import logging
logging.basicConfig(level=logging.DEBUG)

# For DCMTK, use debug flags
# dcmnet: +v for verbose, +d for debug
storescu +v +d pacs-server 11112 image.dcm
```

#### Common Network Problems

**Problem 1: Association Timeout**
```
Symptoms: Connection hangs during association
Causes:
  - Firewall blocking return traffic
  - SCP not listening on specified port
  - Wrong AE Title configuration
  - Network congestion

Solutions:
  1. Verify SCP is running: netstat -an | grep 11112
  2. Check firewall rules
  3. Verify AE Title configuration
  4. Test with echoscu
```

**Problem 2: Association Rejected**
```
Symptoms: A-ASSOCIATE-RJ received
Causes:
  - Called AE Title mismatch
  - Calling AE Title not authorized
  - No compatible presentation contexts
  
Solutions:
  1. Verify AE Titles: check SCU and SCP configs
  2. Check SCP authorization list
  3. Review presentation contexts in logs
  4. Ensure matching SOP Classes
```

**Problem 3: Data Transfer Fails**
```
Symptoms: Association succeeds but C-STORE fails
Causes:
  - Insufficient disk space on SCP
  - Permissions issues
  - Unsupported transfer syntax
  - Corrupted pixel data
  
Solutions:
  1. Check SCP disk space
  2. Verify file permissions
  3. Review accepted transfer syntaxes
  4. Validate DICOM file with dciodvfy
```

**Problem 4: Slow Transfer Speed**
```
Symptoms: Images transfer very slowly
Causes:
  - Small PDU size
  - Network latency
  - Nagle's algorithm enabled
  - Sequential operations (no async)
  
Solutions:
  1. Increase PDU size to 64KB
  2. Enable TCP_NODELAY
  3. Use asynchronous operations window
  4. Check network bandwidth
```

#### Network Debugging Checklist

- [ ] TCP connection establishes successfully
- [ ] AE Titles match on both sides
- [ ] Presentation contexts are compatible
- [ ] Transfer syntaxes are supported
- [ ] Maximum PDU size is reasonable
- [ ] Timeouts are configured appropriately
- [ ] Firewall allows bi-directional traffic
- [ ] No intermediate proxies interfering
- [ ] Sufficient disk space on SCP
- [ ] File permissions correct
- [ ] DICOM files are valid
- [ ] Logging enabled for debugging

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

## DICOM Error Handling & Status Codes

### DIMSE Status Codes Overview

Every DIMSE operation returns a status code indicating success, failure, or pending state. Understanding these codes is critical for robust error handling.

### Status Code Categories

| Category | Range | Meaning |
|----------|-------|---------|
| **Success** | 0x0000 | Operation completed successfully |
| **Pending** | 0xFF00, 0xFF01 | Operation in progress, more responses coming |
| **Cancel** | 0xFE00 | Operation canceled by user |
| **Warning** | 0x0001, 0x0107, 0xB000-0xBFFF | Operation completed with warnings |
| **Failure** | 0x0102-0x0122, 0xA700-0xA9FF, 0xC000-0xCFFF | Operation failed |

### Detailed Status Codes by Service

#### C-STORE Status Codes

| Status | Code | Meaning | Action |
|--------|------|---------|--------|
| Success | 0x0000 | Storage successful | None |
| Warning | 0xB000 | Coercion of data elements | Log warning, data was modified |
| Warning | 0xB006 | Elements discarded | Log warning, some elements not stored |
| Warning | 0xB007 | Data set does not match SOP Class | Log warning, possible validation issue |
| Failure | 0x0117 | Invalid SOP Instance | Reject file, log error |
| Failure | 0x0118 | No such SOP Class | SOP Class not supported |
| Failure | 0x0119 | Class-Instance conflict | Instance UID doesn't match SOP Class |
| Failure | 0x0122 | SOP Class not supported | Reject transfer |
| Failure | 0x0124 | Not authorized | Permission denied |
| Failure | 0x0210 | Duplicate invocation | Request already in progress |
| Failure | 0x0211 | Unrecognized operation | Invalid operation type |
| Failure | 0x0212 | Mistyped argument | Invalid parameter |
| Failure | 0xA700-0xA7FF | Out of resources | Insufficient storage/memory |
| Failure | 0xA900-0xA9FF | Data set does not match SOP Class | Invalid attributes |
| Failure | 0xC000-0xCFFF | Cannot understand | Processing error |

**C-STORE Error Handling Example**:
```python
def handle_store_status(status_code, status_dataset):
    """Handle C-STORE response status"""
    if status_code == 0x0000:
        return "SUCCESS"
    
    elif status_code == 0xB000:
        # Warning: Coercion occurred
        logger.warning("SCP coerced data elements during storage")
        return "SUCCESS_WITH_WARNING"
    
    elif status_code == 0xB007:
        # Warning: Data set doesn't match SOP Class
        logger.warning("Data set mismatch - image stored but may have issues")
        return "SUCCESS_WITH_WARNING"
    
    elif status_code == 0x0117:
        raise ValueError("Invalid SOP Instance UID")
    
    elif status_code == 0x0122:
        raise NotImplementedError("SOP Class not supported by SCP")
    
    elif status_code == 0x0124:
        raise PermissionError("Storage not authorized")
    
    elif 0xA700 <= status_code <= 0xA7FF:
        raise IOError("SCP out of resources (disk space/memory)")
    
    elif 0xC000 <= status_code <= 0xCFFF:
        error_comment = status_dataset.get('ErrorComment', 'Unknown error')
        raise RuntimeError(f"Storage failed: {error_comment}")
    
    else:
        raise RuntimeError(f"Unknown status code: 0x{status_code:04X}")
```

#### C-FIND Status Codes

| Status | Code | Meaning | Action |
|--------|------|---------|--------|
| Pending | 0xFF00 | Matches are continuing | Wait for more responses |
| Pending | 0xFF01 | Matches continuing, warning | Log warning, continue |
| Success | 0x0000 | Matching complete | Process results |
| Cancel | 0xFE00 | Matching terminated | Handle cancellation |
| Failure | 0x0122 | SOP Class not supported | Query not supported |
| Failure | 0xA700 | Out of resources | SCP cannot process query |
| Failure | 0xA900 | Identifier does not match SOP Class | Invalid query keys |
| Failure | 0xC000-0xCFFF | Unable to process | Query failed |

**C-FIND Handling Pattern**:
```python
def perform_find(association, query_dataset):
    """Execute C-FIND and collect results"""
    results = []
    errors = []
    
    # Send C-FIND request
    responses = association.send_c_find(query_dataset)
    
    for status, identifier in responses:
        if status.Status == 0xFF00 or status.Status == 0xFF01:
            # Pending - add result
            if identifier:
                results.append(identifier)
            if status.Status == 0xFF01:
                # Pending with warning
                logger.warning(f"Query warning: {status.ErrorComment}")
        
        elif status.Status == 0x0000:
            # Success - query complete
            logger.info(f"Query complete. Found {len(results)} matches")
            break
        
        elif status.Status == 0xFE00:
            # Canceled
            logger.info("Query was canceled")
            break
        
        elif status.Status == 0x0122:
            raise NotImplementedError("Query/Retrieve SOP Class not supported")
        
        elif status.Status == 0xA900:
            raise ValueError(f"Invalid query keys: {status.ErrorComment}")
        
        elif status.Status >= 0xC000:
            error_msg = status.ErrorComment or "Unknown error"
            raise RuntimeError(f"Query failed: {error_msg}")
    
    return results
```

#### C-GET Status Codes

| Status | Code | Meaning | Action |
|--------|------|---------|--------|
| Pending | 0xFF00 | Sub-operations continuing | Monitor progress |
| Success | 0x0000 | All sub-operations complete | Verify all images received |
| Cancel | 0xFE00 | Retrieval canceled | Handle partial transfer |
| Warning | 0xB000 | One or more sub-operations failed | Check which images failed |
| Failure | 0x0122 | SOP Class not supported | Cannot retrieve |
| Failure | 0xA701 | Out of resources (performing) | SCP cannot send images |
| Failure | 0xA702 | Out of resources (unable to calculate) | Cannot determine matches |
| Failure | 0xA900 | Identifier does not match SOP Class | Invalid retrieve keys |
| Failure | 0xC000-0xCFFF | Unable to perform sub-operations | Retrieval failed |

**C-GET Progress Tracking**:
```python
def perform_get_with_progress(association, query_dataset):
    """Execute C-GET and track progress"""
    received_images = []
    failed_images = []
    
    # Handler for received C-STORE (sub-operation)
    def handle_store(event):
        """Called when SCP sends image via C-STORE"""
        try:
            dataset = event.dataset
            received_images.append(dataset)
            return 0x0000  # Success
        except Exception as e:
            logger.error(f"Failed to receive image: {e}")
            failed_images.append(event)
            return 0xC000  # Failure
    
    # Bind C-STORE handler
    association.bind_c_store(handle_store)
    
    # Send C-GET request
    responses = association.send_c_get(query_dataset)
    
    for status, identifier in responses:
        if status.Status == 0xFF00:
            # Pending - show progress
            remaining = status.NumberOfRemainingSuboperations or 0
            completed = status.NumberOfCompletedSuboperations or 0
            failed = status.NumberOfFailedSuboperations or 0
            warning = status.NumberOfWarningSuboperations or 0
            
            total = remaining + completed + failed + warning
            progress = (completed + failed + warning) / total * 100 if total > 0 else 0
            
            logger.info(f"Progress: {progress:.1f}% ({completed}/{total} complete, {failed} failed)")
        
        elif status.Status == 0x0000:
            # Success
            logger.info(f"Retrieval complete. Received {len(received_images)} images")
            break
        
        elif status.Status == 0xB000:
            # Warning - some images failed
            logger.warning(f"Retrieval completed with {len(failed_images)} failures")
            break
        
        elif status.Status >= 0xC000:
            error = status.ErrorComment or "Unknown error"
            raise RuntimeError(f"Retrieval failed: {error}")
    
    return received_images, failed_images
```

#### C-MOVE Status Codes

Similar to C-GET but includes destination routing:

| Status | Code | Meaning | Action |
|--------|------|---------|--------|
| Pending | 0xFF00 | Sub-operations continuing | Monitor progress |
| Success | 0x0000 | All sub-operations complete | Verify destination received |
| Cancel | 0xFE00 | Move canceled | Handle partial transfer |
| Warning | 0xB000 | One or more sub-operations failed | Check destination |
| Failure | 0xA801 | Move destination unknown | Invalid AE Title |
| Failure | 0xA900 | Identifier does not match SOP Class | Invalid move keys |

#### C-ECHO Status Codes

| Status | Code | Meaning |
|--------|------|---------|
| Success | 0x0000 | Echo successful - connection verified |
| Failure | 0x0122 | Verification SOP Class not supported |

#### N-CREATE, N-SET, N-GET, N-ACTION Status Codes

| Status | Code | Meaning |
|--------|------|---------|
| Success | 0x0000 | Operation successful |
| Warning | 0x0001 | Requested attribute list not available |
| Warning | 0x0107 | Attribute list error |
| Failure | 0x0110 | Processing failure |
| Failure | 0x0112 | No such SOP Instance |
| Failure | 0x0117 | Invalid object Instance |
| Failure | 0x0118 | No such SOP Class |
| Failure | 0x0119 | Class-Instance conflict |
| Failure | 0x0123 | No such action |
| Failure | 0x0124 | Not authorized |

### Error Comment and Offending Element

When errors occur, SCPs may include additional information:

**Error Comment** (0000,0902):
- Human-readable error description
- Use for logging and user feedback

**Offending Element** (0000,0901):
- Tag of element that caused the error
- Helps identify problematic attributes

```python
def extract_error_details(status_dataset):
    """Extract detailed error information from status"""
    error_info = {
        'status': status_dataset.Status,
        'comment': None,
        'offending_elements': []
    }
    
    # Get error comment
    if hasattr(status_dataset, 'ErrorComment'):
        error_info['comment'] = status_dataset.ErrorComment
    
    # Get offending elements
    if hasattr(status_dataset, 'OffendingElement'):
        offending = status_dataset.OffendingElement
        if isinstance(offending, list):
            error_info['offending_elements'] = [
                f"({tag.group:04X},{tag.element:04X})" 
                for tag in offending
            ]
        else:
            tag = offending
            error_info['offending_elements'] = [
                f"({tag.group:04X},{tag.element:04X})"
            ]
    
    return error_info
```

### Retry Strategies

#### Transient vs Permanent Failures

**Transient Failures** (retry recommended):
- 0xA700: Out of resources
- 0x0210: Duplicate invocation
- Network timeouts
- Temporary connection failures

**Permanent Failures** (don't retry):
- 0x0122: SOP Class not supported
- 0x0124: Not authorized
- 0x0117: Invalid SOP Instance
- 0xA900: Invalid identifier

#### Exponential Backoff Pattern

```python
import time
import random

def retry_with_backoff(operation, max_retries=3, base_delay=1.0):
    """Retry operation with exponential backoff"""
    for attempt in range(max_retries):
        try:
            return operation()
        except TransientError as e:
            if attempt == max_retries - 1:
                raise
            
            # Calculate delay with jitter
            delay = base_delay * (2 ** attempt)
            jitter = random.uniform(0, delay * 0.1)  # 10% jitter
            total_delay = delay + jitter
            
            logger.warning(f"Attempt {attempt + 1} failed: {e}. "
                         f"Retrying in {total_delay:.1f}s...")
            time.sleep(total_delay)
        except PermanentError as e:
            logger.error(f"Permanent failure: {e}. Not retrying.")
            raise

# Usage example
def send_image_with_retry(association, dataset):
    def send():
        status = association.send_c_store(dataset)
        
        if status.Status == 0x0000:
            return "SUCCESS"
        elif status.Status == 0xA700:
            # Out of resources - transient
            raise TransientError("SCP out of resources")
        elif status.Status == 0x0122:
            # Not supported - permanent
            raise PermanentError("SOP Class not supported")
        else:
            raise RuntimeError(f"Storage failed: 0x{status.Status:04X}")
    
    return retry_with_backoff(send, max_retries=3, base_delay=2.0)
```

### Exception Hierarchy

**Recommended Exception Structure**:
```python
class DICOMError(Exception):
    """Base class for DICOM errors"""
    pass

class DICOMNetworkError(DICOMError):
    """Network-related errors"""
    pass

class DICOMAssociationError(DICOMNetworkError):
    """Association establishment failed"""
    def __init__(self, result, source, reason):
        self.result = result
        self.source = source
        self.reason = reason
        super().__init__(f"Association rejected: {result}/{source}/{reason}")

class DICOMOperationError(DICOMError):
    """DIMSE operation failed"""
    def __init__(self, status, comment=None, offending=None):
        self.status = status
        self.comment = comment
        self.offending_elements = offending
        msg = f"Operation failed with status 0x{status:04X}"
        if comment:
            msg += f": {comment}"
        super().__init__(msg)

class DICOMStorageError(DICOMOperationError):
    """C-STORE specific error"""
    pass

class DICOMQueryError(DICOMOperationError):
    """C-FIND specific error"""
    pass

class DICOMRetrieveError(DICOMOperationError):
    """C-GET/C-MOVE specific error"""
    pass

class DICOMParseError(DICOMError):
    """File parsing error"""
    pass

class DICOMValidationError(DICOMError):
    """Data validation error"""
    pass
```

### Logging Best Practices

```python
import logging

# Configure structured logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger('dicom.network')

def log_dimse_operation(operation, peer, status, duration):
    """Log DIMSE operation with context"""
    log_data = {
        'operation': operation,
        'peer_ae': peer.ae_title,
        'peer_addr': f"{peer.address}:{peer.port}",
        'status': f"0x{status:04X}",
        'duration_ms': int(duration * 1000)
    }
    
    if status == 0x0000:
        logger.info(f"{operation} succeeded", extra=log_data)
    elif status == 0xFE00:
        logger.warning(f"{operation} canceled", extra=log_data)
    else:
        logger.error(f"{operation} failed", extra=log_data)
```

### Error Recovery Checklist

When operations fail:

- [ ] Log full error context (status, comment, offending elements)
- [ ] Determine if error is transient or permanent
- [ ] For transient errors, retry with exponential backoff
- [ ] For permanent errors, log and alert
- [ ] Update metrics/monitoring
- [ ] Check if association is still valid
- [ ] Clean up resources
- [ ] Notify user if appropriate
- [ ] Consider fallback strategy

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

## Comprehensive Pixel Data Handling

### Understanding Pixel Data Structure

DICOM pixel data is stored in tag (7FE0,0010) and its structure depends on several key attributes that must be interpreted correctly.

#### Critical Pixel Attributes

| Tag | Name | Purpose |
|-----|------|---------|
| (0028,0002) | Samples per Pixel | 1 = grayscale, 3 = RGB/YBR |
| (0028,0004) | Photometric Interpretation | Color space: MONOCHROME1/2, RGB, YBR_FULL, etc. |
| (0028,0006) | Planar Configuration | 0 = interleaved (RGBRGB...), 1 = planar (RRR...GGG...BBB...) |
| (0028,0008) | Number of Frames | Multi-frame images |
| (0028,0010) | Rows | Image height |
| (0028,0011) | Columns | Image width |
| (0028,0100) | Bits Allocated | Storage size per sample (8, 16, 32) |
| (0028,0101) | Bits Stored | Actual data bits (<= Bits Allocated) |
| (0028,0102) | High Bit | Most significant bit position (Bits Stored - 1) |
| (0028,0103) | Pixel Representation | 0 = unsigned, 1 = signed (2's complement) |
| (0028,1050) | Window Center | Display window center |
| (0028,1051) | Window Width | Display window width |
| (0028,1052) | Rescale Intercept | Linear transformation: y = mx + b (b value) |
| (0028,1053) | Rescale Slope | Linear transformation: y = mx + b (m value) |
| (0028,1101) | Red Palette Color Lookup Table Descriptor | For pseudo-color |
| (0028,1102) | Green Palette Color Lookup Table Descriptor | For pseudo-color |
| (0028,1103) | Blue Palette Color Lookup Table Descriptor | For pseudo-color |

### Pixel Data Extraction Algorithm

#### Step 1: Calculate Expected Size

```python
def calculate_pixel_data_size(dataset):
    """Calculate expected pixel data size in bytes"""
    rows = dataset.Rows
    columns = dataset.Columns
    samples_per_pixel = dataset.SamplesPerPixel
    bits_allocated = dataset.BitsAllocated
    number_of_frames = getattr(dataset, 'NumberOfFrames', 1)
    
    bytes_per_sample = bits_allocated // 8
    pixels_per_frame = rows * columns
    bytes_per_frame = pixels_per_frame * samples_per_pixel * bytes_per_sample
    total_bytes = bytes_per_frame * number_of_frames
    
    return total_bytes, bytes_per_frame
```

#### Step 2: Handle Transfer Syntax

```python
def get_pixel_data(dataset):
    """Extract pixel data handling different transfer syntaxes"""
    transfer_syntax = dataset.file_meta.TransferSyntaxUID
    
    # Uncompressed transfer syntaxes
    if transfer_syntax in ['1.2.840.10008.1.2',      # Implicit VR LE
                           '1.2.840.10008.1.2.1',    # Explicit VR LE
                           '1.2.840.10008.1.2.2']:   # Explicit VR BE
        return extract_uncompressed_pixels(dataset)
    
    # RLE Lossless
    elif transfer_syntax == '1.2.840.10008.1.2.5':
        return decode_rle_pixels(dataset)
    
    # JPEG Baseline (lossy)
    elif transfer_syntax == '1.2.840.10008.1.2.4.50':
        return decode_jpeg_baseline(dataset)
    
    # JPEG Lossless
    elif transfer_syntax == '1.2.840.10008.1.2.4.70':
        return decode_jpeg_lossless(dataset)
    
    # JPEG 2000 Lossless
    elif transfer_syntax == '1.2.840.10008.1.2.4.90':
        return decode_jpeg2000_lossless(dataset)
    
    # JPEG 2000 Lossy
    elif transfer_syntax == '1.2.840.10008.1.2.4.91':
        return decode_jpeg2000_lossy(dataset)
    
    else:
        raise NotImplementedError(f"Transfer syntax not supported: {transfer_syntax}")
```

### Uncompressed Pixel Data Extraction

```python
import numpy as np

def extract_uncompressed_pixels(dataset):
    """Extract uncompressed pixel data into numpy array"""
    # Get pixel data bytes
    pixel_bytes = dataset.PixelData
    
    # Determine byte order
    transfer_syntax = dataset.file_meta.TransferSyntaxUID
    if transfer_syntax == '1.2.840.10008.1.2.2':  # Explicit VR Big Endian
        byte_order = '>'
    else:
        byte_order = '<'  # Little Endian
    
    # Determine data type
    bits_allocated = dataset.BitsAllocated
    pixel_representation = dataset.PixelRepresentation
    
    if bits_allocated == 8:
        dtype = np.uint8 if pixel_representation == 0 else np.int8
    elif bits_allocated == 16:
        dtype = np.dtype(f'{byte_order}u2') if pixel_representation == 0 else np.dtype(f'{byte_order}i2')
    elif bits_allocated == 32:
        dtype = np.dtype(f'{byte_order}u4') if pixel_representation == 0 else np.dtype(f'{byte_order}i4')
    else:
        raise ValueError(f"Unsupported bits allocated: {bits_allocated}")
    
    # Convert bytes to numpy array
    pixel_array = np.frombuffer(pixel_bytes, dtype=dtype)
    
    # Reshape to image dimensions
    rows = dataset.Rows
    columns = dataset.Columns
    samples_per_pixel = dataset.SamplesPerPixel
    number_of_frames = getattr(dataset, 'NumberOfFrames', 1)
    
    if number_of_frames > 1:
        if samples_per_pixel == 1:
            shape = (number_of_frames, rows, columns)
        else:
            if dataset.PlanarConfiguration == 0:  # Interleaved
                shape = (number_of_frames, rows, columns, samples_per_pixel)
            else:  # Planar
                shape = (number_of_frames, samples_per_pixel, rows, columns)
                # Transpose to standard shape
                pixel_array = pixel_array.reshape(shape).transpose(0, 2, 3, 1)
                return pixel_array
    else:
        if samples_per_pixel == 1:
            shape = (rows, columns)
        else:
            if dataset.PlanarConfiguration == 0:  # Interleaved
                shape = (rows, columns, samples_per_pixel)
            else:  # Planar
                shape = (samples_per_pixel, rows, columns)
                pixel_array = pixel_array.reshape(shape).transpose(1, 2, 0)
                return pixel_array
    
    return pixel_array.reshape(shape)
```

### Multi-Frame Image Handling

```python
def extract_frame(dataset, frame_index):
    """Extract a specific frame from multi-frame image"""
    number_of_frames = getattr(dataset, 'NumberOfFrames', 1)
    
    if frame_index < 0 or frame_index >= number_of_frames:
        raise IndexError(f"Frame index {frame_index} out of range [0, {number_of_frames})")
    
    # Get all frames
    pixel_array = get_pixel_data(dataset)
    
    if number_of_frames == 1:
        return pixel_array
    else:
        return pixel_array[frame_index]

def iterate_frames(dataset):
    """Generator to iterate through frames efficiently"""
    pixel_array = get_pixel_data(dataset)
    number_of_frames = getattr(dataset, 'NumberOfFrames', 1)
    
    if number_of_frames == 1:
        yield pixel_array
    else:
        for i in range(number_of_frames):
            yield pixel_array[i]
```

### RLE (Run-Length Encoding) Decompression

```python
def decode_rle_pixels(dataset):
    """Decode RLE compressed pixel data"""
    from pydicom.pixel_data_handlers import rle_handler
    
    # RLE encoding structure:
    # Header: 16 x 4-byte segment offsets
    # Segments: Compressed data for each sample/plane
    
    pixel_bytes = dataset.PixelData
    
    # Number of segments = Samples per Pixel x Number of Frames
    samples_per_pixel = dataset.SamplesPerPixel
    number_of_frames = getattr(dataset, 'NumberOfFrames', 1)
    bytes_per_sample = dataset.BitsAllocated // 8
    
    # Parse RLE header (64 bytes)
    import struct
    header = struct.unpack('<16I', pixel_bytes[:64])
    num_segments = header[0]
    segment_offsets = header[1:num_segments + 1]
    
    # Decode each segment
    decoded_segments = []
    for i in range(num_segments):
        offset = segment_offsets[i]
        if i < num_segments - 1:
            end = segment_offsets[i + 1]
        else:
            end = len(pixel_bytes)
        
        segment_data = pixel_bytes[offset:end]
        decoded = decode_rle_segment(segment_data)
        decoded_segments.append(decoded)
    
    # Combine segments based on bits allocated
    if bytes_per_sample == 1:
        pixel_data = decoded_segments[0]
    elif bytes_per_sample == 2:
        # Combine high and low byte segments
        high_bytes = np.array(decoded_segments[0], dtype=np.uint8)
        low_bytes = np.array(decoded_segments[1], dtype=np.uint8)
        pixel_data = (high_bytes.astype(np.uint16) << 8) | low_bytes.astype(np.uint16)
    else:
        raise NotImplementedError("RLE only supports 8-bit and 16-bit data")
    
    # Reshape to image dimensions
    rows = dataset.Rows
    columns = dataset.Columns
    return pixel_data.reshape(number_of_frames, rows, columns) if number_of_frames > 1 else pixel_data.reshape(rows, columns)

def decode_rle_segment(segment_data):
    """Decode a single RLE segment"""
    decoded = bytearray()
    i = 0
    
    while i < len(segment_data):
        header_byte = segment_data[i]
        i += 1
        
        if header_byte <= 127:
            # Literal run: copy next (n+1) bytes
            count = header_byte + 1
            decoded.extend(segment_data[i:i + count])
            i += count
        elif header_byte >= 129:
            # Replicate run: repeat next byte (257-n) times
            count = 257 - header_byte
            if i < len(segment_data):
                value = segment_data[i]
                decoded.extend([value] * count)
                i += 1
        # header_byte == 128 is no-op
    
    return decoded
```

### JPEG Decompression

```python
def decode_jpeg_baseline(dataset):
    """Decode JPEG Baseline (lossy) compressed pixel data"""
    from PIL import Image
    import io
    
    pixel_bytes = dataset.PixelData
    number_of_frames = getattr(dataset, 'NumberOfFrames', 1)
    
    if number_of_frames == 1:
        # Single frame
        img = Image.open(io.BytesIO(pixel_bytes))
        return np.array(img)
    else:
        # Multi-frame: encapsulated format with offsets
        frames = extract_encapsulated_frames(pixel_bytes)
        decoded_frames = []
        for frame_data in frames:
            img = Image.open(io.BytesIO(frame_data))
            decoded_frames.append(np.array(img))
        return np.array(decoded_frames)

def decode_jpeg_lossless(dataset):
    """Decode JPEG Lossless compressed pixel data"""
    # JPEG Lossless requires specialized decoder (not standard JPEG)
    # Use pydicom's pixel_array or specialized library
    from pydicom.pixel_data_handlers.jpeg_ls_handler import get_pixeldata
    return get_pixeldata(dataset)

def extract_encapsulated_frames(pixel_data):
    """Extract individual frames from encapsulated pixel data"""
    import struct
    
    # Encapsulated format:
    # (FFFE,E000) Item with basic offset table
    # (FFFE,E000) Items with frame data
    # (FFFE,E0DD) Sequence delimiter
    
    frames = []
    offset = 0
    
    while offset < len(pixel_data):
        # Read item tag
        if offset + 8 > len(pixel_data):
            break
        
        tag = struct.unpack('<HH', pixel_data[offset:offset+4])
        length = struct.unpack('<I', pixel_data[offset+4:offset+8])[0]
        offset += 8
        
        if tag == (0xFFFE, 0xE000):  # Item
            if length == 0:
                # Basic offset table (skip)
                continue
            else:
                # Frame data
                frame_data = pixel_data[offset:offset+length]
                frames.append(frame_data)
                offset += length
        elif tag == (0xFFFE, 0xE0DD):  # Sequence delimiter
            break
        else:
            # Unknown tag, skip
            offset += length
    
    return frames
```

### JPEG 2000 Decompression

```python
def decode_jpeg2000_lossless(dataset):
    """Decode JPEG 2000 Lossless compressed pixel data"""
    try:
        import glymur
    except ImportError:
        raise ImportError("glymur library required for JPEG 2000: pip install glymur")
    
    pixel_bytes = dataset.PixelData
    number_of_frames = getattr(dataset, 'NumberOfFrames', 1)
    
    if number_of_frames == 1:
        # Single frame
        jp2 = glymur.Jp2k(io.BytesIO(pixel_bytes))
        return jp2[:]
    else:
        # Multi-frame
        frames = extract_encapsulated_frames(pixel_bytes)
        decoded_frames = []
        for frame_data in frames:
            jp2 = glymur.Jp2k(io.BytesIO(frame_data))
            decoded_frames.append(jp2[:])
        return np.array(decoded_frames)

def decode_jpeg2000_lossy(dataset):
    """Decode JPEG 2000 Lossy compressed pixel data"""
    # Same process as lossless, different quality
    return decode_jpeg2000_lossless(dataset)
```

### Color Space Conversion

```python
def convert_color_space(pixel_array, photometric_interpretation):
    """Convert pixel data to RGB color space"""
    if photometric_interpretation == 'RGB':
        # Already RGB
        return pixel_array
    
    elif photometric_interpretation == 'YBR_FULL':
        # YCbCr to RGB conversion
        return ybr_to_rgb(pixel_array)
    
    elif photometric_interpretation == 'YBR_FULL_422':
        # YCbCr 4:2:2 subsampled
        return ybr422_to_rgb(pixel_array)
    
    elif photometric_interpretation in ['MONOCHROME1', 'MONOCHROME2']:
        # Grayscale - return as-is
        return pixel_array
    
    elif photometric_interpretation == 'PALETTE COLOR':
        # Apply color palette
        return apply_palette(pixel_array)
    
    else:
        raise ValueError(f"Unsupported photometric interpretation: {photometric_interpretation}")

def ybr_to_rgb(pixel_array):
    """Convert YBR_FULL (YCbCr) to RGB"""
    # ITU-R BT.601 conversion
    Y = pixel_array[:, :, 0].astype(np.float32)
    Cb = pixel_array[:, :, 1].astype(np.float32) - 128
    Cr = pixel_array[:, :, 2].astype(np.float32) - 128
    
    R = Y + 1.402 * Cr
    G = Y - 0.344136 * Cb - 0.714136 * Cr
    B = Y + 1.772 * Cb
    
    # Clip to valid range
    rgb = np.stack([R, G, B], axis=2)
    rgb = np.clip(rgb, 0, 255).astype(np.uint8)
    
    return rgb

def apply_palette(pixel_array, dataset):
    """Apply color palette lookup table"""
    # Get palette descriptors
    red_desc = dataset.RedPaletteColorLookupTableDescriptor
    green_desc = dataset.GreenPaletteColorLookupTableDescriptor
    blue_desc = dataset.BluePaletteColorLookupTableDescriptor
    
    # Get palette data
    red_lut = dataset.RedPaletteColorLookupTableData
    green_lut = dataset.GreenPaletteColorLookupTableData
    blue_lut = dataset.BluePaletteColorLookupTableData
    
    # Create RGB image by looking up each pixel value
    rgb_image = np.zeros(pixel_array.shape + (3,), dtype=np.uint8)
    rgb_image[:, :, 0] = red_lut[pixel_array]
    rgb_image[:, :, 1] = green_lut[pixel_array]
    rgb_image[:, :, 2] = blue_lut[pixel_array]
    
    return rgb_image
```

### Applying Rescale and Window/Level

```python
def apply_modality_lut(pixel_array, dataset):
    """Apply rescale slope/intercept (Modality LUT)"""
    intercept = getattr(dataset, 'RescaleIntercept', 0)
    slope = getattr(dataset, 'RescaleSlope', 1)
    
    # Convert to float for calculation
    if slope != 1 or intercept != 0:
        pixel_array = pixel_array.astype(np.float64)
        pixel_array = pixel_array * slope + intercept
    
    return pixel_array

def apply_voi_lut(pixel_array, dataset, frame=0):
    """Apply Window Center/Width (VOI LUT)"""
    # Get window parameters
    if hasattr(dataset, 'WindowCenter'):
        window_center = dataset.WindowCenter
        window_width = dataset.WindowWidth
        
        # Handle multi-valued attributes
        if isinstance(window_center, (list, tuple)):
            window_center = window_center[frame] if frame < len(window_center) else window_center[0]
        if isinstance(window_width, (list, tuple)):
            window_width = window_width[frame] if frame < len(window_width) else window_width[0]
    else:
        # Auto-calculate from pixel data range
        window_center = (pixel_array.max() + pixel_array.min()) / 2
        window_width = pixel_array.max() - pixel_array.min()
    
    # Apply windowing
    lower = window_center - window_width / 2
    upper = window_center + window_width / 2
    
    # Linear windowing
    windowed = np.clip(pixel_array, lower, upper)
    windowed = ((windowed - lower) / (upper - lower) * 255).astype(np.uint8)
    
    return windowed

def apply_photometric_interpretation(pixel_array, dataset):
    """Handle MONOCHROME1 vs MONOCHROME2"""
    photometric = dataset.PhotometricInterpretation
    
    if photometric == 'MONOCHROME1':
        # Inverted: lower values = brighter
        if pixel_array.dtype == np.uint8:
            return 255 - pixel_array
        elif pixel_array.dtype == np.uint16:
            return 65535 - pixel_array
        else:
            return pixel_array.max() - pixel_array
    else:
        # MONOCHROME2: higher values = brighter (normal)
        return pixel_array
```

### Complete Pixel Processing Pipeline

```python
def get_display_ready_pixels(dataset, frame=0):
    """Complete pipeline to get display-ready pixel data"""
    # Step 1: Extract raw pixel data
    pixel_array = get_pixel_data(dataset)
    
    # Step 2: Extract specific frame if multi-frame
    if len(pixel_array.shape) > 2 and pixel_array.shape[0] > 1:
        pixel_array = pixel_array[frame]
    
    # Step 3: Apply rescale (Modality LUT)
    pixel_array = apply_modality_lut(pixel_array, dataset)
    
    # Step 4: Handle photometric interpretation
    pixel_array = apply_photometric_interpretation(pixel_array, dataset)
    
    # Step 5: Apply window/level (VOI LUT)
    if dataset.SamplesPerPixel == 1:
        # Grayscale - apply windowing
        pixel_array = apply_voi_lut(pixel_array, dataset, frame)
    else:
        # Color - convert color space if needed
        pixel_array = convert_color_space(pixel_array, dataset.PhotometricInterpretation)
    
    return pixel_array
```

### Handling High Bit and Bits Stored

```python
def mask_pixel_data(pixel_array, dataset):
    """Apply bit masking for Bits Stored < Bits Allocated"""
    bits_stored = dataset.BitsStored
    bits_allocated = dataset.BitsAllocated
    high_bit = dataset.HighBit
    
    if bits_stored == bits_allocated:
        return pixel_array
    
    # Create mask for valid bits
    mask = (1 << bits_stored) - 1
    
    # Apply mask
    masked = pixel_array & mask
    
    # Handle signed data
    if dataset.PixelRepresentation == 1:
        # Check if sign bit is set
        sign_bit = 1 << (bits_stored - 1)
        is_negative = (masked & sign_bit) != 0
        
        # Extend sign bit
        if is_negative.any():
            extension = ((1 << (bits_allocated - bits_stored)) - 1) << bits_stored
            masked[is_negative] |= extension
    
    return masked
```

### Pixel Data Performance Optimization

```python
def lazy_load_pixels(dataset):
    """Generator for memory-efficient frame iteration"""
    number_of_frames = getattr(dataset, 'NumberOfFrames', 1)
    
    for frame_idx in range(number_of_frames):
        # Load only this frame
        frame = extract_frame(dataset, frame_idx)
        yield get_display_ready_pixels(dataset, frame_idx)
        
        # Frame can be garbage collected after yield

def create_thumbnail(dataset, max_size=(256, 256)):
    """Create memory-efficient thumbnail"""
    from PIL import Image
    
    # Get first frame only
    pixel_array = get_display_ready_pixels(dataset, frame=0)
    
    # Convert to PIL Image
    if len(pixel_array.shape) == 2:
        img = Image.fromarray(pixel_array, mode='L')
    else:
        img = Image.fromarray(pixel_array, mode='RGB')
    
    # Create thumbnail
    img.thumbnail(max_size, Image.LANCZOS)
    
    return np.array(img)
```

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
