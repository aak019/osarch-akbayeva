# Ticket 2 — Kitting out the new office

## 1. What was asked

From Daniyar K. (Director)  
Subject: computers for the new office

> We are opening the second office and need four computers — accountant, sales, web developer, designer. Budget 3,000,000 tenge for everything. Everyone says we should just buy the same four machines, is that right? Make sure they last a few years. Monitors included.

## 2. What was actually needed

Four identical machines would misallocate the budget. A dedicated GPU would add cost to the accountant's and sales desks, while assigning 16 GB to everyone would leave less room for the developer's VM and the designer's video projects.

| What I would ask Daniyar | Assumption used in the quote |
|---|---|
| Which software, versions, and sign-in requirements apply? | Accounting uses 1C:Accounting 8 for Kazakhstan, Excel, and a Windows-compatible signing token. Sales uses a browser CRM, Excel, and Teams. The designer uses Photoshop and Premiere. These three PCs use Windows 11 Home, assuming no domain-join requirement. The developer uses Ubuntu, VS Code, Docker Engine, and PostgreSQL. |
| What does each role run at the same time? | Accounting compares reports and spreadsheets. Sales keeps up to 40 tabs open during calls. Development runs up to six containers, a database, and an 8 GB VM. Design works with layered PSDs and up to three 4K video tracks for web delivery. |
| What does “for everything” cover, and where is the office? | The quote covers four systems, four monitors, OS licences, keyboard and mouse sets, network cables, and a webcam and headset for sales. I assumed existing application subscriptions, wired network outlets, shared backup storage, and in-house installation. Delivery depends on the office address. |
| How long should these PCs last? | Four years, with the first upgrade reviewed after 18 months. The developer's AM5/DDR5 platform allows a compatible CPU or RAM upgrade. Sales and design retain the four-slot  MSI board. The accountant's two-slot DDR4 board is the lowest-cost platform in this plan. |

[1C:Accounting for Kazakhstan](https://solutions.1c.ru/catalog/buhv8-kz) · [Windows editions](https://www.microsoft.com/en-us/windows/compare-windows-11-home-vs-pro-versions) · [Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

## 3. What I did

I sized each PC for its expected peak workload in [builds.md](builds.md), matched the parts in PCPartPicker, and checked connectors and dimensions against manufacturer specifications. The [budget](budget.csv) uses online prices captured on 2026-11-22 from sellers serving Kazakhstan. The four builds leave **7,891 KZT** before delivery. I would confirm the destination and delivery charges before ordering because any charges above that amount would exceed the cap.

| Role | Custom build (KZT) | Prebuilt PC (KZT) | Prebuilt workstation (KZT) |
|---|---:|---:|---:|
| Accountant | 511,340 | 559,990 | 643,920 |
| Sales | 728,963 | 799,990 | 900,610 |
| Web developer | 712,243 | 731,390 | 793,420 |
| Designer | 1,039,563 | 899,190 | 1,079,100 |
| **Total** | **2,992,109** | **2,990,560** | **3,417,050** |

**First upgrade.** In May 2028, upgrade the developer from 32 GB to a matched 2 × 32 GB DDR5 kit. The VM, containers, and browser can fill memory during a local build. I would first run the usual project with all three active and use `free -h` and `vmstat 1` to confirm memory pressure and swap activity

**Prebuilt option.** The [prebuilt budget](prebuilt.csv) adds licences and peripherals where needed. The Lenovo IdeaCentre for sales includes a display, webcam, keyboard, and mouse, so I added Windows 11 Home, a headset, and an Ethernet cable. Buying through Shop.kz saves assembly time and gives the company one shop to contact for warranty service. The all-in-ones limit display replacement and internal upgrades, and the towers come with the seller's choice of components. The developer's tower also spends part of the budget on an RTX 3050, although the workload mainly needs CPU time, memory, and storage. The four ready-made workstations cost **424,941 KZT more** and exceed the cap by **417,050 KZT** before delivery

**First performance complaint.** The designer is likely to see choppy playback on a timeline with three 4K tracks and several colour effects. The RTX 5060 has 8 GB of VRAM
## 4. Evidence

- [PCPartPicker lists](evidence/pcpartpicker-links.md) show the exact core hardware, compatibility results, and estimated power for the four builds.
- [Vendor screenshots](evidence/vendor-quotes/README.md) show the products and prices used in both budgets. Each image includes the source URL and capture time.
- [Budget check](evidence/budget-check.txt) records the recalculated subtotals and grand totals.

## 5. What I would do differently

Before buying the designer's GPU, I would ask for one typical Premiere project with three 4K tracks, play it at full resolution, and record dropped frames and peak VRAM on an 8 GB card. If typical projects regularly exceed 8 GB, I would reprice a 16 GB card before ordering.
