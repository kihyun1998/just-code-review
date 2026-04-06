---
name: jss-flutter
description: "Scan Flutter/Dart application security issues. Covers insecure storage, platform channel security, WebView risks, hardcoded secrets, debug mode leaks, and dependency vulnerabilities."
---

# JSS Flutter — Flutter / Dart Security Scan

Security scanning skill for Flutter/Dart applications. Focuses on mobile-specific and Flutter-specific security risks.

## Scan Procedure

1. Detect project structure: look for `pubspec.yaml`, `lib/`, `android/`, `ios/`, `web/` directories.
2. If no Flutter/Dart project is found, report and stop.
3. Run the checks below in order.
4. Format results according to the output format.

## Scan Targets

If arguments are provided, scan only the specified files/directories.
If no arguments are given, scan the entire project (`lib/`, `android/`, `ios/`, `web/`, `pubspec.yaml`).

Excluded from scan:
- `build/`, `.dart_tool/`, `.flutter-plugins`
- Test files (`*_test.dart`, `test/`) unless explicitly targeted
- Generated files (`*.g.dart`, `*.freezed.dart`, `*.mocks.dart`)

## Security Checks

### 1. Hardcoded Secrets & Credentials

Search for patterns indicating hardcoded secrets:
- API keys, tokens, passwords as string literals (e.g., `apiKey = "sk-..."`, `password = "..."`)
- Firebase config values hardcoded in Dart code instead of environment config
- Private keys or certificates embedded in source
- Hardcoded URLs with credentials (e.g., `https://user:pass@host`)

**Severity**: critical

### 2. Insecure Data Storage

- Using `SharedPreferences` for sensitive data (tokens, passwords, PII) — should use `flutter_secure_storage`
- Writing sensitive data to plain text files without encryption
- Storing secrets in `assets/` directory
- SQLite databases without encryption containing sensitive data (check for `sqflite` usage without `sqflite_sqlcipher` or `drift` encryption)

**Severity**: high

### 3. Network Security

- HTTP URLs used instead of HTTPS (except `localhost` / `10.0.2.2` for development)
- Missing certificate pinning for sensitive API communication
- `BadCertificateCallback` returning `true` unconditionally (disabling SSL verification)
- `HttpOverrides` globally overriding certificate checks
- API base URLs hardcoded instead of using environment-specific configuration

**Severity**: high

### 4. WebView Security

If `webview_flutter`, `flutter_inappwebview`, or similar packages are used:
- JavaScript enabled without necessity (`javascriptMode: JavascriptMode.unrestricted`)
- Loading arbitrary URLs from user input without validation
- Missing `navigationDelegate` to restrict navigation to trusted domains
- JavaScript channels exposing sensitive native functionality

**Severity**: high

### 5. Platform Channel Security

If `MethodChannel` / `EventChannel` / `BasicMessageChannel` are used:
- Sensitive data passed through platform channels without sanitization
- Channel names that leak internal structure (e.g., `com.company.internal.admin.secret`)
- Missing input validation on data received from platform channels
- Error details leaking through `PlatformException` messages

**Severity**: medium

### 6. Debug / Release Mode Issues

- `kDebugMode` or `kReleaseMode` checks that leak information in release builds
- `debugPrint` / `print` statements logging sensitive data
- `assert()` used for security checks (assertions are stripped in release mode)
- Flutter DevTools / inspector left enabled
- `android:debuggable="true"` in `AndroidManifest.xml` release config
- Missing ProGuard/R8 rules for release builds
- Missing code obfuscation (`--obfuscate --split-debug-info`) for release builds

**Severity**: medium

### 7. Deep Link / URL Scheme Security

- Custom URL schemes (`AndroidManifest.xml` intent filters, `Info.plist` URL types) accepting unvalidated input
- Universal links / App Links without proper domain verification
- Deep link handlers that navigate to arbitrary routes based on URL parameters without validation

**Severity**: medium

### 8. Dependency Security

- Check `pubspec.yaml` and `pubspec.lock` for:
  - Packages with known vulnerabilities (check pub.dev advisories if available)
  - Packages pulled from git repositories instead of pub.dev (less audited)
  - Unpinned dependency versions (`any` or overly broad ranges)
  - Outdated packages with major version gaps
- Check for dependency_overrides in production (acceptable in development only)

**Severity**: varies (high for known CVEs, low for version hygiene)

### 9. Authentication & Session Management

- Auth tokens stored insecurely (see check 2)
- Missing token refresh / expiry handling
- Biometric authentication bypasses (using `local_auth` without server-side verification)
- Session not invalidated on logout (tokens not cleared)

**Severity**: high

### 10. Input Validation & Injection

- User input directly interpolated into SQL queries (SQLite injection)
- User input directly interpolated into HTML rendered in WebView (XSS)
- User input passed to `Process.run()` or platform channels without sanitization (command injection)
- Missing input validation on form fields handling sensitive data

**Severity**: high

## Severity Criteria

- **critical** — Immediate risk of credential exposure or data breach. Must fix before release.
- **high** — Exploitable vulnerability or insecure handling of sensitive data.
- **medium** — Defense-in-depth issue or information leakage risk.
- **low** — Best practice deviation with minimal direct security impact.
- **info** — Informational observation or recommendation.

## Output Format

```
## Security Scan Summary

- Project: {project name from pubspec.yaml}
- Scanned: {number of files scanned}
- Findings: {total} (critical: X, high: Y, medium: Z, low: W)

## Findings

### [{severity}] {title}
- **File**: `file_path:line_number`
- **Issue**: Description of the security issue
- **Risk**: What could go wrong
- **Fix**: Recommended remediation

...
```

When no issues are found:

```
## Security Scan Summary

No security issues found in the scanned Flutter/Dart code.
```

Output language follows the user's language.

## Guidelines

- Focus on actionable findings. Avoid theoretical risks with no practical attack vector.
- Understand context: a hardcoded URL pointing to a public API docs page is not a secret.
- Do not flag test-only code unless it could accidentally ship to production.
- Consider the Flutter build modes: debug, profile, release. Some issues only matter in release.
- When in doubt about severity, err on the side of reporting (the developer can dismiss).
