% VPS Comparison between Slicehost and Prgmr
% 2009-05-02

For the last year I've been using a 256MB
<abbr title="Virtual private server">VPS</abbr>
from [Slicehost][sli] to host a low
traffic site running Django. I've also used the slice as a development
platform thanks to screen and vim.

Two weeks ago I came across a mention of another [Xen][xen] based VPS provider
called [Prgmr][prg]. What intrigued me was that I could get a 1GB instance
for the $20/month I was paying Slicehost for a 256MB slice. 4 times the
memory, more than double disk capacity, and 60% more bandwidth. Was this too
good to be true?

I decided to rent a 256MB instance from Prgmr (sets you back $8/month) and
compare it against my trusty old slice with the same memory configuration.

Lets first take a look at the system specifications. Slicehost first:

    # uname -nrmo
    slicehost.uggedal.com 2.6.24-23-xen x86_64 GNU/Linux

    # grep "MemTotal" /proc/meminfo 
    MemTotal:       262316 kB

    # grep -m 4 -e "model name" -e "MHz" -e "cache size" -e "bogomips" /proc/cpuinfo
    model name      : Dual-Core AMD Opteron(tm) Processor 2212
    cpu MHz         : 2010.300
    cache size      : 1024 KB
    bogomips        : 4026.86

    # grep "processor" /proc/cpuinfo | wc -l
    4

And the same output from Prgmr: 

    # uname -nrmo
    prgmr.uggedal.com 2.6.26-1-xen-amd64 x86_64 GNU/Linux

    # grep "MemTotal" /proc/meminfo 
    MemTotal:       262360 kB

    # grep -e "model name" -e "MHz" -e "cache size" -e "bogomips" /proc/cpuinfo
    model name      : Quad-Core AMD Opteron(tm) Processor 2347 HE
    cpu MHz         : 1909.787
    cache size      : 512 KB
    bogomips        : 3826.33

    # grep "processor" /proc/cpuinfo | wc -l
    1

As we can see they are both Running Debian GNU/Linux 5.0. As of this writing
Slicehost gives you a 2.6.24 kernel while Prgmr provides a more recent
2.6.26 kernel. Both systems use the x86_64 architecture. Slicehost uses
a dual core Opteron with 1MB L2 cache for each core while Prgmr uses a
quad core Opteron with half the L2 cache (512KB) for each core. This
difference is uninteresting as our individual performance is dependant
on how many of these CPUs the system has in total compared to how many
VPS instances the system hosts. The output from `/proc/cpuinfo` only tells us
about the amount of cores available to our VPS instance, not how many there
are in total.

In reality one VPS instance does not get access to these cores
directly. One are given one or more virtual CPUs (VCPUs). Slicehost gives
us 4 VCPUs while Prgmr gives us 1 VCPU. How does this relate to
performance? Truly 4 is better than 1? According to [Luke Crawford][luk]
(from Prgmr):

> Giving you more VCPUs increases your peak performance (that is, you get
more CPU when nobody else wants it)  but it decreases your worst-case
performance (that is, when everyone is fighting over the CPUs, the
more VCPUs you have, the more context switches you have)  and so I've
chosen better worst-case performance here.

This means that theoretically Prgmr should give you more stable performance
and Slicehost should be more performant, except for when
your VPS neighbors get really busy.

### Django test suite

Since most of my work on my VPS involves working with Django, I decided to
benchmark the two providers by timing their execution of the Django test
suite.

Unneeded services were disabled and the instances was rebooted before
starting the tests.
All tests were run against revision 10108 of the Django trunk, using
the `sqlite3` database engine. The following script was used for running
the tests:

    for i in {1..20}
    do
      time ./runtests.py --settings=sqlite3conf
      sleep $((60*5))
      if [ $i -eq 10 ]
      then
        sleep $((60*60*12))
      fi
    done

The test suite is executed 10 times with pauses of 5 minutes in between,
then we wait 12 hours before running the tests 10 more times.
Note that this approach is highly unscientific. One can expect increased
performance when none of the neighboring instances are doing any work --
since these VPS providers use [credit based scheduling][cre].
A better approach would therefore be to run the tests once per hour for a
week. Sadly I can't afford to strip down my VPS for a week of testing.
You should therefore take these results with a grain of salt.

Lets first look at how the test suite run times varies over time:

<figure>
  <img src='http://chart.apis.google.com/chart?cht=lc&chs=550x150&chd=e:rAqhqHoqqGp9oNrFvJqBxw0DvEyGstvdvGxluMt2,vjmtvHywuRoF3EwRvputGdGYHVHJHCIHIeHEHHJ9&chdl=Slicehost|Prgmr&chco=edc240,afd8f8&chxt=y,x&chxl=0:|225s|245s|265s|285s|305s|325s|1:|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|16|17|18|19|20'>
</figure>

We see that the 10 first test runs if fairly equal for the two providers.
Prgmr is generally a bit slower and tend to vary more in execution time than
Slicehost. But after a 12 hour sleep something strange happens. Prgmr
becomes more stable and significantly faster than Slicehost. My take is
that we suddenly got the CPU all to our self on Prgmr. These results
indicate that we should perform these benchmarks over a longer time period.

With the skewed metrics in mind, lets take a look at the mean execution
time for these two services:

<figure>
  <img src='http://chart.apis.google.com/chart?cht=bhg&chs=550x80&chd=s:9,2&chdl=Slicehost|Prgmr&chco=edc240,afd8f8&chxt=x&chxl=0:|0s|59s|118s|177s|236s'>
</figure>

Prgmr "wins" thanks to its sprint after the 12 hour pause. The differences
is not extraordinary large though. But what if we adjust the run times
according to price? We multiply the Prgmr runtimes with 8/20 to find the
theoretical performance we could get if we spent as much money as we
did on the Slicehost rental:

<figure>
  <img src='http://chart.apis.google.com/chart?cht=bhg&chs=550x80&chd=s:9,W&chdl=Slicehost|Prgmr&chco=edc240,afd8f8&chxt=x&chxl=0:|0s|59s|118s|177s|236s'>
</figure>

With price in mind, Prgmr gives us almost 3 times more performance than
Slicehost. Note that this is just a theoretical exercise.
The 1GB Prgmr offering should be benchmarked against the 256MB
Slicehost offering for finding what you really get for your $20.

### Network

Average ping time was measured using [Just Ping][pin]
to find the network latency from several locations around the globe:

<figure>
  <img src='http://chart.apis.google.com/chart?cht=bhg&chs=550x500&chd=e:HnBGGzHGJn..QJSnYeb8pEjSn-hUh4,AeJNO3NUD77IY7bgf8l9pmczdyRfb3&chdl=Slicehost|Prgmr&chco=edc240,afd8f8&chxt=x,y&chxl=0:|0ms|77ms|154ms|231ms|308ms|1:|Sydney%2C%20Australia|Nagano%2C%20Japan|Singapore%2C%20Singapore|Hong%20Kong%2C%20China|Mumbai%2C%20India|Moscow%2C%20Russia|Oslo%2C%20Norway|Amsterdam%2C%20Netherlands|London%2C%20UK|Johannesburg%2C%20South%20Africa|Vancouver%2C%20Canada|New%20York%2C%20US|Florida%2C%20US|Chicago%2C%20US|San%20Francisco%2C%20US&chbh=a,3,12'>
</figure>

The results shows that Slicehost has the lowest latency from the eastern
parts of the US and Europe. Prgmr on the other hand has lower latency from
the western parts of the US, Asia, and Australia. This is largely due to
the geographical locations of the two providers.
Slicehost is placed in Saint Louis, Missouri and Prgmr is located in
San Jose, California.

### Stability

I've been a Slicehost customer for some time and have never had to restart my
slice or had it gone down due to system failure. Before updating to Debian 5.0
and changing kernels it had an uptime of over [330 days][upt].

As I'm a brand new Prgrm customer I don't have any data to compare the two
services.

### Community

Slicehost has an active community with their forums, IRC, and Wiki. In
addition they have several articles describing how to setup various types
of systems.

Prgmr on the other hand has none of these community building platforms
(except for a fairly empty Wiki). If you want to use Prgmr you are
pretty much on your own. This is highlighted on their web page with
the phrase:

> We don't assume you are stupid.

### Support

Prgmr's tagline is also apparent in their support software. While Slicehost
provides a flashed out control panel (SliceManager) for monitoring and
administering your
VPS, Prgmr only provides console access through a SSH gateway. Much of
what's automated over at Slicehost (like provisioning a new instance) is
handled manually at Prgmr.

I've only had one encounter with the Slicehost support team. Upgrading a
kernel took about one hour, from when I sent a support request to
the VPS was rebooted with the new kernel. The only encounter with the
support staff at Prgmr I've had was when I ordered my instance. That
process took 3 hours, from placing the order to being able to log in.

### Conclusion

We've seen that it's hard to benchmark VPS instances as factors out
of our control largely influences the results. All in all I think
the two offerings is on par performance wise, but more testing
needs to be done to see how the load of other customers influences
our performance.

I rarely use the SliceManager, and so far Prgmr's console have served
me well. Slicehost's network latency is a bit lower for me here in
Norway, but not large enough to exclude Prgmr as a worthy contender.

My only concern is the stability of Prgmr, as my Slicehost experience
has been stellar. I think I'll keep both VPSs for the time being
and switch to using solely Prgmr if my long term experience proves
to be as good as what I've seen so far.

[sli]: http://slicehost.com/
[xen]: http://xen.org/
[prg]: http://prgmr.com/xen/
[luk]: http://prgmr.com/~lsc/
[cre]: http://wiki.xensource.com/xenwiki/CreditScheduler
[unb]: http://www.hermit.org/Linux/Benchmarking/
[pin]: http://just-ping.com
[upt]: http://twitter.com/uggedal/status/1647025940
