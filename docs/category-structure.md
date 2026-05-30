  ┌──────────────┬───────────────────────────────────────────────────────────┐
  │   Category   │                     What belongs here                     │
  ├──────────────┼───────────────────────────────────────────────────────────┤
  │ Concept      │ Abstract ideas, principles, theories, phenomena — things  │
  │              │ you understand                                            │
  ├──────────────┼───────────────────────────────────────────────────────────┤
  │ Method       │ Frameworks, methodologies, approaches, patterns,          │
  │              │ processes — things you apply                              │
  ├──────────────┼───────────────────────────────────────────────────────────┤
  │ Tool         │ Software, platforms, products, applications               │
  ├──────────────┼───────────────────────────────────────────────────────────┤
  │ Work         │ Books, papers, articles, reports, films as subjects of    │
  │              │ study                                                     │
  ├──────────────┼───────────────────────────────────────────────────────────┤
  │ Person       │ Individuals                                               │
  ├──────────────┼───────────────────────────────────────────────────────────┤
  │ Organization │ Companies, institutions, firms, teams                     │
  ├──────────────┼───────────────────────────────────────────────────────────┤
  │ Place        │ Locations, markets, regions                               │
  ├──────────────┼───────────────────────────────────────────────────────────┤
  │ Event        │ Occurrences, milestones, developments, launches           │
  ├──────────────┼───────────────────────────────────────────────────────────┤
  │ Reference    │ Lookup/navigational material                              │
  └──────────────┴───────────────────────────────────────────────────────────┘


  ---

  ## Tagging and Categorization

  Every page in `vault/pages/` gets one **category** and a small set of
  **tags**.
  These answer different questions and must not duplicate each other.

  - **Category** answers: *what kind of thing is this?*
  - **Tags** answer: *what is it about, and what does it connect to?*

  ### Category

  Assign exactly one. Add it to the page frontmatter as `category: Value`.

  | Value | Use when the page is about... |
  |---|---|
  | Concept | An abstract idea, principle, theory, or phenomenon |
  | Method | A framework, methodology, approach, pattern, or process |
  | Tool | A specific software product, platform, or application |
  | Work | A book, paper, article, or other intellectual work as a subject |
  | Person | An individual |
  | Organization | A company, institution, or team |
  | Place | A location, market, or region |
  | Reference | Lookup or navigational material |
  
  ### Tags
  
  Use 3–6 tags per page drawn from two dimensions:
  
  - **Domain** — the field or area this belongs to
    (e.g. `pkm`, `cognitive-science`, `ai-strategy`, `geopolitics`,
  `product-management`)
  - **Association** — tools, platforms, or organizations this page connects to
    (e.g. `claude-code`, `obsidian`, `anthropic`)
    
  Three rules:
  1. Never use a tag to restate the category — no `tool`, `method`, `reference`,
   etc. as tags
  2. For **Tool** pages, tags describe what kind of tool (`memory`, `search`,
  `saas`, `open-source`);
     for all other pages, tags name associated tools (`claude-code`, `obsidian`)
  3. Use wikilinks — not tags — for people and organizations that have or should
   have their own page

  ---
  You'd also want to update the page template's frontmatter to add the category:
   field — probably just below title::

  ---
  title: [Title]
  type: page
  category: [Category]
  sources: []
  updated: YYYY-MM-DD
  tags: []
  ---
  
  The table is the load-bearing part — it gives the LLM a lookup reference that
  removes judgment calls at assignment time. The three rules at the bottom
  handle the edge cases that would otherwise produce inconsistent tags as the
  vault grows.