LinkedList NYC Program
----------------------

inside this file is a webserver that will output the text "LinkedList NYC" on
network port 31337 via HTTP.

killin' it
------------

executes with ruby 1.9.3

`ruby -x ./readme.markdown` to run the server,
do a `killall ruby` to stop the server

`ruby run.rb` to run the server once, output with curl, then stop.

List Output
------

in your terminal run `curl localhost:31337` to get the output,
something resembling "LinkedList NYC" should output, unless there's some
tragic error.

```ruby

#!/usr/bin/env ruby -w

# This script is a simple ruby web server that
# serves the file it is inside.

require 'socket'

Process.daemon

server = TCPServer.new(31337)
DATA.rewind
data = DATA.read

# Pull out first letters
parts = data.split('```')
first = parts.first.split("\n").map{|l| l[0] }

# Pluck out NYC
last = ""
lines = parts.last.split("\n")
lines.each_with_index do |l, i|
  last += ( l[0] || "\n") if lines.fetch(i+1, '')[0] == '-'
end

# Finish output
out = first.join.gsub(/[^\w]/, '') + " " + last

# Serve http
loop do
  Thread.start(server.accept) do |client|
    lines = []
    while line = client.gets and line !~ /^\s*$/
      lines << line.chomp
    end
    client.puts <<-MESSAGE.gsub("\n", "\r\n")
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: #{out.bytesize+2}
Connection: close

#{out}
    MESSAGE
    client.close
    Thread.exit
  end
end

__END__

```

Now how does it work?
------------

The script there in the middle is a web server. Ruby can run scripts embedded
inside of text files. I have no idea why, but the bits between the shebang
and the `__END__` block become executable with the -x flag sent to ruby.

Once the script starts running, everything after `__END__` becomes available as
as the constant DATA, which allows rewinding to the beginning of the file so
that you can see everything before the `__END__` as well.

After that, it's a simple matter of plucking out the message from the rest
of the DATA block and outputting it with http headers.


Yes, but is it web scale?
---------

    ab -c 10 -n 1000 http://localhost:31337/

    Requests per second:    3095.75 [#/sec] (mean)
    Time per request:       3.230 [ms] (mean)
    Time per request:       0.323 [ms] (mean, across all concurrent requests)


Credits
-------

This script brought to you by Hurricane Sandy.

It uses quite a few tricks learned from
<https://speakerdeck.com/jeg2/10-things-you-didnt-know-ruby-could-do>

