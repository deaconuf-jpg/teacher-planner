# Teacher Planner - Comprehensive Audit Report

**Date:** 2026-05-14  
**Repo:** deaconuf-jpg/teacher-planner  
**File:** teacher-planner.html

---

## Executive Summary

The Teacher Planner is a well-designed single-page application with **solid UI/UX design** but has **critical accessibility issues**, **incomplete responsive behavior**, **inconsistent dark mode support**, and **missing error handling**. 

**Grade: C+** (Good design, poor execution on accessibility and completeness)

---

## Critical Issues (Must Fix)

### 1. **JavaScript Code is Truncated** ⚠️ BLOCKING
- **Severity:** CRITICAL
- **Impact:** Core functionality missing
- **Location:** File ends mid-script around line 1000
- **What's Missing:** State management, event handlers, modal controls, drag-drop, keyboard shortcuts
- **Fix:** Restore complete JavaScript implementation

### 2. **Accessibility Violations - Form Labels** 🚨
- **Severity:** HIGH
- **WCAG:** Fails 1.3.1 Info and Relationships
- **Issue:** Form labels missing `for` attributes

```html
<!-- ❌ WRONG - Multiple locations -->
<label>Title</label>
<input type="text" id="editTaskTitle" />

<label>Linked class</label>
<select id="editTaskClass"></select>

<!-- ✅ CORRECT -->
<label for="editTaskTitle">Title</label>
<input type="text" id="editTaskTitle" />

<label for="editTaskClass">Linked class</label>
<select id="editTaskClass"></select>
```

**Affected Elements:**
- Lesson modal: Class, Topic, Notes, Homework
- Task edit modal: Title, Due, Priority, Linked class, Notes
- Settings modal: Subtitle, all period inputs, all class inputs

### 3. **Accessibility Violations - Modals** 🚨
- **Severity:** HIGH
- **WCAG:** Fails 1.3.1 and 4.1.2 Name Role Value
- **Issues:**
  - Missing `role="dialog"`
  - Missing `aria-labelledby`
  - Missing `aria-modal="true"`
  - No focus trap implementation

```html
<!-- ❌ WRONG -->
<div class="modal-backdrop" id="lessonModal">
  <div class="modal">
    <h3 id="lessonTitle">Lesson</h3>

<!-- ✅ CORRECT -->
<div class="modal-backdrop" id="lessonModal" role="presentation">
  <div class="modal" role="dialog" aria-labelledby="lessonTitle" aria-modal="true">
    <h3 id="lessonTitle">Lesson</h3>
```

### 4. **Dark Mode Support Incomplete** 🌙
- **Severity:** HIGH
- **Issues:**
  - CSS variables defined but not applied to inputs/selects
  - Hardcoded colors don't adapt to dark mode
  - Scrollbar theme static
  - No state persistence

```css
/* ❌ Missing in dark mode */
html.dark-mode .field input[type="text"],
html.dark-mode .field select,
html.dark-mode .field textarea {
  background: var(--panel);
  color: var(--text);
}

/* ❌ Hardcoded color (line 833) */
.bf-name code { background: rgba(0,0,0,.05); }

/* ✅ Should be */
html.dark-mode .bf-name code { background: rgba(255,255,255,.1); }
```

---

## High-Priority Issues

### 5. **Responsive Design - Breakpoint Chaos**
- **Severity:** HIGH
- **Breakpoints used:** 480px, 768px, 1100px (inconsistent)
- **Issues:**
  - Week nav buttons wrap on 480px but not optimized
  - Task add form doesn't flex-wrap consistently
  - Settings grid jumps at wrong breakpoint

**Fix:**
```css
/* Consolidate to: mobile (640px), tablet (1024px), desktop */
@media (max-width: 640px) { /* mobile */ }
@media (max-width: 1024px) { /* tablet */ }
@media (min-width: 1025px) { /* desktop */ }
```

### 6. **Error Handling - Storage Access**
- **Severity:** HIGH
- **Issues:**
  - No try/catch around localStorage
  - No check if localStorage is full
  - No graceful fallback
  - Quota exceeded error not handled

```javascript
/* ❌ Current approach likely has no error handling */

/* ✅ Should be */
function saveState(state) {
  try {
    const data = JSON.stringify(state);
    const bytes = new Blob([data]).size;
    
    if (bytes > 5 * 1024 * 1024) {
      showToast('Too much data to save', 'error');
      return false;
    }
    
    localStorage.setItem(STORAGE_KEY, data);
    return true;
  } catch (err) {
    if (err.name === 'QuotaExceededError') {
      showToast('Storage full - delete old snapshots', 'error');
    } else {
      showToast('Save failed: ' + err.message, 'error');
    }
    return false;
  }
}
```

### 7. **Keyboard Navigation - Incomplete Implementation**
- **Severity:** HIGH
- **Issues:**
  - Keyboard hints mention `Ctrl+Z` (undo) - not clear if implemented
  - Help modal (`?` key) referenced but no implementation visible
  - Focus not trapped in modals
  - No focus restoration after modal close

---

## Medium-Priority Issues

### 8. **CSS Redundancy & Maintainability**
- **Severity:** MEDIUM
- **Examples:**
  - Color `rgba(0,0,0,.05)` repeated 3+ times
  - Button hover states hardcoded instead of using variables
  - Font sizes not consolidated

```css
/* ❌ Line 75, 833, 602 - same color, different names */
background: linear-gradient(135deg, #2563eb, #1d4ed8);
background: rgba(0,0,0,.05);

/* ✅ Should be */
--gradient-accent: linear-gradient(135deg, var(--accent), #1d4ed8);
--bg-overlay: rgba(0,0,0,.05);
```

### 9. **Modal Form Fields - Missing Associations**
- **Severity:** MEDIUM
- **Issues:**
  - Textarea fields for notes lack labels
  - No `aria-describedby` pointing to help text
  - No visual required field indicator

### 10. **Task List - Overflow on Mobile**
- **Severity:** MEDIUM
- **Issue:** max-height: 70vh can cut off tasks on small screens
- **Fix:** Make responsive

```css
.task-list {
  max-height: 70vh;  /* ❌ Too tall on mobile */
  /* ✅ Should be */
  max-height: 50vh;  /* or responsive */
}
```

### 11. **Button Styling - Inconsistencies**
- **Severity:** MEDIUM
- **Issues:**
  - Primary hover uses hardcoded `#1d4ed8` (line 127)
  - No disabled state styling
  - Secondary buttons lack hover effects in some contexts

```css
button.btn.primary:hover { 
  background: #1d4ed8;  /* ❌ Hardcoded */
  /* ✅ Should be */
  background: color-mix(in srgb, var(--accent) 85%, black);
}
```

### 12. **Help Modal - Incomplete Content**
- **Severity:** MEDIUM
- **Issue:** Modal content is truncated (marked with `[...]` in HTML)
- **Impact:** Users can't see complete help information

---

## Low-Priority Issues

### 13. **Browser Compatibility**
- **Severity:** LOW
- **Issues:**
  - `-webkit-line-clamp` needs `-moz` fallback
  - File System Access API (modern browsers only)

```css
/* Add fallback */
.cell-topic {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  max-height: 2.7em;  /* Fallback for Firefox */
}
```

---

## Summary of Fixes

| Issue | Fix | Time | Impact |
|-------|-----|------|--------|
| Truncated JS | Restore/rebuild | 2-4h | CRITICAL |
| Label associations | Add `for` attributes | 30min | HIGH |
| Modal ARIA | Add roles and attributes | 20min | HIGH |
| Dark mode inputs | Add CSS variables | 15min | HIGH |
| Error handling | Add try/catch | 1h | HIGH |
| Responsive | Consolidate breakpoints | 1h | HIGH |
| CSS redundancy | Extract variables | 45min | MEDIUM |
| Help modal | Complete content | 30min | MEDIUM |

---

## Testing Checklist

- [ ] Screen reader test (NVDA/JAWS/VoiceOver)
- [ ] Keyboard navigation (Tab, Shift+Tab, Esc)
- [ ] Dark mode toggle and persistence
- [ ] Responsive at 320px, 480px, 768px, 1024px
- [ ] localStorage full scenario
- [ ] File import/export error handling
- [ ] All keyboard shortcuts (arrows, T, ?, Ctrl+Z)

---

## Files to Update

1. `teacher-planner.html` - Main fixes
2. Create `README_ACCESSIBILITY.md` - Accessibility statement
3. Create `.github/SECURITY.md` - Data handling policy

---

## References

- [WCAG 2.1](https://www.w3.org/WAI/WCAG21/quickref/)
- [ARIA Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [MDN Form Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/dialog_role)
