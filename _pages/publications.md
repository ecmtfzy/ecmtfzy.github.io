---
layout: page
permalink: /research/
title: research
nav: true
nav_order: 2
---

<style>
  /* Add small space between publications */
  .publications .bibliography li {
    margin-bottom: 0.5rem !important;
    padding-bottom: 0 !important;
  }

  /* Remove space between publication content and the separator line below it */
  .publications .bibliography li .row {
    margin-bottom: 0 !important;
  }

  /* Remove all space around year indicators (h2.bibliography) */
  .publications h2.bibliography {
    margin-top: 0 !important;
    margin-bottom: 0 !important;
    padding-top: 0 !important;
    padding-bottom: 0 !important;
  }

  /* Remove space on first item after year */
  .publications h2.bibliography + ol.bibliography {
    margin-top: 0 !important;
    padding-top: 0 !important;
  }
</style>

<div class="publications">
  <!-- Publications section (sorted by year) -->
  {% bibliography --query @*[note!=Working Paper] %}

  <!-- Working Papers section (not sorted by year) -->
  <h2 class="bibliography">Working Papers</h2>
  <ol class="bibliography">
    {% bibliography --query @*[note=Working Paper] --group_by none %}
  </ol>
</div>
