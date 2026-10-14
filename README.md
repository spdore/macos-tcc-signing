# macOS TCC Signing Skill

A Codex skill for developers who lose macOS privacy permissions after rebuilding an app. It helps keep permissions such as Accessibility, Input Monitoring, Microphone, and Screen Recording associated with the same local development app.

## The problem

macOS uses a code signature requirement to identify an app for privacy permissions. An ad hoc signature can change with every build, even when the app keeps the same name. As a result, an old permission entry may refer to a previous build identity.

This skill guides Codex to inspect the app that actually requests permission, use a stable local code-signing identity, keep the bundle identifier fixed, and reset only a stale permission when the signing identity first changes.

## No Apple Developer account required for local development

The default workflow uses a persistent self-signed code-signing certificate in the Mac's login keychain. Create it once and reuse it across local projects and builds. The certificate's private key should stay protected and should not be regenerated for each project.

A self-signed identity can provide stable code identity for local macOS checks that do not require an Apple-trusted signer. It does not provide Apple Development or Developer ID credentials, notarization, or a trusted publisher identity for other Macs. Projects that require Apple-only entitlements or distribution must use the appropriate Apple-issued credentials.

## Install in Codex

### macOS or Linux

```sh
skills_root="${CODEX_HOME:-$HOME/.codex}/skills"
mkdir -p "$skills_root"
git clone https://github.com/spdore/macos-tcc-signing.git "$skills_root/macos-tcc-signing"
```

If the skill is already installed, update it from its directory:

```sh
cd "${CODEX_HOME:-$HOME/.codex}/skills/macos-tcc-signing"
git pull
```

Codex discovers skills from their `SKILL.md` name and description. If the skill does not appear in a conversation, start a new conversation or call it explicitly with `$macos-tcc-signing`.

## What the skill does

- Checks the exact app path, bundle identifier, code signature, designated requirement, and responsible process.
- Uses a persistent local signing identity for Debug builds without requiring an Apple Developer account.
- Preserves existing entitlements and avoids broad TCC resets or direct database edits.
- Separates local development signing from release signing and notarization.

The skill does not grant permissions by itself. macOS still controls privacy prompts, and the user must approve them.

## References

- [Apple TN3127: Inside Code Signing: Requirements](https://developer.apple.com/documentation/technotes/tn3127-inside-code-signing-requirements)
- [Apple TN2206: macOS Code Signing In Depth](https://developer.apple.com/library/archive/technotes/tn2206/)
- [Apple Code Signing Tasks](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/Procedures/Procedures.html)
- [Codex skills documentation](https://developers.openai.com/api/docs/guides/tools-skills)

## License

This project is available under the [MIT License](LICENSE).
