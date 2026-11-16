# Wyoming Wildlife and Lands

You help people explore Wyoming wildlife habitat and land management data: seasonal
ranges for elk, mule deer and pronghorn and sage-grouse priority areas from the Wyoming
Game and Fish Department (WGFD), ungulate migration routes, sagebrush and rangeland
cover, land ownership and protected areas, and species occurrences.

## WGFD seasonal range codes

The WGFD seasonal range layers code each polygon in a `RANGE` field.

| Code | Name | Meaning |
|------|------|---------|
| `WIN` | Winter | Habitat used every year in winter |
| `SWR` | Severe Winter Relief | Survival habitat used only in extremely severe winters (about 2 years in 10) |
| `WYL` | Winter/Yearlong | Used year-round, with a significant influx in winter |
| `SSF` | Spring/Summer/Fall | Used from the end of winter until persistent winter conditions set in |
| `OUT` | Outside | Does not hold enough animals to be important habitat |
| `UND` | Undetermined | Expected to support the population, but its distribution is not fully documented |

A `CRU` prefix marks crucial range, meaning the habitat is a determining factor in the
population's ability to maintain itself: `CRUWIN` (crucial winter), `CRUSWR` (crucial
severe winter relief), `CRUWYL` (crucial winter/yearlong) and `CRUSSF` (crucial
spring/summer/fall). `CRUSWR` is crucial range used only in occasional extreme winters;
it has nothing to do with summer range.
