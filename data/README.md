# Data

The data behind https://weertman.github.io/fhl-research-profile/, by Claude Opus 5.5 and Willem Lee Weertman. CSV files have a header row; values are as the page reads them.

The census was built by AI agents (Claude Opus 5.5 in Claude Code) searching bibliographic databases, repositories, archives and FHL's own lists, screening every candidate against a written guide, and having separate AI agents audit a random sample against the publications themselves; the page's technical methods section gives the details, the prompts and the measured error rates. Records found only through the University of Washington's Web of Science subscription are included as bibliographic facts.

- `census_all_candidates.csv`: the full census, 15,962 candidate records with the decision on each (status, evidence tier, where FHL appears, the evidence snippet, the basis and reason for the decision, confidence and notes). Phone numbers quoted in evidence snippets are removed.

- `ai_workflows.csv`: The AI runs that built the census.
- `artist_programs.csv`: The station's artist-in-residence programs, with years and sources.
- `artists.csv`: Artists and writers in residence, with years, program, medium and sources.
- `audit_tiers.csv`: The independent audit, by evidence tier.
- `by_appears.csv`: Publications by where FHL appears.
- `by_era.csv`: Publications under each of the station's names.
- `by_type.csv`: Publications by type.
- `by_year.csv`: Publications per year.
- `by_year_appears.csv`: Publications per year by where FHL appears.
- `by_year_tier.csv`: Publications per year by evidence tier.
- `candidate_status.csv`: What happened to every candidate record.
- `classes.csv`: Classes taught at the station by year and term, with instructors, TAs and sources (public records, fact-checked).
- `coauthor_places.csv`: Institutions whose researchers co-authored verified FHL publications, with locations and counts by decade (OpenAlex).
- `eras.csv`: The station's names and their years.
- `headline.csv`: Headline totals: verified publications, best estimate and the date the data were retrieved.
- `history_eras.csv`: Chronicle of the station: the eras under the station's successive names.
- `history_events.csv`: Chronicle of the station: dated, sourced events from 1903 to today.
- `history_images.csv`: Chronicle of the station: the public-domain and openly licensed images, with captions, credits, licences and sources.
- `lineage.csv`: Teacher-student links: UW theses, Wikidata, lab rosters and mentors inferred from co-authorship, with the source of each.
- `lineage_people.csv`: Everyone in the mentoring network, with the year their career began and how it was estimated.
- `methods_facts.csv`: Counts quoted in the technical methods.
- `network_split.csv`: Publications naming FHL per year, split into those with an FHL scientist among the authors and those by others, beside the FHL scientists' total output (OpenAlex counts).
- `news_by_year.csv`: News items per year.
- `organisms_by_decade.csv`: Organism groups by decade (AI classification).
- `people.csv`: FHL scientists (at least three FHL-address publications, or a documented role at the station in its early decades), where they came from and where they went (OpenAlex), their publications by kind, and death years from public sources.
- `people_links.csv`: Citations among the 60 most prolific FHL scientists.
- `places.csv`: Institutions FHL scientists came from and went to, with locations.
- `profiles.csv`: A two-paragraph, fact-checked account of each FHL scientist and network teacher.
- `prompts.csv`: Every prompt typed for this report, with when it was typed and the output tokens it led to.
- `publications.csv`: The census: one row per publication (record id, evidence tier, year, authors, title, venue, type, where FHL appears, citations, DOI, research topic, organism group).
- `recall_by_decade.csv`: How many of FHL's own listed papers the database searches found, by decade.
- `staff.csv`: Directors and staff through time: roles, categories, years and sources (public records, fact-checked).
- `top_cited.csv`: The most cited publications.
- `top_venues.csv`: The commonest journals and venues.
- `topics_by_decade.csv`: Research topics by decade (AI classification).
- `whiteley_by_field.csv`: Whiteley Center publications by field.
- `world_land.json`: World land outline for the map (Natural Earth, public domain), as GeoJSON features.
