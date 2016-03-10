---
layout: post
title: "Next Version Dojo"
date: 2016-03-10T12:07:59-05:00
comments: true
---

In the [Yammer](http://www.yammer.com) Redmond office, we usually do a little coding Dojo on Wednesday after lunch. We pick a small problem and work on it (individually or in pairs) and then compare our solutions.

Yesterday the problem was called "Next Version":


> Create a function nextVersion, that will take a string in parameter, and will return a string containing the next version number.

> Example:


> <pre>nextVersion("1.2.3") === "1.2.4";
nextVersion("0.9.9") === "1.0.0.";
nextVersion("1") === "2";
nextVersion("1.2.3.4.5.6.7.8") === "1.2.3.4.5.6.7.9";
nextVersion("9.9") === "10.0";</pre>

> Rules:

> All numbers, except the first one, must not be greater than 9: if there are, you have to set them to 0 and increment the next number in sequence.


So I used me some Ruby.

The first version was a naive implementation:

{% highlight ruby %}
  def self.next_version_1 version
    number_as_string = version.split(".").join('')

    string_length = number_as_string.length

    next_version_string = (number_as_string.to_i+1).to_s

    if next_version_string.length==string_length
      next_version_string.split('').join(".")
    else
      next_version_string[0..1] + "." + next_version_string[2..-1].split('').join(".")
    end
  end
{% endhighlight %}

Essentially:

1. Remove the dots from the string.
2. Store the length, to check later if we've gone from, say 99 to 100
3. Add one to the version
4. If our string hasn't gotten any longer, just split it into individual characteres (numbers) and join with dots (".")
5. If it _has_ grown, take the first two digits as the major version number, and dot-separate the rest.

Now, this has some gaping holes, but all the examples given above will pass. However, if you have a number that starts with a "0" and should still have a "0" after incrementing, say, "0.8.9" => "0.9.0", it will fail. When you change the string "089" to an integer, it will become `89`, and when you add one you'll get 90 => "90". Since the length changed ("089" is 3 characters, "90" is two), the above algorithm will return the next version as "90". "0.8.9" to "90". Not the jump we wanted.

We can fix this by checking if we started with zero:

{% highlight ruby %}
  def self.next_version_2 version
    number_as_string = version.split(".").join('')

    starts_with_zero = number_as_string[0].to_i == 0

    string_length = number_as_string.length

    next_version_string = (number_as_string.to_i+1).to_s

    if next_version_string.length > string_length
      next_version_string[0..1] + "." + next_version_string[2..-1].split('').join(".")
    else
      next_version_string = "0" + next_version_string if starts_with_zero && next_version_string.length < string_length
      next_version_string.split('').join(".")
    end
  end
{% endhighlight %}

The same algorithm, except we check if our version begins with zero, and at the end, if it should still have a zero at the front we prepend it to the string.

This _still_ has a problem, though. If we want to go from version "10.9" to "11.0", it will fail; since the lengths of the strings don't change, it will return "1.1.0". Ooops.

{% highlight ruby %}
  def self.next_version_3 version
    version_array = version.split(".").reverse.map(&:to_i)

    version_array[0] = version_array.first + 1

    version_array = version_array.each_with_index.map do |n,index|
      if index == version_array.size - 1
        n
      else
        if n > 9
          version_array[index+1]=version_array[index+1]+1
          0
        else
          n
        end
      end
    end

    version_array.reverse.join(".")
  end
{% endhighlight %}

There are ways you could take the same algorithm I started with and make it work. Instead, I decided to change tactits. The 3rd algorithm:

1. Splits the version string into a array of integers and reverses it.
2. Adds 1 to the first element in the array.
3. For each element except the last, if it is greater than 9, add 1 to the _next_ number in the list.
4. Reverse the array and join into a dot-separated string again.

Full file with tests is [here](https://gist.github.com/philcrissman/cb30642c566e64fe5dce)

