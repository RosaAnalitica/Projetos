# Version History

This file reconstructs the website history from the standalone HTML snapshots that existed before the Git repository was created.

## Snapshot map

- `archive/site_v0.html`: earliest available version
- `archive/site_v1.html`: intermediate revision
- `archive/site_v2.html`: latest provided revision
- `archive/site_v3.html`: live site as promoted from `site_v2.html`, before the hero visual-card fixes (footer read "Versão 3.1")
- `index.html`: current site, now at v4, with the hero visual-card fixes applied (footer reads "Versão 4.0")

## v0 -> v1

### Hero section

- Kept the overall positioning, but replaced the broad intro paragraph with a mini-questionnaire that pushes the visitor to self-diagnose their data maturity.
- Changed the primary CTA from a free diagnosis framing to a more direct numbers-oriented CTA.
- Updated headline copy to the current "intuição ou inteligência" framing.
- Reworked the stats copy to be more explicit and executive-facing.

### Messaging and content

- Rewrote the pain-point cards with clearer business language and less jargon-heavy phrasing.
- Adjusted the culture section copy, including the third principle from a metaphor-heavy phrasing to a more direct decision-oriented message.
- Refined the founder bios and normalized company/sector descriptions.

### Information architecture

- Moved the founders section earlier in the page so credibility appears before the philosophy and methodology sections.
- Reordered the company-logo block to appear after the founder cards.
- Removed the separate dark "compromisso" value banner that existed at the end of the founders section in `v0`.

### New or expanded sections

- Expanded the culture section right-hand panel from two organizational blocks to three, adding the backstage/engineering layer.
- Expanded the technology section from four technical capability blocks to six by adding:
  - visualization and BI
  - automations

### Contact experience

- Rebuilt the contact area from a centered light card into a darker, more premium split layout with direct action buttons and supporting copy.

## v1 -> v2

### Hero redesign

- Rebuilt the hero into a single unified card instead of two more separate visual columns sitting directly on the page background.
- Removed the small badge above the headline.
- Moved the main CTA below the hero card and changed the CTA copy to "Quero decidir com inteligência".
- Switched the quiz card and visual card styling to softer rose backgrounds with white accents.

### Social proof and credibility

- Added an external source link to the first statistics block in the hero.
- Tightened the presentation of the executive value proposition inside the hero visual card.

### Copy refinements

- Rewrote the second, third, and fourth pain-point cards again to make the meeting friction and trust issues more concrete.
- Renamed the fourth pain-point title to a more conversational phrasing.
- Restored the culture principle phrasing from the more literal `v1` wording to the metaphorical "Entra ouro, saem joias".
- Applied smaller editorial refinements in bios and descriptive copy.

### Visual polish

- Increased hero spacing and card emphasis.
- Simplified some borders, shadows, and background treatments to make the top of the page feel more cohesive.
- Kept the broader structure introduced in `v1`, focusing `v2` on polishing rather than reshaping the full page.

## v3 -> v4

### Hero visual card

- Removed the bouncing compass icon (`animate-bounce` badge) that floated over the top-left corner of the hero visual card.
- Changed the hero grid and visual card layout from vertically centered to stretched, so the visual card's top edge now aligns with the top of the headline ("Sua empresa decide...") and its bottom edge aligns with the bottom of the percentage stats row.
- Content inside the visual card is now vertically centered within the taller card instead of being top-packed.

### Footer

- Bumped the displayed version label from "Versão 3.1" to "Versão 4.0".

## Import notes

- The Git repository was created after these three HTML snapshots already existed.
- This history was reconstructed from file-to-file comparison rather than from commit metadata.
- Referenced image assets are still missing from the repository and should be added in a later commit if you want the site to render fully offline.