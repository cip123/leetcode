# Table of Contents
[227 Basic Calculator II](#problem227)

[394. Decode String](#problem394)

[739 Daily Temparatures](#problem739)



### <a name="problem227"></a> 227. Basic Calculator II

Implement a basic calculator to evaluate a simple expression string.

The expression string contains only non-negative integers, +, -, *, / operators and empty spaces . The integer division should truncate toward zero.

Example 1:

    Input: "3+2*2"
    Output: 7

Example 2:

    Input: " 3/2 "
    Output: 1

Example 3:

    Input: " 3+5 / 2 "
    Output: 5

*Solution 1*

Using regex, split first by `+` or `-` and then each one by `*` or `/`. We parse the integers and compute the results. The tricky part is to compute the operator position properly. This is kind of slow cause, first regex is slow and then we traverse the string multiple times.

*Solution 2*

Use a stack holding integers only. Whenever we have a `+` or `-` operation we add to the stack (with `-` operator if it's a `-`). Whenever we have a `*` or `/` we pop one compute the result and then push back in the stack. 

We need to realize that the time to compute expressions is when we see a sign or when we reach the end of the string BUT the sign we are going to use is not the sign we stopped at but the previous one so we have to keep a reference to it..

Note, I try to use the approach with 2 stacks one for operators and the other one for expressions and then to build from end to start but it didn't work out for the following case:

    100000000/1/2/3/4/5/6/7/8/9/10
    
Division with integers in java is not really associative because if we are starting from the end `9/10 = 0` and then it fucks up the whole equation.

We could try another approach without a stack. There we have to make the same distinction between calculatign totals and addign totals to the result but it is more complicated, maybe in the next step of interview preparations.

## <a name="problem394"></a>  394. Decode String

Given an encoded string, return its decoded string.

The encoding rule is: `k[encoded_string]`, where the encoded_string inside the square brackets is being repeated exactly `k` times. Note that `k` is guaranteed to be a positive integer.

You may assume that the input string is always valid; No extra white spaces, square brackets are well-formed, etc.

Furthermore, you may assume that the original data does not contain any digits and that digits are only for those repeat numbers, k. For example, there won't be input like 3a or 2[4].

Examples:

    s = "3[a]2[bc]", return "aaabcbc".
    s = "3[a2[c]]", return "accaccacc".
    s = "2[abc]3[cd]ef", return "abcabccdcdcdef".

*Solution*

We can use an object, Element  which has a multiplier and a string builder.

As a rule of thumb, I notices that problems with parenthesis are a good candidate for stacks, where the parens are the bounds between entries. So I would go like that:

        s = "3[a2[c]]", return "accaccacc".

For the first level we have `3*`, for the second level we have `a + 2*`, for the third level we have a `c`. 

Reached an `]`, resolve the `a + 2 *` last element which is `c`. Current element is now `acc`, we notice another parens we pop one more and we resolve `3 * acc`.

For this particular problem, it is useful to create a structure for each stack element which contains a prefix `a` and a multiplier `2`. 

One tricky thing is to handle the multiplier because it should always have the default value `1`. So we can't rely on it when doing the usual mathematic trick (which works with `0`). Also we should be careful with the inner loop cause it will affect the outer loop when it has more than `1` digit.
```java


while (i < s.length() && Character.isDigit(s.charAt(i))) {
    element.multiplier = 10 * element.multiplier + Character.getNumericValue(s.charAt(i));
    i++;
}
```
An alternate approach would be to store the multiplier as string also, and we use int getter which will ensure the default of 1.



### <a name="problem739"></a> 739 Daily Temparatures

 Given a list of daily temperatures T, return a list such that, for each day in the input, tells you how many days you would have to wait until a warmer temperature. If there is no future day for which this is possible, put 0 instead.

For example, given the list of temperatures `T = [73, 74, 75, 71, 69, 72, 76, 73]`, your output should be `[1, 1, 4, 2, 1, 1, 0, 0]`.

Note: The length of temperatures will be in the range `[1, 30000]`. Each temperature will be an integer in the range `[30, 100]`.     

*Solution 1*

An intresting detail is that the low range for temperatures `[30, 100]`. That means that for seeing the next temperature, instead of looking into the entries of the array, we could just store the temperature and its index into a small array, and we would just iterate this array starting from the `current temperature + 1` to the end, and see it's index. There is problem though, how do we store the temperatures so that we always now the next temperature ?

The answer is start from the end. In our example, we mark `73` with the index `7`. Then we go to index `6`, we iterate through all temperatures from `77` to `100` and when we look for the smaller index. 

Tags: #constant-input

*Solution 2*

`Input:  [73, 74, 75, 71, 69, 72, 76, 73]`

`Output: [1, 1, 4, 2, 1, 1, 0, 0]`

The brute force solution would be to look for the first bigger temperature for each project. For example, for `73` it would be `74`, for `74` it would be `75` , for `75` it would be `76` and so on. N<sup>2</sup> complexity. 

On the other hand we always have to look at the next values to get a result, so we can try to navigate it backwards. While doing this, we notice that once we have a value bigger than the previous we don't need to keep the previous one since it would not be useful anymore. 

So we begin with `73`, this is the last one, so the result would be `0` since there is no recorded temperature after it. We go to `76`, the result would be still `0`, `73` won't help us here. We can forget about `73` and we keep track of `76`. We go to `72` the first one is `76` so we need to store it's index somehow. 

Question is, now that we have `72`, do we need to keep `76` around ? Yes. We will run into `75` later, and we need to know the `76` index in order to compute it. 

So we need to keep track of the index of the temperatures in a increasing order, starting from the end. At any point, we might need to remove from the head of the list. For this situations, the best suited tool would be a stack.
