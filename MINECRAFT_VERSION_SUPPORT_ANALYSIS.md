# Minecraft Version Support Analysis

**Date:** January 15, 2026  
**Toolkit Version:** 0.6.10

## Executive Summary

This document analyzes the current state of Minecraft version support in Essential Gradle Toolkit and provides a plan for adding support for new versions.

## Current Status

### Maximum Supported Version

**Minecraft 1.21.8** (internal code: `12108`)

This is the most recent version currently supported in the toolkit, with full support for:
- **Fabric** with Yarn mappings: `1.21.8+build.1:v2`
- **Forge**: `1.21.8-58.0.0`
- **NeoForge**: `21.8.5-beta`

### Version Encoding System

The toolkit uses a numeric encoding system for Minecraft versions:

```
code = mcMajor * 10000 + mcMinor * 100 + mcPatch
```

**Examples:**
- Minecraft 1.8.9 → `10809`
- Minecraft 1.21.8 → `12108`
- Minecraft 1.22.0 → `12200` (hypothetical)

### Supported Loaders

1. **Fabric** - All versions since 1.14.4
2. **Forge** - All versions since 1.7.10
3. **NeoForge** - Modern versions since 1.20.2

### Currently Supported Version Ranges

#### Legacy Versions (< 1.14.0)
- 1.7.10 (Forge)
- 1.8.0, 1.8.9 (Forge)
- 1.9.4 (Forge)
- 1.10.2 (Forge)
- 1.11.0, 1.11.2 (Forge)
- 1.12.0, 1.12.1, 1.12.2 (Forge)

#### Modern Versions (≥ 1.14.0)
- 1.14 Series: 1.14.4
- 1.15 Series: 1.15.2
- 1.16 Series: 1.16.1, 1.16.2, 1.16.4, 1.16.5
- 1.17 Series: 1.17.0, 1.17.1
- 1.18 Series: 1.18.1, 1.18.2
- 1.19 Series: 1.19.0, 1.19.1, 1.19.2, 1.19.3, 1.19.4
- 1.20 Series: 1.20.0, 1.20.1, 1.20.2, 1.20.3, 1.20.4, 1.20.5, 1.20.6
- 1.21 Series: 1.21.0, 1.21.1, 1.21.2, 1.21.3, 1.21.4, 1.21.5, 1.21.6, 1.21.7, 1.21.8

## Analysis of Potential New Versions

### Potential Future Versions

Based on Minecraft release patterns:

1. **Minecraft 1.21.9+** (code: `12109+`)
   - Potential additional patches in the 1.21 series
   - Would require: Yarn mappings, Forge, NeoForge

2. **Minecraft 1.22.x** (code: `12200+`)
   - Next minor Minecraft version
   - Would require: Yarn mappings, Forge (if available), NeoForge
   - Potential change in Java requirements (currently Java 21 for ≥1.20.5)

3. **Minecraft 1.23.x and beyond** (code: `12300+`)
   - Future versions
   - May require significant updates

## Process for Adding Support for a New Version

### Main File: `src/main/kotlin/gg/essential/defaults/loom.gradle.kts`

This file contains all version configurations in the `Revision` structure.

### Required Steps

#### 1. Verify Dependency Availability

Before adding support, verify that the following exist:

- **Yarn mappings** (for Fabric):
  - Format: `net.fabricmc:yarn:<version>+build.<build>:v2`
  - Example: `1.21.8+build.1:v2`
  - Source: https://maven.fabricmc.net/

- **Forge**:
  - Format: `net.minecraftforge:forge:<mcVersion>-<forgeVersion>`
  - Example: `1.21.8-58.0.0`
  - Source: https://files.minecraftforge.net/

- **NeoForge**:
  - Format: `net.neoforged:neoforge:<version>`
  - Example: `21.8.5-beta`
  - Source: https://maven.neoforged.net/

#### 2. Update loom.gradle.kts

Add new versions to the **first** `Revision` in the file:

```kotlin
revisions.add(Revision(
    yarn = mapOf(
        // ... existing versions ...
        12109 to "1.21.9+build.1:v2",  // New version
    ),
    forge = mapOf(
        // ... existing versions ...
        12109 to "1.21.9-XX.X.X",  // New version
    ),
    neoForge = mapOf(
        // ... existing versions ...
        12109 to "21.9.X-beta",  // New version
    ),
    // ... other fields ...
))
```

#### 3. Update Platform.kt (if necessary)

If the new version requires a different Java version, update the `javaVersion` method in `Platform.kt`:

```kotlin
val javaVersion = when {
    mcVersion >= 12XXX -> JavaVersion.VERSION_XX  // If there are changes
    mcVersion >= 12005 -> JavaVersion.VERSION_21
    mcVersion >= 11800 -> JavaVersion.VERSION_17
    mcVersion >= 11700 -> JavaVersion.VERSION_16
    else -> JavaVersion.VERSION_1_8
}
```

#### 4. Increment Revision

If modifying existing versions (not just adding new ones), create a new revision:

```kotlin
revisions.add(revisions.last().update(
    // Only the versions that change
))
```

### Important Considerations

1. **Add to first Revision**: New versions are added to the first `Revision` so they're available to all users.

2. **Create new Revision for changes**: Only create a new revision when modifying existing entries, to maintain compatibility with users on old revisions.

3. **Fabric Loader**: The fabric-loader version (currently `0.13.3`) should be updated if new Minecraft versions require it.

4. **Special mappings**: 
   - Modern Forge (≥1.17.0) uses official Mojang mappings
   - Legacy Forge uses MCP mappings

## Limitations and Restrictions

### Technical Limitations

1. **External ecosystem dependency**:
   - Cannot add support until Yarn, Forge, and/or NeoForge publish compatible versions
   - Release times vary between platforms

2. **Architectury Loom**:
   - The toolkit depends on architectury-loom
   - New Minecraft versions require Loom to support them first

3. **Java requirements**:
   - New versions may require newer Java versions
   - This affects compatibility with existing projects

### Maintenance Considerations

1. **Test versions**:
   - Snapshots and pre-releases are generally not added
   - Only stable release versions

2. **Dependency updates**:
   - Yarn mappings sometimes receive updates (higher build number)
   - Forge may publish patched versions
   - These updates can be added through new revisions

## Recommendations

### Short Term (Next 3 months)

1. **Monitor Minecraft releases**:
   - Minecraft 1.21.9 or higher if released
   - Update as soon as Yarn/Forge/NeoForge are available

2. **Keep Fabric Loader updated**:
   - Check for newer fabric-loader versions
   - Update if necessary for compatibility

### Medium Term (3-6 months)

1. **Prepare for Minecraft 1.22**:
   - When officially announced
   - Verify changes in Java requirements
   - Update Platform.kt if necessary

2. **Review obsolete versions**:
   - Consider deprecating support for very old versions if needed
   - Document any support policy changes

### Long Term (6+ months)

1. **Automation**:
   - Consider scripts to automatically check for new dependency versions
   - Automate part of the update process if recurring

2. **Documentation**:
   - Keep this document updated
   - Document any changes in the process

## Proposed Update Procedure

### Checklist for Adding a New Version

- [ ] Verify that Minecraft has released the new version
- [ ] Wait for Yarn to publish mappings (usually 1-2 days later)
- [ ] Wait for Forge to publish a compatible version (varies)
- [ ] Wait for NeoForge to publish a compatible version (varies)
- [ ] Calculate version code (mcMajor * 10000 + mcMinor * 100 + mcPatch)
- [ ] Update `loom.gradle.kts` with new entries in the map
- [ ] Verify if `Platform.kt` needs updating (Java requirements)
- [ ] Verify if fabric-loader needs updating
- [ ] Test build with the new version
- [ ] Increment toolkit version
- [ ] Update CHANGELOG (if exists)
- [ ] Create commit and PR

## Key Files

1. **`src/main/kotlin/gg/essential/defaults/loom.gradle.kts`**
   - Definitions of all supported versions
   - Mappings, Forge, NeoForge, Fabric Loader

2. **`src/main/kotlin/gg/essential/gradle/multiversion/Platform.kt`**
   - Version detection logic
   - Java requirements per version
   - Loader detection

3. **`build.gradle.kts`**
   - Toolkit version
   - Toolkit's own dependencies

## Conclusion

The Essential Gradle Toolkit currently supports up to **Minecraft 1.21.8** with a well-structured and maintainable system. Adding support for new versions is a straightforward process that mainly requires:

1. Waiting for external dependencies (Yarn, Forge, NeoForge) to be available
2. Updating the version maps in `loom.gradle.kts`
3. Verifying Java requirements if necessary

The revision system maintains backward compatibility while adding new functionality, making maintenance sustainable in the long term.
