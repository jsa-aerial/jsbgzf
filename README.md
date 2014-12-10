jsbgzf
======

Javascript bgzf inflation and deflation


Include the following libs:

* inflate.js (fetch and place or fetch remotely)
* pako_deflate.min.js (fetch and place or fetch remotely)
* jsbgzf.js (this file)

API consists of top level functions - no constructor needed or
wanted.

Synopsis:


* `buffer2String` - take a unsigned byte array (UBA) and convert to a
  string. Only works for iso-latin-1 (or ascii - 8 bit bytes) - no
  UTF-8 (unicode, ...) here

* `getChunk` - low level function to obtain a chunk of a file as a UBA

* `getBGZFHD` - parses and returns as a map bgzf headers (each
  compressed block has a bgzf header)

* `nextBlockOffset` - from a provided legal compressed block offset,
  obtain the offset of the next compressed block

* `blockSize` - from a provided legal compressed block offset,
  compute size of contained compressed block

* `countBlocks` - counts the total number of bgzf (compressed) blocks
  in file

* `inflateBlock` - from a provided legal compressed block offset,
  inflate the blcck to its UBA representation.

* `inflateBlock2stg` - same as inflateBlock but then use
  buffer2String to convert inflated block to a string

* `inflateRegion` - from a provided starting compressed block offset
  and some ending offset (need not be a block offset), expand all
  blocks covereed by region to a single UBA

* `inflateAllBlocks` - inflate all blocks in file to a single UBA.
  Likely not usable for large files.

* `inflateRegion2Stg` - same as inflateBlock2stg, but for
  inflateRegion

* `inflateAll2Stg` - same as inflateRegioin2Stg, but where region is
  the entire file

* `bgzf` - takes a UBA of data and deflates to a bgzf compressed
  block

Appending buffers - used internally, but are intended for public
use as well

* `appendBuffer` - append two unsigned byte arrays (UBA)
* `appendBuffers` - append vector of UBAs


---

Main functions for processing BGZF inflation and deflation:


---

Inflation:


```javascript
function getChunk (f, beg, end, cbfn)
```
Low level binary file reader.  Reads bytes from base offset BEG to
END inclusive as an array of unsigned bytes using a new FileReader
for each read.  CBFN is the callback to call when read is finished
and it is passed the FileReader object.


```javascript
function getBGZFHD (f, offset, cbfn)
```
Low level function that obtains the BGZF header for the BGZF
compressed file F at base byte offset OFFSET.  Decodes the header
and passes the resulting JS object, representing the header
information with fields as defined by template bgzf_hd_fmt, to
CBFN.


```javascript
function nextBlockOffset (f, offset, cbfn)
```
Low level function that given BGZF file F, base offset OFFSET,
obtains the offset of the next block and passes to CBFN


```javascript
function blockSize (f, offset, cbfn)
```
Low level function that given BGZF file F, base offset OFFSET,
obtains the block size of block at OFFSET and passes to CBFN


```javascript
function countBlocks (f, cbfn)
```
Low level function that given BGZF file F, obtains the total count
of _gzip_ blocks in F.  Each of these will correspond to one of
BGZF's 64KB uncompressed blocks.  NOTE: a chunk or interval may
contain more than one of these blocks! Passes count to CBFN.

WARNING: for large BGZF files this can take a looonnnggggg time.


```javascript
function inflateBlock(f, blockOffset, cbfn)
```
Low level function that given BGZF file F, base off BLOCKOFFSET,
inflates the single _gzip_ compressed block at that location and
passes the base array buffer obtained to CBFN.  NOTE: this uses the
JSZlib library.


```javascript
function inflateBlock2stg(f, blockOffset, cbfn)
```
Low level function that given BGZF file F, base offset BLOCKOFFSET,
inflates the single _gzip_ compressed block at that location,
converts the array buffer so obtained to a string (latin-1) and
passes that to CBFN


```javascript
function inflateRegion (f, begOffset, endOffset, cbfn)
```
Mid level function that given a BGZF file F, a region defined by
offsets BEGOFFSET and ENDOFFSET, fetches, inflates and appends all
(_inclusively_) the _gzip_ blocks in the region into a single array
buffer and passes to CBFN as its first argument, and passes the size
(in bytes) of the last inflated block in the region as second
argument.


```javascript
function inflateAllBlocks(f, cbfn)
```
Mid level function that given a BGZF file F, inflates all the
contained _gzip blocks, appends them all together into a single
array buffer and passes that to CBFN.  Calling this on any 'large'
BGZF _data_ file (bai should be fine) will likely blow up with
memory exceeded.


```javascript
function inflateRegion2Stg (f, begOffset, endOffset, cbfn)
```
Mid level function that given a BGZF file F, a region defined by
offsets BEGOFFSET and ENDOFFSET, fetches, inflates, appends
together and converts to a string all the gzip blocks in region.
Passes the string to CBFN


```javascript
function inflateAll2Stg (f, cbfn)
```
Mid level function.  Inflates the entire BGZF file F, converts to a
total string and passes to CBFN.  Calling this on any 'large' BGZF
_data_ file will likely blow off with memory exceeded.


---

Deflation:


```javascript
function bgzf (uba)
```
BGZF deflation; takes an unsigned byte array UBA, which is taken as
not yet deflated (though you could try deflating an already
deflated block, but this will likely lose) and deflate it to a
legal BGZF formatted compressed uba (including BSIZE payload).


```javascript
function addEOFblk (bgzfUba)
```
Takes a uba composed of a series of bgzf blocks and appends the EOF
block.


---

Utility functions:


```javascript
function appendBuffer( buff1, buff2, asUint8)
```
Take two array buffers BUFF1 and BUFF2 and, treating them as simple
byte arrays, return a new byte array of their catenation.  If asUint8
(boolean) is true, return the uint8 'view' array otherwise return the
underlying ArrayBuffer.


```javascript
function appendBuffers(bufferVec, asUint8)
```
Take a vector of array buffers and treating them as simple byte
arrays, return a new byte array of their catenation.  If asUint8
(boolean) is true, return the uint8 'view' array otherwise return
the underlying ArrayBuffer.


```javascript
function buffer2String (resultBuffer)
```
Take an array buffer considered as a byte stream, and return the
string representation of the buffer.  This works only on latin 1
character encodings (no UTF8).
