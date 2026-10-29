# Research publications of the Friday Harbor Laboratories

By Claude Opus 5.5, Willem Lee Weertman and Marjorie Wonham.

A census of the work published from Friday Harbor Laboratories (FHL), the University of Washington's marine
station on San Juan Island, since it opened in 1904. It includes work published under the station's earlier names: the Puget
Sound Marine Station, the Puget Sound Biological Station and the University of Washington Oceanographic Laboratories.
The page covers when FHL work was published, what it was about, the people (where they came from, who trained whom, who cites
whom), where FHL appears in each publication, citations, news coverage, and how reliable the census is. Its technical methods
section describes how the census was built, including the use of AI.

**The page:** https://weertman.github.io/fhl-research-profile/

## What is here

- `index.html`: the page. It loads d3 from cdnjs and reads its data from `data/`.
- `data/`: the data behind every figure and table, as CSV (and GeoJSON for the map outline). See `data/README.md`.

## Updating

The page and data are built by the census pipeline (not in this repository): `scripts/build_site.py` writes this folder.
Commit and push the changes, and GitHub Pages republishes the site within a minute or two.

## Sources

Bibliographic records come from OpenAlex, Crossref, PubMed and Europe PMC, Web of Science (University of Washington
subscription), DataCite and other data repositories, UW ResearchWorks, the Internet Archive and FHL's own publication lists.
Career histories and citation links come from OpenAlex (CC0). Advisor links come from UW thesis records, Wikidata (CC0),
public lab pages and, where marked as inferred, co-authorship patterns. The map outline is Natural Earth (public domain).
Counts are a lower bound; see the methods section of the page for the caveats.
