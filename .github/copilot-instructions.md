# Copilot Instructions for SuppressViewpunch Plugin

## Repository Overview

This repository contains a SourcePawn plugin for SourceMod that suppresses viewpunch effects when players perform hard landings in Counter-Strike: Global Offensive. The plugin uses function detouring via DHooks to intercept and suppress the `CGameMovement::PlayerRoughLandingEffects` function.

**Key Purpose**: Eliminate screen shake/viewpunch when players slam into the ground, improving gameplay experience by removing this jarring visual effect.

## Architecture & Technical Stack

### Core Components
- **Main Plugin**: `scripting/SuppressViewpunch.sp` - The SourcePawn source code
- **Game Data**: `gamedata/SuppressViewpunch.games.txt` - Contains memory signatures for function location
- **Dependencies**: DHooks extension (for function detouring)

### How It Works
1. Plugin loads gamedata file containing memory signatures for `CGameMovement::PlayerRoughLandingEffects`
2. DHooks creates a detour for this function using the signature
3. When the game tries to call the viewpunch function, our detour intercepts it
4. If `sm_suppress_viewpunch` cvar is enabled (default: 1), the function call is suppressed
5. If disabled, the original function executes normally

## File Structure & Responsibilities

```
/
├── scripting/
│   └── SuppressViewpunch.sp          # Main plugin source code
├── gamedata/
│   └── SuppressViewpunch.games.txt   # Memory signatures for CS:GO
└── .github/
    └── copilot-instructions.md       # This file
```

### Key Files Deep Dive

**SuppressViewpunch.sp**:
- Plugin information and version (currently 1.1)
- ConVar registration for `sm_suppress_viewpunch`
- Gamedata loading and DHook setup
- Detour callback function

**SuppressViewpunch.games.txt**:
- Game-specific memory signatures for Linux and Windows
- Function calling convention and parameter definitions
- Critical for plugin functionality - must match game's memory layout

## Development Workflow

### Building the Plugin
```bash
# Compile using SourceMod compiler (requires SourceMod installation)
spcomp -i/path/to/sourcemod/scripting/include scripting/SuppressViewpunch.sp
```

### Testing Changes
1. **Local Testing**: Deploy to a test CS:GO server with SourceMod
2. **Functional Testing**: 
   - Jump from height to trigger hard landing
   - Verify viewpunch is suppressed when cvar is 1
   - Verify viewpunch occurs when cvar is 0
3. **Console Verification**: Check for plugin load errors in server console

### Installation on Server
```
gameserver/
├── addons/sourcemod/
│   ├── plugins/SuppressViewpunch.smx    # Compiled plugin
│   └── gamedata/SuppressViewpunch.games.txt  # Copy gamedata file
```

## Common Development Tasks

### Adding Support for New Games
1. Research memory signatures for target game
2. Add new game section to `SuppressViewpunch.games.txt`
3. Test signatures work correctly
4. Update plugin info/documentation

### Updating Game Signatures (After Game Updates)
1. Game updates often break memory signatures
2. Use signature scanning tools to find new addresses
3. Update Linux/Windows signatures in gamedata file
4. Test thoroughly before deployment

### Adding Configuration Options
```sourcepawn
// Add new ConVars in OnPluginStart()
ConVar g_cvarNewOption = CreateConVar("sm_suppress_option", "1", "Description");

// Use in detour callback
public MRESReturn Detour_PlayerRoughLandingEffects(Handle hParams)
{
    if (g_cvarSuppressViewpunch.BoolValue && g_cvarNewOption.BoolValue)
        return MRES_Supercede;
    return MRES_Ignored;
}
```

### Version Updates
1. Update version string in plugin info block
2. Update any relevant documentation
3. Test compilation and functionality
4. Tag release appropriately

## Code Style & Standards (SourcePawn Specific)

### Required Pragmas
```sourcepawn
#pragma semicolon 1      // Require semicolons
#pragma newdecls required // Require new declaration syntax
```

### Naming Conventions
- **Global variables**: `g_` prefix (e.g., `g_cvarSuppressViewpunch`)
- **Handles**: descriptive names with Handle suffix (e.g., `g_hPlayerRoughLandingEffects`)
- **Functions**: PascalCase (e.g., `Detour_PlayerRoughLandingEffects`)
- **Local variables**: camelCase

### Memory Management Best Practices
```sourcepawn
// Always clean up handles - use delete, not CloseHandle()
delete hGameData;  // Preferred method
// No need to check for null before delete - it handles null safely

// For StringMaps/ArrayLists - use delete, never .Clear()
StringMap map = new StringMap();
// ... use map ...
delete map;  // Don't use map.Clear() - causes memory leaks
map = new StringMap();  // Create new instance if needed

// Handle validation before use
if (!hGameData)
    SetFailState("Failed to load gamedata");
```

## Testing & Validation

### Manual Testing Checklist
- [ ] Plugin compiles without errors/warnings
- [ ] Plugin loads successfully on server
- [ ] ConVar `sm_suppress_viewpunch` is registered
- [ ] Hard landings are suppressed when cvar = 1
- [ ] Hard landings work normally when cvar = 0
- [ ] No console errors during operation
- [ ] Plugin unloads cleanly

### Common Issues & Debugging
1. **Plugin fails to load**: Check gamedata file path and signatures
2. **Detour fails**: Game updated - signatures need updating
3. **Functionality doesn't work**: Verify DHooks extension is loaded
4. **Memory leaks**: Ensure all handles are properly deleted

## Deployment Considerations

### Server Requirements
- **SourceMod 1.12+** (latest stable recommended - this is the minimum supported version)
- **DHooks extension** installed and loaded (critical dependency)
- **Counter-Strike: Global Offensive** dedicated server
- **64-bit game server** (signatures are for 64-bit architecture)

### Production Deployment
1. Test on staging server first
2. Deploy during low-traffic periods
3. Monitor server console for errors
4. Have rollback plan ready

### Performance Impact
- Minimal - only intercepts one function call
- No ongoing timers or heavy processing
- Safe for production use

## Troubleshooting Common Issues

### Gamedata Signature Failures
**Symptoms**: Plugin fails to load, detour setup errors
**Solutions**:
1. Check if game recently updated
2. Update signatures using signature scanning tools
3. Verify gamedata file syntax is correct

### DHooks Extension Missing
**Symptoms**: Plugin fails to load with DHooks-related errors
**Solutions**:
1. Install DHooks extension
2. Verify extension loads before plugin
3. Check SourceMod extension requirements

### Function Not Being Detoured
**Symptoms**: Viewpunch still occurs despite plugin loaded
**Solutions**:
1. Verify detour was enabled successfully
2. Check if correct function is being called in-game
3. Test with debug prints in detour callback

## Future Enhancement Ideas

1. **Multi-game Support**: Add signatures for CS:S, TF2, etc.
2. **Player-specific Control**: Allow per-player suppression settings
3. **Alternative Effects**: Replace viewpunch with subtle alternative feedback
4. **Admin Commands**: Add admin commands for runtime configuration
5. **Statistics Tracking**: Track suppression events for server analytics

## Important Notes for AI Agents

- **Memory Signatures**: These are fragile and break with game updates. Always verify gamedata is current.
- **DHooks Dependency**: Plugin is completely dependent on DHooks extension - ensure it's available.
- **Testing Requirements**: Always test on actual game server - signatures may work on one OS but not another.
- **Game-Specific**: Currently only supports CS:GO. Other Source games need different signatures.
- **Version Management**: Keep plugin version in sync with repository tags for clarity.
- **Error Handling**: Always check for failures when setting up detours - fail gracefully with clear error messages.
- **SourceMod Version**: Requires SourceMod 1.12+ minimum - use latest stable for best compatibility.
- **Performance**: This plugin has O(1) complexity - just intercepts one function call per hard landing.
- **No Database**: This plugin doesn't use databases, but if adding SQL: always use async queries with methodmaps.
- **No Translations**: This plugin has minimal user-facing text, but for extensions use translation files (.phrases.txt).
- **Methodmaps**: Not used in this simple plugin, but preferred for complex native function implementations.