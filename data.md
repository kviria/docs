---
title: Data and FAQ
description: Coverage, common questions, and map data attribution.
---

## FAQ

### What area does Kviria cover?

Kviria covers the **contiguous United States** (latitude `24.4` to `49.5`, longitude `-125.0` to `-66.8`). Queries outside coverage return `decision: "none"` with `out_of_coverage` in `evidence` and count toward your daily turns.

### Is Kviria free?

Yes. Kviria is in free beta. Every account receives `600` turns per day (reset at midnight UTC) and `60` turns per minute, with no payment option. See [Rate limits and errors](/api/limits).

### Does Kviria store conversations?

No. Kviria keeps no conversation text on its servers between turns. The signed `state` string in each response carries the turn context, and your agent sends it back on the next call. See the [Privacy Policy](https://kviria.com/privacy).

### Do I need to parse the user's message into fields myself?

No. Send `request: "ask"` with the user's message in `words`, and Kviria resolves the area, category, street, address, or follow-up choice automatically. If your agent already extracts structured slots, you can send structured requests (`search`, `find`, `near`, `across`, `next_to`, `corner`, `street_end`, `resolve`, `pick`, `confirm`, `more`) to the same endpoint.

---

## Map data attribution

Kviria's map data is built from [Overture Maps Foundation](https://overturemaps.org) releases and upstream open datasets.

**© OpenStreetMap contributors, Overture Maps Foundation**

- Streets, intersections, and buildings are derived in whole or in part from [OpenStreetMap](https://www.openstreetmap.org/copyright) data licensed under the [Open Database License (ODbL 1.0)](https://opendatacommons.org/licenses/odbl/1-0/).
- Places are derived from sources under [CDLA Permissive 2.0](https://cdla.dev/permissive-2-0/), [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0), and [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
- Addresses are derived from the U.S. Department of Transportation's National Address Database and state and county datasets distributed by OpenAddresses.
- Full upstream source details are listed on [Overture's attribution page](https://docs.overturemaps.org/attribution/).

### Foursquare OS Places notice

> © 2026 Foursquare Labs, Inc. All rights reserved. The Foursquare OS Places dataset (the "Data") is licensed under the Apache License, Version 2.0 (the "License"). You may not use, modify, or distribute the Data except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0. Unless required by applicable law or agreed to in writing, the Data distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

Changes: Overture transformed the Foursquare data to its schema; Kviria indexes the places and addresses for spatial lookup.
