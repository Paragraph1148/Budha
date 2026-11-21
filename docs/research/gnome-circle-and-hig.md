# GNOME Circle acceptance and HIG rules for Budha

Date researched: 2026-11-21

Question: what must Budha (a Python + GTK4 + libadwaita GUI for Nous Research's Hermes Agent) satisfy to get into GNOME Circle, and which GNOME HIG rules matter most for an AI chat/agent app?

## Sources

Circle (GNOME GitLab, read from the `main` branch on 2026-11-21):

- Circle README: https://gitlab.gnome.org/Teams/Circle/-/blob/main/README.md
- App application issue template: https://gitlab.gnome.org/Teams/Circle/-/blob/main/.gitlab/issue_templates/app_application.md
- Review procedure: https://gitlab.gnome.org/Teams/Circle/-/blob/main/review_procedure.md
- Membership guide: https://gitlab.gnome.org/Teams/Circle/-/blob/main/membership_guide.md
- Current member list: https://gitlab.gnome.org/Teams/Circle/-/blob/main/data/apps.json
- App criteria (owned by Release Team, not by the Circle repo): https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md
- Circle Committee announcement, 2026-05-29 (linked from the README): https://blogs.gnome.org/sophieh/2026/05/29/updates-from-the-circle-committee/
- Review threads: Alpaca https://gitlab.gnome.org/Teams/Circle/-/issues/230, Gtk LLM Chat https://gitlab.gnome.org/Teams/Circle/-/issues/249, Dev Toolbox https://gitlab.gnome.org/Teams/Circle/-/issues/253, GCodeKit5 https://gitlab.gnome.org/Teams/Circle/-/issues/263, criteria review https://gitlab.gnome.org/Teams/Circle/-/issues/281, runtime group notice https://gitlab.gnome.org/Teams/Circle/-/issues/280
- GNOME Shell extension review guidelines (where Circle's AI policy text comes from): https://gjs.guide/extensions/review-guidelines/review-guidelines.html

Flathub (docs.flathub.org, cross-checked against https://github.com/flathub-infra/documentation):

- Requirements: https://docs.flathub.org/docs/for-app-authors/requirements
- Submission: https://docs.flathub.org/docs/for-app-authors/submission
- MetaInfo guidelines: https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/
- Quality guidelines: https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/quality-guidelines
- Linter: https://docs.flathub.org/docs/for-app-authors/linter

GNOME developer docs:

- HIG: https://developer.gnome.org/hig/ (source: https://gitlab.gnome.org/Teams/Design/hig-www, last commit 2026-06-20)
- Application ID: https://developer.gnome.org/documentation/tutorials/application-id.html
- Localization best practices: https://developer.gnome.org/documentation/guidelines/localization/practices.html
- Accessibility coding guidelines: https://developer.gnome.org/documentation/guidelines/accessibility/coding-guidelines.html
- Libadwaita docs: https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1-latest/
- Damned Lies "Add module" template: https://gitlab.gnome.org/Infrastructure/damned-lies/-/blob/main/.gitlab/issue_templates/Add%20module.md

AppStream:

- Metadata spec: https://www.freedesktop.org/software/appstream/docs/chap-Metadata.html

---

## 0. Status and AI policy

These come first because they affect everything else.

### Circle submissions are closed

- The Circle README and the app issue template both say: "Submissions are currently closed. … Newly created issues will be closed immediately until submissions are open again." Both were changed on 2026-05-29 in the commit "Close for new submissions for now" (https://gitlab.gnome.org/Teams/Circle/-/blob/main/README.md, https://gitlab.gnome.org/Teams/Circle/-/blob/main/.gitlab/issue_templates/app_application.md).
- The announcement says the committee paused submissions to work through a backlog, some of it years old, and will reopen "when we have made a considerable dent in our backlog". It gives no date. It also says issues waiting on maintainer feedback are now closed and the maintainer reopens them on reply (https://blogs.gnome.org/sophieh/2026/05/29/updates-from-the-circle-committee/, summarised; not quoted in full).
- On 2026-10-20 the committee opened issues to review the App Criteria, the README and the review procedure "before opening submissions again" (https://gitlab.gnome.org/Teams/Circle/-/issues/281). Ideas raised there, none of them adopted yet:
  - Make "Builds Flatpak in Builder out of the box" a hard requirement.
  - Require "Released on Flathub" before submitting.
  - Make "Verified on Flathub" an inclusion requirement. Two committee members agreed on this.
  - Write objective criteria for apps that are a poor fit, such as "specialized apps with a very small target audience".

  Expect the criteria below to change before submissions reopen.

### Circle AI policy (added 2026-05-29)

The policy was added to AppCriteria.md in commit `8f459d9d9f`, "AppCriteria: Add AI policy for Circle", dated 2026-05-29 (https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md). It is a Circle App Criterion. Verbatim:

> Follow's Circle's AI policy: While it is not prohibited to use AI as a learning aid or a development tool (i.e. code completions), app developers should be able to justify and explain the code they submit, within reason. Submissions with large amounts of unnecessary code, inconsistent code style, imaginary API usage, comments serving as LLM prompts, or other indications of AI-generated output will be rejected.

- The wording copies the GNOME Shell extensions rule titled "Extensions must not be AI-generated" (https://gjs.guide/extensions/review-guidelines/review-guidelines.html).
- The announcement says Circle adopted the Shell extensions policy as a precaution against low-quality machine-generated submissions. For now it applies only to new submissions and is open to feedback. In the maintainer survey it reports, about 62% don't use LLMs, 34% use them for small questions or snippets, and 3% take larger portions of code from them. It also notes that Flathub has tightened its own AI policy, which reaches Circle apps indirectly because they ship on Flathub (https://blogs.gnome.org/sophieh/2026/05/29/updates-from-the-circle-committee/, summarised).
- The policy covers how the code is written, not what the app does. No Circle source forbids apps that have AI/LLM features. The only content rules are the Code of Conduct and the ban on promoting cryptocurrency (https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md).

### Flathub Generative AI policy

The "Generative AI policy" section is in https://docs.flathub.org/docs/for-app-authors/requirements#generative-ai-policy. It was added in a detailed form on 2026-01-24 (commit `2f1a16e`). The ban on AI content in manifests was restored on 2026-09-21 (commit `f740608`). Verbatim excerpts:

> Submitters must disclose any AI-generated code, documentation, packaging, or other material they know or reasonably believe is included in the application or its Flathub packaging. The disclosure must identify the affected parts and approximate extent.

> Flathub manifests must not contain AI-generated or AI-assisted content. Disclosure does not exempt manifests from this restriction.

> AI used only for research, discussion, or debugging does not need disclosure when no generated material is included … Reviewers may reject a submission, including without further review, based on the extent or role of generated material … Disclosure does not create a presumption of acceptance.

> AI tools or agents must not open or automate Flathub submission pull requests, or generate their commit messages, descriptions, review comments, or replies. Submitters must not request AI-agent reviews.

> Undisclosed or materially misrepresented AI-generated material … may result in rejection. Repeated violations may result in a permanent ban from future submissions and activities.

Related rules:

- Pull requests "that … contain excessive AI-generated content may be closed without a review" (https://docs.flathub.org/docs/for-app-authors/submission).
- Linter exception requests: "Please do not use LLMs in any way to handle exceptions PRs" (https://docs.flathub.org/docs/for-app-authors/linter).
- Translations: "Low quality, automated or machine-generated or mixed translations cannot be used" (https://docs.flathub.org/docs/for-app-authors/requirements#localisation-policy).

### How Circle has treated LLM apps

- **No current Circle app is an LLM chat or agent app.** `data/apps.json` lists 72 apps (fetched 2026-11-21, https://gitlab.gnome.org/Teams/Circle/-/blob/main/data/apps.json). None of their IDs or names is an LLM client. I judged this by name and did not audit every app's features.
- **Alpaca (Ollama/LLM chat), issue #230, opened 2024-05-24 and closed 2026-05-29, not accepted** (https://gitlab.gnome.org/Teams/Circle/-/issues/230):
  - The reviewer (`bertob`) treated it like any other app and gave many rounds of HIG feedback from 2024-06 to 2025-12, summarised in §3.
  - In 2024-09 the reviewer reported they could not find a cross-reviewer ("everyone's busy"). On 2025-01-14 they said the same again.
  - On 2025-12-14 the reviewer said onboarding had become "much more difficult to get started with a local model". On 2025-12-18 they wrote that a Flatpak extension "seems fine for something optional, but given that it's required for the app's main feature I don't think it's appropriate here."
  - On 2026-05-29 the committee closed it with the standard "awaiting feedback" message. It was closed for inactivity during the backlog cleanup, not rejected for being an AI app.
- **Gtk LLM Chat, issue #249, opened 2025-06-06 as WIP**: there was no review. It was closed on 2025-12-09 after six months without activity (https://gitlab.gnome.org/Teams/Circle/-/issues/249).
- Searching the Circle issues for "AI policy", "generated", "artificial" and "vibe" found no rejections that cite the AI policy so far (searched 2026-11-21). Submissions have been closed since the policy was added.

---

## 1. Circle eligibility, criteria and process

### Who can apply

Apps "must have a good overall quality level and use an OSI-approved license". The detailed criteria live in AppCriteria.md (https://gitlab.gnome.org/Teams/Circle/-/blob/main/README.md). Circle apps must meet both the General App Criteria and the Circle App Criteria (https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md).

### General App Criteria (required)

Source for this whole list: https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md

- **Quality:**
  - Basic features work.
  - "Easy to get started on first run".
  - User reviews are generally positive.
  - "No known issues that can lead to data loss".
- **Maintenance:**
  - Recent development activity.
  - Confidence that the maintainers can keep the app maintained.
  - "Being careful about inflicting harm like data loss on users".
  - At minimum, a supported Flathub runtime.
- **Technology:** GTK 4 + Libadwaita, and the GNOME runtime on Flathub.
- **Design:**
  - Follows the HIG.
  - Dark mode works: contrast is kept everywhere, and there are "No white surfaces".
- **Content:**
  - Does not contradict the GNOME Code of Conduct.
  - Does not promote cryptocurrencies.
- **Legal:**
  - OSI-approved license.
  - No CLA.
  - "The software must work without installing proprietary software".
- **Metadata:**
  - `appstreamcli validate` passes.
  - Name follows the HIG app naming guidelines.
  - Icon follows the HIG app icon guidelines.
  - Screenshot quality and summary quality.
  - An up-to-date `.doap` file.
  - Hardware support information.
  - Content rating information.
- **Accessibility:**
  - Every element can be reached and used with the keyboard.
  - Standard keyboard shortcuts.
  - High-contrast mode works.
  - Large text mode works.
  - Sufficient contrast.
  - Works with Orca. Image buttons have tooltips.
- **Repository:**
  - The default branch is named `main`. This rule was added 2026-01-14.
  - CI builds pass.
  - Public issue tracker and code repository. The repository "does *not* need to be hosted on GNOME's GitLab instance".
  - Contributors are told the GNOME Code of Conduct applies, for example in the README.
- **Recommended, not required:**
  - Builds Flatpak in GNOME Builder out of the box.
  - Build system is Meson, Automake or CMake.

### Circle App Criteria (additional)

Source for this whole list: https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md

- **Flathub:** available on Flathub for both x86_64 and aarch64.
- **No GNOME branding:** the app may not use "GNOME" in its name or author field, and may not present itself as official GNOME software. After inclusion it may say it is "part of GNOME Circle".
- **Valid app ID.** One of:
  - `<public-suffix>.<something>.<app>`, where the developer controls `https://<something>.<public-suffix>/.well-known/org.flathub.VerifiedApps.txt`.
  - `com.github.<user>.<app>`, `io.github.<user>.<app>`, `com.gitlab.<user>.<app>` or `io.gitlab.<user>.<app>`.
  - An ID on the legacy list.

  Other ID rules:
  - `org.gnome.<app>` is not allowed for new apps.
  - IDs must follow the Flatpak rules, for example replacing `-` with `_`.
  - Flathub reserves `com.github.` for GitHub's own projects, so Budha in practice needs `io.github.`. See §2.
- **Internationalization:** the app supports i18n and is connected to localization infrastructure. This is ignored for new apps and only applied "beginning with first regular review".
- **AI policy:** see §0.

### Pre-review checklist in the issue template

Source for this list: https://gitlab.gnome.org/Teams/Circle/-/blob/main/.gitlab/issue_templates/app_application.md

- Test every part of the app with keyboard only, Orca (`Alt+Super+S`), dark mode, high contrast and large text.
- Check these boxes:
  - Every part of the UI can be used with the keyboard alone.
  - Focus indicators are visible and polished.
  - Every icon-only button has a tooltip.
  - Orca reads everything usefully.
  - Standard shortcuts are present, including `Ctrl+Q`, `Ctrl+W` (both must work in single-window apps), `Ctrl+,` for preferences, `F10` for the primary menu and `Ctrl+?` for keyboard shortcuts.
  - CI is set up.
  - A `.doap` file lists the maintainers.
  - The README mentions the GNOME Code of Conduct.
- Confirm a checkbox that the project is willing to follow the GNOME Code of Conduct and to assist its Code of Conduct Committee.
- The committee closes issues whose pre-review checklist is incomplete. Example: GCodeKit5 was closed one day after it was opened (https://gitlab.gnome.org/Teams/Circle/-/issues/263).

### Review process

Source for this list: https://gitlab.gnome.org/Teams/Circle/-/blob/main/review_procedure.md

- The review happens in the application issue.
- The reviewer tests "the latest development version of the app built from source". If they can't build it, the applicant must fix the build.
- Three criteria are "particularly wide in scope" and decided "at the sole discretion of the reviewer":
  - Basic features work.
  - Easy to get started on first run.
  - Follows the HIG.
- After the reviewer approves, a second committee member does an independent **cross-review**.
- Inclusion means an entry is added to `data/apps.json`.
- After inclusion, every stable release must keep meeting the criteria. Problems are handled through:
  - issues filed in the app's repository,
  - **control reviews** with a deadline, after which an app that hasn't passed is excluded,
  - **group notices**. Example: apps had to upgrade from the GNOME 49 runtime by 2026-10-10 (https://gitlab.gnome.org/Teams/Circle/-/issues/280).

### Membership expectations

Source for this list: https://gitlab.gnome.org/Teams/Circle/-/blob/main/membership_guide.md

- Keep checking HIG compliance, test accessibility, and keep metadata up to date.
- Join `#circle:gnome.org`.
- Ask `#design:gnome.org` before making significant changes.
- Tell the committee before changing the app ID.
- Find co-maintainers.
- Notify the committee if the app becomes unmaintained.

### Common reasons for rejection or delay seen in reviews

- **Incomplete pre-review checklist.** The issue is closed immediately (https://gitlab.gnome.org/Teams/Circle/-/issues/263).
- **No reply for 6 months, or waiting on maintainer feedback.** The issue is closed (https://gitlab.gnome.org/Teams/Circle/-/issues/249, https://gitlab.gnome.org/Teams/Circle/-/issues/230).
- **Off-style icon.** Designers offered to redesign the icons for Dev Toolbox and Alpaca (https://gitlab.gnome.org/Teams/Circle/-/issues/253, https://gitlab.gnome.org/Teams/Circle/-/issues/230).
- **Name.** "Dev Toolbox" was asked to find "a more elegant name" (https://gitlab.gnome.org/Teams/Circle/-/issues/253).
- **Non-adaptive layout, wrong menu placement, stray dividers** (https://gitlab.gnome.org/Teams/Circle/-/issues/253).
- **Core function depends on something outside the Flatpak, plus hard onboarding** (https://gitlab.gnome.org/Teams/Circle/-/issues/230).
- **Long committee backlog and no cross-reviewer available** (https://gitlab.gnome.org/Teams/Circle/-/issues/230, https://blogs.gnome.org/sophieh/2026/05/29/updates-from-the-circle-committee/).

### Benefits

Source for this list: https://gitlab.gnome.org/Teams/Circle/-/blob/main/membership_guide.md

- Listing on apps.gnome.org and welcome.gnome.org.
- Eligibility for GNOME Foundation membership.
- Mentoring through GSoC and Outreachy.
- Damned Lies translations.
- Badges.
- A Discourse tag.
- Featured in GNOME Software's Editor's Choice.

---

## 2. Flathub requirements that Circle depends on

### Inclusion policy (risky categories for Budha)

Source for this whole list: https://docs.flathub.org/docs/for-app-authors/requirements

- **Console software** is not accepted.
- **"Minimal submissions"**: an app that "exists as a thin wrapper or launcher around other tools will not be accepted."
- **"Host-dependent applications"**: apps that "rely on host components or complicated post installation setups for core functionality will not be accepted". Exceptions are case by case.
- **"Submissions incompatible or restricted by Flatpak sandbox"**: "development tools, file managers, terminals, IDEs with a broad scope are generally not well-suited for Flatpak". These are accepted only if submitted by upstream or if upstream has committed to supporting Flatpak.
- **"Software design and distribution policy"**: rejects apps that disable or bypass security mechanisms or ship "overly permissive configurations".
- **"Insufficient development history"**: apps need "a meaningful history of development", tagged releases, and must not have "existed for a very short period of time".
- **Trademarks**: "Any part of the application such as the name or icon must not violate any trademarks and must be distinct."
- **License files**: every module's license file must be installed to `$FLATPAK_DEST/share/licenses/$FLATPAK_ID`.
- **Builds**: no network access during the build; everything is built from source.
- **Runtime**: must be the latest version at submission time.
- **Releases**: stable only. New apps cannot go straight to beta.

### App ID

Source for this list: https://docs.flathub.org/docs/for-app-authors/requirements#application-id

- At least 3 and at most 5 components.
- Characters are `[A-Za-z0-9_]`. A dash is allowed only in the last component.
- Must not end in `.desktop`, `.app` or `.linux`.
- Must exactly match the `<id>` in the metainfo.
- Apps hosted on GitHub must use the `io.github.` prefix and have at least 4 components. `com.github.` is reserved for GitHub's official projects.
- The repository URL worked out from the ID must be reachable. For example, `io.github.example_foo.bar` maps to `https://github.com/example-foo/bar`.
- Renaming means resubmitting the app. Renames are accepted only conservatively.
- GNOME's own docs say: "Application identifiers are widely used and changing them later can cause problems", and recommend a `.Devel` suffix for development builds (https://developer.gnome.org/documentation/tutorials/application-id.html).

For Budha this means something like `io.github.<user>.Budha`, with the GitHub repository named `Budha` under that user.

### Permissions and running host commands

- "Static permissions must be kept to an absolute minimum." When a suitable portal exists, using it "becomes mandatory instead of static permissions" (https://docs.flathub.org/docs/for-app-authors/requirements#permissions).
- Permissions are moderated separately from linting. If a pull request changes permissions, its description must explain why (https://docs.flathub.org/docs/for-app-authors/linter).
- `--talk-name=org.freedesktop.Flatpak` (`flatpak-spawn --host`) raises the linter warning `finish-args-flatpak-spawn-access`: "This access allows commands to run outside the sandbox. See the inclusion policy for details." (https://docs.flathub.org/docs/for-app-authors/linter#finish-args-flatpak-spawn-access). It is a warning, not an error, but it counts toward the "host-dependent" and "sandbox-incompatible" rules above.
- Precedent from Circle: Alpaca was told that a Flatpak extension required for the app's main feature was not appropriate (https://gitlab.gnome.org/Teams/Circle/-/issues/230, 2025-12-18). No Flathub or Circle source in this list gives a specific ruling on agent apps that run shell commands.

### MetaInfo (AppStream) requirements

Required for validation:

- The file must pass `flatpak-builder-lint … appstream`, which wraps `appstreamcli validate`. "Both errors and warnings are considered to be fatal" (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/).
- There must be exactly one `<developer id="…"><name>` (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/).
- A `<launchable type="desktop-id">` is required (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/).
- An OARS `<content_rating type="oars-1.1">` is required. "Applications must be properly tagged by OARS data based on their content" (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/). The AppStream spec itself only names `oars-1.0` as natively supported (https://www.freedesktop.org/software/appstream/docs/chap-Metadata.html). Follow Flathub and use 1.1.
- A `<releases>` tag is required. Dates must not be in the future, and versions must be properly ordered (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/).
- Screenshots: at least one is required for graphical apps. Image links must point at a tag or commit, not a branch (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/).
- Device support: `<supports>` or `<requires>` with `<control>`, and optionally `<display_length>` (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/, https://www.freedesktop.org/software/appstream/docs/chap-Metadata.html). Circle requires hardware support information (https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md).
- Translations: `<translation type="gettext">domain</translation>` (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/).
- The file is installed as `/usr/share/metainfo/%{id}.metainfo.xml` (https://www.freedesktop.org/software/appstream/docs/chap-Metadata.html).

Quality guidelines (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/quality-guidelines). Apps must pass these to be featured. Circle checks screenshot and summary quality against GNOME's own pages (https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md).

- **Brand colors:** set both light and dark `<color type="primary" scheme_preference=…>`. They should be colorful, different from each other, and contrast with the icon.
- **Icon:**
  - SVG, or PNG of at least 256 px, square.
  - Reasonable footprint and good contrast.
  - No baked-in shadows.
  - In line with contemporary styles, such as the GNOME icon style.
- **Name:** ideally 15 characters or fewer and always under 20. Just the name, with no tagline.
- **Summary:**
  - Ideally 10–25 characters, at most 35.
  - Non-technical: no toolkit or language, and no generic words like "app" or "client".
  - Sentence case, no trailing period.
  - Don't repeat the name or start with an article.
- **Description:** 3–6 lines, and lists no longer than 10 items.
- **Screenshots:**
  - 3–6 for a medium-sized app, tagged by language.
  - Just the app window, including its shadow and rounded corners, taken on Linux with default settings.
  - 1000×700 or smaller.
  - One-sentence captions.
  - Chat apps "should not show empty states or ugly content".
- **Release notes:** for every release, 2–3 sentences, and not "bug fixes and performance improvements".
- **Age rating:** account for content the user can reach without searching.
- **Verification:** use Flathub verification to show the app is managed by its author.

---

## 3. HIG essentials for Budha

The HIG has no chat-specific pattern. The chat-specific guidance below comes from the Alpaca review (https://gitlab.gnome.org/Teams/Circle/-/issues/230), which is the closest precedent.

### App naming

Source: https://developer.gnome.org/hig/guidelines/app-naming.html

Aim for:

- one or two simple nouns, related to the app's domain,
- under 15 characters,
- easy to pronounce,
- easy to pair with a good icon,
- header capitalization.

Avoid:

- trademarks or names of other projects,
- a "G" prefix,
- acronyms,
- puns and inside jokes,
- non-standard punctuation,
- made-up words or word combinations.

Check whether the name is already taken.

### App icons

Source: https://developer.gnome.org/hig/guidelines/app-icons.html

- "App icons should be unique". Each needs a simple metaphor related to the name or function. Avoid "characters or mascots" and logos.
- The style is simple and geometric. It is drawn on a 128×128 template with a baseline, and has a darker front "profile" for depth.
- Use the GNOME palette, follow the 2 px grid, keep details legible at 32 px, and avoid shadows.
- A symbolic version of the icon is also required.
- Design App Icon Preview is the recommended tool.

Precedent: committee designers have offered to redesign off-style icons. Requests go through https://gitlab.gnome.org/Teams/Design/app-icon-requests (referenced in https://gitlab.gnome.org/Teams/Circle/-/issues/230 and https://gitlab.gnome.org/Teams/Circle/-/issues/253).

### Adaptive layout and breakpoints

- Support 1024×600 at minimum. Phone-suitable apps should scale down to 360×294 (https://developer.gnome.org/hig/guidelines/adaptive.html).
- Design for the smallest size first.
- Prefer list patterns.
- Use max-width containers to keep text lines from getting too long (https://developer.gnome.org/hig/guidelines/adaptive.html).
- Libadwaita breakpoints (https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1-latest/adaptive-layouts.html):
  - Collapse a split view at `max-width: 400sp`.
  - In a triple-pane layout, collapse at `860sp` and `500sp`.
  - Move the view switcher to the bottom bar at `550sp`.
  - Use `AdwClamp` for maximum widths.
  - `AdwDialog` becomes a bottom sheet in narrow windows.
- A reviewer flagged "very far away from being adaptive" as a problem (https://gitlab.gnome.org/Teams/Circle/-/issues/253).

### Navigation: sidebar of conversations

- Sidebars suit "dynamic locations, such as in a messaging app". Sort them so recently updated items are at the top. Controls that affect the list go above it. Use `AdwNavigationSplitView` (https://developer.gnome.org/hig/patterns/nav/sidebars.html).
- Utility panes, such as a tool or activity log, sit beside the main view. Use `AdwOverlaySplitView` so the pane overlaps the content when the window is narrow. If the pane can be toggled, bind it to `F9` (https://developer.gnome.org/hig/patterns/containers/utility-panes.html).
- Keep hierarchies to one level deep (https://developer.gnome.org/hig/guidelines/navigation.html).
- From the Alpaca review (https://gitlab.gnome.org/Teams/Circle/-/issues/230):
  - Sort chats by most recently used.
  - Reopen the last-used chat on start.
  - Generate chat titles automatically.
  - Use a right-click context menu, not actions that appear on hover.
  - Put destructive items such as Delete Chat at the bottom of menus, below a divider.
  - Use the standard `AdwNavigationView` sub-pages for drill-down.

### Header bars and menus

- **Header bars** (https://developer.gnome.org/hig/patterns/containers/header-bars.html):
  - Keep the number of controls small.
  - Update controls when the view changes.
  - Leave space for dragging.
  - Give every header bar control a tooltip.
  - Avoid label-only, suggested or destructive buttons in the primary window's header bar.
- **Primary menu** (https://developer.gnome.org/hig/patterns/controls/menus.html):
  - Uses `open-menu-symbolic`. With a sidebar, it sits above the sidebar list.
  - Tooltip and accessible label: "Main Menu".
  - Ends with Preferences, Keyboard Shortcuts, Help (if any) and "About Budha".
  - No Quit or Close items.
- **Secondary menu**: for per-conversation actions. It uses `view-more-symbolic` (https://gitlab.gnome.org/Teams/Circle/-/issues/230) and does not repeat primary menu items (https://developer.gnome.org/hig/patterns/controls/menus.html).
- Alpaca was told to keep chat actions in a secondary menu, not the primary one: "The menu on the left side should only contain global app-level stuff" (https://gitlab.gnome.org/Teams/Circle/-/issues/230).

### Preferences, About and Shortcuts dialogs

- About and Preferences are secondary windows. They are modal, close with `Esc`, and windows close with `Ctrl+W` (https://developer.gnome.org/hig/patterns/containers/windows.html).
- Use `AdwPreferencesDialog` (libadwaita 1.5+, searchable) (https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1-latest/class.PreferencesDialog.html).
- Use `AdwAboutDialog`. It can be built from AppStream data (`new_from_appdata`), and it has "What's New", Troubleshooting/debug info, credits including `translator-credits`, and legal pages (https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1-latest/class.AboutDialog.html).
- Use `AdwShortcutsDialog` (libadwaita 1.8+) for the `Ctrl+?` dialog (https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1-latest/class.ShortcutsDialog.html).
- From the Alpaca review: on preference pages, drop unnecessary group titles and subtitles, use Title Case for row titles, and don't end subtitles with a period (https://gitlab.gnome.org/Teams/Circle/-/issues/230).
- Put donation links in About or the primary menu, not in an always-visible toolbar (https://gitlab.gnome.org/Teams/Circle/-/issues/230).

### Keyboard shortcuts

Standard shortcuts (https://developer.gnome.org/hig/reference/keyboard.html):

| Shortcut | Action |
|---|---|
| `Ctrl+W` | Close window or tab |
| `Ctrl+Q` | Quit |
| `F1` | Help |
| `F9` | Side pane |
| `F10` | Menu |
| `Ctrl+?` | Shortcuts dialog |
| `Ctrl+,` | Preferences |
| `Ctrl+N` | New (for example, new chat) |
| `Ctrl+F` | Find |
| `Ctrl+Z` / `Shift+Ctrl+Z` | Undo / Redo |

Rules for app-specific shortcuts (https://developer.gnome.org/hig/guidelines/keyboard.html):

- Use Ctrl plus a letter.
- Don't use Alt (it conflicts with access keys) or Super.
- Prefer shortcuts that can be pressed with one hand.
- Every labelled control and menu item gets an access key.
- Focus order must be logical.
- `Esc` closes transient UI.

### Dialogs and confirming destructive actions

Source: https://developer.gnome.org/hig/patterns/feedback/dialogs.html

- "Destructive actions should always be accompanied by either a confirmation dialog or an offer to undo the action (typically using a toast)." Undo is preferred because people click through confirmations by habit. Where undo isn't possible, a confirmation dialog is still recommended.
- Cancel comes first.
- Don't bind Return to an affirmative button that is irreversible or destructive.
- `Esc` cancels.
- Label buttons with specific verbs.
- Never pop up a dialog unexpectedly.
- Use `AdwAlertDialog`.

Precedent: Alpaca was asked to phrase a confirmation title as a question, "Stop Download?" (https://gitlab.gnome.org/Teams/Circle/-/issues/230).

For an agent app this applies to confirming shell commands or tool calls that delete or overwrite data. That is my reading; no source mentions agent tool approval.

### Toasts, banners, notifications and placeholders

- **Toasts** are for single transient events, often with Undo. They sit at the bottom centre, and their titles use the informal heading style (https://developer.gnome.org/hig/patterns/feedback/toasts.html).
- **Banners** are for persistent states, such as "Working offline". They are not for events (https://developer.gnome.org/hig/patterns/feedback/banners.html).
- **Notifications** are for events the user should see while using other apps, for example "when a long-running operation has finished" (https://developer.gnome.org/hig/patterns/feedback/notifications.html):
  - Limit how often they appear.
  - Let users turn them off.
  - Remove stale ones.
  - Don't rely on notifications alone.
  - Alpaca was asked to use the chat title as the notification title and the start of the reply as the body (https://gitlab.gnome.org/Teams/Circle/-/issues/230).
- **Placeholder pages** cover empty states, such as no conversations or no models configured. Give them a heading, guidance and an action button (https://developer.gnome.org/hig/patterns/feedback/placeholders.html).
- Alpaca was told to show an inline error in the view for failures like running out of memory, "instead of a transient toast" (https://gitlab.gnome.org/Teams/Circle/-/issues/230).

### Long-running operations

- Indicate progress for anything over about 3 seconds.
- Spinners suit operations under about 30 seconds. Place them near the element they relate to, show only one at a time, and add a label for long waits (https://developer.gnome.org/hig/patterns/feedback/spinners.html).
- Progress bars suit tasks over 30 seconds. Add pause or cancel when the task is destructive or resource-intensive. Avoid progress windows (https://developer.gnome.org/hig/patterns/feedback/progress-bars.html).
- Alpaca was told the app was "very laggy" while a model was responding (https://gitlab.gnome.org/Teams/Circle/-/issues/230). Streaming output must not block the UI.

### Chat layout (from the Alpaca review)

Source: https://gitlab.gnome.org/Teams/Circle/-/issues/230

- Style prompts and responses consistently. Either use bubbles for both, or distinguish them in another way.
- Show human-readable model names, not raw IDs.
- Give the prompt entry standard entry styling, not `.card` styling.
- Message widgets need a visible focus ring with padding. Orca must read the whole message and say whether it came from the user or the assistant.
- Don't build a custom zoom feature. Rely on the system's scaling and large text settings.

### Writing style

Source: https://developer.gnome.org/hig/guidelines/writing-style.html

- Keep text short and neutral. Avoid "you" and "my". "Your" is acceptable.
- Use the user's terminology, not the system's.
- Don't build sentences across several controls.
- No "e.g." or "i.e.".
- **Header capitalization** for headings, buttons, menu items and tooltips.
- **Sentence capitalization** for body text and labels.
- Use an ellipsis only when more input is required, for example "Delete…".
- Generally no trailing periods.
- Tooltips are short and header-capitalized, for example "Grid View", not "Switch to grid view". If one control in a container has a tooltip, all of them should (https://developer.gnome.org/hig/patterns/feedback/tooltips.html).
- Many of the Alpaca review items were capitalization fixes (https://gitlab.gnome.org/Teams/Circle/-/issues/230).

---

## 4. Internationalization and accessibility

### i18n

- **Circle:** i18n plus a connection to localization infrastructure is required from the first regular review onward, not at submission (https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md).
- **Flathub:** a complete English localisation is required, and machine-generated translations are forbidden (https://docs.flathub.org/docs/for-app-authors/requirements#localisation-policy). Declare the gettext domain with `<translation type="gettext">` (https://docs.flathub.org/docs/for-app-authors/metainfo-guidelines/).
- **Writing strings** (https://developer.gnome.org/documentation/guidelines/localization/practices.html):
  - Write in US English.
  - Use consistent terms.
  - No hard-coded line breaks.
  - Add `Translators:` comments above ambiguous strings. They are extracted into `.po` files.
  - Use only UTF-8 or ASCII.
  - Don't mark empty strings as translatable.
- **Damned Lies (l10n.gnome.org):** Circle membership lets an app enrol (https://gitlab.gnome.org/Teams/Circle/-/blob/main/membership_guide.md). The "Add module" template (https://gitlab.gnome.org/Infrastructure/damned-lies/-/blob/main/.gitlab/issue_templates/Add%20module.md) requires:
  - granting commit access to `@translations`, or `@gnometranslations` on GitHub,
  - a `.doap` file that meets the requirements,
  - strings that follow the HIG typography rules,
  - developers who "never touch po files",
  - translations that "must only come from Damned Lies", with no translation merge requests accepted,
  - announcing releases on Discourse about two weeks ahead.

  Apps outside Circle use a separate request path at Teams/Translation/Coordination (linked from that template).

### Accessibility

- **Required tests**:
  - keyboard only,
  - Orca (`Alt+Super+S`),
  - dark mode,
  - high contrast,
  - large text.

  The template also asks for visible focus indicators, a tooltip on every icon-only button, and useful Orca output everywhere (https://gitlab.gnome.org/Teams/Circle/-/blob/main/.gitlab/issue_templates/app_application.md, https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md). The HIG adds on-screen keyboard testing and asks whether the app is usable with the display off (https://developer.gnome.org/hig/guidelines/accessibility.html).
- **Accessible names**: all elements need short, descriptive accessible names (https://developer.gnome.org/hig/guidelines/accessibility.html).
- **GTK4 API** (https://developer.gnome.org/documentation/guidelines/accessibility/coding-guidelines.html):
  - `GTK_ACCESSIBLE_PROPERTY_LABEL` for image-only controls.
  - `GTK_ACCESSIBLE_PROPERTY_DESCRIPTION` where there is no tooltip.
  - `GTK_ACCESSIBLE_RELATION_LABELLED_BY` to link labels to controls.
  - Custom widgets must implement accessibility, and inaccessible containers make their children inaccessible.
- **Small-scale testing**: for very complex UIs, a reviewer accepted a smoke test of one or two standard workflows "with your eyes closed and only keyboard + screen reader" (https://gitlab.gnome.org/Teams/Circle/-/issues/263).
- **Chat messages**: each message needs a focus ring and must be read with its role, user or assistant (https://gitlab.gnome.org/Teams/Circle/-/issues/230). Streaming text needs particular care with Orca.

---

## 5. Checklist for Budha

Status and policy:

- [ ] Watch for Circle submissions reopening, and check whether the App Criteria changed (https://gitlab.gnome.org/Teams/Circle/-/issues/281).
- [ ] Every maintainer can explain every line of code. Remove LLM-prompt comments, dead code and invented APIs, and keep one code style. This is the Circle AI policy.
- [ ] When submitting to Flathub, disclose the parts and approximate extent of any AI-generated code, docs or packaging.
- [ ] Write the Flathub manifest by hand, with no AI assistance.
- [ ] A human opens the Flathub pull request and writes all its text and replies. No AI-generated commit messages or replies on that pull request.
- [ ] No machine-generated translations.

Legal, repository and project:

- [ ] OSI license and no CLA. The repository already has GPL-3.0 in `COPYING`.
- [ ] Works without proprietary software. At least one fully open model or provider path must work end to end.
- [ ] Public repository and issue tracker. Default branch is `main`. CI passes.
- [ ] Builds in GNOME Builder out of the box. Use Meson.
- [ ] A `.doap` file with maintainers, including a GNOME GitLab account (the announcement says runtime reminder issues go to accounts listed in the `.doap`).
- [ ] The README links the GNOME Code of Conduct.
- [ ] No "GNOME" in the name or author field.
- [ ] A history of tagged releases before submitting to Flathub.

Flathub:

- [ ] App ID `io.github.<user>.Budha`, matching `https://github.com/<user>/Budha` exactly. Choose it once. Use `io.github.<user>.Budha.Devel` for development builds.
- [ ] Latest GNOME runtime. Builds for x86_64 and aarch64. Built from source with no network access during the build.
- [ ] Minimal `finish-args`, with portals wherever possible. Write a justification for any host access in the pull request.
- [ ] Flathub verification.
- [ ] Install every module's license file.

MetaInfo:

- [ ] `appstreamcli validate` and `flatpak-builder-lint appstream` pass with zero warnings.
- [ ] Name ≤15 characters.
- [ ] Summary 10–35 characters, imperative, non-technical, no trailing period.
- [ ] Description 3–6 lines.
- [ ] `<developer id>`, `<launchable>`, `<content_rating type="oars-1.1">`, `<releases>` with real notes for every release, `<supports>`/`<requires>` controls and `display_length`, `<translation type="gettext">`, `vcs-browser` URL.
- [ ] Light and dark brand colors, chosen with the icon designer.
- [ ] 3–6 screenshots: window only, with shadow, default settings, ≤1000×700, real non-empty conversation content, one-sentence captions, image URLs pinned to a tag.

Design:

- [ ] A name that follows the HIG. "Budha" is a made-up spelling, so check it against the "made-up words" rule and run a trademark search.
- [ ] A GNOME-style app icon plus a symbolic version. Ask Teams/Design/app-icon-requests for help.
- [ ] `AdwNavigationSplitView` with a conversation sidebar sorted by most recent use. Reopen the last chat. Generate titles automatically. Context menus for chat actions, with Delete last.
- [ ] Breakpoints (`max-width: 400sp` collapse) and `AdwClamp` for the message column. Works at 360×294 if Budha targets phones, and at 1024×600 at minimum.
- [ ] Primary menu above the sidebar ("Main Menu"): Preferences, Keyboard Shortcuts, About Budha. A secondary menu (`view-more-symbolic`) for per-chat actions.
- [ ] `AdwPreferencesDialog`, `AdwAboutDialog` (from appdata, with troubleshooting info and translator credits) and `AdwShortcutsDialog`.
- [ ] Shortcuts `Ctrl+Q`, `Ctrl+W`, `Ctrl+,`, `F10`, `Ctrl+?` and `Ctrl+N`, plus `F9` if there is a toggleable tool/activity pane. No Alt or Super shortcuts.
- [ ] Undo toasts or `AdwAlertDialog` confirmations for destructive actions. Destructive buttons are never the Return default.
- [ ] Banners for persistent states (offline, agent not connected). Toasts for single events. Inline error views for failures that block the main task. Placeholder pages for empty states. Notifications when a long agent run finishes while the window is unfocused.
- [ ] Spinner or progress for operations over 3 seconds, with cancel for long or destructive runs. Streaming never blocks the main loop.
- [ ] Dark mode with no white surfaces, high contrast, large text.
- [ ] Header and sentence capitalization, tooltips on all icon buttons, no "e.g."/"i.e.", no trailing periods.

Accessibility and i18n:

- [ ] Keyboard-only pass over every view. Visible focus rings on messages.
- [ ] Orca reads each message with its role (user or agent), plus tool output and status changes.
- [ ] Accessible labels on every image-only control.
- [ ] Test with the on-screen keyboard.
- [ ] gettext through Meson i18n. `Translators:` comments. All UI, desktop file and metainfo strings marked translatable.
- [ ] After acceptance, enrol in Damned Lies and stop accepting translation merge requests.

Onboarding:

- [ ] First run that is easy to get started with: Hermes Agent and a model provider can be set up in a few steps from inside the app, with no manual host setup and no required Flatpak extension.

## Open questions and risks

1. **Circle is closed, with no reopening date.** The criteria are being revised (https://gitlab.gnome.org/Teams/Circle/-/issues/281), and the backlog is long. Alpaca waited about 2 years without reaching cross-review. Plan for a long wait.

2. **The AI policies, which conflict with an AI-assisted workflow.** This is the most likely blocker if Budha is built with an AI coding agent.
   - Circle rejects anything with "indications of AI-generated output".
   - Flathub requires disclosure, forbids AI-assisted manifests, and forbids AI agents from opening or handling the submission pull request. Violations can lead to a ban.
   - Open: does Budha's development process (for example, Claude Code writing code) produce code the maintainer can fully justify, in a consistent style and without leftover prompt comments? Decide on a development and disclosure practice early, and record it in an ADR.
   - The Circle announcement says the policy is a starting point and open to feedback, so it may change.

3. **Host command execution and the Flatpak sandbox.** Hermes Agent runs shell commands and tools.
   - Running them on the host needs `org.freedesktop.Flatpak` (`flatpak-spawn --host`). That triggers a linter warning and permission moderation, and conflicts with the "Host-dependent applications" and "overly permissive configurations" rules.
   - If Hermes Agent has to be installed on the host, Budha risks rejection as a "thin wrapper or launcher around other tools" or as host-dependent.
   - Open questions: can Hermes Agent be bundled in the Flatpak and run inside the sandbox, with the user granting folder access through the file chooser portal? Is a sandbox-only mode useful enough? Would Flathub accept host spawning for an agent app?
   - No source found gives a ruling for agent apps. Ask in `#flathub:matrix.org` and `#circle:gnome.org` before building on this assumption.

4. **Proprietary dependency.** "The software must work without installing proprietary software."
   - Open: does Hermes Agent work fully with open models or local providers? What is its license?
   - Not verified here. Check the upstream repository.

5. **Onboarding friction.** "Easy to get started on first run" is up to the reviewer's judgement. Alpaca slipped on exactly this, for model setup.
   - Open: what is the shortest path from first launch to a working agent? It probably involves an API key or a local model download.

6. **OARS rating for LLM output.** OARS has no category for generated text. Flathub requires an "honest" rating based on content the user can easily reach.
   - Open: which OARS fields apply to an LLM that can produce any text, and does Budha need any content controls? Not covered by any source read.

7. **Naming.** "Budha" may fail the HIG rules against "made-up words" and on being easy to pronounce. It may also be confused with "Buddha" and raise cultural or religious sensitivity. Reviewers have asked apps to rename before (Dev Toolbox). Run a trademark and name check before choosing the app ID, because renaming requires resubmitting to Flathub.

8. **Icon design skill.** The GNOME icon style needs design work. Committee designers have helped other apps, but at their discretion.

9. **Runtime churn.** Circle sends group notices with deadlines, for example dropping GNOME 49 by 2026-10-10 (https://gitlab.gnome.org/Teams/Circle/-/issues/280). Budha has to keep up with every runtime release or be excluded.

10. **Possible new requirements.** Flathub release before submission, Flathub verification, and Builder-buildable Flatpak may all become mandatory (https://gitlab.gnome.org/Teams/Circle/-/issues/281).
