# PRD – Paste as plain text (v4)

**Design of record:** [clickable prototype](prototype/index.html) · **Status:** Ready for review

---

## 0. What changed since v2.2 and v3

v2.2 ([this site, archived at v2.3](v2.3/)) is what [webui#8129](https://github.com/saleshandy/saleshandy-webui/pull/8129) and [edge#11339](https://github.com/saleshandy/saleshandy-edge/pull/11339) were built from. v3 ([prd-pasting-styled-content](https://product-team-sh.github.io/prd-pasting-styled-content/)) followed it. v4 replaces both.

| # | v2.2 / v3 said | v4 says | Why |
|---|---|---|---|
| 1 | v2.2: a personal "Email Editor" settings page. v3: no paste setting; paste follows Remove Styling | One per-user switch, **Always paste as plain text**, in the editor's "•••" menu and in the paste chip | The switch sits where pasting happens. Pasting is a personal habit, not a sequence choice |
| 2 | v2.2: a receipt, then a one-step ⌘Z back to the original. v3: the Styling detected bar | A **paste chip** at the end of the paste switches it between Plain text and Keep formatting, both ways, as often as wanted. Ctrl+Z works as normal | One control, both directions, no hidden undo levels |
| 3 | v3: Keep on the Styling detected bar turns Remove Styling off for the sequence | A paste never shows the bar and never changes a sequence setting. The bar stays, unchanged, for styling added from the toolbar | The silent sequence write came from pastes |
| 4 | Spintax: v2.2 had an override; v3 showed the same bar in variants | Variants are always plain text, with a note and no chip | Saving keeps only the text anyway |
| 5 | v3 rewrote the Remove Styling description | Safety Settings are untouched: names, layout, values, saving | Scope is paste only |

- **Void from v2.2:**
    - D2 to D6.
    - S1-1 to S1-18: the Email Editor settings page is gone.
    - S2-F0 to S2-F6 and S2-1 to S2-37: the receipt and the one-step ⌘Z model are gone.
    - S3-1 to S3-4: there is no spintax override and no receipt in variants.
    - D1 is partly carried: the setting is still per user, but its home and name change (row 1).
- **Void from v3:** A1 to A12 and A17 to A20. They test the bar on paste, Keep writing the setting, and the "answered" state, none of which applies to a paste any more.
- **Carried forward unchanged:**
    - v2.2 X-1 to X-7 (chip integrity) and S3-5 to S3-7 (merge-tag and spintax chips between the body and variants), kept in §8.
    - v3 A13 to A16 become P-6, P-7, P-9 and P-8 below.

---

## 1. Problem

**Users paste without seeing what comes in.** People paste drafts from Word, Google Docs, web pages and old emails, and invisible markup comes with the text: a one-paragraph Word paste carries 38.4 KB of HTML for 1.1 KB of text (measured, v2.2 test, Sep 2026). The step editor keeps it, because its own paste cleanup is switched off ([editor.tsx L1374](https://github.com/saleshandy/saleshandy-webui/blob/aeda4e3654fd76c54725fd676463c3694a35eed3/src/shared/editor/editor.tsx#L1374), webui `aeda4e3`, 16 Oct 2026).

**A paste can change the whole sequence.** Remove Styling Automatically is on for 81.8% of sequences (measured, Metabase, sequences created 1 Jul to 12 Oct 2026). Under it, a styled paste brings up the "Styling detected" bar, and its Keep button turns Remove Styling off for every email in the sequence ([email-modal.tsx L1116–1197](https://github.com/saleshandy/saleshandy-webui/blob/c9feb458b3d4d35918d4196d2b265061fe3f9ece/src/components/sequence/shared/modals/email-modal/email-modal.tsx#L1116-L1197)).

**What the fix itself creates:**

- Plain paste drops links and bold the user may have wanted. The chip (§6.1) is the way back.
- A chip that goes unnoticed is no way back. §6.1 fixes where it sits and how it is noticed.
- A chip only a mouse can reach locks keyboard users out. §6.1 gives it a keyboard route.

---

## 2. Why this approach

| Approach | Rejected because |
|---|---|
| Clean the HTML and keep the formatting | The editor still shows formatting Remove Styling won't send |
| A paste setting per sequence (v3) | A personal habit, answered again in every sequence |
| An account setting set by admins (edge#11339) | One person's choice applies to everyone |
| Plain paste with no way back | Deliberate formatted pastes would need retyping. Full formatting (Remove Styling off) is used by 16.7% of sequences (measured, same query) |
| A notice bar with Undo under the paste (board of 3 Nov) | Four notice states, Undo only one way, and an "Always keep?" prompt. The chip replaces all of it with one control |

**Not claimed:** that styling, or Remove Styling, changes deliverability. No test, ticket or data behind that claim was found.

**The bar:** Ctrl+Shift+V is the plain paste people already know, and a paste-options chip is what Word, Outlook and Sheets use. v4 makes the first the default and the second the way back.

---

## 3. What we're building

1. **A paste into the sequence email editor lands as plain text by default**, exactly like Ctrl+Shift+V, and hidden markup never gets in.
2. **A paste chip at the end of every paste that could have landed differently** switches it between Plain text and Keep formatting, and carries the per-user switch.
3. **With the switch off, a paste follows the sequence's Safety Settings as they are today.**

**Invariants:**

- The chip never takes the cursor.
- No paste changes a sequence setting.
- Text outside the paste is never touched by the chip.
- No stored value changes, so there is no migration.

---

## 4. User stories

| ID | Story |
|---|---|
| US-1 | As a writer, I paste a draft from Word or Docs and get clean text without thinking about it |
| US-2 | As a writer who pasted something formatted on purpose, I get the formatting back from the chip, and can switch back again |
| US-3 | As a writer who always pastes formatted content, I turn plain paste off once and am not asked again |
| US-4 | As a sequence owner, a paste never changes my sequence's settings |
| US-5 **(new)** | As a keyboard or screen-reader user, I hear that the paste was converted and can switch it without a mouse |

---

## 5. Decisions

**D1 – Plain paste is exactly Ctrl+Shift+V.** Paragraphs and line breaks stay; links, bold, lists, images and fonts go. **Consequence to accept explicitly:** a pasted CTA link needs Keep formatting. The chip does not say "link removed" (decided 4 Nov), so the user must notice the missing link themselves.

**D2 – The switch is per user, named "Always paste as plain text", and lives in the editor's "•••" menu and in the chip.** Reverses v2.2 D1's home (a User Settings page). **Consequence to accept explicitly:** there is no settings page for it.

**D3 – With the switch off, a paste follows the sequence's Safety Settings as they are today:**

- Remove Styling Automatically: it lands as it will be sent.
- Text only email: plain text.
- Neither: as copied, minus hidden markup.

**Consequence to accept explicitly:** keeping formatting never overrides what the sequence sends; fonts and colours are still removed under Remove Styling.

**D4 – The chip replaces the Styling detected bar for pastes only.** Styling added from the toolbar still shows today's bar, and its Keep still turns Remove Styling off. **Consequence to accept explicitly:** that toolbar behaviour is a known issue left as it is; it is out of scope.

**D5 – Scope is the email draft in the sequence step editor:** its body, subject line and spintax variants. **Consequence to accept explicitly:** templates, Unified Inbox replies, task notes and the signature editor keep today's paste, so the same Word text behaves differently there.

**D6 – Only an edit closes the chip.** That means a key that changes the text, Esc, the next paste, Ctrl+Z, or a click in the text outside the paste. Arrow keys, Tab and scrolling leave it open. **Consequence to accept explicitly:** the chip can stay on screen, pinned, while the user scrolls and reads.

**D7 – "Plain text" is the word everywhere:** the chip, the menu and the switch. Safety Settings and Admin keep "text only".

---

## 6. Surfaces

### 6.1 Paste in the email body, and the paste chip

**What it's for:** every paste or drop into a sequence step's email body.

**What lands:**

| Switch | Remove Styling Automatically (default) | Neither | Text only email |
|---|---|---|---|
| **On** (default) | Plain text. Chip: "Plain text" | Plain text. Chip: "Plain text" | Plain text, no chip |
| **Off** | As Remove Styling will send it: bold, italic, links, lists, images and tables kept; fonts, sizes, colours, highlight, underline, strikethrough, alignment and headings removed. Chip: "Formatting kept" | As copied, minus hidden markup. Chip: "Formatting kept" | Plain text, no chip |

- **Hidden markup** changes nothing visible: Word and Docs classes, comments, empty spans, and font declarations equal to the editor's own. It is always removed, in every cell.
- **The chip shows only when Plain text and Keep formatting would look different** for this paste, meaning a link, bold, a list, an image, a colour or a font differs.
- **No chip** for:
    - a plain-text source (Notepad);
    - a paste copied inside Saleshandy;
    - Ctrl+Shift+V or Cmd+Shift+V;
    - Text only email;
    - a paste that looks the same either way (a name from LinkedIn).
- Literal `{{First Name}}` and `{spin}…{endspin}` in pasted text become chips, whichever way it lands.

![The chip after a paste: just below where the paste ends, the pasted text tinted for a moment](screens/01-chip-after-paste.png)

**The chip:**

| State | Label | Menu |
|---|---|---|
| Landed plain | Clipboard icon · **Plain text ▾** | ✓ Plain text (hint Ctrl+Shift+V, ⇧⌘V on a Mac) · Keep formatting · divider · Always paste as plain text (switch) |
| Landed formatted | Clipboard icon · **Formatting kept ▾** | Plain text · ✓ Keep formatting · divider · Always paste as plain text (switch) |

- Picking the other option rewrites only this paste. The user can switch back and forth as often as wanted.
- The switch changes every future paste, not this one.

![The menu: this paste either way, and the switch for every paste](screens/02-chip-menu.png)

![Switch off: the chip still shows, so plain text stays one click away](screens/03-chip-switch-off.png)

**Where it sits:**

| Rule | Requirement |
|---|---|
| Anchor | 4px below the last pasted line, left edge where the paste ends |
| Sideways | Stays at least 12px inside the editor's edges |
| Room below | If the paste ends near the bottom, the editor scrolls until the chip fits below it. The chip never covers the pasted text; it may cover the next line while open |
| Scrolled away | Pinned to the top edge of the body, or to its bottom edge beside the format badge (just above the badge when the editor is too narrow for both). It returns to the paste when the paste is back in view |
| Switching while pinned | The editor scrolls back to the paste and the tint replays |
| Layers | Above the email text; below the toolbar, the subject line, the badge, menus and dialogs |
| Menu | Opens below the chip, or above it when there is no room, and stays inside the editor |

![Scrolled away: the chip stays pinned beside the badge](screens/04-chip-pinned.png)

**How it is noticed:**

- On entrance it rises in over 180ms with one soft blue ring.
- The pasted text gets a light blue tint that fades over 1.2s, and the tint replays after every switch.
- At rest it has a white fill, a grey-500 border (4.8:1 on white) and a blue clipboard icon.
- With reduced motion on: no rise and no ring; the tint still shows.

**Keyboard and screen readers:**

- Ctrl pressed and released on its own opens the menu with the ticked option focused, as in Word. Tab from the text also reaches the chip.
- In the menu: arrow keys move, Home and End jump, Enter picks, Esc closes and returns the cursor to the text, and Tab closes.
- A polite live announcement, once per paste: "Pasted as plain text. Press Ctrl for paste options." For a formatted paste the first sentence is "Pasted with formatting." Each switch is announced: "Switched to plain text." or "Switched to keep formatting."

**Closes on:** a key that changes the text (typing, Delete, Backspace, Enter), Esc, the next paste, Ctrl+Z, or a click in the text outside the paste. A click inside the paste keeps it. Ctrl+Z works as normal and removes the paste.

**Explicitly out of scope:** pasted image files (screenshots) behave as today.

**Acceptance criteria**

| ID | Case | Expected | P |
|---|---|---|---|
| P-1 | Switch on, Remove Styling: paste a Word draft with bold, a link, a list and a coloured font | Body holds plain text with paragraphs; no `<b>`, `<a>`, list, `style` or Word class; chip "Plain text" | P0 |
| P-2 | Switch on: paste a name copied from LinkedIn | No chip; no hidden markup in the body | P0 |
| P-3 | Switch off, Remove Styling: paste the P-1 draft | Bold, link and list kept; no `style`, font or colour; chip "Formatting kept" | P0 |
| P-4 | Switch off, Neither: paste the P-1 draft | Visible formatting kept; no Word classes or comments; chip "Formatting kept" | P0 |
| P-5 | Text only email: paste the P-1 draft with the switch on, then off | Plain text both times; no chip either time | P0 |
| P-6 | Paste from Notepad | Lands as is; no chip | P0 |
| P-7 | Copy from another step or template inside Saleshandy, then paste | Lands as copied, merge-tag and spintax chips kept; no paste chip | P0 |
| P-8 | Ctrl+Shift+V (Cmd+Shift+V) with the switch off | Plain text; no chip | P0 |
| P-9 | Paste literal `{{First Name}}` and `{spin}a\|b{endspin}` | Both become chips | P0 |
| P-10 | Paste, then press an arrow key, then Tab | The chip stays both times | P0 |
| P-11 | Paste, then type one character | The chip closes; the cursor never left the body | P0 |
| P-12 | Paste twice | One chip, for the second paste | P1 |
| P-13 | Drag and drop the P-1 draft | Same as P-1 | P1 |
| **P-F1** | P-1, then chip → **Keep formatting** | Body holds what Remove Styling sends (bold, link and list back; no fonts or colours); text outside the paste byte-identical to before the paste; chip reads "Formatting kept" | P0 · **sign off by name, owner: QA lead (name to confirm)** |
| P-F2 | P-F1, then chip → Plain text | Back to the P-1 result exactly | P0 |
| P-F3 | Switch on, Neither: P-1, then Keep formatting | The paste as copied, minus hidden markup | P0 |
| P-F4 | P-1, then Ctrl+Z | The paste is removed; body byte-identical to before the paste; no chip | P0 |
| P-F5 | Chip → switch "Always paste as plain text" off | This paste is unchanged; the next paste follows the Off row; after a reload the "•••" menu reads Off (read back: menu state and `/user/meta`) | P0 |
| C-1 | Paste near the bottom of the editor | The editor scrolls; the chip sits 4px under the paste end and covers no pasted text | P0 |
| C-2 | Paste ending near the right edge | The chip stays 12px inside the edge | P1 |
| C-3 | Paste, then scroll the paste out of view, both ways | The chip pins to the top edge, then to the bottom edge beside the badge; it returns when the paste is back in view | P0 |
| C-4 | Pinned chip → Keep formatting | The editor scrolls to the paste; the tint replays | P0 |
| C-5 | Click in the text outside the paste, then (after a new paste) inside it | The chip closes the first time and stays the second | P1 |
| C-6 | Paste, then press and release Ctrl alone | The menu opens with the ticked option focused; arrow keys, Enter and Esc behave as above | P0 |
| C-7 | Screen reader on: paste the P-1 draft, then switch | "Pasted as plain text. Press Ctrl for paste options." once, then "Switched to keep formatting." | P0 |
| C-8 | Reduced motion on: paste | No rise or ring; the tint shows | P2 |
| C-9 | A Mac | The menu hint reads ⇧⌘V | P2 |

### 6.2 The switch in the editor's "•••" menu

**What it's for:** each user's choice to paste as plain text or keep formatting, in every sequence. Reached from the toolbar's "•••" menu, or from the chip.

**What it must do:**

- A toggle button at the end of the "•••" menu, in its own group, labelled with a clipboard icon. Pressed means On. Tooltip: "Always paste as plain text: On" or "…: Off".
- The default is On for every user, new and existing.
- It saves when switched, for that user, across devices. It applies from the next paste.
- It is the same setting as the switch in the chip. It is never greyed out.

| ID | Case | Expected | P |
|---|---|---|---|
| M-1 | New user opens the "•••" menu | Toggle pressed; tooltip "Always paste as plain text: On" | P0 |
| M-2 | Turn it off, reload, open another sequence's step | Still off (read back: menu state and `/user/meta`) | P0 |
| M-3 | User A turns it off | User B in the same account is still on | P0 |
| M-4 | Turn it off in the chip | The "•••" menu shows Off at once | P0 |

### 6.3 Subject line and spintax variants

- **Subject:** a paste is always plain text, on one line, with no chip, whatever the switch.
- **Spintax variant field:** a paste is always plain text, with no chip. The field shows "Variants are saved as plain text." under the inputs. Saving already keeps only text ([helpers.ts L1226](https://github.com/saleshandy/saleshandy-webui/blob/main/src/shared/editor/helpers.ts#L1226)).

| ID | Case | Expected | P |
|---|---|---|---|
| F-1 | Paste the P-1 draft into the subject, switch off | One line of plain text; no chip | P0 |
| F-2 | Paste the P-1 draft into a variant, switch off | Plain text; no chip; the note visible | P0 |

---

## 7. Reconciliation with existing behaviour

**Scope map: every way styling reaches the editor, and what covers it.**

| Path | Covered by |
|---|---|
| Paste or drop into a step body | §6.1 |
| Paste into the subject or a variant | §6.3 |
| Toolbar styling under Remove Styling | Today's Styling detected bar, unchanged (D4) |
| Templates and Unified Inbox replies | Today's paste, unchanged (D5) |
| Signature styling | Today's behaviour, unchanged |

**Unchanged and out of scope:** Safety Settings, the format badge and its pop-up, the toolbar, and drafts lost when leaving the editor through the badge's "Safety Settings" link or "Connect now" ([handleConnectNowClick L1286](https://github.com/saleshandy/saleshandy-webui/blob/aeda4e3654fd76c54725fd676463c3694a35eed3/src/components/sequence/shared/modals/email-modal/email-modal.tsx#L1286-L1302); a separate bug ticket).

---

## 8. Cross-cutting

**Chip integrity, carried from v2.2, release blocker:** X-1 to X-7 and S3-5 to S3-7 apply unchanged. Merge-tag and spintax chips survive copy and paste within the body, to and from variants and between steps; pasting over chips removes them cleanly; malformed spintax stays literal.

**Keyboard:**

| Key | Behaviour |
|---|---|
| Ctrl+Shift+V / Cmd+Shift+V | Always plain text, never a chip, in every field |
| Ctrl+Z / Cmd+Z | Standard. After a paste it removes the paste and the chip |
| Ctrl alone | Opens the chip's menu when a chip is showing; nothing otherwise |
| Esc | Closes the chip's menu, then the chip; never moves the cursor |

---

## 9. Implementation notes

- **The chip sits on top of an editor in an iframe.** It must be positioned at the paste end and pinned to the body's edges while scrolling (§6.1). **Resolve before estimation:** how the chip tracks the paste end inside the TinyMCE iframe, and that "Ctrl alone" is free in the editor and the browsers we support.
- **"Lands as Remove Styling will send it" must match the send step.** The send step's cleanup is [parse-content.ts](https://github.com/saleshandy/saleshandy-email-composer/blob/37b1fb6b11691d5d8ffdb50045fa9bcc378e7432/src/common/utils/parse-content.ts), applied at [app.service.ts L1397](https://github.com/saleshandy/saleshandy-email-composer/blob/37b1fb6b11691d5d8ffdb50045fa9bcc378e7432/src/app.service.ts#L1397-L1405). Paste needs identical output, held to one shared test set. **Resolve before estimation:** one shared implementation, or two with shared tests.
- **The step editor shares one component** with the signature, template, inbox-reply and task-note editors (code search, 3 Nov). The new paste rules must be switched on for the sequence step editor only.
- **The per-user switch:** edge#11339 already returns `pasteAsPlainText` (default on) from `/user/meta`, but per account and admin-only. It must become per user and changeable by the user. edge#11339 must not merge as is.
- **Resolve before estimation:**
  1. With the editor's built-in paste cleanup on, real Word, Google Docs and Gmail pastes lose hidden markup but keep visible formatting, for Keep formatting and switch-off. Assumed, not tested.
  2. The "would look different" check behind the chip is reliable on the same samples. Assumed, not tested.

- **Not adopted:** links kept in plain paste (D1); a "link removed" label (4 Nov); an icon-only chip, which doesn't say the paste changed; a per-sequence or per-account setting (§2).

---

## 10. Measurement

**Privacy:** never log pasted content, email text or URLs. Log only the kinds of formatting removed and the sizes.

| Event | Fires when | Properties |
|---|---|---|
| `paste_landed` | Any paste or drop into a step body | safety_setting, switch, chip_shown, landed (plain, formatted), removed_kinds (link, bold, list, image, colour, font), html_kb, kept_kb |
| `paste_chip_opened` | The chip's menu opens | via (click, ctrl, tab), pinned |
| `paste_chip_switched` | Plain text or Keep formatting is picked | to, pinned, seconds_since_paste |
| `paste_switch_changed` | The switch changes | to, from (menu, chip) |

**Leading indicators**, read from day 1, decide by day 14:

| Metric | Definition | Healthy | Investigate |
|---|---|---|---|
| **Keep-formatting rate (most important)** | switches to Keep formatting ÷ chips shown with the switch on | < 30% (target; assumed, no baseline) | > 50%: plain is wrong too often |
| Chip rate | chips shown ÷ pastes | < 50% (target; assumed) | > 70%: the "looks different" check is too loose |
| Keyboard opens | opens via ctrl or tab ÷ all opens | – (read only) | 0 after 2 weeks: check the shortcut works |

**Lagging indicators**, day 30 and day 90:

- Users with the switch off ÷ users who pasted (US-3).
- Tickets mentioning pasted formatting or lost links (US-1, US-2).
- Sequences whose Remove Styling setting was turned off within a minute of a paste: target 0 (US-4).

**Guardrails:**

| Guardrail | Threshold | Action |
|---|---|---|
| Chip integrity | Any X-1 to X-7 failure in production | Halt rollout |
| Text outside a paste changed by a switch | Any confirmed case | Roll back |
| Formatting or link tickets | 2× the 4-week pre-launch weekly average (target) | Halt rollout |

**Success criteria**, judged at day 30 at 100%:
1. **Non-negotiable:** no chip-integrity failures, and no text outside a paste changed.
2. **Non-negotiable:** no sequence setting changed by a paste.
3. **Iterate, don't revert:** the keep-formatting rate stays under 30%.

Instrumentation ships with the feature; the gates below cannot be read otherwise.

---

## 11. Rollout

Staged exposure limits the blast radius. It is not an A/B test: every user with the feature gets the same default.

| Stage | Who | Gate |
|---|---|---|
| 1 | Internal accounts | Every P0 passes; **P-F1 signed off by name** |
| 2 | 5% of accounts | No guardrail breached for one week |
| 3 | 100% | No guardrail breached at 5% |

No stage starts before the "resolve before estimation" items in §9 are settled. The help article on pasting is updated before stage 3.

---

## 12. Open questions

| # | Question | Why it matters | Recommendation | Blocking |
|---|---|---|---|---|
| 1 | Does Dhruv agree that, with the switch off, pastes follow today's Safety Settings (D3)? | D3 is the model's main rule | Confirm before dev review | Yes |
| 2 | Is "Ctrl alone" free in TinyMCE and our browsers? | It is the keyboard route to Keep formatting | If not, use Alt+Ctrl+V for the menu and keep Tab | Yes |
| 3 | Drafts lost when leaving the editor | Users lose work today, outside this scope | A separate bug ticket for Rajat | No |

---

## 13. Priority & timeline

- **Priority:** P1 (assumed; Hitesh to confirm).
- **Target:** TBD after estimation.
- **Release gate:** §11 stage 1.
- **Named owners still required:**
    - Dev: Sunny, who built #8129 (to confirm).
    - QA lead for the P-F1 sign-off: name to confirm.
    - Tech review: Rajat.
    - PM: Hitesh.
    - Help article: CS, name to confirm.

---

## Appendix – reference index

- **Design of record:** [the clickable prototype](prototype/index.html). Section 2 is the live paste simulator; section 3 has the chip states and the placement rules; section 4 shows what lands under each setting; section 5 covers the subject and spintax. Production Saleshandy webui is the visual source.
- **Code read:** webui `aeda4e3` (16 Oct) and `c9feb45` (12 Oct); email-composer `37b1fb6` (12 Oct). Links are inline above.
- **Data:** Metabase db 2, run 12 Oct 2026.
- **Superseded:**
    - [PRD v2.2 / prototype v2.3](v2.3/), archived on this site;
    - [PRD v3](https://product-team-sh.github.io/prd-pasting-styled-content/).
- **In-flight PRs built from earlier versions:**
    - [webui#8129](https://github.com/saleshandy/saleshandy-webui/pull/8129) (open);
    - [edge#11339](https://github.com/saleshandy/saleshandy-edge/pull/11339) (approved, not merged; must not merge as is).
