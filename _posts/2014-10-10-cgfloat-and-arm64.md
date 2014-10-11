---
layout: post
title: CGFloat in a 64-bit world
summary: Preprocessor macros for a 64-bit world.
tags: ios
---

Apple's A7 chip famously moved to a 64-bit architecture. This ushered in an era of access to more memory* and a `CGFloat` type previously aliased to `float` newly redefined as `double`. Wait, what?

If you're writing an iOS app that targets multiple architectures, you'll want to use the proper `CGFloat` type for the current device's architecture. You'll also want to use the correct `float` or `double` versions of math functions like `floor` (double) or `floorf` (float).

Thanks to some preprocessor magic and the `CGFLOAT_IS_DOUBLE` define, we can construct some helpers that call the correct math function and return a number of the correct type for the current architecture. Behold, `SPLFloat.h`!

**Update:** One alternative to the below is [`tgmath.h`](http://pubs.opengroup.org/onlinepubs/009695399/basedefs/tgmath.h.html), though it must be included before `math.h`. That can be challenging if you're using CocoaPods.

{% highlight objc %}

// SPLFloat.h

@import Foundation;

#pragma once

CG_INLINE CGFLOAT_TYPE SPLFloat_floor(CGFLOAT_TYPE cgfloat)
{
#if CGFLOAT_IS_DOUBLE
	return floor(cgfloat);
#else
	return floorf(cgfloat);
#endif
}

CG_INLINE CGFLOAT_TYPE SPLFloat_ceil(CGFLOAT_TYPE cgfloat)
{
#if CGFLOAT_IS_DOUBLE
	return ceil(cgfloat);
#else
	return ceilf(cgfloat);
#endif
}

CG_INLINE CGFLOAT_TYPE SPLFloat_round(CGFLOAT_TYPE cgfloat)
{
#if CGFLOAT_IS_DOUBLE
	return round(cgfloat);
#else
	return roundf(cgfloat);
#endif
}

CG_INLINE CGFLOAT_TYPE SPLFloat_abs(CGFLOAT_TYPE cgfloat)
{
#if CGFLOAT_IS_DOUBLE
	return fabs(cgfloat);
#else
	return fabsf(cgfloat);
#endif
}

CG_INLINE CGFLOAT_TYPE SPLFloat_pow(CGFLOAT_TYPE cgfloat, 
                                    CGFLOAT_TYPE exp)
{
#if CGFLOAT_IS_DOUBLE
	return pow(cgfloat, exp);
#else
	return powf(cgfloat, exp);
#endif
}
{% endhighlight %}

\* My kingdom for an iPhone with more than 1GB RAM.