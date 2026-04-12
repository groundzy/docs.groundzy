# Complete tree service list (master taxonomy)

A canonical list of arboriculture and tree-care services, grouped by domain. Use this as a **taxonomy** when modeling services in Groundzy—not as free-form labels only.

---

## 1. Planting and establishment

- Tree planting
- Transplanting
- Replacement planting
- Staking / guying
- Soil preparation
- Root ball inspection
- Mulching (initial)
- Watering plan setup
- Species selection consultation
- Site suitability assessment

## 2. Watering and irrigation

- Manual watering
- Deep root watering
- Irrigation system installation
- Irrigation system maintenance
- Drought stress mitigation
- Seasonal watering adjustment
- Moisture monitoring

## 3. Pruning and trimming

- Structural pruning
- Crown cleaning (deadwood removal)
- Crown thinning
- Crown raising
- Crown reduction
- Directional pruning
- Clearance pruning (buildings, roads, utilities)
- Vista pruning
- Pollarding
- Espalier pruning

## 4. Risk reduction / safety

- Hazard assessment
- Deadwood removal
- Cabling
- Bracing
- Lightning protection installation
- Risk mitigation pruning
- Emergency stabilization
- Storm prep pruning

## 5. Removal and stump work

- Tree removal
- Sectional dismantling
- Crane removal
- Emergency removal
- Stump grinding
- Stump removal (full excavation)
- Root removal
- Debris hauling
- Site cleanup

## 6. Pest and disease management

- Pest inspection
- Disease diagnosis
- Integrated Pest Management (IPM)
- Insecticide treatment
- Fungicide treatment
- Biological controls
- Preventative spraying
- Soil drench treatments
- Trunk injections

## 7. Soil and root care

- Soil testing
- Soil amendment
- Fertilization
- Compost application
- Aeration (air spade)
- Vertical mulching
- Root collar excavation (RCX)
- Decompaction
- Biochar application
- Mycorrhizal inoculation

## 8. Storm and emergency services

- Storm damage assessment
- Emergency limb removal
- Fallen tree removal
- Hanging limb removal
- Insurance documentation
- Post-storm cleanup

## 9. Inspection and assessment

- Tree health assessment
- Risk assessment (TRAQ style)
- Pre-construction assessment
- Annual inspection
- Inventory tagging
- Monitoring plan
- Decay detection (resistograph, sonic)
- Root inspection

## 10. Construction and protection

- Tree protection planning
- Tree protection zone (TPZ) setup
- Construction monitoring
- Root pruning (for construction)
- Soil compaction mitigation
- Barrier installation
- Compliance reporting

## 11. Growth management and enhancement

- Growth regulator application (PGR)
- Canopy training
- Structural correction (young trees)
- Fruit production enhancement
- Flowering optimization

## 12. Specialty / orchard / utility

- Fruit tree pruning
- Harvest assistance
- Orchard management
- Utility line clearance
- Right-of-way clearing
- Forestry thinning
- Brush clearing

## 13. Maintenance and recurring care

- Scheduled pruning cycles
- Seasonal inspections
- Mulch refresh
- Fertilization programs
- Health monitoring
- Tree care plans (multi-year)

## 14. Documentation and reporting

- Tree reports (PDF/export)
- Inventory reports
- Risk reports
- Compliance reports
- Photo documentation
- Before/after documentation
- GIS tagging / mapping updates

## 15. Consulting and advisory

- Arborist consultation
- Tree valuation (appraisal)
- Preservation planning
- Species recommendations
- Landscape integration
- Legal / insurance consulting

---

## Groundzy-specific insight

The product already hints at this structure:

- **Trees** have history records (service, inspection, measurement, note).
- **Zones** support `zone_services` (bulk services).
- **CRM** supports workflow: request → quote → job → invoice.

### Implication

This list is **not** just labels. In the data model it should become:

| Concept | Role |
|--------|------|
| **ServiceType** | Taxonomy (what kind of work this is). |
| **WorkItem** | Execution instance (a specific piece of work on a tree or site). |
| **ServiceTemplate** | Reusable bundles (common job packages). |
| **RecurringPlan** | Maintenance cycles (scheduled repeat work). |

### Suggested core fields (keep enums small)

- **category** — e.g. Pruning, Removal, Health, etc. (maps to the sections above at a coarse level).
- **type** — specific service from this taxonomy.
- **priority** — low → emergency.
- **frequency** — one-time / recurring.

### Service as a lifecycle object

A “service” is **not** only a label. Treat it as a **stateful lifecycle**:

`planned` → `scheduled` → `in-progress` → `completed` → `verified`

(Adjust names to match your workflow enums.)

### Missing but critical concept

You do not only have **services** — you have **work on a tree over time**, which closes a loop:

- **Inspections** → trigger services  
- **Services** → update health (and records)  
- **Weather** → trigger recommendations  

That feedback loop is the core product story: ongoing care and risk management, not one-off tasks in isolation.
