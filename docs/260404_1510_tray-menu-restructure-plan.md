# Tray Menu Restructure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure macOS tray menu to promote frequent actions (pause indefinitely, long break, mini break) to top-level items, removing the "Skip to the next" submenu.

**Architecture:** Modify `getTrayMenuTemplate()` in `app/main.js` to reorder and flatten menu items. Add `pauseIndefinitely` i18n key to all 53 locale files.

**Tech Stack:** Electron (Tray/Menu), i18next

---

### Task 1: Add `pauseIndefinitely` i18n key to all locale files

**Files:**
- Modify: `app/locales/en.json:17` (insert after `indefinitely`)
- Modify: all other 54 `app/locales/*.json` files (insert `pauseIndefinitely` in `main` object)

- [ ] **Step 1: Add key to `en.json`**

In `app/locales/en.json`, after line 17 (`"indefinitely": "Indefinitely",`), insert:

```json
"pauseIndefinitely": "Pause indefinitely",
```

- [ ] **Step 2: Add key to all other locale files**

Add `"pauseIndefinitely"` to the `main` object of each locale file. Translations:

| File | Value |
|------|-------|
| `ajp.json` | `"إيقاف مؤقت لأجل غير مسمى"` |
| `arz.json` | `"إيقاف مؤقت لأجل غير مسمى"` |
| `be.json` | `"Прыпыніць на неазначаны тэрмін"` |
| `bg.json` | `"Пауза за неопределено време"` |
| `bn.json` | `"অনির্দিষ্ট কালের জন্যে বিরতি"` |
| `ca.json` | `"Pausa indefinidament"` |
| `cs.json` | `"Pozastavit na neurčito"` |
| `cy.json` | `"Oedi am gyfnod amhenodol"` |
| `da.json` | `"Pause for evigt"` |
| `de.json` | `"Unbegrenzt pausieren"` |
| `el.json` | `"Παύση επ' αόριστον"` |
| `eo.json` | `"Paŭzi senfine"` |
| `es.json` | `"Pausar indefinidamente"` |
| `et.json` | `"Peata määramata ajaks"` |
| `eu.json` | `"Pausatu mugagaberik"` |
| `fa.json` | `"مکث نامحدود"` |
| `fi.json` | `"Tauko toistaiseksi"` |
| `fil.json` | `"I-pause nang walang katapusan"` |
| `fr.json` | `"Pause indéfiniment"` |
| `gl.json` | `"Pausar indefinidamente"` |
| `gu.json` | `"અનિશ્ચિત સમય માટે વિરામ"` |
| `he.json` | `"השהה ללא הגבלת זמן"` |
| `hi.json` | `"अनिश्चित काल के लिए रोकें"` |
| `hu.json` | `"Szüneteltetés határozatlan ideig"` |
| `id.json` | `"Jeda tanpa batas"` |
| `it.json` | `"Pausa indefinitamente"` |
| `ja.json` | `"無期限で一時停止"` |
| `ko.json` | `"무기한 일시 중지"` |
| `lt.json` | `"Pristabdyti neribotam laikui"` |
| `lv.json` | `"Pauzēt bezgalīgi"` |
| `mk.json` | `"Пауза неопределено"` |
| `nb-NO.json` | `"Pause på ubestemt tid"` |
| `nl.json` | `"Pauzeer voor onbepaalde tijd"` |
| `np.json` | `"अनिश्चित अवधिको लागि रोक्नुहोस्"` |
| `pl.json` | `"Wstrzymaj do odwołania"` |
| `pt.json` | `"Pausar indefinidamente"` |
| `pt-BR.json` | `"Pausar indefinidamente"` |
| `pu.json` | `"ਹਮੇਸ਼ਾ ਲਈ ਰੋਕੋ"` |
| `ro.json` | `"Pauză pe termen nelimitat"` |
| `ru.json` | `"Приостановить на неопределённый срок"` |
| `si.json` | `"දිගටම විරාමය"` |
| `sk.json` | `"Pozastaviť na neurčito"` |
| `sl.json` | `"Premor za nedoločen čas"` |
| `sr.json` | `"Пауза до даљњег"` |
| `sv.json` | `"Pausa obegränsat"` |
| `ta.json` | `"காலவரையின்றி இடைநிறுத்து"` |
| `th.json` | `"หยุดพักไม่มีกำหนด"` |
| `tr.json` | `"Süresiz duraklat"` |
| `uk.json` | `"Призупинити на невизначений термін"` |
| `vi.json` | `"Tạm dừng không thời hạn"` |
| `yi.json` | `"פּויזע אומבאַגרענעצט"` |
| `zh-CN.json` | `"暂停"` |
| `zh-TW.json` | `"無限期暫停"` |

- [ ] **Step 3: Commit**

```bash
git add app/locales/*.json
git commit -m "i18n: add pauseIndefinitely key to all locale files"
```

---

### Task 2: Restructure `getTrayMenuTemplate()` menu items

**Files:**
- Modify: `app/main.js:1366-1443`

- [ ] **Step 1: Replace the "Skip to the next" submenu and reorder menu items**

In `app/main.js`, replace lines 1366-1444 (from the first `if` block for skip-to-next through the end of the pause/reset block) with the following restructured code:

Original code (lines 1366-1444):
```javascript
  if (!(breakPlanner.isPaused || breakPlanner.dndManager.isOnDnd || breakPlanner.appExclusionsManager.isSchedulerCleared)) {
    let submenu = []
    if (settings.get('microbreak')) {
      submenu = submenu.concat([{
        label: i18next.t('main.toMicrobreak'),
        click: () => skipToMicrobreak()
      }])
    }
    if (settings.get('break')) {
      submenu = submenu.concat([{
        label: i18next.t('main.toBreak'),
        click: () => skipToBreak()
      }])
    }
    if (settings.get('break') || settings.get('microbreak')) {
      trayMenu.push({
        label: i18next.t('main.skipToTheNext'),
        submenu
      })
    }
  }

  if (breakPlanner.isPaused) {
    trayMenu.push({
      label: i18next.t('main.resume'),
      click: function () {
        resumeBreaks(false)
        updateTray()
      }
    })
  } else if (!(breakPlanner.dndManager.isOnDnd || breakPlanner.appExclusionsManager.isSchedulerCleared)) {
    trayMenu.push({
      label: i18next.t('main.pause'),
      submenu: [
        {
          label: i18next.t('utils.minutes', { count: 30 }),
          accelerator: settings.get('pauseBreaksFor30MinutesShortcut') || null,
          click: function () {
            pauseBreaks(1800 * 1000)
          }
        }, {
          label: i18next.t('main.forHour'),
          accelerator: settings.get('pauseBreaksFor1HourShortcut') || null,
          click: function () {
            pauseBreaks(3600 * 1000)
          }
        }, {
          label: i18next.t('main.for2Hours'),
          accelerator: settings.get('pauseBreaksFor2HoursShortcut') || null,
          click: function () {
            pauseBreaks(3600 * 2 * 1000)
          }
        }, {
          label: i18next.t('main.for5Hours'),
          accelerator: settings.get('pauseBreaksFor5HoursShortcut') || null,
          click: function () {
            pauseBreaks(3600 * 5 * 1000)
          }
        }, {
          label: i18next.t('main.untilMorning'),
          accelerator: settings.get('pauseBreaksUntilMorningShortcut') || null,
          click: function () {
            const untilMorning = new UntilMorning(settings).msToSunrise()
            pauseBreaks(untilMorning)
          }
        }, {
          type: 'separator'
        }, {
          label: i18next.t('main.indefinitely'),
          click: function () {
            pauseBreaks(1)
          }
        }
      ]
    }, {
      label: i18next.t('main.resetBreaks'),
      click: resetBreaks
    })
  }
```

New code:
```javascript
  if (breakPlanner.isPaused) {
    trayMenu.push({
      label: i18next.t('main.resume'),
      click: function () {
        resumeBreaks(false)
        updateTray()
      }
    })
  } else if (!(breakPlanner.dndManager.isOnDnd || breakPlanner.appExclusionsManager.isSchedulerCleared)) {
    trayMenu.push({
      label: i18next.t('main.pauseIndefinitely'),
      click: function () {
        pauseBreaks(1)
      }
    })
    trayMenu.push({
      label: i18next.t('main.pause'),
      submenu: [
        {
          label: i18next.t('utils.minutes', { count: 30 }),
          accelerator: settings.get('pauseBreaksFor30MinutesShortcut') || null,
          click: function () {
            pauseBreaks(1800 * 1000)
          }
        }, {
          label: i18next.t('main.forHour'),
          accelerator: settings.get('pauseBreaksFor1HourShortcut') || null,
          click: function () {
            pauseBreaks(3600 * 1000)
          }
        }, {
          label: i18next.t('main.for2Hours'),
          accelerator: settings.get('pauseBreaksFor2HoursShortcut') || null,
          click: function () {
            pauseBreaks(3600 * 2 * 1000)
          }
        }, {
          label: i18next.t('main.for5Hours'),
          accelerator: settings.get('pauseBreaksFor5HoursShortcut') || null,
          click: function () {
            pauseBreaks(3600 * 5 * 1000)
          }
        }, {
          label: i18next.t('main.untilMorning'),
          accelerator: settings.get('pauseBreaksUntilMorningShortcut') || null,
          click: function () {
            const untilMorning = new UntilMorning(settings).msToSunrise()
            pauseBreaks(untilMorning)
          }
        }, {
          type: 'separator'
        }, {
          label: i18next.t('main.indefinitely'),
          click: function () {
            pauseBreaks(1)
          }
        }
      ]
    })
    if (settings.get('break')) {
      trayMenu.push({
        label: i18next.t('main.toBreak'),
        click: () => skipToBreak()
      })
    }
    if (settings.get('microbreak')) {
      trayMenu.push({
        label: i18next.t('main.toMicrobreak'),
        click: () => skipToMicrobreak()
      })
    }
    trayMenu.push({
      label: i18next.t('main.resetBreaks'),
      click: resetBreaks
    })
  }
```

- [ ] **Step 2: Run linter**

Run: `npm run lint`
Expected: PASS with no errors

- [ ] **Step 3: Run tests**

Run: `npm test`
Expected: All tests pass

- [ ] **Step 4: Manual verification**

Run: `npm run dev`
Verify the tray menu in the following states:
1. Normal running: menu shows Pause indefinitely / Pause breaks / Long break / Mini break / Reset breaks
2. Paused: menu shows Resume breaks only
3. Only microbreak enabled (break disabled): "Long break" item hidden
4. Only break enabled (microbreak disabled): "Mini break" item hidden

- [ ] **Step 5: Commit**

```bash
git add app/main.js
git commit -m "feat: restructure tray menu with promoted quick actions"
```
