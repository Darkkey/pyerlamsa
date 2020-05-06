# pyerlamsa
pyerlamsa -- python client for an erlamsa "`<<smart>>` dumb fuzzer" 

## TL;DR

Connecting to local erlamsa instance:
```
$ pip3 install pyerlamsa
$ python3
...
>>> import pyerlamsa
>>> e = pyerlamsa.Erlamsa('http://127.0.0.1')
>>> e.call('Hello erlamsa!');
(True, 'Hell4amsa15i685\x81\x91')
```

Connecting to erlamsa cloud:
```
...
>>> e = pyerlamsa.Erlamsa('https://erlamsa.online', token="yourtoken")
>>> e.call('Hello erlamsa!');
(True, '%n erlamsa!')
>>> e.call('Hello erlamsa!');
(True, 'lamsa!lo\x1ferHel')
>>> e.call('Hello erlamsa!');
(True, 'Hello erlamsa!Hello erlamsa!')
```

## Short API reference

*class* **Erlamsa**(uri, token = None, timeout = 5*)
    Creating *Erlamsa class* instance to connect to the instance at *uri* with *token*. In case instance is not require authentication to process fuzzing requests token could be omitted. *timeout* specifices the amount of seconds that **call(..)** function will wait before returning a timeout. Note, that, depending on an amount of input data, fuzzing could take from milliseconds to minutes, so choose this number wisely. Usually, erlamsa maximum fuzzing time is 60-90 seconds, however it will take less than second for small amounts of data.

**call**(*data, seed = None, mutations = None, patterns = None, blockscale = None*):
    Calls to the remote fuzzing instance. Returns tuple ia form of `True`,*fuzzed_data* in case of success, `False`,*data* in case of failure. Generates **ErlamsaNotAuthorized** exception in case of invalid token (or attempt to connect to the instance that requires token without suppliying one). Generates **ErlamsaInvalidParams** exception if any of additional supplied paramaters are invalid. Not every call to **call(..)** always returns `True` along with fuzzed data, because sometimes fuzzing may fail to produce anything meaningful in a given time. 
    
    Additional parameters are strings in a form that they are passed to erlamsa commandline (refer to excerpt from `erlamsa --help` and `erlamsa --list` output below or to offical erlamsa github https://github.com/Darkkey/erlamsa for more info). Additional parameters should be use only if want to narrow / direct the fuzzing process. In most cases there is no need to specify them.

## erlamsa mutations and parameters

*blockscale* -- float, increase/decrease default min (256 bytes) fuzzed blocksize multiplier [default: 1.0]

*mutations* -- list, which mutations to use with priorities [default:                      `nil=0,zip,uri,b64=2,len=2,fo=2,fn,ft=2,lrs,lis,lp,ls,lr,lri,lr2,lds,ld,srnd,snand,sd,sr,sp,br,ber,bi,bf,bed,bei,bd,ts2=2,tr=2,ts1=2,num=3,td,tr2,ad,ab,ui=2,uw,js=2,sgm=10`]. Note, that complex mutations (e.g. json (`js`) and SGML (`sgm`) could reuse other mutations on internal elements). Default priority is 1.

*patterns*  -- list, which mutation patterns to use with priorities [default:                  `nu,cp,ar,cs,sz,sk,bu,nd,od`]. Note, that complex patterns (e.g. size field (`sz`) could reuse other mutations on internal elements)

Mutations and patterns short description (output from `erlamsa --list`):
```
Mutations (-m)
    ab : enhance silly issues in ASCII string data handling
    ad : play with delimeters in ASCII string data
    b64: try mutate base64-encoded block
    bd : drop a byte
    bed: decrement a byte by one
    bei: increment a byte by one
    ber: swap a byte with random one
    bf : flip one bit
    bi : insert a byte
    br : repeat a byte
    fn : likely clone data between similar positions
    fo : fuse previously seen data elsewhere
    ft : jump to a similar position in block
    js : JSON tree mutations
    ld : delete a line
    lds: delete many lines
    len: predicted length mutation
    lis: insert a line from elsewhere
    lp : swap order of lines
    lr : repeat a line
    lr2: duplicate a line
    lri: copy a line closeby
    lrs: replace a line with one from elsewhere
    ls : swap two lines
    nil: no mutation will occur (debugging purposes)
    num: try to modify a textual number
    sd : delete a sequence of bytes
    sgm: SGML tree mutations
    sna: NAND/OR/XOR random bytes from block with 2^random values
    sp : permute a sequence of bytes
    sr : repeat a sequence of bytes
    srn: replace random bytes from block with random values
    td : delete a node
    tr : repeat a path of the parse tree
    tr2: duplicate a node
    ts1: swap one node with another one
    ts2: swap two nodes pairwise
    ui : insert funny unicode
    uri: try mutate URI to cause SSRF
    uw : try to make a code point too wide
    zip: ZIP path traversal

Patterns (-p)
    od : Mutate once pattern
    nd : Mutate possibly many times
    bu : Make several mutations closeby once
    sk : Skip random sized block and mutate rest
    sz : Try to find sizer and mutate enclosed data
    cs : Try to find control sum field and mutate enclosed data
    ar : Check whether data is an container (ZIP archive) and mutate enclosed files
    cp : Check whether data compressed, decompress and mutate
    nu : Pattern that calls no mutations
```