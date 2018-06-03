# RO4R 2.0 - Remote Objects for Ruby 2.4+

## Supported Ruby versions

Supports all Ruby versions from 2.4 above, in all combinations of client and server versions.

For support on older versions, please see below under "Other Ruby versions".

## How to see it work

```
- Check out the files:

  git clone git://github.com/docelic/RO4R.git


- In one terminal, start an example server (it exports one Ruby Hash
  variable and provides main loop):

  cd RO4R
  ruby -I. examples/srv.rb


- In another terminal, start an example client (it attaches to the
  shared Hash, iterates a counter 10000 times, and then prints the
  benchmark statistics and the contents of the Hash):

  cd RO4R
  ruby -I. examples/cli.rb
```

NOTE: when the client program does its job and disconnects, on the server
side you will see a message you can ignore:

```
      #<NoMethodError: undefined method `length' for nil:NilClass>)
```

### Benchmarks explanation

When you run the above, it will print statistics such as:

```
    0.390000   0.390000   0.780000 (  1.451144)
    {:counter=>9999, 1=>A, 2=>#<A:0xb7741e38>}
```

The first line is the benchmark output, showing user, system, total
and real times. (That's benchmark for the 10,000 iterations that
the example client does).

Second line are the contents of the shared Hash object. In it you see
the :counter that was created on the client side, and two keys that
were initialized by the example server.

### Modifying/querying shared object

You can open another terminal and run the example client under `irb -r`,
which will do its thing and leave you with an IRB shell, from which
you can query and modify $r yourself:

```
$ cd RO4R
$ irb -I. -r examples/cli.rb

> $r[:counter]
9999
> $r[:test_value]= 717
#-> 717

# On the server side, we've initialized an example method 'm' that returns
# value of 1, and that you can run and test:

> $r[2].m
#-> 1
```
And you can also open a new terminal in which you can query all we've done
so far, since the object persists in the process:

```
$ cd RO4R
$ irb -I. -r examples/cli.rb

> $r
{:counter=>9999, 1=>A, 2=>#<A:0xb7745e34>, :test_value=>717}
```

## Benchmarks

Benchmarks for 10,000 RPC invocations:

```
Date        Server   Client     Stats
Oct 2015:   2.6.0p2   2.6.0p2   0.192000   0.100000   0.292000 (  0.517264)
Jun 2016:   2.2.3     2.2.3     0.196000   0.076000   0.272000 (  0.481286)

Oct 2015 setup: i7-4790K CPU @ 4.00GHz on Linux 4.2.0-040200-lowlatency
Jun 2018 setup: i7-4790K CPU @ 4.00GHz on Linux 3.16.0-4-amd64

Older Benchmarks from 2008:

            Server   Client     Stats
            1.9.2     1.9.2     0.390000   0.390000   0.780000 (  1.451144)
            1.9.2     1.8.7     1.000000   0.280000   1.280000 (  2.009202)
            1.8.7     1.9.2     0.520000   0.300000   0.820000 (  1.914454)
            1.8.7     1.8.7     0.960000   0.260000   1.220000 (  2.472942)
```

## Programming notes

### Shared classes

Generally, you will share instances of classes that are known to both the server and the clients.

But it is also possible to share objects that clients know nothing about.

Included in the distribution is rmarshal/, a modification of
Ruby marshaller that you need to compile and 'require' into
your Ruby app on the server side (the one that exports the object).
(The rmarshal.c is currently out of date, and the version included works with Ruby 1.8).

Do that and the clients will transparently be able to use the remote
object even though they don't know its Class.

### Method return values

When writing methods, you usually don't care about unused return
values because Ruby simply discards them. On methods that will be
invoked remotely, RO4R will pass the return value back to the client,
so pay attention to exit a method with an explicit 'nil' if its return
value is not needed. This is cleaner and can also save you from errors
if the (unnecessary) return value would be a weird object that Ruby
marshaller can't serialize and pass back to the client.

### Code stability

RO4R has been in production use with Ruby 1.8, 1.9, and later since 2007.

## Other Ruby versions

For use with lower versions (Ruby 1.8 to Ruby 2.3) , please either use RO4R release 1.0.0
or edit RO4R.rb to replace "safe= 1" with "safe= 3" and uncomment the line saying
"Bignum, Fixnum".

I believe the C marshaller is operational only for Ruby 1.8 versions.

## TODO

* Add rmarshall for latest Ruby