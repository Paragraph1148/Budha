# Budha

A GNOME app for Hermes Agent (Nous Research), written in Python with GTK4 and libadwaita. The goal is to be accepted into GNOME Circle, so every user-facing choice follows the GNOME HIG.

## Workflow

Work happens in this order. Each phase finishes before the next one starts:

1. **Research**: primary sources only, saved in `docs/research/`.
2. **Grilling**: scope questions answered by the user.
3. **Domain model**: terms go in `GLOSSARY.md`, decisions in `docs/adr/`.
4. **Spec**: `docs/spec/`, split into GitHub issues with version milestones.
5. **Prototype**: throwaway, deleted once the design is settled.
6. **Design**: deep modules, with the Hermes client behind one seam.
7. **Build**: one issue per branch, test-first. Review, then merge.

A decision not recorded in an ADR or the spec is still open. Ask the user, then record the answer.

## Conventions

- Use the words in `GLOSSARY.md` exactly, in code and in docs.
- Every user-visible change gets a line under `[Unreleased]` in `CHANGELOG.md`.
- Versions follow semver. Pre-1.0 minor versions may break things.
- Docs describe only what exists. A planned feature lives in the spec, not the README.

## AI-written code

Budha's code is written by Claude and disclosed on Flathub. The `Co-Authored-By: Claude` trailer on each commit is the record that disclosure draws from, so keep it on every agent-made commit.

- Write only the code the spec needs, in one consistent style, using APIs confirmed in the official docs or source. Comments explain why, for a human reader.
- The Flatpak manifest is the user's to write: Flathub bans AI-generated or AI-assisted manifest content. Flathub submission PRs, their text and replies are also the user's alone. Agents may point to docs for these, and stop there.
