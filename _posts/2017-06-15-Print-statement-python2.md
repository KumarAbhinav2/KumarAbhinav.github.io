Print converts objects to strings and then writes them to file object (BY default its standard file object stdout). It keeps a flag too called file.softspace() to file object. This flag let interpreter know when to write the space. When you print something it will first checks the flag and will write space if it is set.
```
import sys
print sys.stdout.softspace
print "Hello",
print sys.stdout.softspace
```

