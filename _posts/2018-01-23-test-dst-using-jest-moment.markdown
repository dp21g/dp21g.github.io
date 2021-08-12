---
layout: post
title:  "Test DST using Jest / Moment / JS Dates"
date:   2018-01-23 22:22:47 +0100
categories: Javascript
---
If you are using a Javascript dates in your code and are using Jest snapshots for testing and you work in a team across timezones, then this may be helpful.
The snapshot dates are written in UTC time. However, testing a DST function which returns the date/time for the clock change for the timezone you are in will be an issue.
In order for you to consistently return the DST change for GMT timezone, you need to be able to control the timezone when the tests are run. One of the popular solutions is using moment.js to control the timezone.

```js
//DST function using moment to control timezone

  getDSTDate(date1, date2) {
    const durationHour = 36e5;
    const m1 = moment(date1);
    const m2 = moment(date2);

    const checkDST = (d1, d2) => {
      if (d2.valueOf() - d1.valueOf() <= durationHour) {
        d2.set({ minute: 0, second: 0, millisecond: 0 });
        return d2.toDate();
      }
      const mid = moment(d1.valueOf() + ((d2.valueOf() - d1.valueOf()) / 2));
      return d1.isDST() !== mid.isDST() ? checkDST(d1, mid) : checkDST(mid, d2);
    };
    return (m1.isDST() !== m2.isDST()) ? checkDST(m1, m2) : null;
  }
```

In your tests you would need to set the timezone as follows:

```js
// jest tests

describe("getTickValues method", () => {
  const dispformat = "DD-MMM-YYYY hh:mm";

  beforeAll(() => {
    moment.tz.setDefault("Europe/London");
  });

  it("generates tick values with DST", () => {
    const d1 = new Date("2017-10-29T00:26:00Z"); //UTC time
    const d2 = new Date("2017-10-29T02:12:00Z"); //UTC time
    const expectedValue = "29-Oct-2017 01:00";

    expect(moment(getDSTDate(d1, d2)).format(dispformat)).toEqual(
      expectedValue
    );
  });

  afterAll(() => {
    moment.tz.setDefault();
  });
});
```

{% include like.html %}

