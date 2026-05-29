# Email Draft — Config V2 Follow-up (April 16, 2026)

**To:** Adam Curcie, Devon D'Andrea
**CC:** Laura Perry, Richard Sacco, Aaron Diefes
**Subject:** Config V2 — Follow-up from today's sessions + items for your review

---

Hi Adam and Devon,

Thanks for the two great sessions today — we covered a lot of ground on the configuration system design. Wanted to recap what we landed on and outline a few things I need from your side before our next meeting.

**What we decided today:**

- **Terminology:** "Three-way rules" are now **MCS pairs** (Model–Carrier–Service Plan). I'll update all documentation to reflect this.
- **Schema immutability:** Once a parameter is added to the schema, it cannot be deleted — only new parameters can be added (typically when firmware updates introduce new functionality).
- **Migration approach:** Legacy DAT files and the new config system will run side by side. We'll start migration with a small set of pioneer devices to validate that the new system produces identical output to the current DAT files before rolling out more broadly.
- **Comparison tool:** We'll build a validation tool that previews the new config as key-value pairs side by side with the legacy DAT file, so you can confirm they match before switching any device over.
- **Distributor/child company hierarchy:** Child companies can be unlinked from parent override rules for independent configuration, and relinked later if needed.
- **Conflict validation:** The system will detect conflicts at the override-set level (which sets conflict with each other) rather than checking every possible company combination.

**What I need from you:**

1. **Sample company configuration files** — Adam, you mentioned you'd send these over. Specifically, I'm looking for:
   - A few examples of **typical company override blocks** (the common 1–5 parameter sets)
   - At least one **firewall-heavy example** (the 40+ variation scenario we discussed)
   - If possible, pick examples that span different models and carriers so I can validate the override set design against real data

2. **Review the Confluence documentation** — I know you've started adding comments (thank you!). I'll go through those and incorporate what I can. If you haven't finished reading through the full requirements doc, please try to do so before our next session — it covers the versioning/approval workflow, conflict detection rules, and resolution logic that we'll need to discuss next.

3. **Try the prototype** — The interactive prototype is available and walks through the Grand Schema, Model Configuration, MCS pairs, Override Sets, Device Preview, and Config Push screens. Clicking through the actual flows will surface questions and gaps faster than reading alone. I can resend the link if needed.

**Still on our plate to discuss next time:**

- Detailed validation rules (when/how required parameters are enforced)
- Whether company overrides scoped to "Any model" can override model-specific MCS pair values
- Schema flag behavior for controlling which parameters are available at the company level
- Conflict detection UX (what the admin sees when a conflict is caught)
- Cellular backup as a matching dimension vs. a schema parameter (need your confirmation)

I know you're traveling next week (Tue–Fri), so if we can get the sample files and your doc review done before then, we'll be in good shape to hit the ground running when you're back. Happy to do a quick Monday sync if timing works on your end.

Thanks again — really productive day.

Best,
Aksana
