---
title: "Custom File Extension Automatic Syntax Highlighting in Brackets"
date: 2019-10-13
---

I spend a little bit of time today just trying to get custom file extensions to automatically syntax highlight in Brackets.app.  This problem came about because I kept opening my shell scripts, which didn't have extensions for ease-of-calling, but it would open as `Plain Text` "syntax highlighting" (i.e., no syntax highlighting).

To change this, you just go to
```bash
/ApplicAtions/Brackets.app/Contents/www/main.js
```

and change the file names from here!
