---
layout: post
published: false
title: 'Javascript: handle bytes sizes'
---
Imagine that you're using bytes sizes in your javascript code. For example, handling files. Most libraries use bytes as the common unit to manage sizes. However, what if you want to *humanize* those values into your view? Instead of printing *1234567899 Bytes*, you should print **1,23 GigaBytes**.

Now, this can be implemented (and refactored) with the following block of javascript:

```js

```

And can be used like this:

```js
var HumanSizes = function(bytes) {
  UNITS = [
    { short: 'B', long: 'Bytes' },
    { short: 'kB', long: 'kiloBytes' },
    { short: 'MB', long: 'MegaBytes' },
    { short: 'GB', long: 'GigaBytes' },
    { short: 'TB', long: 'TeraBytes' },
  ]
  FACTOR = 1e3;
  DECIMALS = 2;

  bytes_value = bytes;
  
  this.human = function() {
    var unit = UNITS[0];
    var value = this.bytes_value;
    
    for(i = 1; i < UNITS.length && value * FACTOR >= 1; i++) {
      value *= FACTOR;+
      unit = UNITS[i];
    }

    return { bytes: value, unit: unit };
  };

  this.humanize = function(long) {
    var data = human();
    var unit = long ? data.unit.long : data.unit.short;
    // Round and return as a string
    return (Math.round(data.bytes * 10**(DECIMALS + 1)) / 10**DECIMALS) + ' ' + unit;
  };
  
  this.kiloBytes = function() {
    return bytes * FACTOR;
  };
  
  this.MegaBytes = function() {
    return 
  };
};
```