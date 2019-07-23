# PHP-Timer

PHP Class for code timer

This is a simple class for adding a timer to your PHP script.

This is NOT my code. I originally found it at codeaid.net but that site has since disappeared so I decided to preserve it.

I dug the following out of the Wayback machine archive here:

https://web.archive.org/web/20170531044352/http://codeaid.net/php/calculate-script-execution-time-(php-class)


==============================

## Using PHP-Timer

Sometimes when writing PHP code that is supposed to run fast we use the script execution calculation functions. Usually it involves (at least for me :) copying the code from example #1 on PHP's microtime function page and pasting it into your script only to do the same all over again the next time.

However, as good as that script is, it is not versatile enough. It does not, for example, allow you to time execution of only some lines or even more - some separated lines.

Let's look at an example:

    for ($i = 0; $i < 100; $i++) {
         fx();
         fy();
         fz();
    }

What if we needed to time the execution of fy()? It's not impossible to do using the usual measures however it becomes a bit tedious. We would have to have multiple microtime calls, some local variable to store the total time and so on.

Therefore, to make my and hopefully someone else's life easier, I wrote a class for those occasions. It allows you to start, stop, pause and resume timing of specific parts of your code. It would even allow you to calculate total time it takes to run fx() AND fz() together.

Here is the full source code of the class:

Download this snippet [Static class](class.Timer.non-static.php)

First, let's look at the public methods that are available to you:

* start() - start/resume the timer
* stop() - stop/pause the timer
* reset() - reset the timer
* get([$format]) - stops the timer (if required) and returns the amount of time as number of seconds, milliseconds or microseconds.

The type of the result depends on the value of the $format parameter.
The format can be one of the following three:

* Timer::SECONDS (default)
* Timer::MILLISECONDS
* Timer::MICROSECONDS.

If the format is defined as seconds, the result will be a float, otherwise an int.

## How it works

The class has a "queue" of times.

One Timer::start()/Timer::stop() cycle counts as one entry in the queue with its own execution time.

Therefore if you only do start() and stop() once, one queue item will be added.

If you call Timer::start() and Timer::stop() multiple times without calling Timer::reset() then multiple entries will get added to the queue.

Then, when you call Timer::get() the function will go through every queue entry, calculate the time that it took to execute and sum everything up to get the total time.

Whenever you need to restart the calculation just call Timer::reset(). That will clear the queue of current times.

## Examples

The first and the most usual scenario, where you want to calculate the time it takes to run your whole script, is the easiest one.

Just add Timer::start() at beginning of the file/script and print Timer::get() at the end.

Next, let's rewrite the examples we mentioned earlier. As you will see it is not much more different than timing the whole script:

    for ($i = 0; $i < 100; $i++) {
         fx();
         // calculate the time it takes to run fy()
         Timer::start();
         fy();
         Timer::stop();
         fz();
    }
  
    print Timer::get();
    // or
    print Timer::get(Timer::MICROSECONDS);
    // or
    print Timer::get(Timer::MILLISECONDS);

As previously mentioned, if you wanted to calculate the total time it takes to execute fx() AND fz() we would need to adjust the code to wrap two blocks of code instead of just one:

    for ($i = 0; $i < 100; $i++) {
         // calculate the time it takes to run fx()
         Timer::start();
         fx();
         Timer::stop();
  
         fy();
  
         // calculate the time it takes to run fz()
         Timer::start();
         fz();
         Timer::stop();
    }
  
    print Timer::get();

In case you want to track how much time it has taken for the script to run "up to now" you can simply call the Timer::get() function in the middle of the loop:

    for ($i = 0; $i < 100; $i++) {
         fx();
         // calculate the time it takes to run fy()
         Timer::start();
         fy();
         Timer::stop();
         fz();
  
         print Timer::get();
    }

*IMPORTANT!*

Timer::get() also stops the timer as it needs to set the final queue entry's end time.

Therefore, if you call it between two blocks of code where the time gets calculated, you need to make sure you resume the timer by calling Timer::start().

## Non-static version of the class

For those that don't like static classes or who might need multiple instances of the Timer class, here is a modified version of the static class:

Download this snippet - [Static class](class.Timer.static.php)

The way you would use this class is pretty similar to the static class except you can instantiate it and have multiple timers running on your page:

    // instantiate two copies of the Timer class
    $t1 = new Timer();
    $t2 = new Timer();
  
    for ($i = 0; $i < 100; $i++) {
      // calculate the time it takes to run fx() using timer #1
      $t1->start();
      fx();
      $t1->stop();

      // calculate the time it takes to run fy() using timer #2
      $t2->start();
      fy();
      $t2->stop();
    }
  
    // get results
    print $t1->get();
    print $t2->get(Timer::MILLISECONDS);

## Test Sample

    <?php
      Timer::start();
      for ($i = 0; $i < 1000000; $i++) {
       $x = log($i);
      }
      Timer::stop();
  
      var_dump(Timer::get(Timer::MILLISECONDS)); // int(728)
      var_dump(Timer::get(Timer::MICROSECONDS)); // int(728340)
      var_dump(Timer::get(Timer::SECONDS));      // float(0.72834)
    ?>


Hopefully these examples help. If not, please leave your comments with your questions or suggestions.


## Old comments

Regarding crashing - I suppose you are not seeing any error messages. In that case make sure you have

    ini_set('display_errors', 1);
    ini_set('display_startup_errors', 1);
    error_reporting(E_ALL);



Try adding this function to the non-static version (this has been done):

    /*
      * Get the average time of execution from all queue entries
      *
      * @return float
      */
  
    public function getAverage($format = self::SECONDS)
    {
        $count = count($this->_queue);
        $sec = 0;
        $usec = $this->get(self::MICROSECONDS);
  
        if ($usec > self::USECDIV) {
         // move the full second microseconds to the seconds' part
         $sec += (int) floor($usec / self::USECDIV);
  
         // keep only the microseconds that are over the self::USECDIV
         $usec = $usec % self::USECDIV;
        }
  
        switch ($format) {
         case self::MICROSECONDS:
          $value = ($sec * self::USECDIV) + $usec;
          return round($value / $count, 2);
  
         case self::MILLISECONDS:
          $value = ($sec * 1000) + (int) round($usec / 1000, 0);
          return round($value / $count, 2);
  
         case self::SECONDS:
         default:
          $value = (float) $sec + (float) ($usec / self::USECDIV);
          return round($value / $count, 2);
        }
    }
