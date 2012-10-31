LinkedList NYC Program
----------------------

inside this file is a webserver that will output the text "LinkedList NYC" on
network port 31337.

killin' it
------------

executes with ruby 1.9.3

`ruby -x ./readme.markdown` to run,
do a `killall ruby` to stop it.

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
out = DATA.read
splits = out.split('```')
first = splits.first.split("\n").map{|l| l[0] }
last = ""
lines = splits.last.split("\n")
lines.each_with_index{|l, i| last += ( l[0] || "\n") if lines.fetch(i+1, '')[0] == '-' }
out = first.join.gsub(/[^\w]/, '') + " " + last

loop do
  Thread.start(server.accept) do |client|
    lines = []
    while line = client.gets and line !~ /^\s*$/
      lines << line.chomp
    end
    headers = [
      "HTTP/1.1 200 OK",
      "Content-Type: text/plain",
      "Content-Length: #{out.bytesize}",
      "Connection: close\r\n\r\n"
    ].join("\r\n")
    client.puts headers
    client.puts out
    client.close
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


You Like?
---------

This script brought to you by Hurricane Sandy.


Credits
-------

This script uses quite a few tricks learned from
<https://speakerdeck.com/jeg2/10-things-you-didnt-know-ruby-could-do>


---
