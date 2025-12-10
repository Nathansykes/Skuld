# Skuld Schedule Language Specification

## Overview

This document defines the **Skuld** domain-specific language (DSL) for expressing schedules involving:

* Days of week or days of month
* Optional month or year constraints
* Time ranges
* Execution intervals
* Optional extended cron-style features

Skuld is designed to be **human readable**, **unambiguous**, and **machine-parsable** using a formal grammar.

The name “Skuld” is taken from Norse mythology, where Skuld is one of the three Norns, the weavers of fate, and represents the future—what is yet to come.

Skuld is designed to be a more readable and flexible alternative to traditional scheduling formats. Unlike cron, which is limited to fixed points in time and lacks natural support for bounded time windows, Skuld allows you to define explicit start and end times, per-day intervals, and multiple windows per day. Skuld also allows an unlimited number of expressions, making it easy to define complex yet precise schedules.

---

# 1. Basic Structure

A Skuld schedule consists of one or more **schedule expressions** separated by semicolons:

```
expression;expression;expression
```

Each expression defines **when** something can run and **how often** it runs.

General form:

```
WHEN(START-END)/INTERVAL
```

Example:

```
Mon-Fri(09:00-17:00)/15m
```

---

# 2. Days of Week

### Format

```
Mon | Tue | Wed | Thu | Fri | Sat | Sun
```

### Lists

```
Mon,Wed,Fri(09:00-17:00)/15m
```

### Ranges

```
Mon-Fri(09:00-17:00)/15m
```

### Mixed lists & ranges

```
Mon-Wed,Fri(09:00-12:00)/30m
```

### Wildcard

```
*(08:00-10:00)/20m     # every day
```

---

# 3. Days of Month

You may schedule by **day number** instead of weekdays.

### Specific days

```
1,15(09:00-17:00)/30m
```

### Last day of the month

```
L(10:00-11:00)/10m
```

### Nearest weekday

```
15W(09:00-12:00)/20m    # weekday nearest the 15th
```

---

# 4. Nth Weekday of Month

This mirrors Quartz cron’s `#` syntax.

Format:

```
DAY#N
```

Examples:

```
Mon#2(09:00-17:00)/15m    # second Monday
Fri#1(12:00-14:00)/10m    # first Friday
```

---

# 5. Months Constraint

Attach a month filter using `@`.

### Single month

```
Mon-Fri@Jan(09:00-17:00)/15m
```

### Multiple months

```
Mon-Fri@Jan,Feb,Mar(09:00-17:00)/15m
```

---

# 6. Years Constraint

Place after the month expression (if present).

### Single year

```
Mon-Fri(09:00-17:00)/15m@2025
```

### Year range

```
Mon-Fri(09:00-17:00)/15m@2025-2027
```

---

# 7. Time Ranges

Syntax:

```
HH:MM-HH:MM
```

### Basic example

```
Mon(09:00-12:00)/15m
```

### With seconds

```
Mon(09:00:05-17:00:30)/20m
```

Multiple windows on the same day are created via separate rules:

```
Mon-Fri(09:00-12:00)/10m; Mon-Fri(13:00-17:00)/20m
```

---

# 8. Intervals

Intervals specify how often execution happens inside the window.

### Minutes

```
/15m
```

### Minutes + hourly step

```
/15m/2h     # every 15 minutes, every 2 hours
```

### Examples

```
Mon-Fri(09:00-17:00)/30m
Mon-Fri(09:00-17:00)/10m/1h
```

---

# 9. Multiple Rules

Combine separate schedules with semicolons:

```
Mon-Fri(09:00-17:00)/15m; Sat(10:00-14:00)/30m
```

---

# 10. Full Grammar (EBNF)

```
skuld_schedules     = schedule { ";" schedule } ;

schedule      = when [ "@" month_expr ] [ "@" year_expr ]
                "(" time "-" time ")" "/" interval ;

when          = day_expr | dom_expr ;

day_expr      = day_item { "," day_item } ;

day_item      = day [ "#" number ] [ "-" day [ "#" number ] ] ;


# Days of Month

dom_expr      = dom_item { "," dom_item } ;

dom_item      = number | number "W" | "L" ;


# Months

month_expr    = month { "," month } ;

month         = "Jan" | "Feb" | "Mar" | "Apr" | "May" | "Jun"
              | "Jul" | "Aug" | "Sep" | "Oct" | "Nov" | "Dec" ;


# Years

year_expr     = year [ "-" year ] ;

year          = digit digit digit digit ;


# Time

time          = hour ":" minute [ ":" second ] ;
hour          = digit digit ;
minute        = digit digit ;
second        = digit digit ;


# Interval

interval      = number "m" [ "/" number "h" ] ;


# Basic Tokens
day           = "Sun" | "Mon" | "Tue" | "Wed" | "Thu" | "Fri" | "Sat" ;
number        = digit { digit } ;
digit         = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
```

---

# 11. Full Examples

## Daily windows on multiple days

```
Mon-Fri(09:00-17:00)/15m
```

## Multiple window segments

```
Mon-Fri(09:00-12:00)/10m; Mon-Fri(13:00-17:00)/20m
```

## Month + year restrictions

```
Tue,Thu@Jan,Feb(12:00-16:00)/30m@2025-2026
```

## Day-of-month with nearest weekday

```
15W(09:00-11:00)/5m
```

## Nth weekday

```
Mon#2(10:00-15:00)/20m   # second Monday of month
```

## Last day of month

```
L(08:00-09:00)/10m
```

---

# 12. Notes & Restrictions

* Time ranges must not wrap past midnight.
* A rule cannot mix weekday & day-of-month types.
* Year and month filters restrict rule applicability but do not change time.
* Multiple rules do not interact; each is treated independently.

---

# 13. Future Extensibility

Skuld can be extended with:

* Holiday exclusion lists
* Named schedules
* Timezone specification
* RDATE/EXDATE equivalents
* Inline comments (`# ...`)

---
