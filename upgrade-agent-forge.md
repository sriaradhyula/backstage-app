# Agent Forge Plugin Upgrade Guide

## Overview

This guide documents the process for upgrading the `@caipe/plugin-agent-forge` plugin and resolves a critical version display issue that occurred with version 0.3.26.

## Problem Description

### Issue
- UI displayed version `v0.3.25` instead of `v0.3.26` after upgrading
- Version mismatch between `package.json` and built artifacts
- CI environments would show incorrect version information

### Root Cause
The npm package for version 0.3.26 contained **corrupted build artifacts**:
- `package.json` correctly showed version `0.3.26`
- `dist/package.json.esm.js` still contained hardcoded version `0.3.25`
- This happened because the build process wasn't run after the version bump

## Solution Implemented

### 1. Fixed Release Process
Updated the plugin release workflow to ensure proper version embedding:

```bash
# Correct release workflow:
# 1. Update version in package.json
# 2. Rebuild to embed version in dist files
yarn build:all
# 3. Commit both package.json AND dist files
git add . && git commit -m "fix(agent-forge): bump to x.x.x"
# 4. Publish
npm publish
```

### 2. Released Version 0.3.27
- Fixed the build artifacts to contain correct version
- Published to npm with proper version embedding
- Verified `dist/package.json.esm.js` contains `version = "0.3.27"`

## Upgrade Instructions

### For Backstage Applications

1. **Update package.json**:
   ```json
   {
     "dependencies": {
       "@caipe/plugin-agent-forge": "0.3.27"
     }
   }
   ```

2. **Clean install**:
   ```bash
   # Remove old version
   rm -rf node_modules/@caipe
   
   # Install latest version
   yarn install
   ```

3. **Restart development server**:
   ```bash
   source ~/.nvm/nvm.sh && nvm use 20.19.5  # Use compatible Node version
   yarn dev
   ```

4. **Verify fix**:
   - Check UI displays `v0.3.27`
   - Hard refresh browser (Cmd+Shift+R) if needed

### For CI/CD Environments

No special configuration needed - standard `yarn install` will now work correctly with version 0.3.27.

## Technical Details

### Files Affected
- `package.json`: Contains version metadata
- `dist/package.json.esm.js`: Contains hardcoded version for UI display
- Both files must match for correct version display

### Version Check Commands
```bash
# Check package.json version
cat node_modules/@caipe/plugin-agent-forge/package.json | grep '"version"'

# Check built artifacts version
cat node_modules/@caipe/plugin-agent-forge/dist/package.json.esm.js
```

### Expected Output (Version 0.3.27)
```javascript
// dist/package.json.esm.js should contain:
var version = "0.3.27";
var packageInfo = {
    version: version
};
```

## Version History

- **0.3.25**: Last known good version
- **0.3.26**: ❌ Corrupted build artifacts (skip this version)
- **0.3.27**: ✅ Fixed version with correct build artifacts

## Best Practices for Future Releases

### For Plugin Maintainers
1. **Always rebuild after version bumps**:
   ```bash
   # After updating package.json version:
   yarn build:all  # or npm run build
   ```

2. **Verify build artifacts**:
   ```bash
   # Check that dist files contain correct version
   grep -r "version.*=" dist/
   ```

3. **Test locally before publishing**:
   ```bash
   # Link locally and test in a Backstage app
   npm link
   cd /path/to/backstage-app
   npm link @caipe/plugin-agent-forge
   ```

### For Plugin Users
1. **Always use exact versions** for stability:
   ```json
   "@caipe/plugin-agent-forge": "0.3.27"  // Not "^0.3.27"
   ```

2. **Verify version after upgrades**:
   - Check UI displays expected version
   - Test functionality after upgrades

## Troubleshooting

### Issue: Still seeing old version after upgrade

**Solution**: Clear caches and restart
```bash
# Clear yarn cache
yarn cache clean

# Remove node_modules
rm -rf node_modules

# Fresh install
yarn install

# Restart dev server
yarn dev
```

### Issue: Version mismatch in CI

**Solution**: Ensure using version 0.3.27 or later
```bash
# Check installed version
npm list @caipe/plugin-agent-forge
```

## Node.js Compatibility

- **Required**: Node.js 18 or 20
- **Recommended**: Node.js 20.19.5 (tested)
- **Not supported**: Node.js 24+ (incompatible with Backstage engines requirement)

## Contact

For issues related to the agent-forge plugin:
- GitHub: [backstage/community-plugins](https://github.com/backstage/community-plugins)
- Package: [npmjs.com/package/@caipe/plugin-agent-forge](https://www.npmjs.com/package/@caipe/plugin-agent-forge)

---
**Last Updated**: October 22, 2025  
**Plugin Version**: 0.3.27  
**Backstage Compatibility**: 1.42+
