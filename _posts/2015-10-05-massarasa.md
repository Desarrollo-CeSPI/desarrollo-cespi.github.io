---
date: 2015-10-06 09:00:00 -3000
layout: post
title: String Filters
excerpt: conociendo Jeckyll
author: Ana
categories: String Filters
---

We need your help to make Bootstrap 4 the best it can be. Starting today, the source code for v4 will be available in a v4-dev branch on GitHub. In 
addition, we have a v4 development and tracking pull request that includes a 
master checklist of changes we’ve made and our remaining possible todos. We’d love for y’all to help chip away at those todos.
<!-- more -->

Append
Appends characters to a string.

Input
{{ 'sales' | append: '.jpg' }}

Output
sales.jpg
#camelcase

Converts a string into CamelCase.

Input

{{ 'coming-soon' | camelcase }}

Output

ComingSoon

#capitalize

Capitalizes the first word in a string

Input

{{ 'capitalize me' | capitalize }}

Output

Capitalize me

#downcase

Converts a string into lowercase.

Input

{{ 'UPPERCASE' | downcase }}

Output

#uppercase

escape

Escapes a string.

Input

{{ "<p>test</p>" | escape }}

Output

 <!-- The <p> tags are not rendered -->
<p>test</p>

#handle/handleize

Formats a string into a handle.

Input

{{ '100% M & Ms!!!' | handleize }}

Output

100-m-ms

#md5

Converts a string into an MD5 hash.

An example use case for this filter is showing the Gravatar image associated with the poster of a blog comment:

Input

<img src="https://www.gravatar.com/avatar/{{ comment.email | remove: ' ' | strip_newlines | downcase | md5 }}" />

Output

<img src="https://www.gravatar.com/avatar/2a95ab7c950db9693c2ceb767784c201" />

#newline_to_br

Inserts a <br > linebreak HTML tag in front of each line break in a string.

Input

{% capture var %}
One
Two
Three
{% endcapture %}
{{ var | newline_to_br }}

Output

One <br>
Two<br>
Three<br>

#pluralize

Outputs the singular or plural version of a string based on the value of a number. The first parameter is the singular string and the second parameter is the plural string.

Input

{{ cart.item_count }}
{{ cart.item_count | pluralize: 'item', 'items' }}

Output

3 items

#prepend

Prepends characters to a string.

Input

{{ 'sale' | prepend: 'Made a great ' }}

Output

Made a great sale

#remove

Removes all occurrences of a substring from a string.

Input

{{ "Hello, world. Goodbye, world." | remove: "world" }}

Output

Hello, . Goodbye, .

#remove_first

Removes only the first occurrence of a substring from a string.

Input

{{ "Hello, world. Goodbye, world." | remove_first: "world" }}

Output

Hello, . Goodbye, world.

#replace

Replaces all occurrences of a string with a substring.

Input

<!-- product.title = "Awesome Shoes" -->
{{ product.title | replace: 'Awesome', 'Mega' }}

Output

#Mega Shoes

replace_first

Replaces the first occurrence of a string with a substring.

Input

<!-- product.title = "Awesome Awesome Shoes" -->
{{ product.title | replace_first: 'Awesome', 'Mega' }}

Output

#Mega Awesome Shoes

slice

The slice filter returns a substring, starting at the specified index. An optional second parameter can be passed to specify the length of the substring. If no second parameter is given, a substring of one character will be returned.

Input

{{ "hello" | slice: 0 }}
{{ "hello" | slice: 1 }}
{{ "hello" | slice: 1, 3 }}

Output

h
e
ell

If the passed index is negative, it is counted from the end of the string.

Input

{{ "hello" | slice: -3, 2  }}

Output

ll

#split

The split filter takes on a substring as a parameter. The substring is used as a delimiter to divide a string into an array. You can output different parts of an array using array filters.

Input

{% assign words = "Hi, how are you today?" | split: ' ' %}

{% for word in words %}
{{ word }}
{% endfor %}

Output

Hi,
how
are
you
today?

#strip

Strips tabs, spaces, and newlines (all whitespace) from the left and right side of a string.

Input

{{ '   too many spaces      ' | strip }}

Output

too many spaces

lstrip

Strips tabs, spaces, and newlines (all whitespace) from the left side of a string.

Input

"{{ '   too many spaces           ' | lstrip }}"

Output

<!-- Notice the empty spaces to the right of the string -->
too many spaces

#rstrip

Strips tabs, spaces, and newlines (all whitespace) from the right side of a string.

Input

{{ '              too many spaces      ' | rstrip }}

Output

                 too many spaces

strip_html

Strips all HTML tags from a string.

Input

{{ "<h1>Hello</h1> World" | strip_html }}

Output

Hello World

strip_newlines

Removes any line breaks/newlines from a string.

{{ product.description | strip_newlines }}

truncate

Truncates a string down to 'x' characters, where x is the number passed as a parameter. An ellipsis (...) is appended to the string and is included in the character count.

Input

{{ "The cat came back the very next day" | truncate: 10 }}

Output

The cat...

truncatewords

Truncates a string down to 'x' words, where x is the number passed as a parameter. An ellipsis (...) is appended to the truncated string.

Input

{{ "The cat came back the very next day" | truncatewords: 4 }}

Output

The cat came back...

uniq

Removes any duplicate instances of an element in an array.

Input

{% assign fruits = "orange apple banana apple orange" %}
{{ fruits | split: ' ' | uniq | join: ' ' }}

Output

orange apple banana

upcase

Converts a string into uppercase.

Input

{{ 'i want this to be uppercase' | upcase }}

Output

I WANT THIS TO BE UPPERCASE

url_escape

Identifies all characters in a string that are not allowed in URLS, and replaces the characters with their escaped variants.

Input

{{ "<hello> & <shopify>" | url_escape }}

Output

%3Chello%3E%20&%20%3Cshopify%3E

url_param_escape

Replaces all characters in a string that are not allowed in URLs with their escaped variants, including the ampersand (&).

Input

{{ "<hello> & <shopify>" | url_param_escape }}

Output

%3Chello%3E%20%26%20%3Cshopify%3E

