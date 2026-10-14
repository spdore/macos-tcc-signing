---
name: macos-tcc-signing
description: Keep macOS privacy permissions stable across app rebuilds by diagnosing code identity and configuring a persistent local signing identity. Use for Accessibility, Input Monitoring, Microphone, Screen Recording, or related TCC permission issues. Local development does not require an Apple Developer account.
---

# Stable macOS Code Signing for TCC Permissions

## Purpose

Help macOS keep a user's privacy decision associated with the same app after its code is rebuilt. TCC identifies signed code through its designated requirement (DR), not only through the app's display name or file path. An ad hoc signature, including Xcode's “Sign to Run Locally,” is tied to a specific build. A new build gets a new cdhash, so the old permission record may no longer match.

For local development, use one persistent code-signing identity and keep the app's bundle identifier stable. The cdhash is expected to change when code changes; the signing identity and DR should remain stable.

Apple explains this behavior in [TN3127: Inside Code Signing: Requirements](https://developer.apple.com/documentation/technotes/tn3127-inside-code-signing-requirements). Apple also documents that self-signed identities can provide stable code identity for macOS checks that do not require an Apple-trusted signer in [TN2206: macOS Code Signing In Depth](https://developer.apple.com/library/archive/technotes/tn2206/).

## Workflow

1. **Find the code that macOS is authorizing.** Identify the app bundle, bundle identifier, executable, build output path, and launch method. If a command-line process or helper requests access, identify the responsible process first. Terminal, an IDE, a launcher, and the app itself can have different TCC identities. Do not infer identity from the Dock name or icon.
2. **Inspect the current signature and permission target.** Run `codesign -dv --verbose=4 <App.app>` and `codesign --display -r - <App.app>` on the exact app that is launched. Check its identifier, signing authority, TeamIdentifier when present, and DR. Use `security find-identity -p codesigning` to inspect local signing identities. Look for duplicate app copies in DerivedData, Applications, and other folders.
3. **Use a persistent local identity without requiring an Apple Developer account.** Reuse a local code-signing identity if one already exists. If none exists, guide the user through a one-time setup in Keychain Access: open **Certificate Assistant → Create a Certificate**, choose a fixed name such as `Codex Local TCC Signing`, select **Self Signed Root** as the identity type and **Code Signing** as the certificate type, and save it to the login keychain. Choose a long validity period if the wizard allows it and record the expiry date. Keep the private key safe. Do not create a new certificate for each project or build, export the private key without a clear need, or change system-wide trust settings just to suppress warnings.
4. **Configure the project's local Debug build.** Keep the bundle identifier and app launch path stable. For Xcode, set the Debug `CODE_SIGN_IDENTITY` to the exact local certificate name and use manual signing. When no Apple Team is available for the local target, do not invent a `DEVELOPMENT_TEAM` or provisioning profile. For other toolchains, sign the final app bundle with the same identity after the bundle is assembled. Sign nested executables, frameworks, plug-ins, and helpers before the outer app. Do not use `codesign --deep` as a general signing fix.
5. **Preserve required entitlements and identify platform limits.** A local self-signed certificate can provide a stable DR for TCC identity checks, but it is not Apple Development or Developer ID. Some Apple-only entitlements, restricted capabilities, App Store submission, notarization, and public distribution require Apple-issued credentials or provisioning. Explain the specific requirement if the project hits one. Do not remove needed entitlements to make signing appear successful.
6. **Clear only a stale permission after changing identity.** Quit every running copy. If an old TCC record is still tied to the ad hoc identity, reset only the affected service for the app's bundle identifier, for example `tccutil reset Accessibility <bundle-id>`. Relaunch the app and let macOS ask the user for permission again. Do not reset permissions after every build, run a global TCC reset, edit the TCC database, or disable SIP.
7. **Verify that identity stays stable.** After rebuilding, run `codesign --verify --deep --strict --verbose=2 <App.app>`, then inspect the signer, identifier, and DR again with the commands in step 2. Confirm that the inspected bundle is the one the user launches. The cdhash may change. The signing identity, bundle identifier, and DR should stay the same. Replacing or expiring the certificate changes the signing identity and may require one new authorization.

## Codex Behavior

- Use this skill when a macOS project requests TCC-protected permissions or the user reports that permissions disappear after rebuilding.
- Diagnose the actual responsible process and build configuration before changing files. If the user asks for research or a plan only, do not edit the project.
- When implementation work is authorized, change only the signing settings or build steps needed for a stable local identity. Preserve app behavior, existing entitlements, and unrelated project settings.
- Do not treat the lack of an Apple Developer account as a blocker for local development. Start with a persistent self-signed local identity and explain any Apple-only limitation that applies.
- The user must approve macOS privacy prompts. Never bypass consent by editing TCC databases or changing system security protections.

## Limits

This workflow is for development on the Mac that owns the self-signed certificate. A self-signed certificate does not prove the publisher's identity to other Macs, provide notarization, or satisfy Gatekeeper's normal trust expectations for distributed apps. Do not present it as a release-signing solution.

For creating a local signing identity, see Apple's [Code Signing Tasks](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/Procedures/Procedures.html).
