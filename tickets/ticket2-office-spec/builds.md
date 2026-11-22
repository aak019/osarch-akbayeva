# Four office builds

## 1. Accountant - 511,340 KZT

**Load.** Six to seven hours a day in 1C:Accounting 8, Excel, a PDF viewer, and a browser. At month-end, two workbooks of up to 50,000 rows are open alongside 1C.

**Bottleneck.** Screen space during reconciliation. On the 1,920 × 1,080 screen, an even split gives Excel and the statement about 960 pixels of width each before toolbars. A wide worksheet needs horizontal scrolling or the full screen.

**Trade-off.**  Core i5-13400 with UHD 730, an H610 DDR4 board, 16 GB in two modules, and a 1 TB NVMe drive. The H610 board and 650 W power supply save 41,133 KZT against the B760 and 750 W pair. I chose the 67,500 KZT iiyama for its 150 mm height adjustment during long sessions. The designers QHD screen would give each window about 1,280 pixels of width, but the extra 19,490 KZT would put the four builds over budget. The accountant can use split view for short comparisons and give wide sheets the full screen. Both RAM slots are occupied. [iiyama specifications](https://iiyama.com/gl_en/products/prolite-xb2493hsu-b1/), [Monitor prices](evidence/vendor-quotes/README.md)

## 2. Sales - 728,963 KZT

**Load.** Up to four hours of Teams calls per day, with CRM, Excel, email, and 30–40 browser tabs open during screen sharing.

**Bottleneck.** Memory during calls. For sizing, I allowed 4–5 GB for Windows and background apps, 6–9 GB for the browser and CRM, and 2–4 GB for Teams and Excel. That gives a planning range of 12–18 GB. At the higher end, a 16 GB machine may page to the SSD during screen sharing, slowing tab and workbook switches. I would check actual memory use and paging during a typical call with the CRM and workbook open.

**Trade-off.** i5-13400 and MSI B760 DDR4 board. The 32 GB kit costs 241,800 KZT, about a third of the sales workstation, and adds 167,610 KZT compared with the quoted 16 GB kit. It leaves room for the estimated peak and for additional tabs during customer calls. I used the UHD 730 for the Full HD display and calls, which left room for the memory, webcam, and USB headset in the budget. [Memory prices](evidence/vendor-quotes/README.md)

## 3. Web developer - 712,243 KZT

**Load.** Six to seven hours in VS Code, two browsers, and the terminal. A peak session runs six containers, PostgreSQL, a local build, and an 8 GB VM.

**Bottleneck.** Memory under concurrent load. The VM reserves 8 GB, and the containers, database, editor, and browsers share what remains. Swap can delay builds and database queries. The Samsung 990 PRO also gives the database and build cache fast local storage.

**Trade-off.** 3 Ryzen 5 7600, its Wraith Stealth cooler, two matching Kingston DDR5 modules, and the 990 PRO. I considered the ASUS PRIME B650M-A II, which raised a BIOS warning in PCPartPicker. The quoted ASRock B650 works with the selected CPU and costs 49,490 KZT. It has two DIMM slots and two storage M.2 slots. Upgrading to 64 GB requires replacing both DIMMs. The integrated graphics drive the monitor and Ubuntu Desktop 26.04 LTS runs the development stack with no OS licence fee. The distributions component licences permit use.
[AMD Ryzen 5 7600](https://www.amd.com/en/products/processors/desktops/ryzen/7000-series/amd-ryzen-5-7600.html), [Canonical licensing](https://ubuntu.com/legal/intellectual-property-policy), [Docker Engine support](https://docs.docker.com/engine/install/ubuntu/)

## 4. Designer - 1,039,563 KZT

**Load.** Five to six hours in Photoshop or Premiere, with two or three layered PSDs and a 4K project. A heavy timeline uses up to three tracks with colour correction.

**Bottleneck.** GPU and VRAM during effects and 4K preview. The RTX 5060 provides 8 GB of dedicated VRAM, while 32 GB of system memory holds the applications and working files. A 27-inch QHD IPS screen provides space for the canvas and timeline.

**Trade-off.** i5-13400 and MSI B760 board. I chose the 8 GB RTX 5060 over a 16 GB GPU to keep the QHD screen and 32 GB of RAM. The display is for web work in sRGB. Heavy timelines may need proxies, and print colour approval requires a separate proof.
[AOC Q27B36X](https://www.aoc.com/uk/monitors/q27b36x), [Adobe Premiere requirements](https://helpx.adobe.com/premiere/desktop/get-started/technical-requirements/adobe-premiere-pro-technical-requirements.html)

## Compatibility

| Role | CPU, board, and memory | SSD connection | Estimated / rated power |
|---|---|---|---:|
| Accountant | LGA1700 Core i5-13400, Gigabyte H610M H V3 DDR4 rev. 1.0, 2 × 8 GB DDR4 | NV3 M.2 2280 NVMe at PCIe 3.0 ×4 | 242 / 650 W |
| Sales | LGA1700 Core i5-13400, MSI PRO B760M-A WIFI DDR4, 2 × 16 GB DDR4 | NV3 M.2 2280 NVMe at PCIe 4.0 ×4 | 257 / 750 W |
| Web developer | AM5 Ryzen 5 7600, ASRock B650M-H/M.2+, 2 × 16 GB DDR5 | 990 PRO M.2 2280 NVMe in the PCIe 4.0 ×4 Hyper M.2 slot | 159 / 650 W |
| Designer | LGA1700 Core i5-13400, MSI PRO B760M-A WIFI DDR4, 2 × 16 GB DDR4 | NV3 M.2 2280 NVMe at PCIe 4.0 ×4 | 402 / 750 W |

- The Intel CPU is sold as a tray part, so each Intel build includes a separately priced LGA1700 cooler. The 152.7 mm AG400 fits the case's 165 mm limit. The boxed Ryzen includes an AM5 Wraith Stealth cooler. Start DDR5 at its default settings. The Kingston module's 5600 MT/s rating uses XMP. [Intel i5-13400](https://www.intel.com/content/www/us/en/products/sku/230495/intel-core-i513400-processor-20m-cache-up-to-4-60-ghz/specifications.html), [DeepCool cooler](https://www.deepcool.com/products/Cooling/cpuaircoolers/2023/17682.shtml), [Kingston DDR5 datasheet](https://www.kingston.com/datasheets/KF556C40BB-16.pdf)
- The H610 board supports DDR4-3200 and the i5-13400. The MSI B760 board pairs with DDR4 and has four slots. The Crucial DDR4 module was out of stock at the checked retailers, so the Intel builds use available dual-channel kits. The ASRock B650 board supports AM5, DDR5 UDIMMs, and two NVMe drives. The Kingston NV3 runs at the H610 M.2 slot's PCIe 3.0 speed. The Samsung SATA drive from was replaced with NVMe storage on the Intel desks to use their M.2 slots and simplify cabling. [Gigabyte H610](https://www.gigabyte.com/au/Motherboard/H610M-H-V3-DDR4-rev-10/sp), [MSI B760](https://www.msi.com/Motherboard/PRO-B760M-A-WIFI-DDR4/Specification), [ASRock B650](https://www.asrock.com/mb/AMD/B650M-HM.2%2B/), [Kingston NV3](https://www.kingston.com/en/ssd/nv3-nvme-pcie-ssd?capacity=1tb)
- The CC360 accepts all four Micro-ATX boards and both 140 mm ATX power supplies. The A750BN PCIE5 provides the MSI board's 8-pin and additional 4-pin CPU power connections and the designer GPU's 8-pin PCIe connection. The 182 mm GPU fits within the case's 378 mm limit using its standard bracket. Gigabyte specifies a 550 W minimum system PSU. The RM750e was replaced with stocked units. [CC360 specifications](https://www.deepcool.com/products/Cases/CC360-ARGB-M-ATX-Airflow-Case/2023/16791.shtml), [MSI 650 W](https://www.msi.com/Power-Supply/MAG-A650BN/Specification), [MSI 750 W](https://www.msi.com/Power-Supply/MAG-A750BN-PCIE5/Specification), [Gigabyte RTX 5060](https://www.gigabyte.com/Graphics-Card/GV-N5060OC-8GL/sp)
- The case includes three fans. Connect the front pair through the FS-04 splitter and the rear fan to a separate chassis header. Select DC control for three-pin fans. The H610 and ASRock B650 boards have no 5 V ARGB header for the case lights. Connect the accountant, sales, and developer monitors to motherboard HDMI and the designer's monitor to the graphics card. [Case wiring guide](https://www.deepcool.com/download/pdf/CC360_ARGB.pdf)
