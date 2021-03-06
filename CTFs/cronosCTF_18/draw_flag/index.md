# Jack The Swimmer

>We've been following the description on the scroll, swimming in all the
>directions it wants us. Unfortunately we don't know where we should start swimming!

### [~$ cd ..](../)

We are given a [text file](DrawFlag.txt) containing a bunch of shorts instructions. According to the title, I guessed that each paragraph
draws a letter, giving the flag at the end of the day.

## First and last letter

> ```
>Go straight 100 spaces
>Go reverse 200 spaces
>Go straight 100 spaces
>Turn right by 90 degrees
>Go straight 100 spaces
>Turn right by 90 degrees
>Go straight 100 spaces
>Go reverse 200 spaces
> ```

Could be an **H** or an **I**, depending on the direction

## Second letter

> ```
>Turn right by 10 degrees					
>Go straight 200 spaces
>Turn right by 150 degrees
>Go straight 200 spaces
>Go reverse 100 spaces
>Turn right by 120 degrees
>Go straight 50 space
> ```

Because of the 1st and the 3rd instruction, I guessed that it was a "spiky" letter. Because of the last one, which draws a middle-sized bar, it's an **A**

## Third letter

The longest one:

> ```
>Turn left by 90 degrees
>Go straight 50 spaces
>Go straight 1 spaces
>Turn left by 1 degrees
>Go straight 1 spaces
>Turn left by 1 degrees
>Go straight 1 spaces
>Turn left by 1 degrees
>...
>Go straight 1 spaces
>Turn right by 1 degrees
>Go straight 1 spaces
>Turn right by 1 degrees
>Go straight 1 spaces
>Turn right by 1 degrees
>Go straight 50 spaces
> ```

Because of the long sequence of short distances, I guessed that is was a curvy letter. At the beginning, the curves goes to the left, and the direction changes in the middle. It's then an **S**

The flag is then "IASI", or, even better, **HASH**
