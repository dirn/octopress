---
layout: post
title: "When.py: Friendly Python Dates"
date: 2012-05-24 14:26
comments: true
categories: [Python, Friendly]
---

Working with date and times in Python is not a fun task, but it's something
most people have to deal with.

<!-- more -->

To get the current date and time you go through a few different submodules:

{% codeblock lang:python Get current date and time %}
>>> from datetime import date
>>> date.today()
datetime.date(2012, 5, 24)
>>>
>>> from datetime import datetime
>>> datetime.now()  # local time
datetime.datetime(2012, 5, 24, 14, 3, 25, 98829)
>>> datetime.utcnow()  # UTC
datetime.datetime(2012, 5, 24, 18, 3, 25, 99288)
>>>
>>> datetime.time(datetime.now())
datetime.time(14, 3, 25, 100212)
{% endcodeblock %}

Trying to calculate a new `datetime` from an existing one requires introducing
another submodule:

{% codeblock lang:python Calculate new datetimes %}
>>> from datetime import datetime, timedelta
>>> datetime.now() + timedelta(days=1)
datetime.datetime(2012, 5, 25, 14, 3, 41, 721367)
>>> datetime.now() + timedelta(weeks=2, hours=-3)
datetime.datetime(2012, 6, 7, 11, 3, 41, 722905)
{% endcodeblock %}

If you've ever worked with `timedelta` objects before, you might have the same
complaint that I do: it only works with definitive amounts of time. Need to add
some number of days? Sure, that's easy. Need to add a week? Go right ahead.
Need to add a month? Hmm, that's too hard, sorry. A year? You're kidding, right?

And forget about trying to change the time zone of a `datetime`. This isn't
really such a big deal if you're fortunate enough to work with "aware"
datetimes. I spend much of my time working with "naive" objects. For most
people, this probably isn't a big deal either. For me, however, this is a big
deal as I work on a national network of sites providing local content across
four different time zones.

We came up with a solution for this that seemed to work.

{% codeblock lang:python First attempt at changing time zones %}
>>> from datetime import datetime
>>> import pytz
>>>
>>> dt = datetime.now()
>>> dt_utc = datetime.utcnow()
>>>
>>> eastern = pytz.timezone('America/New_York')
>>>
>>> dt
datetime.datetime(2012, 5, 24, 14, 4, 0, 566448)
>>> dt_utc.replace(tzinfo=pytz.UTC).astimezone(eastern).replace(tzinfo=None)
datetime.datetime(2012, 5, 24, 14, 4, 0, 566817)
>>>
>>> dt_utc
datetime.datetime(2012, 5, 24, 18, 4, 0, 566817)
>>> dt.replace(tzinfo=eastern).astimezone(pytz.UTC).replace(tzinfo=None)
datetime.datetime(2012, 5, 24, 19, 4, 0, 566448)
{% endcodeblock %}

As you can see, going from UTC to Eastern works just fine. When going in the
other direction, however, an hour is lost.

After some more digging around we came up with a better approach.

{% codeblock lang:python Second attempt at changing time zones %}
>>> from datetime import datetime
>>> import pytz
>>>
>>> dt = datetime.now()
>>> dt_utc = datetime.utcnow()
>>>
>>> eastern = pytz.timezone('America/New_York')
>>>
>>> dt
datetime.datetime(2012, 5, 24, 14, 4, 20, 436153)
>>> pytz.UTC.localize(dt_utc).astimezone(eastern).replace(tzinfo=None)
datetime.datetime(2012, 5, 24, 14, 4, 20, 436485)
>>>
>>> dt_utc
datetime.datetime(2012, 5, 24, 18, 4, 20, 436485)
>>> eastern.localize(dt).astimezone(pytz.UTC).replace(tzinfo=None)
datetime.datetime(2012, 5, 24, 18, 4, 20, 436153)
{% endcodeblock %}

This new method managed to correctly switch between time zones. But that's a
lot of somewhat awkward code to have to type out. Sure a function can be created
to reduce a lot of the code--we called ours `convert_timezone`--but still,
that's yet another module that needs to be imported.

Why do we need to import all of these different modules to perform such
commonplace tasks? Why not have just one module that covers everything? That's
where When.py comes in.

Here's how to redo all of the above code:

{% codeblock lang:python When.py %}
>>> import when
>>>
>>> when.today()  # date.today()
datetime.date(2012, 5, 24)
>>> when.now()  # datetime.now()
datetime.datetime(2012, 5, 24, 14, 4, 37, 64295)
>>> when.now(utc=True)  # datetime.utcnow()
datetime.datetime(2012, 5, 24, 18, 4, 37, 64796)
>>>
>>> when.future(years=1)  # datetime.now() + timedelta(???)
datetime.datetime(2013, 5, 24, 14, 4, 37, 65610)
>>> when.past(months=2)  # datetime.now() - timedelta(???)
datetime.datetime(2012, 3, 24, 14, 4, 37, 66268)
>>>
>>> # convert_timezone(...)
... when.shift(when.now(utc=True), from_tz='UTC', to_tz='America/New_York')
datetime.datetime(2012, 5, 24, 14, 4, 37, 67562)
{% endcodeblock %}

Check out the documentation on
[Read the Docs](http://readthedocs.org/docs/whenpy/en/latest/)
or get started with `pip install whenpy` or
[Fork it on GitHub](https://github.com/dirn/when.py)
