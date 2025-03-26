[[#var]]
[[#let]]
[[#const]]

### What's the difference between the 3 declarations?

#### var
`var` is global when declared outside function, and it's function scoped when declared inside the function.

`var` declarations can be hoisted to the top, see [[Hoisting]].

```js
// We might accidently re-define the variable
var greeter = 'hi';
if (cond === true) {
	var greeter = 'bye';
}
```


#### let
Block scoped. It ends at next `}`. `let` can be updated, but not re-declared.
While `var` can be re-declared.

It's perfectly safe to return an object declared with `let`. JS garbage collection will only ever clean up something that's not in use.

Like `var`, `let` declarations can also be hoisted to the top.


#### const
Block scoped like `let`. Can not be updated or redeclared.
Note although `const` object can't be updated, its properties can.
Must be initialised during declaration.
Can't be re-assigned.





reflow and justify
```

"""

We are building a word processor and we would like to implement a "reflow" functionality that also applies full justification to the text.

  

Given an array containing lines of text and a new maximum width, re-flow the text to fit the new width. Each line should have the exact specified width. If any line is too short, insert '-' (as stand-ins for spaces) between words as equally as possible until it fits.

  

Note: we are using '-' instead of spaces between words to make testing and visual verification of the results easier.

  
  
  
  
  

reflowAndJustify(lines, 25) "reflow lines and justify to length 25" =>

  

        [ "The-day-began-as-still-as"

          "the-----night----abruptly"

          "lighted---with--brilliant"

          "flame" ]

  

reflowAndJustify(lines, 26) "reflow lines and justify to length 26" =>

  

        [ "The--day-began-as-still-as",

          "the-night-abruptly-lighted",

          "with----brilliant----flame" ]

  

reflowAndJustify(lines, 40) "reflow lines and justify to length 40" =>

  

        [ "The--day--began--as--still--as-the-night",

          "abruptly--lighted--with--brilliant-flame" ]

  

reflowAndJustify(lines, 14) "reflow lines and justify to length 14" =>

  

        ['The--day-began',

         'as---still--as',

         'the------night',

         'abruptly',

         'lighted---with',

         'brilliant',

         'flame']

  

reflowAndJustify(lines, 15) "reflow lines and justify to length 15" =>

  

        ['The--day--began',

         'as-still-as-the',

         'night--abruptly',

         'lighted----with',

         'brilliant-flame']

  

lines2 = [ "a b", "c d" ]         

  

reflowAndJustify(lines2, 20) "reflow lines2 and justify to length 20" =>

  

        ['a------b-----c-----d']

        [a b, c d]

        The--day--began--as-still

  

reflowAndJustify(lines2, 4) "reflow lines2 and justify to length 4" =>

  

        ['a--b',

         'c--d']

  

reflowAndJustify(lines2, 2) "reflow lines2 and justify to length 2" =>

  

        ['a',

         'b',

         'c',

         'd']

  

All Test Cases:

                 lines, reflow width

reflowAndJustify(lines, 24)

reflowAndJustify(lines, 25)

reflowAndJustify(lines, 26)

reflowAndJustify(lines, 40)

reflowAndJustify(lines, 14)

reflowAndJustify(lines, 15)

reflowAndJustify(lines2, 20)

reflowAndJustify(lines2, 4)

reflowAndJustify(lines2, 2)

  

n = number of words OR total characters

"""

lines = ["The day began as still as the","night abruptly lighted with","brilliant flame"]

lines2 = ["a b","c d"]

  
  
  
  
  
  
  
  

# [The--day--began--as--still]  == new

# [as--the--night--abruptly]

  
  
  

# lines = [ "The day began as still as the",

#           "night abruptly lighted with",

#           "brilliant flame" ]

  

# reflowAndJustify(lines, 24) "reflow lines and justify to length 24" =>

  

#         [ "The--day--began-as-still",  

#           "as--the--night--abruptly",

#           "lighted--with--brilliant",

#           "flame" ] // <--- a single word on a line is not padded with spaces

  
  
  

'''

words: "The,day,began,as,still,as,the"   began

number:24

current_count: 7

current_words: [the, day, ] 

  

new_word_length:

  
  

'''

def reflowAndJustify(lines, number):

    current_words = []

    current_count = 0

    output = []

    for line in lines:

        # append word into current_words   

        words = line.split(' ')

        for word in words:

            if current_count < number:

                # I can add word into current_words

  

                new_word_length = current_count + len(word)

                if current_count != 0:

                    new_word_length += 1  #'The-day-began-'

                # we can add

                if new_word_length <= number:

                    current_words.append(word)

                    current_count = new_word_length

                else:

                    # condition for single word

                    if len(current_words) == 1:

                        output.append(current_words[0])

                        # clear output

                        current_count = 0

                        current_words = []

                        continue

                    # not enough to fit a new word

                    idx = 0

                    while current_count <= number:

                        w = current_words[idx]   #[the-, day-]

                        w += '-'

                        current_count += 1

                        idx = (idx + 1) % len(current_words)

                    # current_count == number

                    output_line = '-'.join(current_words)

                    output.append(output_line)

                    # reset

                    current_count = 0

                    current_words = []

    return output     

  
  

print(reflowAndJustify(lines, 24))

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

# n is total number of words

  

def wrapLines(words, max_count):

    current_line = ''

    ans = []

    for idx in range(0, len(words)):

        # add

        if current_line == '':

            current_line += words[idx]

            continue

        new_length = 1 + len(words[idx]) + len(current_line)

        if new_length <= max_count:

            # put into current line

            current_line += ('-' + words[idx])

        else:

            # put current_line into ans

            ans.append(current_line)

            # reset

            current_line = ''

            current_line += words[idx]

    # if current_line != ''

    if current_line != '':

        ans.append(current_line)

    return ans
```
