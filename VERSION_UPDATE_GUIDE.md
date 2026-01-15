# Quick Guide: How to Add Support for New Minecraft Versions

## Summary

**Current maximum supported version:** Minecraft 1.21.8 (code: 12108)

This guide explains how to add support for newer Minecraft versions when they become available.

## How High Can We Go?

The toolkit can support **any future Minecraft version** as long as the necessary dependencies exist:

- ✅ **Yarn mappings** (for Fabric) - Available at https://maven.fabricmc.net/
- ✅ **Forge** (for Forge) - Available at https://files.minecraftforge.net/
- ✅ **NeoForge** (for NeoForge) - Available at https://maven.neoforged.net/

### Potential Versions

| Version | Code | Status | Notes |
|---------|------|--------|-------|
| 1.21.9 | 12109 | Awaiting release | Next possible patch |
| 1.21.10 | 12110 | Awaiting release | Subsequent patch |
| 1.22.0 | 12200 | Awaiting release | Next minor version |

## Steps to Add a New Version

### 1. Verify Availability

Wait for the following to be available:
- ✅ Official Minecraft release
- ✅ Yarn mappings (1-2 days after release)
- ✅ Compatible Forge (variable time)
- ✅ Compatible NeoForge (variable time)

### 2. Calculate Version Code

```
code = (major * 10000) + (minor * 100) + patch
```

**Example for Minecraft 1.21.9:**
```
code = (1 * 10000) + (21 * 100) + 9 = 12109
```

### 3. Edit `loom.gradle.kts`

Open: `src/main/kotlin/gg/essential/defaults/loom.gradle.kts`

Add new versions in the **first** `Revision` block:

```kotlin
revisions.add(Revision(
    yarn = mapOf(
        // ... existing versions ...
        12109 to "1.21.9+build.1:v2",  // ← ADD HERE
    ),
    forge = mapOf(
        // ... existing versions ...
        12109 to "1.21.9-XX.X.X",  // ← ADD HERE (with correct version)
    ),
    neoForge = mapOf(
        // ... existing versions ...
        12109 to "21.9.X-beta",  // ← ADD HERE (with correct version)
    ),
    // ... rest of configuration ...
))
```

### 4. Verify Java Requirements

If the new version requires a different Java version, edit `Platform.kt`:

Open: `src/main/kotlin/gg/essential/gradle/multiversion/Platform.kt`

```kotlin
val javaVersion = when {
    // Add new line if Java requirement changes
    mcVersion >= XXXXX -> JavaVersion.VERSION_XX
    mcVersion >= 12005 -> JavaVersion.VERSION_21  // Current for 1.20.5+
    mcVersion >= 11800 -> JavaVersion.VERSION_17
    mcVersion >= 11700 -> JavaVersion.VERSION_16
    else -> JavaVersion.VERSION_1_8
}
```

### 5. Update Toolkit Version

Edit `build.gradle.kts` (project root):

```kotlin
version = "0.6.11"  // Increment version
```

### 6. Test

```bash
./gradlew build
```

## Complete Example: Adding Minecraft 1.21.9

Suppose we want to add support for Minecraft 1.21.9:

### Step 1: Gather Information

- **Minecraft Version:** 1.21.9
- **Code:** 12109
- **Yarn mappings:** `1.21.9+build.1:v2` (example)
- **Forge:** `1.21.9-58.1.0` (example)
- **NeoForge:** `21.9.1-beta` (example)
- **Required Java:** 21 (same as 1.20.5+, no changes needed)

### Step 2: Edit loom.gradle.kts

```kotlin
revisions.add(Revision(
    yarn = mapOf(
        12109 to "1.21.9+build.1:v2",  // ← NEW LINE
        12108 to "1.21.8+build.1:v2",
        12107 to "1.21.7+build.6:v2",
        // ... rest of versions ...
    ),
    mcp = mapOf(
        // No changes, old versions
    ),
    fabricLoader = "0.13.3",
    forge = mapOf(
        12109 to "1.21.9-58.1.0",  // ← NEW LINE
        12108 to "1.21.8-58.0.0",
        12107 to "1.21.7-57.0.2",
        // ... rest of versions ...
    ),
    neoForge = mapOf(
        12109 to "21.9.1-beta",  // ← NEW LINE
        12108 to "21.8.5-beta",
        12107 to "21.7.11-beta",
        // ... rest of versions ...
    )
))
```

### Step 3: Update build.gradle.kts

```kotlin
version = "0.6.11"
```

### Step 4: Commit

```bash
git add .
git commit -m "Add support for Minecraft 1.21.9"
```

## Important Changes by Version

### Changes in Java Requirements

| MC Version | Required Java | Change needed in Platform.kt |
|------------|--------------|------------------------------|
| 1.7.10 - 1.16.5 | Java 8 | No |
| 1.17.0 - 1.17.1 | Java 16 | No |
| 1.18.0 - 1.19.4 | Java 17 | No |
| 1.20.5+ | Java 21 | No |
| 1.22.0+ | Java 21? | **Verify when released** |

### Changes in Mappings

- **Forge ≥ 1.17.0:** Uses official Mojang mappings automatically
- **Forge < 1.17.0:** Uses MCP mappings (already configured)
- **Fabric:** Always uses Yarn mappings

## Frequently Asked Questions

### Can I add support for snapshots?

Not recommended. The toolkit only supports stable release versions.

### What if Forge isn't available yet?

You can add only Fabric/NeoForge support and update Forge later by creating a new revision.

### Do I need to update Fabric Loader?

Only if new Minecraft versions require it. Check Fabric documentation.

### How do I know which Yarn/Forge/NeoForge versions to use?

- **Yarn:** https://maven.fabricmc.net/net/fabricmc/yarn/
- **Forge:** https://files.minecraftforge.net/net/minecraftforge/forge/
- **NeoForge:** https://maven.neoforged.net/releases/net/neoforged/neoforge/

### What is the "revisions" system?

Revisions allow updating existing versions without breaking projects using old toolkit versions.

- **Add new version:** Modify the first `Revision`
- **Update existing version:** Create new revision with `.update()`

## Files to Modify

| File | When to modify | What to modify |
|------|----------------|----------------|
| `loom.gradle.kts` | Always | Add mappings/versions |
| `Platform.kt` | If Java req. changes | Update `javaVersion` |
| `build.gradle.kts` | Always | Increment toolkit version |

## Complete Checklist

- [ ] New Minecraft version released
- [ ] Yarn mappings available
- [ ] Forge available (optional, can be added later)
- [ ] NeoForge available (optional, can be added later)
- [ ] Version code calculated
- [ ] `loom.gradle.kts` updated
- [ ] `Platform.kt` updated (if necessary)
- [ ] `build.gradle.kts` version incremented
- [ ] Successful build (`./gradlew build`)
- [ ] Commit created
- [ ] PR opened

## Support

For more technical details, see: `MINECRAFT_VERSION_SUPPORT_ANALYSIS.md`
