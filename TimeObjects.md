## Abstract

This SRFI defines a trivial type which is used to represent the
`struct timespec` defined by the POSIX [`<time.h>` header](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/time.h.html).
It is meant to replace SRFI 174, Timestamps, by providing for
other time representations than just UTC.  It is a small subset
of SRFI 19, Time Datatypes and Procedures, plus a few additional procedures.

## Rationale

The reason for putting this very simple and straightforward type into a SRFI
(and library) of its own is that time objects are part of the interface
for more than one SRFI. If they are defined in just one SRFI and imported
by the rest, that produces an otherwise useless and unnecessary dependency
on the SRFI containing the definition. This problem arises particularly
in R6RS and R7RS because record types are generative (distinct definitions
lead to distinct record types) and because most implementations
report a warning or even an error if the same identifier is imported
from more than one library, unless they have both imported it in turn
from the same original library.

The reason for writing a replacement SRFI is to provide compatibility
between SRFI 19 time objects and Posix timestamps.  Both contain fields
for seconds and nanoseconds measured from a specified
[epoch](https://en.wikipedia.org/wiki/Epoch_(computing)), but in SRFI 19 the
epoch is specified by a third field, whereas in Posix it is always
implicitly UTC.  SRFI 170, Posix API, currently depends on SRFI 19
time objects, but doing so directly requires the presence of the whole SRFI 19
implementation at run time, as explained above.  Using this SRFI to underpin both SRFI 19
and SRFI 170 eliminates the dependencies, and implementers of those SRFIs
are urged to do so.

## Specification

Time objects are immutable objects that holds a symbol and two exact integers:

 * The *type* component — a symbol specifying the epoch as explained below.
 
 * The *seconds* component — If non-negative, the number of whole seconds
   since a particular epoch.
   If negative, the number of whole seconds
   to go back in time from the epoch. Normally excludes leap seconds
   (i.e. a leap second that occurs between the epoch and *seconds*
   does not increment or decrement seconds). The minimum range of values
   is 2<sup>-39</sup> (inclusive) to 2<sup>39</sup> (exclusive), allowing a range
   of at least 15,000 years before and after the epoch.

 * The *nanoseconds* component — The number of nanoseconds added to *seconds* to specify
   a time precise to a nanosecond. It ranges from -10<sup>9</sup> to 10<sup>9</sup>, exclusive.
   It is either 0 or has the same sign as *seconds*, except that
   if *seconds* is zero, it may be either positive or negative.

   **Compatibility warning:**  SRFI 174 incorrectly defines this field as
   always non-negative, so that a time object representing 1 nanosecond
   before the epoch has a *seconds* value of -1 and a *nanoseconds* velue
   of 99999999 instead of the correct values of 0 and -1 respectively.
   For compatibility with both SRFI 19 and Posix,
   this SRFI adopts the above definition.  Because references to precise times
   before the Posix and TAI epochs are rare, this is not considered a serious incompatibility.

In no case can this SRFI guarantee anything about the accuracy
or precision of any time object sources. Since it is difficult to usefully
determine and communicate precision and accuracy in most applications,
time objects do not even ontain any standard field to hold such information.

These procedures, except as noted below,
have the same semantics as the
[SRFI 19](https://srfi.schemers.org/srfi-19/srfi-19.html)
procedures with the same names.
Their [SRFI 174](https://srfi.schemers.org/srfi-174/srfi-174.html)
equivalents appear in small type below (FIXME: not yet)
and are deprecated.

Time objects are treated by the procedures of this SRFI as immutable;
if they are the same as SRFI 19 time objects (which is recommended
if an implementation provides both), then using SRFI 19
mutators to alter them is an error.

Note that [SRFI 18](https://srfi.schemers.org/srfi-18/srfi-18.html)
time objects are distinct from the time objects provided by this SRFI.
It would be possible to implement the former using the latter, but it
would be unnecessarily heavyweight.

## Time constructor, predicate, and accessors

`time-utc`  
`time-tai`  
`time-duration`  
`time-monotonic`  
`time-process`  
`time-thread`

Variables whose values are distinct symbols.
They hold the only valid values of the *type*
argument of `make-time`.
Only the first three are fully supported by this SRFI.

The epoch for `time-utc` objects is the Posix
epoch, 1970-01-01T00:00:00Z.
The epoch for `time-tai` objects and for the values
returned by the R7RS-small procedure `current-second`
(which are inexact numbers called *instants* in this SRFI)
is 8 seconds earlier.

The current UTC time can be obtained from the
[SRFI 170](https://srfi.schemers.org/srfi-170/srfi-170.html)
procedure `posix-time`; the current instant can be obtained
from the R7RS-small procedure `current-second`.

`make-time` *type nanosecond second -> time*  
<small>`timespec ` *nanosecond second*</small>

Returns a time object.  It is an error unless
the *type* is one of the values of the exported variables
listed above.

The `timespec` version sets the type to `time-utc`.

`time?` *object -> boolean*  
`timespec?` *object -> boolean*

Returns `#t` if object is a time object, otherwise `#f`.

`time-type` *time -> time-type*  [No SRFI 174 equivalent]

Returns the time type of *time* as a symbol.

`time-nanosecond` *time -> exact-integer*  
`timespec-nanoseconds` *time -> exact-integer*

Returns the nanoseconds of *time* as an exact integer.

`time-second` *time -> exact-integer*  
`timespec-seconds` *time ->exact-integers*

Returns the seconds of *time* as an exact integer.

## Time comparison procedures

All of the time comparison procedures require the time objects to be of the same type.
If these procedures are applied to time objects of different types, an error
satisfying `time-object-error?` is signaled.

The semantics for time objects of type `time-duration` are given in parentheses.

`time=?` *time1 time2 -> boolean*  
`timespec=?` *time1 time2 -> boolean*

Returns `#t` if *time1* is equal to *time2*, `#f` otherwise.

`time<?` *time1 time2 -> boolean*  
`timespec<?` *time1 time2 -> boolean*

Returns `#t` if *time1* is before (less than) *time2*, `#f` otherwise.

`time>?` *time1 time2 -> boolean*  [No SRFI 174 equivalent]

Returns `#t` if *time1* is after (greater than) *time2*, `#f` otherwise.

`time<=?` *time1 time2 -> boolean*  [No SRFI 174 equivalent]

Returns `#t` if *time1* is before or at (less than or equal to) *time2*, `#f` otherwise.

`time>=?` *time1 time2 -> boolean*  [No SRFI 174 equivalent]

Returns `#t` if time1 is at or after (greater than or equal to) time2, `#f` otherwise.

## Time arithmetic procedures

`time-difference` *time1 time2 -> time-duration*  [No SRFI 174 equivalent]

Returns a time object of type `time-duration` representing the time
between *time1* and *time2*.
It is an error unless *time1* and *time2* are both TAI or both UTC.

`add-duration` *time time-duration -> time*  [No SRFI 174 equivalent]

Returns the time object resulting from adding *time-duration* to *time*.
The result has the same type as *time*.  It is an error unless
*time* is TAI or UTC.

`subtract-duration` *time time-duration -> time*  [No SRFI 174 equivalent]

Returns the time resulting from subtracting *time-duration* from *time*.
The result has the same type as *time*.  It is an error unless
*time* is TAI or UTC.

## Conversion

`time-utc->time-tai` *time* [*leap-second*]   [No SRFI 174 equivalent]

Returns a time object of type `time-tai` equivalent to *time*;
it is an error if *time* is not of type `time-utc`.
If *time* is equivalent
to two different `time-tai` objects, one of which is a leap second and
the other of which is not, the boolean argument *leap-second* specifies
which TAI second is returned.
See discussion of leap seconds below.

`time-tai->time-utc` *time*  [No SRFI 174 equivalent]

Returns a time object of type `time-utc` equivalent to *time*;
it is an error if *time* is not of type `time-tai`.


`time->instant` *time*  [No SRFI 19 equivalent] 
`timespec->instant` *time*

Converts a UTC or TAI time object to the equivalent instant
and returns it.

`instant->time` *type inexact*   [No SRFI 19 equivalent]  
`instant->timespec` *type inexact*

Converts an instant to the equivalent UTC or TAI time object
and returns it.

## Time hash and comparator

`time-hash` *time*  [No SRFI 19 equivalent]  
`timespec-hash` *time*

Returns an exact integer hash code for *time*.

`time-comparator`    [No SRFI 19 or SRFI 174 equivalent]

A SRFI 128 comparator for two times of the same type
based on `time?`, `time=?`, `time<`, and `time-hash`.
If they are of different types, an error satisfying
time-object-error?` is signaled.

## Time exceptions

`time-object-error?` *obj*

Returns `#t` if *obj* is raised in any of the
circumstances mentioned above or possibly in

## Implementation

The sample implementation provides
only an approximation of the mapping between TAI
and UTC.  The two time scales are assumed to be synchronized at 00:00:00
on January 1, 1958 and at all times before that.  From 1958 through 1971,
the relationship is complex, but since 1971 the two scales have been kept
within 0.9 seconds of each other by inserting leap seconds as needed.

For the messy period, the implementation pretends that there were leap seconds
at the end of the following days (that is, at 23:59:60 proleptic UTC time):
30 Jun 1959; 30 Jun 1961; 31 Dec 1963; 31 Dec 1964; 30 Jun 1966;
30 Jun 1967; 30 Jun 1968; 30 Jun 1969; 30 Jun 1970.
This has the following desirable effects: the TAI-UTC offset is 0 in 1958
(true by definition), at the Posix epoch it is 8
(which is within a few milliseconds of the true value),
and it is 10 at the start of 1972 when UTC and its leap second regime
begin.  Not having a leap second at the end of 1969 ensures that there is none
just before the Posix epoch.  The implementation also pretends,
*faute de mieux*, that there will be no more leap seconds in the future.

To update the leap second tables, download
[`leap-seconds.list`](https://www.ietf.org/timezones/data/leap-seconds.list)
for IANA's version of such a table, which is maintained.
As explained in the comments there, the first column uses an epoch of
1900-01-01T00:00:00; to convert it to UTC, subtract 2208988800.
The second column is the value of TAI-UTC starting at that second.

For exact leap second data before 1972, see the old USNO file
[`tai-utc.dat`](http://web.archive.org/web/20191022082231/http://maia.usno.navy.mil/ser7/tai-utc.dat).
This file is *not* being updated, and should be used only if the
implementation wants to make exact conversions for the 1961-72 period.

The following table specifies the arbitrary 1958-71 times and offsets
described above, using the Posix epoch in the first column.

```
-378648000 0 # 01 Jan 1958
-283953600 1 # 01 Jan 1961
-252417600 2 # 01 Jan 1962
-205286400 3 # 01 Jul 1963
-157723200 4 # 01 Jan 1965
-110592000 5 # 01 Jul 1966
 -79056000 6 # 01 Jul 1967
 -47433600 7 # 01 Jul 1968
 -15897600 8 # 01 Jul 1969
  15638400 9 # 01 Jul 1970
```
## Acknowledgements

Thanks to Daphne Preston-Kendal for general assistance, and
especially for determining an optimal leap second set from
1958 to 1971.
