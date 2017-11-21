# Date 와 DateTime

```@meta
CurrentModule = Base.Dates
```
`dates`모듈은 날짜와 함께 작업할 수 있는 [`Date`](@ref)와 [`DateTime`](@ref) 두가지 유형을 제공한다. 모두 추상적인 [`TimeType`](@ref)의 하위 유형으로 각각 하루와 밀리 초 단위의 정확도를 제공한다. 유형을 구별한 동기는 간단하다: 코드 및 정신적 추론 측면에서 더 큰 정밀도의 복잡성을 처리 할 필요가 없는 일부 작업이 보다 간단해진다. 예를 들어, [`Date`](@ref) 유형은 오직 한 날짜의 정확도(즉, 어떤 시간, 분, 초가 아닌)로만 결정하기 때문에 표준시간대, 일광 절약 시간/서머타임, 윤초에 대한 일반적인 고려 사항은 필요하지 않다.

[`Date`](@ref)와 [`DateTime`](@ref)은 둘 다 기본적으로 변경할 수 없는 [`Int64`](@ref)래퍼 입니다. 두 타입 중 하나의 `instant`필드는 실제로 UT second [^1]를 기반으로 계속 증가하는 시간대를 나타내는 `UTInstant{p}`타입이다. 
[`DateTime`](@ref)은 Java8의 *LocalDateTime*과 유사한 시간대(*즉*, Phython 언어로 된)를 인식하지 못합니다.
부가직인 시간대의 기능을 [IANA time zone database](http://www.inan.org/time-zones)를 컴파일하는 [TimeZones.jl package](https://github.com/JuliaTiem/TimeZones.jl/) 패키지를 통해서 추가할 수 있습니다. [`Date`](@ref)와 [`DateTime`](@ref)은 [ISO 8601](https://enwikipedia.org/wiki/ISO_8601)표준으로, 그레고리력을 따르고 있습니다.
ISO 8601표준은 BC/BCE 날짜와 관련이 있습니다. 일반적으로, 마지막 날인 1-12-31 BC/BCE 다음날은 1-1-1 AD/CE으로 이어집니다. 그래서 0년은 존재하지 않습니다. 하지만 ISO표준은 1 BC/BCE의 년도를 0으로 표기합니다. 그래서 `0001-01-01`이전은 `0000-12-31`이고, 2 BC/BCE는 `-0001`(음수 1), 3 BC/BCE은 `-0002`년으로 표기합니다.

[^1]:
    UT초의 개념은 사실 아주 기초적인 것입니다. 기본적으로 받아들여진 지구의 물리적 회전(1회전=1일)과 IS초(상수 값)에 기초를 할 개념 두가지가 일반적으로 받아들여집니다.이들은 근본적으로 다릅니다! 생각해보세요, "UT초"는 지구의 회전과 관련있기 때문에 하루에 따라 절대길이가 달라질 수 있습니다! [`Date`](@ref)와 [`DateTime`](@ref)이 UT초를 기반으로 한다는 사실은 단순하지만, 명확한 전제임으로 윤초 같은 복잡한 것들을 피할 수 있습니다. 이 시간의 기준은 공식적으로 [UT](https://en.wikipedia.org/wiki/Universal_Time)또는 UT1이라고 합니다. UT초의 기본타입은 기본적으로 1분마다 60초가 걸리고 하루 24시간이 걸리는 것을 의미하고, 달력날짜를 작업할 때 더 자연스러운 계산을 유도합니다.

## 생성자

[`Date`](@ref) 와 [`DateTime`](@ref) 타입은 정수 또는 [`Period`](@ref) 타입, parsing 또는 adjusters를 통해 생성할 수 있습니다. (자세한 내용은 나중에):

```jldoctest
julia> DateTime(2013)
2013-01-01T00:00:00

julia> DateTime(2013,7)
2013-07-01T00:00:00

julia> DateTime(2013,7,1)
2013-07-01T00:00:00

julia> DateTime(2013,7,1,12)
2013-07-01T12:00:00

julia> DateTime(2013,7,1,12,30)
2013-07-01T12:30:00

julia> DateTime(2013,7,1,12,30,59)
2013-07-01T12:30:59

julia> DateTime(2013,7,1,12,30,59,1)
2013-07-01T12:30:59.001

julia> Date(2013)
2013-01-01

julia> Date(2013,7)
2013-07-01

julia> Date(2013,7,1)
2013-07-01

julia> Date(Dates.Year(2013),Dates.Month(7),Dates.Day(1))
2013-07-01

julia> Date(Dates.Month(7),Dates.Year(2013))
2013-07-01
```

[`Date`](@ref) 또는 [`DateTime`](@ref) 구문 분석은 문자열 형식을 사용해서 수행합니다. 문자열 형식은 구문 분석 할 마침표가 포함되고 *구분*되거나 *고정된 폭*의 "슬롯"을 정의하고, [`Date`](@ref) 또는 [`DateTime`](@ref) 생성자, 즉 `Date("2015-01-01","y-m-d")` 또는 `DateTime("20150101","yyyymmdd")` 형식으로 구문 분석 할 문자열 형식을 본문에 전달하는 개념으로 작동한다. 

구분 된 슬롯은 구문 분석기가 두 개의 subsequent 마침표 사이에 예상되는 구분 기호를 지정하여 표시합니다; 그래서 `"y-m-d"`의 구문 분석기는 첫번째와 두번째 슬롯 사이의 문자열에서 `"2014-07-16"`의 `-`문자를 찾아야 한다. `y`,`m`,`d` 문자는 구문 분석기가 각 슬롯에 구문 분석해야하는 기간을 알려준다. 

고정된 폭의 슬롯은 문자사이에 마침표가 없는 시간에 해당하는 횟수만큼 마침표를  반복해서 지정합니다. 그래서 `"yyyymmdd"`는 `"20140716"`같은 날짜 문자열과 일치합니다. 구문 분석기는 한 마침표 문자부터 다음 마침표까지 `"yyyymm"`의 변화에 주의하고, 구분기호가 없으면 고정된 넓이의 슬롯을 구별합니다. 

달의 문자 형식 분석은 `u`와 `U` 문자를 통해서 단축된 월의 이름과 전체길이 월의 이름을 각각 지원합니다. 그래서 `u`는 "Jan", "Feb", "Mar" 등등에 해당합니다. `U`는 "January", "February",
"March" 등등에 해당합니다. 다른 이름=>변수 매핑 함수 [`dayname`](@ref) 과 [`monthname`](@ref)과 비슷하게, 사용자 정의 locales는 `locale=>용어{String,Int}`를 매핑해서 `MONTHTOVALUEABBR`
와 `MONTHTOVALUE` 용어를 단축된 달의 이름과 전체길이의 달의 이름으로 각각 전달해서 불러올 수 있습니다.

구문 분석의 성능에 주의할 점 : `Date(date_string,format_string)` 함수를 사용하면 몇번만 호출해도 괜찮습니다. 하지만 구문 분석 할 비슷한 형식의 날짜 문자열이 많은 경우, [`Dates.DateFormat`](@ref)형식을 먼저 생성한 후 초기 형식 문자열을 대신 전달하는 것이 훨씬 더 효율적입니다. 

```jldoctest
julia> df = DateFormat("y-m-d");

julia> dt = Date("2015-01-01",df)
2015-01-01

julia> dt2 = Date("2015-01-02",df)
2015-01-02
```

`dateformat""` 문자열 매크로를 사용할 수 있습니다. 이 매크로는 확장될 때 `DateFormat`객체가 확장될 대 한 번 생성하고, 여러번 실행될 때에도 같은 `DateFormat`객체를 사용합니다.

```jldoctest
julia> for i = 1:10^5
           Date("2015-01-01", dateformat"y-m-d")
       end
```

구문 분석과 지정된 테스트 및 예제 전체들을 [`tests/dates/io.jl`](https://github.com/JuliaLang/julia/blob/master/test/dates/io.jl).

## Durations/Comparisons

Finding the length of time between two [`Date`](@ref) or [`DateTime`](@ref) is straightforward
given their underlying representation as `UTInstant{Day}` and `UTInstant{Millisecond}`, respectively.
The difference between [`Date`](@ref) is returned in the number of [`Day`](@ref), and [`DateTime`](@ref)
in the number of [`Millisecond`](@ref). Similarly, comparing [`TimeType`](@ref) is a simple matter
of comparing the underlying machine instants (which in turn compares the internal [`Int64`](@ref) values).

```jldoctest
julia> dt = Date(2012,2,29)
2012-02-29

julia> dt2 = Date(2000,2,1)
2000-02-01

julia> dump(dt)
Date
  instant: Base.Dates.UTInstant{Base.Dates.Day}
    periods: Base.Dates.Day
      value: Int64 734562

julia> dump(dt2)
Date
  instant: Base.Dates.UTInstant{Base.Dates.Day}
    periods: Base.Dates.Day
      value: Int64 730151

julia> dt > dt2
true

julia> dt != dt2
true

julia> dt + dt2
ERROR: MethodError: no method matching +(::Date, ::Date)
[...]

julia> dt * dt2
ERROR: MethodError: no method matching *(::Date, ::Date)
[...]

julia> dt / dt2
ERROR: MethodError: no method matching /(::Date, ::Date)
[...]

julia> dt - dt2
4411 days

julia> dt2 - dt
-4411 days

julia> dt = DateTime(2012,2,29)
2012-02-29T00:00:00

julia> dt2 = DateTime(2000,2,1)
2000-02-01T00:00:00

julia> dt - dt2
381110400000 milliseconds
```

## Accessor Functions

Because the [`Date`](@ref) and [`DateTime`](@ref) types are stored as single [`Int64`](@ref) values, date
parts or fields can be retrieved through accessor functions. The lowercase accessors return the
field as an integer:

```jldoctest tdate
julia> t = Date(2014, 1, 31)
2014-01-31

julia> Dates.year(t)
2014

julia> Dates.month(t)
1

julia> Dates.week(t)
5

julia> Dates.day(t)
31
```

While propercase return the same value in the corresponding [`Period`](@ref) type:

```jldoctest tdate
julia> Dates.Year(t)
2014 years

julia> Dates.Day(t)
31 days
```

Compound methods are provided, as they provide a measure of efficiency if multiple fields are
needed at the same time:

```jldoctest tdate
julia> Dates.yearmonth(t)
(2014, 1)

julia> Dates.monthday(t)
(1, 31)

julia> Dates.yearmonthday(t)
(2014, 1, 31)
```

One may also access the underlying `UTInstant` or integer value:

```jldoctest tdate
julia> dump(t)
Date
  instant: Base.Dates.UTInstant{Base.Dates.Day}
    periods: Base.Dates.Day
      value: Int64 735264

julia> t.instant
Base.Dates.UTInstant{Base.Dates.Day}(735264 days)

julia> Dates.value(t)
735264
```

## Query Functions

Query functions provide calendrical information about a [`TimeType`](@ref). They include information
about the day of the week:

```jldoctest tdate2
julia> t = Date(2014, 1, 31)
2014-01-31

julia> Dates.dayofweek(t)
5

julia> Dates.dayname(t)
"Friday"

julia> Dates.dayofweekofmonth(t) # 5th Friday of January
5
```

Month of the year:

```jldoctest tdate2
julia> Dates.monthname(t)
"January"

julia> Dates.daysinmonth(t)
31
```

As well as information about the [`TimeType`](@ref)'s year and quarter:

```jldoctest tdate2
julia> Dates.isleapyear(t)
false

julia> Dates.dayofyear(t)
31

julia> Dates.quarterofyear(t)
1

julia> Dates.dayofquarter(t)
31
```

The [`dayname`](@ref) and [`monthname`](@ref) methods can also take an optional `locale` keyword
that can be used to return the name of the day or month of the year for other languages/locales.
There are also versions of these functions returning the abbreviated names, namely
[`dayabbr`](@ref) and [`monthabbr`](@ref).
First the mapping is loaded into the `LOCALES` variable:

```jldoctest tdate2
julia> french_months = ["janvier", "février", "mars", "avril", "mai", "juin",
                        "juillet", "août", "septembre", "octobre", "novembre", "décembre"];

julia> french_monts_abbrev = ["janv","févr","mars","avril","mai","juin",
                              "juil","août","sept","oct","nov","déc"];

julia> french_days = ["lundi","mardi","mercredi","jeudi","vendredi","samedi","dimanche"];

julia> Dates.LOCALES["french"] = Dates.DateLocale(french_months, french_monts_abbrev, french_days, [""]);
```

 The above mentioned functions can then be used to perform the queries:

```jldoctest tdate2
julia> Dates.dayname(t;locale="french")
"vendredi"

julia> Dates.monthname(t;locale="french")
"janvier"

julia> Dates.monthabbr(t;locale="french")
"janv"
```

Since the abbreviated versions of the days are not loaded, trying to use the
function `dayabbr` will error.

```jldoctest tdate2
julia> Dates.dayabbr(t;locale="french")
ERROR: BoundsError: attempt to access 1-element Array{String,1} at index [5]
Stacktrace:
[...]
```


## TimeType-Period Arithmetic

It's good practice when using any language/date framework to be familiar with how date-period
arithmetic is handled as there are some [tricky issues](https://codeblog.jonskeet.uk/2010/12/01/the-joys-of-date-time-arithmetic/)
to deal with (though much less so for day-precision types).

The `Dates` module approach tries to follow the simple principle of trying to change as
little as possible when doing [`Period`](@ref) arithmetic. This approach is also often known as
*calendrical* arithmetic or what you would probably guess if someone were to ask you the same
calculation in a conversation. Why all the fuss about this? Let's take a classic example: add
1 month to January 31st, 2014. What's the answer? Javascript will say [March 3](http://www.markhneedham.com/blog/2009/01/07/javascript-add-a-month-to-a-date/)
(assumes 31 days). PHP says [March 2](https://stackoverflow.com/questions/5760262/php-adding-months-to-a-date-while-not-exceeding-the-last-day-of-the-month)
(assumes 30 days). The fact is, there is no right answer. In the `Dates` module, it gives
the result of February 28th. How does it figure that out? I like to think of the classic 7-7-7
gambling game in casinos.

Now just imagine that instead of 7-7-7, the slots are Year-Month-Day, or in our example, 2014-01-31.
When you ask to add 1 month to this date, the month slot is incremented, so now we have 2014-02-31.
Then the day number is checked if it is greater than the last valid day of the new month; if it
is (as in the case above), the day number is adjusted down to the last valid day (28). What are
the ramifications with this approach? Go ahead and add another month to our date, `2014-02-28 + Month(1) == 2014-03-28`.
What? Were you expecting the last day of March? Nope, sorry, remember the 7-7-7 slots. As few
slots as possible are going to change, so we first increment the month slot by 1, 2014-03-28,
and boom, we're done because that's a valid date. On the other hand, if we were to add 2 months
to our original date, 2014-01-31, then we end up with 2014-03-31, as expected. The other ramification
of this approach is a loss in associativity when a specific ordering is forced (i.e. adding things
in different orders results in different outcomes). For example:

```jldoctest
julia> (Date(2014,1,29)+Dates.Day(1)) + Dates.Month(1)
2014-02-28

julia> (Date(2014,1,29)+Dates.Month(1)) + Dates.Day(1)
2014-03-01
```

What's going on there? In the first line, we're adding 1 day to January 29th, which results in
2014-01-30; then we add 1 month, so we get 2014-02-30, which then adjusts down to 2014-02-28.
In the second example, we add 1 month *first*, where we get 2014-02-29, which adjusts down to
2014-02-28, and *then* add 1 day, which results in 2014-03-01. One design principle that helps
in this case is that, in the presence of multiple Periods, the operations will be ordered by the
Periods' *types*, not their value or positional order; this means `Year` will always be added
first, then `Month`, then `Week`, etc. Hence the following *does* result in associativity and
Just Works:

```jldoctest
julia> Date(2014,1,29) + Dates.Day(1) + Dates.Month(1)
2014-03-01

julia> Date(2014,1,29) + Dates.Month(1) + Dates.Day(1)
2014-03-01
```

Tricky? Perhaps. What is an innocent `Dates` user to do? The bottom line is to be aware
that explicitly forcing a certain associativity, when dealing with months, may lead to some unexpected
results, but otherwise, everything should work as expected. Thankfully, that's pretty much the
extent of the odd cases in date-period arithmetic when dealing with time in UT (avoiding the "joys"
of dealing with daylight savings, leap seconds, etc.).

As a bonus, all period arithmetic objects work directly with ranges:

```jldoctest
julia> dr = Date(2014,1,29):Date(2014,2,3)
2014-01-29:1 day:2014-02-03

julia> collect(dr)
6-element Array{Date,1}:
 2014-01-29
 2014-01-30
 2014-01-31
 2014-02-01
 2014-02-02
 2014-02-03

julia> dr = Date(2014,1,29):Dates.Month(1):Date(2014,07,29)
2014-01-29:1 month:2014-07-29

julia> collect(dr)
7-element Array{Date,1}:
 2014-01-29
 2014-02-28
 2014-03-29
 2014-04-29
 2014-05-29
 2014-06-29
 2014-07-29
```

## Adjuster Functions

As convenient as date-period arithmetics are, often the kinds of calculations needed on dates
take on a *calendrical* or *temporal* nature rather than a fixed number of periods. Holidays are
a perfect example; most follow rules such as "Memorial Day = Last Monday of May", or "Thanksgiving
= 4th Thursday of November". These kinds of temporal expressions deal with rules relative to the
calendar, like first or last of the month, next Tuesday, or the first and third Wednesdays, etc.

The `Dates` module provides the *adjuster* API through several convenient methods that
aid in simply and succinctly expressing temporal rules. The first group of adjuster methods deal
with the first and last of weeks, months, quarters, and years. They each take a single [`TimeType`](@ref)
as input and return or *adjust to* the first or last of the desired period relative to the input.

```jldoctest
julia> Dates.firstdayofweek(Date(2014,7,16)) # Adjusts the input to the Monday of the input's week
2014-07-14

julia> Dates.lastdayofmonth(Date(2014,7,16)) # Adjusts to the last day of the input's month
2014-07-31

julia> Dates.lastdayofquarter(Date(2014,7,16)) # Adjusts to the last day of the input's quarter
2014-09-30
```

The next two higher-order methods, [`tonext`](@ref), and [`toprev`](@ref), generalize working
with temporal expressions by taking a `DateFunction` as first argument, along with a starting
[`TimeType`](@ref). A `DateFunction` is just a function, usually anonymous, that takes a single
[`TimeType`](@ref) as input and returns a [`Bool`](@ref), `true` indicating a satisfied
adjustment criterion.
For example:

```jldoctest
julia> istuesday = x->Dates.dayofweek(x) == Dates.Tuesday; # Returns true if the day of the week of x is Tuesday

julia> Dates.tonext(istuesday, Date(2014,7,13)) # 2014-07-13 is a Sunday
2014-07-15

julia> Dates.tonext(Date(2014,7,13), Dates.Tuesday) # Convenience method provided for day of the week adjustments
2014-07-15
```

This is useful with the do-block syntax for more complex temporal expressions:

```jldoctest
julia> Dates.tonext(Date(2014,7,13)) do x
           # Return true on the 4th Thursday of November (Thanksgiving)
           Dates.dayofweek(x) == Dates.Thursday &&
           Dates.dayofweekofmonth(x) == 4 &&
           Dates.month(x) == Dates.November
       end
2014-11-27
```

The [`Base.filter`](@ref) method can be used to obtain all valid dates/moments in a specified
range:

```jldoctest
# Pittsburgh street cleaning; Every 2nd Tuesday from April to November
# Date range from January 1st, 2014 to January 1st, 2015
julia> dr = Dates.Date(2014):Dates.Date(2015);

julia> filter(dr) do x
           Dates.dayofweek(x) == Dates.Tue &&
           Dates.April <= Dates.month(x) <= Dates.Nov &&
           Dates.dayofweekofmonth(x) == 2
       end
8-element Array{Date,1}:
 2014-04-08
 2014-05-13
 2014-06-10
 2014-07-08
 2014-08-12
 2014-09-09
 2014-10-14
 2014-11-11
```

Additional examples and tests are available in [`test/dates/adjusters.jl`](https://github.com/JuliaLang/julia/blob/master/test/dates/adjusters.jl).

## Period Types

Periods are a human view of discrete, sometimes irregular durations of time. Consider 1 month;
it could represent, in days, a value of 28, 29, 30, or 31 depending on the year and month context.
Or a year could represent 365 or 366 days in the case of a leap year. [`Period`](@ref) types are
simple [`Int64`](@ref) wrappers and are constructed by wrapping any `Int64` convertible type, i.e. `Year(1)`
or `Month(3.0)`. Arithmetic between [`Period`](@ref) of the same type behave like integers, and
limited `Period-Real` arithmetic is available.

```jldoctest
julia> y1 = Dates.Year(1)
1 year

julia> y2 = Dates.Year(2)
2 years

julia> y3 = Dates.Year(10)
10 years

julia> y1 + y2
3 years

julia> div(y3,y2)
5

julia> y3 - y2
8 years

julia> y3 % y2
0 years

julia> div(y3,3) # mirrors integer division
3 years
```

## Rounding

[`Date`](@ref) and [`DateTime`](@ref) values can be rounded to a specified resolution (e.g., 1
month or 15 minutes) with [`floor`](@ref), [`ceil`](@ref), or [`round`](@ref):

```jldoctest
julia> floor(Date(1985, 8, 16), Dates.Month)
1985-08-01

julia> ceil(DateTime(2013, 2, 13, 0, 31, 20), Dates.Minute(15))
2013-02-13T00:45:00

julia> round(DateTime(2016, 8, 6, 20, 15), Dates.Day)
2016-08-07T00:00:00
```

Unlike the numeric [`round`](@ref) method, which breaks ties toward the even number by default,
the [`TimeType`](@ref)[`round`](@ref) method uses the `RoundNearestTiesUp` rounding mode. (It's
difficult to guess what breaking ties to nearest "even" [`TimeType`](@ref) would entail.) Further
details on the available `RoundingMode` s can be found in the [API reference](@ref stdlib-dates).

Rounding should generally behave as expected, but there are a few cases in which the expected
behaviour is not obvious.

### Rounding Epoch

In many cases, the resolution specified for rounding (e.g., `Dates.Second(30)`) divides evenly
into the next largest period (in this case, `Dates.Minute(1)`). But rounding behaviour in cases
in which this is not true may lead to confusion. What is the expected result of rounding a [`DateTime`](@ref)
to the nearest 10 hours?

```jldoctest
julia> round(DateTime(2016, 7, 17, 11, 55), Dates.Hour(10))
2016-07-17T12:00:00
```

That may seem confusing, given that the hour (12) is not divisible by 10. The reason that `2016-07-17T12:00:00`
was chosen is that it is 17,676,660 hours after `0000-01-01T00:00:00`, and 17,676,660 is divisible
by 10.

As Julia [`Date`](@ref) and [`DateTime`](@ref) values are represented according to the ISO 8601
standard, `0000-01-01T00:00:00` was chosen as base (or "rounding epoch") from which to begin the
count of days (and milliseconds) used in rounding calculations. (Note that this differs slightly
from Julia's internal representation of [`Date`](@ref) s using Rata Die notation; but since the
ISO 8601 standard is most visible to the end user, `0000-01-01T00:00:00` was chosen as the rounding
epoch instead of the `0000-12-31T00:00:00` used internally to minimize confusion.)

The only exception to the use of `0000-01-01T00:00:00` as the rounding epoch is when rounding
to weeks. Rounding to the nearest week will always return a Monday (the first day of the week
as specified by ISO 8601). For this reason, we use `0000-01-03T00:00:00` (the first day of the
first week of year 0000, as defined by ISO 8601) as the base when rounding to a number of weeks.

Here is a related case in which the expected behaviour is not necessarily obvious: What happens
when we round to the nearest `P(2)`, where `P` is a [`Period`](@ref) type? In some cases (specifically,
when `P <: Dates.TimePeriod`) the answer is clear:

```jldoctest
julia> round(DateTime(2016, 7, 17, 8, 55, 30), Dates.Hour(2))
2016-07-17T08:00:00

julia> round(DateTime(2016, 7, 17, 8, 55, 30), Dates.Minute(2))
2016-07-17T08:56:00
```

This seems obvious, because two of each of these periods still divides evenly into the next larger
order period. But in the case of two months (which still divides evenly into one year), the answer
may be surprising:

```jldoctest
julia> round(DateTime(2016, 7, 17, 8, 55, 30), Dates.Month(2))
2016-07-01T00:00:00
```

Why round to the first day in July, even though it is month 7 (an odd number)? The key is that
months are 1-indexed (the first month is assigned 1), unlike hours, minutes, seconds, and milliseconds
(the first of which are assigned 0).

This means that rounding a [`DateTime`](@ref) to an even multiple of seconds, minutes, hours,
or years (because the ISO 8601 specification includes a year zero) will result in a [`DateTime`](@ref)
with an even value in that field, while rounding a [`DateTime`](@ref) to an even multiple of months
will result in the months field having an odd value. Because both months and years may contain
an irregular number of days, whether rounding to an even number of days will result in an even
value in the days field is uncertain.

See the [API reference](@ref stdlib-dates) for additional information
on methods exported from the `Dates` module.
