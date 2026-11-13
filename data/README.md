# Data

The data behind https://weertman.github.io/fhl-research-profile/, by Claude Opus 5.5, Willem Lee Weertman and Marjorie Wonham. CSV files have a header row; values are as the page reads them.

The census was built by AI agents (Claude Opus 5.5 in Claude Code) searching bibliographic databases, repositories, archives and FHL's own lists, with written rules deciding the clear cases, AI agents judging the ambiguous ones against a written guide (`docs/screening_guide.md`), and separate AI agents auditing a random sample against the publications themselves; the page's technical methods section gives the details, the prompts and the measured error rates. Records found only through the University of Washington's Web of Science subscription are included as bibliographic facts.

- `census_all_candidates.csv`: the full census, 15,971 candidate records with the decision on each (status, evidence tier, where FHL appears, the evidence snippet, the basis and reason for the decision, confidence and notes). Phone numbers, email addresses, and private street and mailing addresses quoted in evidence snippets and reasons are removed (institutional addresses such as FHL's own are kept); a few directory and member-list snippets are replaced whole.
- `FHL_publications_census.xlsx`: the census workbook, 14 sheets (README, Publications, News media, Mention only, Unverified, Excluded, Archive mentions, By year, By category, Topics by decade, Audit, Coverage, Sources, Columns), with contact details removed in the same way. The `Columns` sheet defines every column and the `Sources` sheet lists the sources searched.

- `agent_stages.csv`: Each stage of each AI run: its agents, what they were given, what they did and what they handed on.
- `agents.csv`: One row per AI agent of the runs: run, stage, role, status, start, minutes and tool calls by kind (no ids or text).
- `ai_workflows.csv`: The AI runs that built the census, with their family, shape, prompt, the session's effort setting and the effort the agents logged, and logged times.
- `art_images.csv`: Photographs, published by FHL, of artists at work in its Macfarlane Art Studio, used with FHL's permission and not openly licensed: artist, year, caption, alt text, credit and license. The finished works are not reproduced.
- `artist_programs.csv`: The station's artist-in-residence programs, with years and sources.
- `artists.csv`: Artists and writers in residence, with years, program, medium and sources.
- `audit_by_stratum.csv`: The independent audit by how each record was decided: the stratum's size when the sample was drawn, the records audited, how many had a verdict on the FHL connection, and the share that agreed with the census (95% interval).
- `audit_tiers.csv`: The independent audit, by evidence tier.
- `beyond_spending.csv`: Counts of teaching and training that the spending estimate leaves out, each with its source and limits.
- `by_appears.csv`: Publications by where FHL appears.
- `by_era.csv`: Publications under each of the station's names.
- `by_type.csv`: Publications by type.
- `by_year.csv`: Publications per year.
- `by_year_appears.csv`: Publications per year by where FHL appears.
- `candidate_status.csv`: What happened to every candidate record.
- `census_steps.csv`: The steps of the publication census, the runs (or run stages) and who did each, and the records each handled.
- `classes.csv`: Classes taught at the station by year and term, with instructors, TAs and sources (public records, fact-checked).
- `coauthor_places.csv`: Institutions whose researchers co-authored verified FHL publications, with locations and counts by decade (OpenAlex).
- `course_topics.csv`: Course titles by decade and topic (AI classification of the course list).
- `discoveries.csv`: The discoveries and scientific contributions in section X: what was found, why it matters and FHL's part, by field.
- `discovery_refs.csv`: Key publications and sources for each discovery, with links.
- `econ_award_scopes.csv`: NSF and NIH awards named by census papers, at narrower and narrower scopes, as awarded (Fig. 22).
- `econ_capital_timeline.csv`: Buildings, utilities, housing, land and instruments: year, funder, basis of the value, amount as stated and in 2024 dollars, and source (Fig. 23).
- `econ_facts.csv`: Every number quoted in the text of the economic contribution (section X), with its rounded display form, dollar year, scope and source.
- `econ_funders_by_decade.csv`: Share of census papers matched to OpenAlex that name each funder, by decade (Fig. 22).
- `econ_gfp_chain.csv`: The steps from aequorin and GFP at FHL in 1961 to the 2008 Nobel Prize, with sources (Table IV).
- `econ_gfp_pubmed.csv`: PubMed records a year under the subject headings Green Fluorescent Proteins and Aequorin (Fig. 19).
- `econ_model_by_source.csv`: Economic contribution by spending source (operations, research, students, visitors) and effect (direct, indirect, induced), for each region and basis (Fig. 20).
- `econ_model_inputs.csv`: The main inputs of the economic model, with low, central and high values and their basis (Table X); the full list is in economics/model_inputs.csv.
- `econ_model_sensitivity.csv`: How much each input moves the economic estimate when it goes from its low to its high value (the two household-spending inputs over the range used in the simulations), and the alternative model structures (Fig. 35).
- `econ_model_totals.csv`: Economic contribution (section X, Table V): yearly totals for Washington and San Juan County by basis (contribution, new money) and measure, with construction 2015–2024 and memo lines; central value, 10th, 50th and 90th percentiles and extreme bounds, in 2024 dollars. Estimated from public sources.
- `econ_money_sources.csv`: Grants for work at FHL by source, as awarded and in 2024 dollars, with the rows not counted (Table VI); n counts what n_unit names (NIH rows count projects, with their project-years in n_project_years).
- `econ_research_by_fy.csv`: Federal awards for work at FHL spread over their award periods, by UW fiscal year, as awarded and in 2024 dollars (Fig. 21).
- `eras.csv`: The station's names and their years.
- `headline.csv`: Headline totals: publications by evidence tier, citations, yearly averages for 2016–2025, screening outcomes, search recall against FHL's lists, the audited precision of all included records taken together (audit_precision; each tier's precision is in audit_tiers.csv), the read rates the page uses for its best estimate, and the date the data were retrieved.
- `history_eras.csv`: Chronicle of the station: the eras under the station's successive names.
- `history_events.csv`: Chronicle of the station: dated, sourced events from 1903 to 2026.
- `history_images.csv`: Chronicle of the station: the images shown, with captions, credits, licenses and sources. Most are public domain or openly licensed; those from FHL's own archives are used with FHL's permission and are not free to reuse (see the license column).
- `lineage.csv`: Teacher-student links: lab rosters and CVs, graduate committees, course pages, other public records, advisors named in the fact-checked profiles, UW theses, Wikidata, and mentors inferred from co-authorship (confirmed or not), with the source of each.
- `lineage_people.csv`: Everyone in the mentoring network, with the year their career began and how it was estimated.
- `methods_checks.csv`: Rounds of checking agents' work: items made, re-opened, confirmed, corrected (changed to the checker's verdict), dropped and unsettled.
- `methods_facts.csv`: Counts quoted in the technical methods.
- `network_split.csv`: Publications naming FHL per year, split into those with an FHL core scientist among the authors and those by others, beside the FHL core scientists' total output (OpenAlex counts). Columns ending in _research count research papers only (journal articles, reviews and letters; for the FHL core scientists' total, OpenAlex's article, review and letter types), as on the page.
- `news_by_year.csv`: News items per year.
- `organisms_by_decade.csv`: Organism groups by decade (AI classification).
- `people.csv`: FHL core scientists (at least three FHL-address publications, or a documented role at the station in its early decades), where they came from and where they went (OpenAlex), their publications by kind, and death years from public sources.
- `people_links.csv`: Citations among the 60 most prolific FHL core scientists.
- `places.csv`: Institutions FHL core scientists came from and went to, with locations.
- `profiles.csv`: An entry on each person in the mentoring network: a fact-checked profile or section, or one sentence from their listing (kind), with its public sources.
- `prompts.csv`: The prompts typed from 13 October to 13 November 2026 that built the census and the report (early messages about signing in, saving files and progress summarized in one row; rulings on named people summarized without the names; the feedback form's web-app link withheld), with when each was typed and the output tokens it led to.
- `publications.csv`: The census: one row per publication (record id, evidence tier, year, authors, title, venue, type, where FHL appears, citations, DOI, research topic, organism group, why the record was counted and the passage that names FHL, with contact details removed).
- `recall_by_decade.csv`: How many of FHL's own listed papers the database searches found, by decade.
- `staff.csv`: Directors and staff through time: roles, categories, years and sources (public records, fact-checked).
- `top_cited.csv`: The most cited publications.
- `top_venues.csv`: The commonest journals and venues.
- `topics_by_decade.csv`: Research topics by decade (AI classification).
- `whiteley_by_field.csv`: Whiteley Center publications by field.
- `world_land.json`: World land outline for the map (Natural Earth, public domain), as GeoJSON features.
- `economics/`: the economic model behind the economic contribution in section X (method in `docs/economic_model_method.md`): `model_inputs.csv`, every input with its low, central and high value, unit, dollar year, source and basis (sources of the form model/... name files in the census pipeline that are not published; public sources are given by URL or title); `model_results.csv`, every result by region, basis (contribution or new money), spending source, measure and effect, with the Monte Carlo 10th, 50th and 90th percentiles and the extreme bounds; and `model_sensitivity.csv`, the one-at-a-time swing of each input group. All money is in 2024 dollars.
