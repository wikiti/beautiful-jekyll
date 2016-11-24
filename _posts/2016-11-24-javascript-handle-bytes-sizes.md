---
layout: post
published: false
title: 'Javascript: Handle bytes sizes'
date: '2016-11-24 12:45'
subtitle: Because bytes are too small
---
Imagine that you're using bytes sizes in your javascript code. For example, handling files. Most libraries use bytes as the common unit to manage sizes. However, what if you want to *humanize* those values into your view? Instead of printing *1234567899 Bytes*, you should print **1,23 GigaBytes**.

Now, this can be implemented (and refactored) with the following block of javascript:

```js
var ByteSize = function(bytes) {
  // Constants
  UNITS = [
    { short: 'B', long: 'Bytes' },
    { short: 'kB', long: 'kiloBytes' },
    { short: 'MB', long: 'MegaBytes' },
    { short: 'GB', long: 'GigaBytes' },
    { short: 'TB', long: 'TeraBytes' },
  ]
  UNITS_SEPARATOR = ' ';
  FACTOR = 1e3;
  DECIMALS = 2;
  
  // Private vars
  bytes_value = bytes;
  
  // Public methods
  this.update = function(bytes) {
    bytes_value = bytes;
  };
  
  this.human = function() {
    var unit = UNITS[0];
    var value = bytes_value;
    
    for(i = 1; i < UNITS.length && value * FACTOR >= 1; i++) {
      value *= FACTOR;+
      unit = UNITS[i];
    }

    return { value: value, unit: unit };
  };

  this.humanize = function(long) {
    var data = this.human();
    var unit = long ? data.unit.long : data.unit.short;
    // Round and return as a string
    return (Math.round(data.value * 10**(DECIMALS + 1)) / 10**DECIMALS) + UNITS_SEPARATOR + unit;
  };
  
  this.Bytes = function() {
    return bytes_value;
  };
  
  this.kiloBytes = function() {
    return this.bytes() * FACTOR;
  };
  
  this.MegaBytes = function() {
    return this.kiloBytes() * FACTOR;
  };
  
  this.GigaBytes = function() {
    return this.MegaBytes() * FACTOR;
  };
  
  this.TeraBytes = function() {
    return this.GigaBytes() * FACTOR;
  };
};
```

Which can be used like this:

```js
var data = new ByteSize(123456789);

console.log(data.Bytes());        // 123456789
console.log(data.kiloBytes());    // 123456.789
console.log(data.MegaBytes());    // 123.456789
console.log(data.TeraBytes());    // 0.123456789

console.log(data.human());        // { value: 123.456789, unit: { short: 'MB', long: 'MegaBytes' } }
console.log(data.humanize());     // 123.45 MB
console.log(data.humanize(true)); // 123.45 MegaBytes

data.update(1234);
console.log(data.humanize());     // 1.23 kB
```

Next step: Create a conversor!

TODO
