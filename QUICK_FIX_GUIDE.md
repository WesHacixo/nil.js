# Quick Fix Guide - Critical Security Updates

**⚠️ IMMEDIATE ACTION REQUIRED**

This repository has **3 critical** and **1 high-severity** vulnerabilities in dependencies that need immediate attention.

---

## Step-by-Step Fix Instructions

### Step 1: Backup Current State
```bash
# Create a backup branch
git checkout -b backup-before-security-fixes
git push origin backup-before-security-fixes
git checkout main  # or your working branch
```

### Step 2: Update Critical Dependencies

#### Option A: Automatic Fix (Recommended)
```bash
# Fix all auto-fixable vulnerabilities
npm audit fix

# If some require force update
npm audit fix --force

# Reinstall to ensure clean state
rm -rf node_modules package-lock.json
npm install
```

#### Option B: Manual Fix (More Control)
```bash
# Update vitest and coverage
npm install --save-dev vitest@latest @vitest/coverage-v8@latest

# Update other dev dependencies
npm install --save-dev vite@latest esbuild@latest

# Update babel runtime (transitive)
npm update @babel/runtime

# Regenerate lock file
rm -rf node_modules package-lock.json
npm install
```

### Step 3: Verify the Fixes
```bash
# Check if vulnerabilities are resolved
npm audit

# Expected output should show 0 vulnerabilities
# or significantly reduced count
```

### Step 4: Test Everything
```bash
# Run linter
npm run lint

# Run type checking
npm run lint:types

# Build the project
npm run build

# Run unit tests
npm run test:unit

# If you have access to integration test environment
npm run test:integration
```

### Step 5: Verify Build Artifacts
```bash
# Check that dist files are created
ls -la dist/

# Expected files:
# - niljs.cjs
# - niljs.esm.js
# - niljs.d.ts
```

### Step 6: Commit and Push
```bash
# Add all changes
git add package.json package-lock.json

# Commit with clear message
git commit -m "fix: update dependencies to resolve critical security vulnerabilities

- Update vitest to >=1.6.1 (fixes RCE vulnerability GHSA-9crc-q9x8-hgqq)
- Update axios to >=1.12.0 (fixes SSRF and DoS vulnerabilities)
- Update vite, esbuild, and other dev dependencies
- Run npm audit fix to resolve transitive vulnerabilities

Security scan performed: October 17, 2025"

# Push to your branch
git push origin your-branch-name
```

---

## Verification Checklist

After completing the fixes, verify:

- [ ] `npm audit` shows 0 critical vulnerabilities
- [ ] `npm audit` shows 0 high vulnerabilities
- [ ] `npm run build` completes successfully
- [ ] `npm run test:unit` passes all tests
- [ ] `npm run lint` passes without errors
- [ ] `npm run lint:types` passes without errors
- [ ] All files in `dist/` directory are generated
- [ ] Package size is reasonable (check with `npm run build`)

---

## Expected Results

### Before Fix
```bash
$ npm audit
14 vulnerabilities (6 low, 4 moderate, 1 high, 3 critical)
```

### After Fix
```bash
$ npm audit
0 vulnerabilities
# or minimal low-severity issues only
```

---

## Troubleshooting

### Issue: npm audit fix doesn't resolve all issues
**Solution**: Some vulnerabilities may require breaking changes. Review each one:
```bash
npm audit fix --force
# Review breaking changes carefully
npm test  # Make sure tests still pass
```

### Issue: Tests fail after updates
**Solution**: Check for API changes in updated packages:
```bash
# Revert and update one at a time
git reset --hard HEAD
npm install --save-dev vitest@latest
npm test  # Check if tests pass
# Repeat for other packages
```

### Issue: Build fails after updates
**Solution**: Check rollup and esbuild configurations:
```bash
# Review rollup config
cat rollup/rollup.config.js

# Try cleaning and rebuilding
rm -rf dist node_modules
npm install
npm run build
```

### Issue: Type errors after updates
**Solution**: TypeScript version compatibility:
```bash
# Check TypeScript version
npm list typescript

# Update if needed
npm install --save-dev typescript@latest

# Rebuild type definitions
npm run lint:types
```

---

## Critical Vulnerabilities Fixed

### 1. Vitest RCE (GHSA-9crc-q9x8-hgqq)
- **Before**: vitest 1.6.0
- **After**: vitest >=1.6.1
- **Risk**: Remote Code Execution when accessing malicious website
- **Impact**: Development environment only
- **CVSS**: 9.7 (Critical)

### 2. Axios SSRF (GHSA-jr5f-v2jv-69x6)
- **Before**: axios 1.0.0 - 1.11.0
- **After**: axios >=1.12.0
- **Risk**: Server-Side Request Forgery and credential leakage
- **Impact**: Information disclosure
- **CVSS**: High

### 3. Axios DoS (GHSA-4hjh-wcwx-xvwj)
- **Before**: axios 1.0.0 - 1.11.0
- **After**: axios >=1.12.0
- **Risk**: Denial of Service through lack of data size check
- **Impact**: Service availability
- **CVSS**: 7.5 (High)

### 4. Multiple Vite Bypasses
- **Before**: vite <=6.1.6
- **After**: vite >6.1.6
- **Risk**: Filesystem access control bypasses
- **Impact**: Development environment file disclosure
- **CVSS**: 5.3 (Moderate)

---

## Post-Fix Actions

### Update Documentation
- [ ] Update README.md with new version numbers
- [ ] Update CHANGELOG.md (if exists)
- [ ] Notify users of security update

### Continuous Monitoring
- [ ] Set up Dependabot on GitHub
- [ ] Enable GitHub security advisories
- [ ] Add security scanning to CI/CD

### Communication
- [ ] Announce security update to users
- [ ] Document the vulnerabilities fixed
- [ ] Thank security researchers (if applicable)

---

## Automated Dependency Monitoring Setup

### Enable Dependabot (GitHub)
Create `.github/dependabot.yml`:
```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    reviewers:
      - "your-username"
    labels:
      - "dependencies"
      - "security"
```

### Enable Security Advisories
1. Go to repository Settings
2. Navigate to "Security & analysis"
3. Enable "Dependency graph"
4. Enable "Dependabot alerts"
5. Enable "Dependabot security updates"

---

## Testing Recommendations

After fixing vulnerabilities, run comprehensive tests:

```bash
# Unit tests
npm run test:unit

# Integration tests (if available)
npm run test:integration

# Build and check output
npm run build
ls -lh dist/

# Test in a sample project
cd /tmp
mkdir test-niljs
cd test-niljs
npm init -y
npm install /path/to/nil.js
node -e "const niljs = require('@nilfoundation/niljs'); console.log(niljs);"
```

---

## Alternative: Using npm-check-updates

For a more aggressive update strategy:

```bash
# Install npm-check-updates globally
npm install -g npm-check-updates

# Check what can be updated
ncu

# Update package.json to latest versions
ncu -u

# Install updated packages
npm install

# Run tests
npm test
```

---

## Success Criteria

Your security fix is successful when:

✅ All critical and high vulnerabilities are resolved  
✅ Build process completes without errors  
✅ All tests pass  
✅ Type checking passes  
✅ Linting passes  
✅ Application functionality is maintained  
✅ No new vulnerabilities introduced  
✅ Documentation is updated  

---

## Need Help?

If you encounter issues:

1. Check the full error messages carefully
2. Review the SECURITY_ANALYSIS.md for context
3. Check npm/package documentation for breaking changes
4. Create an issue in the repository with:
   - Error messages
   - Steps to reproduce
   - Your npm and Node.js versions
   - Output of `npm audit`

---

## Timeline Recommendation

- **Day 1**: Review this guide and create backup
- **Day 2**: Apply fixes and test locally
- **Day 3**: Deploy to staging/test environment
- **Day 4**: Monitor for issues
- **Day 5**: Deploy to production
- **Ongoing**: Set up automated monitoring

---

**Priority Level**: 🔴 CRITICAL  
**Estimated Time**: 1-2 hours  
**Risk of Delay**: Increases exposure to known vulnerabilities  

**Start immediately to secure your application.**

---

Last Updated: October 17, 2025
