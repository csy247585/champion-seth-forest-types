# Champion & Seth 1968 forest types of India — interactive browser

**Version 1.0** — 2026-09-24

## Authors

- **Mayank Goyal**, Department of Computer Science and Engineering, Indian Institute of Technology Delhi. ⟨csy247585@iitd.ac.in⟩
- **Aaditeshwar Seth**, Department of Computer Science and Engineering, Indian Institute of Technology Delhi. ⟨aseth@cse.iitd.ac.in⟩

## Abstract

This is an interactive browser for the floristic information in H. G. Champion and S. K. Seth's *A Revised Survey of the Forest Types of India* (Government of India, 1968), joined to site coordinates and to modern accepted plant names. It covers **464 sites** across India (latitude 6.9° N to 34.3° N, longitude 68.7° E to 96.0° E), describing which species occur at which site, in which layer of the forest, at what abundance, and whether the surveyors marked them as particularly characteristic of the forest type. The tool is intended for anyone working on Indian vegetation classification, potential natural vegetation modelling, restoration planning, or floristic biogeography who wants the 1968 survey in a form that can be queried, filtered and mapped.

## Data sources

The browser is derived from two primary sources.

1. **Champion, H. G. and Seth, S. K. (1968).** *A Revised Survey of the Forest Types of India.* Manager of Publications, Government of India, Delhi. The forest-type classification (16 groups, sub-groups, individual types with their successional codes), the site-level floristic lists, the species-group notation (I–V, Epi, Par), the abundance codes (va, a, la, c, lc, f, o, r) and the asterisk marking characteristic species all come from this volume.

2. **Species-points geospatial layer.** A GeoJSON file that pairs species records from the 1968 volume with site coordinates, terrestrial ecoregion (Ecoregions 2017 / RESOLVE, Dinerstein *et al.* 2017), a Champion & Seth species name and an accepted name from WCVP. *This layer was produced by ⟨name(s) and affiliation⟩; see the acknowledgements below.*<!-- FILL THIS IN. -->

Accepted names follow **World Checklist of Vascular Plants (WCVP)**, Royal Botanic Gardens, Kew (Govaerts *et al.* 2021, ongoing; https://powo.science.kew.org).

Terrestrial ecoregion polygons: **Dinerstein, E. et al. (2017).** An Ecoregion-Based Approach to Protecting Half the Terrestrial Realm. *BioScience* 67: 534–545. Distributed as the RESOLVE Ecoregions 2017 layer.


## Structure of the browser

The window is a three-column layout.

**Filter panel** on the left, with the filters below. Every heading carries a small *i* button that expands a short definition drawn from the source volume.

- **Species name** — Champion & Seth or WCVP; both are searched.
- **Bioclimate class** — the six major groups: Moist Tropical, Dry Tropical, Montane Subtropical, Montane Temperate, Sub-Alpine, Alpine Scrub.
- **Group** — one of the 16 forest-type Groups. Selecting a bioclimate class narrows this list to the Groups within it.
- **Successional level** — Climatic climax (C), Edaphic (E, L, TS, FS, SS), Primary seral (1S), Secondary seral (2S), Riparian (RS), Degradation stage (DS).
- **Ecoregion** — Ecoregions 2017 (RESOLVE) terrestrial ecoregion.
- **Species group** — the layer of the forest: top canopy, second storey, bamboos, shrubs, herbs, grasses, climbers, epiphytes, parasites.
- **Abundance** — very abundant, abundant, locally abundant, common, locally common, frequent, occasional, rare.
- **Characteristic species only** — restricts to species the surveyors starred.

**Site list** in the middle, showing every site that matches the filters. Sortable by class, species count, or A–Z. Selecting a site opens its detail view in place of the list: the site's classification chain, its note, every species recorded there arranged by species group and split into abundance columns (very abundant, abundant, locally abundant, common, locally common, frequent, occasional, rare, abundance not stated, characteristic), and a Champion & Seth ↔ WCVP name table. Starred species appear both in their abundance column and in the characteristic column, since the two are independent categories.

**Map** on the right, plotting the filtered sites as circle markers coloured by bioclimate class. Clicking a marker shows the full record for that site. Clicking a site in the middle column narrows the map to that one point; clicking "← All sites" returns to the filtered set.

A Tab sit next to Sites:

- **Species** — every species in the currently filtered sites, ordered by how often it is recorded. Each entry shows the WCVP name, the main species group, the number of sites and types it occurs in. Clicking a name opens the list of sites and types for that species.

## Methodology

The browser is generated from the source GeoJSON in three steps.

**Site definition.** A site is one floristic entry: a locality under a single forest type. Features in the source GeoJSON are grouped by the tuple (longitude, latitude, Distribution, Group, Type Level 1, Type Level 2, Type Level 3). Grouping on coordinates alone would merge sites that share a location but describe different forest types (three such pairs occur in the source); the composite key preserves them. This yields the 464 sites.

**Species-record extraction.** The source layer stores each site's species composition as one free-text field written in the notation of the 1968 volume, for example:

> `I. Shorea robusta,* Anogeissus latifolia,* Terminalia tomentosa (a). II. Cassia fistula, Ziziphus xylopyrus. IIa. Dendrocalamus strictus (a). IVb. Eulaliopsis binata, Heteropogon contortus.`

The text is split at species-group markers (I, II, IIa, III, IVa, IVb, V, Epi, Par); slash-separated markers such as `I/II.` apply to both. Each block is split on commas and semicolons; each token is read as one species. Parenthesised codes are converted from the volume's abbreviations to their full names (`va` → very abundant, and so on). An asterisk on a name is recorded as a separate *characteristic* flag, kept independent of abundance because in the volume the two mean different things (typicality of the type versus quantity at the site). Abbreviated genera such as `C. wightianum` are expanded from the preceding full name in the same block, following the volume's convention that the initial refers to the previous genus. Filler tokens (`etc`, `spp`, `and others`) are dropped, and within one site–stratum a species is listed only once even if the source text repeats it.

**Derived classification fields.** Four fields are computed from the classification strings carried by the source GeoJSON so that filters and views can share a single definition:

- *Bioclimate class* — short name (Moist Tropical, Dry Tropical, ...) inferred from the source *Bioclimate Class* string.
- *Group number and name* — parsed from the source *Group* string (`GROUP 5—TROPICAL DRY DECIDUOUS FORESTS` → `5`, `Tropical Dry Deciduous Forests`).
- *Successional level and code* — read from the letter at the head of *Type Level 1*: Climatic climax (C), Edaphic (E, L, TS, FS, SS), Primary seral (1S), Secondary seral (2S), Riparian (RS), Degradation stage (DS).
- *WCVP accepted name* on each record — taken from the source *WCVP_name* field for the same Champion & Seth name.

The parsed and enriched site list is embedded inside the HTML file as a JavaScript constant. All interaction — filters, list, map, species index, identifier — reads from that embedded array. There is no server, no database and no runtime data fetching: the whole page is a deterministic function of the embedded data, which is why the single file is enough to run everywhere.

## Intended uses

- Selecting reference sites and species lists for one Champion & Seth type when parameterising a vegetation classification, potential natural vegetation model, or restoration prescription.
- Reconciling old floras and working plans that quote the 1968 nomenclature against modern accepted names.
- Teaching Indian forest ecology, and giving students a browsable view of what codes like `5B/C1a` or `3C/DS1` actually contain.

## Known limitations

- **Locality names** are as described in the source geojson. The coordinates are the point given for the locality in the source geojson, not a plot or polygon boundary.
- **Coverage of the abundance scale.** The 1968 volume records abundance codes for only a subset of species (about 24 % of records); the remainder are printed as bare names and appear in the browser as *abundance not stated*. Absence of an abundance code is *not* evidence that the species was rare or infrequent.
- **WCVP crosswalk gaps.** About 7 % of records have no WCVP accepted name in the source layer, either because the Champion & Seth name matched no WCVP record or because reconciliation was not attempted. These retain their Champion & Seth name and are searchable by it.


## References

- Champion, H. G. and Seth, S. K. (1968). *A Revised Survey of the Forest Types of India.* Manager of Publications, Government of India, Delhi.
- Govaerts, R. and 39 co-authors (2021). The World Checklist of Vascular Plants, a continuously updated resource for exploring global plant diversity. *Scientific Data* 8: 215.
- Dinerstein, E., Olson, D., Joshi, A. and 45 others (2017). An Ecoregion-Based Approach to Protecting Half the Terrestrial Realm. *BioScience* 67: 534–545.

## Acknowledgements

## Licence
