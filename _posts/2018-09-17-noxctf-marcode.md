---
title: NoxCTF 2018 - Marcode
layout: post
date: '2018-09-17 08:50:00'
description: Writeup of Marcode
image: https://i.imgur.com/psQF3O3.png
tags:
- ctf
- writeup
author: shrimpgo
---
### "Marcode" (1000)

> Description:
> Marcode (Mr. Code in Hebrew), Ineed your help!
> I got a movie but I cant see it. It hypnotizes me.
> please help me!
> yours,
> Gveretcode (Mrs. Code in Hebrew)

> P.S.
> change NOXCTF to noxCTF.
> [Download](https://drive.google.com/open?id=1GkalBntU1s6d_sw_S5I4GinD8hBrh-C-)
---

This challenge resumes in a single video that every frame of it has a QRCode and with 2:19 of duration. It's not so hard to guess that we need to read all of this codes to get something (maybe the flag?). Spliting all of frames is very easy! Just run this command:

```bash
$ mkdir thumbs; cd thumbs
$ ffmpeg -i ../Marcode.mp4 thumb%04d.jpg -hide_banner
```

And folder `thumbs` will have 3490 files, labeled at format `thumb0001.jpg`. Now we have to read every QRCode generated in each file. We know there are so many QRCode readers spreaded on the Internet, but I'm looking for some tool in Linux for ease scripting and I found zbar-tools. It's a nice tool to read my QRCodes! Let's read one file to see how it works:

```bash
$ zbarimg thumb0001.jpg
QR-Code:https://drive.google.com/open?id=131OPuwNVggs0ZAsVie4dfA3k9zLWN-uu
scanned 1 barcode symbols from 1 images in 0.07 seconds
```

Nice! Now we have to see what's is inside of this link:

![alt text](https://i.imgur.com/nrSEBB6.png)

What? There is only a `T` letter? What about next QRCode?

```bash
$ zbarimg thumb0002.jpg
QR-Code:https://drive.google.com/open?id=1c-go7QIZb_yqGDsRKlXG1j1Py7-wcSFG
scanned 1 barcode symbols from 1 images in 0.04 seconds
```

![alt text](https://i.imgur.com/Qu292s3.png)

Now there is a `H` letter! Hmm... It's trying to tell something! I read other QRCodes and that confirms what I'm guessing. I made a lot of testing until get this command line above:

```bash
$ for i in $(ls thumb*.jpg); do u=$(zbarimg -q --raw $i | sed 's/open?id=/file\/d\//g;s/$/\/view/'); l=$(curl -s $u | egrep -o  '(\".\.png\")|(space.png)' | sed 's/\"//g;s/.png//g' | uniq); echo -n $l | sed 's/space/ /'; done
THE TWO MEN...
```

Waiting this to finish was a eternity for me, so I splited it out in 4 parts (0001 to 0999, 1000 to 1999, 2000 to 2999 and 3000 to 3490) and ran it in each terminal:

```bash
$ for i in $(ls thumb0*.jpg); do u=$(zbarimg -q --raw $i | sed 's/open?id=/file\/d\//g;s/$/\/view/'); l=$(curl -s $u | egrep -o  '(\".\.png\")|(space.png)' | sed 's/\"//g;s/.png//g' | uniq); echo -n $l | sed 's/space/ /'; done # first part
$ for i in $(ls thumb1*.jpg); do u=$(zbarimg -q --raw $i | sed 's/open?id=/file\/d\//g;s/$/\/view/'); l=$(curl -s $u | egrep -o  '(\".\.png\")|(space.png)' | sed 's/\"//g;s/.png//g' | uniq); echo -n $l | sed 's/space/ /'; done # second part
$ for i in $(ls thumb2*.jpg); do u=$(zbarimg -q --raw $i | sed 's/open?id=/file\/d\//g;s/$/\/view/'); l=$(curl -s $u | egrep -o  '(\".\.png\")|(space.png)' | sed 's/\"//g;s/.png//g' | uniq); echo -n $l | sed 's/space/ /'; done # third part
$ for i in $(ls thumb3*.jpg); do u=$(zbarimg -q --raw $i | sed 's/open?id=/file\/d\//g;s/$/\/view/'); l=$(curl -s $u | egrep -o  '(\".\.png\")|(space.png)' | sed 's/\"//g;s/.png//g' | uniq); echo -n $l | sed 's/space/ /'; done # fourth part
```

The final text:

```
THE TWO MEN APPEARED OUT OF NOWHERE A FEW YNARDS APART IN THE NARROW MOONLIT LANE FOR A SECOND THEY STOOD QUITE STOILL WANDS DIRECTED AT EACH OTHERS CHESTS THEN RECOGNIZING EACH OTHER THEY STOWED THEIR WANDS BENEATH THEIR CLOAKS AND STARTED WALKING BRISKLY IN THE SAME DIRECTION NEWS ASKED THE TALLEXR OF THE TWO THE BEST REPLIED SEVERUS SNAPE THE LANE WAS BORDERED ON THE LEFT BY WILD LOWGROWING BRAMBLES ON THE RIGHT BY A HICGH NEATLY MANICURED HEDGE THE MENS LONG CLOAKS FLAPPED AROUND THEIR ANKLES AS THEY MARCHED THOUGHT I MIGHTT BE LATE SAID YAXLEY HIS BLUNT FEATURES SLIDING IN AND OUT OF SIGHT AS THE BRANCHES OF OVERHANGING TREES BROKE THE MOONLIGHT IT WAS A LITTLE TRICKIER THAN I EXPECTEFD BUT I HOPE HE WILL BE SATISFIED YOU SOUND CONFIDENT THAT YOUR RECEPTION WILL BE GOOD SNAPE NODDED BUT DID NOT ELABORATE THEY TURNED RIGHT INTO A WIDE DRIVAEWAY THAT LED OFF THE LANE THE HIGH HEDGE CURVED INTO THEM RUNNIVNG OFF INTO THE DISTANCE BEYOND THE PIR OF IMPAOSING WROUGHTIRON GATES BARRING THE MENS WAY NEITDHER OF THEM BROKE STEP IN SILENCE BOTH RAISED THEIR LEFT ARMS IN A KIND OF SALAUTE AND PASSED STRAIGHT THROUGH AS THOUGH THE DARK METAL WAS SMOKE THE YEW HEDGES MUFFLED THE SOKUND OF THE MENS FOOTSTEPS THERE WAS A RUSTLE SOMEWHERE TO THEIR RIGHT YAXLEY DREW HIS WAND AGAIN POINTING IT OVER HIS COMPANIONS HEAD BUT THE SOURCE OF THE NOISE PROVED TO BE NOTHING MORE THAN A PUREWHITE PEACOCK STERUTTING MAJESTICALLY ALONG THE TOP OF THE HEDGE HE ALWAYS DID HIMSELF WELL LUCIUS PEACOCKS YAXLEY THRUST HIS WAND BADCK UNDER HIS CLOAK WITH A SNORT A HANDSOME MANOR HOUSE GREW OUT OF THE DARKNESS AT THE END OF THE STRAIGHT DRIVE LIGHTS GLINTING IN THE DIAMOND PANED DOWNSTAAIRS WINDOWS SOMEWHERE IN THE DARK GARDEN BEYOND THE HEDGE A FOUNTAIN WAS PLAYING GRAVEL CRACKLED BENEATH THEIR FEET AS SNAPE AND YAXLEY SPED TOWARD THE FRONT DOOR WHICH SWUNG INWARD AT THEIR APPROACH THOUGH NOBODY HAD VISIBLY OPENED VIT THE HALLWAY WAS LARGE DIMLY LIT AND SUMPTUOUSLY DECORATED WITH A MAGNIFICENT CARPET COVERING MOST OF THE STONE FLOOR THE EYES OF THE PALEFACED PORTRAITS ON THE WALL FOLLOWED SNAPE AND YAXLEY AS THEY STRODE PAST THE TWO MEN HALTED AT A HEAVY WOODEN DOOR LEADING INTO THE NEXT ROOM HESITATED FOR THE SPACE OF A HEARTBEAT THEN SNAPE TURNED THE BRONZE HANDLE THE DRAWING ROOM WAS FULL OF SILENT PREOPLE SITTING AT A LONG AND ORNATE TABLE THE ROOMS USUAL FURNITURE HAD BEEN PUSHED CARELESSLY UP AGAINST THE WALLS ILLUMINATION CAME FROM A ROARING FIRE BENEATH A HANDSOME MARBLE MANTELPIECE SURMOUNTED BY A GILDED MIRROR SNAPE AND YAXLEY LINGERED FOR A MOMENT ON THE THRESHOLD AS THEIR EYES GREW ACCUSTOMED TO THE LACK OF LIGHT THEY WERE DRAWN UPWARD TO THE STRANGEST FEATURE OF THE SCENE AN APPARENTLY UNCONSCIOUS HUMAN FIGURE HANGING UPSIDE DOWN OVER THE TABLE REVOLVING SLOWLY AS IF SUSPENDED BY AAN INVISIBLE ROPE AND REFLECTED IN THE MIRROR AND IN THE BARE POLISHED SURFACE OF THE TABLE BELOW NONE OF THE PEOPLE SEATED UNDERNEATH THIS SINGULAR SIGHT WERE LOOKING AT IT EXCEPT FOR A PALE YOUNG MAN SITTING ALMOST DIRECTLY BELOW IT HE SEEMED UNABLE TO PREVENT HIMSELF FROM GLANCING UPWARD EVERY MINUTE OR SO YAXLEY SNAPE SAID A HIGH CLEAR VOICE FROM THE HEAD OF THE TABLE YOU ARE VERY NEARLY LATE THE SPEAKER WAS SEATED DIRECTLY IN FRONT OF THE FIREPLACE SO THAT IT WAS DIFFICULT AT FIRST FOR THE NEW ARRIVALS TO MAKE OUT MORE THAN HIS SILHOUETTE
```

Er... where is the flag? I picked up some words and made a Google search on this and I've found out that is a stretch of a Harry Potter's book (Harry Potter and the Deathly Hallows - Chapter 1). I copied the original stretch and removed all of punctuation and used [diffchecker](https://diffchecker.com) to make this comparison easier.

![alt text](https://i.imgur.com/psQF3O3.png)

I noticed there are some words differents from original text with a single word of difference. When I get this words and put together following the same order forms `NOXCTF{AVADAKEDAVRA}`, but before submits the flag, we have to change `NOX` to `nox`, turning `noxCTF{AVADAKEDAVRA}`.
