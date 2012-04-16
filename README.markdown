cd.js - a method-chaining library for the HTML5 file system API

(c) 2012 Zoran Obradović, Ljudmila.org. GPL 3.0 applies.

##EXAMPLES

###to write a file: 

      cd('foo/bar/myFiles')
      .write('myText.txt','My foobar is great.');

###to read a file and then write to another file: 

      cd('/foo')
      .read('bar.txt', function(done,res) { 
        this
        .echo(res)
        .write('report.txt', 'bar.txt contains '+res 
      })

###to list a directory: 

      cd('/foo')
      .for('*',function(done,file) { 
        this.echo(file.fullPath) 
      });

###to do lots of stuff:

      var store = {}, total = 0, report = '';
      
      cd('/foo/bar')
      .rmrf('backup/*')
      .cp('*.txt','backup')
      .echo('backup done')
      .cd('../baz')
      .echo('creating report')
      .for('*.lst', function(done,file) {
        this.read(file, function (done,res) {
          var numberOfAs = res.replace(/[^aA]/g,'').length;
          store[file.name] = numberOfAs;
          total += numberOfAs;
        })
      })
      .echo('writing report')
      .then(function() {
        var report = 'number of As in /foo/baz/:\n';
        for (var i in store) report+= i+': '+store[i] + '\n';
        report += "total: '+total;
        this.write('/var/report.txt', report);
      })
      .echo('all done');

##CD Object interface:
### constructor
####`cd(path)`
start a new fs method queue, change the current directory to cd

### PROPERTIES
#### `.cwd`
change the current directory

### GENERAL
####`.cd(path)`
  change current directory
####`.up()`
  cd ..
####`.root()`
  cd /
####`.echo(args)`
  output args for debugging purposes. uses console.log by default, can be overriden at CD.echo.
####`.pwd()`
  echo the current directory
####`.onerror(callback)`
  define an error handler
  

###QUEUE
####`.then(cb,arg)` 
  add a callback to the queue, with an optional argument
  
###DIRECTORY ITERATORS
####`.for(glob,cbFound,cbNotFound)`
  call cbFound for each matching file entry, or cbNotFound if no match found
####`.ls(glob,cbFound,cbNotFound)`
  like for, but includes metadata (TODO: figure out if size can be added without actually reading the file, add URL)
####`.byType(glob,cbFile,cbDir,cbNotFound)`
  like for, but allows separate callbacks
####`.files(glob,cbFound,cbNotFound)`
  like for, but only include files
####`.directories(glob,cbFound,cbNotFound)` 
  like for, but only include directories


###FILE OPERATIONS
####`.rm(glob,cbNotFound)`
  remove matching files
####`.mv(glob,dest,cbNotFound)`
  move matching entries to the destination directory, or move a single entry to the destination path, or throw an error if neither is possible
####`.cp(glob,dest,cbNotFound)`
  copy matching entries to the destination directory, or copy a single entry to the destination path, or throw an error
####`.rmrf(glob,cbNotFound)`
  remove matching files and recursively remove matching directories
####`.mkdir(path)`
  recursively create a directory`
  
###READ/WRITE OPERATIONS
####`.read(glob,cbFound,cbNotFound)`
  read content of matching files
####`.write(glob,content)`
  write to matching files
  content can be a string, a blob, TODO: file, image, callback
####`TODO .append(glob,content)`
  append to matching files
  content can be a string, a blob, file, image, callback
####`TODO .truncate(glob,offset,content)`
  truncate matching files, and append optional content
  offset can be a number or a callback
  content can be a string, a blob, file, image, callback
####`TODO .writeAt(glob,offset,content)`
  offset can be a number or a callback
  content can be a string, a blob, file, image, callback

 

##OPTIONS
(must be defined _before_ including cd.js)

    cdOptions = {
      required: size in megabytes,
      dont: if true, the file system won't be requested automatically, it can be done later by calling CD.init()
    }

##ARGUMENT TYPES
###`callback`

    function cb (done,arg) {...}

####`done`
if it returns `true`, then the callback is responsible for advancing the queue by calling done()

TODO: if it returns false, the error handler is called and the queue cancelled (or not, see error handling)
####`arg`
Additional argument, if supplied. In `.for()` loops, this will be the `Entry` object, with the following 
added properties:

- `.list` - the array of all matched file entries
- `.index` - the index of the current file entry
- `.single` - whether this file was the only one found
    
TODO: allow multiple arguments
    

The callback's context (i.e. `this`) will be the same CD object. any methods called on it will be placed in the queue and executed after the callback exits exits. if the callback needs to make any async calls, it can return `true` and call done() manually when the async calls are complete.

If the callback is a non-truthy value, it is ignored.
  
TODO: if the callback is not a function, an error is thrown
  

###`glob`
EITHER:

* an absolute or a relative path from the current directory, possibly including wildcards (* and ?) in the last part (after the last slash), OR
* an entry object, OR
* an array of entry objects, OR
* TODO an array of paths
  
Will throw an error if the directory (the path before the last slash) does not exist, or if the glob is misformed.

