---
title: AsisCTF 2018 - Tokyo
layout: post
date: '2018-05-08 15:53:00'
description: Writeup of Tokyo
image: '/assets/images/logo.png'
tags:
- ctf
- writeup
author: Kauê Doretto
---
>Tokyo
>Category: Forensics
>
>Description:
>    From [Tokyo]({{ site.url }}/assets/files/tokyo_46d159840d7d29f58256746814b2f7dd4fcb8682966b679bb27b9bc4c8afe07e) to Tehran with strange packs! Shake It Up 😜
>
>    Hint 1: Kyoto Cabinet


(This is a very thorough write-up, if you're not interested in the details just jump to the source code at the end)

After extracting the given archive, a extensionless file named `tokyo` was given. The first approach was to find which kind of file this is:

```
$ file tokyo
tokyo: data
```

But what kind of data is this?

```
$ xxd tokyo | head
00000000: 4b43 0a00 100d 05bc 3003 0a00 0000 0000  KC......0.......
00000010: 0000 0000 0010 0007 0000 0000 0000 0000  ................
00000020: 0000 0000 0000 0059 0000 0000 0060 20d0  .......Y.....` .
[...]
```

Searching for `4b 43 0a` or `KC` returned nothing worth so I started the file analysis approach. The file was split in three sections, which I named header, body and footer. The header was defined as the first 48 bytes (offset 00-47):

```
$ xxd -l 48 tokyo 
00000000: 4b43 0a00 100d 05bc 3003 0a00 0000 0000  KC......0.......
00000010: 0000 0000 0010 0007 0000 0000 0000 0000  ................
00000020: 0000 0000 0000 0059 0000 0000 0060 20d0  .......Y.....` .
```

The body was everything from offset 48 to 6297719 and consisted of long `00` values with some disperse groups of 3 bytes values, like this:

```
$ xxd -s 16924 -l 48 tokyo 
0000421c: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0000422c: 0000 0000 0000 000c 03f0 0000 0000 0000  ................
0000423c: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

Finally the footer, from 6297719 to 6299855:

```
$ xxd -s 6297719 tokyo 
00601877: 00cc 0400 0000 0000 0000 0000 0000 0003  ................
00601887: 0100 0000 21ee 0000 00cc 0400 0000 0000  ....!...........
00601897: 0000 0000 0000 0003 0100 0000 5fee 0000  ............_...
006018a7: 00cc 0400 0000 0000 0000 0000 0000 0003  ................
006018b7: 0100 0000 41ee 0000 00cc 0400 0000 0000  ....A...........
006018c7: 0000 0000 0000 0003 0100 0000 62ee 0000  ............b...
[...]
```

It's clear that the footer consists of 24 bytes groups, starting with `00CC` and with some printable ASCII chars:

```
$ xxd -s 6297719 -c 24 tokyo 
00601877: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 21ee 0000  ....................!...
0060188f: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 5fee 0000  ...................._...
006018a7: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 41ee 0000  ....................A...
006018bf: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 62ee 0000  ....................b...
006018d7: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 5fee 0000  ...................._...
006018ef: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 6eee 0000  ....................n...
00601907: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 69ee 0000  ....................i...
0060191f: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 21ee 0000  ....................!...
00601937: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 5fee 0000  ...................._...
0060194f: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 61ee 0000  ....................a...
[...]
0060206f: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 64ee 0000  ....................d...
00602087: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 72ee 0000  ....................r...
0060209f: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 75ee 0000  ....................u...
006020b7: 00cc 0400 0000 0000 0000 0000 0000 0003 0100 0000 61ee 0000  ....................a...
006020cf: 00
```

With `wc` we get 89 flag characters (90 lines minus the last one):

```
$ xxd -s 6297719 -c 24 tokyo | wc -l
90
```

By visually analyzing this section we notice the existence of chars that make for the event's flag format: `ASIS{}`, so we're dealing with the flag itself but in a somehow scrambled order. If it's scrambled then there is a way to sort it back to the original string. But how? Let's go back to the header part:

```
4b43 0a00 100d 05bc 3003 0a00 0000 0000 0000 0000 0010 0007 0000 0000 0000 0000 0000 0000 0000 0059 0000 0000 0060 20d0
```
```
100d 05bc 3003 0a = Very big decimal number, probably a group of values
10 00 07    = 1048583 
59       = 89 (matches the size of the string)
60 20 d0 = 6299856 (matches the file size)
```

We got two important informations but nothing worth when it comes the the structure of the file itself or related to the footer section. Now let's check the body again. We found out it has a lot of null bytes with some 3 bytes groups. We'll offset the file to the first group found, group by 3 and ignore null lines:

```
$ xxd -a -c 3 -s 16947 -g 3 tokyo  | head -n 30
00004233: 0c03f0  ...
00004236: 000000  ...
*
0004061d: 0c03e7  ...
00040620: 000000  ...
*
00041f0d: 0c03a8  ...
00041f10: 000000  ...
*
00047763: 0c0348  ..H
00047766: 000000  ...
*
0004a9fd: 0c0402  ...
0004aa00: 000000  ...
*
0004fe45: 0c0384  ...
0004fe48: 000000  ...
*
00051903: 0c03b1  ...
00051906: 000000  ...
*
0007368f: 0c039f  ...
00073692: 000000  ...
*
00073bab: 0c03ed  ...
00073bae: 000000  ...
*
000788df: 0c0390  ...
000788e2: 000000  ...
*
```

Seems that `0C 03` is a prefix, changed once to `0C 04`, which looks like a sequential value representation, like a counter. Let's extract only these bytes and sort them:

```
$ xxd -a -c 3 -s 16947 -g 3 tokyo | grep -v "\*" | grep -v " 000000  ..." | cut -d" " -f 2 | grep "^0c" | sort
0c030f
0c0312
0c0315
0c0318
[...]
0c0411
0c0414
0c0417
```

That gives us the following decimal values:

```
787215
787218
787221
787224
[...]
787473
787476
787479
```

These decimal values are all 3 units apart from each other, so it really looks like a sequential counter. But how "787215" turns into "6297720" (the first character's byte group offset)? Well, if the distance between the first and second groups is represented by an increment of 3 and the size of each group is a fixed 24 bytes size, then 24/3 = 8, giving us the amount of bytes until that offset. Let's test this theory by multiplying 787215 * 8 = 6297720. This is the offset of the first group of bytes, from the beggining of the file. But now we reached a dead end, as this only gives us the sequence in wich the groups are stored. If the chars are stored sequentially and the sorted offset values gives us the same sequence, then the order of the characters is somewhere else. We really need the file format definition to go forward.

This is where the hint was given: "Hint 1: Kyoto Cabinet"

Turns out that Kyoto Cabinet is a database library based on Tokyo Cabinet. It's documentation is available on these links:

[1] [http://fallabs.com/kyotocabinet/](http://fallabs.com/kyotocabinet/)

[2] [http://fallabs.com/kyotocabinet/kyotoproducts.pdf](http://fallabs.com/kyotocabinet/kyotoproducts.pdf)

[3] [http://fallabs.com/kyotocabinet/spex.html](http://fallabs.com/kyotocabinet/spex.html)

[4] [http://fallabs.com/kyotocabinet/command.html](http://fallabs.com/kyotocabinet/command.html)(plus APIs, sources and more...)

[5] [http://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/](http://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/)

[6] [https://github.com/cloudflare/kyotocabinet/](https://github.com/cloudflare/kyotocabinet/)

[7] [http://en.cppreference.com/w/cpp/types/integer](http://en.cppreference.com/w/cpp/types/integer)

Now it's possible to dig deeper into the file's structure. I will not go into a lot of details here but the first thing noticed was that the file's structure was set almost right. In fact the `.kch` files (a hash based database) have 4 sections: header, FreeBlock pool, bucket array and records. Let's get a hint on how the system works. First, we'll create an empty hash database file, copy it into a second file and add a record:

```
$ kchashmgr create init.kch
$ cp init.kch A.kch
$ kchashmgr set A.kch A V # V stands for "value"
$ cmp -l init.kch A.kch 
     40   0   1
     48 170 220
3486400   0  14
3486401   0   3
3486402   0  17
cmp: EOF on init.kch after byte 6297720
```

The insertion of the key `A` changed the initial database file's offset 3486399 (the `cmp` command's offset is counted from 1, not from 0). The bucket array and record were set like this:

```
$ xxd -s 3486399 -l 3 A.kch 
003532bf: 0c03 0f                                  ...

$ xxd -s 6297720 A.kch 
00601878: cc06 0000 0000 0000 0000 0000 0000 0101  ................
00601888: 4156 ee00 0000 0000                      AV......
```

Now let's do the same with the key "B" and compare results:

```
$ cp init.kch B.kch
$ kchashmgr set B.kch B V
$ cmp -l init.kch B.kch 
     40   0   1
     48 170 220
4708816   0  14
4708817   0   3
4708818   0  17
cmp: EOF on init.kch after byte 6297720
```

Ok, so the modified offset has changed to 4708815. The bucket array and record values are:

```
$ xxd -s 4708815 -l 3 B.kch 
0047d9cf: 0c03 0f                                  ...

$ xxd -s 6297720 B.kch 
00601878: cc06 0000 0000 0000 0000 0000 0000 0101  ................
00601888: 4256 ee00 0000 0000                      BV......
```

And lastly we compare the first records' format from the `tokyo`, `A.kch` and `B.kch`:

```
CC 04 00 00 00 00 00 00 00 00 00 00 00 00 03 01 00 00 00 21 EE 00 00 00 (21 = !)
CC 06 00 00 00 00 00 00 00 00 00 00 00 00 01 01 41 56 EE 00 00 00 00 00 (41 = A, 56 = V)
CC 06 00 00 00 00 00 00 00 00 00 00 00 00 01 01 42 56 EE 00 00 00 00 00 (42 = B, 56 = V)
```


From the [5] reference, we can imply that the key in the `tokyo` file is 3 bytes (or characters) long as for the other two files it's 1 byte long.

Some conclusions drawn from this are:
* The key and the value are stored together in the record;
* Both "A" and "B" keys confirm the first record offset value being `0C030F`;
* The offset changes for different keys;
* The place in the bucket array where the record's offset is stored changes along with the key;
* The key for `tokyo` is 3 bytes long.


Therefore, storing a key/value pair goes something like this:
> key -> magic stuff -> bucket position -> offset to record -> key/value

What is this magic stuff? According to the doc's "Specifications" page:
"The hash function used for hash table is MurMurHash 2.0"
  
We can hash values using MurMurHash using the `kcutilmgr hash` command:

```
echo -n "A" | kcutilmgr hash
f2f690ab5712b279
```

Unfortunately this looks nothing like '4708815' so what's going on here? Searching for it's source we can reach it's GitHub project repository [6]. After some digging I found that the `kchashdb.h` file had some very interesting code, from the header's format to functions that insert, read and search for keys, among others. Let's take a look at a special function: jump().

```C
/**
     * Jump the cursor to a record for forward scan.
     * @param kbuf the pointer to the key region.
     * @param ksiz the size of the key region.
     * @return true on success, or false on failure.
     */
    bool jump(const char* kbuf, size_t ksiz) {
      _assert_(kbuf && ksiz <= MEMMAXSIZ);
      ScopedRWLock lock(&db_->mlock_, true);
      if (db_->omode_ == 0) {
        db_->set_error(_KCCODELINE_, Error::INVALID, "not opened");
        return false;
      }
      off_ = 0;
      (1) uint64_t hash = db_->hash_record(kbuf, ksiz);
      uint32_t pivot = db_->fold_hash(hash);
      (2) int64_t bidx = hash % db_->bnum_;
      (3) int64_t off = db_->get_bucket(bidx);
      if (off < 0) return false;
      Record rec;
      [...]
```

There are the `key buffer` and `key size` parameters, a `hash` variable and a `get_bucket()` function, which is basically everything we need. Going into the nitty-gritty, using the "A/V" pair as the record:

> (1) uint64_t hash = db_->hash_record(kbuf, ksiz);

```
  /**
   * Get the hash value of a record.
   * @param kbuf the pointer to the key region.
   * @param ksiz the size of the key region.
   * @return the hash value.
   */
  uint64_t hash_record(const char* kbuf, size_t ksiz) {
    _assert_(kbuf && ksiz <= MEMMAXSIZ);
    return hashmurmur(kbuf, ksiz);
}
```

If you supply this function with the key and it's size it will return the key's MurMurHash. For the "A" key it is "f2f690ab5712b279" or, in decimal:

```
$ python -c 'print(int("f2f690ab5712b279", 16))'
17507339667024032377
```

Ok, next:

> (2) int64_t bidx = hash % db_->bnum_;

`bidx` is the bucket index parameter for the `get_bucket()` function, and it's calculated from `hash % db_->bnum_`. This `bnum_` variable is read from the file header:

```
  /** The offset of the bucket number. */
  static const int64_t MOFFBNUM = 16;
```
```
  MOFFBNUM        00 00 00 00 00 10 00 07 = 1048583
```
```
  bnum_(DEFBNUM) = 1048583
```
```
bidx = hash % db_->bnum_
bidx = 17507339667024032377 % 1048583
bidx = 580029
```

The last and more complex part depends on values previously either retrieved from the header or calculated from them.

> (3) int64_t off = db_->get_bucket(bidx);

```
if (!file_.read_fast(boff_ + bidx * width_, buf, width_)) {
```

This function will read an offset of the db file according to the calculation of `boff_ + bidx * width_`. Recovering these values from the `A.kch` file header and manually setting all the variables involved in the source code:

```
  /** The offset of the library version. */
  static const int64_t MOFFLIBVER = 4;
  /** The offset of the library revision. */
  static const int64_t MOFFLIBREV = 5;
  /** The offset of the format revision. */
  static const int64_t MOFFFMTVER = 6;
  /** The offset of the module checksum. */
  static const int64_t MOFFCHKSUM = 7;
  /** The offset of the database type. */
  static const int64_t MOFFTYPE = 8;
  /** The offset of the alignment power. */
  static const int64_t MOFFAPOW = 9;
  /** The offset of the free block pool power. */
  static const int64_t MOFFFPOW = 10;
  /** The offset of the options. */
  static const int64_t MOFFOPTS = 11;
  /** The offset of the bucket number. */
  static const int64_t MOFFBNUM = 16;
  /** The offset of the status flags. */
  static const int64_t MOFFFLAGS = 24;
  /** The offset of the record number. */
  static const int64_t MOFFCOUNT = 32;
  /** The offset of the file size. */
  static const int64_t MOFFSIZE = 40;
  /** The offset of the opaque data. */
  static const int64_t MOFFOPAQUE = 48;
  /** The size of the header. */
  static const int64_t HEADSIZ = 64;
  /** The width of the free block. */
  static const int32_t FBPWIDTH = 6;
  
  /** The default bucket number. */
  static const int64_t DEFBNUM = 1048583LL;
  
  /** The default free block pool power. */
  static const uint8_t DEFFPOW = 10;
```

```
KCHDBMAGICDATA  4B 43 0A 00
MOFFLIBVER      10 
MOFFLIBREV      0D 
MOFFFMTVER      05 
MOFFCHKSUM      BC 
MOFFTYPE        30 
MOFFAPOW        03 
MOFFFPOW        0A = 10
MOFFOPTS        00 00 00 00 00 = 0
MOFFBNUM        00 00 00 00 00 10 00 07 = 1048583
MOFFFLAGS       00 00 00 00 00 00 00 00 
MOFFCOUNT       00 00 00 00 00 00 00 00 
MOFFSIZE        00 00 00 00 00 60 18 78 
MOFFOPAQUE      00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

```
HEADSIZ --------------------------\
FBPWIDTH --------------------------\
MOFFOPTS -> opts_ ----\             \
                       >---> width_ ---> boff_
TSMALL ---------------/             /
fpow_ -------> fbpnum_ ------------/
```
```
******* TSMALL
enum Option {
    TSMALL = 1 << 0,                     ///< use 32-bit addressing
    TLINEAR = 1 << 1,                    ///< use linear collision chaining
    TCOMPRESS = 1 << 2                   ///< compress each record
};

TSMALL = 1 << 0 = 1
```
```
******* opts_
load_meta():
    std::memcpy(&opts_, head + MOFFOPTS, sizeof(opts_)); 

opts_ = 0
```
```
******* width_
calc_meta():
    width_ = (opts_ & TSMALL) ? sizeof(uint32_t) : sizeof(uint32_t) + 2; 

width_ = (0 & 1) ? sizeof(uint32_t) : sizeof(uint32_t) + 2;)
width_ = (0) ? sizeof(uint32_t) : sizeof(uint32_t) + 2;
width_ = sizeof(uint32_t) + 2;
width_ = sizeof(uint32_t) + 2;
width_ = 4 + 2;
width_ = 6
```
```
******* fpow_
load_meta():
    std::memcpy(&fpow_, head + MOFFFPOW, sizeof(fpow_));
    
fpow_ = 10;
```

```
******* fbpnum_
calc_meta():
    fbpnum_ = fpow_ > 0 ? 1 << fpow_ : 0;

fbpnum_ = 10 > 0 ?  1 << fpow_ : 0;
fbpnum_ = 1 << 10
fbpnum_ = 1024
```

```
******* boff_
calc_meta(): 
    (a) boff_ = HEADSIZ + FBPWIDTH * fbpnum_;
    (b) if (fbpnum_ > 0) boff_ += width_ * 2 + sizeof(uint8_t) * 2;

(a) boff_ = 64 + 6 * 1024
boff_ = 6208

(b) if (1024 > 0) boff_ += width_ * 2 + sizeof(uint8_t) * 2;
boff_ = 6208 + 6 * 2 + 1 * 2;
boff_ = 6222
```

Ergo:
```
if (!file_.read_fast(boff_ + bidx * width_, buf, width_)) {
6222 + 580029 * 6 = 3486396
```

* NOTE: The correct offset is 3486399 and the above value is off by 3. The same difference appears for all control keys tested and I could not find out where it came from, so I'll just add 3 to the 'boff_' value (6222 + 3). If you find out where was my mistake, let me know. 


Now we know the offset where `kchashmgr` is supposed to insert the reference for a key "A" (or any other key for the matter). Now it's time to figure out the record's keys by brute-forcing the bucket index for all 3 character long strings:

```
#!/usr/bin/python3

from kyotocabinet import *
import binascii
import string

# Read [size] bytes and return a string representing it
def read_byte(fpointer, size=1):
    return binascii.hexlify(f.read(size))

# Get the file's offset of a bucket position
def get_bucket(key):
    hash = hash_murmur(key)
    bidx = hash % 1048583
    bucket = str(6225 + bidx * 6)

    return bucket


# First read the 'tokyo' file (renamed to 'tokyo.kch') to extract the bucket array positions
# related to the key/value pairs. This is calculated from the key's MurMurHash value.
temp = {}
count = 0
with open("tokyo.kch", "rb") as f:
    print("Reading 'tokyo.kch' file...")

    # We'll just ignore the header
    #file.seek
    value = read_byte(f, 48)
    count += 47

    value = read_byte(f)
    count += 1

    # Ignore the file area where the actual key/value pairs are stored
    while count < 6217331: 

        if value != b'00':
            b2 = read_byte(f)
            b3 = read_byte(f)
            value += b2 + b3
            #print(str(value) + " @ " + str(count))

            temp[str(count)] = str(value)
            count += 2

        value = read_byte(f)
        count += 1

# Order the dictionary in ascendig order of it's values
hash_insertion_order = sorted(temp.items(), key=lambda t: t[1])
print("> DONE")


# Knowing that the removed keys were 3 bytes long (or 3 chars), we'll calculate the
# bucket array position of every string with length 3
hashes = {}
for a in string.printable:
    for b in string.printable:
        for c in string.printable:
            key = a + b + c

            bucket = get_bucket(key)

            for hash, addr in hash_insertion_order:
                if bucket == hash:
                    print("key: " + str(key) + " => " + str(bucket))
                    hashes[str(key)] = str(bucket) 

```
Due to the fact that there where matches for strings from "000" to "088" (therefore matching the flag's length) among other orderless keys, we'll stablish that this is the key order for the flag's characters.


Examples of hash collisions:
```
# key: 073 => 877305
# key: 5_o => 877305
# key: dJb => 877305
# key: K=c => 877305
# key: Mqb => 877305
# key: Y0u => 877305
```

Alright we're almost there. Knowing the bucket array index of a key, the key order and having the reference to the record's value, we can rebuild the flag by doing:

> key --> bucket index --> bucket index value --> record offset --> record value (record offset + 19)
   
For the key "000":
> 000 --> 5938071 --> 0C0315 (787221 * 8 = 6297768 + 19 = 6297787) --> "A" 

For the key "001":
> 001 --> 6111519 --> 0C03AB (787371 * 8 = 6298968 + 19 = 6298987) --> "S"


Now going for the kill:

```
#!/usr/bin/python3

from kyotocabinet import *

def read_byte(fpointer, size=1):
    return binascii.hexlify(f.read(size))

def get_bucket(key):
    hash = hash_murmur(key)
    bidx = hash % 1048583
    bucket = str(6225 + bidx *6)

    return bucket

# Having the keys, generate each bucket position in the file
key_hashes = []
print("Creating the keys' hashes")
for key in range (0,89):
    key = '{0:03d}'.format(key)
    key_hashes.append(str(get_bucket(key)))

# Get the 3 bytes value from the bucket array position to find out
# the char's position at the end of the file
flag = []
with open("tokyo.kch", "rb") as f:
    
	for kh in key_hashes:
		f.seek(int(kh))

		position = int.from_bytes(f.read(3), byteorder='big') * 8
		
		# From the beginning, go to the position of the record + offset to the value part
		f.seek(position + 19, 0)
		print(f.read(1).decode('utf-8'), end='')

print() 
```

```
$ ./get_flag.py 
Creating the keys' hashes
ASIS{Kyoto_Cabinet___is___a_library_of_routines_for_managing_a_database_mad3_in_Japan!_!}
```
