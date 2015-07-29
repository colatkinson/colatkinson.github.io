---
layout: post
title: Making a Talking Arduino Oracle
cover: /assets/images/IMG_20141009_074612_crop.jpg
color: black-87
categories: engineering
---

A look into using an Arduino Uno and a great deal of prayer to power a Halloween decoration in the name of school spirit

---

<script src="https://cdnjs.cloudflare.com/ajax/libs/vis/4.7.0/vis.min.js"></script>
<link href="https://cdnjs.cloudflare.com/ajax/libs/vis/4.7.0/vis.min.css" rel="stylesheet" type="text/css" />

<div id="visualization"></div>

<script type="text/javascript">
  // DOM element where the Timeline will be attached
  var container = document.getElementById('visualization');

  // Create a DataSet (allows two way data-binding)
  var items = new vis.DataSet([
    {id: 0, content: 'Spirit Week', start: '2014-10-06', end: '2014-10-11', type: 'range'},
    {id: 1, content: 'Squad hits me up to chill', start: '2014-10-11'},
    {id: 2, content: 'Aaron\'s Halloween Bash', title: 'AKA the night of the living Smercak', start: '2014-10-31'},
    {id: 3, content: 'Heather\'s Intertown Halloween Extravaganza', title: 'I chugged vodka and died', start: '2014-11-01'},
    {id: 4, content: 'Squad steals the Terrace Ave. sign', start: '2014-11-08'},
    {id: 5, content: 'Senior Scavenger Hunt', start: '2014-11-10'},
    {id: 6, content: 'Colin\'s First OC', start: '2015-01-12'},
    {id: 7, content: 'Team Supreme Concert', start: '2015-01-17', title: 'Great Dane was fuego'},
    {id: 8, content: 'Griffin\'s OC', start: '2015-01-17', end: '2015-01-19', type: 'range'},
    {id: 9, content: 'Science Research Party', start: '2015-02-07'},
    {id: 10, content: 'Amanda Dumps Aaron*', start: '2015-02-28', title: 'Approximately'},
    {id: 11, content: 'We steal the toilet', start: '2015-02-23'},
    {id: 12, content: 'JSHS Albany Trip', start: '2015-03-12', end: '2015-03-14', title: 'In which: Ari+Marie, Liz+Aiden/Adam, and ya boii got a lap dance', type: 'range'},
    {id: 13, content: 'Colin\'s Second OC', start: '2015-04-11'},
    {id: 14, content: 'El Dorado', start: '2015-04-18'},
    {id: 15, content: 'Band Trip', start: '2015-04-23', end: '2015-04-26', type: 'range'},
    {id: 16, content: 'Cast Party II: Hank\'s House Boogaloo', start: '2015-05-02'},
    {id: 17, content: 'Accidental Field Night', start: '2015-05-08'},
    {id: 18, content: 'Caelan drinks thot juice/We hotbox the train station', start: '2015-05-24'},
    {id: 19, content: 'Sara "Harvard" Friedman gets drunk', start: '2015-05-30'},
    {id: 20, content: 'Prom', start: '2015-06-04'},
    {id: 21, content: 'Seaside', start: '2015-06-05', end: '2015-06-08', type: 'range'},
    {id: 22, content: 'Senior Skip Day', start: '2015-06-12'},
    {id: 23, content: 'Senior Prank', start: '2015-06-15'},
    {id: 24, content: 'I return to El Dorado', start: '2015-06-16'},
    {id: 25, content: 'Graduation', start: '2015-06-25'},
    {id: 26, content: 'Mina\'s Grad Party', start: '2015-06-27', title: 'In which: Asa almost greens'},
    {id: 27, content: 'Glenn drinks Everclear', start: '2015-07-10'},
    {id: 28, content: 'Glenn gets baked', start: '2015-07-25'},
    {id: 29, content: 'Connor\'s Smelly Garage Party I', start: '2015-07-26'},
    {id: 30, content: 'Connor\'s Smelly Garage Party II', start: '2015-07-27'},
    {id: 31, content: 'Aaron\'s Last Time Out', start: '2015-06-15'},
    {id: 32, content: '8 seconds sent by Phoebe', start: '2015-06-21'},
    {id: 33, content: 'The Fall of Kingsland', start: '2015-06-24'},
    {id: 34, content: 'Aaron Leaves the Group', start: '2015-06-27'},
    {id: 35, content: 'eggs', start: '2015-05-22'},
    {id: 36, content: 'bomb: the confrontation', start: '2015-05-16'},
    {id: 37, content: 'New Chat Started', start: '2015-05-07'},
    {id: 38, content: 'the bomb drops', start: '2015-05-13'},
    {id: 39, content: 'the bomb rises', start: '2015-03-21'},
    {id: 40, content: 'First Semester', start: '2014-09-03', end: '2015-01-30', type: 'background', style: 'background-color: rgba(255,0,0,.4);'},
    {id: 41, content: 'Second Semester', start: '2015-01-30', end: '2015-06-26', type: 'background'},
    {id: 42, content: 'Summer', start: '2015-06-26', end: '2015-08-25', type: 'background'},
  ]);

  // Configuration for the Timeline
  var options = {height: 480, start: '2014-10-01', end: '2014-10-14', type: 'point'};

  // Create a Timeline
  var timeline = new vis.Timeline(container, items, options);
</script>