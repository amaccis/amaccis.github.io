---
title: "TAI and PHP"
date: 2023-03-23
draft: false
tags: ["English", "PHP"]
---

The first time I've dealt with [**TAI**](https://en.wikipedia.org/wiki/International_Atomic_Time) (*Temps Atomique International/International Atomic Time*) was a few years ago, 
when I was looking for a "disappeared" email through qmail logs. My first reaction was like: these guys (the ones who set up the logs like that) are nuts! 

Don't get me wrong, TAI is awesome as means of time-keeping, because it is extremely precise 
([atomic clocks deviate only 1 second in up to 100 million years](https://www.timeanddate.com/time/how-do-atomic-clocks-work.html)) and we can also consider it very nerdy, just 
think that one TAI second is defined as the duration of 9.192.631.770 periods of the radiation corresponding to the transition between the two hyperfine levels of the ground 
state of the cesium atom (or, if it's clearer, the time it takes a cesium-133 atom at the ground state to oscillate exactly 9.192.631.770 times). So, so far so good, but 
when you are used to deal with standard looking log files, and you see something like that
```bash
@4000000052a82012173eb0f4 new msg 4242424242
@4000000052a82012173f02fc info msg 4242424242: bytes 12834 from <ford.prefect@betelgeuse-v.universe> qp 17478 uid 42
@4000000052a82012173f2a0c starting delivery 1056684: msg 4242424242 to local arthur.dent@earth.universe
@4000000052a820121825d0ec delivery 1067852: success: did_1+0+0/
@4000000052a82012193141c4 status: local 0/10 remote 0/40
@4000000052a820121931e1ec end msg 4242424242
```
you can only ask yourself: what?

At that time, since I thought I could need to handle that kind of time format with automated scripts or a web application, I wrote a **PHP** library called 
[tai64-datetime](https://github.com/amaccis/tai64-datetime).
The idea was extending the PHP [`DateTime` class](https://www.php.net/manual/en/class.datetime.php) with a new simple feature that could convert **[UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)** into TAI 
and vice versa. The library turned out to be very bad (poorly thought and written) and at the end of the day fortunately I did not need it, so it remained unused for years.

A few weeks ago I thought to refactor the code, but, reviewing it I recalled what kind of stupid things I wrote, so I decided to abandon the old library and write a new one. 
Not that now I'm capable of doing wonders, but I thought it was funny because of the chance to play a bit with the new features of PHP (especially 
[Enums](https://www.php.net/manual/en/language.types.enumerations.php)) and GitHub Actions. And mostly, thinking I was funny too, I decided to call the new library DateTai! 
You know, sounds similar to DateTime but with TAI...

Joking aside, the new library forced me to address a few interesting topics, also because, before starting, I needed to brush up on 
[a few key concepts](http://cr.yp.to/libtai/tai64.html). 

A _TAI64 label_ is an integer between 0 and 2^64 referring to a particular second of real time. This integer, let's call it _s_, refers to: 

 - the TAI second beginning 2^62 - _s_ seconds before the beginning of 1970 TAI, if 0 ≤ _s_ < 2^62;
 - the TAI second beginning _s_ - 2^62 seconds after the beginning of 1970 TAI, if 2^62 ≤ _s_ < 2^63.

It seems that the integers lesser than 2^63 are enough to cover the entire expected lifetime of the universe, but if you have higher expectations, know that integers ≥ 2^63 are 
reserved for future extensions.

TAI64 labels are normally stored or communicated using the _external TAI64 format_, consisting of eight 8-bit bytes in big-endian format. For example, bytes 
40 00 00 00 00 00 00 00 hexadecimal represent the second that began 1970 TAI.

Continuing, a _TAI64N label_ refers to a particular nanosecond of real time. It has two parts:

 - a TAI64 label;
 - an integer, between 0 and 999.999.999 counting nanoseconds from the beginning of the second represented by the TAI64 label.

A TAI64N label is normally stored or communicated using the _external TAI64N format_, consisting of twelve 8-bit bytes. The first eight bytes are the TAI64 label in external 
TAI64 format. The last four bytes are the nanosecond counter in big-endian format.

And finally, a _TAI64NA label_ refers to a particular attosecond of real time. It has two parts:

- a TAI64N label;
- an integer, between 0 and 999999999, counting attoseconds from the beginning of the nanosecond represented by the TAI64N label.

A TAI64NA label is normally stored or communicated using the _external TAI64NA format_, consisting of sixteen 8-bit bytes. The first twelve bytes are the TAI64N label in external 
TAI64N format. The last four bytes are the attosecond counter in big-endian format.

So putting aside the Pig Latin, basically, how does this stuff work? Let's check the first TAI64 format example, 40 00 00 00 00 00 00 00, that represents the second that began
1970 TAI.

If we isolate the bytes
```
| 40 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |
| b0 | b1 | b2 | b3 | b4 | b5 | b6 | b7 |
```
and we start counting them from the left, because we said it's [big-endian format](https://en.wikipedia.org/wiki/Endianness), we apply
```
b0*2^56 + b1*2^48 + b2*2^40 + b3*2^32 + b4*2^24 + b5*2^16 + b6*2^8 + b7
```
and in our case, since the decimal value of the hexadecimal _40_ is 64, we have
```
64*2^56 + 0*2^48 + 0*2^40 + 0*2^32 + 0*2^24 + 0*2^16 + 0*2^8 + 0 = 4611686018427387904
```
and _4.611.686.018.427.387.904_ is exactly 2^62. So 2^62 - s = 0 and here we are with the second that began 1970 TAI. Quite simple.

Anyway, nothing to be worried about though, because PHP does the job for us through `hexdec()`.

So, to recap what we know:

- 0 < _s_ < 2^64
- TAI second = 2^62 - _s_ seconds before the beginning of 1970 TAI, if 0 ≤ _s_ < 2^62;
- TAI second = _s_ - 2^62 seconds after the beginning of 1970 TAI, if 2^62 ≤ _s_ < 2^63.

in order to compute TAI seconds, we could write something like this
```php
private const TAI64_LABEL_MINIMUM_VALUE = 0;
private const TAI_SECOND_EPOCH = 2**62;
private const TAI64_LABEL_MAXIMUM_VALUE = (2**63)-1;

private static function externalTai64FormatToTaiSecond(string $tai64Label): string
{

    $second = intval(hexdec($tai64Label));
    if ($second < self::TAI64_LABEL_MINIMUM_VALUE) {
        throw new InvalidArgumentException();
    }
    elseif ($second < self::TAI_SECOND_EPOCH) {
        return (string) -(self::TAI_SECOND_EPOCH - $second);
    } elseif ($second <= $this->TAI64_LABEL_MAXIMUM_VALUE) {
        return (string) ($second - self::TAI_SECOND_EPOCH);
    }
    else {
        throw new InvalidArgumentException();
    }

}
```

But, let's focus on the edge cases.

We know that PHP's int type is signed, but [`hexdec()` and `dechex()` (its opposite) deal both with unsigned integers](https://www.php.net/manual/en/function.dechex.php), so 
negative integers will be treated as though they were unsigned, meaning that we can not expect a result below zero by `intval(hexdec($tai64Label))`.

In addition, we know that the maximum value for an integer in a 64-bit architecture is 2^63 - 1, it is no coincidence that is the exact value of `PHP_INT_MAX`. 
We would need to use things like `ext-bcmath` in order to handle (as a string) the result of the operation `(2**63)-1` that, for the purpose of the library, would be pointless, 
since we only need to check if the value of _s_ is greater than or equal to 2^62.

So we can rewrite the code above, this way
```php
private const TAI_SECOND_EPOCH = 2**62;

private static function externalTai64FormatToTaiSecond(string $tai64Label): string
{

    $second = intval(hexdec($tai64Label));
    if ($second < self::TAI_SECOND_EPOCH) {
        return (string) -(self::TAI_SECOND_EPOCH - $second);
    } else {
        return (string) ($second - self::TAI_SECOND_EPOCH);
    }

}
```
and the game is done.

I mentioned nanoseconds and attoseconds above, but [`DateTimeInterface` only supports microseconds at most](https://www.php.net/manual/en/datetime.format.php), so we need to use
rounded values
```php
$nanosecond = intval(hexdec(substr($tai64NLabel, 16)));
$microsecond = floatval(round($nanosecond/1000)/1000000);
```
and also, since [PHP 8.1.7](https://www.php.net/ChangeLog-8.php#8.1.7) solves [a bug](https://github.com/php/php-src/issues/7758) with negative timestamps, we need to set that
version as minimum necessary.

Now let's briefly talk about [leap seconds](https://en.wikipedia.org/wiki/Leap_second). 

On the one hand, as already mentioned, we have TAI, very precise, but on the other hand we have the observed solar time [UT1](https://en.wikipedia.org/wiki/Universal_Time) that 
varies due to irregularities and long-term slowdown in the Earth's rotation. The UTC time standard uses TAI and consequently would run ahead of observed solar time unless it is 
reset to UT1 as needed. In order to accommodate the difference between these two standards, occasionally the so-called leap seconds are applied and since the Earth's rotational 
speed varies in response to climatic and geological events, UTC leap seconds are irregularly spaced and unpredictable.

So even if [leap seconds are going to be scrapped by 2035](https://www.theguardian.com/world/2022/nov/18/do-not-adjust-your-clock-scientists-call-time-on-the-leap-second)
what we know is that the conversions work this way:
```
UTC to TAI = UTC + leap seconds
TAI to UTC = TAI - leap seconds
```

That's why we can implement the conversion like this
```php
public const LEAP_SECONDS = [
    '1972-01-01' => '10.0',
    '1972-07-01' => '11.0',
    '1973-01-01' => '12.0',
    '1974-01-01' => '13.0',
    '1975-01-01' => '14.0',
    '1976-01-01' => '15.0',
    '1977-01-01' => '16.0',
    '1978-01-01' => '17.0',
    '1979-01-01' => '18.0',
    '1980-01-01' => '19.0',
    '1981-07-01' => '20.0',
    '1982-07-01' => '21.0',
    '1983-07-01' => '22.0',
    '1985-07-01' => '23.0',
    '1988-01-01' => '24.0',
    '1990-01-01' => '25.0',
    '1991-01-01' => '26.0',
    '1992-07-01' => '27.0',
    '1993-07-01' => '28.0',
    '1994-07-01' => '29.0',
    '1996-01-01' => '30.0',
    '1997-07-01' => '31.0',
    '1999-01-01' => '32.0',
    '2006-01-01' => '33.0',
    '2009-01-01' => '34.0',
    '2012-07-01' => '35.0',
    '2015-07-01' => '36.0',
    '2017-01-01' => '37.0',
    // the future leap seconds entries need to be added here
];

private static function adjust(DateTimeInterface $dateTimeInterface, string $adjustmentString) : DateTimeInterface
{

    $leapSeconds = self::LEAP_SECONDS;
    $leadSecondsApplicationStartingDate = array_keys($leapSeconds)[0];
    $dateToAdjust = $dateTimeInterface->format('Y-m-d');
    if ($dateToAdjust >= $leadSecondsApplicationStartingDate) {
        if (!array_key_exists($dateToAdjust, $leapSeconds)) {
            $leapSeconds[$dateToAdjust] = '0';
            ksort($leapSeconds);
            $keys = array_keys($leapSeconds);
            $key = (array_search($dateTimeInterface->format('Y-m-d'), $keys, true) - 1);
            $leapSecondsValue = $leapSeconds[$keys[$key]];
        } else {
            $leapSecondsValue = $leapSeconds[$dateToAdjust];
        }
        $seconds = number_format($leapSecondsValue, 0, '.', '');
        $modify = sprintf($adjustmentString, $seconds);
        return $dateTimeInterface->modify($modify);
    }
    return $dateTimeInterface;

}
```

There could be a lot more to say about, but after all this sermon, if you want to play a little with [my library](https://github.com/amaccis/datetai) and tell me how many bugs 
you find, just install it and let me know!
```bash
composer require amaccis/datetai
```
