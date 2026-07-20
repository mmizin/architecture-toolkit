 I've read all 20 files (9 references, 5 skills, 2 agents, plugin.json, README) plus the git history. No changes made. Here's the review.                                                                                         
                                                                                                                                                                                                                                   
  ---                                                                                                                                                                                                                              
  Plugin Architecture Review                                                                                                                                                                                                       
                                                                                                                                                                                                                                   
  Overall Verdict                                                                                                                                                                                                                  
                                                                                                                                                                                                                                   
  Healthy with improvements.                                                                                                                                                                                                       
                                                                                                                                                                                                                                   
  The boundary design is genuinely well-executed and unusually disciplined for a v0.1. Every one of the seven modules carries an explicit Responsible for / Not responsible for / Escalates to block, terminology is centralized   
  rather than redefined per-module, and all 8 shared-reference paths resolve — there is not a single broken link. The Artifact ownership convention in terminology.md:33 is the conceptual spine of the plugin and it holds.       
                                                                                                                                                                                                                                   
  It is not healthy-as-is because of one critical documentation defect (the README documents a module that has never existed) and a small cluster of real ownership seams — mostly where architecture-bootstrap was carved out of  
  the flow, and where a reference file quietly re-litigates a boundary the skill it serves explicitly disclaims. None of these require restructuring; they require targeted repair.                                                
                                                                                                                                                                                                                                   
  ---            
 Executive Summary                                                                                                                                                                                                                
                                                                                                                                                                                                                                   
  The plugin is a set of seven independent modules with a recommended order rather than a runtime orchestrator — a correct and honestly-stated design choice, since Claude Code skills cannot invoke each other. The intended flow 
  (Discovery → ADR/C4/arc42 → Reviewer → Librarian, with Bootstrap as the entry planner) is logically sound and matches the responsibility split you described.                                                                    
                                                                                                                                                                                                                                   
  The dominant problem class is documentation drift, not architecture. README.md has not been touched since the initial commit (a0a7023), while all seven modules have since received boundary-cleanup and wording passes. The     
  README therefore describes a skill named architecture-documentation that does not exist in any commit — the module shipped as architecture-bootstrap. The README's module table, directory layout, and workflow section are all  
  wrong in the same way. Your session's own additional-working-directory setting points at the same phantom path, which suggests this stale name is still actively propagating.                                                    
                                                                                                                                                                                                                                   
  The second class is ownership seams around Bootstrap and the reference files. Bootstrap disclaims writing arc42 sections but its own workflow proposes producing arc42 §3 and §4 content under a different filename. Discovery   
  and Bootstrap both produce a "recommended generation order." And c4-guidelines.md carries a second, divergent copy of the C4 review checklist, one item of which instructs c4-expert to do exactly what its scope forbids.       
                                                                                                                                                                                                                                   
  The third is graph hygiene: escalation edges are almost entirely bidirectional with no stated termination rule, and one agent's declared write behavior exceeds its tool grant.                                                  
                                                                                                                                                                                                                                   
  ---                                                                                                                                                                                                                              
  Architecture Map                                                                                                                                                                                                                 
                                                                                                                                                                                                                                   
  Modules (7): 5 skills + 2 agents   
                                                                                                                                                                                                                                     
  ┌────────────┬────────────────────────────────┬────────────────────────────────────────────────────────┐                                                                                                                         
  │   Layer    │             Module             │                  Question it answers                   │                                                                                                                         
  ├────────────┼────────────────────────────────┼────────────────────────────────────────────────────────┤                                                                                                                         
  │ Planning   │ architecture-bootstrap         │ "How do we organize the initial documentation effort?" │                                                                                                                         
  ├────────────┼────────────────────────────────┼────────────────────────────────────────────────────────┤                                                                                                                         
  │ Extraction │ architecture-discovery         │ "What exists?"                                         │                                                                                                                         
  ├────────────┼────────────────────────────────┼────────────────────────────────────────────────────────┤                                                                                                                         
  │ Authoring  │ adr-expert                     │ "Why was this decision made?"                          │                                                                                                                         
  ├────────────┼────────────────────────────────┼────────────────────────────────────────────────────────┤                                                                                                                         
  │ Authoring  │ c4-expert                      │ "How is it structured, visually?"                      │                                                                                                                         
  ├────────────┼────────────────────────────────┼────────────────────────────────────────────────────────┤                                                                                                                         
  │ Authoring  │ arc42-expert                   │ "How is it documented, narratively?"                   │                                                                                                                         
  ├────────────┼────────────────────────────────┼────────────────────────────────────────────────────────┤                                                                                                                         
  │ Governance │ architecture-reviewer (agent)  │ "Is the reasoning good?"                               │                                                                                                                         
  ├────────────┼────────────────────────────────┼────────────────────────────────────────────────────────┤                                                                                                                         
  │ Governance │ architecture-librarian (agent) │ "Are the documents consistent?"                        │                                                                                                                         
  └────────────┴────────────────────────────────┴────────────────────────────────────────────────────────┘                                                                                                                         
                                                                                                                                                                                                                                   
  Shared references (9): three templates (adr-, arc42-, c4-guidelines), three vocabularies (terminology, quality-attributes, decision-drivers), two governance taxonomies (review-checklist, severity-levels), and one meta-doc    
  (plugin-design-principles).                                                                                                                                                                                                      
                                                                                                                                                                                                                                   
  Reference fan-in (how many modules/refs cite each): review-checklist 7, quality-attributes 6, adr-template 6, c4-guidelines 5, arc42-template 4, terminology 3, severity-levels 3, decision-drivers 2, plugin-design-principles  
  0.                                                                                                                                                                                                                               
                                                                                                                                                                                                                                   
  Workflow as designed — matches your diagram and is logically correct. The one structural nuance: Bootstrap is upstream of Discovery in the lifecycle but Discovery also escalates back to Bootstrap, making the entry point      
  reachable from inside the flow.   
   ---                                                                                                                                                                                                                              
  Findings                                                                                                                                                                                                                         
                                                                                                                                                                                                                                   
  ---                                                                                                                                                                                                                              
  ID: F1                                                                                                                                                                                                                           
  Type: Conflict                                                                                                                                                                                                                   
  Severity: Critical                                                                                                                                                                                                               
  Location: README.md:26, README.md:50, README.md:80-81                                                                                                                                                                            
                                                                                                                                                                                                                                   
  Problem: The README documents a skill named architecture-documentation — in the module table ("Orchestrate an end-to-end documentation set"), in the directory layout, and as the recommended greenfield starting point. This    
  skill does not exist. git log --follow confirms skills/architecture-bootstrap/SKILL.md was added under that name in the initial commit a0a7023 and never renamed; architecture-documentation appears nowhere outside README.md.  
  The README has not been modified since that same initial commit, while every module has had 1–3 revision passes since.                                                                                                           
                                                                                                                                                                                                                                   
  Why it matters: The README is the plugin's front door and its only orchestration-level document. A user following it will invoke a skill name that cannot resolve, and will do so at precisely the moment the README tells them  
  to — the greenfield entry point. It also means the README's entire "Recommended workflow" section has never been reconciled against the seven modules as they actually shipped, so it cannot be trusted as a description of      
  current behavior. This stale name is still live in your environment configuration (the additional working directory points at skills/architecture-documentation), so it is propagating rather than decaying.                     
                                                                                                                                                                                                                                   
  Recommended fix: Replace all three occurrences with architecture-bootstrap and re-derive the workflow section from the seven modules' actual Escalates to blocks rather than editing it in place. Then update the session's      
  additional-working-directory setting. Consider adding a CI check that every skill name in README.md matches a directory under skills/.                                                                                           
                        
 ---                                                                                                                                                                                                                              
  ID: F2                                                                                                                                                                                                                           
  Type: Conflict                                                                                                                                                                                                                   
  Severity: High                                                                                                                                                                                                                   
  Location: agents/architecture-librarian.md:10 vs :8-9, :96-97                                                                                                                                                                    
                                                                                                                                                                                                                                   
  Problem: The frontmatter grants tools: Read, Grep, Glob. The description states the agent "only edits index/cross-link files when explicitly asked," and the body closes with "Apply edits only to index/cross-link files, and   
  only when the user explicitly approves." The agent has no Edit or Write tool and therefore cannot perform the write it twice promises.                                                                                           
                                                                                                                                                                                                                                   
  Why it matters: This is the one finding where declared behavior and enforced capability actually diverge, so it will surface as a runtime failure rather than a doc nit. A user who approves an index regeneration gets a        
  tool-unavailable error, and the agent's carefully-scoped "read-mostly" contract — the thing that makes it safe to run against a docs repo — reads as accidental rather than deliberate.                                          
                                                                                                                                                                                                                                   
  Recommended fix: Decide the intent and make both ends agree. Either (a) grant Edit and keep the approval gate as the sole guardrail, or (b) keep the agent strictly read-only and reword both the description and the closing    
  paragraph to "proposes edits as a patch for the user or adr-expert to apply." Option (b) is more consistent with the reviewer agent's Do not edit the artifact. and with the plugin's overall propose-don't-apply posture.       
                
 ---                                                                                                                                                                                                                              
  ID: F3                                                                                                                                                                                                                           
  Type: Conflict                                                                                                                                                                                                                   
  Severity: High                                                                                                                                                                                                                   
  Location: skills/architecture-bootstrap/SKILL.md:56 vs :32                                                                                                                                                                       
                                                                                                                                                                                                                                   
  Problem: Bootstrap's Not responsible for explicitly lists "writing ADRs, C4 diagrams, or arc42 sections itself." Its workflow step 2 then proposes producing ARCHITECTURE.md overview "(context + solution strategy)" — which is 
  arc42 §3 (Context and Scope) and §4 (Solution Strategy) by name, per references/arc42-template.md:13,17. The skill's own description (:6) also promises a "README/ARCHITECTURE overview" as a deliverable.                       
                                                                                                                                                                                                                                   
  Why it matters: This is arc42 content in a non-arc42 file, produced by the one module that disclaims producing arc42 content. It creates an unowned artifact — no module's scope mentions ARCHITECTURE.md — and it sets up       
  exactly the duplication terminology.md:33 warns against: if the system later gets real arc42 docs, §3 and §4 exist in two places with no stated source of truth, and architecture-librarian has no rule telling it which one     
  wins.                                                                                                                                                                                                                            
                                                                                                                                                                                                                                   
  Recommended fix: Either assign ARCHITECTURE.md to arc42-expert as an explicitly-supported "arc42 condensed to one page" output mode (and say so in that skill's scope), or drop it from Bootstrap's plan and have Bootstrap      
  sequence a real arc42 §3/§4 instead. Whichever is chosen, name the owner of ARCHITECTURE.md in terminology.md under Artifact ownership.                                                                                          
            
 ---                                                                                                                                                                                                                              
  ID: F4                                                                                                                                                                                                                           
  Type: Conflict                                                                                                                                                                                                                   
  Severity: High                                                                                                                                                                                                                   
  Location: skills/architecture-discovery/SKILL.md:83 vs skills/architecture-bootstrap/SKILL.md:28,55                                                                                                                              
                                                                                                                                                                                                                                   
  Problem: Discovery's report output ends with "Documentation gaps and a recommended generation order." Bootstrap's sole owned responsibility is "sequencing architecture documentation work" and "proposing the initial           
  documentation plan," and its workflow step 2 is to "Present the intended documentation set and order."                                                                                                                           
                                                                                                                                                                                                                                   
  Why it matters: Bootstrap's entire reason to exist is the generation order. Discovery already emits one. When Bootstrap runs step 1 ("Run architecture-discovery first, or ingest its report"), it receives an ordering and then 
  produces a second, possibly different ordering — with no stated rule for reconciling them. This is the sharpest boundary blur in the plugin, and it makes Bootstrap look like a thin wrapper over a Discovery output rather than 
  a module with its own responsibility.                                                                                                                                                                                            
                                                                                                                                                                                                                                   
  Recommended fix: Split the two cleanly along evidence vs. plan. Discovery should report gaps only — "these artifacts are missing relative to what the discovered architecture warrants" — as an unordered, evidence-grounded     
  inventory. Bootstrap owns turning that inventory into a sequenced, user-confirmed plan. Remove "and a recommended generation order" from architecture-discovery/SKILL.md:83 and from its step 4, and add "ordering the work" to  
  Discovery's Not responsible for.                                                                                                                                                                                                 
                                    
 ---                                                                                                                                                                                                                              
  ID: F5                                                                                                                                                                                                                           
  Type: Gap                                                                                                                                                                                                                        
  Severity: High                                                                                                                                                                                                                   
  Location: skills/arc42-expert/SKILL.md:55,57 and references/arc42-template.md:27,30 vs skills/c4-expert/SKILL.md:53-60                                                                                                           
                                                                                                                                                                                                                                   
  Problem: arc42 §6 (Runtime View) is annotated "→ sequence / C4 dynamic diagrams" and §7 (Deployment View) covers "mapping of building blocks to nodes." c4-expert declares exactly four levels — Context, Container, Component,  
  Code — and neither C4 Dynamic nor C4 Deployment diagrams appear in its scope, its workflow, or references/c4-guidelines.md. Both are standard C4 diagram types.                                                                  
                                                                                                                                                                                                                                   
  Why it matters: Two of the twelve arc42 sections point at artifact types no module owns. arc42-expert disclaims producing structural views and escalates diagram work to c4-expert; c4-expert doesn't claim these two. The result
  is a dead-end handoff: a user filling §6 or §7 gets routed to a module whose stated scope excludes the thing they need, and the plugin silently produces weaker runtime and deployment documentation than its own template asks  
  for.                                                                                                                                                                                                                             
                                                                                                                                                                                                                                   
  Recommended fix: Extend c4-expert to cover the two supplementary C4 diagram types — add them to its scope and to references/c4-guidelines.md with Mermaid examples matching the existing Context/Container ones. This is the     
  smaller change and keeps all diagram authoring in one module. If they're deliberately out of scope for v0.1, say so explicitly in c4-expert's Not responsible for and note in arc42-template.md that §6/§7 diagrams are          
  hand-authored for now.                                                                                                                                                                                                           
                               
---
ID: F6
Type: Possible inconsistency
Severity: Medium
Location: references/c4-guidelines.md:54-59 vs references/review-checklist.md:41-42

Problem: Two C4 review checklists exist. review-checklist.md carries a one-line C4 entry (single zoom level, labeled relationships) that defers to c4-guidelines.md; c4-guidelines.md then carries its own five-bullet ## Review checklist that includes two criteria the shared one omits (element count / readability, and matching the real system). c4-expert:82 routes through the first to reach the second.

Why it matters: review-checklist.md:4-6 explicitly justifies its own existence as preventing "different, undocumented review bars" — and this is a second, undocumented review bar for C4 specifically. The two lists have already diverged by two criteria at v0.1, which is the drift the file was created to prevent. It also makes the indirection in c4-expert's step 4 pointlessly two-hop.

Recommended fix: Pick one home. Move the two extra criteria up into review-checklist.md's C4 entry and delete the ## Review checklist section from c4-guidelines.md, leaving that file purely as conventions + examples. Then simplify c4-expert:82-84 to cite review-checklist.md directly, matching how adr-expert and arc42-expert already do it.

 ---                                                                                                                                                                                                                              
  ID: F7                                                                                                                                                                                                                           
  Type: Conflict                                                                                                                                                                                                                   
  Severity: Medium                                                                                                                                                                                                                 
  Location: references/c4-guidelines.md:59 vs skills/c4-expert/SKILL.md:31,70-72                                                                                                                                                   
                                                                                                                                                                                                                                   
  Problem: The final bullet of c4-guidelines.md's checklist reads "Diagram matches the real system (cross-check with discovery)?" — but c4-expert's scope lists "discovering architecture or system boundaries from source code"   
  under Not responsible for, and its workflow step 2 instructs it to use only provided architecture information, "never inferred from code yourself, that's architecture-discovery's job."                                         
                                                                                                                                                                                                                                   
  Why it matters: A reference file is instructing a skill to perform a check its own scope forbids — and it's the last thing the skill reads before finishing, so it's positioned to override. This is a boundary violation hidden 
  in a shared file rather than in a module, which makes it the kind that survives module-level boundary reviews. It also has no escalation path: c4-expert can't verify against reality, so the honest outcome is a handoff, not a 
  check.                                                                                                                                                                                                                           
                                                                                                                                                                                                                                   
  Recommended fix: Reword to a handoff rather than a check — e.g. "If the diagram's fidelity to the real system is in question, escalate to architecture-discovery; do not verify against source code here." Fold into F6's        
  consolidation.                                                                                                                                                                                                                   
                
---
ID: F8
Type: Possible inconsistency
Severity: Medium
Location: references/terminology.md:28-32 vs agents/architecture-librarian.md:60-61,70-72

Problem: terminology.md defines Architecture drift as divergence between docs and the actual system, "Detected by architecture-discovery re-scanning the repository, not by architecture-librarian, which only compares documentation artifacts against each other." But the librarian's own check list includes "Obsolete decisions — accepted ADRs overtaken by reality but not marked deprecated/superseded," while its closing constraint forbids reading application source code.

Why it matters: "Overtaken by reality" is drift by the glossary's own definition, and the librarian is simultaneously assigned to find it and forbidden from the only evidence that would reveal it. In practice the agent will either quietly overstep into code (violating its stated boundary and its tool intent) or infer obsolescence from doc-to-doc signals alone and report low-confidence findings as Conflict.

Recommended fix: Narrow the librarian's bullet to what doc-to-doc evidence can actually support — e.g. "Obsolete decisions — accepted ADRs contradicted by a later accepted artifact but not marked deprecated/superseded." Route drift-versus-code to architecture-discovery, which the librarian's Escalates to block (:40-41) already correctly provides for.

 ---                                                                                                                                                                                                                              
  ID: F9                                                                                                                                                                                                                           
  Type: Gap                                                                                                                                                                                                                        
  Severity: Medium                                                                                                                                                                                                                 
  Location: references/plugin-design-principles.md (whole file); README.md:37-40                                                                                                                                                   
                                                                                                                                                                                                                                   
  Problem: plugin-design-principles.md is cited by zero modules and zero other references — the only orphan among nine reference files. It is also absent from the README's references list, which names just 3 of the 9. Nothing  
  in the plugin points a reader or a future contributor to it.                                                                                                                                                                     
                                                                                                                                                                                                                                   
  Why it matters: The file defines the Single-Responsibility rule and the exact Responsible for / Not responsible for / Escalates to contract that every module actually follows — it is the plugin's constitution, and it is      
  unreachable. That has a compounding cost: the plugin has no module or documented process that owns its own meta-architecture, so consistency depends entirely on manual audits like this one. Several findings above (F3, F4, F7)
  are precisely the violations this file exists to prevent.                                                                                                                                                                        
                                                                                                                                                                                                                                   
  Recommended fix: Give it a reachable entry point — cite it from README.md under a "Contributing / adding a module" section, and reference it from architecture-bootstrap (the module most likely to be extended). Longer term,   
  consider a plugin-architecture-reviewer or a CI lint that asserts every SKILL.md contains all three boundary headings and that every skill name in the README resolves.                                                          
                
 ---                                                                                                                                                                                                                              
  ID: F10                                                                                                                                                                                                                          
  Type: Recommendation                                                                                                                                                                                                             
  Severity: Medium                                                                                                                                                                                                                 
  Location: All 7 modules' Escalates to blocks                                                                                                                                                                                     
                                                                                                                                                                                                                                   
  Problem: Nearly every escalation edge is bidirectional: discovery↔bootstrap, discovery↔adr, discovery↔c4, discovery↔arc42, discovery↔librarian, discovery↔reviewer, adr↔reviewer, adr↔librarian, reviewer↔librarian. No module   
  states a termination condition or a "don't hand back" rule.                                                                                                                                                                      
                                                                                                                                                                                                                                   
  Why it matters: These are not true circular dependencies — each direction fires on a genuinely different condition, which is correct design. But there is no stated stopping rule, and at least one pair is a plausible live     
  loop: architecture-reviewer escalates to architecture-discovery when "the proposal reflects an incomplete understanding of the existing system," and architecture-discovery escalates to architecture-reviewer when "architecture
  quality needs evaluation." An agent driving a long session can ping-pong between them without a rule saying who terminates.                                                                                                      
                                                                                                                                                                                                                                   
  Recommended fix: Add a short "Escalation etiquette" section to plugin-design-principles.md: escalate at most once per condition; if the module you escalated to hands the same artifact back, stop and surface the ambiguity to  
  the user rather than re-escalating. Cite it from each module's Escalates to block, or at minimum from the two agents.                                                                                                            
              
 ---                                                                                                                                                                                                                              
  ID: F11                                                                                                                                                                                                                          
  Type: Possible inconsistency                                                                                                                                                                                                     
  Severity: Medium                                                                                                                                                                                                                 
  Location: references/severity-levels.md:3-4,50-56                                                                                                                                                                                
                                                                                                                                                                                                                                   
  Problem: The file's opening scopes it to reports produced by the librarian, the reviewer, "or a skill's self-check." Its closing ## Usage section then lists only the two agents. Correspondingly, no skill references           
  severity-levels.md — the three authoring skills cite only review-checklist.md for self-review.                                                                                                                                   
                                                                                                                                                                                                                                   
  Why it matters: A skill self-reviewing under review-checklist.md has a verdict scale (Accept / Accept with changes / Needs rework) but no vocabulary for classifying the individual findings behind that verdict, even though the
  file claims to serve them. The finding taxonomy and the validation principle — the guardrail against over-calling Conflict — are unavailable exactly where an artifact is being drafted.                                         
                                                                                                                                                                                                                                   
  Recommended fix: Make it true in one direction. Either add a third Usage bullet for skills and cite severity-levels.md from the self-review steps of adr-expert:75, c4-expert:82, and arc42-expert:82; or narrow the opening line
  to the two agents. The former is more useful and matches the file's stated intent.                                                                                                                                               
                 
 ---                                                                                                                                                                                                                              
  ID: F12                                                                                                                                                                                                                          
  Type: Gap                                                                                                                                                                                                                        
  Severity: Low                                                                                                                                                                                                                    
  Location: agents/architecture-reviewer.md:37-43                                                                                                                                                                                  
                                                                                                                                                                                                                                   
  Problem: The reviewer reviews "ADRs, C4 diagrams, arc42 sections, or freeform design proposals" (:47) but escalates only to architecture-librarian, adr-expert, and architecture-discovery. There is no edge to c4-expert or     
  arc42-expert.                                                                                                                                                                                                                    
                                                                                                                                                                                                                                   
  Why it matters: Asymmetric with the artifact types it accepts. When a C4 diagram or an arc42 section needs rework, the reviewer's only authoring handoff is adr-expert — the wrong owner. The existing adr-expert bullet ("the   
  decision looks sound but the ADR itself needs to be written or substantially improved") has no counterpart for the other two artifact types it is explicitly asked to review.                                                    
                                                                                                                                                                                                                                   
  Recommended fix: Add two bullets mirroring the adr-expert one — c4-expert when the architecture is sound but the diagram needs rework, arc42-expert when the narrative documentation is the weak part.                           
             
 ---                                                                                                                                                                                                                              
  ID: F13                                                                                                                                                                                                                          
  Type: Gap                                                                                                                                                                                                                        
  Severity: Low                                                                                                                                                                                                                    
  Location: skills/adr-expert/SKILL.md:31-37                                                                                                                                                                                       
                                                                                                                                                                                                                                   
  Problem: adr-expert escalates to discovery, reviewer, and librarian — but not to c4-expert or arc42-expert. The reverse edges exist (arc42-expert:36 → adr; c4-expert:42 → arc42).                                               
                                                                                                                                                                                                                                   
  Why it matters: An accepted ADR frequently implies a structural change that should be reflected in a diagram, and arc42 §9 should cite it. architecture-librarian:58-59 explicitly reports this as a coverage gap ("an accepted  
  ADR with no diagram reflecting it") and recommends c4-expert — so the plugin already knows this handoff is needed, but routes it through an audit pass instead of letting the authoring skill make it directly.                  
                                                                                                                                                                                                                                   
  Recommended fix: Add c4-expert (the decision changes structure that a diagram should show) and arc42-expert (§9 needs updating to cite the new ADR) to adr-expert's Escalates to.                                                
           
 ---                                                                                                                                                                                                                              
  ID: F14                                                                                                                                                                                                                          
  Type: Recommendation                                                                                                                                                                                                             
  Severity: Low                                                                                                                                                                                                                    
  Location: skills/ naming across all 5 skills                                                                                                                                                                                     
                                                                                                                                                                                                                                   
  Problem: Two naming conventions coexist: architecture-bootstrap / architecture-discovery (prefixed) versus adr-expert / c4-expert / arc42-expert (suffixed, unprefixed). Both agents use the architecture- prefix.               
                                                                                                                                                                                                                                   
  Why it matters: Minor now, but this is the axis along which the plugin is planned to grow — the roadmap names DDD, RFC, and threat-modeling modules, each of which could plausibly take either form. Settling the rule now is far
  cheaper than renaming shipped modules later. There's a defensible logic latent in the current split (architecture-* = works across artifact types; *-expert = owns one notation), but it isn't written down anywhere.            
                                                                                                                                                                                                                                   
  Recommended fix: State the rule in plugin-design-principles.md — likely "<notation>-expert for modules owning a single documentation format; architecture-<verb> for cross-cutting modules" — and keep current names, which      
  already conform.                                                                                                                                                                                                                 
               
 ---                                                                                                                                                                                                                              
  ID: F15                                                                                                                                                                                                                          
  Type: Possible inconsistency                                                                                                                                                                                                     
  Severity: Low                                                                                                                                                                                                                    
  Location: skills/adr-expert/SKILL.md (absent line ~14)                                                                                                                                                                           
                                                                                                                                                                                                                                   
  Problem: adr-expert is the only module without a > Status: v0.1 stub marker; the other four skills and both agents carry one. README.md:9-11 says "The skill/agent bodies marked stub define their workflow and conventions."    
                                                                                                                                                                                                                                   
  Why it matters: Trivial in isolation, but ambiguous: it reads as either an oversight or a deliberate signal that adr-expert is production-ready. The README's roadmap lists adr-expert among the modules to be deepened in v0.2, 
  which suggests it is still a stub and the marker was simply missed.                                                                                                                                                              
                                                                                                                                                                                                                                   
  Recommended fix: Add the marker for consistency, or if it is genuinely more mature, mark it explicitly (e.g. > Status: v0.1 — deepened) so the distinction is intentional.                                                       
                 
---
ID: F16
Type: Gap
Severity: Low
Location: README.md:37-40, README.md:57-60

Problem: The README's "References (shared templates)" section and directory layout list only adr-template.md, arc42-template.md, and c4-guidelines.md — omitting terminology.md, review-checklist.md, severity-levels.md, quality-attributes.md, decision-drivers.md, and plugin-design-principles.md.

Why it matters: Six of nine references are invisible from the front door, including the three highest-fan-in files in the plugin (review-checklist at 7 citations, quality-attributes at 6). The heading "shared templates" is also too narrow — most of these are vocabularies and taxonomies, not templates. Part of the same never-updated-README problem as F1.

Recommended fix: List all nine, grouped by kind (templates / vocabularies / governance), and rename the heading to "References (shared templates & vocabularies)."

 ---                                                                                                                                                                                                                              
  ID: F17                                                                                                                                                                                                                          
  Type: Recommendation                                                                                                                                                                                                             
  Severity: Low                                                                                                                                                                                                                    
  Location: references/arc42-template.md:38                                                                                                                                                                                        
                                                                                                                                                                                                                                   
  Problem: Cites `adr-template.md` without the references/ prefix. Every other cross-reference in the plugin — 33 of 34 occurrences — uses the full references/<file>.md form, including within terminology.md,                    
  review-checklist.md, and decision-drivers.md.                                                                                                                                                                                    
                                                                                                                                                                                                                                   
  Why it matters: Cosmetic and currently resolvable, but it is the single deviation from an otherwise perfectly consistent convention, and bare filenames are what break first if reference files are ever reorganized into        
  subdirectories.                                                                                                                                                                                                                  
                                                                                                                                                                                                                                   
  Recommended fix: Change to references/adr-template.md.                                                                                                                                                                           
                 
---
ID: F18
Type: Info
Severity: Low
Location: Plugin-wide

Problem: No module owns designing a target architecture. architecture-discovery:33 explicitly disclaims it; architecture-reviewer critiques but does not design; the three authoring skills document decisions already made. adr-expert comes closest — it challenges reasoning and compares options — but is scoped to recording one decision, not proposing an architecture.

Why it matters: Likely intentional and correct for a documentation toolkit, and I'd recommend against adding a design module. But it's currently an unstated boundary: a user asking "how should we architect this?" will be routed to adr-expert by proximity, which will then apply an ADR-shaped workflow to a question that isn't yet a decision.

Recommended fix: State the plugin-level boundary once in README.md — this toolkit documents, reviews, and governs architecture; it does not design it. No module change needed.

---
ID: F19
Type: Info
Severity: Low
Location: agents/architecture-reviewer.md:10

Problem: The reviewer is granted WebFetch alongside Read, Grep, Glob, but nothing in its body explains what it fetches. The librarian, doing comparable work, gets no WebFetch.

Why it matters: Plausibly deliberate — the Evidence dimension it's told to push hardest on (:54-56) could reasonably involve verifying a cited RFC, spec, or vendor claim. But an unexplained network capability in an otherwise tightly-scoped read-only agent is worth making intentional.

Recommended fix: If it's for evidence verification, say so in the How you review section. If not, drop it.

 ---                                                                                                                                                                                                                              
  Responsibility Matrix                                                                                                                                                                                                            
                                                                                                                                                                                                                                   
  ┌────────────────────────┬──────────────────────────────────────────────────────┬──────────────────────────────────────────┬────────────────────────────────────────────────────────────────────────────────────────────────┐    
  │         Module         │                         Owns                         │               Does not own               │                                       Potential overlap                                        │    
  ├────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────┤    
  │ architecture-bootstrap │ Documentation plan; sequencing across modules;       │ Discovery; authoring any artifact;       │ Discovery — both emit a generation order (F4). Librarian — both write cross-links (wiring vs.  │    
  │                        │ initial cross-link wiring                            │ ongoing consistency                      │ auditing; thin but stated). arc42-expert — ARCHITECTURE.md carries §3/§4 content (F3)          │    
  ├────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────┤    
  │                        │ Reconstructing architecture from code; components,   │ Judging quality; authoring artifacts;    │ Bootstrap — generation order (F4). Librarian — both report coverage gaps from different        │    
  │ architecture-discovery │ integrations, boundaries; candidate ADRs;            │ designing target architecture;           │ evidence bases; same finding reachable from two modules                                        │    
  │                        │ doc-coverage gaps                                    │ doc-to-doc consistency                   │                                                                                                │    
  ├────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────┤    
  │                        │ Individual ADR lifecycle; ADR-worthiness;            │ Repo-wide ADR audits; code discovery;    │ Librarian — bounded well at :78-80 (stop and hand off if it requires scanning the collection). │    
  │ adr-expert             │ superseding                                          │ diagrams; arc42                          │  Reviewer — both critique reasoning; separated by authorship/independence, which is a sound    │    
  │                        │                                                      │                                          │ seam                                                                                           │    
  ├────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────┤    
  │                        │ C4 diagrams; abstraction level; diagram              │ Discovering boundaries from code;        │ Discovery — c4-guidelines.md:59 pushes it across its own boundary (F7). Missing:               │    
  │ c4-expert              │ well-formedness                                      │ judging the design; decision rationale;  │ dynamic/deployment diagrams (F5)                                                               │    
  │                        │                                                      │ repo-wide audits                         │                                                                                                │    
  ├────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────┤    
  │ arc42-expert           │ arc42 12-section docs; narrative; linking ADRs and   │ Code discovery; making decisions;        │ Bootstrap — ARCHITECTURE.md (F3). c4-expert — §6/§7 diagrams unowned by either (F5)            │    
  │                        │ C4; structural completeness                          │ judging quality; repo-wide audits        │                                                                                                │    
  ├────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────┤    
  │                        │ Quality of architectural reasoning; trade-offs,      │ Repo-wide consistency; rewriting         │ Librarian — both produce findings under severity-levels.md; separated cleanly by reasoning vs. │    
  │ architecture-reviewer  │ risks, consequences                                  │ artifacts; making the decision           │  consistency. adr-expert — checklist's Consistency dimension is deliberately capped at 2–3     │    
  │                        │                                                      │                                          │ artifacts, which is good boundary work                                                         │    
  ├────────────────────────┼──────────────────────────────────────────────────────┼──────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────┤    
  │ architecture-librarian │ Doc-to-doc consistency; cross-artifact integrity;    │ Authoring bodies; judging decision       │ Discovery — coverage gaps (see above) and the "overtaken by reality" contradiction (F8).       │    
  │                        │ ADR lifecycle health; index/discoverability          │ quality; reading source code             │ Bootstrap — cross-link writing                                                                 │    
  └────────────────────────┴──────────────────────────────────────────────────────┴──────────────────────────────────────────┴────────────────────────────────────────────────────────────────────────────────────────────────┘    
                
 ---                                                                                                                                                                                                                              
  Dependency / Flow Issues                                                                                                                                                                                                         
                                                                                                                                                                                                                                   
  1. The documented lifecycle is correct. Bootstrap → Discovery → {ADR, C4, arc42} → Librarian, with Reviewer available at any authoring point, matches both the modules' Escalates to blocks and the responsibility split you     
  specified. No restructuring warranted.                                                                                                                                                                                           
  2. Bootstrap↔Discovery is a genuine two-node cycle, not just a bidirectional edge. Bootstrap's step 1 is Discovery; Discovery escalates back to Bootstrap "when this is the first step of documenting a project from zero." Both 
  are defensible entry points, but combined with the duplicated generation order (F4) the boundary is thin enough that a user can't tell which to invoke. Fixing F4 largely resolves this.                                         
  3. Escalation graph is near-fully bidirectional with no termination rule (F10). Reviewer↔Discovery is the most plausible live loop.                                                                                              
  4. Two authoring modules are under-reachable. c4-expert and arc42-expert are escalation targets for only 2 and 3 modules respectively, and neither is reachable from adr-expert (F13) or from architecture-reviewer (F12) —      
  despite the reviewer explicitly accepting both artifact types as input.                                                                                                                                                          
  5. One dead-end handoff: arc42 §6/§7 → C4 dynamic/deployment diagrams → no owner (F5).                                                                                                                                           
  6. No unnecessary dependencies found. No module depends on a concept it shouldn't know about, with the single exception of the reference-embedded violation in F7. The "sequencing, not orchestration" disclaimer in             
  architecture-bootstrap:75-82 is honest and correctly prevents the hidden-orchestration anti-pattern.                                                                                                                             
                                                                                                                                                                                                                                   
  ---                                                                                                                                                                                                                              
  Shared Reference Issues                                                                                                                                                                                                          
                                                                                                                                                                                                                                   
  Missing / unreachable                                                                                                                                                                                                            
  - plugin-design-principles.md — zero inbound citations, absent from README (F9).                                                                                                                                                 
  - Six of nine references undiscoverable from the README (F16).                                                                                                                                                                   
  - No shared definition of ARCHITECTURE.md ownership in terminology.md (F3).                                                                                                                                                      
  - No shared escalation-etiquette rule (F10).                                                                                                                                                                                     
                                                                                                                                                                                                                                   
  Duplicated concepts                                                                                                                                                                                                              
  - C4 review criteria exist in two files with two extra criteria in the non-canonical copy (F6). This is the only true checklist duplication — and notably, quality attributes, decision drivers, severity taxonomy, and          
  terminology are each defined exactly once and cited by path, which is exactly right.                                                                                                                                             
            
 Conflicting definitions                                                                                                                                                                                                          
  - Architecture drift: terminology.md assigns detection exclusively to Discovery; the librarian's "obsolete decisions" check requires it while being forbidden the evidence (F8).                                                 
  - severity-levels.md audience: opening includes skills, ## Usage excludes them, no skill cites it (F11).                                                                                                                         
  - c4-guidelines.md:59 instructs c4-expert to do what c4-expert:31 forbids (F7).                                                                                                                                                  
                                                                                                                                                                                                                                   
  Templates vs. skills — aligned. adr-template.md sections map cleanly onto adr-expert's workflow and the checklist's ADR criteria. arc42-template.md's 12 sections match arc42-expert's list verbatim, including the C4/ADR       
  annotations. c4-guidelines.md levels match c4-expert's. No template drift.                                                                                                                                                       
                                                                                                                                                                                                                                   
  Link integrity — clean. All 34 references/*.md path mentions resolve to existing files; zero broken links. One prefix-convention deviation (F17).                                                                                
                                                                                                                                                                                                                                   
  ---                                                                                                                                                                                                                              
  Anti-Pattern Scan      
                                                                                                                                                                                                                                    
  ┌──────────────────────────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐                                               
  │       Anti-pattern       │                                                                       Verdict                                                                       │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ God skill                │ Not present. Bootstrap is the only candidate and it explicitly disclaims authoring; the risk is the opposite — it may be too thin once F4 is fixed. │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ Duplicate responsibility │ Present, bounded. F4 (generation order), F3 (arc42 content), F6 (C4 checklist).                                                                     │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ Hidden orchestration     │ Not present. Explicitly and correctly disclaimed at architecture-bootstrap:75-82.                                                                   │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ Circular escalation      │ Latent. F10 — structurally bidirectional, no termination rule.                                                                                      │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ Documentation drift      │ Present, critical. F1 — README describes a nonexistent module; also F16.                                                                            │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ Reference duplication    │ Present, one instance. F6.                                                                                                                          │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ Unclear ownership        │ Present. ARCHITECTURE.md (F3), C4 dynamic/deployment (F5), plugin meta-architecture (F9).                                                           │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ Premature abstraction    │ Not present. All nine references are justified by their fan-in — except the orphan (F9), which is under-used rather than premature.                 │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ Unnecessary shared files │ None.                                                                                                                                               │                                               
  ├──────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤                                               
  │ Missing shared concepts  │ Present. Escalation etiquette, ARCHITECTURE.md ownership.                                                                                           │                                               
  └──────────────────────────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘                                               
                            
 ---                                                                                                                                                                                                                              
  Plugin Maturity Assessment                                                                                                                                                                                                       
                                                                                                                                                                                                                                   
  Maintainable: yes. Small, flat, no runtime coupling, and the three-part boundary block is applied uniformly across all seven modules. That uniformity is the plugin's biggest asset — it makes reviews like this one tractable   
  and makes violations visible.                                                                                                                                                                                                    
                                                                                                                                                                                                                                   
  Can new skills be added safely: mostly. The pattern to follow is obvious from any existing module, and plugin-design-principles.md states it precisely. The problem is that the file is unreachable (F9), so a contributor learns
  the pattern by imitation rather than by rule — and imitation won't teach them the naming convention (F14) or the escalation etiquette that doesn't yet exist (F10).                                                              
                                                                                                                                                                                                                                   
  Extension points: clear for skills, undefined for references. Adding a module is well-understood. Adding or modifying a reference has no stated rule — which is exactly how the C4 checklist ended up in two places (F6) and how 
  a boundary violation got embedded in a guidelines file (F7). Reference files are shared mutable state with no ownership rule.                                                                                                    
                                                                                                                                                                                                                                   
  Ownership obvious: yes, with three named exceptions — ARCHITECTURE.md, C4 dynamic/deployment diagrams, and the plugin's own meta-architecture.                                                                                   
                                                                                                                                                                                                                                   
  Will it scale: yes, with one caveat. The roadmap's DDD / RFC / threat-modeling modules fit the existing shape cleanly. The scaling risk is not the module count but the escalation graph — at 7 modules the near-complete        
  bidirectional mesh is manageable; at 10–12 it will need either a stated etiquette (F10) or an explicit tiering (extraction → authoring → governance) that constrains which layers may escalate to which.                         
                                                                                                                                                                                                                                   
  ---                                                                                                                                                                                                                              
  Recommended Changes                                                                                                                                                                                                              
                    
Must fix — architecture-breaking or runtime-breaking

1. F1 — README documents a nonexistent skill (architecture-documentation → architecture-bootstrap) in three places; also update the session's additional-working-directory setting. Critical.
2. F2 — Librarian's tool grant cannot support its twice-declared edit behavior; reconcile capability and contract. High.
3. F3 — Bootstrap produces arc42 §3/§4 content while disclaiming arc42 authoring; assign an owner to ARCHITECTURE.md. High.
4. F4 — Discovery and Bootstrap both own the generation order; split into gaps (Discovery) vs. plan (Bootstrap). High.
5. F5 — arc42 §6/§7 point at C4 dynamic/deployment diagrams no module owns; extend c4-expert or declare out of scope. High.

Should improve — quality and consistency

6. F6 — Consolidate the duplicated C4 review checklist into review-checklist.md.
7. F7 — Remove the cross-check-with-discovery instruction that violates c4-expert's boundary; make it a handoff.
8. F8 — Narrow the librarian's "obsolete decisions" check to doc-to-doc evidence.
9. F9 — Make plugin-design-principles.md reachable from the README and from at least one module.
10. F10 — Add an escalation-etiquette rule with a termination condition.
11. F11 — Reconcile severity-levels.md's stated audience with its actual citations.
12. F12 / F13 — Add the four missing escalation edges (reviewer → c4/arc42; adr → c4/arc42).
13. F16 — List all nine references in the README, grouped by kind.

Nice to have — future-proofing

14. F14 — Write down the module naming convention before the roadmap modules land.
15. F15 — Make adr-expert's status marker intentional either way.
16. F17 — Fix the one bare-filename reference path.
17. F18 — State the plugin-level boundary: documents architecture, does not design it.
18. F19 — Justify or drop the reviewer's WebFetch grant.
19. New — Consider a CI lint asserting (a) every skill name in the README resolves to a skills/ directory, (b) every references/*.md path mentioned resolves, and (c) every SKILL.md contains all three boundary headings. Items (a) and (c) would have caught F1 and prevented the class entirely.

