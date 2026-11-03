# Research publications of the Friday Harbor Laboratories

By Claude Opus 5.5, Willem Lee Weertman and Marjorie Wonham.

A census of the work published from Friday Harbor Laboratories (FHL), the University of Washington's marine
station on San Juan Island, since it opened in 1904. It includes work published under the station's earlier names: the Puget
Sound Marine Station, the Puget Sound Biological Station and the University of Washington Oceanographic Laboratories.
The page covers when FHL work was published, what it was about, the people (where they came from, who trained whom, who cites
whom), where FHL appears in each publication, citations, news coverage, the station's economic contribution
(estimated from public sources), and how reliable the census is. Its technical methods
section describes how the census was built, including the use of AI. A second tab, the chronicle of the station, tells the
station's history in dated events and images, with its directors and staff, classes and artists in residence.

**The page:** https://weertman.github.io/fhl-research-profile/

## What is here

- `index.html`: the page. It loads d3 from cdnjs and reads its data from `data/`.
- `data/`: the data behind every figure and table, as CSV (and GeoJSON for the map outline), the full census of every
  candidate record (`census_all_candidates.csv`) and the census workbook (`FHL_publications_census.xlsx`). See
  `data/README.md`.
- `docs/`: the written guides the AI agents followed: `harvest_spec.md` (the searches), `screening_guide.md` (deciding
  whether a record is FHL work) and `topic_guide.md` (research topics and organism groups).
  `economic_model_method.md` describes the economic model of the page's section VIII.
- `images/`: the chronicle's images (`images/history/`) and photographs of artists at work in residence (`images/art/`),
  each folder with small previews in `thumb/`. Each image's credit, license and source are in `data/history_images.csv`
  and `data/art_images.csv`. Most chronicle images are public domain or openly licensed; CC BY and CC BY-SA images need
  the credit given (and share-alike for CC BY-SA). Images from FHL's own archives and FHL's artist-in-residence pages
  are used with FHL's permission: they are not openly licensed and may not be reused without FHL's permission.

## Updating

The page and data are built by the census pipeline (not in this repository): `scripts/build_site.py` writes this folder.
Commit and push the changes, and GitHub Pages republishes the site within a minute or two.

## Sources

Bibliographic records come from OpenAlex, Crossref, PubMed and Europe PMC, Web of Science (University of Washington
subscription), DataCite and other data repositories, UW ResearchWorks, the Internet Archive and FHL's own publication lists.
Career histories and citation links come from OpenAlex (CC0). Advisor links come from public lab pages and CVs, graduate
committee lists, the fact-checked profiles, UW thesis records, Wikidata (CC0) and, where marked as inferred, co-authorship
patterns. The map outline is Natural Earth (public domain). The chronicle draws on historic newspapers (Chronicling America,
Library of Congress), University of Washington Libraries Special Collections, FHL's newsletters and web pages (Internet
Archive copies), news items, Wikimedia Commons and other public image collections, and the FHL Historical Centennial
Timeline compiled by Claudia Mills and Colin Hermans (used for facts only); every event, staff post, class, artist and image lists its source in `data/`.
Counts are a lower bound; see the methods section of the page for the caveats.
