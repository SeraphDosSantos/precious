precious-json(7) -- tells precious what is wanted
=================================================

Data format / expectations about what goes in and out of `precious`.


## DESCRIPTION

A precious sample of valid json input.

    { "utc": "1974-06-30T21:45:00.000Z"
    , "stuff": [[0], [0, 1], null]
    }

This query is about where on the ecliptic (0) the Sun (0) and Moon (1) would be found, given a moment of time.

### REQUEST

Specification of the instructions or defaults that `precious` takes.

#### data

A relative or absolute path to the Swiss Ephemeris data files.

#### out

We get "json" by default anyway (same as null).  A bit more human-readable output is "pprint".  Even better is Eden's command line interface.

#### stuff

The kind of stuff to be returned, or null, or empty array(s) [] - mean all the same default equivalent.  The first array `[0, 3]` respectively yields ecliptic position and speed for each item of interest.  These map directly to results of the `swe.calc_ut` function.  The second array contains the main kind of things - returned with key of '1'.  The third array contains the minor kind of things - offset by 10,000 and returned with key of '2'.

#### utc

Undefined (or null) _utc_ time - means the current moment will be used.  Otherwise provide a string in ISO-8601(7) format (which must end with a Z).  Seconds are optional, plus even more optional precision after a decimal.  The default is 3 digit precision, i.e. milliseconds.

#### ut

Besides the _utc_ semi-convenience, this _ut_ array of time fragments will be passed on directly to the `swe.utc_to_jd` function (as its arguments).  It takes precedence over any _utc_ value that it is otherwise derived from.  Such option exists in case an intermediate conversion is not wanted.

#### geo

Given _geo_ location with its _lat_ / _lon_ coordinates such as `"geo": { "lat": 43.2166667, "lon": 27.9166667 }` means there can and will be angles (e.g. Asc / Mc) keyed as '3'.  These come from the `swe.houses` function, however there may or may not be house cusps in the final results output - read about the next option.

#### houses

A value of false for the _houses_ means we are not interested in any cusps, rendering any house system irrelevant (for the particular request).  None will be returned, even if the prerequisite geo coordinates are provided.  Check the swiss ephemeris manual for a list of valid house system codes.  Anyway, the 12 cusps array would be keyed as '4'.

#### extra

Asks for extra info (to be keyed as '0'), including the following:

* 'input': copy of the request object
* 'timing': ['start-time', '<end-time>']


### RESPONSE

The corresponding sample of precious output:

    {"1": {"0": {"0": 98.70827783123845}, "1": {"0": 238.13984880619816}}}


## SEE ALSO

precious(1), JSON(7), ISO-8601(7)


## NAVIGATE

Index(7), Home(7)