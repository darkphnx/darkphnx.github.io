---
title: Adding a progress indicator to your Activity
tags: Android, activity
---

{% filter java %}
// before setContentView in onCreate
requestWindowFeature(Window.FEATURE_INDETERMINATE_PROGRESS);

// In methods
setProgressBarIndeterminateVisibility(true) //shows spinner
setProgressBarIndeterminateVisibility(false) //hides it
{% endfilter %}