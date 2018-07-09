# The Trouble With Posits

When John Gustafson introduced unums, then posits (Type III) we thought the woes of floating point arithmetic were behind us.

Additional dynamic range, precision and the promise of accurate calculation with the help of a quire — what is not to like?

As it turns out, not everything is for the best in posit land.

## The Race to Infinity

Have a look at the illustration below. It is taken from John’s paper, _Posit Arithmetic_.

![Number wheel](https://github.com/jido/race-to-infinity/raw/master/Screen%20Shot%202018-07-08%20at%2017.08.26.png)

This wheel shows the values expressed by a rather small posit, which has just 5 bit overall length and 1 bit 
of exponent. Here you can observe a key feature of posits: tapered accuracy.

_Numbers near 1 have more precision, while extremely big numbers and extremely small numbers have less._

However the illustration does not show clearly how much gap there really is between numbers at the extreme. 
Notice how numbers jump from 4 to 8, then 8 to 16, then 16 to 64 before going to infinity.

Larger posits can express more numbers, but what I call the _**race to infinity**_ at the extreme of the 
range is the same. For example, the largest 32-bit posit with two bits of exponent is ≅1.33⨯10<sup>36</sup>. The second 
largest is ≅8.31⨯10<sup>34</sup>, a factor of 16 smaller. What kind of operation is there that produces such numbers, 
and in which cases does the difference between the two make an actual difference to the result?

It would be more useful to have more intermediate numbers even if not rising to such extreme exponents.

## The Posit Answer

Posits do offer a mechanism to achieve the above, by reducing the exponent bits. A 32-bit posit with one bit 
of exponent has a largest value of ≅1.15⨯10<sup>18</sup> while the next largest value is four times smaller, or 
≅2.88⨯10<sup>17</sup>. 

The next option is to reserve no bits for the exponent, resulting in a largest value of ≅1.07⨯10<sup>9</sup> 
with the next largest value twice as small. This is becoming a game of diminishing returns. There is no perfect 
  value for es, the number of bits in the exponent.

At the other extreme, posits can express the exact reciprocal of these largest numbers. On the example in the 
picture they are 1/64 then 1/16. Zero is an exception value:

* _If all bits are 0, the number represented is zero._

However, what to do if the result lands between 1/64 and 1/16? By reducing the exponent bits, the chasm will not 
be as wide but the dynamic range will also reduce. The smallest 32-bit number with one bit exponent is 
≅8.67⨯10<sup>-19</sup>, as opposed to ≅7.52⨯10<sup>-37</sup> with two bits for the exponent.

## A better tradeoff

I will now introduce a number format with what I think is a better trade-off between dynamic range and the gap 
between extreme numbers.

I want to preserve most features of posits, such as no wasted bits, tapered accuracy and integer-logic comparison.

Let us start by observing that extreme posit values correspond to the bits of the regime taking the entire 
number length. In the illustration above, value 64 is composed of 4 bits regime, leaving just one sign bit in 
the 5-bit number. Similarly in the largest 32-bit number the regime occupies 31 bits.

Therefore my suggestion is to introduce U, the maximum number of bits in the regime.

If the regime does not consume all bits at the extremity of the range there is an opportunity to introduce 
intermediate values by utilising the remaining bits.

The following relationship with nbits, the bit length of the number, applies:

_U + es < nbits_

As an exception, if all the bits of the regime and all the bits of the exponent are same (1 or 0), the gap 
between consecutive numbers is doubled. This replaces the exception for a number where all bits are 0 in 
standard posits. In that case the usual formula becomes:

(-1)<sup>s</sup> × useed<sup>k</sup> × ( 2<sup>e+1</sup> × f - 2<sup>e+1-b</sup> ) where b is the value of 
  the regime and exponent bit.

For example, let us take nbits=5, es=1 and U=2. The numbers in the top-right quadrant of the illustration become:

+<i style=“color:orange;”>1</i>0<b style=“color:blue;”>0</b>0 → 1

+<i style=“color:orange;”>1</i>0<b style=“color:blue;”>0</b>1 → 1.5

+<i style=“color:orange;”>1</i>0<b style=“color:blue;”>1</b>0 → 2

+<i style=“color:orange;”>1</i>0<b style=“color:blue;”>1</b>1 → 3

+<i style=“color:orange;”>11</i><b style=“color:blue;”>0</b>0 → 4

+<i style=“color:orange;”>11</i><b style=“color:blue;”>0</b>1 → 6

+<i style=“color:orange;”>11</i><b style=“color:blue;”>1</b>0 → 8

+<i style=“color:orange;”>11</i><b style=“color:blue;”>1</b>1 → 16

The dynamic range has reduced, but the gap between extreme numbers has reduced too.

How does it translate to a 32-bit number?

Let us take nbits=32, es=2 and U=10. The largest number it can express is ≅1.65⨯10<sup>12</sup>. The gap to 
the next largest value is 2097152 units. There is still over 5 decimal digits precision at this extreme, in 
stark contrast with standard posits!

If the range is too narrow, increasing either es or U makes it wider. Increasing es has more effect on the 
dynamic range but reduces precision overall. Increasing U has less effect on the dynamic range and reduces 
precision at the extremes, to the point where it can become similar to standard posits. It has no effect on 
precision around number 1.

The largest 32-bit number with two bit exponent and the maximal regime size (U=29) is ≅1.04⨯10<sup>35</sup>. 
The gap to the next largest value is ≅2.08⨯10<sup>34</sup>. We are not at the level of standard posits where 
the gap is of the same order as the largest value but it is getting closer.

If the exponent size is increased instead, with es=3 and U=10 the largest number is ≅1.81⨯10<sup>24</sup> 
and the gap to the next number is ≅4.61⨯10<sup>18</sup>.
