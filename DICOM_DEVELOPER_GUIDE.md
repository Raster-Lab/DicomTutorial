# DICOM Developer Guide

## Table of Contents

1. [Introduction to DICOM](#introduction-to-dicom)
2. [DICOM Fundamentals](#dicom-fundamentals)
3. [DICOM File Format](#dicom-file-format)
4. [DICOM Data Elements and Tags](#dicom-data-elements-and-tags)
5. [Value Representations (VR)](#value-representations-vr)
6. [Transfer Syntaxes](#transfer-syntaxes)
7. [DICOM Networking](#dicom-networking)
8. [DICOM Services (DIMSE)](#dicom-services-dimse)
9. [Information Object Definitions (IODs)](#information-object-definitions-iods)
10. [Service-Object Pair (SOP) Classes](#service-object-pair-sop-classes)
11. [Working with DICOM Files](#working-with-dicom-files)
12. [DICOM Libraries and Tools](#dicom-libraries-and-tools)
13. [Code Examples](#code-examples)
14. [Best Practices](#best-practices)
15. [Common Pitfalls](#common-pitfalls)
16. [Security Considerations](#security-considerations)
17. [Resources and References](#resources-and-references)

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

## DICOM Fundamentals

### Core Concepts

1. **Service-Object Pair (SOP)**: Combination of a DICOM service and an information object
2. **Application Entity (AE)**: A software component that communicates using DICOM
3. **Application Entity Title (AET)**: Unique identifier for an Application Entity
4. **Association**: Network connection between two Application Entities

### DICOM Architecture Layers

```
┌─────────────────────────────────────────┐
│     DICOM Applications                   │
├─────────────────────────────────────────┤
│     DICOM Message Services (DIMSE)      │
├─────────────────────────────────────────┤
│     DICOM Upper Layer Protocol          │
├─────────────────────────────────────────┤
│     TCP/IP                               │
└─────────────────────────────────────────┘
```

### Key Standards

- **Part 3**: Information Object Definitions
- **Part 4**: Service Class Specifications
- **Part 5**: Data Structures and Encoding
- **Part 6**: Data Dictionary
- **Part 7**: Message Exchange
- **Part 8**: Network Communication Support
- **Part 10**: Media Storage and File Format

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
// Maven dependency:
// <dependency>
//     <groupId>org.dcm4che</groupId>
//     <artifactId>dcm4che-core</artifactId>
//     <version>5.31.1</version>
// </dependency>
//
// Additional modules:
//   dcm4che-net      - DICOM networking (associations, DIMSE services)
//   dcm4che-imageio  - Image I/O and codec support (JPEG, JPEG 2000, etc.)
//   dcm4che-tool     - Command-line tools (storescu, findscu, etc.)
//   dcm4che-json     - JSON representation of DICOM data
//   dcm4che-hl7      - HL7 message support
//   dcm4che-audit    - Audit logging (ATNA)

// Gradle dependency:
// implementation 'org.dcm4che:dcm4che-core:5.31.1'
// implementation 'org.dcm4che:dcm4che-net:5.31.1'
// implementation 'org.dcm4che:dcm4che-imageio:5.31.1'

// Features:
// - Complete DICOM standard implementation
// - Full DIMSE service support (C-ECHO, C-STORE, C-FIND, C-MOVE, C-GET)
// - Transfer syntax negotiation and transcoding
// - Image codec support (JPEG, JPEG 2000, JPEG-LS, RLE)
// - HL7 v2 integration
// - WADO / DICOMweb support
// - LDAP-based configuration
// - ATNA audit logging
// - Used as the foundation of the dcm4chee PACS server
```

#### PixelMed
```java
// Java DICOM toolkit focused on education and research
// http://www.pixelmed.com/
// JAR available from: http://www.pixelmed.com/pixelmedjavadicom.html

// Features:
// - DICOM file reading and writing
// - Network protocols (C-ECHO, C-STORE, C-FIND, C-MOVE)
// - DICOM validation and conformance tools
// - Structured Report creation and parsing
// - Anonymization utilities
// - Image display and manipulation
// - Education-focused with extensive Javadoc
// - No external dependencies required
```

#### Weasis
```java
// Open-source DICOM viewer and toolkit
// https://weasis.org/
// Maven: org.weasis:weasis-dicom-tools

// Features:
// - Medical image viewing (2D, MPR, 3D)
// - DICOM network operations
// - Image processing and measurements
// - Plugin architecture for extensibility
// - Cross-platform (Windows, macOS, Linux)
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

### Java: Creating DICOM Files with dcm4che

```java
import org.dcm4che3.data.Attributes;
import org.dcm4che3.data.Tag;
import org.dcm4che3.data.UID;
import org.dcm4che3.data.VR;
import org.dcm4che3.io.DicomOutputStream;
import org.dcm4che3.util.UIDUtils;

import java.io.File;
import java.io.IOException;
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;

public class CreateDicom {
    public static void main(String[] args) {
        // Create File Meta Information
        Attributes fmi = new Attributes();
        String sopInstanceUID = UIDUtils.createUID();
        fmi.setString(Tag.MediaStorageSOPClassUID, VR.UI, UID.CTImageStorage);
        fmi.setString(Tag.MediaStorageSOPInstanceUID, VR.UI, sopInstanceUID);
        fmi.setString(Tag.TransferSyntaxUID, VR.UI, UID.ExplicitVRLittleEndian);
        fmi.setString(Tag.ImplementationClassUID, VR.UI, UIDUtils.createUID());
        fmi.setString(Tag.ImplementationVersionName, VR.SH, "DCM4CHE_TUTORIAL");

        // Create dataset
        Attributes attrs = new Attributes();

        // Patient information
        attrs.setString(Tag.PatientName, VR.PN, "Doe^John");
        attrs.setString(Tag.PatientID, VR.LO, "12345");
        attrs.setString(Tag.PatientBirthDate, VR.DA, "19800101");
        attrs.setString(Tag.PatientSex, VR.CS, "M");

        // Study information
        attrs.setString(Tag.StudyInstanceUID, VR.UI, UIDUtils.createUID());
        attrs.setString(Tag.StudyDate, VR.DA,
            LocalDate.now().format(DateTimeFormatter.ofPattern("yyyyMMdd")));
        attrs.setString(Tag.StudyTime, VR.TM,
            LocalTime.now().format(DateTimeFormatter.ofPattern("HHmmss")));
        attrs.setString(Tag.StudyDescription, VR.LO, "Test Study");
        attrs.setString(Tag.StudyID, VR.SH, "1");
        attrs.setString(Tag.AccessionNumber, VR.SH, "ACC001");

        // Series information
        attrs.setString(Tag.SeriesInstanceUID, VR.UI, UIDUtils.createUID());
        attrs.setInt(Tag.SeriesNumber, VR.IS, 1);
        attrs.setString(Tag.Modality, VR.CS, "CT");

        // SOP information
        attrs.setString(Tag.SOPClassUID, VR.UI, UID.CTImageStorage);
        attrs.setString(Tag.SOPInstanceUID, VR.UI, sopInstanceUID);
        attrs.setInt(Tag.InstanceNumber, VR.IS, 1);

        // Image pixel attributes
        attrs.setInt(Tag.SamplesPerPixel, VR.US, 1);
        attrs.setString(Tag.PhotometricInterpretation, VR.CS, "MONOCHROME2");
        attrs.setInt(Tag.Rows, VR.US, 256);
        attrs.setInt(Tag.Columns, VR.US, 256);
        attrs.setInt(Tag.BitsAllocated, VR.US, 16);
        attrs.setInt(Tag.BitsStored, VR.US, 12);
        attrs.setInt(Tag.HighBit, VR.US, 11);
        attrs.setInt(Tag.PixelRepresentation, VR.US, 0);

        // Create sample pixel data (256x256, 16-bit)
        byte[] pixelData = new byte[256 * 256 * 2];
        for (int i = 0; i < pixelData.length; i += 2) {
            int value = (int) (Math.random() * 4096);
            pixelData[i] = (byte) (value & 0xFF);
            pixelData[i + 1] = (byte) ((value >> 8) & 0xFF);
        }
        attrs.setBytes(Tag.PixelData, VR.OW, pixelData);

        // Write DICOM file
        File outputFile = new File("output.dcm");
        try (DicomOutputStream dos = new DicomOutputStream(outputFile)) {
            dos.writeDataset(fmi, attrs);
            System.out.println("DICOM file created: " + outputFile.getAbsolutePath());
        } catch (IOException e) {
            System.err.println("Error creating DICOM file: " + e.getMessage());
        }
    }
}
```

### Java: DICOM Network SCU (C-STORE) with dcm4che

```java
import org.dcm4che3.data.Attributes;
import org.dcm4che3.data.Tag;
import org.dcm4che3.data.UID;
import org.dcm4che3.io.DicomInputStream;
import org.dcm4che3.net.*;
import org.dcm4che3.net.pdu.AAssociateRQ;
import org.dcm4che3.net.pdu.PresentationContext;

import java.io.File;
import java.io.IOException;
import java.security.GeneralSecurityException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;

public class CStoreSCU {
    public static void main(String[] args) {
        if (args.length < 1) {
            System.err.println("Usage: CStoreSCU <dicom-file>");
            return;
        }

        String callingAET = "MY_SCU";
        String calledAET = "PACS_SCP";
        String host = "192.168.1.100";
        int port = 11112;

        ExecutorService executor = Executors.newSingleThreadExecutor();
        ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();

        try {
            // Create device and connection
            Device device = new Device(callingAET);
            Connection conn = new Connection();
            device.addConnection(conn);

            // Create Application Entity
            ApplicationEntity ae = new ApplicationEntity(callingAET);
            device.addApplicationEntity(ae);
            ae.addConnection(conn);

            // Configure executor
            device.setExecutor(executor);
            device.setScheduledExecutor(scheduler);

            // Read DICOM file to get metadata for association negotiation
            File dicomFile = new File(args[0]);
            Attributes fmi;
            Attributes data;
            try (DicomInputStream dis = new DicomInputStream(dicomFile)) {
                fmi = dis.readFileMetaInformation();
                data = dis.readDataset();
            }

            String sopClassUID = fmi.getString(Tag.MediaStorageSOPClassUID);
            String sopInstanceUID = fmi.getString(Tag.MediaStorageSOPInstanceUID);
            String transferSyntax = fmi.getString(Tag.TransferSyntaxUID);

            // Create association request
            AAssociateRQ rq = new AAssociateRQ();
            rq.setCalledAET(calledAET);
            rq.addPresentationContext(
                new PresentationContext(1, sopClassUID, transferSyntax));

            // Connect and send
            Connection remoteConn = new Connection(null, host, port);
            Association assoc = ae.connect(remoteConn, rq);

            // Send C-STORE using the dataset already read
            DimseRSPHandler rspHandler = new DimseRSPHandler(assoc.nextMessageID()) {
                @Override
                public void onDimseRSP(Association as, Attributes cmd, Attributes rspData) {
                    int status = cmd.getInt(Tag.Status, -1);
                    System.out.println("C-STORE Response Status: 0x"
                        + Integer.toHexString(status));
                }
            };

            DataWriterAdapter dataWriter = new DataWriterAdapter(data);
            assoc.cstore(sopClassUID, sopInstanceUID,
                dataWriter, transferSyntax, rspHandler);

            // Release association
            assoc.release();
            System.out.println("C-STORE completed successfully");

        } catch (IOException | InterruptedException | IncompatibleConnectionException
                 | GeneralSecurityException e) {
            System.err.println("C-STORE failed: " + e.getMessage());
        } finally {
            executor.shutdown();
            scheduler.shutdown();
        }
    }
}
```

### Java: DICOM Network SCP (Storage Server) with dcm4che

```java
import org.dcm4che3.data.Attributes;
import org.dcm4che3.data.Tag;
import org.dcm4che3.data.VR;
import org.dcm4che3.io.DicomOutputStream;
import org.dcm4che3.net.*;
import org.dcm4che3.net.pdu.PresentationContext;
import org.dcm4che3.net.service.BasicCEchoSCP;
import org.dcm4che3.net.service.BasicCStoreSCP;
import org.dcm4che3.net.service.DicomServiceRegistry;

import java.io.File;
import java.io.IOException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;

public class CStoreSCP {
    private static final String AE_TITLE = "MY_SCP";
    private static final int PORT = 11112;
    private static final String STORAGE_DIR = "./received_dicom";

    private static final BasicCStoreSCP cstoreSCP = new BasicCStoreSCP("*") {
        @Override
        protected void store(Association as, PresentationContext pc,
                             Attributes rq, PDVInputStream data, Attributes rsp)
                throws IOException {
            // Get SOP Instance UID for filename
            String sopInstanceUID = rq.getString(Tag.AffectedSOPInstanceUID);
            String sopClassUID = rq.getString(Tag.AffectedSOPClassUID);

            // Read the incoming DICOM data
            Attributes attrs = data.readDataset(pc.getTransferSyntax());

            // Create storage directory if needed
            File storageDir = new File(STORAGE_DIR);
            storageDir.mkdirs();

            // Create File Meta Information
            Attributes fmi = as.createFileMetaInformation(sopInstanceUID,
                sopClassUID, pc.getTransferSyntax());

            // Save to disk
            File outputFile = new File(storageDir, sopInstanceUID + ".dcm");
            try (DicomOutputStream dos = new DicomOutputStream(outputFile)) {
                dos.writeDataset(fmi, attrs);
            }

            System.out.println("Stored: " + outputFile.getName()
                + " from " + as.getCallingAET());
        }
    };

    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();

        // Create device
        Device device = new Device(AE_TITLE);
        device.setExecutor(executor);
        device.setScheduledExecutor(scheduler);

        // Create connection
        Connection conn = new Connection();
        conn.setPort(PORT);
        conn.setReceivePDULength(Connection.DEF_MAX_PDU_LENGTH);
        device.addConnection(conn);

        // Create Application Entity
        ApplicationEntity ae = new ApplicationEntity(AE_TITLE);
        ae.setAssociationAcceptor(true);
        ae.addConnection(conn);
        device.addApplicationEntity(ae);

        // Register DICOM services
        DicomServiceRegistry serviceRegistry = new DicomServiceRegistry();
        serviceRegistry.addDicomService(new BasicCEchoSCP());   // C-ECHO support
        serviceRegistry.addDicomService(cstoreSCP);              // C-STORE support
        ae.setDimseRQHandler(serviceRegistry);

        // Accept all transfer syntaxes for all SOP classes
        ae.addTransferCapability(new TransferCapability(null, "*",
            TransferCapability.Role.SCP,
            "*"));

        try {
            device.bindConnections();
            System.out.println("DICOM SCP started on port " + PORT);
            System.out.println("AE Title: " + AE_TITLE);
            System.out.println("Waiting for incoming associations...");
        } catch (Exception e) {
            System.err.println("Failed to start SCP: " + e.getMessage());
            executor.shutdown();
            scheduler.shutdown();
        }
    }
}
```

### Java: DICOM Query (C-FIND) with dcm4che

```java
import org.dcm4che3.data.Attributes;
import org.dcm4che3.data.Tag;
import org.dcm4che3.data.UID;
import org.dcm4che3.data.VR;
import org.dcm4che3.net.*;
import org.dcm4che3.net.pdu.AAssociateRQ;
import org.dcm4che3.net.pdu.PresentationContext;

import java.io.IOException;
import java.security.GeneralSecurityException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;

public class CFindSCU {
    public static void main(String[] args) {
        String callingAET = "MY_FIND_SCU";
        String calledAET = "PACS_SCP";
        String host = "192.168.1.100";
        int port = 11112;

        ExecutorService executor = Executors.newSingleThreadExecutor();
        ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();

        try {
            // Create device and Application Entity
            Device device = new Device(callingAET);
            Connection conn = new Connection();
            device.addConnection(conn);

            ApplicationEntity ae = new ApplicationEntity(callingAET);
            device.addApplicationEntity(ae);
            ae.addConnection(conn);

            device.setExecutor(executor);
            device.setScheduledExecutor(scheduler);

            // Create association request with Patient Root Q/R Find
            AAssociateRQ rq = new AAssociateRQ();
            rq.setCalledAET(calledAET);
            rq.addPresentationContext(new PresentationContext(1,
                UID.PatientRootQueryRetrieveInformationModelFind,
                UID.ExplicitVRLittleEndian));

            // Build query dataset
            Attributes queryKeys = new Attributes();
            queryKeys.setString(Tag.QueryRetrieveLevel, VR.CS, "STUDY");
            queryKeys.setString(Tag.PatientName, VR.PN, "Smith*");
            queryKeys.setString(Tag.PatientID, VR.LO, "");
            queryKeys.setString(Tag.StudyDate, VR.DA, "20240101-20240131");
            queryKeys.setString(Tag.StudyDescription, VR.LO, "");
            queryKeys.setString(Tag.StudyInstanceUID, VR.UI, "");
            queryKeys.setString(Tag.AccessionNumber, VR.SH, "");
            queryKeys.setString(Tag.ModalitiesInStudy, VR.CS, "");

            // Connect
            Connection remoteConn = new Connection(null, host, port);
            Association assoc = ae.connect(remoteConn, rq);

            // Send C-FIND
            DimseRSPHandler rspHandler = new DimseRSPHandler(assoc.nextMessageID()) {
                @Override
                public void onDimseRSP(Association as, Attributes cmd, Attributes data) {
                    int status = cmd.getInt(Tag.Status, -1);

                    if (status == Status.Pending || status == Status.PendingWarning) {
                        // Process matching result
                        System.out.println("\n--- Match Found ---");
                        System.out.println("Patient: " + data.getString(Tag.PatientName));
                        System.out.println("Patient ID: " + data.getString(Tag.PatientID));
                        System.out.println("Study Date: " + data.getString(Tag.StudyDate));
                        System.out.println("Description: " + data.getString(Tag.StudyDescription));
                        System.out.println("Study UID: " + data.getString(Tag.StudyInstanceUID));
                    } else if (status == Status.Success) {
                        System.out.println("\nC-FIND completed.");
                    }
                }
            };

            assoc.cfind(UID.PatientRootQueryRetrieveInformationModelFind,
                0, queryKeys, null, rspHandler);

            // Release association
            assoc.release();

        } catch (IOException | InterruptedException | IncompatibleConnectionException
                 | GeneralSecurityException e) {
            System.err.println("C-FIND failed: " + e.getMessage());
        } finally {
            executor.shutdown();
            scheduler.shutdown();
        }
    }
}
```

### Java: DICOM Anonymization with dcm4che

```java
import org.dcm4che3.data.Attributes;
import org.dcm4che3.data.Tag;
import org.dcm4che3.data.VR;
import org.dcm4che3.io.DicomInputStream;
import org.dcm4che3.io.DicomOutputStream;
import org.dcm4che3.util.UIDUtils;

import java.io.File;
import java.io.IOException;

public class AnonymizeDicom {
    public static void main(String[] args) {
        if (args.length < 2) {
            System.err.println("Usage: AnonymizeDicom <input.dcm> <output.dcm>");
            return;
        }

        File inputFile = new File(args[0]);
        File outputFile = new File(args[1]);

        try (DicomInputStream dis = new DicomInputStream(inputFile)) {
            Attributes fmi = dis.readFileMetaInformation();
            Attributes attrs = dis.readDataset();

            // Remove/replace patient identifying information
            attrs.setString(Tag.PatientName, VR.PN, "ANONYMOUS");
            attrs.setString(Tag.PatientID, VR.LO, "ANON_" + System.currentTimeMillis());
            attrs.setNull(Tag.PatientBirthDate, VR.DA);
            attrs.setNull(Tag.PatientAddress, VR.LO);
            attrs.setNull(Tag.PatientTelephoneNumbers, VR.SH);

            // Remove institution and physician info
            attrs.setNull(Tag.InstitutionName, VR.LO);
            attrs.setNull(Tag.InstitutionAddress, VR.ST);
            attrs.setNull(Tag.ReferringPhysicianName, VR.PN);
            attrs.setNull(Tag.PerformingPhysicianName, VR.PN);
            attrs.setNull(Tag.OperatorsName, VR.PN);

            // Generate new UIDs to break linkage to original data
            attrs.setString(Tag.StudyInstanceUID, VR.UI, UIDUtils.createUID());
            attrs.setString(Tag.SeriesInstanceUID, VR.UI, UIDUtils.createUID());
            String newSOPInstanceUID = UIDUtils.createUID();
            attrs.setString(Tag.SOPInstanceUID, VR.UI, newSOPInstanceUID);

            // Update file meta information with new SOP Instance UID
            fmi.setString(Tag.MediaStorageSOPInstanceUID, VR.UI, newSOPInstanceUID);

            // Remove private tags (odd group numbers may contain PHI)
            attrs.removePrivateAttributes();

            // Write anonymized file
            try (DicomOutputStream dos = new DicomOutputStream(outputFile)) {
                dos.writeDataset(fmi, attrs);
            }

            System.out.println("Anonymized DICOM file saved: " + outputFile.getAbsolutePath());

        } catch (IOException e) {
            System.err.println("Anonymization failed: " + e.getMessage());
        }
    }
}
```

### Java: Maven Project Setup for DICOM Development

```xml
<!-- pom.xml - Maven project configuration for dcm4che-based DICOM development -->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>dicom-tutorial</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <dcm4che.version>5.31.1</dcm4che.version>
    </properties>

    <repositories>
        <repository>
            <id>dcm4che-repo</id>
            <url>https://www.dcm4che.org/maven2</url>
        </repository>
    </repositories>

    <dependencies>
        <!-- Core DICOM data model and I/O -->
        <dependency>
            <groupId>org.dcm4che</groupId>
            <artifactId>dcm4che-core</artifactId>
            <version>${dcm4che.version}</version>
        </dependency>
        <!-- DICOM networking (associations, DIMSE services) -->
        <dependency>
            <groupId>org.dcm4che</groupId>
            <artifactId>dcm4che-net</artifactId>
            <version>${dcm4che.version}</version>
        </dependency>
        <!-- Image I/O and codec support -->
        <dependency>
            <groupId>org.dcm4che</groupId>
            <artifactId>dcm4che-imageio</artifactId>
            <version>${dcm4che.version}</version>
        </dependency>
        <!-- JSON representation of DICOM data -->
        <dependency>
            <groupId>org.dcm4che</groupId>
            <artifactId>dcm4che-json</artifactId>
            <version>${dcm4che.version}</version>
        </dependency>
    </dependencies>
</project>
```

```groovy
// build.gradle - Gradle project configuration for dcm4che-based DICOM development
plugins {
    id 'java'
    id 'application'
}

java {
    sourceCompatibility = JavaVersion.VERSION_11
    targetCompatibility = JavaVersion.VERSION_11
}

repositories {
    mavenCentral()
    maven { url 'https://www.dcm4che.org/maven2' }
}

def dcm4cheVersion = '5.31.1'

dependencies {
    implementation "org.dcm4che:dcm4che-core:${dcm4cheVersion}"
    implementation "org.dcm4che:dcm4che-net:${dcm4cheVersion}"
    implementation "org.dcm4che:dcm4che-imageio:${dcm4cheVersion}"
    implementation "org.dcm4che:dcm4che-json:${dcm4cheVersion}"
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

### 11. Java Resource Management

**Problem**: Not properly closing DICOM streams and network connections, leading to resource leaks

**Solution**: Use try-with-resources and ensure proper cleanup
```java
// Always use try-with-resources for DicomInputStream
try (DicomInputStream dis = new DicomInputStream(new File("image.dcm"))) {
    Attributes attrs = dis.readDataset();
    // Process attributes...
} // Stream is automatically closed

// Always shut down executors after network operations
ExecutorService executor = Executors.newSingleThreadExecutor();
ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();
try {
    // ... perform DICOM operations ...
} finally {
    executor.shutdown();
    scheduler.shutdown();
}

// Release associations in a finally block
Association assoc = null;
try {
    assoc = ae.connect(remoteConn, rq);
    // ... send DIMSE messages ...
} finally {
    if (assoc != null && assoc.isReadyForDataTransfer()) {
        assoc.release();
    }
}
```

### 12. Java Thread Safety with dcm4che

**Problem**: Sharing DICOM objects across threads without proper synchronization

**Solution**: Create thread-local copies or synchronize access
```java
// Attributes objects are NOT thread-safe
// Create a copy for each thread if sharing is needed
Attributes original = dis.readDataset();
Attributes threadCopy = new Attributes(original); // Deep copy

// For concurrent SCP implementations, each association handler
// runs in its own thread - avoid sharing mutable state between handlers
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

**Document Version**: 1.0  
**Last Updated**: 2024  
**License**: This document is provided for educational purposes.
