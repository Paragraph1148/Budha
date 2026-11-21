# Building, packaging and testing a Python GTK4/libadwaita app (late 2026)

Date researched: 2026-11-21

Question: what is the current, recommended way to build, structure, package and test a Python + GTK4 + libadwaita app (PyGObject, Flatpak, aimed at GNOME Circle), and what does that mean for Budha?

Method: upstream repositories were cloned (shallow) on 2026-11-21 and read at the commits listed below. Runtime versions were also checked by running `python3` inside the installed `org.gnome.Platform//51` and `org.gnome.Sdk//50` runtimes on the dev machine. Web docs were read the same day. Labels like [PYGO] are used for inline citations.

## Sources

Upstream source (repo @ commit):

- [PYGO] PyGObject, https://gitlab.gnome.org/GNOME/pygobject @ `673a5df298cf` (2026-11-19). Files: `NEWS`, `gi/events.py`, `gi/overrides/Gio.py`, `gi/_ossighelper.py`, `tests/test_events.py`, `docs/guide/asynchronous.rst`, `docs/guide/threading.rst`, `docs/guide/testing.rst`, `docs/guide/flatpak.rst`, `docs/guide/gtk_template.rst`, `docs/getting_started.rst`.
- [ADW] libadwaita, https://gitlab.gnome.org/GNOME/libadwaita @ `39fff6661b28` (2026-11-07). File: `NEWS`.
- [ADWDOC] libadwaita API docs (main, documents 1.11.alpha), https://gnome.pages.gitlab.gnome.org/libadwaita/doc/main/
- [GTK] GTK, https://gitlab.gnome.org/GNOME/gtk @ `d3d719a16efd`. Files: `NEWS`, `docs/reference/gtk/running.md`, `meson.options`, `.gitlab-ci/run-tests.sh`.
- [GBM] gnome-build-meta branch `gnome-51`, https://gitlab.gnome.org/GNOME/gnome-build-meta @ `e21b7734ff1a`. Files: `elements/freedesktop-sdk.bst`, `elements/sdk-platform.bst`, `elements/sdk/{pygobject,gtk,libadwaita,blueprint-compiler}.bst`. Also `elements/sdk-platform.bst` on branches `gnome-48`, `gnome-49`, `gnome-50`.
- [FDO] freedesktop-sdk tag `freedesktop-sdk-26.08.4`, `elements/include/python3.yml`, https://gitlab.com/freedesktop-sdk/freedesktop-sdk
- [BUILDER] GNOME Builder, https://gitlab.gnome.org/GNOME/gnome-builder @ `d996e6c96aba`. Directory `src/plugins/meson-templates/` (Python template files) and `meson_options.txt`.
- [BLP] blueprint-compiler, https://gitlab.gnome.org/GNOME/blueprint-compiler @ `f7198c52f723`. Files: `NEWS.md`, `docs/index.rst`, `docs/setup.rst`.
- [SECRET] libsecret, https://gitlab.gnome.org/GNOME/libsecret @ `7ab0b42f5581`. Files: `README.md`, `libsecret/secret-backend.c`.
- [FLATPAK] flatpak, https://github.com/flatpak/flatpak @ `99e1af1d9115`. File: `data/org.freedesktop.Flatpak.xml`.
- [FBT] flatpak-builder-tools, https://github.com/flatpak/flatpak-builder-tools @ `9d86d336a39b`. File: `pip/readme.md`.
- [R2F] req2flatpak, https://github.com/johannesjh/req2flatpak @ `0357aec97ed3`. File: `README.rst`.
- [FGA] flatpak-github-actions, https://github.com/flatpak/flatpak-github-actions @ `79327416609a`. File: `README.md`.
- [CIT] GNOME citemplates, https://gitlab.gnome.org/GNOME/citemplates @ `3c2990f1d938`. Files: `flatpak/flatpak_ci_initiative.yml`, `runtime-images/flatpak-job/flatpak-job.sh`, `runtime-images/Dockerfile`.
- [STUBS] pygobject-stubs, https://github.com/pygobject/pygobject-stubs @ `16af78ff1ad0`. Files: `README.md`, `pyproject.toml`, `CHANGELOG.md`, `src/gi-stubs/repository/`.
- [CIRCLE] GNOME Circle, https://gitlab.gnome.org/Teams/Circle @ `228513a5d1c1`. File: `data/apps.json`.
- [CRIT] App Criteria, https://gitlab.gnome.org/Teams/Releng/AppOrganization/-/blob/main/AppCriteria.md @ `bd049d3cd775`.
- [HERMES] hermes-agent, https://github.com/NousResearch/hermes-agent @ `aeb5a071069d`. File: `pyproject.toml`.

Python GNOME Circle apps read as reference (all listed in [CIRCLE] `data/apps.json`; all verified Python by their source tree):

- [APOS] Apostrophe, https://gitlab.gnome.org/World/apostrophe @ `7fe8f1d66b46` (2026-11-21). Circle since 2020-07-28 [CIRCLE].
- [DIAL] Dialect, https://github.com/dialect-app/dialect @ `d237666bec2f` (2026-05-12). Circle since 2020-12-22 [CIRCLE].
- [SECR] Secrets, https://gitlab.gnome.org/World/secrets @ `369c3255c1cd` (2026-11-20). Circle since 2020-11-18 [CIRCLE].
- [WIKE] Wike, https://github.com/hugolabe/Wike @ `b9ebebcefebd` (2026-07-06). Circle since 2021-07-27 [CIRCLE].
- Eyedropper is in Circle [CIRCLE] but was not read; it was not verified as Python and is not used here.

Web documentation (read 2026-11-21):

- [PYGO-ASYNC] https://pygobject.gnome.org/guide/asynchronous.html
- [PYDOC-POLICY] https://docs.python.org/3.14/library/asyncio-policy.html
- [MESON-PY] https://mesonbuild.com/Python-module.html
- [MESON-GNOME] https://mesonbuild.com/Gnome-module.html
- [MESON-I18N] https://mesonbuild.com/i18n-module.html
- [GNOME-TUT] https://developer.gnome.org/documentation/tutorials/beginners/getting_started.html and https://developer.gnome.org/documentation/tutorials/beginners.html
- [BUILDER-INST] https://builder.readthedocs.io/installation.html
- [FP-PY] https://docs.flatpak.org/en/latest/python.html
- [FP-FIRST] https://docs.flatpak.org/en/latest/first-build.html
- [FP-PERM] https://docs.flatpak.org/en/latest/sandbox-permissions.html
- [FP-CMD] https://docs.flatpak.org/en/latest/flatpak-command-reference.html (flatpak-spawn section)
- [FH-REQ] https://docs.flathub.org/docs/for-app-authors/requirements
- [FH-SUB] https://docs.flathub.org/docs/for-app-authors/submission
- [FH-LINT] https://docs.flathub.org/docs/for-app-authors/linter
- [PORTAL-SECRET] https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.Secret.html
- [SOUP] https://libsoup.gnome.org/libsoup-3.0/method.Session.send_async.html
- [GSUB] https://docs.gtk.org/gio/class.Subprocess.html

Local checks (dev machine, Fedora 45, 2026-11-21):

- [LOCAL-RT] `flatpak run --command=python3 org.gnome.Platform//51 -c ...` printed Python 3.14.9, PyGObject 3.58.1, GTK 4.24.1, libadwaita 1.10.0, libsoup 3.8, GLib 2.90.1; `/usr/bin/blueprint-compiler` and `/usr/bin/gtk4-broadwayd` exist in the Platform; `pytest` is not installed. In `org.gnome.Sdk//50`: Python 3.13.18; `pip3`, `meson`, `blueprint-compiler` 0.20.4, `appstreamcli`, `desktop-file-validate`, `glib-compile-schemas` exist; `pytest`, `ruff`, `mypy` do not.
- [LOCAL-FH] `flatpak remote-ls flathub`: the system `flathub` remote is marked `filtered` but still lists `org.gnome.Platform`/`org.gnome.Sdk` 49, 50 and 51, `org.flatpak.Builder` and `org.gnome.Builder`. Installed already: `org.gnome.Platform` 50 and 51, `org.gnome.Sdk` 50.
- [LOCAL-RPM] `rpm -q`: python3 3.15.2, gtk4 4.24.1, libadwaita 1.10.0, python3-gobject 3.58.1 (Fedora 45 packages).

---

## 1. Current versions

### GNOME runtime / SDK

- The newest GNOME runtime on Flathub is 51; 49 and 50 are also published [LOCAL-FH].
- GNOME 51 is built on freedesktop-sdk 26.08 (`track: freedesktop-sdk-26.08*`, pinned `freedesktop-sdk-26.08.4`) [GBM `elements/freedesktop-sdk.bst`].
- Python in GNOME 51 is 3.14 (freedesktop-sdk tracks `v3.14.*`, pinned `v3.14.9`) [FDO `elements/include/python3.yml`]; confirmed 3.14.9 at runtime [LOCAL-RT]. GNOME 50 ships Python 3.13 [LOCAL-RT].
- Fedora 45's system Python is 3.15.2 [LOCAL-RPM], so a host venv and the Flatpak runtime run different Python versions. Code must work on 3.14.

### GTK 4

- GNOME 51 ships GTK 4.24.1 [GBM `elements/sdk/gtk.bst`, LOCAL-RT]. GTK 4.24.0 was released 2026-09-11; 4.25.x is the current development series (4.25.1 on 2026-11-19) [GTK `NEWS`].

### libadwaita

- GNOME 51 ships libadwaita 1.10.0 [GBM `elements/sdk/libadwaita.bst`, LOCAL-RT]. The `main` docs already describe 1.11.alpha [ADWDOC].
- Widgets added in recent releases, by "since" version in the API docs [ADWDOC]:
  - 1.6: `AdwBottomSheet`, `AdwButtonRow`, `AdwMultiLayoutView` (with `AdwLayout`, `AdwLayoutSlot`), `AdwSpinner`, `AdwSpinnerPaintable`.
  - 1.7: `AdwToggleGroup` / `AdwToggle`, `AdwInlineViewSwitcher`, `AdwWrapBox`, `AdwWrapLayout`.
  - 1.8: `AdwShortcutsDialog`, `AdwShortcutsSection`, `AdwShortcutsItem`, `AdwShortcutLabel`.
  - 1.9: `AdwSidebar`, `AdwSidebarItem`, `AdwSidebarSection`, `AdwViewSwitcherSidebar`, `AdwNoneAnimationTarget`.
- 1.9 also deprecated `style-dark.css`, `style-hc.css` and `style-hc-dark.css` in `AdwApplication` [ADW `NEWS`, "Version 1.9.alpha"].
- 1.10 added `AdwCssClassBinding` and deprecated `AdwEnumListModel` in favour of `GtkEnumList` [ADW `NEWS`, "Version 1.10.alpha"]. 1.10.beta.1 added `:prefix` to `AdwSidebarItem` and `:suffix` to `AdwSidebarSection` [ADW `NEWS`].

### PyGObject

- GNOME 51 ships PyGObject 3.58.1 [GBM `elements/sdk/pygobject.bst`, LOCAL-RT]; GNOME 50 shipped 3.56.3 [GBM branch `gnome-50`, `elements/sdk/pygobject.bst`].
- 3.58.0 was released 2026-08-27 [PYGO `NEWS`]. Unreleased 3.59.0 drops Python 3.10, requires GLib >= 2.84, and adds free-threading Python support [PYGO `NEWS`].
- 3.57.x added `Gio.File` `/` operator (like `pathlib`), `PathLike` for `Gtk.Template`, generic annotations on `Async`, and a fix for a memory leak in GTK templates [PYGO `NEWS`].

### Blueprint

- GNOME 51 ships blueprint-compiler 0.22.2 [GBM `elements/sdk/blueprint-compiler.bst`]. It has been in the GNOME runtime stack since GNOME 49 (`sdk/blueprint-compiler.bst` appears in `elements/sdk-platform.bst` on `gnome-49`, `gnome-50`, `gnome-51`, but not `gnome-48`) [GBM], and the binary is present in `org.gnome.Platform//51` [LOCAL-RT].

---

## 2. Project layout and build

### GNOME Builder's Python template

GNOME's beginner tutorial starts every app from Builder's "GNOME Application" template [GNOME-TUT]. Builder's Python + GTK4 template generates [BUILDER `src/plugins/meson-templates/gbp-meson-template-provider.c` lines 88-93]:

- `src/<name>.in`: launcher script with `#!@PYTHON@`. It sets up gettext, loads `<pkgdatadir>/<name>.gresource` with `Gio.Resource.load(...)._register()`, puts `pkgdatadir` on `sys.path`, then calls `main.main(VERSION)` [BUILDER `resources/src/hello.py.in`].
- `src/main.py`: an `Adw.Application` subclass with `application_id`, `resource_base_path`, `quit`/`about`/`preferences` actions, `Adw.AboutDialog` [BUILDER `resources/src/main-gtk4.py`].
- `src/window.py`: `@Gtk.Template(resource_path=...)` window class with `Gtk.Template.Child()` [BUILDER `resources/src/window-gtk4.py`].
- `src/window.ui` and `shortcuts-dialog.ui` in **GtkBuilder XML**, not Blueprint [BUILDER `resources/src/window-gtk4.ui`, `resources/src/hello.gresource.xml`]. Nothing in the template directory references Blueprint.
- `src/meson.build`: `gnome.compile_resources(..., gresource_bundle: true, install: true, install_dir: pkgdatadir)`, `configure_file` for the launcher, and `install_data(sources, install_dir: moduledir)` where `moduledir = pkgdatadir / name` [BUILDER `resources/src/meson-py-gtk4.build`]. It does not use `python.install_sources`.
- `data/`: `.desktop.in` with `DBusActivatable=true`, a D-Bus `.service.in` with `Exec=@bindir@/<exec> --gapplication-service`, an empty `.gschema.xml`, `.metainfo.xml.in`, icons [BUILDER `resources/data/`].
- `data/meson.build`: `i18n.merge_file` for desktop and metainfo, plus `meson test` targets that run `desktop-file-validate`, `appstreamcli validate --no-net --explain`, and `glib-compile-schemas --strict --dry-run` [BUILDER `resources/data/meson.build`].
- Top-level `meson.build`: `meson_version: '>= 1.0.0'`, `gnome.post_install(glib_compile_schemas: true, gtk_update_icon_cache: true, update_desktop_database: true)` [BUILDER `resources/meson.build`].
- A Flatpak manifest with `org.gnome.Platform`/`org.gnome.Sdk`, finish-args `--share=network --share=ipc --socket=fallback-x11 --device=dri --socket=wayland`, and one `meson` module building from `"type": "dir"` [BUILDER `resources/flatpak.json.in`]. Builder's default `gnome_sdk_version` option is `master` [BUILDER `meson_options.txt`].

### Meson pieces

- `python.find_installation('python3', modules: [...])` checks required Python modules (since 0.51.0) [MESON-PY]. Secrets uses this to fail early if `pykeepass` etc. are missing [SECR `meson.build`].
- `py.install_sources(files, subdir: ..., pure: ...)` installs `.py` files to Python's site dir; `py.get_install_dir()` returns that dir for use with `configure_file` [MESON-PY].
- `gnome.compile_resources(..., gresource_bundle: true, install: true, install_dir: ..., dependencies: ...)` builds a `.gresource` bundle; `dependencies` is how Blueprint output is wired in [MESON-GNOME, BLP `docs/setup.rst`].
- `gnome.post_install(glib_compile_schemas:, gtk_update_icon_cache:, update_desktop_database:)` (since 0.57.0; `update_desktop_database` since 0.59.0) [MESON-GNOME].
- `i18n.gettext()` and `i18n.merge_file(type: 'desktop'|'xml', po_dir: ...)` handle translations and translated desktop/metainfo files [MESON-I18N].

### How the Circle apps actually lay out Python code

Three install styles are in use; none of the four apps uses `py.install_sources`:

| App | Python package location | How installed | UI format |
|---|---|---|---|
| Apostrophe | Python site dir | `install_subdir('apostrophe', install_dir: py_installation.get_install_dir())` [APOS `meson.build`] | 24 `.ui`, 0 `.blp` |
| Secrets | Python site dir | `install_subdir(..., install_dir: python_bin.get_install_dir())`; `const.py` via `configure_file` [SECR `gsecrets/meson.build`] | 34 `.ui`, 0 `.blp` |
| Dialect | `pkgdatadir/dialect` (Builder style) | `install_subdir('widgets', install_dir: moduledir)` etc. [DIAL `dialect/meson.build` line 76-79] | 7 `.blp`, 1 `.ui` |
| Wike | `pkgdatadir` (Builder style) | `install_data(wike_sources, install_dir: moduledir)` [WIKE `src/meson.build` line 35] | 27 `.ui`, 0 `.blp` |

(File counts from `find` over each tree at the commits above.)

Common patterns:

- A `profile` meson option producing a `.Devel` app ID and a separate Devel Flatpak manifest [APOS `meson.build`; SECR `meson.build`; DIAL `build-aux/flatpak/app.drey.Dialect.Devel.json`].
- A `config.py.in` / `const.py.in` filled by `configure_file` with `APP_ID`, `VERSION`, `LOCALE_DIR`, `PKGDATA_DIR` [APOS `config.py.in` + `meson.build`; SECR `gsecrets/meson.build`].
- Apostrophe also generates a `local-apostrophe` launcher that runs from the source tree and prepends the build dir to `XDG_DATA_DIRS` so uninstalled GSettings schemas are found [APOS `meson.build`, `apostrophe.in` lines 26-33].

### GResource, GSettings, desktop file, metainfo, D-Bus service

All four are part of Builder's template (see above) [BUILDER `resources/data/`]. `@Gtk.Template` can load from `string`, `filename` or `resource_path` [PYGO `docs/guide/gtk_template.rst`]. GSettings inside Flatpak works without dconf permissions on any runtime newer than `org.gnome.Platform//3.34`, because GLib >= 2.60.5 uses a keyfile backend with the settings portal [FP-PERM, "dconf access"].

### Blueprint vs `.ui` XML

- Blueprint's own docs still say: "**Blueprint is still experimental.** Future versions may have breaking changes, and most GTK tutorials use XML syntax." [BLP `docs/index.rst` lines 4-9]. Latest release v0.22.0/0.22.2 added a minimum Python check, extern enums, GList in expressions [BLP `NEWS.md`]; v0.20.0 added a linter and `--minify` [BLP `NEWS.md`].
- Integration: either a `subprojects/blueprint-compiler.wrap` subproject or the copy in the GNOME SDK; then a `custom_target` running `blueprint-compiler batch-compile` whose output is passed as `dependencies:` to `gnome.compile_resources` [BLP `docs/setup.rst`]. Dialect does exactly this, with a wrap pinned to `v0.16.0` and also building Blueprint as a Flatpak module [DIAL `subprojects/blueprint-compiler.wrap`, `dialect/meson.build` lines 6-28, Devel manifest modules].
- Since GNOME 49 the SDK includes blueprint-compiler [GBM], so a Flatpak build no longer needs its own Blueprint module.
- Among the four Circle apps read, only Dialect uses Blueprint; Apostrophe, Secrets and Wike use XML (table above). Builder's template uses XML [BUILDER].

---

## 3. Async and concurrency in PyGObject

### Native asyncio integration

- Available since PyGObject 3.50, still marked **experimental**: "the API is subject to change as potential issues are addressed" [PYGO `docs/guide/asynchronous.rst`; PYGO-ASYNC].
- Setup: `asyncio.set_event_loop_policy(gi.events.GLibEventLoopPolicy())`, then run the app with `Gio.Application.run()` / `Gtk.Application.run()`. `asyncio.run()` is called "incompatible" for app startup [PYGO-ASYNC].
- Awaitable Gio APIs: if the callback argument of a Gio `*_async` function is left out, PyGObject returns an awaitable (like `asyncio.Future`) that can be awaited and cancelled [PYGO-ASYNC].
- `Gio.Application.create_asyncio_task(coro)` (PyGObject override): creates the task, calls `self.hold()` so the app does not quit while the task runs, keeps a strong reference (asyncio only keeps weak ones), and releases on completion [PYGO `gi/overrides/Gio.py` lines 141-169].
- `Gio.Application.run()` itself enters the GLib-backed asyncio loop for the default main context [PYGO `gi/overrides/Gio.py` lines 134-139].
- The loop is a real `asyncio.SelectorEventLoop` subclass running inside GLib, so ordinary asyncio sockets and subprocesses work. PyGObject's test suite runs CPython's own `EventLoopTestsMixin` and `SubprocessMixin` against it [PYGO `gi/events.py` around line 465, `tests/test_events.py` lines 1-60]. `asyncio.run(coro, loop_factory=gi.events.GLibEventLoop)` is tested and works (useful for tests) [PYGO `tests/test_events.py` `test_asyncio_run`].
- Guidance: async tasks run at `GLib.PRIORITY_DEFAULT`, so consider lower priority for background work [PYGO-ASYNC]. Do not iterate the GLib main context recursively from inside asyncio (no `GLib.MainLoop.run`, no `Gtk.Dialog.run`) [PYGO `gi/events.py` `GLibEventLoopPolicy` docstring].
- **Policy deprecation**: Python 3.14 deprecates `asyncio.set_event_loop_policy` and all policy classes, with removal in 3.16 [PYDOC-POLICY]. PyGObject's docstring says "Python 3.16 is removing the concept of the event loop policy. FIXME: say what to do in the future" [PYGO `gi/events.py` lines 809-813]. In 3.58/main, a GLib loop is only created implicitly if the policy is a `GLibEventLoopPolicy` [PYGO `gi/events.py` lines 888-896], so the policy call is still needed and will emit a `DeprecationWarning` on Python 3.14 (GNOME 51). Replacement API is not yet defined upstream.
- In practice: Dialect and Secrets both call `asyncio.set_event_loop_policy(GLibEventLoopPolicy())` at startup and use `create_asyncio_task` [DIAL `dialect/main.py` lines 201-202; SECR `gsecrets/application.py` lines 8, 31, 56]. Dialect wraps it as a `@background_task` decorator so `async def` methods can be signal handlers and template callbacks [DIAL `dialect/asyncio.py`]. Dialect awaits libsoup directly: `await Session.get().send_and_read_async(message, 0)` [DIAL `dialect/providers/soup.py` line 93].

### Callbacks and Gio.Task (older style, still common)

- The callback style (`*_async` + `*_finish` + `Gio.Cancellable`) is documented as "a robust alternative" [PYGO `docs/guide/asynchronous.rst`]. Wike uses `send_and_read_async(message, 0, None, callback, user_data)` [WIKE `src/wikipedia.py` line 108]. Secrets still uses `Gio.Task.new(...)` in several places alongside asyncio [SECR `gsecrets/database_manager.py`, `gsecrets/utils.py`].

### Threads

- GTK is not thread safe; only the main thread may call GTK. Run blocking work in a `threading.Thread` and hand results back with `GLib.idle_add` [PYGO `docs/guide/threading.rst`]. Signals run in the thread that emits them [same].
- PyGObject 3.59.0 (unreleased) supports free-threading Python, but "the libraries you're accessing through PyGObject may not be" safe [PYGO `docs/guide/threading.rst`, `NEWS`].
- Apostrophe and Secrets use exactly this thread + `GLib.idle_add` pattern for blocking work [APOS `apostrophe/inline_preview.py`; SECR `gsecrets/password_generator.py`, `gsecrets/widgets/database_settings_dialog.py`].

### Subprocesses (Gio.Subprocess)

- `Gio.Subprocess` gives stdin/stdout/stderr as `GInputStream`/`GOutputStream`, `wait_async`/`wait_check_async`, `communicate_utf8_async`; children are reaped automatically; only listed fds are inherited unless `INHERIT_FDS` is set; since GLib 2.82 creating one ignores SIGPIPE process-wide [GSUB].
- Because Gio `*_async` methods are awaitable [PYGO-ASYNC], a line-by-line reader over `get_stdout_pipe()` can be written as a coroutine. `asyncio.create_subprocess_exec` also works on the GLib loop (CPython's `SubprocessMixin` tests pass) [PYGO `tests/test_events.py`].

### Streaming HTTP

- libsoup 3.8 is in the GNOME 51 runtime [LOCAL-RT; GBM `elements/sdk-platform.bst` lists `sdk/libsoup.bst`]. `Soup.Session.send_async` completes when response headers arrive and `send_finish` returns a `GInputStream` for the body, so the body can be read incrementally [SOUP]. This is the right shape for server-sent events / token streaming.
- `send_and_read_async` buffers the whole body (used by Dialect and Wike for non-streaming calls) [DIAL `dialect/providers/soup.py`; WIKE `src/wikipedia.py`].
- httpx or aiohttp would have to be bundled as pip modules (they are not in the runtime: no pip packages beyond what the SDK ships) and would run on the GLib asyncio loop, which passes CPython's event-loop tests [PYGO `tests/test_events.py`]. No Circle app read uses them; Secrets uses `requests` in a thread [SECR `gsecrets/widgets/database_settings_dialog.py`]. Network needs `--share=network` either way [BUILDER `resources/flatpak.json.in`].

---

## 4. Secrets and settings

- libsecret: "If available, secrets are stored in the freedesktop secret service. Otherwise, secrets are stored in a file that is encrypted using a master secret that was provided by the secret portal." [SECRET `README.md`]. In code, if `/.flatpak-info` exists and the Secret portal is new enough, libsecret picks the `file` backend [SECRET `libsecret/secret-backend.c` lines 156-158]. So a Flatpak app calling `Secret.password_store`/`password_lookup` needs no D-Bus permission.
- The Secret portal's `RetrieveSecret` returns a per-app master secret that stays the same while the app is installed (interface version 2) [PORTAL-SECRET].
- Flathub: `--talk-name=org.freedesktop.Secrets` (wrong case) is an error that is "never granted"; the correct name is `org.freedesktop.secrets` [FH-LINT]. Portal names never need to be added [FH-LINT].
- Dialect stores API keys with `Secret.Schema.new(...)` and `Secret.password_store_sync`/`password_lookup_sync`/`password_clear_sync`, with no secret-service talk-name in its manifest [DIAL `dialect/providers/settings.py`; Devel manifest finish-args]. Note these are `_sync` calls on the main thread.
- `Secret` typelib is in the runtime (`sdk/libsecret.bst` in [GBM `elements/sdk-platform.bst`]; imported OK in [LOCAL-RT]).
- GSettings: schema installed to `share/glib-2.0/schemas` and compiled by `gnome.post_install` [BUILDER `resources/data/meson.build`, `resources/meson.build`]; no dconf permission needed in modern runtimes [FP-PERM].

---

## 5. Host commands, Flathub policy, and Python dependencies

### Running commands on the host

- `flatpak-spawn --host` runs a command "unsandboxed on the host. This requires access to the org.freedesktop.Flatpak D-Bus interface." [FP-CMD]. Without `--host`, `flatpak-spawn` creates a copy of the app's own sandbox via the Flatpak portal, optionally with tighter permissions (`--sandbox`, `--no-network`) [FP-CMD].
- The "Development" interface is `org.freedesktop.Flatpak.Development` (method `HostCommand`), exported by the same service that owns `org.freedesktop.Flatpak`. Upstream calls it "a sandbox escape allowing arbitrary code execution" that "must not be allowed for applications where sandboxing is intended to be a security boundary", and gives GNOME Builder as the intended user [FLATPAK `data/org.freedesktop.Flatpak.xml` lines 30-96]. It is not an xdg-desktop-portal.
- Flathub: `--talk-name=org.freedesktop.Flatpak` triggers `finish-args-flatpak-spawn-access` (warning: "allows commands to run outside the sandbox") and `finish-args-flatpak-talk-name` (error); an exception is "Granted on sufficient explanation being provided and only for Flatpak clients or development apps." [FH-LINT]. Exceptions are case-by-case, "temporary", and requested by PR; "Please do not use LLMs in any way to handle exceptions PRs. The exception can be permanently denied in that case." [FH-LINT]. Flathub also rejects designs "disabling or bypassing security mechanisms" and requires portals where one exists [FH-REQ].
- `--socket=session-bus` / `--socket=system-bus` "must be avoided, unless the application is a development tool" [FP-PERM].

### Bundling Python dependencies

- Flathub builds have no network access; every dependency must be a source with a public URL in the manifest; `--share=network` in `build-args` does not work [FH-REQ].
- flatpak-pip-generator (flatpak-builder-tools): `flatpak-pip-generator --runtime='org.freedesktop.Sdk//25.08' --requirements-file='requirements.txt' --output pypi-dependencies` writes a module JSON to include in `modules` [FBT `pip/readme.md`; FP-PY]. It prefers `none-any` wheels, then sdists; platform wheels only with `--prefer-wheels=...` plus `--runtime`, defaulting to x86_64 and aarch64 [FBT `pip/readme.md`]. The generated module runs `pip3 install --no-index --find-links=file://${PWD} --prefix=${FLATPAK_DEST}` [DIAL `build-aux/flatpak/pypi-dependencies.json`; SECR `flatpak/python3-pykeepass.json`].
- req2flatpak: takes a fully pinned `requirements.txt`, lets you target a given Python version and architecture, and outputs a flatpak-builder module [R2F `README.rst`]. No Circle app read uses it.
- Dialect keeps deps in `pyproject.toml`, locks with `uv` (`uv.lock`), exports `requirements.txt` with `uv export`, then generates `pypi-dependencies.json` [DIAL `pyproject.toml`, `requirements.txt` header, `build-aux/flatpak/`]. Secrets keeps one generated JSON per dependency (and one for pytest) under `flatpak/` [SECR `flatpak/`].
- Flathub requires generated dependency manifests to be included in the submission [FH-REQ].

---

## 6. Testing, CI, linting

### Unit tests

- Circle apps run tests through `meson test`, which `flatpak-builder` runs when a module has `run-tests` [FGA `README.md` input `run-tests`; CIT `flatpak-job.sh`]. Secrets declares `test('Test Element', python_bin, args: files('test_element.py'), env: [G_TEST_SRCDIR, G_TEST_BUILDDIR])` and the test files use pytest fixtures [SECR `tests/meson.build`, `tests/test_element.py`]. Secrets bundles pytest into its Devel Flatpak (`python3-pytest.json`) [SECR `flatpak/`] because the SDK has no pytest [LOCAL-RT].
- Apostrophe has `tests/test_regex_commonmark.py` and `tests/test_lint.py` [APOS `tests/`].
- Secrets' own test file notes a gap: "Hack around async methods / TODO Provide a sync method which tests unlock_async" [SECR `tests/test_element.py`]. For async code, `asyncio.run(coro, loop_factory=gi.events.GLibEventLoop)` gives a GLib-backed loop per test [PYGO `tests/test_events.py`].

### Headless display

- PyGObject's testing guide: run under headless Mutter: `export XDG_RUNTIME_DIR=/tmp; eval $(dbus-launch --auto-syntax); mutter --wayland --no-x11 --sm-disable --headless -- python my_script.py` [PYGO `docs/guide/testing.rst`].
- GNOME's Flatpak CI job wraps the whole build in `dbus-run-session -- mutter --headless --wayland --no-x11 --virtual-monitor 1024x768 -- flatpak-builder ...` and forces software rendering (`LIBGL_ALWAYS_SOFTWARE=true`) [CIT `runtime-images/flatpak-job/flatpak-job.sh` lines 184-197; CIT `flatpak/flatpak_ci_initiative.yml`].
- GTK's own CI runs tests three ways: `xvfb-run` (x11), `mutter --headless --wayland` (wayland), and `gtk4-broadwayd :5` with `BROADWAY_DISPLAY=:5` (broadway) [GTK `.gitlab-ci/run-tests.sh` lines 14-60]. `GDK_BACKEND` selects `x11`, `wayland`, `broadway` etc. [GTK `docs/reference/gtk/running.md` line 319]. Broadway is off by default in GTK builds [GTK `meson.options`] but `gtk4-broadwayd` is present in the GNOME 51 Platform [LOCAL-RT].
- The GitHub action's `run-tests` enables network and X11 "with a display server provided by `xvfb-run`" [FGA `README.md`].

### CI

- GNOME GitLab: include `GNOME/citemplates` `flatpak/flatpak_ci_initiative.yml` and extend `.flatpak@x86_64` / `.flatpak@aarch64`; image default `quay.io/gnome_infrastructure/gnome-runtime-images:gnome-master`; JUnit report from meson test logs; `RUN_TESTS: "1"` by default [CIT `flatpak/flatpak_ci_initiative.yml`]. Secrets uses this plus separate `codespell`, `mypy`, `ruff format --check`, `ruff check` jobs on a Fedora image [SECR `.gitlab-ci.yml`]. Apostrophe uses the same template [APOS `.gitlab-ci.yml`].
- GitHub: `flatpak/flatpak-github-actions/flatpak-builder@v6` in container `ghcr.io/flathub-infra/flatpak-github-actions:gnome-48` (README example) with `--privileged`; inputs include `manifest-path`, `bundle`, `run-tests`, `arch`, `cache-key` [FGA `README.md`]. Dialect still uses the older `bilelmoussaoui/flatpak-github-actions/flatpak-builder@v6` with image `bilelmoussaoui/flatpak-github-actions:gnome-nightly` and `run-tests: "true"` [DIAL `.github/workflows/CI.yml`].
- Circle requires "Passing CI builds of the app" and recommends "Builds Flatpak in Builder out of the box" and Meson/Automake/CMake [CRIT].

### Lint and types

- pygobject-stubs 2.20.0 (requires Python >= 3.10); install with `uv add --dev pygobject-stubs` or pip; Gtk 3 vs 4 chosen at install time via `--config-settings=config=...` or `PYGOBJECT_STUB_CONFIG`, defaulting to the newest of each library [STUBS `README.md`, `pyproject.toml`]. Ships stubs for Adw (updated to 1.10.0), Gtk 4, Gio, GLib, Soup 3, Secret, Xdp, etc. [STUBS `CHANGELOG.md`, `src/gi-stubs/repository/`]. PyGObject's own docs point to it for IDE support [PYGO `docs/getting_started.rst` line 40].
- Secrets: ruff with `select = ["ALL"]` and a long ignore list, mypy with `ignore_missing_imports` and `disallow_incomplete_defs` [SECR `pyproject.toml`].
- Dialect: ruff, pyright config, and dev group `pygobject-stubs`, `ruff`, `ty` [DIAL `pyproject.toml`].
- PyGObject now documents `gobject-linter` for linting GObject code [PYGO `NEWS` 3.57.0, MR 543/551/552]; not investigated further.

---

## 7. Local dev setup on Fedora

Three options appear in primary sources:

1. **GNOME Builder (Flatpak)**. Recommended by GNOME's tutorial [GNOME-TUT] and Builder's docs ("the preferred installation method"):
   ```
   flatpak install --from https://flathub.org/repo/appstream/org.gnome.Builder.flatpakref
   flatpak run org.gnome.Builder
   ```
   [BUILDER-INST]. `org.gnome.Builder` is available in this machine's `flathub` remote [LOCAL-FH]. Circle recommends that the app "Builds Flatpak in Builder out of the box" [CRIT].

2. **flatpak-builder only (no Builder)**. Flathub recommends `org.flatpak.Builder` [FH-SUB]:
   ```
   flatpak install -y flathub org.flatpak.Builder
   flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo
   flatpak run --command=flathub-build org.flatpak.Builder --install <manifest>
   flatpak run <app id>
   flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest <manifest>
   flatpak run --command=flatpak-builder-lint org.flatpak.Builder repo repo
   ```
   [FH-SUB]. Plain flatpak-builder (installed "from your distribution") with `flatpak-builder --force-clean --user --install-deps-from=flathub --repo=repo --install builddir <manifest>` [FP-FIRST]. The SDK is ~875 MB download / 2.5 GB installed for `org.gnome.Sdk//51` [LOCAL-FH `flatpak remote-info`]. The SDK already contains meson, blueprint-compiler, appstreamcli, desktop-file-validate, glib-compile-schemas and pip3 [LOCAL-RT, checked on Sdk//50], so no host dev packages are required for building.

3. **Host Python (fast edit-run loop, tests, IDE)**. PyGObject's Fedora instructions: system packages `sudo dnf install python3-gobject gtk4`; or in a venv `sudo dnf install gcc gobject-introspection-devel cairo-gobject-devel pkg-config python3-devel gtk4` then `pip3 install pycairo` and `pip3 install PyGObject` [PYGO `docs/getting_started.rst` lines 103-118]. Fedora 45 already has `python3-gobject` 3.58.1, `gtk4` 4.24.1 and `libadwaita` 1.10.0 installed [LOCAL-RPM], matching GNOME 51's library versions; only Python differs (3.15 vs 3.14).

Note: this machine's system `flathub` remote is Fedora's `filtered` one but still lists the GNOME SDK, `org.flatpak.Builder` and `org.gnome.Builder` [LOCAL-FH]; Flathub's docs add an unfiltered `--user` remote [FH-SUB, FP-FIRST].

Exact package names for `meson`, `blueprint-compiler` and `flatpak-builder` on Fedora were not checked against a primary source here.

---

## 8. Recommended stack for Budha

Derived from the findings above. Each item names the alternative and the trade-off.

1. **Runtime: `org.gnome.Platform`/`Sdk` 51** (Python 3.14, GTK 4.24, libadwaita 1.10, PyGObject 3.58) [GBM, LOCAL-RT]. Circle requires the GNOME runtime and keeping on a supported one [CRIT]. Using 51 unlocks `AdwSidebar` (1.9), `AdwToggleGroup` (1.7), `AdwSpinner`/`AdwBottomSheet` (1.6), `AdwShortcutsDialog` (1.8) [ADWDOC]. Alternative: 50 (Python 3.13, libadwaita 1.9) for a wider installed base; costs nothing in widgets except 1.10's additions.

2. **Build: Meson, Builder-template layout** [BUILDER], with a `profile` option and a `.Devel` app ID/manifest as in Apostrophe/Secrets/Dialect [APOS, SECR, DIAL]. Install the Python package into the site dir with `py.install_sources(..., subdir: 'budha', preserve_path: true)` or `install_subdir` (Apostrophe/Secrets) [MESON-PY, APOS, SECR] rather than Builder's `pkgdatadir` + `sys.path` trick, so the package is importable by tests and type checkers without path hacks. Keep the template's validation tests for desktop/metainfo/schema [BUILDER `resources/data/meson.build`].

3. **UI files: Blueprint**, with plain XML as a fallback. Pro: it is in the SDK since GNOME 49 so no subproject is needed [GBM], and Dialect ships with it in Circle [DIAL]. Con: upstream still calls it experimental with possible breaking changes [BLP `docs/index.rst`], and Builder and 3 of 4 Circle Python apps use XML [BUILDER, table in §2]. If ADR-level stability matters more than readability, choose XML. Pin by building with the SDK's version rather than a `revision = main` wrap.

4. **Concurrency: PyGObject asyncio**: `GLibEventLoopPolicy` + `Adw.Application.run()` + `create_asyncio_task`, with a Dialect-style `@background_task` helper [PYGO, DIAL]. Await Gio/Soup `*_async` calls directly. Use threads + `GLib.idle_add` only for unavoidable blocking calls [PYGO `threading.rst`]. Risk: still experimental, and the policy API is deprecated in Python 3.14 and removed in 3.16 with no upstream replacement documented yet [PYDOC-POLICY, PYGO `gi/events.py`]. Wrap the setup in one small module so it can be swapped. Alternative: callbacks + `Gio.Cancellable` (stable, more verbose) [PYGO-ASYNC].

5. **Talking to Hermes.** Hermes ships a `hermes-acp` entry point (Agent Client Protocol adapter) and requires Python `>=3.11,<3.15` [HERMES `pyproject.toml` lines 15, 450, 596], so it is compatible with GNOME 51's Python 3.14 [FDO]. Options:
   - (a) Bundle Hermes in the Flatpak via flatpak-pip-generator and run it as a child with `Gio.Subprocess` (stdio pipes, awaitable reads) [GSUB, FBT]. Stays inside the sandbox, Flathub-friendly, but Hermes' tools then run in Budha's sandbox and its dependency tree must be vendored as manifest sources [FH-REQ].
   - (b) Run a host-installed Hermes via `flatpak-spawn --host`. Needs `--talk-name=org.freedesktop.Flatpak`, which is a sandbox escape and a Flathub linter error needing an exception granted only for "Flatpak clients or development apps" [FLATPAK, FH-LINT]. Circle requires Flathub [CRIT], so this is a high-risk path.
   - (c) Connect over the network to a Hermes server the user runs (HTTP streaming with libsoup `send_async` → `GInputStream`) [SOUP]. Needs only `--share=network`.
   The choice belongs in an ADR; see `docs/research/hermes-agent-integration.md` for the Hermes side.

6. **HTTP: libsoup 3** from the runtime (no bundling, awaitable, streaming via `send_async`) [SOUP, LOCAL-RT, DIAL]. Alternative: httpx/aiohttp bundled via pip; works on the GLib asyncio loop [PYGO `tests/test_events.py`] but adds vendored deps.

7. **Secrets: libsecret** (`Secret.password_store`/`lookup`, async variants) with no extra finish-args; it uses the Secret portal + encrypted file inside Flatpak [SECRET, PORTAL-SECRET, FH-LINT]. **Settings: GSettings** schema in `data/` [BUILDER, FP-PERM].

8. **Dependencies: `pyproject.toml` + `uv.lock` → `uv export` → flatpak-pip-generator** (Dialect's flow) [DIAL, FBT]. req2flatpak is the alternative when exact per-arch wheel control is needed [R2F].

9. **Tests: pytest**, run by `meson test` both on the host and inside `flatpak-builder` (bundle pytest in the Devel manifest as Secrets does) [SECR]. Headless via `mutter --headless --wayland --virtual-monitor` (what GNOME CI uses) [CIT, PYGO `testing.rst`]; broadway or xvfb as alternatives [GTK]. Keep most logic GTK-free so it tests without a display.

10. **CI**: if hosted on GitHub, `flatpak/flatpak-github-actions/flatpak-builder@v6` with a GNOME image (x86_64 + aarch64, since Circle requires both) [FGA, CRIT]; if on GNOME GitLab, `citemplates` `flatpak_ci_initiative.yml` [CIT]. Add `ruff format --check`, `ruff check`, a type checker with `pygobject-stubs`, and `flatpak-builder-lint` [SECR, STUBS, FH-SUB].

11. **Keep the hand-written Flatpak manifest small.** Flathub's Generative AI policy: "Flathub manifests must not contain AI-generated or AI-assisted content"; AI-generated code and packaging must be disclosed, "AI tools or agents must not open or automate Flathub submission pull requests", and reviewers may reject based on "the extent or role of generated material" [FH-REQ, "Generative AI policy"]. Budha's manifest will therefore be written by the human. Things that keep it short:
    - **Use what the runtime already ships** instead of pip modules: PyGObject, GTK, libadwaita, libsoup 3, libsecret, blueprint-compiler are all in GNOME 51 [GBM `elements/sdk-platform.bst`, LOCAL-RT]. Choosing libsoup over httpx/aiohttp and libsecret over `keyring` means zero Python dependency modules for those features. Each pip dependency adds wheel/sdist sources (Secrets' `python3-pykeepass.json` alone lists argon2-cffi, cffi, construct, lxml, ...) [SECR `flatpak/python3-pykeepass.json`].
    - **Do not use a Blueprint subproject or module**: the SDK's blueprint-compiler (since GNOME 49) removes the extra module Dialect carries [GBM, DIAL Devel manifest].
    - **One app module** of `buildsystem: meson` with a `dir` (local) or `git` source, like Builder's template, and the template's finish-args (`--share=network --share=ipc --socket=fallback-x11 --socket=wayland --device=dri`) as the starting point [BUILDER `resources/flatpak.json.in`]. No `org.freedesktop.Flatpak` talk-name if Hermes is not host-spawned (§8.5).
    - **Generate dependency modules with a tool the human runs**, never by hand or by an agent: `flatpak-pip-generator --requirements-file=requirements.txt --output pypi-dependencies` [FBT `pip/readme.md`, FP-PY] or req2flatpak [R2F], then reference the single `pypi-dependencies.json` from `modules` (Dialect's pattern: one generated file, from a `uv export`ed `requirements.txt`) [DIAL]. The agent can maintain `pyproject.toml`/`requirements.txt`, which are not the manifest. Whether Flathub treats tool output run by the human as acceptable was not stated explicitly in the policy; this is assumed from Flathub's own pointer to "community provided" generator tools [FH-REQ] and should be confirmed.
    - **Hermes bundling (option 5a) is the biggest manifest cost**: Hermes' full dependency tree (plus extras such as `acp`) [HERMES `pyproject.toml`] would all be generated modules. Options 5b/5c avoid that, at the cost described in §8.5.
    - **Test-only deps (pytest) only in the Devel manifest**, as Secrets does [SECR `flatpak/`], or run tests on the host instead so the Flathub manifest has no test modules.
    - Lint the hand-written manifest locally with `flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest <manifest>` before submitting [FH-SUB].

12. **Dev setup on this machine**: install `org.gnome.Sdk//51` (only Sdk 50 is installed now) and `org.flatpak.Builder` from Flathub; optionally GNOME Builder [LOCAL-FH, FH-SUB, BUILDER-INST]. Use the already-installed Fedora `python3-gobject`/`gtk4`/`libadwaita` for a fast host loop and tests [LOCAL-RPM, PYGO `getting_started.rst`].

### Open questions

- Which Hermes integration (bundled ACP child, host spawn, or network)? Decides finish-args, Flathub viability, and how tools reach the user's files.
- Blueprint or XML? Readability vs. "experimental" status.
- What will PyGObject offer to replace `GLibEventLoopPolicy` before Python 3.16? Not documented as of `673a5df` [PYGO `gi/events.py`].
- Hosting: GitHub (app ID `io.github.<user>.Budha`) or GNOME GitLab World? Affects CI template and app ID rules [CRIT].
- Circle's AI policy rejects submissions with "indications of AI-generated output" such as "imaginary API usage" or "comments serving as LLM prompts"; developers "should be able to justify and explain the code" [CRIT]. Flathub's exception process bans LLM-written exception PRs [FH-LINT]. This affects how Budha itself is developed.
- Whether to target GNOME 50 for reach or 51 for newest widgets.
- Does Flathub count manifest modules produced by flatpak-pip-generator/req2flatpak (run by the human) as acceptable under "must not contain AI-generated or AI-assisted content"? Likely yes, but not stated [FH-REQ]; ask on Flathub before submission.
- How will the Devel manifest for CI (also a manifest) be handled under the same rule? Simplest: the human writes both, the Devel one differing only in app ID, `-Dprofile=development`, and test modules.
