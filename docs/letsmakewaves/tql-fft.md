---
title: FFT in TQL
parent: Let's make waves
layout: default
permalink: /docs/letsmakewaves/tql-fft/
order: 30
nav_order: 11
---

# Fast Fourier Transform in TQL
{: .no_toc}

1. TOC
{: toc }

## Generates sample data

Open a new *tql* editor on the web ui and copy the code below and run it.

In this example, `oscillator()` generates a composite wave of 15Hz 1.0 + 24Hz 1.5.
And `CHART_SCATTER()` has `dataZoom()` option function that provides an slider under the x-Axis.

```js
INPUT( FAKE( 
  oscillator(
    freq(15, 1.0), freq(24, 1.5),
    range('now', '10s', '1ms')) 
))
OUTPUT( CHART_SCATTER( dataZoom('slider', 95, 100)) )
```

![web-fft-tql-fake](/assets/img/web-fft-tql-fake.jpg)

## Store data into database

Store the generated data into the database with the tag name 'signal'.

```js
INPUT( FAKE( oscillator(
    freq(15, 1.0), freq(24, 1.5),
    range('now', '10s', '1ms')) 
))
OUTPUT( INSERT( 'time', 'value', table('example'), tag('signal') ) )
```

It will show "10000 rows inserted." message in the "Result" pane.

For a comment, it took about *270ms* in a test machine (Apple mac mini M1), but using `APPEND()` method in the example below, took *65ms* (x4 faster).

```js
INPUT( FAKE( oscillator(
    freq(15, 1.0), freq(24, 1.5),
    range('now', '10s', '1ms')) 
))
PUSHKEY('signal')
OUTPUT( APPEND( table('example') ) )
```

{:.note}
> The 'APPEND' works only when fields of input records exactly match with columns of the table in order and types.

## Read data from database

The code below reads the stored data from the 'example' table.

```js
INPUT( QUERY('value', from('example', 'signal'), between('last-10s', 'last')) )
OUTPUT( CHART_LINE( dataZoom('slider', 95, 100)) )
```

![web-fft-tql-query](/assets/img/web-fft-tql-query.jpg)

## Fast Fourier Transform

Add few data manipulation function between `INPUT()` and `OUTPUT()`.

```js
INPUT( QUERY('value', from('example', 'signal'), between('last-10s', 'last')) )

PUSHKEY('sample')
GROUPBYKEY()
FFT()
FLATTEN()
POPKEY()

OUTPUT(
  CHART_LINE(
        xAxis(0, 'Hz'),
        yAxis(1, 'Amplitude'),
        dataZoom('slider', 0, 10) 
    )
)
```

![web-fft-tql-2d](/assets/img/web-fft-tql-2d.jpg)

### How it works

1. `INPUT( QUERY(...))` yields records from the query result, and *tql* treats the first field as *key* and the others are *value* tuple. `{key: time, value: (value) }`
2. `PUSHKEY('sample')` sets the constant string 'sample' as new keys for all records and "push" original key into value tuple. As result all records have same *key* `'sample'` and `(time, value)` as *value*. `{key: 'sample', value:(time, value)}`
3. `GROUPBYKEY()` merge all records that has the same key. In this example, all query results are combined into a record that has same *key* 'sample' and value is an array of tuples which formed `{key: 'sample', value:[ (time1, value1), (time2, value2), ..., (timeN, valueN) ]}`.
4. `FFT()` applies Fast Fourier Transform on the value of the record and transform the value (time-value) into an array of tuples (frequency-amplitude). `{key: 'sample', value:[ (Hz1, Ampl1), (Hz2, Ampl2), ... ]}`.
5. `FLATTEN()` reduces the dimension of the value array by splitting into multiple records. As result it yields `{key:'sample', value:(Hz1, Ampl1)}`, `{key:'sample', value:(Hz2, Ampl2)}`, ...
6. `POPKEY()` drops current key of record and promote the first element of value as a new key. `{key:Hz1, value:(Ampl1)}`, `{key:Hz2, value:(Ampl2)}`...


## Adding time axis

```js
INPUT( QUERY( 'value', from('example', 'signal'), between('last-10s', 'last')) )

PUSHKEY( roundTime(K, '500ms') )
GROUPBYKEY()
FFT(minHz(0), maxHz(100))

OUTPUT(
  CHART_BAR3D(
        xAxis(0, 'time', 'time'),
        yAxis(1, 'Hz'),
        zAxis(2, 'Amp'),
        size('600px', '600px'), visualMap(0, 1.5), theme('westeros')
  )
)
```

![web-fft-tql-3d](/assets/img/web-fft-tql-3d.jpg)


### How it works

1. `INPUT( QUERY(...))` yields records from the query result, and *tql* treats the first field as *key* and the others are *value* tuple. `{key: time, value: (value) }`
2. `PUSHKEY( roundTime(K, '500ms'))` sets the new key with the result of roundTime `K` by 500 miliseconds and "push" original key into value tuple. *tql* reserves capital letter `K` and `V` variables for *key* and *value* of a record. `{key: (time/500ms)*500ms, value:(time, value)}`
3. `GROUPBYKEY()` makes records grouped in every 500ms. `{key: time1In500ms, value:[(time1, value1), (time2, value2)...]}`
4. `FFT()` applies Fast Fourier Transform for each record. The optional functions `minHz(0)` and `maxHz(100)` limits the scope of the output just for the better visualization. `{key:time1In500ms, value:[(Hz1, Ampl1), ...]}`, `{key:'time2In500ms', value:[(Hz1, Ampl1), ...]}`, ...