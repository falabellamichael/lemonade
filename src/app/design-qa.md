# Nexus account dropdown and page

Initial visual result: passed. Subsequent integration and passwordless account
verification are recorded below; the original UI-only scope is historical.

## Scope and visual target

- Source visual truth: `D:/Lemonade-GUI3-build/output/concepts/nexus-account/concept-combined-admin-avatar-v3.png`.
- Implementation: `http://127.0.0.1:9123/#/connect/nexus`.
- Primary screenshot: `D:/Lemonade-GUI3-build/output/nexus-qa/desktop-final.jpg`.
- Source and primary implementation captures are both 1448 × 1086 pixels, at a 1448 × 1086 CSS viewport. No image rescaling was needed for comparison.
- Both show the Nexus Settings page with the account dropdown open in the dark theme. The approved concept is signed in; the implementation intentionally starts signed out, as subsequently requested. Names, role badges, connected-device rows, and image selection appear only when account data is supplied.
- Manage users, account/device settings, user creation, and advanced settings are intentionally omitted. Authentication and device pairing are outside this UI-only change.

## Visual comparison

The source image and final browser screenshot were opened together in the same comparison input. The warm neutral surfaces, circular avatar, compact upper-right dropdown, account/device hierarchy, section dividers, and restrained yellow selection edge follow the selected concept. Existing GUI3 page-header typography and settings-rail density are retained rather than resizing the rest of the application to the generated mockup.

The dropdown header, avatar, section labels, and navigation were readable in the full-resolution comparison; a separate crop was unnecessary. The image-error state was separately inspected at full resolution in `output/nexus-qa/avatar-error-final.jpg`.

Required fidelity checks:

- Typography: existing GUI3 font stack and tokens; account names wrap safely, header identity truncates.
- Spacing: 360px dropdown with bounded viewport height; account and device sections use existing spacing tokens.
- Colors: existing light/dark tokens, neutral avatar and buttons, yellow limited to the selected Nexus rail edge.
- Images/icons: shared 256px square image preview, initial fallback, existing vendored Lucide icon system. No new runtime dependency.
- Content: signed-out state and unavailable sign-in are explicit; no sample identity or connected devices in the application.

## Comparison history and fixes

1. P2: At 320px the account control overlapped the top navigation. Reserved space for the controls and reduced the icon layout at the narrow breakpoint. Verified non-overlapping bounds and captured `output/nexus-qa/mobile-320-final.jpg`.
2. P2: The existing selected settings row applied too much yellow compared with the approved direction. Scoped a neutral selection surface and thin yellow edge to Nexus only. Verified in `output/nexus-qa/desktop-final.jpg`.
3. P2: An invalid-avatar message wrapped into a narrow column under the image. Moved the error below the full account summary, keeping the image and name aligned. Verified in `output/nexus-qa/avatar-error-final.jpg`.
4. P2: Desktop window controls at 400px exposed an additional header collision. Adjusted the desktop compact layout with sufficient CSS specificity. Verified at 400px, 500px, and 800px; all navigation buttons and window controls fit without overlap. Evidence: `output/nexus-qa/desktop-controls-400-final.jpg`. This was a browser fixture using the real App with a stubbed native host, not an installed Tauri run.

No actionable P0/P1/P2 findings remain within the requested UI scope.

## Interaction and responsive verification

- Dropdown opens, dismisses on outside interaction, and closes on Escape while returning focus to the trigger.
- Open Nexus navigates from Chat to `#/connect/nexus` and closes the dropdown.
- Opening the account dropdown closes the download manager.
- Light theme checked in `output/nexus-qa/light-final.jpg`.
- Mobile checked at 390 × 844 and 320 × 640. Final mobile evidence: `output/nexus-qa/mobile-final.jpg`.
- Isolated signed-in fixture: PNG selection updated the header, menu, and page to the same decoded 256 × 256 image. Invalid image data displayed an error while preserving the existing image. Removing the image restored the initial in all three locations.
- Switching fixture accounts removed the previous image and admin badge; returning to the original account did not restore its discarded local image. Signing out hid device rows. An unrelated input draft survived these changes.
- Inspected browser warning/error logs for the main page and account fixture; neither reported warnings or errors during those checks.

## Build and contract checks

- `npm run typecheck`: passed.
- `npm run test:i18n-coverage`: passed, 1380 unique Chinese translation entries and no untranslated literal keys.
- `npm run test:icons`: passed, 91 registered icons.
- `npm run test:ui-regressions`: passed, including backend-manager lifecycle checks. Updated the existing titlebar class assertion for the account-layout modifier.
- `npm run build:renderer:prod`: passed.
- Scoped `git diff --check`: passed.

## Integration boundary

`NexusAccountProvider` accepts typed account/device data and optional sign-in, sign-out, and connect-device callbacks. The initial UI build added no authentication or persistence. The subsequent backend and passwordless integration uses Nexus's Cluster/account model with verified device membership. See `docs/dev/nexus-account-ui.md` for the current integration boundary. Selected images are local to the current UI session and cleared on account changes. The account/menu/page components provide the boundaries for adding management sections later.

The installed desktop application and live server were not rebuilt or replaced. Native OS file-dialog and Tauri-host behavior still require a packaged-app check when this UI is integrated with Nexus.

## Backend integration verification

PR #3930's backend was subsequently integrated while retaining this UI. All 35
GUI runtime tests, type checking and the production renderer build pass. The
Windows Nexus SDK DLL and `lemond.exe` build successfully; NexusManagerTest,
HttpClientSecurityTest and HttpClientTimeoutTest pass. An isolated live-server
smoke verifies route-prefix registration, authentication, locked startup and
management redirects. Native Tauri `cargo check` passes using the SDK's built
OpenSSL. Details and limits are recorded in
`output/nexus-integration/verification.md`. These checks do not establish personal
account authentication or live hardware-backed passkey pairing.


## Recursive group management (2026-10-23)

Basic/Advanced radio controls now select compact browsing or the recursive tree.
The main GUI3 page remains signed out; the account dropdown is unchanged.

Verified with the real components in an isolated in-memory fixture:

- Three nested group levels with device leaves; ArrowRight expands branches.
- Creating a child group and renaming it update the tree and breadcrumbs.
- A rejected create keeps the draft and error visible without a phantom row;
  retry succeeds and returns keyboard focus to the action button.
- Group permission editing reads the existing assignment, excludes the current
  device, and sends the selected identity, role and permissions to the service.
- Read-only groups omit edit actions; device nodes have no group controls.
- Basic mode retains group navigation while hiding administration; search finds
  discovered nested devices and restores the corresponding tree path.
- Light and dark themes; 390px compact and 320px advanced layouts have no document
  horizontal overflow. Tree/detail panels stack on narrow screens.
- Fixture browser warning/error logs were empty during these checks.

Evidence: `output/nexus-qa/tree-advanced.jpg`, `tree-light.jpg`, `tree-mobile.jpg`.
These images contain explicitly labeled sample groups, not real account data.

Validation: TypeScript typecheck, production renderer build, all 36 GUI runtime
suites, i18n coverage (1468 entries), Windows lemond/SDK build, NexusManagerTest,
API docs generation check, 32 comment-style tests, and git diff whitespace check
passed. The new native tests cover recursive ancestry, scope escape, cycles,
leaf-parent rejection, capabilities, private-material omission, all route
prefixes, locked state and listener isolation. The client test also verifies
that a lost POST response never causes an automatic duplicate write.

Live federation group changes and hardware-backed passkey setup remain untested.
The installed server/application were not replaced; the backend executable in
`build/Release` includes the new routes. The passwordless account integration
below supersedes the earlier unsupported-sign-in state.

## Passwordless account flow (2026-10-23)

The default provider now opens the real create, invitation and returning-account
flow. It starts signed out and uses an explicit passkey check before displaying
the verified membership. Basic mode shows account devices; Advanced retains the
recursive network management surface. Existing account headings and copy edits
are preserved.

Browser checks used the real components and a clearly labeled isolated service
fixture, without credentials or live federation changes:

- Header and page sign-in open the same dialog. Empty names and malformed
  invitations cannot submit. Busy state disables duplicate submissions.
- Cancelled verification preserves the draft and leaves the account signed out.
- A verified creator shows the chosen label, server-confirmed Admin badge and
  linked devices. A read-only invited member has no Admin badge or invite action.
- Invitation generation displays a sample code, copy action and bounded expiry.
  Closing explains that it does not revoke the invitation.
- Sign-out clears the name, role, image and device list. Returning sign-in
  pre-fills the saved label and requires verification again. Loss of membership
  returns the UI to signed out with an actionable notice.
- At 320 × 640 the returning dialog fits within the viewport with no document
  horizontal overflow. Screenshot: `output/nexus-qa/account-sign-in-mobile.jpg`.
- Invalid tree data is now validated before rendering; the earlier fixture
  crash was repaired and logout was retested successfully.

Actual checks and build results are recorded in
`output/nexus-integration/account-verification.md`. Hardware-backed passkey use,
live federation account creation/joining and a packaged native-app run remain
unverified. The browser preview cannot perform native USB-key verification.
