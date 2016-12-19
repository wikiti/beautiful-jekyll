---
layout: post
published: true
title: 'Javascript: Handle bytes sizes'
date: '2016-11-24 12:45'
subtitle: Because bytes are too small
---
Imagine that you're using bytes sizes in your javascript code. For example, handling files. Most libraries use bytes as the common unit to manage sizes. However, what if you want to *humanize* those values into your view? Instead of printing *1234567899 Bytes*, you should print **1,23 GigaBytes**.

Now, this can be implemented (and refactored) with the following block of javascript:

```js
var ByteSize = function(bytes, size) {
  'use strict';

  // Constants
  var UNITS = [
    { short: 'B', long: 'Bytes', factor: 1 },
    { short: 'kB', long: 'kiloBytes', factor: 1e3 },
    { short: 'MB', long: 'MegaBytes', factor: 1e6 },
    { short: 'GB', long: 'GigaBytes', factor: 1e9 },
    { short: 'TB', long: 'TeraBytes', factor: 1e12 },
  ];
  var UNITS_SEPARATOR = ' ';
  var DECIMALS = 2;

  // Private vars
  var bytes_value = null;

  // Public methods
  this.set = function(bytes, unit) {
    if(unit) {
      unit = unit.toLowerCase();
      unit = UNITS.find(function(u) {
        return u.short.toLowerCase() == unit || u.long.toLowerCase() == unit;
      });
      if(!unit) throw('Invalid unit name: ' + unit);
      bytes_value = Math.round(bytes * unit.factor);
    }
    else {
      bytes_value = Math.round(bytes);
    }
  };

  this.human = function() {
    var unit = UNITS[0];
    var value = bytes_value;

    for(var i = 1; i < UNITS.length && value / UNITS[i].factor >= 1; i++) {
      unit = UNITS[i];
    }

    return { value: bytes_value / unit.factor, unit: unit };
  };

  this.humanize = function(long) {
    var data = this.human();
    var unit = long ? data.unit.long : data.unit.short;

    // Round and return as a string
    return (Math.round(data.value * 10**(DECIMALS)) / 10**DECIMALS) + UNITS_SEPARATOR + unit;
  };

  // Define Bytes(), kiloBytes(), ... TeraBytes() methods.
  var self = this;
  UNITS.forEach(function(unit, i) {
    self[unit.long] = function() {
      return bytes_value / unit.factor;
    };
  });

  // Initialize
  this.set(bytes, size);
};
```

Which can be used like this:

```js
var data = new ByteSize(123456789);

console.log(data.Bytes());        // 123456789
console.log(data.kiloBytes());    // 123456.789
console.log(data.MegaBytes());    // 123.456789
console.log(data.GigaBytes());    // 0.123456789
console.log(data.TeraBytes());    // 0.123456789

console.log(data.human());        // { value: 123.456789, unit: { short: 'MB', long: 'MegaBytes', factor: 1000000 } }
console.log(data.humanize());     // 123.45 MB
console.log(data.humanize(true)); // 123.45 MegaBytes

data.set(1234);
console.log(data.humanize());     // 1.23 kB

data.set(1234, 'kB');
console.log(data.humanize());     // 1.23 MB

data.set(1234, 'MegaBytes');
console.log(data.humanize());     // 1.23 GB

data = new ByteSize(1.2, 'kB');
console.log(data.Bytes());        // 1200

data = new ByteSize(1.2555, 'kiloBytes');
console.log(data.Bytes());        // 1256
```
