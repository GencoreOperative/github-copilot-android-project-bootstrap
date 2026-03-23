# github-copilot-android-project-bootstrap
A simple bootstrap project that we can use to demonstrate creating an Android app + Github CI/CD

## Running GitHub Copilot with a Personal Account

This repo includes a `./copilot` script that authenticates with your personal GitHub account via a token before launching Copilot.

### Setup

1. **Create a Personal Access Token (PAT)**
   - Go to https://github.com/settings/personal-access-tokens/new
   - Under **Permissions**, add **"Copilot Requests"**
   - Generate and copy the token

2. **Save the token to `.token`** in this folder:
   ```bash
   echo 'ghp_yourtoken' > .token
   ```
   The `.token` file is listed in `.gitignore` and will never be committed.

3. **Launch Copilot:**
   ```bash
   ./copilot
   ```

The script reads your token from `.token`, exports it as `GH_TOKEN`, then starts Copilot via `npx`.

### Personal Access Token Permissions:

For security, grant only the minimum required permissions:

**Repository Permissions:**
- Metadata (read-only)

**Account Permissions:**
- Copilot Requests

These minimal permissions allow Copilot to function while limiting exposure if your token is compromised.

## Priming the Build System

Provide these instructions to `copilot` to set up GitHub Actions CI/CD workflows for Android APK building and release automation.

```
Create two GitHub Actions workflow files for Android CI/CD. Prerequisites: Android project with app/build.gradle, settings.gradle, and gradlew in repository root.

1. Create directory: .github/workflows

2. Create .github/workflows/build.yml with:
   - name: "Build APK"
   - on: push (branches: main, develop) and pull_request (branches: main, develop)
   - permissions: contents: read
   - jobs.build: runs-on: ubuntu-latest, timeout-minutes: 30
   - Step 1: actions/checkout@v4
   - Step 2: actions/setup-java@v4 with java-version: 17, distribution: temurin, cache: gradle
   - Step 3: run chmod +x ./gradlew
   - Step 4: run ./gradlew lint
   - Step 5: run ./gradlew test
   - Step 6: run ./gradlew assembleDebug
   - Step 7: actions/upload-artifact@v4 with name: debug-apk, path: app/build/outputs/apk/debug/*.apk, retention-days: 7, if: always()
   - Step 8: run ./gradlew assembleRelease
   - Step 9: actions/upload-artifact@v4 with name: release-apk-unsigned, path: app/build/outputs/apk/release/*.apk, retention-days: 7, if: always()
   - Step 10: actions/upload-artifact@v4 with name: build-reports, path: app/build/reports/, retention-days: 7, if: always()

3. Create .github/workflows/release.yml with:
   - name: "Release Build"
   - on: push (tags: v*)
   - permissions: contents: write
   - jobs.release: runs-on: ubuntu-latest, timeout-minutes: 30
   - Step 1: actions/checkout@v4
   - Step 2: actions/setup-java@v4 with java-version: 17, distribution: temurin, cache: gradle
   - Step 3: run chmod +x ./gradlew
   - Step 4: run ./gradlew assembleRelease
   - Step 5: run ./gradlew bundleRelease
   - Step 6: id: version, run: echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT
   - Step 7: softprops/action-gh-release@v1 with files: app/build/outputs/apk/release/*.apk and app/build/outputs/bundle/release/*.aab, draft: true, prerelease: false, env: GITHUB_TOKEN: secrets.GITHUB_TOKEN
   - Step 8: actions/upload-artifact@v4 with name: release-artifacts-${{ steps.version.outputs.VERSION }}, path: app/build/outputs/apk/release/ and app/build/outputs/bundle/release/, retention-days: 30

4. Verify: Push to main branch; confirm build.yml workflow runs in GitHub Actions. Create a git tag (v1.0.0) and push it; confirm release.yml workflow runs.
```