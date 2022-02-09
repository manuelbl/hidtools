# Project

This repo is the central location of Microsoft HID Tools.  Currently, the only tool available is Waratah.

# Waratah

> What A Really Awesome Tool for Authoring HIDs

Waratah is a HID descriptor composition tool.  It offers a high-level of abstraction, eliminates common errors (by design), and optimizes the descriptor to reduce byte size.  It implements the [HID 1.11](https://www.usb.org/sites/default/files/hid1_11.pdf) specification so developers don't have to.

It is expected to be used by device firmware authors during device bring-up.

See [Wiki](https://github.com/microsoft/hidtools/wiki) for more details 

## Overview

Waratah uses a [TOML-like](https://toml.io/en/) hierarchical language of sections and keys to represent a HID Report Descriptor  (*Note: There is currently no support for HID Physical Descriptors*).  This can then be compiled to to either a simple plain-text format, or a C++ header file suitable for ingestion into device firmware.

> Waratah is NOT a direct `dt.exe` replacement.  `dt.exe` permits the use of specialized items (e.g. Push/Pop) and non-optimal practices (e.g. ReportSize larger than LogicalMinimum/Maximum).  There are also known bugs in `dt.exe`, that have not been replicated in Waratah.  It is reasonable to think of Waratah as high-level compiler like `C` and `dt.exe` as an assembler.  *No further development of `dt.exe` is planned.*

### Features:
- Human-readable text for easy composition and meaningful source-control management.
- Inbuilt with all public Usages from [HID Usage Table](https://www.usb.org/sites/default/files/hut1_22.pdf)
- Composition of custom Vendor Usages.
- Inbuilt with all defined Units.
- Support for composition of new named Units.
- Comprehensive error messaging with line-level blame.
- C++ struct generation with context-aware variable-name generation.
- Optimistic descriptor optimization (redundant global items removed, and items combined).
- Report summary with itemized Id/Type/Size.
- Context-aware integer types bounds validation (ReportId, UsageId,...).
- Context-aware Usage type validation.
- Automatic generation of ReportIds.
- Automatic 8bit report alignment (padding inserted as needed).
- Automatic collection termination.
- Report Item size inferred from LogicalMin/Max (or vice-versa).
- Overspanning preventation, guaranteeing no item spans more than 4bytes (padding inserted as needed).
- Validate conditionally invalid Report Flags.
- Inline Usage transformation.
- Usage Range validation.

### Example: Simple mouse with 3 buttons.

```toml
[[applicationCollection]]
usage = ['Generic Desktop', 'Mouse']

    [[applicationCollection.inputReport]]
    id = 1

        [[applicationCollection.inputReport.physicalCollection]]
        usage = ['Generic Desktop', 'Pointer']

            [[applicationCollection.inputReport.physicalCollection.variableItem]]
            usage = ['Generic Desktop', 'X']
            logicalValueRange = [-128, 127]
            reportFlags = ['relative']

            [[applicationCollection.inputReport.physicalCollection.variableItem]]
            usage = ['Generic Desktop', 'Y']
            logicalValueRange = [-128, 127]
            reportFlags = ['relative']

            [[applicationCollection.inputReport.physicalCollection.variableItem]]
            usageRange = ['Button', 'Button 1', 'Button 3']
            logicalValueRange = [0, 1]
```

#### Plain-text output
```
05-01....UsagePage(Generic Desktop[1])
09-02....UsageId(Mouse[2])
A1-01....Collection(Application)
85-01........ReportId(1)
09-01........UsageId(Pointer[1])
A1-00........Collection(Physical)
09-30............UsageId(X[48])
09-31............UsageId(Y[49])
15-80............LogicalMinimum(-128)
25-7F............LogicalMaximum(127)
95-02............ReportCount(2)
75-08............ReportSize(8)
81-06............Input(Data, Variable, Relative, NoWrap, Linear, PreferredState, NoNullPosition, BitField)
05-09............UsagePage(Button[9])
19-01............UsageIdMin(Button 1[1])
29-03............UsageIdMax(Button 3[3])
15-00............LogicalMinimum(0)
25-01............LogicalMaximum(1)
95-03............ReportCount(3)
75-01............ReportSize(1)
81-02............Input(Data, Variable, Absolute, NoWrap, Linear, PreferredState, NoNullPosition, BitField)
C0...........EndCollection()
95-01........ReportCount(1)
75-05........ReportSize(5)
81-03........Input(Constant, Variable, Absolute, NoWrap, Linear, PreferredState, NoNullPosition, BitField)
C0.......EndCollection()
```

#### C++ output
```C++
#include <memory>

static const uint8_t hidReportDescriptor [] = 
{
    0x05, 0x01,    // UsagePage(Generic Desktop[1])
    0x09, 0x02,    // UsageId(Mouse[2])
    0xA1, 0x01,    // Collection(Application)
    0x85, 0x01,    //     ReportId(1)
    0x09, 0x01,    //     UsageId(Pointer[1])
    0xA1, 0x00,    //     Collection(Physical)
    0x09, 0x30,    //         UsageId(X[48])
    0x09, 0x31,    //         UsageId(Y[49])
    0x15, 0x80,    //         LogicalMinimum(-128)
    0x25, 0x7F,    //         LogicalMaximum(127)
    0x95, 0x02,    //         ReportCount(2)
    0x75, 0x08,    //         ReportSize(8)
    0x81, 0x06,    //         Input(Data, Variable, Relative, NoWrap, Linear, PreferredState, NoNullPosition, BitField)
    0x05, 0x09,    //         UsagePage(Button[9])
    0x19, 0x01,    //         UsageIdMin(Button 1[1])
    0x29, 0x03,    //         UsageIdMax(Button 3[3])
    0x15, 0x00,    //         LogicalMinimum(0)
    0x25, 0x01,    //         LogicalMaximum(1)
    0x95, 0x03,    //         ReportCount(3)
    0x75, 0x01,    //         ReportSize(1)
    0x81, 0x02,    //         Input(Data, Variable, Absolute, NoWrap, Linear, PreferredState, NoNullPosition, BitField)
    0xC0,          //     EndCollection()
    0x95, 0x01,    //     ReportCount(1)
    0x75, 0x05,    //     ReportSize(5)
    0x81, 0x03,    //     Input(Constant, Variable, Absolute, NoWrap, Linear, PreferredState, NoNullPosition, BitField)
    0xC0,          // EndCollection()
};
```

# Legal

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
