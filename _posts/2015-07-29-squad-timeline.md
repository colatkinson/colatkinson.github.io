---
layout: page
title: Squad Timeline
cover: https://lh3.googleusercontent.com/vd3V9Em8irFpgWHFQC5ZQaJPgibhAZ537YhnXxBMhkQ=w697-h941-no
color: black-87
categories: squad
---

A look into the history of our esteemed group of friends.

---

<link title="timeline-styles" rel="stylesheet" href="//cdn.knightlab.com/libs/timeline3/latest/css/timeline.css" />
<script src="https://cdn.knightlab.com/libs/timeline3/latest/js/timeline.js"></script>

<style>
    </style>

<div id="timeline" style="height: 800px;"></div>

<script type="text/javascript">
    var timeline = new VCO.Timeline('timeline', '/public/timeline.json', {});
    window.onresize = function(event) {
        timeline.updateDisplay();
    }
</script>